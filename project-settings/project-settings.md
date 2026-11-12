# Ustawienia projektu

Pasek boczny „Ustawienia projektu” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

w programie Chloros pozwala skonfigurować wszystkie aspekty przetwarzania obrazów, wykrywania celów kalibracyjnych, obliczania wskaźników wielospektralnych oraz opcji eksportu dla danego projektu. Ustawienia te są zapisywane wraz z projektem i można je zapisać jako szablony do ponownego wykorzystania w wielu projektach.

## Dostęp do ustawień projektu

Aby uzyskać dostęp do ustawień projektu:

1. Otwórz projekt w programie Chloros


2. Kliknij kartę **Ustawienia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

na lewym pasku bocznym
3. W panelu ustawień wyświetlą się wszystkie dostępne opcje konfiguracyjne uporządkowane według kategorii

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Ustawienia zależne od innych ustawień są wyszarzone.** Gdy przełącznik nadrzędny uniemożliwia daną opcję (na przykład odznaczenie opcji *Kalibracja odbicia / balans bieli* uniemożliwia *Eksport odbicia*), zależna opcja jest wyłączona, a jej podpowiedź wskazuje przełącznik, który należy zmienić.
{% endhint %}

***

## Wyświetlanie

### Rozdzielczość miniatur obrazów

* **Typ**: Lista rozwijana
* **Opcje**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Domyślnie**: Domyślna (512 pikseli)
* **Opis**: Rozdzielczość (najdłuższy bok, w pikselach), w której renderowane są miniatury w siatce obrazów. Wyższe wartości zapewniają ostrzejszy obraz po powiększeniu, ale powodują wolniejsze ładowanie i zużywają więcej pamięci. Pełna rozdzielczość odpowiada oryginalnemu rozmiarowi obrazu.
* **Uwaga**: Tylko do wyświetlania — nie ma to żadnego wpływu na przetwarzanie ani na eksportowane pliki.

***

## Wykrywanie celów

Te ustawienia kontrolują sposób, w jaki Chloros wykrywa i przetwarza cele kalibracyjne na obrazach. Oba są aktywne tylko wtedy, gdy włączona jest opcja **Kalibracja odbicia / balans bieli** (w przeciwnym razie są wyszarzone, ponieważ wykrywanie celów jest całkowicie pomijane).

### Minimalny obszar próbki kalibracyjnej (px)

* **Typ**: Liczba
* **Zakres**: od 0 do 10 000 pikseli
* **Wartość domyślna**: 25 pikseli
* **Opis**: Określa minimalny obszar (w pikselach) wymagany, aby wykryty region został uznany za prawidłową próbkę celu kalibracyjnego. Mniejsze wartości pozwolą wykrywać mniejsze cele, ale mogą zwiększyć liczbę fałszywych alarmów. Większe wartości wymagają większych i wyraźniejszych obszarów celów do wykrycia.
* **Kiedy dostosować**:
  * Zwiększ wartość, jeśli występują fałszywe wykrycia na małych artefaktach obrazu
  * Zmniejsz wartość, jeśli cele kalibracyjne wydają się małe na obrazach i nie są wykrywane

### Minimalne grupowanie celów (0–100)

* **Typ**: Liczba
* **Zakres**: od 0 do 100
* **Wartość domyślna**: 60
* **Opis**: Kontroluje próg grupowania podobnych obszarów kolorystycznych podczas wykrywania celów kalibracyjnych. Wyższe wartości wymagają zgrupowania większej liczby podobnych kolorów, co skutkuje bardziej konserwatywnym wykrywaniem celów. Niższe wartości pozwalają na większą zmienność kolorów w obrębie grupy celów.
* **Kiedy dostosować**:
  * Zwiększ, jeśli cele kalibracyjne są rozdzielane na wiele wykrytych obiektów
  * Zmniejsz, jeśli cele kalibracyjne o zróżnicowanej kolorystyce nie są w pełni wykrywane

***

## Przetwarzanie

Te ustawienia określają sposób, w jaki program Chloros przetwarza i kalibruje obrazy.

### Korekcja winietowania

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Stosuje korekcję winietowania w celu skompensowania przyciemnienia obiektywu na krawędziach obrazów. Winietowanie to powszechne zjawisko optyczne, w którym rogi i krawędzie obrazu wydają się ciemniejsze niż jego środek ze względu na właściwości obiektywu.
* **Efekt uboczny**: To przełącznik pozwala również wybrać, który *niekalibrowany produkt rezerwowy* zostanie zapisany po zakończeniu przebiegu (patrz poniżej).

### Kalibracja odbicia / balans bieli

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Opis**: Włącza kalibrację odbicia — na podstawie wykrytych w kadrze celów kalibracyjnych i/lub danych o natężeniu światła z czujnika DAQ, w zależności od kamery i dostępnych danych. Normalizuje to wartości odbicia w całym zbiorze danych i zapewnia spójność pomiarów niezależnie od warunków oświetleniowych.
* **Gdy opcja jest wyłączona**: Wykrywanie celów jest całkowicie pomijane i**żadna kamera nie może wygenerować produktu odbicia** — dotyczy to zarówno systemu opartego na celach Survey3, jak i systemu opartego na DAQ LATTICE. Powiązane ustawienia (*Eksportuj odbicie*, *Minimalny interwał ponownej kalibracji* oraz progi wykrywania celów) są wyszarzone.

