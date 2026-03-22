# Dynamiczne dostosowanie mocy obliczeniowej

Wersja Chloros 1.1.0 wprowadza inteligentne wykrywanie sprzętu oraz automatyczny wybór strategii przetwarzania. Silnik przetwarzania dostosowuje się do posiadanego sprzętu — od Jetson Nano po stację roboczą z wieloma procesorami graficznymi — bez konieczności ręcznej konfiguracji.

***

## Jak to działa

Po uruchomieniu Chloros automatycznie profiluje system:

1. **Wykrywa system operacyjny** — Windows lub Linux
2. **Identyfikuje rdzenie procesora i całkowitą pamięć RAM**

3.**Wykrywa obecność procesora graficznego** — obsługa NVIDIA CUDA, pamięć VRAM, model
4. **Identyfikuje model Jetson** (jeśli dotyczy) — za pośrednictwem `/proc/device-tree/model`
5. **Sprawdza czujniki temperatury** (Jetson) — w celu przetwarzania z uwzględnieniem temperatury
6. **Wybiera optymalną strategię obliczeniową** — na podstawie całego wykrytego sprzętu
7. **Konfiguruje liczbę procesów roboczych, typ potoku i alokację pamięci** automatycznie

Wynik jest buforowany, dzięki czemu kolejne uruchomienia rozpoczynają się szybciej. Jeśli sprzęt ulegnie zmianie (np. zostanie dodany procesor graficzny), Chloros ponownie profiluje podczas następnego uruchomienia.

***

## Strategie obliczeniowe

Chloros wybiera jedną z trzech strategii obliczeniowych w oparciu o posiadany sprzęt:

| Strategia | Wymagany procesor graficzny (GPU) | Pracownicy | Potok | Najlepsze zastosowanie |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Tak (12 GB+ pamięci VRAM lub 16 GB+ współdzielonej) | 3-4 | `fused_gpu` | Karty graficzne do komputerów stacjonarnych z 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Tak (&lt; 12 GB pamięci VRAM) | 1–3 | `tiled_gpu` | Karty graficzne klasy podstawowej, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nie | rdzenie - 1 | `cpu_fallback` | Systemy bez procesora graficznego NVIDIA |

### Typy potoków

* **`fused_gpu`** — Pełna ścieżka przetwarzania na procesorze graficznym. Wszystkie operacje debayeringu, korekcji i indeksowania są wykonywane na procesorze graficznym w jednym połączonym przebiegu. Najwyższa przepustowość, ale wymaga więcej pamięci VRAM.
* **`tiled_gpu`** — Ścieżka procesora graficznego oszczędzająca pamięć. Przetwarza obrazy w postaci kafelków, aby zmieścić się w ograniczonej pamięci procesora graficznego. Niższa przepustowość, ale działa na urządzeniach z ograniczoną pamięcią.
* **`cpu_fallback`** — Przetwarzanie wyłącznie na procesorze (CPU) z wykorzystaniem wielowątkowej równoległości. Stosowane, gdy nie jest dostępny procesor graficzny NVIDIA.***

## Zachowanie specyficzne dla platformy

| Platforma | Strategia | Pracownicy | Potok | Uwagi |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (szeregowe) | Tryb oszczędzający pamięć, przetwarza jeden obraz na raz |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (współbieżny) | Zalecane urządzenie brzegowe — prawdziwe równoległe przetwarzanie na procesorze graficznym |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (współbieżny) | Maksymalna wydajność brzegowa |
| **Komputer stacjonarny z procesorem graficznym 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobra wydajność komputera stacjonarnego z efektywnymi pod względem pamięci kafelkami |
| **Komputer stacjonarny z kartą graficzną 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optymalna wydajność na komputerze stacjonarnym |
| **System wyłącznie z procesorem** | `CPU_PARALLEL` | rdzenie – 1 | `cpu_fallback` | Nie wymaga procesora graficznego, wykorzystuje ThreadPool |

{% hint style="info" %}
**Pamięć zunifikowana Jetson**: Urządzenia Jetson współdzielą pamięć GPU i CPU. Jetson Orin NX 16 GB zgłasza ~15,3 GB pamięci VRAM, ale jest to ta sama fizyczna pamięć RAM, z której korzystają procesy systemu operacyjnego i procesora. Chloros uwzględnia to podczas ustalania progów alokacji pamięci.
{% endhint %}

***

## Dynamiczny przydział pamięci GPU

Chloros wykorzystuje [4-wątkowy potok przetwarzania](processing-pipeline.md):

