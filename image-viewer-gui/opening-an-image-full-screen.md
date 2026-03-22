# Otwieranie obrazu na pełnym ekranie

Przeglądarka obrazów Chloros oferuje dedykowany interfejs pełnoekranowy do przeglądania, analizowania i edycji obrazów wielospektralnych. Niezależnie od tego, czy przeglądasz oryginalne obrazy, czy przetworzone wyniki, przeglądarka obrazów zapewnia zaawansowane narzędzia do kontroli i analizy.

## Dostęp do przeglądarki obrazów

### Z poziomu przeglądarki plików

Najpopularniejszy sposób otwierania obrazu w przeglądarce obrazów:

1. Upewnij się, że znajdujesz się w zakładce **Przeglądarka plików** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknij dowolną **miniaturę obrazu** w siatce obrazów
3. Obraz otworzy się w **głównym obszarze podglądu** (środek ekranu)
4. Obraz jest teraz załadowany i gotowy do wyświetlenia na pełnym ekranie

### Otwieranie zakładki przeglądarki obrazów

Po załadowaniu obrazu w obszarze podglądu:

1. Kliknij ikonę **przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na pasku bocznym po lewej stronie
2. Otworzy się zakładka przeglądarki obrazów, wyświetlając wybrany obraz na pełnym ekranie
3. Na pasku bocznym po lewej stronie pojawią się zaawansowane narzędzia do przeglądania i analizy

***

## Przegląd interfejsu przeglądarki obrazów

### Główny obszar wyświetlania

Największa część ekranu pokazuje obraz:

* **Pełna rozdzielczość**: Obrazy wyświetlane w natywnej rozdzielczości
* **Możliwość powiększania**: użyj elementów sterujących lub kółka myszy, aby powiększyć
* **Możliwość przesuwania**: kliknij i przeciągnij, aby przesuwać obraz po powiększeniu
* **Zachowanie proporcji**: obrazy są skalowane proporcjonalnie***

## Opcje przeglądania

### Podstawowa nawigacja po obrazach

#### Przeglądanie obrazów

Poruszaj się po zestawie obrazów za pomocą skrótów klawiaturowych lub przycisków:

* **Następny obraz**: Kliknij przycisk → lub naciśnij klawisz**→** (strzałka w prawo)
* **Poprzedni obraz**: Kliknij przycisk ← lub naciśnij klawisz**←** (strzałka w lewo)
* **Przejdź do konkretnego obrazu**: Wróć do przeglądarki plików i kliknij wybraną miniaturę

#### Elementy sterujące powiększeniem

Dostosuj powiększenie, aby obejrzeć szczegóły obrazu:

**Powiększ:*** Kliknij przycisk **+** (plus)
* Naciśnij klawisz **+**lub**=*** Przewiń kółkiem myszy **w górę**

**Pomniejsz:*** Kliknij przycisk **−** (minus)
* Naciśnij klawisz **−** (minus)
* Przewiń kółkiem myszy **w dół**

#### Przesuwanie po powiększeniu

Po powiększeniu poza rozmiar ekranu:

1. Przesuń kursor myszy nad obraz
2. Kliknij i **przytrzymaj lewy przycisk myszy**

3.**Przeciągnij**, aby przesuwać obraz
4. Zwolnij przycisk, aby zatrzymać przesuwanie

**Alternatywnie**: Użyj klawiszy strzałek, aby przesuwać obraz małymi krokami***

## Sprawdzanie wartości pikseli

### Wyświetlanie wartości pikseli w miejscu kursora

Podczas przesuwania kursora myszy nad obrazem wartości pikseli wyświetlają się w czasie rzeczywistym:**Miejsce wyświetlania wartości:*** **Liczba zmiennoprzecinkowa i czerwona linia w legendzie gradientu LUT po prawej stronie*** **Przy dalszym powiększeniu – wartość zmiennoprzecinkowa w pobliżu kursora i podświetlonego piksela*** Wyświetla wartości dla piksela **pod kursorem lub podświetlonego*** Aktualizacja podczas przesuwania myszy

***

## Typy obrazów, które można wyświetlać

### JPG

**Obrazy JPG z aparatu:**

* Wyświetlanie danych JPG zgodnie z podglądem
* Wyświetlanie oryginalnych, niepoprawionych wartości
* Przydatne do sprawdzania jakości obrazu przed przetworzeniem

### RAW (oryginalny)

### RAW (odbicie)

**Po przetworzeniu:**

* Skorygowana winieta
* Skalibrowana odbiciowość
* Wielopasmowy TIFF (Red, Green, NIR itp.)
* Dane naukowe gotowe do analizy

