# Ustawienia projektu

Pasek boczny „Ustawienia projektu” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) na stronieChloros

umożliwia skonfigurowanie wszystkich aspektów przetwarzania obrazów, wykrywania celów kalibracyjnych, obliczania wskaźników wielospektralnych oraz opcji eksportu dla danego projektu. Ustawienia te są zapisywane wraz z projektem i można je zapisać jako szablony do ponownego wykorzystania w wielu projektach.

## Dostęp do ustawień projektu

Aby uzyskać dostęp do ustawień projektu:

1. Otwórz projekt na stronieChloros


2. Kliknij zakładkę **Ustawienia projektu**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

na lewym pasku bocznym
3. W panelu ustawień wyświetlą się wszystkie dostępne opcje konfiguracyjne uporządkowane według kategorii



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Ustawienia zależne od innych ustawień są wyszarzone.** Gdy przełącznik nadrzędny uniemożliwia zastosowanie danego ustawienia (na przykład odznaczenie opcji *Kalibracja odbicia / balans bieli* uniemożliwia *Eksport odbicia*), zależny element sterujący jest wyłączony, a jego podpowiedź wskazuje przełącznik, który należy zmienić.
{% endhint %}

***

## Wyświetlanie

### Rozdzielczość miniatur obrazów

* **Typ**: Lista rozwijana
* **Opcje**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Domyślnie**: Domyślnie (512 px)
* **Opis**: Rozdzielczość (najdłuższy bok, w pikselach), w której renderowane są miniatury w siatce obrazów. Wyższe wartości zapewniają ostrzejszy obraz po powiększeniu, ale powodują wolniejsze ładowanie i zużywają więcej pamięci. Pełna rozdzielczość odpowiada oryginalnemu rozmiarowi obrazu.
* **Uwaga**: Tylko do wyświetlania — nie ma to żadnego wpływu na przetwarzanie ani na eksportowane pliki.***

## Wykrywanie celów

Te ustawienia kontrolują sposób, w jaki program „Chloros

” wykrywa i przetwarza cele kalibracyjne na obrazach. Oba są aktywne tylko wtedy, gdy włączona jest opcja **Kalibracja odbicia / balans bieli** (w przeciwnym razie są wyszarzone, ponieważ wykrywanie celów jest całkowicie pomijane).

### Minimalny obszar próbki kalibracyjnej (px)

* **Typ**: Liczba
* **Zakres**: od 0 do 10 000 pikseli
* **Wartość domyślna**: 25 pikseli
* **Opis**: Określa minimalny obszar (w pikselach) wymagany, aby wykryty obszar został uznany za prawidłową próbkę celu kalibracji. Mniejsze wartości pozwolą wykrywać mniejsze cele, ale mogą zwiększyć liczbę fałszywych alarmów. Większe wartości wymagają większych i wyraźniejszych obszarów docelowych do wykrycia.
* **Kiedy należy dostosować**:
  * Zwiększ wartość, jeśli występują fałszywe wykrycia na małych artefaktach obrazu
  * Zmniejsz wartość, jeśli cele kalibracyjne wydają się małe na obrazach i nie są wykrywane

### Minimalne grupowanie celów (0–100)

* **Typ**: Liczba
* **Zakres**: od 0 do 100
* **Wartość domyślna**: 60
* **Opis**: Kontroluje próg grupowania podobnych obszarów kolorystycznych podczas wykrywania celów kalibracyjnych. Wyższe wartości wymagają zgrupowania większej liczby podobnych kolorów, co skutkuje bardziej konserwatywnym wykrywaniem celów. Niższe wartości pozwalają na większą zmienność kolorów w obrębie grupy celów.
* **Kiedy należy dostosować**:
  * Zwiększ, jeśli cele kalibracyjne są rozdzielane na wiele wykrytych obiektów
  * Zmniejsz wartość, jeśli cele kalibracyjne o zróżnicowanej kolorystyce nie są w pełni wykrywane

***

## Przetwarzanie

Te ustawienia określają sposób, w jaki program „Chloros

” przetwarza i kalibruje obrazy.

### Korekcja winietowania

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Stosuje korekcję winietowania w celu skompensowania przyciemnienia na krawędziach zdjęć spowodowanego charakterystyką obiektywu. Winietowanie to powszechne zjawisko optyczne, w którym rogi i krawędzie zdjęcia wydają się ciemniejsze niż jego środek z powodu właściwości obiektywu.
* **Efekt uboczny**: To przełącznik określa również, który *niekalibrowany produkt rezerwowy* zostanie zapisany po zakończeniu serii pomiarów (patrz poniżej).

### Kalibracja odbicia / balans bieli

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Włącza kalibrację odbicia — na podstawie wykrytych w kadrze celów kalibracyjnych i/lub danych o natężeniu światła z czujnika DAQ, w zależności od kamery i dostępnych danych. Normalizuje to wartości odbicia w całym zbiorze danych i zapewnia spójność pomiarów niezależnie od warunków oświetleniowych.
* **Gdy opcja jest wyłączona**: Wykrywanie celów jest całkowicie pomijane i**żadna kamera nie może wygenerować produktu odbicia** — dotyczy to zarówno systemu opartego na celach „Survey3

