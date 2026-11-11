# Ustawienia i tryby rejestracji

Rejestracja w zakładce „Kamery” odbywa się za pomocą jednego czerwonego przycisku **Capture All**oraz jednego panelu**Capture Settings**, który określa, jakie wyniki generuje ten przycisk: które kamery biorą udział w rejestracji, jakie typy plików eksportowych zapisuje każda z kamer oraz czy migawka wyzwala się jednorazowo, w trybie ciągłym czy w określonych odstępach czasu. Na tej stronie opisano cały proces — konfigurację, sam proces przechwytywania, lokalizację plików na dysku oraz sposób ich późniejszego przetworzenia na skalibrowane produkty. Same elementy sterujące kamerami i układami kamer znajdują się w sekcji [Ustawienia kamer](camera-settings.md).

{% hint style="info" %}
**Przeprowadzanie sesji wymaga otwartego projektu.** Opcja „Przechwyć wszystko” oraz ikona koła zębatego „Ustawienia przechwytywania” są nieaktywne, dopóki projekt nie zostanie otwarty („Utwórz lub otwórz projekt, aby zapisać przechwycone dane”). Każde przechwycenie jest zapisywane w folderze projektu w `captures/`.
{% endhint %}

## Panel „Ustawienia przechwytywania”

Można go otworzyć za pomocą **ikony koła zębatego obok opcji „Przechwyć wszystko”**na liście kamer w pasku bocznym lub przycisku**„Otwórz ustawienia przechwytywania…”** znajdującego się na dole dowolnego panelu ustawień danej kamery. Nagłówek brzmi „Ustawienia przechwytywania” i zawiera przycisk ← (wstecz).

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Wybrane tutaj opcje — uwzględnione kamery, pola wyboru dla poszczególnych typów oraz tryb przechwytywania — są zapisywane **dla każdego projektu** i przywracane po ponownym otwarciu projektu.

### Tryby przechwytywania

Trzy przyciski trybów u góry panelu:

| Tryb | Działanie | Podustawienia (domyślne) |
| --- | --- | --- |
| **Pojedyncze** *(domyślnie)* | Jedno przechwycenie ze wszystkich wybranych kamer. | — |
| **Ciągłe**| Przechwytywanie jedno po drugim aż do spełnienia warunku zatrzymania. | Zatrzymanie po**liczbie zdjęć** (domyślnie 1) *lub* **czasie trwania sesji** (domyślnie 10 s; jednostki: sekundy / minuty / godziny / dni). |
| **Interwał**(timelapse) | Seria zdjęć według timera. |**Liczba zdjęć / interwał**(domyślnie 1) ·**Co**N jednostek (domyślnie 5 s) ·**Przez** N jednostek (domyślnie 1 m). |