### Produkty rezerwowe dla danych nieskalibrowanych: Eksport odpowiedzi czujnika / Eksport z korekcją winietowania

* **Typ**: Dwa pola wyboru
* **Ustawienia domyślne**: Oba włączone (zaznaczone)
* **Opis**: Gdy nie można przeprowadzić kalibracji współczynnika odbicia dla klatki (nie znaleziono celu kalibracyjnego lub kalibracja współczynnika odbicia jest wyłączona), jest ona zapisywana jako *niekalibrowany produkt rezerwowy*. **Dla każdego modelu kamery istnieje dokładnie jeden z dwóch produktów rezerwowych na każdy przebieg**, wybrany za pomocą przełącznika *Korekcja winietowania*:
  * Korekcja winietowania **włączona**→ `Vignette_Corrected_Images/` (regulowana przez opcję**Eksportuj z korekcją winietowania**)
  * Korekcja winietowania **wyłączona**→ `Sensor_Response_Images/` (regulowana przez opcję**Eksportuj charakterystykę czujnika**)
* Produkt rezerwowy, który nie jest aktualnie używany, jest wyszarzony. Odznaczenie opcji aktualnie używanego produktu powoduje całkowite wstrzymanie zapisu tego pliku.

### Produkty eksportowe LATTICE

W przypadku projektów zawierających ujęcia LATTICE każda zaimportowana klatka LATTICE jest rozdzielana na wszystkie włączone **i odpowiednie**produkty w ramach jednego przebiegu przetwarzania. Cztery pola wyboru kontrolują rozdzielanie (wszystkie domyślnie**włączone**):

| Ustawienie | Folder wyjściowy | Co eksportuje |
| --- | --- | --- |
| **Eksport po usunięciu efektu bayera** | `Debayered_Images/` | Obraz po liniowym usunięciu efektu bayera: po usunięciu ciemnego przesunięcia z pakietu kalibracyjnego, a następnie z zastosowaniem demosaic uwzględniającego krawędzie, bez korekcji liniowości, płaskiego pola ani demiksowania — ten sam plik, jaki ujęcie zapisuje na poziomie po usunięciu efektu bayera. Zachowana jest natywna orientacja czujnika (zapisana w XMP). Dotyczy kamer RGB i wielospektralnych. |
| **Eksport podglądu** | `Preview_Images/` | Podgląd wyświetlany na ekranie. RGB = balans bieli (źródło światła DAQ, jeśli dostępne, w przeciwnym razie „szary świat”) + gamma; wielospektralne = rozciągnięcie w fałszywych kolorach. |
| **Eksport promieniowania** | `Radiance_Images/` | Promieniowanie spektralne typu Float32 w W/m²/sr/nm. Tylko dla obrazów wielospektralnych (M3C/M3M) — nie dotyczy obrazów źródłowych RGB. Zawsze zapisywane jako 32-bitowy plik TIFF, niezależnie od ustawienia *Kalibrowany format obrazu*. |
| **Eksport współczynnika odbicia**| `Reflectance_Calibrated_Images/` | Współczynnik odbicia typu Uint16, skalowany tak, że**32768 = współczynnik odbicia 1,0** (oznaczone jako XMP `Chloros:PixelScale`). Tylko w przypadku obrazów wielospektralnych, zapisywane, gdy pasujący rekord `.daq` dla promieniowania padającego (lub cel w ramce, który przeszedł kontrolę jakości) obejmuje daną ramkę. |

