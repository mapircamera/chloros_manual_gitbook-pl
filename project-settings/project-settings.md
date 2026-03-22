# Ustawienia projektu

Panel <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> w Chloros umożliwia konfigurację wszystkich aspektów przetwarzania obrazów, wykrywania celów kalibracyjnych, obliczeń indeksów wielospektralnych oraz opcji eksportu dla danego projektu. Ustawienia te są zapisywane wraz z projektem i mogą być zapisane jako szablony do ponownego wykorzystania w wielu projektach.

## Dostęp do ustawień projektu

Aby uzyskać dostęp do ustawień projektu:

1. Otwórz projekt w Chloros
2. Kliknij zakładkę **Ustawienia projektu**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na pasku bocznym po lewej stronie
3. Panel ustawień wyświetli wszystkie dostępne opcje konfiguracyjne uporządkowane według kategorii

***

## Wykrywanie celów

Te ustawienia kontrolują sposób, w jaki Chloros wykrywa i przetwarza cele kalibracyjne na obrazach.

### Minimalny obszar próbki kalibracyjnej (px)

* **Typ**: Liczba
* **Zakres**: od 0 do 10 000 pikseli
* **Domyślnie**: 25 pikseli
* **Opis**: Ustawia minimalny obszar (w pikselach) wymagany, aby wykryty region został uznany za prawidłową próbkę celu kalibracji. Mniejsze wartości pozwolą wykrywać mniejsze cele, ale mogą zwiększyć liczbę fałszywych alarmów. Większe wartości wymagają większych, wyraźniejszych regionów docelowych do wykrycia.
* **Kiedy dostosować**:
  * Zwiększ, jeśli otrzymujesz fałszywe wykrycia na małych artefaktach obrazu
  * Zmniejsz, jeśli cele kalibracji wydają się małe na obrazach i nie są wykrywane

### Minimalne klastrowanie celów (0–100)

* **Typ**: Liczba
* **Zakres**: od 0 do 100
* **Domyślnie**: 60
* **Opis**: Kontroluje próg grupowania podobnych obszarów kolorystycznych podczas wykrywania celów kalibracyjnych. Wyższe wartości wymagają zgrupowania większej liczby podobnych kolorów, co skutkuje bardziej konserwatywnym wykrywaniem celów. Niższe wartości pozwalają na większą różnorodność kolorów w obrębie grupy celów.
* **Kiedy dostosować**:
  * Zwiększ, jeśli cele kalibracyjne są rozdzielane na wiele wykryć
  * Zmniejsz, jeśli cele kalibracyjne z różnicami kolorystycznymi nie są w pełni wykrywane

***

## Przetwarzanie

Te ustawienia kontrolują sposób, w jaki Chloros przetwarza i kalibruje Twoje obrazy.

### Korekcja winietowania

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Stosuje korekcję winietowania w celu skompensowania przyciemnienia obiektywu na krawędziach obrazów. Winietowanie to powszechne zjawisko optyczne, w którym rogi i krawędzie obrazu wydają się ciemniejsze niż środek z powodu charakterystyki obiektywu.
* **Kiedy wyłączyć**: Wyłącz tylko wtedy, gdy kombinacja aparatu i obiektywu zastosowała już korekcję winietowania lub jeśli chcesz ręcznie skorygować winietowanie podczas obróbki końcowej.

### Kalibracja odbicia / balans bieli

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Włącza automatyczną kalibrację odbicia przy użyciu wykrytych celów kalibracyjnych na zdjęciach. Normalizuje to wartości odbicia w całym zbiorze danych i zapewnia spójne pomiary niezależnie od warunków oświetleniowych.
* **Kiedy wyłączyć**: Wyłącz tylko wtedy, gdy chcesz przetwarzać surowe, nieskalibrowane obrazy lub jeśli korzystasz z innego procesu kalibracji.

### Metoda debayera

* **Typ**: Lista rozwijana
* **Opcje**:
  * Standardowa (szybka, średnia jakość)
  * Z uwzględnieniem tekstury (wolna, najwyższa jakość) \[Chloros+]
* **Domyślnie**: Standardowa (szybka, średnia jakość)
* **Opis**: Wybiera algorytm demosaicingu używany do konwersji surowych danych z czujnika o wzorze Bayera na pełnokolorowe obrazy. Metoda „Standardowa (szybka, średnia jakość)” zapewnia optymalną równowagę między szybkością przetwarzania a jakością obrazu. Metoda „Z uwzględnieniem tekstury (wolna, najwyższa jakość)” \[Chloros+] wykorzystuje wysokiej jakości algorytm demosaicingu z uwzględnieniem krawędzi w połączeniu z modelem odszumiania opartym na sztucznej inteligencji (AI/ML), który usuwa prawie cały szum powstały podczas demosaicingu. Model z uwzględnieniem tekstury wymaga pamięci GPU (VRAM) do działania. Zalecamy korzystanie z niego, gdy masz do dyspozycji &gt;4 GB pamięci VRAM, aby przyspieszyć przetwarzanie.
* **Uwaga**: W przyszłych wersjach Chloros mogą zostać dodane dodatkowe metody debayeringu.

