# Potok przetwarzania

ChlorosWersja 1.2.0 wykorzystuje 4-wątkowy potok przetwarzania, który działa na zasadzie etapowej linii montażowej. Każdy wątek obsługuje odrębną fazę przepływu pracy, dzięki czemu kilka obrazów może jednocześnie znajdować się na różnych etapach przetwarzania.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektura potoku

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Każdy obraz przechodzi kolejno przez wszystkie cztery wątki. Dzięki wielowątkowemu przetwarzaniu w Chloros+ wiele obrazów zajmuje jednocześnie różne wątki — podczas gdy wątek 3 przetwarza jeden obraz, wątek 1 może wykrywać następny, wątek 2 kalibrować kolejny, a wątek 4 zapisywać gotowy obraz na dysk.

Postęp jest raportowany dla każdego wątku i przesyłany strumieniowo za pośrednictwem Server-Sent Events (backend publikuje je na `/api/events`). Na ekranie postępu w czasie rzeczywistym w CLI cztery etapy są oznaczone jako **Wykrywanie, Analiza, Przetwarzanie, Eksport**.***

## Szczegóły wątków

### Wątek 1: Wykrywanie

**Cel**: Wczytanie obrazów i wykrycie celów kalibracyjnych.

* Odczytuje pliki obrazów z dysku — pary Survey3 `.raw`+`.jpg`, zdjęcia z LATTICE `.tif`/`.tiff` oraz `.dng`
* Wyodrębnia metadane EXIF (GPS, model aparatu, sygnatury czasowe, ekspozycja)
* Wykrywa cele kalibracyjne: geometrie celów oznaczone symbolami ArUco dla ujęć LATTICE oraz klasyczny detektor panelowy dla zdjęć z celami kalibracyjnymi Survey3
* Wyniki: dane obrazu + metadane + wyniki wykrywania celów

Wątek obciążony głównie operacjami wejścia/wyjścia i obciążający procesor.

### Wątek 2: Kalibracja

**Cel**: Obliczanie parametrów kalibracyjnych na podstawie wykrytych celów.

* Oblicza współczynniki kalibracji odbicia na podstawie obrazów celów
* Oblicza parametry korekcji winietowania
* Określa krzywe kalibracji dla poszczególnych pasm
* Wyniki: parametry kalibracji dla każdego obrazu

Wątek obliczeniowy obciążony pracą procesora. Wątek 3 oczekuje na jego zakończenie, gdy włączona jest kalibracja odbicia, dzięki czemu współczynniki są gotowe przed przetworzeniem jakiegokolwiek obrazu.

### Wątek 3: Przetwarzanie (GPU)

**Cel**: Zastosowanie korekcji i obliczenie indeksów wegetacyjnych.**Jest to wątek wymagający największej mocy obliczeniowej.*** **Debayering**: konwersja danych RAW w formacie Bayera na obrazy wielokanałowe
  * Standardowy (szybki, średnia jakość) — domyślny, `--debayer standard`
  * Z uwzględnieniem tekstury (powolny, najwyższa jakość) — tylko Chloros+, `--debayer texture-aware`, wykorzystuje model odszumiania oparty na sztucznej inteligencji/uczeniu maszynowym
  * Zdjęcia LATTICE mono (M3M) są jednopasmowe: w ich przypadku pomijane są etapy demosaicowania i balansu bieli (z wyświetleniem jednowierszowego komunikatu dziennika), podczas gdy wszelkie obrazy M3C/Bayer z tej samej serii nadal przechodzą te etapy
* **Korekcja winietowania**: stosuje korekcję winietowania obiektywu w całym obrazie
* **Kalibracja odbicia**: stosuje współczynniki kalibracyjne w celu konwersji na wartości odbicia
* **Obliczanie wskaźników**: oblicza wskaźniki wegetacyjne (NDVI, NDRE, GNDVI, …)
* Wyniki: przetworzone dane obrazu gotowe do eksportu

