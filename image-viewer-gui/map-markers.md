# Znaczniki na mapie

Zakładka „Mapa” wyświetla zdjęcia na interaktywnej mapie 2D na podstawie ich współrzędnych GPS. Zapewnia to przegląd geograficzny sesji zdjęciowej i pomaga w wizualizacji zasięgu przestrzennego. Jest to również przydatne podczas pierwszego importowania zdjęć, aby szybko usunąć te, których nie trzeba przetwarzać.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Dostęp do zakładki „Mapa”

1. Otwórz lub utwórz projekt w Chloros
2. Zaimportuj zdjęcia zawierające metadane GPS
3. Kliknij zakładkę **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na lewym pasku bocznym
4. Na mapie pojawią się znaczniki w lokalizacjach GPS poszczególnych zdjęć

{% hint style="info" %}
**Wymagany GPS**: Na mapie pojawią się tylko zdjęcia z wbudowanymi współrzędnymi GPS w metadanych EXIF. Upewnij się, że podczas robienia zdjęć w aparacie włączona jest funkcja GPS.
{% endhint %}

***

## Dostosowywanie zdjęć z zakładki Mapa

Zakładka **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ma tę samą funkcję dodawania  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  i usuwania  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  jak zakładka [**Przeglądarka plików**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Wyświetla również tę samą listę plików projektu, ale z innymi nagłówkami kolumn:

### Nazwa pliku

* Oryginalna nazwa pliku z aparatu
* Zachowuje konwencję nazewniczą aparatu (np. IMG\_0001.RAW)

### Szerokość geograficzna

* Szerokość geograficzna zdjęcia

### Długość geograficzna

* Długość geograficzna zdjęcia

### Wysokość

* Wysokość zdjęcia

{% hint style="info" %}
Kliknięcie nagłówków kolumn tabeli powoduje również sortowanie danych w wierszach
{% endhint %}

***

## Znaczniki zdjęć

Każde zdjęcie z danymi GPS jest reprezentowane przez znacznik na mapie:

### Wyświetlanie znaczników

* Znaczniki wskazują dokładne współrzędne GPS miejsca, w którym zostało zrobione dane zdjęcie
* Znaczniki mogą się grupować przy pomniejszeniu widoku
* Powiększ, aby zobaczyć lokalizacje poszczególnych zdjęć

{% hint style="success" %}
SUPER-ZOOM: Po osiągnięciu maksymalnego poziomu powiększenia oferowanego przez dostawcę kafelków mapy, kafelek jest powiększany przy dalszym powiększaniu, co pozwala zobaczyć znaczniki znajdujące się blisko siebie.
{% endhint %}

### Podgląd po najechaniu kursorem

* **Najedź kursorem myszy** na dowolny znacznik, aby wyświetlić miniaturę tego zdjęcia
* Pozwala to na szybką identyfikację wizualną bez opuszczania widoku mapy
* Przydatne do lokalizowania konkretnych zdjęć w ramach dużej sesji zdjęciowej

***

## Dostawcy kafelków map

{% hint style="success" %}
**Wybór automatyczny**: Chloros automatycznie wybiera usługę kafelkową, która zapewnia najlepszy poziom powiększenia dla aktualnej lokalizacji na mapie. W razie potrzeby można ręcznie przełączać się między dostawcami.
{% endhint %}

Zakładka Mapa obsługuje dwóch dostawców kafelków dla obrazów tła mapy:

### Google Maps

* Standardowe zdjęcia satelitarne i mapy od Google
* Najlepsze do ogólnego pokrycia na całym świecie

### ESRI

* Zdjęcia satelitarne i lotnicze z ESRI ArcGIS
* Często zapewniają zdjęcia o wyższej rozdzielczości w niektórych regionach

***

## Typy kafelków mapy

Możesz wybrać typ warstwy mapy (od lewej do prawej):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Teren

Wyświetla profile wysokościowe i kafelki mapy ze szczegółami (drogi itp.)

### Mapa

Wyświetla standardowe (o niższej przepustowości) kafelki mapy ze szczegółami (drogi itp.)

### Satelita

Wyświetla szczegółowe (o wyższej przepustowości) kafelki mapy satelitarnej

### Hybrydowa

Wyświetla kafelki mapy satelitarnej z dodanymi szczegółami (drogi itp.)

***

## Nawigacja po mapie

### Elementy sterujące powiększeniem

* **Powiększanie/pomniejszanie**: Użyj kółka myszy lub przycisków powiększania
* **Pełny ekran**: Wyświetl mapę na pełnym ekranie

### Elementy sterujące przesuwaniem

* **Przesuwanie**: Kliknij i przeciągnij, aby poruszać się po mapie***

## Przykłady zastosowań

### Wizualizacja trasy lotu

* Wyświetlanie obszaru pokrycia sesji zdjęciowych z drona
* Identyfikacja luk w pokryciu obrazowym
* Weryfikacja przebiegu trasy lotu

### Przegląd pomiarów naziemnych

* Wyświetlanie rozmieszczenia przestrzennego zdjęć naziemnych
* Lokalizowanie obrazów celów kalibracyjnych względem obszaru pomiarowego
* Planowanie dodatkowych lokalizacji zdjęciowych

### Kontrola jakości

* Szybka identyfikacja zdjęć wykonanych w nieoczekiwanych lokalizacjach
* Weryfikacja dokładności GPS w całym zbiorze danych
* Porównanie lokalizacji zdjęć z notatkami terenowymi

***

## Rozwiązywanie problemów

### Brak wyświetlania znaczników

**Możliwe przyczyny:**

* Zdjęcia nie zawierają metadanych GPS
* Podczas rejestracji w aparacie wyłączono GPS
* Dane EXIF zostały usunięte przez oprogramowanie zewnętrzne

**Rozwiązanie**: Sprawdź, czy GPS jest włączony w aparacie i ponownie zaimportuj oryginalne pliki

### Znaczniki w niewłaściwej lokalizacji

**Możliwe przyczyny:**

* GPS aparatu miał słaby sygnał satelitarny
* Drift GPS podczas rejestracji

**Rozwiązanie**: Jest to zazwyczaj problem związany z momentem rejestracji; rozważ użycie GPS PPK/RTK do zastosowań wymagających precyzji
