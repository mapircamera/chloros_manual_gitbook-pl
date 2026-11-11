# Dokumentacja Chloros CLI

**Wersja:**

1.3.0**Data utworzenia:**

29.07.2026, godz. 19:19 ·**Data aktualizacji:**

06.11.2026**Grupa docelowa:** Zoptymalizowane pod kątem wykorzystania przez LLM; zrozumiałe dla ludzi.**Zakres:** Wszystkie podpolecenia narzędzia `chloros-cli` przeznaczone dla użytkowników, wraz z opcjami i przykładami, które można skopiować i wkleić.

Niniejszy dokument stanowi kompletny przewodnik po narzędziu wiersza poleceń `chloros-cli`, dostarczanym wraz z MAPIR Chloros. Został on celowo opracowany w sposób wyczerpujący, tak aby model LLM (lub człowiek) mógł skomponować dowolny obsługiwany przepływ pracy na podstawie poniższych wykazów bez konieczności przeglądania kodu źródłowego.

Jeśli potrzebujesz tylko najważniejszych informacji, przejdź do:
- [Pięciominutowy przewodnik szybkiego startu](#five-minute-quickstart)
- [Przebieg pracy przy pierwszym podłączeniu kamery LATTICE](#lattice-camera-first-connect-workflow)
- [Przebieg pracy przy pierwszym podłączeniu czujnika DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Tryby przechwytywania, rejestratory i przetwarzanie offline](#capture-modes-recorders--offline-reprocess)

> **Sterowanie Chlorosem za pomocą asystenta AI?** Claude Desktop, Claude Code, Cursor i inne klienty MCP mogą sterować przetwarzaniem, kamerami i matrycami LATTICE oraz czujnikami światła DAQ za pośrednictwem `chloros-mcp`, serwer MCP instalowany wraz z Python SDK (oraz z pakietem .deb dla systemu Linux, w `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Konfiguracja, opcje i narzędzia znajdują się w dokumentacji SDK w sekcji [Serwer MCP (asystenci AI)](sdk-reference.md#mcp-server-ai-assistants). Asystent z powłoką może również bezpośrednio uruchamiać każde polecenie z tej strony.

> **Zmiany w wersji 1.3.0** (zauważalne przez istniejące skrypty lub przepływy pracy w wersji 1.2.0).
>
> **Zmiany, które skrypty w wersji 1.2.0 mogą wymagać uwzględnienia:**
>
> - **Obowiązuje `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` oraz `align-stream` zostały usunięte w wersji 1.2.0 i działały na podstawie własnych ustawień kamery. Teraz ustawia format presetu, ekspozycję, wzmocnienie i liczbę klatek na sekundę: `--preset high_speed` rejestruje obraz 8-bitowy z czasem ekspozycji 5 ms, wzmocnieniem 6 dB i 60 klatek na sekundę. `lattice cam-connect --preset` również zachowuje format ustawienia wstępnego (w wersji 1.2.0 po zastosowaniu ustawienia wstępnego kamera ustawiała 12 bitów). Zobacz [Opcje wspólne](#common-options-shared-by-most-subcommands).
> - **Format pikseli Bayera w kamerze M3M (mono) to jej głębia.** Ustawienia wstępne określają formaty Bayera, których czujnik mono nie może przyjąć: w wersji 1.2.0 kamera odrzuciła ten format i zachowała dotychczasowy, więc `--preset high_speed` nie był tam 8-bitowy. Obecnie jest to format Mono o tej samej głębi: `high_speed` i `--pixel-format BayerRG8` dają `Mono8`, a `high_quality`, `triggered` oraz `BayerRG12` dają `Mono12`. W mieszanej macierzy M3C + M3M każda kamera przyjmuje wspólny format w swojej rodzinie, a projekt `network-analysis` / `analyze-array --pixel-format` przypisują każdą kamerę do tej głębokości. To samo działa w drugą stronę: kamera M3C, dla której żądano formatu Mono, przyjmuje format Bayera dla tej głębokości, więc macierz, której główną kamerą jest M3M, nie otwiera już ponownie swoich kamer M3C w trybie Mono. Zobacz [Kamery jednopasmowe Mono (M3M)](#mono-m3m-single-band-cameras).
> - **Pliki eksportowe `lattice array-capture` są domyślnie wyrównane, podobnie jak pliki aplikacji.** Pliki `debayered`, `display`, `radiance`, `reflectance` oraz pliki indeksowe są dopasowywane do klatki kamery głównej i przycinane do obszaru nakładania się członków (mniejszy od matrycy o przesunięcia kamer, `XMP-mapir:AlignmentJSON` `"applied": true`), więc zapis z CLI to te same pliki, co zapis tej samej sceny z aplikacji Chloros; `raw` pozostaje w formacie natywnym. Wcześniej podczas przechwytywania bez `--aligned` każdy plik był zapisywany w rozmiarze natywnym, bez dopasowania. `--no-aligned` zachowuje tę funkcję. To samo domyślne ustawienie dotyczy plików `--smart`, `--continuous` oraz `--interval`, a także do SDK i `chloros-mcp`. Zobacz [tryby `array-capture`](#tryby-przechwytywania-tablic).
> - **`lattice array-capture --compression` domyślnie przyjmuje wartość `none`.** Pliki są zapisywane w postaci nieskompresowanej (około 6,3 MB na klatkę w pełnej rozdzielczości), co zapewnia najszybszą stałą szybkość zapisu; wersja 1.2.0 zapisywała w formacie `deflate` (około 4,1 MB). Aby uzyskać mniejsze pliki, należy podać `--compression deflate`. Oba formaty są bezstratne.
> - **Formaty PNG i JPEG obsługują wyłącznie produkty typu `display` / `preview` oraz pliki indeksowe.** Próba przechwycenia pliku `--format png` lub `jpg` z poziomem, którego plik 8-bitowy nie jest w stanie pomieścić, zostanie odrzucona przed rozpoczęciem jakiegokolwiek przechwytywania (kod HTTP 400, zakończenie `1`), podobnie jak domyślne `--processing all` z `--format png`/`jpg`. Zobacz [Poziomy eksportu przechwytywania](#capture-export-levels-the-all-default).
> - **`lattice dls` wymaga `--mac`.**Nie ma już domyślnego adresu MAC (był to adres jednej jednostki); `daq pool-discover --only ble` wyszukuje adres czujnika.**`lattice dls --camera SERIAL`** jest nowym elementem.
> - **Więcej błędów powoduje zakończenie działania `1`.** `process` pojawia się, gdy nie można zweryfikować eksportów danego przebiegu lub utracono obraz, mimo że pozostałe produkty zostały zapisane; polecenie przechwytywania z dowolnym plikiem, którego nie udało się zapisać (pozostałe pliki nadal są zapisywane); `lattice continuous` i `multi-stream` w przypadku zapełnionego dysku; `array-burst` / `array-record`, gdy rejestrator zgłasza błąd; `array-build-video`, gdy żądany produkt został pominięty lub nie został utworzony zgodnie z żądaniem; `project burst` / `stream`, gdy przechwytywanie nie powiodło się w trakcie; `project sensor log`, gdy nagranie nie zawiera żadnych odczytów; `lattice exposure --set`, `gain --set` i `configure --set` z wartością odrzucaną przez kamerę. `lattice network-analysis --json` kończy działanie, podobnie jak podsumowanie (`1` dla `needs_force_slip`). I odwrotnie: `daq pool-disconnect` czujnika, który nie znajduje się już w puli, kończy działanie `0`. Zobacz [Kody wyjścia](#exit-codes).
> - **Powolny proces `lattice array-connect` nigdy nie kończy się niepowodzeniem, gdy tablica jest podłączona.** Czeka do 600 s (180 s w wersji 1.2.0; `project connect` 90 s), wyświetlając na bieżąco etapy działania zaplecza, a połączenie, które nie zostanie nawiązane do tego czasu, jest cofane przez zaplecze — każda kamera zostaje przywrócona do stanu, w jakim została wykryta — więc błąd `Error:` (zakończenie `1`) oznacza, że macierz nie jest podłączona. W wersji 1.2.0 połączenie nawiązane zaraz po włączeniu zasilania kamer mogło trwać dłużej niż czas oczekiwania i doprowadzić do podłączenia matrycy po niepowodzeniu polecenia. Teraz klawisze Ctrl+C przerywają nawiązywanie połączenia. Zobacz [Połączenia trwałe](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` zachowuje pełny czujnik.** W przypadku braku opcji `--width` / `--height` / `--binning` każda kamera jest ustawiona na pełny czujnik, a liczba klatek na sekundę spada do poziomu, jaki zapewnia dany komputer i sieć; na liście wyników połączenia wyświetlane są szybsze opcje (binning lub mniejszy obszar zainteresowania) wraz z ich przewidywanymi częstotliwościami. Wersja 1.2.0 zmniejszała lub grupowała klatki, aby utrzymać wysoką częstotliwość. Zobacz [Opcje `array-connect`](#array-connect-options).
> - **Powolna sieć nie powoduje już odrzucenia macierzy.** Każda kamera jest dostosowywana do swojej przepustowości łącza, a liczba klatek na sekundę spada wraz z nią. W przypadku, gdy kamer nie da się dostosować do tak niskiej prędkości, macierz działa w trybie seryjnej emisji (`slip-emit-and-capture`): każda klatka dociera, ale kamery nie są naświetlane jednocześnie. `--force-tier sim-capture-sim-emit` wymaga jednoczesnej ekspozycji. Zobacz [Liczba klatek na sekundę w macierzy i model seryjny](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` określają, jakie ustawienia connect mają zastosowanie**: bez `--pixel-format` formaty ustalane przez „connect” (Mono12 / BayerRG12, wcześniej BayerRG8) oraz decyzja dotycząca [spakowanego 12-bitowego](#packed-12-bit-transport---packed) formatu. To samo polecenie może zatem podawać inną częstotliwość klatek niż w wersji 1.2.0.
> - **Ścieżki względne odnoszą się do lokalizacji, w której uruchamiane jest polecenie.** `-o` oraz inne ścieżki otwierane przez backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), `output` etapu receptury, oraz domyślne `output/` z `lattice index --live` są rozpoznawane względem folderu, z którego uruchamiasz `chloros-cli`. W wersji 1.2.0 backend rozpoznawał je w swoim własnym folderze instalacyjnym, co w przypadku zainstalowanej kompilacji kończyło się błędem „Odmowa dostępu”. Zobacz [Konwencje](#conventions).
> - **`lattice index` działa na backendzie w każdej postaci**, podobnie jak inne polecenia `lattice`, więc najpierw uruchom backend. Indeks offline (`--input`) jest obliczany przez backend, który odczytuje plik TIFF i zapisuje wyniki obok niego lub w `-o`, a polecenia `--list-presets` / `--list-gradients` wyświetlają tabele serwera. W systemie Windows działają one teraz: wcześniej interfejs Windows CLI odpowiadał na każdą formę `lattice index`, w tym `--live`, komunikatem `Error: lattice_sdk not available`. W systemie Linux indeks offline działał wcześniej w CLI bez modułu zaplecza i bez logowania; obecnie wymaga obu tych elementów, podobnie jak każde inne polecenie CLI. Zobacz [Indeks / Obliczenia dotyczące roślinności](#index--vegetation-maths).
> - **Nowe pliki obok produktów.** Każdy produkt dotyczący promieniowania, odbicia i indeksu, w którym proces rozdzielania pasm spowodował ograniczenie wartości piksela, otrzymuje maskę w folderze `clamp_masks/` obok folderu produktu oraz zawiera w wydruku wiersz `clamp mask:`. Zobacz [Maska ograniczenia rozdzielania pasm](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientacja kamery.** Nagranie z jednej kamery zapisuje pliki `raw` i `debayered` zgodnie z odczytem czujnika oraz rejestruje odwrócenie/obrót kamery w pliku XMP; plik `process` stosuje te informacje do tworzonych produktów. Pliki składowe tablicy pozostają w formacie natywnym, chyba że przekażesz nowy plik `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Zobacz [Orientacja aparatu](#jak-wygląda-folder-z-zdjęciami).
> - **Nazwa poziomu eksportu, która nie jest liczbą, zostanie odrzucona.** Lista `lattice capture --processing` oraz wszystkie listy `--levels` obejmują `raw`, `debayered`, `radiance`, `reflectance` lub `display` (lub alias, taki jak `preview`; `--processing` oraz `all`). Każda inna nazwa jest odrzucana przed dotknięciem kamery (400 HTTP `UNKNOWN_LEVEL`, wyjście `1`); 1.2.0 zapisał dla niej `raw` i zakończył działanie `0`.
> - **`daq pool-set-cap` wymaga `--cap-id`.** Bez tego polecenie kończy działanie `2`; wersja 1.2.0 usunęła korekcję osłony, pozostawiając goły czujnik. `--cap-id none` wymaga czujnika bez korekcji.
> - **`lattice resolution --binning-mode` został usunięty.** Binning jest zawsze ustawiony na średnią; flaga jest nadal akceptowana, wyświetla ostrzeżenie i nie powoduje żadnego działania. Zobacz [Dostrajanie czujnika](#sensor-tuning).
> - **`process --debayer texture-aware` jest odrzucane w przypadku przechwytywań LATTICE.** W przypadku folderu zawierającego zdjęcia LATTICE, `process` kończy działanie `1` po imporcie, podając liczbę zdjęć, zanim rozpocznie przetwarzanie. Zrzuty z LATTICE zawsze przechodzą standardową demosaikę, więc wersja 1.2.0 akceptowała tę opcję i ją ignorowała; aplikacja Chloros również nie oferuje jej dla tych zrzutów. Zobacz [`chloros-cli process`](#chloros-cli-process).
> - **Kamery powracają do stanu, w jakim zostały znalezione — chyba że zastosowano opcję `--keep-settings`.** `lattice array-disconnect`, każde polecenie przejściowe (`multi-capture` bez tablicy, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) oraz `lattice continuous` przywracają każdą kamerę do stanu, w jakim ją zastały: okablowanie wyzwalacza i linii synchronizacji, ustawienia PTP oraz wszystkie parametry kamery zmienione podczas sesji (ekspozycja i wzmocnienie, wartość docelowa AE, limity i tłumienie, balans bieli, poziom czerni, format pikseli, binning i obszar zainteresowania (ROI), odwrócenia, częstotliwość klatek). Natomiast `--keep-settings` zapisuje ustawienia każdej kameryw pamięci własnej kamery, dzięki czemu po włączeniu uruchamia się ona z tymi ustawieniami; `--clear-settings` resetuje tę pamięć do ustawień fabrycznych. W obu przypadkach kamery pozostają w trybie swobodnej pracy. Projekt ustawia kamery na podstawie własnych zapisanych ustawień po podłączeniu ich. Zobacz [Pamięć kamery](#camera-memory----keep-settings----clear-settings).
> - **Każdy tryb zapisuje projekt w ten sam sposób.** Każda zmiana wprowadzona w kamerze, macierzy lub czujniku światła podłączonym w ramach projektu — za pomocą aplikacji, interfejsu CLI, zestawu SDK lub serwera MCP — jest zapisywana w pliku `cameras.json` / `sensors.json`, a otwarcie projektu w dowolnym trybie powoduje skonfigurowanie urządzeń zgodnie z tymi ustawieniami. Wcześniej tylko aplikacja zapisywała projekt, a kolejne zapisanie nadpisywało wszystko inne. Zobacz [Jeden rekord projektu, każdy tryb](#one-project-record-every-mode).
> - **Pliki utworzone z danych z kilku kamer zawierają nazwy każdej z nich.** Stos wielopasmowy (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks na żywo utworzony z pasm kilku kamer oraz obraz `composite/` z przechwytywania tablicowego zawierają numery seryjne, modele i pakiety kalibracyjne każdej kamery źródłowej w postaci list (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), a listy EXIF `SerialNumber` zawierają numery seryjne połączone przecinkami. Wcześniej pliki te nie zawierały żadnych informacji o kamerach lub zawierały wyłącznie numer seryjny pierwszej kamery. Zobacz [Pliki utworzone z danych z kilku kamer](#files-made-from-several-cameras).
> - **Tylko plik `raw` zawiera tagi kalibracyjne kamery.** Zestaw zgodny ze standardem MicaSense (`BlackLevel`, `RadiometricCalibration`, plik dla poszczególnych kanałów `Band{R,G,B}RadiometricCalibration`, zestawy `MicaSenseVignetting*`) oraz ładunek `CalibrationJSON` opisują surowe wartości czujnika, więc znajdują się wyłącznie w pliku przechwytywania `raw`. Wszystkie pozostałe pliki — pliki przechwytywania `debayered`, `display`, `radiance`, `reflectance` oraz pliki indeksowe, ramki `lattice continuous` i `multi-stream`, oraz każdy plik produktu `process` — zawiera identyfikator kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), a podczas rejestracji — własny zapis klatki (`XMP-mapir:FrameJSON`: ekspozycja, wzmocnienie, temperatura, format pikseli, czas rejestracji). W wersji 1.2.0 zawierały one również zestaw surowych danych, który narzędzie stosujące je (Pix4D, Metashape) stosowało do pikseli, które nie są wartościami surowymi. `process` odnajduje kalibrację takiego pliku na podstawie jego numeru seryjnego oraz `CalBundleSha256`.
>
> **Inne zmiany:**
>
> - **Automatyczna częstotliwość klatek matrycy dostosowuje się do tego, co dostarcza matryca.** Częstotliwość, która traci ponad 1% swoich wyzwaleń, zostaje obniżona – ocena opiera się na kilku długich obserwacjach, a nie na jednej – a częstotliwość, z której została obniżona, jest od czasu do czasu ponownie testowana. Częstotliwość ustawiona ręcznie pozostaje niezmieniona.
> - **Automatyczna ekspozycja działa stabilniej.** Utrzymuje stały poziom naświetlenia, ustabilizuje się w ciągu kilku sekund po zmianie oświetlenia, ignoruje niewielkie prześwietlenia i przywraca działanie aparatu, który przestaje reagować w jasnym słońcu. Zobacz [Jak działa automatyczna ekspozycja](#how-auto-exposure-behaves).
> - **`process` zlicza wyłącznie Twoje zdjęcia i nagrania.** Ponowne uruchomienie na folderze zawierającym plik `camera_cal/` projektu nie zlicza już tych plików kalibracyjnych („Znaleziono 96 zdjęć” w zestawie zawierającym 90 zdjęć), a projekt, który już je uwzględnił, usuwa je.
> - **`lattice multi-test` przejmuje stan z momentu włączenia zasilania kamer.** Włącza protokół PTP kamer na czas działania, czeka na zsynchronizowanie i ustabilizowanie się ich zegarów przed pierwszym wyzwoleniem — każda kamera jest synchronizowana przez 120 s, czyli około 2 minut od momentu włączenia zasilania (`--ptp-settle SECONDS` ustawia ten czas), ponieważ zegar, który właśnie się zsynchronizował, ma odchylenie do ~200 us przez około 2 minuty — i przywraca PTP każdej kamery do stanu, w jakim go zastał. Nie czeka się ponownie na kamerę, która pozostała zsynchronizowana z wcześniejszego testu. Wcześniej pozostawiało PTP wyłączone w sieci z jednym przełącznikiem, w której żadne urządzenie DAQ-E nie odpowiadało, i odczytywało zegar swobodny każdej kamery jako zegar PTP, który nie mógł się zsynchronizować (`PTP is not locked`, przy każdym uruchomieniu). Zobacz [Wiele kamer (sesje przejściowe)](#multi-camera-transient-sessions).
> - **Zapis `.daq` wskazuje, czy nasadka została wybrana, czy też została założona domyślnie.** Urządzenia DAQ-U, DAQ-M i DAQ-E są dostarczane bez nasadki, a program Chloros nie jest w stanie stwierdzić, czy jest ona zamontowana, więc czujnik podłączony bez nazwanej nasadki otrzymuje domyślnie wartość `sunshine_cosine`, co jest *założeniem*. Każdy zapis Chlorosa (schemat 1.30) przechowuje teraz `cap_id_source` obok `cap_id`: `operator`, jeśli nadałeś nazwę nasadce (w tym `none`), oraz `auto_default`, jeśli była to nazwa domyślna. Podczas przetwarzania zapisu, w którym nasadka została domyślnie przypisana, a ustawienie nasadki DAQ w projekcie pozostawiono na „Auto”, powoduje wyświetlenie komunikatu `[daq-dls] WARNING` z nazwą pliku: sam DAQ-U bez korekcji nasłonecznienia wskazuje wartość około 30 razy wyższą. **Należy nadać nazwę dopasowanemu ogranicznikowi** podczas podłączania lub ustawić ogranicznik DAQ projektu, a ostrzeżenie zniknie.
> - **Koniec z okienkami konsoli pip w systemie Windows.** Na komputerze z zainstalowanym Pythonem backend i każdy z jego modułów przetwarzających obrazy uruchamiały kod błędu `pip` raz podczas startu systemu, a każdy z nich otwierał okno konsoli. Teraz już tego nie robią.
> - **Nagrania na żywo `.daq` przetrwały przerwę w zasilaniu.**Nagranie jest zapisywane wraz z dziennikiem i pełną synchronizacją dysku przy każdym zatwierdzeniu, za czujnikiem, więc powolny dysk opóźnia zapis pliku, ale nigdy nie wpływa na odczyty. Nagranie zatrzymane w trakcie zatwierdzania (przerwa w zasilaniu, zamknięcie procesu) może pozostawić obok siebie plik `<name>.daq-journal`:**skopiuj oba pliki razem.** Chloros przywraca dziennik przy pierwszym odczycie pliku, więc zapis otwiera się ze wszystkimi zmianami zatwierdzonymi przed zatrzymaniem (utracone są około ostatnich 3 s); w wersji 1.2.0 przerwa w zasilaniu w nieodpowiednim momencie mogła spowodować, że cały plik stał się nieczytelny („obraz dysku bazy danych jest uszkodzony”).
> - **Zapis na zajętym dysku nie powoduje już zawieszenia czujnika.** `daq pool-record` umieszcza odczyty w kolejce i zapisuje je w osobnym wątku; odczyt, którego nie udało się zapisać, jest zliczany (`daq pool-list` pokazuje `N write error(s)` z ostatnim błędem) zamiast powodować zawieszenie czujnika (jeden zapis trwał 8,4 s przy czterech kamerach zapisujących pliki TIFF).
> - **`lattice multi-capture` bez tablicy nie traci już klatek, które zapisał, gdy wygaśnie limit czasu wyzwalacza.** Wygaśnięcie czasu oczekiwania wyzwalacza kamery (`SC_ERR_TIMEOUT -1011`, zaobserwowane przy czterech kamerach podłączonych do jednego przełącznika 1 GbE) kończy sesję przy tej grupie klatek: grupy już zapisane są wyświetlane jako `Saved:`, następnie pojawia się błąd, wyjście `1`. W wersji 1.2.0 był to błąd HTTP500 bez listy, mimo że pliki znajdowały się na dysku.
> - **Kamery wyzwalane programowo nie pomijają już co drugiego wyzwolenia.** Kamera w trybie wyzwalania programowego umieszcza teraz w kolejce wyzwolenie, które pojawia się podczas jej odczytu (lub Chloros czeka, aż będzie gotowa), tak jak robiły to już kamery z synchronizacją kablową.
> - **`daq pool-discover` określa DAQ-E-S `DAQ-E-S`** w kolumnie `MODEL` (a `--json` w kolumnie `model`). Dla każdego czujnika Ethernetowego podawano `DAQ-E`; ten, który nie odpowiada na sprawdzanie tożsamości w ciągu około 2 s, nadal figuruje na liście jako `DAQ-E`.
> - **Ulepszone produkty indeksowe LATTICE** (NDVI i inne indeksy) z `process`, we wszystkich trybach przetwarzania.
> - **Kamery OCN: ulepszone produkty dotyczące promieniowania, odbicia i indeksów w świetle słonecznym.** Po rozdzieleniu pasm stosowana jest korekcja floty; kamery RGN pozostają bez zmian, a wartości OCN różnią się od tych z wersji 1.2.0. Dotyczy to również ujęć OCN wykonanych przy użyciu wcześniejszej wersji, jeśli są one przetwarzane na komputerze, na którym zapisana jest kalibracja użyta podczas wykonania ujęcia (program Chloros zachowuje kalibrację kamery po jej podłączeniu).
> - **`lattice continuous` rejestruje surową klatkę.** Uruchomienie programu `tiff` zapisało 3 klatki z demosaicingu-płaszczyznowe klatki, od których `process` nie może rozpocząć pracy. Jego klatki są teraz plikiem `raw`, który zapisuje przechwytywanie tablicowe: jedna płaszczyzna mozaiki z wartościami właściwymi dla czujnika, nieskompresowana, wraz z pakietem kalibracyjnym. Skrypt odczytujący trzy płaszczyzny z tych plików odczytuje tylko jedną.
>
> - **Każde przechwycenie zapisuje odczyt z modułu DAQ, z którego korzystało, podobnie jak robi to interfejs graficzny.** `lattice capture`, `lattice continuous` (po jednym pliku `.daq` na klatkę), `array-capture` oraz `multi-capture` z podłączoną tablicą (i SDK) zapisują plik `.daq` i jego `.match.json` za każdym razem, gdy DAQ jest powiązany lub umieszczony w puli, w tym podczas przechwytywania wyłącznie w formacie surowym; w wersji 1.2.0 zapisano przechwycenie surowe lub po usunięciu matrycy Bayesa, zapisywano tylko jeden plik z `--force-daq`. `--no-daq` wyłącza tę funkcję, `--daq-tolerance-ms MS` ustawia tolerancję dopasowania dla tego przechwycenia, a nowy `daq pool-bind` wiąże kamerę lub tablicę z modułem DAQ z poziomu CLI. W przypadku `.daq` przechwytywane dane są dopasowywane w `process` wyłącznie do kamer, które wymienia. Przechwycenie czeka teraz na następny odczyt z modułu DAQ, aby móc połączyć odczyty z obu stron ekspozycji. Zobacz [Odczyt z modułu DAQ zapisywany podczas przechwytywania](#the-daq-reading-a-capture-saves) oraz [Odczyt z modułu DAQ daleko od ekspozycji](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** dodaje indeks do projektu dla pasm każdej kamery, w tym nowego projektu. W wersji 1.2.0 nazwa, której projekt jeszcze nie zawierał, była ignorowana przez `Unknown index name(s) ignored`.
> - **`process` obok tablicy na żywo:**nie powoduje już ponownego uruchomienia backendu obsługującego kamery lub urządzenia DAQ i czeka do 15 s na zakończenie trwającego przebiegu.**Ctrl+C** zatrzymuje przebieg w backendzie, a nie tylko w interfejsie CLI. Zobacz [Uwagi dotyczące `process`](#notes).
> - **Folder projektu, którego ścieżka jest zbyt długa w stosunku do limitu 260 znaków w systemie Windows**, jest odrzucany od razu, a zamiast uruchomienia, które nic nie zapisuje, wybierany jest najdłuższy folder, który mieści się w limicie. Instalator dla systemu Windows oferuje teraz opcję „Włącz długie ścieżki w systemie Windows” oraz wykluczenie backendu z programu Microsoft Defender; obie opcje są domyślnie zaznaczone. Zobacz [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** uruchamia własną funkcję ciągłego przechwytywania modułu zaplecza: pasek postępu co 2 s, podsumowanie zamiast listy poszczególnych plików oraz kod wyjścia niezerowy w przypadku każdego niezapisanego pliku lub zapełnionego dysku. Zobacz [tryby `array-capture`](#array-capture-modes).
> - **Pierwszy `array-capture` po `array-connect`** czeka do 20 s na pierwszą grupę klatek macierzy zamiast zwracać błąd HTTP 503 „Macierz wciąż się rozgrzewa”.
> - **Ekspozycja matrycy w obszarze ROI:** mały obszar ROI nie ogranicza już ekspozycji elementu matrycy do okresu swobodnej pracy czujnika (4,9 ms przy rozdzielczości 768 × 304). Zobacz [Ekspozycja na elemencie matrycy](#exposure-on-an-array-member).
> - **Automatyczna ekspozycja nie obniża już automatycznej częstotliwości klatek matrycy.** Matryca zsynchronizowana kablem działa z częstotliwością, na jaką pozwalają sieć i odczyt z kamer; automatyczna ekspozycja wypełnia okres wynikający z tej częstotliwości, a następnie zwiększa wzmocnienie. Ręczna ekspozycja trwająca dłużej niż ten okres (`lattice exposure --set`, `configure --set`) spowalnia wyzwalanie, aby dostosować się do niej, zamiast być odrzucona, a częstotliwość wraca do normy, gdy element powraca do trybu automatycznego. Zobacz [Ekspozycja w elemencie macierzy](#exposure-on-an-array-member).
> - **Macierze generują mniej niekompletnych grup klatek** oraz rzadziej występują błędy 503 („Brak nowej zsynchronizowanej grupy klatek”): sygnał wyzwalający każdej kamery podrzędnej jest teraz oczyszczony z zakłóceń.
> - **Pakowany transport 12-bitowy** (`--packed`, domyślnie `auto`) zwiększa częstotliwość klatek w matrycach z pełnym czujnikiem o około jedną trzecią. Obecnie stosowany jest `array-connect --packet-size` (wersja 1.2.0 akceptowała go, ale ignorowała). Zobacz [Pakowany transport 12-bitowy](#packed-12-bit-transport---packed).
> - **Macierz, której kamery samoczynnie przywracają działanie po wyłączeniu i włączeniu zasilania.** Po wyłączeniu i włączeniu zasilania przełącznika macierz `array-connect` ponownie-otwiera kamery, gdy te odpowiadają, i wznawia transmisję po ponownym włączeniu PTP; w wersji 1.2.0 proces ten zatrzymywał się na stałe i wymagał nowego `array-connect`. `array-status` zgłasza `ptp_enabled: False` (oraz `ptp_off:`), gdy kamera odczytuje wyłączenie PTP, podczas gdy wcześniej zgłaszał ustawienie, z którym macierz była połączona. Zobacz [Połączenia trwałe](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` rejestruje dane przy zamkniętym interfejsie graficznym.** Gdy połączony widok interfejsu graficznego jest zamknięty, backend sam renderuje widok: matrycaNDVI macierzy lub indeks przekazany wraz z nowym `--expr`. Nagrywanie, które nie zapisuje żadnej klatki, kończy się teraz niepowodzeniem (`no-frames: …`, wyjście `1`). Wcześniej nagrywanie odbywało się tylko wtedy, gdy widok połączony interfejsu graficznego był otwarty, a nagrywanie, które niczego nie zapisywało, nadal powodowało zamknięcie programu `0`. Zobacz [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kamera, którą Chloros już posiada, nie jest otwierana po raz drugi.** Polecenie, które próbowałoby ją otworzyć, otrzymuje zamiast tego odpowiedź HTTP 409 `CAMERA_HELD`, wskazującą, kto ją posiada oraz polecenie, które ją zwalnia. Dotyczy to również kamery, z której w danej chwili korzysta inne polecenie (np. `multi-test` lub `multi-capture` otwierające swoje kamery, `lattice capture`): drugie polecenie zostanie odrzucone, dopóki pierwsze nie zakończy działania. Ponowne otwarcie może zatrzymać strumień posiadacza lub spowodować awarię zaplecza. Zobacz [Połączenia trwałe](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Przejściowe polecenia dotyczące wielu kamer dostarczają obraz z każdej kamery lub kończą się niepowodzeniem.** `lattice multi-capture` bez podłączonej matrycy, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` oraz `index --live` kończą działanie, a `1` podaje nazwę kamery, która nie dostarczyła żadnej klatki. Strumienie pomijają grupę klatek, w której brakuje kamery, i zatrzymują się po trzech kolejnych pominięciach; `multi-test` wyświetla wiersz na każdy sygnał wyzwalający, `(no frame)` wskazuje, gdzie kamera nie dostarczyła klatki; `align-stream` również kończy działanie `1` w przypadku błędu zadania. Następnie każde uruchomienie przywraca każdą kamerę do stanu, w jakim ją zastało: jej okablowanie wyzwalacza i linii synchronizacji, jej PTP (`PtpEnable`) oraz wszystkie ustawienia kamery zmienione podczas przebiegu — ekspozycję i wzmocnienie, cel AE, limity i tłumienie, balans bieli, poziom czerni, format pikseli, binning i ROI, odwrócenia, częstotliwość klatek — chyba że `--keep-settings` / `--clear-settings` (patrz [Pamięć kamery](#camera-memory----keep-settings----clear-settings)). Wcześniej polecenia te zachowywały wszystkie otrzymane klatki i kończyły działanie w trybie `0` (`align-stream` wyświetlał błąd zadania jako `Note:`), i pozostawiały każdą kamerę w trybie `TriggerMode` `On`. Ustawiały również ekspozycję każdej kamery na podstawie sceny sprzed zachowania klatki (maksymalnie kilka sekund), przy czym podłączona macierz uruchamia każdą kamerę z następującymi ustawieniami: 40 multispektralnych, 60 RGB (ustawia to `--brightness N`) i utrzymuje te wartości przez cały czas trwania sesji; tak samo postępuje pojedynczy moduł `lattice capture` kamery, której Chloros nie obsługuje. Wcześniej pierwsza klatka każdej kamery była rejestrowana przy własnym limicie ekspozycji kamery, wynoszącym około 100 ms, niezależnie od oświetlenia, i często ulegała obcięciu. Zobacz [Wielokamerowość (sesje przejściowe)](#multi-camera-transient-sessions).
> - **Przejściowe polecenia wielokamerowe współdzielą sieć w taki sam sposób jak `array-connect` i otwierają wyłącznie `--serials`.** Każda kamera jest dostosowywana do swojej części łącza hosta, więc cztery kamery działają na łączu 1 Gb/s; wcześniej wszystkie cztery żądały łącznie 125 MB/s, co w całości wyczerpywało przepustowość 1 GbE, a każda klatka traciła pakiety. Macierz, której sieć nie jest w stanie obsłużyć, jest odrzucana przed rozpoczęciem transmisji przez jakąkolwiek kamerę. `--serials` wybiera teraz kamery, przy czym pierwsza z nich uruchamia wyzwalacz; wcześniej każde polecenie otwierało wszystkie kamery w sieci, niezależnie od tego, co wskazywał `--serials`. Błąd polegający na utracie pakietów GVSP jest teraz tak oznaczany, zamiast wskazywać na kabel synchronizacyjny M8. Zobacz [Wiele kamer (sesje przejściowe)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` pozostawia kamery w trybie swobodnej pracy, zgodnie z ustaleniami.** `TriggerMode`, `Off` oraz wszystko inne zgodnie z ustaleniami podczas nawiązywania połączenia — ustawienia wyzwalacza i linii synchronizacji (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP oraz ustawienia kamery zmienione podczas sesji — dzięki czemu pojedynczepolecenie skierowane do kamery dotyczące byłego członka grupy pobiera wszystkie klatki naraz, a kamera główna nie pozostaje już ustawiona na `TriggerSource`, `Software` ani `--keep-settings`. Polecenie `--clear-settings` zapisuje ustawienia sesji w pamięci każdej kamery, a `--clear-settings` resetuje tę pamięć (zobacz [Pamięć kamery](#camera-memory----keep-settings----clear-settings)). `lattice continuous` ustawia również kamerę, którą wykryje jako uzbrojoną, w tryb swobodnego działania na czas trwania przebiegu (a po jego zakończeniu przywraca ją do poprzedniego stanu), a przebieg bez ramki kończy działanie `1`.
> - **`lattice configure --get` / `lattice exposure`** wyświetlają `note:` zrzeczywistą ekspozycją lub wzmocnieniem kamery podczas działania trybu automatycznego. Odrzucone polecenie `lattice exposure --set` / `configure --set` podaje zakres akceptowany przez kamerę i przywraca tryb automatyczny zamiast wyłączać automatyczną ekspozycję. Zobacz [Dostrajanie czujnika](#sensor-tuning).
> - **Nowe polecenia sterujące kamerą:**`lattice highlights` (zmniejsza jasność jasnych białych powierzchni w renderowaniu wyświetlacza RGB) oraz `lattice exposure --brightness-auto`.**`lattice multi-detect`** wykrywa kamery przewodowe podczas jednego cyklu sondowania i domyślnie ustawia kamerę RGB jako główną.
> - **Nowe polecenia DAQ:** `daq pool-bind`; `pool-enable-ptp`; oraz dla ustawień własnych urządzenia DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` oraz `pool-update-firmware`. Zobacz [Ustawienia urządzenia DAQ-E](#daq-e-device-settings).
> - **Urządzenie DAQ-E, które przestaje przesyłać dane, jest ponownie podłączane przez moduł zaplecza.** Gdy strumień danych z urządzenia DAQ-E należącego do puli przestaje działać (w wyniku wyłączenia i ponownego włączenia zasilania przełącznika, który je zasila, utrata połączenia z kamerą), serwer zaplecza ponownie łączy się z nim, przywracając poprzednie ustawienia i ponawiając próby w coraz dłuższych odstępach czasu, dopóki urządzenie pozostaje niedostępne. Nagrywanie na tym urządzeniu jest kontynuowane w tym samym pliku `.daq`, z zaznaczoną w nim przerwą, a plik `pool-record --stop` zawiera listę wszystkich przerw. W wersji 1.2.0 czujnik pozostawał nieaktywny do momentu wygenerowania plików `pool-disconnect` i `pool-connect`, co jednocześnie zakończyło nagrywanie. Zobacz [DAQ-E, który przestaje dostarczać dane](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** informuje, czy pakiety z serwera PTP grandmaster docierają do sieci (`health`), a w przypadku braku połączenia na stderr pojawia się komunikat `WARNING:`. Zobacz [`time-sync`](#chloros-cli-time-sync).
> - **Pakiet .deb dla systemu Linux:** zwiększa rozmiar buforów odbiorczych gniazd jądra, których potrzebują strumienie z kamer (przy domyślnych ustawieniach jądra dochodzi do utraty klatek i spadku liczby klatek na sekundę), dodaje usługę zaplecza działającą pod uprawnieniami użytkownika oraz sprawdza stan buforów za pomocą `lattice network`. CLI nigdy nie zatrzymuje backendu uruchomionego przez usługę, a aktualizacja ponownie uruchamia włączone usługi backendowe. Zobacz [Uruchamianie backendu jako usługi](#running-the-backend-as-a-service-headless) oraz [Hosty Linux — bufory odbiorcze gniazd](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` informuje, kiedy nie udało się wykonać wyrównania:**każda przyczyna, dla której przechwycenie zostało zapisane bez wyrównania (brak profilu wyrównania, elementy o różnych rozdzielczościach, tablica niesynchronizowana z kablem, ramka, na którą nie można zmapować profilu) jest wyświetlany jako wiersz `Alignment:`.**`chloros-cli daq help`** to `daq --help` (zakończyło działanie jako `2`), a w systemie Linux pomoc zawiera listę poleceń sprzętowych `daq` zawarte w tej kompilacji.
> - **Jetson:** pakiet to `chloros_1.3.0_arm64_jp6.deb`, przeznaczony dla JetPack 6, a przetwarzanie na GPU odbywa się na Orinie przy użyciu biblioteki CUDA PyTorch firmy JetPack. Zobacz [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konwencje

- Wszystkie polecenia mają prefiks `chloros-cli`. W systemie Windows plik binarny nosi nazwę `chloros-cli.exe`; w systemie Linux/Jetson jest to `chloros-cli`.
- Argumenty opcjonalne są przedstawione jako `--flag`. Wymagane argumenty pozycyjne są przedstawione bez nawiasów.
- Jeśli podano wartość domyślną, pominięcie flagi powoduje użycie tej wartości.
- Interfejs CLI jest lekkim klientem HTTP działającym na zapleczu Chloros (serwerze o nazwie `127.0.0.1:5000`, z którego korzysta również aplikacja desktopowa). **Które polecenia go uruchamiają:**
  - Podstawowe polecenia — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` oraz `selftest` — korzystają z uruchomionego backendu lub uruchamiają go, gdy żaden nie działa, a po zakończeniu pracy ponownie go zatrzymują.
  - **Polecenia `lattice`, `project` oraz `daq` nigdy nie uruchamiają serwisu.** Wymagają one już uruchomionego backendu: aplikacji desktopowej dla systemu Windows, [usługi backendowej](#running-the-backend-as-a-service-headless) dla systemu Linux lub backendu uruchomionego samodzielnie (zobacz [Uruchom backend](#start-the-backend)). W przypadku braku serwera wyświetlają komunikat `Error: Chloros backend not reachable at http://127.0.0.1:5000` wraz z poleceniem, które go uruchamia, a następnie kończą działanie `1`.
- Kod `CHLOROS_BACKEND_URL=<url>` wskazuje rodziny poleceń **`lattice`**,**`project`**i**`daq pool-*`** na inny backend (`lattice network-analysis` kieruje je zamiast tego do `--backend-url`). Polecenia podstawowe celowo przypisują `http://127.0.0.1:<port>` i ignorują je (literał IPv4 pozwala uniknąć kary w systemie Windows w postaci około 2 sekund na żądanie przy przekształcaniu `localhost` na `::1`). Zobacz [Zmienne środowiskowe](#environment-variables).
- **Ścieżki względne są rozpoznawane względem folderu, z którego uruchamiany jest program `chloros-cli`.** Backend to proces, który otwiera foldery wyjściowe i wejściowe, więc przed wysłaniem ścieżki interfejs CLI przekształca ją na ścieżkę bezwzględną: `-o` / `--output` (w tym jego domyślna wartość, `output`), `--input`, `--burst-dir`, `--profile`, `--save` oraz `--mask`, a także `output` (domyślnie `output`) w kroku [receptury](#recipe-dsl)oraz domyślny folder `output` o nazwie `lattice index --live`. W związku z tym `-o captures/` zapisuje do `<current folder>/captures/`, a wyświetlane ścieżki są bezwzględne. `~` jest rozwijany. W przypadku zdalnego backendu (`CHLOROS_BACKEND_URL` wskazującego inną maszynę) ścieżki są wysyłane w postaci wpisanej i rozpoznawane na tej maszynie.
- Do wszystkich wywołań SDK/CLI (uruchom `chloros-cli login` raz na każdym komputerze; dane są buforowane w `~/.chloros/`).
- Przykłady wykorzystują ścieżki systemu Linux; w systemie Windows należy zastąpić `/home/user/...` na `C:/Users/.../...`. Port szeregowy urządzenia DAQ-Uma postać `COM3` w systemie Windows oraz `/dev/ttyUSB0` / `/dev/ttyACM0` w systemie Linux.
- W opisach funkcji flagi wzajemnie się wykluczające zapisuje się jako `{--a | --b}`, a opcjonalne jako `[...]`.

---

## Składnia najwyższego poziomu

```
chloros-cli [global options] COMMAND [command options]
```

### Opcje globalne

| Flaga | Opis |
| --- | --- |
| `--backend-exe PATH` | Zastąpienie automatycznie wykrytego pliku wykonywalnego backendu. |
| `--port N` | Port HTTP backendu (domyślnie: `5000`). Używany wyłącznie przez polecenia podstawowe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); polecenia `lattice`, `project` i `daq` wykorzystują `CHLOROS_BACKEND_URL` (domyślny `http://127.0.0.1:5000`). |
| `-v, --verbose` | Włącz tryb szczegółowego wyświetlania. |
| `--restart` | Wymusza ponowne uruchomienie backendu (zamyka wszystkie uruchomione procesy `backend_server.py`). Backend uruchamiany przez usługę systemd (pakiet .deb dla systemu Linux) nigdy nie jest zatrzymywany: polecenie wyświetla wówczas komunikat `sudo systemctl restart <unit>` i kończy działanie `1`. |
| `--version` | Wyświetla wersję (`Chloros CLI 1.3.0`). |
| `--help` | Wyświetla pomoc ogólną. |

### Indeks poleceń

| Polecenie | Przeznaczenie |
| --- | --- |
| [`process`](#chloros-cli-process) | Przetwarzaj folder zawierający dane zebrane przez Survey3 lub LATTICE od początku do końca. |
| [`login`](#chloros-cli-login) | Uwierzytelnij to urządzenie za pomocą konta Chloros+. |
| [`logout`](#chloros-cli-logout) | Wyczyść dane uwierzytelniające z pamięci podręcznej. |
| [`status`](#chloros-cli-status) | Wyświetl aktualny stan licencji / uwierzytelnienia. |
| [`export-status`](#chloros-cli-export-status) | Wyświetl na żywo postęp eksportu Thread-4 podczas działania `process`. |
| [`language`](#chloros-cli-language) | Ustaw lub wyświetl listę języków wyświetlania CLI (obsługiwanych jest 38). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Domyślny folder projektu (wspólny z interfejsem graficznym). |
| [`update`](#chloros-cli-update) | Sprawdź dostępność aktualizacji Chloros i zainstaluj je (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostyka systemu + testy wstępne. |
| [`time-sync`](#chloros-cli-time-sync) | Status i sterowanie serwerem głównym PTP. |
| [`lattice`](#chloros-cli-lattice) | Sterowanie kamerą LATTICE i przechwytywanie obrazu (ponad 45 podpoleceń). |
| [`daq`](#chloros-cli-daq) | Sterowanie czujnikami spektralnymi DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otwieranie i uruchamianie zapisanego projektu Chloros (kamery + urządzenia DAQ). |

---

## Instalacja

`chloros-cli` jest dostarczany w ramach instalatora Chloros na każdą obsługiwana platformę — nie ma osobnego pliku do pobrania dla interfejsu CLI. Zainstalowanie pakietu platformowego dodaje `chloros-cli` do katalogu `PATH` wraz z aplikacją desktopową i plikiem binarnym zaplecza, który ona obsługuje.

Najnowsze pliki do pobrania: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Instalator zawiera również skrypty uruchamiające (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), które otwierają gotową do użycia powłokę CLI.

### System Windows (.exe)

1. Pobierz instalator dla systemu Windows z strony pobierania.
2. Uruchom plik `Chloros Installer 1.3.0.exe` i postępuj zgodnie z instrukcjami kreatora. Domyślna ścieżka instalacji to `C:\Program Files\Chloros\` (CLI znajduje się w `C:\Program Files\Chloros\cli\`, który instalator dodaje do zmiennej PATH).
3. Otwórz nowe okno terminala (`cmd.exe`, PowerShell lub Terminal Windows), aby zaktualizowany plik `PATH` został wykryty.

```powershell
chloros-cli --version
```

Instalator automatycznie dodaje `chloros-cli.exe` do systemowego katalogu `PATH` i dołącza sterowniki kamer LATTICE. Backendem jest `C:\Program Files\Chloros\backend\chloros-backend.exe` (folder `backend` w folderze instalacyjnym), którego nie ma w `PATH`.

**Ustawienia systemowe.** Strona instalatora *Wybierz dodatkowe zadania* zawiera dwie opcje w sekcji **Ustawienia systemowe**, obie**domyślnie zaznaczone**:

| Opcja (zgodnie z opisem w instalatorze) | Co robi | Wyłączenie instalacji w trybie cichym |
| --- | --- | --- |
| **Włącz długie ścieżki w systemie Windows** (pozwala na umieszczanie folderów projektów w lokalizacjach dłuższych niż 260 znaków; zalecane) | Ustawia wartość `LongPathsEnabled` dla całego komputera w systemie Windows na 1. System Windows jest dostarczany z tą opcją wyłączoną, w związku z czym żadna ścieżka nie może przekraczać 260 znaków; produkty Chloros znajdują się około 130 znaków poniżej folderu projektu, więc głębszy folder projektu zostanie odrzucony (zobacz [Rozwiązywanie problemów](#wskazówki-dotyczące-rozwiązywania-problemów)). Zmiana zaczyna obowiązywać dla programów uruchomionych po instalacji. Pozostaje włączona po odinstalowaniu, ponieważ inne programy mogą z niej korzystać. | `/MERGETASKS="!longpaths"` |
| **Wyklucz moduł zaplecza Chloros z skanowania przez Microsoft Defender**(znacznie szybsze przechwytywanie obrazu z kamery; zalecane) | Dodaje wykluczenie**procesu** Defender dla `chloros-backend.exe` w folderze instalacyjnym, dzięki czemu ochrona w czasie rzeczywistym nie skanuje każdego pliku zapisywanego przez moduł zaplecza, niezależnie od lokalizacji zapisu. Żaden inny program nie jest wykluczany; wykluczenie jest usuwane podczas odinstalowywania. Jeśli program Defender nie jest zainstalowany lub jest zablokowany przez zasady, ten krok kończy się bez komunikatu, a instalacja jest kontynuowana. | `/MERGETASKS="!defenderexclusion"` |**Dlaczego opcja Defender ma znaczenie:** przy wysokich częstotliwościach przechwytywania skanowanie w czasie rzeczywistym plików przechwyconych pochłania około 30% wydajności przechwytywania (zmierzono podczas ciągłego przechwytywania z 9 kamer: 77,7 przechwyceń/s pominiętych, 52,7 przeskanowanych). `lattice array-capture --continuous` wyświetla komunikat, gdy wykryje, że Defender wykorzystuje jedno lub więcej rdzeni procesora podczas przechwytywania; uruchom ponownie instalator z zaznaczoną tą opcją.

Instalacja w trybie cichym (`/SILENT` lub `/VERYSILENT`) stosuje obie opcje, chyba że zostaną one wyłączone, np. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Zapora systemu Windows.** Instalator zawsze dodaje dwie reguły przychodzące dla `chloros-backend.exe` w każdym profilu sieciowym, a deinstalator je usuwa: `Chloros Backend (UDP-In)` — dla strumieni z kamer, PTP, mDNS oraz strumień danych DAQ-E, a także `Chloros Backend (TCP-In)`, ograniczoną do lokalnej podsieci, służącą do aktualizacji oprogramowania sprzętowego DAQ-E, podczas której czujnik łączy się ponownie z tym komputerem w celu pobrania obrazu (patrz [Ustawienia urządzenia DAQ-E](#daq-e-device-settings)). HTTP API zaplecza pozostaje powiązane z `127.0.0.1`.

### Linux amd64 (.deb)

Dla Ubuntu 22.04 LTS lub nowszych wersji / stacji roboczych opartych na Debianie z architekturą x86_64.

> **Ubuntu 20.04 nie jest obsługiwany.** Lista zależności pakietu wynika z
> tego, z czym faktycznie łączy się backend, a obejmuje to `libc6 (>= 2.35)`;
> wersja focal zawiera glibc 2.31. `apt` odrzuca instalację, zamiast dopuścić do jej niepowodzenia w
> czasie wykonywania. Lista wymienia również `libgl1`, który jest ładowany przez dołączony pakiet OpenCV i którego może brakować w minimalnym
> obrazie serwera; `apt install` go dostarcza.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Plik .deb instaluje:
- `chloros-cli` do `/usr/bin/chloros-cli` oraz `chloros-backend` (uruchamia backend) do wersji `/usr/bin/chloros-backend`
- Skompilowany backend i interfejs CLI, rozpakowane, w plikach `/usr/lib/chloros/backend/` i `/usr/lib/chloros/cli/`
- Sterowniki kamer LATTICE
- Modele filtrów odszumiających, pakiety kalibracyjne oraz konfiguracja kanału aktualizacji
- Zestaw Python SDK dla systemu `python3` (`import chloros_sdk`). Pakiet `apt install` zawiera również swoje wymagania: `python3-numpy` i `python3-requests` (zalecane, ale nieobowiązkowe: powyższe programy są samowystarczalne; `dpkg -i` pomija je, a wynik instalacji wskazuje, czego brakuje). W środowisku virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, które zwiększają bufory odbiorcze gniazd jądra dla strumieni z kamery strumieni z kamery do 32 MB i stosuje je od razu (wartość już ustawiona na wyższy poziom pozostaje bez zmian). Usunięcie pakietu powoduje usunięcie tego pliku. Zobacz [Hosty Linux — bufory odbiorcze gniazd](#linux-hosts--socket-receive-buffers).
- Dwie opcjonalne usługi systemd dla backendu, z których żadna nie jest włączona (następna sekcja)

#### Uruchamianie backendu jako usługi (bez interfejsu graficznego)

Bez usługi backend (a wraz z nim grandmaster PTP) działa tylko wtedy, gdy coś go uruchomiło: `chloros-backend` w terminalu, skrypt SDK lub podstawowe polecenie `chloros-cli` przez cały czas jego działania. Aby backend działał nieprzerwanie na komputerze bez interfejsu graficznego, włącz usługę **jako swój użytkownik**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` uruchamia backend jako ten użytkownik. Zapisy, które tworzy, należą do niego, a jego szczegółowy dziennik znajduje się w pliku `~/.cache/chloros/logs`, i korzysta z jego `chloros-cli login` (`~/.chloros`), więc najpierw zaloguj się raz jako ten użytkownik. PTP nie wymaga uprawnień roota. Podobnie jak każdy zainstalowany backend, nasłuchuje wyłącznie na porcie `127.0.0.1:5000`.

`chloros-backend.service` uruchamia moduł zaplecza jako **root**. Jest on nadal zainstalowany, aby maszyna, na której został włączony, mogła nadal działać, ale wszystko, co zapisuje backend jako root, należy do użytkownika root, więc użytkownik nie może usunąć własnych przechwyconych danych. Aby przełączyć się na swojego użytkownika:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Szczegółowy dziennik usługi root znajduje się w pliku `/var/log/chloros`, do którego dostęp mają członkowie grupy `adm` (usługa ustawia `CHLOROS_LOG_DIR`; zobacz [Zmienne środowiskowe](#environment-variables)). Podczas instalacji pakietu wyświetla się polecenie `enable` z wpisanym imieniem użytkownika oraz opcją przełączania, gdy usługa root jest włączona.

**Włącz jedną jednostkę zaplecza.** W danym momencie działa tylko jeden backend. Druga włączona jednostka (`chloros-backend.service` obok `chloros-backend@<user>.service` lub dwie jednostki użytkowników `@`) nie uruchamia się i pozostaje w stanie `failed` ze statusem zakończenia 98; systemd nie uruchamia jej ponownie. Jej `systemctl status` wyjaśnia dlaczego: `Another instance is already running. Exiting.`, a następnie co blokuje backend (port 5000 lub plik blokady i użytkownik, do którego należy), a następnie `systemd will not restart <unit> (exit status 98)`. Po ponownym uruchomieniu komputera może działać dowolna z tych jednostek. Aby dokonać zmiany, wyłącz tę, której nie używasz, zgodnie z powyższym opisem.**CLI nigdy nie zatrzymuje backendu usługi.** `chloros-cli` korzysta z niego w obecnej postaci, tak jak robią to GUI i SDK: `process` działa na nim, ładując swój własny projekt w miejsce poprzedniego. W sytuacji, w której CLI w innym przypadku zrestartowałoby backend (`--restart` lub `process`, którego import nie znalazł niczego), podaje nazwę jednostki, wyświetla `sudo systemctl restart <unit>` i kończy działanie `1`, ponieważ backend zatrzymany w ten sposób pozostałby wyłączony. `GET /api/status` nadaje jednostce nazwę `service` (`null` w przypadku backendu, na którym nie działa żadna usługa).**Aktualizacja i usunięcie.** Aktualizacja pakietu zatrzymuje wszystkie usługi backendowe, a następnie ponownie uruchamia każdą z nich, która jest włączona, zaczynając od jednostek `@<user>` (jednostka uruchomiona ręcznie, ale nigdy nie włączona, pozostaje zatrzymana, tak jak po ponownym uruchomieniu systemu). Jeśli włączonych jest więcej niż jedna usługa, działa tylko jedna, a pozostałe pozostają w stanie awarii, jak opisano powyżej; podczas aktualizacji wyświetlane są komunikaty `Chloros: N backend units are enabled: …` i `Keep one enabled: sudo systemctl disable --now <unit>`. Usunięcie pakietu powoduje zatrzymanie i wyłączenie tych usług. Oczyszczenie katalogu powoduje również usunięcie pliku `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ten sam układ co w przypadku pakietu .deb dla architektury amd64, zawiera bufory odbiorcze oraz [usługi zaplecza](#running-the-backend-as-a-service-headless), przeznaczony dla Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Zawarty w nim PyTorch to kompilacja CUDA z JetPack 6 dla Orin, więc przetwarzanie na GPU, w tym debayering z uwzględnieniem tekstur, odbywa się na procesorze graficznym Jetsona. Pakiet nie zawiera samej biblioteki CUDA: korzysta z bibliotek CUDA i cuDNN zainstalowanych przez JetPack 6 (`sudo apt install nvidia-jetpack` na obrazie, na którym ich brakuje). W przypadku braku dostępnego procesora graficznego z obsługą CUDA przetwarzanie odbywa się na procesorze centralnym, a funkcja uwzględniająca tekstury korzysta ze standardowego algorytmu debayera (zobacz [`process`](#chloros-cli-process)).

### Jednorazowe uwierzytelnianie na każdym urządzeniu

Każda platforma wymaga jednorazowego logowania do Chloros+, zanim wywołania SDK lub CLI zaczną działać:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Dane uwierzytelniające są buforowane w `~/.chloros/user_session.json`.

### Uruchomienie backendu

Polecenia `lattice`, `project` i `daq` komunikują się z już uruchomionym backendem; nigdy go nie uruchamiają (zobacz [Konwencje](#conventions)). Podczas korzystania z nich należy utrzymywać jeden z nich w stanie uruchomionym:

| Platforma | Sposób |
| --- | --- |
| Windows | Otwórz aplikację Chloros na pulpicie (jej backend obsługuje również interfejs CLI) lub uruchom polecenie `"C:\Program Files\Chloros\backend\chloros-backend.exe"` w osobnym terminalu (w folderze `backend` w folderze instalacyjnym) i pozostaw je otwarte. |
| Linux / Jetson | Uruchom `chloros-backend` w osobnym terminalu lub utrzymuj jeden proces zawsze uruchomiony za pomocą [usługi backendowej](#running-the-backend-as-a-service-headless). |

W danym momencie działa tylko jeden backend; wszyscy klienci (interfejs graficzny, CLI, skrypty SDK) korzystają z niego wspólnie. Podstawowe polecenia (`process`, `login`, `status`, …) również z niego korzystają i uruchamiają go, gdy żaden nie jest aktywny.

**Backend działa w tle między poleceniami**, więc kolejne polecenie nie uruchamia go ponownie. Zatrzymuje się po wylogowaniu (`chloros-cli logout`, chyba że aplikacja Chloros jest otwarta lub trwa uruchomienie lub kompilacja offline), po uruchomieniu polecenia `chloros-cli backend stop` lub samoczynnie po 30 minutach bez aktywności: brak polecenia, skryptu lub aplikacji, brak uruchomionego zadania lub kompilacji offline, brak podłączonej kamery lub czujnika światła. Polecenie `backend stop` nie zostanie wykonane, gdy trwa zadanie lub kompilacja offline, podając nazwę tego zadania; polecenie `--force` zatrzymuje je natychmiast, a zadanie lub kompilacja zostają utracone (seria jest zachowywana, a `array-build-video` tworzy ją ponownie). Ustaw `CHLOROS_BACKEND_IDLE_MIN` przed uruchomieniem backendu, aby zmienić limit 30 minut (`0`: nigdy); [usługa backendu](#running-the-backend-as-a-service-headless) nigdy nie zatrzymuje się samodzielnie. `chloros-cli backend status` pokazuje, czy któryś z nich jest uruchomiony, co z niego korzysta oraz czy przebiega jakaś kompilacja w trybie offline. `process` otwiera projekt w uruchomionym backendzie, chyba że aplikacja Chloros jest otwarta z projektem, który otworzyła: w takim przypadku operacja nie powiedzie się i należy najpierw wykonać ją z poziomu aplikacji lub zamknąć ten projekt. Projekt pozostawiony otwartym przez poprzednią instancję `process` lub skrypt SDK jest po prostu zastępowany.

### Sprawdź instalację

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` zamyka się `0` dopiero po pomyślnym przejściu wszystkich siedmiu testów. Ostatni z nich wymaga procesora graficznego CUDA (procesora graficznego NVIDIA, lub Jetson Orin) oraz modeli odszumiających, więc na komputerze bez takiego procesora wyświetla komunikat `Failed: CUDA + Denoiser` i kończy działanie `1`; wszystko oprócz algorytmu debayera z obsługą tekstur, działającego wyłącznie na procesorze graficznym, nadal tam działa.

> **Wymagana subskrypcja Chloros+.**Interfejs CLI wymaga aktywnego planu Chloros+.**Copper**to podstawowy poziom Chloros+ — każdy płatny poziom Chloros+ zapewnia dostęp do CLI/SDK; jedynie bezpłatny poziom**Iron** go nie zapewnia. (Mapowanie identyfikatorów planów: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Przejdź na wyższy plan pod adresem [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Ten limit jest egzekwowany przez backend, a nie tylko przez CLI: żądanie z flagą SDK/CLI bez opłaconego planu jest odrzucane z kodem `403 PLAN_UPGRADE_REQUIRED`, niezależnie od tego, czy pochodzi z `chloros-cli`, Python SDK, czy z ręcznie napisanego klienta HTTP. Użytkownik, który się wylogował, otrzymuje zamiast tego kod błędu `401 AUTH_REQUIRED`. Dostęp działa w trybie offline przez okres karencji planu (30 dni w przypadku planu miesięcznego, do momentu wygaśnięcia w przypadku planu rocznego) i zostaje wstrzymany po jego upływie; kod `chloros-cli status` nadal działa, dzięki czemu przyczyna jest widoczna (jest to jedyna trasa SDK/CLI zwolniona z ograniczeń poziomu — `GET /api/license-status`).

---

## Pięciominutowy przewodnik szybkiego startu

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (krok 2) uruchamia backend dla własnego przebiegu, gdy żaden nie jest uruchomiony, i zatrzymuje go po zakończeniu, więc uruchom go od kroku 4. Zobacz [Uruchomienie backendu](#start-the-backend).

---

## `chloros-cli process`

Przetwórz folder zawierający obrazy poprzez pełny potok Chlorosa (wykrywanie obiektów → kalibracja → winieta → odbicie → eksport indeksu).

### Składnia

```
chloros-cli process INPUT [OPTIONS]
```

### Argumenty pozycyjne

| Argument | Opis |
| --- | --- |
| `INPUT` | Ścieżka do folderu wejściowego zawierającego pliki `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) lub `.dng`. |

### Opcje ogólne

| Flaga | Wartość domyślna | Opis |
| --- | --- | --- |
| `-o, --output PATH` | nowy folder o nazwie `-n` w domyślnym folderze projektu | Folder projektu. Projekt otrzymuje nazwę na jego podstawie, a folder, w którym znajduje się już plik `project.json`, otrzymuje plik `_1` / `_2` zamiast zostać nadpisany. Domyślnym folderem projektu jest ten wybrany przez `set-project-folder` (lub interfejs graficzny), w przeciwnym razie `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (znacznik czasu) | Nazwa projektu, używana wyłącznie **bez** `-o` (w przypadku `-o` projekt przyjmuje nazwę folderu). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` wykorzystuje algorytm demosaicingu oparty na sieci neuronowej Chloros+ dla obrazów Survey3; działa wolniej, ale zapewnia wyższą jakość. Jest przeznaczony wyłącznie dla Survey3: Zrzuty z LATTICE zawsze wykorzystują standardowy algorytm demosaic, więc w przypadku folderu zawierającego zrzuty z LATTICE `process` odrzuca `texture-aware` po zaimportowaniu, wyświetlając komunikat o błędzie z informacją o liczbie plików, a następnie zamyka `1` przed rozpoczęciem jakiegokolwiek przetwarzania (aplikacja Chloros również nie oferuje dla nich opcji Texture Aware). Wymaga to procesora graficznego z obsługą CUDA (procesor graficzny NVIDIA w systemach Windows / Linux x64, procesor graficzny Orin w Jetsonie): bez niego proces wykorzystuje `standard`, a backend rejestruje jeden wiersz `[DEBAYER] … needs a CUDA GPU …` wyjaśniający przyczynę (pokazany wraz z `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Korekcja winietowania. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibracja odbicia (wykorzystuje panel wzorcowy, jeśli zostanie znaleziony, lub kalibrację seryjną NIST dla LATTICE). W przypadku danych wielospektralnych LATTICE pełni to również funkcję przełącznika **produktu** odbicia — zobacz [Przełączniki eksportu dla poszczególnych produktów](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | wyłączone | Zastosuj korekcje PPK GNSS z plików sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | brak | Model kamery podłączony do pinu ekspozycji nr 1, w celu synchronizacji czasowej PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | brak | Model kamery podłączony do pinu ekspozycji nr 2, służący do synchronizacji czasowej PPK. |
| `--recal-interval SECONDS` | `0` | Minimalny interwał rekalibracji w sekundach (*Minimalny interwał rekalibracji* projektu). |
| `--timezone-offset HOURS` | `0` | Różnica strefy czasowej czujnika światła względem czasu UTC, w godzinach (parametr projektu *Różnica strefy czasowej czujnika światła*). |
| `--format FORMAT` | `TIFF (16-bit)` | Jedna z wartości: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Program Chloros nigdy nie zwiększa głębi bitowej obrazu: w przypadku obrazów całkowicie 8-bitowych format o większej głębi jest zapisywany jako `TIFF (8-bit)`, a informacja o tym pojawia się tylko raz. W projekcie łączącym ujęcia 8-bitowe oraz 12- i 16-bitowe każde ujęcie jest eksportowane z własną głębią lub niższą. Jasność jest zawsze 32-bitowa zmiennoprzecinkowa. |
| `--indices NAME [NAME ...]` | brak | Wskaźniki wegetacji według nazwy presetu (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), bez rozróżniania wielkości liter. Każdy z nich jest dodawany do projektu dla pasm danej kamery, w tym nowego projektu. Nazwa, która nie jest ustawieniem wstępnym lub której pasma kamery nie mogą utworzyć (np. `NDRE` wymaga pasm RE i NIR, których żadna pojedyncza kamera nie zapewnia), jest wymieniona w ostrzeżeniu i nie otrzymuje obrazu indeksowego. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Wymuś punkt wejścia potoku dla plików LATTICE TIFF (nie ma to wpływu na pliki Survey3.raw). Jest to również „drzwi awaryjne”, które pozwalają na przetworzenie serii zdjęć **bez plików RAW** — zobacz [Jak wygląda folder z seriami zdjęć](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | wł. | Generuje produkt po liniowym usunięciu efektu Bayera (`Debayered_Images`). Zobacz [Przełączniki eksportu dla poszczególnych produktów](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | włączone | Generuje podgląd wyświetlania (`Preview_Images`): RGB = balans bieli (źródło światła DAQ, jeśli jest dostępne, w przeciwnym razie „gray-world”) + gamma; multispektralne = rozciągnięcie w fałszywych kolorach. |
| `--radiance / --no-radiance` | włączone | Wyświetla promieniowanie typu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Odniesienie dla produktu odbicia LATTICE: `auto` = docelowa wartość w ramce, która przeszła kontrolę jakości (QA), stanowi odniesienie absolutne (docelowa wartość z flagą QA konkuruje z DAQ pod względem wyniku pomiaru), rezerwowa wartość DAQ dla promieniowania padającego (ρ = π·L/E); `target` = ścisłe (bez zastępowania przez DAQ); `daq` = DAQ jako źródło ostateczne. Zobacz [Przełączniki eksportu dla poszczególnych produktów](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | brak | Katalog skanów **zmierzonej** docelowej reflektancji dla poszczególnych jednostek (`<serial>.csv`); w przypadku braku danych stosuje się nominalne widma T3/T4P. |
| `--array-alignment / --no-array-alignment` | włączone | Macierze LATTICE: zastosowanie wyrównania między modułami, oznaczonego w pliku XMP `Chloros:Alignment*` każdego ujęcia, do każdego przetworzonego produktu (usunięcie matrycy Bayer / podgląd / promieniowanie / odbicie / indeks). Brak operacji dla obrazów bez tych tagów. |
| `--array-alignment-crop / --no-array-alignment-crop` | przycinanie | Przycinaj wyrównane eksporty do wspólnego obszaru nakładania się macierzy, tak aby wszystkie moduły miały ten sam obszar pokrycia; `--no-…` zachowuje pełny obszar czujnika (czarne wypełnienie poza źródłem). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Próbkowanie ponowne w celu korekcji zniekształceń wynikających z wyrównania. `nearest` zachowuje dokładne wartości DN źródła (bez mieszania wartości radiometrycznych między pikselami). |

### Opcje wykrywania celów

| Flaga | Opis |
| --- | --- |
| `--min-target-size PIXELS` | Minimalny rozmiar celu kalibracyjnego w pikselach dla detektora. |
| `--target-clustering 0-100` | Procentowe skupienie celów (domyślnie `60`). |
| `--target / --targets` | Wyszukiwanie celów kalibracyjnych wyłącznie w podfolderze `target` lub `targets` pliku wejściowego (szybsze). |

### Przykłady

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Przełączniki eksportu dla poszczególnych produktów (LATTICE multispektralny)

Przetwarzanie LATTICE rozgałęzia się na **każdy odpowiedni produkt w jednym przebiegu**. Cztery przełączniki dla poszczególnych typów — `--debayered`, `--preview`, `--radiance`, `--reflectance` — są**domyślnie włączone**; użyj formularza `--no-<type>`, aby wyłączyć jeden z nich. Kamery główne RGB zawsze wysyłają jedynie dane po usunięciu efektu Bayesa + podgląd (bez promieniowania/odbicia dla poszczególnych pasm), więc `--radiance`/`--reflectance` nie mają dla nich żadnego działania. Przełączniki są ignorowane w przypadku `.raw` w Survey3 (który działa zgodnie ze standardową ścieżką odbicia/celu). *(Stara flaga `--radiometric-output {reflectance,radiance,sensor-response}` została **usunięta** i zastąpiona tymi przełącznikami; nie ma już poziomu `sensor-response`.)*

| Produkt | Wyjście | Czy wymagane jest zbieranie danych DAQ w kierunku w dół? |
| --- | --- | --- |
| `--debayered` | Liniowa demosaika (`Debayered_Images`): ten sam plik, jaki zapisuje przechwytywanie na swoim poziomie `debayered` — odchylenie ciemności (DSNU) z zestawu kalibracyjnego od danych surowych, a następnie demosaikowanie z uwzględnieniem krawędzi za pomocą OpenCV; brak tablicy LUT liniowości, płaskiego pola ani demiksowania. | Nie. |
| `--preview` | Podgląd na ekranie (`Preview_Images`): RGB = WB + gamma; multispektralny = rozciągnięcie w fałszywych kolorach. | Nie. |
| `--radiance` | float32 W/m²/sr/nm z pełnego łańcucha radiometrycznego (`Radiance_Images`). | Nr |
| `--reflectance` | uint16 współczynnik odbicia ρ (`32768` = 1,0), Gotowe do Pix4D. | **Tak**, chyba że jest zakotwiczone przez obiekt w kadrze, który przeszedł kontrolę jakości (patrz poniżej). |

`--reflectance-source` wybiera odniesienie odbicia:**`auto`**(domyślnie) ustanawia cel w kadrze, który przeszedł kontrolę jakości, jako**odniesienie absolutne**— łańcuchy linii empirycznych zakotwiczone na celach są porównywane krzyżowo na panelach wykluczonych z analizy, a zmierzony zwycięski wynik jest stosowany pasmo po pasmie: pasmo, którego DAQ nie może skalibrować lub którego dopasowanie nachylenia DAQ nie powiodło się, przyjmuje linię właściwą dla danego celu (`ReflectanceElBandMethods` określa, który) — cel, który przeszedł kontrolę jakości z flagą (pominięty panel, pasmo niesprawne), również konkuruje z podziałem DAQ na podstawie tego zmierzonego wyniku, a klatka, z którą przegrywa, jest oznaczana jako `ReflectanceChain=daq` — w przypadku braku celu lub niepowodzenia kontroli jakości stosuje się domyślny próg selekcji DAQ (ρ = π·L/E);**`target`**jest opcją ścisłą (bez zastępowania przez DAQ);**`daq`**rezygnuje na rzecz zachowania opartego na autorytecie DAQ. Geometria celu (ArUco / stały obszar zainteresowania / pasek) pochodzi z konfiguracji celu projektu; `--target-reflectance-dir DIR` przechowuje**zmierzone** skanów (`<serial>.csv`) wyszukiwanych na podstawie numeru seryjnego/kodu QR jednostki docelowej, z nominalnymi widmami T3/T4P jako rozwiązaniem awaryjnym.

Ścieżka odbicia DAQ automatycznie rozpoznaje **promieniowanie opadające dopasowane czasowo**na podstawie zarejestrowanego pliku**`.daq`**(DAQ-U/M/E)**lub natywnego pliku `.csv`**z DAQ-M, znalezionego wraz z obrazami. Jeśli pakiet kalibracyjny dla danej kamery lub urządzenia DAQ nie jest przechowywany lokalnie w pamięci podręcznej, potok**automatycznie pobiera go z AWS** przy pierwszym użyciu (wymaga jednorazowego połączenia z internetem; zapisywany w pamięci podręcznej pod nazwą `~/.chloros/`).**Czyj to odczyt?**Odczyt `.daq` zapisany podczas przechwytywania jest odczytem kamery (kamer), którą opisuje — nazwanej w pliku pomocniczym `<file>.daq.cameras.json` znajdującym się obok niego (`lattice capture`, `multi-capture`, `array-capture` oraz interfejs graficzny; surowa seria podaje odczyt każdego elementu w swoim manifeście) — a `process` dopasowuje go**wyłącznie do obrazów z tych kamer**. Samodzielna kamera bez własnego odczytu nie korzysta zatem już z modułu DAQ innej kamery; należy przypisać do niej moduł DAQ lub wykonać nagranie. Własne nagranie modułu DAQ (`daq pool-record` lub nagranie rozpoczęte w GUI), jak i każdy plik `.daq` bez tego pliku towarzyszącego (w tym starsze przechwyty), nadal obsługuje każdą kamerę.

#### Odczyt pikseli odbicia (Pix4D / Metashape / własne skrypty)

Odbicie jest przechowywane jako całkowita liczba DN, a **wartość DN oznaczająca ρ = 1,0 zależy od kamery źródłowej**:

| Źródło | ρ = 1,0 to | Jak to sprawdzić |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (z zapasem do ρ 2,0) | Plik zawiera znacznik XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (ograniczone do ρ 1,0) | Brak tagów XMP `Chloros:*` — ten brak *jest* sygnałem. |

**Odczytaj `Chloros:PixelScale` i podziel przez tę wartość** zamiast zakładać stałą wartość. Tag jest zdefiniowany w domenie uint16, więc pozostaje niezmienny jako `32768` we wszystkich formatach wyjściowych, które zmieniają skalę — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` i `TIFF (32-bit, Percent)` są samopisujące (należy najpierw znormalizować zapisany typ danych z powrotem do uint16: ×257 z 8-bitowego, ×65535 z typu float).

> **Pliki ze starszych kompilacji mogą nie zawierać skali.** 8-bitowy plik TIFF z danymi odbicia jest przeskalowywany i oznaczany jako `Chloros:PixelScale`, podobnie jak pozostałe. Kompilacje sprzed wersji 1.3.0 *przycinały* ją zamiast tego do zakresu 0..255, więc każda wartość powyżej ρ≈0,008 była spłaszczana do 255, a pomijano zarówno `Chloros:PixelScale`, jak i krotkę `MicaSense:RadiometricCalibration`. **Jeśli w pliku odbicia LATTICE brakuje tego tagu, nie należy zakładać skali — należy go ponownie przetworzyć** zamiast dzielić piksele, które nigdy nie były podzielne.

#### Odczytywanie pikseli indeksowych (wskaźnik NDVI i inne wskaźniki wegetacyjne)

**Raster indeksu całkowitoliczbowego to rozciągnięcie kontrastu dla poszczególnych obrazów, a nie mapa w zakresie od −1 do 1.** Produkty indeksowe `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` oraz `JPG (8-bit)` — w tym 8-bitowy plik TIFF, w którym zapisano 8-bitowy obraz — przechowują

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

gdzie `IndexMin` / `IndexMax` to **najniższa i najwyższa skończona wartość indeksu tej klatki**. DN 0 nie jest równe −1, a pełna skala nie jest równa +1.

Program Chloros oznacza oba punkty końcowe, dzięki czemu odwzorowanie pozostaje odwracalne:

| Tag | Znaczenie |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — zgodnie z poniższą konwencją |
| `Chloros:IndexMin` | wartość indeksu zapisana jako DN 0 |
| `Chloros:IndexMax` | wartość indeksu zapisana jako pełna skala |

Należy je odczytać i odwrócić, normalizując zapisany DN do zakresu 0…1 najpierw względem maksymalnej wartości typu danych (dtype), tak aby jedno wyrażenie obejmowało każdy format liczb całkowitych:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Odczyt nie wymaga `-config pix4d.config` — exiftool odczytuje te właściwości XMP samodzielnie. Konfiguracja jest wymagana wyłącznie do ich *zapisu*.

> **`IndexMin == IndexMax`** oznacza, że każdy piksel tej klatki miał tę samą wartość indeksu. Raster jest wypełniony w połowie skali i nie ma czym dzielić — należy przyjąć wartość końcową.

> **Brak tagów `Chloros:Index*` oznacza brak określonego mapowania.** Eksport LUT to renderowanie z kolorami, którego piksele mają kolory gradientowe przypisane do okna progowego LUT, a klatka bez piksela o skończonym indeksie nie ma zakresu do określenia. W żadnym z tych przypadków nie należy zakładać istnienia mapowania — obowiązuje ta sama zasada, co w przypadku braku tagu `PixelScale`.**Ponieważ rozciągnięcie dotyczy poszczególnych obrazów, dwie klatki z jednego lotu nie są porównywalne pod względem DN**— każda z nich ma własne punkty końcowe, więc ten sam wskaźnik NDVI odpowiada różnym wartościom DN na różnych obrazach. W przypadku analiz ilościowych obejmujących cały lot (statystyki strefowe, stałe progi, szeregi czasowe) należy wyeksportować**`TIFF (32-bit, Percent)`**: rastrowe dane indeksowe są tam zapisywane jako wartości typu float32**samych wartości indeksu**, bez konieczności przeskalowywania i bez tagów. Pomimo nazwy formatu produkty indeksowe nie są pomnożone przez 100 — skalowanie to dotyczy wyłącznie odbicia o typie danych całkowitoliczbowym.

#### Maska ograniczenia rozdzielania (które piksele zostały ograniczone w danym paśmie)

Spektralne rozdzielanie 3×3 może spowodować, że pasmo uzyska wartość ujemną — tam, gdzie światło ledwo zawiera to pasmo lub gdzie scena wykracza poza zakres opisany przez macierz rozdzielania kamery. Nie jest to wartość rzeczywista, więc podczas eksportu jest ona ustalana na 0 — a podgląd na żywo pokazuje tę samą wartość 0. Pasmo może być ograniczone na kilku pikselach lub na prawie wszystkich; maska rejestruje obie te sytuacje. `Chloros:Band{R,G,B}UnmixClampFraction` określa, jaka część klatki została objęta tym zjawiskiem; maska ograniczenia wskazuje **które piksele**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— w folderze `clamp_masks/`**obok** folderu produktu, nigdy w jego wnętrzu (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; eksport z `process` to `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — dla każdego produktu dotyczącego promieniowania, odbicia i wskaźnika, w którym co najmniej jeden piksel został ograniczony: zarówno pliki przechwycone, jak i eksportowane jako `process` (oraz pliki TIFF dla poszczególnych klatek o nazwie `array-build-video --save-tiffs`). Jednopasmowe pliki `uint8`, na własnej siatce pikselowej produktu (wraz z orientacją, obszarem zainteresowania (ROI), binningiem i wyrównaniem tablicy). Każda maska określa nazwę kamery produktu (`Make` / `Model` / numer seryjny), czas przechwycenia oraz tagi obiektywu, a także wymienia pasma, które reprezentują jej bity (`XMP-Camera:BandName`, najpierw bit 0); jej `ImageDescription` i `Software` oznaczają ją jako maskę ograniczającą, a nie zawiera ona kalibracji.
- **Bity**: bit 0 = pasmo w kanale R produktu (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Ustawiony bit oznacza, że wartość piksela w tym paśmie została ograniczona do 0 — należy traktować ten bit oraz wszelkie wskaźniki na nim oparte jako mało wiarygodne.
- **`Chloros:UnmixClampMask`** jest umieszczany na każdym produkcie, na którym przeprowadzono rozdzielanie: ścieżka do maski względem folderu produktu (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, zawsze oddzielona przez `/` — należy połączyć ją z folderem obrazu, aby otworzyć maskę), lub `none`, gdy żaden piksel nie został ograniczony. Brak oznaczenia oznacza, że produkt nigdy nie przeszedł procesu unmix (kamery RGB lub monochromatyczne, poziomy surowe/po usunięciu matrycy).
- `lattice capture` / `array-capture` zapisują ścieżkę maski pod nazwą zapisanego pliku (`clamp mask: …`), a lista wyników przechwytywania wymienia ją jako `clamp_mask` klatki.
- `lattice index --live --save-multiband` zapisuje jedną maskę na każdą kamerę źródłową dla stosu wielopasmowego — zobacz [`lattice index` — maski stosu wielopasmowego](#lattice-index--multiband-stack-masks). Materiał wideo nie posiada maski: `array-build-video` zamiast tego podaje ułamki ograniczenia dla każdej klatki (zobacz [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Ponieważ żaden folder produktu nie zawiera maski na żadnym poziomie, folder produktu (`radiance/`, `reflectance/`, folder `*_Images/` z eksportu `process`) można przekazać do Pix4D, Metashape lub innego oprogramowania fotogrametrycznego w obecnej postaci — wraz z podfolderami — a oprogramowanie to akceptuje wyłącznie zdjęcia. Przekaż mu folder produktu, a nie folder znajdujący się nad nim (ten zawiera również `clamp_masks/` oraz wszystkie inne typy eksportu). Własna funkcja importu Chloros pomija maski. Odczyt maski: `(mask & 1) != 0` oznacza „pasmo kanału R zostało tutaj ograniczone”, `& 2` – G, `& 4` – B.

#### Dane EXIF przenoszone do eksportu

`process` kopiuje **blok GPS i jego ExifIFD** z oryginalnego zdjęcia do każdego produktu, więc
eksport zawiera `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` oraz
`CameraSerialNumber` wraz z georeferencjami.

**`FocalLength` nie jest opcjonalny w fotogrametrii.** Pix4D oblicza odległość próbkowania naziemnego na podstawie
ogniskowej i wysokości; w przypadku braku tego tagu program stosuje skrajnie błędną skalę. Podczas jednego
lotu nad gajem pomarańczowym, obejmującego 49 ujęć, brak tego tagu spowodował, że obszar o wymiarach 411 m × 160 m został zrekonstruowany jako
47,8 km × 13 km — ortofotomapę o rozdzielczości 455 MP składającą się głównie z obszarów bez danych, co następnie zostało zinterpretowane jako problem z układaniem kafelków i
problem z plikiem BigTIFF, zanim ktokolwiek sprawdził GSD. Jeśli Twoja ortofotomapa ma nieprawdopodobną
skalę, najpierw uruchom `exiftool -FocalLength` na wyeksportowanym produkcie.

Kopia ta celowo **nie** jest `-all:all`: tagi strukturalne pliku IFD0'tagi strukturalne IFD0' zakłócają wynik LATTICE po
skopiowaniu, a pliki `ExifImageWidth` / `ExifImageHeight` są wykluczone, ponieważ opisują
*źródłowe* ujęcie — eksport, którego rozmiar kiedykolwiek został zmieniony, zawierałby w przeciwnym razie wymiary
sprzeczne z własną siatką. Plik XMP jest zapisywany bezpośrednio, a nie kopiowany, ponieważ ExifTool
usuwają tagi XMP z tego samego wywołania podczas kopiowania bloku XMP (co spowodowałoby utratę tagów kalibracji MAPIR).

### Gdzie trafiają pliki wyjściowe

Pliki wyjściowe są zapisywane **w folderze projektu, pogrupowane według aparatu, a następnie według formatu pliku**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folder aparatu to `LATT-<sensor>-<lens>-F<filter>` dla LATTICE (zgodny z tagami EXIF
`Model`), a dla aparatu Survey3 — `<model>_<filter>` — dwa aparaty korzystające z tego samego czujnika i filtra, ale różniące się
obiektywem, mają oddzielne drzewa katalogów, ponieważ różnią się winietowaniem, polem widzenia i zniekształceniami. Format
folderu jest następujący: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32` dla
`TIFF (32-bit, Percent)`.

> **Każdy wyeksportowany plik zachowuje nazwę pliku ŹRÓDŁOWEGO.** Eksport danych promieniowania z
> `capture_…_raw.tif` nadal nosi nazwę `capture_…_raw.tif` — po prostu znajduje się w
> `tiff32/Radiance_Images/`. **To folder identyfikuje produkt, a nie nazwa pliku**, więc wyszukiwanie
> pliku `*radiance*.tif` nie da żadnego wyniku; zamiast tego należy dopasować katalog.

### Nagrania z czujnika światła — skalibrowane `.daq` + `.csv`

`process` obsługuje również zapisy `.daq` znajdujące się w folderze wejściowym i **nie**
potrzebuje do tego żadnych obrazów: samodzielnie wysłany DAQ-U / DAQ-M / DAQ-E stanowi kompletny
zestaw danych, a folder zawierający wyłącznie pliki `.daq` jest prawidłowym danymi wejściowymi.

Rejestrator DAQ może być używany **bez** kalibracji — tak właśnie działają domyślnie publiczne
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): zapisują surowe odczyty czujników i oznaczają plik tak, aby
program Chloros pobrał fabryczną kalibrację tego czujnika **przez port szeregowy** (najpierw z lokalnej pamięci podręcznej,
a następnie z chmury MAPIR) i ją stosuje. `process` zapisuje wynik z powrotem:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Plik `.csv` zawiera jeden wiersz na każdy odczyt: sygnaturę czasową UTC, czas integracji, moc całkowitą,
luksy fotopowe/skotopowe, PPFD (oraz jego podział na niebieski/zielony/czerwony), długość fali szczytowej, a następnie
pełne widmo na własnej siatce długości fal czujnika. Plik `.daq` jest ponownie importowany bez konieczności
ponownej kalibracji.

`<name>` to nazwa pliku danego zapisu. Dwa zapisy w różnych folderach mogą dzielić
jedne — odczyty dla poszczególnych ujęć (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) zapisane w
tej samej milisekundzie przez dwie kamery — a każdy z nich nadal otrzymuje własne produkty: pierwszy zachowuje
`<name>`, a następny dodaje foldery, które go odróżniają **po** nazwie
(`<name>__<capture>_daq_calibrated.daq`), więc data i godzina nagrania nadal znajdują się na początku.

W przypadku powodzenia przebieg generuje raport `Light-sensor products written: N (calibrated .daq + .csv)`.
Tekst w nawiasie opisuje to, co faktycznie zostało zapisane, więc brzmi on
`(RAW COUNTS — this sensor has no calibration bundle)` w przypadku czujnika bez pakietu oraz
`(N calibrated, M raw counts)` w przypadku folderu zawierającego oba elementy. Własne nagłówki zaplecza
`[DAQ-EXPORT]` i `[RUN-SUMMARY]` czerpią swoje sformułowania w ten sam sposób — żaden z
tych trzech nie może nazwać surowego eksportu skalibrowanym.

Zapis DAQ-U / DAQ-M / DAQ-E, którego pakiet kalibracyjny nie może zostać pobrany — jesteś
w trybie offline lub ten czujnik nie ma kalibracji w pliku — jest **pomijany z podaniem przyczyny** w
wierszu `[DAQ-EXPORT]` i nigdy nie jest zapisywany jako plik „skalibrowany” zawierający surowe odczyty.
Należy połączyć się z Internetem i ponownie uruchomić proces. Przyczyna jest tą, którą czytnik faktycznie
ustalił dla tego pliku (nieczytelny schemat, brak pakietu, błąd zapisu), a podsumowanie przebiegu
wymienia **oddzielne** przyczyny — dwadzieścia pominiętych plików z jednej przyczyny jest odczytywanych jako jedna
przyczyna, a nie dwadzieścia powtórzeń tej samej przyczyny. CLI podaje nazwę każdego pominiętego zapisu na końcu
przebiegu wraz z przyczyną:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Folder z nagraniami, z którego **żadne** nie zostało wyeksportowane, kończy się niepowodzeniem: zobacz
[Przebieg, który nie zapisuje żadnych obrazów, kończy się niepowodzeniem](#a-run-that-writes-no-images-fails).

Nagranie, którego nazwa kończy się na `_calibrated` lub `_raw` (`field_raw.daq`), **nie jest
importowany**: są to nazwy, jakie `process` nadaje swoim własnym produktom, a produkt odczytany
jako nagranie zostałby zliczony dwukrotnie. CLI informuje o tym i podaje nazwę pliku —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— zamiast zgłaszać pusty folder. Przed przetworzeniem należy zmienić nazwę takiego zapisu.

#### Zapisy DAQ-A są eksportowane jako surowe zliczenia

Rodzina **DAQ-A** powstała przed wprowadzeniem systemu pakietów przypisanych do poszczególnych numerów seryjnych i nie ma pakietu kalibracyjnego,
który można by pobrać — zamiast tego jest kalibrowana w terenie względem wzorca odbicia, co
wyjaśnia, dlaczego nigdy go nie potrzebowała. Odrzucenie tych nagrań uniemożliwiłoby w ogóle uzyskanie ich
danych, więc są one eksportowane pod **inną nazwą**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Inna nazwa pliku zamiast flagi wewnątrz pliku, ponieważ informacja ta musi przetrwać
przesyłanie pocztą elektroniczną jako sama nazwa. Nagłówek `.csv` zawiera informację
`raw spectral sensor counts (NOT irradiance)` i ostrzega, że wartości są porównywalne
**w obrębie** pliku — co jest dokładnie tym, do czego służy kalibracja oparta na wzorcu — a
nie między czujnikami. Kolumny fotometryczne zależne od mocy (moc całkowita, luks fotopowy i
skotopowy, PPFD) są zapisywane jako **NULL** zamiast być zintegrowane na podstawie zliczeń, a podsumowanie
przebiegu zawiera wartość `RAW COUNTS`, więc „wyeksportowane” w dzienniku nie może być odczytane jako natężenie promieniowania.

Starsze zapisy **v1.01 / v1.02** (zapisywane przez DAQ-A-SD) nie zawierają epoki dla poszczególnych odczytów,
a jedynie czas zapisu pliku. Moduł dopasowujący obrazy↔promieniowanie padające nadal je odrzuca — dopasowanie
klatki do czasu zapisu byłoby niewidocznym błędem — ale eksportator je odczytuje, a
plik CSV wyświetla `clock=daq_created_on`, więc produkt wskazuje, na którym zegarze się znajduje.

### Uwagi

- `process` automatycznie wykrywa, czy folder zawiera dane typu Survey3, „LATTICE”, czy mieszane.
- Postęp jest przesyłany strumieniowo za pośrednictwem Server-Sent Events; interfejs CLI pokazuje na żywo postęp dla poszczególnych wątków (wykrywanie, analiza, przetwarzanie, eksport).
- W przypadku systemu Linux/Jetson CLI sprawdza pamięć wymiany i może wyświetlić ostrzeżenie przed przetwarzaniem dużych folderów. Algorytm debayeringu uwzględniający tekstury automatycznie nakłada również ograniczenie częstotliwości GPU na urządzeniach Jetson o niskim poborze mocy (Nano, Orin Nano).
- W przypadku powodzenia operacja zgłasza liczbę zapisanych produktów obrazowych (`Image products written: N`, a w przypadku, gdy proces rozdzielania ograniczył wartość jakiegokolwiek piksela – `(plus M clamp masks)`).
- **`process` bezpiecznie współdzieli uruchomiony backend.** Automatycznie ponownie uruchamia już działający backend tylko wtedy, gdy backend ten jest w stanie bezczynności z załadowanym poprzednim projektem i nie obsługuje żadnych kamer, tablic ani urządzeń DAQ. Backend, który obsługuje którekolwiek z nich (interfejs graficzny przesyłający tablicę, skrypt SDK, nagranie z urządzenia DAQ), jest używany w obecnej postaci i nigdy nie jest restartowany, dzięki czemu `process` może działać równolegle z aktywną tablicą. Jeśli backend zgłasza trwające uruchomienie, `process` czeka do 15 s na jego zakończenie (wyczyści się po około sekundzie od zakończenia), a następnie zgłasza błąd `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Globalny znacznik `--restart` nadal wymusza uruchomienie nowego backendu i kończy na nim wszelkie sesje kamer, tablic lub DAQ. Backend uruchamiany jako usługa systemd w systemie Linux nigdy nie jest restartowany: `process` korzysta z niego, a `--restart` wyświetla zamiast tego `sudo systemctl restart <unit>` (zobacz [Uruchamianie backendu jako usługi](#running-the-backend-as-a-service-headless)). Podobnie jest z własnym backendem aplikacji Chloros: gdy aplikacja jest otwarta, korzysta z niego proces `process`, ładując swój projekt zamiast projektu aplikacji (backend uruchomiony w tym miejscu zablokowałby dostęp otwartej aplikacji).
- **Uruchomienie w toku zachowuje swój projekt.** Podczas trwania przebiegu przetwarzania — uruchomionego przez aplikację, `process`, skrypt SDK lub `chloros-mcp` — backend odmawia tworzenia, otwierania, importowania do, dodawania lub usuwania plików z, przenoszenia, usuwania ani zmiany ustawień przetwarzania projektu, zwracając kod błędu HTTP 409 `PROCESSING_IN_PROGRESS` i `A processing run is in progress on project '<name>', so ... now is refused`; kod `process`, który to wykrywa, powoduje zamknięcie `1`. Ustawienia wyświetlania (rozmiar miniatur) nadal są zapisywane. Tryb przetwarzania oraz ustawienia docelowej odbiciowości projektu są odrzucane w ten sam sposób (przebieg zachowuje tryb, w którym został uruchomiony). Przechwycenie zapisane w projekcie w trakcie działania (z aplikacji, CLI, SDK lub `chloros-mcp`) jest zapisywane jak zwykle, ale dołącza do listy plików projektu dopiero po zakończeniu działania; potwierdza to komunikat (`project_add`), podobnie jak w przypadku zatrzymania w projekcie rejestracji z czujnika światła. Wcześniej drugi klient mógł przełączyć otwarty projekt w trakcie sesji, a trwająca sesja była eksportowana do innego projektu.
- **Kombinacja klawiszy Ctrl+C zatrzymuje przebieg w backendzie, a nie tylko w CLI.** CLI prosi backend o zatrzymanie przetwarzania, wyświetla komunikat `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` i czeka do 120 s na komunikat `Processing stopped.`, aby `chloros-cli process … && next` nie uruchomił `next` na backendzie, który jest nadal zajęty biegiem. Drugie naciśnięcie Ctrl+C kończy oczekiwanie. `process` następnie kończy działanie `130`, tak jak robi to każde przerwane polecenie.

#### Przebieg, który nie zapisuje żadnych obrazów, kończy się niepowodzeniem

Jeśli poprosiłeś o produkty, a przebieg nie zapisał **żadnego** — tylko `project.json` i
`calibration_data.json` — `process` traktuje to jako niepowodzenie: wyświetla
`Processing finished but wrote no image products.` i **kończy działanie z wynikiem niezerowym**, dzięki czemu skrypt może
to wykryć. Komunikat podaje nazwę folderu projektu oraz typowe przyczyny:

- folder wejściowy nie został rozpoznany jako przechwycony (sprawdź układ i `--input-level`) lub
- każdy żądany produkt został pominięty jako nieodpowiedni dla tych kamer (np. żądanie
  promieniowania/odbicia od kamer obsługujących wyłącznie RGB).

Uruchom ponownie z opcją `--verbose` i sprawdź dziennik backendu pod kątem wierszy `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
które wyjaśniają pominięcia dla poszczególnych kamer, które w innym przypadku nie pojawiają się w wynikach CLI.

Celowe uruchomienie wyłącznie z metadanymi — wszystkie opcje produktów wyłączone i brak `--indices` — nadal jest
**sukcesem**, ponieważ pusty obraz wyjściowy jest w tym przypadku prawidłowym wynikiem.

#### Uruchomienie, którego eksportów nie udało się zweryfikować, kończy się niepowodzeniem

Po zakończeniu przebiegu `process` sprawdza tagi wyeksportowanych plików za pomocą ExifTool. Jeśli nie udało się uruchomić ExifTool,
wyświetla komunikat `Weryfikacja eksportu NIEUDANA: nie udało się uruchomić ExifTool, więc tagi N z M wyeksportowanych plików
nie zostały sprawdzone (zobacz wiersze [EXIFTOOL] w dzienniku backendu)` and `Wyniki są zapisywane i pozostają
na miejscu; uruchom ponownie, aby je zweryfikować.`, does not print `Przetwarzanie zakończone!` oraz **kończy działanie z wynikiem niezerowym**.
Wyniki są zachowywane.

#### Które zrzuty ekranu uwzględnia dane uruchomienie

Zrzuty ekranu zakończone przed uruchomieniem `process` są uwzględnione. Zrzuty ekranu zapisane podczas uruchomienia nie są uwzględniane; `process`
wyświetla ostrzeżenie o liczbie pominiętych zrzutów, a ponowne uruchomienie narzędzia uwzględnia je.

#### Przebieg, w którym utracono obraz, kończy się niepowodzeniem

Jeśli podczas przebiegu wystąpiła awaria obrazu, `process` wyświetla komunikat `N image(s) failed and were not exported` z
najczęstszą przyczyną, nie wyświetla komunikatu `Processing complete!` i **kończy działanie z wynikiem innym niż zero**. Pozostałe produkty są
zapisywane i pozostają na swoim miejscu, a dziennik zaplecza zawiera informacje o każdej awarii.

Tak samo jest w przypadku **przebiegu wyłącznie z czujnikiem światła**: folder z zapisami `.daq` z definicji nie zawiera obrazów do eksportu,
a przebieg jest oceniany na podstawie skalibrowanych plików `.daq` / `.csv`, które zostały zapisane zamiast tego.
Jeśli nie zapisał **żadnego** — wszystkie zapisy zostały pominięte — wyświetla
`Processing finished but wrote no light-sensor products.`, nazywa każdy zapis, podając
przyczynę pominięcia, i **kończy działanie z wynikiem niezerowym**.

---

## `chloros-cli login`

Uwierzytelnij to urządzenie za pomocą konta w chmurze Chloros+. Dane uwierzytelniające są bezpiecznie buforowane w pliku `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Przykłady

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (usuwając fragmenty hasła lub powielając jego części). W przypadku błędu 401 interfejs CLI automatycznie ponawia próbę, dodając na końcu `$$`, a następnie z pozbawioną duplikatów połową hasła; jeśli ponowna próba się powiedzie, system zaloguje użytkownika i wyświetli poprawną składnię z pojedynczymi cudzysłowami do wykorzystania następnym razem.

> **Użycie w trybie bezinterfejsowym/skryptowym: brak sesji w pamięci podręcznej oznacza wyświetlenie monitu o logowanie.** Polecenia podstawowe (`process`, `status`, `export-status`, `time-sync`, …) uruchomione bez sesji z pamięci podręcznej przed kontynuacją żądają podania `Email:` / `Password:` na stdin. Gdy stdin jest zamknięte (cron, jednostka systemd), program natychmiast kończy się niepowodzeniem z kodem `login cancelled`; gdy stdin jest otwartym terminalem lub potokiem, czeka na dane wejściowe. Przed zaplanowaniem zadań bezinterfejsowych należy uruchomić program `chloros-cli login EMAIL PASSWORD` raz na każdym komputerze.

---

## `chloros-cli logout`

Czyszczenie buforowanej sesji i wymuszenie ponownego logowania przy następnym wywołaniu.

Wylogowanie się w trakcie trwania przetwarzania nie powoduje jego zatrzymania: przetwarzanie kończy się w trybie, w którym zostało uruchomione, a następne przetwarzanie odbywa się w trybie standardowym. Informację tę zawiera komunikat `logout` (`The processing run in progress finishes in <mode> mode; the next run uses standard`), a także aplikacja po wylogowaniu się się w tym miejscu.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Wyświetla aktualny poziom licencji (Iron/Copper/Bronze/Silver/Gold), uwierzytelnionego użytkownika oraz liczbę powiązań z urządzeniami.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Sprawdź aktualny postęp eksportu Thread-4. Można bezpiecznie wywołać tę funkcję **podczas** działania `process` z innej powłoki.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Ustawia język wyświetlania interfejsu CLI (obsługiwanych jest 38 języków, w tym CJK, RTL i języki indyjskie). Na starszych konsolach, które nie obsługują skryptu, język jest płynnie przełączany na angielski.

```
chloros-cli language [LANG_CODE] [--list]
```

### Przykłady

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Polecenia dotyczące folderu projektu

Służą one do zarządzania domyślną lokalizacją folderu projektu (wspólną z interfejsem graficznym).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` wyświetla folder, do którego trafiają nowe projekty. Jeśli skonfigurowany folder już nie istnieje (został usunięty, przemianowany lub znajduje się na dysku, który nie jest podłączony), CLI, podobnie jak interfejs graficzny, przechodzi na folder domyślny: folder domyślny jest wyświetlany na stdout, ostrzeżenie z nazwą brakującego folderu trafia do stderr, a kod wyjścia to `0`.

---

## `chloros-cli update`

Tylko Linux/Jetson. Sprawdza `version_url` z `/etc/chloros/update.conf` i, jeśli istnieje nowsza wersja, pobiera odpowiednią wersję `.deb` i natychmiast ją instaluje za pomocą `sudo dpkg -i` (bez monitu o potwierdzenie; `sudo` może poprosić o podanie hasła). Na Jetsonie aktualizacja jest oferowana wyłącznie dla systemu JetPack 6. W systemie Windows polecenie to nie wykonuje żadnej czynności i wyświetla komunikat `up to date`: zaktualizuj za pomocą nowego instalatora.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

W systemie Linux/Jetson interfejs CLI**przeprowadza również**automatyczne sprawdzanie dostępności aktualizacji przy każdym uruchomieniu** (nieblokujące, nigdy nie opóźnia wykonania polecenia): odczytuje `/etc/chloros/update.conf`, buforuje wynik przez 1 godzinę w `~/.chloros/update_cache.json` i wyświetla komunikat `Update available: vX.Y.Z / Run: chloros-cli update`, gdy dostępna jest nowsza wersja. W przypadku jakiegokolwiek błędu oraz w systemie Windows operacja ta jest pomijana bez powiadomienia; plik `CHLOROS_DISABLE_UPDATE_CHECK=1` wyłącza tę funkcję. Ze względu na godzinny czas buforowania wersja opublikowana w ciągu tej godziny może nadal być odczytywana jako aktualna.

---

## `chloros-cli selftest`

Przeprowadza 7-etapowy test wstępny: wersja, dostępność portów, uruchomienie backendu, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), obecność modelu odszumiającego, gotowość CUDA + odszumianie. Wykorzystuje działający backend lub uruchamia go na potrzeby testu, a następnie zatrzymuje. Wyświetla `N/7 checks passed` oraz nieudane testy i kończy działanie `0` tylko wtedy, gdy wszystkie siedem testów zakończy się powodzeniem; sprawdzenie CUDA i modułu odszumiającego kończy się niepowodzeniem na komputerze bez procesora graficznego obsługującego CUDA (zobacz [Weryfikacja instalacji](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Status i sterowanie serwera głównego PTP. Host Chloros pełni rolę serwera głównego PTP; kamery LATTICE i moduły DAQ-E działają jako urządzenia podrzędne w celu uzyskania znaczników czasu między urządzeniami (około 1 ms między urządzeniami). Protokół PTP synchronizuje **znaczniki czasu**: kamery w macierzy wykonują ekspozycję jednocześnie, ponieważ uruchamia je sprzętowy wyzwalacz M8 (z opóźnieniem poniżej 100 µs między modułami), a nie z powodu protokołu PTP. `chloros-cli time-sync` bez podkomendy uruchamia `status`.

| Podkomenda | Opis |
| --- | --- |
| `status` | Wyświetla stan serwera nadrzędnego, priorytety BMCA, tożsamość zegara oraz informację, czy jego pakiety docierają do sieci: `health` to `OK` (`STOPPED`, gdy serwer główny nie działa) lub `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` wraz z `health_detail`, a następnie do stderr jest wyświetlana linia `WARNING:` (stdout pozostaje w formacie JSON). `sync_sent` / `announce_sent` zliczają tylko wiadomości, które zostały wysłane; `*_send_errors` i `last_send_error` zliczają te, które nie zostały wysłane. Po utracie i przywróceniu połączenia z kamerą serwer główny samodzielnie odbudowuje swoje gniazda (`rebinds`, `last_rebind`). |
| `peers` | Lista urządzeń podrzędnych wykrytych za pomocą Delay_Req (kamery + czujniki DAQ-E). |
| `cameras` | Stan PTP dla poszczególnych kamer (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Ponowne uruchomienie procesu grandmastera. |
| `set-priority --priority1 N --priority2 N` | Zastąpienie priorytetów BMCA. |

Grandmaster ogłasza na kartach sieciowych kamer (każda karta sieciowa z adresem `169.254.x.x` w zestawie). Jeśli żadna z nich nie jest aktywna, przechodzi na kartę sieciową trasy domyślnej tylko wtedy, gdy jest to połączenie przewodowe, nigdy Wi-Fi ani karta wirtualna (Tailscale, WSL), ponieważ żadna kamera ani urządzenie DAQ-E nie znajduje się za żadną z nich. Jeśli nie ma nic do ogłoszenia, działa bez gniazd (`health` `NO_SOCKETS`), a kolejne połączenie kamery lub urządzenia DAQ odbudowuje je na karcie sieciowej, która jest w tym momencie aktywna; `time-sync restart` robi to od razu.

Grandmaster działa we własnym procesie, obok backendu (`own_process` / `process_pid` w `time-sync status`), z wysokim priorytetem w systemie Windows (`process_priority`), dzięki czemu obciążony backend (przebieg przetwarzania, eksport, kilka poleceń jednocześnie) nie może opóźniać czasów, które rejestruje; wcześniej jedno takie opóźnienie mogło przesunąć zegar każdej kamery jednocześnie. Proces ten kończy się wraz z zakończeniem działania modułu zaplecza; jeśli nie można go uruchomić, grandmaster działa wewnątrz modułu zaplecza tak jak poprzednio (`own_process` false, co potwierdza dziennik modułu zaplecza).

### Przykłady

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Sterowanie kamerami LATTICE. Każde podpolecenie przechodzi przez backend Chloros; backend zarządza pulą kamer, więc kolejne wywołania z poziomu CLI ponownie wykorzystują ten sam otwarty uchwyt.

### Typowe opcje (wspólne dla większości podpoleceń)

| Flaga | Opis |
| --- | --- |
| `-d, --device N` | Indeks kamery (domyślnie: 0). |
| `-s, --serial SN` | Określony numer seryjny; zastępuje `--device`. |
| `--serials SN1,SN2,…` | Numery seryjne oddzielone przecinkami do pracy z wieloma kamerami. [Polecenia przejściowe dla wielu kamer](#multi-camera-transient-sessions) otwierają tylko te kamery, przy czym pierwsza z nich uruchamia wyzwalacz. |
| `--all` | Działa na każdej wykrytej kamerze. |
| `--exposure US` | Czas naświetlania w mikrosekundach. |
| `--gain DB` | Wzmocnienie w dB. |
| `--pixel-format FMT` | np. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` w modelu M3M. Nazwa określa głębię bitową w ramach rodziny aparatów: w przypadku M3M nazwa typu Bayera określa format Mono jego głębi (`BayerRG8` ustawia `Mono8`), w przypadku M3C nazwa typu Mono określa nazwę typu Bayera (`Mono12` ustawia `BayerRG12`). |
| `--width N` / `--height N` | Wymiary obrazu. |
| `--preset {default,high_quality,high_speed,triggered}` | Zastosuj gotowy zestaw ustawień: format pikseli, ekspozycję, wzmocnienie i liczbę klatek na sekundę (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 klatek na sekundę; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` zawiera ich listę). W przypadku M3M formatem jest Mono o tej samej głębokości: `high_speed` przechwytuje `Mono8`, `high_quality` oraz `triggered` i `Mono12`. Podana wraz z nim flaga (`--exposure`, `--gain`, `--pixel-format`) ma pierwszeństwo przed ustawieniem wstępnym, a ustawienie, którego nie wymienia ustawienie wstępne, zachowuje ustawienie własne kamery. Wszystkie wolnedziałają w trybie swobodnym, z wyjątkiem `triggered`, który uzbraja kamerę na zmianę stanu linii 2 — jeśli nic nie steruje tą linią, kamera będzie czekać w nieskończoność zamiast rejestrować obraz. |
| `-o, --output DIR` | Katalog wyjściowy (domyślnie: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Rozmiar pakietu GVSP. `auto` uruchamia sondy ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Procedura pierwszego połączenia z kamerą LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Opis podpoleceń

#### Wykrywanie i informacje

| Podpolecenie | Cel |
| --- | --- |
| `lattice info` | Wyświetla listę podłączonych kamer (producent, model, numer seryjny, adres IP, adres MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analiza systemu hosta pod kątem optymalnej konfiguracji kamery. Polecenie `--no-discover` pomija wykrywanie kamer (szybsza analiza oparta wyłącznie na karcie sieciowej). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Sprawdź/popraw ustawienia karty sieciowej (w systemie Linux również bufory odbiorcze gniazd jądra — zobacz [hosty Linux](#linux-hosts--socket-receive-buffers)); oszacuj przepustowość/liczbę klatek na sekundę. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Zalecenia dotyczące możliwości sieciowych backendu o stabilnej strukturze+ rekomendacja tablicy (zwraca `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` domyślnie przyjmują pełny czujnik, `2048` × `1536`. Wyłącza `0` dla `ok`, `auto_capped_fps` oraz `auto_shrunk`, a także `1` dla `needs_force_slip` i `error`, z lub bez `--json`. Dociera do backendu przy wartości `--backend-url` (domyślnie `http://127.0.0.1:5000`), a nie przy `CHLOROS_BACKEND_URL`. `auto_capped_fps` zachowuje żądaną rozdzielczość, ale ogranicza docelową liczbę klatek na sekundę — odczytaj `recommended.recommended_target_fps` i przekaż go jako cel połączenia; potraktuj to jako sukces, a nie jako błąd. `auto_shrunk` pojawia się tylko wtedy, gdy żądanej klatki nie można dostarczyć przy żadnej częstotliwości klatek; connect nigdy samodzielnie nie zmniejsza rozmiaru klatek. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analiza „co by było, gdyby” bez uruchamiania kamer (`--width` / `--height` domyślnie `2048` × `1536`). Zarówno ten, jak i `network-analysis` przewidują, co będzie przesyłać connect: bez `--pixel-format`, obowiązują formaty określone przez „connect” dla poszczególnych kamer (Mono12 / BayerRG12), a przy `--packed auto` (domyślnie) [spakowany 12-bitowy](#packed-12-bit-transport---packed) — podana liczba klatek na sekundę (fps) to częstotliwość, z jaką działa ta macierz. Oba wyświetlają decyzję dotyczącą pakowania. **`--n-active` to całkowita liczba kamer podłączonych do sieci, a nie tylko tej macierzy**— należy ją zwiększyć, gdy kamery autonomiczne przesyłają strumień jednocześnie lub gdy limit przepustowości sieci jest obliczany na podstawie zapotrzebowania, które nie uwzględnia wszystkich kamer (domyślnie: `len(--models)`). Zawsze wyświetla zagregowane wiersze `Wire budget:` (wymagana przepustowość w MB/s w porównaniu z pułapem zabezpieczającym przed kolizjami) i `Max cameras:` oraz sygnalizuje `** OVER-SUBSCRIBED**`, gdy macierz nadmiernie obciąża magistralę — zobacz [Model fps i przepustowości szczytowej macierzy](#array-fps--burst-model). |
| `lattice gpu` | Wyświetl stan procesora graficznego. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Sprawdź lub zaktualizuj oprogramowanie sprzętowe kamery. Lokalny wybór `.fwa` jest zablokowany: plik w `firmware/<MODEL_PREFIX>/` pasujący do `MIN_FIRMWARE_VERSION` kompilacji jest wgrywany, jeśli jest obecny (tylko najwyższa wersja jako rozwiązanie awaryjne), więc nowszy obraz dostawcy przygotowany na dysku pozostaje nieaktywny, dopóki ten pin nie zostanie zmieniony — celowo nowsze wersje trafiają do urządzeń za pośrednictwem podpisanego manifestu AWS, co jest preferowane, gdy są nowsze. |
| `lattice presets [--apply NAME]` | Wyświetla listę lub stosuje ustawienia wstępne kamery. `--apply` określa, co oznaczają nazwy ustawień wstępnych (ich format pikseli, ekspozycja, wzmocnienie, liczba klatek na sekundę, wyzwalacz) i wyświetla te ustawienia tak, jak odczytuje je kamera po: w przypadku M3M — format Mono głębokości ustawienia wstępnego; częstotliwość klatek, z jaką działa, która może być niższa niż wymaga tego ustawienie wstępne (`high_speed` wymaga 60 klatek na sekundę; kamera pełnoklatkowa podłączona do łącza 1 Gbps działa z prędkością około 35 klatek na sekundę). Wszystkie pozostałe ustawienia zapisane przez program „open” są przywracane w stanie, w jakim zostały znalezione. `--apply default` resetuje kamerę do domyślnych ustawień fabrycznych: ustawień fabrycznych z automatyczną ekspozycją w trybie swobodnym na linii 2, wyłączonym limitem przepustowości łącza i wyłączoną korekcją defektów, przy zachowaniu skalibrowanego balansu bieli kamery RGB (jedność w przypadku wszystkich innych). Żadne dane nie są zapisywane w pamięci kamery, więc przy następnym włączeniu zasilania kamera uruchamia się z własnej pamięci tak jak poprzednio. Podczas przechwytywania lub nawiązywania połączenia polecenia `--preset` i `default` nie są ustawieniami wstępnymi. |
| `lattice status` | Stan kamery na żywo. |

#### Przechwytywanie

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Pojedyncza klatka. **Domyślnie zapisuje każdy typ eksportu**(`--processing all`); zobacz [Poziomy eksportu przechwytywania](#capture-export-levels-the-all-default). `--processing` przyjmuje `all`, `raw`, `debayered`, `radiance`, `reflectance` lub `display` (`preview` jest akceptowane dla `display`); każda inna nazwa jest odrzucana przed dotknięciem aparatu (wyjście `1`). `--levels` zapisuje jawny podzbiór (zastępuje `--processing`). Kamera, której Chloros nie posiada, jest otwierana na potrzeby sesji, a jej ekspozycja jest najpierw ustalana na podstawie sceny (maksymalnie kilka sekund), tak jak w przypadku [polecenia przejściowe](#multi-camera-transient-sessions); gdy przechwytywanie się kończy, niezależnie od sposobu zakończenia, kamera ta zostaje przywrócona do stanu, w jakim została znaleziona (format, ekspozycja, potok kolorów, każde ustawienie zmienione podczas przechwytywania — lista w sekcji [Wiele kamer (sesje przejściowe)](#multi-camera-transient-sessions)). Odczyt z modułu DAQ użytego do przechwytywania jest zapisywany jako plik pomocniczy `.daq`, zawierający wyłącznie surowe dane; pliki `--no-daq` / `--daq-tolerance-ms` dostosowują go ([Urządzenie DAQ odczytujące przechwycenie zapisuje](#the-daq-reading-a-capture-saves)). Plik `--force-daq` jest nadal akceptowany. `--jpeg-quality` = jakość JPEG 1–100 (domyślnie 95).**Pliki zapisane w otwartym projekcie dołączają się do niego.** Podobnie jak w aplikacji, przechwycenie, którego pliki trafiają do folderu projektu otwartego w Chloros, dodaje je do listy plików tego projektu; pliki zapisane gdzie indziej pozostają bez zmian. Polecenie wyświetla jeden wiersz z informacją o tym (`Project: ...`) lub, jeśli trwa przetwarzanie, informację, że są one wstrzymane i zostaną dołączone po zakończeniu przetwarzania. Podobnie działają `multi-capture`, `array-capture` (`--smart` również) oraz `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Przesyłaj strumieniowo na dysk do momentu naciśnięcia Ctrl+C. Przebieg `tiff`to plik `raw` stanowi tablicę zapisów z tej kamery: mozaikę czujnika, jedną płaszczyznę z własnymi wartościami, nieskompresowaną, wraz z pakietem kalibracyjnym (klatka o wymiarach 2048 x 1536 i 12 bitach zajmuje 6,0 MiB plus pakiet, niezależnie od sceny), więc `process` rozpoczyna się od nich. Pliki `png` i `jpg` zawierają 8 bitów, więc te serie to klatki poddane demosaicowaniu w celu wyświetlenia, od których plik `process` nie zaczyna. W przypadku powiązania z systemem DAQ (`daq pool-bind`) lub w puli, każda klatka otrzymuje własne pliki `.daq` i `.match.json` w folderze `daq/` danego przebiegu, ponieważ `lattice capture` zapisuje jedną; `--no-daq` wyłącza tę funkcję, a po zatrzymaniu wyświetla liczbę klatek. Każda klatka jest zapisywana raz wraz z pełnymi metadanymi; po naciśnięciu Ctrl+C klatki już przechwycone są zapisywane przed zwolnieniem aparatu. Program zatrzymuje się samoczynnie, gdy dysk się zapełni, wyświetla komunikat `Failed:` / `Disk full:` i kończy działanie `1`; każda klatka, której nie udało się zapisać, również powoduje zakończenie działania `1`. Aparat pozostawiony w trybie gotowości do wyzwolenia sprzętowego (`TriggerMode` `On`) jest ustawiona na tryb swobodny na czas serii. Po zakończeniu serii kamera zostaje przywrócona do stanu, w jakim została znaleziona — jej ustawienia wyzwalania oraz wszystkie parametry zmienione podczas serii (ekspozycja, wzmocnienie, format itp.) — chyba że wystąpi błąd `--keep-settings`, który zapisuje ustawienia sesji w pamięci wewnętrznej kamery (tryb swobodny), lub `--clear-settings`, który resetuje tę pamięć do ustawień fabrycznych. Cykl, w którym nie zarejestrowano żadnej klatki, kończy się niepowodzeniem: po 10 sekundach bez klatki zatrzymuje się samoczynnie, wyświetla komunikat `No frames:` wraz z przyczyną i kończy działanie `1`. Klatki, z którymi zapisywacze nie nadążają, są traktowane jako utracone, a zatrzymanie powoduje wyświetlenie komunikatu `Dropped:` wraz z przyczyną (częstotliwość klatek kamery, zapisywaczy i kolejki); niższa częstotliwość klatek lub 8-bitowy format pikseli pozwala zachować je wszystkie. Polecenie komunikuje się z backendem co 2 s; jeśli ulegnie awarii bez zatrzymania nagrywania (`kill -9`, brak pamięci), backend zatrzymuje je 30 s po ostatnim zgłoszeniu i przywraca kamerę (`stop_reason` `client_gone` oraz wpis w dzienniku serwera). Polecenia `multi-stream` i `align-stream` działają w ten sam sposób. Uruchomienie, podczas którego otwieranie kamery trwa dłużej niż czas oczekiwania polecenia (60 s; 180 s dla `multi-stream` / `align-stream`) — kamera wciąż się uruchamia — jest cofane przez backend: kamera zostaje przywrócona do poprzedniego stanu i zamknięta, a żadne nagrywanie nie jest kontynuowane (kod HTTP 504 `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Podgląd na żywo w formacie MJPEG w przeglądarce: łączy się z kamerą, wyświetla URL strumienia i otwiera go w przeglądarce. `--ae-damping` ustawia tłumienie automatycznej ekspozycji (0,4–100). |

##### Transmisje na żywo (MJPEG; brak RTSP)

Chloros dostarcza obraz na żywo w formacie **MJPEG przez HTTP**(`multipart/x-mixed-replace`, granica `frame`) z serwera.**Protokół RTSP nie jest obsługiwany.** Otwórz adres URL w przeglądarce lub odczytaj go za pomocą dowolnego klienta MJPEG (`cv2.VideoCapture(url)` z biblioteki OpenCV, ffmpeg, VLC). Kamera musi być najpierw podłączona w backendzie (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` lub w interfejsie graficznym).

| URL (`http://127.0.0.1:5000` + …) | Strumień |
| --- | --- |
| `/api/camera/stream/<serial>` | Podgląd obrazu, do około 30 klatek na sekundę. W przypadku elementu tablicy, `?max_h=` (domyślnie `720`) oraz `?jpeg_q=` (domyślnie `80`) ustawiają wysokość i jakość JPEG. Jest to adres URL, który otwiera `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | Ten sam podgląd z narysowanymi nakładkami interfejsu graficznego: `zebra=1`, `crosshair=1`, `grid=3x3` (lub `9x9`), `histogram=1`, `peaking=1`, `spot=1`; bez zapytania, zapisane w projekcie ustawienia nakładek dla tej kamery. |
| `/api/camera/<serial>/stream-reflectance` | 8-bitowa reflektancja na pasmo: ρ = piksel ÷ częśćnagłówka `X-Chloros-Pixel-Scale` (255, o ile nie jest zgłoszony wyjątek `refl_max`). `fps` (domyślnie `10`), `jpeg_q` (domyślnie `95`), `max_h` (domyślnie `0` = natywne), `band=<name>`, `align=1`. Format JPEG, a więc stratny. Odmowa z kodem HTTP 409, gdy kamera nie ma współczynnika odbicia (kamera RGB, brak pakietu kalibracyjnego, brak odczytu DAQ). |

- **Dopasowywanie klatek między kamerami.**Każda część wysyłana przez**element tablicy**, we wszystkich trzech strumieniach, zawiera nagłówki `X-Chloros-Group-Id` (licznik zsynchronizowanej grupy tablicy), nagłówki `X-Chloros-Frame-Id` oraz `X-Chloros-Timestamp-Ns`. Części z różnych kamer o tym samym `X-Chloros-Group-Id` pochodzą z tego samego wyzwalacza. Kamera autonomiczna nie ma grupy i nie wysyła identyfikatora grupy. Przeglądarki ignorują nagłówki poszczególnych części; odczytuje je klient, który samodzielnie analizuje strumień wieloczęściowy.
- **Wstrzymane, gdy widok macierzy jest otwarty.** Element `/api/camera/stream/<serial>` należący do macierzy wysyła statyczną klatkę „paused”, gdy połączony strumień na żywo macierzy jest otwarty (na przykład widok macierzy w GUI), i wznawia działanie po jego zamknięciu.

#### Konfiguracja czujników

| Podpolecenie | Cel |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Odczyt/zapis dowolnego węzła GenICam. `--set ExposureTime=…` (`Gain=…`) bez przełączników `ExposureAuto` (`GainAuto`), które najpierw wyłączają tryb automatyczny, jak robi to `lattice exposure --set`; wartość odrzucona przez kamerę powoduje wyświetlenie komunikatu `Failed to set <node>: …` na stderr i zakończenie działania `1`. `--get` oraz domyślny widok zgrupowany wyświetlają komunikat `note:` pod `ExposureTime` / `Gain` podczas działania trybu automatycznego — zobacz [Odczytywanie ekspozycji i wzmocnienia w trybie automatycznym](#reading-exposure-and-wzmocnienie-w-trybie-automatycznym). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Ekspozycja i AE. Wartość ekspozycji `--set` lub wartość docelowa `--brightness` są traktowane jako twoje (automatyczna kontrola ekspozycji Chlorosa nie ingeruje w to); wzmocnienie pozostaje bez zmian, więc w trybie automatycznym aparat dostosowuje się do wartości docelowej, dopóki nie ustawisz `lattice gain`. `--auto` przywraca kontrolę nad ekspozycją, a `--brightness-auto` przywraca docelową wartość do automatycznego ustawienia Chlorosa. Bez żadnych flag odczytuje węzły ekspozycji, z tym samym `note:` co `configure --get`. Wartość `--set`, której aparat nie akceptuje, powoduje wyświetlenie `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` na stderr — tryb automatyczny, w którym zapis został wyłączony, zostaje przywrócony, a nie pozostawiony w stanie wyłączonym — nie wyświetla się wiersz `Exposure: … us (auto OFF)`, a program kończy działanie `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Wzmocnienie i automatyczne wzmocnienie. Przy wartości `--set` kamera odrzuca, wyświetla komunikat `Failed to set Gain: …` na stderr, nie wyświetla linii `Gain: … dB (auto OFF)` i kończy działanie `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Obszar zainteresowania (ROI) czujnika i binning. Binning jest zawsze typu **Średnia** — patrz poniżej. |
| `lattice format [--set FMT] [--list]` | Format pikseli. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Wyzwalacz sprzętowy/programowy. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (brak flag = jednorazowa korekcja balansu bieli) | Operacje balansu bieli. Tylko kamery RGB/Bayer; operacja pomijana (no-op) w przypadku monochromatycznych M3M. Tryb jednorazowy zachowuje ustalony balans bieli (`BalanceWhiteAuto` oraz wzmocnienia); wszystkie pozostałe ustawienia zapisane w otwartym pliku są przywracane do stanu pierwotnego, a całość jest przywracana, gdy tryb jednorazowy nie może zostać uruchomiony. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Potok kolorów wyświetlacza RGB. `natural` (domyślnie) to ekonomiczne wykończenie na żywo; `enhanced` dodaje redukcję fringingu + żywość + lokalny kontrast CLAHE, zapewniając pełny efekt „hub-parity” przy około dwukrotnie wyższym koszcie przetwarzania na klatkę, co skutkuje niższą **na żywo** liczbą klatek na sekundę — zapisane ujęcia zawsze otrzymują pełne przetworzenie niezależnie od tego. Tylko kamery RGB/Bayer; pomijane w przypadku monochromatycznego M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Nasycenie/kontrast w podglądzie na żywo (kamery z filtrem RGB); `100` jest neutralne, `--reset` ustawia obie wartości na 100. Nie ma to wpływu na zapisane zdjęcia. Pomijane w przypadku monochromatycznego M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (brak flag = odczyt) | Wyświetlanie **jasnych obszarów** (kamery z filtrami RGB): obniżanie najjaśniejszych, najbardziej białych pikseli — np. jasna biała powierzchnia, która jest renderowana jako płaska biała plama. `0` = wyłączone, `-100` = najsilniejsze. Podgląd na żywo i zrzuty na poziomie wyświetlania dostosowują się do tego ustawienia; jasność/odbicie nigdy tego nie robią, a funkcja nie może odtworzyć danych z czujnika, które uległy przesterowaniu (w tym celu należy zmniejszyć ekspozycję). Pominięto w przypadku monochromatycznego M3M. |
| `lattice filter [--list]` | Wyświetla listę modeli filtrów znanych programowi Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). Informacja o filtrze aparatu jest odczytywana na podstawie ciągu znaków modelu i nie można jej ustawić: `--set NAME` jedynie wyszukuje model, a następnie kończy działanie `1` za pomocą `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Sprawdza węzły zasilania/termiczne; włącza/wyłącza tryb czuwania o niskim poborze mocy. `--sleep` pozostawia kamerę w stanie uśpienia (limit przepustowości łącza włączony (`DeviceLinkThroughputLimitMode` `On`) na minimalnym poziomie, tryb oszczędzania energii włączony, jeśli kamera go posiada) wszystkie pozostałe ustawienia są przywracane do stanu, w jakim zostały znalezione; nieudane przejście w stan uśpienia jest przywracane w całości. Wybudzenie przywraca łącze do stanu, w jakim zastało je przejście w stan uśpienia (limit, tryb oraz opóźnienie pakietów `GevSCPD`, które kamera utrzymuje na poziomie 0, gdy limit jest włączony; zapis jest przechowywany na tym komputerze, więc budzenie po ponownym uruchomieniu serwera również je odnajduje), lub w przypadku kamery, której ten komputer nie wprowadził w stan uśpienia, a która wygląda na uśpioną (jej limit ustawiony na minimalną wartość), przy wyłączonym limicie i wartości `GevSCPD` pozostawionej na 0 przy 80 (ustawienia fabryczne); limit ustawiony powyżej wartości minimalnej pozostaje bez zmian. Sonda i wybudzenie przywracają wartości zapisane podczas ich otwarcia. |

> **Binning jest zawsze średni.** `lattice resolution --binning N` ustawia binning średni i
> nie oferuje przełączania trybów. Binning sumujący mnoży każdą wartość DN przez N², co kosztuje około dwóch stopni
> rezerwy jasności w jasnych obszarach na każdy krok binningu; Chloros ponownie dzieli ten współczynnik przed
> łańcuchem radiometrycznym, ale piksele, które binning sumujący doprowadził do pułapu czujnika, są obcięte i nie można ich
> odzyskać. Nie daje to nic więcej niż tryb średni — wskaźnik jest niezmienny względem skali, a skalarne wartości promieniowania
> są skalibrowane w skali średniej. Jeśli aparat jest dostarczony z ustawieniem sumującym (ustawionym
> przez inne narzędzie lub w wyniku przywrócenia ustawień fabrycznych), Chloros wykrywa to na podstawie odczytu z węzła i
> kompensuje; uruchomienie skryptu `lattice resolution --binning N` przywraca ustawienie „Average”. W wersji 1.2.0
> `--binning-mode` jest nadal akceptowana, więc starsze skrypty działają, ale wyświetla się tylko ostrzeżenie:
> binning pozostaje w trybie średnim.

> **Zmiana binningu, formatu pikseli i wyzwalacza w kamerze nie ma żadnego efektu.** W przypadku kamery, w której
> backend pozostaje otwarty i przesyła strumień (`cam-connect`, interfejs graficzny), polecenie `lattice resolution --binning`
> nie wchodzi w życie (wyświetla komunikat, że binning nadal obowiązuje, `Binning: 1x1`), a
> `lattice format --set` i `lattice trigger --mode` są odrzucane przez kamerę
> (`SC_ERR_ERROR -1001`). Należy je najpierw zwolnić (`lattice cam-disconnect -s <SN>`) lub ustawić
> je podczas nawiązywania połączenia (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Odczyt ekspozycji i wzmocnienia w trybie automatycznym

Podczas gdy w kamerze działa `ExposureAuto` (`GainAuto`), jej węzeł `ExposureTime` (`Gain`) zachowuje **ostatnią zapisaną wartość**, a nie aktualny stan kamery. W przypadku kamery, której backend pozostaje otwarty (`cam-connect`, `array-connect`, GUI), `lattice configure --get`, domyślny widok `lattice configure` oraz `lattice exposure` (bez flag) wyświetlają zatem wartość węzła, a pod nią informację o najnowszej klatce z kamery:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Odczytaj wartość klatki z notatki; wartość węzła to tylko ostatni zapis. Przez HTTP ta sama odpowiedź `POST /api/camera/<serial>/read-nodes` pozostawia `nodes` bez zmian i dodaje `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, najnowszej klatki) oraz `notes` (zdanie powyżej). Oba występują tylko wtedy, gdy tryb automatyczny nie ma wartości `Off` i znana jest klatka; kamera otwarta wyłącznie w celu odczytu nie ma klatki i nie otrzymuje żadnego z nich. `lattice gain` (bez flag) wyświetla tylko węzeł — użyj `lattice configure --get Gain` dla notatki.

##### Ekspozycja na elemencie tablicy

Podczas gdy tablica przesyła strumień, każdy jej element działa z włączoną opcją **`AcquisitionFrameRateEnable`**z częstotliwością klatek nieco wyższą od częstotliwości wyzwalania tablicy: jej okres wynosi okres wyzwalania pomniejszony o 15% (co najmniej 3 ms) lub maksymalną częstotliwość samej kamery, gdy częstotliwość wyzwalania jest już bliska tej wartości. `configure --get` na elemencie odczytuje zatem `AcquisitionFrameRateEnable` jako `True` i `AcquisitionFrameRate` przy tym ustawieniu; włączenie wyłącza się, gdy matryca zatrzymuje się.**Nie należy zapisywać tych dwóch węzłów na elemencie macierzy** — macierz synchronizuje je z własnym wyzwalaczem i nadpisuje je.

Ustawienie to stanowi również górną granicę czasu naświetlania elementu: około 85 ms przy wyzwalaczu 10 klatek na sekundę (okres 100 ms), przy **dowolnym obszarze zainteresowania (ROI) lub binningu**. Przed wersją 1.3.0 mały obszar zainteresowania (ROI) ograniczał ekspozycję elementu do okresu swobodnej pracy czujnika (4,9 ms przy rozdzielczości 768 × 304), niezależnie od tego, jak wolno wyzwalała się matryca.**Ręczne ustawienie `ExposureTime` dłuższe niż limit spowalnia wyzwalanie, aby je dostosować**, gdy jest zapisywane za pomocą `lattice exposure --set` lub `configure --set` (lub z poziomu GUI). Matryca najpierw otwiera limit tego elementu dla danej wartości i pozwala, by wolniejsza częstotliwość działała przez kilka okresów wyzwalania, więc zapis może potrwać o kilka sekund dłużej; wyzwalacz uruchamia się następnie nie szybciej, niż pozwala na to ekspozycja. W przypadku matrycy z automatyczną częstotliwością poprzednia częstotliwość powraca, gdy każdy element, którego ręczna ekspozycja spowolniła działanie, powróci do ekspozycji automatycznej. Wartość, której sama kamera nie może przyjąć, nadal jest odrzucana, a w komunikacie o błędzie podawany jest zakres, jaki akceptuje; automatyczna ekspozycja elementu zostaje przywrócona, jeśli została wyłączona wyłącznie na czas tego zapisu. Ekspozycja z receptury `apply` nie spowalnia wyzwalacza: jest utrzymywana na poziomie górnej granicy. Chloros nigdy nie skraca ręcznej ekspozycji, aby dostosować się do szybszego wyzwalania; zablokowana częstotliwość (`array-connect --target-fps F`), szybsza niż pozwala na to ekspozycja, pozostawia temu członkowi limit wystarczająco długi dla tej częstotliwości, a członek ten przyjmuje wyzwalania nie szybsze niż pozwala na to jego ekspozycja.

#### Kalibracja i czujniki

| Podpolecenie | Cel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibracja na podstawie celu odbicia. Przeszukiwanych jest maksymalnie `--attempts` klatek (domyślnie `30`); niekompletna ramka (utrata pakietu GVSP) kosztuje jedną próbę, a nie cały przebieg. Operacja kończy się niepowodzeniem przy wartości `no target detected in N attempts`, lub przy wartości `no complete frame in N attempts: <last error>`, gdy żadna ramka nie dotarła w całości. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Widmo i amplitudy pasmowe urządzenia DAQ-M, odczytane przez Bluetooth (BLE) za pośrednictwem zaplecza. `--mac` to własny adres MAC BLE czujnika i jest wymagany przy każdej z trzech czynności: nie ma wartości domyślnej, ponieważ każde urządzenie ma swój własny (`daq pool-discover --only ble` wyświetla ich listę); bez niego polecenie kończy działanie `1`. W przypadku braku akcji wyświetla pomoc. `--camera SERIAL` integruje wyświetlone wartości pasmowe poprzez własną, zmierzoną charakterystykę spektralną tej kamery zamiast krzywej floty modelu filtra — kamery RGB, z pakietem kalibracyjnym kamery załadowanym (podłączonym) lub zapisanym w pamięci podręcznej na tym komputerze. Nagłówek wskazuje, czyja krzywa została użyta: `Band magnitudes (camera <SERIAL>):` lub `Band magnitudes (model <filter>):`, gdy kamera nie posiada żadnej (nie jest to kamera RGB, lub nie ma załadowanego ani zapisane w pamięci podręcznej pakietu), a zamiast tego zastosowano krzywą typu „fleet”. Przez HTTP, `POST /api/dls/cli-read` pobiera `camera_serial` i zwraca `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Zastosuj korekcję winietowania do istniejących obrazów, przy czym każdy z nich ma własne płaskie pole obiektywu (z modelu EXIF). Polecenie `--lens-model` ma zastosowanie do obrazów, których model nie określa konkretnego aparatu; obraz, którego model jest sprzeczny z tym warunkiem, jest pomijany. |

#### Wiele kamer (sesje przejściowe)

Polecenie tymczasowe otwiera kamery na jeden przebieg i zamyka je po jego zakończeniu: `multi-capture` bez podłączonej trwałej tablicy, `multi-stream` oraz `multi-test` w tym przypadku, oraz `align-calibrate`, `align-apply`, `align-stream` i `index --live` poniżej. Dla wszystkich z nich obowiązują następujące zasady:

- **`--serials SN1,SN2,…` wybiera kamery.** Otwierane są tylko te kamery, a pierwsza z nich uruchamia wyzwalacz (w przypadku `align-calibrate`, `--reference`, jeśli zostały podane). Urządzenie szeregowe, które nie znajduje się w sieci, powoduje niepowodzenie przebiegu jeszcze przed otwarciem jakiejkolwiek kamery. Bez `--serials` otwierane są wszystkie kamery w sieci. Kamera, którą Chloros już posiada (interfejs graficzny, `cam-connect`, `array-connect`), odrzuca uruchomienie niezależnie od tego, czy `--serials` ją wymienia, czy nie: współdzieli ona sieć, dla której przeznaczony jest przebieg. (`index --live` bez `--serials` otwiera zamiast tego kamery włączone w swoim profilu: zobacz [Indeks / Matematyka roślinności](#index--vegetation-maths).)
- **Kamery współdzielą sieć w taki sam sposób, w jaki współdzieli ją `array-connect`.** Każda kamera jest dostosowana do swojej części przepustowości łącza głównego (opóźnienie międzypakietowe, a nie limit przepustowości kamery, którego minimalna wartość 31,25 MB/s jest już przekraczana przez cztery kamery w sieci 1 GbE), więc cztery kamery działają na łączu 1 Gbps. Gdy kamery nie mogą wysyłać danych jednocześnie z najwolniejszym tempem, jakie osiągają, są uruchamiane jedna po drugiej, każda z osobnym łączem, a polecenie wyświetla `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Gdy nawet jedna kamera na raz nie mieści się w przepustowości, operacja zostaje odrzucona, zanim jakakolwiek kamera rozpocznie transmisję: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, kod błędu HTTP 409, zakończenie `1`.
- **Każda kamera musi przesłać dane, w przeciwnym razie operacja zakończy się niepowodzeniem.** Kamera, która nie przesyła żadnej klatki, stanowi błąd: polecenie wyświetla komunikat `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, gdzie `<why>` oznacza błąd pobierania danych z tej kamery, a następnie wskazuje przyczyny, po czym kończy działanie z kodem `1`. Ramka, która dotarła z utratą pakietów GVSP (`IncompleteBufferError: GVSP packet loss …`), oznacza, że sieć nie obsługuje łącznej przepustowości kamer (`Every missing frame arrived with GVSP packet loss: …`): sprawdź prędkość łącza hosta oraz łącza uplink przełącznika (`lattice network`), użyj ramek jumbo, lub uruchom mniej kamer za pomocą polecenia `--serials`. Kamera, która nie przesłała żadnych danych, zazwyczaj nie otrzymuje sygnału wyzwalającego od kamery głównej: sprawdź jej kabel synchronizacyjny M8, a następnie uruchom polecenie `lattice multi-detect`. Działanie każdego polecenia w przypadku grupy ramek, w której brakuje kamery, jest opisane w odpowiednim wierszu.
- **Każda kamera jest przywracana do stanu, w jakim została znaleziona.** Po zakończeniu cyklu każda kamera odzyskuje stan, w jakim znajdowała się w momencie jej uruchomienia: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, oraz `LineMode` / `LineSource` na `Line2` i `Line3` (a układ usuwający zakłócenia z wejścia wyzwalającego dostarcza sygnał do linii urządzenia podrzędnego: `LineFilterSelector` / `LineFilterWidth`); jego `PtpEnable` (oraz zestaw `PtpSlaveOnly` / `PtpDomainNumber` z nim powiązany); `ExposureAuto` / `GainAuto` (oraz `ExposureTime` / `Gain`, gdzie była to wartość ręczna); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (oraz każdy `BalanceRatio`, w którym balans bieli był ustawiony ręcznie); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (oraz `Gamma`, w których włączono korekcję gamma), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; rozmiar pakietu (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); częstotliwość transmisji, `DeviceLinkThroughputLimitMode` (oraz `DeviceLinkThroughputLimit`, gdzie tryb wynosił `On`), opóźnienie międzypakietów `GevSCPD` oraz przesunięcie początkowe `GevSCFTD`, które tablica przypisuje każdej kamerze; dane fragmentów, które każdy strumień włącza (`ChunkModeActive`, każdy `ChunkEnable` w ramach swojego `ChunkSelector`); `AcquisitionFrameRateEnable` (oraz `AcquisitionFrameRate`, gdzie włącznik był włączony — gdy jest wyłączony, częstotliwość jest tylko do odczytu i odczytuje maksymalną wartość, na jaką pozwala taktowanie, więc nie jest to ustawienie). Zapisywane są tylko różnice: kamera najpierw się zatrzymała, a przed ekspozycją, która wymaga określonego okresu klatki, odczytuje się częstotliwość stymulacji i liczbę klatek na sekundę (kamera ogranicza `ExposureTime` do jednego okresu klatki). W przypadku `--keep-settings` okablowanie wyzwalacza i linii synchronizacji zostaje wyłączone, kamera pracuje w trybie swobodnym, a pozostałe dane są zapisywane w pamięci kamery zgodnie z ustawieniami sesji; `--clear-settings` przywraca tam ustawienia fabryczne.
- **Ekspozycja jest ustalana na podstawie sceny sprzed zachowania pierwszej klatki.** Cykl rejestracji pobiera zsynchronizowane grupy klatek i odrzuca je, ustawiając jednocześnie ekspozycję i wzmocnienie każdej kamery na podstawie tych klatek. Celem jest uzyskanie średniej jasności na obszarze docelowym kamery, przy czym nie więcej niż 1% klatki może być prześwietlone; scena z jasnymi światłami kończy się przed osiągnięciem wartości docelowej. Zazwyczaj zajmuje to 2–6 grup klatek i maksymalnie 5 s.
  - Każda kamera kieruje się na cel, od którego rozpoczyna się podłączona matryca, zgodnie z jej filtrem: TargetBrightness **40**w przypadku kamery wielospektralnej (RGN, OCN, NGB) oraz**60** w przypadku kamery RGB. Jest to połowa wartości docelowej ekspozycji wynoszącej 80 (trzy czwarte w przypadku kamery RGB), więc jasny panel odbijający światło rzadziej ulega przesterowaniu. `--brightness N` (0-255) ustawia natomiast wartość docelową dla każdej kamery (`multi-test` nie ma `--brightness`: jego automatyczna ekspozycja jest wyłączona). W logach backendu każda kamera ma nazwę odpowiadającą jej wartości docelowej: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Najpierw ustalana jest ekspozycja, do limitu automatycznej ekspozycji danej kamery. Wzmocnienie obejmuje tylko pozostałą część.
  - Ustalona ekspozycja i wzmocnienie są zachowywane przez cały czas trwania sesji, więc każda klatka z serii `multi-stream` jest rejestrowana przy tej samej ekspozycji. Rejestracja, która musi dostosowywać się do zmieniającego się oświetlenia, powinna odbywać się w połączonej tablicy (`array-connect`), której automatyczna ekspozycja dostosowuje się na bieżąco.
  - Sekwencje `--exposure` i `--gain` są zachowywane dokładnie w podanej postaci. Jeśli podano oba parametry, żadna klatka nie zostanie odrzucona.
  - Po zakończeniu serii automatyczna ekspozycja każdej kamery zostaje ponownie włączona.
  - Dziennik backendu zawiera informację o wyniku: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` lub `exposure NOT solved after …` wraz z przyczyną oraz `(not settled)` dla każdej kamery, która nie została uwzględniona. W takim przypadku klatki są nadal rejestrowane.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` zmienia limit 5 s; `0` wyłącza tę funkcję (zobacz [Zmienne środowiskowe](#environment-variables)).

**Gdy program Chloros obsługuje jakąkolwiek kamerę** (interfejs graficzny, `cam-connect` lub podłączoną macierz), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` oraz `index --live` odmawiają uruchomienia, ponieważ otwierają wszystkie kamery w sieci, a drugie otwarcie otwarcia zablokowanej kamery może spowodować awarię serwera. (`index --live` odmawia działania, gdy kamera, którą miałby otworzyć, jest zablokowana: jest to `--serials` lub kamery z jego profilu, a także dowolna kamera z profilu, który nie wymienia żadnej.) Backend zwraca kod błędu HTTP 409 `CAMERA_HELD`, podając nazwy wszystkich zablokowanych kamer oraz polecenia ich odblokowania (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), po czym polecenie kończy działanie `1`. W przypadku podłączonej matrycy `array-capture --continuous` zapisuje ją w miejsce `multi-stream`, a `array-capture` (domyślnie wyrównany) zapisuje wyrównany zestaw w miejsce `align-apply`; `multi-capture` i `align-calibrate` wykorzystują połączoną tablicę, która przechowuje ich kamery, zamiast je otwierać.

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Wyświetla listę wszystkich kamer pełniących rolę synchronizacyjną; w przypadku `--serials` — tylko te, a w przypadku `MASTER` — tylko pierwszą z nich. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Jedna zsynchronizowana klatka z każdej kamery. Domyślnie zapisuje **wszystkie typy eksportu**, gdy podłączona jest trwała macierz, oraz odczyt z modułu DAQ, który został użyty ([Odczyt z modułu DAQ podczas przechwytywania](#the-daq-reading-a-capture-saves)); tymczasowe rozwiązanie awaryjne bez matrycy jest**tylko poddane debayeringowi** (w przypadku pozostałych uruchom najpierw `array-connect`). `--brightness` dotyczy wyłącznie trybu awaryjnego: podłączona matryca zachowuje własną automatyczną ekspozycję. Tryb awaryjny nie zapisuje `.daq` (`--no-daq` / `--daq-tolerance-ms` nie mają tam żadnego wpływu), a `multi-capture` nie uwzględnia `--force-daq`. W trybie awaryjnym kamera, która nie dostarcza klatki, powoduje niepowodzenie przechwytywania (`Error: multi-capture failed: no frame from …`, zakończenie `1`) i żaden plik z tej grupy klatek nie jest zapisywany. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Strumień zsynchronizowanych klatek (tymczasowy), `--fps` domyślnie `3`, aż do naciśnięcia Ctrl+C lub `--count` grup. Grupa klatek, dla której kamera nie dostarczyła żadnych danych, nie jest zapisywana, a trzy z rzędu kończą strumień. Jeśli jakakolwiek grupa została pominięta, przebieg kończy się wyświetleniem `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` oraz `Error: the stream stopped: …` z nazwą kamery, która zakończyła strumień, a następnie program kończy działanie `1`X. Zatrzymuje się samoczynnie, gdy dysk się zapełni, wyświetla komunikat `Failed:` / `Disk full:` i kończy działanie z kodem `1`; każdy plik, którego nie udało się zapisać, powoduje również zamknięcie programu `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test synchronizacji czasowej GPIO (przejściowy), `--count` domyślne wyzwalacze `10`. Wyświetla sposób współdzielenia sieci (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` oraz wiersz `Serial emit:`, gdy kamery są wyzwalane jedna po drugiej, co powoduje, że znaczniki czasu zawierają przesunięcie), wiersz `Sync group: master SN:<serial> -> SN:<serial>, …` na każdą grupę synchronizacji (`(no slaves: not synced)` dla kamery bez kabla synchronizacyjnego, która jest wyzwalana samodzielnie), następnie jeden wiersz na każdy sygnał wyzwalający: `[  0] SN:<serial>=<timestamp> \| …`, przy czym `SN:<serial>=(no frame)` dotyczy kamery, która nie dostarczyła żadnych danych dla tego sygnału wyzwalającego. Następnie `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` i wiersz `SN:<serial>: mean <±x> us, jitter <y> us` na każdą kamerę, każda kamera względem urządzenia nadrzędnego w ramach jednego taktu: wyzwalacz, którego znaczniki łączą zegary urządzeń kamer oraz PTP (PTP wyłączony lub wciąż w trakcie synchronizacji) jest pomijany, a `Note:` je zlicza. Każda pominięta klatka powoduje niepowodzenie testu (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, wyjście `1`); trzy niekompletne sygnały wyzwalające z rzędu kończą test przedwcześnie. Test kończy się niepowodzeniem (wyjście `1`) również wtedy, gdy mniej niż dwa wyzwalacze zsynchronizowały wszystkie kamery w ramach jednego taktu (`… the clocks cannot be compared`) lub gdy zegary nigdy nie są zsynchronizowane: zegary PTP różnią się od zegara głównego o ponad 1 ms (plus ewentualne opóźnienie warstwy szeregowej) (`… PTP is not locked`) lub zegary urządzeń, których przesunięcie zmienia się o więcej niż tę wartość między wyzwoleniami. Uruchamia serwer PTP typu „grandmaster”, jeśli nie działa, włącza protokół PTP w każdej kamerze (tak jak robi to `array-connect`, niezależnie od konfiguracji sieci) i przed pierwszym sygnałem wyzwalającym czeka, aż zegar PTP każdej kamery zsynchronizuje się z serwerem typu „grandmaster” i ustabilizuje się: `PtpStatus` Slave na jednym serwerze głównym, z odchyleniem nieprzekraczającym 100 us i stabilny przez 3 s, a każda kamera zsynchronizowana w ten sposób, bez przerwy, przez 120 s — około 2 minut od włączenia zasilania. Zegar, który właśnie się zsynchronizował, wykazuje odchylenie do ~200 us przez około 2 minuty po odczytaniu statusu „Slave”, a własny zegar kamery `PtpOffsetFromMaster` tego nie pokazuje; po 120 s kamery osiągają odchylenie ~10 us od ustabilizowanego zegara. Kamera nadal pozostaje zsynchronizowana od czasu wcześniejszego testu, w którym pozostawiono włączony protokół PTP – tak to postrzega ten backend – liczy od momentu tego zsynchronizowania, więc test przeprowadzony bezpośrednio po innym nie czeka ponownie; kamera, którą test włącza, lub taka, która została wykryta z włączonym PTP, a której ten backend nie widział w stanie zsynchronizowania (włączona gdzie indziej lub przed ponownym uruchomieniem backendu), czeka swoje 120 s, a w tablicy obie kamery, które właśnie się zsynchronizowały, nadal czekają na swoje. Kamera, która opuszcza tryb Slave, przekracza 100 us lub śledzi innego grandmastera przez 3 odczyty z rzędu (1,5 s) podczas oczekiwania, rozpoczyna odliczanie od nowa; jeden lub dwa takie odczyty są rejestrowane (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), a odliczanie trwa dalej; identyfikator grandmastera, którego nie da się odczytać, traktowany jest jako nieznany, a nie jako zmiana. Jednoczesne ponowne rozpoczęcie liczenia przez wszystkie kamery (w ciągu jednej sekundy) oznacza zmianę czasu „grandmastera”, a nie kamer: log zaplecza zawiera wpis `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` wymienia to w `grandmaster_steps`, a wyczerpanie się czasu oczekiwania rozpoczyna błąd od `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limit nie jest w tym przypadku przedłużany. `--ptp-settle SECONDS` ustawia czas ustabilizowania dla danego przebiegu (0–600; `0` oznacza brak oczekiwania na ustabilizowanie). Czas oczekiwania jest ograniczony do czasu ustabilizowania plus 90 s (domyślnie 210 s, co najmniej 90 s), a polecenie czeka na backend 240 s plus czas ustabilizowania (domyślnie 360 s). Wyświetla komunikat `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, a w dzienniku modułu zaplecza co 5 s pojawia się wiersz zawierający stan i przesunięcie każdej kamery, czas trwania blokady oraz pozostały czas: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Każda kamera włącza się z wyłączonym PTP; po czym zegary synchronizują się w ciągu około 10 s. Następnie każda kamera jest przywracana do stanu, w jakim została znaleziona podczas testu — jej PTP, automatyczna ekspozycja i wzmocnienie są wyłączone na czas trwania testu, a pozostałe ustawienia (o ile nie pojawi się komunikat `--keep-settings`) — tak więc kamera, w której pozostawiono włączony PTP (`lattice configure -s <serial> --set PtpEnable=True` lub przebieg z `--keep-settings`), pozostaje zsynchronizowana, a gdy ten moduł zaplecza stwierdzi, że była zsynchronizowana przez wymagany czas ustabilizowania, późniejszy test nie będzie na nią czekał. Jeśli zegary nie ustabilizują się w trakcie oczekiwania, test kończy się niepowodzeniem przed wyzwoleniem (`PTP: NOT locked …`, wyjście `1`) i podaje instrukcję postępowania: kamera, która nigdy nie została zsynchronizowana (`PtpStatus Listening`), oznacza, że serwer nadrzędny nie ma do niej dostępu (`time-sync status`); kamera, która ciągle się rozłączała, jest oznaczona czasem, przez jaki pozostawała zsynchronizowana (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); zegary wciąż się zsynchronizowują: sprawdź `time-sync status`, a aby zsynchronizować je przed testem, włącz PTP i pozostaw włączone (`lattice configure -s <serial> --set PtpEnable=True` dla każdej kamery lub uruchom test z `--keep-settings`; test przywraca znalezione ustawienia, więc pozostaje włączony), a następnie uruchom test. |
| `lattice multi-detect [--line LINE] [--json]` | Znajdź kamery w linii synchronizacyjnej M8 oraz domyślny serwer: kamerę RGB (model kończący się na `-FRGB`), jeśli jest podłączona kablem, w przeciwnym razie pierwszą podłączoną kablem kamerę według numeru seryjnego. Kable M8 łączą się w jednym punkcie, więc **dowolna podłączona kamera może pełnić rolę urządzenia nadrzędnego** — nadrukowany numer `MASTER` jest wartością domyślną, a nie odzwierciedla faktycznego okablowania. Każda kamera po kolei steruje linią, a pozostałe są sprawdzane pod kątem jej impulsu; kamera już znaleziona w podłączonej grupie nie jest ponownie sprawdzana, więc skanowanie w pełni okablowanego zestawu zajmuje jedną rundę. W przypadku braku `--line` skanuje `Line2`, a następnie `Line3`. Wyświetla `MASTER: SN:…`, po jednym `SLAVE: SN:…` dla każdej kolejnej podłączonej kamery oraz `(probed on <line>)`; `--json` dodaje pełny wynik (`master`, `slaves`, `no_cable` — kamery, do których impuls nie dotarł — `line` oraz dla każdej kamery `results`). Po zakończeniu każda kamera odzyskuje swoje poprzednie ustawienia wyzwalacza i linii synchronizacji (te same węzły, co powyżej). |

#### Wyrównanie

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus pokrętła detektora/dopasowania `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, pokrętła RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinacja wielu klatek `[--averaging mean\|median\|inlier_weighted]`, ograniczenia geometryczne `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, ograniczenia przestrzenne `[--roi X0,Y0,X1,Y1] [--mask PATH]` oraznadrzędnego nadpisania `[--per-cam-override SN:KEY=VALUE]` (powtarzalne) | Oblicz profil dopasowania na podstawie obrazu z kamer na żywo. Ustawienia domyślne: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (w folderze, z którego uruchamiasz program), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` ma własne ustawienia domyślne; zobacz [jego tabeli](#project-align-calibrate-options).) Domyślnym ustawieniem dla `--prefilter` jest `gradient` (mapa krawędzi; odpowiada narzędziu do wyrównywania w interfejsie graficznym/tablicowym — krawędzie zachowują się w różnych pasmach spektralnych). `--matcher flann` przynosi korzyści przy liczbie cech powyżej ~5000; `--averaging median` jest odporny na jedno złe ujęcie, `inlier_weighted` waży według liczby dopasowań; `--lock-scale` rzutuje na najbliższy obrót (bez skali), `--lock-axis` zeruje jeden składnik translacji; `--mask` ma zastosowanie do każdej kamery (użyj `--per-cam-override` dla ustawień indywidualnych dla każdej kamery, np. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odmawia zapisania kalibracji, której RMS reprojekcji przekracza próg. `--method manual` pobiera swoje punkty z `--points PATH`: obiekt JSON `{"<serial>": [[x, y], ...], ...}` podający współrzędne pikselowe tych samych elementów charakterystycznych dla każdej kamery, w tym kamery referencyjnej (co najmniej 2 na kamerę); każdy z elementów `--frames` jest obliczany na podstawie tych punktów. Gdy kamery znajdują się w połączonej tablicy, kalibracja wykorzystuje klatki na żywo z tej tablicy, a profil staje się również wyrównaniem tej tablicy dla jej ujęć. W przeciwnym razie otwiera się [tablicę tymczasową](#multi-camera-transient-sessions): kamera, która nie dostarcza klatki w żadnym z `--frames`, nie przechodzi kalibracji (wyjście z `1` z podaniem nazwy), a profil nie jest zapisywany. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Przechwytuje jedną wyrównaną klatkę wielopasmową. `--bit-depth` domyślnie dopasowuje się do kamery; `--no-crop` zachowuje pełną klatkę (wypełniając czarną przestrzenią); `--interpolation` (domyślnie `linear`) oraz `--border-mode`/`--border-value` (domyślnie `constant`/0) kontrolują zniekształcenie procesora — ścieżka procesora graficznego jest w każdym przypadku dwuliniowa. Otwiera [tablicę przejściową](#multi-camera-transient-sessions): kamera, która nie dostarcza klatek, powoduje niepowodzenie przechwytywania (wyjście `1`, z podaniem nazwy), a nic nie jest zapisywane. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Strumieniowo wyrównane klatki wielopasmowe (te same regulatory warpu co w `align-apply`), domyślne ustawienie `--fps` to `3`. Otwiera [tablicę przejściową](#multi-camera-transient-sessions): grupa klatek, dla której kamera nie dostarczyła żadnych danych, nie jest zapisywana, a trzy takie klatki z rzędu kończą strumień. Jeśli jakakolwiek grupa została pominięta, przebieg kończy się wyświetleniem komunikatów `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` i `Error: …`, a następnie kończy działanie z kodem `1`. Strumień, który kończy się z powodu jakiegokolwiek innego błędu, również wyświetla komunikaty `Error: …` i kończy działanie z kodem `1`. |
| `lattice align-info --profile PATH [--json]` | Wyświetla szczegóły profilu. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Zmienia kolejność warstw. |

#### Indeks / Obliczenia dotyczące roślinności

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` przypisuje symbole z presetu do pasm** (NDVI: `red`, `nir`), po jednym `--channel SYMBOL=BAND` dla każdego.

- **`--live`: tylko dla symbolu, na który stos nie może odpowiedzieć.** Podobnie jak w przypadku `--input`, pominięty symbol presetu przyjmuje to jedno pasmo ze stosu (pasma włączonych kamer w profilu), którego nazwa zaczyna się od niego. Jest to sprawdzane przed otwarciem jakiejkolwiek kamery: symbol, na który nie odpowiada żadne pasmo lub odpowiada kilka pasm, jest wówczas odrzucany, a po ich wymienieniu następuje wyjście z `1`. Niestandardowy `--formula` wymaga `--channel` dla każdego symbolu.
- **Produkty `--live` podają, do kogo należą.** Produkty `index.tif` i `index_LUT.png` zawierają informacje o kamerze ze stosu (marka, model, numer seryjny), czasie wykonania zdjęcia (czas naświetlania kamery referencyjnej, UTC) oraz obiektyw, a oba określają indeks jako swoje jedno pasmo (`XMP-Camera:BandName` `NDVI`; kolorowy plik PNG określa indeks, a nie jego kanały kolorów), tak jak robi to produkt `--input`. Indeks utworzony z pasm z kilku kamer zawiera zamiast tego nazwy każdego z nich, a jego nazwa to `Model` — zobacz [Pliki utworzone z kilku kamer](#files-made-from-several-cameras).
- **`--input`: dotyczy wyłącznie symbolu, na który plik nie może odpowiedzieć.** Pominięty symbol przyjmuje nazwę jednego pasma z pliku, którego nazwa zaczyna się od tego symbolu: `red` przyjmuje `Red_660` lub `Red_660_RGN`, a `nir` przyjmuje `NIR_850`. Nazwy pochodzą z samego pliku: z listy pasm stosu lub, w przypadku rejestracji promieniowania / odbicia Chloros lub produktu `process` – z jego tagów `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Zatem `lattice index --input <reflectance capture> --preset NDVI` nie wymaga `--channel`.
- **Nie rozróżnia się dwóch pasm odpowiadających jednemu symbolowi** (stos z 9 kamerami ma dwa pasma NIR): należy je nazwać. Symbol, któremu nic nie odpowiada, jest odrzucany za pomocą `channel_map missing entries`, a następnie należy przejść do `1`.**Nazwy pasm w zestawie** odzwierciedlają filtr kamery: pasma kamery M3C to `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` w przypadku kamery typu RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` w trybie OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` w trybie RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` na NGB — a pojedynczym paśmie kamery M3Mpasmo pojedyncze to `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` zawiera listę pasm profilu.

Pełny zestaw flag: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (możliwość powtórzenia), `--capture-level raw|debayered|radiance|reflectance|unknown` (zastępuje poziom przechwytywania zapisany w źródłowym pliku TIFF; domyślnie: odczyt z metadanych pliku TIFF, z awaryjnym wykorzystaniem wartości  z pliku Chloros, więc przechwytywanie odbicia zwraca `Chloros:ProcessingLevel`, a przechwytywanie reflektancji zwraca `reflectance`), `--output PATH` / `-o` (w trybie offline: ścieżka; w przypadku plików graficznych nazwy plików są tworzone na podstawie ich rdzenia — `-o ndvi_FRGN.png` zapisuje `ndvi_FRGN.tif` i `ndvi_FRGN_LUT.png` — a bez rozszerzenia jest to prefiks, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; domyślnie obok danych wejściowych, jak `<input name>_<preset>`; z `--live` i bez `-o`, folder `output/` w folderze, z którego uruchamiasz polecenie), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. W przypadku pliku `--live` pokrętła zniekształcenia wyrównania mają również zastosowanie: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (cel automatycznej ekspozycji kamer, patrz reguły dotyczące sesji przejściowych), `--serials SN1,SN2,…`. `--live` otwiera [tablicę sesji przejściowych](#sesje-przejściowe-wielokamerowe): kamera, która nie dostarcza żadnej klatki, powoduje niepowodzenie przebiegu (wyjście z `1`, nadając jej nazwę), a stos ani indeks nie są zapisywane. Otwiera `--serials` (pierwsza uruchamia wyzwalacz), w przeciwnym razie kamery z profilu, których wpis `layer_order` jest włączony, najpierw ta referencyjna — nawet jeśli jej pasma są wyłączone, ponieważ pozostałe są do niej dostosowane do niej i to ona uruchamia wyzwalacz. Kamera, której pasma są wyłączone, nigdy nie jest kalibrowana, więc kamera RGB wyłączona w profilu nie powoduje już niepowodzenia indeksu odbicia (`no-absolute-radiance-scale`). Profil, który nie wymienia żadnych kamer, otwiera każdą kamerę w sieci, tak jak poprzednio.

Każdy formularz działa na backendzie, więc musi on być uruchomiony: uruchom go z poziomu interfejsu graficznego Chloros lub za pomocą polecenia wyświetlanego przez CLI, gdy nie może się z nim połączyć. W trybie offline backend odczytuje `--input` i samodzielnie zapisuje plik TIFF z wartościami zmiennoprzecinkowymi oraz pokolorowany plik PNG, więc względny `--input` lub `-o` odnosi się do folderu, z którego uruchamiasz polecenie (zobacz [Konwencje](#conventions)). Oba pliki generowane w trybie offline zawierają dane rejestru aparatu z pliku wejściowego, podobnie jak eksport indeksu o nazwie `process`: marka / model / numer seryjny, GPS, tagi obiektywu, czas wykonania zdjęcia (z pliku wejściowego; jeśli jego dane EXIF nie podają tej informacji, to moment naświetlenia zarejestrowany przez aparat lub znacznik czasu w nazwie pliku, w czasie UTC) oraz identyfikator kalibracji MAPIR — nigdy kalibracji raw-DN, która nie ma zastosowania do indeksu — a także `Chloros:ProcessingLevel=index` oraz wyrażenie indeksu; zmienna typu float TIFF określa jedno pasmo, czyli ustawienie wstępne. `--list-presets` i `--list-gradients` wyświetlają tabele ustawień wstępnych i gradientów modułu zaplecza. Formuła odrzucona przez backend lub pasmo `--channel`, którego plik TIFF nie zawiera, powoduje wyjście z funkcji `1` z podaniem przyczyny podanej przez backend.

> **W symbolach `--channel` rozróżniana jest wielkość liter.** Strona symbolu musi dokładnie odpowiadać nazwom kanałów presetu (presety używają małych liter, np. NDVI = `red`,`nir` — sprawdź `--list-presets`), a strona pasma musi odpowiadać nazwie pasma w wyrównanym stosie (lub być indeksem pasma liczonym od 0 w trybie offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` działa; `--channel RED=660` kończy się niepowodzeniem z błędem `channel_map missing entries` .

#### `lattice index` — maski stosów wielopasmowych

W `--capture-level radiance` lub `reflectance` funkcja rozdzielania 3×3 każdej kamery może ograniczyć pasmo do 0 (zobacz [Maska ograniczenia rozdzielania](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Stos wielopasmowy zawiera pasma z kilku kamer, a każda kamera ogranicza swoje własne, więc `--live --save-multiband` zapisuje **jedną maskę na każdą kamerę źródłową** obok stosu:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Każda maska jest jednopasmowa `uint8` na siatce pikselowej **stosu** (po zastosowaniu tej samej transformacji wyrównującej i kadrowania), z bitami właściwymi dla tej kamery: bit 0 = pasmo jej kanału R, bit 1 = G, bit 2 = B. Pasmo stosu pobrane z kanału *c* kamery jest ograniczane tam, gdzie ustawiony jest bit *c* tej kamery (R = 0, G = 1, B = 2).
- Wbudowana informacja o pochodzeniu każdego pasma określa nazwę maski tej kamery (`unmix_clamp_mask`, względem folderu stosu, lub `none`) oraz bit do odczytu (`unmix_clamp_mask_bit`).
- Kamera, której proces oddzielania kolorów nie ograniczył żadnych wartości, nie otrzymuje pliku; kamera monochromatyczna lub RGB nie przeprowadza oddzielania kolorów i również nie otrzymuje pliku. Interfejs CLI wyświetla każdą maskę pod stosem (`clamp mask SN…: …`); odpowiedź wymienia je jako `clamp_masks`.
- Indeksowy plik TIFF/PNG z tego samego przebiegu jest obliczany na podstawie tego samego stosu, na tej samej siatce, więc maski opisują również ten plik. `--capture-level raw` nie przeprowadza operacji unmix i nie zapisuje żadnej maski.
- Stos i indeks znajdują się bezpośrednio w `--output`, więc maski trafiają do własnego folderu `clamp_masks/` w ramach `--output` — nigdy powyżej folderu, który nazwałeś. W tym folderze mieszają się różne rodzaje produktów (stos, indeks, render w kolorze): wybierz z niego pliki, nie przekazuj całego folderu do oprogramowania do fotogrametrii.

#### Trwałe połączenia (Smart-Prep, przepływ odpowiadający interfejsowi graficznemu)

Te polecenia utrzymują kamery otwarte w puli zaplecza między wywołaniami CLI.

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Dodaj jedną kamerę do puli (pojedyncza kamera, bez tablicy). [Opcje wspólne](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` oraz `--packet-size` (`standard` = 1500, `jumbo` = 9000 lub bajtów; sondy `auto`) są stosowane przy podłączeniu (przełączniki `--exposure` lub `--gain` wyłączają tryb automatyczny). Zwraca wynik, gdy strumień z kamery zostanie uruchomiony (czekając na to około 15 s, a następnie 10 s na pierwszy pełny bufor) i wyświetla `stream: streaming` lub `stream: no_frames -- connected, no frames yet: …`, gdy strumień nie dostarczył niczego — w przypadku kamery korzystającej z pakietów jumbo, z zaleceniem ponownego połączenia za pomocą `--packet-size standard`. Podczas gdy jedna kamera nawiązuje połączenie (pierwsze połączenie pobiera jej kalibrację, co trwa około 70 s), inne polecenie wymagające kamery czeka na nią maksymalnie 5 s, następnie kończy się niepowodzeniem z kodem HTTP 503 `CAMERA_POOL_BUSY`, podając nazwę kamery łączącej się; uruchom to ponownie za chwilę. `lattice status` oraz lista kamer nigdy nie czekają: kamera w trakcie łączenia jest wyświetlana jako łącząca się, wraz z fazą połączenia. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Zwolnij. Kamera zostaje przywrócona do stanu, w jakim została wykryta podczas łączenia (okablowanie wyzwalacza, PTP, ustawienia); `--keep-settings` zapisuje natomiast jej ustawienia we własnej pamięci, a `--clear-settings` resetuje tę pamięć do ustawień fabrycznych (zobacz [Pamięć kamery](#camera-memory----keep-settings----clear-settings)). Kamera będąca członkiem macierzy zwalnia całą macierz. |
| `lattice cam-list` | Wyświetla listę kamer w puli. |
| **`lattice array-connect`**|**Podłącz trwałą, zsynchronizowaną macierz (zalecany punkt wejścia).** Uruchamia pełny proces przygotowania w interfejsie graficznym. Trwa to około minuty dla 9 kamer; zaraz po włączeniu kamer może to potrwać kilka minut. Podczas oczekiwania wyświetla każdy krok osiągnięty przez backend, jako `    <s> s: <step>` (na przykład `    41 s: open SN:214000535 (4 of 9)`), a dziennik serwera zawiera jedną linię czasową na każdy etap (`[array-connect <id>] <step>: <s> s …`) oraz ostatnią linię z najwolniejszymi etapami. Czeka maksymalnie 600 s. Połączenie, które nie zostało nawiązane do tego czasu, jest cofane przez backend: każda otwarta kamera jest przywracana do stanu, w jakim została znaleziona, i zamykana, polecenie wyświetla `Error:` wraz z krokiem, do którego dotarło, a następnie kończy działanie `1`, przy czym żadna macierz nie zostaje podłączona (kod HTTP 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C anuluje połączenie w ten sam sposób (`Interrupted: …`); jeśli macierz była już połączona, wiersz informuje o tym i podaje numer `array-disconnect`, który ją zwalnia. Polecenie przerwane w trakcie oczekiwania (`kill -9`) powoduje, że połączenie zostanie cofnięte, gdy serwer zaplecza nie otrzyma od niego sygnału przez 30 s. Macierz, która jest uruchomiona, ale nie ma jeszcze zsynchronizowanej grupy ramek, nadal jest zgłaszana jako podłączona, a jej strumień to `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Zwolnij tablicę. Każda kamera pozostaje w trybie swobodnej pracy (`TriggerMode` `Off`), a wszystkie pozostałe parametry pozostają takie, jak w momencie nawiązania połączenia — ustawienia wyzwalacza i linii synchronizacji (ustawienia urządzenia nadrzędnego`TriggerSource` przywraca się do poprzedniego stanu, a jej linia synchronizacji z powrotem do trybu wejściowego), jej PTP oraz ustawienia kamery zmienione podczas sesji — dzięki czemu polecenie dotyczące pojedynczej kamery na urządzeniu członkowskim działa natychmiast. `--keep-settings` zapisuje ustawienia każdej kameryustawienia kamery w stanie, w jakim pozostawiła je sesja, w pamięci własnej kamery; `--clear-settings` resetuje tę pamięć do ustawień fabrycznych. Zobacz [Pamięć kamery](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Wyświetl listę podłączonych macierzy. |
| `lattice array-status [--array-id ID]` | Liczba klatek na sekundę na żywo, PTP, ostatni błąd. `ptp_enabled` ma wartość `True` tylko wtedy, gdy każdy element ma włączoną obsługę PTP; `ptp_off:` wskazuje element, który jej nie obsługuje. Linia `state:` odczytuje `restarting (applying settings)` podczas ponownego uruchamiania ustawień, `recovering (…)` podczas ponownego otwierania członka, który stracił zasilanie (patrz uwaga poniżej), `stalled (…)` podczas działania pętli przechwytywania, która nie publikuje żadnej grupy ramek, oraz `stopped (…)` po zakończeniu działania pętli przechwytywania (należy ponownie podłączyć macierz). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno zsynchronizowane przechwycenie z macierzy na żywo — Pojedyncze / Ciągłe / Interwałowe / Najszybszy. **Domyślnie `all`**(jeden plik na każdy odpowiedni typ eksportu na kamerę),**wyrównane** tak samo jak przechwytywanie w aplikacji (`--no-aligned` dla plików natywnych). Pominięte kamery (np. RGB, wykluczone z pomiaru promieniowania/odbicia) są zgłaszane z kodem `Skipped: SN:<serial> (<reason>)`; odczyt z modułu DAQ użytego do przechwytywania jest zapisywany razem z danymi, dołączany jest również zapis zawierający wyłącznie dane surowe, a całość jest zgłaszana z kodem `DAQ: <path>` ([Moduł DAQ odczytujący przechwycony zapis zapisuje](#the-daq-reading-a-capture-saves)). Zobacz [Tryby przechwytywania, rejestratory i przetwarzanie offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Nagrywaj na żywo połączony widok indeksowy jako wideo/GIF (jakość monitoringu; przy zamkniętym oknie GUI backend renderuje NDVI macierzy lub `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Seria zdjęć w formacie surowym Bayer o wysokiej liczbie klatek na sekundę (klasa analityczna; ponowne przetwarzanie w trybie offline). Podobnie jak `array-record` i `array-capture --continuous`, proces ten zgłasza się do serwera przy każdym odpytywaniu: w przypadku przerwania bez zatrzymania (`kill -9`), serwer zaplecza zatrzymuje go 30 s po ostatnim zgłoszeniu (`stop_reason` `client_gone`; manifest serii zdjęć jest finalizowany i rozpoczyna się tworzenie połączonego indeksu, tak jak w przypadku zatrzymania), a tablica pozostaje podłączona. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Przetworzenie zapisanej surowej serii zdjęć na skalibrowane nagrania wideo. |

> **Kamera, która straciła zasilanie lub połączenie, wraca do działania samodzielnie.** Gdy jeden z elementów przestaje odpowiadać (przełączenie zasilania powoduje ponowne uruchomienie każdej zasilanej przez nie kamery), macierz czeka na niego, ponownie otwiera połączenie, gdy ten znów odpowie, i przywraca jej ustawienia określone przez `array-connect`: PTP, sygnał wyzwalający i linię synchronizacji, taktowanie sieciowe, format pikseli i rozmiar klatki oraz automatyczną ekspozycję. Następnie wznawia transmisję bez wprowadzania nowych ustawień `array-connect`. Tymczasem `array-status` pokazuje `state: recovering (...)`, a `last_error` wskazuje, na którą kamerę czeka; następnie wyświetla `re-opened:`. Aparat, który się zrestartował, wraca z wyłączonym PTP, więc dopóki nie zostanie ponownie otwarty, `ptp_enabled` to `False`, a `ptp_off:` podaje jego nazwę.

> **Kamera zatrzymana nigdy nie jest otwierana po raz drugi.** Drugie otwarcie zatrzymuje strumień podmiotu zatrzymującego i nadpisuje jego ustawienia, co może spowodować awarię zaplecza. Tak więc w przypadku kamery utrzymywanej przez pulę (interfejs GUI lub `cam-connect`), elementu tablicy lub kamery utrzymywanej przez zadanie w tle (uruchomiony proces `lattice continuous`, `multi-stream` lub `align-stream`, własnego lub innego klienta), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, jedno-shot `white-balance` oraz `presets --apply` odrzucają żądanie z kodem HTTP 409 `CAMERA_HELD`, podając nazwę właściciela i polecenie zwolnienia (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` lub, w przypadku zadania, jego identyfikator i osoba, która je uruchomiła) oraz zakończenie działania `1`. To samo dotyczy kamery, z której w danej chwili korzysta inne polecenie (`multi-test` lub `multi-capture` otwierające jej kamery, kolejne polecenie jednorazowe) – dla każdego polecenia tutaj: komunikat o błędzie nakazuje poczekać na zakończenie działania tego polecenia. `lattice power` bez `--sleep` odczytuje z własnego uchwytu posiadacza, jeśli taki istnieje (pula, tablica, otwarta kamera zadania), i zostaje odrzucony w przypadku kamery używanej przez inne polecenie. Każda odpowiedź `CAMERA_HELD` zawiera kod zarówno w `code`, jak i w `error_code`. Informacje na temat poleceń otwierających każdą kamerę można znaleźć w sekcji [Wiele kamer (sesje przejściowe)](#multi-camera-transient-sessions).

##### Opcje `array-connect`

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatyczne wykrywanie wszystkich kamer LATTICE (wymagane ≥2) | Pierwsza kamera szeregowa jest MASTEREM. W przypadku pominięcia tej opcji wykrywanie ogranicza się do modeli LATTICE (M3C / M3M) i łączy je wszystkie. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linia synchronizacji GPIO. |
| `--target-fps F` | auto | Częstotliwość wyzwalania urządzenia MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Zastąpienie selektora poziomu. |
| `--wire-ceiling-mbps MB_PER_S` | wykrywane automatycznie | **Trwała przepustowość łącza hosta w MB/s — wartość, od której zależy cała alokacja macierzy.** Należy go zmniejszyć, gdy macierz zgłasza uszkodzone ramki GVSP: wartość automatyczna jest wyznaczana na podstawie ogłaszanej przez kartę sieciową prędkości łącza, która zawyża prędkość adapterów USB, wąskich pasm PCIe oraz obciążonych struktur współdzielonych. Jest ona zapisywana w bloku przechwytywania macierzy projektu, więc ponowne otwarcie / ponowne połączenie przez CLI/SDK przywraca tę wartość. Zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | brak (pełny czujnik) | Binning sprzętowy (zawsze średnia). |
| `--width N` / `--height N` / `--pixel-format FMT` | pełny czujnik; format dla poszczególnych kamer | Wyraźnie określony rozmiar klatki lub format pikseli dla każdej kamery, zamiast pełnego czujnika i wyboru formatu dla poszczególnych kamer (Mono12 / BayerRG12). |
| `--no-recommend` | wyłączone | Pomiń etap analizy sieci (każda kamera zachowuje wówczas swój aktualny rozmiar zamiast być ustawiona na pełny czujnik). |
| `--no-ptp` | wyłączone | Wyłącz PTP (znaczniki czasu między kamerami nie będą wówczas **porównywalne**). |
| `--packed {auto,on,off}` | `auto` | Przesyłaj 12-bitowe klatki w postaci spakowanej i rozpakowuj je na tym komputerze — więcej klatek na sekundę przy pełnym rozmiarze matrycy, identyczne dane. `auto` pakuje tylko wtedy, gdy jest to korzystne i ten komputer ma na to wystarczające zasoby; `off` nigdy nie pakuje. Zobacz [Pakowany transport 12-bitowy](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Rozmiar pakietu GVSP dla każdej kamery w macierzy. `auto` pozwala każdej kamerze na sondowanie sieci; `jumbo` = 9000, `standard` = 1500 lub rozmiar w bajtach blokuje to. Przed wersją 1.3.0 `array-connect` akceptował tę flagę i ignorował ją. |

> **Domyślnie pełna rozdzielczość czujnika.** Bez `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` ustawia każdą kamerę na pełny czujnik — bez
> binningu, bez kadrowania obszaru zainteresowania (ROI) — i pozwala, by liczba klatek na sekundę spadła do poziomu, jaki zapewnia ten komputer i
> sieć, zamiast zmniejszać klatki w celu utrzymania częstotliwości.
> Wiersz analizy sieci pokazuje przewidywaną częstotliwość, a następny wiersz zawiera listę
> szybszych opcji (binning lub mniejszy obszar ROI) wraz z ich przewidywanymi częstotliwościami; należy pominąć
> `--binning` lub `--width`/`--height`, aby wybrać jedną z nich. Program Connect nigdy nie stosuje binningu ani kadrowania
> z własnej inicjatywy.

##### Pakowany transport 12-bitowy (`--packed`)

Przy pełnym obciążeniu czujnika częstotliwość odświeżania macierzy zależy od tego, jak szybko każda kamera może wysyłać
swoje klatki. Formaty 12-bitowe (`BayerRG12`, `Mono12`) są zazwyczaj przesyłane jako 16-bitowe
słowa; transport **spakowany** wysyła te same 12 bitów w 1,5 bajta na piksel i
rozpakowuje je na tym komputerze, dzięki czemu wysłanie każdej klatki zajmuje o 25% mniej czasu —
około **33% więcej klatek na sekundę przy pełnym wykorzystaniu matrycy** (wynik pomiaru: 10,7 → 14,0 klatek na sekundę
w układzie 9 kamer). **Zapisane dane są identyczne**: te same wartości, te same
16-bitowe pliki, te same metadane.

- `auto` (domyślnie) kompresuje tylko wtedy, gdy jest to korzystne **i** gdy ten komputer jest w stanie
  sobie pozwolić na dekompresję. Po nawiązaniu połączenia moduł zaplecza mierzy własną prędkość rozpakowywania, liczbę rdzeni procesora,
  aktualne obciążenie procesora oraz przepustowość sieci, dzięki czemu komputer stacjonarny z wolnymi rdzeniami kompresuje
  przy pełnym wykorzystaniu matrycy, podczas gdy komputer zajęty lub o niskiej mocy przesyła dane w postaci rozpakowanej. Małe obszary zainteresowania (ROI) i klatki połączone w pakiety zyskują
  niewiele.
- `off` nigdy nie kompresuje — należy go używać w celu wyłączenia kompresji podczas diagnozowania lub na
  komputerze, który musi zachować całą moc procesora na inne zadania.
- `on` zawsze kompresuje formaty 12-bitowe.

Wynik polecenia „connect” pokazuje decyzję i jej uzasadnienie (`skompresowano 12-bit: w trybie (auto:
prognozowane 11,0 -> 14,4 klatek na sekundę, …)`); `/api/camera/array/status` zgłasza
`packed_transport` i `packed_decision`. Aby ustawić domyślną wartość dla całej maszyny
zamiast za każdym razem przekazywać flagę, należy ustawić `CHLOROS_ARRAY_PACKED12=off` (lub
`on`) w środowisku backendu — jednak jawne ustawienie `--packed on|off` nadal ma pierwszeństwo.
Formaty 8-bitowe i 16-bitowe nigdy nie są pakowane.

### Jeden rekord projektu, wszystkie tryby

Folder projektu zawiera ustawienia wszystkich urządzeń: `cameras.json` — dla każdej kamery (`cameras[].settings`, dla pojedynczej kamery i dla każdego elementu macierzy) oraz ustawienia dotyczące całej macierzy (`arrays[].capture`: format, obszar zainteresowania (ROI), binning, częstotliwość wyzwalania, limit przepustowości) — a także `sensors.json`, każdego czujnika światła (czas integracji, średnia klatki, limit). Aplikacja, interfejs CLI, zestaw SDK oraz serwer MCP odczytują i zapisują te same pliki w ten sam sposób:

- **Urządzenie podłączone w ramach projektu jest w nim zapisywane i rejestruje w nim każdą zmianę.** „W ramach projektu” oznacza, że projekt jest otwarty w zapleczu (aplikacji) lub — w przypadku SDK — poprzez plik `ChlorosProject`, który określa nazwę własnego folderu. Kamera, matryca lub czujnik światła, których projekt jeszcze nie zawiera, są dodawane do jego plików w momencie połączenia (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplikacja, SDK, `chloros-mcp`) wraz z jego ustawieniami. Zmiana z dowolnego trybu (`lattice exposure`, `gain`, `configure --set`, `resolution`, ustawienia matrycyustawień i częstotliwości wyzwalania, ograniczenia czujnika światła, elementów sterujących w aplikacji) zostaje zapisana w plikach tego projektu chwilę później. Ustawienia ekspozycji, wzmocnienia lub wartości docelowej AE w aparacie są zapisywane jako przytrzymane, więc ponowne otwarcie projektu powoduje ich ponowne przytrzymanie.
- **Otwarcie projektu powoduje ustawienie urządzeń zgodnie z nim**, w każdym trybie: aparat, matryca lub czujnik światła podłączone bez własnych ustawień otrzymują ustawienia zapisane w projekcie (`lattice cam-connect` / `array-connect` przy otwartym projekcie, `chloros-cli project connect`, `open_project().connect_all()`, ponowne otwarcie aplikacji). Macierz podłączona pod nowym identyfikatorem aktualizuje macierz zapisaną w projekcie z tymi samymi kamerami; obok niej dodawana jest macierz z innymi kamerami.
- **Własne zapisywanie aplikacji nigdy nie nadpisuje bieżących ustawień urządzenia** starszą ich kopią i zachowuje nadpisania poszczególnych elementów zapisane przez wywołującego z poziomu CLI lub SDK.
- **Format pikseli ustawiony przez użytkownika jest tym, który projekt zachowuje.** Format ustawiony w ustawieniach tablicy, za pomocą `--pixel-format` lub `connect_array(pixel_format=)`, w tym 8-bitowy, jest zapisywany i odtwarzany dokładnie tak samo. Jeśli nie ustawiono żadnego formatu, każda kamera łączy się przy użyciu domyślnego formatu 12-bitowego (`BayerRG12`, `Mono12` w przypadku czujnika monochromatycznego). Nazwa formatu oznacza jego głębię bitową; rodzina formatu odpowiada czujnikowi, więc kamera kolorowa nigdy nie jest ustawiona na format monochromatyczny, a kamera monochromatyczna na format Bayera, a wiersz `[PixelFormat <serial>] ... written` informuje o tym, gdy format został zmapowany. Pliki eksportowe z rejestracji w formacie 8-bitowego przechwytywania są 8-bitowe, łącznie z odbiciem: przeskalowane i opatrzone oznaczeniem `PixelScale`.
- Urządzenie podłączone bez projektu niczego nie rejestruje.
- **Nagranie podlega projektowi utworzonemu po nim; otwarcie pozostawia je w ramach tego projektu.**Gdy tworzony jest projekt — SDK `create_project()`, `chloros-mcp`’s `create_project` lub opcja**Nowy projekt**w aplikacji — lub jest powielany (opcja**Duplikuj**w aplikacji, która otwiera kopię), podczas gdy nagranie nadal zapisuje dane do projektu, który był otwarty wcześniej, nagranie jest kontynuowane, a zapisane dane trafiają do nowego projektu, w tym samym miejscu w jego strukturze (poniżej). Otwarcie istniejącego projektu — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, opcja**Otwórz projekt** w aplikacji — powoduje, że nagrywanie nadal trwa w ramach projektu, do którego jest zapisywane: kończy się tam, a pliki dołączają do listy plików tego projektu (aplikacja zatrzymuje każde nagrywanie przed otwarciem projektu).
  - `array-record` i `array-burst`: plik wideo lub folder serii zawierający wszystkie klatki jest przenoszony w całości do tego miejsca po zatrzymaniu nagrywania (jego `--duration`, Ctrl+C). Plik `Burst dir`, który jest wyświetlany, jest nowym plikiem, a pliki `--build` i `array-build-video` działają na jego podstawie. Jeśli dysk odmawia przeniesienia, nagranie pozostaje w miejscu, w którym zostało zapisane;dziennik podaje powód, podobnie jak odpowiedź HTTP po zatrzymaniu (`relocation_error`). Surowe klatki zatrzymanej serii oraz `.daq` dołączają do listy plików projektu, w którym się znalazły; film (`array-record`, film utworzony z serii) nie jest dodawany do listy plików projektu.
  - `array-capture --continuous`: każde przechwycenie wykonane po zmianie jest zapisywane w nowym projekcie; te zapisane wcześniej pozostają na swoich miejscach.
  - `daq pool-record` i `project sensor log` (oraz nagranie z wbudowanego czujnika światła aplikacji): plik `.daq` zostaje przeniesiony do nowego projektufolder `light_sensor` po zakończeniu nagrywania i dołącza do listy plików tego projektu.

  Nagranie w folderze poza otwartym projektem (np. `-o` w innym miejscu) pozostaje tam, gdzie jest, a pliki `lattice continuous` i `multi-stream` zachowują przypisany im folder.
- **W aplikacji nagrania wykonane po utworzeniu nowego projektu trafiają do niego.** Utworzenie lub skopiowanie projektu nie przerywa żadnego z trwających procesów przechwytywania ani nagrywania, a każde kolejne przechwycenie (opcja „Przechwyć wszystko” lub „Rozpocznij przechwytywanie”) jest zapisywane w nowym projekcie i dołączane do jego listy plików. Otwarcie istniejącego projektu w aplikacji powoduje najpierw zatrzymanie wszystkich tych procesów.
- **Projekt nie jest przenoszony podczas nagrywania.**Przeniesienie otwartego projektu do innego katalogu roboczego (w aplikacji**Ustawienia → Katalog roboczy**; żadne polecenie CLI nie przenosi projektu, natomiast klient HTTP wysyłający żądanie do `/api/move-project` to robi) kończy się odmową z kodem HTTP 409 `PROJECT_RECORDING`, gdy do projektu zapisuje się: nagranie macierzy lub seria zdjęć, ciągłe przechwytywanie macierzy, zadanie typu `lattice continuous` / `multi-stream` lub nagrywanie z czujnika światła zapisuje do niego dane: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplikacja odrzuca to w ten sam sposób, zanim poprosi o wskazanie folderu, podczas gdy działa jej własna funkcja „Capture All”, „Start Capturing” lub nagrywanie z czujnika światła.

Jest to niezależne od tego, w jakim stanie pozostają kamery po rozłączeniu (zobacz [Pamięć kamery](#camera-memory----keep-settings----clear-settings)): projekt sam ustawia swoje kamery po otwarciu.

### Pamięć kamery — `--keep-settings` / `--clear-settings`

Gdy podłączony jest zestaw, kamery przejmują jego ustawienia, które można następnie zmienić. Gdy program Chloros zwalnia kamery — `array-disconnect`, koniec polecenia przejściowego (`multi-capture` bez macierzy, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), zatrzymanie `lattice continuous`, zakończenie pracy `lattice capture` kamery, której nie obsługuje — przywraca każdą z nich do stanu, w jakim znajdowała się przed połączeniem: okablowanie wyzwalacza i linii synchronizacji, ustawienia PTP oraz kamery (pełna lista znajduje się w sekcji [Wiele kamer (sesje przejściowe)](#multi-camera-transient-sessions)).

Aby korzystać z kamer poza projektem, dwie flagi w dowolnym z tych poleceń zapisują ustawienia w **własnej pamięci** kamery — czyli te, z którymi kamera uruchamia się w dowolnym oprogramowaniu:

- **`--keep-settings`** zapisuje ustawienia każdej kamery w jej pamięci w stanie, w jakim pozostawiła je sesja (UserSet2, ustanowiony jako zestaw startowy). Kamery nadal działają w trybie swobodnym (`TriggerMode` `Off`), a okablowanie linii synchronizacji pozostaje bez zmian. Kamera bez UserSet2 zapisuje ustawienia do UserSet1; ten komputer zapamiętuje to, więc mechanizm samonaprawy macierzy Chlorosa — co powoduje zresetowanie UserSet1, który odbiega od wartości bazowej Chlorosa — pozostawia go bez zmian.
- **`--clear-settings`** resetuje pamięć każdej kamery do ustawień fabrycznych: wartości bazowej Chlorosa, tej, którą system samonaprawy macierzy utrwala — domyślne ustawienia fabryczne z `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatyczną ekspozycją oraz zachowanym skalibrowanym balansem bieli kamery RGB — w UserSet1, utworzonym jako zestaw startowy.
- Nie można łączyć tych dwóch opcji. Każda kamera otrzymuje wiersz: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` lub `SN:…: camera memory reset to the factory settings.`; odpowiedź `camera_memory` jest taka sama dla każdej kamery.

Aplikacja nie udostępnia tych flag.

**Każde wyjście przywraca kamery do stanu początkowego.**To samo przywracanie do stanu początkowego odbywa się przy każdej ścieżce wyjścia, nie tylko w przypadku powyższych poleceń: `cam-disconnect` (oraz kamery**X**w aplikacji);**X** aplikacji na tablicy lub w wierszu dowolnego elementu tablicy (tablica nie może działać bez elementu, więc cała tablica jest zwalniana); zamknięcie aplikacji (daje to backendowi do 20 s na przywrócenie każdej kamery, tablicy i uruchomionego `lattice continuous` z powrotem, zanim ją zatrzyma; polecenie nadal działające w tym momencie — przejściowe polecenie otwierające swoje kamery lub oczekujące na ich zegary PTP, jednorazowe `lattice capture`, `power`, `white-balance` lub `presets --apply` — zatrzymuje się w następnym bezpiecznym punkcie, najpierw przywraca swoje kamery, przez maksymalnie 8 s, i odpowiada HTTP 503 `SHUTTING_DOWN`; połączenie typu „array”, które nadal działa — `lattice array-connect`, czyli połączenie aplikacji — zatrzymuje się przed kolejną kamerą i przywraca kamery, które otworzyło, na maksymalnie 10 s, i zwraca kod HTTP 503 `SHUTTING_DOWN`; a ta, która pojawia się podczas wyłączania, nie otwiera niczego: HTTP 503 `SHUTTING_DOWN`); oraz polecenie CLI, które zostaje przerwane lub którego terminal się zamyka (sygnały SIGTERM / SIGHUP zatrzymują jego działanie — i przywraca kamery do stanu poprzedniego — przed zakończeniem działania `143` / `129`, podobnie jak Ctrl+C). `lattice continuous`, `multi-stream` lub `align-stream` zabity od razu (SIGKILL), który nie może niczego zatrzymać, ma swoje zadanie zatrzymane, a kamery przywrócone przez backend 30 s po ostatnim zgłoszeniu. Backend, który ulegnie awarii lub zostanie przymusowo zamknięty, nie może przywrócić żadnych ustawień; wyłączenie i ponowne włączenie zasilania przywraca każdą kamerę do jej pamięci, a kolejne połączenie z Chlorosem ponownie konfiguruje kamerę, którą uzna za gotową do pracy.**Projekt sam ustawia swoje kamery.** Gdy macierz łączy się z otwartym projektem, każdy element otrzymuje ustawienia zapisane dla niego przez projekt — ekspozycję, wzmocnienie lub docelową wartość AE utrzymywaną dla niego (traktowaną jako własną, więc automatyczna ekspozycja nie ingeruje w te wartości) oraz tryb balansu bieli (korekcja Chloros; własny balans bieli kamery pozostaje wyłączony) — niezależnie od tego, co przechowuje pamięć kamery.

### Smart-AE / Smart-Capture

Macierze LATTICE uruchamiają ciągłą ekspozycję automatyczną (AE) w tle zaraz po połączeniu, ale nowo wycelowana scena potrzebuje chwili, by się ustabilizować. `array-capture --smart` to **gotowe rozwiązanie**: czeka na ustabilizowanie się AE we wszystkich kamerach w macierzy, a następnie uruchamia przechwytywanie. Używaj go, gdy zmieniasz scenę w trakcie sesji.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Domyślnie zasady ustabilizowania są konserwatywne: limit czasu 5 s, okno stabilności 1,5 s, tolerancja rozrzutu ekspozycji ±5 %. Jeśli potrzebujesz innego zachowania automatyki, dostosuj ustawienia za pomocą SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

#### Jak działa automatyczna ekspozycja

- **Najpierw ekspozycja, potem wzmocnienie.** Gdy wzmocnienie jest ustawione na tryb automatyczny, pozostaje na poziomie 0 dB, dopóki ekspozycja nie osiągnie najdłuższej dozwolonej wartości.
- **Ręczna ekspozycja pozostawia wzmocnienie w trybie automatycznym**, dopóki nie zmienisz wzmocnienia lub nie ustawisz go na tryb ręczny. Po powrocie do trybu automatycznego limit wzmocnienia jest taki sam jak poprzednio.
- **Niewielkie jasne obszary nie przyciemniają sceny.** Odblaski zajmujące do 1% kadru są ignorowane, a w przypadku paneli znanego wzorca kalibracyjnego – do 0,5% powierzchni panelu.
- **Górny limit ekspozycji DAQ ma zastosowanie tylko wtedy, gdy wzorzec kalibracyjny znajduje się w polu widzenia.** Zapobiega to przesterowaniu białego panelu; gdy panel nie jest widoczny, kamera ustawia ekspozycję odpowiednio do sceny.
- **Stałe oświetlenie pozostaje niezmienne, a po zmianie oświetlenia obraz stabilizuje się w ciągu kilku sekund.** Przy stałym oświetleniu ekspozycja i jej limit pozostają niezmienne, a po znacznej zmianie oświetlenia obraz stabilizuje się w ciągu kilku sekund. Przesycenie nie ogranicza już ekspozycji na stałe, a zmiana obszaru zainteresowania (ROI) powoduje ponowne dostosowanie ekspozycji.
- **Docelowa jasność pozostaje w zasięgu.** Nie wzrasta, gdy kamera osiąga limit ekspozycji, a jej wzmocnienie nie może wzrosnąć, więc powracające światło nie powoduje prześwietlenia kadru.
- **Kamera, której automatyczna ekspozycja przestaje reagować, jest utrzymywana na poziomie odpowiednim dla aktualnego oświetlenia.** W jasnym słońcu kamera może przejść na najkrótszy czas naświetlania i pozostać w tym trybie, wyświetlając ciemny obraz, a wszystkie jej ustawienia wskazują wartości w normie. Chloros monitoruje kadr, utrzymuje kamerę na poziomie dostosowanym do danego oświetlenia i zwalnia ją, gdy światło się zmienia. Nie zakłóca tego nawet zaciemniony lub zasłonięty czujnik światła.

### Poziomy eksportu ujęć (ustawienie domyślne `all`)

Począwszy od tej wersji, `lattice capture`, `lattice multi-capture` oraz `lattice array-capture` **domyślnie ustawione na `--processing all`** — jeden zapisany plik na typ eksportu, który ma zastosowanie do każdej kamery, co odpowiada zachowaniu opcji „Capture All” w interfejsie graficznym. Poziomy są następujące:

| Poziom | Wyjście | Dotyczy |
| --- | --- | --- |
| `raw` | Jednokanałowy Bayer (kamery monochromatyczne: pojedyncze pasmo) bezpośrednio z czujnika. | Wszystkie kamery. |
| `debayered` | 3-kanałowy kolor, zapisany w kolejności kanałów **RGB**(plik TIFF z fotometryczną interpretacją RGB; pliki PNG/JPEG również otwierają się jako RGB): od surowych danych odjęto przesunięcie ciemności (DSNU) z pakietu kalibracyjnego, a następnie zastosowano demosaikowanie z uwzględnieniem krawędzi— bez tablicy LUT liniowości, korekcji płaskiego pola ani demiksowania (kamery monochromatyczne: 1-kanałowa skala szarości). Klatki po demosaikowaniu w pamięci, zwracane przez Python SDK (`grab()`, `frame_stream()`), są w kolejności**BGR**, zgodnej z OpenCV; tylko zapisany plik jest w formacie RGB. | Wszystkie kamery. |
| `radiance` | float32 W/m²/sr/nm poprzez pełny łańcuch radiometryczny. | Tylko kamery wielospektralne (M3C/M3M) — **pomijane w przypadku kamery z filtrem RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), gotowe do użycia w Pix4D. | Tylko multispektralne i **tylko wtedy, gdy podłączony jest moduł DAQ + kamera jest skalibrowana**; w przeciwnym razie pomijane. |
| `display` | Pełny łańcuch podglądu GUI (CCM + WB + gamma zgodnie z profilem kamery). Zapisywany w folderze `preview/`. `lattice capture` akceptuje również nazwę `preview`; `array-capture` / `multi-capture` akceptują wyłącznie `display`. | Wszystkie kamery. |

Podaj pojedynczy poziom, aby zapisać tylko ten jeden (`--processing debayered`). Gdy poprosisz o `all`, poziomy, które nie mają zastosowania do danej kamery, są pomijane (i zgłaszane), a nie traktowane jako błąd — kamera niepodłączona lub nieskalibrowana nadal otrzymuje `raw` / `debayered` / `preview`.

#### Odczyt z modułu DAQ podczas zapisywania przechwytu

Za każdym razem, gdy moduł DAQ jest powiązany z kamerą lub matrycą albo znajduje się w puli, odczyt promieniowania padającego wykorzystany podczas przechwytu jest zapisywany do pliku **`.daq`** w podfolderze `daq/` folderu captures, wraz z wartością `.match.json` (odległość odczytu od ekspozycji) oraz `.cameras.json` (do kogo należy odczyt), a także zgłaszany w wierszu `DAQ:`. Dotyczy to każdego poziomu, w tym obrazów surowych i poddanych debayeringowi, więc każdy obraz można później ponownie przetworzyć na promieniowanie, odbicie lub indeks. Jest to ten sam plik, o tej samej nazwie i z tymi samymi plikami pomocniczymi, niezależnie od tego, czy obraz pochodzi z GUI, CLI czy SDK. W przypadku braku jakiegokolwiek odczytu z DAQ nie ma pliku `.daq`, a plik `multi-capture` bez podłączonej tablicy (rezerwa na wypadek przejściowych zakłóceń) nie zapisuje żadnych danych.

| Flaga | Efekt |
| --- | --- |
| `--no-daq` | Nie zapisuj odczytu w przypadku przechwycenia, którego poziomy go nie wykorzystują (`raw`, `debayered`). Poziom `radiance`, `reflectance` lub `display` nadal zapisuje ten odczyt, ponieważ ponowny import wymaga odczytu, na podstawie którego produkt został wytworzony. |
| `--daq-tolerance-ms MS` | Jak daleko od ekspozycji może się znajdować odczyt, aby nadal mieścił się w tolerancji (domyślnie 500). Zmienia się jedynie werdykt (w przypadku `.match.json`, `in_tolerance` i `tolerance_ms` oraz współczynnika odbicia `DlsInTolerance`), nigdy nie zmienia się sama wartość odczytu. Zobacz [Odczyt DAQ znacznie odbiegający od ekspozycji](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Starszy sposób żądania `.daq` przy przechwytywaniu wyłącznie w formacie surowym. Obecnie jest to ustawienie domyślne, a `lattice capture` i `array-capture` nadal akceptują tę flagę (`multi-capture` nie). `--no-daq` ma nad nią pierwszeństwo. |

Który moduł DAQ: ten powiązany z kamerą lub matrycą (wybór czujnika światła w GUI, zapisane powiązanie projektu lub `chloros-cli daq pool-bind`), w przeciwnym razie najnowszy dostępny w backendzie. Seria zdjęć w formacie RAW (`array-burst`) zachowuje swój własny plik `.daq` dla każdego oddzielnego odczytu w folderze serii, tak jak poprzednio.

**Pliki PNG i JPEG przechowują 8 bitów na kanał, więc przyjmują wyłącznie pliki `display` / `preview` oraz produkty indeksowe.** Pliki `--format png` lub `jpg` o poziomie, którego nie mogą pomieścić, są odrzucane przed rozpoczęciem przechwytywania, z podaniem nazwy poziomu oraz formatu TIFF, który go obsługuje:

- `radiance` (float32) oraz `reflectance` (uint16, `32768` = 1,0) — zawsze, więc `--processing all` (wartość domyślna) wraz z `--format png`/`jpg` również są odrzucane: należy użyć `--format tiff` lub `--levels display`.
- `raw` i `debayered` z kamery przesyłającej strumień o rozdzielczości większej niż 8 bitów (klatki 12-bitowe M3C / Mono12 M3M) — CLI nie rozpoznaje głębi bitowej kamery, więc backend odrzuca te klatki (kod stanu HTTP 400). Pojedyncza-klatkowy `lattice capture --processing debayered` pobiera klatkę 8-bitową dla formatów PNG/JPEG i nie jest odrzucany, podobnie jak `multi-capture` bez trwałej tablicy (8-bitowy po usunięciu bayera).

Każda trasa przechwytywania odpowiada w ten sam sposób (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` oraz trasy HTTP znajdujące się za nimi.

### Jak wygląda folder z plikami przechwyconymi

Każdy typ eksportu trafia do **własnego podfolderu**w katalogu `-o`, dzięki czemu w przypadku przechwytywania wielopoziomowego typy nigdy się nie mieszają. Na rysunku przedstawiono**domyślne przechwycenie `all` z jednej kamery przez `lattice capture`** — kilka poziomów na kamerę, więc każdy plik ma rozszerzenie `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Jak różnią się nazwy plików:

- **Pole kamery — numer seryjny, a następnie filtr:** każdy plik przechwytywania nosi nazwę kamery, która go wykonała, w formacie `SN<serial>-<filter>`, gdzie `<filter>` to część nazwy modelu kamery odpowiadająca filtrowi: `SN261902142-F450` dla `M3M-L87-F450`, `SN213800234-FRGN` dla modelu `M3C-L87-FRGN`, `SN214701292-FRGB` dla kamery RGB. Filtr łączy numer seryjny z `-`, a nie `_`, więc nazwa zawiera te same pola oddzielone znakiem `_`, co poprzednio (prefiks, data, godzina, milisekundy, kamera, poziom). Kamera, której modelu nie można odczytać (wymazany identyfikator DeviceUserID), otrzymuje nazwę `SN<serial>`. Nagrania z wcześniejszych wersji nie mają filtra w nazwie; program Chloros odczytuje oba rodzaje.
- **Prefiks:** `lattice capture` nadaje plikom nazwy `capture_…`; `array-capture`, `multi-capture` w przypadku połączonego macierzy oraz `project capture` / `burst` / `stream`, a pliki z tablicy są nazywane `sync_<ts>_SN<serial>-<filter>…`.
- **Sufiks poziomu występuje tylko w przypadku kilku poziomów.**Kamera, która zapisuje**kilka**poziomów w jednym ujęciu (`all` lub `--levels a,b`), dodaje `_<level>`. Kamera, która zapisuje**jeden** poziom zachowuje zwykłą nazwę bez sufiksów — `--processing reflectance` zapisuje `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — dzięki czemu skrypty napisane z myślą o ujęciach jednopoziomowych nadal działają. Poziom jest nadal określany przez folder.
- **Maski ograniczeń są zgodne z nazwą produktu:** `<product name>_clampmask.tif`. Przechwytywanie odbicia jednopoziomowego zapisuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; przechwytywanie `all` zapisuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Rozszerzenie** następuje po `--format`: domyślne `tiff` daje `.tiff`, a `--format tif` daje `.tif` (maski ograniczające to zawsze `.tif`).
- **Jeden wyjątek:** `lattice capture --processing debayered` (lub `--levels debayered`) pobiera szybką pojedynczą- i zapisuje `capture_<ts>_SN<serial>-<filter>.tiff` bezpośrednio do `-o` — bez podfolderu.
- **Strumienie i serie** zawierają to samo pole: `lattice continuous` zapisuje `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` oraz surową serię `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` to sygnatura czasowa przechwycenia, a `<serial>` to numer seryjny kamery, więc jedna zsynchronizowana grupa ma wspólną
sygnaturę czasową dla wszystkich kamer. **Należy zwrócić uwagę na jedną asymetrię:** poziom `display` jest przechowywany w folderze
o nazwie `preview/`, podczas gdy pliki wielopoziomowe zachowują `_display` w nazwie — folder i rozszerzenie różnią się
tylko dla tego poziomu. Nieznane poziomy są umieszczane w folderze o własnej nazwie, a jeśli podfolder
nie może zostać utworzony, plik jest zapisywany w katalogu głównym wyjściowym zamiast zostać utracony.

**Ponowne przetwarzanie folderu „captures”:**skieruj `chloros-cli process` na**katalog główny „captures”**
(`output/`). `process` zazwyczaj importuje tylko folder, który podasz, ale gdy ten folder nie zawiera
obrazów, a zawiera podfoldery, automatycznie przechodzi do nich — dzięki czemu podfoldery na wszystkich poziomach oraz
podfolder `daq/` są pobierane za jednym razem. Każdy poziom przechwycenia jest importowany jako pojedynczy obraz, a
pozostałe poziomy są dostępne jako tryby, a nie jako osobne obrazy dla każdego poziomu.

Bezpośrednie nazwanie **podfolderu poziomu** (np. `output/raw/`) również działa. W takim przypadku `daq/`
zostaje pominięty, więc skopiuj lub wskaż odczyt DAQ obok podczaswyprowadzasz produkt radiometryczny
z pliku `raw/` — w przeciwnym razie nie będzie z czym porównać znacznika czasu.

**Przetwarzanie zawsze rozpoczyna się od pliku `raw`.** W ramach każdego zapisu źródłem potoku danych jest surowa klatka;
`debayered`, `radiance`, `reflectance` i `preview` występują jako tryby wyświetlania, ale nigdy nie są ponownie wprowadzane
do potoku przetwarzania. Ponowne przetwarzanie produktu pochodnego spowodowałoby ponowne zastosowanie winietowania, CCM oraz
obliczeń radiancji, które są już wbudowane w jego piksele, dlatego Chloros, zamiast
poddawać je podwójnemu przetwarzaniu, rezygnuje z tego. Dwie konsekwencje, o których warto wiedzieć:

- Rendery `index/` i `composite/` **nigdy** przetwarzane. Są to dane wyjściowe, a nie zrzuty —
  renderowanie LUT NDVI nie ma żadnej znaczącej interpretacji promieniowania.
- Folder zrzutów wyeksportowany **bez** pliku `raw` (np. `array-capture --processing reflectance`) nie ma
  żadnego prawidłowego źródła w potoku przetwarzania. Te przechwyty importują się i wyświetlają normalnie, ale `process` pomija
  je i informuje o tym:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Jeśli naprawdę musisz przepuścić produkt pochodny — sesję hubu przechwyconą z
  włączonym `demosaic` lub ze starszego folderu — `--input-level {raw,debayered,processed}` wymusza punkt
  wejścia i pomija pominięcie. Ta flaga stanowi celowe wyjście awaryjne; `auto` (ustawienie domyślne)
  nigdy nie przetwarza przechwycenia, które nie zawiera danych surowych.

**Tagi kalibracyjne: tylko w przypadku `raw`.** Plik `raw` z zapisu zawiera pakiet kalibracyjny kamery: ładunek `CalibrationJSON` oraz tagi zgodne ze standardem MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), które opisują wartości surowe czujnika. Pliki z pozostałych poziomów (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) oraz każda ramka `lattice continuous` / `multi-stream` zawiera identyfikator kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) oraz własny rekord klatki, `XMP-mapir:FrameJSON` (ekspozycja, wzmocnienie, temperatura, format pikseli, czas przechwycenia, dopasowanie DAQ), ponieważ każdy produkt `process` posiada swój identyfikator. `process` wyszukuje kalibrację takiego pliku na podstawie jego numeru seryjnego, a `CalBundleSha256` w pamięci podręcznej kalibracji Chloros, pobierając pakiet jednokrotnie, jeśli komputer go nie posiada.

#### Pliki utworzone z danych z kilku kamer

Plik utworzony z pasm z więcej niż jednej kamery — stos wielopasmowy (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks na żywo utworzony z kilku kamer (`index.tif`, `index_LUT.png`), obraz z przechwytywania macierzowego `composite/` — wymienia każdą kamerę, która się do niego przyczyniła:

- **Listy poszczególnych kamer, w kolejności pasm** (kompozycja: warstwa tła, następnie pierwszy plan; warstwa indeksowa, kamery, z których pochodzą pasma): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (pakiet kalibracyjny, przez który przeszły pasma każdej kamery, w przeciwnym razie — zestaw `align-apply` / `align-stream`, którego pasma nie przeszły przez żadną kalibrację — pakiet kamery Chloros, który został zapisany w pamięci podręcznej, ten, którego nazwa pojawia się w nazwach plików przechwyconych obrazów), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listy równoległe, przy czym pozycja *n* każdej z nich opisuje tę samą kamerę; `-`, gdzie kamera nie ma żadnej wartości (brak pakietu kalibracyjnego, nieznany obiektyw). Każdy aparat to **jedna** pozycja: jeśli jego pakiet zawiera kilka wartości dla danego pola — na przykład certyfikaty lamp halogenowych — są one połączone za pomocą `;` (`F-1479-1;F-1480-1;F-1481-1`), nigdy przecinkiem, więc lista zawiera jedną pozycję na kamerę, niezależnie od sposobu jej wydruku.
- **EXIF `SerialNumber`** zawiera te same numery seryjne, połączone przecinkami (`214000535,214000551`).
- Nie zawiera **żadnego** identyfikatora pojedynczej kamery (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): te identyfikują jeden aparat i jego zestaw, a `process` pozwala na ustalenie kalibracji pliku na ich podstawie. Indeks obliczony na podstawie pasm z kilku aparatów (`index.tif`, `index_LUT.png`) zawiera w metadanych EXIF `Model` `LATTICE` (zestawu), a nie model `LATT-…` jednej kamery.

Plik zawierający pasma z **jednej** kamery — plik TIFF (`--per-band`), klatka `align-apply --per-camera`, stos lub indeks na żywo z pojedynczej kamery — zawiera własne tagi identyfikacyjne tej kamery, tak jak każdy inny plik przechwycony. Stos `align-apply`, pliki dla poszczególnych pasm i dla poszczególnych kamer, a także każdy stos `align-stream` i jego pliki dla poszczególnych pasm zawierają czas przechwycenia swojej grupy (czas naświetlania kamery referencyjnej, UTC), a klatka `--per-camera` z kamery monochromatycznej stanowi jedną płaszczyznę, podobnie jak stos `--bit-depth`.**Orientacja kamery (odbicie lustrzane / obrót).** Ustawienie orientacji kamery (opcje „Mirror H” /
Mirror V / obrót) w interfejsie graficznym określa, w jaki sposób zapisywane są dane z ujęcia:

- **Pliki `raw` i `debayered` są zapisywane dokładnie tak, jak odczytał je czujnik**, niezależnie od ustawień.
  Obrócona lub odwrócona mozaika Bayera stanowiłaby inny wzór filtrów kolorowych, a mapy kalibracyjne
  (offset ciemności, pole płaskie) są osadzone na własnej siatce czujnika.
- **`preview`/`display`, `radiance`, `reflectance` oraz render `index/` są zorientowane** — obrócone
  jednokrotnie, po zakończeniu wszystkich obliczeń kalibracyjnych, dzięki czemu otwierają się tak, jak kadrowano zdjęcie. Ich
  `clamp_masks/…_clampmask.tif` znajdują się na tej samej siatce.
- **Każdy plik przechwycony rejestruje ustawienie** w formacie XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, zgodnie z ruchem wskazówek zegara, po
  odwróceniach) oraz `ChlorosOrientationApplied` — `false` dla danych surowych i po usunięciu efektu Bayera (piksele natywne),
  `true` na zorientowanych produktach. `ChlorosProcessing` zawiera listę operacji zastosowanych do pikseli tego pliku
  (`none` w przypadku pikseli natywnych). Obejmuje to każdą klatkę strumieniowaną przez `lattice continuous` (
  własne ustawienie kamery, niezastosowane) oraz każdą klatkę `lattice multi-stream` (tożsamość: jej
  klatki składowe są natywne, podobnie jak w przypadku przechwytywania tablicowego). `lattice align-stream` / `align-apply`
  stosy rejestrują ustawienia kamery głównej, niezastosowane: ich pasma są zniekształcane do klatki kamery głównej,
  ale nie są obracane.
- **`process` stosuje zarejestrowaną orientację** podczas przetwarzania danych surowych: najpierw przesunięcie ciemności, wyrównanie
  pola, demosaikowanie i rozdzielanie w współrzędnych czujnika, następnie zniekształcenie związane z wyrównaniem matrycy, a
  orientację na końcu — dzięki czemu produkty podglądu, radiancji, odbicia i indeksu wychodzą dokładnie tak, jak
  w przypadku przechwytywania na żywo, i zapisuje `ChlorosOrientationApplied=true`. Nieskalibrowane produkty rezerwowe
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) są zorientowane w ten sam sposób po
  pola płaskiego. Produkt po usunięciu efektu Bayera pozostaje w formacie natywnym.
- **Przeglądarka obrazów Chloros pokazuje plik surowy lub po usunięciu efektu Bayera obrócony zgodnie z jego zapisem** (tylko na ekranie —
  plik nie ulega zmianie), więc wygląda on tak samo jak jej produkty; miniatura, widok w pełnym rozmiarze,
  odczyt kursora oraz piaskownica indeksu/LUT wykorzystują ten sam obrót. Plik bez rekordu oraz
  produkt (już zorientowany) są wyświetlane tak, jak zostały zapisane. Eksport do środowiska testowego rejestruje orientację, jaką
  posiadają jego piksele (`ChlorosOrientationApplied=true`; identyfikator warstwy bez zapisu).
- **Ujęcia bez zapisu są przetwarzane jako natywne** (bez zmian w działaniu). Przed wersją 1.3.0
  ujęcie z jednej kamery z ustawionym odwróceniem lub obrotem samo obracało swoje pliki surowe i po usunięciu efektu bayera;
  program Chloros nie potrafi rozpoznać, że taki plik jest już-obrócony plik surowy od natywnego i nie podejmuje domysłów, więc
  przetwarza go w postaci, w jakiej został zapisany. Mapy kalibracyjne znajdują się na własnej siatce czujnika, więc w przypadku takiego
  nagrania należy użyć nagrania wykonanego w wersji 1.3.0, gdy kalibracja ma znaczenie.
- **Tablice domyślnie nie obracają żadnego pliku składowego** (zniekształcenie wynikające z wyrównania oraz plik XMP `Alignment*` znajdują się w
  własnych, nieobróconych klatkach kamer) i rejestrują tożsamość. `array-capture --orient` obraca produkty każdego
  elementu po transformacji wyrównującej: zgodnie z orientacją **ramy głównej** w momencie, gdy zostały
  przekształcone do jej układu (dzięki czemu elementy pozostają współzarejestrowane), w przeciwnym razie zgodnie z orientacją ich własnych kamer. Każdy
  plik rejestruje orientację układu, w którym znajdują się jego własne piksele — plik surowy, z własnej kamery —
  a plik surowy zawierający transformację wyrównania rejestruje również `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: orientację obrazu wzorcowego, dla produktu dopasowanego
  do ramki kamery głównej. `process` obraca każdy produkt o wartość należącą do jego pikseli:
  o wartość wyrównaną, gdy zniekształca plik surowy (`--array-alignment`, domyślnie — produkty z
  produkty z przechwytywania na żywo), a o własnym, gdy nie jest (`--no-array-alignment` — z
  niewyrównanego przechwytywania na żywo). Ta sama opcja istnieje dla `array-burst`, `project capture` / `burst` /
  `stream`, receptury oraz SDK.

### Pominięte ujęcia w układach z mieszanymi filtrami

Gdy w jednej macierzy mieszają się kamery RGB i wielospektralne, `array-capture --processing radiance` (lub `reflectance`) zapisuje klatki wielospektralne i **pomija** kamery RGB — promieniowanie na piksel w układzie Bayera nie ma znaczenia w przypadku czujnika szerokopasmowego. CLI wyświetla wyraźnie każdy zapisany plik (wraz z poziomem eksportu), każdy zapisany plik `.daq` oraz każde pominięcie, więc liczba plików nie jest zaskakująca:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeny powodów pominięcia mają wzór `<level>-not-applicable-to-rgb-cam`. Kamera, której pakiet kalibracyjny nie został załadowany, pomija `radiance` / `reflectance` wraz z `<level>-needs-calibration (connect the camera so its cal pack loads)`, a w przypadku braku użytecznego odczytu współczynnika odbicia z modułu DAQ pomija z kodem `reflectance-skipped-no-dls`. Współczynnik odbicia może również zostać pominięty z kodami `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, oraz w przypadku `dls-uncalibrated-band-<nm>`, gdy pasmo leży w większości poza zakresem radiometrycznie skalibrowanym czujnika światła DAQ (~374–974 nm) — spośród dostępnych w sprzedaży numerów katalogowych tylko F988, którego obsługiwana ścieżka to proces z wykorzystaniem panelu odbicia.

Użyj `--processing debayered` (lub `display`), aby uwzględnić każdą kamerę niezależnie od typu filtra, lub domyślnego `all`, aby uzyskać wszystkie odpowiednie poziomy dla każdej kamery za jednym razem.

### Odczyt DAQ daleki od ekspozycji

Każdy zapisany przez `.daq` odczyt DAQ jest dopasowany do tej ekspozycji, a `.match.json` obok niego rejestruje, jak daleko od siebie znajdowały się te dwa odczyty (`gap_ms`) oraz czy mieści się to w granicach tolerancji (`in_tolerance`, 500 ms, o ile nie ustawiono inaczej w konfiguracji przechwytywania `--daq-tolerance-ms`). Odczyt wykraczający poza ten zakres jest nadal wykorzystywany, więc żadna klatka nie jest tracona, a odczyt ten jest oznaczony: plik XMP pliku odbicia zawiera wartości `DlsInTolerance=False` oraz `DlsMatchGapS`. `lattice capture`, `multi-capture` i `array-capture` również wyświetlają tę wartość pod odpowiednim `.daq`, co wiąże się zstrumienia DAQ, gdy wyjaśnia się to w następujący sposób:

**W jaki sposób odczyt jest dopasowywany do ekspozycji.** Operacja przechwytywania czeka na pierwszy odczyt z modułu DAQ po zakończeniu ekspozycji, aby móc połączyć odczyty po obu jej stronach (`method` `interpolated`). Urządzenie DAQ-E wysyła odczyt co 0,4 s w świetle dziennym i co 1–2 s w pomieszczeniach, więc czas oczekiwania wydłuża polecenie o około półtora takiego interwału, a nie samą ekspozycję. Nie czeka się na urządzenie DAQ, które przestało wysyłać odczyty. OdczytOdczyt DAQ-E stanowi średnią natężenia światła w przedziale czasu od poprzedniego odczytu, dlatego jest umieszczany w środku tego przedziału: `gap_ms` jest mierzony od momentu ekspozycji do tego środka, a `bracket_stamps_ns` podaje własne znaczniki czasu odczytów. `array-capture --continuous` czeka bez spowalniania przechwytywania: pliki surowe są zapisywane natychmiast, a ich `.daq` po nadejściu odczytu. W przypadku `--interval` każda iteracja oczekuje tak, jakby była pojedynczym przechwyceniem.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Jeśli na podstawie tego odczytu nie zapisano żadnego pliku odbicia, pierwszy wiersz kończy się zamiast tego wartością `saved anyway; its .match.json says in_tolerance: false`.

Przechwytywanie nadal kończy się kodem `0`: wszystkie pliki zostały zapisane i zawierają informacje o swojej zawartości. Urządzenie DAQ, którego strumień danych utknął, nadal dostarcza swój ostatni odczyt, więc należy to sprawdzić za pomocą kodu `chloros-cli daq pool-list` przed poleganiem na współczynniku odbicia z tego przechwytywania. Backend ponownie łączy się z urządzeniem DAQ-E z puli, które przestało dostarczać dane (patrz [Urządzenie DAQ-E, które przestaje dostarczać dane](#a-daq-e-that-stops-delivering)). Odczyt wykraczający poza tolerancję wyświetla tylko swój wiersz `DAQ:`, chyba że jest to odczyt zachowany (poniżej).

**Odczyt zachowany po utracie połączenia z modułem DAQ.** Gdy żaden moduł DAQ nie może dostarczyć odczytu dla danej ekspozycji (przypisany moduł DAQ jest niedostępny lub żaden moduł DAQ z puli nie posiada odczytu, np. po `daq pool-disconnect`), proces przechwytywania ponownie wykorzystuje ostatni odczyt, jaki posiadał Chloros, a jego `.match.json` opisuje ten odczyt w odniesieniu do tej ekspozycji: `method`, `retained`, `gap_ms` z tej ekspozycji do próbek, z których pochodzi odczyt, `in_tolerance` z tej luki oraz `retained_from_epoch_ns` – ekspozycja, z którą odczyt został po raz pierwszy dopasowany. Zrzut wykonany długo po wyłączeniu systemu DAQ wyświetla zatem powyższe wiersze, a pod nich jeszcze jedną, która jest drukowana nawet wtedy, gdy odczyt nadal mieści się w granicach tolerancji:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Jego `.daq` nadal wskazuje urządzenie DAQ, które wykonało odczyt, wraz z kalibracją i nasadką, przy których ten odczyt został wykonany, dzięki czemu podczas ponownego importu wiadomo, które urządzenie DAQ, kalibracja i nasadka go wygenerowały.

### Pliki, których nie udało się zapisać

Kamera, której pliku nie można zapisać — z powodu zapełnionego dysku lub błędu uprawnień — nie uniemożliwia zapisywania plików z pozostałych kamer. Każde polecenie przechwytywania (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; oraz `capture`, `burst` i `stream` z przepisu) wyświetla każdy plik, którego nie udało się zapisać, na stderr, a następnie **zamyka program `1`**, mimo że pozostałe pliki zostały zapisane:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Uwzględniono wszystkie pliki należące do danego przechwycenia, w tym pojedynczyz odbayeryzowaną pojedynczą klatką `lattice capture --levels debayered`. Odczyt `.daq`, którego plik pomocniczy pochodzenia `.match.json` nie mógł zostać zapisany, jest wymieniony jako `[match]` (sam plik `.daq` został zapisany).

W przypadku zapełnionego dysku powtarzające się nagrania (`array-capture --continuous` / `--interval`, `project burst` / `stream`, przepis `burst` / `stream`) **zatrzymuje się na tym przebiegu** zamiast nie zapisywać niczego na tym samym dysku przez resztę przebiegu. Każda inna awaria jest zgłaszana przy każdym przebiegu, w którym wystąpiła, a przebieg trwa do końca, po czym kończy się kodem `1`. `array-capture --continuous` (ciągłe przechwytywanie przez backend) nie wyświetla listy plików, w tym tych, których przetwarzanie zakończyło się niepowodzeniem: zlicza je w wierszach postępu i podsumowania (`N failed`, `N file(s) failed to write.`), a następnie kończy działanie `1` — zobacz [Tryb ciągły](#continuous-mode---continuous).

Rejestratory — `array-burst` i `array-record` — zgłaszają własne awarie. `array-burst` zatrzymuje się samoczynnie, gdy dysk się zapełni lub tuż przed tym momentem. Oba urządzenia kontynuują pracę pomijając wszelkie inne nieudane operacje zapisu. Gdy którekolwiek z nich zgłasza błąd, jest on wyświetlany po licznikach, a polecenie **kończy działanie `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` zamiast `Disk full:` oznacza, że seria została zatrzymana przed zapełnieniem dysku. Ramki nagrane przed zatrzymaniem są zachowane. Polecenie `array-burst --build` pomija kompilację w przypadku wyczerpania miejsca, ponieważ filmy trafiłyby na ten sam dysk. Najpierw zwolnij miejsce, a następnie uruchom `array-build-video --burst-dir …`. Po wystąpieniu jakiegokolwiek innego błędu kompilacja nadal przebiega.

Błąd `array-record`, w wyniku którego nie zapisano żadnej klatki, również oznacza nieudane nagranie (`Recorder error: no-frames: the combined view published no frame to record …`, zakończ `1`) — zobacz [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Tryby przechwytywania, rejestratory i przetwarzanie offline

Wszystkie te funkcje działają na **trwałej tablicy** (należy najpierw uruchomić `array-connect`). Odzwierciedlają one panel przechwytywania w interfejsie graficznym.

### Tryby `array-capture`

`array-capture` to pojedyncze polecenie z czterema trybami migawki oraz zestawem przełączników eksportu:

| Tryb | Flaga | Zachowanie |
| --- | --- | --- |
| **Pojedynczy** *(domyślnie)* | (brak) | Jedna zsynchronizowana grupa przechwytywania, a następnie zakończenie. |
| **Ciągły** | `--continuous` | Przechwytywanie jeden po drugim, aż do przechwycenia `Ctrl+C`, `--count N`, `--duration S` lub zapełnienia dysku. Uruchamia własną funkcję ciągłego przechwytywania backendu (tryb ciągły interfejsu graficznego) — zobacz [Tryb ciągły](#continuous-mode---continuous) poniżej. |
| **Interwał** | `--interval S` | Jedno przejście co `S` sekund (mierzone od początku każdego przejścia), te same granice. |
| **Najszybszy** | `--fastest` | Tylko surowedane + przypisany odczyt DAQ + złożony wskaźnik łączony; pomija obliczenia dotyczące promieniowania/odbicia/wyświetlania, dzięki czemu klatka jest wyświetlana szybko. Oznacza `--processing raw --force-daq`. Zapisane dane `.daq` należy później przetworzyć na skalibrowane produkty. |

Przełączniki eksportu (można łączyć z dowolnym trybem; wszystkie korzystają z tego samego interfejsu GUI/punktu końcowego SDK):

| Flaga | Efekt |
| --- | --- |
| `--array-id ID` | Tablica, z której ma nastąpić przechwytywanie (pominąć, gdy podłączona jest tylko jedna). |
| `-o, --output DIR` | Folder wyjściowy (domyślnie `output`, w folderze, z którego uruchamiane jest polecenie). |
| `--format {tiff,tif,png,jpg,jpeg}` | Format pliku (domyślnie `tiff`). PNG / JPEG akceptują wyłącznie `display` oraz renderowanie indeksowe; zobacz [Poziomy eksportu przechwytywania](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Pojedynczy poziom eksportu (`raw`, `debayered`, `radiance`, `reflectance`, `display`) lub `all` (domyślnie). |
| `--levels L1,L2,…` | Wyraźny podzbiór typów eksportu (np. `raw,radiance,reflectance`); **zastępuje `--processing`**. |
| `--aligned` / `--no-aligned` | **Domyślnie: wyrównane, tak jak w przypadku przechwytywania tej samej tablicy przez aplikację Chloros**(jej ustawienie przechwytywania**Wyrównane** dla poszczególnych tablic, włączone, chyba że je wyłączysz). Eksport w formacie innym niż surowy dla każdego członka (`debayered`, `display`, `radiance`, `reflectance`, renderowanie indeksu) jest przekształcany do układu ramki kamery głównej i przycinany do wspólnego obszaru nakładania się elementów, dzięki czemu pliki są zarejestrowane pikselowo i mają taki sam rozmiar, kadrowanie oraz tagi `Alignment*` jak w aplikacji — mniejsze niż czujnik (lub obszaru zainteresowania) o przesunięcia członków; w przypadku obszaru zainteresowania transformacja jest ponownie oparta na oknie. `XMP-mapir:AlignmentJSON` wskazuje `"applied": true` i podaje `crop_roi`. Plik surowy pozostaje nieprzekształcony w rozmiarze natywnym, ale zawiera transformację w metadanych (`"applied": false`). `--no-aligned` zapisuje każdy plik w rozmiarze natywnym, bez przekształcenia. `--aligned` jest domyślnym ustawieniem. Niezależnie od tego, czy zniekształcenie jest stosowane, czy nie, każdy plik składowy rejestruje transformację `Alignment*` swojej kamery, gdy tablica posiada profil wyrównania — z wyjątkiem kamery, której profil nie wyrównuje (pominiętej lub przymusowo ustawionej na tożsamość z powodu nieudanego rozwiązania), oraz kamery referencyjnej, gdy żadna inna kamera nie jest wyrównana: te nie zawierają tagów `Alignment*`. W przypadku braku profilu w macierzy dane z przechwytywania są zapisywane jako niewyrównane, a interfejs CLI wyświetla przyczynę w wierszu `Alignment:`, tak jak ma to miejsce w każdym innym przypadku, w którym nie można zastosować zniekształcenia (elementy o różnych rozdzielczościach, tablica niesynchronizowana kablem, klatka, na którą nie można nałożyć profilu). Funkcja automatycznego wyrównywania tablicy tworzy profil po ustabilizowaniu się ekspozycji; `align-calibrate` tworzy go na żądanie. |
| `--orient` | Przekształć **produkty**każdego elementu — `display`, `radiance`, `reflectance`, render `index/` oraz `composite/` — zgodnie z orientacją kamery po zniekształceniu wynikającym z wyrównania: produkt zniekształcony do układu odniesienia obrazu głównego (domyślnie) zgodnie z**orientacji**klatki wzorcowej, dzięki czemu elementy pozostają współzarejestrowane; niezrównoważony (`--no-aligned` lub ujęcie, do którego nie można było zastosować zniekształcenia) — według orientacji własnej kamery.**Domyślnie: wyłączone — każdy plik elementu pozostaje w formacie natywnym**, tak jak zawsze w przypadku ujęć tablicowych. Pliki `raw` i `debayered` pozostają w formacie natywnym niezależnie od ustawienia. Każdy plik rejestruje orientację swoich elementów (zobacz [Orientacja kamery](#jak-wygląda-folder-z-nagraniami)); plik XMP `Alignment*` zachowuje transformację w ramce natywnej, zastosowaną przed orientacją, więc `process` (zniekształcenie, a następnie orientacja) odtwarza rzeczywisty obraz. |
| `--index` / `--no-index` | Zapisz / pomiń nakładkę wskaźnika roślinności dla poszczególnych kamer, jeśli została skonfigurowana. Domyślnie: renderuj ją. |
| `--no-daq` | Pomiń plik dodatkowy `.daq`, gdy żaden z wybranych poziomów nie wykorzystuje DAQ (pobieranie wyłącznie danych surowych). Domyślnie odczyt jest zapisywany, dzięki czemu klatki można ponownie przetworzyć na odbicie/indeks w trybie offline. `--force-daq` jest nadal akceptowane (ustawienie domyślne). |
| `--daq-tolerance-ms MS` | Tolerancja dopasowania DAQ dla tych przechwyceń (domyślnie 500 ms). |
| `--smart` | Oczekiwanie na ustabilizowanie się AE we wszystkich kamerach przed wyzwoleniem (patrz [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresja pikseli w formacie TIFF. `none` (domyślnie od wersji 1.3.0; w wersji 1.2.0 zapisywano `deflate`) = bez kompresji — piksele z matrycy dokładnie w postaci odczytanej, ~6,3 MB na klatkę w pełnej rozdzielczości oraz najszybsza stała prędkość zapisu; `deflate` = bezstratny zlib L1 + predyktor poziomy, ~4,1 MB na klatkę, wolniejszy. Oba formaty są bezstratne i po zaimportowaniu wyświetlają się identycznie. |

> **TIFF z pojedynczym zapisem + model z ciągłą szybkością zapisu.**Każdy plik jest zapisywany**raz**: piksele, pakiet XMP, IFD0 (marka/model/numer seryjny aparatu) oraz tagi EXIF `ExposureTime` / `SerialNumber` w jednym przebiegu, bez żadnego późniejszego nadpisywania. `--continuous`X przechwytuje kolejne zsynchronizowane grupy — każde przechwycenie obejmuje grupę następującą po poprzedniej, w tym tę opublikowaną, gdy poprzednie przechwycenie było jeszcze w trakcie zapisywania — dzięki czemu działa z prędkością tablicy. DEFLATE blokuje GIL w języku Python, więc zapisy skompresowane**nie**są równoległe w wątkach zapisujących dla poszczególnych kamer; ciągłe przechwytywanie w pełnej rozdzielczości z prędkością czujnika wymaga zapisów nieskompresowanych (ustawienie domyślne)**oraz** dysk klasy NVMe (prędkość ciągła ~600 MB/s dla macierzy 9 kamer przy ~10 klatkach na sekundę). Ten sam parametr jest dostępny jako `compression` w `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Tryb ciągły (`--continuous`)

`array-capture --continuous` (bez `--interval` lub `--smart`) przekazuje kontrolę do **własnego, ciągłego przechwytywania backendu** — sesji, w której działa tryb ciągły interfejsu graficznego. Backend samodzielnie zapisuje każdą zsynchronizowaną grupę, realizując jednocześnie dwa przechwyty i nie wysyłając żądań z interfejsu CLI między grupami, dzięki czemu nadąża za szybką matrycą, w której jedno żądanie na przechwyt pozostaje w tyle (zmierzone 73–77 przechwytów/s na matrycy z 9 kamerami przy obszarze zainteresowania (ROI) o rozdzielczości 512 pikseli). Każda z powyższych opcji eksportu ma zastosowanie do każdego przechwycenia.

- **Ograniczenia:** `--count N` zatrzymuje się po N przechwyceniach (jedno przechwycenie = jedna zsynchronizowana grupa: pliki wszystkich członków); `--duration S` po upływie S sekund. Bez żadnego z tych ustawień program działa do momentu wystąpienia `Ctrl+C`, zapełnienia dysku, wystąpienia błędu lub wyłączenia macierzy. `Ctrl+C` zatrzymuje sesję zaplecza, a następnie wyświetla podsumowanie.
- **Postęp** co 2 s, z dołączonymi `, N failed` i `, N groups skipped`, gdy wartość jest niezerowa (pominięta grupa to taka, którą macierz opublikowała, ale nie została przechwycona):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Brak wykazu poszczególnych plików.** Przebieg kończy się jednym wierszem podsumowującym — przechwyty, szybkość, pliki i folder wyjściowy — po którym następują kody `N file(s) failed to write.` / `N group(s) skipped.`, jeśli wartość jest niezerowa:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Zatrzymuje się** w przypadku zapełnienia dysku z kodem `Continuous capture stopped: the drive is full.` oraz w przypadku błędu lub utraty macierzy z kodem `Continuous capture stopped: <reason>.` (oba wyświetlane na stderr).
- **Kod zakończenia:** `0`, gdy zapisano przechwycone dane i wszystkie pliki zostały zapisane, w tym `Ctrl+C`; `1` w przypadku każdego pliku, którego nie udało się zapisać, pełny dysk, błąd, usunięcie tablicy lub brak jakiegokolwiek przechwycenia.
- **Microsoft Defender:** gdy program Defender zużywa jedno lub więcej rdzeni procesora na skanowanie plików zapisywanych przez przechwycenie, podczas działania wyświetla się jednorazowo na stderr komunikat: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Zobacz [Ustawienia systemowe](#windows-exe).
- Jedno ciągłe przechwytywanie na macierz w danym momencie; drugie zostanie odrzucone z komunikatem `a continuous capture is already running for this array`. W przypadku protokołu HTTP sesja ma postać `POST /api/camera/array/continuous/start` (treść `/api/camera/array/capture` wraz z `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` oraz `POST /api/camera/array/continuous/stop`.

**Interwał**(`--interval S`, z lub bez `--continuous`) oraz**`--continuous --smart`** zachowują pętlę na żądanie: jedno żądanie przechwytywania na przejście — w każdym zaplanowanym momencie dla trybu Interwał, po ustabilizowaniu się AE dla `--smart` — z plikami każdego przebiegu wymienionymi tak jak w trybie pojedynczym, wierszem `Done — N capture pass(es).` na końcu oraz `--count N` zliczającym przebiegi.

### `array-record` — połączony indeks wideo/GIF (klasa monitorowania)

Zapisuje **widok indeksu połączonego na żywo** macierzy do pliku `.avi` (i opcjonalnie do pliku `.gif`). Gdy widok połączony w interfejsie graficznym jest otwarty, zapisuje ten widok tak, jak jest wyświetlany. Jeśli żaden widok nie jest otwarty, backend sam renderuje widok: indeks przekazany za pomocą `--expr` lub domyślnie NDVI tablicy, po jej wyrównaniu. Domyślny NDVI wykorzystuje pasma czerwone i NIR najbliższe 660 nm i 850 nm, tak jak w widoku na żywo. Widok GUI otwarty podczas nagrywania jest rejestrowany tak długo, jak długo pozostaje otwarty. Sprawdza postęp co 2 s i zatrzymuje się na `--duration`, `Ctrl+C` lub gdy rejestrator samoczynnie zakończy działanie.

Nagranie, które nie zapisuje żadnej klatki, kończy się niepowodzeniem: rejestrator zgłasza kod `no-frames: …` (matryca może nadal być w trakcie wyrównywania lub ustalania ekspozycji), a polecenie **kończy działanie z kodem `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Flaga | Domyślna | Opis |
| --- | --- | --- |
| `--array-id ID` | tylko matryca | Matryca docelowa (pominąć, jeśli podłączona jest tylko jedna). |
| `-o, --output DIR` | `output` | Katalog wyjściowy. Ścieżka względna znajduje się w folderze, z którego uruchamiasz polecenie. |
| `--fps F` | `10` | Liczba klatek na sekundę podczas nagrywania. |
| `--duration S` | do naciśnięcia Ctrl+C | Automatyczne zatrzymanie po `S` sekundach. |
| `--gif` | wył. | Zapisz również animowany plik GIF. |
| `--gif-only` | wyłączone | Zapisz tylko plik GIF (bez `.avi`). |
| `--expr EXPR` | NDVI tablicy | Wyrażenie indeksowe do zapisania, gdy nie jest otwarty żaden widok GUI, w nazwach pasm tablicy (np. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; zobacz [nazwy pasm](#index--vegetation-maths)). Krótka nazwa koloru (`NIR`, `Red`) działa tylko wtedy, gdy posiada ją jedno pasmo tablicy. Zamiast tego rejestrowany jest otwarty widok GUI zgodnie z tym, co jest wyświetlane. |

### `array-burst` — seria zdjęć w formacie surowym Bayer o wysokiej częstotliwości klatek (klasa analityczna)

Odczytuje bezpośrednio bufor zsynchronizowanej grupy pętli przechwytywania — **bez łańcucha kalibracji, bez exiftool, bez konieczności podglądu na żywo** — dzięki czemu działa z pełną częstotliwością przechwytywania aparatu. Zapisuje klatki w formacie RAW + manifest dla każdej klatki + jeden plik `.daq` dla każdego odrębnego odczytu DLS w ramach `<output>/bursts/<base>/`. Przetwórz dane w trybie offline (następne polecenie) lub przekaż `--build`, aby wykonać to natychmiast po zatrzymaniu.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `--array-id ID` | tylko tablica | Tablica docelowa. |
| `-o, --output DIR` | `output` | Katalog wyjściowy (dane z serii trafiają do `<DIR>/bursts/<base>/`). |
| `--duration S` | do naciśnięcia Ctrl+C | Automatyczne zatrzymanie po `S` sekundach. |
| `--max-frames N` | bez ograniczeń | Automatycznezatrzymanie po `N` surowych klatek. |
| `--orient` | wyłączone | Produkty tworzone przez kompilację na podstawie tej serii są obracane zgodnie z orientacją kamery — produkty poszczególnych kamer o orientację ich własnej kamery, a indeks zbiorczy — o orientację kamery głównej — tak jak robi to `array-capture --orient`. Surowe klatki pozostają w formacie natywnym; `manifest.json` serii rejestruje orientację każdego elementu oraz wybraną opcję. |
| `--build` | wył. | Po zatrzymaniu natychmiast przetworzyć ponownie serię (tak samo jak w przypadku `array-build-video`). |
| `--products …` | `combined:index` | W przypadku `--build`: które pliki wideo mają zostać utworzone (patrz poniżej). |
| `--fps F` | `10` | W połączeniu z `--build`: liczba klatek na sekundę (fps) w pliku wyjściowym. |
| `--save-tiffs` | wyłączone | Wraz z `--build`: zapisuj również skalibrowane pliki TIFF dla poszczególnych klatek (każdy z [maską ograniczenia unmix](#unmix-clamp-mask-which-pixels-pasmo-zostało-ograniczone) w przypadku ograniczenia piksela; wynik kompilacji wymienia je jako `clamp_masks`). |
| `--gif` | wyłączone | Z opcją `--build`: zapisuje również animowane pliki GIF. |

### `array-build-video` — offline'owe ponowne przetwarzanie zapisanej serii

Dopasowuje czasowo każdą surową klatkę do najbliższego zapisanego odczytu `.daq` i przepuszcza ją przez **ten sam łańcuch promieniowania / odbicia / indeksu, co potok importu**, renderując jeden lub więcej filmów.

`--products` to lista elementów `kind:level` rozdzielonych przecinkami, gdzie `kind` ∈ `per_cam` | `combined` oraz `level` ∈ `radiance` | `reflectance` | `index`. Sam `level` (bez `kind:`) domyślnie przyjmuje wartość `per_cam`. Wartością domyślną jest `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flaga | Wartość domyślna | Opis |
| --- | --- | --- |
| `--burst-dir DIR` | (wymagane) | Ścieżka do folderu serii zdjęć (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, jak powyżej. |
| `--fps F` | `10` | Liczba klatek na sekundę (fps) w pliku wideo wyjściowym. |
| `--save-tiffs` | wyłączone | Zapisuj również skalibrowane pliki TIFF dla poszczególnych klatek wraz z plikami wideo. |
| `--gif` | wył. | Zapisuj również animowane pliki GIF. |

**Film nie zawiera maski ograniczającej** (jeden strumień obrazu, 8 bitów). Zamiast tego wynik kompilacji zawiera `clamp_fractions`: dla każdego produktu wideo, udział pikseli każdej klatki, które proces oddzielania kolorów ograniczył w paśmie kanału R / G / B — wartość `Chloros:Band{R,G,B}UnmixClampFraction`, którą zostałby oznaczony iloczyn promieniowania/odbicia tej klatki (dla indeksu łącznego, dla każdej z kamer w zestawie). Proces drukuje najgorszą klatkę dla każdego produktu w wierszu `Clamp:`. `--save-tiffs` dodaje maski na piksel dla plików TIFF, w pliku `clamp_masks/<level>/` obok folderu `<level>/` zawierającego pliki TIFF.**Przerwana kompilacja nie powoduje powstania uszkodzonych plików wideo.** Każdy plik wideo jest zapisywany jako `<name>.partial.avi` i otrzymuje ostateczną nazwę dopiero po zakończeniu tworzenia, więc starszy plik o tej nazwie pozostaje, dopóki jego zamiennik nie będzie kompletny. Uruchom kompilację ponownie, aby zastąpić pozostawiony plik `.partial.avi`.

Proces wyświetla każdy zapisany plik wideo, a następnie każdy plik typu `Skipped:` (żądana kamera/poziom — lub połączony indeks —, który nie wygenerował pliku wideo, wraz z przyczyną — np. brak pakietu kalibracyjnego, brak odczytu DLS dla odbicia światła lub brak pary pasm czerwonego i NIR dla indeksu łącznego) oraz wszelkie pliki `Note:`. Kamera RGB, dla której żądano promieniowania lub odbicia światła, otrzymuje zamiast tego plik wideo wyświetlającego obraz, z balansem bieli, jeśli dostępny jest pakiet kalibracyjny kamery; kamera renderująca bez niego jest wymieniona w `Note:`. Zwraca **wynik niezerowy**, gdy żądany produkt został pominięty lub nie został zbudowany zgodnie z żądaniem (skonfigurowane wyrażenie indeksu nie jest oceniane w trybie offline); nagrania, które zostały zbudowane, są nadal zapisywane i wyświetlane na liście.

> **Wybierz odpowiedni rejestrator.** `array-record` jest przeznaczony do *monitoringu* — rejestruje renderowany obraz kompozytowy na żywo: połączony widok z interfejsu graficznego lub, jeśli interfejs nie jest otwarty, widok widok NDVI (lub `--expr`). `array-burst` → `array-build-video` to *klasa analityczna* — zapisuje surowe dane z czujnika z pełną częstotliwością, a następnie odtwarza skalibrowane filmy przedstawiające promieniowanie/odbicie/wskaźniki, bez konieczności wyświetlania podglądu na żywo.

### Kamery jednopasmowe Mono (M3M)

Seria **M3M**jest monochromatycznym odpowiednikiem serii Bayer**M3C**: jedna wąskopasmowa filtracja interferencyjna na kamerę (`M3M-<lens>-F<wavelength>`, np. `M3M-L87-F685`), dzięki czemu czujnik dostarcza**pojedyncze pasmo w skali szarości** bez mozaiki Bayera. Nie ma tu nic do demosaikowania, żadnych zakłóceń międzykanałowych do rozdzielania ani balansu bieli do ustawiania — cały proces przetwarzania kolorów w systemie RGB po prostu nie ma zastosowania.

Co to oznacza w interfejsie CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**wykrywają kamerę monochromatyczną i**pomijają ją, wyświetlając jednozdaniowy komunikat** zamiast narzucać bezsensowne ustawienia. W tej samej sesji nadal działają normalnie w przypadku kamery RGB lub kamery M3C z matrycą Bayera.
- **Format pikseli Bayera oznacza jej głębię.** Matryca M3M oferuje wyłącznie formaty monochromatyczne, więc format Bayer z `--pixel-format` lub `--preset` ustawia format monochromatyczny o tej samej głębi: `BayerRG8` oraz `--preset high_speed` dają `Mono8`; `BayerRG12`, `high_quality` i `triggered` dają `Mono12`; nazwa spakowana pozostaje spakowana (`BayerRG12p` daje `Mono12p`). Pliki o nazwach `lattice format` oznaczają format Mono. Tablica łącząca kamery M3C i M3M przyjmuje jeden format w tym samym: Bayer na każdym M3C, jego głębia Mono na każdym M3M. Nazwa odpowiada jego głębi bitowej: to samo obowiązuje w drugą stronę, więc M3C nigdy nie jest umieszczana w formacie Mono (`--pixel-format Mono12` to `BayerRG12` na M3C, niezależnie od tego, która kamera jest główną w układzie), a `lattice format --set` mapuje się w ten sam sposób i wyświetla ustawiony format.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** nadal działają — promieniowanie i odbicie są mapami radiometrycznymi *dla poszczególnych pasm* i są doskonale zdefiniowane dla jednego pasma. Klatki monochromatyczne zawierają **tożsamą** macierz odpowiedzi czujnika (bez rozdzielania 3×3), więc płaszczyzna przechodzi przez obliczenia kalibracyjne bez zmian.
- **Pojedyncza kamera monochromatyczna nie może wygenerować wskaźnika wegetacyjnego.**NDVI, NDRE itp. wymagają co najmniej dwóch pasm (np. Red + NIR). Aby uzyskać wskaźnik z sprzętu monochromatycznego, należy skierować**kilka** kamer M3M na różne długości fal, wyrównać je w jeden stos wielopasmowy i obliczyć wskaźnik *dla tego* zestawu:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` sam uruchamia kamery (jako [sesja przejściowa](#multi-camera-transient-sessions)), więc najpierw nie podłączaj niczego: `--serials`, w przeciwnym razie kamery włączone w profilu (zobacz [Indeks / Obliczenia dotyczące roślinności](#index--vegetation-maths)). Gdy Chloros posiada kontrolę nad jakąkolwiek kamerą (interfejs graficzny, `cam-connect`, `array-connect`) , żądanie jest odrzucane z kodem HTTP 409 `CAMERA_HELD` i program się zamyka `1`; zwolnij je za pomocą `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel` symbole muszą **dokładnie** odpowiadać nazwom kanałów w presecie (rozróżniana jest wielkość liter; NDVI są pisane małymi literami `red`,`nir` — patrz `--list-presets`), a nazwa strony pasma musi odpowiadać pasmu w wyrównanym stosie (tryb offline akceptuje również indeksy pasm zaczynające się od 0, np. `--channel red=0 --channel nir=1`).

Elementem rozróżniającym w całym stosie jest token `M3M` w ciągu znaków modelu (nigdy nie pojawia się on w ciągu znaków `M3C`), udostępniany w interfejsie graficznym (GUI) i SDK jako `is_mono`.

---

## Konfiguracja i dostrajanie karty sieciowej hosta (układy LATTICE)

Kamery LATTICE przesyłają strumień GVSP przez kartę sieciową hosta, więc w przypadku układów wielokamerowych **sterownik**karty i**rozmiar pierścienia odbiorczego** mają równie duże znaczenie jak prędkość łącza. Nieprawidłowe ustawienia objawiają się jako `FRAMES WILL DROP` / `Reduce ROI to enable` w panelu ustawień macierzy (oraz w `lattice network-analysis` / `analyze_array_network()` w SDK), nawet jeśli same kamery działają prawidłowo.

### Karty USB 10GbE — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

Poniższe ustawienia dotyczą właściwości sterownika w systemie Windows. W systemie Linux karta USB również ma ograniczenie przepustowości (400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson) (zobacz [Model fps i burst dla macierzy](#array-fps--burst-model)), a bufory odbiorcze gniazd jądra mają znaczenie dla każdej karty — zobacz [Hosty Linux](#linux-hosts--socket-receive-buffers).

| Pozycja | Wymagana wartość | Dlaczego ma to znaczenie |
| --- | --- | --- |
| **Wersja sterownika**|**≥ v10.67 (styczeń 2026)**, INF `rtump64x64sta.inf` | Starszy sterownik z**2016**roku (v10.65, `rtump64x64.inf`) nieprawidłowo obsługuje wyłączanie zasilania i błędy systemowe związane z**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**podczas wyłączania, ponownego uruchamiania lub przechodzenia w stan uśpienia. Przejście zawiesza się (limit czasu ~5 min), użytkownik wymusza wyłączenie zasilania, a powtarzające się nieprawidłowe wyłączenia**uszkadzają repozytorium WMI**(PowerShell/narzędzia zaczynają zawodzić z błędem `Invalid class`) i**blokują stos USB** przy następnym uruchomieniu (karta sieciowa nie włącza się; napędy USB przestają być wykrywane). Przed podjęciem prób prawidłowego ponownego uruchomienia należy pobrać aktualizację ze strony realtek.com (lub od dostawcy klucza sprzętowego). |
| **Bufory odbiorcze**— słowo kluczowe `ReceiveBufferLen` |**256**(maksymalna wartość sterownika) | Pierścień odbiorczy karty sieciowej. Domyślna wartość sterownika wynosząca**32**pozostawia jedynie ~0,26 MB użytecznego pierścienia — zdecydowanie za mało na serię danych z wielu kamer — więc panel macierzy zgłasza błąd `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` i blokuje połączenia. Przy wartości**256**pierścień jest duży (**~13,5 MB zmierzone na laboratoryjnym hoście 10GbE**), co zapewnia potokowi odbiorczemu rzeczywisty zapas wydajności na serię danych GVSP z wielu kamer. (O tym, czy dana konfiguracja faktycznie *nawiąże połączenie*, decydują dwie kontrole — **kontrola przepustowości**oraz**agregowana nadsubskrypcja** — a nie na podstawie surowego porównania wielkości serii z pojemnością pierścienia; zobacz [Model fps i serii macierzy](#array-fps--burst-model).) |
| **Odbiorcze URB**— słowo kluczowe `PendingReceives` |**64** (maks.) | Bloki żądań USB w trakcie przesyłania; należy zwiększać wraz z buforami odbiorczymi w celu absorpcji serii. |
| **Ramka typu jumbo** — słowo kluczowe `*JumboPacket` | **9014** | Wymagane dla pakietów GVSP o rozmiarze 9000 bajtów (6× mniej pakietów na ramkę niż w przypadku 1500). |

> ⚠️ **Aktualizacja sterownika karty sieciowej RESETUJE te zaawansowane właściwości do wartości domyślnych.**Po aktualizacji lub wymianie sterownika karty sieciowej należy**ponownie zastosować** `ReceiveBufferLen=256` i `PendingReceives=64`, w przeciwnym razie panel macierzy ponownie zablokuje dostęp, mimo że „nic się nie zmieniło w sprzęcie”. Jest to główna przyczyna, dla której wcześniej działająca konfiguracja nagle odmawia połączenia.

Zastosuj z poziomu **z podwyższonymi uprawnieniami** PowerShell (zastąp nazwę swojej karty, np. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` obejmuje karty sieciowe USB 10GbE.** Teraz wykrywa typ karty i dostosowuje właściwe słowo kluczowe pierścienia odbiorczego: `*ReceiveBuffers`→2048 dla kart sieciowych PCIe (Intel I219, itp.) lub `ReceiveBufferLen`→256 + `PendingReceives`→64 dla kontrolera Realtek **USB** 10GbE (który nie udostępnia `*ReceiveBuffers`). Wartości docelowe są ograniczane do maksymalnej wartości zgłaszanej przez dany sterownik (`NumericParameterMaxValue`), więc nigdy nie zapisuje się wartości spoza zakresu. Uruchom to z terminala z **podwyższonymi** uprawnieniami; podobnie jak w przypadku każdej modyfikacji opartej na rejestrze, zmiana zacznie obowiązywać po ponownym uruchomieniu karty sieciowej lub ponownym uruchomieniu komputera. Powyższe ręczne polecenia `Set-NetAdapterAdvancedProperty` pozostają dobrą alternatywą — działają na bieżąco (ponownie wiążą kartę) bez konieczności ponownego uruchamiania.

### Komputery z systemem Linux — bufory odbiorcze gniazd

W systemie Linux, w tym na platformie NVIDIA Jetson, bufory odbiorcze gniazd w jądrze mają równie duże znaczenie, co pierścień odbiorczy karty sieciowej. Przy domyślnych ustawieniach jądra (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 bajtów) seria pakietów GVSP z kamery powoduje przepełnienie gniazda strumieniowego, zanim program Chloros zdąży je odczytać. Pakiety są odrzucane w jądrze, podczas gdy sama karta sieciowa nie zgłasza żadnych strat, a macierz traci grupy klatek i zmniejsza swoją częstotliwość odświeżania. W pomiarach przeprowadzonych na macierzy składającej się z 4 kamer podłączonych do karty USB 10 GbE zwiększenie obu wartości do 32 MB spowodowało wzrost liczby klatek na sekundę w macierzy z 4,96 do 7,52 po nawiązaniu połączenia, a w przypadku ciągłego przechwytywania surowych danych – z 4,7 do 7,6 przechwyceń na sekundę, przy czym w jądrze nie odnotowano żadnych utraty pakietów.

- **Plik .deb ustawia te wartości.** Instalacja pakietu zapisuje wartości `/etc/sysctl.d/60-chloros-gige.conf` (obie na poziomie `33554432`) i natychmiast je stosuje. Wartość, która została już ustawiona na wyższy poziom, pozostaje bez zmian.
- **`lattice network` sprawdza te wartości.** Gdy któraś z tych wartości jest niższa, w sekcji *Wykryte problemy* wyświetla się wpis `socket receive buffers too small for GigE Vision (…)`, a dziennik backendu zawiera to samo ostrzeżenie przy pierwszym podłączeniu kamery lub macierzy.
- **Aby ręcznie zwiększyć te wartości** (do następnego restartu; plik pakietu stosuje je przy każdym uruchomieniu):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` również podnosi i utrzymuje te wartości, ale tylko wtedy, gdy backend działa jako root. Backend działający jako użytkownik (uruchomiony z `chloros-backend` lub [`chloros-backend@<user>`](#uruchamianie-backendu-jako-usługi-bez-interfejsu-graficznego)) nie może zmieniać ustawień jądra i zamiast tego wyświetla wiersz `sudo sysctl`.

**Automatyczne dostosowywanie kart sieciowych dotyczy wyłącznie podłączonych kart.** Przy pierwszym podłączeniu kamery backend działający jako root dostosowuje karty sieciowe, których pierścień odbiorczy jest mały. Obecnie optymalizuje tylko karty z aktywnym połączeniem, więc odłączony port wbudowany zachowuje swoją wartość MTU, a bufory odbiorcze pozostają bez zmian. Komunikat `lattice network` nie wymienia już MTU odłączonej kartyMTU jako problem. `lattice network`, `--fix`, `lattice probe` oraz optymalizacja kart sieciowych uwzględniają wyłącznie przewodowe porty Ethernet, a nie magistralę CAN urządzenia Jetson (`can0`) ani połączeń w trybie urządzenia USB (`usb0`, `usb1`), które oferuje komputerowi hosta; adapter Ethernet USB dla kamer nadal się liczy.

### Podstawy sieciowe (wszystkie łącza LATTICE)

- **Adresowanie:** adres lokalny łącza `169.254.0.0/16` (GigE Vision LLA). Komputer hosta przyjmuje statyczny adres `169.254.x.x/16`; kamery i moduł DAQ-E samodzielnie przypisują sobie adresy w tym samym zakresie. Nie jest wymagany DHCP ani brama.
- **Rozmiar pakietu:**preferowany jest rozmiar jumbo (9000), ale należy pozwolić, aby funkcja automatycznego wykrywania go ustaliła — omija ona ograniczenie ICMP kamery wynoszące 1500 bajtów za pomocą sondy GVSP, dzięki czemu ustala rozmiar jumbo wszędzie tam, gdzie linia faktycznie go obsługuje.**Każda kamera jest mierzona raz na sesję serwera zaplecza oraz ponownie w przypadku zmiany jej połączenia (inna karta sieciowa, inna prędkość łącza lub utrata połączenia, np. w wyniku zamiany przełącznika); przy ponownym połączeniu wykorzystywany jest poprzedni wynik. Po zmianie za przełącznikiem, który pozostał aktywny, należy ponownie uruchomić serwer zaplecza, aby przeprowadzić pomiar od nowa.**Pierwsze połączenie może czasami zająć kilka minut podczas tej kontroli, a dziennik informuje o tym, gdy tak się dzieje. Ustaw stałą wartość (pin) z kodem `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` tylko wtedy, gdy wiesz więcej niż sonda, i preferuj ustawienia na poszczególne polecenia zamiast stałych: stała wartość pomija sondę, więc jeśli ścieżka faktycznie nie jest w stanie przesyłać 9000,**każde** przechwycenie zakończy się przekroczeniem limitu czasu z błędem `SC_ERR_TIMEOUT -1011` (zobacz [Zmienne środowiskowe](#environment-variables)).
- **Pierścień RX skaluje się wraz z `ReceiveBufferLen`:**przy domyślnej wartości `32` użyteczny pierścień wynosi ~0,26 MB (zbyt mały dla jakiejkolwiek serii z wielu kamer); przy maksymalnej wartości `256` jest on duży (~13,5 MB zmierzone na laboratoryjnym hoście 10GbE), co zapewnia rzeczywisty zapas. To, czy konfiguracja zostanie połączona, zależy wówczas od kontroli dopuszczalności uwzględniającej obciążenie**oraz** od poniższej kontroli agregowanej nadsubskrypcji — a nie od surowego porównania serii z pierścieniem.

### Liczba klatek na sekundę macierzy i model transmisji seryjnej

Jak odczytywać panel ustawień macierzy (oraz `lattice analyze-array` / `analyze_array_network` w SDK):

- **Wartość burst jest sumowana dla każdej kamery w rzeczywistym formacie pikseli danej kamery.**Kamery Mono**M3M**przesyłają strumień w formacie**Mono12 (2 b/px)**lub, w przypadku `--pixel-format`, w formacie Mono odpowiadającym jej głębi (`BayerRG8` to `Mono8`, 1 bit na piksel); kamery**M3C**typu Bayer przesyłają strumień w formacie 8- lub 12-bitowym (oprogramowanie sprzętowe M3C domyślnie generuje BayerRG12, nawet gdy żądany jest BayerRG8). Zatem klatka w pełnej rozdzielczości z 4 kamerami ma**~12,6 MB, jeśli wszystkie są 8-bitowe, ale ~25 MB w przypadku trzech 12-bitowych kamer mono**. Projekcja rozpoznaje format każdej kamery na podstawie jej modelu (pamięć podręczna tożsamości), dzięki czemu transmisja seryjna odpowiada temu, co faktycznie przesyła łącze — a nie opiera się na uniwersalnym założeniu formatu BayerRG8.
- **Prędkość adaptera USB Ethernet jest ograniczona do 200 MB/s, niezależnie od danych na tabliczce znamionowej.** Tabela wydajności, która przekształca prędkość łącza na wartość ciągłą, pochodzi z PCIe; karta sieciowa USB podaje swoją prędkość łącza *Ethernet*, ale jest ograniczona przez magistralę USB i jej sterownik. Klucz USB 10GbE osiągał prędkość ~1063 MB/s „stałą” prędkość — wartość, która nigdy nie została zweryfikowana — a wynikające z tego spowolnienie powodowało uszkodzenie 6–18 % ramek, mimo że nadal zgłaszano prawidłową docelową liczbę klatek na sekundę. Karty sieciowe podłączane przez USB mają obecnie absolutne ograniczenie: **200 MB/s** w systemie Windows i na platformie NVIDIA Jetson, **400 MB/s** w systemie Linux x86_64 (w systemie Linux karta podłączona do portu USB 2.0 ma przydzieloną przepustowość **30 MB/s**, a w specyfikacji zaleca się podłączenie jej do portu USB 3) (ograniczeniem jest magistrala USB i sterownik komputera, więc prędkość nie jest proporcjonalna do parametrów znamionowych; karta USB 1 GbE osiąga ~80 MB/s i nie ma to na nią wpływu). W systemie Linux a adapter jest identyfikowany na podstawie sysfs (w razie potrzeby z wykorzystaniem `ethtool -i` lub `bus-info`). Każda wartość została zmierzona: w systemie Windows ten sam układ czterech kamer powodował uszkodzenie 6–10 % klatek przy prędkościach powyżej 200 MB/s; w systemie Linux x86_64 ta sama karta działała bezbłędnie przy każdej prędkości do 480 MB/s (przesyłano 273 MB/s, co stanowiło ograniczenie samych kamer). Na platformie NVIDIA Jetson ta sama karta działała bezbłędnie tylko przy prędkości 200 MB/s, a powyżej tej wartości sporadycznie traciła klatki. `wire_ceiling_source` w opisie możliwości potwierdza to słowami, a `nic_is_usb` sygnalizuje ten fakt. W obu przypadkach można to nadpisać za pomocą `--wire-ceiling-mbps`.
- **Dopuszczalność uwzględnia obciążenie drenu, a nie porównanie całej serii z pierścieniem.** Jednoczesna seria musi zmieścić się jedynie w *przejściowym zaległości* = `max(0, Σ per-cam arrival − host drain) × emit_window`, a nie w całej serii. W przypadku struktury z szybkim hostem i wolnymi kartami (**PCIe**host 10G + 4× karty 1 GbE: przychodzące dane ≈ 320 MB/s, odprowadzanie ≈ 1063 MB/s) host opróżnia się szybciej niż kamery się zapełniają, zaległość ≈ 0, więc symulowana emisja w pełnej rozdzielczości**jest dopuszczana**, mimo że seria 25 MB przekracza pojemność pierścienia wynoszącą 13,5 MB. Umieść te same cztery kamery za adapterem**USB** 10GbE w systemie Windows, a prędkość wysyłania wyniesie 200 MB/s, a nie 1063 — prędkość przychodząca przewyższa prędkość wysyłania, a utrata danych objawia się jako uszkodzone ramki, a nie jako niższa liczba klatek na sekundę. Na hoście 1 GbE dolna granica przepustowości pobierania (DLThr) kamer wynosząca 31,25 MB/s sprawia, że przybywające dane przewyższają przepustowość odbioru → system poprawnie **blokuje** (dla *tej* klasy blokad należy zmniejszyć obszar zainteresowania (ROI) lub zastosować binning ≥ 2). Dopuszczalność jest jedną z **dwóch** bramek łączących — drugą jest opisana poniżej kontrola agregowanej nadmiernej subskrypcji.
- **Prognozowana liczba klatek na sekundę (fps) odpowiada cyklowi najwolniejszej kamery.**W trybie jednoczesnego wysyłania każda kamera wysyła swoją klatkę w tym samym czasie, więc macierz pracuje w cyklu odpowiadającym dłuższemu z dwóch czasów: odczytu czujnika najwolniejszego elementu oraz czasu wysyłania (liczba bajtów klatki ÷ szybkość wysyłania kamery:**łącze dostępowe** kamery, 1 GbE ≈ 80 MB/s lub jej udział w maksymalnej przepustowości łącza hosta, jeśli jest ona mniejsza). Każda ekspozycja nakłada się na odczyt poprzedniej klatki, więc ekspozycja nie wchodzi w cykl: ekspozycja automatyczna wypełnia okres wyznaczony przez częstotliwość, a następnie zwiększa wzmocnienie, a słabo oświetlona scena nie obniża już częstotliwości (w wersji 1.2.0 tak było). Jedynie ręczna ekspozycja trwająca dłużej niż ten okres spowalnia wyzwalanie (zobacz [Ekspozycja w elemencie macierzy](#exposure-on-an-array-member)). Dodanie kamer nie spowalnia macierzy, dopóki ich udział w przepustowości nie zmniejszy się. W trybie emisji szeregowej (`slip-emit-and-capture`) czasy emisji kamer sumują się, a czas naświetlania każdego elementu jest uwzględniany w rozłożeniu w czasie. Pomiar przy pełnym czujniku, 12 bitów, na macierzy 9 kamer z hostem PCIe 10 GbE: 10,7 klatek na sekundę lub 14,0 klatek na sekundę przy [transporcie spakowanym](#packed-12-bit-transport---packed).
- **Po zakończeniu ciągłego przechwytywania projekcja pokazuje, jaką częstotliwość utrzymała.** Przechwytywanie trwające 30 s lub dłużej przy automatycznej częstotliwości rejestruje częstotliwość, na której ustabilizowała się przy danej wielkości macierzy na tym komputerze; pomijane są krótkie próby kontrolera częstotliwości z zastosowaniem szybszej częstotliwości. Od tego momentu wiersz `array-connect` oraz panel „Ustawienia macierzy” podają tę częstotliwość, gdy jest ona niższa od modelowej, wraz z informacją o tym, co ją ograniczyło: `clock-bound` (macierz działała z częstotliwością, na której się ustabilizowała, pętla przechwytywania czekała na zegar wyzwalający), `loop-bound` (pętla przechwytywania nie miała już czasu), `capture-bound` (przechwytywanie nie nadążało i pomijało grupy) lub `measured` (nie wiadomo, który z nich). Dane modelu znajdują się w `wire_fps_max`. Tablica nadal osiąga wartość odpowiadającą przepustowości łącza.
- **Macierz działa w każdej sieci — wszystko zależy od liczby klatek na sekundę.**Każda kamera otrzymuje równy udział w**bezpiecznym pod względem kolizji limicie łącza**(`sustained × sim_emit_factor`), zsynchronizowanym z GevSCPD, a wszystkie nadal eksponują przy tym samym wyzwalaczu. W przypadku wolnego łącza udział ten jest niewielki, a wraz z nim spada liczba klatek na sekundę: 8 kamer o pełnej rozdzielczości i 12-bitowej głębi kolorów podłączonych do sieci 1 GbE bez ramek jumbo otrzymuje po 52 / 8 = 6,5 MB/s każda, co daje około**1 klatki na sekundę**(około 1,3 klatki na sekundę po kompresji), podczas gdy host 10G obsługuje je z prędkością 11–14 klatek na sekundę. Jeśli kamer nie da się zsynchronizować w tak wolnym tempie — sprawdź to, odczytując zakres GevSCPD każdej kamery — macierz przełącza się na**szeregowe**wysyłanie (`slip-emit-and-capture`): kamery korzystają z łącza na zmianę z pełną prędkością łącza, uruchamiane jedna po drugiej, więc każda ramka nadal dociera, ale elementy**nie są eksponowane jednocześnie**. `analyze-array` oraz nazwa wyjścia polecenia `connect` określają tryb; należy ustawić `--force-tier sim-capture-sim-emit` na stałą wartość, aby odrzucić tryb awaryjny. Rozmiar ramki nie ma wpływu na ten podział, więc**binning i mniejszy obszar zainteresowania (ROI) zwiększają częstotliwość klatek, ale nie zmieniają tego, czy macierz zmieści się**. Tylko linia, która nie jest w stanie przesyłać tablicy nawet szeregowo, jest**przeciążona**— np. znacznie więcej kamer niż jedna linia jest w stanie obsłużyć lub wiele innych kamer przesyłających strumień na tej samej linii — i w tym przypadku objawem jest utrata pakietów GVSP, a nie niższa szybkość: `analyze-array` zeruje wartości osiągalnej liczby klatek na sekundę i wyświetla komunikat `** OVER-SUBSCRIBED**`, połączenie zostaje nawiązane z głośnym ostrzeżeniem, a połączenie HTTP z komunikatem `pin_resolution: true` (tryb ścisły) **odrzuca**. Rozwiązaniem jest zmniejszenie liczby kamer, zastosowanie ramek typu jumbo w całej sieci lub szybsza karta sieciowa. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zmniejsza poziom odrzucenia do głośnego ostrzeżenia w przypadku testów porównawczych — zobacz [Zmienne środowiskowe](#environment-variables).

### Stan macierzy — który podsystem traci ramki

`GET /api/camera/array/<array_id>/capability` podłączonej macierzy zawiera aktualny
blok `health`, ponownie oceniany w ruchomym oknie **10-sekundowym**. Podziela on utratę ramek
na dwie przyczyny wymagające przeciwnych działań naprawczych, zamiast zgłaszać jeden wskaźnik „niekompletności”,
który nie określa żadnej z nich:

| Pole | Co oznacza | Który podsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na port szeregowy) | Ramka **dotarła, ale była uszkodzona strukturalnie**— utrata pakietów GVSP. |**Sieć**: przepustowość łącza, taktowanie, pierścień odbiorczy karty sieciowej, MTU |
| `never_arrived_rate_pct` (na numer seryjny) | Ramka **w ogóle nie dotarła**— kamera nie wykonała zdjęcia lub nic z niej nie wyszło. |**Wyzwalacz / synchronizacja**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najgorszy wskaźnik dla każdej kamery. | — |
| `per_cam_rate_pct` | Łączny wskaźnik niekompletności na kamerę (obie przyczyny łącznie). | — |
| `stable_for_seconds` | Jak długo każda kamera pozostawała poniżej 0,01 %. | — |

Powyżej 5% backend rejestruje wiersz `[array-health <id>] WARN` z nazwą podziału — przy
pierwszym naruszeniu, przy zmianie przedziału ważności, raz na minutę, dopóki stan ten utrzymuje się, oraz raz, gdy
stan ten ustąpi. Uszkodzona połowa zapisuje wiersz `[gvsp-corrupt <SN>]` do pliku dziennika zaplecza przy
pierwszym wystąpieniu dla każdej kamery i przyczyny, a następnie podsumowanie co 60 s; panel dziennika aplikacji wyświetla zamiast tego wiersz
zamiast tego wiersz `[array-health <id>]`. Każda ocena nadal trafia do pliku dziennika zaplecza;
liczniki zwiększają się przy każdym buforze, niezależnie od tego, co jest wyświetlane.

Ten sam rekord podaje liczbę, od której zależy cała alokacja:

| Pole | Co oznacza |
| --- | --- |
| `wire_ceiling_mbps` | Aktualny, stały limit przepustowości hosta, MB/s. |
| `wire_ceiling_source` | Skąd pochodzi ta liczba, w słowach — np. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (w systemie Linux x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) lub `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, gdy wartość ta została ustawiona przez `--wire-ceiling-mbps` (lub pole **Wire Budget** w interfejsie graficznym). |
| `nic_is_usb` | `true` dla adaptera Ethernet USB — patrz ograniczenie USB powyżej. |

**Interpretacja:** wartość `gvsp_corrupt_rate_pct` różna od zera przy wartości `never_arrived_rate_pct` równej 0
oznacza, że wyzwalanie i synchronizacja kabla są idealne, a 100% strat występuje na ścieżce sieciowej
— zmniejsz wartość `--wire-ceiling-mbps` i podłącz ponownie. Odwrotny wzorzec wskazuje natomiast na
kabel synchronizacyjny lub linię wyzwalającą.

> **Wartość `--target-fps` nie jest czynnikiem decydującym o uszkodzonych ramkach.** Tempo GevSCPD jest ustalane
> jednorazowo przy nawiązaniu połączenia, więc obniżenie częstotliwości wyzwalania zmienia cykl pracy, a nie
> szybkość transmisji w trybie jednoczesnej emisji. Zmniejszenie zapotrzebowania o 5× nie przyniosło poprawy;
> obniżenie limitu przepustowości z 240 do 200 MB/s spowodowało, że w tym samym układzie odsetek
> uszkodzonych ramek spadł z 10,4 % do 0,00 %.

> **Działająca macierz nie może samodzielnie przeplanować się w oprogramowaniu układowym LATTICE (M3C / M3M).** Nie
> może samodzielnie tego naprawić; należy odłączyć i ponownie podłączyć, aby moduł wyboru czasu połączenia mógł
> ponownie zaplanować działanie z nową wartością maksymalną.

### Objaw → rozwiązanie

| Objaw (Ustawienia macierzy / połączenie / `analyze_array_network`) | Przyczyna | Rozwiązanie |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` resetuje się do 32 (zazwyczaj po aktualizacji sterownika) | Ustaw `ReceiveBufferLen`→256, `PendingReceives`→64; ponownie otwórz panel (uruchom ponownie backend, jeśli zapamiętał w pamięci podręcznej stary rozmiar pierścienia) |
| Zawieszanie się podczas restartu/wyłączania; później błędy WMI `Invalid class`, karta sieciowa nie włącza się, brakuje napędów USB | Stary sterownik Realtek USB 10GbE z 2016 r. → BSOD `0x9F` → wymuszone wyłączenia zasilania | Zaktualizuj sterownik karty do wersji ≥ v10.67 (2026), a następnie ponownie zastosuj powyższe ustawienia pierścienia odbiorczego |
| Macierz łączy się przy pełnej czułości czujnika, ale z niską liczbą klatek na sekundę | Tryb automatyczny utrzymuje pełną czułość czujnika i pozwala, by liczba klatek spadła do wartości, jaką przenosi łącze | Wybierz jedną z szybszych opcji z listy wyjść połączeń (`--binning 2` / `4` lub mniejszą `--width`/`--height`), włącz ramki jumbo lub zmodernizuj łącze |
| Macierz zgłasza prawidłową docelową liczbę klatek na sekundę, ale dostarcza jedynie jej ułamek; `health.gvsp_corrupt_rate_pct` niezerowe, `never_arrived_rate_pct` 0 | Wyprowadzony budżet przepustowości hosta jest zawyżony w stosunku do rzeczywistej przepustowości (typowe w przypadku karty Ethernet USB, wąskiej linii PCIe lub współdzielonej sieci) | Należy ponownie nawiązać połączenie z niższą wartością `--wire-ceiling-mbps` i ponownie sprawdzić blok stanu. **Nie** `--target-fps` — tempo GevSCPD jest ustalone w momencie połączenia |
| Linux: macierz traci grupy ramek i zmniejsza swoją częstotliwość ramek, podczas gdy karta sieciowa nie zgłasza żadnych utrat; `lattice network` wyświetla `socket receive buffers too small` | Bufory odbiorcze gniazd jądra mają wartości domyślne, więc pakiety z kamery są odrzucane w jądrze | Zwiększ wartość `net.core.rmem_max` / `rmem_default` do 32 MB — zobacz [hosty z systemem Linux](#linux-hosts--socket-receive-buffers). Pakiet .deb robi to podczas instalacji |
| W opublikowanych grupach brakuje kamer; wartość `health.never_arrived_rate_pct` jest niezerowa, `gvsp_corrupt_rate_pct` 0 | Wyzwalacz / ścieżka synchronizacji — kamery nie uruchamiają się, nie jest to problem sieciowy | Sprawdź kabel synchronizacyjny M8 oraz `--line`; upewnij się, że wszystkie elementy systemu są uzbrojone (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` przekroczone w `analyze-array` lub odmowa połączenia w trybie ścisłym (`pin_resolution`; `array over-subscribes the wire`) | Kable nie są w stanie obsłużyć macierzy przy żadnej częstotliwości pracy, jaką osiągają kamery, ani jednocześnie, ani po kolei — jest znacznie więcej kamer, niż kable mogą obsłużyć, lub inne kamery przesyłają dane tym samym łączem | Mniejsza liczba kamer, ramki typu jumbo w całym łańcuchu lub szybsza karta sieciowa. **ROI/binning NIE pomogą** (współdzielenie jest niezależne od rozmiaru ramki). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zastępuje ustawienia na stanowisku testowym (akceptuje utratę pakietów) |
| Macierz łączy się w `slip-emit-and-capture` (szeregowo) w wolnej sieci | Kamer nie da się zsynchronizować na tyle wolno, by mogły jednocześnie korzystać z łącza, więc działają na zmianę; ramki nie są eksponowane w tym samym momencie | Ramki typu jumbo lub szybsza karta sieciowa przywracają jednoczesne przechwytywanie; `--force-tier sim-capture-sim-emit` nalega na to (i zamiast tego traci pakiety) |

---

## `chloros-cli daq`

Polecenia czujnika spektralnego. Polecenia `pool-*` są „cienkimi” klientami HTTP, które sterują czujnikiem poprzez stałą pulę backendu, więc wymagają uruchomionego backendu (zobacz [Uruchom backend](#start-the-backend)). Backend jest właścicielem kanału transportowego, więc skrypty CLI i SDK współdzielą jeden aktywny uchwyt zamiast rywalizować o port szeregowy. `pool-discover` to jedyny sposób na poznanie adresu MAC BLE urządzenia DAQ-M.

> **`chloros-cli daq --help`** (lub `-h`, lub `daq help`) wyświetla listę podpoleceń `pool-*` (w systemie Linux wraz z poleceniami bezpośredniego sterowania sprzętem, które są dołączone do kompilacji), a `chloros-cli daq <subcommand> --help` (lub `daq help <subcommand>`) podaje opcje każdego z nich. Obejmują one nawiązywanie połączenia, strumieniowanie, rejestrowanie skalibrowanych plików `.daq`, zamianę profili pojemnościowych oraz powiązanie modułu DAQ z kamerą, a w przypadku modułu DAQ-E – jego ustawienia sieciowe, PTP, ponowne uruchomienie i aktualizacje oprogramowania układowego. Urządzenie DAQ można również sterować z poziomu języka Python za pomocą polecenia `chloros_sdk.connect_daq_sensor()`, które korzysta z tej samej ścieżki zbiorczej.
>
> **Należy używać poleceń `pool-*` na każdej platformie.** Działają one identycznie w systemach Windows, Linux i Jetson. W systemie Windows `daq discover` i `daq list` są aliasami polecenia `pool-discover`, a każde inne polecenie `daq`, które nie jest `pool-*` kończy działanie `1` z `DAQ direct-hardware commands require the full daq package`. W systemach Linux i Jetson polecenia `daq discover`, `daq list` oraz inne polecenia `pool-*` i `daq`, które nie są `daq discover`, są bezpośrednimi-sprzętowe narzędzia DAQ: otwierają czujnik w procesie należącym do samego CLI, poza pulą backendu (a więc nie na czujniku utrzymywanym przez backend, GUI lub skrypt), a `daq discover` przyjmuje tam `--only` i `--timeout`, ale nie `--json`. Niniejszy dokument opisuje wyłącznie polecenia `pool-*`.

### Przebieg pierwszego połączenia z czujnikiem DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Dokumentacja `pool-*`

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otwiera czujnik w puli zaplecza. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U na określonym porcie szeregowym. |
| `daq pool-connect --ble` | DAQ-M przez BLE, automatyczne skanowanie adresów MAC. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M przez BLE przy znanym adresie MAC (zakłada `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E przez Ethernet przy znanej nazwie hosta lub adresie IP (oznacza `--eth`). |
| `daq pool-connect --eth` | DAQ-E przez Ethernet, automatyczne wykrywanie hosta (mDNS + rezerwowe ARP; działa przy pustej pamięci podręcznej ARP w systemach Windows i Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Dostosuj okno integracji / stan AE. |
| `daq pool-connect --no-stream` | Połącz się, ale nie rozpoczynaj jeszcze przesyłania strumieniowego (wznowienie za pomocą `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcji Cap. Domyślnym ustawieniem po stronie serwera jest `sunshine_cosine`; `none` to czujnik bez osłony. Czujnik typu [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignoruje go (stały dyfuzor). **Czujniki są dostarczane bez osłony, więc należy nadać nazwę zamontowanej osłonie:** domyślna wartość jest rejestrowana jako zakładana (`cap_id_source=auto_default`), a system przetwarzania wyświetla ostrzeżenie, ponieważ goły czujnik DAQ-U przy korekcji nasłonecznienia odczytuje wartości około 30 razy wyższe. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skanuj każdy transport w poszukiwaniu czujników, z którymi można się połączyć, bez nawiązywania połączenia. **W ten sposób można znaleźć adres MAC BLE modułu DAQ-M.** `--timeout` dotyczy poszczególnych protokołów (domyślnie 5 s, 1–20). Czujniki już otwarte w puli nie są wyświetlane — podłączony moduł DAQ-M przestaje nadawać sygnał — więc w tym przypadku należy użyć polecenia `pool-list`. `usb` wyświetla wyłącznie urządzenia szeregowe USB: własne porty szeregowe komputera (`COM1`, `/dev/ttyS*` w systemie Linux) nie mogą pełnić funkcji czujników i są pomijane. |
| `daq pool-list` | Wyświetla wszystkie czujniki w puli zaplecza. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Powiązuje kamerę lub tablicę z modułem DAQ, tak jak robi to wybór czujnika światła w interfejsie graficznym: jej przechwytywania wykorzystują, dopasowują i zapisują odczyty tego modułu DAQ. `SENSOR` to identyfikator czujnika z `pool-list` lub adres, pod którym dostępny jest moduł DAQ. `--clear` (brak `SENSOR`) powoduje usunięcie powiązania, a przechwytywanie powraca do najnowszego modułu DAQ. Powiązanie trwa do momentu ponownego uruchomienia backendu lub do momentu odłączenia tej kamery, matrycy lub modułu DAQ: odłączenie (w tym przypadku z poziomu SDK lub w aplikacji) powoduje zerwanie wszystkich powiązań z odłączanym sprzętem, niezależnie od tego, kto je ustawił, a powiązania elementów matrycy z samą matrycą. Kamera musi być podłączona, znajdować się w otwartym projekcie lub została wcześniej otwarta na tym komputerze, a matryca musi być podłączona lub znajdować się w otwartym projekcie; każdy inny numer seryjny lub identyfikator jest odrzucany na podstawie nazwy i nic nie jest powiązane. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Zwolnienie — niezależnie od tego, kto otworzył czujnik, więc zwalnia to również czujnik pozostawiony w puli przez skrypt, który uległ awarii (wszelkie nagrania na nim są najpierw kończone). Czujnik, który nie znajduje się w puli, nie ma nic do zwolnienia: wyświetla komunikat `ID: already released (not in the pool)` i kończy działanie `0`; zwolnienie, które backend odrzucił lub nie potwierdził, kończy działanie `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Najnowsze N ramek widma (domyślnie 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Wznowienie / wstrzymanie strumieniowania. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Rozpoczęcie / zatrzymanie nagrywania pliku .daq. `-o` domyślnie to `~/Documents/DAQ Live View` na komputerze serwera (ścieżka względna znajduje się w folderze, z którego uruchamiane jest polecenie); `--duration` (`-d`, domyślnie `0` = do `--stop`). `--confirm-after S` wyświetla ostrzeżenie, gdy po upływie S sekund od rozpoczęcia nie nadeszło żadne odczyt (domyślnie 20; 0 wyłącza tę funkcję); w przypadku `--duration`, `--heartbeat S` wyświetla liczbę odczytów co S sekund i ostrzega, gdy przestaje ona rosnąć (domyślnie 30; 0 wyłącza tę funkcję). Rejestracja na urządzeniu DAQ-E, który przestaje wysyłać dane, jest kontynuowany w tym samym pliku po ponownym nawiązaniu połączenia przez backend, a `--stop` wyświetla listę luk ([DAQ-E, który przestaje dostarczać dane](#a-daq-e-that-stops-delivering)). Kombinacja klawiszy Ctrl+C przedwcześnie kończy nagrywanie `--duration` i zgłasza to; bez `--duration` nagrywanie trwa po naciśnięciu Ctrl+C (wyjście z `130`) aż do `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmiana profilu korekcji pojemnościowej w trakcie działania. Wymagane jest `--cap-id`; `--cap-id none` usuwa korekcję pojemnościową (czujnik bez osłony). Urządzenie DAQ-E-S zachowuje ustawienie `as_recorded` niezależnie od tego, co zostało ustawione. |
| `daq pool-enable-ptp --sensor-id ID` | Ponowne włączenie PTP dla urządzenia DAQ-E, którego PTP jest wyłączone. Ustawienie jest zapisywane w urządzeniu, więc urządzenie pozostawione w stanie wyłączonym pozostaje wyłączone — jego znaczniki czasu nie są zsynchronizowane z PTP — dopóki nie zostanie uruchomione to polecenie. `pool-list` oznacza takie urządzenie (`!! PTP is switched OFF`) i wyświetla dla niego to polecenie. Chloros nigdy samoczynnie nie włącza urządzenia z powrotem, ponieważ wyłączenie może być zamierzone. |
| `daq pool-info --sensor-id ID [--json]` | Własne ustawienia urządzenia DAQ-E: jego oprogramowanie układowe (oraz informacja, czy ta wersja Chloros a zawiera nowszą wersję), jego adres i sposób jego uzyskania (DHCP / link-local / statyczny), zapisane ustawienia sieciowe do momentu uruchomienia czujnika (w tym te zapisane wraz z `--no-reboot`), strumień danych po włączeniu zasilania oraz PTP. W przypadku czujnika, którego strumień danych nie dociera do tego komputera, odczytuje on czujnik przy nowym połączeniu i informuje o tym. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Ustaw czujnik DAQ-E na DHCP (`--auto`) lub adres statyczny (`--mask` domyślnie `255.255.255.0`). Czujnik uruchamia się ponownie w celu zastosowania tych ustawień i ponownie łączy się z dotychczasowymi ustawieniami, czekając na to do `--timeout` sekund (domyślnie 45); `--no-reboot` jedynie zapisuje te ustawienia. Zobacz [Ustawienia urządzenia DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Czy urządzenie DAQ-E rozpoczyna transmisję po włączeniu zasilania bez podłączonego oprogramowania (zapisane w czujniku). Program Chloros uruchamia transmisję niezależnie od tego ustawienia. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Zrestartuj urządzenie DAQ-E i podłącz je ponownie z poprzednimi ustawieniami (oczekiwanie trwa do `--timeout`, domyślnie 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Wgranie oprogramowania sprzętowego DAQ-E dołączonego do tej wersji Chloros na czujnik z starszą wersją, a następnie ponowne podłączenie go (trwa to około minuty; nie wyłączaj zasilania; domyślny czas `--timeout` na przywrócenie działania wynosi 45 s). Czujnik, który ma już aktualną wersję oprogramowania, pozostaje bez zmian. |

### DAQ-E, który przestaje przesyłać dane

DAQ-E w puli, który nie przesyła widma przez 20 sekund, jest ponownie podłączany przez backend. Odbywa się to poprzez wyłączenie i włączenie przełącznika zasilającego czujnik: czujnik uruchamia się ponownie wraz z nim i nie przesyła danych, dopóki nie otrzyma polecenia (chyba że `pool-set-auto-stream --on`).

- **Najpierw sprawdza, czy jest to ten sam czujnik.** Wysyła zapytanie na adres, który miał czujnik, nie zmieniając niczego, o nazwę czujnika, adres MAC i numer seryjny. Wysyła również zapytanie pod nazwą czujnika `daq-e-<id>.local`, ale ufa tej odpowiedzi tylko wtedy, gdy pochodzi ona z własnej karty sieciowej czujnika (adres MAC). Żadne inne urządzenie, które odpowie, nie zostanie ponownie podłączone ani nie zostaną do niego zapisane żadne dane.
- **Nawiązuje ponownie połączenie z zachowaniem ustawień sesji:** czasu integracji, uśredniania klatek, automatycznej ekspozycji i profilu ograniczenia, tak jak były, a nie domyślnych ustawień czujnika po włączeniu zasilania. Czujnik, który miał włączony protokół PTP, a powrócił z wyłączonym, zostaje ponownie włączony. Następnie ponownie uruchamia strumień oraz odbiornik komputera przeznaczony do jego odbioru.
- **Powtarza próby w coraz dłuższych odstępach czasu:** od razu, następnie po 5, 10, 20 i 40 sekundach, a potem co 60 sekund, dopóki czujnik pozostaje niedostępny. Każda próba polega jedynie na zapytaniu czujnika o jego tożsamość. Czujnik, który odpowiada, ale nadal nie wysyła żadnych danych po trzech próbach ponownego połączenia, jest pozostawiany w spokoju i zgłaszany (`auto-reconnect GAVE UP`). `pool-stream --start` podejmuje kolejną próbę.
- **`pool-list` pokazuje aktualny stan.** Gdy czujnik milczy, wiersz `STREAM FAULT` informuje o przebiegu procesu ponownego nawiązywania połączenia. Wiersz ten znika wraz z pojawieniem się pierwszego widma, a czujnik wyświetla wówczas `reconnected automatically N time(s)`.
- **Rejestracja jest kontynuowana w tym samym `.daq`.** Plik oznacza każdą przerwę dwoma wierszami: `event_type` 5 przy ostatnim widmie przed przerwą i 6 przy pierwszym po niej, z których każdy zawiera wyłącznie znacznik czasu. `pool-record --duration` wyświetla `!! STREAM LOST …`, gdy czujnik jest nieobecny, a `stream resumed after …`, gdy powraca. `pool-record --stop` wymienia każdą przerwę i wyświetla ostrzeżenie, tak jak w przypadku pusty plik (zapis, który nie zawiera żadnych odczytów, kończy się na `1`).
- **Zapis zachowuje jedną kalibrację.** Jeśli czujnik powróci z inną kalibracją podczas trwania zapisu, pozostała część zapisu zachowuje kalibrację z początku, a ostrzeżenie o zatrzymaniu informuje o tym. `pool-disconnect` + `pool-connect` przyjmuje nową kalibrację.
- **Twoje polecenia mają pierwszeństwo.** Czujnik, który wyłączyłeś za pomocą `pool-disconnect` lub wstrzymałeś za pomocą `pool-stream --stop`, nigdy nie zostanie ponownie podłączony, nawet jeśli proces ponownego podłączania już trwa.
- **`pool-info` na nieczynnym czujniku** odczytuje czujnik przy nowym połączeniu i dodaje `!! stream NOT reaching this computer`. Czujnik, który nie odpowiada, jest natychmiast zgłaszany. Polecenie `pool-list` nie wyświetla już stanu PTP dla czujnika, który przestał odpowiadać.
- Urządzenia DAQ-U i DAQ-M nie są objęte tą zmianą. Urządzenie DAQ-M samodzielnie ponownie łączy się przez Bluetooth, a urządzenie DAQ-U odłączone od USB wymaga ponownego użycia `pool-connect`.

### DAQ-E-S — stały dyfuzor i czujnik pochylenia

**DAQ-E-S** to urządzenie typu DAQ-E z dwiema różnicami istotnymi z punktu widzenia oprogramowania. Wszystko inne — wykrywanie, łączenie, PTP, rejestrowanie oraz poniższe ustawienia urządzenia — działa dokładnie tak samo jak w przypadku DAQ-E.**Jego dyfuzor jest stały, więc nie wymaga korekcji nasadki.** Dyfuzor typu Sunshine jest wbudowany i znajdował się na urządzeniu podczas pomiaru kalibracji fabrycznej, dzięki czemu kalibracja już odczytuje rzeczywiste natężenie promieniowania padającego w dół. Chloros ustawia nasadkę na `as_recorded` (bez profilu) niezależnie od tego, o co zostanie poproszony: limit o nazwie odpowiadającej temu urządzeniu (`sunshine_cosine`, `fov_*`, `none`) jest ignorowany wraz z wyświetleniem komunikatu, a jego zapisy przechowują wartość `cap_id_source=model`, więc ostrzeżenie o domniemanym ograniczeniu nigdy się w tym przypadku nie uruchamia. Nie należy również ręcznie nakładać ograniczenia na jego dane: `sunshine_cosine` na górze wskazywałby wartość około 11 razy wyższą.

**Urządzenie jest wyposażone w 3-osiowy akcelerometr.** Charakterystyka cosinusowa dyfuzora światła słonecznego jest ważna tylko wtedy, gdy jest on wypoziomowany, więc przechylony czujnik zgłasza błędne natężenie promieniowania — a współczynnik odbicia dziedziczy ten błąd — bez żadnych innych objawów. Każdy odczyt z DAQ-E-S zawiera zatem informacje o jego położeniu:

| Pole | Znaczenie |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Kąty w stopniach. `None`, gdy oprogramowanie sprzętowe oznacza kąt jako nieokreślony (swobodny spadek, martwa oś). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Przyspieszenie w mili-g. Wartość `mag_mg` znacznie odbiegająca od ~1000 oznacza, że urządzenie było w ruchu, a kąty dla tej próbki nie mają znaczenia. |
| `cal_applied` | Ustawione: osie i kąty są w układzie odniesienia dyfuzora (zastosowano kalibrację montażową urządzenia). Wyczyść: surowy układ odniesienia płytki, który może wykazywać nachylenie do ~2,3°, które nie jest rzeczywiste. |
| `sample_age_ms` | Wiek próbki z akcelerometru w momencie wysłania widma. |
| `temp_raw` | Nieskalibrowany **trend** temperatury matrycy — wyłącznie różnice, nigdy odczyt termometru. |

`daq pool-latest --json` zwraca je pod kluczem `imu` każdej ramki (`null` w przypadku DAQ-U, DAQ-M lub zwykłym DAQ-E). Plik `.daq` zarejestrowany z DAQ-E-S przechowuje te same wartości w kolumnach `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) obok każdego widma, dzięki czemu czytelnik może odróżnić chmurę od przewróconego statywu. W przypadku wszystkich pozostałych modeli kolumny te są puste.

Strumień danych o położeniu jest raportowany osobno: `daq pool-list` wyświetla `!! ATTITUDE STREAM FAULT (spectra unaffected)`, gdy urządzenie jest wyłączone, oraz podaje liczbę usterek, z których się później zregenerowało. Usterka w tym miejscu nigdy nie powoduje luki w zapisie widmowym. `daq pool-discover` wykazuje urządzenie jako `DAQ-E-S` (jako `DAQ-E`, jeśli nie odpowie na sprawdzenie tożsamości w ciągu około 2 s).

### Ustawienia urządzenia DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` bez `--gateway` nie zapisuje bramy** (wszelkie wcześniej ustawione bramy są usuwane), co jest właściwe dla sieci kamer bez routera.
- **Adres statyczny musi znajdować się w sieci tego komputera.** Program Chloros odrzuca adresy, których podsieć nie obejmuje adresu tego komputera w sieci czujnika: po ponownym uruchomieniu nic nie będzie mogło połączyć się z czujnikiem w celu przywrócenia poprzednich ustawień. Ustawienie `--force` należy zastosować tylko wtedy, gdy ten komputer otrzyma nowy adres lub zostanie przekierowany do nowej podsieci.
- **Restart jest odrzucany, gdy czujnik prowadzi nagrywanie.** Najpierw należy zatrzymać nagrywanie (`pool-record --stop`). Nagrywanie rozpoczęte tuż przed restartem zostanie przez niego zamknięte, a komunikaty `pool-set-network`, `pool-reboot` i `pool-update-firmware` kończą się wierszem `Warning:` zawierającym nazwę pliku nagrania (nawet jeśli czujnik nie powróci).
- **Po ponownym uruchomieniu Chloros ponownie łączy się wyłącznie z samym czujnikiem.** Czeka w miejscu, w którym znajdował się czujnik — a po `--auto` również pod jego nazwą `daq-e-<id>.local` — na odpowiedź zawierającą nazwę czujnika, nazwy hosta i modelu, z adresu sieciowego (MAC), który posiadał, i ponownie łączy się z nim pod adresem, który udzielił odpowiedzi. Odpowiedź pod tą nazwą jest akceptowana wyłącznie z tym adresem MAC, więc z komputera, który łączy się z czujnikiem przez router, lub z WSL2 w domyślnej sieci NAT, czujnik, który DHCP przeniosło na nowy adres, nie zostanie ponownie podłączony: polecenie informuje o tym i wyświetla polecenie `pool-connect --eth-host <its address> …`, które przywraca czujnik z dotychczasowymi ustawieniami czułości i ekspozycji.
- **`pool-update-firmware` odrzuca czujnik, którego wersji oprogramowania nie może odczytać.** Czujnik, którego powrót po błysku trwa dłużej niż czas oczekiwania, nie uległ awarii: gdy tylko odpowie, zostaje ponownie podłączony z ustawieniami nowego oprogramowania układowego, a polecenie ostrzega, że działanie było opóźnione.
- **Aktualizacja oprogramowania układowego wymaga przychodzącego połączenia TCP.** Czujnik ponownie łączy się z tym komputerem w celu pobrania obrazu. Program Chloros nasłuchuje na karcie sieciowej, która kieruje ruch do czujnika (na każdej karcie, tylko wtedy, gdy nie może określić, która to jest) i obsługuje wyłącznie adres tego czujnika; każde inne połączenie jest zamykane bez obsługi. W systemie Windows reguła zapory `Chloros Backend (TCP-In)` instalatora zezwala na to połączenie (zobacz [Windows (.exe)](#windows-exe)); zapora sieciowa, która blokuje to połączenie, powoduje niepowodzenie aktualizacji z kodem błędu `… device never connected back on TCP <port> …`.
- **W środowisku WSL2 należy użyć sieci lustrzanej** (`networkingMode=mirrored` w `%UserProfile%\.wslconfig`). W domyślnym trybie NAT adres tego komputera w sieci czujnika jest adresem samego WSL, więc każdy adres statyczny jest odrzucany (z wyjątkiem `--force`), strumień na żywo nie dociera do WSL, a aktualizacja oprogramowania układowego nie może zostać zakończona, ponieważ czujnik łączy się z powrotem z komputerem w celu odebrania obrazu.
- **Identyfikator czujnika i nazwa hosta nie ulegają zmianie.** Niezależnie od adresu, czujnik DAQ-E nadal ma identyfikatory `daq-e-<id>` i `daq-e-<id>.local`, więc `--eth-host daq-e-<id>.local` wciąż go wykrywa.
- **Interfejs graficzny ma te same ustawienia**: zakładka „Czujniki światła”, ustawienia czujnika (ikona koła zębatego), „Sieć”. Interfejs graficzny utrzymuje własne połączenie z czujnikiem, więc zmiana czujnika, z którym interfejs ma otwarte połączenie, dokonywana z poziomu interfejsu graficznego: ponowne uruchomienie z poziomu CLI kończy sesję GUI z tym czujnikiem.
- Odpowiedniki w SDK to `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` oraz `update_firmware()`.

---

## `chloros-cli project`

Otwieranie, łączenie się i sterowanie zapisanym projektem Chloros (folder zawierający pliki `cameras.json` + `sensors.json` + `project.json`). Wszystko przechodzi przez backend, więc interfejs graficzny (GUI) i interfejs wiersza poleceń (CLI) generują identyczny stan sprzętu.

### Opis podpoleceń

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Wyświetla manifest urządzeń projektu (kamery, macierze, czujniki), odczytuje go z plików i sprawdza pliki źródłowe (zobacz [Przeniesiony projekt](#a-moved-project---relink)). Nie otwiera projektu w backendzie: projekt otwarty tam, wraz z urządzeniami i nagraniami, pozostaje bez zmian. Zamyka `1`, jeśli nie można znaleźć żadnego z plików źródłowych projektu. |
| `project devices PATH [--reconnect]` | Wyświetla listę lub ponownie uruchamia wykrywanie. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Łączy każdą zapisaną kamerę / matrycę / czujnik. Każda matryca łączy się tak samo jak w przypadku `lattice array-connect`: przez maksymalnie 600 s, a poszczególne etapy są wyświetlane na stderr (`[array <id>]    <s> s: <step>`; stdout to raport w formacie JSON), a macierz, której nie udało się podłączyć w tym czasie, zostaje cofnięta — jej kamery przywracane są do stanu, w jakim zostały wykryte — i zgłaszana w ramach `errors`, nigdy nie pozostaje podłączona. `--packed` ustawia dla każdej macierzy skompresowany 12-bitowy transport (domyślnie `auto`; projekt nie zapisuje ustawień dla poszczególnych macierzy; zobacz [Pakowany 12-bitowy transport](#packed-12-bit-transport---packed)). Każdy czujnik światła jest otwierany w puli DAQ backendu pod zapisanym adresem wraz z zapisanym czasem integracji, uśrednianiem klatek, automatyczną ekspozycją i ograniczeniem: czujnik bez wybranego ograniczenia otrzymuje `sunshine_cosine`, czujnik zapisany jako „bare” (`none`) pozostaje w stanie „bare”, a czujnik typu DAQ-E-S nigdy nie przyjmuje ograniczenia. Panel DAQ w GUI nie jest częścią puli: najpierw należy zamknąć czujnik w tym miejscu. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Pojedyncze przechwycenie z nazwanej kamery lub macierzy (domyślnie `--prefix`, `capture`). `--orient` (matryce): matryca zapisuje dane surowe, które pozostają w formacie natywnym — oznacza to, że produkty tworzone na ich podstawie przez `process` mają być zorientowane, tak jak `array-capture --orient`. Domyślnie: natywny. `project burst` / `stream` oraz elementy receptury `capture` / `burst` / `stream` (`orient: true`) mają tę samą opcję. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Seria N klatek z określonej kamery lub macierzy (`-n/--count` domyślnie 5; `-i/--interval` liczba sekund między klatkami, domyślnie 0). Seria z macierzy usuwa zduplikowane, powtarzające się zsynchronizowane grupy (mechanizm kontroli aktualności), dzięki czemu macierz działająca w trybie częściowego cyklu nie może zwrócić N kopii jednej klatki; wyświetla wyniki dla każdej iteracji. Nieudane przechwycenie w trakcie zatrzymuje serię i kończy działanie `1`, podając liczbę klatek zapisanych przed niepowodzeniem. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Przechwytywanie na dysk w pętli: `--fps F` przechwytuje co sekundę (domyślnie 3), jedną klatkę na przechwycenie z kamery, jedną zsynchronizowaną grupę klatek z tablicy (z tym samym mechanizmem kontroli aktualności co `project burst`), aż `-n N` przechwyci (domyślnie `0` = do momentu naciśnięcia Ctrl+C, co powoduje wyświetlenie liczby zapisanych klatek i zakończenie działania `0`, lub `1`, gdy żadna klatka nie została zapisana lub wystąpił błąd pliku; naciśnięcie Ctrl+C przed przechwyceniem przez `-n N` powoduje zamknięcie programu `130`). Nieudane przechwycenie w trakcie zatrzymuje strumień i powoduje zakończenie działania funkcji `1`, podając liczbę klatek zapisanych przed niepowodzeniem. Funkcja `--poll-interval` jest przestarzała i ignorowana: jest nadal akceptowany, więc starsze skrypty działają, z adnotacją w stderr. |
| `project sensor read PATH NAME [--json]` | Najnowsza ramka widma. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Nagrywaj plik .daq dla `--seconds` (domyślnie 10). Nagranie niezawierające żadnych odczytów jest zgłaszane na stderr i powoduje zakończenie działania `1`. |
| `project run PATH RECIPE.yaml` | Wykonaj recepturę przechwytywania YAML/JSON. `--dry-run` przeprowadza walidację bez uruchamiania. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Oblicz wyrównanie dla tablicy — zobacz [poniższą tabelę flag](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Wyświetla bieżący profil wyrównania. |
| `project align clear PATH NAME` | Usuwa profil z pamięci podręcznej. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Przesuwa transformację jednego serwera podrzędnego (domyślnie `0`, `0`, `0`, `1.0`: bez zmian). |
| `project align export PATH NAME --to FILE` | Zapisz profil w formacie JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Załaduj zapisany profil. |

#### Opcje `project align calibrate`

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda wyrównywania. **Te nazwy różnią się od `lattice align-calibrate`**, które akceptuje skrócone formy `orb` / `akaze` / `phase`; te dwa polecenia nie są zamienne w przypadku tej flagi. |
| `--model {translation, rigid, affine, homography}` | `affine` | Przekształć model, aby dopasować. |
| `--frames N` | `1` | Zsynchronizowane migawki klatek do średnią. |
| `--reference SN` | kamery głównej | Numer seryjny kamery referencyjnej; wszystkie pozostałe elementy są do niej dopasowywane. |
| `--max-features N` | `5000` | Ograniczenie liczby cech ORB. |
| `--ratio-threshold F` | `0.75` | Test współczynnika Lowe’atest współczynnika Lowe’a. |
| `--ransac-threshold-px F` | `3.0` | Próg elementów wewnętrznych RANSAC. |
| `--min-matches N` | `15` | **Próg jakości** — odrzucenie rozwiązania poniżej tej liczby dopasowań punktów wewnątrz przedziału. |
| `--max-reproj-err-px F` | `4.0` | **Próg jakości** — odrzucenie rozwiązania powyżej tego błędu reprojekcji RMS. |
| `--checkerboard RxC` | — | Geometria płytki dla `--method checkerboard`, np. `9x6`. |
| `--name PROFILE` | puste | Nazwa profilu osadzona w zapisanym pliku JSON. **Nie nazwa tablicy** — jest to pozycyjny `NAME`. |

Te dwa kryteria jakości są powodem, dla którego kalibracja może zakończyć się sukcesem pod względem rozwiązania, a mimo to
zostać odrzucona przy zapisywaniu: profil, który nie spełnia któregokolwiek z nich, powodowałby ciche błędne zarejestrowanie każdego
późniejszego ujęcia, dlatego jest odrzucany, a nie zapisywany.

### Przeniesiony projekt (`--relink`)

Projekt `project.json` nazywa każdy plik źródłowy — każdy obraz i zapis z czujnika światła
— według jego pełnej ścieżki. Po przeniesieniu folderu projektu lub całego folderu projektów
na inny dysk ścieżki te nadal wskazują starą lokalizację.

Otwarcie projektu naprawia to, co może samodzielnie: plik, który znajdował się w folderze projektu,
jest wyszukiwany w folderze, w którym projekt się obecnie znajduje, a każda znaleziona tam ścieżka
jest przekierowywana i zapisywana (`project.json` jest najpierw archiwizowany jako
`project.json.before-relink-<date>-<time>`). Dzieje się to przy każdym otwarciu — aplikacji,
`chloros-cli project open`, `chloros-cli process` do istniejącego projektu, do folderów SDK oraz `chloros-mcp`. Ścieżka jest przekierowywana wyłącznie do pliku, który się tam znajduje.

Pliki, które znajdowały się poza folderem projektu — sesja hubu, import za pomocą opcji „Dodaj folder” —
nie mogą zostać w ten sposób znalezione. Należy podać nazwę folderu, w którym się obecnie znajdują:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` obejmuje folder, do którego trafiły pliki, lub dowolny folder nadrzędny: każdy brakujący
plik jest wyszukiwany w tym folderze na podstawie końcowych nazw folderów
(`Hub Projects/<session>/raw/<serial>`, następnie `<session>/raw/<serial>`, ...), a pliki z każdego
plików z danego folderu są pobierane z tego miejsca, w którym znaleziono ich najwięcej. `project open` wyświetla
informację o tym, co zostało ponownie przypisane, ile plików źródłowych znaleziono oraz w jakich folderach znajdowały się pozostałe
pliki. Niektóre nadal brakują: ostrzeżenie, zakończenie działania `0` — projekt otwiera się wraz z pozostałymi. Nie
znaleziono żadnych: błąd z podaniem nazw tych folderów oraz `--relink`, zakończenie działania `1`; moduł zaplecza również
odmawia otwarcia takiego projektu dla `process` lub `project devices --reconnect` (kod HTTP 409 `PROJECT_SOURCES_MISSING`).

### Przykłady

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Język DSL receptur

`project run RECIPE.yaml` akceptuje plik w formacie YAML lub JSON, opisujący sekwencję działań:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Obsługiwane akcje: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` pobiera klucze `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (starsze nazwy `exposure_time`, `exposure_auto`, `gain`, `gain_auto` są akceptowane jako aliasy). Ustawiona w ten sposób ekspozycja, wzmocnienie lub wartość docelowa są traktowane jako Twoje: automatyczna kontrola ekspozycji w Chloros (w tym ekspozycja mierzona przez DAQ) nie ingeruje w nie. Ręczna ekspozycja pozostawia wzmocnienie bez zmian: w trybie automatycznym aparat dostosowuje wzmocnienie (w całym zakresie) w kierunku docelowej jasności, dopóki nie ustawisz `gain_db` lub `auto_gain: "Off"`. Późniejsze użycie `auto_exposure: "Continuous"` przywraca ekspozycję do trybu automatycznego, a `target_brightness_auto: true` przywraca utrzymywaną wartość docelową do automatycznego wyboru Chlorosa. Akcja `burst` przyjmuje parametry `name` (wymagane), `count` (domyślnie 5), `interval` (w sekundach, domyślnie 0), `output`, `format`, oraz `settings` (ten sam kształt ustawień dla poszczególnych kamer co `apply`); serie zdjęć z tablicy wykorzystują ten sam zegar kontrolny świeżo zsynchronizowanej grupy co `project burst`.

W przypadku `capture`, `burst` lub `stream`, wartość domyślna dla `output` to `output`. Względna ścieżka `output` — domyślnie zawarty — jest rozpoznawany względem folderu, z którego uruchamiasz `chloros-cli project run`, a nie folderu pliku receptury ani folderu projektu (w wersji 1.2.0 znalazł się on w folderze instalacyjnym backendu).

Uruchom go:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Każde uruchomienie jest najpierw sprawdzane względem projektu.**Każda akcja musi mieć jako klucz dokładnie jeden z powyższych czasowników — `- capture: {name: FrontLeft}`, a nie `- action: capture` ani `- type: capture` — w przypadku `name` projekt zawiera `cameras.json` / `sensors.json`, kamerę lub tablicę dla `apply` / `capture` / `burst` / `stream` oraz czujnik światła dla `sensor`; model `stream` wymaga, aby wartość `count` była większa od 0, a model `wait` wymaga określenia czasu trwania. Przepis zawierający którykolwiek z tych błędów**nie** wykonuje żadnej akcji (`[recipe] error: …`, `[recipe] not run: …`, wyjście `1`), nawet przy `stop_on_error: false`. Klucze, których dany krok nie używa, to `[recipe] warning:`, podobnie jak każda tablica lub czujnik światła sterowany przez recepturę, który nie jest obecnie podłączony. Przepis nigdy nie podłącza urządzeń: najpierw uruchom `chloros-cli project connect PATH`. `--dry-run` wyświetla przepis, a następnie te same błędy i ostrzeżenia, a `[recipe] valid: …` / `NOT valid: …`, nie uruchamia niczego i kończy działanie `1`, gdy w przepisie występuje błąd.

---

## Zmienne środowiskowe

| Zmienna | Efekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Zastępuje URL zaplecza (domyślnie `http://127.0.0.1:5000`) — **akceptowane tylko przez rodziny poleceń `lattice`, `project` i `daq pool-*`** (`lattice network-analysis` przyjmuje zamiast tego `--backend-url`). Polecenia podstawowe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ustawiają wartość `http://127.0.0.1:<port>` i ignorują tę zmienną (literał IPv4 pozwala ominąć karę w systemie Windows, wynoszącą około 2 sekundy na żądanie, związaną z `localhost`→`::1`), dzięki czemu zawsze kierują się do lokalnego komputera. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Ustawiane w środowisku backendu. `1` łagodzi odmowę spowodowaną nadmierną subskrypcją — połączenie tablicowe z `pin_resolution` lub [przejściowe polecenie wieloaparatowe](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — do głośnego ostrzeżenia, akceptując utratę pakietów GVSP. Wyłącznie do celów testowych — zobacz [Model fps i burst dla tablic](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Ustawia się w środowisku backendu: `on` lub `off` ustawia domyślnie [12-bitowy transport pakietowy](#packed-12-bit-transport---packed) dla każdej tablicy (domyślnie `auto`). Wyraźne ustawienie `--packed` nadal ma pierwszeństwo. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (lub `true` / `yes`) wyłącza [automatyczne sprawdzanie aktualizacji](#chloros-cli-update) w Linux CLI podczas uruchamiania. |
| `CHLOROS_CLI_MODE` | Ustawiane przez sam interfejs CLI; nakazuje backendowi włączenie przetwarzania równoległego. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` pomija sondę awaryjną GVSP (tylko wyniki ICMP). **Wyłącza to obsługę pakietów jumbo, a nie tylko ogranicza liczbę wpisów w dzienniku** — kamera odpowiada na pingi DF tylko do 1500 na każdej ścieżce, więc ten test jest jedyną metodą wykrywania pakietów jumbo. Oszczędza ~1 s na kamerę na każde połączenie; kosztuje ~1,45× maksymalnej przepustowości łącza, jeśli sieć *mogłaby* obsługiwać pakiety jumbo. SDK wyświetla ostrzeżenie po ustawieniu tej opcji. |
| `CHLOROS_GVSP_SESSION_REUSE` | Ustawienie w środowisku backendu: `0` mierzy rozmiar pakietu przy każdym połączeniu zamiast raz na kamerę na sesję backendu. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Ustal rozmiar pakietu GVSP na N bajtów; całkowicie pomija sondowanie. Zaleca się stosowanie opcji na polecenie (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) zamiast ustawiania na stałe: ustalony rozmiar nie dostosowuje się już do sieci przed nim, a ustalenie wartości 9000 na ścieżce, która nie obsługuje pakietów jumbo, powoduje, że **każde** przechwytywanie kończy się przekroczeniem limitu czasu przy `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Ustawia się w środowisku zaplecza: maksymalna liczba sekund, przez jaką trwa [polecenie przejściowe dla wielu kamer](#multi-camera-transient-sessions) lub `lattice capture` kamery, której nic nie utrzymuje, może poświęcić na ustawienie ekspozycji każdej kamery na podstawie poprzedniej sceny, zanim zachowa klatkę (domyślnie `5`). `0` wyłącza tę funkcję: automatyczna ekspozycja każdej kamery działa wówczas od pierwszej klatki, zaczynając od jej limitu ekspozycji. |
| `CHLOROS_LOG_DIR` | Ustawiane w środowisku backendu: folder, do którego trafia szczegółowy dziennik. Domyślnie `%LOCALAPPDATA%\Chloros\logs` w systemie Windows, `~/.cache/chloros/logs` w systemie Linux (`$XDG_CACHE_HOME/chloros/logs`, gdy jest to ustawione). Usługa główna pakietu .deb ustawia `/var/log/chloros`, do którego dostęp ma grupa `adm` — zobacz [Uruchamianie backendu jako usługi](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (system Linux) | Miejsce przechowywania plików tymczasowych. Interfejs CLI automatycznie wykorzystuje `/mnt/ssd/tmp`, jeśli istnieje. |

---

## Kody wyjścia

| Kod | Znaczenie |
| --- | --- |
| `0` | Pomyślne zakończenie. |
| `1` | Ogólny błąd (większość błędów podpoleceń). Polecenie przechwytywania kończy również działanie z kodem `1`, gdy nie udało się zapisać choćby jednego z plików, o których zapisanie poproszono, nawet jeśli pozostałe zostały zapisane, oraz z kodem `array-burst` / `array-record` kończy działanie, gdy rejestrator zgłasza błąd — zobacz [Pliki, których nie udało się zapisać](#files-that-fail-to-write). `process` kończy również działanie `1`, gdy nie można zweryfikować eksportów danego przebiegu lub gdy utracono obraz, mimo że pozostałe produkty zostały zapisane. |
| `2` | Błąd argumentu (nieznana lub brakująca flaga; dotyczy również `time-sync set-priority` bez żadnego priorytetu). |
| `130` | Przerwane przez Ctrl+C: każde polecenie, które zostało przerwane — `lattice`, `project`, `daq` oraz `process` (`process` po zatrzymaniu działania przez backend, patrz [Uwagi dotyczące `process`](#notes)). Polecenie, którego normalnym sygnałem zatrzymania jest Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` oraz `project stream` z `-n 0` — kończy działanie w zależności od tego, co wykonał: `0` po zapisaniu wszystkich danych, `1` w przypadku niepowodzenia jakiejkolwiek operacji. |
| `143` / `129` | Zakończony przez sygnał SIGTERM / SIGHUP (128 + sygnał), po zakończeniu operacji sprzątania polecenia: uruchomione zadanie zostało zatrzymane, a kamery przywrócone do stanu początkowego; przetwarzanie w tle zostało zatrzymane. `lattice continuous` lub inne nagranie, które normalnie zatrzymuje się za pomocą Ctrl+C, również kończy działanie `143`, a nie `0`. |

`lattice network-analysis` kończy działanie `1` w przypadku werdyktów `needs_force_slip` i `error`, a `selftest` kończy działanie `1`, chyba że wszystkie siedem testów zakończy się powodzeniem.

---

## Wskazówki dotyczące rozwiązywania problemów

- **„Wymagane logowanie”** → Uruchom program `chloros-cli login EMAIL PASSWORD` raz na tym komputerze.
- **„Folder projektu jest zbyt głęboko osadzony na tym komputerze”** → Na tym komputerze wyłączono obsługę długich ścieżek w systemie Windows (ustawienie domyślne systemu Windows), więc żadna ścieżka nie może przekraczać 260 znaków, a produkty znajdują się maksymalnie około 130 znaków poniżej folderu projektu. Nie przetworzono żadnych danych. Komunikat podaje najdłuższy folder projektu, który się mieści: użyj krótszego (`-o` lub `chloros-cli set-project-folder`) albo włącz obsługę długich ścieżek w systemie Windows i uruchom ponownie program Chloros. Instalator Windowsa oferuje tę opcję jako „Włącz długie ścieżki w systemie Windows”, zaznaczoną domyślnie; w przeciwnym razie należy skorzystać z zasad grupy: *Konfiguracja komputera > Szablony administracyjne > System > System plików > Włącz długie ścieżki Win32*.
- **„brak dostępu do serwera”** → Uruchom aplikację Chloros na komputerze (Windows) lub uruchom serwer samodzielnie: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` w systemie Windows (folder `backend` w folderze instalacyjnym; nie znajduje się w `PATH`), `chloros-backend` w systemie Linux. Lub sprawdź `CHLOROS_BACKEND_URL`, jeśli korzystasz z połączenia zdalnego.
- **Polecenia `lattice` kończą się niepowodzeniem z komunikatem „Nie znaleziono sterowników kamer LATTICE”** → Backend nie mógł załadować środowiska uruchomieniowego kamer LATTICE, z którym został zainstalowany. Każde polecenie `lattice` jest wykonywane w backendzie; sam interfejs CLI nie zawiera żadnych sterowników kamer. Zainstaluj ponownie Chloros: instalator dla systemu Windows oraz pakiet .deb instalują środowisko uruchomieniowe obok backendu.
- **W oknie „Array connect” / „Array Settings” pojawia się komunikat „FRAMES WILL DROP” lub „Reduce ROI to enable”** → Pierścień odbiorczy karty sieciowej hosta jest zbyt mały (zwykle po aktualizacji sterownika karty sieciowej należy go zresetować do wartości 32). Zobacz [Konfiguracja i optymalizacja karty sieciowej hosta](#host-nic-setup--tuning-lattice-arrays) — ustaw `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Komputer zawiesza się podczas ponownego uruchamiania/wyłączania, a następnie pojawia się błąd WMI `Invalid class` / karta sieciowa nie włącza się / brakuje napędów USB** → Przestarzały sterownik karty USB 10 GbE powodujący błąd `DRIVER_POWER_STATE_FAILURE` (niebieski ekran śmierci `0x9F`). Zaktualizuj sterownik karty — zobacz [Konfiguracja i optymalizacja karty sieciowej hosta](#host-nic-setup--tuning-lattice-arrays).
- **System Linux: zapisy należą do użytkownika root, więc nie można ich usunąć, a dziennik backendu nie znajduje się w katalogu użytkownika `~/.cache`** → Backend działa jako root poprzez `chloros-backend.service`. Przełącz się na katalog `chloros-backend@<user>.service`, w którym backend działa pod Twoim kontem — zobacz [Uruchamianie backendu jako usługi](#running-the-backend-as-a-service-headless). Do tego momentu dziennik usługi uruchomionej jako root znajduje się w `/var/log/chloros` (dostępny do odczytu dla grupy `adm`).
- **System Linux: jednostka zaplecza to `failed` z komunikatem „Kolejna instancja już działa” (kod zakończenia 98)** → Inna jednostka zaplecza już zajmuje ten port: w danym momencie działa tylko jedna. Jeśli włączone są dwie jednostki backendowe, zachowaj jedną: `sudo systemctl disable --now <unit>` dla tej, której nie używasz — zobacz [Uruchamianie backendu jako usługi](#running-the-backend-as-a-service-headless). Jeśli włączona jest tylko ta jednostka, port jest zajęty przez backend uruchomiony poza systemd (przez `chloros-backend`, `chloros-cli` lub skrypt SDK): zatrzymaj ten backend, a następnie uruchom `sudo systemctl start <unit>`.
- **Ostrzeżenie dotyczące pamięci wymiany w Jetson** → Dodaj pamięć wymiany opartą na pliku; interfejs CLI wyświetla dokładne polecenia `fallocate` / `swapon`.
- **Komunikat „backend not reachable” po wydaniu polecenia `lattice`, `project` lub `daq`** → żaden backend nie działa: te polecenia nigdy go nie uruchamiają. Uruchom polecenie `chloros-cli backend status` i zapoznaj się z sekcją [Uruchomienie serwisu backend](#start-the-backend).
- **„`daq net ...` nie jest dostępny w Chloros”** (Linux i Jetson) → Niektóre ustawienia urządzenia DAQ-E są zarezerwowane dla usługi MAPIR. Ustawienia sieciowe, ponowne uruchomienia i aktualizacje oprogramowania układowego obsługują polecenia `pool-set-network`, `pool-reboot` oraz `pool-update-firmware`; w pozostałych przypadkach skontaktuj się z pomocą techniczną MAPIR.
- **„Bezpośrednie polecenia sprzętowe DAQ wymagają pełnego pakietu daq”** (system Windows) → Należy użyć `pool-*`, który steruje tym samym czujnikiem za pośrednictwem zaplecza, lub `chloros_sdk.connect_daq_sensor()` z języka Python.

---

## Zobacz też

- [Dokumentacja Python SDK](sdk-reference.md) — programowy odpowiednik każdego polecenia CLI.
- [Czujniki światła DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) w instrukcji obsługi Chloros — czujniki, ich ograniczenia i skalibrowany zakres, rejestracja, sieć DAQ-E.
- [Kamery LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) w instrukcji obsługi Chloros — podłączanie, ustawienia kamer, tryby przechwytywania, macierze.
- Dokumentacja online: `https://mapir.gitbook.io/chloros/cli`
