# API: Python SDK

{% hint style="info" %}
**Szukasz pełnej dokumentacji API?** Ta strona stanowi praktyczny samouczek. Wszystkie publiczne klasy, metody, dokładne sygnatury oraz przykłady, które można skopiować i wkleić, znajdują się w [dokumentacji SDK](reference/sdk-reference.md), która jest zoptymalizowana pod kątem asystentów AI.**Korzystasz z asystenta AI?** Wklej ten URL do czatu, aby uzyskać dostęp do pełnej, aktualnej wersji Chloros 1.3.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Każda strona niniejszego podręcznika jest dostępna w formacie surowego markdownu pod adresem o nazwie składającej się z małych liter + `.md`, a cały podręcznik jest zindeksowany pod adresem `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) umożliwia sterowanie wszystkimi funkcjami aplikacji desktopowej z poziomu języka Python: przetwarzanie obrazów w trybie wsadowym, sterowanie kamerą LATTICE i matrycą w czasie rzeczywistym, sesje DAQ z czujnikami światła oraz automatyzację zapisanych projektów. Jest to cienka warstwa nakładająca się na ten sam lokalny backend, z którego korzystają interfejs graficzny (GUI) i interfejs wiersza poleceń (CLI) (HTTP pod adresem `127.0.0.1:5000`), dzięki czemu zachowanie jest identyczne we wszystkich trzech środowiskach.

## Instalacja

SDK jest dostarczany wraz z pakietem Chloros, który zawiera również backend przetwarzania oraz środowiska uruchomieniowe sprzętu i zawsze jest z nim zgodny pod względem wersji.

**Krok 1 — Zainstaluj Chloros.** Windows: uruchom instalator wersji desktopowej (domyślna ścieżka `C:\Program Files\Chloros\`) ze strony [Pobierz](download.md). Linux: zainstaluj pakiet `.deb` za pomocą `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalacja w systemie Linux](linux/linux-installation.md)).**Krok 2 — Sprawdź SDK** (Python 3.7+). Zazwyczaj nie trzeba niczego instalować:

