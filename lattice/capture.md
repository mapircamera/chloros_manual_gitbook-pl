# Ustawienia i tryby rejestracji

Rejestracja w zakładce „Kamery” odbywa się za pomocą jednego czerwonego przycisku **Capture All**oraz jednego panelu**Capture Settings**, który określa, jakie wyniki generuje ten przycisk: które kamery biorą udział w rejestracji, jakie typy plików eksportowych zapisuje każda kamera oraz czy migawka wyzwala się jednorazowo, w trybie ciągłym czy w określonych odstępach czasu. Na tej stronie opisano cały proces — konfigurację, sam proces przechwytywania, miejsce zapisywania plików na dysku oraz sposób ich późniejszego przetworzenia na skalibrowane produkty. Same elementy sterujące kamerami i układami kamer znajdują się w sekcji [Ustawienia kamer](camera-settings.md).

{% hint style="info" %}
**Do wykonania serii zdjęć wymagany jest otwarty projekt.** Opcja „Capture All” (Zrób wszystkie zdjęcia) oraz ikona koła zębatego „Capture Settings” (Ustawienia rejestracji) są nieaktywne, dopóki projekt nie zostanie otwarty („Utwórz lub otwórz projekt, aby zapisać zdjęcia”). Każda seria zdjęć jest zapisywana w folderze projektu w `captures/`.
{% endhint %}

## Panel „Ustawienia przechwytywania”

Można go otworzyć za pomocą **ikony koła zębatego obok opcji „Przechwyć wszystko”**na liście kamer w pasku bocznym lub przycisku**„Otwórz ustawienia przechwytywania…”** znajdującego się na dole dowolnego panelu ustawień danej kamery. W nagłówku widnieje napis „Ustawienia przechwytywania” wraz z przyciskiem ← (wstecz).

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Wybrane tutaj opcje — uwzględnione kamery, pola wyboru dla poszczególnych typów oraz tryb przechwytywania — są zapisywane **dla każdego projektu** i przywracane po ponownym otwarciu projektu.

### Tryby przechwytywania

Trzy przyciski trybów u góry panelu:

| Tryb | Jak działa | Ustawienia dodatkowe (domyślne) |
| --- | --- | --- |
| **Pojedynczy** *(domyślny)* | Jedno przechwycenie ze wszystkich wybranych kamer. | — |
| **Ciągły**| Przechwytywanie jeden po drugim aż do spełnienia warunku zatrzymania. | Zatrzymanie po**liczbie przechwyceń** (domyślnie 1) *lub* **czasie trwania przechwytywania** (domyślnie 10 s; jednostki: sekundy / minuty / godziny / dni). |
| **Interwał**(timelapse) | Seria zdjęć według timera. |**Liczba zdjęć / interwał**(domyślnie 1) ·**Co**N jednostek (domyślnie 5 s) ·**Przez** N jednostek (domyślnie 1 m). |

W trybie ciągłym lub interwałowym przycisk „Zrób wszystkie zdjęcia” zmienia się podczas działania w przycisk **Zatrzymaj (N)**, zliczając zdjęcia w miarę ich pojawiania się.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Wybór aparatów i typów eksportu

Tekst pomocy w tym panelu podsumowuje to następująco: wybierz, które kamery i typy eksportu ma generować opcja „Zrób wszystkie zdjęcia” — domyślnie wszystko jest włączone, a wybory są zapisywane wraz z tym projektem.

* Przyciski **Zaznacz wszystko / Odznacz wszystko** zmieniają stan pola wyboru „Włącz” dla wszystkich kamer jednocześnie.
* **Przełączniki typów eksportu zbiorczego**(dwa rzędy przycisków):**Wszystkie surowe / Wszystkie z usuniętym efektem Bayera / Wszystkie podglądy / Wszystkie promieniowania / Wszystkie odbicia / Wszystkie indeksowe**. Każdy z nich ma trzy stany kolorystyczne: zielony ✓ = włączony dla każdej kamery, która to obsługuje, bursztynowy – = włączony dla niektórych, szary = brak. Przełącznik jest nieaktywny, gdy żadna podłączona kamera nie obsługuje danego typu. Wszystkie stają się wyszarzone, gdy włączona jest opcja „Najszybsze przechwytywanie”.
* **Wiersze dla poszczególnych kamer**: pole wyboru „Włącz”, a także rozwijana (▸/▾) lista dostępnych dla danej kamery typów eksportu z indywidualnymi polami wyboru. W wierszu wyświetlana jest liczba włączonych opcji, np. „4/6”.

