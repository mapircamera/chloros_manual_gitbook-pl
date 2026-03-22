# Potok przetwarzania

Chloros 1.1.0 wykorzystuje 4-wątkowy potok przetwarzania, który działa na zasadzie etapowej linii montażowej. Każdy wątek obsługuje odrębną fazę procesu przetwarzania, co pozwala na równoczesne przetwarzanie wielu obrazów na różnych etapach.

***

## Architektura potoku

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Każdy obraz przechodzi kolejno przez wszystkie cztery wątki. Dzięki wielowątkowemu przetwarzaniu w Chloros+ wiele obrazów może znajdować się jednocześnie w różnych wątkach — podczas gdy wątek 3 przetwarza jeden obraz, wątek 1 może wykrywać następny, wątek 2 może kalibrować kolejny, a wątek 4 może zapisywać wcześniej przetworzony obraz na dysk.

***

## Szczegóły dotyczące wątków

### Wątek 1: Wykrywanie

**Cel**: Ładowanie obrazów i wykrywanie celów kalibracyjnych.

* Odczytuje pliki obrazów z dysku (RAW, JPG)
* Wyodrębnia metadane EXIF (GPS, model aparatu, znaczniki czasu, ekspozycja)
* Wykrywa cele kalibracyjne ArUco na oznaczonych obrazach docelowych
* Wyniki: dane obrazu + metadane + wyniki wykrywania celów

Jest to przede wszystkim wątek związany z operacjami wejścia/wyjścia i obciążający procesor.

### Wątek 2: Kalibracja

**Cel**: Obliczanie parametrów kalibracyjnych na podstawie wykrytych celów.

* Oblicza współczynniki kalibracji odbicia światła na podstawie obrazów celów
* Oblicza parametry korekcji winietowania
* Określa krzywe kalibracji dla poszczególnych pasm
* Wyniki: parametry kalibracji dla każdego obrazu

Jest to wątek obliczeniowy obciążony pracą procesora.

### Wątek 3: Przetwarzanie (GPU)

**Cel**: Zastosowanie korekt i obliczenie wskaźników wegetacyjnych.**Jest to wątek wymagający największej mocy obliczeniowej.*** **Debayering**: Konwertuje dane w formacie RAW z wzorem Bayera na obrazy wielokanałowe
  * Standardowy (szybki, średnia jakość) — domyślny
  * Z uwzględnieniem tekstury (powolny, najwyższa jakość) — tylko Chloros+, wykorzystuje odszumianie AI/ML
* **Korekcja winietowania**: Stosuje korekcję winietowania obiektywu na całym obrazie
* **Kalibracja odbicia**: Stosuje współczynniki kalibracji w celu konwersji na wartości odbicia
* **Obliczanie wskaźników**: Oblicza wskaźniki wegetacji (NDVI, NDRE, GNDVI itp.)
* Wyniki: przetworzone dane obrazu gotowe do eksportu

Ten wątek czerpie największe korzyści z przyspieszenia GPU. System [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optymalizuje przede wszystkim działanie tego wątku.

### Wątek 4: Eksport

**Cel**: Zapisanie przetworzonych obrazów na dysk.

* Zapisuje pliki wyjściowe w wybranym formacie (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Osadza metadane EXIF w plikach wyjściowych (GPS, znaczniki czasu, parametry przetwarzania)
* Organizuje dane wyjściowe w podfolderach według modeli aparatów
* Wyniki: ostateczne pliki na dysku

Jest to przede wszystkim wątek ograniczony operacjami wejścia/wyjścia. Pamięć SSD znacznie poprawia wydajność wątku 4.

***

## Przetwarzanie sekwencyjne a przetwarzanie potokowe

### Tryb darmowy (sekwencyjny)

W darmowej wersji Chloros obrazy są przetwarzane **po jednym**, sekwencyjnie przez wszystkie cztery etapy:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Pasek postępu w interfejsie graficznym pokazuje 2 etapy: wykrywanie celu i przetwarzanie.

### Tryb Chloros+ (potokowy)

Z licencją Chloros+ wszystkie cztery wątki działają **równocześnie** na różnych obrazach:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Pasek postępu w interfejsie graficznym pokazuje 4 etapy: wykrywanie, analizę, kalibrację i eksport. Najedź kursorem na pasek postępu, aby zobaczyć postęp dla poszczególnych wątków.

{% hint style="success" %}
**Przetwarzanie potokowe z Chloros+** może być 3–5 razy szybsze niż przetwarzanie sekwencyjne, w zależności od sprzętu i rozmiaru zbioru danych. Największe przyspieszenie występuje w systemach z szybkimi procesorami graficznymi i dyskami SSD.
{% endhint %}

***

## Postęp eksportu w wątku 4

W Chloros 1.1.0 wątek eksportu (wątek 4) ma własny, dedykowany system śledzenia postępu. Postęp eksportu można monitorować oddzielnie:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Przetwarzanie jest zakończone, gdy wątek 4 osiągnie 100%.

***

## Związek z dynamiczną adaptacją obliczeniową

System [dynamicznej adaptacji obliczeniowej](dynamic-compute-adaptation.md) wpływa przede wszystkim na **wątek 3 (przetwarzanie)**:

* Strategia **`GPU_PARALLEL`**: wątek 3 przetwarza jednocześnie wiele obrazów za pomocą procesora graficznego, korzystając z potoku `fused_gpu`
* Strategia **`GPU_SINGLE`**: Wątek 3 przetwarza po jednym obrazie na raz, korzystając z wydajnego pod względem pamięci potoku `tiled_gpu`
* Strategia **`CPU_PARALLEL`**: Wątek 3 wykorzystuje przetwarzanie oparte na procesorze z równoległością wielowątkową

Alokacja pamięci GPU w wątku 3 również zmienia się dynamicznie w miarę kończenia pracy wątków 1 i 2 — zobacz [Dynamiczna alokacja pamięci GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Kolejne kroki

* [Dynamiczna adaptacja obliczeniowa](dynamic-compute-adaptation.md) — Jak Chloros wybiera optymalną strategię dla danego sprzętu
* [Przewodnik po platformie NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Zachowanie potoku specyficzne dla platformy Jetson
* [Monitorowanie przetwarzania](../processing-images-gui/monitoring-the-processing.md) — Monitorowanie postępu za pomocą interfejsu graficznego