”, jak i systemu opartego na DAQ „LATTICE”. Powiązane ustawienia (*Eksport odbicia*, *Minimalny interwał ponownej kalibracji* oraz progi wykrywania celów) są wyszarzone.

### Nieskalibrowane produkty awaryjne: Eksport odpowiedzi czujnika / Eksport skorygowany o winietę

* **Typ**: Dwa pola wyboru
* **Ustawienia domyślne**: Oba włączone (zaznaczone)
* **Opis**: Gdy nie można przeprowadzić kalibracji odbicia dla danej klatki (nie znaleziono celu kalibracyjnego lub kalibracja odbicia jest wyłączona), jest ona zapisywana jako *niekalibrowany produkt rezerwowy*. **Dla każdego modelu kamery istnieje dokładnie jeden z dwóch produktów rezerwowych na każdy przebieg**, wybierany za pomocą przełącznika *Korekcja winietowania*:
  * Korekcja winietowania **włączona**→ `Vignette_Corrected_Images/` (regulowana przez opcję**Eksport z korekcją winietowania**)
  * Korekcja winietowania **wyłączona**→ `Sensor_Response_Images/` (regulowana przez opcję**Eksportuj charakterystykę czujnika**)
* Produkt rezerwowy, który nie jest aktywny, jest wyszarzony. Odznaczenie aktywnego produktu powoduje całkowite wstrzymanie zapisywania tego pliku.

### Produkty eksportowe LATTICE

W przypadku projektów zawierających ujęcia LATTICE każda zaimportowana klatka LATTICE jest rozdzielana na wszystkie włączone **i odpowiednie**produkty w ramach jednego cyklu przetwarzania. Rozdzielanie kontrolują cztery pola wyboru (wszystkie domyślnie**włączone**):

| Ustawienie | Folder wyjściowy | Co eksportuje |
| --- | --- | --- |
| **Eksport z usunięciem efektu bayera** | `Debayered_Images/` | Obraz z liniowym usunięciem efektu bayera. Dotyczy kamer typu „RGB

” oraz kamer wielospektralnych. |
| **Eksport podglądu** | `Preview_Images/` | Podgląd wyświetlany na ekranie.RGB

= balans bieli (źródło światła DAQ, jeśli dostępne, w przeciwnym razie „gray-world”) + gamma; wielospektralne = rozciągnięcie w fałszywych kolorach. |
| **Eksport promieniowania** | `Radiance_Images/` | Promieniowanie spektralne typu Float32 w W/m²/sr/nm. Tylko w trybie wielospektralnym (M3C/M3M) — nie dotyczy plików źródłowych typu „RGB

”. Zawsze zapisywane jako 32-bitoweTIFF

niezależnie od ustawienia *Kalibrowany format obrazu*. |
| **Współczynnik odbicia eksportu**| `Reflectance_Calibrated_Images/` | Współczynnik odbicia typu Uint16, skalowany tak, że**32768 = współczynnik odbicia 1,0** (oznaczony w pliku XMP jako `Chloros:PixelScale`). Tylko w trybie wielospektralnym, zapisywane, gdy pasujący rekord `.daq` z promieniowaniem opadającym (lub cel w kadrze, który przeszedł kontrolę jakości) pokrywa kadr. |

* Kamery główne typu „RGB

” emitują obraz po debayeringu + podgląd; wartości promieniowania/odbicia są dla nich pomijane, ponieważ nie mają zastosowania.
* Głębokość bitowa obrazu po debayeringu/podglądu zależy od ustawienia *Format obrazu skalibrowanego*; promieniowanie ma zawsze typ float32.
* Te cztery przełączniki nie mają wpływu na przetwarzanie typu „Survey3

”.

Te same cztery przełączniki istnieją w trybie bezinterfejsu jako `chloros-cli process --debayered / --preview / --radiance / --reflectance` oraz jako odpowiadające im parametry w pliku „SDK

”. Zastąpiły one starą flagę `--radiometric-output`, która już nie istnieje.

{% hint style="warning" %}
**Wyłączenie wszystkich odpowiednich produktów powoduje niepowodzenie przebiegu.** Od wersji 1.2.0 przebieg przetwarzania, który zażądał produktów, ale nie zapisał żadnego produktu obrazowego, zgłasza błąd, a plik „CLI

” kończy się wynikiem niezerowym, zamiast zgłaszać ciche zakończenie sukcesem. W logu podana jest nazwa produktu, którego nie udało się zapisać, oraz przyczyna. Przebieg celowo ograniczony wyłącznie do metadanych (bez żadnych żądań) nadal kończy się sukcesem.
{% endhint %}

