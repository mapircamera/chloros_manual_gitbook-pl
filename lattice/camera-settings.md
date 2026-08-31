# Ustawienia kamer

Zakładka **Kamery**stanowi panel sterowania na żywo Chloros dla kamer LATTICE: główny obszar podglądu, na którym każda podłączona kamera jest wyświetlana jako kafelek na żywo, oraz pasek boczny, który umożliwia przełączanie między trzema stronami —**listą kamer**,**panelem ustawień**(ustawienia poszczególnych kamer, macierzy lub przechwytywania — po jednym na raz) oraz**Kalkulator indeksu**. Niniejsza strona zawiera opis wszystkich elementów sterujących na liście kamer, w panelu ustawień poszczególnych kamer oraz w panelu ustawień macierzy. Tryby przechwytywania, wybór typu eksportu oraz proces „Przechwyć wszystko” opisano na stronie towarzyszącej [Ustawienia i tryby przechwytywania](capture.md).

Karta „Kamery” pojawia się na pasku bocznym, gdy backend Chloros jest gotowy. Wszystkie poniższe elementy sterujące komunikują się z lokalnym backendem za pośrednictwem `127.0.0.1:5000`; zmiany są natychmiast stosowane do obrazu na żywo z kamery, o ile nie zaznaczono inaczej.

## Typy kamer używane na tej stronie

Elementy sterujące są wyświetlane lub ukrywane w zależności od wybranego typu kamery. W niniejszej instrukcji stosowane są następujące terminy:

| Termin | Znaczenie | Kanały filtra |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C z filtrem FRGB (model zawiera `-FRGB`) | Red / Green / Blue |
| **Bayer wielospektralny** | LATTICE M3C z FRGN, FOCN lub FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — jeden filtr wąskopasmowy, jedno skalibrowane pasmo | Pojedyncze pasmo |
| **Element macierzy** | Kamera podłączona jako część zsynchronizowanej macierzy (wyświetlacz połączony lub oddzielny) | Zgodnie z filtrem |

Kamery RGB przechodzą przetwarzanie fotometryczne (balans bieli, profile kolorów, gamma); kamery wielospektralne i monochromatyczne przechodzą przez łańcuch radiometryczny i pomijają regulacje fotometryczne. Elementy macierzy przekazują ustawienia na poziomie strumienia (format pikseli, rozdzielczość, binning, wyzwalacz, częstotliwość klatek) do macierzy — wiersze te stają się tylko do odczytu w panelu poszczególnych kamer i są przenoszone do panelu ustawień macierzy.

## Główny obszar

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

podglądu Gdy żadna kamera nie jest podłączona, w obszarze podglądu wyświetla się ekran powitalny **„Podłącz kamerę, aby rozpocząć”**z dwoma przyciskami:**Podłącz kamerę**(zielony, otwiera okno dialogowe podłączania pojedynczej kamery) oraz**Podłącz macierz** (niebieski, otwiera okno dialogowe podłączania macierzy). Same okna dialogowe podłączania zostały opisane w sekcji [Podłączanie kamer](connecting.md); pojęcia dotyczące układów kamer (synchronizacja, poziomy, przepustowość) w [Układach wielokamerowych](arrays.md). Po otwarciu zapisanego projektu zawierającego kamery na ekranie powitalnym wyświetla się ikona ładowania z napisem „Ponowne otwieranie N zapisanych kamer…”, podczas gdy Chloros przywraca strumienie z ostatniej sesji.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Pasek górny

| Element sterujący | Funkcja |
| --- | --- |
| **Przełączanie trybu widoku**| Przełącza między**widokiem siatki**(wszystkie kafelki jako komórki) a**widokiem listy** (układy na całej szerokości u góry, JEDNA aktywna kamera poniżej). Podpowiedzi: „Przełącz do widoku siatki” / „Przełącz do widoku listy”. |
| **Blokada siatki**(kłódka) | Domyślnie**zablokowana** — kafelki unieruchomione w miejscu. Odblokuj, aby przeciągać i zmieniać kolejność kafelków w dowolnym miejscu (luki są zachowane). Siatka automatycznie ponownie się blokuje za każdym razem, gdy podłącza się nowa kamera. Wskazówki: „Odblokuj siatkę (włącz przeciąganie kafelków)” / „Zablokuj siatkę (unieruchom kafelki)”. |
| Suwak **Powiększenie strumienia** | Rozmiar kafelka, od 60 pikseli do pełnej szerokości kontenera. Komórki zachowują proporcje 4:3. Przy szerokości komórki poniżej 200 px nakładki z nazwą i liczbą klatek na sekundę są ukrywane, aby kafelek pozostał przejrzysty. |

### Kafelki strumienia

