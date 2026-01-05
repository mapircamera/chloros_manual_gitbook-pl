# Znaczniki mapy

Zakładka Mapa wyświetla obrazy na interaktywnej mapie 2D na podstawie ich współrzędnych GPS. Zapewnia to geograficzny przegląd sesji przechwytywania i pomaga wizualizować zasięg przestrzenny. Jest to również przydatne podczas pierwszego importowania obrazów, aby szybko usunąć te, które nie wymagają przetwarzania.

## Dostęp do zakładki Mapa

1. Otwórz lub utwórz projekt w Chloros.
2. Zaimportuj obrazy zawierające metadane GPS.
3. Kliknij zakładkę **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na pasku bocznym po lewej stronie
4. Mapa wyświetli znaczniki w lokalizacji GPS każdego zdjęcia

{% hint style=&quot;info&quot; %}
**Wymagane GPS**: Na mapie pojawią się tylko obrazy z wbudowanymi współrzędnymi GPS w metadanych EXIF. Upewnij się, że podczas robienia zdjęć w aparacie włączona jest funkcja GPS.
{% endhint %}

***

## Dostosowywanie obrazów z zakładki Mapa

Zakładka **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ma te same opcje dodawania  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  i usuwania  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  jak zakładka [**Przeglądarka plików**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Wyświetla również tę samą listę plików projektu, ale z innymi nagłówkami kolumn:

### Nazwa pliku

* Oryginalna nazwa pliku z aparatu
* Zachowuje konwencję nazewniczą aparatu (np. IMG\_0001.RAW)

### Szerokość geograficzna

* Szerokość geograficzna obrazu

### Długość geograficzna

* Długość geograficzna obrazu

### Wysokość

* Wysokość obrazu

{% hint style=&quot;info&quot; %}
Kliknięcie nagłówków kolumn tabeli powoduje również posortowanie danych w wierszach.
{% endhint %}

***

## Znaczniki obrazów

Każdy obraz z danymi GPS jest reprezentowany przez znacznik na mapie:

### Wyświetlanie znaczników

* Znaczniki wskazują dokładne współrzędne GPS, w których każdy obraz został zarejestrowany.
* Znaczniki mogą być zgrupowane po pomniejszeniu widoku.
* Powiększ widok, aby zobaczyć lokalizacje poszczególnych obrazów.

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Po osiągnięciu maksymalnego poziomu powiększenia dostarczanego przez dostawcę kafelków mapy, kafelek jest powiększany przy dalszym powiększaniu, co pozwala zobaczyć znaczniki, które znajdują się blisko siebie.
{% endhint %}

### Podgląd po najechaniu kursorem

* **Najedź myszką** na dowolny znacznik, aby wyświetlić miniaturę podglądu tego obrazu.
* Pozwala to na szybką identyfikację wizualną bez opuszczania widoku mapy.
* Przydatne do lokalizowania konkretnych obrazów w ramach dużej sesji przechwytywania.

***

## Dostawcy kafelków map

{% hint style=&quot;success&quot; %}
**Automatyczny wybór**: Chloros automatycznie wybiera usługę kafelków, która zapewnia najlepszy poziom powiększenia dla aktualnej lokalizacji na mapie. W razie potrzeby można ręcznie przełączać się między dostawcami.
{% endhint %}

Karta Mapa obsługuje dwóch dostawców kafelków dla obrazów mapy tła:

### Mapy Google

* Standardowe zdjęcia satelitarne i mapy od Google.
* Najlepsze rozwiązanie dla ogólnego pokrycia całego świata.

### ESRI

* Zdjęcia satelitarne i lotnicze od ESRI ArcGIS.
* Często zapewnia zdjęcia o wyższej rozdzielczości w niektórych regionach.

***

## Typy kafelków mapy

Możesz wybrać typ warstwy mapy (od lewej do prawej):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Teren

Wyświetla profile wysokościowe i kafelki mapy ze szczegółami (drogi itp.)

### Mapa

Wyświetla standardowe (niższa przepustowość) kafelki mapy ze szczegółami (drogi itp.)

### Satelita

Wyświetla szczegółowe (wyższa przepustowość) kafelki mapy satelitarnej

### Hybrydowa

Wyświetla kafelki mapy satelitarnej z dodatkowymi szczegółami (drogi itp.)

***

## Nawigacja po mapie

### Elementy sterujące powiększaniem

* **Powiększanie/pomniejszanie**: użyj kółka myszy lub przycisków powiększania
* **Pełny ekran**: wyświetl mapę na pełnym ekranie

### Elementy sterujące przesuwaniem

* **Przesuwanie**: kliknij i przeciągnij, aby przesuwać mapę***

## Przykłady zastosowań

### Wizualizacja trasy lotu

* Wyświetl obszar pokrycia sesji przechwytywania dronem
* Zidentyfikuj luki w pokryciu obrazu
* Sprawdź realizację trasy lotu

### Przegląd pomiarów naziemnych

* Zobacz rozmieszczenie przestrzenne przechwytywanych obrazów naziemnych
* Zlokalizuj obrazy kalibracyjne względem obszaru pomiarowego
* Zaplanuj dodatkowe lokalizacje przechwytywania

### Kontrola jakości

* Szybka identyfikacja zdjęć wykonanych w nieoczekiwanych lokalizacjach.
* Weryfikacja dokładności GPS w całym zbiorze danych.
* Porównanie lokalizacji zdjęć z notatkami terenowymi.

***

## Rozwiązywanie problemów

### Brak znaczników

**Możliwe przyczyny:**

* Zdjęcia nie zawierają metadanych GPS.
* GPS był wyłączony w aparacie podczas wykonywania zdjęć.
* Dane EXIF zostały usunięte przez oprogramowanie zewnętrzne.

**Rozwiązanie**: Sprawdź, czy GPS jest włączony w aparacie i ponownie zaimportuj oryginalne pliki.

### Znaczniki w niewłaściwej lokalizacji

**Możliwe przyczyny:**

* GPS aparatu miał słaby sygnał satelitarny.
* Drift GPS podczas rejestrowania.

**Rozwiązanie**: Jest to zazwyczaj problem związany z czasem rejestrowania; rozważ użycie GPS PPK/RTK do zastosowań wymagających precyzji.
