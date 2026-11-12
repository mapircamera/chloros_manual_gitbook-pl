# Ustawienia kamer

Zakładka **Kamery** to panel sterowania na żywo programu Chloros przeznaczony dla kamer LATTICE: główny obszar podglądu, na którym każda podłączona kamera jest wyświetlana jako kafelek na żywo, oraz pasek boczny, który umożliwia przełączanie się między trzema stronami — **listą kamer**,**panelem ustawień**(ustawienia poszczególnych kamer, macierzy lub przechwytywania — po jednym na raz) oraz**Kalkulator indeksu**. Ta strona opisuje wszystkie elementy sterujące na liście kamer, w panelu ustawień poszczególnych kamer oraz w panelu ustawień macierzy. Tryby przechwytywania, wybór typu eksportu oraz sekwencja „Capture All” znajdują się na stronie towarzyszącej [Ustawienia i tryby przechwytywania](capture.md).

Zakładka „Kamery” pojawia się na pasku bocznym, gdy backend Chloros jest gotowy. Wszystkie poniższe elementy sterujące komunikują się z lokalnym backendem poprzez `127.0.0.1:5000`; zmiany są natychmiast stosowane do obrazu na żywo z kamery, o ile nie zaznaczono inaczej.

## Typy kamer używane na tej stronie

Elementy sterujące są wyświetlane lub ukrywane w zależności od wybranego typu kamery. W niniejszej instrukcji stosowane są następujące terminy:

| Termin | Znaczenie | Kanały filtra |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C z filtrem FRGB (model zawiera `-FRGB`) | Red / Green / Blue |
| **Wielospektralna kamera typu Bayera** | LATTICE M3C z filtrem FRGN, FOCN lub FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — jeden filtr wąskopasmowy, jedno skalibrowane pasmo | Pojedyncze pasmo |
| **Element macierzy** | Kamera podłączona jako część zsynchronizowanej macierzy (wyświetlanie połączone lub oddzielne) | Zgodnie z filtrem |

Kamery RGB przechodzą przetwarzanie fotometryczne (balans bieli, profile kolorów, gamma); kamery wielospektralne i monospektralne przechodzą łańcuch radiometryczny i pomijają ustawienia fotometryczne. Elementy macierzy przekazują ustawienia na poziomie strumienia (format pikseli, rozdzielczość, binning, wyzwalacz, częstotliwość klatek) do macierzy — wiersze te stają się tylko do odczytu w panelu poszczególnych kamer i są przenoszone do panelu ustawień macierzy.

## Główny obszar podglądu



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Gdy żadna kamera nie jest podłączona, w obszarze podglądu wyświetla się ekran powitalny **„Podłącz kamerę, aby rozpocząć”**z dwoma przyciskami:**Podłącz kamerę**(zielony, otwiera okno dialogowe podłączania pojedynczej kamery) oraz**Podłącz macierz** (niebieski, otwiera okno dialogowe podłączania macierzy). Same okna dialogowe podłączania zostały opisane w sekcji [Podłączanie kamer](connecting.md); pojęcia związane z macierzą (synchronizacja, poziomy, przepustowość) w sekcji [Macierze wielokamerowe](arrays.md). Po otwarciu zapisanego projektu zawierającego kamery na ekranie powitalnym wyświetla się ikona ładowania z napisem „Ponowne otwieranie N zapisanych kamer…”, podczas gdy Chloros przywraca strumienie z ostatniej sesji.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Górny pasek

| Element sterujący | Funkcja |
| --- | --- |
| **Przełączanie trybu widoku**| Przełącza między**widokiem siatki**(wszystkie kafelki jako komórki) a**widokiem listy**(wiersze o pełnej szerokości: każdy połączony układ jako jeden złożony wiersz, każdy element**oddzielnego** układu jako osobny wiersz z własnym strumieniem i tabelą celów kalibracyjnych, a poniżej JEDNA samodzielna kamera — ta aktywna). Podpowiedzi: „Przełącz na widok siatki” / „Przełącz na widok listy”. |
| **Blokada siatki**(kłódka) | Domyślnie**zablokowana** — kafelki unieruchomione w miejscu. Odblokuj, aby przeciągać i zmieniać kolejność kafelków w dowolnym miejscu (luki pozostają zachowane). Siatka automatycznie ponownie się blokuje za każdym razem, gdy podłącza się nowa kamera. Podpowiedzi: „Odblokuj siatkę (włącz przeciąganie kafelków)” / „Zablokuj siatkę (unieruchom kafelki)”. |
| Suwak **Powiększenie strumienia** | Rozmiar kafelka, od 60 pikseli do pełnej szerokości kontenera. Komórki zachowują proporcje 4:3. Przy szerokości komórki poniżej 200 px nakładki z nazwą i liczbą klatek na sekundę są ukrywane, aby kafelek wyglądał przejrzyście. |

### Kafelki strumienia