Każda kamera renderuje złożony kafelek na żywo; kamera może dodatkowo wyświetlać trzy kafelki w skali szarości z **podziałem na kanały** (zobacz [Podziały na kanały](#display-overlays-drawn-over-the-live-feed)), a tablice renderują kafelek łączony. Aktywny kafelek posiada pierścień zaznaczenia w kolorze kamery (lub macierzy).

Po najechaniu kursorem na kafelek pojawia się przycisk zamykania **X**:

* Zamknięcie **złożonego** kafelka, podczas gdy jego podziały kanałów pozostają widoczne, powoduje jedynie ukrycie kafelka złożonego.
* Zamknięcie **ostatniego widocznego kafelka samodzielnej kamery** powoduje odłączenie tej kamery.
* **Podzielone kafelki należące do połączonego układu nigdy nie powodują odłączenia** kamery — są jedynie ukrywane.

Przy odblokowanej siatce przeciągnij dowolny kafelek do dowolnego miejsca; układ jest zapisywany wraz z projektem.

## Pasek boczny — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamer Pierwsza strona paska bocznego zawiera listę wszystkich podłączonych kamer i układów:

* **Podłącz kamerę**(zielony) /**Podłącz układ** (niebieski, podczas skanowania wyświetla komunikat „Wykrywanie...”). Obie opcje są nieaktywne, gdy otwarte jest okno dialogowe podłączania.
* **Przechwyć wszystko** (czerwony) — przechwytuje obraz ze wszystkich wymienionych kamer z wykorzystaniem typów eksportu wybranych w ustawieniach przechwytywania. Wymaga otwartego projektu. Pełna dokumentacja znajduje się w sekcji [Ustawienia i tryby przechwytywania](capture.md).
* **Ikona koła zębatego Ustawień przechwytywania** (obok opcji Przechwyć wszystko) — otwiera [panel Ustawień przechwytywania](capture.md#the-capture-settings-pane). Opcja jest wyłączona, jeśli nie ma projektu lub podczas przechwytywania.

### Wiersze kamer

Każdy wiersz kamery zawiera obramowanie oznaczone kolorem (niestandardowy kolor kamery), etykietę „CAM” — z niebieską literą **M**(master) lub zieloną literą**S** (slave) dla elementów macierzy — oraz nazwę wyświetlaną. Domyślna nazwa to `LATTICE-MODEL (serial)`; można ją zmienić w panelu ustawień poszczególnych kamer. Przyciski w wierszu:

| Przycisk | Efekt |
| --- | --- |
| **Oko**| Przełącz widoczność. Ukryte kamery znikają z siatki i są**wykluczane z funkcji „Capture All”**. |
| **Koło zębate** | Otwórz panel ustawień poszczególnych kamer (następna sekcja). |
| **Pauza / Odtwarzanie**| Zatrzymuje podgląd na żywo**tylko po stronie wyświetlacza** — pobieranie danych w tle nadal trwa. Zatrzymane kamery nie mogą rejestrować obrazu. |
| **X** | Rozłącz. Interfejs użytkownika aktualizuje się natychmiast (w idealnym przypadku); samo rozłączenie w tle może zająć 10–30 s. |

### Wiersze tablicy

Wiersz tablicy zawiera etykietę „ARRAY” w kolorze tablicy, nazwę tablicy (można ją zmienić w ustawieniach tablicy) oraz przycisk **DAQ · włącz/wyłącz**—**włączony**, gdy czujnik światła na poziomie macierzy jest włączony *lub* którykolwiek z elementów posiada czujnik przypisany do konkretnej kamery; podpowiedź pokazuje dokładnie, który czujnik dostarcza sygnał do czego. Kamery wchodzące w skład macierzy są wymienione poniżej z wcięciem, w osobnych wierszach. Przyciski w wierszu macierzy: **oko**(ukrywa/wyświetla WSZYSTKIE elementy jednocześnie),**koło zębate**(okienko ustawień macierzy),**X**(odłącza całą matrycę).

Stan czujnika światła (DLS) używany w wierszach matrycy oraz w panelu ustawień matrycy ma cztery stany:**wyłączony**,**oczekiwanie**(brak widma),**aktywny**(widmo nadeszło w ciągu ostatnich 3 s), oraz**nieaktualny** — brak nowego widma przez 3 s, ale ostatni odczyt jest *nadal używany* (odczyty DAQ nigdy nie tracą ważności na ścieżce przechwytywania).

Możesz przeciągać pojedyncze kamery i całe grupy układów obok siebie na pasku bocznym, aby zmienić kolejność listy; elementy układu nie mogą być przeciągane niezależnie.

## Panel ustawień poszczególnych kamer

Otwieraj za pomocą ikony **koła zębatego** w wierszu kamery. Panel przesuwa się nad listą kamer.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Nagłówek**:**próbka koloru**kamery (kliknij, aby otworzyć natywny selektor kolorów — ustawia kolor obramowania paska bocznego i pierścienia wyboru kafelków),**nazwa**z przyciskiem ołówka**Zmień nazwę**(zapisanie pustej nazwy powoduje przywrócenie domyślnej nazwy `MODEL (serial)`), oraz**×** do zamknięcia.

### Histogram na żywo

W górnej części panelu znajduje się histogram luminancji na żywo obliczany na podstawie podglądu JPEG z częstotliwością ~8 Hz. Średnia jest ważona według schematu Bayera — (R+2G+B)/4 — aby odpowiadała pomiarowi AE właściwemu dla danej kamery.

* **Linia przerywana Orange**= cel AE.**Przeciągnij ją w poziomie, aby zmienić cel** — po zwolnieniu myszy wysyłane jest jedno polecenie, a przeciągnięcie powoduje przełączenie trybu celu AE na ręczny.
* **Green linia ciągła** = rzeczywista średnia luminancja (wartość aktualnie dostarczana przez AE).
* **Przycisk RGB** (w prawym górnym rogu): przełącza nakładkowe histogramy dla poszczególnych pasm, pokolorowane zgodnie z filtrem aparatu (np. w trybie FRGN: szary NIR, zielony, czerwony). W aparatach mono (M3M) przycisk ma napis „MONO” i jest wyłączony — tryb mono zawsze wyświetla histogram luminancji dla pojedynczego pasma.
* Etykiety osi X są zgodne z głębią bitową czujnika dla bieżącego formatu pikseli: 0..255, 0..1023, 0..4095 lub 0..65535.

### Wiersze

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

z informacjami o kamerze | Wiersz | Zachowanie |
| --- | --- |
| **Model** | Tylko do odczytu (np. `LATT-M3C-L87-FRGN`). |
| **Kalibracja radiometryczna**| Green**Oznaczenie „Aktywna”**wraz z podpisem wskazującym poziom kalibracji, skrót, datę kalibracji oraz listę pasm, pobrane z pakietu kalibracyjnego kamery (patrz [Fabryczna kalibracja radiometryczna](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Ukryte w przypadku kamer RGB** — posiadają one fotometryczną kalibrację balansu bieli, a nie promieniowanie dla poszczególnych pasm. |
| **Raport kalibracji**|**Pobierz** — otwiera plik PDF z certyfikatem kalibracji NIST dla danego numeru seryjnego kamery w przeglądarce systemu operacyjnego. Jeśli certyfikat nie został jeszcze zapisany w pamięci podręcznej, Chloros wyświetla zamiast tego podpowiedź. |
| **Numer seryjny** | Tylko do odczytu. |
| **Oprogramowanie sprzętowe**| Wyświetla aktualną wersję, a następnie wyszukuje dostępną wersję dla tego modelu (dane buforowane dla poszczególnych modeli — układ składający się z N kamer sprawdza serwer tylko raz). Stany: „Sprawdzanie…” → przycisk**„Zaktualizuj do X”**→ „Aktualizacja…” → „Zaktualizowano z A do B” / „Nie powiodło się: …” / „Pominięto: …” / zielony napis**„Aktualne”**. Podpowiedź przycisku aktualizacji: „Przywrócenie ustawień fabrycznych + flashowanie + przeprogramowanie UserSet1. ~2–3 minuty; nie odłączać.” |
| **IP** | Tylko do odczytu. |
| **Temperatura** | Tylko do odczytu, odświeżana co 3 s. Zmienia kolor na pomarańczowy przy ≥65 °C i na czerwony z symbolem ⚠ przy ≥75 °C. |
| Pole wyboru **Cel kalibracji** | Włącza wykrywanie celu odbicia ArUco za pomocą tabeli walidacyjnej NDVI dla poszczególnych paneli, znajdującej się poniżej podglądu na żywo (widok listy). Działa tylko w ramach sesji — zawsze jest wyłączone. |
| Lista rozwijana **Czujnik światła** | Powiązuje czujnik światła DAQ (DAQ-E/M/U, z listy na zakładce Czujniki światła) z tą kamerą w celu korekcji oświetlenia typu DLS (downwelling-light) oraz predykcyjnej automatycznej ekspozycji. Opcja „Brak” usuwa powiązanie. Jeśli żadne czujniki nie są podłączone, w menu rozwijanym wyświetla się komunikat „(brak podłączonych czujników — otwórz zakładkę DAQ)”. Powiązanie jest zapisywane wraz z projektem. |

### Ekspozycja i wzmocnienie

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Wszystkie pola liczbowe w tym miejscu wykorzystują pokrętła typu „przytrzymaj, aby przyspieszyć”: dotknięcie = ±1, przytrzymanie &gt;1,5 s = ±10, przytrzymanie &gt;3 s = ±100. Wartość jest wysyłana do kamery po zwolnieniu przycisku.

| Sterowanie | Zakres / opcje | Domyślnie | Dotyczy | Jak działa |
| --- | --- | --- | --- | --- |
| **Ekspozycja (us)**| Minimalna/maksymalna wartość na żywo z kamery | Auto | Wszystkie | Czas ekspozycji w mikrosekundach, z przełącznikiem**Auto/Ręcznie**. Auto = ciągła automatyczna ekspozycja po stronie kamery. |
| **Wzmocnienie (dB)**| Minimalna i maksymalna wartość w czasie rzeczywistym kamery (np. do 48 dB) | Ręczne (wyłączone) | Wszystkie | Wzmocnienie analogowe/cyfrowe z własnym przełącznikiem**Auto/Ręczne**. |
| **Docelowa jasność AE**| 0–255 | 80, tryb**Auto**| Wszystkie (możliwość edycji, gdy włączona jest ekspozycja automatyczna lub automatyczne wzmocnienie) | Jasność, do której dąży ekspozycja automatyczna. W trybie**Auto**(domyślnym) kontroler oparty na histogramie sam wybiera tę wartość docelową, utrzymując ekspozycję na poziomie 60–75% maksymalnej wartości czujnika. Wpisanie wartości lub przeciągnięcie pomarańczowej linii histogramu przełącza tryb na**Ręczny**. |
| **Wygładzanie AE** | 0,5–40, krok 0,1 | 8,0 | Wszystkie | Tłumienie AE. Podpowiedź: „Niższa wartość = szybsza reakcja AE (może pulsować przy wysokiej liczbie klatek na sekundę). Wyższa wartość = płynniej / wolniej”. Wartości znacznie poniżej domyślnej mogą powodować pulsowanie AE i destabilizację transmisji przy wysokich częstotliwościach klatek; 8,0 to stabilna wartość domyślna. |
| **Obszar zainteresowania AE**| Pole wyboru Włącz + przycisk**Wyceluj**| Wyłączone | Wszystkie | Gdy opcja jest włączona, AE mierzy tylko zielony obszar zaznaczony linią przerywaną zamiast całej klatki. Przycisk**Wyceluj** uruchamia funkcję „kliknij, aby umieścić” na podglądzie na żywo: kliknięcie wyśrodkowuje obszar na 30 % klatki; kliknięcie i przeciągnięcie tworzy niestandardowy prostokąt (minimum 5 % × 5 %). Funkcja „Aim” wyłącza się po jednym umieszczeniu. Obszar ten jest mapowany z powrotem na natywne współrzędne kamery zgodnie z ustawionym obrotem/odbiciem lustrzanym i jest zapisywany wraz z projektem. |
| **Szybkość dostosowywania AE** | 0,1–5, krok co 0,1 | 1,0 | Tylko dla członków tablicy | Jak szybko cel automatycznego AE śledzi zmiany jasności sceny; przy wartości 1,0× ponowna kontrola odbywa się co 2,5 s. |
| **Ochrona jasnych obszarów** | Ścisła (1 %) / Normalna (5 %) / Luźna (15 %) | Ścisła | Kamery, które udostępniają to ustawienie | Jaka część kadru może ulec prześwietleniu do bieli, zanim automatyczna ekspozycja (AE) przyciemni obraz. |

{% hint style="info" %}
**Wymagania oświetleniowe dla wielospektralnych kamer typu Bayer (RGN / OCN / NGB):** scena musi być wystarczająco oświetlona we wszystkich trzech kanałach, w przeciwnym razie kalibracja nie będzie działać poprawnie — pojedyncza ekspozycja czujnika obejmuje wszystkie trzy spektra. Użyj czujnika światła DAQ do pomiaru oświetlenia lub przełącz się na tryb całkowicie monochromatyczny (M3M), aby każde pasmo miało własną ekspozycję. Jeśli przechwytywanie narusza ten warunek, Chloros wykrywa to i wyświetla ostrzeżenie (powiadomienie o zablokowaniu rozdzielania).
{% endhint %}

### Format pikseli i

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

rozdzielczość**Elementy macierzy** wyświetlają w trybie tylko do odczytu wiersze „Aktualne” (format + WxH) i „Binning” z adnotacją „Ustawione w ustawieniach macierzy” — ponowne uruchomienie strumienia na jednym elemencie zakłóciłoby synchronizację, dlatego są one zarządzane w [panelu ustawień macierzy](#array-settings-pane).**Kamery autonomiczne** mają następujące opcje:

| Sterowanie | Opcje | Działanie |
| --- | --- | --- |
| **Format pikseli** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format pikseli czujnika (głębia bitowa). |
| **Rozdzielczość** | Pełna / Połowa / Jedna czwarta | W odniesieniu do bieżącego binningu: Pełna = 2048/N × 1536/N dla binningu N×N. |
| **Łączenie pikseli** | 1x1 (brak) / 2x2 / 4x4 | Sprzętowe łączenie pikseli N×N — większe wartości obniżają rozdzielczość, ale zwiększają stosunek sygnału do szumu (SNR) i liczbę klatek na sekundę. Zmiana tej wartości powoduje ponowne uruchomienie strumienia i zresetowanie wszelkich obszarów zainteresowania (ROI) do nowego pełnego pola widzenia. |
| **Aktualne** | tylko do odczytu | Rzeczywiste wymiary WxH oraz przesunięcie (x, y) obowiązujące w danym momencie. |

### Podgląd na żywo

Wszystko w tej sekcji dotyczy **wyłącznie strony wyświetlania**— zmienia to, co widzisz w transmisji na żywo, podczas gdy zapisane zrzuty pozostają liniowe i niezmodyfikowane — z jednym wyjątkiem:**Vignette** ma charakter radiometryczny i wpływa również na eksporty (opisano poniżej).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Sterowanie | Zakres / opcje | Domyślne | Dotyczy | Jak działa |
| --- | --- | --- | --- | --- |
| **Rozdzielczość renderowania** | 360p (najszybsza) / 480p / 720p / 1080p / Natywna rozdzielczość czujnika (najwolniejsza) | 720p | Wszystkie | Rozdzielczość, przy której zaplecze uruchamia łańcuch podglądu radiometrycznego. Niższa wartość zapewnia wyższą liczbę klatek na sekundę bez zmiany pola widzenia. |
| **Indeks**| Pole wyboru „Włącz” + ikona koła zębatego | Wyłączone | Tylko kamery wielospektralne typu Bayer,**nie** dotyczy kamer z matrycą kombinowaną | Podgląd indeksu wegetacji na żywo. Ikona koła zębatego otwiera wspólny [Kalkulator indeksu](#index-calculator-pane) z wstępnie załadowanymi naturalnymi pasmami filtrów kamery (np. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Wyrażenie niestandardowe wraz z tabelą LUT (włączone/wyłączone, poziom domyślny 3, min. domyślny 0,2, maks. domyślny 1) jest obliczane dla każdej klatki podglądu. Członkowie macierzy połączonej ukrywają ten wiersz — macierz posiada jeden wspólny wskaźnik. |
| **Balans bieli** | Wył. / Jednorazowo / Ciągle + przycisk ponownego pomiaru | Ciągle | Tylko RGB | Balans bieli na żywo. Przycisk odświeżania powoduje ponowny pomiar balansu bieli na podstawie bieżącego widma DLS (wyłączony, gdy tryb jest wyłączony). |
| **Gamma** | Włączone / Wyłączone | Włączone | Tylko RGB | Wyświetlanie gamma (γ = 2,2 LUT) w podglądzie na żywo. Zapisane ujęcia pozostają liniowe. |
| **Redukcja szumów** | Pole wyboru + siła 0–100 | Wył. / 50 | Wszystkie (dla każdej kamery, nawet w matrycach) | Filtr dwustronny w podglądzie na żywo. Wyższa wartość = gładszy, ale mniej wyrazisty obraz. |
| **Ostrość** | Pole wyboru + siła 0–100 | Wył. / 30 | Wszystkie | Maska wyostrzająca w podglądzie na żywo, stosowana jako ostatnia. Może wzmacniać szum. Tylko w podglądzie. |
| **Winietowanie**| Pole wyboru + siła 0–100 | Wyłączone / 0 | Wszystkie | Ręczne usuwanie resztkowego winietowania (rozjaśnia rogi), nakładane na szacunkową wartość inteligentnego winietowania matrycy.**Radiometryczne — wpływa na podgląd na żywo ORAZ eksport**, w przeciwieństwie do opcji „Redukcja szumu” i „Ostrość”. |
| **Profil kolorów** | Raw / Liniowy / Naturalny / Ulepszony / Niestandardowa temperatura | Naturalny | Tylko RGB | Zobacz poniżej. |
| **Temperatura barwowa** | 2000–10000 K, krok co 100 | 5500 K | Tylko RGB, profil „Temperatura niestandardowa” | Ustawia balans bieli na stałą skorelowaną temperaturę barwową (dane wejściowe DLS są ignorowane). Ostatnio wybrana wartość w kelwinach jest zapamiętywana przy zmianie profilu. |
| **Nasycenie** | 0–200 (100 = neutralne) | 100 | Tylko RGB | Nasycenie HSV w podglądzie na żywo. |
| **Kontrast** | 0–200 (100 = neutralny) | 100 | Tylko RGB | Kontrast liniowy wokół średniej szarości w podglądzie na żywo. |
| **Odbicie lustrzane w poziomie / Odbicie lustrzane w pionie** | Pola wyboru | Wyłączone | Wszystkie | Odbij lustrzanie podgląd w poziomie / w pionie. |
| **Obrót**| 0° / 90° / 180° / 270° | 0° | Wszystkie | Obróć podgląd. Orientacja jest stosowana na końcu łańcucha podglądu zaplecza —**zapisane zdjęcia zachowują orientację natywną dla aparatu**, a widoki kompozytowe tablicowe ją ignorują. |**Semantyka profili kolorów** (aparaty RGB):

* **Raw** — całkowite pominięcie łańcucha przetwarzania.
* **Liniowy** — sygnał ciemny + pole płaskie + balans bieli; bez macierzy kolorów, bez gamma.
* **Naturalny** *(domyślny)* — tryb liniowy z dodaną zmierzoną macierzą korekcji kolorów oraz krzywą tonową dostosowującą się do sceny.
* **Wzbogacony**— tryb Naturalny z dodatkową intensywnością barw i lokalnym kontrastem CLAHE. Dodatkowa opłata dotyczy**wyłącznie podglądu na żywo** — zapisane zdjęcia zawsze uzyskują pełne wykończenie niezależnie od profilu.
* **Temperatura niestandardowa** — tryb Natural z balansem bieli ustalonym na wybraną wartość w kelwinach.

{% hint style="warning" %}
W przypadku trybów Natural, Ulepszony i Temperatura niestandardowa w panelu wyświetlana jest uwaga dotycząca tonacji: klatki są rozjaśniane w zależności od danej sceny, więc zapisane *obrazy wyświetlane* nie są porównywalne między poszczególnymi klatkami. **Eksportuj promieniowanie lub współczynnik odbicia w celu przeprowadzenia pomiarów.**
{% endhint %}

### Nakładki wyświetlacza (nakładane na obraz na żywo)

Są one dostępne wyłącznie w interfejsie użytkownika — nakładane na obraz wideo, nie mają wpływu na strumień ani na zrzuty ekranu.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Nakładka | Elementy sterujące | Domyślnie | Działanie |
| --- | --- | --- | --- |
| **Zebra** | Pole wyboru + próg 200–255 | Wył. / 250 | Magentowe ukośne paski na przyciętych pikselach. |
| ****Krzyżyk** | Pole wyboru | Wyłączone | Znak środka klatki. |
| **Siatka** | Wyłączona / 3 × 3 / 9 × 9 | Wyłączona | Siatka kompozycyjna. |
| **Histogram** | Pole wyboru + szerokość 0,10–0,90 kadru | Wyłączone / 0,25 | Pasek histogramu wyświetlany na bieżąco. |
| **Wyróżnienie ostrości** | Pole wyboru + próg 20–200 + próbka koloru | Wył. / 80 / `#ff5722` | Podświetlenie krawędzi metodą Sobela do ustawiania ostrości. |
| **Podział kanałów** | „Pokaż podziały (Red / Green / NIR)” / Przycisk „Ukryj podziały” | Ukryte | Dodaje trzy niezależne kafelki w skali szarości dla każdego kanału obok obrazu złożonego (nazwa przycisku odpowiada kanałom filtrów kamery). Każdy podzielony panel można przeciągać i ma on ten sam kolor obramowania co kamera. Niedostępne w kamerach monochromatycznych. Zapisywane wraz z projektem. |

### Miernik punktowy

* Pole wyboru **Kliknij, aby pobrać próbkę**: kliknij obraz na żywo, aby pobrać próbkę pojedynczego piksela (zostanie on zaznaczony krzyżykiem), lub kliknij i przeciągnij, aby zaznaczyć obszar i uzyskać średnią pikselową. Opcja**Wyczyść**usuwa próbkę i krzyżyk. Funkcja ta wyklucza się wzajemnie z trybem**Aim** w AE-ROI.
* Menu rozwijane **Pokaż**:**Raw (głębokość bitowa)**— natywne wartości cyfrowe z głębokością bitową czujnika (np. 12-bitowe → 0..4095) — lub**Wyświetlacz (8-bitowe)** (domyślnie). Gdy aktywny jest wskaźnik na żywo, wyświetlacz pokazuje zamiast tego obliczoną wartość indeksu (np. NDVI).
* Panel odczytu zawiera współrzędne pikseli, rozmiar klatki, format pikseli, głębię bitową oraz tabelę kanałów (Chan / Wartość / %) z etykietami pasm i długościami fal; pary zielonych pikseli w układzie Bayera są uśredniane; próbki z regionu wyświetlają „N px avg”.

Stan miernika punktowego jest dostępny tylko w ramach sesji.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Predykcyjna automatyczna ekspozycja (sterowana przez DLS)

Ta sekcja pojawia się tylko wtedy, gdy **podłączony jest co najmniej jeden czujnik światła DAQ** — solwer potrzebuje aktualnego widma promieniowania padającego, aby ją sterować.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Sterowanie | Zakres | Domyślnie | Działanie |
| --- | --- | --- | --- |
| **Włącz** | Pole wyboru | Włączone (kamery autonomiczne) | Solwer zamkniętej postaci wykorzystuje widmo DLS oraz skalarne wartości z pakietu kalibracyjnego kamery, aby ustawić najjaśniejsze pasmo w pobliżu nasycenia, jednocześnie utrzymując najciemniejsze pasmo powyżej minimalnego poziomu SNR — jedno zapisanie ekspozycji na każde rozwiązanie, bez pętli ustabilizowania. Przeznaczony do timelapse&#x27;ów zasilanych energią słoneczną, gdzie każde ujęcie musi być poprawnie naświetlone. Moduł zaplecza dyskretnie przełącza się na reaktywną ekspozycję automatyczną (AE), gdy odczyt DLS jest nieaktualny/brakuje go lub pakiet kalibracyjny nie został załadowany. |
| **Wygładzanie (α)** | 0,05–1,0, krok 0,05 | 0,3 | Wygładzanie kolejnych rozwiązań predykcyjnych (im niższa wartość, tym większe wygładzenie). |
| **Odbicie sceny**| Przycisk**Rekalibruj ρ** | — | Ponownie szacuje współczynnik odbicia sceny używany przez solwer. |

{% hint style="info" %}
**W przypadku układów matrycowych funkcja „Array connect” domyślnie wyłącza predykcyjną ekspozycję automatyczną** — w przypadku układów matrycowych ekspozycję obsługują inteligentna ekspozycja automatyczna Chloros oraz automatyczna ekspozycja po stronie aparatu (z ochroną przed nasyceniem), a oszacowanie odbicia pojedynczej scenynie jest bezpieczne w scenach mieszanych. Można ją ponownie włączyć dla poszczególnych kamer w tym miejscu, jeśli zależy nam konkretnie na ekspozycji radiometrycznej sterowanej przez DLS.
{% endhint %}

**Górna granica ekspozycji sterowana przez DAQ i AE z przypięciem do światła padającego.**Niezależnie od powyższego pola wyboru, gdy czujnik światła DAQ jest przypisany do kamery RGB, funkcja Chloros oblicza — na podstawie zmierzonego bezwzględnego natężenia promieniowania padającego — maksymalną ekspozycję × wzmocnienie, przy której powierzchnia o 100%powierzchnia o zerowym współczynniku odbicia pozostaje poniżej poziomu przesterowania, i stosuje ją jako**ograniczenie**w automatycznej ekspozycji. Gdy ograniczenie jest aktywne, kamera działa w trybie**przywiązania do natężenia światła padającego**: działa w trybie pętli otwartej przy ekspozycji mierzonej na świetle padającym i wzmocnieniu na poziomie 0 dB — ekspozycja podąża za zmierzonym światłem, a nie za treścią sceny. Ponieważ limit ten może jedynie skrócić czas naświetlania, sam w sobie nie może spowodować przesterowania. Limit wyłącza się automatycznie — i przywraca się normalną automatyczną ekspozycję sceny — zawsze, gdy odczyt z systemu DAQ jest nieobecny, nieaktualny (&gt;30 s) lub ciemny, albo jeśli ≥15 % klatek ulega przesterowaniu przy przypisanej ekspozycji (co oznacza, że czujnik i kamera rejestrują różne natężenie oświetlenia). Nie ma przełącznika w interfejsie graficznym; jest to standardowe zachowanie zawsze, gdy kamera RGB ma powiązany moduł DAQ.

### Członkowie macierzy akwizycji i wyzwalania

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

wyświetlają dodatkowo wiersze tylko do odczytu: **Rola**(Master na niebiesko / Slave na zielono),**Linia synchronizacji**oraz**Rówieśnicy**.

| Sterowanie | Opcje | Domyślnie | Uwagi |
| --- | --- | --- | --- |
| **Tryb wyzwalania** | Wył. / Wł. | Wł. | Wyłączony dla elementów macierzy (wyzwalanie zarządza macierz). |
| **Źródło wyzwalacza** | Oprogramowanie / Linia 0 (M8) / Linia 1 / Linia 2 | Linia 0 | Ukryte, gdy tryb wyzwalania jest wyłączony; wyłączone dla elementów macierzy. Linia 0 to optoizolowane zewnętrzne wejście wyzwalacza M8. |
| **Liczba klatek na sekundę**| Auto / Ręcznie + wartość | Auto |**Auto**: ograniczenie liczby klatek na sekundę kamery jest wyłączone — ekspozycja określa liczbę klatek na sekundę, a w polu wyświetlana jest aktualna liczba klatek na żywo.**Ręcznie**: można ograniczyć liczbę klatek na sekundę za pomocą suwaka (od 1 do maksymalnej wartości ograniczonej przepustowości), przy czym wartość początkowa jest oparta na aktualnej rzeczywistej częstotliwości. Użytkownicy macierzy widzą tylko do odczytu wartość „N klatek na sekundę (na żywo)” z adnotacją „Ustawiono w ustawieniach macierzy”. |

### Sieć / Transport

| Wiersz | Zachowanie |
| --- | --- |
| **Rozmiar pakietu**| 1500 (standardowy) / 9000 (jumbo) — domyślnie**jumbo**. |
| **Przepustowość** | Limit przepustowości łącza tylko do odczytu w MB/s. Moduł zaplecza rozkłada tę wartość równomiernie na wszystkie podłączone kamery przy każdym podłączeniu/odłączeniu. |
| **Obsługa bufora** | Tryb obsługi bufora tylko do odczytu. |

### Przechwytywanie

Panel kończy się przyciskiem **„Otwórz ustawienia przechwytywania…”**, który przenosi do [panelu ustawień przechwytywania](capture.md#the-capture-settings-pane) (nieaktywnego, dopóki nie zostanie otwarty projekt — „Utwórz lub otwórz projekt, aby zapisać przechwycone dane”). Jeśli kamera jest ukryta lub wstrzymana, pojawia się podpowiedź przypominająca o konieczności jej odkrycia/wznowienia przed rozpoczęciem przechwytywania.

## Panel ustawień macierzy

Otwiera się za pomocą ikony **koła zębatego**w wierszu ARRAY. Nagłówek: nazwa macierzy z ołówkiem do zmiany nazwy oraz ikoną**×** do zamknięcia. Sekcje poniżej oznaczone jako *tylko połączone* pojawiają się wyłącznie w przypadku układów połączonych w trybie wyświetlania połączonego.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synchronizacja

Wiersze **Master**,**Slaves**i**Sync Line** są tylko do odczytu.

### Czujnik światła otoczenia

Wyświetlane zarówno dla układów połączonych, jak i oddzielnych:

* ****Cel kalibracji** — „Wykryj cel ArUco MAPIR i zweryfikuj NDVI względem tabeli LUT odbicia panelu”; steruje nakładką celu i tabelą weryfikacyjną dla połączonej płytki.
* Lista rozwijana **Czujnik światła** — przypisuje jeden moduł DAQ do całej matrycy. Wybór zostaje natychmiast zachowany, przenosi się do listy rozwijanej „Czujnik światła” każdej kamery wchodzącej w skład matrycy (nadal można nadpisać ustawienia dla poszczególnych kamer) i rozpoczyna przekazywanie widm do matrycy.
* Wiersz **Statusu** na żywo: Wyłączony · „Oczekiwanie na pierwsze widmo…” · „Aktywne — wszystkie kamery w macierzy mają skorygowane oświetlenie” · „Brak nowego widma w ciągu ostatnich 3 s — nadal używany jest ostatni odczyt (brak limitu czasu dla nieaktualnych danych)…”.
* Uwaga w panelu: „Korekcja radiometryczna dla całej macierzy. Ustawienia poszczególnych kamer mają pierwszeństwo”.

### Przechwytywanie — jednolite ustawienia czujników *(tylko połączone)*

Ustawienia te mają jednolite zastosowanie do wszystkich elementów macierzy (zmiany dla poszczególnych elementów zakłóciłyby synchronizację). Zmiany są buforowane i stosowane łącznie.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Element sterujący | Opcje / zakres | Działanie |
| --- | --- | --- |
| **Format pikseli** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Jednolity format czujnika dla wszystkich elementów. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning sprzętowy — pozwala zachować pełne pole widzenia przy jednoczesnym zwiększeniu stosunku sygnału do szumu (SNR) i liczby klatek na sekundę. Zmiana tej opcji resetuje pola ROI do nowego pełnego pola widzenia. |
| Ustawienie wstępne **rozdzielczości** | Pełna / Połowa / Jedna czwarta | W odniesieniu do binningu; wypełnia pola ROI wycinkiem wyśrodkowanym. |
| **Wycinek ROI (px)**| Pola liczbowe W / H / X / Y | Kadrowanie matrycy. Szerokość/wysokość przyciągają się do wielokrotności 16 (minimum 64); przesunięcia przyciągają się do wielokrotności 4. Wskazówka „max WxH” pokazuje górną granicę, a opcja**Reset** przywraca pełne pole widzenia. Podczas edycji na kafelku matrycy rysowane jest pomarańczowe okienko podglądu kadrowania na żywo (w tym schemat całego czujnika przy rozszerzaniu kadru na zewnątrz). |
| **Częstotliwość wyzwalania**| Przełącznik Auto / Ręcznie + fps 0,5–10, krok 0,5 |**Auto**(domyślnie): moduł zaplecza oblicza częstotliwość wyzwalania na podstawie rozdzielczości i przepustowości — pole wprowadzania jest wyłączone i wyświetla obliczoną wartość.**Ręcznie**: ustawia wybraną wartość po kliknięciu przycisku Zastosuj. |

Uwaga w panelu: „Zmiany formatu/rozdzielczości powodują krótkotrwałe ponowne uruchomienie wszystkich kamer. Częstotliwość wyzwalania ma zastosowanie na żywo”. Przyciski **Zastosuj / Anuluj** znajdują się na dole panelu.

### Wyrównanie (współrejestracja) *(tylko w trybie łączonym)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Pole **Odchylenie**: „RMS x,xx px” — zielone dla wartości poniżej 1 px, bursztynowe dla wartości poniżej 3 px, czerwone w pozostałych przypadkach lub jeśli jakakolwiek kamera zawiodła; „brak profilu” przed pierwszym obliczeniem.
* Wiersz podsumowujący: „✓ Wszystkie kamery wyrównane (N)” / „⚠ p/N kamer wyrównanych —  <serial (filter)="">niepowodzenie” / „Aktywne kadrowanie — ponowna kalibracja w celu wyrównania (wykorzystuje cały czujnik)” / „Oczekiwanie na ustabilizowanie się ekspozycji…”.
* Tabela dla poszczególnych kamer: kamera (ostatnie 4 cyfry numeru seryjnego + filtr), błąd reprojekcji w pikselach wraz z liczbą dopasowań („ref” dla kamery głównej) oraz wynik znormalizowanej korelacji krzyżowej nakładania się w stosunku do minimalnego progu 0,35.
* Przycisk **„Ponowna kalibracja wyrównania”** (przed pierwszym profilem wyświetla się „Kalibruj wyrównanie”) — ponownie uruchamia współrejestrację na nowych klatkach.
* Pole wyboru **„Automatyczna ekspozycja kamer do wyrównania”** (domyślnie zaznaczone) — tymczasowo rozjaśnia ciemne lub płaskie kamery (najpierw ekspozycja, potem wzmocnienie), aby uzyskały teksturę do dopasowania, a następnie przywraca automatyczną ekspozycję (AE).

Połączony podgląd wyrównuje się automatycznie po otwarciu; należy ponownie skalibrować, jeśli zmieniła się ostrość lub głębia sceny. Wyrównanie jest **z założenia ograniczone do danej sesji** — nigdy nie jest zapisywane w profilu, ponieważ zależy od aktualnej odległości sceny. Ujęcia nadal można eksportować z rejestracją pikselową (zobacz [Eksporty z wyrównaniem](capture.md#per-array-controls)).

### Inteligentna winieta

* Pole wyboru **Włącz korekcję**— stosuje oszacowanie winietowania dla poszczególnych kamer do łańcucha radiometrycznego (na żywo**i** w eksportach).
* **Kalibruj na podstawie bieżącego widoku**— najpierw skieruj układ kamer na jednolity cel (płaski panel, ścianę lub niebo); każda kamera jest indywidualnie wyrównywana, a status wskazuje poprawę płaskości w formacie „n/N kamer · −x,x %”. Opcja**Wyczyść** usuwa oszacowanie.
* Dokonaj precyzyjnej regulacji dla każdej kamery za pomocą suwaka **Winietowanie** w [Podglądzie na żywo](#live-preview).

### Podgląd na żywo *(tylko połączony)** **Indeks**: zaznacz pole wyboru + ikona koła zębatego — otwiera wspólny [Kalkulator indeksu](#index-calculator-pane) z pasmami wygenerowanymi na podstawie danych z**wszystkich** kamer w układzie. W linii podglądu wyrażenia poniżej wyświetlane jest bieżące wyrażenie („Nie ustawiono wyrażenia — otwórz kalkulator, aby je utworzyć”), odświeżane co sekundę.
* Menu rozwijane **Rozdzielczość renderowania**(te same ustawienia wstępne co dla poszczególnych kamer, domyślnie 720p): wysokość strumienia podglądu na żywo**oraz** rozmiar zapisanego eksportu kompozycji. Uwaga w panelu: „Podgląd + rozmiar zapisanej kompozycji. Obrazy z poszczególnych kamer są zawsze eksportowane w pełnej rozdzielczości”.

### Warstwy wyświetlania *(tylko połączone)** Pole wyboru **Włącz** (domyślnie wyłączone — kamera główna jest wyświetlana bezpośrednio; włączone = kompozycja warstwowa).
* Listy rozwijane **Pierwszy plan**/**Tło**: każda kamera członkowska (według nazwy) lub**Indeks**. Gdy opcja „Pierwszy plan” ma wartość „Indeks”, piksele poza zakresem minimalnym/maksymalnym LUT wyświetlają warstwę „Tło”.

### Widok podzielony *(tylko w trybie połączonym)*

**„Pokaż kamery składowe”**— przycisk**Podziel / Ukryj kamery składowe**, który dodaje obraz na żywo z każdej kamery składowej jako oddzielne kafelki siatki obok obrazu złożonego. Kafelki odczytują istniejący bufor klatek macierzy (bez dodatkowego podłączenia kamer). Tylko widok siatki; zapisywane dla każdej macierzy wraz z projektem.

### Możliwości

Panel tylko do odczytu odświeżany co 5 s:

* **Etykieta poziomu**: „Jednoczesne przechwytywanie” (zielony) · „Jednoczesne przechwytywanie (emitowanie rozłożone w czasie FTD)” (zielony) · „Przechwytywanie rozłożone w czasie (przesunięcie 100 ms)” (pomarańczowy) · „Konfiguracja zbyt duża” (czerwony).
* **Stan klatek**: „x,xx % niekompletne” — zielony poniżej 1 %, bursztynowy poniżej 5 %, czerwony przy 5 % lub więcej.
* **Linia łącza**: „NIC {mbps} Mbps – stała {MB/s} MB/s”.

Jest to aktualny budżet przepustowości macierzy. Aby zapoznać się z podstawowymi parametrami klatek na sekundę i modelem sieciowym — oraz dowiedzieć się, co należy zmienić, gdy poziom przechodzi na pomarańczowy lub czerwony — zobacz [Macierze wielokamerowe](arrays.md) oraz [Podręcznik CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panel „Kalkulator indeksu”

Trzecia strona paska bocznego, wspólna dla narzędzia „Indeks” dla poszczególnych kamer oraz narzędzia „Indeks” dla macierzy połączonej (po jednym naraz — nagłówek brzmi „Kalkulator indeksu — <camera name="">” lub „Kalkulator indeksu —<array name="">

”). Otrzymuje on listę pasm (naturalne pasma filtra kamery lub wszystkie pasma z członków macierzy), bieżące wyrażenie oraz konfigurację LUT (włączone/wyłączone, poziom — domyślnie 3, min. — domyślnie 0,2, maks. — domyślnie 1), a także histogram indeksu na żywo. **Zastosuj** zatwierdza wyrażenie; zmiany LUT są stosowane na bieżąco w podglądzie.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Ustawienia poszczególnych kamer a ustawienia zarządzane przez macierz

Krótki przewodnik po tym, co znajduje się gdzie, gdy kamera jest członkiem macierzy:

| Zarządzane przez macierz (tylko do odczytu w panelu kamery) | Nadal dla poszczególnych kamer w ramach macierzy |
| --- | --- |
| Format pikseli, rozdzielczość, binning | Automatyczna ekspozycja (ekspozycja, wzmocnienie, wartość docelowa, wygładzanie, obszar zainteresowania) |
| Tryb/źródło wyzwalania, liczba klatek na sekundę | Redukcja szumów, ostrość, winietowanie |
| | Orientacja (odbicie lustrzane/obrót), nakładki wyświetlacza, pomiar punktowy |
| | Indeks (macierze z oddzielnym wyświetlaczem), powiązanie z czujnikiem światła |

Inne zachowania o charakterze przekrojowym:

* **Wyświetlanie połączone vs oddzielne** wybiera się podczas podłączania macierzy: połączone = jedna wyrównana kompozytowa płytka (kamery członkowskie przesyłają obraz wyłącznie poprzez Split View); oddzielne = każda kamera renderuje własną zsynchronizowaną płytkę. Kamera nigdy nie wyświetla jednocześnie samodzielnego obrazu i płytki macierzy.
* **Automatyczne ponowne połączenie**: otwarcie zapisanego projektu przywraca kamery i macierze oraz ponownie stosuje wszystkie zapisane ustawienia do zaplecza przed wznowieniem strumieni.
* **Filtrowanie podczas przechwytywania**: ukryte lub wstrzymane kamery są wykluczane z funkcji „Capture All”; macierz jest całkowicie zablokowana tylko wtedy, gdy WSZYSTKIE elementy są ukryte/wstrzymane. Zobacz [Ustawienia i tryby przechwytywania](capture.md).

## Jak zachowywane są ustawienia

Stan karty „Kamery” jest zapisywany **wraz z projektem**, a nie w przeglądarce:

* Każda zmiana powoduje wykonanie migawki kamer i macierzy w pliku projektu `cameras.json` (z opóźnieniem 500 ms). Obejmuje to nazwy i kolory kamer, ustawienia ekspozycji/wzmocnienia/AE, format pikseli/rozdzielczość/binning, częstotliwość wyzwalania, ustawienia podglądu (rozdzielczość renderowania, redukcja szumów, ostrość, winietowanie, profil kolorów, nasycenie/kontrast), orientację, nakładki, podział kanałów, konfigurację indeksu, ustawienia predykcyjnej automatyki ekspozycji (Predictive-AE), obszar zainteresowania (ROI) dla automatyki ekspozycji, nazwy matryc, tryb wyświetlania, ustawienia przechwytywania matryc (w tym pozycję kadrowania obszaru zainteresowania) oraz blok siatki (powiększenie strumienia, tryb widoku, blokada siatki, ręczna kolejność kafelków, ukryte kamery, zamknięte kafelki, aktywna kamera).
* Powiązania czujników światła są zapisywane w pliku projektu `sensors.json`.
* Ponowne otwarcie projektu powoduje ponowne podłączenie sprzętu i ponowne zastosowanie wszystkich tych ustawień.
* **Brak otwartego projektu = tylko sesja**: bez projektu żadne ustawienia nie zostaną zachowane po zamknięciu programu Chloros.
* Tylko sesja, niezależnie od projektu: stan wstrzymania, próbki pomiaru punktowego, pole wyboru „Cel kalibracji” dla poszczególnych kamer (zawsze domyślnie wyłączone) oraz profil wyrównania matrycy (zgodnie z założeniami przeliczany dla każdej sesji).
* Jedyny wyjątek: wybrane opcje eksportu w **Ustawieniach przechwytywania** oraz tryb przechwytywania są zachowywane dla każdego projektu w lokalnej pamięci aplikacji, a nie w `cameras.json` — zobacz [Ustawienia i tryby przechwytywania](capture.md).</array></camera></serial>