### Minimalny interwał ponownej kalibracji

* **Typ**: Liczba
* **Zakres**: od 0 do 3600 sekund
* **Domyślnie**: 0 sekund
* **Opis**: Ustawia minimalny interwał czasowy (w sekundach) między użyciem celów kalibracyjnych. Gdy ustawiono wartość 0, Chloros wykorzysta każdy wykryty cel kalibracyjny. Gdy ustawiono wyższą wartość, Chloros wykorzysta tylko cele kalibracyjne, które są oddzielone co najmniej tą liczbą sekund, skracając czas przetwarzania dla zbiorów danych z częstym przechwytywaniem celów kalibracyjnych.
* **Kiedy dostosować**:
  * Ustaw na 0, aby uzyskać maksymalną dokładność kalibracji przy zmiennych warunkach oświetleniowych
  * Zwiększ (np. do 60–300 sekund), aby przyspieszyć przetwarzanie, gdy oświetlenie jest stałe i występują częste obrazy celów kalibracyjnych

### Przesunięcie strefy czasowej czujnika światła

* **Typ**: Liczba
* **Zakres**: od -12 do +12 godzin
* **Domyślnie**: 0 godzin
* **Opis**: Określa przesunięcie strefy czasowej (w godzinach od czasu UTC) dla znaczników czasu danych czujnika światła. Jest to używane podczas przetwarzania plików danych PPK (Post-Processed Kinematic) w celu zapewnienia prawidłowej synchronizacji czasu między przechwyceniem obrazu a danymi GPS.
* **Kiedy dostosować**: Ustaw tę wartość na przesunięcie lokalnej strefy czasowej, jeśli dane PPK używają czasu lokalnego zamiast czasu UTC. Na przykład:
  * Czas pacyficzny: -8 lub -7 (w zależności od czasu letniego)
  * Czas wschodni: -5 lub -4 (w zależności od czasu letniego)
  * Czas środkowoeuropejski: +1 lub +2 (w zależności od czasu letniego)

### Zastosuj korekty PPK

* **Typ**: Pole wyboru
* **Domyślnie**: Wyłączone (niezaznaczone)
* **Opis**: Włącza stosowanie korekt kinematycznych przetwarzanych po zebraniu danych (PPK) z rejestratorów MAPIR DAQ wyposażonych w GPS (GNSS). Po włączeniu Chloros wykorzysta wszelkie pliki dziennika .daq zawierające dane pinów ekspozycji w katalogu projektu i zastosuje precyzyjne korekty geolokalizacyjne do obrazów.
* **Wymagania**: W katalogu projektu musi znajdować się plik dziennika .daq zawierający wpisy dotyczące pinów ekspozycji
* **Kiedy włączyć**: Zaleca się zawsze włączać korekcję PPK, jeśli w pliku dziennika .daq znajdują się wpisy dotyczące sprzężenia zwrotnego ekspozycji.

### Pin ekspozycji 1

* **Typ**: Wybór z listy rozwijanej
* **Widoczność**: Widoczne tylko wtedy, gdy włączona jest opcja „Zastosuj korekty PPK” ORAZ dostępne są dane ekspozycji dla pinu 1
* **Opcje**:
  * Nazwy modeli kamer wykrytych w projekcie
  * „Nie używaj” – Zignoruj ten pin ekspozycji
* **Domyślnie**: Wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: Przypisuje konkretną kamerę do pinu ekspozycji 1 w celu synchronizacji czasowej PPK. Pin ekspozycji rejestruje dokładny moment wyzwolenia migawki kamery, co ma kluczowe znaczenie dla dokładnej geolokalizacji PPK.
* **Zachowanie przy automatycznym wyborze**:
  * Pojedyncza kamera + pojedynczy pin: Automatycznie wybiera kamerę
  * Pojedyncza kamera + dwa piny: Pin 1 automatycznie przypisany do kamery
  * Wiele kamer: Wymagany wybór ręczny

### Pin ekspozycji 2

* **Typ**: Wybór z listy rozwijanej
* **Widoczność**: Widoczne tylko wtedy, gdy włączona jest opcja „Zastosuj korekty PPK” ORAZ dostępne są dane ekspozycji dla pinu 2
* **Opcje**:
  * Nazwy modeli kamer wykrytych w projekcie
  * „Nie używaj” – zignoruj ten pin ekspozycji
