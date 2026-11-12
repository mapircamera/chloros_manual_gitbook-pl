# Dynamiczne dostosowanie mocy obliczeniowej W wersji 1.3.0 programu

Chloros wykorzystywane jest wykrywanie sprzętu oraz automatyczny wybór strategii przetwarzania. Silnik przetwarzania dostosowuje się do posiadanego sprzętu — od Jetson Orin Nano po stację roboczą z wieloma procesorami graficznymi — bez konieczności ręcznej konfiguracji.

***

## Jak to działa

Po uruchomieniu Chloros przeprowadza profilowanie systemu:

1. **Wykrywa system operacyjny** — Windows lub Linux
2. **Identyfikuje rdzenie procesora i całkowitą pamięć RAM**

3.**Wykrywa obecność procesora graficznego** — obsługa NVIDIA CUDA, pamięć VRAM, model
4. **Identyfikuje model Jetson** (jeśli dotyczy) — poprzez `/proc/device-tree/model`
5. **Sprawdza czujniki temperatury** (Jetson) — w celu przetwarzania uwzględniającego temperaturę
6. **Wybiera strategię obliczeniową** — na podstawie całego wykrytego sprzętu
7. **Automatycznie konfiguruje liczbę procesów roboczych, typ potoku oraz alokację pamięci**

Wykryty profil jest buforowany na czas sesji w pamięci i na dysku, dzięki czemu kolejne uruchomienia rozpoczynają się szybciej:

| Platforma | Profil w pamięci podręcznej |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (uwzględnia `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Usuń ten plik, aby wymusić ponowne wykrycie — przydatne po dodaniu procesora graficznego (GPU) lub większej ilości pamięci RAM. Chloros automatycznie przeprowadza ponowne wykrywanie również wtedy, gdy pamięć podręczna została zapisana przez niekompatybilną starszą wersję.

***

## Strategie obliczeniowe

Chloros wybiera jedną z trzech strategii obliczeniowych w oparciu o posiadaną konfigurację sprzętową:

| Strategia | Wybierana, gdy | Pracownicy | Wykonawca | Potok |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| Karta graficzna CUDA zgłaszająca**12 GB+ pamięci VRAM**(w pamięci zunifikowanej Jetson, wymaga również co najmniej 12 GB współdzielonej pamięci RAM) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimum 2 —**ograniczone do 2 w Jetson** | Procesy robocze (oddzielne konteksty CUDA) | `fused_gpu` |
| **`GPU_SINGLE`**| Procesor graficzny CUDA z**2–12 GB pamięci VRAM**| 3 (nakładanie się operacji we/wy; dostęp do procesora graficznego sekwencjonowany za pomocą semafora).**1 (sekwencyjnie) na urządzeniach Jetson z mniej niż 12 GB pamięci RAM** | Procesy robocze (oddzielne konteksty CUDA); sekwencyjne w ramach procesu na urządzeniach Jetson z małą ilością pamięci RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Brak procesora graficznego z obsługą CUDA lub mniej niż 2 GB pamięci VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Przykłady zastosowania wzoru na liczbę procesorów roboczych `GPU_PARALLEL`: 12 GB pamięci VRAM → 2 procesory robocze, 16 GB → 3 procesory robocze, 20 GB+ → 4 procesory robocze. Każdemu procesowi roboczemu przydzielono 3,5 GB, co odpowiada zmierzonemu zapotrzebowaniu zadania wykorzystującego tekstury, w ramach 90% pamięci VRAM pozostałej po zarezerwowaniu 0,5 GB.

{% hint style="info" %}
**Strategie związane z GPU wymagają środowiska PyTorch obsługującego CUDA.** Urządzenie, na którym PyTorch nie ma dostępnego urządzenia CUDA, otrzymuje wartość `CPU_PARALLEL`, niezależnie od posiadanego procesora graficznego. Pakiet Jetson 1.3.0 wykorzystuje wersję PyTorch dla CUDA z JetPack 6, więc urządzenie Jetson kwalifikuje się po zainstalowaniu bibliotek CUDA i cuDNN z JetPack. Urządzenie Jetson z mniej niż 12 GB pamięci RAM zawsze przetwarza jeden obraz na raz, w ramach tego samego procesu (rejestrowane jako `GPU_SINGLE`).
{% endhint %}

Strategie oparte na GPU uruchamiają pulę procesów roboczych, z których każdy jest oddzielnym procesem systemu operacyjnego z własnym kontekstem CUDA; Chloros uruchamia je poprzez ponowne uruchomienie własnego zainstalowanego programu zaplecza (a nie poprzez `multiprocessing` w języku Python, którego uruchamianie procesów roboczych nie działa wewnątrz aplikacji zawartej w pakiecie). Strategia oparta na procesorze CPU wykorzystuje pulę wątków (funkcja `ThreadPoolExecutor` w języku Python). Chloros nie korzysta z żadnych zewnętrznych frameworków rozproszonych (takich jak Ray).

### Typy potoków

* **`fused_gpu`** — Ścieżka przetwarzania w całości na GPU. Operacje debayeringu, korekcji i indeksowania są wykonywane na GPU w jednym połączonym przebiegu. Najwyższa przepustowość, wymaga najwięcej pamięci VRAM.
* **`tiled_gpu`** — Ścieżka GPU oszczędzająca pamięć. Przetwarza obrazy w postaci kafelków, aby zmieścić się w ograniczonej pamięci GPU. Niższa przepustowość, ale działa na urządzeniach z ograniczoną pamięcią.
* **`cpu_fallback`** — Przetwarzanie wyłącznie na procesorze (CPU) z wykorzystaniem równoległości wielowątkowej. Stosowane, gdy nie jest dostępny procesor graficzny NVIDIA, oraz jako ostateczna opcja awaryjna w przypadku niepowodzenia obu ścieżek przetwarzania na procesorze graficznym.

Łańcuch rozwiązań awaryjnych w czasie wykonywania to zawsze `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ręczne nadpisanie strategii

Ustaw zmienną środowiskową `CHLOROS_STRATEGY`, aby wymusić konkretną strategię — jest to specjalistyczne rozwiązanie awaryjne na wypadek, gdy automatyczne wykrywanie wybierze opcję nieodpowiednią dla danej sytuacji (na przykład w celu pozostawienia procesora graficznego wolnego dla innych zadań):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Zmienna jest dopasowywana bez uwzględniania wielkości liter; wszystko, co nie jest jedną z trzech nazw, jest ignorowane, a automatyczne wykrywanie przebiega normalnie. W przypadku nadpisania Chloros nadal wybiera za Ciebie liczbę procesów roboczych:

| Nadpisanie | Zastosowana liczba procesów roboczych |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Nadpisanie `GPU_SINGLE` lub `GPU_PARALLEL` na komputerze bez dostępnego urządzenia CUDA zostanie odrzucone, co zostanie odnotowane w dzienniku jako wiersz `[STRATEGY]`, a uruchomienie wykorzystuje `CPU_PARALLEL`.

Zaleca się ustawianie tego parametru dla poszczególnych poleceń, a nie na stałe, aby normalne uruchomienia mogły nadal automatycznie dostosowywać się.

***

## Zachowanie specyficzne dla platformy

| Platforma | Strategia | Pracownicy | Potok | Uwagi |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekwencyjny) | Tryb oszczędzający pamięć, jeden obraz na raz |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekwencyjny) | Wspólna pamięć RAM poniżej 12 GB wymusza przetwarzanie sekwencyjne |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (równoczesne) | Zalecane urządzenie brzegowe — ograniczenie Jetson do 2 procesów roboczych |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (równoczesne) | Maksymalna wydajność w sieci brzegowej (również ograniczenie Jetson do 2 procesów roboczych) |
| **Komputer stacjonarny z kartą graficzną 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 procesy nakładają się na operacje wejścia/wyjścia, podczas gdy semafor sekwencjonuje dostęp do procesora graficznego |
| **Komputer stacjonarny z kartą graficzną o pojemności 12 GB lub większej** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (równoczesne) | Optymalna wydajność komputera stacjonarnego: 12 GB → 2 procesy, 16 GB → 3, 20 GB+ → 4 |
| **System oparty wyłącznie na procesorze** | `CPU_PARALLEL` | rdzenie fizyczne − 1 (min. 2) | `cpu_fallback` | Nie wymaga procesora graficznego, wykorzystuje pulę wątków |