W trybie ciągłym lub interwałowym przycisk „Zrób wszystkie zdjęcia” zmienia się podczas działania w przycisk **Zatrzymaj (N)**, zliczając zdjęcia w miarę ich pojawiania się. Zapełnienie dysku zatrzymuje przebieg (zobacz [Przebieg funkcji „Zrób wszystkie zdjęcia”](#the-capture-all-flow)).**Tryb ciągły dla macierzy działa wewnątrz backendu (nowość w wersji 1.3.0).** Gdy wszystkie kamery na liście należą do macierzy (kamery autonomiczne są ukryte lub wykluczone), tryb ciągły nie wysyła już jednego żądania na każdy przechwyt: backend Chloros sam uruchamia pętlę przechwytywania, zapisując jedną zsynchronizowaną grupę podczas wybierania kolejnej, a przycisk sprawdza jej liczbę. Jeśli na liście znajdują się kamery autonomiczne, Chloros przechwytuje dane pasmo po pasmie, tak jak dotychczas.

{% hint style="warning" %}
**Szybkie przechwytywanie w systemie Windows: należy wykluczyć backend Chloros z skanowania przez Microsoft Defender.** Skanowanie w czasie rzeczywistym każdego pliku zapisywanego podczas przechwytywania z dużą częstotliwością pochłania około 30% szybkości przechwytywania w testach MAPIR (setki plików na sekundę przy niewielkim obszarze zainteresowania). Instalator dla systemu Windows oferuje opcję **„Wyklucz backend Chloros a ze skanowania przez Microsoft Defender (znacznie szybsze przechwytywanie z kamery; zalecane)”**, domyślnie zaznaczoną, która usuwa to wykluczenie podczas odinstalowywania. Jeśli opcja ta została odznaczona, uruchom ponownie instalator z zaznaczoną opcją lub dodaj wykluczenie procesu Defender dla `chloros-backend.exe` (w folderze `backend` w folderze instalacyjnym Chloros). Gdy program Defender wykorzystuje jedno lub więcej rdzeni procesora podczas ciągłego przechwytywania obrazu z macierzy, backend Chloros odnotowuje to jednorazowo w swoim dzienniku, podając te same dwa sposoby rozwiązania problemu.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Wybór kamer i typów eksportu

Tekst pomocy w tym panelu podsumowuje to następująco: wybierz, które kamery i typy eksportu mają być generowane przez opcję „Capture All” — domyślnie wszystko jest włączone, a wybrane opcje są zapisywane wraz z tym projektem.

* Przyciski **Zaznacz wszystko / Odznacz wszystko** zmieniają stan pól wyboru dla wszystkich kamer jednocześnie.
* **Przełączniki typów eksportu zbiorczego**(dwa rzędy przycisków):**Wszystkie pliki RAW / Wszystkie z usuniętym efektem bayera / Wszystkie podglądy / Wszystkie pliki Radiance / Wszystkie pliki Reflectance / Wszystkie pliki indeksowe**. Każdy z nich ma trzy stany kolorystyczne: zielony ✓ = włączony dla każdej kamery, która to obsługuje, bursztynowy – = włączony dla niektórych, szary = brak. Przełącznik jest nieaktywny, gdy żadna podłączona kamera nie obsługuje danego typu. Wszystkie przełączniki są wyszarzone, gdy włączona jest opcja „Najszybsze przechwytywanie”.
* **Rzędy dla poszczególnych kamer**: pole wyboru „Włącz”, a także rozwijana (▸/▾) lista dostępnych typów eksportu dla danej kamery z indywidualnymi polami wyboru. W wierszu wyświetlana jest liczba włączonych opcji w formacie „4/6”.

### Typy eksportu i kamery, które je obsługują

Istnieje sześć typów eksportu: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. W wierszu każdej kamery pojawiają się tylko te typy, które są dla niej dostępne:

| Typ eksportu | Zawartość | RGB (FRGB) | Multispektralny Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mozaika Bayera (mono: pojedyncze pasmo) bezpośrednio z czujnika | ✓ | ✓ | ✓ |
| **Po usunięciu mozaiki** | Liniowe usuwanie mozaiki: z pliku surowego usunięto ciemne przesunięcie (DSNU) z pakietu kalibracyjnego, a następnie zastosowano demosaikowanie uwzględniające krawędzie — bez korekcji liniowości, płaskiego pola ani rozdzielania kolorów. Zapisane jako 3-kanałowy plik RGB TIFF (mono: 1-kanałowa skala szarości) | ✓ | ✓ | ✓ |
| **Podgląd** | Pełny łańcuch wyświetlania (balans bieli + gamma zgodnie z profilem aparatu, a także ustawienie „Highlights” aparatu RGB; wielospektralne: rozciągnięcie w fałszywych kolorach) | ✓ | ✓ | ✓ |
| **Promieniowanie** | float32 W/m²/sr/nm poprzez pełny łańcuch radiometryczny | — (niedostępne) | ✓ | ✓ |
| **Odbicie** | uint16 ρ (32768 = 1,0) | — (nie jest dostępne) | ✓ — wyświetlane tylko wtedy, gdy kamera posiada czujnik światła DAQ (własny lub odziedziczony po macierzy) | tak samo jak w przypadku wielospektralnym |
| **Indeks** | Renderowanie indeksu roślinności (LUT) | — | ✓ — wymaga włączonego, niepustego wyrażenia indeksu w kamerze i nie jest dostępne dla elementów macierzy połączonej (macierz posiada jeden wspólny indeks) | — (indeks wymaga ≥2 pasm; patrz [Kamery mono i wskaźniki wegetacji](mono-indices.md)) |

Promieniowanie i odbicie nigdy nie są dostępne dla kamer RGB — promieniowanie na piksel Bayera nie ma znaczenia w przypadku szerokopasmowego czujnika fotometrycznego.

### Najszybsze przechwytywanie

Przełącznik **⚡ Najszybsze przechwytywanie — tylko surowe dane**(pomarańczowy, gdy jest włączony) zastępuje wszystkie opcje eksportu ustawieniem**tylko surowe dane** — oraz bezpłatną kompozycję z indeksem połączonym dla macierzy — dzięki czemu klatka trafia do systemu tak szybko, jak to możliwe: obliczenia związane z promieniowaniem/odbiciem/wyświetlaniem są całkowicie pomijane w momencie przechwytywania.

{% hint style="info" %}
**Plik `.daq` jest nadal zapisywany.** Gdy przypisany jest czujnik światła, opcja „Najszybsze przechwytywanie” nadal zapisuje odczyt promieniowania padającego z czujnika DAQ (w folderze `daq/` plików przechwyconych) — dzięki czemu produkty dotyczące promieniowania, odbicia i indeksu można później wygenerować poprzez ponowne przetwarzanie (zobacz [Ponowne przetwarzanie ujęć](#re-processing-captures-into-calibrated-products)). Funkcja „Fastest Capture” nie powoduje utraty zaznaczeń w polach wyboru: po jej wyłączeniu wszystkie opcje powracają.
{% endhint %}

### Elementy sterujące dla poszczególnych układów

Każdy podłączony układ ma w panelu własną kartę grupy:

* **Pole wyboru „Włącz”** (trójstanowe dla wszystkich elementów) oraz nazwa układu wraz z trybem wyświetlania: „(połączone | oddzielne)”.
* Pole wyboru **Wyrównane**(domyślnie**włączone**): dostosowuje eksporty elementów do profilu wyrównania macierzy, dzięki czemu eksporty są zarejestrowane pikselowo między kamerami. Dane surowe pozostają nieprzekształcone, ale zawierają transformację w swoich metadanych. (Sam profil jest obliczany w [panelu ustawień macierzy](camera-settings.md#alignment-co-registration).) Pliki elementów macierzy są zapisywane w natywnej orientacji każdej kamery, a wszelkie ustawienia odbicia lustrzanego/obrotu są zapisywane w pliku, a nie stosowane.
* Wiersze kamer składowych są zagnieżdżone wewnątrz karty.

Karta macierzy zawiera również dwa rejestratory. Można je traktować jako **monitorowanie vs analiza**:

| Rejestrator | Poziom | Co rejestruje |
| --- | --- | --- |
| **● Nagrywaj wideo indeksowe / ■ Zatrzymaj nagrywanie** *(tylko macierze połączone)* | **Monitorowanie** | Kompozytowy obraz indeksowy na żywo w formacie wideo z prędkością 10 klatek na sekundę — 8-bitowy, rozdzielczość podglądu, wbudowana tablica LUT. Wymaga otwartego projektu i strumieniowego podglądu na żywo. Wyświetla klatki oraz czas, który upłynął podczas nagrywania. |
| **⦿ Nagrywaj serię surowych klatek / ■ Zatrzymaj serię surową** *(dowolna matryca)* | **Analiza**| Surowe klatki w formacie Bayera z częstotliwością przechwytywania na żywo (bez przetwarzania) wraz z manifestem dla każdej klatki oraz odczytami `.daq`, zapisywane do pliku `captures/bursts/`. Po zakończeniu serii zdjęć pojawia się przycisk**Utwórz wideo**: przetwarza on serię zdjęć w trybie offline na skalibrowane wideo — połączony indeks i/lub promieniowanie / odbicie / indeks dla każdej kamery — oraz opcjonalne pliki TIFF. Tworzenie połączonego indeksu rozpoczyna się automatycznie po zatrzymaniu serii zdjęć. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Przebieg

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

funkcji „Capture All”

Naciśnij przycisk **Capture All** na liście kamer w pasku bocznym:

1. Każda uwzględniona, widoczna i nie wstrzymana kamera rejestruje obraz zgodnie z wybranymi typami eksportu. **Układ kamer uruchamia się jako jeden zsynchronizowany wyzwalacz** (pojedyncza zsynchronizowana grupa obejmująca wszystkich członków — zobacz [Układy wielokamerowe](arrays.md)); kamery samodzielne rejestrują obraz indywidualnie.
2. Kamery ukryte (o symbolu oka) lub wstrzymane są pomijane. Układ jest w pełni zablokowany tylko wtedy, gdy *wszystkie* jego elementy są ukryte lub wstrzymane.
3. Za każdym razem, gdy przypisany jest czujnik światła, odpowiadający mu odczyt natężenia promieniowania padającego z czujnika DAQ jest zapisywany jako plik `.daq` w folderze `daq/` nagrań — nawet w przypadku nagrań zawierających wyłącznie dane surowe — dzięki czemu produkty radiometryczne można zawsze uzyskać później.
4. Przycisk pokazuje bieżący postęp — „Przechwytywanie… zakończone/łącznie” — a w trybie ciągłym/interwałowym zmienia się w **Stop (N)**. Każda pozycja przechwytywania ma limit czasu wynoszący 300 s.
5. Po zakończeniu przebiegu pojawia się komunikat z wynikiem **„Zapisano N plików”**lub**„Zapisano N, F nie powiodło się”**, a także „(S ukryte/wstrzymane/pominięte)”, jeśli pominięto kamery. Jeśli dysk się zapełni, proces zostanie zatrzymany i pojawi się komunikat**„Zatrzymano: dysk pełny. Zapisano N, F nie powiodło się”** — każdy plik, którego nie udało się zapisać, jest traktowany jako nieudany.

## Gdzie zapisywane są pliki przechwycone

Pliki przechwycone są zapisywane w ramach otwartego projektu w lokalizacji `<project>/captures/`. Każdy typ eksportu trafia do **własnego podfolderu**, dzięki czemu w przypadku przechwytywania wielopoziomowego typy nigdy się nie mieszają:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` to sygnatura czasowa przechwycenia, a `<serial>` to numer seryjny kamery. Pojedyncze przechwyty są nazywane `capture_<ts>_SN<serial>_<level>`; przechwyty macierzowe z jednego zsynchronizowanego wyzwalacza są nazywane `sync_<ts>_SN<serial>_<level>` i **mają jeden wspólny znacznik czasu dla wszystkich kamer w grupie** (sufiks poziomu jest pomijany, gdy kamera zapisuje tylko jeden poziom).
* **Jedna asymetria, o której warto wiedzieć:** poziom wyświetlania jest przechowywany w folderze o nazwie `preview/`, podczas gdy pliki zachowują w nazwie `_display` — folder i przyrostek różnią się tylko dla tego poziomu.
* Nieznane poziomy są umieszczane w folderze o własnej nazwie; jeśli nie można utworzyć podfolderu, plik jest zapisywany w katalogu głównym nagrań, zamiast zostać utracony.
* Pliki TIFF z nagraniami są domyślnie kompresowane bezstratnie (DEFLATE) i zawierają pełne metadane dotyczące kalibracji i przetwarzania **wewnątrz pliku XMP** — ujęcia są samopisujące się i nie zawierają żadnych plików towarzyszących poza plikiem `.daq` (oraz jego odpowiednikiem `.daq.cameras.json`).
* **Maski ograniczające (nowość w wersji 1.3.0).**W przypadkach, gdy proces rozdzielania spektralnego 3×3 wymagał ograniczenia pasma do wartości 0 w co najmniej jednym pikselu, produkty dotyczące promieniowania, odbicia i indeksu otrzymują maskę dla każdego piksela, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = pasmo w kanale R produktu, bit 1 = G, bit 2 = B); tag XMP produktu `Chloros:UnmixClampMask` wskazuje na tę maskę. Maski znajdują się**obok** folderów produktów, nigdy w ich wnętrzu, więc folder produktu, taki jak `reflectance/`, można przekazać do oprogramowania fotogrametrycznego w obecnej postaci. Własny moduł importu programu Chloros pomija maski.
* **Orientacja (1.3.0).** Produkty typu „Preview”, „Radiance”, „Reflectance” i „Index” z aparatu autonomicznego są obracane zgodnie z ustawieniem „Mirror/Rotation”; pliki „Raw” i „Debayered” pozostają w orientacji, w jakiej zostały odczytane przez czujnik. Wszystkie pliki wchodzące w skład tablicy pozostają w formacie natywnym. Każdy plik zapisuje to ustawienie w swoim pliku XMP, więc ponowne przetwarzanie stosuje je w ten sam sposób.
* **Do której kamery należy plik `.daq` (1.3.0).** Podczas przetwarzania identyfikator `.daq` danego ujęcia jest dopasowywany wyłącznie do obrazów pochodzących z kamer wymienionych w jego identyfikatorze `.daq.cameras.json`. Samodzielna kamera bez własnego czujnika światła nie korzysta zatem już z odczytów innej kamery — należy przypisać do niej czujnik lub zarejestrować go w zakładce „Czujniki światła”. Zapis z czujnika światła (oraz każdy `.daq` bez tego pliku, w tym starsze przechwyty) nadal obsługuje każdą kamerę.

Jest to ten sam układ, jaki pliki `chloros-cli lattice capture` / `array-capture` zapisują w katalogu `-o` — opisany w [Podręczniku CLI § Jak wygląda folder z przechwyconymi danymi](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ponowne przetwarzanie ujęć w skalibrowane produkty

Zarejestrowane surowe klatki wraz z zapisanym plikiem `.daq` to wszystko, czego potrzebuje potok przetwarzania — dlatego funkcja „Fastest Capture” jest bezpieczna do rzeczywistej pracy.

* **GUI**: dodaj folder z przechwyconymi obrazami do projektu ([Dodawanie plików do projektu](../processing-images-gui/adding-files-to-a-project.md)) i przetwarzaj jak zwykle.
* **Interfejs CLI**: wskaż plik `process` jako **katalog główny przechwyconych obrazów**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` zazwyczaj importuje tylko wskazany folder, ale jeśli nie zawiera on żadnych obrazów, a ma podfoldery, automatycznie przechodzi do nich — dzięki czemu podfoldery poszczególnych poziomów oraz folder `daq/` są pobierane za jednym razem. Każde przechwycenie jest importowane jako **pojedynczy obraz**, a pozostałe poziomy są dołączane jako tryby wyświetlania, a nie jako osobne obrazy dla każdego poziomu.

Bezpośrednie nazwanie podfolderu poziomu (np. `…/captures/raw/`) również działa, ale powoduje pominięcie folderu `daq/` — skopiuj go obok podczas-wyprowadzania produktu radiometrycznego z `raw/`, w przeciwnym razie nie będzie z czym dopasować sygnatury czasowej.

{% hint style="warning" %}
**Przetwarzanie zawsze rozpoczyna się od `raw`.** W ramach każdego przechwycenia źródłem potoku danych jest surowa klatka; pliki `debayered`, `radiance`, `reflectance`, oraz `preview` występują jako tryby wyświetlania, ale nigdy nie są ponownie wprowadzane do potoku — ponowne przetwarzanie produktu pochodnego spowodowałoby ponowne zastosowanie winietowania, koloru i obliczeń promieniowania już wbudowanych w jego piksele, więc Chloros rezygnuje z tego, zamiast przeprowadzać podwójne przetwarzanie. Rendery `index/` i `composite/` nie są w ogóle przetwarzane (są to pliki wyjściowe, a nie przechwycone). Folder „captures” zapisany **bez** importowanych danych surowych wyświetla się normalnie, ale `process` pomija go i informuje o tym; `--input-level {raw,debayered,processed}` jest celowym rozwiązaniem awaryjnym, które wymusza punkt wejścia. Dokładne komunikaty o pominięciu można znaleźć w [Podręczniku CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Dwie kolejne cechy, o których warto wiedzieć podczas tworzenia skryptów do ponownego przetwarzania:

* Uruchomienie `chloros-cli process`, które zażądało produktów, ale **nie zapisało żadnych produktów obrazowych**, kończy się wyraźnym błędem i wychodzi z wynikiem niezerowym** — nigdy nie otrzymasz cichego, pustego przebiegu. Pomyślne przebiegi zgłaszają liczbę produktów. (Celowe uruchomienie wyłącznie z metadanymi nadal liczy się jako sukces.)
* Ponownie zaimportowane, przetworzone eksporty nigdy nie zajmują miejsca surowego pliku przechwycenia — oryginalny plik surowy zawsze pozostaje źródłem potoku.

## Odpowiedniki w CLI



Wszystkie operacje opisane na tej stronie można wykonać w trybie bezinterfejsowym. Tryby przechwytywania w GUI odpowiadają bezpośrednio `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Pojedynczy | `chloros-cli lattice array-capture` |
| Ciągły | `array-capture --continuous [--count N] [--duration S]` |
| Interwał | `array-capture --interval S [--duration S]` |
| Najszybsze przechwytywanie | `array-capture --fastest` |
| Pole wyboru „Wyrównaj” | `--aligned / --no-aligned` |
| Pola wyboru typu eksportu | `--processing LEVEL` lub `--levels L1,L2,…` (domyślnie `all`) |
| Nagrywanie wideo indeksowego | `chloros-cli lattice array-record` |
| Nagrywanie serii surowych danych / Tworzenie wideo | `chloros-cli lattice array-burst` / `array-build-video` |

Pełne tabele flag, opcja przechwytywania z ustabilizowaną ekspozycją w trybie inteligentnej automatycznej ekspozycji (`--smart`) oraz model stałej częstotliwości zostały omówione w sekcji [Podręcznik CLI: § Tryby przechwytywania, rejestratory i przetwarzanie offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