### Typy eksportu i aparaty, które je obsługują

Istnieje sześć typów eksportu: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. W wierszu każdego aparatu pojawiają się tylko te, które są dla niego dostępne:

| Typ eksportu | Zawartość | RGB (FRGB) | Multispektralny Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mozaika Bayera (mono: pojedyncze pasmo) prosto z czujnika | ✓ | ✓ | ✓ |
| **Debayered** | Liniowe usunięcie mozaiki (mono: 1-kanałowa skala szarości) | ✓ | ✓ | ✓ |
| **Podgląd** | Pełny łańcuch wyświetlania (balans bieli + gamma zgodnie z profilem aparatu; wielospektralne: rozciągnięcie w fałszywych kolorach) | ✓ | ✓ | ✓ |
| **Promieniowanie** | float32 W/m²/sr/nm poprzez pełny łańcuch radiometryczny | — (nieoferowane) | ✓ | ✓ |
| **Odbicie** | uint16 ρ (32768 = 1,0) | — (niedostępne) | ✓ — wyświetlane tylko wtedy, gdy kamera posiada czujnik światła DAQ (własny lub odziedziczony po macierzy) | tak samo jak w przypadku wielospektralnym |
| **Wskaźnik** | Renderowanie wskaźnika roślinności (LUT) | — | ✓ — wymaga włączonego, niepustego wyrażenia wskaźnika w kamerze i nie jest dostępne dla członków macierzy połączonej (macierz posiada jeden wspólny wskaźnik) | — (indeks wymaga co najmniej 2 pasm; zob. [Kamery monochromatyczne i wskaźniki wegetacji](mono-indices.md)) |

Promieniowanie i odbicie nigdy nie są dostępne dla kamer RGB — promieniowanie na piksel Bayera nie ma znaczenia w przypadku szerokopasmowego czujnika fotometrycznego.

### Najszybsze przechwytywanie

Przełącznik **⚡ Najszybsze przechwytywanie — tylko surowe dane**(pomarańczowy, gdy jest włączony) zastępuje wszystkie opcje eksportu ustawieniem**tylko surowe dane** — oraz bezpłatną kompozycję z indeksem łącznym dla układów — dzięki czemu klatka trafia do systemu tak szybko, jak to możliwe: obliczenia dotyczące promieniowania, odbicia i wyświetlania są całkowicie pomijane w momencie przechwytywania.

