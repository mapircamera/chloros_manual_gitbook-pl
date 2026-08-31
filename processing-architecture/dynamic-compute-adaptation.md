# Dynamiczne dostosowanie mocy obliczeniowej

Chloros 1.2.0 wykorzystuje wykrywanie sprzętu i automatyczny wybór strategii przetwarzania. Silnik przetwarzania dostosowuje się do posiadanego sprzętu — od Jetson Orin Nano po stację roboczą z wieloma procesorami graficznymi — bez konieczności ręcznej konfiguracji.

***

## Jak to działa

Po uruchomieniu Chloros przeprowadza profilowanie systemu:

1. **Wykrywa system operacyjny** — Windows lub Linux
2. **Identyfikuje rdzenie procesora i całkowitą pojemność pamięci RAM**

3.**Wykrywa obecność procesora graficznego** — obsługę NVIDIA CUDA, pamięć VRAM, model
4. **Identyfikuje model Jetson** (jeśli dotyczy) — za pośrednictwem `/proc/device-tree/model`
5. **Sprawdza czujniki temperatury** (Jetson) — w celu przetwarzania z uwzględnieniem temperatury
6. **Wybiera strategię obliczeniową** — na podstawie całego wykrytego sprzętu
7. **Automatycznie konfiguruje liczbę procesów roboczych, typ potoku oraz alokację pamięci**

Wykryty profil jest buforowany dla bieżącej sesji w pamięci i na dysku, dzięki czemu kolejne uruchomienia rozpoczynają się szybciej:

