# Ustawienia projektu

Pasek boczny Ustawienia projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> w Chloros umożliwiają skonfigurowanie wszystkich aspektów przetwarzania obrazu, wykrywania celów kalibracji, obliczeń indeksów wielospektralnych oraz opcji eksportu dla projektu. Ustawienia te są zapisywane wraz z projektem i mogą być zapisane jako szablony do ponownego wykorzystania w wielu projektach.

## Dostęp do ustawień projektu

Aby uzyskać dostęp do ustawień projektu:

1. Otwórz projekt w Chloros
2. Kliknij kartę **Ustawienia projektu**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. Panel ustawień wyświetli wszystkie dostępne opcje konfiguracyjne uporządkowane według kategorii

***

## Wykrywanie celów

Te ustawienia kontrolują sposób, w jaki Chloros wykrywa i przetwarza cele kalibracyjne w obrazach.

### Minimalny obszar próbki kalibracyjnej (px)

* **Typ**: Liczba
* **Zakres**: od 0 do 10 000 pikseli
* **Domyślnie**: 25 pikseli
* **Opis**: Ustawia minimalny obszar (w pikselach) wymagany, aby wykryty region został uznany za prawidłową próbkę celu kalibracji. Mniejsze wartości pozwolą wykrywać mniejsze cele, ale mogą zwiększyć liczbę fałszywych alarmów. Większe wartości wymagają większych, wyraźniejszych regionów docelowych do wykrycia.
* **Kiedy dostosować**:
  * Zwiększ, jeśli pojawiają się fałszywe wykrycia na małych artefaktach obrazu.
  * Zmniejsz, jeśli cele kalibracji wydają się małe na obrazach i nie są wykrywane.

### Minimalne grupowanie celów (0–100)

* **Typ**: Liczba
* **Zakres**: od 0 do 100
* **Wartość domyślna**: 60
* **Opis**: Kontroluje próg grupowania podobnych obszarów kolorystycznych podczas wykrywania celów kalibracji. Wyższe wartości wymagają grupowania bardziej podobnych kolorów, co skutkuje bardziej konserwatywnym wykrywaniem celów. Niższe wartości pozwalają na większą różnorodność kolorów w obrębie grupy celów.
* **Kiedy dostosować**:
  * Zwiększ, jeśli cele kalibracji są dzielone na wiele wykryć.
  * Zmniejsz, jeśli cele kalibracji z różnicami kolorów nie są w pełni wykrywane.

***

## Przetwarzanie

Te ustawienia kontrolują sposób przetwarzania i kalibracji obrazów przez Chloros.

### Korekcja winietowania

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Stosuje korekcję winietowania, aby skompensować przyciemnienie obiektywu na krawędziach obrazów. Winietowanie to powszechne zjawisko optyczne, w którym rogi i krawędzie obrazu wydają się ciemniejsze niż środek ze względu na właściwości obiektywu.
* **Kiedy wyłączyć**: Wyłącz tylko wtedy, gdy kombinacja aparatu/obiektywu zastosowała już korekcję winietowania lub jeśli chcesz ręcznie skorygować winietowanie w post-processingu.

### Kalibracja odbicia / balans bieli

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Włącza automatyczną kalibrację odbicia przy użyciu wykrytych celów kalibracyjnych na obrazach. Normalizuje to wartości odbicia w całym zestawie danych i zapewnia spójne pomiary niezależnie od warunków oświetleniowych.
* **Kiedy wyłączyć**: Wyłącz tylko wtedy, gdy chcesz przetwarzać surowe, nieskalibrowane obrazy lub jeśli używasz innego procesu kalibracji.

### Metoda debayerowania

* **Typ**: Wybór z listy rozwijanej
* **Opcje**:
  * Standardowa (szybka, średnia jakość)
  * Uwzględniająca teksturę (wolna, najwyższa jakość) \[Chloros+]
* **Domyślnie**: Standardowa (szybka, średnia jakość)
* **Opis**: Wybiera algorytm demosaicingu używany do konwersji surowych danych z czujnika wzorca Bayera na pełnokolorowe obrazy. Metoda „Standardowa (szybka, średnia jakość)” zapewnia optymalną równowagę między szybkością przetwarzania a jakością obrazu. Metoda „Uwzględniająca teksturę (wolna, najwyższa jakość)” \[Chloros+] wykorzystuje wysokiej jakości algorytm demosaicingu uwzględniający krawędzie w połączeniu z modelem redukcji szumów AI/ML, który usuwa prawie wszystkie szumy demosaicingu. Model uwzględniający teksturę wymaga pamięci GPU (VRAM) do działania. Zalecamy korzystanie z niego, gdy dostępna jest pamięć VRAM &gt;4 GB, aby przyspieszyć przetwarzanie.
* **Uwaga**: W przyszłych wersjach Chloros mogą zostać dodane dodatkowe metody debayeringu.