{% hint style="info" %}
**Plik `.daq` jest nadal zapisywany.** Gdy przypisany jest czujnik światła, funkcja „Najszybsze przechwytywanie” nadal zapisuje odczyt promieniowania padającego z czujnika DAQ obok klatek w formacie RAW — dzięki czemu produkty związane z promieniowaniem, odbiciem i indeksem można później utworzyć poprzez ponowne przetwarzanie (zobacz [Ponowne przetwarzanie przechwyconych danych](#re-processing-captures-into-calibrated-products)). Program Fastest Capture nie powoduje utraty ustawień zaznaczeń pól wyboru: po jego wyłączeniu powracają one do poprzedniego stanu.
{% endhint %}

### Elementy sterujące dla poszczególnych układów

Każdy podłączony układ ma w panelu własną kartę grupy:

* **Pole wyboru „Włącz”** (trójstanowe dla wszystkich elementów) oraz nazwa układu wraz z trybem wyświetlania: „(połączone | oddzielne)”.
* Pole wyboru **Wyrównane**(domyślnie**włączone**): dostosowuje eksporty elementów do profilu wyrównania macierzy, dzięki czemu eksporty są zarejestrowane pikselowo między kamerami. Dane surowe pozostają nieskorygowane, ale zawierają transformację w swoich metadanych. (Sam profil jest obliczany w [okienku ustawień macierzy](camera-settings.md#alignment-co-registration-combined-only).)
* Rzędy kamer składowych są zagnieżdżone wewnątrz karty.

Karta macierzy zawiera również dwa rejestratory. Można je traktować jako **monitorowanie vs analiza**:

| Rejestrator | Klasa | Co rejestruje |
| --- | --- | --- |
| **● Nagrywaj wideo indeksowe / ■ Zatrzymaj nagrywanie** *(tylko macierze połączone)* | **Monitorowanie** | Kompozycja indeksu połączonego na żywo do formatu wideo z prędkością 10 klatek na sekundę — 8-bitowa, rozdzielczość podglądu, wbudowana tablica LUT. Wymaga otwartego projektu i strumieniowego podglądu na żywo. Wyświetla klatki i czas, który upłynął podczas nagrywania. |
| **⦿ Nagrywaj serię surowych klatek / ■ Zatrzymaj serię surowych klatek** *(dowolna matryca)* | **Analiza**| Surowe klatki w formacie Bayera z częstotliwością przechwytywania na żywo (bez przetwarzania) wraz z manifestem dla każdej klatki oraz odczytami `.daq`, zapisywane w formacie `captures/bursts/`. Po zakończeniu serii klatek pojawia się przycisk**Utwórz wideo**: przetwarza on serię klatek w trybie offline na skalibrowane wideo — połączony indeks i/lub promieniowanie / współczynnik odbicia / indeks dla każdej kamery — oraz opcjonalne pliki TIFF. Tworzenie połączonego indeksu rozpoczyna się automatycznie po zatrzymaniu serii klatek.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Przebieg

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

funkcji „Capture All” Naciśnij przycisk **Capture All** na liście kamer w pasku bocznym:

1. Każda uwzględniona, widoczna i nieprzerwana kamera rejestruje obraz zgodnie z wybranymi typami eksportu. **Układ kamer uruchamia się jako jeden zsynchronizowany wyzwalacz** (pojedyncza zsynchronizowana grupa obejmująca wszystkich członków — zobacz [Układy wielokamerowe](arrays.md)); kamery samodzielne rejestrują obraz indywidualnie.
2. Kamery ukryte (oznaczone ikoną oka) lub wstrzymane są pomijane. Macierz jest w pełni zablokowana tylko wtedy, gdy *wszystkie* jej elementy są ukryte lub wstrzymane.
3. Za każdym razem, gdy przypisany jest czujnik światła, odpowiadający odczyt DAQ promieniowania padającego jest zapisywany jako plik `.daq` wraz z obrazami — nawet w przypadku nagrań wyłącznie w formacie surowym — dzięki czemu produkty radiometryczne można zawsze uzyskać później.
4. Przycisk pokazuje postęp na żywo — „Przechwytywanie… gotowe/łącznie” — a w trybie ciągłym/interwałowym zmienia się w **Stop (N)**. Każda pozycja przechwytywania ma limit czasu wynoszący 300 s.
5. Po zakończeniu przejścia pojawia się komunikat z wynikiem **„Zapisano N plików”**lub**„Zapisano N, F nie powiodło się”**, a także „(S ukryte/wstrzymane/pominięte)”, jeśli pominięto kamery.

## Gdzie zapisywane są przechwycone dane

Przechwycone dane są zapisywane w ramach otwartego projektu w lokalizacji `<project>/captures/`. Każdy typ eksportu trafia do **własnego podfolderu**, dzięki czemu w przypadku przechwytywania wielopoziomowego typy nigdy się nie mieszają:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` to sygnatura czasowa przechwycenia, a `<serial>` to numer seryjny kamery. Pojedyncze przechwyty mają nazwy typu `capture_<ts>_SN<serial>_<level>`; przechwyty macierzowe z jednego zsynchronizowanego wyzwalacza mają nazwy typu `sync_<ts>_SN<serial>_<level>` i **wspólny znacznik czasu dla wszystkich kamer w grupie** (sufiks poziomu jest pomijany, gdy kamera zapisuje tylko jeden poziom).
* **Jedna rzecz, o której warto wiedzieć:** poziom wyświetlania jest przechowywany w folderze o nazwie `preview/`, podczas gdy pliki zachowują w nazwie `_display` — folder i przyrostek różnią się tylko dla tego poziomu.
* Poziomy nieznane są umieszczane w folderze o własnej nazwie; jeśli nie można utworzyć podfolderu, plik jest zapisywany w katalogu głównym „captures”, zamiast zostać utracony.
* Pliki TIFF z ujęć są domyślnie kompresowane bezstratnie (DEFLATE) i zawierają pełne metadane dotyczące kalibracji i przetwarzania **wewnątrz pliku XMP** — ujęcia są samopisujące się i nie wymagają żadnych plików pomocniczych poza plikiem o nazwie `.daq`.

Jest to ten sam układ, w jakim pliki `chloros-cli lattice capture` / `array-capture` są zapisywane w katalogu `-o` — opisano to w [Podręczniku CLI, w sekcji „Jak wygląda folder z ujęciami”](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ponowne przetwarzanie ujęć w celu uzyskania skalibrowanych produktów

Zarejestrowane surowe klatki wraz z zapisanym plikiem `.daq` to wszystko, czego potrzebuje potok przetwarzania — właśnie dlatego tryb „Fastest Capture” jest bezpieczny do rzeczywistej pracy.

* **GUI**: dodaj folder z ujęciami do projektu ([Dodawanie plików do projektu](../processing-images-gui/adding-files-to-a-project.md)) i przetwarzaj jak zwykle.
* **CLI**: skieruj plik `process` na**katalog główny ujęć**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` zazwyczaj importuje tylko folder, który podasz, ale jeśli ten folder nie zawiera żadnych obrazów, a ma podfoldery, to automatycznie przechodzi do nich — dzięki czemu podfoldery poszczególnych poziomów oraz pliki w katalogu głównym `.daq` są pobierane za jednym razem. Każde przechwycenie jest importowane jako **pojedynczy obraz**, a pozostałe poziomy są do niego dołączone jako tryby wyświetlania, a nie jako osobny obraz dla każdego poziomu.

Bezpośrednie nazwanie podfolderu poziomu (np. `…/captures/raw/`) również działa, ale powoduje pominięcie plików głównych `.daq` — należy je skopiować razem z nimi podczas ponownego generowania produktu radiometrycznego z `raw/`, w przeciwnym razie nie będzie z czym dopasować znacznika czasu.

{% hint style="warning" %}
**Przetwarzanie zawsze rozpoczyna się od pliku `raw`.**W ramach każdego przechwycenia źródłem dla potoku danych jest surowa klatka; pliki `debayered`, `radiance`, `reflectance` oraz `preview` występują jako tryby wyświetlania, ale nigdy nie są ponownie wprowadzane do potoku — ponowne przetwarzanie produktu pochodnego spowodowałoby ponowne zastosowanie efektu winietowania, koloru i obliczeń promieniowania już wbudowanych w jego piksele, więc Chloros rezygnuje z przetwarzania, zamiast przeprowadzać podwójne przetwarzanie. Rendery `index/` i `composite/` nie są w ogóle przetwarzane (są to pliki wyjściowe, a nie przechwycone). Folder „captures” zapisany**bez** importowanych plików RAW wyświetla się normalnie, ale `process` pomija go i informuje o tym; `--input-level {raw,debayered,processed}` jest celowym „wyjściem awaryjnym”, które wymusza punkt wejścia. Dokładne komunikaty o pominięciu można znaleźć w [Podręczniku CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Dwie kolejne cechy, o których warto wiedzieć podczas tworzenia skryptów do ponownego przetwarzania:

* Przebieg `chloros-cli process`, który zażądał produktów, ale **nie zapisał żadnych produktów obrazowych**, kończy się wyraźnym błędem i zwraca wynik niezerowy** — nigdy nie otrzymasz cichego, pustego przebiegu. Pomyślne przebiegi zgłaszają liczbę produktów. (Celowy przebieg obejmujący wyłącznie metadane nadal liczy się jako sukces.)
* Ponownie zaimportowane przetworzone eksporty nigdy nie zajmują miejsca surowych danych z przechwycenia — oryginalne surowe dane zawsze pozostają źródłem potoku.

## Odpowiedniki CLI

Wszystkie operacje opisane na tej stronie można przeprowadzać w trybie bezinterfejsowym. Tryby przechwytywania w GUI odpowiadają bezpośrednio `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Pojedynczy | `chloros-cli lattice array-capture` |
| Ciągły | `array-capture --continuous [--count N] [--duration S]` |
| Interwał | `array-capture --interval S [--duration S]` |
| Najszybsze przechwytywanie | `array-capture --fastest` |
| Wyrównane pola wyboru | `--aligned / --no-aligned` |
| Pola wyboru typu eksportu | `--processing LEVEL` lub `--levels L1,L2,…` (domyślnie `all`) |
| Nagrywanie wideo indeksowego | `chloros-cli lattice array-record` |
| Nagrywanie serii zdjęć w formacie surowym / Tworzenie wideo | `chloros-cli lattice array-burst` / `array-build-video` |

Pełne tabele flag, opcja przechwytywania z ustabilizowaną ekspozycją w trybie Smart-AE (`--smart`) oraz model o stałej szybkości znajdują się w [CLI – Referencje § Tryby przechwytywania, rejestratory i przetwarzanie offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
