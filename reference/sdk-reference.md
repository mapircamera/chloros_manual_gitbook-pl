# Chloros Python SDK Odniesienie

**Wersja:**

1.2.0**Wygenerowano:**

29.07.2026 r., godz. 19:19 ·**Zaktualizowano:**

30.08.2026 r.**Pakiet:** `chloros-sdk` (PyPI)**Grupa docelowa:** Zoptymalizowane pod kątem wykorzystania przez duże modele językowe (LLM); czytelne dla ludzi.**Zakres:** Wszystkie publiczne klasy, funkcje i elementy pomocnicze udostępniane przez `import chloros_sdk`, wraz z przykładami, które można skopiować i wkleić, obejmującymi przetwarzanie obrazu, sterowanie pojedynczą kamerą, zsynchronizowane tablice, czujniki DAQ oraz automatyzację projektów.

Jeśli interesują Cię tylko najważniejsze informacje, przejdź do:
- [Instalacja i szybki start](#installation)
- [Smart-Connect dla układów LATTICE](#smart-connect-for-lattice-cameras)
- [Sesje czujników DAQ](#daq-sensor-sessions)
- [Automatyzacja projektów](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architektura w 60 sekund

SDK to cienka warstwa Python nakładająca się na backend Chloros (ten sam serwer Flask, z którego korzystają graficzny interfejs użytkownika oraz CLI). W celu automatyzacji należy zaimportować `chloros_sdk` i wywołać metody wysokiego poziomu; w tle każde wywołanie staje się żądaniem HTTP skierowanym do lokalnego backendu na porcie 5000 — `http://127.0.0.1:5000/api/...` (celowo nie `localhost`, który przekierowuje najpierw do `::1` na Windows i kosztuje ~2 s na żądanie w przypadku backendu obsługującego wyłącznie IPv4). Backend zarządza pulą sprzętu — kamerami, czujnikami DAQ, profilami wyrównania, buforami ramek — dzięki czemu skrypty SDK mogą współistnieć z interfejsem graficznym bez rywalizacji o porty szeregowe lub przepustowość karty sieciowej.

Będziesz korzystać z trzech powierzchni:

1. **`ChlorosLocal` + funkcje swobodne** (`process_folder`, `process_lattice_capture`) — potok przetwarzania obrazu. Przeprowadź kalibrację, debayering i eksport indeksu całego folderu za pomocą jednego wywołania Python.
2. **Uchwyty Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otwórz trwałą sesję zaplecza dla sprzętu działającego na żywo. Ten sam przebieg „smart-prep” co w GUI: sonda sieciowa, automatyczny wybór warstwy, PTP, inicjowanie AE, konfiguracja wyzwalacza GPIO.
3. **`ChlorosProject` / `open_project`** — Wczytanie zapisanego projektu (folder zawierający pliki `cameras.json` + `sensors.json` + `project.json`), podłącz wszystko jednocześnie i przeprowadzaj przechwytywanie za pomocą nazwanych uchwytów.

Powierzchnie 1 i 2 **automatycznie uruchamiają lokalny backend**, jeśli żaden jeszcze nie nasłuchuje (ten sam dołączony plik binarny, który uruchamia GUI/CLI) — dzięki czemu prosty skrypt działa w nowo uruchomionej powłoce bez konieczności uprzedniego uruchamiania backendu. Aby zrezygnować z tej funkcji, należy przekazać `auto_start_backend=False` (np. w przypadku wskazania zdalnego backendu, który nigdy nie jest uruchamiany). Zobacz [Automatyczne uruchamianie backendu](#backend-auto-start). Interfejs 3 zachowuje się inaczej: `open_project()` nie przyjmuje parametru `auto_start_backend`, a `connect_all()` nigdy nie uruchamia backendu — raz sprawdza `http://127.0.0.1:5000` i, jeśli nic nie odpowiada, po cichu przechodzi na bezpośrednie (bez zaplecza) sterowanie urządzeniem `lattice_sdk`. Tylko `proj.process()` i `stream(..., overlays=True)` leniwie tworzą `ChlorosLocal()` (które wykonuje automatyczneuruchamianie).

Wszystkie trzy wymagają uwierzytelnienia: uruchom `chloros-cli login` raz na komputerze lub zaloguj się za pomocą graficznego interfejsu użytkownika na pulpicie. Wywołania SDK bez ważnej sesji powodują wygenerowanie błędu `ChlorosAuthenticationError`.

Wymagania:
- Python 3.7+ (zgodnie z informacją w pakiecie; opracowany/przetestowany w wersji 3.10)
- Lokalnie zainstalowany program Chloros Desktop (plik binarny zaplecza znajduje się w instalatorze)
- Aktywne logowanie Chloros+. Poziom SDK/CLI musi należeć do poziomu **Copper**lub wyższego (Copper / Bronze / Silver / Gold); bezpłatny poziom**Iron**nie ma dostępu do SDK/CLI. Jest to egzekwowane**po stronie serwera**: każde żądanie oznaczone flagą SDK/CLI musi zawierać zarówno aktywną sesję, jak i płatny plan, w przeciwnym razie backend zwraca kod `403` z `error_code: PLAN_UPGRADE_REQUIRED` (wyświetlany jako `ChlorosLicenseError` przez `ChlorosLocal` oraz jako `ChlorosConnectError` przez pomocniki `connect_*`). Zamiast tego wylogowany użytkownik otrzymuje `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — te dwa błędy różnią się od siebie, ponieważ ponowne uruchomienie `chloros-cli login` naprawia pierwszy błąd, a drugiego nie jest w stanie naprawić.
- Korzystanie w trybie offline jest obsługiwane w ramach okresu karencji planu: poziom dostępu jest odczytywany z pamięci podręcznej walidacji serwerowej (5 minut) lub z pamięci podręcznej podpisanej, przypisanej do konkretnego urządzenia licencji (30 dni dla planów miesięcznych, do wygaśnięcia subskrypcji dla planów rocznych). Po upływie tego okresu karencji plan przechodzi w tryb bezpłatny, a dostęp do SDK/CLI zostaje wstrzymany do momentu, gdy urządzenie uzyska połączenie z serwerem. `chloros-cli status` (`GET /api/license-status`) pozostaje dostępny w ramach bezpłatnego poziomu, więc przyczyna jest widoczna — jest to jedyna trasa SDK/CLI zwolniona z ograniczeń związanych z poziomem taryfowym.
- Windows 10/11 64-bitowy, **Ubuntu 22.04 LTS lub nowszy**, albo Jetson (JetPack 6). Ubuntu 20.04**nie** jest obsługiwany: zależności `.deb` wynikają z tego, z czym łączy się backend, w tym `libc6 (>= 2.34)`, a Focal dostarcza glibc 2.31.

---

## Instalacja

Python SDK to cienka warstwa Python nakładająca się na backend Chloros. W przypadku wszystkich zastosowań wykraczających poza kilka przepływów pracy związanych wyłącznie z DAQ konieczne jest **lokalne zainstalowanie pakietu desktopowego Chloros** (instalator Windows lub Linux `.deb`) — to właśnie on zapewnia plik binarny backendu, środowisko uruchomieniowe Arena SDK dla kamer LATTICE oraz pakiety kalibracyjne.

Najnowsze pliki do pobrania: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Krok 1 — Zainstaluj pakiet platformowy Chloros

#### Windows (.exe)

1. Pobierz plik `Chloros-Setup-x.y.z.exe` ze strony pobierania.
2. Uruchom instalator i postępuj zgodnie z instrukcjami kreatora. Domyślna ścieżka instalacji to `C:\Program Files\MAPIR\Chloros\`.
3. Uruchom co najmniej raz program Chloros i zaloguj się na swoje konto Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Krok 2 — Zainstaluj Python SDK

**Instalator Chloros zawiera pasujący pakiet wheel SDK.** Każdy instalator Windows oraz plik Linux .deb umieszcza na dysku plik `chloros_sdk-X.Y.Z-py3-none-any.whl`, który dokładnie odpowiada wersji GUI / CLI / backendu. Nie musisz śledzić PyPI, aby zachować synchronizację.

#### Windows

Instalator automatycznie uruchamia `pip install` z dołączonym plikiem wheel, korzystając z systemowego Python (preferowany jest `py.exe` program uruchamiający, w przeciwnym razie stosowany jest `python -m pip`). Nie trzeba nic robić — `import chloros_sdk` działa w środowisku Python po pomyślnej instalacji. Jeśli na komputerze nie ma Python, instalator w tle pomija ten krok, a interfejs graficzny oraz CLI nadal działają.

#### Linux (.deb)

Plik .deb umieszcza koło w `/usr/lib/chloros/sdk/`. `postinst` wyświetla dokładne polecenie — dystrybucje zgodne z PEP 668 domyślnie odrzucają globalne zapisy w pip, więc nie instalujemy automatycznie:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

W przypadku wdrożeń Jetson w trybie air-gapped proces ten przebiega całkowicie w trybie offline — pakiet wheel znajduje się już na dysku.

#### Publiczny PyPI

W przypadku hostów korzystających wyłącznie z pip (bez zainstalowanego pakietu desktopowego Chloros; przepływy pracy oparte wyłącznie na zdalnym backendzie lub DAQ):

```bash
pip install chloros-sdk
```

PyPI jest aktualizowane w kompilacjach instalatora o wersji wydanej, więc opublikowany plik wheel odpowiada najnowszej stabilnej wersji. Kompilacje deweloperskie (np. `1.1.4.dev1`) są dostarczane wyłącznie za pośrednictwem dołączonego pliku wheel instalatora.

#### Sprawdź

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Wymagana subskrypcja Chloros+.** Wszystkie wywołania SDK wymagają aktywnego loginu Chloros+. Uruchom `chloros-cli login user@example.com 'YourPassword'` raz na każdym komputerze; dane uwierzytelniające są buforowane w `~/.chloros/`.

### Czy potrzebuję pakietu Desktop?

Sam pakiet pip **nie** wystarcza w przypadku większości procesów roboczych. Oto, czego wymaga każda platforma SDK:

| Platforma SDK | Czy potrzebny jest pakiet Desktop? | Dlaczego |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Tak** | Automatycznie uruchamia plik binarny zaplecza na `/usr/lib/chloros/chloros-backend` (Linux) lub `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Tak**(lokalnie)**/ Nie**(zdalne) | Czysti klienci HTTP korzystający z zaplecza. Lokalne zaplecze → wymagany pakiet na pulpit. Zdalny backend → `backend_url=`**przez tunel** (zobacz Tryb zdalnego backendu — dostarczone backendy wiążą się wyłącznie z pętlą zwrotną). |
| `ChlorosProject` / `open_project` | **Tak** | Obsługuje zapisane projekty za pośrednictwem backendu. |
| Bezpośrednie klasy LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Tak** | Wymaga natywnego środowiska uruchomieniowego Arena SDK, które jest dostarczane w pakiecie na komputery stacjonarne. W przeciwnym razie `CAMERA_AVAILABLE` jest równoważne `False` przy importowaniu. |
| Klasy Direct DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nie** | Czysty Python przez pyserial/bleak/zeroconf. Środowisko oparte wyłącznie na pip może obsługiwać urządzenia DAQ od początku do końca. |

### Tryb zdalnego backendu (host oparty wyłącznie na pip, przez tunel)

> **Dostarczony backend nie jest dostępny w sieci LAN.** Wersje produkcyjne
> wiążą się wyłącznie z pętlą zwrotną (obie rodziny pętli zwrotnych) i kategorycznie odrzucają
> jedyny tryb bez pętli zwrotnej (`CHLOROS_CLOUD_MODE`), więc
> `backend_url="http://<lan-ip>:5000"` **nie może działać w połączeniu z zainstalowanym
> Chloros** — ten schemat działał wyłącznie w połączeniu z backendem typu source/dev
> . Aby sterować backendem na innym komputerze, należy samodzielnie przekierować jego port loopback
> i skierować SDK na tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Hosty bez monitorów / CI / roboty mogą utrzymywać jedną maszynę z pełną instalacją pulpitu jako „serwer Chloros”, a `pip install chloros-sdk` wszędzie indziej — jednak transport między nimi odbywa się za pośrednictwem wspomnianego powyżej tunelu skonfigurowanego przez użytkownika, a nie poprzez bezpośrednie połączenie LAN URL.

> **Znane ograniczenie — `ChlorosLocal` nie obsługuje wyłącznie funkcji pip-only-capable.** `ChlorosLocal(backend_url=BACKEND)` obecnie rozpoznaje lokalny plik binarny backendu w swoim konstruktorze *przed* sprawdzeniem URL i zgłasza błąd `ChlorosBackendError` („Chloros backend not found…”) w przypadku braku zainstalowanego pakietu desktopowego — nawet przy dostępnym zdalnym backendzie. Działa wyłącznie powyższy interfejs smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`, a także `analyze_array_network` oraz pomocnicze `list_*` / `discover_*`) działają na hoście korzystającym wyłącznie z pip.

### Przepływ pracy wyłącznie z DAQ (host wyłącznie z pakietem pip)

Jeśli potrzebujesz wyłącznie czujników DAQ i nie korzystasz z kamer LATTICE ani przetwarzania obrazu, pakiet pip jest samowystarczalny:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Brak backendu, brak pliku .deb, brak konieczności logowania się do Chloros+ w celu bezpośredniej pracy z sprzętem DAQ.

---

## Szybki start

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indeks najwyższego poziomu API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Przetwarzanie obrazów — `ChlorosLocal`

Główna klasa potoku. Uruchamia backend przy pierwszym użyciu, tworzy i konfiguruje projekty, monitoruje postępy oraz zwraca podsumowania po zakończeniu działania.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metody

| Metoda | Opis |
| --- | --- |
| `create_project(project_name, camera=None)` | Utwórz nowy projekt (opcjonalnie z szablonem kamery, np. `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importuje obrazy w formatach RAW/TIF/JPG/DNG **oraz nagrania z czujnika światła `.daq`**. Zwraca `count` (obrazy) i `scan_count` (nagrania). Wyświetla ostrzeżenie tylko wtedy, gdy w folderze nie ma ani jednego, ani drugiego. |
| `export_light_sensor(daq=True, csv=True)` | Zapisuje skalibrowane pliki `.daq` + `.csv` dla każdego zapisu z czujnika światła w projekcie do pliku `<project>/Light Sensor/`. Zobacz [Zapisy z czujnika światła](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Ustaw pokrętła przetwarzania. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Uruchom potok. Zwraca `{"status": "complete", "async": False}`, oraz klucz `summary`, jeśli backend go udostępnia — zobacz [Podsumowanie po uruchomieniu i wskazówki](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Sprawdź stan modułu zaplecza. |
| `logout()` | Wyczyść poświadczenia z pamięci podręcznej. |
| `shutdown_backend()` | Zakończ działanie backendu (jeśli został uruchomiony przez SDK). |
| `discover_cameras()` | Wykryj kamery LATTICE **za pośrednictwem backendu tej instancji** (`/api/camera/discover`). Zwraca listę słowników (`serial`, `model`, `ip`, …) — o tej samej strukturze, co w GUI/CLI. Pusta lista, jeśli nie znaleziono żadnej znaleziono lub backend jest niedostępny. |
| `camera_capture(output_dir, format="tiff", **settings)` | Przechwytuje pojedynczą klatkę**za pośrednictwem backendu**(automatyczneuruchomiony przez ten uchwyt), tak aby otrzymała takie samo przygotowanie jak GUI/CLI (domyślnie 12 bitów, ponowne wykorzystanie puli, osadzone metadane kalibracyjne). Rozpoznaj cel za pomocą `serial=` lub `device_index=`; przekaż `exposure`/`gain`/`pixel_format`/`preset` jako `**settings`. Zwraca słownik metadanych starszego typu (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Generuje klatki podglądu z nałożonymi elementami z puli — lekkiego klienta MJPEG poprzez trasę `/api/camera/<serial>/stream-annotated` backendu (zebra / siatka / krzyżyk / histogram / peaking / punkt rysowany po stronie serwera). `decode=True` generuje tablice BGR; `False` generuje surowe bajty JPEG. Dostępne również na poziomie projektu jako `ChlorosProject.stream(overlays=True)`. |

Użyj jako menedżera kontekstu w celu zagwarantowanego czyszczenia:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Zapisy z czujników światła — skalibrowane `.daq` + `.csv`

Urządzenie DAQ-U / DAQ-M / DAQ-E może być rejestrowane **bez** pakietu kalibracyjnego. Tak właśnie
działają domyślnie publiczne rejestratory [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): zapisują surowe wartości z czujników i oznaczają plik
pliku, dzięki czemu Chloros pobiera fabryczną kalibrację danego czujnika **przez port szeregowy** — najpierw z lokalnej pamięci podręcznej,
a następnie z chmury MAPIR — i stosuje ją podczas importu.

Chloros zapisuje wynik z powrotem jako dwa produkty na każdy zapis, w sekcji
`<project>/Light Sensor/`:

| Produkt | Co to jest |
| --- | --- |
| `<name>_calibrated.daq` | Archiwum nadające się do ponownego przetworzenia — ten sam schemat co w przypadku zapisu na żywo, teraz z deklaracją pakietu, który je wygenerował. Ponowny import **nie** powoduje jego ponownej kalibracji. |
| `<name>_calibrated.csv` | Natężenie promieniowania spektralnego w W/m²/nm w siatce długości fal czujnika, jeden wiersz na odczyt, plus kolumny fotometryczne (moc całkowita, luks fotopowy/skotopowy, PPFD i jego podział na niebieski/zielony/czerwony, długość fali szczytowej). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Tylko czujniki bez pakietu (DAQ-A).** Surowe wartości liczbowe czujnika spektralnego — *nie* natężenie promieniowania. Zobacz poniżej. |

`process()` wykonuje ten eksport jako jeden z etapów. **Nie** wymaga on obrazów:
samodzielnie latający czujnik światła stanowi pełnoprawny przepływ pracy, a taki projekt z definicji nie zawiera
żadnych obrazów.

**Zapisy z DAQ-A są eksportowane jako surowe odczyty.** Rodzina DAQ-A powstała przed wprowadzeniem systemu wiązek na
i nie ma żadnego pakietu do pobrania — jest kalibrowana w terenie względem
celu odbicia, dlatego nigdy go nie potrzebowała. Te zapisy są eksportowane
pod prefiksem `_raw` zamiast `_calibrated`: inna nazwa pliku, a nie flaga
wewnątrz pliku, ponieważ informacja ta musi przetrwać wysyłkę e-mailem jako sama nazwa. Nagłówek
`.csv` zawiera wartość `raw spectral sensor counts (NOT irradiance)` i ostrzega, że
wartości są porównywalne **w obrębie** pliku — dokładnie do tego służy kalibracja oparta na
celu — a nie między czujnikami. Kolumny fotometryczne zależne od mocy (moc całkowita,
luks fotopowy/skotopowy, PPFD) zwracają wartość **NULL** zamiast być zintegrowane na podstawie zliczeń.

Urządzenie DAQ-U / DAQ-M / DAQ-E, którego pakietu po prostu nie udało się pobrać, jest nadal **pomijane**,
a nie zapisywany w formacie surowym: w tym przypadku pakiet istnieje i „ponowne połączenie i ponowne przetworzenie” to realna rada.

Starsze zapisy **v1.01 / v1.02** (zapisywane przez DAQ-A-SD) nie zawierają epoki dla poszczególnych odczytów,
a jedynie czas zapisu pliku. Moduł dopasowujący obraz↔promieniowanie padające nadal je odrzuca — dopasowanie
klatki do czasu zapisu byłoby błędne w sposób niewidoczny — jednak eksportator je odczytuje, a
CSV wyświetla `clock=daq_created_on`, dzięki czemu produkt wskazuje, na którym zegarze się znajduje.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Rejestracja, której pakiet kalibracyjny nie może zostać pobrania (tryb offline lub czujnik bez
kalibracji w pliku) jest zgłaszany pod numerem `skipped` **wraz z przyczyną**. Nigdy nie jest
zapisywany jako „skalibrowany” plik zawierający surowe dane liczbowe — wystarczy połączyć się z internetem i
uruchom ponownie, a eksport zostanie zakończony.

### Wywołania zwrotne dotyczące postępu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Podsumowanie po zakończeniu i wskazówki

Po zakończeniu `process()` pobiera `GET /api/processing-summary` i dołącza treść jako `result["summary"]`. Pobieranie odbywa się na zasadzie „best-effort” i nigdy nie blokuje pomyślnego powrotu — jeśli podsumowanie jest niedostępne, `process()` przechodzi na zwykły format `{"status": "complete", "async": False}`. Każdy wpis w `summary["hints"]` — pełne zdania zawierające sugerowane działania naprawcze, np. dlaczego uruchomienie nie dało żadnego wyniku — jest również ponownie wysyłany jako Python `UserWarning`, więc uruchomienia bez wyniku są samodiagnostyczne, nawet jeśli nigdy nie sprawdzisz słownika:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` to część nadająca się do odczytu maszynowego:

| Klucz | Co liczy |
| --- | --- |
| `models` | Grupy kamer w przebiegu. |
| `images_in_groups` | Obrazy źródłowe w tych grupach. |
| `targets_found` | Wykryte cele odbicia . |
| `images_calibrated` | Obrazy, które zostały skalibrowane w przebiegu. |
| `exported_files` | **Pliki produktów obrazowych zapisane w przebiegu.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Zapisy z czujników światła, celowo zliczane oddzielnie — pochodzą one z innego etapu i występują w przebiegach, w których w ogóle nie ma obrazów, więc uwzględnienie ich sprawiłoby, że przebieg obejmujący wyłącznie akwizycję danych wyglądałby tak, jakby wyeksportowano z niego obrazy. |

Oprócz nich: `summary["output_dirs"]` (każdy katalog, do którego zapisano dane),
`summary["light_sensor_export"]`, `summary["stopped"]` (ma wartość true, gdy użytkownik przerwał
przebieg, dzięki czemu częściowe wyniki nie są interpretowane jako niekompletny przebieg, który dał zbyt małą liczbę wyników) oraz
`summary["groups"]` (podział na grupy).

`exported_files` jest rejestrowany przez potok **w trakcie zapisu**, a nie skanowany później z
obiektów obrazu projektu. Strategie równoległe i wykorzystujące GPU tworzą własne obiekty obrazu
(w podprocesach roboczych dla ścieżek GPU), więc stare skanowanie zgłaszało
`0 file(s) written` dla każdego takiego przebiegu, a następnie generowało wskazówkę o zerowej liczbie eksportów — w przypadku przebiegów,
w których wszystko działało poprawnie. Jeśli tworzysz skrypt oparty na tym numerze, prawidłowe uruchomienie równoległe
zgłasza teraz liczbę niezerową.

Raporty pominięć czujnika światła podają rzeczywistą przyczynę ustaloną przez czytnik dla każdego pliku —
nieczytelny schemat, brakujący pakiet, błąd zapisu — **po deduplikacji**, więc dwadzieścia plików
pominiętych z jednej przyczyny traktowanych jest jako jedna przyczyna, a nie dwadzieścia jej powtórzeń.

> **`process()` nie jest generowany, gdy przebieg nie generuje żadnych obrazów.** Jest to jedyne miejsce, w którym SDK i
> CLI celowo się różnią: `chloros-cli process` traktuje sytuację „produkty zostały zażądane, ale żadne nie zostały
> zapisane” jako błąd i kończy działanie z wynikiem niezerowym, podczas gdy SDK kończy działanie normalnie i zgłasza ten
> stan poprzez `summary` / wskazówki. Jeśli Twój potok powinien zatrzymać się w przypadku pustego przebiegu, sprawdź to
> samodzielnie — przeanalizuj `summary` (lub policz pliki w folderze projektu), zamiast polegać na
> braku wyjątku. Typowymi przyczynami są: folder wejściowy, który nie został rozpoznany jako
> przechwytywanie, oraz produkty pominięte jako nieodpowiednie dla obecnych kamer (np. promieniowanie z kamer obsługujących wyłącznie RGB
>).

### Funkcje pomocnicze

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Obsługiwane wartości

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Wyniki radiometryczne (potok wielospektralny LATTICE)

Poziom eksportu wielospektralnego (M3C/M3M) potoku `process` — `reflectance` (domyślny), `radiance`, `sensor-response` lub `all` (każdy odpowiedni tryb dla każdego obrazu) — odpowiada ustawieniu przetwarzania **„Wyjście radiometryczne”** w projekcie. `configure()` ma dedykowane słowo kluczowe:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Zaawansowana furtka awaryjna — zapisanie klucza `"Radiometric output"` projektu za pomocą `custom_settings` — nadal działa, ale należy pamiętać, że zastępuje to cały blok ustawień (zobacz ostrzeżenie poniżej):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (wartość domyślna) dzieli promieniowanie kamery przez **dane DAQ o dopasowanym znaczniku czasu**, ustalane automatycznie na podstawie zarejestrowanego `.daq` (DAQ-U/M/E)**lub natywnego `.csv`**-M natywnego `.csv`**znalezionego wraz z obrazami; wszelkie brakujące lokalnie pakiety kalibracyjne dla poszczególnych kamer lub urządzeń DAQ są**automatycznie pobierane z AWS** przy pierwszym użyciu. CLI udostępnia to w postaci przełączników dla poszczególnych typów produktów w `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **zastępuje** cały blok obliczonych ustawień (zgodnie z założeniami pomija pozostałe słowa kluczowe i walidację `configure()`). Korzystając z niego, należy uwzględnić wszystkie kluczowe elementy `Project Settings`, które są istotne, tak jak w powyższym przykładzie.

---

## Smart-Connect dla kamer LATTICE

Trwałe sesje zaplecza dla sprzętu działającego w trybie na żywo. Te same punkty końcowe, z których korzysta GUI, więc zachowanie jest identyczne w SDK / CLI / GUI.

### Pojedyncza kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Sygnatura `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metody

| Metoda | Opis |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Odczyt węzłów GenICam; zwraca `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Zapis węzłów według nazwy przyjaznej (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Przechwytuje **pojedynczą** klatkę. Zwraca listę składającą się z jednego elementu, zawierającą słowniki metadanych klatki. (Przechwytywanie seryjne/wieloklatkowe zostało usunięte — jeśli potrzebujesz serii, wywołaj funkcję `capture()` w pętli.) |
| `disconnect()` | Zwolnij z puli. Nie wykonuje żadnej operacji, jeśli dołączono do już otwartej sesji. |

Kontrola eksportu `capture()` (ten sam model co tablica + GUI):

- `processing` / `levels` — `processing="all"` zapisuje każdy odpowiedni typ eksportu; `levels=["raw","radiance"]` zapisuje tylko te (zastępuje `processing`). Aby użyć domyślnych ustawień zaplecza, należy pominąć oba parametry.
- `force_daq=True` — zapisuje przypisany odczyt z DAQ/DLS jako plik dodatkowy `.daq` nawet w przypadku pobrania wyłącznie danych surowych, dzięki czemu klatka może zostać później przetworzona na współczynnik odbicia/indeks . Nie ma efektu, jeśli nie jest podłączony żaden moduł DAQ.

### Zsynchronizowana matryca — `ArraySession` (Smart-Prep)

`connect_array` jest **zalecanym punktem wejścia** dla konfiguracji wielokamerowych. W tle uruchamia pełny proces przygotowania inteligentnego (smart-prep) za pomocą interfejsu graficznego:

1. **Analiza sieci** (`/api/camera/array/recommend`) — wyszukuje największy rozmiar ramki, który mieści się w warstwie sim-emit bez utraty ramek.
2. **Automatyczny wybór warstwy** — `sim-capture-sim-emit`, jeśli łącze jest w stanie to obsłużyć; w przeciwnym razie `sim-capture-ftd-stagger` lub `slip-emit-and-capture`.
3. **Automatyczne zmniejszanie**— w tle zmniejsza rozmiar ramki / zwiększa binning, gdy łącze nie jest w stanie utrzymać żądanej rozdzielczości.**To zabezpieczenie nie obejmuje zagregowanego nadmiernego przydziału**: zbyt dużej liczby kamer dla danego łącza nie da się rozwiązać poprzez zmniejszenie rozmiaru ramek — zobacz [Nadmierny przydział](#over-subscription-the-per-cam-floor).
4. **PTP włączone** domyślnie — znaczniki czasu między kamerami są porównywalne z dokładnością do mikrosekund.
5. **Automatyczny wybór formatu pikseli dla każdej kamery** — kamery RGB → `BayerRG8`, multispec → `BayerRG12`.
6. **Inicjowanie AE** — rejestruje aktualny stan AE każdej kamery, dzięki czemu połączenie nie resetuje ekspozycji w trakcie działania.
7. **Konfiguracja wyzwalacza GPIO** — `connect_array` uruchamia każdą kamerę (`TriggerMode=On`, `TriggerSource=Line2`), dzięki czemu impuls z kamery głównej steruje kamerami podrzędnymi przez kabel M8. Jest to krok przeznaczony wyłącznie dla układów wielokamerowych: pojedyncza kamera uruchomiona za pomocą `LatticeCamera` działa natomiast w trybie swobodnym.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Sygnatura `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier` wartości:
- `"sim-capture-sim-emit"` — prawdziwa synchronizacja (wszystkie kamery wyzwalają się na tej samej krawędzi zegara).
- `"sim-capture-ftd-stagger"` — elastyczne rozłożenie w dziedzinie czasu (kamery emitują sygnały w nieznacznie przesuniętych momentach, dzięki czemu pakiety są szeregowane w sieci).
- `"slip-emit-and-capture"` — sekwencyjne przechwytywanie dla poszczególnych kamer (brak synchronizacji czasowej; jedyna opcja, gdy żaden rozmiar ramki nie pasuje do trybu symetrycznego).

`wire_ceiling_mbps` zastępuje **trwały limit przepustowości łącza hosta** w MB/s — pojedynczy
liczba, od której zależy cała alokacja macierzy. Pozostaw ustawienie `None`, aby korzystać z automatycznie wykrytej
wartości. Zmniejsz tę wartość, gdy macierz zgłasza uszkodzone ramki GVSP: wartość automatyczna jest wyznaczana
z ogłaszanej przez kartę sieciową prędkości łącza, która zawyża prędkość adapterów USB, wąskich pasm PCIe oraz
obciążonych struktur współdzielonych — a to zawyżenie objawia się uszkodzonymi ramkami, a nie
widocznym spowolnieniem łącza. Wartość ta jest zapisywana w bloku przechwytywania macierzy projektu, więc
ponowne otwarcie lub późniejsze ustawienie `connect_array` przywraca ją tak samo jak każde inne ustawienie macierzy.
Zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames).

#### Nadmierna subskrypcja (limit minimalny na kamerę)

Regulacja tempa Sim-emit przydziela każdej kamerze część budżetu łącza zabezpieczonego przed kolizjami, z dolnym limitem wynoszącym **8 MB/s na kamerę**(`per_cam_floor_bps`). Gdy `N × floor` przekroczy górny limit zabezpieczony przed kolizjami, macierz**przeciąża łącze**— trybem awaryjnym jest utrata pakietów GVSP, a nie niższa liczba klatek na sekundę — i nie ma rozwiązania polegającego na zmianie rozmiaru klatki:**binning i ROI zmniejszają liczbę bajtów na klatkę, a nie liczbę bajtów na sekundę przydzielaną w ramach algorytmu**, którą porównuje kontrola agregowana. Praktyczne limity przy pełnej rozdzielczości na hoście 1 GbE:**6 kamer przy MTU 1500, 9 z ramkami jumbo** (`max_cams_collision_safe` w odpowiedzi analitycznej podaje limit dla danego łącza). Rozwiązania: mniej kamer, ramki jumbo odod początku do końca lub szybsza karta sieciowa.

- Odpowiedzi `analyze_array_network()` i `/api/camera/array/connect` zawierają `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe`, oraz `per_cam_floor_bps`. Gdy wartość `oversubscribed` jest prawdziwa, projekcja **zeruje pola fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) zamiast zgłaszać mylącą wartość wskazującą na powolną, ale działającą szybkość.
- `POST /api/camera/array/connect` akceptuje parametr `pin_resolution` (**tylko HTTP — a nie kwarg SDK**; `connect_array` nie udostępnia go). Przypisanie usuwa zabezpieczenie polegające na stopniowym zmniejszaniu wielkości, więc połączenie z nadmierną liczbą subskrypcji z zestawem `pin_resolution`X jest**kategorycznie odrzucane** wraz z komunikatem błędu wymieniającym wszystkie możliwe rozwiązania. Bez przypisania połączenie przechodzi przez proces zmniejszania, ale wyświetla ostrzeżenie, że zmniejszenie nie pozwoli na wyczyszczenie agregatu.
- Łuk ratunkowy w środowisku testowym: ustaw `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` w środowisku backendu, aby zmienić odrzucenie na głośne ostrzeżenie — i tak nawiążesz połączenie i zaakceptujesz utratę pakietów.

#### Stan macierzy — który podsystem traci ramki

`GET /api/camera/array/<array_id>/capability` zawiera aktualny blok `health` na
podłączonej macierzy, ponownie oceniany w ruchomym oknie **10-sekundowym**. Dzieli on utratę ramek
na dwie przyczyny wymagające przeciwnych rozwiązań, zamiast jednego wskaźnika „niekompletności”, który
nie określa żadnej z nich:

| Pole | Co oznacza | Który podsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na port szeregowy) | Ramka **dotarła, ale miała błąd strukturalny**— utrata pakietów GVSP. |**Sieć**: przepustowość łącza, taktowanie, pierścień odbiorczy karty sieciowej, MTU |
| `never_arrived_rate_pct` (na numer seryjny) | Ramka **w ogóle nie dotarła**— kamera nie zadziałała lub nic z niej nie wyszło. |**Wyzwalacz / synchronizacja**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najgorsza częstotliwość transmisji dla każdej z kamer. | — |
| `per_cam_rate_pct` | Łączny wskaźnik niekompletności na kamerę (obie przyczyny łącznie). | — |
| `stable_for_seconds` | Jak długo każda kamera pozostawała poniżej 0,01 %. | — |

Oprócz `health` ten sam rekord podaje liczbę, od której zależy cała alokacja:

| Pole | Znaczenie |
| --- | --- |
| `wire_ceiling_mbps` | Aktualny, stały limit przepustowości łącza hosta, MB/s. |
| `wire_ceiling_source` | Źródło tej wartości, opisane słowami — np. `USB-capped 200 MB/s (was theoretical 1062; …)` lub `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, gdy `wire_ceiling_mbps=` go ustawił. |
| `nic_is_usb` | `true` dla adaptera Ethernet USB. |

Nie ma opakowania SDK dla tego punktu końcowego — odczytaj go bezpośrednio:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Odczyt:** wartość `gvsp_corrupt_rate_pct` różna od zera przy wartości `never_arrived_rate_pct` równej 0 oznacza, że
wyzwalanie i synchronizacja kabla są idealne, a 100% strat występuje na ścieżce sieciowej — zmniejsz
wartość `wire_ceiling_mbps` i ponownie nawiąż połączenie. Odwrotny wzorzec wskazuje natomiast na kabel synchronizacyjny lub
linię wyzwalającą.

> **Wartość `target_fps` nie jest czynnikiem decydującym o uszkodzonych ramkach.** Częstotliwość GevSCPD jest ustalana jednorazowo przy
> nawiązaniu połączenia, więc obniżenie częstotliwości wyzwalania zmienia cykl pracy, a nie
> częstotliwość wysyłania serii impulsów o równoczesnej emisji. Zmierzona 5-krotna redukcja zapotrzebowania nie przyniosła poprawy, podczas gdy
> obniżenie górnej granicy przepustowości z 240 do 200 MB/s spowodowało spadek wskaźnika uszkodzeń tej samej platformy z 10,4 % uszkodzeń do
> 0,00 %.

> **Funkcja automatycznego zmniejszania przepustowości w trakcie transmisji nie jest dostępna w oprogramowaniu układowym TRI032S.** Działająca macierz nie może
> samodzielnie tego naprawić; należy odłączyć i ponownie podłączyć urządzenie, aby moduł planowania połączeń ponownie zaplanował transmisję zgodnie z
> nową wartością maksymalną.

**Karta sieciowa USB Ethernet jest ograniczona do 200 MB/s** przez sondę, niezależnie od jej
danych technicznych: tabela wydajności, która przekształca prędkość łącza na wartość ciągłą, jest
oparta na standardzie PCIe, a karta sieciowa USB podaje swoją prędkość łącza Ethernet, będąc jednocześnie ograniczona przez
magistrali USB i jej sterownika. Ograniczenie to jest wartością bezwzględną, a nie ułamkową — adapter USB 1 GbE
osiąga prędkość ~80 MB/s i nie ma to na niego wpływu.

#### Metody `ArraySession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Działa na żywo `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Jedna zsynchronizowana grupa przechwytywania. Zwraca `CaptureResult` (lista słowników klatek + `.skipped`). Poniżej znajdują się opcje eksportu. |
| `capture(..., smart=True)` | **Inteligentne przechwytywanie** — czeka na ustabilizowanie się AE we wszystkich kamerach, a następnie uruchamia się. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Najszybsze przechwytywanie: tylko dane surowe + przypisany odczyt DAQ (+ swobodny indeks łączny). Odzwierciedla przycisk „Najszybsze przechwytywanie” w interfejsie graficznym. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Pojedyncze / ciągłe / w odstępach czasu w jednej pętli ograniczonej. Zwraca `list[CaptureResult]`.**Wymaga `count` i/lub `duration_s`**, aby zakończyć działanie (SDK nie obsługuje skrótu Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Rozpocznij nagrywanie podglądu na żywo z połączonym indeksem do pliku wideo/GIF → `RecorderHandle`. Jeden rejestrator kompozytowy na tablicę. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Rozpocznij serię zdjęć surowych w formacie Bayer z dużą liczbą klatek na sekundę → `RecorderHandle`. Przetwórz offline za pomocą `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Przetwarzaj w trybie offline zapisany ciąg zdjęć w formacie RAW, tworząc skalibrowane pliki wideo. Blokuje działanie do zakończenia (`wait=True`) i zwraca `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Sprawdź status zadania kompilacji w trybie offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Zwolnij całą tablicę. |

Kontrola eksportu `capture()` (ten sam punkt końcowy, z którego korzysta GUI/CLI):

- `processing` / `levels` — `processing="all"` (lub `levels=["raw","radiance",…]`) zapisuje każdy odpowiedni typ eksportu dla każdej kamery; pojedyncza wartość `processing` zapisuje tylko ten poziom.
- `aligned=True` — dopasowuje eksport wszystkich elementów (z wyjątkiem danych surowych) do [profilu wyrównania](#array-alignment) (zrejestrowane współrzędnie); dane surowe pozostają bez zniekształceń, ale zawierają transformację w metadanych. Jeśli tablica nie ma profilu, następuje powrót do stanu niewyrównanego (z ostrzeżeniem wyświetlanym w wyniku `alignment`).
- `render_index=False` — pomija nakładkę wskaźnika wegetacji dla poszczególnych kamer; domyślnie renderuje ją tam, gdzie została skonfigurowana.
- `force_daq=True` — zapisuje przypisany odczyt DAQ/DLS jako plik towarzyszący `.daq`, nawet jeśli żaden z wybranych poziomów tego nie wymaga.

**Kompresja TIFF (pokrętło HTTP- tylko pokrętło):**`ArraySession.capture()` nie wysyła klucza `compression`, więc obowiązuje domyślne ustawienie zaplecza — `POST /api/camera/array/capture` odczytuje parametr treści `compression`, domyślnie `"deflate"` (bezstratny zlib L1 + predyktor poziomy, ~4,1 MB na klatkę w pełnej rozdzielczości). `"none"` zapisuje nieskompresowane dane (~6,3 MB/klatka) z**~5× szybszym zapisem** — oba są bezstratne i odczytują się identycznie podczas importu. SDK nie udostępnia dla tego żadnego kwargu; rozwiązaniem awaryjnym jest `chloros-cli lattice array-capture --compression none` lub surowy HTTP. DEFLATE również utrzymuje GIL Python, więc zapis skompresowanych danych nie jest równoległy w wątkach zapisu poszczególnych kamer — do ciągłego przechwytywania w pełnej rozdzielczości z 8 kamer z prędkością czujnika potrzebny jest `compression: "none"`. Szczegóły: [Odwołanie do CLI → przechwytywanie tablicowe](cli-reference.md).**Nadpisywanie eksportu dla poszczególnych elementów (tylko HTTP):**ten sam punkt końcowy akceptuje również `exclude_serials` (lista — usunięcie elementów z zapisanego zestawu; tablica nadal uruchamia się jako jedna zsynchronizowana grupa, a wykluczone elementy są zwracane w `excluded`), `serial_levels` (nadpisania na poziomie poszczególnych kamer w formacie `{serial: [level tokens]}`) oraz `serial_index` (nadpisania nakładki indeksu kamery w formacie `{serial: bool}`kamery). Są to parametry główne zgodne z GUI i**nie są to jeszcze argumenty klucz-wartość SDK**; elementy nieobecne w mapach są zastępowane wartościami `levels` / `render_index` obowiązującymi dla całej tablicy.

##### Sprawdzanie pominiętych kamer — `CaptureResult.skipped`

`ArraySession.capture()` zwraca `CaptureResult`, który jest podklasą `list`: iteruj po nim, indeksuj go, stosuj `len()` — każdy istniejący wzorzec nadal działa. Nowy kod może sprawdzić atrybut `.skipped`, aby zobaczyć, które kamery zostały wykluczone i dlaczego. Najczęstszym przypadkiem są kamery RGB w macierzy z mieszanymi filtrami, gdy żądane jest `processing="radiance"` lub `"reflectance"` — promieniowanie na piksel Bayera nie ma znaczenia w przypadku czujnika szerokopasmowego, więc backend pomija te kamery, zamiast generować bezsensowne dane.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeny przyczyny mają postać `<level>-not-applicable-to-rgb-cam` (jeden wpis na każdy pominięty poziom, z których każdy zawiera `level`). Pominięcia związane z odbiciem to: `reflectance-skipped-no-fresh-dls` (brak dostępnego aktualnego odczytu promieniowania opadającego), `reflectance-skipped-bound-daq-unavailable (…)` (nie udało się uzyskać dostępu do podłączonego urządzenia DAQ) oraz `dls-uncalibrated-band-<nm>` — pasmo leży w większości poza zakresem radiometrycznie skalibrowanym czujnika światła urządzenia DAQ (~374–974 nm), więc absolutny podział na podstawie współczynnika odbicia z modułu DAQ zostaje odrzucony, a klatka zostaje wyraźnie sprowadzona do poziomu odpowiedzi czujnika. Spośród dostępnych w sprzedaży modeli tylko F988 wywołuje ten błąd; obsługiwanym trybem pracy tej kamery jest schemat pracy z panelem odbiciowym.

Poziomy `processing`:

| Poziom | Wynik |
| --- | --- |
| `"raw"` | Jednokanałowy układ Bayera (kamery monochromatyczne: pojedyncze pasmo) bezpośrednio z czujnika. |
| `"debayered"` *(domyślnie SDK)* | 3-kanałowy BGR poprzez demosaikowanie biliniowe (kamery monochromatyczne: 1-kanałowa skala szarości). |
| `"radiance"` | float32 W/m²/sr/nm poprzez pełny łańcuch radiometryczny. Tylko w trybie wielospektralnym — kamery RGB są pomijane. |
| `"reflectance"` | uint16 0..32768 (gotowe do Pix4D); wymaga sparowania z systemem DAQ na żywo w celu uzyskania odniesienia bezwzględnego. Tylko tryb wielospektralny. |
| `"display"` | Pełny łańcuch zgodny z podglądem w interfejsie graficznym (CCM + WB + gamma zgodnie z profilem kamery). |
| `"all"` | **Jeden plik na każdy odpowiedni poziom** dla każdej kamery (zgodny z opcją „Capture All” / domyślnym ustawieniem CLI). Zwrócony plik `CaptureResult` zawiera wówczas jeden słownik klatki na `(cam, level)`, z poziomem w każdym słowniku; poziomy, które nie mają zastosowania, pojawiają się w `.skipped`. Odczyt DAQ użyty dla dowolnej klatki odbicia jest zapisywana jako plik towarzyszący `.daq`. |

> **Uwaga — wartość domyślna różni się od CLI.** Domyślną wartością `ArraySession.capture()` jest `processing="debayered"`; domyślna wartość polecenia `chloros-cli lattice array-capture`na `processing="all"`. Należy jawnie przekazać `processing="all"` z SDK, aby odzwierciedlić wielopoziomowe zapisywanie CLI/GUI.

### Tryby przechwytywania i rejestratory

Powierzchnia matrycy odzwierciedla panel przechwytywania w interfejsie graficznym: tryby pojedynczego, ciągłego, interwałowego i najszybszego migawki, a także dwa rejestratory (kompozytowe wideo na żywo oraz seria zdjęć w formacie surowym → ponowne przetwarzanie w trybie offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**to pętla trybu ciągłego/interwałowego z modelu SDK. Ponieważ nie ma funkcji `Ctrl+C` umożliwiającej przerwanie jej działania ze skryptu,**należy** przekazać `count` i/lub `duration_s` (pętla zatrzymuje się po osiągnięciu dowolnego z nich). `interval_s` jest mierzony od początku każdego przebiegu (zgodnie z interfejsem graficznym). Pozostałe argumenty kwargs są przekazywane bezpośrednio do `capture()`.
- **`record`** ma *poziom monitorowania*: przechwytuje wyświetlany na żywo złożony obraz z połączonymi indeksami, więc strumień połączony musi być otwarty, aby klatki mogły się pojawić. Jeden rejestrator kompozytowy na tablicę (wywołuje wyjątek, jeśli jeden już działa).
- **`burst` → `build_video`** to *poziom analityczny*: `burst` zapisuje surowe klatki wraz z-klatkowy manifest oraz jeden plik `.daq` na każdy odrębny odczyt DLS w ramach `<output>/bursts/<base>/` z pełną prędkością pętli przechwytywania (bez łańcucha, bez exiftool, bez podglądu na żywo). `build_video` dopasowuje czasowo każdą klatkę do najbliższego `.daq` i ponownie uruchamia łańcuch promieniowania/odbicia/indeksu w potoku importu. `products` to lista `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (domyślnie: połączony indeks). `burst().stop()` automatycznie generuje również połączoną wersję indeksu z najlepszym możliwym wynikiem, zwaną jako `build_job` w wyniku zatrzymania.

#### `RecorderHandle`

Zwracany przez `ArraySession.record()` i `ArraySession.burst()`. Należy go używać jako menedżera kontekstu w celu automatycznego zatrzymania po wyjściu z zakresu lub sterować nim ręcznie.

| Element | Opis |
| --- | --- |
| `job_id` | Identyfikator zadania backendowego (ciąg znaków). |
| `kind` | `"composite"` (z `record`) lub `"raw"` (z `burst`). |
| `start_stats` | Słownik zwrócony przez wywołanie `start`. |
| `result` | `None` podczas działania; końcowy słownik wyników zatrzymania po zakończeniu. |
| `stats(timeout=10.0)` | Aktualne statystyki zadania (zapisane klatki, rzeczywista liczba klatek na sekundę, czas, który upłynął). |
| `stop(timeout=60.0)` | Zatrzymuje rejestrator; zwraca i buforuje wynik końcowy. Idempotentne (drugie wywołanie zwraca wynik z pamięci podręcznej). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Dołączanie do już podłączonej tablicy — `attach_array`

Jeśli tablica jest już uruchomiona (została otwarta przez GUI, lub poprzednia sesja SDK wywołała funkcję `connect_array`), należy użyć funkcji `attach_array`, aby pobrać do niego uchwyt zamiast ponownego-łączenia. <sn><id>W takiej sytuacji</id></sn> `connect_array` zawsze zwraca błąd „Kamera  znajduje się<sn> już w <id>macierzy”, ponieważ wysłanie żądania POST do `/array/connect` dla elementu z puli nie jest idempotentne; `attach_array` odczytuje `/api/camera/array/list` i dopasowuje na podstawie identyfikatora tablicy (array_id) lub numerów seryjnych.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Wzorzec: Skrypty SDK współdziałające z graficznym interfejsem użytkownika powinny najpierw próbować `attach_array`, a jeśli w puli nie ma jeszcze żadnej tablicy, przejść na `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Ważne — zamknięcie menedżera kontekstu POWODUJE rozłączenie.**`ArraySession.disconnect()` zawsze wysyła żądanie POST do `/array/disconnect`; nie ma mechanizmu zabezpieczającego typu „attached-notjak w przypadku `CameraSession` / `DAQSensorSession`. Jeśli współdzielisz środowisko z interfejsem graficznym i nie chcesz likwidować tablicy przy wyjściu z zakresu,**nie używaj bloku `with`** — zachowaj uchwyt w zwykłej zmiennej i pomiń jawne wywołanie `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Narzędzie pomocnicze do analizy sieci

Przydatne przed otwarciem tablicy — pozwala sprawdzić, czy proponowane ustawienia będą pasować:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` jest jednym z `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (w przeciwnym razie `error`). `auto_capped_fps` oznacza, że żądana rozdzielczość pasuje do pierścienia RX tylko przy ograniczonej częstotliwości wyzwalania — zachowaj rozdzielczość i przejdź od `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (zobacz [Przykład 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Jak odczytywać projekcję** (ten sam model co w panelu ustawień macierzy GUI):

- **Seria (`frame_bytes_total`) jest sumowana dla każdej kamery w rzeczywistym formacie pikseli danej kamery.**Mono**M3M**przesyłają strumień Mono12 (2 B/px) niezależnie od przekazanej wartości `pixel_format`, więc klatka w pełnej rozdzielczości z 4 kamerami ma rozmiar**~25 MB** przy trzech kamerach mono, a nie ~12,6 MB, jak wynikałoby z założenia, że wszystkie są 8-bitowe. Backend rozpoznaje format każdej kamery na podstawie jej modelu.
- **Admittance (`burst_fits_nic_ring`) uwzględnia prędkość opróżniania**, a nie porównanie całej serii z pierścieniem: symulacja emisji pasuje, gdy host opróżnia pierścień odbiorczy szybciej niż kamery go wypełniają. Host 10G + kamery 1 GbE**przesyła** w pełnej rozdzielczości, nawet gdy burst przekracza pojemność pierścienia; host 1 GbE blokuje transmisję (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` to konserwatywny-odzyskiwania** — `max(readout+emit, N×emit)` z emisją na kamerę ograniczoną do łącza kamery 1 GbE, niezależnie od ekspozycji. Np. ~2,8 klatek na sekundę dla 4-kamerowej matrycy o pełnej rozdzielczości 12-bitowej (odpowiada zmierzonej w środowisku uruchomieniowym wartości ~2,7–3,0). Pełny model: [CLI Odniesienie → Model klatek na sekundę i serii zdjęć dla macierzy](cli-reference.md#array-fps--burst-model).
- **Nadmierna subskrypcja (`oversubscribed: true`) oznacza, że minimalna wartość N × na kamerę przekracza bezpieczny pułap zapobiegający kolizjom** — pola fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) przyjmują wartość 0, a automatyczne zmniejszanie rozmiaru/binning nie są w stanie tego naprawić (zmniejszają one liczbę bajtów na klatkę, a nie liczbę bajtów na sekundę w trybie pacing). Rozwiązaniem jest zmniejszenie liczby kamer, zastosowanie ramek jumbo lub szybsza karta sieciowa; `max_cams_collision_safe` zgłasza limit (6 kamer o pełnej rozdzielczości na 1 GbE przy 1500 MTU, 9 z ramkami typu jumbo). Odpowiedź zawiera również `aggregate_demand_bps`, `collision_safe_ceiling_bps` oraz `per_cam_floor_bps` (8 MB/s). Zobacz [Nadmierna subskrypcja](#over-subscription-the-per-cam-floor).

### Wykrywanie i wyświetlanie listy

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Macierze LATTICE uruchamiają ciągłą regulację ekspozycji (AE) w tle natychmiast po podłączeniu, ale nowo wycelowana scena potrzebuje chwili, aby osiągnąć stabilność. **Smart-Capture** to gotowe, wygodne rozwiązanie: sprawdza ekspozycję każdej kamery, czeka, aż macierz ustabilizuje się w całym oknie, a następnie uruchamia przechwytywanie. Działa to analogicznie do interfejsu graficznego: przycisk „„inteligentne” przechwytywanie wywołuje ten sam punkt końcowy zaplecza.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Podczas sterowania za pomocą `ChlorosProject` (następna sekcja) dostępnych jest więcej opcji:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Polityka inteligentnej ekspozycji (smart-AE) jest domyślnie konserwatywna. Zmniejsz wartość `exposure_tolerance_pct` w przypadku wymagających prac radiometrycznych; zwiększ ją w przypadku szybko zmieniających się scen, gdzie wystarczy, że obraz będzie „wystarczająco dokładny”.

---

## Sesje czujników DAQ

Trwała pula backendowa dla czujników spektralnych (DAQ-U przez USB, DAQ-M przez BLE, DAQ-E przez Ethernet). Odzwierciedla działanie kamery: inteligentne wykrywanie, ponowne wykorzystanie puli, idempotentne dołączanie.

### Inteligentne wykrywanie (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorytet: Ethernet → BLE → USB. Przekaż dowolną jawną wskazówkę, aby przypiąć protokół transportowy.

### Zablokowany protokół transportowy

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metody `DAQSensorSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Podsumowanie wpisu w puli (stan transmisji strumieniowej/rejestrowania, zakres długości fal, hash kalibracji, czas integracji, frame_avg, stan AE). |
| `latest(n=1, timeout=10.0)` | Zwraca maksymalnie N najnowszych ramek widma. |
| `stream_start()` / `stream_stop()` | Wznowienie / wstrzymanie transmisji strumieniowej (uchwyt pozostaje otwarty). |
| `record_start(output_dir=None, device_name=None)` | Rozpocznij rejestrowanie pliku .daq. Zwraca ścieżkę do pliku. Odmawia wykonania dla DAQ-U/M bez pakietu kalibracyjnego AWS (DAQ-E jest wyłączony z tego ograniczenia). |
| `record_stop()` | Zatrzymaj rejestrowanie. Zwraca `{path, rows}`. |
| `disconnect()` | Zwolnienie z puli. Nie wykonuje żadnej operacji w przypadku uchwytów podłączonych, ale niebędących własnością. |

> **Profile-korekcyjne (`cap_id`) nie są regulatorem SDK.** `connect_daq_sensor()` / `DAQSensorSession` nie udostępniają parametru `cap_id` ani metody `set_cap`. Wybierz profil korekcji limitu floty za pomocą CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) lub trasy zaplecza `/api/daq` HTTP (`/api/daq/connect` i `/api/daq/<id>/cap-id` akceptują `cap_id`).

### Wykrywanie — znalezienie adresu, z którym można się połączyć

`discover_daq_sensors()` skanuje USB / BLE / ETH w poszukiwaniu czujników, które *można* otworzyć. Jest to odpowiednik `discover_lattice_cameras()` dla DAQ i jedyny sposób na uzyskanie **adres MAC BLE urządzenia DAQ-M** — urządzenie DAQ-E ma nazwę hosta, a DAQ-U port COM, ale adres MAC nie jest ani wydrukowany na urządzeniu, ani wyświetlany przez system operacyjny.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Pole | Opis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / adres MAC BLE / nazwa hosta — przekaż do `connect_daq_sensor` jako `port=` / `mac=` / `eth_host=`. |
| `display` | Etykieta czytelna dla człowieka. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` lub `None` dla portu, który skanowanie możezidentyfikować (adaptery szeregowe USB są nie do odróżnienia bez sondy, więc nieznane elementy są wyświetlane, a nie ukrywane). |
| `extra` | Szczegóły dotyczące poszczególnych protokołów (nazwa reklamowana BLE, producent USB, adres IP/fw/… DAQ-E). Puste wartości są pomijane. |

| Parametr | Domyślna wartość | Opis |
| --- | --- | --- |
| `transports` | wszystkie trzy | Sekwencja (lub ciąg znaków w formacie CSV) ograniczająca skanowanie. Warto podać tę wartość, gdy wiesz, czego chcesz — BLE jest najwolniejszym ogniwem. |
| `scan_timeout` | 5 | Okno skanowania dla poszczególnych protokołów w sekundach; backend ogranicza wartość do zakresu 1–20. |
| `timeout` | 60.0 | HTTP – limit maksymalny dla całego wywołania (tak jak w innych miejscach w SDK). |
| `auto_start_backend` | `True` | Spawn lokalny backend, jeśli żaden nie jest uruchomiony. Nigdy nie uruchamia się dla zdalnego `backend_url`. |

> **Czujniki już otwarte w puli nie pojawiają się.** Podłączone urządzenie peryferyjne BLE przestaje nadawać sygnał, a otwarty port COM nie może być skanowany, więc wykrywanie wyświetla listę tego, co jest *dostępne do połączenia*. Pusty wynik zaraz po podłączeniu urządzenia jest oczekiwany — użyj `list_daq_sensors()` dla tego, co już posiadasz. Transporty, których skanowanie nie może zostać uruchomione (brak zainstalowanego bleak / zeroconf), są pomijane zamiast generować błąd, więc urządzenie bez Bluetooth nadal otrzymuje odpowiedzi dotyczące USB i ETH.

### Lista

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Współdziałanie z GUI / CLI

Jeśli w GUI jest już otwarty czujnik, wywołanie funkcji `connect_daq_sensor(port="COM3")` z poziomu Python zwraca uchwyt oznaczony jako `already_connected=True`. `disconnect()` sesji jest wówczas operacją bezczynną, dzięki czemu skrypt SDK nie wyrywa czujnika spod GUI przy zamknięciu oscyloskopu.

### Bezpośrednie– klasy sprzętowe (bez zaplecza)

`daq_sdk` jest ponownie eksportowany przez `chloros_sdk`, dzięki czemu można również sterować czujnikami w trybie end-to-end w ramach procesu bez zaplecza:

> **Dostępność:**`daq_sdk` jest dostarczany wraz z instalacją desktopową Chloros,**nie** z pakietem PyPI — `pip install chloros-sdk` udostępnia `lattice_sdk`, ale pomija `chloros_sdk.DAQ_AVAILABLE == False`. Przed użyciem tych klas należy sprawdzić ten znacznik; na hoście obsługującym wyłącznie pip należy sterować czujnikiem za pomocą [`connect_daq_sensor()`](#daq-sensor-sessions), który nie wymaga lokalnych bibliotek transportowych.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Jeśli chcesz współdzielić własność z interfejsem graficznym, wybierz ścieżkę smart-connect (`connect_daq_sensor`); w przypadku skryptów bezinterfejsowych, które są wyłącznymi właścicielami czujnika, użyj klas bezpośrednich.

---

## Automatyzacja projektów — `ChlorosProject`

Zapisany projekt Chloros to folder zawierający pliki `cameras.json` + `sensors.json` + `project.json`. `open_project` ładuje manifest, a `connect_all` uruchamia wszystkie zapisane urządzenia z ich zapisanymi ustawieniami — w takim samym stanie sprzętowym, jaki zapewniłby interfejs graficzny.

### Minimalny przykład

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Lub jako menedżer kontekstu:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metody `ChlorosProject`

| Metoda | Opis |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Wykrywa i łączy się z każdym zapisanym urządzeniem. Zwraca raport połączeń dla poszczególnych klas. Korzysta z uruchomionego backendu, jeśli taki nasłuchuje na `127.0.0.1:5000`; w przeciwnym razie bez ostrzeżenia przechodzi na bezpośrednie (bez zaplecza) sterowanie urządzeniami `lattice_sdk` — nigdy nie uruchamia zaplecza. |
| `disconnect_all()` | Zakończ wszystkie połączenia. |
| `capture_all(output_dir=".")` | Jedna ramka z każdej kamery + tablica + widmo z każdego czujnika. |
| `stream(camera, overlays=False, fps=10.0)` | Generator generujący klatki BGR `numpy` z nazwanej kamery (lub macierzy). `overlays=False` jest bezpośrednią pętlą przechwytywania `lattice_sdk` (macierze generują słowniki `{serial: frame}`). `overlays=True` kieruje dane przez `ChlorosLocal.camera_stream()` → strumień MJPEG `/api/camera/<serial>/stream-annotated` backendu, z zapisanym blokiem `ui.overlay` przekazywany jako parametry zapytania. Wymaga trybu backendowego i **kamery autonomicznej**: kamera w trybie bezpośrednim generuje błąd `RuntimeError` (moduł zaplecza nie może przejąć kamery należącej do tego procesu), a tablica generuje błąd `NotImplementedError` (nakładki kompozytowe dla każdej kamery — przesyłaj strumień dla elementu na podstawie nazwy). Odpowiednik jednorazowy: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Uruchom wyrównanie dla każdej aktualnie podłączonej macierzy. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Uruchom potok kalibracji / indeksowania na obrazach projektu (zawiera `ChlorosLocal.process`; te cztery są **jedynymi** akceptowanymi argumentami kluczowymi — `indices=` itp. wywołuje wyjątek `TypeError`; ustawia indeksy za pomocą `ChlorosLocal.configure()`). W sposób leniwy tworzy obiekt `ChlorosLocal()`, który automatycznie uruchamia backend. |

Atrybuty:
- `proj.cameras` — `Dict[str, CameraHandle]` indeksowany według nazwy ORAZ numeru seryjnego.
- `proj.arrays` — `Dict[str, ArrayHandle]` indeksowany według nazwy ORAZ identyfikatora tablicy.
- `proj.sensors` — `Dict[str, SensorHandle]` indeksowane według nazwy ORAZ identyfikatora slotu.
- `proj.config` — słownik `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Poziomy przetwarzania.** `capture()`, `grab()` oraz `frame_stream()` przyjmują ten sam token `processing`,
a łańcuch ma charakter kumulatywny — każdy poziom uruchamia wszystkie poziomy znajdujące się powyżej niego:

| Poziom | Wynik | Uwagi |
| --- | --- | --- |
| `raw` | 1-kanałowy Bayer, natywny dla czujnika | Brak demosaic. Nakładki nie są dostępne na tym poziomie. |
| `debayered` | 3-kanałowy BGR (**domyślnie**) | Demosaikowanie biliniowe. Jedyny poziom działający bez trybu backendowego. |
| `radiance` | float32, W/m²/sr/nm | Pełny łańcuch radiometryczny: demosaikowanie + rozdzielanie 3×3 (multispektralne) + DSNU + pole płaskie + skala NIST, z wyeliminowaniem ekspozycji i wzmocnienia, dzięki czemu wartości są bezwzględne. |
| `reflectance` | uint16, 32768 = 1,0 | Promieniowanie podzielone przez natężenie promieniowania padającego (ρ = π·L/E). Wymaga odczytu z DLS/DAQ — patrz uwaga poniżej. |
| `display` | 8-bitowy, zbliżony do sRGB | Renderowanie odpowiadające interfejsowi graficznemu: CCM + balans bieli + gamma poprzez aktywny profil kolorów kamery. |

Wszelkie wartości inne niż `debayered` wymagają trybu backendowego; kamera w trybie bezpośrednim generuje
`NotImplementedError`. `reflectance` wymaga użytecznego odczytu natężenia promieniowania padającego — punkt końcowy klatki automatycznie pobiera
zgromadzone dane DAQ automatycznie do gniazda DLS kamery, ale bez przypisanych danych DAQ łańcuch odrzuca
wyjście odbicia i uczciwie zaznacza obniżenie jakości w zwracanych metadanych, zamiast po cichu
zwracać produkt o niższej jakości.

> **Skala DN odbicia — nie należy jej zakodować na stałe.** Odbicie LATTICE wykorzystuje `32768` = ρ 1,0 i oznacza
> XMP `Chloros:PixelScale=32768`; reflektancja Survey3 wykorzystuje `65535` = ρ 1,0 i nie zawiera
> tagów `Chloros:*`. Odczytaj tag i podziel przez jego wartość. Jest on zdefiniowany w domenie uint16, więc pozostaje
> `32768` dla każdego formatu, który zmienia skalę (16-bitowy TIFF, 8-bitowy PNG/JPG, 32-bitowy procent) — najpierw znormalizuj
> zapisany typ danych z powrotem do uint16 (×257 z 8-bitowego, ×65535 z float). Jedyny wyjątek:
> dane z 8-bitowego źródła zapisane jako 8-bitowe TIFF są *obcinane*, a nie przeskalowywane, więc żadna skala ich nie opisuje
> — w takim przypadku Chloros całkowicie pomija `PixelScale` oraz krotkę MicaSense. Brakujący
> tag w pliku odbicia LATTICE należy traktować jako „brak prawidłowej skali”, a nie jako wartość domyślną.

> **Dane EXIF są przenoszone do eksportowanego pliku.** `process()` kopiuje blok GPS źródłowego ujęcia
> **oraz jego ExifIFD** do każdego produktu, więc eksportowane pliki zawierają `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` i `CameraSerialNumber`, a także
> georeferencje. `FocalLength` to plik, na podstawie którego Pix4D oblicza odległość próbkowania terenu — bez niego
> rekonstrukcja przyjmuje całkowicie błędną skalę (w jednym z przypadków zmierzonego terenu o powierzchni 411 m
> przekształciło się w teren o powierzchni 47,8 km). Kopia celowo nie zawiera `-all:all`: tagi strukturalne IFD0 zakłócają
> dane wyjściowe LATTICE, a `ExifImageWidth`/`Height` są wykluczone, ponieważ opisują źródło
> przechwycenia, a nie wyeksportowanego rastra.

Podflagi etapu przechwytywania (dotyczą poziomów radiometrycznych — `radiance`, `reflectance`, `display`):

| Flaga | Domyślna | Znaczenie |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + pole płaskie + rozdzielenie 3x3 + skala radiometryczna NIST. |
| `apply_white_balance` | `True` | Tablica LUT dla balansu bieli. Obsługa DLS, gdy moduł DAQ jest powiązany z kamerą. |
| `apply_index` | `False` | Ocena wskaźnika wegetacji. |
| `index_expression` | `None` | Formuła zastępcza.puste → automatycznie włącza wskaźnik. |
| `annotated` | `False` | Nakładanie elementów dekoracyjnych GUI (zebra/siatka/peaking). Niedostępne dla `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Typem zwracanej wartości jest `CapturePathMap`, a nie `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` to `Dict[str, Union[str, List[str]]]`: ścieżka jednopoziomowa
> `processing` przypisuje każdemu numerowi seryjnemu jedną ścieżkę, podczas gdy wielopoziomowy (`"all"` lub
> jawna lista `levels`) przypisuje mu **uporządkowaną listę** wszystkich produktów zapisanych dla tej
> kamery. Połączony obraz na żywo, jeśli jest transmitowany, trafia pod dodatkowym
> kluczem `"combined"`, a nie pod numerem seryjnym. Kod, który zakłada `str`, ulega awarii w przypadku
> formy listy bez żadnej kontroli typu— adnotacja przez pewien czas po wprowadzeniu formy listy wskazywała `Dict[str, str]`,
> dlatego istnieje ten alias. Należy normalizować
> w przypadku, gdy wymagana jest forma płaska:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Wyrównanie tablic

`ArrayHandle` udostępnia pełną powierzchnię wyrównania. Profile są domyślnie przypisane tylko do sesji — aby je zapisać trwale, należy jawnie wywołać `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Wyrównanie w momencie nawiązania połączenia

`connect_all(align=...)` może automatycznie wyrównać każdą tablicę w momencie nawiązania połączenia:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

W przypadku braku określenia stosowana jest funkcja `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Bezpośredni sprzęt (bez backendu)

Jeśli chcesz całkowicie wyeliminować zależność od backendu (CI, roboty bezinterfejsowe, rozwiązania wbudowane), należy bezpośrednio zaimportować `lattice_sdk` i `daq_sdk` — oba są ponownie eksportowane przez `chloros_sdk`. Ograniczenie dotyczące `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` znajduje się w pakiecie PyPI (ale wymaga obecności środowiska uruchomieniowego Arena SDK), podczas gdy `daq_sdk` jest dostarczany wyłącznie wraz z instalacją na komputery stacjonarne.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Ustawienia wstępne i wyzwalacz

Trzy z czterech ustawień wstępnych działają w trybie **free-run**: kamera naświetla obraz w sposób ciągły, a
`capture()` zwraca kolejną klatkę. Wyjątkiem jest `triggered` — uruchamia ono
aparat na sygnał krawędziowy na linii 2, więc nie rejestruje niczego, dopóki taki sygnał nie nadejdzie.

| Ustawienie wstępne | Wyzwalacz | Zastosowanie |
| --- | --- | --- |
| `default` | tryb swobodny | ogólne zastosowanie |
| `high_speed` | tryb swobodny-run | 8-bitowy, ograniczenie do 60 klatek na sekundę, krótka ekspozycja |
| `high_quality` | free-run | 12-bitowy, bez ograniczenia klatek na sekundę — typowy wybór do zdjęć |
| `triggered` | **uzbrojony, linia 2** | aparat jest podłączony do kabla synchronizacyjnego M8 i wyzwala go coś innego |

Jeśli wybierzesz `triggered` (lub samodzielnie ustawisz `trigger_mode="On"`) bez żadnego
sygnału sterującego na linii 2, każde `capture()` zakończy się przekroczeniem limitu czasu — słusznie, ponieważ poprosiłeś
kamerę o oczekiwanie. Komunikat SDK wyjaśnia to, gdy tak się dzieje; zobacz
[SC_ERR_TIMEOUT podczas przechwytywania](#direct-hardware-backend-free).

> **Uwaga — komunikaty „GVSP probe” / `SC_ERR_TIMEOUT -1011` pojawiające się podczas nawiązywania połączenia nie są błędami.**&gt; Podczas nawiązywania połączenia SDK próbuje wynegocjować**ramki jumbo** (pakiety GVSP o rozmiarze 9000 bajtów) w celu uzyskania większej przepustowości. W przypadku bezpośredniego połączenia punkt-punkt za pomocą karty sieciowej (np. adres lokalny `169.254.x.x`) sieć zazwyczaj nie obsługuje ramek typu jumbo, więc próba ta kończy się przekroczeniem limitu czasu i w dzienniku pojawiają się wpisy takie jak:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Jest to **przewidziane rozwiązanie awaryjne**: urządzenie SDK automatycznie powraca do standardowych pakietów o rozmiarze 1500bajtów, a kamera nadal łączy się normalnie (następujące po nim wiersze `[chunk-enable …]` są częścią normalnej sekwencji łączenia). Przechwytywanie nadal działa.
>
> Można pominąć tę sondę, ale **nie jest to tylko narzędzie wyciszające logi — wyłącza ona ramki jumbo.** Kamera odpowiada na pingi z opcją „Don&#x27;t-Fragment” tylko do 1500 bajtów, niezależnie od tego, jak dobra jest sieć, więc sam test ping nigdy nie wykryje ramek jumbo; ta sonda jest jedyną rzeczą, która może to wykryć. Wyłącz ją, a kamera będzie na zawsze wysyłać standardowe pakiety o rozmiarze 1500 bajtów w każdej sieci:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Warto to zrobić tylko w sieci, o której *wiesz*, że nie obsługuje pakietów typu jumbo, gdzie pozwala to zaoszczędzić około sekundy czasu połączenia na każdą kamerę. Ponieważ jest to rzeczywista zmiana, a nie tylko kosmetyczna, SDK informuje o tym teraz podczas użycia:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nie zmieniaj tego, chyba że masz ku temu powód.** Jeśli opcja pozostanie włączona, przy każdym połączeniu nastąpi ponowny pomiar rzeczywistej sieci: podłącz się do przełącznika obsługującego pakiety jumbo, a następne połączenie samoczynnie wykryje je, bez konieczności konfiguracji i restartu.
>
> Jeśli *chcesz* uzyskać przepustowość w trybie jumbo, włącz tryb jumbo w obie strony (MTU karty sieciowej 9000 + przełącznik, który je przepuszcza), lub ustaw stały rozmiar za pomocą `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, gdy wiesz, że łącze je obsługuje — choć lepiej jest używać opcji `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` dla poszczególnych poleceń zamiast ustawiać to na stałe, ponieważ stały rozmiar pomija testy i uniemożliwia dostosowywanie się do sieci znajdującej się przed nim. **Każde** urządzenie na ścieżce musi przepuszczać pakiety typu jumbo — w tym każdy rozgałęźnik lub iniektor PoE, co jest najczęstszą przyczyną, dla której konfiguracja, która w innym przypadku obsługiwałaby pakiety typu jumbo, nie może ich przesyłać.

> **Błąd `SC_ERR_TIMEOUT -1011` występujący podczas `capture()` / `grab*()` to inna sprawa — ten błąd jest prawdziwym błędem.**&gt; Powyższa uwaga dotyczy wyłącznie błędu `-1011` zarejestrowanego przez**sondę czasu połączenia**. Ten sam błąd zgłoszony w wyniku**przechwytywania** oznacza, że kamera połączyła się poprawnie, ale nie wysyła żadnych obrazów:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Cechą charakterystyczną jest kamera, której kanał *sterowania* działa poprawnie — wykrywanie przebiega pomyślnie, ustawienia oraz zapisy `[chunk-enable …]` kończą się sukcesem — podczas gdy *każda* klatka przekracza limit czasu.
>
> **Najczęstszą przyczyną jest to, że kamera jest ustawiona na wyzwalanie sprzętowe.** W przypadku kodów `trigger_mode="On"` i `trigger_source="Line2"` kamera nie wysyła żadnych sygnałów, dopóki na kablu synchronizacyjnym M8 nie pojawi się zbocze sygnału. Jeśli nie ma kabla obsługującego tę linię, każde pobranie danych trwa w nieskończoność. Kamera nie jest uszkodzona, a sieć działa prawidłowo — robi dokładnie to, co jej nakazano.
>
> Kod błędu `CameraSettings()` oraz `default` / `high_speed` / `high_quality` są zaprogramowane na tryb swobodny, a przechwycenie, które przekroczy limit czasu podczas stanu gotowości, wyświetla odpowiedni komunikat zamiast samego kodu `-1011`. `PRESETS["triggered"]` uzbraja linię 2 zgodnie z założeniami.
>
> Aby wymusić tryb swobodnej pracy dowolnej kamery:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Jeśli nadal występuje przekroczenie limitu czasu przy `trigger_mode="Off"`, oznacza to, że kamera faktycznie nie przesyła danych — prześlij nam log oraz `ip link show`.

#### Profile kolorów (podgląd na żywo z RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` wybiera profil kolorów wyświetlacza dla **podgląd na żywo** w kamerach RGB (kamery wielospektralne ignorują to ustawienie):

| Profil | Znaczenie |
| --- | --- |
| `raw` | Całkowite pominięcie łańcucha radiometrycznego. |
| `linear` | DSNU + flat + WB, bez CCM, bez gamma. |
| `natural` | Liniowy + zmierzony CCM + gamma sRGB, wyłącznie z podstawową obróbką końcową (wygładzanie chrominancji + desaturacja świateł) — realistyczne ustawienie domyślne. |
| `enhanced` | `natural` wraz z pełną obróbką końcową typu hub-parity (usuwanie fringów, intensywność, lokalny kontrast CLAHE). Bogatszy wygląd przy mniej więcej **dwukrotnie wyższym koszcie obróbki na klatkę**, a więc niższej częstotliwości odświeżania na żywo. |
| `custom_temp` | `natural`, ale balans bieli przypisany do `custom_cct_k` w kelwinach (DLS ignorowane; ograniczone do zakresu 2000–10000 K po stronie). |

Profil ten służy wyłącznie do regulacji szybkości i wyglądu **podczas podglądu na żywo**: zapisane ujęcia zawsze uzyskują pełne, bogate wykończenie niezależnie od wybranego profilu, więc wybór `natural` w celu odzyskania czasu klatki nie obniża jakości tego, co trafia na dysk. Nieznany profil podnosi wartość `ValueError`; gdy backend chloros jest dostępny, zmiana jest również wysyłana do niego w trybie POST, więc następna klatka podglądu to odzwierciedla (użytkownicy bezpośredniego SDK bez modułu zaplecza nadal otrzymują zmiany ustawień).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Kamery monochromatyczne (M3M) i `Calibration`

Kamera monochromatyczna **M3M** (`M3M-<lens>-F<wavelength>`) jest jednopasmowa: jedna płaszczyzna w skali szarości, bez mozaiki Bayera, bez macierzy przesłuchu spektralnego 3×3. `Calibration` rozpoznaje ją i udostępnia flagę `is_mono`. Odbicie nadal obowiązuje jako mapa radiometryczna dla poszczególnych pasm (matryca rozdzielająca to macierz tożsamościowa), ale obliczenia wielopasmowe na pojedynczej kamerze dają sensowny wynik, a nie zwracają nonsensu:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Aby zbudować indeks wegetacji przy użyciu sprzętu monochromatycznego, należy połączyć kilka kamer M3M działających w różnych długościach fal w wyrównany stos wielopasmowy (zobacz [Wyrównanie macierzy](#array-alignment)) i obliczyć indeks dla całego stosu zamiast dla pojedynczej kamery.

Tryb bezpośredni DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Akceptowane klucze `apply_sensor_settings`**— dokładnie `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; wycofane na rzecz `cap_id`), `filter_model` (DAQ-M) oraz `cap_id` (wszystkie rodzaje DAQ; `None`/`""`/`"none"` = sam czujnik, bez korekcji pokrywy). Nieznane klucze są**ignorowane bez komunikatu** — np. `{"integration_time": 64}` nie wykonuje żadnej czynności (musi to być `integration_time_ms`). Zwraca `{"applied": [...], "errors": {...}}` i nigdy nie zgłasza wyjątku.

`chloros_sdk` ponownie eksportuje tylko powierzchnię rdzenia używaną powyżej. Pełna wersja `daq_sdk` publiczna API (22 nazwy) dodaje następujące elementy — należy je zaimportować bezpośrednio z `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Wyjątki

Przechwytuj klasę bazową, aby obsłużyć „wszelkie błędy Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` i `ChlorosConfigurationError` są eksportowane na najwyższym poziomie wraz z pozostałymi; można je również zaimportować z `chloros_sdk.exceptions`, jak pokazano.

Hierarchia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Przykłady kompleksowe

### 1. Przetwarzanie folderu z niestandardowym paskiem postępu

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Tablica LATTICE na żywo → Odbicie + odniesienie DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Kampania przechwytywania danych w ramach projektu

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Strumień klatek z wielu kamer → potok NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skrypt przechwytywania bez interfejsu graficznego, bezpośrednio na sprzęcie (bez zaplecza)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Sprawdzanie możliwości przed podłączeniem macierzy 4 kamer

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Odpowiednik receptury przechwytywania (czysto Python)

Język DSL receptury CLI ma bezpośredni odpowiednik w Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatyczne uruchamianie backendu

Punkty wejścia funkcji smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` oraz `discover_lattice_cameras` — są lekkimi klientami typu HTTP, które zakładają, że backend nasłuchuje na adresie `127.0.0.1:5000` (domyślny adres URL interfejsu Smart-Connect). Gdy GUI lub CLI już działa, jeden z nich jest uruchomiony. W przypadku samego skryptu może go nie być — więc funkcje te **automatycznie uruchamiają dołączony plik binarny serwera** (bezokienkowy, tak samo jak robi to `ChlorosLocal`), a następnie czekają do `backend_startup_timeout` na jego uruchomienie.

Zasady:

- **Uruchamiany jest wyłącznie lokalny proces URL.** `backend_url` wskazujący na `localhost` / `127.0.0.1` / `[::1]` jest dopuszczalny; każdy inny host jest traktowany jako maszyna należąca do kogoś innego i nigdy nie jest uruchamiany.
- **Backend pozostaje uruchomiony w celu ponownego wykorzystania** (tak samo jak w przypadku CLI) — po zakończeniu działania skryptu nie następuje domyślne wyłączenie. Ponowne uruchomienie skryptu powoduje ponowne wykorzystanie aktywnego backendu.
- **Można zrezygnować z tej opcji za pomocą `auto_start_backend=False`** w dowolnym z tych wywołań (np. gdy wskazano zdalny backend lub gdy samemu zarządza się cyklem życia backendu).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Jeśli nie można zlokalizować lub uruchomić dołączonego pliku binarnego, kolejne wywołanie HTTP generuje obsługiwany błąd `ChlorosConnectError`, **zależny od platformy**, zamiast zwykłego komunikatu o odmowie połączenia — w systemie Windows kieruje użytkownika do aplikacji desktopowej lub polecenia `chloros-cli`; w przypadku Linux (bez GUI) wskazuje na polecenie `chloros-cli` lub `.deb`.

---

## Środowisko i nagłówki

SDK oznacza każde wywołanie zaplecza HTTP za pomocą `X-Chloros-Client: sdk`. Zaplecze stosuje zasady licencjonowania SDK/CLI (wymagane jest zalogowanie się **oraz** płatny plan Chloros+), a nie ścieżkę bezpłatnego poziomu w interfejsie graficznym. Jest to ustawiane automatycznie w momencie importu — nie musisz nic robić.

`http://localhost` i `http://127.0.0.1` są wykrywane jako lokalny backend. Wywołania do innych hostów (np. do własnej usługi analitycznej) pozostają bez zmian.

Aby zastąpić backend URL, należy przekazać `backend_url=` (lub `api_url=` na `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` bez pętli zwrotnej dociera wyłącznie do zaplecza źródłowego/urządzeniowego — dostarczone zaplecza wiążą się wyłącznie z pętlą zwrotną; wzorzec tunelu opisano w sekcji Tryb zdalnego zaplecza.)

---

## Wersjonowanie i kompatybilność

- Wersja SDK jest udostępniana jako `chloros_sdk.__version__`.
- SDK dostosowuje zachowanie do wersji dołączonego backendu. Łączenie starszego SDK z nowszym backendem zazwyczaj działa (punkty końcowe są kompatybilne z nowszymi wersjami), ale połączenie nowszego SDK ze starszym backendem może powodować błędy `404` w nowych punktach końcowych — należy zaktualizować aplikację komputerową, aby zapewnić zgodność.
- Interfejs Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) oraz punkt końcowy analizy sieciowejanalizy sieciowej zwracają stabilne schematy JSON; nowe pola są dodatkowe.

---

## Wskazówki dotyczące rozwiązywania problemów

- **`ChlorosAuthenticationError: Login required`** → Uruchom `chloros-cli login EMAIL PASSWORD` raz na tym komputerze lub zaloguj się za pomocą aplikacji komputerowej Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkcja Smart-Connect automatycznie uruchamia lokalny backend, więc ten komunikat pojawia się tylko wtedy, gdy nie można znaleźć lub uruchomić dołączonego pliku binarnego (np. na hoście obsługującym wyłącznie pip, bez pakietu desktopowego). Komunikat uwzględnia platformę: w systemie Windows otwórz aplikację desktopową lub uruchom dowolne polecenie `chloros-cli`; w systemie Linux uruchom polecenie `chloros-cli` (nie ma interfejsu graficznego) lub zainstaluj `.deb`. W przypadku zdalnego backendu przekaż `backend_url=` (oraz `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** podczas importu → `lattice_sdk` nie udało się załadować (zazwyczaj nie są zainstalowane biblioteki DLL środowiska uruchomieniowego Arena SDK). Powierzchnia nieobjęta kamerą nadal działa.
- **Funkcja Array connect zwraca rozdzielczość niższą od natywnej**→ Funkcja smart-prep backendu automatycznie zmniejsza rozmiar klatki, aby dopasować ją do przepustowości łącza. Użyj `analyze_array_network()`, aby sprawdzić przyczynę, a następnie zaktualizuj łącze, zaakceptuj zmniejszenie rozmiaru lub przekaż `force_tier="slip-emit-and-capture"` w celu sekwencyjnego przechwytywania. Zmniejszenie rozmiaru**nie** obejmuje zagregowanego nadmiernego przydziału (`oversubscribed: true`, pola fps 0): zbyt dużej liczby kamer dla łącza nie da się naprawić za pomocą binningu/ROI — zmniejsz liczbę kamer, włącz ramki jumbo lub przejdź na szybszą kartę sieciową (zobacz [Nadmierna subskrypcja](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` zgłasza, że pierścień odbiorczy karty sieciowej jest zbyt mały (~0,26 MB) / bramki połączeń wyświetlają komunikat „FRAMES WILL DROP”** → Pierścień odbiorczy karty sieciowej hosta ma wartość domyślną (często resetowaną do 32 po aktualizacji sterownika karty). W przypadku karty Realtek USB 10GbE należy ustawić `ReceiveBufferLen=256` i `PendingReceives=64` (z podwyższonymi wartościami), a następnie zrestartować backend, aby ponownie odczytał pierścień. Pełna procedura: [Odnośnik CLI → Konfiguracja i dostrajanie karty sieciowej hosta](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host zawiesza się podczas ponownego uruchamiania/wyłączania, później pojawiają się błędy WMI `Invalid class` / karta sieciowa nie włącza się** → Przestarzały sterownik USB 10GbE powodujący błąd `DRIVER_POWER_STATE_FAILURE` (niebieski ekran śmierci (BSOD) `0x9F`). Zaktualizuj sterownik karty sieciowej do najnowszej wersji (≥ 2026) i ponownie zastosuj ustawienia pierścienia odbiorczego. Zobacz [CLI – Informacje → Konfiguracja i dostrajanie karty sieciowej hosta](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Odmowa pomiaru odbicia** → Aby uzyskać odbicie w skali bezwzględnej, do kamery (lub matrycy) musi być przypisany aktywny moduł DAQ. Przypisz go za pomocą interfejsu graficznego lub użyj `processing="radiance"` (W/m²/sr/nm), który nie wymaga sparowanego czujnika.
- **Rejestracja `smart=True` trwa dłużej niż oczekiwano** → Zbieżność AE zależy od dynamiki sceny; należy zwiększyć wartość `exposure_tolerance_pct` lub skrócić `stability_window_s`, jeśli chcesz uzyskać szybszy (mniej stabilny) wyzwalacz.

---

## Zobacz też

- [Podręcznik CLI](cli-reference.md) — każda podkomenda CLI odpowiada wywołaniu SDK.
- [Przewodnik po czujnikach DAQ](../daq/README.md) — zasady okablowania, kalibracji i rejestracji danych dla poszczególnych czujników.
- Dokumentacja online: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