### Minimalny interwał ponownej kalibracji

* **Typ**: Liczba
* **Zakres**: od 0 do 3600 sekund
* **Domyślnie**: 0 sekund
* **Opis**: Ustawia minimalny interwał czasowy (w sekundach) między użyciem celów kalibracyjnych. Po ustawieniu wartości 0 program Chloros będzie używał wszystkich wykrytych celów kalibracyjnych. Po ustawieniu wyższej wartości program Chloros będzie używał tylko celów kalibracyjnych, które są oddzielone co najmniej taką liczbą sekund, co skraca czas przetwarzania zestawów danych z częstymi przechwytywaniami celów kalibracyjnych.
* **Kiedy dostosować**:
  * Ustaw wartość 0, aby uzyskać maksymalną dokładność kalibracji w zmiennych warunkach oświetleniowych
  * Zwiększ wartość (np. do 60–300 sekund), aby przyspieszyć przetwarzanie w przypadku stałego oświetlenia i częstych obrazów celów kalibracyjnych

### Przesunięcie strefy czasowej czujnika światła

* **Typ**: Liczba
* **Zakres**: od -12 do +12 godzin
* **Domyślnie**: 0 godzin
* **Opis**: Określa przesunięcie strefy czasowej (w godzinach od czasu UTC) dla znaczników czasu danych czujnika światła. Jest to używane podczas przetwarzania plików danych PPK (Post-Processed Kinematic) w celu zapewnienia prawidłowej synchronizacji czasu między przechwytywaniem obrazów a danymi GPS.
* **Kiedy dostosować**: Ustaw tę opcję zgodnie z przesunięciem strefy czasowej lokalnej, jeśli dane PPK wykorzystują czas lokalny zamiast czasu UTC. Na przykład:
  * Czas pacyficzny: -8 lub -7 (w zależności od czasu letniego)
  * Czas wschodnioamerykański: -5 lub -4 (w zależności od czasu letniego)
  * Czas środkowoeuropejski: +1 lub +2 (w zależności od czasu letniego)

### Zastosuj korekty PPK

* **Typ**: Pole wyboru
* **Domyślnie**: Wyłączone (niezaznaczone)
* **Opis**: Umożliwia stosowanie korekt kinematycznych przetworzonych po zakończeniu procesu (PPK) z rejestratorów MAPIR DAQ zawierających GPS (GNSS). Po włączeniu Chloros będzie używać wszystkich plików dziennika .daq zawierających dane dotyczące pinów ekspozycji w katalogu projektu i stosować precyzyjne korekty geolokalizacji do obrazów.
* **Wymagania**: plik dziennika .daq z wpisami dotyczącymi pinów ekspozycji musi znajdować się w katalogu projektu
* **Kiedy włączyć**: zaleca się, aby zawsze włączać korektę PPK, jeśli plik dziennika .daq zawiera wpisy dotyczące ekspozycji.

### Pin ekspozycji 1

* **Typ**: Wybór z listy rozwijanej
* **Widoczność**: Widoczne tylko wtedy, gdy włączona jest opcja „Zastosuj korekty PPK” ORAZ dostępne są dane ekspozycji dla pinu 1
* **Opcje**:
  * Nazwy modeli kamer wykrytych w projekcie
  * „Nie używaj” — zignoruj ten pin ekspozycji
* **Domyślnie**: wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: przypisuje określoną kamerę do pinu ekspozycji 1 w celu synchronizacji czasu PPK. Pin ekspozycji rejestruje dokładny czas wyzwolenia migawki aparatu, co ma kluczowe znaczenie dla dokładnej geolokalizacji PPK.
* **Zachowanie automatycznego wyboru**:
  * Pojedyncza kamera + pojedynczy pin: automatyczny wybór kamery
  * Pojedyncza kamera + dwa piny: pin 1 automatycznie przypisany do kamery
  * Wiele kamer: wymagany wybór ręczny

### Pin ekspozycji 2

* **Typ**: wybór z listy rozwijanej
* **Widoczność**: widoczny tylko wtedy, gdy włączona jest opcja „Zastosuj korekty PPK” ORAZ dostępne są dane ekspozycji dla pinu 2
* **Opcje**:
  * Nazwy modeli kamer wykrytych w projekcie
  * „Nie używaj” — zignoruj ten pin ekspozycji
