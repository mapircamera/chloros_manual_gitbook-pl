# CLI : Wiersz poleceń

> **Pełna dokumentacja:**[CLI Reference](reference/cli-reference.md) opisuje**każdy parametr każdego podpolecenia** i jest zoptymalizowana pod kątem asystentów AI — wklej jej URL do swojego asystenta i poproś o działające polecenie: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Wskazówka dotycząca narzędzi AI:** każda strona niniejszego podręcznika jest dostępna w formacie surowego Markdown poprzez dodanie `.md` do jej adresu URL (np. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indeksuje cały podręcznik do wykorzystania przez model LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Czym jestCLI


`chloros-cli` to interfejs wiersza poleceń dla tego samego silnika przetwarzania, z którego korzysta aplikacja desktopowaChloros
. Jest to lekki klient typu „HTTP
” oparty na backendzieChloros
(lokalnym serwerze na `127.0.0.1:5000`) — większość poleceń uruchamia backend automatycznie, więc skrypt potrzebuje jedynie jednego wywołania `chloros-cli process …`.

Działa na **systemachWindows
10/11 (x64)**oraz**Linux
(x86_64 oraz NVIDIA Jetson arm64 na JetPack 6)**, w dowolnym terminalu, bez konieczności korzystania z GUI. Sprawdź poprawność instalacji za pomocą:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Przegląd grup poleceń:

* **Przetwarzanie i konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 języków — zobacz [Obsługiwane języki](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (tylkoLinux
/ Jetson)
* **Sprzęt działający na żywo** — `lattice` (sterowanie kamerą LATTICE, ponad 45 podpoleczeń), `daq pool-*` (czujniki światła DAQ), `time-sync` (PTP)
* **Automatyzacja** — `project` (sterowanie zapisanym projektemChloros
w trybie bezinterfejsu graficznego, w tym recepturami przechwytywania w formacie YAML)

Warto znać następujące opcje globalne: `--port N` (port zaplecza, domyślnie `5000`), `-v/--verbose`, `--restart` (wymuś ponowne uruchomienie backendu), `--backend-exe PATH`. Pełną listę znajdziesz w [CLI
Reference](reference/cli-reference.md).

***

## Instalacja

CLI
**jest dołączony do instalatoraChloros** na każdej platformie — nie ma osobnego pliku do pobraniaCLI
. Pobierz instalator ze strony [Pobierz](download.md).

###Windows


Instalator umieszcza plikCLI
w lokalizacji:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

i dodaje ten folder do systemu `PATH` — **po zakończeniu instalacji otwórz nowy terminal**, aby zaktualizowany plik `PATH` został pobrany. Instalator umieszcza również skrypty uruchamiające (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) w katalogu głównym instalacji oraz**skrót w menu StartChlorosCLI
** skrót w menu Start, z których każdy otwiera terminal z gotowym do użycia `chloros-cli`.

###Linux


Zainstaluj `.deb` dla swojej architektury:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Spowoduje to instalację `chloros-cli` do `/usr/bin/chloros-cli` (już na wersji `PATH`), a moduł zaplecza do wersji `/usr/lib/chloros/chloros-backend`, wraz ze środowiskiem uruchomieniowym ArenaSDK
wymaganym dla kamer LATTICE. Szczegółowe informacje można znaleźć w sekcji [InstalacjaLinux
](linux/linux-installation.md).

### Weryfikacja

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Logowanie i licencjonowanie

CLI
(orazPython
SDK
) dostęp wymaga **płatnego planuChloros
+**— każdy płatny plan go obejmuje; plan bezpłatny nie. Ograniczenie to jest egzekwowane**po stronie serwera** przez backend, a nie przez plik binarnyCLI
: połączenie przy wylogowanym użytkowniku jest odrzucane z kodem `401 AUTH_REQUIRED`, a wywołanie zalogowanego użytkownika w planie darmowym – kodem `403 PLAN_UPGRADE_REQUIRED`, niezależnie od tego, czy pochodzi ono z `chloros-cli`,SDK
czy z własnoręcznie stworzonego klientaHTTP
. Aktualizacja dostępna pod adresem [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Zaloguj się **raz na każdym komputerze**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Hasła zawierające znaki specjalne**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` jest zniekształcane przez powłokę (CLI
wykrywa to na błędzie 401 i automatycznie ponawia próbę, ale cudzysłowy pojedyncze całkowicie pozwalają uniknąć tego problemu).
{% endhint %}

Sesja jest buforowana w `~/.chloros/user_session.json` i działa w trybie offline przez okres karencji planu (30 dni w przypadku planów miesięcznych, do momentu wygaśnięcia w przypadku planów rocznych). `chloros-cli status` działa nawet bez płatnego planu, więc przyczyna odmowy jest zawsze widoczna.

{% hint style="danger" %}
**Planujesz zadania bezinterfejsowe? Zaloguj się najpierw.**Polecenia uruchamiające procesy w tle (`process`, `status`, `export-status`, …) uruchomione**bez sesji w pamięci podręcznej**nie kończą się natychmiastowym błędem — przechodzą w tryb interaktywny z wierszem poleceń `Email:` / `Password:` na stdin. W związku z tym zadanie cron lub etap CI uruchomione w trybie nienadzorowanym**zawieszą się, czekając na dane wejściowe**. Przed zaplanowaniem czegokolwiek należy raz uruchomić polecenie `chloros-cli login EMAIL 'PASSWORD'` na danym komputerze.
{% endhint %}

***

## Twoje pierwsze uruchomienie przetwarzania

Skieruj `process` na folder z danymi przechwyconymi — program automatycznie wykryjeSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` lub ich kombinację:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Postęp jest wyświetlany na żywo dla każdego wątku potoku (wykrywanie, analiza, przetwarzanie, eksport), a pomyślne zakończenie operacji następuje po zgłoszeniu liczby zapisanych produktów obrazowych (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Gdzie trafiają wyniki

`process` zapisuje dane w **folderze projektu**, a nie w folderze wejściowym:

* Bez `-o`: projekt jest tworzony w domyślnym folderze projektów (współdzielonym z interfejsem graficznym; zarządzaj nim za pomocą `get-project-folder` / `set-project-folder`, opcja awaryjna `~/Chloros Projects`), a jego nazwa pochodzi od `-n/--project-name` lub znacznika czasu (`YYYYMMDD_HHMMSS`), jeśli nie zostanie podana.
* W przypadku `-o PATH`: ten folder **jest** folderem projektu. Jeśli zawiera już plik `project.json`, zamiast nadpisywania tworzony jest plik o tym samym nazwie z przyrostkiem `_1`/`_2`…

W ramach projektu produkty są pogrupowane **według aparatu, a następnie według formatu pliku**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folder aparatu to `LATT-<sensor>-<lens>-F<filter>` dla LATTICE (zgodny z EXIF ujęcia `Model`) oraz `<model>_<filter>` (np. `Survey3N_RGN`) w przypadku modeluSurvey3
. Folder formatu ma nazwy zgodne z `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32` dla `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Każdy wyeksportowany produkt zachowuje nazwę pliku ŹRÓDŁOWEGO.**Eksport danych promieniowania z pliku `capture_..._raw.tif` nadal nosi nazwę `capture_..._raw.tif` — po prostu znajduje się w katalogu `tiff32/Radiance_Images/`.**To folder identyfikuje produkt, a nie nazwa pliku**, więc należy używać wyrażenia wzorcowego dla katalogu, a nie dla rozszerzenia `*radiance*`.
{% endhint %}

### Opcje, z których faktycznie będziesz korzystać

| Flaga | Domyślnie | Działanie |
| --- | --- | --- |
| `-o, --output PATH` | domyślny folder projektu | Lokalizacja folderu projektu (patrz wyżej). |
| `-n, --project-name NAME` | sygnatura czasowa | Nazwa projektu. |
| `--format FMT` | `TIFF (16-bit)` | Jedna z wartości: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | brak | Wskaźniki roślinności do wyeksportowania (zobacz [Wskaźniki roślinności](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronalna eliminacja efektu bayera, wolniejsza, najwyższa jakość (systemChloros
lub nowszy, procesor graficzny NVIDIA). |
| `--vignette / --no-vignette` | włączone | Korekcja winietowania. |
| `--reflectance / --no-reflectance` | włączone | Kalibracja odbicia; w przypadku plików LATTICE jest to również przełącznik produktu odbicia. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Wymuś punkt wejścia potoku dla plików TIFF LATTICE. |

W przypadku wszystkich pozostałych opcji — dostrajania wykrywania celów, PPK, pinów ekspozycji, flag wyrównania tablic — zapoznaj się z [sekcją `process` w podręcznikuCLI
](reference/cli-reference.md).

***

## Wybór elementów do eksportu (produkty LATTICE)

Przetwarzanie LATTICE rozgałęzia się na **każdy odpowiedni produkt w jednym przebiegu**. Cztery przełączniki dla poszczególnych produktów są**domyślnie włączone**; użyj formularza `--no-`, aby wyłączyć jeden z nich:

| Przełącznik | Produkt |
| --- | --- |
| `--debayered` | Liniowe usuwanie mozaiki → `Debayered_Images/` |
| `--preview` | Podgląd na ekranie (balans bieli + gamma; rozciągnięcie w fałszywych kolorach dla obrazów wielospektralnych) → `Preview_Images/` |
| `--radiance` | promieniowanie typu float32, W/m²/sr/nm → `Radiance_Images/` (zawsze `tiff32/`) |
| `--reflectance` | uint16 współczynnik odbicia, gotowy do użycia w Pix4D → `Reflectance_Calibrated_Images/` |

RGB
kamery główne zawsze generują jedynie dane po usunięciu efektu Bayesa oraz podgląd — promieniowanie/odbicie w poszczególnych pasmach nie ma znaczenia w przypadku czujnika szerokopasmowego, więc te przełączniki nie mają dla nich żadnego działania.Survey3
`.raw` ignoruje te przełączniki i stosuje standardową ścieżkę odbicia/docelową.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (domyślnie `auto`) wybiera odniesienie do współczynnika odbicia: `auto` tworzy [cel kalibracyjny](calibration-targets.md) w kadrze, który przechodzi kontrolę jakości jako odniesienie bezwzględne i w przypadku braku celu powraca do podziału promieniowania padającego czujnika światła DAQ (ρ = π·L/E); `target` działa ściśle (bez substytucji DAQ); `daq` opiera się na danych z DAQ. Skany celów zmierzone w jednostkach miary mogą być dostarczane za pomocą `--target-reflectance-dir`.

{% hint style="info" %}
**Odczyt pikseli odbicia:**wartość DN oznaczająca ρ = 1,0 odnosi się**do źródła** — Pliki LATTICE umieszczają znacznik `Chloros:PixelScale=32768` w metadanych XMP; plikiSurvey3
używają wartości 65535 (i nie zawierają znaczników `Chloros:*`). Należy odczytać ten tag i podzielić przez jego wartość, zamiast zakładać stałą wartość. Szczegóły oraz jeden celowo wprowadzony skrajny przypadek bez skalowania znajdują się w [CLI
Dokumentacji](reference/cli-reference.md).
{% endhint %}

**Przetwarzanie zawsze rozpoczyna się od `raw`.** Produkty pochodne (eksporty danych po usunięciu efektu debayeringu, promieniowania lub odbicia) nigdy nie są ponownie wprowadzane do potoku — ponowne ich zaimportowanie i przetworzenie spowodowałoby podwójne zastosowanie obliczeń kalibracyjnych, dlategoChloros
pomija je i informuje o tym. `--input-level` stanowi celowo przewidziane wyjście awaryjne na wypadek, gdy rzeczywiście konieczne jest wymuszenie punktu wejścia.

***

## Gdy przetwarzanie zakończy się niepowodzeniem

Od wersji 1.2.0 `process` wyraźnie zgłasza błąd zamiast „zakończyć się powodzeniem” bez wyświetlania żadnych wyników:

* Uruchomienie, które **zażądało produktów, ale nie zapisało żadnego**— tylko `project.json` i `calibration_data.json` — wyświetla komunikat `Processing finished but wrote no image products.` i**kończy się wynikiem niezerowym**, więc skrypty mogą to wykryć. Typowe przyczyny: folder wejściowy nie został rozpoznany jako przechwycony (sprawdź układ i `--input-level`) lub każdy żądany produkt był nieodpowiedni dla tych kamer (np. żądanie promieniowania/odbicia od kamer obsługujących wyłącznie trybRGB
).
* **Celowe uruchomienie wyłącznie z metadanymi** (wszystkie produkty wyłączone, brak `--indices`) nadal jest traktowane jako sukces — pusty obraz wyjściowy jest w tym przypadku prawidłowym wynikiem.
* Uruchom ponownie z opcją `--verbose` i sprawdź dziennik backendu pod kątem wierszy `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, które wyjaśniają pominięcia poszczególnych kamer.

Kody wyjścia: `0` — sukces · `1` — ogólna awaria · `2` — błąd argumentu · `130` — przerwanie za pomocą Ctrl+C.

***

## Wskaźniki roślinności

Uruchom `--indices` z jedną lub kilkoma nazwami presetów; każdy wskaźnik trafia do własnego folderu `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 nazwy predefiniowanych ustawień akceptowanych przez `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Istnieją trzy listy indeksów — nie należy ich mylić.**Lista rozwijana „Ustawienia projektu” w interfejsie graficznym zawiera 27 wzorów (dodano `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — te pięć jest dostępnych wyłącznie w interfejsie graficznym i**nie** ma zastosowania do `--indices`). Polecenie „live/offline” `lattice index --preset` korzysta z własnej, oddzielnej listy 22 ustawień wstępnych. Wzory i obliczenia pasmowe zostały opisane w [Wzory indeksów wielospektralnych](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Czujniki światła DAQ: krótki przegląd

Rodzina `daq pool-*` steruje czujnikami spektralnymi DAQMAPIR
(DAQ-U przez USB, DAQ-M przez BLE, DAQ-E przez Ethernet) za pośrednictwem stałej puli backendu — GUI,CLI
iSDK
współdzielą jeden uchwyt na żywo. **`pool-*` to obsługiwana ścieżka DAQ w dostarczonym pakiecieCLI
**; inne podpolecenia `daq`, o których możesz się spotkać, sąMAPIR
— wewnętrznym interfejsem dostępnym wyłącznie dla źródła i kończą się wyraźnym błędem odsyłającym do `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` bez `--duration` działa do momentu `pool-record --stop`; domyślnym katalogiem wyjściowym jest `~/Documents/DAQ Live View/` **na komputerze backendu**. Profil korekcji pojemności jest wybierany w momencie nawiązania połączenia (`--cap-id`, domyślny dla serwera `sunshine_cosine`) i można go zmienić na żywo za pomocą `pool-set-cap` — profile czułości oraz skalibrowany zakres czujnika zostały omówione w rozdziałach dotyczących DAQ niniejszej instrukcji.

{% hint style="warning" %}
**DAQ-E na hoście z wieloma kartami sieciowymi:** pierwsze automatyczne wykrywanie `pool-connect --eth` po uruchomieniu systemu może zakończyć się niepowodzeniem, nawet jeśli czujnik jest sprawny. `--eth-host <ip-or-hostname>` jest niezawodną opcją — należy z niej skorzystać zawsze, gdy wykrywanie nie przyniesie rezultatów.
{% endhint %}

***

## Kamery LATTICE, PTP i automatyzacja projektów

Rodzina poleceń `lattice` (ponad 45 podpoleceń) obejmuje kompleksową obsługę kamer LATTICE: wykrywanie, pojedyncze przechwytywanie, trwałe zsynchronizowane tablice z inteligentnym procesem przygotowania połączenia w interfejsie graficznym, podgląd na żywo w przeglądarce, wyrównywanie, obliczenia indeksowe oraz diagnostykę karty sieciowej hosta. Przykład:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Oprócz tego: `chloros-cli time-sync` generuje raporty dotyczące serwera PTP typu „grandmaster”, na którym działa hostChloros
(kamery LATTICE i czujniki DAQ-E są podłączone do niego jako urządzenia podrzędne w celu uzyskania znaczników czasu między urządzeniami), a `chloros-cli project` otwiera zapisany projektChloros
i steruje kamerami, matrycami oraz czujnikami w trybie bezinterfejsowym — w tym za pomocą skryptowych receptur przechwytywania w formacie YAML.

Te trzy rodziny (`lattice`, `project`, `daq pool-*`) są również jedynymi, które obsługują opcję `CHLOROS_BACKEND_URL` do sterowania **zdalnym** backendem; podstawowe polecenia zawsze są skierowane do komputera lokalnego.

Pełne instrukcje krok po kroku znajdują się w rozdziałach poświęconych LATTICE w niniejszej instrukcji; wszystkie flagi są opisane w [CLI
dokumentacji](reference/cli-reference.md).

***

## Rozwiązywanie problemów: 5 najczęstszych

| Objaw | Rozwiązanie |
| --- | --- |
| `Login required` lub zaplanowane zadanie zawiesza się przy monicie `Email:` | Uruchom `chloros-cli login EMAIL 'PASSWORD'` raz na tym komputerze — polecenia bez buforowanej sesji będą wykonywane w trybie interaktywnym zamiast kończyć się natychmiastowym błędem. |
| `backend unreachable` | Uruchom aplikację desktopowąChloros
lub bezpośrednio plik wykonywalny zaplecza (`chloros-backend`). Jeśli skierujesz `lattice`/`project`/`daq pool-*` na zdalny backend, sprawdź `CHLOROS_BACKEND_URL`. |
| Blokada połączenia z macierzą: `FRAMES WILL DROP` / `Reduce ROI to enable` | Pierścień odbiorczy karty sieciowej hosta został zresetowany do ustawień domyślnych — jest to najczęstsza przyczyna, dla której wcześniej działająca platforma odmawia połączenia, zazwyczaj po aktualizacji sterownika karty sieciowej. Uruchom polecenie `chloros-cli lattice network --fix` z terminala o **podwyższonych uprawnieniach** (lub ustaw `ReceiveBufferLen=256`, `PendingReceives=64`); zobacz sekcję *Konfiguracja i dostrajanie karty sieciowej hosta* w dokumentacji. |
| Podpolecenie `daq` kończy działanie z komunikatem: „wymaga pełnego pakietu daq…” | Jest to oczekiwane w dostarczonych kompilacjach — skompilowany pakietCLI
zawiera wyłącznie rodzinę poleceń `daq pool-*`, która obejmuje nawiązywanie połączenia, przesył strumieniowy, rejestrację oraz wybór pliku cap. Należy użyć `pool-*` (lub `chloros_sdk.connect_daq_sensor()` zPython
). |
| Jetson wyświetla ostrzeżenie o pamięci wymiany przed obsługą dużych folderów | Dodaj pamięć wymiany opartą na plikach — plikCLI
zawiera dokładne polecenia `fallocate`/`swapon`, które należy uruchomić. |

***

## Pomoc

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Każda flaga, każde podpolecenie:** [CLI
Dokumentacja](reference/cli-reference.md)
* **Odpowiedniki wPython
:** [Python
SDK
](api-python-sdk.md) oraz [SDK
Dokumentacja](reference/sdk-reference.md)
* **Wsparcie:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