| Platforma | Profil w pamięci podręcznej |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (uwzględnia `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Usuń ten plik, aby wymusić ponowne wykrycie — przydatne po dodaniu karty graficznej lub większej ilości pamięci RAM. Chloros również automatycznie ponownie wykrywa sprzęt, gdy pamięć podręczna została zapisana przez niekompatybilną starszą wersję.

***

## Strategie obliczeniowe

Chloros wybiera jedną z trzech strategii obliczeniowych w zależności od posiadanego sprzętu:

| Strategia | Wybierana, gdy | Pracownicy | Wykonawca | Potok |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| Karta graficzna CUDA zgłaszająca**12 GB+ pamięci VRAM**(w przypadku zunifikowanej pamięci Jetson wymaga również co najmniej 12 GB+ wspólnej pamięci RAM) | `min(4, VRAM ÷ 4GB)`, minimum 2 —**ograniczone do 2 na Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| Procesor graficzny CUDA z**2–12 GB pamięci VRAM**| 3 (nakładanie się operacji we/wy; dostęp do procesora graficznego sekwencjonowany za pomocą semafora).**1 (sekwencyjnie) na urządzeniach Jetson z mniej niż 12 GB pamięci RAM** | `ProcessPoolExecutor` (uruchomienie); sekwencyjnie w ramach tego samego procesu na urządzeniach Jetson z małą ilością pamięci RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Brak procesora graficznego CUDA lub mniej niż 2 GB pamięci VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Przykłady zastosowania wzoru na liczbę procesów roboczych `GPU_PARALLEL`: 12 GB pamięci VRAM → 3 procesy robocze, 16 GB+ → 4 procesy robocze, dowolny model Jetson → 2 procesy robocze.

Równoległość jest realizowana przy użyciu standardowego `concurrent.futures` z Python: strategie GPU wykorzystują `ProcessPoolExecutor` z metodą startową **spawn** (każdy procesor roboczy jest oddzielnym procesem z własnym kontekstem CUDA — `fork` skopiowałby już zainicjowany stan CUDA i uszkodziłby procesy potomne), a strategia procesora wykorzystuje `ThreadPoolExecutor`. Chloros nie wykorzystuje żadnych zewnętrznych frameworków rozproszonych (takich jak Ray).

### Rodzaje potoków

* **`fused_gpu`** — Pełna ścieżka przetwarzania na GPU. Operacje debayera, korekcji i indeksowania są wykonywane na GPU w jednym połączonym przebiegu. Najwyższa przepustowość, wymaga najwięcej pamięci VRAM.
* **`tiled_gpu`** — Ścieżka GPU oszczędzająca pamięć. Przetwarza obrazy w postaci kafelków, aby zmieścić je w ograniczonej pamięci GPU. Niższa przepustowość, ale działa na urządzeniach z ograniczoną pamięcią.
* **`cpu_fallback`** — Przetwarzanie wyłącznie na procesorze (CPU) z wykorzystaniem wielowątkowej równoległości. Stosowane, gdy nie jest dostępny procesor graficzny NVIDIA, oraz jako ostateczna opcja awaryjna w przypadku awarii obu ścieżek przetwarzania na procesorze graficznym.

Łańcuch rozwiązań awaryjnych w czasie wykonywania to zawsze `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ręczne nadpisanie strategii

Ustaw zmienną środowiskową `CHLOROS_STRATEGY`, aby wymusić określoną strategię — jest to specjalistyczne rozwiązanie awaryjne na wypadek, gdy automatyczne wykrywanie wybierze opcję nieodpowiednią dla danej sytuacji (na przykład w celu pozostawienia procesora graficznego wolnego dla innych zadań):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Zmienna jest dopasowywana bez uwzględniania wielkości liter; wszystko, co nie jest jedną z tych trzech nazw, jest ignorowane, a automatyczne wykrywanie przebiega normalnie. W przypadku nadpisania zmienna Chloros nadal wybiera liczbę procesów roboczych:

| Nadpisanie | Zastosowana liczba procesów roboczych |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Zalecamy ustawienie tego na poziomie poszczególnych poleceń, a nie na stałe, aby normalne uruchomienia mogły się nadal automatycznie dostosowywać.

***

## Zachowanie specyficzne dla platformy

| Platforma | Strategia | Pracownicy | Potok | Uwagi |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekwencyjny) | Tryb oszczędzający pamięć, jeden obraz na raz |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekwencyjny) | Wspólna pamięć RAM poniżej 12 GB wymusza przetwarzanie sekwencyjne |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (równoległe) | Zalecane urządzenie brzegowe — ograniczenie Jetson do 2 procesów roboczych |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (równoczesne) | Maksymalna wydajność na urządzeniu brzegowym (również ograniczenie Jetson do 2 procesorów roboczych) |
| **Komputer stacjonarny z kartą graficzną 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 procesy nakładają się na operacje wejścia/wyjścia, podczas gdy semafor sekwencjonuje dostęp do karty graficznej |
| **Komputer stacjonarny z kartą graficzną 12 GB+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (równoczesne) | Optymalna wydajność komputera stacjonarnego: 12 GB → 3 procesy robocze, 16 GB+ → 4 |
| **System oparty wyłącznie na procesorze** | `CPU_PARALLEL` | rdzenie fizyczne − 1 (min. 2) | `cpu_fallback` | Nie wymaga karty graficznej, wykorzystuje pulę wątków |

{% hint style="info" %}
**Pamięć zunifikowana Jetson**: Urządzenia Jetson współdzielą pamięć GPU i procesora. Urządzenie Jetson Orin NX 16 GB zgłasza około 15,3 GB pamięci VRAM, ale jest to ta sama fizyczna pamięć RAM, z której korzystają system operacyjny i procesy procesora. Dlatego urządzenia Jetson o pojemności 16 GB i większej kwalifikują się do `GPU_PARALLEL` tak samo jak procesory graficzne komputerów stacjonarnych o pojemności 12 GB i większej, ale są ograniczone do 2 procesów roboczych — procesor graficzny, procesy robocze oraz ich konteksty CUDA dla poszczególnych procesów czerpią z tej samej wspólnej puli.
{% endhint %}

### Budżet GPU według pamięci VRAM (dyskretne procesory graficzne)

Na hostach x86_64 z dyskretnym procesorem graficznym NVIDIA wykryta pamięć VRAM określa również, jaką część zasobów obliczeniowych karty można przydzielić oraz jak duże mogą być partie:

| Wykryta pamięć VRAM | Górny limit budżetu GPU | Współczynnik wielkości partii |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Poniżej 2 GB** | 70% | ×1,0 |