* **Domyślnie**: Wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: Przypisuje konkretną kamerę do pinu ekspozycji 2 w celu synchronizacji czasu PPK podczas korzystania z konfiguracji z dwiema kamerami.
* **Zachowanie przy automatycznym wyborze**:
  * Pojedyncza kamera + pojedynczy pin: Pin 2 jest automatycznie ustawiany na „Nie używaj”
  * Pojedyncza kamera + dwa piny: Pin 2 jest automatycznie ustawiany na „Nie używaj”
  * Wiele kamer: Wymagany jest wybór ręczny
* **Uwaga**: Ta sama kamera nie może być przypisana jednocześnie do pinu 1 i pinu 2.***

## Indeks

Te ustawienia pozwalają skonfigurować indeksy wielospektralne do analizy i wizualizacji.

### Dodaj indeks

* **Typ**: Specjalny panel konfiguracji indeksów
* **Opis**: Otwiera interaktywny panel, w którym można wybrać i skonfigurować wielospektralne indeksy roślinności (NDVI, NDRE, EVI itp.) do obliczenia podczas przetwarzania obrazu. Można dodać wiele wskaźników, z których każdy ma własne ustawienia wizualizacji.
* **Dostępne wskaźniki**: System zawiera ponad 30 predefiniowanych wskaźników wielospektralnych, w tym:
  * NDVI (znormalizowany wskaźnik różnicy wegetacji)
  * NDRE (znormalizowana różnica RedEdge)
  * EVI (ulepszony wskaźnik wegetacji)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * I wiele innych (pełna lista znajduje się w sekcji [Wzory indeksów wielospektralnych](multispectral-index-formulas.md))
* **Funkcje**:
  * Wybór spośród predefiniowanych wzorów indeksów
  * Konfiguracja gradientów kolorów wizualizacji (LUT – tabele odnośników)
  * Ustawianie wartości progowych do analizy
  * Tworzenie niestandardowych wzorów indeksów

### Wzory niestandardowe (Funkcja Chloros+)

* **Typ**: Tablica definicji formuł niestandardowych
* **Opis**: Umożliwia tworzenie i zapisywanie niestandardowych formuł indeksów wielospektralnych przy użyciu operacji matematycznych na pasmach. Formuły niestandardowe są zapisywane wraz z ustawieniami projektu i można z nich korzystać tak samo jak z wbudowanych indeksów.
* **Jak utworzyć**:
  1. W panelu konfiguracji indeksów znajdź opcję formuł niestandardowych
  2. Zdefiniuj formułę, używając identyfikatorów pasm (np. NIR, Red, Green, Blue)
  3. Zapisz formułę pod opisową nazwą
* **Składnia formuły**: Obsługiwane są standardowe operacje matematyczne, w tym:
  * Arytmetyka: `+`, `-`, `*`, `/`
  * Nawiasy określające kolejność operacji
  * Odwołania do pasm: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Te ustawienia kontrolują format i jakość eksportowanych przetworzonych obrazów.

### Skalibrowany format obrazu

* **Typ**: Lista rozwijana
* **Opcje**:
  * **TIFF (16-bitowy)** - Nieskompresowany 16-bitowy format TIFF
  * **TIFF (32-bitowy, procentowy)** — 32-bitowy format zmiennoprzecinkowy TIFF z wartościami odbicia wyrażonymi w procentach
  * **PNG (8-bitowy)** – skompresowany 8-bitowy format PNG
  * **JPG (8-bitowy)** – skompresowany 8-bitowy format JPEG
* **Domyślnie**: TIFF (16-bitowy)
* **Opis**: Wybiera format pliku do zapisywania przetworzonych i skalibrowanych obrazów.
* **Zalecenia dotyczące formatu**:
  * **TIFF (16-bitowy)**: Zalecany do analiz naukowych i profesjonalnych procesów roboczych. Zachowuje maksymalną jakość danych bez artefaktów kompresji. Najlepszy do analizy wielospektralnej i dalszego przetwarzania w oprogramowaniu GIS.
  * **TIFF (32-bitowy, procent)**: Najlepszy do procesów wymagających wartości odbicia jako procentów (0–100%). Zapewnia maksymalną precyzję pomiarów radiometrycznych.
  * **PNG (8-bit)**: Dobry do przeglądania w sieci i ogólnej wizualizacji. Mniejsze rozmiary plików dzięki bezstratnej kompresji, ale zredukowany zakres dynamiczny.
  * **JPG (8-bit)**: Najmniejsze rozmiary plików, najlepszy wyłącznie do podglądów i wyświetlania w sieci. Wykorzystuje kompresję stratną, która nie nadaje się do analiz naukowych.***