Ten wątek w największym stopniu korzysta z przyspieszenia przez procesor graficzny i jest to wątek, który dostosowuje funkcja [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Wątek 4: Eksport

**Cel**: Zapisanie przetworzonych obrazów na dysk.

* Zapisuje pliki wyjściowe w wybranym formacie — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Osadza metadane w plikach wyjściowych (GPS, sygnatury czasowe, parametry przetwarzania)
* Organizuje pliki wyjściowe w folderze projektu jako `<camera>/<format>/<Product>_Images/` — na przykład `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Eksportowane pliki zachowują nazwę pliku źródłowego; folder identyfikuje produkt.**
* W przypadku przechwytywania obrazów za pomocą LATTICE jedna klatka źródłowa może zostać rozdzielona na kilka produktów (Debayered, Preview, Radiance, Reflectance, Index), z których każdy znajduje się we własnym folderze produktu
* Wyniki: pliki końcowe na dysku

Wątek ten jest przede wszystkim ograniczony przez operacje wejścia/wyjścia — pamięć SSD zauważalnie poprawia jego wydajność.

***

## Za kulisami: moduły wykonawcze

W ramach wątku 3 operacje na poszczególnych obrazach są równolegle przetwarzane przy użyciu standardowego modułu `concurrent.futures` z biblioteki „Python”:

* **Strategie wykorzystujące GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) wykorzystują metodę**spawn** — każdy proces roboczy jest oddzielnym procesem z własnym kontekstem CUDA (`fork` dziedziczyłby zainicjowany stan CUDA procesu nadrzędnego i uszkodziłby procesy potomne)
* **`CPU_PARALLEL`** wykorzystuje `ThreadPoolExecutor` — NumPy i OpenCV zwalniają blokadę GIL, więc wystarczą wątki
* Urządzenia Jetson z 8 GB lub mniejszą wspólną pamięcią RAM całkowicie pomijają egzekutora i przetwarzają dane w ramach tego samego procesu, sekwencyjnie
* Funkcja Texture Aware na procesorze graficznym z mniej niż 7 GB pamięci VRAM również działa sekwencyjnie — model odszumiający nie może zmieścić się więcej niż raz

Chlorosnie korzysta z żadnych zewnętrznych frameworków rozproszonych (takich jak Ray). Zobacz [Dynamic Compute Adaptation](dynamic-compute-adaptation.md), aby dowiedzieć się, w jaki sposób wybierana jest strategia i liczba procesów roboczych.

***

## Przetwarzanie sekwencyjne a przetwarzanie potokowe

### Tryb swobodny (sekwencyjny)

W bezpłatnej wersji Chloros obrazy są przetwarzane **po jednym na raz**, sekwencyjnie, przechodząc przez wszystkie cztery etapy:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

W trybie bezpłatnym interfejs graficzny wyświetla uproszczony pasek postępu; poszczególne fazy sekwencyjne są oznaczone jako **Wykrywanie celów**, a następnie**Przetwarzanie**.

### Tryb „Chloros”+ (potokowy)

Dzięki licencji „Chloros”+ wszystkie cztery wątki działają **równocześnie** na różnych obrazach:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Pasek postępu w interfejsie graficznym pokazuje cztery etapy; najedź na niego, aby zobaczyć postęp poszczególnych wątków. W interfejsie CLI te same cztery etapy są wyświetlane na żywo jako **Wykrywanie, Analiza, Przetwarzanie, Eksport**.

{% hint style="info" %}
**Jedna etykieta, dwie nazwy.** W pliku `CLI` etap 3 nosi nazwę _Przetwarzanie_. Kanał postępu w trybie premium backendu — ten, który wyświetla pasek postępu w interfejsie graficznym — oznacza ten sam etap jako _Kalibracja_. Są to te same wątki wykonujące tę samą pracę (Wątek 3: debayering, korekcje, indeksy).
{% endhint %}

{% hint style="success" %}
**Przetwarzanie potokowe z wykorzystaniem funkcji „Chloros” może być 3–5 razy szybsze niż przetwarzanie sekwencyjne, w zależności od sprzętu i rozmiaru zbioru danych.** Największe przyspieszenie występuje w systemach wyposażonych w szybkie procesory graficzne (GPU) i dyski SSD.
{% endhint %}

***

## Postęp eksportu w wątku 4

Wątek eksportu posiada własny system śledzenia postępu, który można sprawdzać oddzielnie:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Przetwarzanie jest zakończone, gdy wątek 4 osiągnie 100%.

{% hint style="info" %}
**Przebieg, w którym nie zapisano żadnych obrazów, kończy się niepowodzeniem.**W przypadku powodzenia `chloros-cli process` zgłasza, ile produktów graficznych zostało zapisanych (`Image products written: N`). Jeśli zamówiono produkty, a**żadnego**nie zapisano — tylko `project.json` i `calibration_data.json` — plik CLI wyświetla `Processing finished but wrote no image products.` i**kończy działanie z wynikiem niezerowym**, podając nazwę folderu projektu oraz typowe przyczyny (folder wejściowy nie został rozpoznany jako przechwycenie — sprawdź układ oraz `--input-level` — lub wszystkie żądane produkty były nieodpowiednie dla tych kamer). Skrypty mogą polegać na kodzie wyjścia.
{% endhint %}

***

## Związek z dynamiczną adaptacją obliczeniową

[Dynamiczna adaptacja obliczeniowa](dynamic-compute-adaptation.md) wpływa przede wszystkim na **wątek 3 (przetwarzanie)**:

* **`GPU_PARALLEL`**: Wątek 3 przetwarza jednocześnie wiele obrazów za pomocą procesora graficznego (GPU), korzystając z potoku `fused_gpu`
* **`GPU_SINGLE`**: Wątek 3 sekwencjonuje dostęp do procesora graficznego za pomocą semafora, podczas gdy procesy robocze nakładają się na operacje wejścia/wyjścia, wykorzystując potok `fused_gpu` lub oszczędzający pamięć potok `tiled_gpu`
* **`CPU_PARALLEL`**: Wątek 3 wykorzystuje przetwarzanie oparte na procesorze z równoległością wielowątkową

Przydział pamięci GPU dla wątku 3 również wzrasta w miarę kończenia pracy wątków 1 i 2 — zobacz [Dynamiczny przydział pamięci GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Kolejne kroki

* [Dynamiczna adaptacja obliczeniowa](dynamic-compute-adaptation.md) — Jak Chloros wybiera optymalną strategię dla danego sprzętu
* [Przewodnik po platformie NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Zachowanie potoku obliczeniowego specyficzne dla platformy Jetson
* [Monitorowanie przetwarzania](../processing-images-gui/monitoring-the-processing.md) — Monitorowanie postępu za pomocą interfejsu graficznego
* [Dokumentacja CLI](../reference/cli-reference.md) — `process`, `export-status`, kody wyjścia i układ danych wyjściowych