### Źródło odbicia (ustawienie projektu, konfigurowane poprzezCLI

/SDK

)

Projekt przechowuje również informację o tym, jakiego **odniesienia odbicia** używa produkt odbicia LATTICE. W panelu ustawień nie ma dedykowanego elementu sterującego; wartość ta jest przechowywana w konfiguracji projektu jako `Processing → "Target reflectance source"` i ustalana za pomocą `chloros-cli process --reflectance-source {auto,target,daq}` lub parametru `reflectance_source` w pliku „SDK

”:

* **`auto`** (domyślnie): cel kalibracyjny w ramce, który przeszedł kontrolę jakości (QA), staje się bezwzględnym punktem odniesienia; w przypadku braku celu lub niepowodzenia kontroli jakości stosowana jest wartość podziału promieniowania padającego z systemu DAQ (ρ = πL/E).
* **`target`**: ściśle oparte na celach odbicie — brak substytucji przez DAQ.
* **`daq`**: odbicie oparte na danych z DAQ; cele w kadrze nie są wykorzystywane jako odniesienie.

Zapisana wartość jest dopasowywana bez uwzględniania wielkości liter, a kilka wariantów pisowni jest akceptowanych jako aliasy: `target`, `target_image`, `empirical` i `empirical_line` oznaczają **cel**; `daq`, `dls`, `light_sensor` oraz `sensor` oznaczają**daq**. Wszelkie inne wartości — w tym brak klucza — są rozpoznawane jako**auto**.**Zmierzone** są wyszukiwane na podstawie numeru seryjnego/kodu QR urządzenia docelowego, np. `<serial>.csv`, w trzech miejscach: w katalogu podanym przez `--target-reflectance-dir` (zapisanym jako `Processing → "Target reflectance dir"`), własnym folderze projektu `target_reflectance/` oraz ścieżce określonej w zmiennej środowiskowej `CHLOROS_TARGET_REFLECTANCE_DIR`. Jeśli dla danej jednostki nie istnieje skan pomiarowy, zamiast tego wykorzystywana jest nominalna opublikowana krzywa dla danego modelu docelowego.

### Metoda demosaicingu

* **Typ**: Lista rozwijana
* **Opcje**:
  * Standardowa (szybka, średnia jakość)
  * Z uwzględnieniem tekstury (powolna, najwyższa jakość) \[Chloros

+]
* **Domyślnie**: Standardowa (szybka, średnia jakość)
* **Opis**: Wybiera algorytm demosaicingu używany do konwersji surowych danych z czujnika o układzie Bayera na pełnokolorowe obrazy. Metoda „Standardowa (szybka, średnia jakość)” zapewnia optymalną równowagę między szybkością przetwarzania a jakością obrazu. Metoda „Z uwzględnieniem tekstury (powolna, najwyższa jakość)” \[Chloros

+] wykorzystuje wysokiej jakości algorytm demosaicingu uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na sztucznej inteligencji i uczeniu maszynowym, który eliminuje niemal cały szum powstały podczas demosaicingu. Model „Texture Aware” wymaga pamięci GPU (VRAM). Zalecamy korzystanie z niego, gdy masz do dyspozycji &gt;4 GB VRAM, co zapewni szybsze przetwarzanie.
* **Gdy wiersz jest menu rozwijanym**: menu rozwijane z dwiema opcjami pojawia się tylko wtedy, gdy spełnione są**obie** warunki — użytkownik jest zalogowany z kwalifikującą się subskrypcjąChloros

+ **oraz** projekt nie zawiera żadnych ujęć LATTICE. W przeciwnym razie wiersz wyświetla się jako zwykły tekst o treści `Standard (Fast, Medium Quality)` bez możliwości wyboru.
* **Uwaga dotycząca LATTICE**: Nie ma modelu „Texture Aware” wytrenowanego na danych LATTICE, a potok przetwarzania wymusza standardowe demosaikowanie klatek LATTICE niezależnie od zapisanej wartości. Jeśli dodasz folder LATTICE do projektu, w którym opcja „Texture Aware” była już zaznaczona, program „Chloros

” przywróci ustawienie do wartości „Standard”, zamiast pozostawić nieaktualną wartość w polu `project.json`.

### Minimalny interwał ponownej kalibracji

* **Typ**: Liczba
* **Zakres**: od 0 do 3 600 sekund
* **Wartość domyślna**: 0 sekund
* **Opis**: Ustawia minimalny odstęp czasowy (w sekundach) między wykorzystaniem celów kalibracyjnych. Gdy wartość wynosi 0, program „Chloros

” będzie wykorzystywał każdy wykryty cel kalibracyjny. Gdy ustawiona zostanie wyższa wartość, program „Chloros