* **Domyślnie**: Automatyczny wybór na podstawie konfiguracji projektu
* **Opis**: Przypisuje określoną kamerę do pinu ekspozycji 2 w celu synchronizacji czasu PPK podczas korzystania z konfiguracji z dwiema kamerami.
* **Zachowanie podczas automatycznego wyboru**:
  * Pojedyncza kamera + pojedynczy pin: Pin 2 automatycznie ustawiony na „Nie używaj”
  * Pojedyncza kamera + dwa piny: Pin 2 automatycznie ustawiony na „Nie używaj”
  * Wiele kamer: Wymagany wybór ręczny
* **Uwaga**: Ta sama kamera nie może być przypisana jednocześnie do pinu 1 i pinu 2.***

## Indeks

Te ustawienia umożliwiają konfigurację indeksów wielospektralnych do analizy i wizualizacji.

### Dodaj indeks

* **Typ**: Panel konfiguracji indeksów specjalnych
* **Opis**: Otwiera interaktywny panel, w którym można wybrać i skonfigurować wielospektralne indeksy roślinności (NDVI, NDRE, EVI itp.) do obliczenia podczas przetwarzania obrazu. Można dodać wiele wskaźników, każdy z własnymi ustawieniami wizualizacji.
* **Dostępne wskaźniki**: System zawiera ponad 30 predefiniowanych wskaźników wielospektralnych, w tym:
  * NDVI (znormalizowany wskaźnik różnicy wegetacji)
  * NDRE (znormalizowana różnica RedEdge)
  * EVI (ulepszony wskaźnik wegetacji)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * I wiele innych (pełna lista znajduje się w sekcji [Wzory indeksów wielospektralnych](multispectral-index-formulas.md))
* **Funkcje**:
  * Wybór spośród predefiniowanych wzorów indeksów
  * Konfiguracja gradientów kolorów wizualizacji (LUT — tabele przeglądowe)
  * Ustawianie wartości progowych do analizy
  * Tworzenie niestandardowych wzorów indeksów

### Formuły niestandardowe (funkcja Chloros+)

* **Typ**: Tablica definicji formuł niestandardowych
* **Opis**: Umożliwia tworzenie i zapisywanie niestandardowych formuł indeksów wielospektralnych przy użyciu matematyki pasmowej. Formuły niestandardowe są zapisywane wraz z ustawieniami projektu i mogą być używane tak samo jak wbudowane indeksy.
* **Jak utworzyć**:
  1. W panelu konfiguracji indeksu znajdź opcję formuły niestandardowej
  2. Zdefiniuj formułę za pomocą identyfikatorów pasm (np. NIR, Red, Green, Blue)
  3. Zapisz formułę pod opisową nazwą
* **Składnia formuły**: Obsługiwane są standardowe operacje matematyczne, w tym:
  * Arytmetyka: `+`, `-`, `*`, `/`
  * Nawiasy dla kolejności operacji
  * Odwołania do pasm: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Te ustawienia kontrolują format i jakość eksportowanych przetworzonych obrazów.

### Skalibrowany format obrazu

* **Typ**: Wybór z listy rozwijanej
* **Opcje**:
  * **TIFF (16-bit)** - Nieskompresowany format 16-bitowy TIFF
  * **TIFF (32-bitowy, procentowy)** — 32-bitowy format zmiennoprzecinkowy TIFF z wartościami odbicia wyrażonymi w procentach
  * **PNG (8-bitowy)** - Skompresowany format 8-bitowy PNG
  * **JPG (8-bitowy)** - Skompresowany format 8-bitowy JPEG
* **Domyślny**: TIFF (16-bitowy)
* **Opis**: Wybiera format pliku do zapisywania przetworzonych i skalibrowanych obrazów.
* **Zalecane formaty**:
  * **TIFF (16-bitowy)**: Zalecany do analiz naukowych i profesjonalnych procesów roboczych. Zachowuje maksymalną jakość danych bez artefaktów kompresji. Najlepszy do analizy wielospektralnej i dalszego przetwarzania w oprogramowaniu GIS.
  * **TIFF (32-bitowy, procentowy)**: Najlepszy do procesów wymagających wartości odbicia w procentach (0-100%). Zapewnia maksymalną precyzję pomiarów radiometrycznych.
  * **PNG (8-bit)**: Dobry do przeglądania w Internecie i ogólnej wizualizacji. Mniejsze rozmiary plików dzięki bezstratnej kompresji, ale zmniejszony zakres dynamiczny.
  * **JPG (8-bit)**: Najmniejsze rozmiary plików, najlepszy tylko do podglądu i wyświetlania w Internecie. Wykorzystuje kompresję stratną, która nie nadaje się do analiz naukowych.***

## Zapisz szablon projektu

