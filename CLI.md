# CLI : Wiersz poleceń

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** zapewnia zaawansowany dostęp z poziomu wiersza poleceń do silnika przetwarzania obrazów Chloros, umożliwiając automatyzację, tworzenie skryptów i pracę bez monitorowania dla procesów przetwarzania obrazów.

### Najważniejsze cechy

* 🚀 **Automatyzacja** — skryptowe przetwarzanie wsadowe wielu zestawów danych
* 🔗 **Integracja** — osadzenie w istniejących przepływach pracy i potokach
* 💻 **Praca bez interfejsu graficznego** — działanie bez GUI
* 🌍 **Wielojęzyczność** — obsługa 38 języków
* ⚡ **Równoległe przetwarzanie** — dynamiczna skalowalność do mocy procesora (do 16 równoległych procesów)

### Wymagania

| Wymaganie          | Szczegóły                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **System operacyjny** | Windows 10/11 (64-bitowy)                                              |
| **Licencja**          | Chloros+ ([wymagany płatny plan](https://cloud.mapir.camera/pricing)) |
| **Pamięć**           | Minimum 8 GB pamięci RAM (zalecane 16 GB)                                  |
| **Internet**         | Wymagany do aktywacji licencji                                     |
| **Miejsce na dysku**       | Zależy od wielkości projektu                                              |

{% hint style=&quot;warning&quot; %}
**Wymagania licencyjne**: CLI wymaga płatnej subskrypcji Chloros+. Standardowe (bezpłatne) plany nie mają dostępu do CLI. Odwiedź [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), aby dokonać aktualizacji.
{% endhint %}

## Szybki start

### Instalacja

CLI jest automatycznie dołączany do instalatora Chloros:

1. Pobierz i uruchom **Chloros Installer.exe**

2. Ukończ kreatora instalacji
3. CLI zainstalowany w: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Instalator automatycznie dodaje `chloros-cli` do ścieżki PATH systemu. Po zakończeniu instalacji uruchom ponownie terminal.
{% endhint %}

### Pierwsza konfiguracja

Przed użyciem CLI należy aktywować licencję Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Podstawowe zastosowanie

Przetwarzanie folderu przy użyciu ustawień domyślnych:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Opis poleceń

### Ogólna składnia

```
chloros-cli [global-options] <command> [command-options]
```

***

## Polecenia

### `process` — przetwarzanie obrazów

Przetwarzanie obrazów w folderze z kalibracją.

**Składnia:**

```bash
chloros-cli process <input-folder> [options]
```

**Przykład:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opcje polecenia przetwarzania

| Opcja                | Typ    | Domyślna        | Opis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Ścieżka    | _Wymagane_     | Folder zawierający obrazy wielospektralne RAW/JPG                                         |
| `-o, --output`        | Ścieżka    | Tak samo jak dane wejściowe  | Folder wyjściowy dla przetworzonych obrazów                                                     |
| `-n, --project-name`  | Ciąg znaków  | Generowany automatycznie | Niestandardowa nazwa projektu                                                                    |
| `--vignette`          | Flaga    | Włączone        | Włącz korekcję winietowania                                                             |
| `--no-vignette`       | Flaga    | -              | Wyłącz korekcję winietowania                                                            |
| `--reflectance`       | Flaga    | Włączona        | Włącz kalibrację odbicia                                                         |
| `--no-reflectance`    | Flaga    | -              | Wyłącz kalibrację odbicia                                                        |
| `--ppk`               | Flaga    | Wyłączone       | Zastosuj korekty PPK z danych czujnika światła .daq                                      |
| `--format`            | Wybór  | TIFF (16-bit)  | Format wyjściowy: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Liczba całkowita | Auto           | Minimalny rozmiar docelowy w pikselach dla wykrywania panelu kalibracyjnego                          |
| `--target-clustering` | Liczba całkowita | Auto           | Próg grupowania docelowego (0-100)                                                    |
| `--exposure-pin-1`    | Ciąg znaków  | Brak           | Blokada ekspozycji dla modelu kamery (pin 1)                                                 |
| `--exposure-pin-2`    | Ciąg znaków  | Brak           | Blokada ekspozycji dla modelu kamery (pin 2)                                                 |
| `--recal-interval`    | Liczba całkowita | Auto           | Interwał rekalibracji w sekundach                                                      |
| `--timezone-offset`   | Liczba całkowita | 0              | Przesunięcie strefy czasowej w godzinach                                                               |

***

### `login` — uwierzytelnianie konta

Zaloguj się przy użyciu danych uwierzytelniających Chloros+, aby włączyć przetwarzanie CLI.

**Składnia:**

```bash
chloros-cli login <email> <password>
```

**Przykład:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Znaki specjalne**: Używaj pojedynczych cudzysłowów wokół haseł zawierających znaki takie jak `$`, `!` lub spacje.
{% endhint %}

**Wynik:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` — Wyczyść dane logowania

Wyczyść zapisane dane logowania i wyloguj się ze swojego konta.

**Składnia:**

```bash
chloros-cli logout
```

**Przykład:**

```powershell
chloros-cli logout
```

**Wynik:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**Użytkownicy SDK**: Python SDK udostępnia również programową metodę `logout()` do czyszczenia poświadczeń w skryptach Python. Szczegółowe informacje można znaleźć w [dokumentacji Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` — sprawdzanie statusu licencji

Wyświetla aktualny status licencji i uwierzytelnienia.

**Składnia:**

```bash
chloros-cli status
```

**Przykład:**

```powershell
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

### `export-status` — sprawdzanie postępu eksportu

Monitorowanie postępu eksportu wątku 4 podczas przetwarzania lub po jego zakończeniu.

**Składnia:**

```bash
chloros-cli export-status
```

**Przykład:**

```powershell
chloros-cli export-status
```

**Przykład użycia:** Wywołaj to polecenie podczas przetwarzania, aby sprawdzić postęp eksportu.***

### `language` — zarządzanie językiem interfejsu

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

```powershell
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

| Kod    | Język              | Nazwa natywna      |
| ------- | --------------------- | ---------------- |
| `en`    | Angielski               | English          |
| `es`    | Hiszpański               | Español          |
| `pt`    | Portugalski            | Português        |
| `fr`    | Francuski                | Français         |
| `de`    | Niemiecki                | Deutsch          |
| `it`    | Włoski               | Italiano         |
| `ja`    | Japoński              | 日本語              |
| `ko`    | Koreański                | 한국어              |
| `zh`    | Chiński (uproszczony)  | 简体中文             |
| `zh-TW` | Chiński (tradycyjny) | 繁體中文             |
| `ru`    | Rosyjski               | Русский          |
| `nl`    | Holenderski                 | Nederlands       |
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
| `et`    | estoński              | Eesti            |
| `sl`    | słoweński             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatyczne zachowanie**: Twoje preferencje językowe są zapisywane w `~/.chloros/cli_language.json` i zachowywane we wszystkich sesjach.
{% endhint %}

***

### `set-project-folder` — ustaw domyślny folder projektu

Zmień domyślną lokalizację folderu projektu (współdzieloną z GUI).

**Składnia:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Przykład:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` — Pokaż folder projektu

Wyświetla bieżącą domyślną lokalizację folderu projektu.

**Składnia:**

```bash
chloros-cli get-project-folder
```

**Przykład:**

```powershell
chloros-cli get-project-folder
```

**Wynik:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` — przywróć domyślne ustawienia

Przywróć domyślną lokalizację folderu projektu.

**Składnia:**

```bash
chloros-cli reset-project-folder
```

***

## Opcje globalne

Te opcje mają zastosowanie do wszystkich poleceń:

| Opcja          | Typ    | Domyślna       | Opis                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Ścieżka    | Wykrywana automatycznie | Ścieżka do pliku wykonywalnego zaplecza                       |
| `--port`        | Liczba całkowita | 5000          | Numer portu zaplecza API                          |
| `--restart`     | Flaga    | -             | Wymuś ponowne uruchomienie backendu (zabija istniejące procesy) |
| `--version`     | Flaga    | -             | Pokaż informacje o wersji i zamknij                |
| `--help`        | Flaga    | -             | Pokaż informacje pomocy i zamknij                   |

**Przykład z opcjami globalnymi:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Przewodnik po ustawieniach przetwarzania

### Przetwarzanie równoległe

Chloros+ CLI **automatycznie skaluje**przetwarzanie równoległe, aby dopasować je do możliwości komputera:**Jak to działa:**

* Wykrywa rdzenie procesora i pamięć RAM
* Przydziela procesy robocze: **2× rdzenie procesora** (wykorzystuje technologię hyperthreading)
* **Maksymalnie: 16 równoległych procesów roboczych** (dla stabilności)**Poziomy systemu:**

| Typ systemu   | Procesor        | Pamięć RAM      | Procesy robocze  | Wydajność     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Wysoka klasy**  | 16+ rdzeni  | 32+ GB   | Do 16 | Maksymalna prędkość   |
| **Średni** | 8–15 rdzeni | 16–31 GB | 8–16     | Doskonała prędkość |
| **Niski**   | 4–7 rdzeni  | 8–15 GB  | 4–8      | Dobra prędkość      |

{% hint style=&quot;success&quot; %}
**Automatyczna optymalizacja**: CLI automatycznie wykrywa specyfikację systemu i konfiguruje optymalne przetwarzanie równoległe. Nie jest wymagana ręczna konfiguracja!
{% endhint %}

### Metody debayerowania

CLI używa **Wysokiej jakości (szybszej)** jako domyślnego i zalecanego algorytmu debayerowania:

| Metoda                      | Jakość | Szybkość | Opis                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Wysoka jakość (szybsza)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algorytm uwzględniający krawędzie (domyślny, zalecany) |

### Korekcja winietowania

**Funkcja:** Koryguje spadek jasności na krawędziach obrazu (ciemniejsze rogi, często występujące w obrazach z kamer).

* **Włączone domyślnie** — większość użytkowników powinna pozostawić tę opcję włączoną
* Aby wyłączyć, użyj `--no-vignette`

{% hint style=&quot;success&quot; %}
**Zalecenie**: Zawsze włączaj korekcję winietowania, aby zapewnić jednolitą jasność w całym kadrze.
{% endhint %}

### Kalibracja odbicia

Konwertuje surowe wartości czujnika na znormalizowane wartości procentowe odbicia za pomocą paneli kalibracyjnych.

* **Włączone domyślnie** — niezbędne do analizy roślinności.
* Wymaga paneli kalibracyjnych w obrazach.
* Aby wyłączyć, użyj `--no-reflectance`.

{% hint style=&quot;info&quot; %}
**Wymagania**: Aby zapewnić dokładną konwersję współczynnika odbicia, upewnij się, że panele kalibracyjne są prawidłowo naświetlone i widoczne na obrazach.
{% endhint %}

### Korekty PPK

**Funkcja:** Stosuje korekty kinematyczne przetworzone po zakończeniu procesu przy użyciu danych dziennika DAQ-A-SD w celu poprawy dokładności GPS.

* **Domyślnie wyłączone**
* Aby włączyć, użyj `--ppk`
* Wymaga plików .daq w folderze projektu z czujnika światła MAPIR DAQ-A-SD.

### Formaty wyjściowe

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Głębia bitowa</th><th width="116.5999755859375">Rozmiar pliku</th><th>Najlepszy dla</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitowy)</strong> ⭐</td><td>16-bitowa liczba całkowita</td><td>Duże</td><td>Analiza GIS, fotogrametria (zalecane)</td></tr><tr><td><strong>TIFF (32-bitowa, procentowa)</strong></td><td>32-bitowa liczba zmiennoprzecinkowa</td><td>Bardzo duża</td><td>Analiza naukowa, badania</td></tr><tr><td><strong>PNG (8-bitowy)</strong></td><td>8-bitowa liczba całkowita</td><td>Średni</td><td>Kontrola wzrokowa, udostępnianie w sieci</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bitowa liczba całkowita</td><td>Mały</td><td>Szybki podgląd, skompresowane dane wyjściowe</td></tr></tbody></table>***

## Automatyzacja i skrypty

### Przetwarzanie wsadowe PowerShell

Automatyczne przetwarzanie wielu folderów zestawów danych:

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

### Skrypt wsadowy Windows

Prosta pętla do przetwarzania wsadowego:

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

### Python Skrypt automatyzacji

Zaawansowana automatyzacja z obsługą błędów:

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

## Przepływ pracy przetwarzania

### Standardowy przepływ pracy

1. **Dane wejściowe**: Folder zawierający pary obrazów RAW/JPG
2. **Wykrywanie**: CLI automatycznie skanuje w poszukiwaniu obsługiwanych plików obrazów
3. **Przetwarzanie**: Tryb równoległy skaluje się do liczby rdzeni procesora (Chloros+)
4. **Wynik**: Tworzy podfoldery modeli aparatów z przetworzonymi obrazami

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

### Szacowany czas przetwarzania

Typowy czas przetwarzania 100 obrazów (każdy o rozdzielczości 12 MP):

| Tryb              | Czas      | Sprzęt                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Tryb równoległy** | 5–10 min  | i7/Ryzen 7, 16 GB pamięci RAM, dysk SSD (do 16 procesorów) |
| **Tryb równoległy** | 10–15 min | i5/Ryzen 5, 8 GB pamięci RAM, dysk twardy (do 8 procesorów)   |

{% hint style=&quot;info&quot; %}
**Wskazówka dotycząca wydajności**: Czas przetwarzania zależy od liczby obrazów, rozdzielczości i specyfikacji komputera.
{% endhint %}

***

## Rozwiązywanie problemów

### CLI Nie znaleziono

**Błąd:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Rozwiązania:**

1. Sprawdź lokalizację instalacji:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Użyj pełnej ścieżki, jeśli nie ma jej w PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Dodaj ręcznie do PATH:
   * Otwórz Właściwości systemu → Zmienne środowiskowe
   * Edytuj zmienną PATH
   * Dodaj: `C:\Program Files\Chloros\resources\cli`
   * Uruchom ponownie terminal

***

### Nie udało się uruchomić zaplecza**Błąd:**

```

Backend failed to start within 30 seconds
```

**Rozwiązania:**

1. Sprawdź, czy zaplecze już działa (najpierw je zamknij)
2. Sprawdź, czy zapora sieciowa Windows nie blokuje
3. Wypróbuj inny port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Wymuś ponowne uruchomienie backendu:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemy z licencją / uwierzytelnianiem**Błąd:**

```

Chloros+ license required for CLI access
```

**Rozwiązania:**

1. Sprawdź, czy masz aktywną subskrypcję Chloros+.
2. Zaloguj się przy użyciu swoich danych uwierzytelniających:

```powershell
chloros-cli login user@example.com 'password'
```

3. Sprawdź status licencji:

```powershell
chloros-cli status
```

4. Skontaktuj się z pomocą techniczną: info@mapir.camera

***

### Nie znaleziono obrazów**Błąd:**

```

No images found in the specified folder
```

**Rozwiązania:**

1. Sprawdź, czy folder zawiera obsługiwane formaty (.RAW, .TIF, .JPG).
2. Sprawdź, czy ścieżka folderu jest poprawna (użyj cudzysłowów dla ścieżek zawierających spacje).
3. Upewnij się, że masz uprawnienia do odczytu folderu.
4. Sprawdź, czy rozszerzenia plików są poprawne.

***

### Przetwarzanie zatrzymuje się lub zawiesza**Rozwiązania:**

1. Sprawdź dostępną przestrzeń dyskową (upewnij się, że jest wystarczająca do przetworzenia danych).
2. Zamknij inne aplikacje, aby zwolnić pamięć.
3. Zmniejsz liczbę obrazów (przetwarzaj partiami).

***

### Port jest już używany**Błąd:**

```

Port 5000 is already in use
```

**Rozwiązanie:**

Określ inny port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Często zadawane pytania

### P: Czy potrzebuję licencji na CLI?

**O:**Tak! CLI wymaga płatnej**licencji Chloros+**.

* ❌ Plan standardowy (bezpłatny): CLI wyłączony
* ✅ Plany Chloros+ (płatne): CLI w pełni włączony

Subskrybuj na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: Czy mogę używać CLI na serwerze bez GUI?**O:** Tak! CLI działa całkowicie bez interfejsu graficznego. Wymagania:

* Windows Server 2016 lub nowszy
* Zainstalowany pakiet Visual C++ Redistributable
* Wystarczająca ilość pamięci RAM (minimum 8 GB, zalecane 16 GB)
* Jednorazowa aktywacja licencji GUI na dowolnym komputerze

***

### P: Gdzie są zapisywane przetworzone obrazy?**O:**Domyślnie przetworzone obrazy są zapisywane w**tym samym folderze co dane wejściowe** w podfolderach modeli kamer (np. `Survey3N_RGN/`).

Aby określić inny folder wyjściowy, użyj opcji `-o`:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### P: Czy mogę przetwarzać wiele folderów jednocześnie?**O:** Nie bezpośrednio za pomocą jednego polecenia, ale można użyć skryptów do przetwarzania folderów sekwencyjnie. Zobacz sekcję [Automatyzacja i skrypty](CLI.md#automation--scripting).***

### P: Jak zapisać dane wyjściowe CLI w pliku dziennika?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Plik wsadowy:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### P: Co się stanie, jeśli podczas przetwarzania naciśnę klawisze Ctrl+C?**O:** CLI:

1. Płynnie zatrzyma przetwarzanie
2. Wyłączy backend
3. Zakończy działanie z kodem 130

Częściowo przetworzone obrazy mogą pozostać w folderze wyjściowym.

***

### P: Czy mogę zautomatyzować przetwarzanie CLI?**O:** Oczywiście! CLI jest przeznaczony do automatyzacji. Przykłady dla PowerShell, Batch i Python można znaleźć w sekcji [Automatyzacja i skrypty](CLI.md#automation--scripting).***

### P: Jak sprawdzić wersję CLI?**O:**

```powershell
chloros-cli --version
```

**Wynik:**

```

Chloros CLI 1.0.2
```

***

## Pomoc

### Pomoc wiersza poleceń

Informacje pomocy można wyświetlić bezpośrednio w CLI:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanały pomocy technicznej

* **E-mail**: info@mapir.camera
* **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Kompletne przykłady

### Przykład 1: Podstawowe przetwarzanie

Przetwarzanie z ustawieniami domyślnymi (winieta, współczynnik odbicia):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Przykład 2: Wysokiej jakości wyniki naukowe

32-bitowy zmiennoprzecinkowy TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Przykład 3: Szybkie przetwarzanie podglądu

8-bitowy PNG bez kalibracji do szybkiego przeglądu:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Przykład 4: Przetwarzanie z korekcją PPK

Zastosowanie korekcji PPK z odbiciem:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Przykład 5: Niestandardowa lokalizacja wyników

Przetwarzanie na innym dysku w określonym formacie:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Przykład 6: Przepływ pracy uwierzytelniania

Zakończ przepływ uwierzytelniania:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Przykład 7: Użycie wielu języków

Zmień język interfejsu:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