” będzie wykorzystywał tylko cele kalibracyjne, między którymi upłynęło co najmniej tyle sekund, co skraca czas przetwarzania zestawów danych zawierających częste ujęcia celów kalibracyjnych.
* **Kiedy należy dostosować**:
  * Ustaw wartość 0, aby uzyskać maksymalną dokładność kalibracji w zmiennych warunkach oświetleniowych
  * Zwiększ wartość (np. do 60–300 sekund), aby przyspieszyć przetwarzanie, gdy oświetlenie jest stałe i występują częste obrazy celów kalibracyjnych

### Przesunięcie strefy czasowej czujnika światła

* **Typ**: Liczba
* **Zakres**: od -12 do +12 godzin
* **Wartość domyślna**: 0 godzin
* **Opis**: Określa przesunięcie strefy czasowej (w godzinach względem czasu UTC) dla sygnatur czasowych danych czujnika światła, wykorzystywane podczas dopasowywania logów czujnika światła do czasów wykonania zdjęć. Nowsze zapisy `.daq` zawierają własne informacje o strefie czasowej, więc opcja ta jest potrzebna głównie w przypadku starszych dzienników zarejestrowanych w czasie lokalnym.

### Zastosuj korekty PPK

* **Typ**: Pole wyboru
* **Domyślnie**: Wyłączone (niezaznaczone)
* **Opis**: Włącza stosowanie korekcji kinematycznych po przetworzeniu (PPK) z rejestratorów danychMAPIR

wyposażonych w GPS (GNSS). Po włączeniu tej opcji programChloros

wykorzysta wszystkie pliki dziennika .daq zawierające dane dotyczące czasu naświetlania, znajdujące się w katalogu projektu, i zastosuje precyzyjne korekcje geolokalizacyjne do zdjęć.
* **Wymagania**: W katalogu projektu musi znajdować się plik dziennika .daq zawierający wpisy dotyczące pinu ekspozycji
* **Kiedy włączyć**: Zaleca się zawsze włączać korekcję PPK, jeśli w pliku dziennika .daq znajdują się wpisy dotyczące ekspozycji.

### Pin ekspozycji 1

* **Typ**: Lista rozwijana
* **Widoczność**: Widoczne tylko wtedy, gdy opcja „Zastosuj korekcje PPK” jest włączona ORAZ dostępne są dane ekspozycji dla pinu 1
* **Opcje**:
  * Nazwy modeli aparatów wykrytych w projekcie
  * „Nie używaj” – zignoruj ten pin ekspozycji
* **Ustawienie domyślne**: Wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: Przypisuje konkretną kamerę do pinu ekspozycji nr 1 w celu synchronizacji czasowej PPK. Pin ekspozycji rejestruje dokładny moment wyzwolenia migawki kamery, co ma kluczowe znaczenie dla dokładnej geolokalizacji PPK.
* **Sposób automatycznego wyboru**:
  * Jedna kamera + jeden pin: Kamera jest wybierana automatycznie
  * Jedna kamera + dwa piny: Pin 1 jest automatycznie przypisywany do kamery
  * Wiele kamer: Wymagany jest wybór ręczny

### Pin ekspozycji 2

* **Typ**: Wybór z listy rozwijanej
* **Widoczność**: Widoczne tylko wtedy, gdy włączona jest opcja „Zastosuj korekty PPK” ORAZ dostępne są dane ekspozycji dla pinu 2
* **Opcje**:
  * Nazwy modeli kamer wykrytych w projekcie
  * „Nie używaj” — zignoruj ten pin ekspozycji
* **Ustawienie domyślne**: Wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: Przypisuje konkretną kamerę do pinu ekspozycji 2 w celu synchronizacji czasowej PPK w przypadku korzystania z konfiguracji z dwiema kamerami.
* **Sposób automatycznego wyboru**:
  * Jedna kamera + jeden pin: Pin 2 jest automatycznie ustawiany na „Nie używaj”
  * Jedna kamera + dwa piny: Pin 2 jest automatycznie ustawiany na „Nie używaj”
  * Wiele kamer: Wymagany jest wybór ręczny
* **Uwaga**: Ta sama kamera nie może być przypisana jednocześnie do pinu 1 i pinu 2.***

## Czujnik światła DAQ

Ta sekcja pojawia się w ustawieniach projektu i zawiera listę wszystkich plików DAQ dotyczących promieniowania padającego w projekcie — nagrania `.daq` oraz dzienniki pomiarów promieniowania padającego DAQ-M `.csv`. Nagrania wykonane w zakładce Czujniki światła są automatycznie dodawane do otwartego projektu.



<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Każdy wiersz zawiera nazwę pliku, model czujnika oraz korekcję nasadki rozpraszającej faktycznie obowiązującą dla tego pliku. Nad wierszami znajduje się pojedynczy element sterujący obejmujący cały projekt:

### Nadpisanie nasadki (wszystkie pliki)

