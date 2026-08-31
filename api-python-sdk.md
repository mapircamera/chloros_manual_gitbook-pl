# API : Python SDK

{% hint style="info" %}
**Szukasz pełnej dokumentacji API?** Ta strona stanowi praktyczny poradnik. Wszystkie klasy publiczne, metody, dokładne sygnatury oraz przykłady, które można skopiować i wkleić, znajdują się w [Podręczniku SDK](reference/sdk-reference.md), który jest zoptymalizowany pod kątem asystentów AI.**Korzystasz z asystenta AI?** Wklej ten kod URL do czatu, aby uzyskać pełną, aktualną wersję Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Każda strona niniejszej instrukcji jest dostępna w formacie surowego markdownu pod adresem o nazwie składającej się z małych liter + `.md`, a cała instrukcja jest zindeksowana pod adresem `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) steruje wszystkimi funkcjami aplikacji desktopowej z Python: przetwarzaniem obrazów w trybie wsadowym, sterowaniem kamerą LATTICE i matrycą na żywo, sesjami DAQ z czujnikami światła oraz automatyzacją zapisanych projektów. Jest to cienka warstwa nakładająca się na ten sam lokalny backend, z którego korzystają interfejs graficzny oraz CLI (HTTP na `127.0.0.1:5000`), więc zachowanie jest identyczne we wszystkich trzech środowiskach.

## Instalacja

Instalacja składa się z dwóch etapów: najpierw należy zainstalować pakiet Chloros na komputerze stacjonarnym (zapewnia on zaplecze przetwarzania oraz środowiska uruchomieniowe sprzętu), a następnie pakiet Python.

**Krok 1 — Zainstaluj Chloros.** Windows: uruchom instalator dla komputerów stacjonarnych (domyślna ścieżka `C:\Program Files\MAPIR\Chloros\`) ze strony [Pobierz](download.md). Linux: zainstaluj pakiet `.deb` ([Instalacja Linux](linux/linux-installation.md)).**Krok 2 — Zainstaluj SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Być może nie będziesz nawet potrzebować pip: każdy instalator zawiera pasujące koło SDK. Instalator Windows automatycznie instaluje je w systemowym Python; instalator Linux `.deb` umieszcza go w `/usr/lib/chloros/sdk/` i wyświetla dokładne polecenie `pip install --user`. PyPI jest aktualizowane przy wydaniu nowych wersji, więc `pip install chloros-sdk` odpowiada najnowszej stabilnej wersji.

**Krok 3 — Zaloguj się raz na każdym komputerze:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Dane uwierzytelniające są buforowane w `~/.chloros/` (na obu platformach). W Windows można się zalogować w ten sam sposób, korzystając z zakładki „<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">” użytkownika w aplikacji komputerowej. Urządzenie SDK wymaga płatnego planu Chloros+ — zobacz [Wymagania licencyjne](#license-requirement) poniżej.

| Wymaganie | Szczegóły |
| --- | --- |
| **Zainstalowany Chloros** | Windows: instalator na komputery stacjonarne; Linux: pakiet `.deb` (zawiera plik binarny zaplecza) |
| **Python** | 3.7 lub nowsza (opracowany/przetestowany na wersji 3.10) |
| **System operacyjny** | Windows 10/11 64-bitowy, Ubuntu 22.04 LTS lub nowszy, lub NVIDIA Jetson (JetPack 6) |
| **Licencja** | Aktywne konto Chloros+, dowolny płatny plan (Copper lub wyższy) |

## 60 sekund i gotowe

Jedno wywołanie tworzy projekt, importuje folder, konfiguruje przetwarzanie i uruchamia potok — automatycznie uruchamiając backend, jeśli jeszcze nie działa:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(W Linux należy używać ścieżek Linux: `/home/user/drone_images/flight001`. Polecenie SDK działa identycznie na obu platformach.)

Przetwarzasz folder przechwyconych obrazów LATTICE? Użyj nakładki dostosowanej do LATTICE — stosuje ona odpowiednie ustawienia domyślne (brak wykrywania panelu docelowego, standardowy algorytm debayeringu):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — pełna kontrola nad potokiem przetwarzania

W przypadku zadań wykraczających poza polecenie jednowierszowe należy użyć `ChlorosLocal`. Uruchamia on backend przy pierwszym użyciu (`auto_start_backend=True`), tworzy i konfiguruje projekty, monitoruje postęp oraz zwraca podsumowanie po zakończeniu zadania.

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
Zachowaj domyślny `http://127.0.0.1:5000` zamiast zastępować go `localhost` — w przypadku Windows, `localhost` najpierw przekierowuje do `::1` i kosztuje około 2 sekund na żądanie w przypadku backendu obsługującego wyłącznie IPv4.
{% endhint %}