## Zapisz szablon projektu

Ta funkcja pozwala zapisać bieżące ustawienia projektu jako szablon, który można ponownie wykorzystać.

* **Typ**: Pole tekstowe + przycisk Zapisz
* **Opis**: Wpisz opisową nazwę szablonu ustawień i kliknij ikonę zapisu. Szablon zapisze wszystkie bieżące ustawienia projektu (wykrywanie celów, opcje przetwarzania, wskaźniki i format eksportu) w celu łatwego ponownego wykorzystania w przyszłych projektach.
* **Przykłady zastosowań**:
  * Tworzenie szablonów dla różnych systemów kamer (RGB, wielospektralny, NIR)
  * Zapisywanie standardowych konfiguracji dla określonych rodzajów upraw lub procesów analitycznych
  * Udostępnianie spójnych ustawień w zespole
* **Jak korzystać**:
  1. Skonfiguruj wszystkie żądane ustawienia projektu
  2. Wprowadź nazwę szablonu (np. „RedEdge Survey3 NDVI Standard”)
  3. Kliknij ikonę zapisu
  4. Szablon można teraz załadować podczas tworzenia nowych projektów

***

## Zapisz folder projektu

To ustawienie określa, gdzie domyślnie zapisywane są nowe projekty.

* **Typ**: Wyświetlanie ścieżki katalogu + przycisk edycji
* **Domyślnie (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Domyślnie (Linux)**: `~/.local/share/chloros/projects`
* **Opis**: Pokazuje aktualny katalog domyślny, w którym tworzone są nowe projekty Chloros. Kliknij ikonę edycji, aby wybrać inny katalog.
* **Kiedy zmienić**:
  * Ustaw na dysk sieciowy w celu współpracy zespołowej
  * Zmień na dysk z większą przestrzenią dyskową dla dużych zbiorów danych
  * Organizuj projekty według roku, klienta lub typu projektu w różnych folderach
* **Uwaga**: Zmiana tego ustawienia ma wpływ tylko na NOWE projekty. Istniejące projekty pozostają w swoich pierwotnych lokalizacjach.***

## Trwałość ustawień

Wszystkie ustawienia projektu są automatycznie zapisywane wraz z plikiem projektu (format projektu `.mapir`). Po ponownym otwarciu projektu wszystkie ustawienia są przywracane dokładnie tak, jak zostały zapisane.

### Hierarchia ustawień

Ustawienia są stosowane w następującej kolejności:

1. **Ustawienia domyślne systemu** — wbudowane ustawienia domyślne zdefiniowane przez Chloros
2. **Ustawienia szablonu** — jeśli podczas tworzenia projektu wczytasz szablon
3. **Zapisane ustawienia projektu** — ustawienia zapisane wraz z plikiem projektu
4. **Ręczne dostosowania** — wszelkie zmiany wprowadzone podczas bieżącej sesji

### Ustawienia i przetwarzanie obrazów

Większość zmian ustawień (zwłaszcza w kategoriach Przetwarzanie i Eksport) spowoduje ponowne przetworzenie obrazów w celu uwzględnienia nowych ustawień. Niektóre ustawienia mają jednak charakter „tylko do eksportu” i nie wymagają natychmiastowego ponownego przetworzenia:

* Zapisz szablon projektu
* Katalog roboczy
* Skalibrowany format obrazu (ma zastosowanie podczas eksportu)

***

## Najlepsze praktyki

1. **Zacznij od ustawień domyślnych**: Ustawienia domyślne sprawdzają się dobrze w przypadku większości systemów kamer MAPIR i typowych procesów pracy.
2. **Twórz szablony**: Po zoptymalizowaniu ustawień dla konkretnego procesu pracy lub kamery zapisz je jako szablon, aby zapewnić spójność między projektami.
3. **Przetestuj przed pełnym przetwarzaniem**: Podczas eksperymentowania z nowymi ustawieniami przetestuj je na niewielkim podzbiorze obrazów przed przetworzeniem całego zestawu danych.
4. **Dokumentuj swoje ustawienia**: Używaj opisowych nazw szablonów, które wskazują system kamer, typ przetwarzania i zamierzone zastosowanie (np. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Wybór formatu eksportu**: Wybierz format eksportu w zależności od przeznaczenia:
   * Analiza naukowa → TIFF (16-bitowy lub 32-bitowy)
   * Przetwarzanie GIS → TIFF (16-bitowy)
   * Szybka wizualizacja → PNG (8-bitowy)
   * Udostępnianie w sieci → JPG (8-bitowy)

***

Więcej informacji na temat wskaźników wielospektralnych w Chloros można znaleźć na stronie [Wzory wskaźników wielospektralnych](multispectral-index-formulas.md).