* **Typ**: lista rozwijana
* **Opcje**: `Auto` oraz profile korekcji nasadki obowiązujące dla typów czujników obecnych w projekcie
* **Domyślnie**: Auto
* **Zapisane jako**: `Processing → "DAQ cap id"` (domyślnie `auto`)
* **Opis**: `Auto` wykorzystuje wartość nasłonecznienia zarejestrowaną w każdym pliku (jeśli nic nie zostało zarejestrowane, przyjmuje się nasadkę typu „Sunshine” — wszystkie urządzenia pomiaroweMAPIR

są dostarczane z korektorem typu „Sunshine”). Wybranie konkretnej nasadki zastępuje ustawienia **wszystkich** plików danych promieniowania padającego w projekcie: surowe zapisy są korygowane przy jej użyciu, a zapisy, które już zawierają nasadkę, są ponownie referencjonowane (zapisana korekcja jest cofana, a zastosowana zostaje wybrana nasadka).
* **Ważne**: Wybrana nasadka musi odpowiadać nasadce, która była fizycznie zamontowana podczas rejestracji. Ani czujnik, ani oprogramowanie nie są w stanie wykryć fizycznej nasadki — niedopasowany identyfikator nasadki powoduje błędną korekcję widm.

Celowo zastosowano **jedno** ustawienie dla całego projektu zamiast list rozwijanych dla poszczególnych plików: ustawienie to obejmuje każde źródło promieniowania padającego w projekcie.***

## Wyrównanie macierzy

Ta sekcja pojawia się **tylko** wtedy, gdy co najmniej jeden obraz w projekcie zawiera transformację wyrównującą między modułami, którą macierze LATTICE nanoszą w momencie przechwytywania (tagi XMP `Chloros:Alignment*`). Pokazuje ona, ile obrazów zawiera tagi wyrównania, która kamera jest referencyjna (oznaczenie `REF`), oraz tabelę liczby obrazów dla poszczególnych kamer.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Zastosuj wyrównanie tablicy

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Zapisywane jako**: `Processing → "Array alignment"`
* **Opis**: Przekształca każdy przetworzony produkt (bez efektu debayeringu / podgląd / promieniowanie / odbicie / indeks) do wspólnej geometrii odniesienia tablicy przy użyciu transformacji zapisanej w momencie przechwycenia. Wyłączone = eksport w natywnej geometrii poszczególnych czujników.

### Przytnij do wspólnego nakładania się

* **Typ**: Pole wyboru (aktywne tylko wtedy, gdy opcja *Zastosuj wyrównanie tablicy* jest włączona)
* **Domyślnie**: Włączone (zaznaczone)
* **Zapisane jako**: `Processing → "Array alignment crop"`
* **Opis**: Przycina wyrównane eksporty do obszaru wspólnego dla wszystkich modułów kamery, dzięki czemu każde pasmo ma ten sam obszar pokrycia. Wyłączone zachowuje pełny obszar matrycy (czarne wypełnienie poza źródłem).

### Próbkowanie

* **Typ**: Lista rozwijana (aktywna tylko przy włączonej opcji *Zastosuj wyrównanie macierzy*)
* **Opcje**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Domyślnie**: Bilinearne
* **Zapisywane jako**: `Processing → "Array alignment interpolation"`
* **Opis**: Interpolacja stosowana przez transformację wyrównującą. Opcja „Najbliższa” zachowuje dokładne wartości źródłowe (bez mieszania pikseli) na potrzeby ścisłej analizy radiometrycznej; opcja „Dwuliniowa” najlepiej sprawdza się w przypadku mapowania i zastosowań wizualnych.