Używaj go jako menedżera kontekstu, aby zagwarantować wyczyszczenie:

```python
import chloros_sdk

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

`configure()` akceptuje następujące słowa kluczowe: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` oraz `custom_settings`. Główne wartości:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Pokrętła specyficzne dla LATTICE (`input_level`, `radiometric_output`, rodzina `array_alignment*`) zostały udokumentowane wraz z pełnymi tabelami wartości w [Podręczniku SDK](reference/sdk-reference.md#supported-values).

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

### Odczytywanie podsumowania po zakończeniu przebiegu — i wykrywanie pustych przebiegów

Po zakończeniu `process()` dołącza podsumowanie przetwarzania backendu jako `result["summary"]`. Każdy wpis w `summary["hints"]` to pełne zdanie wyjaśniające wszelkie istotne kwestie — na przykład, dlaczego przebieg nie wygenerował żadnego wyniku — a każda wskazówka jest również ponownie wysyłana jako Python `UserWarning`, dzięki czemu przebiegi bez wyników są samodiagnozujące, nawet jeśli nigdy nie sprawdzisz słownika:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` nie jest generowany, gdy przebieg nie tworzy żadnych obrazów.** Jest to jedyne miejsce, w którym SDK i CLI celowo się różnią: `chloros-cli process` traktuje sytuację „zaprośono o produkty, ale żadne nie zostały zapisane” jako błąd i kończy działanie z wynikiem niezerowym, podczas gdy SDK kończy działanie normalnie i zgłasza ten stan za pomocą `summary` / wskazówek. Jeśli Twój potok ma się zatrzymać w przypadku pustego przebiegu, sprawdź to samodzielnie — przejrzyj `summary` (lub policz pliki w folderze projektu), zamiast polegać na wyjątku.
{% endhint %}

## Smart Connect — sprzęt działający na żywo

Trzy moduły pomocnicze otwierają trwałe sesje w puli sprzętu zaplecza — tej samej puli, z której korzysta interfejs graficzny, dzięki czemu skrypty SDK współistnieją z aplikacją desktopową bez rywalizacji o porty szeregowe lub przepustowość sieci. Wszystkie trzy automatycznie uruchamiają lokalne zaplecze, jeśli żadne nie jest uruchomione.

### Pojedyncza kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Zsynchronizowany zestaw kamer — `connect_array`

`connect_array` to zalecany punkt wyjścia dla zestawów wielokamerowych. Uruchamia ten sam proces inteligentnego przygotowania, co interfejs graficzny: analizę sieci, automatyczny wybór warstwy synchronizacji, synchronizację czasu PTP, wybór formatu pikseli dla każdej kamery, inicjowanie AE oraz uzbrojenie wyzwalacza GPIO. **Pierwsza kamera szeregowa pełni rolę urządzenia nadrzędnego** (wysyła impuls wyzwalający sprzęt); pozostałe są urządzeniami podrzędnymi.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Dodaj `smart=True` do dowolnego przechwytywania macierzowego, aby przed wyzwoleniem poczekać na ustabilizowanie się automatycznej ekspozycji we wszystkich kamerach. Informacje na temat trybów przechwytywania (Pojedynczy / Ciągły / Interwałowy / Najszybszy), rejestratorów, trybu „burst-to-video” oraz wyrównywania macierzy można znaleźć w [Podręczniku SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Czujnik światła DAQ — `connect_daq_sensor`

Bez argumentów funkcja `connect_daq_sensor()` automatycznie wykrywa protokół transmisji (kolejność: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Każda ramka zawiera 135-punktowy `spectrum` (W/m²/nm po kalibracji), flagę `is_saturated` oraz CIE `x`, `y`, `z`. Aby przypisać konkretny czujnik lub protokół — co jest niezawodnym rozwiązaniem w przypadku hostów z wieloma interfejsami sieciowymi, gdzie automatyczne wykrywanie sieci Ethernet może pominąć sprawny moduł DAQ-E przy pierwszej próbie — należy przekazać jedną wyraźną wskazówkę:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Należy pamiętać, że profile korekcji poziomu sygnału (`cap_id`) **nie** są pokrętłem SDK — należy je wybierać za pomocą `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Zapisane projekty — `open_project`

Zapisany projekt Chloros zachowuje informacje o podłączonym sprzęcie (`cameras.json` + `sensors.json` wraz z `project.json`), a `chloros_sdk.open_project(path)` może ponownie podłączyć wszystko jednocześnie i sterować przechwytywaniem danych według nazw urządzeń. Zobacz sekcję [Automatyzacja projektów](reference/sdk-reference.md#project-automation--chlorosproject) w dokumentacji.

## Co zapewnia instalacja wyłącznie za pomocą pip

Przed użyciem powierzchni sprzętowych należy sprawdzić flagi dostępności na poziomie modułów:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hoście z **wyłącznie** `pip install chloros-sdk` i bez pakietu pulpitu Chloros:

* Moduły `ChlorosLocal`, `process_folder` i `process_lattice_capture` **nie** działają — wymagają pliku binarnego zaplecza, który jest dostarczany w ramach instalatora środowiska graficznego.
* Programy pomocnicze smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) są czystymi klientami HTTP, więc działają z serwerem na innym komputerze — jednak dostarczone serwery łączą się wyłącznie z pętlą zwrotną, więc musisz samodzielnie przekierować port (np. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) oraz przekazać `backend_url="http://127.0.0.1:5000"` wraz z `auto_start_backend=False`. Zobacz [Tryb zdalnego serwera zaplecza](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Klasy LATTICE obsługujące sprzęt bezpośrednio (`LatticeCamera`, `CameraPool`, …) można zaimportować, ale wymagają środowiska uruchomieniowego Arena SDK z pakietu na komputery stacjonarne — bez niego `CAMERA_AVAILABLE` jest `False`.
* `daq_sdk` (klasy bezpośredniego DAQ) jest dostarczana wraz z instalacją na komputerze stacjonarnym, a nie w pakiecie PyPI, więc `DAQ_AVAILABLE` to `False` na hoście korzystającym wyłącznie z pip — zamiast tego steruj czujnikami DAQ za pomocą `connect_daq_sensor()` w połączeniu z (tunelowanym) backendem.

## Wymagania licencyjne

Dostęp do SDK wymaga aktywnego loginu Chloros+ na dowolnym płatnym poziomie — **Copper lub wyższym**(Copper / Bronze / Silver / Gold); bezpłatny plan Iron nie zapewnia dostępu do SDK/CLI. Wymóg ten jest egzekwowany**po stronie serwera**: każde żądanie SDK musi zawierać zarówno aktywną sesję, jak i płatny plan, w przeciwnym razie backend zwraca kod błędu `403` / `PLAN_UPGRADE_REQUIRED` (generowany jako `ChlorosLicenseError` przez `ChlorosLocal` oraz jako `ChlorosConnectError` przez pomocnicze funkcje `connect_*`). Wypisany z systemu użytkownik otrzymuje błąd `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — ponowne uruchomienie `chloros-cli login` rozwiązuje pierwszy przypadek, ale nie drugi.

Korzystanie w trybie offline działa w ramach okresu karencji planu: poziom dostępu jest odczytywany z pamięci podręcznej walidacji serwera (5 minut) lub z pamięci podręcznej podpisanej, przypisanej do urządzenia licencji (30 dni w przypadku planów miesięcznych; do wygaśnięcia subskrypcji w przypadku planów rocznych). Po upływie okresu karencji plan przechodzi w tryb bezpłatny, a dostęp poprzez SDK zostaje wstrzymany do momentu, gdy urządzenie nawiąże połączenie z serwerem. Kod błędu `chloros-cli status` pozostaje dostępny w ramach bezpłatnego poziomu, więc przyczyna jest zawsze widoczna. Zobacz [Chloros+ Logowanie](chloros+-login.md).

## Wyjątki

Przechwytuj klasę bazową, aby obsłużyć „wszelkie nieprawidłowości związane z Chloros”:

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

Wszystkie wyjątki potoku (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) wywodzą się z `ChlorosError`. Jedna pułapka: `ChlorosConnectError` — generowany wyłącznie przez `connect_camera` / `connect_array` / `connect_daq_sensor` — wywodzi się z zwykłego `Exception`, **a nie** z `ChlorosError`, więc `except ChlorosError` nie wychwyci tego błędu. Pełna hierarchia znajduje się w [Opisie SDK](reference/sdk-reference.md#exceptions).

## Zobacz też

* [Opis SDK](reference/sdk-reference.md) — kompletna powierzchnia API, zoptymalizowana pod kątem asystentów AI.
* [Dokumentacja CLI](reference/cli-reference.md) — każde podpolecenie CLI odpowiada wywołaniu SDK.
* [Pobierz](download.md) — instalatory dla Windows i Linux.
