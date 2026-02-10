# Otwieranie obrazu na pełnym ekranie

Przeglądarka obrazów Chloros zapewnia dedykowany interfejs pełnoekranowy do przeglądania, analizowania i manipulowania obrazami wielospektralnymi. Niezależnie od tego, czy przeglądasz oryginalne obrazy, czy przetworzone wyniki, przeglądarka obrazów oferuje potężne narzędzia do kontroli i analizy.

## Dostęp do przeglądarki obrazów

### Z przeglądarki plików

Najpopularniejszy sposób otwierania obrazu w przeglądarce obrazów:

1. Upewnij się, że znajdujesz się w zakładce **Przeglądarka plików**. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknij dowolną **miniaturę obrazu** w siatce obrazów
3. Obraz otworzy się w **głównym obszarze podglądu** (środek ekranu)
4. Obraz został załadowany i jest gotowy do wyświetlenia na pełnym ekranie

### Otwieranie karty przeglądarki obrazów

Po załadowaniu obrazu w obszarze podglądu:

1. Kliknij ikonę **przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na lewym pasku bocznym.
2. Otworzy się zakładka przeglądarki obrazów, wyświetlając wybrany obraz na pełnym ekranie.
3. Zaawansowane narzędzia do przeglądania i analizy są dostępne na lewym pasku bocznym.

***

## Omówienie interfejsu przeglądarki obrazów

### Główny obszar wyświetlania

Największa część ekranu pokazuje obraz:

* **Pełna rozdzielczość**: obrazy wyświetlane w natywnej rozdzielczości.
* **Możliwość powiększania**: użyj elementów sterujących lub kółka myszy, aby powiększyć obraz
* **Możliwość przesuwania**: kliknij i przeciągnij, aby przesuwać obraz po powiększeniu
* **Zachowanie proporcji**: obrazy są skalowane proporcjonalnie***

## Opcje przeglądania

### Podstawowa nawigacja po obrazach

#### Przeglądanie obrazów

Poruszaj się po zestawie obrazów za pomocą skrótów klawiaturowych lub przycisków:

* **Następny obraz**: kliknij przycisk → lub naciśnij klawisz**→** (strzałka w prawo)
* **Poprzedni obraz**: kliknij przycisk ← lub naciśnij klawisz**←** (strzałka w lewo)
* **Przejdź do konkretnego obrazu**: wróć do przeglądarki plików i kliknij żądaną miniaturę

#### Elementy sterujące powiększaniem

Dostosuj powiększenie, aby sprawdzić szczegóły obrazu:

**Powiększ:*** Kliknij przycisk **+** (plus)
* Naciśnij klawisz **+**lub**=*** Przewiń kółkiem myszy **w górę**

**Pomniejsz:*** Kliknij przycisk **−** (minus)
* Naciśnij klawisz **−** (minus)
* Przewiń kółkiem myszy **w dół**

#### Przesuwanie po powiększeniu

Po powiększeniu ponad rozmiar ekranu:

1. Przesuń kursor myszy nad obraz.
2. Kliknij i **przytrzymaj lewy przycisk myszy**.
3. **Przeciągnij**, aby przesunąć obraz.
4. Zwolnij przycisk, aby zatrzymać przesuwanie.

**Alternatywa**: Użyj klawiszy strzałek, aby przesuwać obraz małymi krokami.***

## Sprawdzanie wartości pikseli

### Wyświetlanie wartości pikseli w miejscu kursora

Podczas przesuwania kursora myszy nad obrazem wartości pikseli są wyświetlane w czasie rzeczywistym:**Miejsce wyświetlania wartości:*** **Liczba zmiennoprzecinkowa i czerwona linia w legendzie gradientu indeksu LUT po prawej stronie*** **Po dalszym powiększeniu wartość zmiennoprzecinkowa w pobliżu kursora i podświetlonego piksela*** Wyświetla wartości pikseli **pod kursorem lub podświetlonych*** Aktualizacje podczas przesuwania myszy

***

## Typy obrazów, które można wyświetlać

### JPG

**Obrazy JPG z aparatu:**

* Wyświetlanie danych JPG jako podgląd
* Wyświetlanie oryginalnych, niepoprawionych wartości
* Przydatne do sprawdzania jakości obrazu przed przetwarzaniem

### RAW (oryginalny)

### RAW (odbicie)

**Po przetworzeniu:**

* Korekcja winietowania
* Kalibracja odbicia
* Wielopasmowy TIFF (Red, Green, NIR itp.)
* Dane naukowe gotowe do analizy

### RAW (indeks)