Dyskretne procesory graficzne rezerwują dla systemu jedynie 0,5 GB, ponieważ nie współdzielą pamięci RAM systemu. Profile Jetson rezerwują znacznie więcej pamięci, a limit jest niższy — zobacz [Przewodnik po NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamiczny przydział pamięci GPU

Chloros wykorzystuje [4-wątkowy potok przetwarzania](processing-pipeline.md):

* **Wątek 1** (Wykrywanie) — Ładowanie obrazu, analizowanie danych EXIF, wykrywanie obiektów
* **Wątek 2** (kalibracja) — obliczenia kalibracji współczynnika odbicia
* **Wątek 3** (przetwarzanie) — usuwanie efektu „bayer” przez GPU, korekcja winietowania, obliczanie indeksu
* **Wątek 4** (eksport) — zapis pliku, osadzanie metadanych

Wątki 1, 2 i 4 zużywają niewiele zasobów GPU; wątek 3 jest najbardziej wymagający. W miarę zakończenia pracy wcześniejszych wątków potoku ich przydział zasobów GPU jest **przekazywany pozostałym aktywnym wątkom**, dzięki czemu wątek 3 otrzymuje coraz więcej pamięci w miarę postępu przetwarzania.

### Etapy alokacji

| Etap | Aktywne wątki | Rozkład pamięci GPU |
| --- | --- | --- |
| **Wczesny** | 1, 2, 3, 4 | Rozdzielony między wszystkie wątki, większość przydzielona do wątku 3 |
| **Środkowo-wczesny** | 2, 3, 4 | Część wątku 1 jest redystrybuowana |
| **Środkowo-późny** | 3, 4 | Udziały wątków 1 i 2 trafiają do 3 i 4 |
| **Późny** | 3 lub 4 | Ostatni aktywny wątek otrzymuje maksymalny przydział |

Liczby podlegają dwóm zasadom:

* Wątek, który jest **jedynym** aktywnym, otrzymuje maksymalny przydział określony w profilu.
* Gdy aktywnych jest więcej niż jedno *obciążające* zadanie GPU, podstawowy przydział każdego z nich jest dzielony między nie (nigdy poniżej skonfigurowanego minimum).

Wartością faktycznie stosowaną w czasie wykonywania jest **niższa** z dwóch wartości: przydziału z profilu platformy oraz aktualnej rekomendacji z monitora pamięci GPU, więc obciążona karta zawsze ma pierwszeństwo przed optymistycznym profilem.***

## Przetwarzanie z uwzględnieniem tekstur

Debayer z obsługą tekstur (**Chloros+ wyłącznie** — `--debayer texture-aware`) uruchamia model odszumiania oparty na sztucznej inteligencji i uczeniu maszynowym, który wymaga około 1,75 GB pamięci VRAM w trybie FP16 na każdą kopię, więc zużywa znacznie więcej pamięci GPU niż metoda standardowa:

* Systemy z **poniżej 7 GB pamięci VRAM**przetwarzają algorytm z uwzględnieniem tekstury w**pętli synchronicznej, po jednym obrazie na raz** — nie ma miejsca na kilka kopii modelu, a pula procesów roboczych tylko zwiększyłaby rywalizację o zasoby
* Systemy z **ponad 7 GB pamięci VRAM** mogą przetwarzać Texture Aware równolegle, choć przy mniejszej liczbie procesów roboczych w porównaniu z metodą Standard
* Na platformie **Jetson** funkcja Texture Aware jest zawsze przypisana do pojedynczego pracownika, a w modelach o niskim poborze mocy (Nano, Orin Nano) automatycznie nakłada również ograniczenie częstotliwości procesora graficznego — zobacz [Przewodnik po platformie NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Zarządzanie temperaturą (Jetson)

Urządzenia Jetson mają ograniczenia termiczne, zwłaszcza w instalacjach zamkniętych lub lotniczych. Chloros monitoruje wbudowane czujniki temperatury Jetsona i automatycznie skaluje rozmiary partii:

| Temperatura | Reakcja |
| --- | --- |
| **&lt; 70°C** | Normalna praca — pełna prędkość |
| **70°C** (Ostrzeżenie) | Wielkość partii jest stopniowo zmniejszana (100% → 50% w zakresie od 70°C do 80°C) |
| **80°C** (Stan krytyczny) | Agresywne ograniczanie wydajności (z 50% do 0% w zakresie od 80°C do 90°C) |
| **90°C** (Wyłączenie) | Całkowite wstrzymanie przetwarzania przez procesor graficzny |

W systemach stacjonarnych z odpowiednim chłodzeniem dławienie termiczne jest uruchamiane rzadko.

***

## Obsługa obciążenia pamięci

Chloros stale monitoruje pamięć procesora graficznego podczas przetwarzania i reaguje na trzech poziomach.

**Wielkość partii.** Partia rozpoczyna się od 8 obrazów pomnożonych przez mnożnik platformy podany w tabelach powyżej. Chloros następnie sprawdza ilość wolnej pamięci VRAM, rezerwuje 20% jej pojemności na potrzeby PyTorcha i zakłada, że na każdy obraz o rozdzielczości 12 MP przypada około 100 MB pamięci GPU — wielkość partii jest równa mniejszej z dwóch wartości: limitowi wynikającemu z dostępnej pamięci lub wartości bazowej dla danej platformy. Nigdy nie spada poniżej 1.**Redukcja prewencyjna.**Powyżej**85% wykorzystania pamięci VRAM** rozmiary partii są zmniejszane, zanim dojdzie do jakiejkolwiek awarii.**Ograniczanie alokacji na wątek.** W miarę wzrostu aktualnego wykorzystania budżet GPU każdego wątku jest zmniejszany: ×0,75 przy wykorzystaniu powyżej 80%, ×0,5 przy wykorzystaniu powyżej 90%. Progi monitorowania to 70% (konserwatywny), 85% (normalny limit operacyjny) oraz 95% (ryzyko OOM).**Odsuwanie się i odzyskiwanie po OOM.** Jeśli mimo wszystko wystąpi zdarzenie braku pamięci:

* rozmiar partii jest **zmniejszany o połowę**, a przy każdym kolejnym przypadku braku pamięci — ponownie o połowę; każda kolejna pomyślnie przetworzona partia cofa tę karę o jeden stopień
* przydziały dla aktywnych wątków są ograniczane do 70% ich bieżącej wartości, a alokator przechodzi na strategię konserwatywną, łagodząc ją ponownie po serii pomyślnych przydziałów
* w warunkach znacznego obciążenia potok przechodzi z trybu `fused_gpu` do `tiled_gpu`, a w ostateczności do `cpu_fallback`

**Pamięć RAM hosta (Jetson).** Przed rozpoczęciem przetwarzania CLI szacuje szczytowe zapotrzebowanie na pamięć hosta na podstawie liczby obrazów i trybu debayera oraz ostrzega, jeśli pamięć RAM wraz z przestrzenią wymiany opartą na plikach może okazać się niewystarczająca, wyświetlając dokładne polecenia służące do dodania przestrzeni wymiany — zobacz [Przewodnik po NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

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

Strategia i liczba procesów roboczych są wybierane wewnątrz backendu na początku każdego przebiegu — nie ma komunikatu CLI, który by je ogłaszał. Gdy coś zachowuje się nieoczekiwanie (przejście na ścieżkę GPU, brak pamięci (OOM), nieudane załadowanie modułu usuwającego szumy), pojawia się to w logu backendu dla tej sesji:

| Platforma | Lokalizacja logu |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jeden plik na uruchomienie) |
| **Linux, CLI-uruchomiony backend** | również `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Postęp na żywo

Podczas działania CLI wyświetla na żywo postęp dla poszczególnych wątków (wykrywanie, analiza, przetwarzanie, eksport) przesyłany za pośrednictwem Server-Sent Events — jest to praktyczny wskaźnik pozwalający stwierdzić, czy wątek 3 stanowi wąskie gardło. Zobacz [Potok przetwarzania](processing-pipeline.md).

***

## Kolejne kroki

* [Potok przetwarzania](processing-pipeline.md) — Omówienie architektury potoku z 4 wątkami
* [Przewodnik po platformie NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — wdrażanie i optymalizacja specyficzne dla platformy Jetson
* [CLI: Wiersz poleceń](../CLI.md) — Przewodnik CLI
* [CLI — Opis funkcji](../reference/cli-reference.md) — Wyczerpująca lista poleceń dla wersji 1.2.0