* **Windows:** instalator instaluje dołączony pakiet SDK wheel w systemowym Python (jeśli Python nie jest zainstalowany, ten krok jest pomijany).
* **Linux:** pakiet `.deb` instaluje SDK dla systemowego `python3` w `/usr/lib/python3/dist-packages`. Wymagania, `python3-numpy` i `python3-requests`, są zalecane przez pakiet: `apt install` je zawiera, a `dpkg -i` nie.
* **Środowisko virtualenv** nie widzi pakietów systemowych; zainstaluj w nim dołączony plik wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` w systemie Linux lub `chloros_sdk-*.whl` w folderze `sdk` w folderze instalacyjnym systemu Windows (domyślnie `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Na komputerze bez pakietu Chloros, którego SDK steruje backendem na innym komputerze (zobacz [Co daje instalacja wyłącznie za pomocą pip](#what-a-pip-only-install-gets)), zainstaluj go z PyPI: `pip install chloros-sdk`. PyPI jest aktualizowane przy wydaniu nowych kompilacji, dzięki czemu zawiera najnowszą stabilną wersję.

**Krok 3 — Zaloguj się raz na każdym komputerze:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Dane uwierzytelniające są buforowane w `~/.chloros/` (na obu platformach). W systemie Windows można równie dobrze zalogować się za pomocą karty „Użytkownik” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> w aplikacji komputerowej. SDK wymaga płatnego planu Chloros+ — zobacz [Wymagania licencyjne](#license-requirement) poniżej.

| Wymaganie | Szczegóły |
| --- | --- |
| **Zainstalowany Chloros** | Windows: instalator na komputery stacjonarne; Linux: pakiet `.deb` (zawiera plik binarny zaplecza oraz SDK) |
| **Język Python** | 3.7 lub nowszy (opracowano i przetestowano na wersji 3.10) |
| **System operacyjny** | Windows 10/11 64-bitowy, Ubuntu 22.04 LTS lub nowszy albo NVIDIA Jetson (JetPack 6) |
| **Licencja** | Aktywne konto w Chloros+, dowolny płatny plan (Copper lub wyższy) |

## Sukces w 60 sekund

Jedno wywołanie tworzy projekt, importuje folder, konfiguruje przetwarzanie i uruchamia potok — automatycznie uruchamiając backend, jeśli jeszcze nie działa:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(W systemie Linux należy używać ścieżek Linux: `/home/user/drone_images/flight001`. Pakiet SDK działa identycznie na obu platformach.)

Przetwarzasz folder z danymi z LATTICE? Użyj opakowania dostosowanego do LATTICE — stosuje ono odpowiednie ustawienia domyślne (brak wykrywania panelu docelowego, standardowy algorytm debayeringu):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — pełna kontrola nad potokiem

W przypadku zadań wykraczających poza polecenie jednowierszowe należy użyć `ChlorosLocal`. Wykorzystuje on już uruchomiony backend (aplikacji desktopowej) lub uruchamia zainstalowany backend przy pierwszym użyciu (`auto_start_backend=True`; w systemie Windows z folderu `backend` w folderze instalacyjnym Chloros, domyślnie `C:\Program Files\Chloros\backend\`), tworzy i konfiguruje projekty, monitoruje postęp oraz zwraca podsumowanie po zakończeniu działania.

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

{% hint style="info" %}
Należy zachować domyślny `http://127.0.0.1:5000` zamiast zastępować go `localhost` — w systemie Windows `localhost` najpierw przekierowuje do `::1`, co kosztuje około 2 sekund na żądanie w przypadku backendu obsługującego wyłącznie IPv4.
{% endhint %}

Używaj go jako menedżera kontekstu. Opuszczenie bloku kończy korzystanie z backendu przez tę instancję, a nie sam backend: ten pozostaje uruchomiony dla następnego skryptu i wyłącza się samoczynnie po 30 minutach bezczynności. `shutdown_backend()` zatrzymuje instancję, która została uruchomiona.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` akceptuje następujące słowa kluczowe: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` oraz `custom_settings`. Główne wartości:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Regulatory specyficzne dla LATTICE (`input_level`, `radiometric_output` oraz rodzina `array_alignment*`) są udokumentowane wraz z pełnymi tabelami wartości w [Podręczniku SDK](reference/sdk-reference.md#supported-values).

### Monitorowanie postępu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Odczytywanie podsumowania po zakończeniu uruchomienia — oraz wykrywanie pustych uruchomień

Po zakończeniu `process()` dołącza podsumowanie przetwarzania backendu jako `result["summary"]`. Każdy wpis w `summary["hints"]` to pełne zdanie wyjaśniające wszelkie istotne kwestie — na przykład, dlaczego uruchomienie nie dało żadnego wyniku — a każda wskazówka jest również ponownie wysyłana jako plik `UserWarning` w języku Python, dzięki czemu puste uruchomienia są samodiagnozujące, nawet jeśli nigdy nie sprawdzisz słownika:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` nie generuje wyjątku, gdy uruchomienie nie tworzy żadnych obrazów.** Jest to jedyne miejsce, w którym SDK i CLI celowo się różnią: `chloros-cli process` traktuje sytuację „zaprošono o produkty, ale żadnego nie zapisano” jako błąd i kończy działanie z wynikiem niezerowym, podczas gdy SDK kończy działanie normalnie i zgłasza ten stan poprzez `summary` / wskazówki. Jeśli Twój potok ma się zatrzymać w przypadku pustego przebiegu, sprawdź to samodzielnie — przeanalizuj `summary` (lub policz pliki w folderze projektu), zamiast polegać na wyjątku.
{% endhint %}

## Smart Connect — sprzęt działający na żywo

Trzy pomocnicze programy otwierają trwałe sesje w puli sprzętowej backendu — tej samej puli, z której korzystają CLI, inne skrypty oraz zakładki kamer w GUI, dzięki czemu skrypty SDK współistnieją z nimi bez rywalizacji o porty szeregowe lub przepustowość sieci. Wszystkie trzy automatycznie uruchamiają lokalny backend, jeśli żaden nie jest uruchomiony. Panel DAQ interfejsu graficznego nie jest częścią tej puli: przed otwarciem czujnika światła ze skryptu należy zamknąć go w interfejsie graficznym.

### Pojedyncza kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Ścieżki względne odnoszą się do lokalizacji, w której uruchamia się skrypt (zmiana wprowadzona w wersji 1.3.0).** W przypadku lokalnego backendu ścieżka względna `output_dir` (w tym domyślna `"output"`) — oraz pozostałe ścieżki otwierane przez backend (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — są rozpoznawane względem katalogu roboczego skryptu, więc `cam.capture("output/")` zapisuje do `<current folder>/output/`. W wersji 1.2.0 zainstalowany backend rozpoznawał je w swoim własnym folderze instalacyjnym i zgłaszał błąd „Odmowa dostępu”. W przypadku zdalnego backendu ścieżki są wysyłane zgodnie z wpisanym tekstem.
{% endhint %}

### Zsynchronizowana tablica — `connect_array`

`connect_array` jest zalecanym punktem wejścia dla zestawów wielokamerowych. Uruchamia ten sam proces inteligentnego przygotowania co interfejs graficzny: analizę sieci, automatyczny wybór warstwy synchronizacji, synchronizację czasu PTP, wybór formatu pikseli dla poszczególnych kamer, inicjowanie AE oraz uzbrojenie wyzwalacza GPIO. **Pierwsza kamera szeregowa pełni rolę urządzenia nadrzędnego** (wysyła impuls wyzwalający sprzęt); pozostałe są urządzeniami podrzędnymi.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Dodaj `smart=True` do dowolnego przechwytywania tablicowego, aby przed wyzwoleniem poczekać na ustabilizowanie się automatycznej ekspozycji we wszystkich kamerach.

Nowość w wersji 1.3.0: `ArraySession` również dostarcza zsynchronizowane klatki bezpośrednio, bez zapisywania plików:

* `arr.grab()` zwraca jedną klatkę z każdej kamery, wszystkie z jednego wyzwolenia, jako tablice numpy (`{serial: array}`); `.group_id` to wspólny sygnał wyzwalający, a `.errors` określa każdą pominiętą kamerę.
* `arr.frame_stream(fps=5.0)` zwraca jeden taki zestaw na każdy takt, do przetwarzania na żywo w Twoim własnym potoku.
* `arr.continuous("output/")` uruchamia własną funkcję ciągłego przechwytywania backendu (tryb ciągły w interfejsie graficznym) i zwraca `ContinuousHandle` wraz z `status()`, `wait()` oraz `stop()` — należy go używać zamiast pętli przechwytywania, gdy konieczne jest zapisywanie każdego wyzwolenia z dużą częstotliwością.

Informacje na temat trybów przechwytywania (Pojedynczy / Ciągły / Interwałowy / Najszybszy), rejestratorów, funkcji „burst-to-video” oraz wyrównywania tablic można znaleźć w [Podręczniku SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Czujnik światła DAQ — `connect_daq_sensor`

Bez argumentów funkcja `connect_daq_sensor()` automatycznie wykrywa protokół transmisji (kolejność priorytetów: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Każda ramka zawiera 135-punktową wartość `spectrum` (W/m²/nm po kalibracji), flagę `is_saturated` oraz CIE `x`, `y`, `z`. Aby przypisać konkretny czujnik lub protokół transmisji, należy przekazać jedną wyraźną wskazówkę:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Należy pamiętać, że profile korekcji jasności (`cap_id`) **nie** są elementem sterującym w SDK — należy je wybierać za pomocą `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Zapisane projekty — `open_project`

Zapisany projekt Chloros zachowuje informacje o podłączonym sprzęcie (`cameras.json` + `sensors.json` wraz z `project.json`), a `chloros_sdk.open_project(path)` może natychmiast ponownie podłączyć swoje kamery, matryce i czujniki światła oraz sterować przechwytywaniem na podstawie nazwy urządzenia. Czujniki światła ponownie łączą się za pośrednictwem uruchomionego backendu Chloros, zachowując swoje zapisane ustawienia i limit. Zobacz [Automatyzacja projektu](reference/sdk-reference.md#project-automation--chlorosproject) w dokumentacji.

## Co zapewnia instalacja wyłącznie z pakietem pip

Przed użyciem powierzchni sprzętowych sprawdź flagi dostępności na poziomie modułów:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hoście z **wyłącznie** `pip install chloros-sdk` i bez pakietu pulpitu Chloros:

* Moduły `ChlorosLocal`, `process_folder` i `process_lattice_capture` **nie** działają — wymagają pliku binarnego zaplecza dostarczanego w instalatorze środowiska graficznego.
* Narzędzia pomocnicze smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) są czystymi klientami HTTP, więc działają w połączeniu z serwerem zaplecza na innym komputerze — jednak dostarczone moduły backendowe łączą się wyłącznie z pętlą zwrotną, więc należy samodzielnie przekierować port (np. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) oraz przekazać `backend_url="http://127.0.0.1:5000"` wraz z `auto_start_backend=False`. Zobacz [Tryb zdalnego serwera zaplecza](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Klasy LATTICE obsługujące sprzęt bezpośrednio (`LatticeCamera`, `CameraPool`, …) są importowane, ale wymagają środowiska uruchomieniowego Arena SDK z pakietu na komputery stacjonarne — bez niego `CAMERA_AVAILABLE` jest `False`.
* Czujniki DAQ są sterowane wyłącznie poprzez (tunelowany) backend — `connect_daq_sensor()` lub `proj.sensors` projektu; backend otwiera czujnik na swojej własnej maszynie.

## Asystenci AI — `chloros-mcp`

SDK zawiera `chloros-mcp`, serwer MCP, który umożliwia aplikacjom Claude Desktop, Claude Code, Cursor i innym klientom MCP sterowanie Chlorosem: przetwarzanie obrazów oraz obsługę kamer LATTICE, matryc i czujników światła DAQ. Jego narzędzia obejmują funkcje tego SDK. Instaluje się wraz z SDK, więc w przypadku Claude Code konfiguracja sprowadza się do jednego wiersza:

```bash
claude mcp add chloros -- chloros-mcp
```

Inne klienty: [Korzystanie z SDK z asystentami AI](ai-assistants.md). Opcje i wszystkie narzędzia: [Serwer MCP (asystenci AI)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Wymagania licencyjne

Dostęp do SDK wymaga aktywnego loginu do Chloros+ na dowolnym płatnym planie — **Copper lub wyższym**(Copper / Bronze / Silver / Gold); bezpłatny plan Iron nie zapewnia dostępu do SDK ani CLI. Kontrola odbywa się**po stronie serwera**: każde żądanie SDK musi zawierać zarówno aktywną sesję, jak i płatny plan, w przeciwnym razie backend zwraca błędy `403` / `PLAN_UPGRADE_REQUIRED` (generowany jako `ChlorosLicenseError` przez `ChlorosLocal` oraz jako `ChlorosConnectError` przez funkcje pomocnicze `connect_*`). Wylogowany użytkownik otrzymuje błąd `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ponowne uruchomienie `chloros-cli login` rozwiązuje pierwszy przypadek, ale nie drugi.

Korzystanie w trybie offline działa w ramach okresu karencji planu: poziom dostępu jest odczytywany z pamięci podręcznej walidacji serwera (5 minut) lub z pamięci podręcznej podpisanej licencji przypisanej do urządzenia (30 dni w przypadku planów miesięcznych; do wygaśnięcia subskrypcji w przypadku planów rocznych). Po upływie okresu karencji plan przechodzi w tryb bezpłatny, a dostęp do SDK zostaje wstrzymany do momentu, gdy urządzenie nawiąże połączenie z serwerem. Kod `chloros-cli status` pozostaje dostępny w ramach bezpłatnego poziomu, więc przyczyna jest zawsze widoczna. Zobacz [Logowanie do Chloros+](chloros+-login.md).

## Wyjątki

Przechwytuj klasę bazową, aby obsłużyć „wszelkie nieprawidłowości w Chloros”:

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

Wszystkie wyjątki potoku (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) wywodzą się z `ChlorosError`. Jedna pułapka: `ChlorosConnectError` — generowany wyłącznie przez `connect_camera` / `connect_array` / `connect_daq_sensor` — wywodzi się z zwykłego `Exception`, **a nie** z `ChlorosError`, więc `except ChlorosError` nie wykryje tego błędu. Pełna hierarchia znajduje się w [Podręczniku SDK](reference/sdk-reference.md#exceptions).

## Zobacz też

* [Podręcznik SDK](reference/sdk-reference.md) — kompletna powierzchnia API, zoptymalizowana pod kątem asystentów AI.
* [Dokumentacja CLI](reference/cli-reference.md) — każde podpolecenie CLI odpowiada wywołaniu SDK.
* [Korzystanie z Chlorosa z asystentami AI](ai-assistants.md) — konfiguracja `chloros-mcp` i przykłady monitów.
* [Pobierz](download.md) — instalatory dla systemów Windows i Linux.