Ta funkcja pozwala zapisać bieżące ustawienia projektu jako szablon wielokrotnego użytku.

* **Typ**: Wprowadzanie tekstu + przycisk Zapisz
* **Opis**: Wprowadź opisową nazwę szablonu ustawień i kliknij ikonę zapisu. Szablon zapisze wszystkie bieżące ustawienia projektu (wykrywanie celu, opcje przetwarzania, indeksy i format eksportu), aby można było je łatwo wykorzystać w przyszłych projektach.
* **Przykłady zastosowania**:
  * Tworzenie szablonów dla różnych systemów kamer (RGB, wielospektralne, NIR)
  * Zapisywanie standardowych konfiguracji dla określonych rodzajów upraw lub procesów analitycznych
  * Udostępnianie spójnych ustawień wszystkim członkom zespołu
* **Sposób użycia**:
  1. Skonfiguruj wszystkie żądane ustawienia projektu
  2. Wprowadź nazwę szablonu (np. „RedEdge Survey3 NDVI Standard”).
  3. Kliknij ikonę zapisu.
  4. Szablon można teraz załadować podczas tworzenia nowych projektów.

***

## Zapisz folder projektu

To ustawienie określa, gdzie domyślnie zapisywane są nowe projekty.

* **Typ**: Wyświetlanie ścieżki katalogu + przycisk Edytuj
* **Domyślnie**: `C:\Users\[Username]\Chloros Projects`
* **Opis**: Pokazuje bieżący domyślny katalog, w którym tworzone są nowe projekty Chloros. Kliknij ikonę edycji, aby wybrać inny katalog.
* **Kiedy zmienić**:
  * Ustaw dysk sieciowy w celu współpracy zespołowej.
  * Zmień na dysk z większą przestrzenią dyskową dla dużych zbiorów danych.
  * Organizuj projekty według roku, klienta lub typu projektu w różnych folderach.
* **Uwaga**: Zmiana tego ustawienia ma wpływ tylko na NOWE projekty. Istniejące projekty pozostają w swoich pierwotnych lokalizacjach.***

## Trwałość ustawień

Wszystkie ustawienia projektu są automatycznie zapisywane wraz z plikiem projektu (format projektu `.mapir`). Po ponownym otwarciu projektu wszystkie ustawienia są przywracane dokładnie tak, jak zostały pozostawione.

### Hierarchia ustawień

Ustawienia są stosowane w następującej kolejności:

1. **Ustawienia domyślne systemu** — wbudowane ustawienia domyślne zdefiniowane przez Chloros
2. **Ustawienia szablonu** — jeśli podczas tworzenia projektu załadowano szablon
3. **Zapisane ustawienia projektu** — ustawienia zapisane wraz z plikiem projektu
4. **Ręczne dostosowania** — wszelkie zmiany wprowadzone podczas bieżącej sesji

### Ustawienia i przetwarzanie obrazów

Większość zmian ustawień (szczególnie w kategoriach Przetwarzanie i Eksport) spowoduje ponowne przetworzenie obrazów w celu odzwierciedlenia nowych ustawień. Niektóre ustawienia są jednak „tylko do eksportu” i nie wymagają natychmiastowego ponownego przetworzenia:

* Zapisz szablon projektu
* Katalog roboczy
* Skalibrowany format obrazu (ma zastosowanie podczas eksportowania)

***

## Najlepsze praktyki

1. **Zacznij od ustawień domyślnych**: Ustawienia domyślne sprawdzają się w przypadku większości systemów kamer MAPIR i typowych procesów roboczych.
2. **Twórz szablony**: Po zoptymalizowaniu ustawień dla konkretnego procesu roboczego lub kamery zapisz je jako szablon, aby zapewnić spójność między projektami.
3. **Przetestuj przed pełnym przetwarzaniem**: Podczas eksperymentowania z nowymi ustawieniami przetestuj je na niewielkiej podgrupie obrazów przed przetworzeniem całego zestawu danych.
4. **Dokumentuj swoje ustawienia**: Używaj opisowych nazw szablonów, które wskazują system kamer, typ przetwarzania i przeznaczenie (np. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Wybór formatu eksportu**: Wybierz format eksportu w oparciu o docelowe zastosowanie:
   * Analiza naukowa → TIFF (16-bitowy lub 32-bitowy)
   * Przetwarzanie GIS → TIFF (16-bitowy)
   * Szybka wizualizacja → PNG (8-bitowy)
   * Udostępnianie w Internecie → JPG (8-bitowy)

***

Więcej informacji na temat wskaźników wielospektralnych w Chloros można znaleźć na stronie [Wzory wskaźników wielospektralnych](multispectral-index-formulas.md).