* **Wątek 1** (Wykrywanie) — Ładowanie obrazu, parsowanie EXIF, wykrywanie obiektu
* **Wątek 2** (Kalibracja) — Obliczenia kalibracji odbicia
* **Wątek 3** (Przetwarzanie) — debayering na GPU, korekcja winietowania, obliczanie indeksu
* **Wątek 4** (Eksport) — zapis plików, osadzanie metadanych

Gdy wcześniejsze wątki potoku zakończą swoją pracę (np. wszystkie obrazy zostały wykryte), przydzielona im pamięć GPU zostaje zwolniona i **przekazana pozostałym aktywnym wątkom**. Oznacza to, że wątek 3 (etap wymagający dużej mocy obliczeniowej GPU) otrzymuje stopniowo więcej pamięci w miarę postępu potoku, co poprawia przepustowość dla zadań wymagających największej mocy obliczeniowej.

### Etapy alokacji

| Etap | Aktywne wątki | Rozkład pamięci GPU |
| --- | --- | --- |
| **Wczesny** | 1, 2, 3, 4 | Podział między wszystkie wątki |
| **Środkowo-wczesny** | 2, 3, 4 | Pamięć wątku 1 redystrybuowana |
| **Środkowo-późny** | 3, 4 | Pamięć wątków 1+2 trafia do 3+4 |
| **Późny** | 3 lub 4 | Maksymalna pamięć dla pozostałego wątku |***

## Przetwarzanie z uwzględnieniem tekstur

Metoda debayeringu z uwzględnieniem tekstur (tylko Chloros+) zużywa znacznie więcej pamięci GPU niż metoda standardowa ze względu na model odszumiania AI/ML:

* Systemy z **&lt; 7 GB pamięci VRAM** są zmuszone do pracy w pętli przetwarzania synchronicznego w trybie z uwzględnieniem tekstur (jeden obraz na raz)
* Systemy z **pamięcią VRAM powyżej 7 GB** mogą przetwarzać dane z uwzględnieniem tekstur równolegle, choć przy zmniejszonej liczbie procesów roboczych w porównaniu z trybem standardowym***

## Zarządzanie temperaturą (Jetson)

Urządzenia Jetson mają ograniczenia termiczne, zwłaszcza w instalacjach zamkniętych lub powietrznych. Chloros monitoruje temperatury procesora graficznego i procesora centralnego oraz automatycznie dostosowuje przetwarzanie:

| Temperatura | Reakcja |
| --- | --- |
| **&lt; 70°C** | Normalna praca — pełna prędkość |
| **70°C** (Ostrzeżenie) | Zmniejszenie rozmiaru partii |
| **80°C** (Krytyczne) | Agresywne dławienie — zmniejszenie współbieżności i liczby procesów |
| **90°C** (Wyłączenie) | Całkowite zatrzymanie przetwarzania przez GPU |

Monitorowanie temperatury wykorzystuje `tegrastats` na platformach Jetson. W systemach stacjonarnych z odpowiednim chłodzeniem dławienie termiczne jest rzadko uruchamiane.

***

## Obsługa obciążenia pamięci

Chloros monitoruje obciążenie pamięci systemowej podczas przetwarzania:

* **Próg pamięci**: Wykorzystanie na poziomie 85% uruchamia zachowanie konserwatywne
* **Redukcja OOM**: Jeśli wystąpi zdarzenie braku pamięci, alokacja jest zmniejszana o 25% (mnożnik 0,75x)
* **Przełączenie potoku**: W przypadku poważnego obciążenia pamięci potok automatycznie przechodzi z `fused_gpu` do `tiled_gpu`
* **Zalecenia dotyczące pamięci wymiany**: Na Jetson Chloros ostrzega, jeśli przestrzeń wymiany jest niewystarczająca dla rozmiaru zbioru danych***

## Monitorowanie adaptacji obliczeniowej

### Wynik statusu CLI

Po rozpoczęciu przetwarzania CLI wyświetla wykryty profil sprzętowy:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostyka systemu

Uruchom `chloros-cli selftest`, aby wyświetlić pełny profil sprzętowy i zweryfikować możliwości obliczeniowe:

```bash
chloros-cli selftest
```

Sprawdza to dostępność CUDA, pamięć GPU, modele odszumiania oraz łączność z zapleczem.

***

## Kolejne kroki

* [Potok przetwarzania](processing-pipeline.md) — Zrozumienie architektury potoku 4-wątkowego
* [Przewodnik po NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Wdrażanie i optymalizacja specyficzne dla Jetson
* [CLI : Wiersz poleceń](../CLI.md) — Pełna dokumentacja CLI
