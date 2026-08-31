# Chloros CLI Odniesienie

**Wersja:**

1.2.0**Wygenerowano:**

29.07.2026 r., godz. 19:19 ·**Zaktualizowano:**

30.08.2026 r.**Grupa docelowa:** Zoptymalizowany pod kątem wykorzystania przez modele LLM; czytelny dla ludzi.**Zakres:** Każde podpolecenie `chloros-cli` przeznaczone dla użytkowników, wraz z opcjami i przykładami, które można skopiować i wkleić.

Niniejszy dokument stanowi kompletny przewodnik po poleceniu `chloros-cli`dostarczanego wraz z MAPIR Chloros. Został on celowo opracowany w sposób wyczerpujący, tak aby model LLM (lub człowiek) mógł skomponować dowolny obsługiwany przepływ pracy na podstawie poniższych wykazów bez konieczności wglądu w kod źródłowy.

Jeśli potrzebujesz tylko najważniejszych informacji, przejdź do:
- [Pięciominutowy przewodnik szybkiego startu](#five-minute-quickstart)
- [Przepływ pracy „Pierwsze podłączenie kamery LATTICE”](#lattice-camera-first-connect-workflow)
- [Przebieg pierwszego podłączenia czujnika DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Tryby przechwytywania, rejestratory i przetwarzanie offline](#capture-modes-recorders--offline-reprocess)

---

## Konwencje

- Wszystkie polecenia mają prefiks `chloros-cli`. W przypadku Windows plik binarny to `chloros-cli.exe`; w przypadku Linux/Jetson jest to jest to `chloros-cli`.
- Argumenty opcjonalne są przedstawione jako `--flag`. Wymagane argumenty pozycyjne są przedstawione bez nawiasów.
- W przypadku podania wartości domyślnej pominięcie flagi powoduje użycie tej wartości.
- CLI jest „cienkim” klientem HTTP opartym na backendzie Chloros (serwer Flask na `127.0.0.1:5000`). Backend jest uruchamiany automatycznie przez większość poleceń. `CHLOROS_BACKEND_URL=<url>` kieruje rodziny poleceń **`lattice`**,**`project`**oraz**`daq pool-*`** na zdalny backend — podstawowe polecenia (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) celowo blokują `http://127.0.0.1:<port>` i ignorują go (literał IPv4 pozwala uniknąć kary wynoszącej około 2 sekund na żądanie w przypadku Windows → `localhost` → `::1`). Zobacz [Zmienne środowiskowe](#environment-variables).
- Do wszystkich wywołań SDK/CLI wymagane jest zalogowanie się na konto Chloros+ (uruchom `chloros-cli login` raz na każdym komputerze; dane są buforowane w `~/.chloros/`).
- W przykładach użyto ścieżek Linux; w przypadku Windows należy zastąpić `/home/user/...` na `C:/Users/.../...`.

---

## Przegląd najwyższego poziomu

```
chloros-cli [global options] COMMAND [command options]
```

### Opcje globalne

| Flaga | Opis |
| --- | --- |
| `--backend-exe PATH` | Zastąpienie automatycznie wykrytego pliku wykonywalnego modułu zaplecza. |
| `--port N` | Port modułu zaplecza HTTP (domyślnie: `5000`). |
| `-v, --verbose` | Włącz tryb szczegółowego wyświetlania. |
| `--restart` | Wymuśponownego uruchomienia backendu (zamyka wszystkie uruchomione procesy `backend_server.py`). |
| `--version` | Wyświetl wersję (`Chloros CLI 1.2.0`). |
| `--help` | Wyświetla pomoc najwyższego poziomu. |

### Indeks poleceń

| Polecenie | Przeznaczenie |
| --- | --- |
| [`process`](#chloros-cli-process) | Przetwarzaj folder zawierający zapisy Survey3 lub LATTICE od początku. |
| [`login`](#chloros-cli-login) | Uwierzytelnij to urządzenie za pomocą konta Chloros+. |
| [`logout`](#chloros-cli-logout) | Wyczyść dane uwierzytelniające z pamięci podręcznej. |
| [`status`](#chloros-cli-status) | Wyświetl aktualny status licencji / uwierzytelnienia. |
| [`export-status`](#chloros-cli-export-status) | Wyświetl na żywo postęp eksportu Thread-4 podczas działania `process`. |
| [`language`](#chloros-cli-language) | Ustawianie lub wyświetlanie listy języków wyświetlania CLI (obsługiwanych jest 38). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#polecenia-folderu-projektu) | Domyślny folder projektu (wspólny z interfejsem graficznym). |
| [`update`](#chloros-cli-update) | Sprawdź dostępność aktualizacji CLI i zainstaluj je (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostyka systemu + testy wstępne. |
| [`time-sync`](#chloros-cli-time-sync) | Status i sterowanie serwerem głównym PTP. |
| [`lattice`](#chloros-cli-lattice) | Sterowanie kamerą LATTICE i przechwytywanie obrazu (ponad 45 podpoleceń). |
| [`daq`](#chloros-cli-daq) | Sterowanie czujnikami spektralnymi DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otwieranie i uruchamianie zapisanego projektu Chloros (kamery + urządzenia DAQ). |

---

## Instalacja

`chloros-cli` jest dostarczany w ramach instalatora Chloros na każdą obsługiwana platformę — nie ma osobnego pliku do pobrania CLI. Zainstalowanie pakietu platformowego powoduje dodanie `chloros-cli` do katalogu `PATH` wraz z aplikacją desktopową i obsługującym ją plikiem binarnym zaplecza.

Najnowsze pliki do pobrania: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Instalator zawiera również skrypty uruchamiające (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), które otwierają gotową do użycia powłokę CLI; zostały one omówione w [Podręczniku użytkownika CLI](../CLI.md) i nie są powielane w niniejszym dokumencie.

### Windows (.exe)

1. Pobierz instalator Windows ze strony pobierania.
2. Uruchom `Chloros-Setup-x.y.z.exe` i postępuj zgodnie z instrukcjami kreatora. Domyślną ścieżką instalacji jest `C:\Program Files\Chloros\` (plik CLI trafia do `C:\Program Files\Chloros\cli\`, który instalator dodaje do zmiennej PATH).
3. Otwórz nowe okno terminala (`cmd.exe`, PowerShell lub Windows Terminal) , aby uwzględnić zaktualizowany plik `PATH`.

```powershell
chloros-cli --version
```

Instalator automatycznie dodaje `chloros-cli.exe` do systemu `PATH` oraz dołącza środowisko uruchomieniowe Arena SDK niezbędne dla kamer LATTICE.

### Linux amd64 (.deb)

Dla Ubuntu 22.04 LTS lub nowszego / stacji roboczych x86_64 opartych na Debianie.

> **Ubuntu 20.04 nie jest obsługiwane.** Lista zależności pakietu pochodzi z
> tego, z czym faktycznie łączy się moduł zaplecza, a obejmuje to `libc6 (>= 2.34)`;
> w dystrybucji Focal znajduje się glibc 2.31. `apt` odrzuca instalację, zamiast dopuścić do jej niepowodzenia w
> czasie wykonywania.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Plik .deb instaluje:
- `chloros-cli` do `/usr/bin/chloros-cli`
- Skompilowany backend do `/usr/lib/chloros/chloros-backend`
- Środowisko uruchomieniowe Arena SDK (dla kamer LATTICE)
- modele odszumiania, pakiety kalibracyjne oraz konfigurację kanału aktualizacji

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ten sam układ co plik .deb dla architektury amd64, z kompilacją CUDA dostosowaną do Jetson Orin / Orin NX / Orin Nano.

### Uwierzytelnianie jednorazowe na urządzeniu

Każda platforma wymaga jednorazowego logowania Chloros+ przed uruchomieniem funkcji SDK/CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Dane uwierzytelniające są buforowane w `~/.chloros/user_session.json`.

### Sprawdź instalację

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Wymagana subskrypcja Chloros+.**Usługa CLI wymaga aktywnego planu Chloros+.**Copper**to podstawowy poziom Chloros+ — każdy płatny poziom Chloros+ zapewnia dostęp do CLI/SDK; jedynie bezpłatny poziom**Iron** nie zapewnia takiego dostępu. (Mapowanie identyfikatorów planów: `0`=Iron/bezpłatny, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Uaktualnienie dostępne pod adresem [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Ten limit jest egzekwowany przez zaplecze, a nie tylko przez CLI: żądanie oznaczone jako SDK/CLIbez opłaconego planu jest odrzucane z kodem `403 PLAN_UPGRADE_REQUIRED`, niezależnie od tego, czy pochodzi z serwera `chloros-cli`, serwera Python SDK, czy też z własnoręcznie skompilowanego klienta HTTP. Użytkownik, który się wylogował, otrzymuje zamiast tego kod błędu `401 AUTH_REQUIRED`. Dostęp działa w trybie offline przez okres karencji planu (30 dni w przypadku planu miesięcznego, do upływu terminu ważności w przypadku planu rocznego) i wygasa po upływie tego okresu; kod `chloros-cli status` nadal działa, dzięki czemu przyczyna jest widoczna (jest to trasa SDK/CLI — `GET /api/license-status`).

---

## Pięciominutowy przewodnik szybkiego startu

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Przetwórz folder obrazów przez pełny potok Chloros (wykrywanie obiektów → kalibracja → winieta → odbicie → eksport indeksu).

### Składnia

```
chloros-cli process INPUT [OPTIONS]
```

### Argumenty pozycyjne

| Argument | Opis |
| --- | --- |
| `INPUT` | Ścieżka do folderu wejściowego zawierającego pliki `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) lub `.dng`. |

### Opcje ogólne

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `-o, --output PATH` | nowy folder z sygnaturą czasową w domyślnej ścieżce projektu (`~/Chloros Projects`, o ile nie skonfigurowano inaczej) | Folder projektu do utworzenia lub ponownego wykorzystania. Jeśli folder zawiera już plik `project.json`, zamiast nadpisywania tworzony jest plik równorzędny `_1`/`_2`. |
| `-n, --project-name NAME` | auto (znacznik czasu) | Nazwa projektu. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` wykorzystuje algorytm neuronowego usuwania efektu bayera Chloros+; działa wolniej, ale zapewnia wyższą jakość. |
| `--vignette / --no-vignette` | `--vignette` | Korekcja winietowania. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibracja odbicia (wykorzystuje panel docelowy, jeśli zostanie znaleziony, kalibrację NIST dla poszczególnych seriali w przypadku LATTICE). W przypadku LATTICE multispektralnego pełni to również funkcję przełącznika **produktu** odbicia — zobacz [Przełączniki eksportu dla poszczególnych produktów](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | wył. | Zastosuj korekty PPK GNSS z plików sidecar. |
| `--exposure-pin-1 MODEL` | wył. | Zablokuj model „pin-1” zestawu z dwoma kamerami Survey3 „pin-1”. |
| `--exposure-pin-2 MODEL` | wył. | Przypisz model „pin-2”. |
| `--recal-interval SECONDS` | 0 | Wymuś ponowne wykonanie obliczeń kalibracyjnych co N sekund czasu rejestracji. |
| `--timezone-offset HOURS` | lokalny | Zastąp przesunięcie strefy czasowej zapisane w metadanych wyjściowych. |
| `--format FORMAT` | `TIFF (16-bit)` | Jeden z `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | brak | Wskaźniki roślinności (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Wymuszanie punktu wejścia do potoku dla plików LATTICE TIFF (nie ma to wpływu na plik Survey3 .raw). Również „wyjście awaryjne”, które pozwala na przetworzenie przechwycenia **bez danych surowych** — zobacz [Jak wygląda folder przechwyceń](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | włączone | Generowanie produktu z liniowego usuwania efektu Bayera (`Debayered_Images`). Zobacz [Przełączniki eksportu dla poszczególnych produktów](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | włączone | Generowanie podglądu na ekranie (`Preview_Images`): RGB = balans bieli (źródło światła DAQ, jeśli jest dostępne, w przeciwnym razie świat szary) + gamma; multispec = rozciągnięcie w fałszywych kolorach. |
| `--radiance / --no-radiance` | wł. | Wyświetla promieniowanie typu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Odniesienie dla produktu odbicia LATTICE: `auto` = cel w kadrze, który przeszedł kontrolę jakości (QA), jest odniesieniem bezwzględnym, rezerwa dla danych DAQ dotyczących promieniowania padającego (ρ = π·L/E); `target` = ścisłe (bez substytucji DAQ); `daq` = autorytatywne dla DAQ. Zobacz [Przełączniki eksportu dla poszczególnych produktów](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | brak | Katalog skanów **zmierzonej** odbicia docelowego dla poszczególnych jednostek (`<serial>.csv`); w przypadku braku danych stosuje się nominalne widma T3/T4P. |
| `--array-alignment / --no-array-alignment` | włączone | Tablice LATTICE: zastosowanie wyrównania modułu-do-modułu zapisane w pliku XMP `Chloros:Alignment*` każdego ujęcia do każdego przetworzonego produktu (odbayering / podgląd / promieniowanie / odbicie / indeks). Nie wykonuje operacji w przypadku obrazów bez tych tagów. |
| `--array-alignment-crop / --no-array-alignment-crop` | przycinanie | Przycinaj wyrównane eksporty do wspólnego obszaru nakładania się macierzy, tak aby wszystkie moduły miały ten sam obszar pokrycia; `--no-…` zachowuje pełny obszar czujnika (czarne wypełnienie poza źródłem). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Próbkowanie ponowne w celu korekcji zniekształceń wynikających z wyrównania. `nearest` zachowuje dokładne wartości DN źródła (bez mieszania wartości radiometrycznych między pikselami). |

### Opcje wykrywania celów

| Flaga | Opis |
| --- | --- |
| `--min-target-size PIXELS` | Minimalny rozmiar panelu-celu (px) dla detektora. |
| `--target-clustering 0-100` | Czułość klastrowania. |
| `--target / --targets` | Traktuj folder wejściowy jako zawierający wyłącznie panele docelowe (pomiń wykrywanie zdjęć przeglądowych). |

### Przykłady

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

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

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Przełączniki eksportu dla poszczególnych produktów (LATTICE multispektralny)

Przetwarzanie LATTICE rozgałęzia się na **każdy odpowiedni produkt w jednym przebiegu**. Cztery przełączniki dla poszczególnych typów — `--debayered`, `--preview`, `--radiance`, `--reflectance` — są**domyślnie włączone**; aby wyłączyć jeden z nich, należy użyć formularza `--no-<type>`. Kamery główne typu RGB wysyłają wyłącznie dane po usunięciu efektu Bayera oraz podgląd (bez promieniowania/odbicia dla poszczególnych pasm), więc `--radiance`/`--reflectance` nie mają dla nich żadnego działania. Przełączniki są ignorowane dla Survey3 `.raw` (które działają zgodnie ze standardową ścieżką odbicia/celu). *(Stara flaga `--radiometric-output {reflectance,radiance,sensor-response}` została **usunięta** i zastąpiona tymi przełącznikami; nie ma już poziomu `sensor-response`.)*

| Produkt | Wynik | Czy wymagane jest zbieranie danych DAQ w kierunku w dół? |
| --- | --- | --- |
| `--debayered` | Demosaikowanie liniowe (`Debayered_Images`). | Nie |
| `--preview` | Podgląd wyświetlania (`Preview_Images`): RGB = WB + gamma; multispec = false– rozciągnięcie kolorów. | Nr |
| `--radiance` | float32 W/m²/sr/nm z pełnego łańcucha radiometrycznego (`Radiance_Images`). | Nr |
| `--reflectance` | współczynnik odbicia ρ typu uint16 (`32768` = 1,0), gotowe do Pix4D. | **Tak**, chyba że jest zakotwiczone przez obiekt w kadrze, który przeszedł kontrolę jakości (patrz poniżej). |

`--reflectance-source` wybiera odniesienie dla współczynnika odbicia:**`auto`**(domyślnie) ustanawia cel w kadrze, który przeszedł kontrolę jakości, jako**odniesienie bezwzględne**— łańcuchy linii empirycznych zakotwiczone na celu są porównywane krzyżowo na zatrzymanychpaneli odrzuconych, a zwycięska wartość jest stosowana — w przypadku braku celu lub niepowodzenia kontroli jakości stosowany jest podział oparty na danych z DAQ (ρ = π·L/E);**`target`**jest opcją ścisłą (bez substytucji DAQ);**`daq`**rezygnuje z zachowania opartego na autorytecie DAQ. Geometria celu (ArUco / stały obszar zainteresowania / pasek) pochodzi z konfiguracji celu projektu; `--target-reflectance-dir DIR` przechowuje skany**zmierzone** skany (`<serial>.csv`) wyszukiwane na podstawie numeru seryjnego/kodu QR jednostki docelowej, z nominalnymi widmami T3/T4P jako opcją rezerwową.

Ścieżka odbicia DAQ automatycznie rozpoznaje **promieniowanie padające o dopasowanym znaczniku czasu**na podstawie zarejestrowanego pliku**`.daq`**(DAQ-U/M/E)**lub natywnego pliku DAQ-M `.csv`**znalezionego wraz z obrazami. Jeśli pakiet kalibracyjny dla konkretnej kamery lub urządzenia DAQ nie jest przechowywany lokalnie w pamięci podręcznej, potok**automatycznie pobiera go z AWS** przy pierwszym użyciu (wymaga jednorazowego połączenia z Internetem; zapisywany w pamięci podręcznej pod nazwą `~/.chloros/`).

#### Odczyt pikseli odbicia (Pix4D / Metashape / własne skrypty)

Odbicie jest przechowywane jako całkowita liczba DN, a **wartość DN odpowiadająca ρ = 1,0 zależy od kamery źródłowej**:

| Źródło | ρ = 1,0 to | Jak to rozpoznać |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (zakres do ρ 2,0) | Plik zawiera adnotację XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (ograniczony do ρ 1,0) | Brak tagów XMP `Chloros:*` — ten brak *jest* sygnałem. |

**Odczytaj wartość `Chloros:PixelScale` i podziel przez nią** zamiast zakładać stałą wartość. Tag jest zdefiniowany w domenie uint16, więc pozostaje niezmienny `32768` we wszystkich formatach wyjściowych, które zmieniają skalę — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` i `TIFF (32-bit, Percent)` są samopisujące (najpierw normalizuj przechowywany typ danych z powrotem do uint16: ×257 z 8-bitowego, ×65535 z typu float).

> **Jeden przypadek, zgodnie z założeniami, nie zawiera skali.** Gdy dane źródłowe 8-bitowe (BayerRG8) są zapisywane jako 8-bitowe TIFF, potok *przycina* wartości do zakresu 0..255 zamiast dokonywać przeskalowania, więc każda wartość powyżej ρ≈0,008 zostaje spłaszczona do 255 i plik nie zawiera informacji o skali. Chloros celowo pomija zarówno `Chloros:PixelScale`, jak i krotkę `MicaSense:RadiometricCalibration` w tym miejscu i rejestruje powód. **Jeśli tag ten nie występuje w pliku odbicia LATTICE, nie należy zakładać skali — należy ponownie wyeksportować w trybie 16-bitowym lub 32-bitowym** zamiast dzielić piksele, które nigdy nie były podzielne.

#### Dane EXIF przeniesione do eksportu

`process` kopiuje **blok GPS i jego ExifIFD** z oryginalnego zdjęcia do każdego produktu, więc
eksport zawiera `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` oraz
`CameraSerialNumber` wraz z danymi georeferencyjnymi.

**`FocalLength` nie jest opcjonalny w przypadku fotogrametrii.** Pix4D oblicza odległość próbkowania terenu na podstawie
odległości próbki naziemnej na podstawie ogniskowej i wysokości; w przypadku braku tego tagu program stosuje znacznie błędną skalę. Podczas jednego
lotu nad gajem pomarańczowym, obejmującego 49 ujęć, brak tego tagu spowodował przekształcenie terenu o wymiarach 411 m × 160 m w zrekonstruowany
obszar o wymiarach 47,8 km × 13 km — ortofotomapę o rozdzielczości 455 MP składającą się głównie z danych „nodata”, co następnie zostało zinterpretowane jako problem z układaniem kafelków i
problem z plikiem BigTIFF, zanim ktokolwiek sprawdził GSD. Jeśli twoja ortofotomapa ma nieprawdopodobną
skalę, najpierw uruchom program `exiftool -FocalLength` na wyeksportowanym produkcie.

Kopia ta celowo **nie** `-all:all`: tagi strukturalne IFD0 zakłócają wynik LATTICE podczas
kopiowania, a pliki `ExifImageWidth` / `ExifImageHeight` zostały wykluczone, ponieważ opisują
*źródłowe* ujęcie — eksport, którego rozmiar kiedykolwiek został zmieniony, zawierałby w przeciwnym razie wymiary
sprzeczne z własną siatką. Plik XMP jest zapisywany bezpośrednio, a nie kopiowany, ponieważ ExifTool
usuwają tagi XMP z tego samego wywołania podczas kopiowania bloku XMP (co spowodowałoby utratę tagów kalibracyjnych MAPIR
).

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

Folder aparatu to `LATT-<sensor>-<lens>-F<filter>` dla LATTICE (zgodny z EXIF
`Model`), a dla Survey3 — dwa aparaty, które mają ten sam czujnik i filtr, ale różnią się
obiektywem, mają oddzielne drzewa katalogów, ponieważ różnią się winietowaniem, polem widzenia i zniekształceniami. Folder formatu
ma nazwę `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32` dla
`TIFF (32-bit, Percent)`.

> **Każdy wyeksportowany produkt zachowuje nazwę pliku ŹRÓDŁOWEGO.** Eksport radiancji z
> `capture_…_raw.tif` nadal nosi nazwę `capture_…_raw.tif` — po prostu znajduje się w
> `tiff32/Radiance_Images/`. **To folder identyfikuje produkt, a nie nazwa pliku**, więc wyszukiwanie
> `*radiance*.tif` nie daje żadnych wyników; zamiast tego należy dopasować katalog.

### Nagrania z czujnika światła — skalibrowane `.daq` + `.csv`

`process` obsługuje również zapisy `.daq` znajdujące się w folderze wejściowym i **nie**
wymaga do tego żadnych obrazów: samodzielnie uruchomiony moduł DAQ-U / DAQ-M / DAQ-E używane samodzielnie zapewniają pełną
rejestrację, a folder zawierający wyłącznie pliki `.daq` stanowi prawidłowe dane wejściowe.

Rejestracja z DAQ jest możliwa **bez** jego kalibracji — właśnie to zapewnia publicznie dostępny
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) rejestratory
(`record_daq.py`) domyślnie: zapisują surowe odczyty czujników i oznaczają plik tak, aby
Chloros pobrał fabryczną kalibrację tego czujnika **przez port szeregowy** (najpierw z lokalnej pamięci podręcznej,
a następnie z chmury MAPIR) i ją stosuje. `process` zapisuje wynik z powrotem:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Plik `.csv` zawiera jeden wiersz na każdy odczyt: sygnaturę czasową UTC, czas integracji, moc całkowitą,
luksy fotopowe/skotopowe, PPFD (oraz jego podział na niebieski/zielony/czerwony), szczytową długość fali, a następnie
pełne widmo w siatce długości fal właściwej dla czujnika. Plik `.daq` jest ponownie importowany bez konieczności
ponownej kalibracji.

W przypadku powodzenia przebieg generuje raport `Light-sensor products written: N (calibrated .daq + .csv)`.
Tekst w nawiasie opisuje to, co faktycznie zostało zapisane, więc dla czujnika bez pakietu brzmi to
`(RAW COUNTS — this sensor has no calibration bundle)`, a
`(N calibrated, M raw counts)` w przypadku folderu zawierającego oba elementy. Nagłówki własnego backendu
`[DAQ-EXPORT]` i `[RUN-SUMMARY]` czerpią swoje brzmienie w ten sam sposób — żaden z
tych trzech nie może nazwać surowego eksportu kalibrowanego.

Zapis DAQ-U / DAQ-M / DAQ-E, którego pakiet kalibracyjny nie może zostać pobrany — jesteś
poza siecią lub dla tego czujnika nie ma kalibracji w pliku — jest **pomijany z podaniem przyczyny** w
wierszu `[DAQ-EXPORT]`, nigdy nie jest zapisywany jako plik „skalibrowany” zawierający surowe odczyty.
Połącz się z internetem i uruchom ponownie. Przyczyna to ta, którą czytnik faktycznie
ustalił dla tego pliku (nieczytelny schemat, brak pakietu, błąd zapisu), a podsumowanie
przebiegu wymienia **oddzielne** przyczyny — dwadzieścia pominiętych plików z jednej przyczyny jest traktowanych jako jedna
przyczyna, a nie dwadzieścia powtórzeń tej samej przyczyny.

#### Eksport nagrań DAQ-A jako surowe zliczenia

Rodzina **DAQ-A** powstała przed wprowadzeniem systemu pakietów przypisanych do poszczególnych numerów seryjnych i nie posiada pakietu kalibracyjnego,
który można by pobrać — zamiast tego jest kalibrowana w terenie względem wzorca odbicia, co jest
powodem, dla którego nigdy go nie potrzebowała. Odrzucenie tych nagrań pozbawiło ich możliwości uzyskania
w ogóledanych, więc są one eksportowane pod **inną nazwą**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Inna nazwa pliku zamiast flagi wewnątrz pliku, ponieważ informacja ta musi przetrwać
wysyłkę e-mailem jako sama nazwa. Nagłówek `.csv` zawiera informację
`raw spectral sensor counts (NOT irradiance)` i ostrzega, że wartości są porównywalne
**w obrębie** pliku — co jest dokładnie tym, do czego wykorzystuje je kalibracja oparta na celach — a
nie między czujnikami. Kolumny fotometryczne zależne od mocy (moc całkowita, luks fotopowy i
skotopowy, PPFD) są zapisywane jako **NULL** zamiast być zintegrowane na podstawie zliczeń, a podsumowanie
przebiegu wskazuje `RAW COUNTS`, więc dane „wyeksportowane” do dziennika nie mogą być odczytywane jako natężenie promieniowania.

Starsze zapisy **v1.01 / v1.02** (zapisywane przez DAQ-A-SD) nie zawierają epoki dla poszczególnych odczytów,
a jedynie czas zapisu pliku. Moduł dopasowujący obraz↔promieniowanie padające nadal je odrzuca — dopasowanie
klatki do czasu zapisu byłoby niewidocznym błędem — ale eksportator je odczytuje, a
CSV wyświetla `clock=daq_created_on`, dzięki czemu produkt wskazuje, na którym zegarze działa.

### Uwagi

- `process` automatycznie wykrywa, czy folder ma nazwę Survey3, LATTICE czy mieszany.
- Postęp jest przesyłany za pośrednictwem Server-Sent Events; CLI pokazuje na żywo postęp dla poszczególnych wątków (wykrywanie, analiza, przetwarzanie, eksport).
- W przypadku Linux/Jetson, CLI sprawdza pamięć wymiany i może wyświetlić ostrzeżenie przed przetwarzaniem dużych folderów. Algorytm debayeringu uwzględniający tekstury automatycznie nakłada również ograniczenie częstotliwości GPU na urządzenia Jetson o niskim poborze mocy (Nano, Orin Nano).
- W przypadku powodzenia proces informuje, ile plików obrazów zostało zapisanych (`Image products written: N`).

#### Proces, który nie zapisuje żadnych obrazów, kończy się niepowodzeniem

Jeśli zażądano plików wyjściowych, a proces nie zapisał **żadnego** — jedynie `project.json` i
`calibration_data.json` — `process` traktuje to jako niepowodzenie: wyświetla komunikat
`Processing finished but wrote no image products.` i **kończy działanie z wynikiem niezerowym**, dzięki czemu skrypt może
to wykryć. Komunikat podaje nazwę folderu projektu oraz typowe przyczyny:

- folder wejściowy nie został rozpoznany jako zestaw zdjęć (sprawdź układ i `--input-level`) lub
- wszystkie żądane produkty zostały pominięte jako nieodpowiednie dla tych kamer (np. żądanie
  radiancji/odbicia od kamer obsługujących wyłącznie RGB).

Uruchom ponownie- uruchom z `--verbose` i sprawdź dziennik zaplecza pod kątem wierszy `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
które wyjaśniają pominięcia poszczególnychkamery, które w innym przypadku nie trafiają do danych wyjściowych CLI.

Celowe uruchomienie wyłącznie z metadanymi — wszystkie produkty wyłączone i brak `--indices` — nadal jest
**sukces**, ponieważ pusty wynik obrazu jest w tym przypadku prawidłowym rezultatem.

Podobnie jest w przypadku **przebiegu wyłącznie z czujnikiem światła**: folder z nagraniami `.daq` z definicji nie zawiera obrazów do eksportu
, a przebieg jest oceniany na podstawie skalibrowanych plików `.daq` / `.csv`, które zostały zapisane w jego miejsce.

---

## `chloros-cli login`

Uwierzytelnij to urządzenie za pomocą konta w chmurze Chloros+. Dane uwierzytelniające są bezpiecznie przechowywane w pamięci podręcznej w pliku `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Przykłady

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (usuwając fragmenty hasła lub powielając jego części). W przypadku błędu 401 CLI automatycznie ponawia próbę, dodając na końcu `$$`, a następnie używając połowy hasła bez powtórzeń; jeśli ponowna próba zakończy się powodzeniem, program zaloguje użytkownika i wyświetli poprawny składniowy znak cudzysłowu , której należy użyć następnym razem.

> **Użycie w trybie bezinterfejsowym/skryptowym: brak sesji w pamięci podręcznej oznacza interaktywny wiersz poleceń, a nie szybką porażkę.** Każde polecenie uruchamiające proces w tle (`process`, `status`, `export-status`, `time-sync`, …) uruchomione bez buforowanej licencji/sesji przechodzą do interaktywnego wiersza poleceń `Email:` / `Password:` na stdin przed kontynuowaniem. Zadanie wykonywane w trybie bezobsługowym bez sesji w pamięci podręcznej zostanie zatem zawieszone w oczekiwaniu na dane wejściowe — przed zaplanowaniem pracy w trybie bezobsługowym należy uruchomić polecenie `chloros-cli login EMAIL PASSWORD` raz na każdym komputerze.

---

## `chloros-cli logout`

Czyści sesję z pamięci podręcznej i wymusza nowe logowanie przy następnym wywołaniu.

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

Sprawdza bieżący postęp eksportu Thread-4. Wywołanie jest bezpieczne **podczas** działania `process` z innej powłoki.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Ustawia język wyświetlania CLI (obsługiwanych jest 38 języków, w tym CJK, RTL i języki indyjskie). Na starszych konsolach, które nie obsługują danego alfabetu, płynnie przełącza się na język angielski.

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

---

## `chloros-cli update`

Linux / Tylko dla Jetson. Sprawdza `version_url` z `/etc/chloros/update.conf` i proponuje pobranie + zainstalowanie pasującego pliku `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

W przypadku Linux/Jetson program CLI uruchamia również **automatyczne sprawdzanie aktualizacji przy każdym uruchomieniu** (nieblokujące, nigdy nie opóźnia wykonania polecenia): odczytuje `/etc/chloros/update.conf`, buforuje wynik na 1 godzinę w `~/.chloros/update_cache.json`, i wyświetla `Update available: vX.Y.Z / Run: chloros-cli update`, gdy dostępna jest nowsza wersja. W przypadku jakiegokolwiek błędu oraz na Windows operacja jest pomijana bez komunikatu.

---

## `chloros-cli selftest`

Przeprowadza 7-etapowy test sprawdzający: wersja, dostępność portu, uruchomienie backendu, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), obecność modelu odszumiającego, gotowość CUDA i odszumiającego.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stan i sterowanie serwerem głównym PTP. Host Chloros pełni rolę grandmastera PTP; kamery LATTICE i urządzenia DAQ-E działają jako urządzenia podrzędne w celu uzyskania znaczników czasu między urządzeniami.

| Podpolecenie | Opis |
| --- | --- |
| `status` | Wyświetl stan serwera głównego, priorytety BMCA, tożsamość zegara. |
| `peers` | Wyświetl listę urządzeń podrzędnych wykrytych za pomocą Delay_Req (kamery + czujniki DAQ-E). |
| `cameras` | Stan PTP dla poszczególnych kamer (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Ponowne uruchomienie procesu grandmaster. |
| `set-priority --priority1 N --priority2 N` | Zastąpienie priorytetów BMCA. |

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

Sterowanie kamerami LATTICE. Każde podpolecenie jest kierowane przez backend Chloros; backend ten zarządza pulą kamer, więc kolejne wywołania CLI ponownie wykorzystują ten sam otwarty uchwyt.

### Opcje wspólne (wykorzystywane przez większość podpoleczeń)

| Flaga | Opis |
| --- | --- |
| `-d, --device N` | Indeks kamery (domyślnie: 0). |
| `-s, --serial SN` | Określony numer seryjny; zastępuje `--device`. |
| `--serials SN1,SN2,…` | Numery seryjne oddzielone przecinkami dla pracy z wieloma kamerami. |
| `--all` | Działa na każdej wykrytej kamerze. |
| `--exposure US` | Czas naświetlania w mikrosekundach. |
| `--gain DB` | Wzmocnienie w dB. |
| `--pixel-format FMT` | np. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Wymiary obrazu. |
| `--preset {default,high_quality,high_speed,triggered}` | Zastosuj gotowy zestaw ustawień. Wszystkie działają w trybie swobodnym z wyjątkiem `triggered`, który uzbraja kamerę na sygnał zbocza sprzętowego na linii 2 — jeśli nic nie steruje tą linią, będzie czekać w nieskończoność zamiast wykonać przechwycenie. |
| `-o, --output DIR` | Katalog wyjściowy (domyślnie: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Rozmiar pakietu GVSP. `auto` uruchamia sondy ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Procedura pierwszego połączenia z kamerą LATTICE

```bash
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
#      - Auto-shrink frame size to fit the wire
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
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analiza systemu hosta pod kątem optymalnej konfiguracji kamery. Polecenie `--no-discover` pomija wykrywanie kamer (szybsza analiza wyłącznie na podstawie karty sieciowej). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Sprawdź/popraw ustawienia karty sieciowej; oszacuj przepustowość/liczbę klatek na sekundę. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Możliwości sieciowe zaplecza o stabilnej strukturze+ zalecenia dotyczące macierzy (zwraca `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` zachowuje żądaną rozdzielczość, ale ogranicza docelową liczbę klatek na sekundę — odczytaj `recommended.recommended_target_fps` i przekaż go jako cel połączenia; potraktuj to jako sukces, a nie błąd. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analiza scenariuszy bez uruchamiania kamer. **`--n-active` to łączna liczba kamer w sieci, a nie tylko w tej tablicy**— zgłaszaj ten błąd, gdy kamery autonomiczne przesyłają strumień jednocześnie lub gdy budżet sieci jest obliczany na podstawie zapotrzebowania, które nie uwzględnia wszystkich kamer (domyślnie: `len(--models)`). Zawsze wyświetla zagregowane wiersze `Wire budget:` (wymagane MB/s w porównaniu z pułapem zabezpieczającym przed kolizjami) i `Max cameras:` oraz sygnalizuje `** OVER-SUBSCRIBED**`, gdy macierz nadmiernie obciąża magistralę — zobacz [Model fps i burst macierzy](#array-fps--burst-model). |
| `lattice gpu` | Pokaż stan GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Sprawdź lub zaktualizuj oprogramowanie sprzętowe kamery. Lokalny wybór `.fwa` jest zablokowany: plik w `firmware/<MODEL_PREFIX>/` pasujący do `MIN_FIRMWARE_VERSION` kompilacji jest wgrywany, jeśli jest obecny (najwyższa wersja służy jedynie jako rozwiązanie awaryjne), więc nowszy obraz dostawcy przygotowany na dysku pozostaje nieaktywny, dopóki ten pin nie zostanie zmieniony — celowo nowsze wersje docierają do urządzeń poprzez podpisany manifest AWS, co jest preferowane, gdy jest nowszy. |
| `lattice presets [--apply NAME]` | Wyświetl lub zastosuj ustawienia wstępne kamery. |
| `lattice status` | Stan kamery na żywo. |

#### Przechwytywanie

| Podpolecenie | Cel |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Pojedyncza klatka. **Domyślnie zapisuje każdy typ eksportu** (`--processing all`); zobacz [Poziomy eksportu przechwytywania](#capture-export-levels-the-all-default). `--levels` zapisuje określony podzbiór (zastępuje `--processing`); `--force-daq` zapisuje przypisany odczyt DAQ jako plik pomocniczy `.daq` nawet w przypadku przechwytywania wyłącznie w formacie surowym. `--jpeg-quality` = JPEG jakość 1–100 (domyślnie 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Przesyłaj strumieniowo na dysk do momentu naciśnięcia Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Podgląd na żywo w formacie MJPEG w przeglądarce. `--ae-damping` ustawia tłumienie automatycznej ekspozycji (0,4–100). |

#### Dostrajanie czujnika

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Odczyt/zapis dowolnego węzła GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Ekspozycja i AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Wzmocnienie i automatyczne wzmocnienie. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Obszar zainteresowania (ROI) czujnika i binning. |
| `lattice format [--set FMT] [--list]` | Format pikseli. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Wyzwalacz sprzętowy/programowy. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (brak flag = jednorazowa korekcja balansu bieli) | Operacje WB. Tylko kamery RGB/Bayer; operacja bez działania (pomijana) w trybie mono M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB – potok kolorów wyświetlacza. `natural` (domyślnie) to ekonomiczne wykończenie na żywo; `enhanced` dodaje usuwanie fringingu + intensywność barw + lokalny kontrast CLAHE, zapewniając pełny wygląd zgodny z hub-parity przy około dwukrotnie wyższym koszcie wykończenia na klatkę, co skutkuje niższą **na żywo** — zapisane nagrania zawsze otrzymują pełne wykończenie niezależnie od tego. Tylko kamery RGB/Bayer; pomijane w przypadku kamery mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Wyświetlanie nasycenia/kontrastu (kamery z filtrem RGB). Pomijane w przypadku kamery mono M3M. |
| `lattice filter [--set NAME] [--list]` | Ustaw model filtra kamery(`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Węzły zasilania/termiczne sondy; przełączanie trybu czuwania o niskim poborze mocy. |

#### Kalibracja i czujniki

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibracja za pomocą tarczy odbijającej. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Wbudowane polecenia czujnika światła padającego. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Zastosuj korekcję winietowania do istniejących obrazów. |

#### Wiele kamer (sesje przejściowe)

| Podpolecenie | Cel |
| --- | --- |
| `lattice multi-info` | Wyświetl listę wszystkich kamer pełniących role synchronizacyjne. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Jedna zsynchronizowana klatka z każdej kamery. Domyślnie zapisuje **wszystkie typy eksportu**, gdy podłączona jest trwała tablica; tymczasowe rozwiązanie awaryjne bez tablicy jest**tylko debayeryzowane** (aby uzyskać pozostałe, należy najpierw uruchomić `array-connect`). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Strumieniowanie zsynchronizowanych klatek (tymczasowe). |
| `lattice multi-test [--count N]` | Test synchronizacji czasowej GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Automatyczne wykrywanie okablowania GPIO typu master/slave. |

#### Wyrównanie

| Podpolecenie | Cel |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — oraz pokrętła detektora/dopasowywania `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, pokrętła RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinacja wielu klatek `[--averaging mean\|median\|inlier_weighted]`, ograniczenia geometryczne `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, ograniczenia przestrzenne `[--roi X0,Y0,X1,Y1] [--mask PATH]` oraz nadpisania dla poszczególnych podwładnych `[--per-cam-override SN:KEY=VALUE]` (powtarzalne) | Oblicz profil dopasowania na podstawie obrazów z kamer na żywo. `--prefilter` domyślnie przyjmuje wartość `gradient` (mapa krawędzi; odpowiada dopasowywaczowi GUI/macierzy — krawędzie pozostają niezmienne w różnych pasmach spektralnych). `--matcher flann` sprawdza się przy ponad ~5000 cech; `--averaging median` jest odporny na jedno złe ujęcie, `inlier_weighted` waży według liczby dopasowań; `--lock-scale` rzutuje na najbliższy obrót (bez skalowania), `--lock-axis` zeruje jeden składnik translacji; `--mask` ma zastosowanie do każdej kamery (użyj `--per-cam-override` dla ustawień indywidualnych dla każdej kamery, np. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odrzuca zapis kalibracji, której RMS reprojekcji przekracza próg. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Przechwytuje jedną wyrównaną klatkę wielopasmową. `--bit-depth` domyślnie dopasowuje się do kamery; `--no-crop` zachowuje pełną klatkę (wypełniając czarną przestrzenią); `--interpolation` (domyślnie `linear`) oraz `--border-mode`/`--border-value` (domyślnie `constant`/0) sterują zniekształceniem procesora — ścieżka procesora graficznego jest w każdym przypadku dwuliniowa. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Wielopasmowe ramki wyrównane strumieniowo (te same regulatory warpu co `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Wyświetl szczegóły profilu. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Zmień kolejność warstw. |

#### Indeks / Obliczenia dotyczące roślinności

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Pełny zestaw flag: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (powtarzalne), `--capture-level raw|debayered|radiance|reflectance|unknown` (zastępuje poziom przechwytywania zapisany w źródle TIFF; domyślnie: odczyt z metadanych TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. W przypadku `--live` pokrętła wyrównania zniekształceń mają również zastosowanie do: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` uwzględniają wielkość liter.** Strona symbolu musi dokładnie odpowiadać nazwom kanałów presetu (presety używają małych liter, np. NDVI = `red`, `nir` — sprawdź `--list-presets`), a strona pasma strona pasma musi odpowiadać nazwie pasma w wyrównanym stosie (lub być indeksem pasma liczonym od 0 w trybie offline). `--channel red=Red_660 --channel nir=NIR_850` działa; `--channel RED=660` kończy się niepowodzeniem z błędem `channel_map missing entries`.

#### Trwałe połączenia (Smart-Prep, przepływ odpowiadający interfejsowi graficznemu)

Polecenia te utrzymują kamery otwarte w puli zaplecza podczas kolejnych wywołań CLI.

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Dodaj jedną kamerę do puli (pojedyncza kamera, bez macierzy). |
| `lattice cam-disconnect [--serial SN] [--all]` | Zwolnij. |
| `lattice cam-list` | Wyświetl listę kamer w puli. |
| **`lattice array-connect`**|**Podłączenie trwałej, zsynchronizowanej macierzy (zalecany punkt wejścia).** Uruchamia pełny proces przygotowania w interfejsie graficznym. |
| `lattice array-disconnect [--array-id ID] [--all]` | Zwolnienie macierzy. |
| `lattice array-list` | Wyświetl listę podłączonych macierzy. |
| `lattice array-status [--array-id ID]` | Liczba klatek na sekundę na żywo, PTP, ostatni błąd. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno zsynchronizowane przechwycenie z macierzy działającej na żywo — pojedyncze / ciągłe / w odstępach / najszybsze. **Domyślnie `all`** (jeden plik na każdy odpowiedni typ eksportu na każdą kamerę). Pominięte kamery (np. RGB wykluczone z pomiaru promieniowania/odbicia) są zgłaszane z kodem `Skipped: SN:<serial> (<reason>)`; odczyt z układu DAQ użyty do pomiaru odbicia jest zapisywany osobno i zgłaszany z kodem `DAQ: <path>`. Zobacz [Tryby przechwytywania, rejestratory i przetwarzanie offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Rejestruje podgląd połączonego indeksu na żywo w formacie wideo/GIF (klasa monitorowania; wymaga otwartego strumienia połączonego). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Seria zdjęć w formacie surowym Bayer o wysokiej liczbie klatek na sekundę (klasa analityczna; przetwarzanie w trybie offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Przetwarzanie zapisanej serii zdjęć w formacie surowym na skalibrowane wideo. |

##### Opcje `array-connect`

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | auto-wykryj wszystkie kamery LATTICE (wymagane ≥2) | Pierwsza kamera w szeregu jest MASTEREM. W przypadku pominięcia tej opcji wykrywanie ogranicza się do modeli LATTICE (`TRI032*`) i łączy je wszystkie. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linia synchronizacji GPIO. |
| `--target-fps F` | auto | Częstotliwość wyzwalania urządzenia MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Zastąpienie selektora warstwy. |
| `--wire-ceiling-mbps MB_PER_S` | wykrywane automatycznie | **Trwała przepustowość łącza hosta w MB/s — wartość, od której zależy przydział całej macierzy.** Należy ją zmniejszyć, gdy macierz zgłasza uszkodzone ramki GVSP: wartość automatyczna jest wyliczana na podstawie ogłaszanej przez kartę sieciową prędkości łącza, która zawyża wartości dla adapterów USB, wąskich pasm PCIe i obciążonych struktur współdzielonych. Jest ona zapisywana w bloku przechwytywania macierzy projektu, więc ponowne otwarcie / CLI / SDK przywróci tę wartość. Zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Grupowanie sprzętowe. |
| `--no-recommend` | wył. | Pomiń etap analizy sieci. |
| `--no-ptp` | wył. | Wyłącz PTP (znaczniki czasu między kamerami **nie** będą wtedy porównywalne). |

### Smart-AE / Smart-Capture

Matryce LATTICE uruchamiają ciągłą regulację ekspozycji (AE) w tle zaraz po podłączeniu, ale nowo ustawiona scena potrzebuje chwili, by osiągnąć stabilność. `array-capture --smart` to **wygodne rozwiązanie**: czeka, aż regulacja ekspozycji ustabilizuje się we wszystkich kamerach w matrycy, a następnie uruchamia przechwytywanie. Używaj go, gdy zmieniasz scenę w trakcie sesji.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Domyślna polityka ustabilizowania jest konserwatywna: limit czasu 5 s, okno stabilności 1,5 s, tolerancja rozrzutu ekspozycji ±5 %. Jeśli potrzebujesz innego zachowania automatyzacji, dostosuj ustawienia za pomocą SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

### Poziomy eksportu ujęć (ustawienie domyślne `all`)

Począwszy od tej wersji, `lattice capture`, `lattice multi-capture`, oraz `lattice array-capture` **domyślnie ustawione na `--processing all`** — jeden zapisany plik na typ eksportu, który ma zastosowanie do każdej kamery, zgodnie z zachowaniem opcji „Capture All” (Zrób wszystkie zdjęcia). Poziomy są następujące:

| Poziom | Wynik | Dotyczy |
| --- | --- | --- |
| `raw` | Jednokanałowy Bayer (kamery monochromatyczne: pojedyncze pasmo) bezpośrednio z czujnika. | Wszystkie kamery. |
| `debayered` | 3-kanałowa demosaika BGR (kamery monochromatyczne: 1-kanałowa skala szarości). | Wszystkie kamery. |
| `radiance` | float32 W/m²/sr/nm poprzez pełny łańcuch radiometryczny. | Tylko wielospektralne (M3C/M3M) — **pomijane w przypadku kamer z filtrem RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Gotowe do Pix4D. | Tylko wielospektralne i **tylko wtedy, gdy powiązany jest moduł DAQ + kamera jest skalibrowana**; w przeciwnym razie pomijane. |
| `preview` / `display` | Pełny łańcuch podglądu GUI (CCM + WB + gamma zgodnie z profilem kamery). `lattice capture` nazywa to `preview`; `array-capture`/`multi-capture` użyj `display`. | Wszystkie kamery. |

Przekaż pojedynczy poziom, aby zapisać tylko ten jeden (`--processing debayered`). Gdy żądasz `all`, poziomy, które nie mają zastosowania do danej kamery, są pomijane (i zgłaszane), a nie traktowane jako błąd — kamera niepodłączona lub nieskalibrowana nadal otrzymuje `raw` / `debayered` / `preview`.

W przypadku każdej klatki odbicia faktycznie wykorzystany odczyt promieniowania padającego z systemu DAQ jest zapisywana w pliku **`.daq`** dołączonym do obrazu (aby umożliwić ponowne przetworzenie zapisu w późniejszym czasie) i jest zgłaszana w wierszu `DAQ:`.

### Jak wygląda folder z przechwyconymi danymi

Każdy typ eksportu trafia do **własnego podfolderu** w ramach `-o`, dzięki czemu w wielopoziomowym przechwycie nigdy nie mieszają się różne typy:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` to sygnatura czasowa przechwycenia, a `<serial>` to numer seryjny kamery, więc jedna zsynchronizowana grupa ma wspólną
sygnaturę czasową dla wszystkich kamer. **Należy zwrócić uwagę na jedną asymetrię:** poziom `display` jest przechowywany w folderze
o nazwie `preview/`, podczas gdy same pliki zachowują `_display` w nazwie — folder i rozszerzenie różnią się
tylko dla tego poziomu. Nieznane poziomy są umieszczane w folderze o własnej nazwie, a jeśli podfolder
nie może zostać utworzony, plik jest zapisywany w katalogu głównym wyjściowym, zamiast zostać utracony.

**Ponowne przetwarzanie folderu „captures”:**skieruj `chloros-cli process` na**katalog główny przechwyconych obrazów**
(`output/`). `process` zazwyczaj importuje tylko folder, który nazwiesz, ale gdy ten folder nie zawiera żadnych
obrazów, a ma podfoldery, automatycznie przechodzi do nich — dzięki czemu podfoldery na każdym poziomie oraz
katalog główny `.daq` są pobierane za jednym razem. Każdy poziom przechwycenia jest importowany jako pojedynczy obraz, a
pozostałe poziomy są dostępne jako tryby, a nie jako jeden obraz na poziom.

Bezpośrednie podanie nazwy **podfolderu poziomu** (np. `output/raw/`) również działa. Spowoduje to pominięcie katalogu głównego
`.daq`, więc należy skopiować lub wskazać odczyt z DAQ obok niego podczas ponownego wyznaczania produktu radiometrycznego
z `raw/` — w przeciwnym razie nie będzie z czym dopasować sygnatury czasowej.

**Przetwarzanie zawsze rozpoczyna się od `raw`.** W ramach każdego przechwycenia źródłem potoku danych jest surowa klatka;
`debayered`, `radiance`, `reflectance` oraz `preview` występują jako tryby wyświetlania, ale nigdy nie są przekazywane
z powrotem przez potok przetwarzania. Ponowne przetwarzanie produktu pochodnego spowodowałoby ponowne zastosowanie winietowania, CCM oraz
obliczeń radiancji, które są już wbudowane w jego pikselach, dlatego Chloros rezygnuje z tego, zamiast
przeprowadzać podwójne przetwarzanie. Dwie konsekwencje, o których warto wiedzieć:

- Rendery `index/` i `composite/` **nigdy** nie są przetwarzane. Są to dane wyjściowe, a nie przechwyty —
  renderowanie LUT NDVI nie ma sensownej interpretacji promieniowania.
- Folder przechwyty wyeksportowany **bez** `raw` (np. `array-capture --processing reflectance`) nie ma
  żadnego prawidłowego źródła w potoku. Te przechwyty importują się i wyświetlają normalnie, ale `process` pomija
  je i informuje o tym:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Jeśli naprawdę musisz przesłać produkt pochodny przez — sesję hubu przechwyconą przy
  włączonym `demosaic` lub folder starszej generacji — `--input-level {raw,debayered,processed}` wymusza ten wpis
  i zastępuje pominięcie. Ta flaga stanowi celowe wyjście awaryjne; `auto` (ustawienie domyślne)
  nigdy nie przetwarza przechwycenia, które nie zawiera danych surowych.

### Pominięte przechwycenia w mieszanychukładach filtrów

Gdy w jednym układzie mieszają się kamery RGB i wielospektralne, `array-capture --processing radiance` (lub `reflectance`) zapisuje klatki wielospektralne i **pomi**kamery RGB — promieniowanie na piksel Bayera nie ma znaczenia w przypadku czujnika szerokopasmowego. Program CLI wyświetla każdy zapisany plik (wraz z poziomem eksportu), każdy zapisany plik `.daq` oraz każde pominięcie wyraźnie, więc liczba plików nie jest zaskakująca:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeny powodów pominięcia mają postać `<level>-not-applicable-to-rgb-cam`. Odbicie światła może również zostać pominięte za pomocą `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, a także za pomocą `dls-uncalibrated-band-<nm>`, gdy pasmo leży w większości poza radiometrycznie skalibrowanym zakresem czujnika światła DAQ (~374–974 nm) — spośród dostępnych w sprzedaży jedynym modelem jest F988, który obsługuje proces z wykorzystaniem panelu reflektancji.

Aby uwzględnić wszystkie kamery niezależnie od typu filtra, należy użyć kodu `--processing debayered` (lub `display`)lub domyślnego `all`, aby uzyskać wszystkie odpowiednie poziomy dla każdej kamery za jednym razem.

---

## Tryby przechwytywania, rejestratory i przetwarzanie offline

Wszystkie one działają na **trwałej tablicy** (najpierw uruchom `array-connect`). Odzwierciedlają one panel przechwytywania w interfejsie graficznym.

### Tryby `array-capture`

`array-capture` to pojedyncze polecenie z czterema trybami migawki oraz zestawem opcji eksportu doggles:

| Tryb | Flaga | Zachowanie |
| --- | --- | --- |
| **Pojedynczy** *(domyślny)* | (brak) | Jedna zsynchronizowana grupa przechwytywania, a następnie zakończenie. |
| **Ciągły** | `--continuous` | Kolejne przebiegi jeden po drugim, aż do `Ctrl+C`, `--count N` lub `--duration S`. |
| **Interwał** | `--interval S` | Jeden przebieg co `S` sekund (liczone od początku każdego przebiegu), te same granice. |
| **Najszybszy** | `--fastest` | Tylko dane surowe + przypisany odczyt z akwizycji danych + kompozyt z indeksem łącznym; pomija obliczenia dotyczące promieniowania/odbicia/wyświetlania, dzięki czemu klatka jest wyświetlana szybko. Oznacza `--processing raw --force-daq`. Zapisane dane `.daq` należy później ponownie przetworzyć na produkty skalibrowane. |

Przełączniki eksportu (można łączyć z dowolnym trybem; wszystkie korzystają z tego samego interfejsu graficznego/punktu końcowego SDK):

| Flaga | Efekt |
| --- | --- |
| `--processing LEVEL` | Pojedynczy poziom eksportu lub `all` (domyślnie). |
| `--levels L1,L2,…` | Wyraźny podzbiór typów eksportu (np. `raw,radiance,reflectance`); **zastępuje `--processing`**. |
| `--aligned` / `--no-aligned` | Zastosuj transformację do wszystkich eksportów niebędących danymi surowymi w ramach [profilu wyrównania](#alignment) (współzarejestrowanego). Dane surowe pozostają bez przekształcenia, ale zawierają transformację w metadanych. W przypadku braku profilu tablicy stosuje się domyślne ustawienie niewyrównane (z ostrzeżeniem). |
| `--index` / `--no-index` | Zapisuj / pomijaj nakładkę wskaźnika roślinności dla poszczególnych kamer, jeśli została skonfigurowana. Domyślnie: renderuj ją. |
| `--force-daq` | Zapisz przypisany odczyt DAQ/DLS jako plik towarzyszący `.daq`, nawet jeśli żaden z wybranych poziomów tego nie wymaga (np. w przypadku przechwytywania wyłącznie danych surowych), aby klatki można było ponownie przetworzyć na odbicie/indeks w trybie offline. |
| `--smart` | Przed wyzwoleniem poczekaj, aż AE ustabilizuje się na wszystkich kamerach (zobacz [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresja pikseli TIFF. `deflate` (domyślnie) = bezstratna kompresja zlib L1 + predyktor poziomy, ~4,1 MB na klatkę w pełnej rozdzielczości; `none` = bez kompresji, ~5× szybszy zapis przy ~6,3 MB na klatkę — używaj w celu uzyskania maksymalnej stałej szybkości, gdy pozwala na to dysk. Oba są bezstratne i odczytują się identycznie podczas importu. |

> **Pojedynczyzapis TIFF + model zapewnający stałą szybkość.**Ujęcia są zapisywane w**jednym**przebiegu pliku TIFF zawierającym piksele + XMP + IFD0 Marka/Model (zmierzono na pełnej rozdzielczości Mono12: 36 ms skompresowane / 6,5 ms nieskompresowane, w porównaniu z ~148 ms dla starego sposobu „zapisz, a potem przepisz za pomocą ExifTool”); jedyna pozostała praca ExifTool (dopracowanie pod-IFD EXIF) działa w asynchronicznym procesie w tle, a klatka jest kompletna i gotowa do importu, nawet jeśli ten proces nigdy się nie uruchomi. Należy pamiętać, że kompresja DEFLATE blokuje GIL Python, więc zapisy skompresowane**nie**równolegle w wątkach zapisujących dla poszczególnych kamer — ciągłe przechwytywanie w pełnej rozdzielczości z 8 kamer z prędkością czujnika (~10,4 kl./s) wymaga `--compression none`**oraz** dysku klasy NVMe (~500 MB/s ciągłego zapisu). Ten sam parametr jest dostępny jako `compression` w `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — połączony indeks wideo/GIF (klasa monitoringu)

Rejestruje wszystko, co wyświetla się w **widoku połączonego indeksu na żywo**** na `.avi` (oraz opcjonalnie na `.gif`). Ponieważ pobiera sygnał z kompozytowego podglądu na żywo, strumień złożony musi być otwarty (tj.g. tablica jest wyświetlana w podglądzie w interfejsie graficznym), aby klatki mogły się pojawić. Co 2 s sprawdza postęp i zatrzymuje się na `--duration`, `Ctrl+C` lub gdy rejestrator samoczynnie.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Flaga | Domyślna | Opis |
| --- | --- | --- |
| `--array-id ID` | tylko macierz | Macierz docelowa (pominąć, jeśli podłączona jest tylko jedna). |
| `-o, --output DIR` | `output` | Katalog wyjściowy (lokalny dla backendu). |
| `--fps F` | `10` | Liczba klatek na sekundę podczas nagrywania. |
| `--duration S` | do naciśnięcia Ctrl+C | Automatyczne-zatrzymanie po `S` sekundach. |
| `--gif` | wyłączone | Zapisz również animowany plik GIF. |
| `--gif-only` | wyłączone | Zapisz tylko plik GIF (bez `.avi`). |

### `array-burst` — seria zdjęć w formacie surowym Bayer o wysokiej częstotliwości klatek (klasa analityczna)

Odczytuje zsynchronizowany— **bez łańcucha kalibracji, bez exiftool, bez konieczności podglądu na żywo** — dzięki czemu działa z pełną częstotliwością przechwytywania aparatu. Zapisuje surowe klatki + manifest dla każdej klatki + jeden plik `.daq` dla każdego odrębnego odczytu DLS w ramach `<output>/bursts/<base>/`. Przetwarzaj offline (następne polecenie) lub przekaż `--build`, aby wykonać to natychmiast po zatrzymaniu.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flaga | Domyślna | Opis |
| --- | --- | --- |
| `--array-id ID` | tylko tablica | Tablica docelowa. |
| `-o, --output DIR` | `output` | Katalog wyjściowy (seria trafia do `<DIR>/bursts/<base>/`). |
| `--duration S` | do naciśnięcia Ctrl+C | Automatyczne zatrzymanie po `S` sekundach. |
| `--max-frames N` | bez ograniczeń | Automatyczne zatrzymanie po `N` surowych klatkach. |
| `--build` | wyłączone | Po zatrzymaniu natychmiast ponownie przetworzyć serię (tak samo jak `array-build-video`). |
| `--products …` | `combined:index` | Z `--build`: które wideo() należy utworzyć (patrz poniżej). |
| `--fps F` | `10` | Wraz z `--build`: liczba klatek na sekundę w wyjściowym pliku wideo. |
| `--save-tiffs` | wyłączone | Wraz z `--build`: zapisuj również skalibrowane pliki TIFF dla poszczególnych klatek. |
| `--gif` | wył. | Z `--build`: zapisuj również animowane pliki GIF. |

### `array-build-video` — przetwarzanie offline zapisanej serii zdjęć

Dopasowuje czasowo każdą surową klatkę do najbliższego zapisanego odczytu `.daq` i przepuszcza ją przez **ten sam łańcuch indeksów promieniowania / odbicia /**jak w potoku importu**, renderując jeden lub więcej filmów.

`--products` to lista elementów `kind:level` oddzielonych przecinkami, gdzie `kind` ∈ `per_cam` | `combined` oraz `level` ∈ `radiance` | `reflectance` | `index`. Sam `level` (bez `kind:`) domyślnie przyjmuje wartość `per_cam`. Wartością domyślną jest `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flaga | Domyślna | Opis |
| --- | --- | --- |
| `--burst-dir DIR` | (wymagane) | Ścieżka do folderu burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, jak powyżej. |
| `--fps F` | `10` | Liczba klatek na sekundę (fps) w wyjściowym pliku wideo. |
| `--save-tiffs` | wyłączone | Zapisuj również skalibrowane pliki TIFF dla poszczególnych klatek wraz z plikiem wideo. |
| `--gif` | wył. | Zapisuj również animowane pliki GIF. |

> **Wybierz odpowiedni rejestrator.** `array-record` jest przeznaczony do *monitoringu* — rejestruje kompozycję na żywo tak, jak jest wyświetlana, i wymaga otwartego strumienia. `array-burst` → `array-build-video` jest przeznaczony do *analizy* — zapisuje surowe dane z czujnika z pełną częstotliwością, a następnie rekonstruuje skalibrowane filmy przedstawiające promieniowanie/odbicie/wskaźnik, bez konieczności podglądu na żywo.

### Kamery monochromatyczne (M3M) jednopasmowe

Seria **M3M**to monochromatyczny odpowiednik serii Bayer**M3C**: jedna wąskopasmowa filtracja interferencyjna na kamerę (`M3M-<lens>-F<wavelength>`, np. `M3M-L87-F685`), dzięki czemu czujnik dostarcza**pojedyncze pasmo skali szarości** bez mozaiki Bayera. Nie ma tu nic do demosaikowania, nie makanałów do rozdzielenia ani balansu bieli do ustawienia — cały proces przetwarzania kolorów w systemie RGB po prostu nie ma zastosowania.

Co to oznacza w przypadku modelu CLI:

- **modele `lattice white-balance`, `lattice color-profile`, `lattice color`**wykrywają kamerę mono i**pomijają ją, wyświetlając jednowierszowy komunikat**, zamiast narzucać bezsensowne ustawienia. Nadal działają normalnie w tej samej sesji z kamerą RGB/Bayer M3C.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** nadal działają — promieniowanie i odbicie są mapami radiometrycznymi *dla poszczególnych pasm* i są doskonale zdefiniowane dla jednego pasma. Klatki monochromatyczne zawierają **tożsamościową** macierz odpowiedzi czujnika (bez demiksowania 3×3), więc płaszczyzna przechodzi przez obliczenia kalibracyjne w niezmienionej postaci.
- **Pojedyncza kamera mono nie może wygenerować wskaźnika wegetacji.**NDVI/NDRE/itp. potrzebne są co najmniej dwa pasma (np. Red + NIR). Aby uzyskać wskaźnik z sprzętu monochromatycznego, należy skierować**kilka** kamer M3M na różne długości fal, wyrównać je w jeden stos wielopasmowy i obliczyć wskaźnik *z tego*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Symbole `--channel` muszą **dokładnie*** odpowiadać nazwom kanałów w ustawieniach wstępnych* (rozróżniana jest wielkość liter; NDVI to małe litery `red`, `nir` — patrz `--list-presets`), a nazwa pasma pasma w wyrównanym stosie (tryb offline akceptuje również indeksy pasm zaczynające się od 0, np. `--channel red=0 --channel nir=1`).

Elementem rozróżniającym w całym stosie jest token `M3M` w ciągu modelowym (nigdy nie pojawia się on w ciągu `M3C`), wyświetlany w GUI/SDK jako `is_mono`.

---

## Konfiguracja i optymalizacja karty sieciowej hosta (macierze LATTICE)

Kamery LATTICE przesyłają strumień GVSP przez kartę sieciową hosta, więc w przypadku macierzy wielokamerowych **sterownik**karty oraz**rozmiar pierścienia odbiorczego** mają równie duże znaczenie jak prędkość łącza. Nieprawidłowe ustawienia są sygnalizowane jako `FRAMES WILL DROP` / `Reduce ROI to enable` w panelu ustawień macierzy (oraz w `lattice network-analysis` / `analyze_array_network()` w SDK), nawet jeśli same kamery działają prawidłowo.

### Karty sieciowe USB 10GbE — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

| Pozycja | Wymagana wartość | Dlaczego ma to znaczenie |
| --- | --- | --- |
| **Wersja sterownika**|**≥ v10.67 (styczeń 2026 r.)**, INF `rtump64x64sta.inf` | Starszy sterownik z**2016 r.**(wersja 10.65, `rtump64x64.inf`) nieprawidłowo obsługuje wyłączanie zasilania oraz generuje błędy**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**podczas wyłączania/restartowania/przechodzenia w tryb uśpienia. Przejście zawiesza się (limit czasu ~5 min), użytkownik jest zmuszony do przymusowego wyłączenia zasilania, a powtarzające się nieprawidłowe wyłączenia**uszkadzają repozytorium WMI**(PowerShell i narzędzia zaczynają zawodzić z błędem `Invalid class`) oraz**blokują stos USB** przy następnym uruchomieniu (karta sieciowa nie włącza się; napędy USB przestają być wykrywane). Zaktualizuj oprogramowanie z realtek.com (lub od dostawcy klucza sprzętowego), zanim zaczniesz polegać na prawidłowych ponownych uruchomieniach. |
| **Bufory odbiorcze**— słowo kluczowe `ReceiveBufferLen` |**256**(maksymalna wartość sterownika) | Pierścień odbiorczy karty sieciowej. Domyślna wartość sterownika wynosząca**32**pozostawia jedynie ~0,26 MB użytecznego pierścienia — zdecydowanie za mało na serię danych z wielu kamer — więc panel macierzy zgłasza błąd `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` i blokuje połączenia. Przy wartości**256**pierścień jest duży (**~13,5 MB zmierzone na laboratoryjnym hoście 10GbE**), co zapewnia potokowi odbiorczemu (RX) rzeczywisty zapas przepustowości na serię danych GVSP z wielu kamer. (O tym, czy dana konfiguracja faktycznie *nawiąże połączenie*, decydują dwie kontrole — **kontrola dopuszczalności z uwzględnieniem wyczerpania******oraz**kontrola agregowanej nadsubskrypcji** — a nie na zwykłym porównaniu wielkości serii z pojemnością pierścienia; zobacz [Model fps i serii macierzy](#array-fps--burst-model).) |
| **URB odbiorcze**— słowo kluczowe `PendingReceives` |**64** (maks.) | Bloki żądań USB w trakcie przesyłania; należy zwiększać wraz z buforami odbiorczymi w celu absorpcji serii danych. |
| **Ramka typu jumbo** — słowo kluczowe `*JumboPacket` | **9014** | Wymagane dla pakietów GVSP o rozmiarze 9000 bajtów (6× mniej pakietów na ramkę niż w przypadku 1500). |

> ⚠️ **Aktualizacja sterownika karty sieciowej RESETUJE te zaawansowane właściwości do wartości domyślnych.**Po aktualizacji lub wymianie sterownika karty**ponownie zastosuj** `ReceiveBufferLen=256` i `PendingReceives=64`, w przeciwnym razie panel macierzy ponownie zablokuje dostęp, mimo że „nic się nie zmieniło w sprzęcie”. Jest to główna przyczyna nagłej utraty łączności przez wcześniej działający zestaw.

Zastosuj zmiany z poziomu **PowerShell z uprawnieniami administratora** (zastąp nazwę swojej karty, np. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` dotyczy kart USB 10GbE.** Teraz wykrywa typ karty i dostosowuje właściwe słowo kluczowe dla pierścienia odbiorczego: `*ReceiveBuffers`→2048 dla kart sieciowych PCIe (Intel I219 itp.) lub `ReceiveBufferLen`→256 + `PendingReceives`→64 dla kontrolera Realtek **USB** 10GbE (który nie udostępnia `*ReceiveBuffers`). Wartości docelowe są ograniczone do maksymalnej wartości zgłaszanej przez każdy sterownik (`NumericParameterMaxValue`), dzięki czemu nigdy nie zapisuje wartości wykraczającej poza ten zakres. Uruchom to z terminala z **podwyższonymi** uprawnieniami; podobnie jak w przypadku każdej optymalizacji opartej na rejestrze, zmiana zacznie obowiązywać po ponownym uruchomieniu karty sieciowej lub ponownym uruchomieniu systemu. Powyższe ręczne polecenia `Set-NetAdapterAdvancedProperty` pozostają dobrą alternatywą — działają na bieżąco (ponownie wiążą kartę) bez konieczności ponownego uruchamiania.

### Podstawy sieciowe (wszystkie łącza LATTICE)

- **Adresowanie:** lokalne dla łącza `169.254.0.0/16` (GigE Vision LLA). Host przyjmuje statyczny adres `169.254.x.x/16`; kamery i DAQ-E samodzielnie przypisują sobie adresy w tym samym zakresie. Nie jest wymagane DHCP ani brama.
- **Rozmiar pakietu:**preferowany rozmiar jumbo (9000), ale niech funkcja automatycznego wykrywania go ustali — dokonuje ona ponownego pomiaru przy każdym połączeniu i już uwzględnia limit ICMP kamerylimit 1500 bajtów dla pakietów ICMP za pomocą sondy GVSP, więc ustawia rozmiar na „jumbo” wszędzie tam, gdzie linia faktycznie go obsługuje. Ustawiaj stałą wartość za pomocą `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` tylko wtedy, gdy wiesz więcej niż sonda, i preferuj ustawienia na poszczególne polecenia zamiast stałych: stała wartość omija sondę, więc jeśli ścieżka faktycznie nie jest w stanie przenieść 9000,**każde** przechwycenie kończy się przekroczeniem limitu czasu przy ustawieniu `SC_ERR_TIMEOUT -1011` (zobacz [Zmienne środowiskowe](#environment-variables)).
- **Pierścień RX skaluje się wraz z `ReceiveBufferLen`:**przy domyślnej wartości `32` użyteczny pierścień wynosi ~0,26 MB (zbyt mały dla jakiejkolwiek serii danych z wielu kamer); przy maksymalnej wartości `256` jest duży (~13,5 MB zmierzone na laboratoryjnym hoście 10GbE), co zapewnia rzeczywisty zapas. O tym, czy konfiguracja się połączy, decyduje wówczas kontrola dopuszczalności uwzględniająca obciążenie**oraz** poniższa kontrola agregowanej nadsubskrypcji — a nie surowe porównanie serii z pierścieniem.

### Liczba klatek na sekundę macierzy i model transmisji seryjnej

Jak odczytywać panel ustawień macierzy (oraz `lattice analyze-array` / `analyze_array_network` w modelu SDK):

- **Seria jest sumowana dla każdej kamery w formacie rzeczywistych pikseli danej kamery.**Kamery mono**M3M**przesyłają strumień**Mono12 (2 b/px)**; kamery Bayer**M3C**przesyłają strumień 8- lub 12-bitowy (model TRI032S cicho wysyła BayerRG12, nawet gdy żądany jest BayerRG8). Zatem klatka w pełnej rozdzielczości z 4 kamer ma rozmiar**~12,6 MB, jeśli wszystkie są 8-bitowe, ale ~25 MB w przypadku trzech 12-bitowych kamer mono**. Projekcja określa format każdej kamery na podstawie jej modelu (pamięci tożsamości), więc transmisja seryjna odpowiada temu, co faktycznie jest przesyłane — a nie opiera się na uniwersalnym założeniu o formacie BayerRG8.
- **Prędkość adaptera USB Ethernet jest ograniczona do 200 MB/s, niezależnie od podanych na tabliczce danych.** Tabela wydajności, która przekształca prędkość łącza na wartość ciągłą, pochodzi z PCIe; karta sieciowa USB podaje swoją prędkość łącza *Ethernet*, ale jest ograniczona przez magistralę USB i jej sterownik. Klucz USB 10GbE osiągał wcześniej ~1063 MB/s „ciągłej” przepustowości — wartość ta nigdy nie została zbadana — a wynikające z tego spowolnienie powodowało uszkodzenie 6–18 % ramek, mimo że nadal zgłaszano prawidłową docelową liczbę klatek na sekundę. Karty sieciowe podłączane przez USB są obecnie ograniczone do **200 MB/s** jako wartość bezwzględna (ograniczeniem jest magistrala, więc nie skaluje się to zgodnie z danymi znamionowymi; adapter USB 1 GbE osiąga ~80 MB/s i nie ma to na niego wpływu). `wire_ceiling_source` w rekordzie możliwości wyraźnie to określa, a `nic_is_usb` to sygnalizuje. W obu przypadkach można to nadpisać za pomocą `--wire-ceiling-mbps`.
- **Dopuszczalność uwzględnia obciążenie drenu, a nie cały- w rozróżnieniu na tryb burst i ring.** Jednoczesny burst musi zmieścić się jedynie w *przejściowym zaległości* = `max(0, Σ per-cam arrival − host drain) × emit_window`, a nie w całym burście. W strukturze z szybkim hostem i wolnymi kamerami (host **PCIe**10G + 4 kamery 1 GbE: przychodzące dane ≈ 320 MB/s, odbiór ≈ 1063 MB/s) host odbiera dane szybciej niż kamery je wypełniają, zaległość ≈ 0, więc symulacja emisji w pełnej rozdzielczości**dopuszcza**transmisję, mimo że seria 25 MB przekracza pojemność pierścienia wynoszącą 13,5 MB. Umieść te same cztery kamery za**USB**10GbE, a prędkość wysyłania wynosi 200 MB/s, a nie 1063 — przybywające dane wyprzedzają tę prędkość, a utrata pojawia się w postaci uszkodzonych ramek, a nie niższej częstotliwości odświeżania. Na hoście 1 GbE minimalna przepustowość pobierania (DLThr) wynosząca 31,25 MB/s sprawia, że tempo przychodzących danych przewyższa przepustowość odbiorczą → system poprawnie**blokuje** dane (w przypadku *tej* klasy blokad należy zmniejszyć obszar zainteresowania (ROI) lub zastosować binning ≥ 2). Dopuszczenie danych jest jedną z **dwóch** bramek łączących — drugą jest opisana poniżej kontrola agregowanej nadmiernej subskrypcji.
- **Prognozowana liczba klatek na sekundę (fps) stanowi konserwatywny pułap pobierania szeregowego.**Pętla pobierania danych przez host obecnie pobiera bufor każdej kamery**szeregowo**(około jednego okna wysyłania na kamerę), więc cykl jest ograniczony przez `max(readout+emit, N × emit)`, przy czymkamery ograniczoną do**łącza dostępowego**kamery (1 GbE ≈ 80 MB/s), a nie do łącza uplink hosta. W przypadku macierzy składającej się z 4 kamer o pełnej rozdzielczości i 12-bitowej głębi, co daje**~2,8 kl./s**, co odpowiada zmierzonej wartości ~2,7–3,0. Wartość kl./s jest celowo**niezależna od ekspozycji**, więc w słabo oświetlonych scenach rzeczywista wartość może nieznacznie spaść poniżej maksymalnej wartości wraz z wydłużaniem się ekspozycji. Prawdziwym ogranicznikiem liczby klatek na sekundę jest szeregowe pobieranie danych; równoległe przetwarzanie podniosłoby górną granicę w kierunku szybkości transmisji pojedynczej emisji.
- **Łączne nadmierne wykorzystanie przepustowości stanowi poważną przeszkodę w nawiązaniu połączenia.**Minimalna alokacja przepustowości na kamerę wynosi**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), więc gdy osiągnięta zostanie ta dolna granica, łączne zapotrzebowanie (`per_cam × N`) może przekroczyć**bezpieczny pod względem kolizji limit łącza**(`sustained × sim_emit_factor`). Praktyczne maksymalne wartości przy pełnej rozdzielczości w sieci 1 GbE:**6 kamer przy 1500 MTU, 9 przy ramkach jumbo**. Limit ten wynika wyłącznie z właściwości łącza i dolnej granicy — jest**niezależny od rozmiaru ramki**, więc**binning i mniejsze obszary zainteresowania (ROI) NIE pomagają** (zmniejszają one liczbę bajtów na *ramkę*, a nie liczbę bajtów na *sekundę* regulowaną przez GevSCPD*); jedynymi rozwiązaniami są: mniejsza liczba kamer, ramki typu jumbo w całej sieci lub szybsza karta sieciowa. Objawem byłaby utrata pakietów GVSP, a nie płynne zmniejszenie liczby klatek na sekundę, więc model `analyze-array` ustawia osiągalną liczbę klatek na sekundę na zero i wyświetla `**OVER-SUBSCRIBED**`, a kod `array-connect` przy zablokowanej rozdzielczości **odmawia nawiązania połączenia** (w przeciwnym razie funkcja „walk-down” redukuje liczbę klatek, co również nie usuwa tego rodzaju bloków). Kod `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zamienia odmowę na głośne ostrzeżenie przeznaczone do pracy testowej — zobacz [Zmienne środowiskowe](#environment-variables).

### Stan macierzy — który podsystem traci klatki

`GET /api/camera/array/<array_id>/capability` podłączonej macierzy zawiera aktywny
blok `health`, ponownie oceniany w ruchomym oknie **10-sekundowym**. Dzieli on utratę klatek
na dwie przyczyny wymagające przeciwnych działań naprawczych, zamiast zgłaszać jeden wskaźnik „niekompletności”,
który nie określa żadnej z nich:

| Pole | Co oznacza | Który podsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na port szeregowy) | Ramka **dotarła i miała błędy strukturalne**— utrata pakietów GVSP. |**Sieć**: przepustowość łącza, taktowanie, pierścień odbiorczy karty sieciowej, MTU |
| `never_arrived_rate_pct` (na port szeregowy) | Ramka **w ogóle nie dotarła**— kamera nie wykonała zdjęcia lub nie pozostało . |**Wyzwalacz / synchronizacja**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najgorsza częstotliwość transmisji dla każdej z nich. | — |
| `per_cam_rate_pct` | Łączny wskaźnik niekompletności na kamerę (obie przyczyny łącznie). | — |
| `stable_for_seconds` | Jak długo każda kamera pozostawała poniżej 0,01 %. | — |

Powyżej 5% backend rejestruje w logach wiersz `[array-health <id>] WARN` z nazwą podziału — przy
pierwszym przekroczeniu, przy zmianie przedziału ważności, raz na minutę, dopóki trwa, oraz raz, gdy
zostanie usunięte. Uszkodzona połowa wyświetla `[gvsp-corrupt <SN>]` przy pierwszym wystąpieniu dla każdej kamery i
przyczyny, a następnie podsumowanie co 60 s. Każda ocena nadal trafia do pliku dziennika zaplecza;
liczniki są aktualizowane dla każdego bufora, niezależnie od tego, co jest wyświetlane.

Ten sam rekord podaje liczbę, na której opiera się cała alokacja:

| Pole | Znaczenie |
| --- | --- |
| `wire_ceiling_mbps` | Aktualny, stały limit przepustowości łącza hosta, MB/s. |
| `wire_ceiling_source` | Skąd pochodzi ta liczba, słowami — np. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` lub `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, gdy wartość ta została ustawiona przez `--wire-ceiling-mbps` (lub pole **Wire Budget** w interfejsie graficznym). |
| `nic_is_usb` | `true` dla adaptera Ethernet USB — patrz limit 200 MB/powyżej. |

**Interpretacja:** wartość `gvsp_corrupt_rate_pct` różna od zera przy wartości `never_arrived_rate_pct` równej 0
oznacza, że wyzwalanie i synchronizacja kabla są idealne, a 100% strat występuje na ścieżce sieciowej
— zmniejsz wartość `--wire-ceiling-mbps` i podłącz ponownie. Odwrotny wzór wskazuje raczej na
kabel synchronizacyjny lub linię wyzwalającą.

> **Wartość `--target-fps` nie ma wpływu na uszkodzone ramki.** Tempo GevSCPD jest ustalane
> jednorazowo przy nawiązaniu połączenia, więc obniżenie częstotliwości wyzwalania zmienia cykl pracy, a nie
> szybkość transmisji w trybie jednoczesnej emisji. Zmniejszenie zapotrzebowania o 5× nie przyniosło poprawy;
> obniżenie limitu przepustowości z 240 do 200 MB/s spowodowało, że ten sam zestaw przeszedł z 10,4%
> uszkodzeń do 0,00%.

> **Automatyczne zmniejszanie przepustowości w trakcie transmisji nie jest dostępne w oprogramowaniu TRI032S.** Działająca macierz
> nie może samodzielnie tego naprawić; należy odłączyć i ponownie podłączyć, aby moduł wyboru czasu połączenia mógł
> ponownie zaplanować działanie z uwzględnieniem nowej wartości maksymalnej.

### Objaw → rozwiązanie

| Objaw (Ustawienia macierzy / połączenie / `analyze_array_network`) | Przyczyna | Rozwiązanie |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` zresetowano do 32 (zazwyczaj po aktualizacji sterownika) | Ustaw `ReceiveBufferLen`→256, `PendingReceives`→64; ponownie otwórz panel (uruchom ponownie backend, jeśli zapamiętał w pamięci podręcznej stary rozmiar pierścienia) |
| Restart/wyłączenie zawiesza się; później `Invalid class` błędy WMI, karta sieciowa nie włącza się, brakuje napędów USB | Stary sterownik Realtek USB 10GbE z 2016 r. → BSOD `0x9F` → wymuszone- | Zaktualizuj sterownik karty do wersji ≥ v10.67 (2026), a następnie ponownie zastosuj powyższe ustawienia pierścienia odbiorczego |
| Połączenie się udaje, ale zwraca rozdzielczość niższą od natywnej | Funkcja Smart-prep automatycznie-zmniejsza ramkę, aby dopasować ją do łącza | Zaktualizuj łącze / zaakceptuj zmniejszenie / `--force-tier slip-emit-and-capture` |
| Macierz zgłasza prawidłową docelową liczbę klatek na sekundę, ale dostarcza jedynie ułamek tej wartości; `health.gvsp_corrupt_rate_pct` niezerowe, `never_arrived_rate_pct` 0 | Wyprowadzony budżet przepustowości łącza po stronie hosta jest zawyżony w stosunku do rzeczywistej przepustowości (typowe w przypadku adaptera USB Ethernet, wąskiej linii PCIe lub współdzielonej sieci) | Należy ponownie nawiązać połączenie z niższym `--wire-ceiling-mbps` i ponownie sprawdzić blok stanu. **Brak** `--target-fps` — tempo GevSCPD jest ustalone przy połączeniu |
| Brak kamer w opublikowanych grupach; `health.never_arrived_rate_pct` niezerowe, `gvsp_corrupt_rate_pct` 0 | Ścieżka wyzwalania / ścieżka synchronizacji — kamery nie uruchamiają się, nie jest to problem sieciowy | Sprawdź kabel synchronizacyjny M8 i `--line`; upewnij się, że wszystkie elementy są uzbrojone (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` przekroczono w `analyze-array` lub odmowa połączenia przy ustalonej rozdzielczości (`array over-subscribes the wire`) | Łączne zapotrzebowanie na kamerę (minimum 8 MB/s × N kamer) przekracza bezpieczny pod względem kolizji limit przepustowości łącza — 6 kamer w pełnej rozdzielczości na 1 GbE przy 1500 MTU, 9 z ramkami jumbo | Mniejsza liczba kamer, ramki jumbo w całym łańcuchu lub szybsza karta sieciowa. **ROI/binning NIE pomogą** (limit ten jest niezależny od rozmiaru ramki). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zastępuje ustawienia testowe (akceptuje utratę pakietów) |

---

## `chloros-cli daq`

Polecenia czujnika spektralnego. Dwie klasy:
- **`pool-*`**— lekkie klienty HTTP, które sterują czujnikiem poprzez stałą pulę backendu.**Jest to obsługiwana ścieżka i jedyna dostępna w dostarczonym CLI.** Backend jest właścicielem transportu, więc GUI, skrypty CLI i SDK współdzielą jeden aktywny uchwyt zamiast rywalizować o port szeregowy.
- **Wszystko inne**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — bezpośredni dostęp do sprzętu, opisany poniżej dla kompletności. Wymagają one pakietu `daq` Python, który**nie jest zawarty w żadnym dostarczonym artefakcie**: skompilowany CLI go nie zawiera (`scripts/Build-CLI.ps1` ustawia `--nofollow-import-to=daq`, a transporty `pyserial` / `bleak` / `zeroconf`), a pakiet PyPI SDK również go nie zawiera. Działają one wyłącznie po pobraniu kodu źródłowego, więc należy traktować je jako wewnętrzną ścieżkę rozwojową MAPIR, a nie jako coś, po co należy sięgać.
- **`discover` / `list`** łączą te dwa podejścia: są to bezpośredniepochodzą z kopii źródłowej, ale w dostarczonej kompilacji przechodzą na `pool-discover`, a skanowanie wykonuje backend. Dzięki temu skanowanie działa wszędzie — co ma znaczenie, ponieważ jest to jedyny sposób na poznanie adresu MAC BLE urządzenia DAQ-M.

> **`chloros-cli daq --help`** (oraz `-h` / `help`) wyświetla listę podpoleczeń `pool-*` — pomoc jest celowo kierowana do klienta puli, aby odzwierciedlała polecenia, które faktycznie są wykonywane. Jeśli w gotowej kompilacji wywołasz podpolecenie bezpośredniego dostępu do sprzętu w dostarczonej kompilacji, program zakończy działanie z wyraźnym błędem wskazującym brakujący pakiet i odsyłającym do `pool-*`; nic nie kończy się bez komunikatu. (`discover` / `list` stanowią wyjątek — przekierowują do `pool-discover` i po prostu działają.)
>
> **Wszystko, czego potrzebuje klient, jest dostępne za pośrednictwem `pool-*`** — nawiązywanie połączenia, przesyłanie strumieniowe, nagrywanie skalibrowanych plików `.daq` oraz zamiana profili czapek. Urządzenie DAQ może być również sterowane z poziomu Python za pomocą `chloros_sdk.connect_daq_sensor()`, który korzysta z tej samej ścieżki zbiorczej.

### Przebieg pierwszego połączenia z czujnikiem DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Informacje o `pool-*`

| Podpolecenie | Cel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otwórz czujnik w puli zaplecza. |
| `daq pool-connect --port PORT` | DAQ-U na określonym porcie szeregowym. |
| `daq pool-connect --ble` | DAQ-M przez BLE, automatyczne skanowanie adresów MAC. |
| `daq pool-connect --mac MAC` | DAQ-M przez BLE na znanym adresie MAC (zakłada użycie `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E przez Ethernet na znanym hoście. |
| `daq pool-connect --eth` | DAQ-E przez Ethernet, host wykryty automatycznie (mDNS + rezerwa ARP; działa przy pustej pamięci podręcznej ARP na Windows i Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Dostosuj okno integracji / stan AE. |
| `daq pool-connect --no-stream` | Połącz się, ale nie rozpoczynaj jeszcze przesyłania strumieniowego (wznów za pomocą `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcji Cap. Domyślnym ustawieniem na backendzie jest `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skanuj każdy transport w poszukiwaniu czujników, z którymi można się połączyć, bez nawiązywania połączenia. **W ten sposób można znaleźć adres MAC BLE urządzenia DAQ-M.** `daq discover` / `daq list` są automatycznie przekierowywane tutaj w dostarczonych kompilacjach. Czujniki już otwarte w puli nie są wyświetlane — podłączony DAQ-M przestaje nadawać — w takim przypadku użyj `pool-list`. |
| `daq pool-list` | Pokaż wszystkie czujniki w puli zaplecza. |
| `daq pool-disconnect --sensor-id ID [--all]` | Zwolnij. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Najnowsze ramki widma N. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Wznowienie / wstrzymanie strumieniowania. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Rozpoczęcie / zatrzymanie nagrania .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmiana profilu korekcji cap w trakcie działania. |

### Podpolecenia bezpośredniego sterowania sprzętem (dostępne tylko po pobraniu kodu źródłowego — nie występują w dostarczanych kompilacjach)

> Wymienione dla kompletności. Wymagają one pakietu `daq` Python oraz `pyserial` / `bleak` / `zeroconf`, z których żadne nie znajduje się w skompilowanej wersji CLI ani w wersji SDK z PyPI — działają one wyłącznie po pobraniu kodu źródłowego MAPIR. **Jeśli korzystasz z wydanej kompilacji Chloros, użyj zamiast tego powyższych poleceń `pool-*`**; obejmują one nawiązywanie połączenia, przesyłanie strumieniowe, nagrywanie oraz wybór ograniczeń.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Otwórz, połącz się z zapisanym projektem Chloros (folder zawierający pliki `cameras.json` + `sensors.json` + `project.json`) i steruj nim. Wszystko przechodzi przez backend, więc interfejs graficzny i CLI generują identyczny stan sprzętu.

### Opis podpoleczeń

| Podpolecenie | Przeznaczenie |
| --- | --- |
| `project open PATH` | Wyświetla manifest urządzeń projektu (kamery, macierze, czujniki). |
| `project devices PATH [--reconnect]` | Wyświetla listę lub ponownie uruchamia wykrywanie. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Połącz wszystkie zapisane kamery / macierze / czujniki. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Pojedyncze przechwycenie z określonej kamery lub macierzy. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Seria-klatkowa seria z określonej kamery lub macierzy (`-n/--count` domyślnie 5; `-i/--interval` sekundy między klatkami, domyślnie 0). Serie z macierzy eliminują-duplikują powtarzające się zsynchronizowane grupy (mechanizm kontroli aktualności), dzięki czemu macierz o częściowym cyklu nie może zwrócić N kopii jednej klatki; wyświetla wyniki dla każdej iteracji. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Przesyłanie strumieniowe na dysk za pośrednictwem zadania zaplecza. `--poll-interval` = liczba sekund między odpytaniami `/stats` (domyślnie 2,0). |
| `project sensor read PATH NAME [--json]` | Najnowsza ramka widma. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Zapis .daq. |
| `project run PATH RECIPE.yaml` | Wykonaj recepturę przechwytywania YAML/JSON. `--dry-run` weryfikuje bez uruchamiania. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Oblicz wyrównanie dla tablicy — zobacz [poniższą tabelę flag](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Wyświetl bieżący profil wyrównania. |
| `project align clear PATH NAME` | Usuń profil z pamięci podręcznej. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Przesuń transformację jednego z podsystemów. |
| `project align export PATH NAME --to FILE` | Zapisz profil w pliku JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Załaduj zapisany profil. |

#### Opcje `project align calibrate`

| Flaga | Domyślna | Opis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda wyrównywania. **Te nazwy różnią się od `lattice align-calibrate`**, który przyjmuje skrócone formy `orb` / `akaze` / `phase`; te dwa polecenia nie są zamienne w przypadku tej flagi. |
| `--model {translation, rigid, affine, homography}` | `affine` | Przekształcenie modelu w celu dopasowania. |
| `--frames N` | `1` | Zsynchronizowane migawki klatek do średnią. |
| `--reference SN` | kamera główna | Numer seryjny kamery referencyjnej; wszystkie pozostałe elementy są do niej dopasowywane. |
| `--max-features N` | `5000` | Ograniczenie liczby cech ORB. |
| `--ratio-threshold F` | `0.75` | Test współczynnika Lowe&#x27;atest współczynnika. |
| `--ransac-threshold-px F` | `3.0` | Próg elementów wewnętrznych RANSAC. |
| `--min-matches N` | `15` | **Próg jakości** — odrzucenie rozwiązania poniżej tej liczby dopasowań punktów wewnętrznych. |
| `--max-reproj-err-px F` | `4.0` | **Kryterium jakości** — odrzucenie rozwiązania powyżej tego błędu reprojekcji RMS. |
| `--checkerboard RxC` | — | Geometria płyty dla `--method checkerboard`, np.g. `9x6`. |
| `--name PROFILE` | puste | Nazwa profilu osadzona w zapisanym JSON. **Nie jest to nazwa tablicy** — jest to pozycja `NAME`. |

Te dwa kryteria jakości są powodem, dla którego kalibracja może zakończyć się sukcesem pod względem rozwiązania, a mimo to
zostać odrzucona przy zapisywaniu: profil, który nie spełnia któregokolwiek z nich, powodowałby ciche błędne zarejestrowanie każdego
późniejszego przechwycenia, dlatego jest odrzucany zamiast zapisywany.

### Przykłady

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

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

### Język DSL receptury

`project run RECIPE.yaml` akceptuje plik YAML lub JSON opisujący sekwencję działań:

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
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
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

Obsługiwane akcje: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Akcja `burst` przyjmuje `name` (wymagane), `count` (domyślnie 5), `interval` (w sekundach, domyślnie 0), `output`, `format` oraz `settings` (takie same ustawienia dla poszczególnych kamer jak w przypadku `apply`); serie zdjęć z tablicy wykorzystują tego samego mechanizmu kontrolnego świeżo zsynchronizowanej grupy, co `project burst`.

Uruchom:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Zmienne środowiskowe

| Zmienna | Efekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Zastąpienie backendu URL (domyślnie `http://127.0.0.1:5000`) — **akceptowane tylko przez rodziny poleceń `lattice`X, `project` oraz `daq pool-*`.** Polecenia podstawowe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) przypisują `http://127.0.0.1:<port>` i ignorują tę zmienną (literał IPv4 omija Windows `localhost`→`::1` karę wynoszącą około 2 s na żądanie), więc zawsze kierują się do lokalnej maszyny. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` obniża poziom odrzucania połączeń spowodowanych nadmierną subskrypcją tablicy (łączny popyt na kamerę &gt; bezpieczny od kolizji pułap łącza przy `pin_resolution`) na głośne ostrzeżenie i kontynuację, akceptując utratę pakietów GVSP. Wyłącznie do użytku testowego — zobacz [Model fps i burst macierzy](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Ustawiane przez sam CLI; nakazuje backendowi włączenie przetwarzania równoległego. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` pomija sondę awaryjną GVSP (tylko wyniki ICMP). **To wyłącza tryb jumbo, a nie tylko wycisza log** — kamera odpowiada na pingi DF tylko do 1500 na każdej ścieżce, więc ten test jest jedyną metodą wykrywania pakietów jumbo. Oszczędza ~1 s na kamerę na każde połączenie; kosztuje ~1,45× maksymalnej przepustowości łącza, jeśli sieć *mogłaby* obsługiwać pakiety jumbo. SDK wyświetla ostrzeżenie po ustawieniu tej opcji. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Ustawia stały rozmiar pakietu GVSP na N bajtów; całkowicie pomija sondowanie. Zaleca się stosowanie opcji na poziomie polecenia (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) zamiast ustawiania na stałe: stały rozmiar uniemożliwia dostosowywanie się do sieci znajdującej się przed kamerą, a ustalenie wartości 9000 na ścieżce, która nie obsługuje pakietów jumbo, powoduje, że **każdy** przechwytywanie kończy się przekroczeniem limitu czasu przy użyciu `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Zastępuje katalog ekstrakcji plików Nuitka. CLI automatycznie używa `/mnt/ssd/tmp`, jeśli istnieje. |

---

## Kody wyjścia

| Kod | Znaczenie |
| --- | --- |
| `0` | Operacja zakończona powodzeniem. |
| `1` | Ogólna awaria (większość błędów podpoleceń). |
| `2` | Błąd argumentu. |
| `130` | Przerwano za pomocą Ctrl+C. |

---

## Wskazówki dotyczące rozwiązywania problemów

- **„Wymagane logowanie”** → Uruchom raz program `chloros-cli login EMAIL PASSWORD` na tym komputerze.
- **„Brak dostępu do serwera”** → Uruchom aplikację desktopową Chloros, uruchom bezpośrednio plik binarny serwera (`chloros-backend`) lub sprawdź `CHLOROS_BACKEND_URL`, jeśli korzystasz z połączenia zdalnego.
- **Polecenia `lattice` kończą się niepowodzeniem z komunikatem „Nie znaleziono sterowników kamery LATTICE”** → Środowisko uruchomieniowe Arena SDK nie jest zainstalowane; pakiet CLI zawiera `win32api` dołączony do Windows, ale środowisko uruchomieniowe C jest częścią instalatora GUI.
- **W oknie „Połącz macierz” / „Ustawienia macierzy” pojawia się komunikat „FRAMES WILL DROP” lub „Zmniejsz obszar zainteresowania (ROI), aby włączyć”** → Pierścień odbiorczy karty sieciowej hosta jest zbyt mały (zwykle resetuje się do 32 po aktualizacji sterownika karty sieciowej). Zobacz [Konfiguracja i dostrajanie karty sieciowej hosta](#host-nic-setup--tuning-lattice-arrays) — ustaw `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Komputer zawiesza się podczas ponownego uruchamiania/wyłączania, a następnie WMI `Invalid class` / karta sieciowa nie włącza się / brakuje napędów USB** → Przestarzały sterownik karty USB 10GbE powodujący błąd `DRIVER_POWER_STATE_FAILURE` (niebieski ekran śmierci `0x9F`). Zaktualizuj sterownik karty — zobacz [Konfiguracja i optymalizacja karty sieciowej hosta](#host-nic-setup--tuning-lattice-arrays).
- **Ostrzeżenie dotyczące pamięci wymiany w Jetson** → Dodaj pamięć wymiany opartą na pliku; kod CLI wyświetla dokładne polecenia `fallocate` / `swapon`.
- **Brak bezpośrednich poleceń DAQ** → Oczekiwane: dostarczony `chloros-cli` celowo nie zawiera pakietu `daq`, więc obecny jest tylko `pool-*` (wersja SDK z PyPI również go nie zawiera). Należy użyć `pool-*`, który steruje tym samym czujnikiem za pośrednictwem backendu, lub `chloros_sdk.connect_daq_sensor()` z Python.

---

## Zobacz też

- [Dokumentacja Python SDK](sdk-reference.md) — programowy odpowiednik każdego polecenia CLI.
- [Przewodnik po czujnikach DAQ](../daq/README.md) — okablowanie i kalibracja poszczególnych czujników.
- Dokumentacja online: `https://mapir.gitbook.io/chloros/cli`