* Kamery główne typu RGB emitują dane po debayeringu + podgląd; promieniowanie/odbicie są w ich przypadku pomijane, ponieważ nie mają zastosowania.
* Głębia bitowa danych po debayeringu/podglądu zależy od ustawienia *Format obrazu skalibrowanego*; promieniowanie ma zawsze typ float32.
* Te cztery przełączniki nie mają wpływu na przetwarzanie danych typu *Survey3 *.
* Produkty promieniowania i odbicia z wielospektralnych kamer M3C mogą również zapisywać **maskę ograniczenia** do folderu `clamp_masks/` znajdującego się obok folderów produktów, rejestrując miejsca, w których podczas rozdzielania pasm wartość ujemna została ograniczona do zera. Zobacz [Formaty obrazów wyjściowych](../output-image-formats.md#clamp-masks).

Te same cztery przełączniki istnieją w trybie bezinterfejsu graficznego jako `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Zastąpiły one starą flagę CLI’ową `--radiometric-output`, która już nie istnieje.

{% hint style="warning" %}
**Wyłączenie wszystkich odpowiednich produktów powoduje niepowodzenie przebiegu.** Od wersji 1.2.0 przebieg przetwarzania, w którym zażądano produktów, ale nie zapisano żadnego produktu obrazowego, zgłasza błąd, a interfejs CLI kończy działanie z wynikiem innym niż zero, zamiast zgłaszać ciche zakończenie sukcesem. W logu podana jest nazwa produktu, którego nie udało się zapisać, oraz przyczyna. Celowy przebieg zawierający wyłącznie metadane (bez żadnych żądań) nadal kończy się powodzeniem.
{% endhint %}

### Źródło odbicia (ustawienie projektu, konfigurowane za pomocą CLI)

Projekt przechowuje również informację o tym, jakiego **odniesienia odbicia** używa produkt odbicia LATTICE. W panelu ustawień nie ma dedykowanego elementu sterującego; wartość jest przechowywana w konfiguracji projektu jako `Processing → "Target reflectance source"` i ustawia się ją za pomocą `chloros-cli process --reflectance-source {auto,target,daq}` (w SDK `configure()` nie ma dla tego parametru słowa kluczowego):

* **`auto`** (domyślnie): cel kalibracyjny w ramce, który przeszedł kontrolę jakości (QA), staje się odniesieniem bezwzględnym; w przypadku braku celu lub niepowodzenia kontroli jakości stosowana jest wartość dzielnika promieniowania padającego z systemu DAQ (ρ = πL/E). Cel, który przeszedł kontrolę jakości z flagą (opuszczony panel lub niesprawne pasmo), nie jest automatycznie preferowany: łańcuchy „cel”, „cel+DAQ” i „tylko DAQ” są wypróbowywane w kolejności zgodnej z ich wynikami pomiarowymi na panelach testowych.
* **`target`**: ścisła reflektancja oparta na celu — bez zastępowania przez DAQ.
* **`daq`**: reflektancja oparta na DAQ; cele w kadrze nie są wykorzystywane jako odniesienie.

Zapisana wartość jest dopasowywana bez uwzględniania wielkości liter, a kilka wariantów pisowni jest akceptowanych jako aliasy: `target`, `target_image`, `empirical` oraz `empirical_line` – wszystkie oznaczają **cel**; `daq`, `dls`, `light_sensor` oraz `sensor` oznaczają**daq**. Wszystko inne — w tym brak klucza — jest interpretowane jako**auto**.

Skanowanie **zmierzonych** celów w poszczególnych jednostkach jest wyszukiwane na podstawie numeru seryjnego/kodu QR jednostki docelowej, np. `<serial>.csv`, w trzech miejscach: w katalogu podanym przez `--target-reflectance-dir` (zapisanym jako `Processing → "Target reflectance dir"`), w folderze projektu `target_reflectance/`, oraz w ścieżce określonej przez zmienną środowiskową `CHLOROS_TARGET_REFLECTANCE_DIR`. Jeśli dla danej jednostki nie istnieje skan pomiarowy, zamiast niego wykorzystywana jest nominalna, opublikowana krzywa dla danego modelu docelowego.

### Metoda debayera

* **Typ**: Wybór z listy rozwijanej
* **Opcje**:
  * Standardowa (szybka, średnia jakość)
  * Z uwzględnieniem tekstury (powolna, najwyższa jakość) \[Chloros+]
* **Domyślnie**: Standardowa (szybka, średnia jakość)
* **Opis**: Wybiera algorytm demosaicingu używany do konwersji surowych danych z czujnika o układzie Bayera na pełnokolorowe obrazy. Metoda „Standardowa (szybka, średnia jakość)” zapewnia optymalną równowagę między szybkością przetwarzania a jakością obrazu. Metoda „Z uwzględnieniem tekstury (powolna, najwyższa jakość)” \[Chloros+] wykorzystuje wysokiej jakości algorytm demosaicingu uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na sztucznej inteligencji (AI/ML), który eliminuje niemal cały szum powstały podczas demosaicingu. Model uwzględniający teksturę wymaga do działania pamięci GPU (VRAM). Zalecamy korzystanie z niego, gdy masz do dyspozycji >4 GB pamięci VRAM, co zapewni szybsze przetwarzanie.
* **Gdy wiersz jest rozwijanym menu**: menu rozwijane z dwiema opcjami pojawia się tylko wtedy, gdy**oba** warunki są spełnione — użytkownik jest zalogowany z kwalifikującą się subskrypcją Chloros+ oraz **projekt** nie zawiera żadnych ujęć LATTICE. W przeciwnym razie wiersz wyświetla się jako zwykły tekst o treści `Standard (Fast, Medium Quality)` bez możliwości wyboru.
* **Uwaga dotycząca LATTICE**: Nie istnieje model Texture Aware wytrenowany dla LATTICE, a potok wymusza standardową demosaikę dla klatek LATTICE niezależnie od zapisanej wartości. Jeśli dodasz folder LATTICE do projektu, w którym opcja Texture Aware była już zaznaczona, Chloros przywróci ustawienie do wartości Standard zamiast pozostawić nieaktualną wartość w `project.json`.

### Minimalny interwał ponownej kalibracji

* **Typ**: Liczba
* **Zakres**: od 0 do 3 600 sekund
* **Wartość domyślna**: 0 sekund
* **Opis**: Ustawia minimalny odstęp czasowy (w sekundach) między wykorzystaniem celów kalibracyjnych. Gdy wartość wynosi 0, Chloros wykorzysta każdy wykryty cel kalibracyjny. Gdy ustawiona zostanie wyższa wartość, Chloros wykorzysta tylko cele kalibracyjne, które są od siebie oddalone o co najmniej tę liczbę sekund, co skraca czas przetwarzania zestawów danych zawierających częste ujęcia celów kalibracyjnych.
* **Kiedy dostosować**:
  * Ustaw wartość 0, aby uzyskać maksymalną dokładność kalibracji przy zmiennych warunkach oświetleniowych
  * Zwiększ wartość (np. do 60–300 sekund), aby przyspieszyć przetwarzanie, gdy oświetlenie jest stałe, a obrazy celów kalibracyjnych pojawiają się często

### Przesunięcie strefy czasowej czujnika światła

* **Typ**: Liczba
* **Zakres**: od -12 do +12 godzin
* **Wartość domyślna**: 0 godzin
* **Opis**: Określa przesunięcie strefy czasowej (w godzinach względem czasu UTC) dla znaczników czasu danych z czujnika światła, wykorzystywane podczas dopasowywania dzienników czujnika światła do czasów wykonania zdjęć. Nowsze nagrania `.daq` zawierają własną informację o strefie czasowej, więc opcja ta jest potrzebna głównie w przypadku starszych dzienników zarejestrowanych w czasie lokalnym.

### Zastosuj korekty PPK

* **Typ**: Pole wyboru
* **Domyślnie**: Wyłączone (niezaznaczone)
* **Opis**: Włącza stosowanie korekcji kinematycznych po przetworzeniu (PPK) z rejestratorów danych MAPIR DAQ wyposażonych w GPS (GNSS). Po włączeniu tej opcji program Chloros wykorzysta wszystkie pliki dziennika .daq zawierające dane dotyczące czasu naświetlania, znajdujące się w katalogu projektu, i zastosuje precyzyjne korekcje geolokalizacyjne do zdjęć.
* **Wymagania**: W katalogu projektu musi znajdować się plik dziennika .daq zawierający wpisy dotyczące pinów ekspozycji
* **Kiedy włączyć**: Zaleca się zawsze włączać korekcję PPK, jeśli w pliku dziennika .daq znajdują się wpisy dotyczące ekspozycji.

### Pin ekspozycji 1

* **Typ**: Lista rozwijana
* **Widoczność**: Widoczne tylko wtedy, gdy opcja „Zastosuj korekty PPK” jest włączona ORAZ dostępne są dane ekspozycji dla punktu 1
* **Opcje**:
  * Nazwy modeli aparatów wykrytych w projekcie
  * „Nie używaj” – zignoruj ten punkt ekspozycji
* **Ustawienie domyślne**: Wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: Przypisuje konkretną kamerę do pinu ekspozycji 1 w celu synchronizacji czasowej PPK. Pin ekspozycji rejestruje dokładny moment wyzwolenia migawki kamery, co ma kluczowe znaczenie dla dokładnej geolokalizacji PPK.
* **Sposób automatycznego wyboru**:
  * Jedna kamera + jeden pin: Automatyczny wybór kamery
  * Jedna kamera + dwa piny: Pin 1 automatycznie przypisywany do kamery
  * Wiele kamer: Wymagany wybór ręczny

### Pin ekspozycji 2

* **Typ**: Wybór z listy rozwijanej
* **Widoczność**: Widoczne tylko wtedy, gdy opcja „Zastosuj korekty PPK” jest włączona ORAZ dostępne są dane ekspozycji dla pinu 2
* **Opcje**:
  * Nazwy modeli kamer wykrytych w projekcie
  * „Nie używaj” – zignoruj ten pin ekspozycji
* **Ustawienie domyślne**: Wybierane automatycznie na podstawie konfiguracji projektu
* **Opis**: Przypisuje konkretną kamerę do pinu ekspozycji 2 w celu synchronizacji czasowej PPK w przypadku konfiguracji z dwiema kamerami.
* **Sposób automatycznego wyboru**:
  * Pojedyncza kamera + pojedynczy pin: Pin 2 jest automatycznie ustawiany na „Nie używaj”
  * Pojedyncza kamera + dwa piny: Pin 2 jest automatycznie ustawiany na „Nie używaj”
  * Wiele kamer: Wymagany jest wybór ręczny
* **Uwaga**: Ta sama kamera nie może być przypisana jednocześnie do styku 1 i styku 2.

***

## Czujnik światła DAQ

Ta sekcja znajduje się w ustawieniach projektu i zawiera listę wszystkich plików DAQ dotyczących promieniowania padającego w projekcie — zapisy `.daq` oraz logi DAQ-M `.csv`. Nagrania wykonane w zakładce „Czujniki światła” są automatycznie dodawane do otwartego projektu.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Każdy wiersz zawiera plik, model czujnika oraz korekcję nasadki rozpraszającej faktycznie obowiązującą dla tego pliku. Nad wierszami znajduje się pojedynczy element sterujący dla całego projektu:

### Nadpisanie ustawień nasadki (wszystkie pliki)

* **Typ**: lista rozwijana
* **Opcje**: `Auto` oraz profile korekcji nasadki obowiązujące dla typów czujników obecnych w projekcie
* **Domyślnie**: Auto
* **Zapisywane jako**: `Processing → "DAQ cap id"` (domyślnie `auto`)
* **Opis**: `Auto` wykorzystuje osłonę zarejestrowaną w każdym pliku (w przypadku braku rejestracji przyjmowana jest osłona typu „Sunshine”, jednak urządzenia DAQ-U, DAQ-M i DAQ-E są dostarczane bez osłony, a program Chloros nie jest w stanie rozpoznać, jaka osłona została zamontowana, więc podczas przetwarzania pojawia się ostrzeżenie dotyczące domyślnej nasadki: należy wybrać nasadkę, która była zamontowana). Wybranie konkretnej nasadki zastępuje ustawienia **wszystkich** plików pomiarów w kierunku w dół w projekcie, z wyjątkiem plików z DAQ-E-S, którego dyfuzor stanowi część kalibracji; w projekcie zawierającym wyłącznie dane z DAQ-E-S nie wyświetla się lista rozwijana: surowe zapisy są korygowane przy użyciu tej nasadki, a zapisy, które już zawierają korekcję, są ponownie kalibrowane (zapisana korekcja jest cofana, a zastosowana zostaje wybrana).
* **Ważne**: Wybrana korekcja musi odpowiadać korekcji fizycznie zamontowanej podczas rejestracji. Ani czujnik, ani oprogramowanie nie są w stanie wykryć fizycznej korekcji — niezgodny identyfikator korekcji powoduje błędną korekcję widm.

Celowo zastosowano **jedną** opcję kontrolną dla całego projektu zamiast list rozwijanych dla poszczególnych plików: ustawienie to dotyczy każdego źródła promieniowania padającego w projekcie.

***

## Wyrównanie macierzy

Ta sekcja pojawia się **tylko** wtedy, gdy co najmniej jeden obraz w projekcie zawiera transformację wyrównania między modułami, którą matryce LATTICE nanoszą w momencie przechwytywania (tagi XMP `Chloros:Alignment*`). Pokazuje, ile obrazów zawiera tagi wyrównania, która kamera jest referencyjna (oznaczenie `REF`) oraz tabelę liczby obrazów dla poszczególnych kamer.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Zastosuj wyrównanie tablicy

* **Typ**: Pole wyboru
* **Domyślnie**: Włączone (zaznaczone)
* **Zapisywane jako**: `Processing → "Array alignment"`
* **Opis**: Przekształca każdy przetworzony produkt (po usunięciu matrycy Bayer / podgląd / promieniowanie / odbicie / indeks) do wspólnej geometrii odniesienia tablicy przy użyciu transformacji zapisanej w momencie przechwycenia. Wyłączone = eksport w natywnej geometrii poszczególnych czujników.

### Przytnij do wspólnego obszaru nakładania się

* **Typ**: Pole wyboru (aktywne tylko wtedy, gdy opcja *Zastosuj wyrównanie tablicy* jest włączona)
* **Domyślnie**: Włączone (zaznaczone)
* **Zapisane jako**: `Processing → "Array alignment crop"`
* **Opis**: Przycina wyrównane eksporty do obszaru wspólnego dla wszystkich modułów kamery, dzięki czemu każde pasmo ma ten sam obszar pokrycia. Wyłączenie tej opcji zachowuje pełny obszar czujnika (czarne wypełnienie poza źródłem).

### Próbkowanie

* **Typ**: Lista rozwijana (aktywna tylko przy włączonej opcji *Zastosuj wyrównanie macierzy*)
* **Opcje**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Domyślnie**: Bilinearne
* **Zapisywane jako**: `Processing → "Array alignment interpolation"`
* **Opis**: Interpolacja stosowana podczas zniekształcenia wynikającego z wyrównania. Opcja „Najbliższe” zachowuje dokładne wartości źródłowe (bez mieszania pikseli) na potrzeby ścisłej analizy radiometrycznej; opcja „Bilinearna” najlepiej sprawdza się w przypadku mapowania i zastosowań wizualnych.

Te same trzy opcje występują w wersji bez przedrostka jako `chloros-cli process --array-alignment`, `--array-alignment-crop` i `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeksy

Te ustawienia pozwalają skonfigurować indeksy wielospektralne do analizy i wizualizacji.

### Dodaj indeks

* **Typ**: Specjalny panel konfiguracji indeksów
* **Opis**: Otwiera interaktywny panel, w którym można wybierać i konfigurować wielospektralne indeksy roślinności (NDVI, NDRE, EVI itp.) do obliczenia podczas przetwarzania obrazu. Można dodać wiele indeksów, z których każdy ma własne ustawienia wizualizacji.
* **Dostępne wskaźniki**: Lista rozwijana w interfejsie graficznym zawiera**27** predefiniowanych wzorów wskaźników wielospektralnych (pełna lista, w tym nazwy akceptowane również przez opcję CLI/SDK `--indices`).
* **Funkcje**:
  * Wybór spośród predefiniowanych wzorów indeksów
  * Przeciąganie kanałów filtrów kamery do odpowiednich pól pasmowych wzoru
  * Konfiguracja gradientów kolorów wizualizacji (LUT – tabele odnośników)
  * Ustawianie wartości progowych i trybów przycinania
  * Tworzenie niestandardowych wzorów indeksów
* **Uwaga**: Indeksy nie są obliczane dla jednopasmowych kamer monochromatycznych LATTICE M3M — indeksy wielopasmowe są niezdefiniowane w przypadku jednego pasma. Nie ma to wpływu na modele Survey3 i LATTICE M3C.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Każdy dodany wskaźnik renderuje swoją formułę jako wyrażenie matematyczne, z kolorowym kółkiem dla każdego miejsca na pasmo: czerwony = Red, zielony = Green, niebieski = Blue, pomarańczowy = Orange, cyjan = Cyan, fioletowy = NIR, magenta = RE. Przeciągnij okrąg z wiersza nad formułą na slot, aby go przypisać; kliknij dwukrotnie przypisany slot, aby go wyczyścić. Indeks jest obliczany tylko wtedy, gdy każdy slot używany w formule ma przypisany kanał.

### Formuły niestandardowe (funkcja Chloros+)

* **Typ**: Tablica definicji formuł niestandardowych
* **Dostępność**: Wymaga zalogowania się przy użyciu kwalifikującej się subskrypcji Chloros+.
* **Opis**: Umożliwia tworzenie i zapisywanie niestandardowych formuł indeksów wielospektralnych przy użyciu operacji matematycznych na pasmach. Formuły niestandardowe są zapisywane wraz z ustawieniami projektu i można z nich korzystać tak samo jak z wbudowanych indeksów.
* **Jak utworzyć**:
  1. W panelu konfiguracji indeksu otwórz kalkulator formuł niestandardowych
  2. Wpisz formułę, używając **symboli pasm i slotów**, a nie nazw pasm
  3. Zapisz formułę pod opisową nazwą — pojawi się ona wówczas na dole listy rozwijanej formuł, a koła kanałów z kamery można przeciągać na odpowiednie sloty dokładnie tak samo, jak w przypadku wbudowanych ustawień wstępnych
* **Składnia formuły**:
  * Sloty pasm: `x`, `y`, `z`, `a`, `b`, `c` — sześć pozycji, które można przypisać do rzeczywistych kanałów poprzez przeciągnięcie
  * Operatory: `+`, `-`, `*`, `/`, `^` oraz `()` do grupowania
  * Funkcje: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Dlaczego symbole, a nie nazwy zespołów**: wzór zapisany jako `(y-x)/(y+x)` działa na każdym aparacie, ponieważ mapowanie metodą „przeciągnij i upuść” decyduje, czy `y` to NIR 850 nm filtra RGN, czy NIR 808 nm filtra OCN. Wbudowane ustawienia wstępne są przechowywane w ten sam sposób — zobacz [Wzory indeksów wielospektralnych](multispectral-index-formulas.md), aby poznać dokładną postać symbolową wszystkich 27 wzorów.
* **Gdzie działają**: formuły niestandardowe są zapisywane wraz z ustawieniami projektu i można z nich korzystać zarówno w [Sandboxie indeksów/LUT](../image-viewer-gui/index-lut-sandbox.md), jak i podczas przetwarzania. Nie są one**akceptowane** przez listę nazw CLI/SDK `--indices`, która rozszerza jedynie 22 nazwy wbudowanych ustawień wstępnych.

***

## Eksport

Te ustawienia kontrolują format i jakość eksportowanych, przetworzonych obrazów.

### Format obrazu skalibrowanego

* **Typ**: Lista rozwijana
* **Opcje**:
  * **TIFF (16-bit)** – nieskompresowany format TIFF 16-bitowy
  * **TIFF (8-bitowy)** – nieskompresowany 8-bitowy format TIFF


  * **TIFF (32-bitowy, procentowy)** – 32-bitowy format TIFF z wartościami odbicia wyrażonymi w procentach
  * **PNG (8-bitowy)** – skompresowany 8-bitowy format PNG


  * **JPG (8-bitowy)** — skompresowany 8-bitowy format JPEG


* **Domyślnie**: TIFF (16-bitowy); w przypadku projektu opartego wyłącznie na 8-bitach wyświetla się opcja TIFF (8-bitowy) i dostępne są wyłącznie formaty 8-bitowe
* **Opis**: Wybiera format pliku do zapisywania przetworzonych i skalibrowanych obrazów. Eksportowane pliki są umieszczane w podfolderach odpowiadających poszczególnym formatom wewnątrz folderu każdej kamery (`tiff16`, `tiff32`, `png8`, `jpg8`), przy czym na każdy produkt przypada jeden folder `<Product>_Images/`. Eksportowane pliki zachowują nazwę pliku źródłowego — to folder, a nie rozszerzenie nazwy pliku, identyfikuje produkt.
* **Zalecenia dotyczące formatów**:
  * **TIFF (16-bitowy)**: Zalecany do analiz naukowych i profesjonalnych procesów roboczych. Zapewnia maksymalną jakość danych bez artefaktów kompresji. Najlepszy do analizy wielospektralnej i dalszego przetwarzania w oprogramowaniu GIS.
  * **Format TIFF (32-bitowy, procentowy)**: Najlepszy do procesów roboczych wymagających wartości odbicia wyrażonych w procentach (0–100%). Zapewnia maksymalną precyzję pomiarów radiometrycznych.
  * **PNG (8-bitowy)**: Dobry do przeglądania w sieci i ogólnej wizualizacji. Mniejsze rozmiary plików dzięki bezstratnej kompresji, ale z ograniczonym zakresem dynamicznym.
  * **JPG (8-bitowy)**: Najmniejsze rozmiary plików, najlepszy wyłącznie do podglądów i wyświetlania w sieci. Wykorzystuje kompresję stratną, która nie nadaje się do analiz naukowych.
* **Uwaga**: Dane LATTICE radiance są zawsze eksportowane jako 32-bitowe pliki TIFF typu float, niezależnie od tego ustawienia.

***

## Zapisz szablon projektu

Ta funkcja pozwala zapisać bieżące ustawienia projektu jako szablon, który można ponownie wykorzystać.

* **Typ**: Pole tekstowe + przycisk „Zapisz”
* **Opis**: Wprowadź opisową nazwę szablonu ustawień i kliknij ikonę zapisu. Szablon zapisze wszystkie bieżące ustawienia projektu (wykrywanie obiektów, opcje przetwarzania, indeksy, oraz format eksportu) w celu łatwego ponownego wykorzystania w przyszłych projektach. Szablony są przechowywane w folderze `Project Templates/` w folderze zapisu projektu i można je również wybierać lub eksportować z menu głównego (*Wybierz szablon* / *Zapisz szablon* / *Eksportuj szablon*).
* **Przykłady zastosowań**:
  * Tworzenie szablonów dla różnych systemów kamer (RGB, wielospektralne, NIR)
  * Zapisywanie standardowych konfiguracji dla określonych rodzajów upraw lub procesów analitycznych
  * Udostępnianie spójnych ustawień członkom zespołu
* **Sposób użycia**:
  1. Skonfiguruj wszystkie żądane ustawienia projektu
  2. Wprowadź nazwę szablonu (np. „RedEdge Survey3 NDVI Standard”)
  3. Kliknij ikonę zapisu
  4. Szablon można teraz wczytać podczas tworzenia nowych projektów

***

## Folder zapisu projektów

To ustawienie określa domyślną lokalizację zapisywania nowych projektów.

* **Typ**: Wyświetlanie ścieżki katalogu + przycisk edycji
* **Domyślnie (system Windows)**: `Documents\MAPIR\Chloros Projects` w folderze „Dokumenty” — zazwyczaj `C:\Users\[Username]\Documents\MAPIR\Chloros Projects` lub w miejscu, do którego usługa OneDrive lub profil mobilny przeniosły folder „Dokumenty”
* **Domyślnie (system Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Opis**: Wyświetla bieżący katalog domyślny, w którym tworzone są nowe projekty Chloros. Kliknij ikonę edycji, aby wybrać inny katalog. Zmiana ta jest zapisywana jako pojedynczy wiersz tekstu w pliku `~/.chloros/working_directory.txt` — w systemie Windows jest to plik `C:\Users\<Username>\.chloros\working_directory.txt`. Jeśli ten plik nie istnieje lub wskazuje ścieżkę, która już nie istnieje, Chloros powraca do powyższego ustawienia domyślnego. Interfejs CLI odczytuje i zapisuje ten sam plik, więc `chloros-cli` i interfejs graficzny zawsze są zgodne co do lokalizacji projektów.
* **Szablony projektów** znajdują się w podfolderze `Project Templates/` tego katalogu.
* **Kiedy należy zmienić**:
  * Ustaw na dysk sieciowy w celu współpracy zespołowej
  * Zmień na dysk z większą przestrzenią dyskową w przypadku dużych zbiorów danych
  * Porządkuj projekty według roku, klienta lub typu projektu w różnych folderach
* **Uwaga**: Zmiana tego ustawienia ma wpływ wyłącznie na NOWE projekty. Istniejące projekty pozostają w swoich pierwotnych lokalizacjach.
* **Długość ścieżki w systemie Windows**: o ile nie włączono długich ścieżek w systemie Windows (opcja **Włącz długie ścieżki w systemie Windows** w instalatorze), folder projektu o nazwie dłuższej niż około 130 znaków zostanie odrzucony przed przetworzeniem, ponieważ jego produkty przekroczyłyby limit 260 znaków dla ścieżek w systemie Windows. Zobacz [Rozpoczynanie przetwarzania](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Trwałość ustawień

Projekt Chloros to **folder**. Wszystkie ustawienia projektu są zapisywane w pliku `project.json` znajdującym się w tym folderze; podłączony sprzęt jest zapamiętywany wraz z nimi w plikach `cameras.json` i `sensors.json`, więc ponowne otwarcie projektu powoduje również ponowne podłączenie kamer i czujników światła. Po ponownym otwarciu projektu wszystkie ustawienia są przywracane dokładnie tak, jak zostały pozostawione. Zapisane projekty można również obsługiwać w trybie bezinterfejsu za pomocą pliku `chloros-cli project` lub pliku `open_project` z zestawu SDK.

### Hierarchia ustawień

Ustawienia są stosowane w następującej kolejności:

1. **Ustawienia domyślne systemu** – wbudowane ustawienia domyślne zdefiniowane przez firmę Chloros

2. **Ustawienia szablonu** – jeśli podczas tworzenia projektu wczytasz szablon
3. **Ustawienia zapisanego projektu** – ustawienia zapisane wraz z plikiem projektu
4. **Ręczne dostosowania** – wszelkie zmiany wprowadzone podczas bieżącej sesji

### Ustawienia a przetwarzanie obrazów

Ustawienia przetwarzania są odczytywane w momencie rozpoczęcia cyklu przetwarzania. Zmiana ustawienia nie powoduje retroaktywnej modyfikacji wyników już zapisanych na dysku — aby zastosować nowe ustawienia, należy ponownie uruchomić przetwarzanie. Niektóre ustawienia nie mają żadnego wpływu na przetwarzanie:

* Rozdzielczość miniatur obrazów (tylko do wyświetlania)
* Zapisz szablon projektu
* Zapisz folder projektu

***

## Opis kluczy konfiguracyjnych

W przypadku automatyzacji (CLI: `--config`, SDK: `configure` lub bezpośredniego odczytu `project.json`) są to dokładne klucze znajdujące się w pliku `Project Settings`:

| Ścieżka klucza | Typ | Wartość domyślna |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | liczba 0–10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | liczba 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | liczba 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | liczba -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | identyfikator profilu limitu lub `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista konfiguracji indeksów | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Klucze `Array alignment` są zapisywane przy pierwszym renderowaniu sekcji „Wyrównanie tablicy” lub gdy ustawia je wywołanie automatyzacji. Gdy ich brakuje, potok wykorzystuje te same wartości, co powyżej (`true`, `true`, bilinearne), więc projekt .json bez nich zachowuje się identycznie jak ten z nimi.

### Klucze przechowywane w `project.json` bez możliwości sterowania w panelu ustawień

Znajdują się one w tym samym drzewie `Project Settings` i są odczytywane podczas przetwarzania, ale w pasku bocznym nie ma dla nich widżetu:

| Ścieżka klucza | Typ | Domyślna | Ustawiana przez |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Zastępuje sposób interpretacji plików TIFF wejściowych LATTICE; `auto` wywodzi to z tagu XMP każdego pliku `Chloros:ProcessingLevel` oraz liczby kanałów. Ignorowane w przypadku przechwytywania danych `.raw` w programie Survey3. Celowo nie jest to ustawienie w interfejsie graficznym — opcja „auto” jest poprawna w każdym normalnym przypadku. |
| `Processing → Target reflectance dir` | ciąg znaków ścieżki | `""` | `chloros-cli process --target-reflectance-dir` lub docelowe API projektu |
| `Processing → Target reflectance config` | słownik indeksowany numerem seryjnym kamery | `{}` | Rejestracja celu w kadrze (tryb `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | ciąg znaków ścieżki | `""` | SDK `process_folder(daq_log_path=…)`. Wskazuje na nagranie `.daq` lub folder zawierający takie nagrania |
| `Target Detection → Minimum calibration target squares` | liczba | `4` | Domyślne ustawienie starszego typu; brak kontroli i brak flagi CLI |
| `UI → Grid thumbnail size` | liczba | `160` | Suwak powiększania miniatur w siatce obrazów |

Dwa ustawienia przeglądarki są przechowywane **na najwyższym poziomie w `project.json`**, całkowicie poza `Project Settings`, ponieważ dotyczą one stanu wyświetlania, a nie ustawień przetwarzania:

| Ścieżka klucza | Typ | Wartość domyślna | Ustawiane przez |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | liczba całkowita 1–256 | `1` | Element sterujący GSD (px) na karcie obrazu — zobacz [Otwieranie obrazu na pełnym ekranie](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Najlepsze praktyki

1. **Zacznij od ustawień domyślnych**: Ustawienia domyślne sprawdzają się dobrze w przypadku większości systemów kamerowych MAPIR i typowych procesów pracy.
2. **Twórz szablony**: Po zoptymalizowaniu ustawień dla konkretnego przepływu pracy lub kamery zapisz je jako szablon, aby zapewnić spójność w różnych projektach.
3. **Przetestuj przed pełnym przetwarzaniem**: Eksperymentując z nowymi ustawieniami, przetestuj je na niewielkim podzbiorze obrazów przed przetworzeniem całego zbioru danych.
4. **Dokumentuj swoje ustawienia**: Używaj opisowych nazw szablonów, które wskazują system kamer, rodzaj przetwarzania i przeznaczenie (np. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Wybór formatu eksportu**: Wybierz format eksportu w zależności od przeznaczenia:
   * Analiza naukowa → TIFF (16-bitowy lub 32-bitowy)
   * Przetwarzanie GIS → TIFF (16-bitowy)
   * Szybka wizualizacja → PNG (8-bitowy)
   * Udostępnianie w sieci → JPG (8-bitowy)

***

Więcej informacji na temat wskaźników wielospektralnych w programie Chloros można znaleźć na stronie [Wzory wskaźników wielospektralnych](multispectral-index-formulas.md).
