# Instalacja Linux

Chloros jest dystrybuowany dla Linux w postaci pakietów `.deb`, które instalują CLI oraz moduł zaplecza. Python SDK instaluje się osobno za pomocą pip.

***

## Linux amd64 (x86_64)

### Wymagania systemowe

| Wymaganie | Minimalne | Zalecane |
| --- | --- | --- |
| **Dystrybucja** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 lub lepszy |
| **Pamięć (RAM)** | 8 GB | 16 GB lub więcej |
| **Karta graficzna** | Brak (przetwarzanie przez procesor) | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM |
| **Pamięć masowa** | 2 GB wolnego miejsca | Dysk SSD z co najmniej 10 GB wolnego miejsca |
| **Python** | Python 3.7+ (dla SDK) | Python 3.10+ |

### Instalacja

Pobierz pakiet `.deb` i zainstaluj:

```bash
sudo dpkg -i chloros-amd64.deb
```

Sprawdź instalację:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Wymagania systemowe

| Wymaganie | Minimalne | Zalecane |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB lub AGX Orin |
| **JetPack** | JetPack 6.x | Najnowszy JetPack 6 |
| **Pamięć (RAM)** | 8 GB (współdzielona przez GPU/CPU) | 16 GB+ współdzielona |
| **Pamięć masowa** | 2 GB wolnego miejsca | Dysk SSD NVMe z co najmniej 10 GB wolnego miejsca |
| **Python** | Python 3.7+ (dla SDK) | Python 3.10+ |

### Instalacja

Pobierz pakiet JetPack 6 `.deb` i zainstaluj:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Sprawdź instalację:

```bash
chloros-cli --version
```

Szczegółowe informacje na temat konfiguracji Jetson, w tym zarządzania temperaturą i wdrożenia w terenie, znajdziesz w [Przewodniku NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalacja Python SDK (wszystkie Linux)

Python SDK jest instalowany oddzielnie za pomocą pip i działa zarówno na amd64, jak i arm64:

```bash
pip install chloros-sdk
```

Aby włączyć opcjonalną obsługę strumieniowania postępu:

```bash
pip install chloros-sdk[progress]
```

Sprawdź SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pakiet `.deb` instaluje Chloros, CLI oraz backend. Python SDK to oddzielny pakiet pip, który komunikuje się z backendem za pośrednictwem lokalnego HTTP API.
{% endhint %}

***

## Katalogi konfiguracyjne

Chloros w Linux jest zgodny ze [specyfikacją katalogu bazowego XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Cel | Linux Ścieżka | Windows Odpowiednik |
| --- | --- | --- |
| **Konfiguracja** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Dane / Projekty** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Pamięć podręczna / poświadczenia** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Lokalizacje plików wykonywalnych zaplecza

Pakiet `.deb` instaluje zaplecze w standardowej lokalizacji. Pakiety CLI i SDK automatycznie wykrywają ścieżkę do backendu:

| Metoda instalacji | Ścieżka do backendu |
| --- | --- |
| Pakiet `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ręczna / niestandardowa | `/opt/mapir/chloros/backend/chloros-backend` |

Ścieżkę backendu można nadpisać za pomocą flagi `--backend-exe` CLI lub parametru konstruktora `backend_exe` SDK.

***

## Pierwsza konfiguracja

### 1. Aktywuj swoją licencję

Do uzyskania dostępu do CLI i SDK wymagana jest licencja Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Sprawdź status swojej licencji

```bash
chloros-cli status
```

### 3. Przetwórz swój pierwszy zbiór danych

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Uruchom diagnostykę systemu

Sprawdź, czy system jest poprawnie skonfigurowany:

```bash
chloros-cli selftest
```

Spowoduje to uruchomienie 7 testów diagnostycznych, w tym sprawdzanie wersji, uruchamiania zaplecza, API łączności oraz dostępności CUDA/GPU.

***

## Przykłady skryptów Bash

### Przetwarzanie wielu zestawów danych

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Przetwarzanie z ustawieniami niestandardowymi

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatyczne przetwarzanie za pomocą Cron

Dodaj do swojego crontabu (`crontab -e`), aby automatycznie przetwarzać nowe zbiory danych:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Przykład

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Rozwiązywanie problemów

### CLI Nie znaleziono po instalacji

Jeśli po zainstalowaniu pakietu `.deb` nie można znaleźć `chloros-cli`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Odmowa dostępu

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Nie udało się uruchomić zaplecza

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### Nie wykryto CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Brakujące biblioteki współdzielone

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Aktualizacja Chloros na Linux

Użyj wbudowanego polecenia aktualizacji, aby sprawdzić dostępność aktualizacji i je zainstalować:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Kolejne kroki

* [Przewodnik po NVIDIA Jetson](nvidia-jetson-guide.md) — Optymalizacja i wdrażanie specyficzne dla Jetson
* [CLI : Wiersz poleceń](../CLI.md) — Pełny opis poleceń CLI
* [API : Python SDK](../api-python-sdk.md) — Pełny opis SDK
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — Jak Chloros dostosowuje się do sprzętu