{% hint style="info" %}
**Pamięć zunifikowana Jetson**: Urządzenia Jetson współdzielą pamięć GPU i CPU. Urządzenie Jetson Orin NX z 16 GB pamięci zgłasza około 15,3 GB pamięci VRAM, ale jest to ta sama fizyczna pamięć RAM, z której korzystają system operacyjny i procesy procesora. Dlatego urządzenia Jetson o pojemności 16 GB i większej kwalifikują się do `GPU_PARALLEL` tak samo jak procesory graficzne komputerów stacjonarnych o pojemności 12 GB i większej, ale są ograniczone do 2 procesów roboczych — procesor graficzny, procesy robocze oraz ich konteksty CUDA dla poszczególnych procesów czerpią z tej samej wspólnej puli.
{% endhint %}

### Budżet procesora graficznego w zależności od pamięci VRAM (dyskretne procesory graficzne)

Na hostach x86_64 z dyskretnym procesorem graficznym NVIDIA wykryta pamięć VRAM określa również, jaką część zasobów obliczeniowych karty można przydzielić oraz jak duże mogą być partie danych:

| Wykryta pamięć VRAM | Górna granica budżetu GPU | Współczynnik wielkości partii |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Poniżej 2 GB** | 70% | ×1,0 |

Dyskretne procesory graficzne rezerwują dla systemu jedynie 0,5 GB, ponieważ nie współdzielą pamięci RAM systemu. Profile Jetson rezerwują znacznie więcej pamięci i mają niższe limity — zobacz [Przewodnik po NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamiczny przydział pamięci GPU

Chloros wykorzystuje [4-wątkowy potok przetwarzania](processing-pipeline.md):

* **Wątek 1** (Wykrywanie) — ładowanie obrazu, analizowanie danych EXIF, wykrywanie obiektów
* **Wątek 2** (Kalibracja) — Obliczenia związane z kalibracją odbicia światła
* **Wątek 3** (Przetwarzanie) — usuwanie efektu debayera przez GPU, korekcja winietowania, obliczanie indeksu
* **Wątek 4** (Eksport) — zapis plików, osadzanie metadanych

Wątki 1, 2 i 4 zużywają niewiele zasobów GPU; Wątek 3 jest najbardziej obciążający. Gdy wcześniejsze wątki potoku kończą działanie, ich budżet GPU jest **przekazywany pozostałym aktywnym wątkom**, dzięki czemu wątek 3 otrzymuje coraz więcej pamięci w miarę postępu przetwarzania.

### Etapy alokacji

| Etap | Aktywne wątki | Rozkład pamięci GPU |
| --- | --- | --- |
| **Wczesny** | 1, 2, 3, 4 | Rozdzielona między wszystkie wątki, w większości przydzielona do wątku 3 |
| **Środkowo-wczesny** | 2, 3, 4 | Udział wątku 1 jest redystrybuowany |
| **Środkowo-późny** | 3, 4 | Udziały wątków 1 i 2 trafiają do wątków 3 i 4 |
| **Późny** | 3 lub 4 | Ostatni aktywny wątek otrzymuje maksymalny przydział |

Liczby te podlegają dwóm zasadom:

* Wątek, który jest **jedynym** aktywnym, otrzymuje maksymalny przydział określony w profilu.
* Gdy aktywnych jest więcej niż jedno *obciążające* zadanie GPU, podstawowy przydział każdego z nich jest dzielony między te zadania (nigdy poniżej skonfigurowanego minimum).

Wartością faktycznie stosowaną w czasie wykonywania jest **niższa** z dwóch wartości: przydział określony w profilu platformy oraz aktualna rekomendacja z monitora pamięci GPU, więc obciążona karta zawsze ma pierwszeństwo przed zbyt optymistycznym profilem.

***

## Przetwarzanie z uwzględnieniem tekstur

Moduł debayera z obsługą tekstur (**tylko Chloros+** — `--debayer texture-aware`) uruchamia model odszumiania oparty na sztucznej inteligencji/uczeniu maszynowym, który wymaga około 1,75 GB pamięci VRAM w trybie FP16 na każdą kopię, więc zużywa znacznie więcej pamięci GPU niż metoda standardowa:

* Systemy z **poniżej 7 GB pamięci VRAM**przetwarzają tryb z uwzględnieniem tekstur w**pętli synchronicznej, po jednym obrazie na raz** — nie zmieści się kilka kopii modelu, a pula procesów roboczych tylko zwiększyłaby opresję
* Systemy z **pamięcią VRAM powyżej 7 GB** mogą przetwarzać algorytm Texture Aware równolegle, choć przy mniejszej liczbie procesów roboczych w porównaniu z metodą Standard
* Na platformie **Jetson** funkcja Texture Aware jest zawsze przypisana do jednego procesora roboczego
* W przypadku **braku dostępnego procesora graficznego z obsługą CUDA** operacja wykorzystuje standardowy algorytm debayeringu, a backend rejestruje jeden wiersz `[DEBAYER] … needs a CUDA GPU …` wyjaśniający przyczynę
* Na urządzeniach Jetson o niskim poborze mocy (Nano, Orin Nano) interfejs CLI nakłada ograniczenie częstotliwości procesora graficznego po uruchomieniu trybu „Texture Aware” — zobacz [Przewodnik po NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Zarządzanie temperaturą (Jetson)

Urządzenia Jetson mają ograniczenia termiczne, zwłaszcza w instalacjach zamkniętych lub pokładowych. Chloros monitoruje wbudowane czujniki temperatury Jetsona i automatycznie skaluje rozmiary partii:

| Temperatura | Reakcja |
| --- | --- |
| **< 70°C** | Normalna praca — pełna prędkość |
| **70°C** (Ostrzeżenie) | Wielkość partii jest stopniowo zmniejszana (100% → 50% w zakresie od 70°C do 80°C) |
| **80°C** (Stan krytyczny) | Agresywne ograniczanie wydajności (50% → 0% w zakresie od 80°C do 90°C) |
| **90°C** (Wyłączenie) | Całkowite wstrzymanie przetwarzania przez procesor graficzny |

W systemach stacjonarnych z odpowiednim chłodzeniem dławienie termiczne rzadko jest uruchamiane.

***

## Obsługa obciążenia pamięci

Program Chloros nieustannie monitoruje pamięć GPU podczas przetwarzania i reaguje na trzech poziomach.

**Wielkość partii.** Partia zaczyna się od 8 obrazów pomnożonych przez mnożnik platformy z powyższych tabel. Następnie Chloros sprawdza wolną pamięć VRAM, rezerwuje 20% jej pojemności na własne obciążenie PyTorcha i zakłada około 100 MB pamięci GPU na obraz o rozdzielczości 12 MP — wielkość partii jest równa mniejszej z dwóch wartości: limitowi wynikającemu z pamięci lub wartości bazowej platformy. Nigdy nie spada poniżej 1.

**Redukcja prewencyjna.**Przy**wykorzystaniu pamięci VRAM powyżej 85%** rozmiary partii są zmniejszane, zanim dojdzie do jakiejkolwiek awarii.**Ograniczanie alokacji na wątek.** W miarę wzrostu aktualnego wykorzystania budżet GPU każdego wątku jest zmniejszany: ×0,75 przy wykorzystaniu powyżej 80%, ×0,5 przy wykorzystaniu powyżej 90%. Przedziały monitorowania to 70% (konserwatywny), 85% (normalny limit operacyjny) oraz 95% (ryzyko braku pamięci).**Wycofanie i odzyskiwanie po braku pamięci.** Jeśli mimo wszystko wystąpi zdarzenie braku pamięci:

* rozmiar partii zostaje **zmniejszony o połowę**, a przy każdym kolejnym przypadku braku pamięci — ponownie o połowę; każda kolejna pomyślnie przetworzona partia cofa tę karę o jeden stopień
* przydziały dla aktywnych wątków są ograniczane do 70% ich bieżącej wartości, a alokator przechodzi na strategię konserwatywną, łagodząc ograniczenia po serii pomyślnych przydziałów
* w warunkach silnego obciążenia potok przechodzi z trybu `fused_gpu` do `tiled_gpu`, a w ostateczności do `cpu_fallback`

**Pamięć RAM hosta (Jetson).** Przed rozpoczęciem przetwarzania interfejs CLI szacuje szczytowe zapotrzebowanie na pamięć hosta na podstawie liczby obrazów i trybu debayeringu oraz ostrzega, jeśli pamięć RAM wraz z przestrzenią wymiany opartą na plikach może okazać się niewystarczająca, wyświetlając dokładne polecenia służące do dodania przestrzeni wymiany — zobacz [Podręcznik NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Monitorowanie adaptacji obliczeniowej

### Diagnostyka systemu

`chloros-cli selftest` to najszybszy sposób na sprawdzenie, co widzi warstwa obliczeniowa:

```bash
chloros-cli selftest
```

Jego 7 testów obejmuje wersję, dostępność portów, uruchomienie backendu, `/api/test`, informacje o systemie, obecność modelu odszumiającego oraz gotowość CUDA i odszumiającego. Test 5 wyświetla bezpośrednio linię sprzętową:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Sprawdzenie 7 wyświetla `CUDA: <bool>, Denoiser: <bool>` — oba muszą być prawdziwe, aby funkcja Texture Aware mogła w ogóle działać.

### Logi backendu

Strategia i liczba procesów roboczych są wybierane wewnątrz backendu na początku każdego uruchomienia — nie ma komunikatu w interfejsie CLI, który by je ogłaszał. Gdy coś zachowuje się nieoczekiwanie (przejście na ścieżkę rezerwową GPU, błąd OOM, nieudane załadowanie modułu usuwania szumów), pojawia się to w logu backendu dla tej sesji:

| Platforma | Lokalizacja logu |
| --- | --- |
| **System Linux/Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jeden plik na uruchomienie) |
| **Linux, backend uruchomiony z poziomu CLI** | również `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Postęp na żywo

Podczas działania CLI wyświetla na bieżąco postęp dla poszczególnych wątków (wykrywanie, analiza, przetwarzanie, eksport) przesyłany za pośrednictwem Server-Sent Events — to praktyczny wskaźnik pozwalający stwierdzić, czy wątek 3 stanowi wąskie gardło. Zobacz [Potok przetwarzania](processing-pipeline.md).

***

## Kolejne kroki

* [Potok przetwarzania](processing-pipeline.md) — Omówienie architektury potoku z 4 wątkami
* [Przewodnik po platformie NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Wdrażanie i optymalizacja specyficzne dla Jetson
* [CLI: wiersz poleceń](../CLI.md) — Przewodnik po CLI

* [Opis CLI](../reference/cli-reference.md) — Wyczerpująca lista poleceń dla wersji 1.3.0