Te same trzy opcje występują w wersji bez nagłówka jako `chloros-cli process --array-alignment`, `--array-alignment-crop` i `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Te ustawienia pozwalają skonfigurować indeksy wielospektralne do analizy i wizualizacji.

### Dodaj indeks

* **Typ**: Specjalny panel konfiguracji indeksów
* **Opis**: Otwiera interaktywny panel, w którym można wybrać i skonfigurować wielospektralne indeksy roślinności (NDVI

,NDRE

,EVI

itp.) do obliczenia podczas przetwarzania obrazu. Można dodać wiele wskaźników, z których każdy ma własne ustawienia wizualizacji.
* **Dostępne wskaźniki**: Lista rozwijana w interfejsie graficznym zawiera**27** predefiniowanych wzorów na wskaźniki wielospektralne (pełna lista, w tym nazwy akceptowane również przez opcjęCLI

/SDK

`--indices`, znajduje się w sekcji [Wzory na wskaźniki wielospektralne](multispectral-index-formulas.md)).
* **Funkcje**:
  * Wybór spośród gotowych wzorów wskaźników
  * Przeciąganie kanałów filtrów kamery do odpowiednich pól pasmowych wzoru
  * Konfiguracja gradientów kolorów wizualizacji (LUT – tabele odnośników)
  * Ustawianie wartości progowych i trybów przycinania
  * Tworzenie niestandardowych wzorów indeksów
* **Uwaga**: Indeksy nie są obliczane dla jednopasmowych kamer monochromatycznych LATTICE M3M — indeksy wielopasmowe są niezdefiniowane w przypadku jednego pasma. Nie ma to wpływu na modeleSurvey3

i LATTICE M3C.



<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Każdy dodany wskaźnik renderuje swoją formułę jako wyrażenie matematyczne, z kolorowym kółkiem dla każdego pola pasma: czerwony =Red

, zielony =Green

, niebieski =Blue

, pomarańczowy =Orange

, cyjan =Cyan

, fioletowy =NIR

, magenta = RE. Przeciągnij kółko z wiersza nad formułą na pole, aby je przypisać; kliknij dwukrotnie powiązane pole, aby je wyczyścić. Indeks jest obliczany tylko wtedy, gdy każde pole używane w formule ma przypisany kanał.

### Formuły niestandardowe (funkcjaChloros

+)

* **Typ**: Tablica definicji formuł niestandardowych
* **Dostępność**: Wymaga zalogowania się przy użyciu kwalifikującej się subskrypcjiChloros

+.
* **Opis**: Umożliwia tworzenie i zapisywanie niestandardowych formuł indeksów wielospektralnych z wykorzystaniem operacji matematycznych na pasmach. Formuły niestandardowe są zapisywane wraz z ustawieniami projektu i można z nich korzystać tak samo jak z wbudowanych indeksów.
* **Jak utworzyć**:
  1. W panelu konfiguracji indeksów otwórz kalkulator formuł niestandardowych
  2. Wpisz formułę, używając **symboli pasm**, a nie nazw pasm
  3. Zapisz wzór pod opisową nazwą — pojawi się on wówczas na dole listy rozwijanej wzorów, a koła kanałów z kamery można przeciągać na odpowiednie pola dokładnie tak samo, jak w przypadku wbudowanych ustawień wstępnych
* **Składnia wzoru**:
  * Sloty pasm: `x`, `y`, `z`, `a`, `b`, `c` — sześć pozycji, które przypisuje się do rzeczywistych kanałów poprzez przeciągnięcie
  * Operatory: `+`, `-`, `*`, `/`, `^` oraz `()` w celu zgrupowania
  * Funkcje: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Dlaczego symbole, a nie nazwy zespołów**: formuła zapisana jako `(y-x)/(y+x)` działa na każdym aparacie, ponieważ mapowanie metodą „przeciągnij i upuść” decyduje o tym, czy `y` jest wartościąNIR

dla filtra 850 nm (RGN

) czy wartościąNIR

dla filtra 808 nm (OCN

). Wbudowane ustawienia wstępne są przechowywane w ten sam sposób — dokładną postać symbolową wszystkich 27 ustawień można znaleźć w sekcji [Wzory indeksów wielospektralnych](multispectral-index-formulas.md).
* **Gdzie działają**: formuły niestandardowe są zapisywane wraz z ustawieniami projektu i można z nich korzystać zarówno w [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md), jak i podczas przetwarzania. Nie są one**akceptowane** przez listę nazwCLI

/SDK

`--indices`, która rozszerza jedynie nazwy 22 wbudowanych ustawień wstępnych.

***

## Eksport

Te ustawienia kontrolują format i jakość eksportowanych, przetworzonych obrazów.

### Format obrazu skalibrowanego

* **Typ**: Lista rozwijana
* **Opcje**:
  * **TIFF

(16-bit)** – nieskompresowany 16-bitowy formatTIFF


  * **TIFF

(32-bit, Percent)** – 32-bitowy format zmiennoprzecinkowyTIFF

z wartościami odbicia wyrażonymi w procentach
  * **PNG

(8-bit)** – skompresowany 8-bitowy formatPNG


  * **JPG (8-bitowy)** – skompresowany 8-bitowy formatJPEG


* **Domyślnie**:TIFF

(16-bitowy)
* **Opis**: Wybiera format pliku do zapisywania przetworzonych i skalibrowanych obrazów. Z eksportowane pliki trafiają do podfolderów przypisanych do poszczególnych formatów w obrębie każdego folderu kamery(`tiff16`, `tiff32`, `png8`, `jpg8`), przy czym na każdy produkt przypada jeden folder `<Product>_Images/`. Eksportowane pliki zachowują nazwę pliku źródłowego — to folder, a nie rozszerzenie nazwy pliku, identyfikuje produkt.
* **Zalecenia dotyczące formatu**:
  * **TIFF

(16-bitowy)**: Zalecany do analiz naukowych i profesjonalnych procesów roboczych. Zapewnia maksymalną jakość danych bez artefaktów kompresji. Najlepszy do analizy wielospektralnej i dalszego przetwarzania w oprogramowaniu GIS.
  * **TIFF

(32-bitowy, procent)**: Najlepszy do procesów wymagających wartości odbicia wyrażonych w procentach (0–100%). Zapewnia maksymalną precyzję pomiarów radiometrycznych.
  * **PNG

(8-bitowy)**: Dobry do przeglądania w sieci i ogólnej wizualizacji. Mniejsze rozmiary plików dzięki bezstratnej kompresji, ale z ograniczonym zakresem dynamicznym.
  * **JPG (8-bitowy)**: Najmniejsze rozmiary plików, najlepszy wyłącznie do podglądów i wyświetlania w sieci. Wykorzystuje kompresję stratną, która nie nadaje się do analiz naukowych.
* **Uwaga**: Radiancja LATTICE jest zawsze eksportowana jako 32-bitowa zmiennoprzecinkowa wartość typu „TIFF

”, niezależnie od tego ustawienia.

***

## Zapisz szablon projektu

Ta funkcja pozwala zapisać bieżące ustawienia projektu jako szablon do ponownego wykorzystania.

* **Typ**: Pole tekstowe + przycisk „Zapisz”
* **Opis**: Wprowadź opisową nazwę szablonu ustawień i kliknij ikonę zapisu. Szablon zapisze wszystkie bieżące ustawienia projektu (wykrywanie obiektów, opcje przetwarzania, indeksy i format eksportu), co ułatwi ich ponowne wykorzystanie w przyszłych projektach. Szablony są przechowywane w folderze `Project Templates/` znajdującym się w folderze zapisu projektu i można je również wybierać lub eksportować z menu głównego (*Wybierz szablon* / *Zapisz szablon* / *Eksportuj szablon*).
* **Przykłady zastosowań**:
  * Twórz szablony dla różnych systemów kamer (RGB

, multispektralne,NIR

)
  * Zapisuj standardowe konfiguracje dla konkretnych rodzajów upraw lub procesów analitycznych
  * Udostępniaj spójne ustawienia w całym zespole
* **Sposób użycia**:
  1. Skonfiguruj wszystkie żądane ustawienia projektu
  2. Wprowadź nazwę szablonu (np. „RedEdge

Survey3

NDVI

Standard”)
  3. Kliknij ikonę zapisu
  4. Szablon można teraz wczytać podczas tworzenia nowych projektów

***

## Zapisywanie folderu projektu

To ustawienie określa, gdzie domyślnie zapisywane są nowe projekty.

* **Typ**: Wyświetlanie ścieżki katalogu + przycisk edycji
* **Domyślnie (Windows

)**: `C:\Users\[Username]\Chloros Projects`
* **Domyślnie (Linux

)**: `~/Chloros Projects`
* **Opis**: Pokazuje aktualny katalog domyślny, w którym tworzone są nowe projektyChloros

. Kliknij ikonę edycji, aby wybrać inny katalog. Zmiana ta jest zapisywana jako pojedynczy wiersz tekstu w pliku `~/.chloros/working_directory.txt` — na serwerzeWindows

jest to plik `C:\Users\<Username>\.chloros\working_directory.txt`. Jeśli ten plik nie istnieje lub wskazuje ścieżkę, która już nie istnieje, program „Chloros

” powróci do powyższego ustawienia domyślnego. PlikCLI

odczytuje i zapisuje ten sam plik, więc `chloros-cli` i interfejs graficzny zawsze są zgodne co do lokalizacji projektów.
* **Szablony projektów** znajdują się w podfolderze `Project Templates/` tego katalogu.
* **Kiedy należy zmienić**:
  * Ustaw dysk sieciowy w celu współpracy zespołowej
  * Zmień na dysk z większą przestrzenią dyskową w przypadku dużych zbiorów danych
  * Porządkuj projekty według roku, klienta lub typu projektu w różnych folderach
* **Uwaga**: Zmiana tego ustawienia ma wpływ wyłącznie na NOWE projekty. Istniejące projekty pozostają w swoich pierwotnych lokalizacjach.***

## Trwałość ustawień

Projekt w programie „Chloros

” to **folder**. Wszystkie ustawienia projektu są zapisywane w folderze `project.json` znajdującym się w jego obrębie; podłączony sprzęt jest zapamiętywany wraz z nim w folderach `cameras.json` i `sensors.json`, więc ponowne otwarcie projektu powoduje również ponowne podłączenie kamer i czujników światła. Po ponownym otwarciu projektu wszystkie ustawienia są przywracane dokładnie tak, jak je pozostawiono. Zapisane projekty można również obsługiwać bez interfejsu graficznego za pomocą pliku `chloros-cli project` lub pliku `open_project` z pakietu „SDK

”.

### Hierarchia ustawień

Ustawienia są stosowane w następującej kolejności:

1. **Ustawienia domyślne systemu** – wbudowane ustawienia domyślne zdefiniowane przezChloros

2. **Ustawienia szablonu** – jeśli podczas tworzenia projektu wczytasz szablon
3. **Ustawienia zapisanego projektu** – ustawienia zapisane wraz z plikiem projektu
4. **Ręczne dostosowania** – wszelkie zmiany wprowadzone podczas bieżącej sesji

### Ustawienia a przetwarzanie obrazów

Ustawienia przetwarzania są odczytywane w momencie rozpoczęcia cyklu przetwarzania. Zmiana ustawienia nie powoduje retroaktywnej modyfikacji produktów, które już znajdują się na dysku — aby zastosować nowe ustawienia, należy ponownie uruchomić przetwarzanie. Niektóre ustawienia w ogóle nie mają wpływu na przetwarzanie:

* Rozdzielczość miniatur obrazów (tylko do wyświetlania)
* Zapisz szablon projektu
* Zapisz folder projektu

***

## Opis kluczy konfiguracyjnych

W przypadku automatyzacji (CLI

`--config`,SDK

`configure` lub bezpośredniego odczytu `project.json`) są to dokładne klucze znajdujące się w pliku `Project Settings`:

| Ścieżka klucza | Typ | Wartość domyślna |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | liczba 0–10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | liczba 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | logiczny | `true` |
| `Processing → Export vignette corrected` | logiczny | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | liczba 0–3600 | `0` |
| `Processing → Light sensor timezone offset` | liczba -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | identyfikator identyfikator profilu lub `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista konfiguracji indeksów | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Klucze `Array alignment` są zapisywane przy pierwszym renderowaniu sekcji „Wyrównanie tablicy” lub gdy wywołanie automatyzacji je ustawia. Gdy ich brakuje, potok wykorzystuje te same wartości, co powyżej (`true`, `true`, bilinearne), więc projekt .json bez nich zachowuje się identycznie jak ten z nimi.

