# CLI: Wiersz poleceń

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** zapewnia rozbudowany dostęp z wiersza poleceń do silnika przetwarzania obrazów Chloros, umożliwiając automatyzację, tworzenie skryptów i pracę w trybie bezinterfejsowym w ramach procesów przetwarzania obrazów.

### Kluczowe funkcje

* 🚀 **Automatyzacja** — przetwarzanie wsadowe wielu zestawów danych za pomocą skryptów
* 🔗 **Integracja** — wbudowanie w istniejące procesy i potoki
* 💻 **Praca bez interfejsu graficznego** — uruchamianie bez GUI
* 🌍 **Wielojęzyczność** — obsługa 38 języków
* ⚡ **Przetwarzanie równoległe** — [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) automatycznie optymalizuje działanie pod kątem posiadanego sprzętu

### Wymagania

| Wymaganie          | Szczegóły                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **System operacyjny** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licencja**          | Chloros+ ([wymagany płatny plan](https://cloud.mapir.camera/pricing)) |
| **Pamięć**           | Minimum 8 GB pamięci RAM (zalecane 16 GB)                                  |
| **Internet**         | Wymagany do aktywacji licencji                                     |
| **Miejsce na dysku**       | Zależy od wielkości projektu                                              |

{% hint style="warning" %}
**Wymagania licencyjne**: CLI wymaga płatnej subskrypcji Chloros+. Plany standardowe (bezpłatne) nie zapewniają dostępu do CLI. Odwiedź stronę [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), aby dokonać aktualizacji.
{% endhint %}

## Szybki start

### Instalacja

#### Windows

CLI jest automatycznie dołączony do instalatora Chloros:

1. Pobierz i uruchom **Chloros Installer.exe**

2. Ukończ pracę kreatora instalacji
3. CLI zainstalowano w: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Instalator automatycznie dodaje `chloros-cli` do ścieżki systemowej PATH. Po instalacji uruchom ponownie terminal.
{% endhint %}

#### Linux

Zainstaluj pakiet `.deb` dla swojej architektury:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Szczegółowe informacje na temat konfiguracji Linux można znaleźć w sekcji [Instalacja Linux](linux/linux-installation.md).

### Pierwsza konfiguracja

Przed rozpoczęciem korzystania z CLI należy aktywować licencję Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Podstawowe użycie

Przetwarzanie folderu przy użyciu ustawień domyślnych:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Opis poleceń

### Ogólna składnia

```
chloros-cli [global-options] <command> [command-options]
```

***

## Polecenia

### `process` — Przetwarzanie obrazów

Przetwarzanie obrazów w folderze z kalibracją.

**Składnia:**

```bash
chloros-cli process <input-folder> [options]
```

**Przykłady:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opcje polecenia przetwarzania

| Opcja                | Typ    | Domyślna        | Opis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Ścieżka    | _Wymagane_     | Folder zawierający obrazy wielospektralne w formacie RAW/JPG                                         |
| `-o, --output`        | Ścieżka    | Tak samo jak dane wejściowe  | Folder wyjściowy dla przetworzonych obrazów                                                     |
| `-n, --project-name`  | Ciąg znaków  | Generowane automatycznie | Niestandardowa nazwa projektu                                                                    |
| `--vignette`          | Flaga    | Włączone        | Włącz korekcję winietowania                                                             |
| `--no-vignette`       | Flaga    | -              | Wyłącz korekcję winietowania                                                            |
| `--reflectance`       | Flaga    | Włączone        | Włącz kalibrację odbicia                                                         |
| `--no-reflectance`    | Flaga    | -              | Wyłącz kalibrację odbicia                                                        |
| `--ppk`               | Flaga    | Wyłączone       | Zastosuj korekcje PPK na podstawie danych z czujnika światła .daq                                      |
| `--format`            | Wybór  | TIFF (16-bit)  | Format wyjściowy: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Liczba całkowita | Auto           | Minimalny rozmiar celu w pikselach dla wykrywania panelu kalibracyjnego                          |
| `--target-clustering` | Liczba całkowita | Auto           | Próg grupowania celów (0-100)                                                    |
| `--debayer`           | Wybór  | `standard`     | Metoda debayera: `standard` lub `texture-aware` (tylko Chloros+)                          |
| `--target`, `--targets` | Flaga  | Wyłączone       | Szukaj celów kalibracji tylko w podfolderze „target” lub „targets” (przyspiesza przetwarzanie) |
| `--indices`           | Lista    | Brak           | Wskaźniki wegetacji do obliczenia (np. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Ciąg znaków  | Brak           | Zablokuj ekspozycję dla modelu kamery (pin 1)                                                 |
| `--exposure-pin-2`    | Ciąg znaków  | Brak           | Blokada ekspozycji dla modelu kamery (pin 2)                                                 |
| `--recal-interval`    | Liczba całkowita | Auto           | Interwał ponownej kalibracji w sekundach                                                      |
| `--timezone-offset`   | Liczba całkowita | 0              | Różnica stref czasowych w godzinach                                                               |

***

### `login` - Uwierzytelnianie konta

Zaloguj się przy użyciu danych logowania Chloros+, aby włączyć przetwarzanie CLI.

**Składnia:**

```bash
chloros-cli login <email> <password>
```

**Przykład:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Znaki specjalne**: Użyj pojedynczych cudzysłowów wokół haseł zawierających znaki takie jak `$`, `!` lub spacje.
{% endhint %}

**Wynik:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` — Wyczyść dane uwierzytelniające

Wyczyść zapisane dane uwierzytelniające i wyloguj się ze swojego konta.

**Składnia:**

```bash
chloros-cli logout
```

**Przykład:**

```bash
chloros-cli logout
```

**Wynik:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Użytkownicy SDK**: Python SDK udostępnia również programową metodę `logout()` do czyszczenia danych uwierzytelniających w skryptach Python. Szczegółowe informacje można znaleźć w [dokumentacji Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` — Sprawdź stan licencji

Wyświetla aktualny stan licencji i uwierzytelnienia.

**Składnia:**

```bash
chloros-cli status
```

**Przykład:**

```bash
chloros-cli status
```

**Wynik:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` — Sprawdź postęp eksportu

Monitoruj postęp eksportu wątku 4 podczas przetwarzania lub po jego zakończeniu.

**Składnia:**

```bash
chloros-cli export-status
```

**Przykład:**

```bash
chloros-cli export-status
```

**Przypadek użycia:** Wywołaj to polecenie podczas trwania przetwarzania, aby sprawdzić postęp eksportu.***

### `language` — Zarządzanie językiem interfejsu

Wyświetl lub zmień język interfejsu CLI.

**Składnia:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Przykłady:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Obsługiwane języki (łącznie 38)

| Kod    | Język              | Nazwa w języku ojczystym      |
| ------- | --------------------- | ---------------- |
| `en`    | angielski               | English          |
| `es`    | hiszpański               | Español          |
| `pt`    | portugalski            | Português        |
| `fr`    | Francuski                | Français         |
| `de`    | Niemiecki                | Deutsch          |
| `it`    | Włoski               | Italiano         |
| `ja`    | Japoński              | 日本語              |
| `ko`    | Koreański                | 한국어              |
| `zh`    | Chiński (uproszczony)  | 简体中文             |
| `zh-TW` | Chiński (tradycyjny) | 繁體中文             |
| `ru`    | Rosyjski               | Русский          |
| `nl`    | Holenderski                | Nederlands       |
| `ar`    | Arabski                | العربية          |
| `pl`    | Polski                | Polski           |
| `tr`    | Turecki               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonezyjski            | Bahasa Indonesia |
| `vi`    | Wietnamski            | Tiếng Việt       |
| `th`    | Tajski                  | ไทย              |
| `sv`    | Szwedzki               | Svenska          |
| `da`    | Duński                | Dansk            |
| `no`    | Norweski             | Norsk            |
| `fi`    | Fiński               | Suomi            |
| `el`    | Grecki                 | Ελληνικά         |
| `cs`    | Czeski                 | Čeština          |
| `hu`    | Węgierski             | Magyar           |
| `ro`    | Rumuński              | Română           |
| `uk`    | Ukraiński             | Українська       |
| `pt-BR` | Brazylijski portugalski  | Português Brasileiro |
| `zh-HK` | Kantoński             | 粵語             |
| `ms`    | Malajski                 | Bahasa Melayu    |
| `sk`    | Słowacki                | Slovenčina       |
| `bg`    | Bułgarski             | Български        |
| `hr`    | Chorwacki              | Hrvatski         |
| `lt`    | Litewski            | Lietuvių         |
| `lv`    | Łotewski               | Latviešu         |
| `et`    | Estoński              | Eesti            |
| `sl`    | Słoweński             | Slovenščina      |

{% hint style="success" %}
**Automatyczne zachowanie**: Twoje preferencje językowe są zapisywane w `~/.chloros/cli_language.json` i zachowywane we wszystkich sesjach.
{% endhint %}

***

### `set-project-folder` - Ustaw domyślny folder projektu

Zmień lokalizację domyślnego folderu projektu (współdzieloną z GUI w Windows).

**Składnia:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Przykłady:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` — Pokaż folder projektu

Wyświetla aktualną lokalizację domyślnego folderu projektu.

**Składnia:**

```bash
chloros-cli get-project-folder
```

**Przykład:**

```bash
chloros-cli get-project-folder
```

**Wynik:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` — Przywróć ustawienia domyślne

Przywróć domyślną lokalizację folderu projektu.

**Składnia:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` — Uruchom diagnostykę systemu

Uruchamia 7 testów diagnostycznych w celu sprawdzenia konfiguracji systemu.

**Składnia:**

```bash
chloros-cli selftest
```

**Przeprowadzone testy diagnostyczne:**

1. Sprawdzenie wersji
2. Dostępność portu (5000)
3. Uruchomienie zaplecza
4. Test łączności API
5. Informacje o systemie i wykrywanie GPU
6. Weryfikacja modeli odszumiania
7. Sprawdzenie dostępności CUDA

{% hint style="info" %}
**Przydatne przy rozwiązywaniu problemów**: Uruchom `selftest` po instalacji, aby sprawdzić, czy system jest poprawnie skonfigurowany, zwłaszcza na Linux/Jetson, gdzie może być konieczna weryfikacja ustawień GPU i CUDA.
{% endhint %}

***

### `update` — Sprawdź dostępność aktualizacji (tylko Linux)

Sprawdź dostępność aktualizacji CLI i zainstaluj je na systemach Linux.

**Składnia:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opcja    | Opis                        |
| --------- | ---------------------------------- |
| `--check` | Tylko sprawdź dostępność aktualizacji, nie instaluj |

{% hint style="info" %}
To polecenie jest dostępne tylko w systemie Linux. W systemie Windows aktualizacje są dostarczane za pośrednictwem instalatora.
{% endhint %}

***

## Opcje globalne

Te opcje dotyczą wszystkich poleceń:

| Opcja            | Typ    | Domyślnie       | Opis                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Ścieżka    | Wykrywane automatycznie | Ścieżka do pliku wykonywalnego zaplecza                       |
| `--port`          | Liczba całkowita | 5000          | Numer portu zaplecza API                          |
| `--restart`       | Flaga    | -             | Wymuś ponowne uruchomienie backendu (zabija istniejące procesy) |
| `--version`       | Flaga    | -             | Pokaż informacje o wersji i zamknij program                |
| `--help`          | Flaga    | -             | Pokaż informacje pomocy i zamknij program                   |

{% hint style="info" %}
**Automatyczne wykrywanie backendu**: Ścieżka `--backend-exe` jest wykrywana automatycznie dla danej platformy:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ręcznie)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Przykład z opcjami globalnymi:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Przewodnik po ustawieniach przetwarzania

### Przetwarzanie równoległe i dynamiczna adaptacja obliczeniowa

Chloros 1.1.0 zawiera funkcję [dynamicznej adaptacji obliczeniowej](processing-architecture/dynamic-compute-adaptation.md) — silnik przetwarzania **automatycznie wykrywa sprzęt** i wybiera optymalną strategię:

| Platforma | Strategia | Pracownicy | Potok | Uwagi |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Efektywne wykorzystanie pamięci, sekwencyjne |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Równoległe przetwarzanie na GPU |
| **Komputer stacjonarny z procesorem graficznym 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobra wydajność komputera stacjonarnego |
| **Komputer stacjonarny z kartą graficzną 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optymalna wydajność komputera stacjonarnego |
| **System wyłącznie z procesorem** | `CPU_PARALLEL` | rdzenie - 1 | `cpu_fallback` | Nie wymaga karty graficznej |

{% hint style="success" %}
**Nie wymaga ręcznej konfiguracji!** Chloros automatycznie wykrywa procesor, kartę graficzną, pamięć RAM oraz (w przypadku Jetson) czujniki temperatury, a następnie automatycznie konfiguruje optymalny potok przetwarzania.
{% endhint %}

### Metody debayera

| Metoda | Flaga CLI | Jakość | Szybkość | Licencja |
| --- | --- | --- | --- | --- |
| **Standardowa (szybka, średnia jakość)** | `--debayer standard` | Dobra | Szybka | Bezpłatna / Chloros+ |
| **Z uwzględnieniem tekstury (wolna, najwyższa jakość)** | `--debayer texture-aware` | Najwyższa | Wolna | Tylko Chloros+ |

Domyślną metodą debayera jest **Standardowa**. Metoda**Z uwzględnieniem tekstury** wykorzystuje model usuwania szumów oparty na sztucznej inteligencji i uczeniu maszynowym, zapewniając najwyższą jakość wyników, ale wymaga licencji Chloros+ oraz procesora graficznego NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Korekcja winietowania

**Co robi:** Koryguje spadek jasności na krawędziach obrazu (ciemniejsze rogi, typowe dla zdjęć z aparatu).

* **Włączone domyślnie** — większość użytkowników powinna pozostawić tę opcję włączoną
* Użyj `--no-vignette`, aby wyłączyć

{% hint style="success" %}
**Zalecenie**: Zawsze włączaj korekcję winietowania, aby zapewnić jednolitą jasność w całej klatce.
{% endhint %}

### Kalibracja odbicia

Konwertuje surowe wartości czujnika na znormalizowane wartości procentowe odbicia przy użyciu paneli kalibracyjnych.

* **Włączone domyślnie** – niezbędne do analizy roślinności
* Wymaga paneli kalibracyjnych na zdjęciach
* Użyj `--no-reflectance`, aby wyłączyć

{% hint style="info" %}
**Wymagania**: Upewnij się, że panele kalibracyjne są odpowiednio naświetlone i widoczne na zdjęciach, aby zapewnić dokładną konwersję współczynnika odbicia.
{% endhint %}

### Korekcje PPK

**Co robi:** Stosuje korekty kinematyczne przetwarzane po zebraniu danych (Post-Processed Kinematic) przy użyciu danych dziennika DAQ-A-SD w celu poprawy dokładności GPS.

* **Domyślnie wyłączone**
* Użyj `--ppk`, aby włączyć
* Wymaga plików .daq w folderze projektu z czujnika światła DAQ-A-SD MAPIR.

### Formaty wyjściowe

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Głębia bitowa</th><th width="116.5999755859375">Rozmiar pliku</th><th>Najlepsze zastosowanie</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitowy)</strong> ⭐</td><td>16-bitowa liczba całkowita</td><td>Duży</td><td>Analiza GIS, fotogrametria (zalecane)</td></tr><tr><td><strong>TIFF (32-bitowe, procent)</strong></td><td>32-bitowa liczba zmiennoprzecinkowa</td><td>Bardzo duży</td><td>Analiza naukowa, badania</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bitowa liczba całkowita</td><td>Średni</td><td>Kontrola wzrokowa, udostępnianie w sieci</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>Liczba całkowita 8-bitowa</td><td>Mały</td><td>Szybki podgląd, skompresowany wynik</td></tr></tbody></table>***

## Automatyzacja i skrypty

### Przetwarzanie wsadowe w PowerShell (Windows)

Automatyczne przetwarzanie wielu folderów z zestawami danych w Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Skrypt wsadowy Windows (Windows)

Prosta pętla do przetwarzania wsadowego w Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Przetwarzanie wsadowe w Bash (Linux)

Przetwarzanie wielu folderów z zestawami danych w Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Skrypt automatyzacji Python (wieloplatformowy)

Zaawansowana automatyzacja z obsługą błędów (działa na Windows i Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Przebieg przetwarzania

### Standardowy przebieg

1. **Dane wejściowe**: Folder zawierający pary obrazów RAW/JPG
2. **Wykrywanie**: CLI automatycznie skanuje w poszukiwaniu obsługiwanych plików obrazów
3. **Przetwarzanie**: Tryb równoległy skaluje się do liczby rdzeni procesora (Chloros+)
4. **Wynik**: Tworzy podfoldery według modeli aparatów z przetworzonymi obrazami

### Przykładowa struktura wyników

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Szacunkowy czas przetwarzania

Typowy czas przetwarzania 100 obrazów (każdy o rozdzielczości 12 MP):

| Platforma | Tryb | Szacowany czas | Uwagi |
| --- | --- | --- | --- |
| **Komputer stacjonarny z kartą graficzną 12 GB+** | `GPU_PARALLEL` | 5–10 min | Najszybsza opcja |
| **Komputer stacjonarny z kartą graficzną 8 GB** | `GPU_SINGLE` | 10–15 min | Dobra wydajność |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 min | Przetwarzanie brzegowe |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Ograniczona pamięć |
| **Tylko procesor** | `CPU_PARALLEL` | 20–40 min | Nie wymaga procesora graficznego |

{% hint style="info" %}
**Wskazówka dotycząca wydajności**: Czas przetwarzania zależy od liczby obrazów, rozdzielczości, metody debayeringu oraz sprzętu. Debayering z uwzględnieniem tekstur trwa znacznie dłużej niż standardowy. Szczegółowe informacje można znaleźć w sekcji [Dynamiczna adaptacja obliczeniowa](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Rozwiązywanie problemów

### Nie znaleziono CLI

**Błąd Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Rozwiązania:**

1. Sprawdź lokalizację instalacji:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Użyj pełnej ścieżki, jeśli nie ma w PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Dodaj ręcznie do PATH:
   * Otwórz Właściwości systemu → Zmienne środowiskowe
   * Edytuj zmienną PATH
   * Dodaj: `C:\Program Files\Chloros\resources\cli`
   * Uruchom ponownie terminal

**Błąd Linux:**

```
chloros-cli: command not found
```

**Linux Rozwiązania:**

1. Sprawdź instalację:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Zrestrukturyzuj powłokę:

```bash
source ~/.bashrc
```

3. Sprawdź uprawnienia:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Nie udało się uruchomić backendu**Błąd:**

```

Backend failed to start within 30 seconds
```

**Rozwiązania:**

1. Sprawdź, czy backend już działa (najpierw go zamknij)
2. Sprawdź, czy zapora sieciowa nie blokuje połączenia (Windows) lub sprawdź dostępność portu (Linux: `lsof -i :5000`)
3. Wypróbuj inny port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Wymuś ponowne uruchomienie serwera:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. W przypadku Linux sprawdź, czy plik wykonywalny zaplecza istnieje:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problemy z licencją / uwierzytelnianiem**Błąd:**

```

Chloros+ license required for CLI access
```

**Rozwiązania:**

1. Sprawdź, czy posiadasz aktywną subskrypcję Chloros+
2. Zaloguj się przy użyciu swoich danych uwierzytelniających:

```bash
chloros-cli login user@example.com 'password'
```

3. Sprawdź status licencji:

```bash
chloros-cli status
```

4. Skontaktuj się z pomocą techniczną: info@mapir.camera

***

### Nie znaleziono obrazów**Błąd:**

```

No images found in the specified folder
```

**Rozwiązania:**

1. Sprawdź, czy folder zawiera obsługiwane formaty (.RAW, .TIF, .JPG)
2. Sprawdź, czy ścieżka do folderu jest poprawna (używaj cudzysłowów w przypadku ścieżek zawierających spacje)
3. Upewnij się, że masz uprawnienia do odczytu dla tego folderu
4. Sprawdź, czy rozszerzenia plików są poprawne

***

### Przetwarzanie zatrzymuje się lub zawiesza**Rozwiązania:**

1. Sprawdź dostępną przestrzeń dyskową (upewnij się, że jest wystarczająca dla wyników)
2. Zamknij inne aplikacje, aby zwolnić pamięć
3. Zmniejsz liczbę obrazów (przetwarzaj partiami)

***

### Port jest już używany**Błąd:**

```

Port 5000 is already in use
```

**Rozwiązania:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Często zadawane pytania

### P: Czy potrzebuję licencji na CLI?

**O:**Tak! CLI wymaga płatnej**licencji Chloros+**.

* ❌ Plan standardowy (bezpłatny): CLI wyłączony
* ✅ Plany Chloros+ (płatne): CLI w pełni włączony

Subskrybuj na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: Czy mogę używać CLI na serwerze bez GUI?**O:** Tak! CLI działa całkowicie bez interfejsu graficznego. Jest to główny przypadek użycia w Linux.**Serwer Windows:**
* Serwer Windows 2016 lub nowszy
* Zainstalowany pakiet Visual C++ Redistributable

**Serwer Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) lub JetPack 6 (arm64)
* Instalacja za pomocą pakietu `.deb`

**Obie platformy:**
* Minimum 8 GB pamięci RAM (zalecane 16 GB)
* Jednorazowa aktywacja licencji: `chloros-cli login user@example.com 'password'`

***

### P: Gdzie zapisywane są przetworzone obrazy?**O:**Domyślnie przetworzone obrazy są zapisywane w**tym samym folderze co pliki wejściowe**, w podfolderach nazwanych według modeli aparatów (np. `Survey3N_RGN/`).

Użyj opcji `-o`, aby określić inny folder wyjściowy:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### P: Czy mogę przetwarzać wiele folderów jednocześnie?**O:** Nie bezpośrednio za pomocą jednego polecenia, ale można użyć skryptów do sekwencyjnego przetwarzania folderów. Zobacz sekcję [Automatyzacja i skrypty](CLI.md#automation--scripting).***

### P: Jak zapisać dane wyjściowe CLI do pliku dziennika?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### P: Co się stanie, jeśli podczas przetwarzania naciśnę Ctrl+C?**Odp.:** CLI:

1. Płynnie zatrzyma przetwarzanie
2. Wyłączy backend
3. Zakończy działanie z kodem 130

Częściowo przetworzone obrazy mogą pozostać w folderze wyjściowym.

***

### Pyt.: Czy mogę zautomatyzować przetwarzanie CLI?**Odp.:** Oczywiście! Program CLI został zaprojektowany z myślą o automatyzacji. Zobacz [Automatyzacja i skrypty](CLI.md#automation--scripting) dla PowerShell (Windows), Batch (Windows), Bash (Linux) oraz Python (wieloplatformowe).***

### P: Jak sprawdzić wersję CLI?**Odp.:**

```bash
chloros-cli --version
```

**Wynik:**

```

Chloros CLI 1.1.0
```

***

## Pomoc

### Pomoc w wierszu poleceń

Wyświetl informacje pomocy bezpośrednio w CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanały wsparcia

* **E-mail**: info@mapir.camera
* **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Kompletne przykłady

### Przykład 1: Podstawowe przetwarzanie

Przetwarzanie z ustawieniami domyślnymi (winieta, odbicie):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Przykład 2: Wysokiej jakości wyniki naukowe

32-bitowa zmiennoprzecinkowa TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Przykład 3: Szybkie przetwarzanie podglądu

8-bitowy PNG bez kalibracji do szybkiego przeglądu:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Przykład 4: Przetwarzanie z korekcją PPK

Zastosuj korekcje PPK z wykorzystaniem współczynnika odbicia:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Przykład 5: Niestandardowa lokalizacja wyjściowa

Przetwarzaj do innej lokalizacji w określonym formacie:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Przykład 6: Proces uwierzytelniania

Kompletny proces uwierzytelniania (taki sam na wszystkich platformach):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Przykład 7: Obsługa wielu języków

Zmiana języka interfejsu (taka sama na wszystkich platformach):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