### RAW (Indeks)

**NDVI, NDRE, GNDVI itp. (pliki \_NDVI.tif):**

* Obrazy w skali szarości w jednym paśmie
* Wartości pikseli reprezentują wyniki obliczeń indeksu
* Zakres zazwyczaj od -1 do +1 dla indeksów znormalizowanych
* Możliwość zastosowania tabel LUT kolorów do wizualizacji

***

## Zastosowanie indeksów i tabel LUT

Zastosuj indeksy wielospektralne i tabele Look-Up kolorów:

1. Znajdź **Index/LUT Sandbox**w**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pasku bocznym
2. Wybierz indeks roślinności (NDVI, NDRE itp.)
3. Wybierz wzór wielospektralny lub utwórz własny (tylko Chloros+)
4. Zastosuj gradient kolorowej tabeli LUT w celu wizualizacji
5. Dostosuj zakresy wartości i progi

Szczegółowe instrukcje znajdziesz w [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Skróty klawiaturowe

### Nawigacja

* **→** (strzałka w prawo): następny obraz
* **←** (strzałka w lewo): poprzedni obraz
* **Home**: pierwszy obraz na liście
* **End**: Ostatni obraz na liście

### Powiększenie

* **+**lub**=**: Powiększ
* **−**: Pomniejsz
* **Kółko myszy**: Powiększ/pomniejsz***

### Sprawdzanie obliczeń indeksów

Sprawdź, czy indeksy zostały obliczone poprawnie:

1. Otwórz NDVI lub inny obraz indeksu
2. Sprawdź obszary roślinności:
   * **NDVI**: Powinien wykazywać wartość 0,4–0,9 dla zdrowych roślin
   * **NDRE**: Wyższe wartości dla bujnego wzrostu
   * **GNDVI**: Podobny do NDVI, ale wrażliwy na chlorofil
3. Sprawdź obszary bez roślinności:
   * **Gleba**: Blisko 0 lub lekko ujemna
   * **Woda**: Wartości ujemne (od -0,5 do 0)***

## Rozwiązywanie problemów z wyświetlaniem

### Obraz się nie otwiera

**Możliwe przyczyny:**

* Plik uszkodzony podczas przetwarzania
* Nieobsługiwany format pliku
* Niewystarczająca ilość pamięci dla dużego obrazu

**Rozwiązania:**

1. Spróbuj otworzyć plik w zewnętrznej przeglądarce, aby sprawdzić jego integralność
2. Sprawdź, czy format pliku odpowiada oczekiwanemu typowi
3. Zamknij inne aplikacje, aby zwolnić pamięć
4. Spróbuj otworzyć mniejszy/inny obraz

### Obraz wyświetla się jako czarny lub biały

**Możliwe przyczyny:**

* Zakres wartości wykracza poza możliwości wyświetlania
* Obraz 32-bitowy typu float z nietypowymi wartościami
* Błąd obliczeń indeksu

**Rozwiązania:**

1. Sprawdź wartości pikseli – jeśli wszystkie są bardzo niskie lub bardzo wysokie, dostosuj zakres wyświetlania
2. Spróbuj otworzyć plik w QGIS lub podobnym programie z automatyczną regulacją zakresu
3. Sprawdź dziennik debugowania z przetwarzania pod kątem błędów

### Wartości pikseli wydają się nieprawidłowe

**Możliwe przyczyny:**

* Wyświetlanie niewłaściwego obrazu (oryginalnego zamiast przetworzonego)
* Kalibracja nie została zastosowana poprawnie
* Dane z czujnika światła nie zostały uwzględnione w danych wejściowych
* Tryb procentowy został nieprawidłowo przełączony

**Rozwiązania:**

1. Sprawdź, czy przeglądasz przetworzony wynik (sprawdź rozszerzenie nazwy pliku)
2. Sprawdź stan przycisku trybu procentowego
3. Porównaj z obrazami o znanej poprawności z tego samego zestawu danych

***

## Kolejne kroki

Teraz, gdy możesz wyświetlać obrazy na pełnym ekranie:

* [**Warstwy obrazu**](image-layers.md) — dowiedz się więcej o wizualizacji wielopasmowej
* [**Piaskownica indeksów/LUT**](index-lut-sandbox.md) — zastosuj niestandardowe indeksy i mapowanie kolorów
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — zapoznaj się z dostępnymi indeksami

Aby zapoznać się z procesem przetwarzania, zobacz:

* [**Przetwarzanie obrazów (GUI)**](../processing-images-gui/adding-files-to-a-project.md) — kompletny przewodnik po przetwarzaniu