### Klucze przechowywane w `project.json` bez możliwości sterowania w panelu ustawień

Znajdują się one w tym samym drzewie `Project Settings` i są odczytywane podczas przetwarzania, ale w pasku bocznym nie ma dla nich żadnego widżetu:

| Ścieżka klucza | Typ | Domyślna wartość | Ustawiana przez |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`,SDK

`input_level=`. Zastępuje sposób interpretacji plików TIFF wejściowych LATTICE; `auto` wywodzi się z tagu XMP każdego pliku `Chloros:ProcessingLevel` oraz liczby kanałów. Ignorowane w przypadkuSurvey3

`.raw` . Celowo nie jest to ustawienie w interfejsie graficznym — wartość „auto” jest poprawna we wszystkich typowych przypadkach. |
| `Processing → Target reflectance dir` | ciąg ścieżki | `""` | `chloros-cli process --target-reflectance-dir`, lub cel projektuAPI

|
| `Processing → Target reflectance config` | słownik indeksowany numerem seryjnym kamery | `{}` | Rejestracja celu w kadrze (tryb `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | ciąg znaków ścieżki | `""` |SDK

`process_folder(daq_log_path=…)`. Wskazuje na nagranie `.daq` lub folder zawierający takie nagrania |
| `Target Detection → Minimum calibration target squares` | liczba | `4` | Domyślne ustawienie starszej wersji; brak kontroli i brak flagi „CLI