Każda kamera renderuje złożony kafelek na żywo; kamera może dodatkowo wyświetlać trzy kafelki w skali szarości z **podziałem na kanały** (zobacz [Podział na kanały](#display-overlays-drawn-over-the-live-feed)), a macierze renderują połączoną kafelkę. Aktywna kafelka posiada pierścień zaznaczenia w kolorze kamery (lub macierzy).

Po najechaniu kursorem na kafelkę pojawia się przycisk zamknięcia **X**:

* Zamknięcie **kompozytowej** kafelki przy nadal wyświetlanych podziałach kanałów powoduje jedynie ukrycie obrazu kompozytowego.
* Zamknięcie **ostatniej widocznej kafelki samodzielnej kamery** powoduje odłączenie tej kamery.
* **Kafelki z podziałami należące do połączonego układu nigdy nie powodują odłączenia** kamery — są jedynie ukrywane.

Gdy siatka jest odblokowana, przeciągnij dowolną płytkę do dowolnego miejsca; układ zostanie zapisany wraz z projektem.

## Pasek boczny — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamer Pierwsza strona paska bocznego zawiera listę wszystkich podłączonych kamer i macierzy:

* **Połącz kamerę**(zielony) /**Połącz układ** (niebieski, podczas skanowania wyświetla komunikat „Wykrywanie...”). Oba przyciski są nieaktywne, gdy otwarte jest okno dialogowe połączenia.
* **Zrób wszystkie zdjęcia** (czerwony) — wykonuje zdjęcia ze wszystkich wymienionych kamer z typami eksportu wybranymi w ustawieniach przechwytywania. Wymaga otwartego projektu. Pełna dokumentacja znajduje się w sekcji [Ustawienia i tryby przechwytywania](capture.md).
* **Ikona koła zębatego Ustawień przechwytywania** (obok opcji Przechwyć wszystko) — otwiera [panel Ustawień przechwytywania](capture.md#the-capture-settings-pane). Funkcja jest wyłączona, jeśli nie ma projektu lub podczas przechwytywania.

### Wiersze kamer

Każdy wiersz kamery zawiera obramowanie oznaczone kolorem (niestandardowy kolor kamery), etykietę „CAM” — z niebieską literą **M**(master) lub zieloną literę**S** (slave) oznaczającą rolę elementów macierzy — oraz nazwę wyświetlaną. Domyślna nazwa to `LATTICE-MODEL (serial)`; można ją zmienić w panelu ustawień poszczególnych kamer. Przyciski w wierszu:

| Przycisk | Efekt |
| --- | --- |
| **Oko**| Przełącz widoczność. Ukryte kamery znikają z siatki i są**wykluczone z funkcji „Capture All”**. |
| **Koło zębate** | Otwiera panel ustawień poszczególnych kamer (następna sekcja). |
| **Pauza / Odtwarzanie**| Zatrzymuje podgląd na żywo**tylko po stronie wyświetlacza** — pobieranie danych na serwerze trwa nadal. Kamery w stanie pauzy nie mogą rejestrować obrazu. |
| **X** | Rozłącz. Interfejs użytkownika aktualizuje się natychmiast (w idealnym przypadku); sam proces rozłączenia po stronie serwera może zająć 10–30 s. |

### Wiersze tablicy

Wiersz tablicy zawiera etykietę „ARRAY” w kolorze tablicy, nazwę tablicy (można ją zmienić w ustawieniach tablicy) oraz przycisk **DAQ · włącz/wyłącz**—**włączony**, gdy czujnik światła na poziomie tablicy jest włączony *lub* dowolny element tablicy posiada czujnik przypisany do konkretnej kamery; w podpowiedzi podano dokładnie, który czujnik przekazuje sygnał do czego. Kamery wchodzące w skład macierzy są wymienione poniżej z wcięciem, w osobnych wierszach. Przyciski w wierszu macierzy: **oko**(ukrywa/wyświetla WSZYSTKIE elementy jednocześnie),**koło zębate**(panel ustawień macierzy),**X**(odłącza całą macierz).

Stan czujnika światła (DLS) używany w wierszach macierzy i panelu ustawień macierzy ma cztery stany:**wyłączony**,**oczekiwanie**(brak widma),**aktywny**(widmo dotarło w ciągu ostatnich 3 s) oraz**nieaktualny** — brak świeżego widma w ciągu 3 s, ale ostatni odczyt jest *nadal używany* (odczyty DAQ nigdy nie tracą ważności na ścieżce przechwytywania).

Dopóki nie zmienisz kolejności, lista wyświetla najpierw matryce, a następnie kamery samodzielne, przy czym kamery (i elementy każdej matrycy) są uporządkowane według **kolejności kamer**: M3C (Bayer) przed M3M (mono); wśród M3C, FRGB, FRGN, FOCN, a następnie FNGB; M3M według długości fali filtra, najpierw najniższa. Domyślny układ siatki jest zgodny z tą samą kolejnością. Możesz przeciągać pojedyncze kamery i całe grupy matryc w pasku bocznym, aby zmienić kolejność na liście; elementów matrycy nie można przeciągać niezależnie.

## Panel ustawień poszczególnych kamer

Otwieraj za pomocą ikony **koła zębatego** w wierszu kamery. Panel przesuwa się nad listą kamer.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Nagłówek**:**próbka koloru**kamery (kliknij, aby otworzyć natywny selektor kolorów — ustawia kolor obramowania paska bocznego oraz pierścienia wyboru kafelków),**nazwa**z przyciskiem ołówka**Zmień nazwę**(zapisanie pustej nazwy powoduje przywrócenie domyślnej nazwy `MODEL (serial)`) oraz**×** do zamknięcia.

### Histogram na żywo

W górnej części panelu znajduje się histogram luminancji na żywo, obliczany na podstawie podglądu JPEG z częstotliwością ~8 Hz. Średnia jest ważona według schematu Bayera — (R+2G+B)/4 — aby odpowiadała pomiarowi ekspozycji automatycznej (AE) właściwemu dla danej kamery.

* **Pomarańczowa (Orange) linia przerywana**= cel AE.**Przeciągnij ją w poziomie, aby zmienić cel** — po zwolnieniu myszy wysyłane jest jedno polecenie, a przeciągnięcie powoduje przełączenie trybu celu AE na ręczny.
* **Zielona (Green) linia ciągła** = rzeczywista średnia luminancji (wynik aktualnego działania AE).
* **Przycisk RGB** (w prawym górnym rogu): przełącza nakładane histogramy poszczególnych pasm, pokolorowane zgodnie z filtrem aparatu (np. w trybie FRGN: szary NIR, zielony, czerwony). W aparatach monochromatycznych (M3M) przycisk ten ma napis „MONO” i jest wyłączony — w trybie monochromatycznym zawsze wyświetlany jest jednopasmowy histogram luminancji.
* Etykiety na osi X są zgodne z głębią bitową czujnika dla bieżącego formatu pikseli: 0..255, 0..1023, 0..4095 lub 0..65535.

### Wiersze

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

z informacjami o kamerze

| Wiersz | Działanie |
| --- | --- |
| **Model** | Tylko do odczytu (np. `LATT-M3C-L87-FRGN`). |
| **Kalibracja radiometryczna** | Green na **„Aktywna”**plakietka z podpisem pokazującym poziom kalibracji, skrót, datę kalibracji oraz listę pasm, pobranych z pakietu kalibracyjnego kamery (patrz [Fabryczna kalibracja radiometryczna](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Ukryte dla kamer RGB** — posiadają one fotometryczną kalibrację balansu bieli, a nie promieniowanie dla poszczególnych pasm. |
| **Raport kalibracji**| Przycisk**Pobierz** — otwiera plik PDF certyfikatu kalibracji NIST dla danego numeru seryjnego kamery w przeglądarce systemu operacyjnego. Jeśli certyfikat nie został jeszcze zapisany w pamięci podręcznej, Chloros wyświetla zamiast tego podpowiedź. |
| **Numer seryjny** | Tylko do odczytu. |
| **Oprogramowanie sprzętowe**| Wyświetla aktualną wersję, a następnie wyszukuje dostępną wersję dla tego modelu (buforowaną dla każdego modelu — układ N kamer sprawdza serwer tylko raz). Stany: „Sprawdzanie…” → przycisk**„Zaktualizuj do X”**→ „Aktualizacja…” → „Zaktualizowano z A do B” / „Nie udało się: …” / „Pominięto: …” / zielony napis**„Aktualne”**. Podpowiedź przycisku aktualizacji: „Przywrócenie ustawień fabrycznych + flashowanie + przeprogramowanie UserSet1. ~2–3 minuty; nie odłączać.” |
| **IP** | Tylko do odczytu. |
| **Temperatura** | Tylko do odczytu, odświeżana co 3 s. Zmienia kolor na pomarańczowy przy ≥65 °C i na czerwony z symbolem ⚠ przy ≥75 °C. |
| **Cel kalibracyjny** pole wyboru | Włącza wykrywanie celu odbicia ArUco z tabelą walidacji NDVI dla poszczególnych paneli poniżej podglądu na żywo (widok listy) — zobacz [Tabelę celów kalibracji](#the-calibration-target-table). Dostępne tylko w sesji — zawsze otwiera się w stanie wyłączonym. |
| Menu rozwijane **Czujnik światła** | Powiązuje czujnik światła DAQ (DAQ-E/M/U, z listy na karcie Czujniki światła) z tą kamerą w celu korekcji oświetlenia padającego (DLS) oraz predykcyjnej automatycznej ekspozycji. „„Brak” usuwa powiązanie. Jeśli żadne czujniki nie są podłączone, menu rozwijane wyświetla komunikat „(brak podłączonych czujników — otwórz kartę DAQ)”. Powiązanie jest zapisywane wraz z projektem. |

#### Tabela celów kalibracyjnych

Tabela służy **wyłącznie do odczytu danych walidacyjnych** („tylko walidacja — nie modyfikuje kalibracji”). W nagłówku podano numer seryjny kamery, filtr, numer celu oraz rozmiar markera w pikselach, a każdy wiersz odpowiada jednemu panelowi:

* **Panel**, a następnie jedna kolumna na pasmo z wartością panelu — kamera typu Bayer wyświetla swoje trzy pasma według nazwy i długości fali (np. FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); **kamera monochromatyczna (M3M) wyświetla jedno pasmo** (np. `F750`) i nie posiada kolumn NDVI.
* **Sat** — odsetek pikseli panelu, które są nasycone (kolor bursztynowy od 1 %, czerwony od 5 %).
* **NDVI Meas**,**NDVI Theo** (referencyjny współczynnik odbicia NDVI panelu) oraz **Δ**— wyłącznie w przypadku wielospektralnych kamer typu Bayer. Mały znak po „NDVI Meas” wskazuje, na podstawie czego obliczono zmierzony współczynnik NDVI:**ρ**(odbicie — dostępny jest odczyt czujnika światła),**L**(radiancja — brak odczytu z czujnika światła) lub**DN**(surowe wartości z kamery). Tylko**ρ** NDVI jest porównywalne z NDVI Theo: w przypadku L i DN kolumna Δ jest wyszarzona, a adnotacja informuje o tym („Brak odczytu czujnika światła: NDVI pochodzi z promieniowania, więc Δ nie służy do sprawdzania kalibracji” / „NDVI pochodzi z surowych wartości z kamery, więc Δ nie służy do sprawdzania kalibracji”). W przypadku ρ wartość Δ jest zielona poniżej 0,02, pomarańczowa poniżej 0,05, a w pozostałych przypadkach czerwona.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Ekspozycja i wzmocnienie

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Wszystkie pola liczbowe wykorzystują suwaki przyspieszające po przytrzymaniu: dotknięcie = ±1, przytrzymanie >1,5 s = ±10, przytrzymanie >3 s = ±100. Wartość jest wysyłana do kamery po zwolnieniu przycisku.

| Element sterujący | Zakres / opcje | Domyślnie | Dotyczy | Funkcja |
| --- | --- | --- | --- | --- |
| **Ekspozycja (us)**| Minimalna/maksymalna wartość na żywo z kamery | Auto | Wszystkie | Czas ekspozycji w mikrosekundach, z przełącznikiem**Auto/Ręczne**przełącznikiem. Auto = ciągła automatyczna ekspozycja po stronie kamery. Ustawienie ręcznej ekspozycji pozostawia**Wzmocnienie** w trybie Auto, jeśli tak było, umożliwiając swobodne korzystanie z całego zakresu wzmocnienia kamery, dopóki nie edytujesz wzmocnienia lub nie ustawisz go na tryb Ręczny. |
| **Wzmocnienie (dB)**| Aktualne wartości minimalne/maksymalne kamery (np. do 48 dB) | Ręczne (wyłączone) | Wszystkie | Wzmocnienie analogowe/cyfrowe z własnym przełącznikiem**Auto/Ręczne**. |
| **Docelowa jasność AE**| 0–255 | 80, tryb**Auto**| Wszystkie (możliwość edycji, gdy włączona jest ekspozycja automatyczna lub automatyczne wzmocnienie) | Jasność, do której dąży system AE. W trybie**Auto**(ustawienie domyślne) kontroler oparty na histogramie sam wybiera wartość docelową, utrzymując ekspozycję na poziomie 60–75 % maksymalnej wartości czujnika. Wpisanie wartości lub przeciągnięcie pomarańczowej linii histogramu powoduje przełączenie trybu na**Ręczny**. |
| **Wygładzanie AE** | 0,5–40, krok 0,1 | 8,0 | Wszystkie | Tłumienie AE. Podpowiedź: „Niższa wartość = szybsza reakcja AE (może pulsować przy wysokiej liczbie klatek na sekundę). Wyższa wartość = płynniejsza / wolniejsza reakcja”. Wartości znacznie niższe od domyślnej mogą powodować pulsowanie AE i destabilizację transmisji przy wysokiej liczbie klatek na sekundę; 8,0 to stabilna wartość domyślna. |
| **Obszar zainteresowania AE**| Pole wyboru „Włącz” + przycisk**Celowanie**| Wyłączone | Wszystkie | Gdy opcja jest włączona, AE mierzy tylko obszar zaznaczony zieloną linią przerywaną zamiast całej klatki.**Celowanie** aktywuje funkcję „kliknij, aby umieścić” na podglądzie na żywo: kliknięcie wyśrodkowuje obszar w 30% kadru; kliknięcie i przeciągnięcie wyznacza niestandardowy prostokąt (minimum 5% × 5%). Funkcja „Aim” wyłącza się automatycznie po jednym umieszczeniu. Obszar jest mapowany z powrotem na natywne współrzędne kamery zgodnie z ustawionym obrotem/odbiciem lustrzanym i jest zapisywany wraz z projektem. |
| **Szybkość dostosowywania AE** | 0,1–5, krok 0,1 | 1,0 | Tylko dla członków tablicy | Jak szybko cel automatycznej ekspozycji śledzi zmiany jasności sceny; przy wartości 1,0× ponowna kontrola odbywa się co 2,5 s. |
| **Ochrona jasnych obszarów** | Ścisła (1 %) / Normalna (5 %) / Łagodna (15 %) | Ścisła | Kamery, które udostępniają to ustawienie | Jak duża część kadru może ulec prześwietleniu do bieli, zanim AE przyciemni obraz. |

{% hint style="info" %}
**Wymagania oświetleniowe dla wielospektralnych kamer typu Bayer (RGN / OCN / NGB):** scena musi mieć wystarczającą ilość światła we wszystkich trzech kanałach, w przeciwnym razie kalibracja nie będzie działać poprawnie — pojedyncza ekspozycja czujnika obejmuje wszystkie trzy spektra. Użyj czujnika światła DAQ do pomiaru oświetlenia lub przełącz na tryb całkowicie monochromatyczny (M3M), aby każde pasmo miało własną ekspozycję. Jeśli ujęcie narusza ten warunek, Chloros wykrywa to i ostrzega użytkownika (powiadomienie o ograniczeniu rozdzielania pasm). W trybie na żywo kafelek z podziałem na kanały wyświetla komunikat **„Dane pasma o niskim poziomie pewności · N%”**, gdy funkcja unmix ograniczyła to pasmo na 25% lub większej liczbie jego pikseli.
{% endhint %}

{% hint style="info" %}
**Jak funkcja automatycznej ekspozycji rozdziela czas naświetlania i wzmocnienie (1.3.0).** Gdy wzmocnienie jest ustawione na tryb automatyczny, Chloros utrzymuje wzmocnienie na poziomie 0 dB, dopóki ekspozycja nie osiągnie najdłuższej dozwolonej wartości (górnej granicy AE, okresu wyzwalania matrycy lub limitu ekspozycji DAQ); dopiero wtedy wzmocnienie wzrasta. W przypadku matrycy priorytet ma częstotliwość wyzwalania: automatyczna ekspozycja wypełnia okres wyznaczony przez tę częstotliwość, a następnie zwiększa wzmocnienie, dzięki czemu ciemniejąca scena nie obniża już częstotliwości wyzwalania trybu automatycznego (w wersji 1.2.0 tak było). **Ręczna**ekspozycja na elemencie macierzy trwająca dłużej niż okres wyzwalania spowalnia wyzwalanie macierzy, aby się do niej dostosować; częstotliwość automatyczna powraca, gdy każdy element, który ją spowolnił, powróci do automatycznej ekspozycji. Gdy zarówno**Ekspozycja**, jak i**Docelowa jasność AE** są ustawione na „Auto”, a czujnik światła DAQ dostarcza odczyt, ekspozycja każdej kamery jest utrzymywana po ustabilizowaniu się automatycznej ekspozycji, o ile zmierzone natężenie światła pozostaje w zakresie około ±25 %; większa, utrzymująca się zmiana natężenia światła, ponowne uruchomienie ustawień (na przykład zmiana obszaru zainteresowania) lub własna zmiana ekspozycji powodują ponowne uruchomienie tego trybu.
{% endhint %}

{% hint style="info" %}
**Jak działa automatyczna ekspozycja (wersja 1.3.0).**

* Ręczna ekspozycja pozostawia wzmocnienie w trybie automatycznym, dopóki nie zmienisz wzmocnienia.
* Niewielkie jasne obszary nie powodują przyciemnienia sceny: ignorowane są odbicia lustrzane zajmujące do 1% kadru, a w przypadku paneli znanego wzorca kalibracyjnego – do 0,5% powierzchni panelu.
* Górna granica ekspozycji DAQ obowiązuje tylko wtedy, gdy wzorzec kalibracyjny znajduje się w polu widzenia; gdy żaden panel nie jest widoczny, kamera ustawia ekspozycję dla sceny.
* Ekspozycja utrzymuje się na stałym poziomie przy stałym oświetleniu i stabilizuje się w ciągu kilku sekund po dużej zmianie. Przesycenie nie ogranicza już ekspozycji na stałe, a zmiana obszaru zainteresowania (ROI) powoduje ponowne ustalenie ekspozycji.
* Docelowa jasność nie wykracza poza zasięg kamery, więc powracające światło nie powoduje prześwietlenia kadru.
* Kamera, której automatyczna ekspozycja przestaje reagować w jasnym słońcu, utrzymuje poziom odpowiedni dla aktualnego oświetlenia, odczytywany przez czujnik światła lub – w przypadku braku podłączonego czujnika – na podstawie własnych klatek kamery, i zwolnione, gdy światło się zmienia. Nie zakłóca tego zaciemniony lub zakryty czujnik światła.
{% endhint %}

### Format pikseli i rozdzielczość

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Elementy macierzy** wyświetlają wiersze tylko do odczytu „Aktualny” (format + WxH) i „Binning” z adnotacją „Ustawiane w ustawieniach macierzy” — ponowne uruchomienie strumienia na jednym elemencie spowodowałoby utratę synchronizacji, dlatego są one zarządzane w [panelu ustawień macierzy](#array-settings-pane).**Kamery autonomiczne** mają:

| Sterowanie | Opcje | Działanie |
| --- | --- | --- |
| **Format pikseli** | 8-bit / 10-bit / 12-bit / 16-bit | Głębia bitowa czujnika; rodzina formatów odpowiada czujnikowi. Domyślnie jest to 12 bitów, a projekt zapisuje ustawione wartości, w tym 8 bitów, i ponownie łączy się zgodnie z nimi. |
| **Rozdzielczość** | Pełna / Połowa / Ćwierć | W odniesieniu do bieżącego binningu: Pełna = 2048/N × 1536/N dla binningu N×N. |
| **Binning** | 1x1 (brak) / 2x2 / 4x4 | Sprzętowy binning N×N — większe wartości obniżają rozdzielczość, ale zwiększają stosunek sygnału do szumu (SNR) i liczbę klatek na sekundę. Zmiana tej wartości powoduje ponowne uruchomienie strumienia i zresetowanie wszelkich obszarów zainteresowania (ROI) do nowego pełnego pola widzenia. |
| **Aktualne** | tylko do odczytu | Rzeczywiste wymiary WxH oraz przesunięcie (x, y) obowiązujące w danym momencie. |

### Podgląd na żywo

Wszystko w tej sekcji dotyczy **wyłącznie strony wyświetlania**— zmienia to, co widzisz w podglądzie na żywo, podczas gdy zapisane zrzuty pozostają liniowe i niezmodyfikowane — z jednym wyjątkiem:**Vignette** ma charakter radiometryczny i wpływa również na eksportowane pliki (jak opisano poniżej).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Element sterujący | Zakres / opcje | Domyślnie | Dotyczy | Działanie |
| --- | --- | --- | --- | --- |
| **Rozdzielczość renderowania** | 360p (najszybsza) / 480p / 720p / 1080p / Natywna rozdzielczość czujnika (najwolniejsza) | 720p | Wszystkie | Wysokość, na której backend uruchamia łańcuch podglądu radiometrycznego. Niższa wartość zapewnia wyższą liczbę klatek na sekundę bez zmiany pola widzenia. |
| **Indeks**| Pole wyboru „Włącz” + ikona koła zębatego | Wyłączone | Tylko wielospektralne matryce Bayera,**nie** dotyczy matryc typu combined-array | Podgląd indeksu roślinności na żywo. Ikona koła zębatego otwiera wspólny [Kalkulator indeksu](#index-calculator-pane) z wstępnie załadowanymi naturalnymi pasmami filtrów kamery (np. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Niestandardowe wyrażenie wraz z tabelą LUT (włączone/wyłączone, domyślny poziom 7 stopni, domyślne minimum 0,2, domyślne maksimum 1) jest obliczane dla każdej klatki podglądu. Członkowie macierzy połączonej ukrywają ten wiersz — macierz posiada jeden wspólny wskaźnik. |
| **Balans bieli** | Wył. / Jednorazowo / Ciągle + przycisk ponownego pomiaru | Ciągle | Tylko RGB | Balans bieli na żywo. Przycisk odświeżania ponownie rejestruje balans bieli na podstawie bieżącego widma DLS (wyłączony, gdy tryb jest wyłączony). |
| **Gamma** | Włączona / Wyłączona | Włączona | Tylko RGB | Wyświetlanie gamma (γ = 2,2 LUT) w podglądzie na żywo. Zapisane ujęcia pozostają liniowe. |
| **Redukcja szumów** | Pole wyboru + siła 0–100 | Wył. / 50 | Wszystkie (dla każdej kamery, nawet w matrycach) | Filtr dwustronny w podglądzie na żywo. Wyższa wartość = gładszy, ale mniej wyrazisty obraz. |
| **Ostrość** | Pole wyboru + siła 0–100 | Wył. / 30 | Wszystkie | Maska wyostrzająca w podglądzie na żywo, stosowana jako ostatnia. Może wzmacniać szum. Tylko w podglądzie. |
| **Wignetta**| Pole wyboru + siła 0–100 | Wyłączone / 0 | Wszystkie | Ręczna korekcja resztkowej wignetty (rozjaśnia rogi), nakładana na oszacowanie inteligentnej wignetty matrycy.**Radiometryczna — wpływa na podgląd na żywo ORAZ eksport**, w przeciwieństwie do opcji „Redukcja szumu” i „Ostrość”. |
| **Profil kolorów** | Raw / Liniowy / Naturalny / Ulepszony / Niestandardowa temperatura | Naturalny | Tylko RGB | Zobacz poniżej. |
| **Temperatura barwowa** | 2000–10000 K, krok co 100 | 5500 K | Tylko RGB, tylko profil temperatury niestandardowej | Ustawia balans bieli na stałą skorelowaną temperaturę barwową (dane wejściowe DLS są ignorowane). Ostatnio wybrana wartość w kelwinach jest zapamiętywana przy zmianie profilu. |
| **Najjaśniejsze obszary** | od −100 do 0, krok co 1 | 0 (wyłączone) | Tylko RGB | Nowość w wersji 1.3.0. Przyciemnia najjaśniejsze, najbardziej białe obszary obrazu (0 = bez zmian, −100 = najsilniejsze przyciemnienie). Działa w podglądzie na żywo **oraz w zrzutach na poziomie wyświetlacza (Podgląd)**, które rejestrują to ustawienie (`Chloros:DisplayHighlights`); nie ma to nigdy wpływu na eksport danych promieniowania i odbicia. Nie pozwala to na odzyskanie danych z czujnika, które uległy przesterowaniu — w tym celu należy zmniejszyć ekspozycję. Nakładka zebry dostosowuje się do przyciemnionego obrazu. Wyłączone przy profilu kolorów Raw („Nie jest stosowana przy profilu kolorów Raw.”). |
| **Nasycenie** | 0–200 (100 = neutralne) | 100 | Tylko RGB | Nasycenie HSV w podglądzie na żywo. |
| **Kontrast** | 0–200 (100 = neutralny) | 100 | Tylko RGB | Kontrast liniowy wokół średniej szarości w podglądzie na żywo. |
| **Odbicie lustrzane w poziomie / Odbicie lustrzane w pionie** | Pola wyboru | Wyłączone | Wszystkie | Odwróć podgląd w poziomie / w pionie. |
| **Obrót**| 0° / 90° / 180° / 270° | 0° | Wszystkie | Obróć podgląd. Orientacja jest stosowana na końcu łańcucha podglądu zaplecza.**Zapisane ujęcia (1.3.0):**produkty samodzielnego aparatu, takie jak Podgląd, Radiancja, Odbicie i Indeks, są obracane za pomocą funkcji Odbicie lustrzane/Obrót, podczas gdy**pliki RAW i po usunięciu efektu Bayesa zachowują orientację natywną dla kamery**; każdy plik z ujęciem zapisuje to ustawienie w metadanych XMP, więc późniejsze przetwarzanie pliku RAW stosuje je w ten sam sposób.**Pliki składowe tablicy zachowują orientację natywną** (ustawienie jest zapisywane, ale nie stosowane), a widoki złożone z tablicy ignorują je. |

**Semantyka profili kolorów** (kamery RGB):

* **Raw** — całkowite pominięcie łańcucha przetwarzania.
* **Liniowy** — sygnał ciemności + pole płaskie + balans bieli; brak macierzy kolorów, brak gamma.
* **Naturalny** *(domyślny)* — liniowy plus zmierzona macierz korekcji kolorów i krzywa tonów dostosowująca się do sceny.
* **Wzbogacony**— Naturalny plus intensywność i lokalny kontrast CLAHE. Dodatkowy koszt dotyczy**wyłącznie podglądu na żywo** — zapisane zdjęcia zawsze otrzymują pełne wykończenie niezależnie od profilu.
* **Temperatura niestandardowa** — tryb Natural z balansem bieli ustawionym na wybraną wartość w kelwinach.

{% hint style="warning" %}
W przypadku trybów Naturalny, Ulepszony i Niestandardowa temperatura w panelu wyświetlana jest uwaga dotycząca tonacji: klatki są rozjaśniane w zależności od konkretnej sceny, więc zapisane obrazy *wyświetlane* nie są porównywalne między sobą. **Eksportuj promieniowanie lub współczynnik odbicia w celu przeprowadzenia pomiarów.**
{% endhint %}

### Nakładki wyświetlacza (nakładane na obraz na żywo)

Są one dostępne wyłącznie w interfejsie użytkownika — nakładane na wideo, nie mają wpływu na strumień ani na zrzuty ekranu.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Nakładka | Elementy sterujące | Domyślnie | Działanie |
| --- | --- | --- | --- |
| **Zebra** | Pole wyboru + próg 200–255 | Wyłączone / 250 | Magentowe ukośne paski na przyciętych pikselach. |
| **Krzyżyk** | Pole wyboru | Wyłączone | Znak wyznaczający środek klatki. |
| **Siatka** | Wyłączone / 3 × 3 / 9 × 9 | Wyłączone | Siatka kompozycyjna. |
| **Histogram** | Pole wyboru + szerokość 0,10–0,90 klatki | Wyłączone / 0,25 | Pasek histogramu wyświetlany na strumieniu. |
| **Focus Peak** | Pole wyboru + próg 20–200 + próbka koloru | Wyłączone / 80 / `#ff5722` | Podświetlenie krawędzi metodą Sobela do ustawiania ostrości. |
| **Podział kanałów** | „Pokaż podział (Red / Green / NIR)” / przycisk „Ukryj podziały” | Ukryte | Dodaje trzy niezależne kafelki w skali szarości dla każdego kanału obok obrazu złożonego (nazwa przycisku odpowiada kanałom filtrów aparatu). Każdy kafelek z podziałem można przeciągać i ma on ten sam kolor obramowania co aparat. Niedostępne w aparatach monochromatycznych. Zapisywane wraz z projektem. |

### Pomiar punktowy

* Pole wyboru **Kliknij, aby pobrać próbkę**: kliknij obraz na żywo, aby pobrać próbkę pojedynczego piksela (zostanie on zaznaczony krzyżykiem), lub kliknij i przeciągnij, aby zaznaczyć obszar i uzyskać średnią pikselową. Przycisk**Wyczyść**usuwa próbkę i krzyżyk. Funkcja ta wyklucza się z trybem**Aim** w AE-ROI.
* **Menu rozwijane**Pokaż**:**Raw (głębia bitowa)**— natywne wartości cyfrowe o głębokości bitowej czujnika (np. 12 bitów → 0..4095) — lub**Wyświetlanie (8 bitów)** (domyślnie). Gdy aktywny jest indeks na żywo, opcja Wyświetlanie pokazuje zamiast tego obliczoną wartość indeksu (np. NDVI).
* Panel odczytu zawiera współrzędne pikseli, rozmiar klatki, format pikseli, głębię bitową oraz tabelę kanałów (Chan / Wartość / %) z etykietami pasm i długościami fal; pary zielonych pikseli w układzie Bayera są uśredniane; próbki z regionu wyświetlają „N px avg”.

Stan pomiaru punktowego jest dostępny tylko w ramach bieżącej sesji.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Predykcyjna automatyczna ekspozycja (sterowana przez DLS)

Ta sekcja pojawia się tylko wtedy, gdy **podłączony jest co najmniej jeden czujnik światła DAQ** — solwer potrzebuje aktualnego widma promieniowania padającego, aby ją sterować.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Sterowanie | Zakres | Domyślnie | Działanie |
| --- | --- | --- | --- |
| **Włącz** | Pole wyboru | Włączone (kamery autonomiczne) | Solwer zamkniętej postaci wykorzystuje widmo DLS oraz skalarne wartości z pakietu kalibracyjnego aparatu, aby najjaśniejsze pasmo znalazło się blisko nasycenia, jednocześnie utrzymując najciemniejsze pasmo powyżej dolnej granicy stosunku sygnału do szumu (SNR) — pojedynczy zapis ekspozycji na każde rozwiązanie, bez pętli stabilizacji. Przeznaczony do timelapse'ów zasilanych energią słoneczną, gdzie każde ujęcie musi być prawidłowo naświetlone. Moduł zaplecza automatycznie przechodzi na reaktywną ekspozycję automatyczną (AE), gdy odczyt DLS jest nieaktualny/brakuje go lub pakiet kalibracyjny nie jest załadowany. |
| **Wygładzanie (α)** | 0,05–1,0, krok 0,05 | 0,3 | Wygładzanie kolejnych rozwiązań prognostycznych (im niższa wartość, tym większe wygładzenie). |
| **Odbicie sceny**| Przycisk**Rekalibruj ρ** | — | Ponownie szacuje współczynnik odbicia sceny używany przez solwer. |

{% hint style="info" %}
**Połączenie matryc domyślnie wyłącza predykcyjną ekspozycję automatyczną (AE)** — w przypadku matryc ekspozycję obsługuje inteligentna ekspozycja automatyczna Chloros oraz automatyczna ekspozycja po stronie aparatu (z ochroną przed nasyceniem), a oszacowanienie jest bezpieczne w przypadku scen mieszanych. Można ją tutaj ponownie włączyć dla poszczególnych kamer, jeśli zależy nam konkretnie na ekspozycji radiometrycznej sterowanej przez DLS.
{% endhint %}

**Górna granica ekspozycji sterowana przez DAQ oraz AE z przypiętym natężeniem światła padającego.** Niezależnie od powyższego pola wyboru, gdy czujnik światła DAQ jest przypisany do kamery RGB, Chloros oblicza — na podstawie zmierzonego bezwzględnego natężenia promieniowania padającego — maksymalną ekspozycję × wzmocnienie, przy której powierzchnia o 100% odbiciu pozostaje poniżej poziomu przesterowania, i stosuje ją jako **limit**w trybie automatycznej ekspozycji. Od wersji 1.3.0 limit ten obowiązuje tylko wtedy, gdy w polu widzenia tej kamery znajduje się cel kalibracyjny (widoczny w kadrze w ciągu ostatnich 10 s); gdy w polu widzenia nie ma celu, zamiast tego działa zwykła automatyczna ekspozycja kamery z własną ochroną przed prześwietleniem, dzięki czemu reszta sceny nie jest utrzymywana w ciemności większej niż to konieczne. Gdy limit jest aktywny, kamera działa w trybie**przypiętej ekspozycji padającej**: działa w trybie pętli otwartej przy ekspozycji mierzonej na światło padające, z wzmocnieniem ustawionym na 0 dB — ekspozycja dostosowuje się do zmierzonego natężenia światła, a nie do zawartości sceny. Ponieważ limit może jedynie skrócić czas naświetlania, sam w sobie nie może spowodować przesterowania. Od wersji 1.3.0 limit jest obliczany na podstawie własnej stałej ekspozycji kamery z pakietu kalibracyjnego (wartość średnia dla floty tylko wtedy, gdy pakiet nie zawiera własnej). Limit wyłącza się automatycznie — a normalna automatyczna ekspozycja sceny zostaje wznowiona — gdy w polu widzenia nie ma celu kalibracyjnego, brakuje odczytu z systemu DAQ, odczyt jest nieaktualny (>30 s), ciemny lub jeśli ≥15 % klatki ulega przesterowaniu przy zablokowanej ekspozycji (co oznacza, że czujnik i kamera rejestrują różne natężenie oświetlenia). Nie ma przełącznika w interfejsie graficznym; jest to standardowe zachowanie, gdy kamera RGB ma powiązany moduł DAQ.

### Pozyskiwanie danych i wyzwalanie

Elementy<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

macierzy wyświetlają dodatkowo wiersze tylko do odczytu: **Rola**(Master w kolorze niebieskim / Slave w kolorze zielonym),**Linia synchronizacji**oraz**Rówieśnicy**.

| Sterowanie | Opcje | Domyślnie | Uwagi |
| --- | --- | --- | --- |
| **Tryb wyzwalania** | Wył. / Wł. | Wł. | Wyłączony dla elementów tablicy (tablica zarządza wyzwalaniem). |
| **Źródło wyzwalania** | Oprogramowanie / Linia 0 (M8) / Linia 1 / Linia 2 | Linia 0 | Ukryte, gdy tryb wyzwalania jest wyłączony; wyłączone dla elementów macierzy. Linia 0 to optoizolowane zewnętrzne wejście wyzwalające M8. |
| **Częstotliwość klatek**| Auto / Ręcznie + wartość | Auto |**Auto**: ograniczenie częstotliwości klatek kamery jest wyłączone — ekspozycja decyduje o liczbie klatek na sekundę, a w oknie wyświetlana jest aktualna częstotliwość na żywo.**Ręcznie**: limit liczby klatek na sekundę ustawia się za pomocą suwaka (od 1 do maksymalnej wartości ograniczonej przepustowości), którego wartość początkowa jest oparta na aktualnej rzeczywistej częstotliwości. Użytkownicy macierzy widzą tylko do odczytu informację „N klatek na sekundę (na żywo)” z adnotacją „Ustaw w ustawieniach macierzy”. |

### Sieć / Transport

| Wiersz | Zachowanie |
| --- | --- |
| **Rozmiar pakietu**| 1500 (standardowy) / 9000 (Jumbo) — domyślnie**Jumbo**. |
| **Przepustowość** | Limit przepustowości łącza tylko do odczytu w MB/s. Backend rozkłada tę wartość równomiernie na wszystkie podłączone kamery przy każdym podłączeniu/odłączeniu. |
| **Obsługa bufora** | Tryb obsługi bufora tylko do odczytu. |

### Przechwytywanie

Panel kończy się przyciskiem **„Otwórz ustawienia przechwytywania…”**, który przenosi do [panelu ustawień przechwytywania](capture.md#the-capture-settings-pane) (nieaktywnego, dopóki nie zostanie otwarty projekt — „Utwórz lub otwórz projekt, aby zapisać przechwycone dane”). Jeśli kamera jest ukryta lub wstrzymana, pojawia się podpowiedź przypominająca o konieczności jej odkrycia/wznowienia przed rozpoczęciem przechwytywania.

## Panel ustawień macierzy

Otwierany za pomocą **ikony koła zębatego**w wierszu**ARRAY**. Nagłówek: nazwa macierzy z ołówkiem do zmiany nazwy oraz**×** do zamknięcia. Sekcje poniżej oznaczone jako *tylko połączone* pojawiają się wyłącznie w przypadku macierzy połączonych w trybie wyświetlania połączonego.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronizacja

Wiersze **Master**,**Slaves**i**Sync Line** są tylko do odczytu.

### Czujnik światła otoczenia

Wyświetlane zarówno dla macierzy połączonych, jak i oddzielnych:

* Menu rozwijane **Czujnik światła**— przypisuje jeden moduł DAQ do całej macierzy. Wybór obowiązuje natychmiast, przenosi się do menu rozwijanego**Czujnik światła** każdej kamery w macierzy (nadal można zmienić ustawienia dla poszczególnych kamer) oraz rozpoczyna przesyłanie widm do macierzy.
* Wiersz **Status** na żywo: Wyłączony · „Oczekiwanie na pierwsze widmo…” · „Aktywny — wszystkie kamery w macierzy mają skorygowane oświetlenie” · „Brak nowego widma w ciągu ostatnich 3 s — nadal używany jest ostatni odczyt (brak limitu czasu dla nieaktualnych danych)…”.
* Uwaga w panelu: „Korekcja radiometryczna dla całej macierzy. Ustawienia poszczególnych kamer mają pierwszeństwo”.

### Cel kalibracyjny

Własna grupa bezpośrednio po czujniku światła otoczenia, zarówno dla macierzy połączonych, jak i oddzielnych:

* Pole wyboru **Włącz** — „Wykryj cel MAPIR ArUco i zweryfikuj NDVI względem tablicy LUT odbicia panelu”; steruje nakładką celu i tabelami walidacji dla połączonej płytki. W przypadku **oddzielnej** matrycy włącza własną [tabelę celu kalibracji](#the-calibration-target-table) (po jednej na każdy wiersz kamery w widoku listy).

### Przechwytywanie — jednolite ustawienia czujników

Wyświetlane zarówno dla układów połączonych, jak i oddzielnych (nagłówek grupy brzmi „Przechwytywanie (jednolite dla wszystkich kamer)”). Ustawienia te mają jednolite zastosowanie do każdego elementu (zmiany dla poszczególnych elementów zakłóciłyby synchronizację). Zmiany są buforowane i stosowane łącznie po naciśnięciu przycisku **Zastosuj**— z wyjątkiem przełącznika**ROI włączone / ROI wyłączone**, który działa natychmiastowo.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Sterowanie | Opcje / zakres | Działanie |
| --- | --- | --- |
| **Format pikseli** | 8-bitowy / 10-bitowy / 12-bitowy / 16-bitowy | Jednolita głębia bitowa dla wszystkich elementów; każda kamera przyjmuje format tej głębi dla swojego czujnika. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning sprzętowy — zachowuje pełne pole widzenia, jednocześnie zwiększając stosunek sygnału do szumu (SNR) i liczbę klatek na sekundę. Zmiana tej opcji resetuje pola ROI do nowego pełnego pola widzenia. |
| Ustawienie wstępne **rozdzielczości** | Pełna / Połowa / Ćwierć | W odniesieniu do binningu; wypełnia pola ROI wycięciem wyśrodkowanym. |
| **Kadrowanie ROI (px)**| Przełącznik**ROI włączone / wyłączone**, przycisk**Wybierz**, pola Szer. / Wys. / X / Y, mapa całego czujnika | Kadrowanie czujnika — zobacz [Edytor ROI](#the-roi-editor). Szerokość/wysokość dopasowują się do wielokrotności 16 (minimum 64); przesunięcia przyciągają się do wielokrotności 4. Wskazówka „maks. WxH” pokazuje górną granicę, a przycisk**Resetuj** przywraca pola do pełnego pola widzenia. Podczas edycji pomarańczowe pole kadrowania śledzi każdą zmianę pola na mapie i na kafelku matrycy. |
| **Częstotliwość wyzwalania**| Przełącznik Auto / Ręcznie + fps 0,5–10, krok 0,5 |**Auto**(domyślnie): backend oblicza częstotliwość wyzwalania na podstawie rozdzielczości i przepustowości — pole wprowadzania danych jest wyłączone i wyświetla obliczoną wartość.**Ręcznie**: ustala wartość po kliknięciu „Zastosuj”. Ustalona częstotliwość, której nowy rozmiar klatki nie jest w stanie obsłużyć, zostaje ograniczona do najszybszej częstotliwości, na jaką pozwala ten rozmiar, i przywracana, gdy rozmiar klatki znów na to pozwala. |

Uwaga w panelu: „Zmiany formatu/rozdzielczości powodują krótkotrwałe ponowne uruchomienie wszystkich kamer. Częstotliwość wyzwalania obowiązuje na żywo”. Przyciski **Zastosuj / Anuluj** znajdują się na dole panelu.

#### Edytor obszaru zainteresowania (ROI)

Nowość w wersji 1.3.0. Wiersz przycinania ROI i mapa poniżej współpracują ze sobą:

* **ROI włączone / ROI wyłączone**—**natychmiast**włącza lub wyłącza przycinanie czujnika (kamery na krótko się restartują; kafelek matrycy wyświetla komunikat „Stosowanie ustawień…”).**Wyłączone**odczytuje pełny obraz z czujnika i**zapamiętuje Twoje przycięcie**— pola nadal je wyświetlają;**Włączone** ponownie je stosuje. Włączenie obszaru ROI bez ustawionego przycięcia powoduje zastosowanie wyśrodkowanego przycięcia o połowie rozmiaru („Nie ustawiono przycięcia, więc zastosowano wyśrodkowane przycięcie o połowie rozmiaru. Dostosuj je za pomocą pól.”).
* **Wybierz**— rysuje obszar wycinania na mapie: przeciągnij po nim lub kliknij dwa przeciwległe rogi (klawisz Esc anuluje). Prostokąt pojawia się w polach W/H/X/Y, zgodnie z powyższymi krokami; nic nie zostanie zastosowane, dopóki nie naciśniesz**Zastosuj**, więc możesz najpierw precyzyjnie dostosować kadr za pomocą strzałek w polach. Jedno naciśnięcie przycisku Wybierz tworzy jeden prostokąt.
* **Mapa**pokazuje kadr na całym czujniku, więc możesz go ponownie poszerzyć: „Pełny czujnik, na żywo”, podczas gdy kamery odczytują pełny czujnik, lub „Pełnoklatkowa migawka z {czas}. Kamery odczytują tylko wykadrowany obszar”, gdy są one wykadrowane. Jeśli macierz przesyłała dotychczas wyłącznie wykadrowane dane, mapa wyświetla komunikat „Brak jeszcze widoku pełnoklatkowego: ta macierz przesyłała wyłącznie wykadrowane dane.” wraz z przyciskiem**Pokaż pełną klatkę**, który wyłącza obszar zainteresowania (ROI). Etykieta kadru `W×H @ X,Y` znajduje się poza ramką (nad nią lub pod nią, gdy ramka styka się z górną krawędzią), zarówno na mapie, jak i na kafelku na żywo.
* Przycisk **Zastosuj**zatwierdza ustawienia pól: kadrowanie włącza obszar zainteresowania (ROI); pełna klatka (po naciśnięciu**Resetuj**) wyłącza go, zachowując w pamięci poprzednie kadrowanie; obszar zainteresowania, który jest wyłączony, a którego pola nadal pokazują zapamiętane kadrowanie, pozostaje wyłączony po zastosowaniu kolejnej zmiany (formatu, binningu, częstotliwości).
* **W przypadku wyrównanej matrycy okno kadrowania każdego elementu jest umieszczane zgodnie z wyrównaniem**, dzięki czemu każda kamera kadruje ten sam fragment sceny. Matryca wykadrowana przed wyrównaniem jest najpierw wyrównywana na całym czujniku (kadrowanie jest usuwane, wyrównanie ustalane, a następnie przywracane z oknami dla poszczególnych kamer).
* Po zmianie kadrowania automatyczna ekspozycja ponownie dostosowuje się do nowego kadru. W ramach kadrowania ekspozycja elementu może nadal trwać do upływu okresu wyzwalania.
* Stan włączenia/wyłączenia obszaru zainteresowania (ROI) oraz zapamiętane kadrowanie są zapisywane wraz z projektem.

### Wyrównanie (współrejestracja)

Wyświetlane zarówno dla **połączonych, jak i oddzielnych tablic** (wersja 1.2.0 pokazywała to tylko dla połączonych tablic); nagłówek grupy brzmi „Wyrównanie”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Pole **Residual**: „RMS x.xx px” — zielone przy wartości poniżej 1 px, bursztynowe przy wartości poniżej 3 px, czerwone w pozostałych przypadkach lub jeśli jakakolwiek kamera nie przeszła testu; „brak profilu” przed pierwszym obliczeniem.
* Wiersz podsumowujący: „✓ Wszystkie kamery wyrównane (N)” / „⚠ p/N kamer wyrównanych —  <serial (filter)="">niepowodzenie” / „Aktywne kadrowanie — ponowna kalibracja w celu wyrównania (wykorzystuje cały czujnik)” / „Oczekiwanie na ustabilizowanie się ekspozycji…” / „Stosowanie ustawień…” (kamery uruchamiają się ponownie) / „Układ przestał dostarczać klatki — podłącz ponownie układ”.
* Tabela dla poszczególnych kamer: kamera (ostatnie 4 cyfry numeru seryjnego + filtr), błąd reprojekcji w pikselach wraz z liczbą dopasowań („ref” dla kamery głównej) oraz wynik znormalizowanej korelacji krzyżowej nakładania się w stosunku do progu 0,35.
* **Przeprowadź ponowną kalibrację wyrównania** przycisk (z napisem „Kalibruj wyrównanie” przed pierwszym profilem) — ponownie uruchamia współrejestrację na nowych klatkach.
* **Pole wyboru „Automatyczna ekspozycja kamer do wyrównania”** (domyślnie zaznaczone) — tymczasowo rozjaśnia ciemne lub płaskie kamery (najpierw ekspozycja, potem wzmocnienie), aby miały teksturę do dopasowania, a następnie przywraca automatyczną ekspozycję.

Chloros automatycznie wyrównuje każdy układ po połączeniu, w obu trybach wyświetlania, gdy tylko ustabilizuje się automatyczna ekspozycja elementów (próby trwają do 5 minut); połączony podgląd również wyrównuje się po otwarciu. Osobny układ nie ma kompozytowej płytki, więc ta sekcja sprawdza stan wyrównania i sama uruchamia funkcję „Recalibrate”. Należy przeprowadzić ponowną kalibrację, jeśli zmieniła się ostrość lub głębia sceny. Wyrównanie jest **z założenia ograniczone do danej sesji** — nigdy nie jest zapisywane w profilu, ponieważ zależy od aktualnej odległości sceny. Nagrania nadal można eksportować z rejestracją pikselową (zobacz [Eksporty z wyrównaniem](capture.md#per-array-controls)).

### Inteligentna korekcja winietowania

* Pole wyboru **Włącz korekcję**— stosuje oszacowanie winietowania dla poszczególnych kamer do łańcucha radiometrycznego (na żywo**i** w eksportach).
* **Kalibruj na podstawie bieżącego widoku**— najpierw skieruj matrycę na jednolity obiekt (płaski panel, ścianę lub niebo); każda kamera jest indywidualnie wyrównywana, a status wskazuje „n/N kamer · −x,x %” poprawy płaskości. Przycisk**Wyczyść** usuwa oszacowanie.
* Status: „Skalibrowano”, „Nie skalibrowano” lub „Częściowo skalibrowano — uruchom ponownie”, gdy tylko szacunki niektórych kamer nadal mają zastosowanie przy ich aktualnym kadrowaniu (na przykład po tym, jak funkcja „Skalibruj ponownie” przesunęła okno kadrowania danej kamery) — uruchom ponownie funkcję „Kalibruj”.
* Dostrój ustawienia dla poszczególnych kamer za pomocą suwaka **Vignette** w [Podglądzie na żywo](#live-preview).

### Podgląd na żywo

* Pole wyboru **Histogram**+ suwak szerokości (0,10–0,90 klatki) — pasek histogramu wyświetlany na żywo na kafelkach wszystkich elementów jednocześnie (w tym na kafelkach z podzielonymi kanałami), oraz [Nakładka histogramu](#display-overlays-drawn-over-the-live-feed) ustawiona dla całej matrycy. Wyświetla się jako włączona, gdy każdy członek już ją posiada. W przypadku**oddzielnej** matrycy jest to jedyny wiersz w grupie.
* *Tylko w trybie połączonym:* **Indeks**: pole wyboru „Włącz” + ikona koła zębatego — otwiera wspólny [Kalkulator indeksu](#index-calculator-pane) z pasmami wygenerowanymi na podstawie danych z**wszystkich** kamer w sieci. Linia podglądu wyrażenia poniżej pokazuje bieżące wyrażenie („Nie ustawiono wyrażenia — otwórz kalkulator, aby je utworzyć”), odświeżane co sekundę.
* *Tylko w trybie połączonym:* Lista rozwijana **Rozdzielczość renderowania**(te same ustawienia wstępne co dla poszczególnych kamer, domyślnie 720p): wysokość strumienia podglądu na żywo**oraz** rozmiar zapisanego eksportu kompozycji. Uwaga w panelu: „Podgląd + rozmiar zapisanej kompozycji. Obrazy z poszczególnych kamer są zawsze eksportowane w pełnej rozdzielczości”.

### Warstwy wyświetlania *(tylko w trybie połączonym)** Pole wyboru **Włącz** (domyślnie wyłączone — kamera główna jest wyświetlana bezpośrednio; włączone = kompozycja warstwowa).
* **Listy rozwijane**Pierwszy plan**/**Tło**: każda kamera członkowska (według nazwy) lub**Indeks**. Gdy w polu Pierwszy plan wybrano opcję Indeks, piksele poza zakresem minimalnym/maksymalnym LUT wyświetlają warstwę Tło.

### Widok podzielony *(tylko w trybie połączonym)*

**„Pokaż kamery członkowskie”**— przycisk**Podziel / Ukryj kamery członkowskie**, który dodaje obraz na żywo z każdej kamery członkowskiej jako oddzielne kafelki siatki obok obrazu złożonego. Kafelki odczytują istniejący bufor klatek macierzy (bez dodatkowego podłączenia kamery). Tylko widok siatki; zapisywane dla każdej macierzy wraz z projektem.

### Możliwości

Panel tylko do odczytu odświeżany co 5 s:

* **Etykieta poziomu**: „Jednoczesne przechwytywanie” (zielony) · „Jednoczesne przechwytywanie (emitowanie FTD z przesunięciem)” (zielony) · „Przechwytywanie z przesunięciem (odchylenie 100 ms)” (bursztynowy) · „Konfiguracja zbyt duża” (czerwony).
* **Stan klatki**: „x,xx % niekompletne” — zielony poniżej 1 %, bursztynowy poniżej 5 %, czerwony przy 5 % lub więcej.
* **Linia łącza**: „NIC {mbps} Mbps – stała prędkość {MB/s} MB/s”.

Jest to aktualny budżet przepustowości macierzy. Aby zapoznać się z podstawowymi parametrami fps i modelem sieciowym — oraz dowiedzieć się, co należy zmienić, gdy poziom przejdzie na kolor bursztynowy lub czerwony — zobacz [Macierze wielokamerowe](arrays.md) oraz [Podręcznik CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Panel „Kalkulator indeksu”

Trzecia strona paska bocznego, wspólna dla narzędzia „Indeks” dla poszczególnych kamer oraz narzędzia „Indeks” dla całej macierzy (po jednym naraz — nagłówek brzmi „Kalkulator indeksu — <camera name="">” lub „Kalkulator indeksu — <array name="">”). Otrzymuje ona listę pasm (pasma naturalne filtralub wszystkie pasma we wszystkich kamerach w macierzy), bieżące wyrażenie oraz konfigurację LUT (włączone/wyłączone, poziom — domyślnie 7 stopni, min. — domyślnie 0,2, maks. — domyślnie 1), a także histogram indeksu na żywo. Przycisk **Zastosuj** zatwierdza wyrażenie; zmiany LUT są na bieżąco stosowane w podglądzie.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Ustawienia poszczególnych kamer a ustawienia zarządzane przez macierz

Skrócony przewodnik po tym, co znajduje się gdzie, gdy kamera jest elementem macierzy:

| Zarządzane przez macierz (tylko do odczytu w panelu kamery) | Nadal dla poszczególnych kamer w obrębie macierzy |
| --- | --- |
| Format pikseli, rozdzielczość, binning | Automatyczna ekspozycja (ekspozycja, wzmocnienie, wartość docelowa, wygładzanie, obszar zainteresowania) |
| Tryb wyzwalania/źródło, liczba klatek na sekundę | Redukcja szumów, ostrość, winietowanie |
| | Orientacja (odbicie lustrzane/obrót), nakładki wyświetlacza (ustawienia macierzy → podgląd na żywo ustawia histogram jednocześnie na każdym elemencie), pomiar punktowy |
| | Indeks (macierze z oddzielnym wyświetlaczem), powiązanie czujnika światła |

Inne zachowania dotyczące całej matrycy:

* **Wyświetlanie połączone vs oddzielne** wybiera się podczas podłączania matrycy: połączone = jedna wyrównana kompozytowa płytka (kamery przesyłają obraz wyłącznie poprzez widok podzielony); oddzielne = każda kamera renderuje własną zsynchronizowaną płytkę. Kamera nigdy nie wyświetla jednocześnie samodzielnego obrazu i płytki matrycy.
* **Automatyczne ponowne połączenie**: otwarcie zapisanego projektu przywraca kamery i matryce oraz ponownie stosuje wszystkie zapisane ustawienia do zaplecza przed wznowieniem strumieni.
* **Filtrowanie podczas przechwytywania**: ukryte lub wstrzymane kamery są wykluczane z funkcji „Capture All”; matryca jest całkowicie zablokowana tylko wtedy, gdy WSZYSTKIE elementy są ukryte/wstrzymane. Zobacz [Ustawienia i tryby przechwytywania](capture.md).

## Jak zachowywane są ustawienia

Stan zakładki kamery jest zapisywany **wraz z projektem**, a nie w przeglądarce:

* Każda reaktywna zmiana tworzy migawkę kamer i macierzy w pliku projektu `cameras.json` (z opóźnieniem 500 ms). Obejmuje to nazwy i kolory kamer, ustawienia ekspozycji/wzmocnienia/AE, format pikseli/rozdzielczość/binning, częstotliwość wyzwalania, ustawienia podglądu (rozdzielczość renderowania, redukcja szumów, ostrość, winietowanie, profil kolorów, światła, nasycenie/kontrast), orientację, nakładki, podział kanałów, konfigurację indeksu, ustawienia predykcyjnego AE, obszar zainteresowania AE (ROI), nazwy tablic, tryb wyświetlania, ustawienia przechwytywania tablic (w tym kadrowanie obszaru zainteresowania, to, czy obszar zainteresowania jest włączony, oraz zapamiętane kadrowanie, gdy jest wyłączony), budżet przewodów tablicy oraz blok siatki (powiększenie strumienia, tryb widoku, blokada siatki, ręczna kolejność kafelków, ukryte kamery, zamknięte kafelki, aktywna kamera).
* Powiązania czujników światła są zapisywane w pliku projektu `sensors.json`.
* Ponowne otwarcie projektu powoduje ponowne podłączenie sprzętu i ponowne zastosowanie wszystkich tych ustawień.
* **Brak otwartego projektu = tylko sesja**: bez projektu nic nie zostanie zachowane po zamknięciu programu Chloros.
* Tylko sesja, niezależnie od projektu: stan wstrzymania, próbki pomiaru punktowego, pole wyboru „Cel kalibracji” dla poszczególnych kamer (zawsze domyślnie wyłączone) oraz profil wyrównania matrycy (zgodnie z założeniami przeliczany na nowo dla każdej sesji).
* Jedyny wyjątek: wybrane opcje eksportu w **Ustawieniach przechwytywania** oraz tryb przechwytywania są zachowywane dla każdego projektu w lokalnej pamięci aplikacji, a nie w `cameras.json` — zobacz [Ustawienia i tryby przechwytywania](capture.md).</array></camera></serial>
