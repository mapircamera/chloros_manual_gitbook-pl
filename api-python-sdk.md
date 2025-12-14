# API : Python SDK

**Chloros Python SDK** zapewnia programowy dostęp do silnika przetwarzania obrazów Chloros, umożliwiając automatyzację, dostosowywanie przepływów pracy oraz płynną integrację z aplikacjami Python i procesami badawczymi.

### Najważniejsze cechy

* 🐍 **Natywny Python** - Czysty, pythonic API do przetwarzania obrazów
* 🔧 **Pełny dostęp do API** - Pełna kontrola nad przetwarzaniem Chloros
* 🚀 **Automatyzacja** - Tworzenie niestandardowych przepływów pracy przetwarzania wsadowego
* 🔗 **Integracja** — osadź Chloros w istniejących aplikacjach Python
* 📊 **Gotowość do badań** — idealny do naukowych procesów analizy
* ⚡ **Przetwarzanie równoległe** — skalowalność do rdzeni procesora (Chloros+)

### Wymagania

| Wymaganie          | Szczegóły                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Musi być zainstalowany lokalnie                                           |
| **Licencja**          | Chloros+ ([wymagany płatny plan](https://cloud.mapir.camera/pricing)) |
| **System operacyjny** | Windows 10/11 (64-bitowy)                                              |
| **Python**           | Python 3.7 lub nowszy                                                |
| **Pamięć**           | Minimum 8 GB pamięci RAM (zalecane 16 GB)                                  |
| **Internet**         | Wymagany do aktywacji licencji                                     |

{% hint style=&quot;warning&quot; %}
**Wymagania licencyjne**: Python SDK wymaga płatnej subskrypcji Chloros+ w celu uzyskania dostępu do API. Standardowe (bezpłatne) plany nie zapewniają dostępu do API/SDK. Odwiedź stronę [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), aby dokonać aktualizacji.
{% endhint %}

## Szybki start

### Instalacja

Zainstaluj za pomocą pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Pierwsza konfiguracja**: Przed użyciem SDK należy aktywować licencję Chloros+, otwierając Chloros, Chloros (przeglądarka) lub Chloros CLI i zaloguj się przy użyciu swoich danych uwierzytelniających. Czynność tę wystarczy wykonać tylko raz.
{% endhint %}

### Podstawowe zastosowanie

Przetwarzanie folderu zawierającego tylko kilka wierszy:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Pełna kontrola

W przypadku zaawansowanych procesów:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

## Instrukcja instalacji

### Wymagania wstępne

Przed zainstalowaniem SDK upewnij się, że masz:

1. Zainstalowany **Chloros Desktop** ([pobierz](download.md))
2. **Python 3.7+** ([python.org](https://www.python.org))
3. **Aktywną licencję Chloros+** ([aktualizacja](https://cloud.mapir.camera/pricing))

### Instalacja za pomocą pip

**Instalacja standardowa:**

```bash
pip install chloros-sdk
```

**Z obsługą monitorowania postępu:**

```bash
pip install chloros-sdk[progress]
```

**Instalacja rozwojowa:**

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

SDK korzysta z tej samej licencji co Chloros, Chloros (przeglądarka) i Chloros CLI. Aktywuj raz za pomocą GUI lub CLI:

1. Otwórz **Chloros lub Chloros (przeglądarka)** i zaloguj się na karcie użytkownika <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Lub otwórz **CLI**.
2. Wprowadź swoje dane logowania Chloros+ i zaloguj się
3. Licencja jest buforowana lokalnie (utrzymuje się po ponownym uruchomieniu komputera)

{% hint style=&quot;success&quot; %}
**Jednorazowa konfiguracja**: Po zalogowaniu się za pośrednictwem GUI lub CLI, SDK automatycznie korzysta z zapisanej w pamięci podręcznej licencji. Nie jest wymagane dodatkowe uwierzytelnianie!
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

## API Odwołanie

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

| Parametr                 | Typ | Domyślny                   | Opis                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokalnego backendu Chloros          |
| `auto_start_backend`      | bool | `True`                    | Automatyczne uruchomienie zaplecza w razie potrzeby |
| `backend_exe`             | str  | `None` (automatyczne wykrywanie)      | Ścieżka do pliku wykonywalnego zaplecza            |
| `timeout`                 | int  | `30`                      | Limit czasu żądania w sekundach            |
| `backend_startup_timeout` | int  | `60`                      | Limit czasu uruchamiania backendu (sekundy) |

**Przykłady:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metody

#### `create_project(project_name, camera=None)`

Utwórz nowy projekt Chloros.

**Parametry:**

| Parametr      | Typ | Wymagane | Opis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Tak      | Nazwa projektu                                     |
| `camera`       | str  | Nie       | Szablon kamery (np. „Survey3N\_RGN”, „Survey3W\_OCN”) |

**Zwraca:** `dict` — odpowiedź dotycząca utworzenia projektu

**Przykład:**

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
| `recursive`   | bool     | Nie       | Przeszukaj podfoldery (domyślnie: Fałsz) |

**Zwraca:** `dict` — Wyniki importu z liczbą plików

**Przykład:**

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
| `debayer`                 | str  | „Wysoka jakość (szybsza)” | Metoda debayera                  |
| `vignette_correction`     | bool | `True`                  | Włącz korekcję winietowania      |
| `reflectance_calibration` | bool | `True`                  | Włącz kalibrację odbicia  |
| `indices`                 | lista | `None`                  | Indeksy roślinności do obliczenia |
| `export_format`           | str  | „TIFF (16-bit)”         | Format wyjściowy                   |
| `ppk`                     | bool | `False`                 | Włącz korekty PPK          |
| `custom_settings`         | dict | `None`                  | Zaawansowane ustawienia niestandardowe        |

**Formaty eksportu:**

* `"TIFF (16-bit)"` — zalecany dla GIS/fotogrametrii
* `"TIFF (32-bit, Percent)"` — analiza naukowa
* `"PNG (8-bit)"` — kontrola wzrokowa
* `"JPG (8-bit)"` — skompresowane dane wyjściowe

**Dostępne indeksy:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 i inne.

**Przykład:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Przetwarzanie obrazów projektu.

**Parametry:**

| Parametr           | Typ     | Domyślny      | Opis                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Tryb przetwarzania: „równoległy” lub „szeregowy”   |
| `wait`              | bool     | `True`       | Oczekiwanie na zakończenie                       |
| `progress_callback` | callable | `None`       | Funkcja wywołania zwrotnego postępu (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interwał odpytywania postępu (sekundy)   |

**Zwraca:** `dict` — wyniki przetwarzania

{% hint style=&quot;warning&quot; %}
**Tryb równoległy**: Wymaga licencji Chloros+. Automatycznie skaluje się do liczby rdzeni procesora (do 16 procesów).
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

Pobierz aktualną konfigurację projektu.

**Zwraca:** `dict` — aktualną konfigurację projektu.

**Przykład:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Pobierz informacje o stanie zaplecza.

**Zwraca:** `dict` — stan zaplecza

**Przykład:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Wyłącza backend (jeśli został uruchomiony przez SDK).

**Przykład:**

```python
chloros.shutdown_backend()
```

***

### Funkcje pomocnicze

#### `process_folder(folder_path, **options)`

Jednowierszowa funkcja pomocnicza do przetwarzania folderu.

**Parametry:**

| Parametr                 | Typ     | Domyślny         | Opis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Wymagane        | Ścieżka do folderu z obrazami     |
| `project_name`            | str      | Generowane automatycznie  | Nazwa projektu                   |
| `camera`                  | str      | `None`          | Szablon kamery                |
| `indices`                 | lista     | `["NDVI"]`      | Indeksy do obliczenia           |
| `vignette_correction`     | bool     | `True`          | Włącz korekcję winietowania     |
| `reflectance_calibration` | bool     | `True`          | Włącz kalibrację odbicia |
| `export_format`           | str      | „TIFF (16-bit)” | Format wyjściowy                  |
| `mode`                    | str      | `"parallel"`    | Tryb przetwarzania                |
| `progress_callback`       | wywoływalny | `None`          | Wywołanie zwrotne postępu              |

**Zwraca:** `dict` — Wyniki przetwarzania

**Przykład:**

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

## Obsługa menedżera kontekstu

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

## Kompletne przykłady

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
    debayer="High Quality (Faster)",
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

### Przykład 4: Integracja potoku badawczego

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

Solidne obsługiwanie błędów do użytku produkcyjnego:

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### Przykład 7: Narzędzie wiersza poleceń

Zbuduj niestandardowe narzędzie CLI za pomocą SDK:

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
    
    args = parser.parse_args()
    
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
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Obsługa wyjątków

SDK udostępnia określone klasy wyjątków dla różnych typów błędów:

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
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Tematy zaawansowane

### Konfiguracja niestandardowego zaplecza

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

### Backend nie uruchamia się

**Problem:** SDK nie uruchamia backendu

**Rozwiązania:**

1. Sprawdź, czy zainstalowano Chloros Desktop:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Sprawdź, czy zapora Windows nie blokuje połączenia
3. Spróbuj ręcznie podać ścieżkę do zaplecza:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Nie wykryto licencji

**Problem:** SDK wyświetla ostrzeżenie o braku licencji

**Rozwiązania:**

1. Otwórz Chloros, Chloros (przeglądarka) lub Chloros CLI i zaloguj się.
2. Sprawdź, czy licencja jest zapisana w pamięci podręcznej:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Skontaktuj się z pomocą techniczną: info@mapir.camera

***

### Błędy importu

**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Rozwiązania:**

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

### Przekroczenie limitu czasu przetwarzania

**Problem:** Przekroczenie limitu czasu przetwarzania

**Rozwiązania:**

1. Zwiększ limit czasu:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Przetwarzaj mniejsze partie danych
3. Sprawdź dostępną przestrzeń dyskową
4. Monitoruj zasoby systemowe

***

### Port już używany

**Problem:** Port zaplecza 5000 zajęty

**Rozwiązania:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Lub znajdź i zamknij proces powodujący konflikt:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Wskazówki dotyczące wydajności

### Optymalizacja szybkości przetwarzania

1. **Użyj trybu równoległego** (wymaga Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Zmniejsz rozdzielczość wyjściową** (jeśli jest to dopuszczalne)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Wyłącz niepotrzebne indeksy**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Przetwarzaj na dysku SSD** (nie HDD)

***

### Optymalizacja pamięci

W przypadku dużych zbiorów danych:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Przetwarzanie w tle

Zwolnij Python dla innych zadań:

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

**O:** Tylko do początkowej aktywacji licencji. Po zalogowaniu się za pomocą Chloros, Chloros (przeglądarka) lub Chloros CLI licencja jest buforowana lokalnie i działa w trybie offline przez 30 dni.

***

### P: Czy mogę używać SDK na serwerze bez GUI?

**O:** Tak! Wymagania:

* Windows Server 2016 lub nowszy
* Zainstalowany Chloros (jednorazowo)
* Licencja aktywowana na dowolnym komputerze (licencja z pamięci podręcznej skopiowana na serwer)

***

### P: Jaka jest różnica między Desktop, CLI i SDK?

| Funkcja         | Desktop GUI | CLI Command Line | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interfejs**   | Punkt-klik | Polecenie          | Python API  |
| **Najlepsze do**    | Pracy wizualnej | Skryptów        | Integracji |
| **Automatyzacja**  | Ograniczona     | Dobra             | Doskonała   |
| **Elastyczność** | Podstawowa       | Dobra             | Maksymalna     |
| **Licencja**     | Chloros+    | Chloros+         | Chloros+    |

***

### P: Czy mogę dystrybuować aplikacje stworzone za pomocą SDK?

**O:** Kod SDK można zintegrować z aplikacjami, ale:

* Użytkownicy końcowi muszą mieć zainstalowany Chloros
* Użytkownicy końcowi muszą posiadać aktywne licencje Chloros+
* Dystrybucja komercyjna wymaga licencji OEM

W sprawie pytań dotyczących licencji OEM prosimy o kontakt z info@mapir.camera.

***

### P: Jak zaktualizować SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### P: Gdzie są zapisywane przetworzone obrazy?

Domyślnie w ścieżce projektu:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### P: Czy mogę przetwarzać obrazy ze skryptów Python uruchamianych zgodnie z harmonogramem?

**O:** Tak! Użyj harmonogramu zadań Windows ze skryptami Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Zaplanuj codzienne uruchamianie za pomocą harmonogramu zadań.

***

### P: Czy SDK obsługuje async/await?

**O:** Obecna wersja jest synchroniczna. Aby uzyskać zachowanie asynchroniczne, użyj `wait=False` lub uruchom w oddzielnym wątku:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Pomoc

### Dokumentacja

* **API Reference**: Ta strona

### Kanały pomocy technicznej

* **E-mail**: info@mapir.camera
* **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Przykładowy kod

Wszystkie wymienione tutaj przykłady zostały przetestowane i są gotowe do użycia. Skopiuj je i dostosuj do swoich potrzeb.

***

## Licencja

**Oprogramowanie zastrzeżone** — Copyright (c) 2025 MAPIR Inc.

SDK wymaga aktywnej subskrypcji Chloros+. Nieuprawnione użycie, dystrybucja lub modyfikacja są zabronione.