” |
| `UI → Grid thumbnail size` | liczba | `160` | Własny suwak powiększania miniatur siatki obrazów |

Dwa ustawienia przeglądarki są przechowywane **na najwyższym poziomie w `project.json`**, całkowicie poza `Project Settings`, ponieważ dotyczą one stanu wyświetlania, a nie ustawień przetwarzania:

| Ścieżka klucza | Typ | Wartość domyślna | Ustawiana przez |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | liczba całkowita 1–256 | `1` | Kontrola GSD (px) — zobacz [Otwieranie obrazu na pełnym ekranie](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Najlepsze praktyki

1. **Zacznij od ustawień domyślnych**: Ustawienia domyślne sprawdzają się dobrze w przypadku większości systemów kamerMAPIR

nych i typowych procesów pracy.
2. **Twórz szablony**: Po zoptymalizowaniu ustawień dla konkretnego przepływu pracy lub kamery zapisz je jako szablon, aby zapewnić spójność w różnych projektach.
3. **Przetestuj przed pełnym przetwarzaniem**: Eksperymentując z nowymi ustawieniami, przetestuj je na niewielkiej podgrupie obrazów przed przetworzeniem całego zbioru danych.
4. **Dokumentuj swoje ustawienia**: Używaj opisowych nazw szablonów, które wskazują system kamer, rodzaj przetwarzania i przeznaczenie (np. „Survey3

_RGB_NDVI_Agriculture”).
5. **Wybór formatu eksportu**: Wybierz format eksportu w zależności od docelowego zastosowania:
   * Analiza naukowa →TIFF

(16-bitowy lub 32-bitowy)
   * Przetwarzanie GIS →TIFF

(16-bitowy)
   * Szybka wizualizacja →PNG

(8-bitowy)
   * Udostępnianie w sieci → JPG (8-bitowy)

***

Więcej informacji na temat wskaźników wielospektralnych wChloros

można znaleźć na stronie [Wzory wskaźników wielospektralnych](multispectral-index-formulas.md).