**NDVI, NDRE, GNDVI itp. (pliki \_NDVI.tif):**

* Obrazy w skali szarości w jednym paśmie
* Wartości pikseli reprezentują wyniki obliczeń indeksu
* Zakres zazwyczaj od -1 do +1 dla indeksów znormalizowanych
* Możliwość zastosowania kolorowych tabel LUT do wizualizacji

***

## Zastosowanie indeksu i tabeli LUT

Zastosowanie indeksów wielospektralnych i kolorowych tabel LUT:

1. Znajdź **Index/LUT Sandbox**w**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na pasku bocznym
2. Wybierz indeks roślinności (NDVI, NDRE itp.)
3. Wybierz formułę wielospektralną lub utwórz własną (tylko Chloros+)
4. Zastosuj gradient kolorów LUT do wizualizacji
5. Dostosuj zakresy wartości i progi

Szczegółowe instrukcje można znaleźć w sekcji [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Skróty klawiaturowe

### Nawigacja

* **→** (strzałka w prawo): następny obraz
* **←** (strzałka w lewo): poprzedni obraz
* **Home**: pierwszy obraz na liście
* **End**: ostatni obraz na liście

### Powiększanie

* **+**lub**=**: powiększanie
* **−**: pomniejszanie
* **Kółko myszy**: powiększanie/pomniejszanie***

### Sprawdzanie obliczeń indeksu

Sprawdź, czy indeksy zostały obliczone poprawnie:

1. Otwórz NDVI lub inny obraz indeksu
2. Sprawdź obszary roślinności:
   * **NDVI**: Powinien wykazywać 0,4-0,9 dla zdrowych roślin
   * **NDRE**: Wyższe wartości dla bujnego wzrostu
   * **GNDVI**: Podobnie jak NDVI, ale wrażliwy na chlorofil
3. Sprawdź obszary bez roślinności:
   * **Gleba**: Wartość bliska 0 lub lekko ujemna
   * **Woda**: Wartości ujemne (od -0,5 do 0)***

## Rozwiązywanie problemów z wyświetlaniem

### Obraz nie otwiera się

**Możliwe przyczyny:**

* Plik uszkodzony podczas przetwarzania
* Nieobsługiwany format pliku
* Niewystarczająca ilość pamięci dla dużego obrazu

**Rozwiązania:**

1. Spróbuj otworzyć plik w zewnętrznej przeglądarce, aby sprawdzić jego integralność.
2. Sprawdź, czy format pliku odpowiada oczekiwanemu typowi.
3. Zamknij inne aplikacje, aby zwolnić pamięć.
4. Spróbuj otworzyć mniejszy/inny obraz.

### Obraz wyświetlany jest czarno-biały

**Możliwe przyczyny:**

* Zakres wartości wykracza poza możliwości wyświetlacza.
* Obraz 32-bitowy typu float z nietypowymi wartościami.
* Błąd obliczeń indeksu.

**Rozwiązania:**

1. Sprawdź wartości pikseli — jeśli wszystkie są bardzo niskie lub bardzo wysokie, dostosuj zakres wyświetlania.
2. Spróbuj otworzyć plik w programie QGIS lub podobnym z automatyczną regulacją zakresu.
3. Sprawdź dziennik debugowania z przetwarzania pod kątem błędów.

### Wartości pikseli wydają się nieprawidłowe

**Możliwe przyczyny:**

* Wyświetlanie nieprawidłowego obrazu (oryginalnego zamiast przetworzonego).
* Kalibracja nie została zastosowana poprawnie.
* Dane z czujnika światła nie zostały uwzględnione w danych wejściowych.
* Tryb procentowy został nieprawidłowo przełączony.

**Rozwiązania:**

1. Sprawdź, czy przeglądasz przetworzone dane wyjściowe (sprawdź rozszerzenie nazwy pliku).
2. Sprawdź stan przycisku trybu procentowego.
3. Porównaj z obrazami o znanej poprawności z tego samego zestawu danych.

***

## Kolejne kroki

Teraz, gdy możesz wyświetlać obrazy na pełnym ekranie:

* [**Warstwy obrazu**](image-layers.md) — dowiedz się więcej o wizualizacji wielopasmowej
* [**Sandbox indeksów/LUT**](index-lut-sandbox.md) — zastosuj niestandardowe indeksy i mapowanie kolorów
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — zrozum dostępne indeksy

Aby zapoznać się z procesem przetwarzania, zobacz:

* [**Przetwarzanie obrazów (GUI)**](../processing-images-gui/adding-files-to-a-project.md) — kompletny przewodnik po przetwarzaniu
