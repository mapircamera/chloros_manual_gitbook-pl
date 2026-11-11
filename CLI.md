# CLI: Wiersz poleceń

> **Pełna dokumentacja:**[Dokumentacja CLI](reference/cli-reference.md) opisuje**każdą opcję każdego podpolecenia** i jest zoptymalizowana pod kątem asystentów AI — wklej jej URL do swojego asystenta i poproś o działające polecenie: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Wskazówka dotycząca narzędzi AI:** każda strona niniejszego podręcznika jest dostępna w formacie surowego Markdown poprzez dodanie `.md` do jej URL-u (np. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indeksuje cały podręcznik do wykorzystania przez modele LLM.

> **Chcesz, aby Twój asystent sam uruchamiał Chloros?** `chloros-mcp`, serwer MCP zainstalowany wraz z Python SDK, współpracuje z Claude Desktop, Claude Code, Cursor i innymi klientami MCP: zobacz [Korzystanie z Chloros z asystentami AI](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Czym jest interfejs CLI


`chloros-cli` to interfejs wiersza poleceń dla tego samego silnika przetwarzania, z którego korzysta aplikacja desktopowa Chloros
. Jest to lekki klient HTTP
działający w oparciu o backend Chloros
a (lokalny serwer na `127.0.0.1:5000`) — większość poleceń uruchamia backend automatycznie, więc skrypt potrzebuje jedynie jednego wywołania `chloros-cli process …`. Backend pozostaje następnie uruchomiony w oczekiwaniu na kolejne polecenie: zatrzymuje się po wywołaniu `chloros-cli logout` (chyba że aplikacja jest otwarta lub trwa uruchamianie lub kompilacja), po wywołaniu `chloros-cli backend stop` albo samoczynnie po 30 minutach bez klienta, uruchomionego zadania lub podłączonego urządzenia. Widać to w `chloros-cli backend status`.

Działa na **systemach Windows
10/11 (x64)**oraz**Linux
(x86_64 i NVIDIA Jetson arm64 na JetPack 6)**, w dowolnym terminalu, bez konieczności korzystania z GUI. Sprawdź poprawność instalacji za pomocą:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Przegląd grup poleceń:

* **Przetwarzanie i konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 języków — zobacz [Obsługiwane języki](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (tylko Linux
/Jetson)
* **Sprzęt działający na żywo** — `lattice` (sterowanie kamerą LATTICE, ponad 45 podpoleceń), `daq pool-*` (czujniki światła DAQ), `time-sync` (PTP)
* **Automatyzacja** — `project` (uruchamianie zapisanego projektu Chloros
a w trybie bezinterfejsowym, w tym receptury przechwytywania w formacie YAML)

Warto znać następujące opcje globalne: `--port N` (port zaplecza, domyślnie `5000`), `-v/--verbose`, `--restart` (wymuś ponowne uruchomienie backendu), `--backend-exe PATH`. Pełną listę opcji znajdziesz w [Podręczniku CLI
](reference/cli-reference.md).

***

## Instalacja

CLI
**jest dostarczany wraz z instalatorem Chloros** na każdej platformie — nie ma osobnego pliku do pobrania CLI
. Instalator można pobrać ze strony [Pobierz](download.md).

### Windows


Instalator umieszcza CLI
w lokalizacji:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

i jednorazowo dodaje ten folder do systemu `PATH` (aktualizacja usuwa również zduplikowane wpisy pozostawione przez wcześniejsze instalatory) — **po instalacji otwórz nowy terminal**, aby zaktualizowany `PATH` został wykryty. Instalator umieszcza również skrypty uruchamiające (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) w katalogu głównym instalacji oraz skrót**Chloros
CLI
** skrót w menu Start, z których każdy otwiera terminal z gotowym do użycia `chloros-cli`.

### Linux


Zainstaluj `.deb` dla swojej architektury za pomocą `apt` (a nie `dpkg -i`, który pomija zalecane wymagania Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Spowoduje to zainstalowanie `chloros-cli` do `/usr/bin/chloros-cli` (już w wersji `PATH`), a także backend i programy CLI
rozpakowane w `/usr/lib/chloros/backend/` i `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` to program uruchamiający, który uruchamia backend), środowisko uruchomieniowe Arena SDK
, którego backend potrzebuje do obsługi kamer LATTICE, oraz Python
SDK
dla systemu `python3`. Szczegółowe informacje można znaleźć w sekcji [Instalacja w systemie Linux
](linux/linux-installation.md).

### Weryfikacja

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Logowanie i licencjonowanie

Dostęp do CLI
(oraz Python
SDK
) wymaga **płatnego planu Chloros
+**— jest on dostępny w każdym płatnym planie; w planie darmowym nie ma go. Ograniczenie to jest egzekwowane**po stronie serwera** przez backend, a nie przez plik binarny CLI
: wywołanie bez zalogowania jest odrzucane z kodem `401 AUTH_REQUIRED`, a wywołanie po zalogowaniu się w ramach bezpłatnego planu — kodem `403 PLAN_UPGRADE_REQUIRED`, niezależnie od tego, czy pochodzi ono z `chloros-cli`, SDK
czy ręcznie napisanej aplikacji klienckiej HTTP
. Przeprowadź aktualizację pod adresem [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Zaloguj się **raz na każdym komputerze**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Hasła zawierające znaki specjalne**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` jest zniekształcane przez powłokę (CLI
wykrywa to na błędzie 401 i automatycznie ponawia próbę, ale cudzysłowy pojedyncze całkowicie pozwalają uniknąć tego problemu).
{% endhint %}

Sesja jest buforowana w `~/.chloros/user_session.json` i działa w trybie offline przez okres karencji planu (30 dni w przypadku planów miesięcznych, do momentu wygaśnięcia w przypadku planów rocznych). `chloros-cli status` działa nawet bez płatnego planu, więc przyczyna odmowy jest zawsze widoczna.

{% hint style="danger" %}
**Planujesz zadania bezinterfejsowe? Zaloguj się najpierw.**Polecenia uruchamiające procesy w tle (`process`, `status`, `export-status`, …) uruchomione**bez sesji w pamięci podręcznej**nie kończą się natychmiastowym błędem — przechodzą w tryb interaktywny z wierszem poleceń `Email:` / `Password:` na stdin. W związku z tym zadanie cron lub etap CI uruchomione w trybie nienadzorowanym**zawieszą się, czekając na dane wejściowe**. Przed zaplanowaniem jakichkolwiek zadań należy raz uruchomić polecenie `chloros-cli login EMAIL 'PASSWORD'` na danym komputerze.
{% endhint %}

***

## Pierwsze uruchomienie przetwarzania

Skieruj program `process` na folder zawierający dane zebrane — automatycznie wykryje on pliki Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` lub ich kombinację:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Strumienie postępu są przesyłane na żywo dla każdego wątku potoku (wykrywanie, analiza, przetwarzanie, eksport), a pomyślne zakończenie przebiegu kończy się zgłoszeniem liczby zapisanych produktów obrazowych: `Image products written: N`, a następnie `(plus M clamp masks)`, jeśli podczas uruchomienia zapisano również [maski clamp](output-image-formats.md#clamp-masks) (nie są one liczone jako produkty).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Jeśli backend Chloros
już działa (na przykład w aplikacji desktopowej), `process` korzysta z niego i otwiera w nim swój projekt. Odmawia działania, gdy aplikacja jest otwarta na projekcie, który sama otworzyła: należy uruchomić proces z poziomu aplikacji lub zamknąć ten projekt. Jeśli poprzednie uruchomienie wciąż się kończy, czeka na nie do 15 sekund, zanim odmówi. `--restart` wymusza uruchomienie nowego backendu. W systemie Linux
backend uruchamiany jako usługa systemd nigdy nie jest restartowany: `process` korzysta z niego w obecnym stanie, a w sytuacjach, w których w innym przypadku musiałby go restartować (`--restart` lub import, który nic nie znalazł), wyświetla komunikat `sudo systemctl restart <unit>` i kończy działanie `1`.

### Gdzie trafiają wyniki

`process` zapisuje wyniki w **folderze projektu**, a nie w folderze wejściowym:

* Bez `-o`: projekt jest tworzony w domyślnym folderze projektów (współdzielonym z interfejsem graficznym; zarządzaj nim za pomocą `get-project-folder` / `set-project-folder`; jeśli nie jest ustawiony, to w folderze użytkownika `Documents/MAPIR/Chloros Projects`), nazwanego zgodnie z `-n/--project-name` lub sygnaturą czasową (`YYYYMMDD_HHMMSS`), jeśli nie podano nazwy.
* W przypadku `-o PATH`: ten folder **jest** folderem projektu. Jeśli zawiera już plik `project.json`, zamiast nadpisywania tworzony jest plik o rozszerzeniu `_1`/`_2`… zamiast nadpisywania. Ścieżka względna `PATH` jest obliczana względem folderu, z którego uruchamiasz `chloros-cli`.

{% hint style="info" %}
**Ścieżki względne (zmienione w wersji 1.3.0).** Każda ścieżka otwierana przez backend — `-o`/`--output` przy dowolnym poleceniu (w tym domyślne `output` poleceń przechwytywania), `--input`, `--burst-dir`, `--profile`, `--save` oraz `--mask` — jest rozpoznawana względem folderu, z którego uruchamiane jest polecenie `chloros-cli`, więc polecenie `-o captures/` zapisuje dane do `<current folder>/captures/`. W wersji 1.2.0 backend rozpoznawał je we własnym folderze instalacyjnym, co kończyło się błędem „Odmowa dostępu”. W przypadku zdalnego backendu (`CHLOROS_BACKEND_URL`) ścieżki są wysyłane zgodnie z wpisanym tekstem i rozpoznawane na tej maszynie.
{% endhint %}

{% hint style="warning" %}
**Windows
: folder projektu powinien być krótki, chyba że włączono obsługę długich ścieżek.** Gdy w systemie Windows
wyłączona jest obsługa długich ścieżek (ustawienie domyślne), żadna ścieżka do pliku nie może przekraczać 260 znaków, a produkty znajdują się maksymalnie około 130 znaków poniżej folderu projektu. `process` sprawdza to przed uruchomieniem: zbyt głęboki folder projektu powoduje komunikat o błędzie **„Folder projektu jest zbyt głęboki dla tego komputera”**, podaje najdłuższy folder projektu, który się mieści, oraz instrukcję włączenia długich ścieżek — nic nie jest przetwarzane. Należy użyć krótszego `-o` (lub `set-project-folder`) albo włączyć obsługę długich ścieżek w systemie Windows
(opcja instalatora **Włącz długie ścieżki w systemie Windows**, zaznaczona domyślnie) i ponownie uruchomić program Chloros
.
{% endhint %}

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

Folder aparatu to `LATT-<sensor>-<lens>-F<filter>` dla LATTICE (zgodny z danymi EXIF zdjęcia `Model`) oraz `<model>_<filter>` (np. `Survey3N_RGN`) dla aparatu typu Survey3
. Foldery formatów mają nazwy zgodne z `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32` dla `TIFF (32-bit, Percent)`.

W przypadku wielospektralnej kamery M3C w folderze formatowym, obok folderów produktów, może również pojawić się folder `clamp_masks/` (na przykład `tiff16/clamp_masks/Reflectance_Calibrated_Images/` i `tiff32/clamp_masks/Radiance_Images/`). Zawiera on maski na piksel produktów promieniowania i odbicia, w których proces rozdzielania pasm ograniczył wartości ujemne do zera — zobacz [Maski ograniczające](output-image-formats.md#clamp-masks). Żaden folder produktu nigdy nie zawiera maski.

{% hint style="info" %}
**Każdy wyeksportowany produkt zachowuje nazwę pliku ŹRÓDŁOWEGO.**Eksport promieniowania z pliku `capture_..._raw.tif` nadal nosi nazwę `capture_..._raw.tif` — po prostu znajduje się w katalogu `tiff32/Radiance_Images/`.**To folder identyfikuje produkt, a nie nazwa pliku**, więc należy używać wyrażenia wzorcowego dla katalogu, a nie dla rozszerzenia `*radiance*`.
{% endhint %}

### Opcje, z których faktycznie będziesz korzystać

| Flaga | Domyślnie | Działanie |
| --- | --- | --- |
| `-o, --output PATH` | domyślny folder projektu | Lokalizacja folderu projektu (patrz wyżej). |
| `-n, --project-name NAME` | sygnatura czasowa | Nazwa projektu. |
| `--format FMT` | `TIFF (16-bit)` | Jedna z wartości: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
nie zwiększa rozdzielczości: w przypadku obrazów całkowicie 8-bitowych format o większej głębi jest zapisywany jako `TIFF (8-bit)`, o czym informuje się tylko raz. |
| `--indices NAME [NAME ...]` | brak | Wskaźniki wegetacji do eksportu (zobacz [Wskaźniki wegetacji](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronalna eliminacja efektu bayera, wolniejsza, najwyższa jakość (Chloros
+, wymaga procesora graficznego z obsługą CUDA: bez niego program korzysta z `standard`; nie jest dostępny dla ujęć LATTICE, które zawsze otrzymują `standard`). |
| `--vignette / --no-vignette` | włączone | Korekcja winietowania. |
| `--reflectance / --no-reflectance` | włączone | Kalibracja odbicia; w przypadku LATTICE jest to również przełącznik produktu odbicia. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Wymuś punkt wejścia potoku dla plików TIFF z LATTICE. |

W przypadku wszystkich pozostałych opcji — dostrajania wykrywania celów, PPK, pinów ekspozycji, flag wyrównania tablic — zapoznaj się z [sekcją `process` w Podręczniku CLI
](reference/cli-reference.md).

***

## Wybór elementów do eksportu (produkty LATTICE)

Przetwarzanie LATTICE rozgałęzia się na **każdy odpowiedni produkt w jednym przebiegu**. Cztery przełączniki dla poszczególnych produktów są**domyślnie włączone**; użyj formularza `--no-`, aby wyłączyć jeden z nich:

| Przełącznik | Produkt |
| --- | --- |
| `--debayered` | Liniowa demosaika → `Debayered_Images/`: usunięto ciemne przesunięcie z pakietu kalibracyjnego, a następnie zastosowano demosaikowanie z uwzględnieniem krawędzi, bez korekcji liniowości, płaskiego pola ani rozdzielania — ten sam plik, jaki zapisuje się na poziomie `debayered`. Zachowano natywną orientację czujnika (zapisaną w XMP). |
| `--preview` | Podgląd na ekranie (balans bieli + gamma; rozciągnięcie w fałszywych kolorach dla obrazów wielospektralnych) → `Preview_Images/` |
| `--radiance` | promieniowanie typu float32, W/m²/sr/nm → `Radiance_Images/` (zawsze `tiff32/`) |
| `--reflectance` | uint16 współczynnik odbicia, gotowy do użycia w Pix4D → `Reflectance_Calibrated_Images/` |

Kamery główne RGB
zawsze emitują wyłącznie dane po usunięciu efektu Bayesa + podgląd — promieniowanie/odbicie w poszczególnych pasmach nie ma znaczenia dla czujnika szerokopasmowego, więc te przełączniki nie mają dla nich żadnego wpływu. Survey3
`.raw` ignoruje te przełączniki i stosuje standardową ścieżkę odbicia/docelową.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (domyślnie `auto`) wybiera odniesienie dla współczynnika odbicia: `auto` ustala jako absolutny punkt odniesienia [cel kalibracyjny](calibration-targets.md) znajdujący się w kadrze i przeszedł kontrolę jakości (QA), a w przypadku braku celu powraca do podziału światła padającego czujnika DAQ (ρ = π·L/E), gdy nie ma żadnego celu (cel, który przeszedł kontrolę jakości z flagą, konkuruje z DAQ pod względem zmierzonego wyniku); `target` działa ściśle (bez zastępowania przez DAQ); `daq` opiera się na danych z DAQ. Skany celów zmierzone na jednostkę można dostarczyć za pomocą `--target-reflectance-dir`.

{% hint style="info" %}
**Odczyt pikseli odbicia:**wartość DN oznaczająca ρ = 1,0 odnosi się**do poszczególnych źródeł** — pliki LATTICE umieszczają znacznik `Chloros:PixelScale=32768` w metadanych XMP; Pliki Survey3
używają wartości 65535 i nie zawierają tagu `Chloros:PixelScale` (zawierają natomiast inne tagi przetwarzania `Chloros:*`). Należy odczytać ten tag i podzielić przez jego wartość, zamiast zakładać stałą wartość. Szczegóły oraz jeden celowy przypadek braku skali znajdują się w [Podręczniku CLI
](reference/cli-reference.md).
{% endhint %}

**Przetwarzanie zawsze rozpoczyna się od `raw`.** Produkty pochodne (eksporty danych po usunięciu efektu debayeringu, promieniowania lub odbicia) nigdy nie są ponownie wprowadzane do potoku przetwarzania — ponowne ich zaimportowanie i przetworzenie spowodowałoby podwójne zastosowanie obliczeń kalibracyjnych, więc Chloros
pomija je i informuje o tym. `--input-level` stanowi celowo przewidziane wyjście awaryjne na wypadek, gdy naprawdę konieczne jest wymuszenie punktu wejścia.

***

## Gdy przetwarzanie kończy się niepowodzeniem

Od wersji 1.2.0 kod `process` wyraźnie sygnalizuje niepowodzenie zamiast „zakończyć się powodzeniem” bez wyświetlania żadnych wyników:

* Uruchomienie, które **zażądało produktów, ale nie zapisało żadnego**— tylko `project.json` i `calibration_data.json` — wyświetla komunikat `Processing finished but wrote no image products.` i**kończy się wynikiem niezerowym**, dzięki czemu skrypty mogą to wykryć. Typowe przyczyny: folder wejściowy nie został rozpoznany jako przechwycony (sprawdź układ i `--input-level`) lub każdy żądany produkt był nieodpowiedni dla tych kamer (np. żądanie radiancji/odbicia od kamer obsługujących wyłącznie RGB
).
* **Celowe uruchomienie wyłącznie z metadanymi** (wszystkie produkty wyłączone, brak `--indices`) nadal kończy się sukcesem — pusty obraz wyjściowy jest w tym przypadku prawidłowym wynikiem.
* Uruchom ponownie z opcją `--verbose` i sprawdź dziennik zaplecza pod kątem wierszy `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, które wyjaśniają pominięcia poszczególnych kamer.
* W systemie Windows
, gdy opcja długich ścieżek jest wyłączona, folder projektu, którego nazwa przekracza limit 260 znaków, zostanie odrzucony **przed** rozpoczęciem przetwarzania, a zostanie wybrany najdłuższy folder projektu, który mieści się w tym limicie (zobacz [Gdzie trafiają pliki wyjściowe](#where-the-outputs-land)).
* Polecenia przechwytywania (`lattice capture`, `lattice array-capture` i inne rodziny poleceń przechwytywania) podają nazwy wszystkich plików, których nie udało się zapisać, i kończą działanie z kodem `1`; powtarzające się operacje przechwytywania zatrzymują się, gdy dysk jest pełny.

Kody zakończenia: `0` — sukces · `1` — błąd ogólny · `2` — błąd argumentu · `130` — przerwanie za pomocą Ctrl+C.

***

## Wskaźniki roślinności

Uruchom `--indices` z jedną lub kilkoma nazwami presetów; każdy wskaźnik trafia do własnego folderu `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 nazwy ustawień wstępnych akceptowane przez `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Do projektu dodawana jest nazwa, której jeszcze w nim nie ma. Nazwa, która nie jest ustawieniem domyślnym lub której pasma kamer nie są w stanie utworzyć, jest pomijana wraz z ostrzeżeniem podającym jej nazwę. Jeśli interfejs CLI
nie może odczytać bieżącej listy indeksowej projektu, pozostawia ją bez zmian i wyświetla ostrzeżenie zamiast ją zastępować.

{% hint style="warning" %}
**Istnieją trzy listy indeksowe — nie należy ich mylić.**Lista rozwijana „Ustawienia projektu” w interfejsie graficznym zawiera 27 formuł (dodaje `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — te pięć jest dostępnych wyłącznie w interfejsie graficznym i**nie** ma zastosowania do `--indices`). Polecenie `lattice index --preset` (na żywo/offline) korzysta z własnej, oddzielnej listy 22 ustawień wstępnych. Wzory i obliczenia pasmowe zostały opisane w [Wzory indeksów wielospektralnych](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Czujniki światła DAQ: krótki przegląd

Rodzina `daq pool-*` steruje czujnikami spektralnymi DAQ firmy MAPIR
(DAQ-U przez USB, DAQ-M przez BLE, DAQ-E przez Ethernet) za pośrednictwem stałej puli backendu — GUI, CLI
i SDK
współdzielą jeden uchwyt na żywo. **`pool-*` to obsługiwana ścieżka DAQ w dostarczonym CLI
**; inne podpolecenia `daq`, o których możesz spotkać się w kontekście, stanowią wyłącznie wewnętrzną powierzchnię MAPIR
przeznaczoną wyłącznie do odczytu i kończą działanie z wyraźnym komunikatem o błędzie, kierującym użytkownika do `pool-*`.

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

`pool-record` bez `--duration` działa do momentu `pool-record --stop`; domyślnym katalogiem wyjściowym jest `~/Documents/DAQ Live View/` **na komputerze backendu**. Profil korekcji pojemnościowej jest wybierany w momencie nawiązania połączenia (`--cap-id`, domyślny dla serwera `sunshine_cosine`) i można go zmienić na żywo za pomocą `pool-set-cap` — profile limitów oraz skalibrowany zakres czujnika zostały omówione w rozdziałach dotyczących DAQ niniejszej instrukcji.

{% hint style="warning" %}
**DAQ-E na hoście z wieloma kartami sieciowymi:** pierwsze automatyczne wykrywanie `pool-connect --eth` po uruchomieniu systemu może zakończyć się niepowodzeniem, nawet jeśli czujnik działa prawidłowo. `--eth-host <ip-or-hostname>` jest niezawodną opcją — należy z niej skorzystać zawsze, gdy wykrywanie nie przyniesie rezultatów.
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

Oprócz tego: `chloros-cli time-sync` generuje raport dotyczący serwera nadrzędnego PTP, na którym działa host Chloros
(kamery LATTICE i czujniki DAQ-E są podłączone do niego jako urządzenia podrzędne w celu uzyskania znaczników czasu między urządzeniami), a plik `chloros-cli project` otwiera zapisany projekt Chloros
i steruje jego kamerami, matrycami oraz czujnikami w trybie bezinterfejsowym — w tym za pomocą skryptowych receptur przechwytywania w formacie YAML.

Te trzy rodziny plików (`lattice`, `project`, `daq pool-*`) są również jedynymi, które obsługują opcję `CHLOROS_BACKEND_URL` do sterowania **zdalnym** backendem; podstawowe polecenia zawsze są skierowane do maszyny lokalnej.

Pełne instrukcje krok po kroku znajdują się w rozdziałach poświęconych LATTICE w niniejszej instrukcji; wszystkie opcje są opisane w [Podręczniku CLI
](reference/cli-reference.md).

***

## Rozwiązywanie problemów: 5 najczęstszych

| Objaw | Rozwiązanie |
| --- | --- |
| `Login required` lub zaplanowane zadanie zawiesza się przy wierszu poleceń `Email:` | Uruchom raz polecenie `chloros-cli login EMAIL 'PASSWORD'` na tym komputerze — polecenia bez buforowanej sesji będą wykonywane interaktywnie zamiast natychmiastowego zakończenia z błędem. |
| `backend unreachable` | Uruchom aplikację desktopową Chloros
lub samodzielnie uruchom backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` w systemie Windows
(folder `backend` w folderze instalacyjnym; nie ma go w `PATH`), `chloros-backend` w systemie Linux
. `chloros-cli backend status` pokazuje, czy któryś z nich jest uruchomiony; `lattice`/`project`/`daq pool-*` nigdy go nie uruchamiają. Jeśli skierujesz je na zdalny backend, sprawdź `CHLOROS_BACKEND_URL`. |
| Zablokowano połączenie z macierzą: `FRAMES WILL DROP` / `Reduce ROI to enable` | Pierścień odbiorczy karty sieciowej hosta został zresetowany do ustawień domyślnych — jest to najczęstsza przyczyna, dla której wcześniej działający system odmawia połączenia, zazwyczaj po aktualizacji sterownika karty sieciowej. Uruchom polecenie `chloros-cli lattice network --fix` z terminala z **podwyższonymi uprawnieniami** (lub ustaw `ReceiveBufferLen=256`, `PendingReceives=64`); zobacz sekcję *Konfiguracja i dostrajanie karty sieciowej hosta* w dokumentacji. |
| Podpolecenie `daq` kończy działanie: „wymaga pełnego pakietu DAQ…” | Użyj poleceń `daq pool-*`, które sterują tym samym czujnikiem za pośrednictwem zaplecza: połączenie, strumieniowanie, rejestracja, wybór limitu oraz — w przypadku DAQ-E — ustawienia sieciowe, restart i aktualizacje oprogramowania układowego (lub `chloros_sdk.connect_daq_sensor()` z języka Python
). |
| Jetson wyświetla ostrzeżenie o pamięci wymiany przed obsługą dużych folderów | Dodaj pamięć wymiany opartą na plikach — interfejs CLI
wyświetla dokładne polecenia `fallocate`/`swapon`, które należy uruchomić. |

***

## Pomoc

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Każda opcja, każde podpolecenie:** [Podręcznik CLI
](reference/cli-reference.md)
* **Odpowiednik w Python
:** [Python
SDK
](api-python-sdk.md) oraz [Podręcznik SDK
](reference/sdk-reference.md)
* **Pomoc techniczna:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
