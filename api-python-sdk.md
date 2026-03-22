# API : Python SDK

**Chloros Python SDK** zapewnia programowy dostęp do silnika przetwarzania obrazów Chloros, umożliwiając automatyzację, tworzenie niestandardowych przepływów pracy oraz płynną integrację z aplikacjami Python i procesami badawczymi.

### Kluczowe funkcje

* 🐍 **Natywny Python** — przejrzysty, napisany w stylu Pythona kod API do przetwarzania obrazów
* 🔧 **Pełny dostęp do API** — całkowita kontrola nad przetwarzaniem Chloros
* 🚀 **Automatyzacja** — Twórz niestandardowe przepływy pracy przetwarzania wsadowego
* 🔗 **Integracja** — Osadzaj Chloros w istniejących aplikacjach Python
* 📊 **Gotowość do badań** — Idealne rozwiązanie dla naukowych procesów analitycznych
* ⚡ **Przetwarzanie równoległe** — Skalowalność w zależności od liczby rdzeni procesora (Chloros+)

### Wymagania

| Wymaganie          | Szczegóły                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Zainstalowany Chloros** | Windows: Instalator na komputery stacjonarne; Linux: pakiet `.deb`                  |
| **Licencja**          | Chloros+ ([wymagany płatny plan](https://cloud.mapir.camera/pricing)) |
| **System operacyjny** | Windows 10/11 (64-bitowy), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 lub nowszy                                                |
| **Pamięć**           | Minimum 8 GB pamięci RAM (zalecane 16 GB)                                  |
| **Internet**         | Wymagany do aktywacji licencji                                     |

{% hint style="warning" %}
**Wymagania licencyjne**: Python SDK wymaga płatnej subskrypcji Chloros+ w celu uzyskania dostępu do API. Standardowe (bezpłatne) plany nie zapewniają dostępu do API/SDK. Odwiedź stronę [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), aby dokonać aktualizacji.
{% endhint %}

## Szybki start

### Instalacja

Zainstaluj za pomocą pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Pierwsza konfiguracja**: Przed rozpoczęciem korzystania z SDK należy aktywować licencję Chloros+ poprzez otwarcie Chloros, Chloros (przeglądarka) lub Chloros CLI i zaloguj się przy użyciu swoich danych uwierzytelniających. Wystarczy to zrobić tylko raz. W Linux (bez GUI) użyj: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Podstawowe użycie

Przetwarzaj folder za pomocą zaledwie kilku wierszy:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Ścieżki międzyplatformowe**: Przykłady kodu na tej stronie wykorzystują ścieżki w stylu Windows (np. `C:\\DroneImages\\Flight001`). W systemie Linux należy zamiast tego używać ścieżek Linux (np. `/home/user/drone_images/flight001` lub `~/drone_images/flight001`). SDK działa identycznie na obu platformach.
{% endhint %}

### Pełna kontrola

Dla zaawansowanych procesów:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Przewodnik instalacji

### Wymagania wstępne

Przed instalacją SDK upewnij się, że masz:

1. **Zainstalowany Chloros** — Windows: Instalator na komputer stacjonarny ([pobierz](download.md)); Linux: pakiet `.deb` ([Instalacja](linux/linux-installation.md))
2. **Python 3.7+** zainstalowany ([python.org](https://www.python.org))
3. **Aktywna licencja Chloros+** ([aktualizacja](https://cloud.mapir.camera/pricing))

### Instalacja za pomocą pip

**Instalacja standardowa:**

```bash
pip install chloros-sdk
```

**Z obsługą monitorowania postępu:**

```bash
pip install chloros-sdk[progress]
```

**Instalacja deweloperska:**

```bash
pip install chloros-sdk[dev]
```

### Sprawdź instalację

Sprawdź, czy SDK jest poprawnie zainstalowany:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Pierwsza konfiguracja

### Aktywacja licencji

SDK korzysta z tej samej licencji co Chloros, Chloros (przeglądarka) oraz Chloros CLI. Aktywuj raz za pomocą GUI lub CLI:**Windows:**Otwórz**Chloros lub Chloros (przeglądarka)** i zaloguj się w zakładce Użytkownik <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> lub skorzystaj z CLI.**Linux:** Użyj CLI (brak dostępnego GUI):

```bash
chloros-cli login user@example.com 'your_password'
```

Licencja jest buforowana lokalnie i pozostaje aktywna po ponownym uruchomieniu systemu.

{% hint style="success" %}
**Konfiguracja jednorazowa**: Po zalogowaniu się za pomocą GUI lub CLI, SDK automatycznie korzysta z licencji zapisanej w pamięci podręcznej. Nie jest wymagane dodatkowe uwierzytelnianie!
{% endhint %}

{% hint style="info" %}
**Wylogowanie**: Użytkownicy SDK mogą programowo wyczyścić dane uwierzytelniające zapisane w pamięci podręcznej, korzystając z metody XPROTX000066X. Zobacz [metodę logout()](#logout) w dokumentacji API.
{% endhint %}

### Test połączenia

Sprawdź, czy SDK może połączyć się z Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Dokumentacja API

### Klasa ChlorosLocal

Główna klasa do lokalnego przetwarzania obrazów Chloros.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametry:**

| Parametr                 | Typ | Wartość domyślna                   | Opis                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokalnego backendu Chloros          |
| `auto_start_backend`      | bool | `True`                    | Automatycznie uruchom backend w razie potrzeby |
| `backend_exe`             | str  | `None` (autodetekcja)      | Ścieżka do pliku wykonywalnego backendu            |
| `timeout`                 | int  | `30`                      | Limit czasu żądania w sekundach            |
| `backend_startup_timeout` | int  | `60`                      | Limit czasu uruchomienia backendu (sekundy) |

**Przykłady:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Automatyczne wykrywanie platformy**: SDK automatycznie próbuje znaleźć właściwą ścieżkę do backendu dla danej platformy:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ręcznie)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metody

#### `create_project(project_name, camera=None)`

Utwórz nowy projekt Chloros.

**Parametry:**

| Parametr      | Typ | Wymagany | Opis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Tak      | Nazwa projektu                                     |
| `camera`       | str  | Nie       | Szablon kamery (np. „Survey3N\_RGN”, „Survey3W\_OCN”) |

**Zwraca:** `dict` — Odpowiedź dotycząca utworzenia projektu**Przykład:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importuj obrazy z folderu.

**Parametry:**

| Parametr     | Typ     | Wymagane | Opis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Tak      | Ścieżka do folderu z obrazami         |
| `recursive`   | bool     | Nie       | Przeszukaj podfoldery (domyślnie: False) |

**Zwraca:** `dict` - Wyniki importu wraz z liczbą plików**Przykład:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfiguracja ustawień przetwarzania.

**Parametry:**

| Parametr                 | Typ | Domyślnie                 | Opis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | „Standard (szybki, średnia jakość)” | Metoda debayera            |
| `vignette_correction`     | bool | `True`                  | Włącz korekcję winietowania      |
| `reflectance_calibration` | bool | `True`                  | Włącz kalibrację odbicia  |
| `indices`                 | lista | `None`                  | Wskaźniki wegetacji do obliczenia |
| `export_format`           | str  | „TIFF (16-bit)”         | Format wyjściowy                   |
| `ppk`                     | bool | `False`                 | Włącz korekty PPK          |
| `custom_settings`         | dict | `None`                  | Zaawansowane ustawienia niestandardowe        |

**Formaty eksportu:**

* `"TIFF (16-bit)"` - Zalecany do GIS/fotogrametrii
* `"TIFF (32-bit, Percent)"` - Analiza naukowa
* `"PNG (8-bit)"` - Kontrola wizualna
* `"JPG (8-bit)"` - Skompresowany wynik

**Dostępne indeksy:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 i inne.**Przykład:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Przetwarzaj obrazy projektu.

**Parametry:**

| Parametr           | Typ     | Domyślne      | Opis                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Tryb przetwarzania: „parallel” lub „serial”   |
| `wait`              | bool     | `True`       | Czekaj na zakończenie                       |
| `progress_callback` | wywoływalny | `None`       | Funkcja wywołania zwrotnego postępu (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interwał odpytywania o postęp (sekundy)   |

**Zwraca:** `dict` - Wyniki przetwarzania

{% hint style="warning" %}
**Tryb równoległy**: Wymaga licencji Chloros+. Automatycznie skaluje się do rdzeni procesora (do 16 procesów roboczych).
{% endhint %}

**Przykład:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Pobierz bieżącą konfigurację projektu.

**Zwraca:** `dict` - Bieżąca konfiguracja projektu**Przykład:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Pobierz informacje o stanie zaplecza, w tym postęp przetwarzania dla poszczególnych wątków.

**Zwraca:** `dict` — stan zaplecza o następującej strukturze:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Przykład:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Wyłącz backend (jeśli został uruchomiony przez SDK).

**Przykład:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Wyczyść buforowane poświadczenia z systemu lokalnego.

**Opis:**

Programowo wylogowuje użytkownika poprzez usunięcie buforowanych danych uwierzytelniających. Jest to przydatne w następujących sytuacjach:
* Przełączanie się między różnymi kontami Chloros+
* Czyszczenie danych uwierzytelniających w środowiskach zautomatyzowanych
* Ze względów bezpieczeństwa (np. usuwanie danych uwierzytelniających przed odinstalowaniem)

**Zwraca:** `dict` - Wynik operacji wylogowania**Przykład:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Wymagane ponowne uwierzytelnienie**: Po wywołaniu `logout()` należy ponownie zalogować się za pomocą Chloros, Chloros (przeglądarka)lub Chloros CLI przed użyciem SDK.
{% endhint %}

***

### Funkcje pomocnicze

#### `process_folder(folder_path, **options)`

Jednowierszowa funkcja pomocnicza do przetwarzania folderu.

**Parametry:**

| Parametr                 | Typ     | Domyślna wartość         | Opis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Wymagane        | Ścieżka do folderu z obrazami     |
| `project_name`            | str      | Generowane automatycznie  | Nazwa projektu                   |
| `camera`                  | str      | `None`          | Szablon aparatu                |
| `indices`                 | lista     | `["NDVI"]`      | Indeksy do obliczenia           |
| `vignette_correction`     | bool     | `True`          | Włącz korekcję winietowania     |
| `reflectance_calibration` | bool     | `True`          | Włącz kalibrację odbicia |
| `export_format`           | str      | „TIFF (16-bit)” | Format wyjściowy                  |
| `mode`                    | str      | `"parallel"`    | Tryb przetwarzania                |
| `progress_callback`       | wywoływalny | `None`          | Wywołanie zwrotne postępu              |

**Zwraca:** `dict` - Wyniki przetwarzania**Przykład:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Obsługa menedżerów kontekstu

SDK obsługuje menedżery kontekstu do automatycznego czyszczenia:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Pełne przykłady

{% hint style="info" %}
**Użytkownicy Linux**: Wszystkie poniższe przykłady wykorzystują ścieżki Windows. Zastąp ścieżki `C:\\...` swoimi ścieżkami Linux (np. `/home/user/...` lub `~/...`). Wszystkie funkcje SDK są identyczne na wszystkich platformach.
{% endhint %}

### Przykład 1: Podstawowe przetwarzanie

Przetwarzanie folderu przy użyciu ustawień domyślnych:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Przykład 2: Niestandardowy przepływ pracy

Pełna kontrola nad potokiem przetwarzania:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Przykład 3: Przetwarzanie wsadowe wielu folderów

Przetwarzanie wielu zestawów danych lotniczych:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Przykład 4: Integracja z procesem badawczym

Integracja Chloros z analizą danych:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Przykład 5: Niestandardowe monitorowanie postępów

Zaawansowane śledzenie postępów z rejestrowaniem:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Przykład 6: Obsługa błędów

Solidna obsługa błędów do użytku produkcyjnego:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Przykład 7: Zarządzanie kontem i wylogowanie

Zarządzaj danymi uwierzytelniającymi programowo:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Przykład 8: Narzędzie wiersza poleceń

Stwórz niestandardowe narzędzie CLI za pomocą SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Sposób użycia:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Obsługa wyjątków

SDK udostępnia konkretne klasy wyjątków dla różnych typów błędów:

### Hierarchia wyjątków

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Przykłady wyjątków

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Zagadnienia zaawansowane

### Niestandardowa konfiguracja zaplecza

Użyj niestandardowej lokalizacji lub konfiguracji zaplecza:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Przetwarzanie nieblokujące

Rozpocznij przetwarzanie i kontynuuj inne zadania:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Zarządzanie pamięcią

W przypadku dużych zbiorów danych przetwarzaj je partiami:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Rozwiązywanie problemów

### Backend się nie uruchamia

**Problem:** SDK nie uruchamia backendu**Rozwiązania:**

1. Sprawdź, czy Chloros jest zainstalowany:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Sprawdź zaporę sieciową (Windows) lub dostępność portu (Linux: `lsof -i :5000`)
3. Spróbuj ręcznie podać ścieżkę do serwera zaplecza:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Nie wykryto licencji**Problem:** SDK wyświetla ostrzeżenie o brakującej licencji**Rozwiązania:**

1. Otwórz Chloros, Chloros (przeglądarka) lub Chloros CLI i zaloguj się.
2. Sprawdź, czy licencja jest zapisana w pamięci podręcznej:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Jeśli występują problemy z danymi uwierzytelniającymi, wyczyść dane uwierzytelniające z pamięci podręcznej i zaloguj się ponownie:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Skontaktuj się z pomocą techniczną: info@mapir.camera

***

### Błędy importu**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Rozwiązania:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Przekroczenie limitu czasu przetwarzania**Problem:** Przekroczenie limitu czasu przetwarzania**Rozwiązania:**

1. Zwiększ limit czasu:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Przetwarzaj mniejsze partie
3. Sprawdź dostępną przestrzeń dyskową
4. Monitoruj zasoby systemowe

***

### Port już używany**Problem:** Port backendowy 5000 zajęty**Rozwiązania:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Lub znajdź i zamknij proces powodujący konflikt:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Wskazówki dotyczące wydajności

### Optymalizacja szybkości przetwarzania

1. **Użyj trybu równoległego** (wymaga Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Zmniejsz rozdzielczość wyjściową** (jeśli to dopuszczalne)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Wyłącz niepotrzebne indeksy**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Przetwarzaj na dysku SSD** (nie na dysku HDD)***

### Optymalizacja pamięci

W przypadku dużych zbiorów danych:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Przetwarzanie w tle

Zwolnij Python na inne zadania:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Przykłady integracji

### Integracja z Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Często zadawane pytania

### P: Czy SDK wymaga połączenia z Internetem?

**O:** Tylko do początkowej aktywacji licencji. Po zalogowaniu się przez Chloros, Chloros (przeglądarka) lub Chloros CLI licencja jest zapisywana lokalnie w pamięci podręcznej i działa w trybie offline przez 30 dni.***

### P: Czy mogę używać SDK na serwerze bez GUI?**Odp.:** Tak! Licencja SDK działa w trybie bezinterfejsowym zarówno na serwerach Windows, jak i Linux.**Linux (zalecane dla trybu bezinterfejsowego):**
* Zainstaluj za pomocą pakietu `.deb`
* Aktywuj licencję: `chloros-cli login user@example.com 'password'`

**Serwer Windows:**
* Serwer Windows 2016 lub nowszy
* Zainstalowany Chloros (jednorazowo)
* Licencja aktywowana za pomocą CLI lub na dowolnym komputerze

***

### P: Jaka jest różnica między wersją Desktop, CLI i SDK?

| Funkcja         | Interfejs graficzny Desktop | Wiersz poleceń CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interfejs**   | Wskazanie i kliknięcie | Polecenie          | Python API  |
| **Najlepsze zastosowanie**    | Praca wizualna | Tworzenie skryptów        | Integracja |
| **Automatyzacja**  | Ograniczona     | Dobra             | Doskonała   |
| **Elastyczność** | Podstawowa       | Dobra             | Maksymalna     |
| **Licencja**     | Chloros+    | Chloros+         | Chloros+    |***

### P: Czy mogę rozpowszechniać aplikacje stworzone przy użyciu SDK?**Odp.:** Kod SDK można zintegrować z aplikacjami, ale:

* Użytkownicy końcowi muszą mieć zainstalowany Chloros
* Użytkownicy końcowi muszą posiadać aktywne licencje Chloros+
* Dystrybucja komercyjna wymaga licencji OEM

W sprawie zapytań dotyczących licencji OEM prosimy o kontakt z info@mapir.camera.

***

### P: Jak zaktualizować SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### P: Gdzie zapisywane są przetworzone obrazy?

Domyślnie w ścieżce projektu:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### P: Czy mogę przetwarzać obrazy ze skryptów Python uruchamianych zgodnie z harmonogramem?**O:** Tak! Użyj harmonogramu systemu operacyjnego ze skryptami Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Zaplanuj za pomocą Harmonogramu zadań, aby uruchamiał się codziennie.**Linux:** Zaplanuj za pomocą crona:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### P: Czy SDK obsługuje async/await?**O:** Obecna wersja jest synchroniczna. Aby uzyskać zachowanie asynchroniczne, użyj `wait=False` lub uruchom w oddzielnym wątku:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### P: Jak przełączać się między różnymi kontami Chloros+?**O:** Użyj metody `logout()`, aby wyczyścić buforowane dane uwierzytelniające, a następnie zaloguj się ponownie na nowe konto:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Po wylogowaniu uwierzytelnij się na nowym koncie za pomocą interfejsu graficznego, przeglądarki lub CLI przed ponownym użyciem SDK.

***

## Pomoc

### Dokumentacja

* **API Reference**: Ta strona

### Kanały wsparcia

* **E-mail**: info@mapir.camera
* **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Przykładowy kod

Wszystkie wymienione tutaj przykłady zostały przetestowane i są gotowe do użycia w środowisku produkcyjnym. Skopiuj je i dostosuj do swoich potrzeb.

***

## Licencja**Oprogramowanie zastrzeżone** - Copyright (c) 2025 MAPIR Inc.

SDK wymaga aktywnej subskrypcji Chloros+. Nieuprawnione użycie, dystrybucja lub modyfikacja są zabronione.
