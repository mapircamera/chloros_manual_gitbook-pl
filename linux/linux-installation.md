# Instalacja Linux

Chloros jest dystrybuowany dla Linux jako pakiety `.deb`, które instalują CLI oraz serwer zaplecza. Python SDK jest oddzielnym pakietem pip (dołączonym również do `.deb` jako plik wheel o dopasowanej wersji).

Nazwy plików pakietów zawierają informację o wersji i architekturze: `chloros_1.2.0_amd64.deb` dla x86_64 oraz `chloros_1.2.0_arm64_jp6.deb` dla kompilacji JetPack 6 Jetson. W poniższych poleceniach należy zastąpić nazwę pliku nazwą faktycznie pobranego pliku.

***

## Linux amd64 (x86_64)

### Wymagania systemowe

| Wymaganie | Minimalne | Zalecane |
| --- | --- | --- |
| **Dystrybucja** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 lub lepszy |
| **Pamięć (RAM)** | 8 GB | 16 GB lub więcej |
| **Karta graficzna** | Brak (przetwarzanie przez procesor) | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM (12 GB lub więcej odblokowuje `GPU_PARALLEL`, 7 GB lub więcej wyłącza funkcję Texture Aware w trybie pojedynczego obrazu) |
| **Pamięć masowa** | 2 GB wolnego miejsca | Dysk SSD z co najmniej 10 GB wolnego miejsca |
| **Python** | Python 3.7+ (dla SDK) | Python 3.10+ |

> **Ubuntu 20.04 i Debian 11 nie są obsługiwane.** Lista zależności `.deb`
> pochodzi z tego, z czym faktycznie łączy się backend Chloros, a obejmuje to
> `libc6 (>= 2.34)`. Zarówno Focal, jak i Bullseye zawierają bibliotekę glibc 2.31, więc `apt` odrzuca
> instalację od razu, zamiast dopuścić do jej niepowodzenia w późniejszym czasie podczas działania programu.

### Instalacja

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` nie rozstrzyga zależności. Jeśli zgłasza brakujące pakiety, `sudo apt-get install -f` (lub `sudo apt --fix-broken install`) kończy instalację — jest to normalny przebieg procesu, a nie błąd.
{% endhint %}

Sprawdź instalację:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Wymagania systemowe

| Wymaganie | Minimalne | Zalecane |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB lub AGX Orin |
| **JetPack** | JetPack 6.x | Najnowsza wersja JetPack 6 |
| **Pamięć (RAM)** | 8 GB (współdzielona przez GPU/CPU) | 16 GB+ współdzielonej pamięci (12 GB+ to próg dla równoległych procesów roboczych na GPU) |
| **Pamięć masowa** | 2 GB wolnego miejsca | Dysk SSD NVMe z co najmniej 10 GB wolnego miejsca |
| **Python** | Python 3.7+ (dla SDK) | Python 3.10+ |

### Instalacja

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ten sam układ co w przypadku amd64 `.deb`, z kompilacją CUDA dostosowaną do Jetson Orin / Orin NX / Orin Nano. Informacje na temat pamięci, charakterystyki termicznej i wdrażania w terenie urządzeń Jetson można znaleźć w [Przewodniku NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalacja Python i SDK (wszystkie wersje Linux)

SDK jest klientem typu „pure-Python HTTP” dla backendu, więc ten sam pakiet działa zarówno na architekturze amd64, jak i arm64. Dwa źródła:**Z PyPI** — opublikowana stabilna wersja:

```bash
pip install chloros-sdk
```

**Z dołączonego pliku wheel** — gwarantowana zgodność z właśnie zainstalowanym CLI/backendem (użyj tego, gdy twoja wersja `.deb` jest nowsza niż ta z PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Dystrybucje zgodne z PEP 668** (Ubuntu 23.10+, Debian 12+) nie zezwalają na instalacje pip w całym systemie. Użyj `pip install --user …`, środowisko wirtualne lub `sudo pip install --break-system-packages …`. Instalator pakietów nigdy nie instaluje automatycznie SDK w systemowym Python — wybór pozostaje w twojej gestii.
{% endhint %}

Opcjonalne dodatki:

| Dodatek | Polecenie | Dodaje |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` do strumieniowego przesyłania postępu na żywo |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` do transmisji BLE (DAQ-M) |

Sprawdź SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Plik `.deb` instaluje Chloros, CLI oraz moduł zaplecza. Python oraz SDK komunikują się z tym backendem za pośrednictwem lokalnego HTTP oraz API (`http://127.0.0.1:5000`) i uruchamia go automatycznie w razie potrzeby. Zawsze używaj dosłownego adresu IPv4 zamiast `localhost` — `localhost` może rozdzielać się na `::1` i kosztować około dwóch sekund na każde żądanie.
{% endhint %}

***

## Konfiguracja początkowa

### 1. Zaloguj się

Dostęp do adresów CLI i SDK wymaga płatnego planu Chloros+ (**Copper** lub wyższy), co jest egzekwowane po stronie serwera: użytkownik wylogowany otrzymuje `401 AUTH_REQUIRED`, a użytkownik korzystający z bezpłatnego planu (Iron) otrzymuje `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Dane uwierzytelniające są buforowane w `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Po każdej instalacji lub aktualizacji należy zalogować się ponownie.** Skrypt `prerm` dołączony do pakietu celowo czyści `~/.chloros/user_session.json` oraz buforowaną licencję dla każdego użytkownika na komputerze, dzięki czemu nowa kompilacja zawsze ponownie weryfikuje licencję, zamiast polegać na nieaktualnej pamięci podręcznej.
{% endhint %}

### 2. Sprawdź status swojej licencji

```bash
chloros-cli status
```

`chloros-cli status` działa na każdym poziomie (w tym bezpłatnym), więc zawsze możesz sprawdzić, dlaczego dostęp jest lub nie jest dostępny.

### 3. Uruchom diagnostykę systemu

```bash
chloros-cli selftest
```

Siedem testów jest wykonywanych po kolei, a polecenie zwraca wynik niezerowy, jeśli którykolwiek z nich zakończy się niepowodzeniem:

| # | Test | Co sprawdza |
| --- | --- | --- |
| 1 | **Wersja** | CLI podaje swoją wersję (`v1.2.0`). |
| 2 | **Port dostępny** | Port 5000 jest wolny *lub* już obsługiwany przez sprawny backend Chloros (co oznacza, że test zakończył się powodzeniem). |
| 3 | **Uruchomienie backendu** | Plik binarny backendu uruchamia się. |
| 4 | **Test API (`/api/test`)** | Serwer odpowiada `status: ok`. |
| 5 | **Informacje o systemie** | Wyświetla `GPU: <name>, CUDA: <bool>, PyTorch: <version>` z `/api/system-info`. |
| 6 | **Modele odszumiania** | Znajduje modele `*.pth.enc` (na Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + filtr odszumiający**| Funkcja „Texture Aware” faktycznie działa — wymaga CUDA**oraz** co najmniej jednego pliku modelu. |

Przebieg kończy się na `N/7 checks passed`, z wyszczególnieniem wszystkich niepowodzeń według nazwy.

### 4. Przetwarzanie pierwszego zbioru danych

```bash
chloros-cli process ~/datasets/flight001
```

***

## Pliki i katalogi

### Dla każdego użytkownika

Chloros przechowuje swoje poświadczenia, a CLI konfigurację w jednym katalogu działającym na wielu platformach, **`~/.chloros/`** (w Windows, `%USERPROFILE%\.chloros\`). Dwie pamięci podręczne specyficzne dla Linux działają natomiast zgodnie z konwencjami XDG — uwzględniają one ustawienia `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, jeśli zostały one zdefiniowane.

| Ścieżka | Przeznaczenie |
| --- | --- |
| `~/.chloros/user_session.json` | Pamięć podręczna sesji logowania zapisywana przez `chloros-cli login` (czyścona przy każdej instalacji/aktualizacji pakietu) |
| `~/.chloros/working_directory.txt` | Domyślne nadpisanie folderu projektu (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferencje językowe CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Ustawienie języka współdzielone z interfejsem graficznym Windows — wartość `language` ma tutaj pierwszeństwo przed `cli_language.json` |
| `~/.chloros/update_cache.json` | Godzinna pamięć podręczna dla sprawdzania aktualizacji podczas uruchamiania Linux/Jetson |
| `~/.chloros/backend.log` | Log zaplecza po uruchomieniu zaplecza przez CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pakiety kalibracyjne LATTICE dla poszczególnych kamer w pamięci podręcznej, indeksowane według numeru seryjnego i skrótu pakietu |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Opcjonalne nadpisania przez użytkownika profili korekcji ograniczeń DAQ |
| `~/.config/chloros/system_config.json` | Profil sprzętowy z bufora pochodzący z funkcji Dynamic Compute Adaptation — usunięcie go wymusza ponowne wykrycie sprzętu |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Logi serwera zaplecza, jeden plik na każde uruchomienie |
| `~/Chloros Projects/` | Domyślny folder projektu, gdy nie ustawiono żadnego nadpisania |

### Ogólnosystemowe

| Ścieżka | Przeznaczenie |
| --- | --- |
| `/usr/bin/chloros-cli` | Skrypt opakowujący — ustawia `LD_LIBRARY_PATH` dla dołączonych bibliotek natywnych, a następnie uruchamia rzeczywisty plik binarny |
| `/usr/bin/chloros-backend` | Skrypt opakowujący — to samo, plus `CHLOROS_PRODUCTION=1`, dzięki czemu bramka uwierzytelniająca backendu nigdy nie może się wyłączyć w tle |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Skompilowane pliki binarne |
| `/usr/lib/chloros/arena_runtime/` | Środowisko uruchomieniowe Arena SDK wymagane przez kamery LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Zaszyfrowane modele odszumiania używane przez algorytm debayera uwzględniający teksturę |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK – zestaw pasujący dokładnie do tej kompilacji |
| `/usr/lib/chloros/exiftool` | Dołączone narzędzie exiftool (z dowiązaniem symbolicznym do `/usr/local/bin/exiftool` tylko wtedy, gdy nie istnieje systemowe narzędzie exiftool) |
| `/etc/chloros/update.conf` | Konfiguracja kanału aktualizacji odczytywana przez `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Ustawia `net.ipv4.ip_unprivileged_port_start = 319`, aby backend mógł powiązać porty PTP bez uprawnień roota |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Kieruje dynamiczny moduł ładujący na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Przyznaje zalogowanemu użytkownikowi dostęp do mostka szeregowego DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Włącz opcję zawsze aktywnej usługi zaplecza (zainstalowanej, **nieaktywnej**) |
| `/usr/share/applications/chloros-cli.desktop` | Pozycja w menu aplikacji „Chloros CLI”, która otwiera terminal |

## Lokalizacja pliku wykonywalnego serwisu zaplecza

Pliki CLI i SDK automatycznie wykrywają serwis zaplecza:

| Komponent | Ścieżka |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Ścieżkę backendu można nadpisać za pomocą flagi `--backend-exe` CLI lub parametru konstruktora `backend_exe` SDK, a port za pomocą `--port` (domyślnie `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` wskazuje rodziny poleceń **`lattice`**,**`project`**, oraz**`daq pool-*`** na zdalny backend. Polecenia podstawowe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) celowo ją ignorują i zawsze kierują działanie na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamery LATTICE i czujniki światła DAQ na Linux

Wszystkie rodziny poleceń live-hardware działają na Linux (amd64 i Jetson):

* **`chloros-cli lattice`** — wykrywanie, łączenie, konfigurowanie i przechwytywanie obrazu z kamer LATTICE oraz zsynchronizowanych macierzy. `.deb` zawiera wymagane środowisko uruchomieniowe Arena SDK i rejestruje je w dynamicznym modułach ładującym.
* **`chloros-cli daq pool-*`** — podłączanie czujników światła DAQ-U/M/E poprzez pulę backendową, strumieniowe przesyłanie skalibrowanych widm oraz rejestrowanie plików `.daq`. Skompilowany CLI zawiera wyłącznie rodzinę `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — uruchamia zapisany projekt (wraz z kamerami, czujnikami i ustawieniami przetwarzania) w trybie bezinterfejsowym.
* **`chloros-cli time-sync`** — sprawdź serwer główny PTP, na którym działa backend Chloros dla kamer LATTICE i czujników DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` jest wymagany przez `pool-latest`, `pool-stream`, `pool-record` oraz `pool-set-cap`; `pool-list` pokazuje identyfikatory znajdujące się obecnie w puli.

{% hint style="info" %}
**W przypadku pierwszego połączenia z DAQ-E na komputerze z wieloma adresami sieciowymi zaleca się użycie identyfikatora `--eth-host`.** Funkcja automatycznego wykrywania przeszukuje mDNS i może pominąć interfejs czujnika z powodu pustej pamięci podręcznej ARP, więc pierwsze połączenie `pool-connect --eth` po uruchomieniu systemu może zakończyć się niepowodzeniem, nawet jeśli czujnik działa bez zarzutu. Podanie adresu IP lub nazwy hosta czujnika całkowicie pomija proces wykrywania.
{% endhint %}

**Uprawnienia szeregowe DAQ-U** są obsługiwane przez zainstalowaną regułę udev (`uaccess` + grupa `dialout`). Jeśli czujnik, który był już podłączony, pozostaje niedostępny, należy ponownie załadować reguły lub podłączyć go ponownie:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Pełny zestaw poleceń znajdziesz w [podręczniku CLI](../CLI.md).

### PTP zawsze włączony dla hostów bez monitorów

Podczas pierwszej instalacji generowana jest jednostka systemd `chloros-backend.service`, ale **nie jest ona włączona**. Na komputerze Jetson bez monitora lub serwerze, który powinien utrzymywać ciągłą synchronizację czasu PTP dla czujników DAQ-E i kamer LATTICE, należy ją włączyć:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Bez tego PTP działa tylko wtedy, gdy działa backend Chloros — czyli podczas aktywnej sesji CLI/SDK.

Urządzenie wiąże backend z `127.0.0.1:5000` (ustawienia środowiskowe `CHLOROS_HOST` / `CHLOROS_PORT` wewnątrz urządzenia; można je nadpisać za pomocą `sudo systemctl edit chloros-backend.service`) i uruchamia go ponownie po 5 sekundach w przypadku niepowodzenia.

**W jaki sposób PTP uzyskuje swoje porty.** PTP wykorzystuje porty UDP 319/320, oba poniżej standardowego progu 1024 portów uprzywilejowanych. Pakiet `postinst` zapisuje `/etc/sysctl.d/60-chloros-ptp.conf` z `net.ipv4.ip_unprivileged_port_start = 319`, co pozwala backendowi powiązać je podczas działania pod uprawnieniami użytkownika. Jako dodatkowe zabezpieczenie nakłada również plik `setcap cap_net_bind_service,cap_net_raw=+ep` na plik binarny backendu — dlatego właśnie plik `libcap2-bin` jest zadeklarowaną zależnością tego pakietu.***

## Przykłady skryptów w języku Bash

{% hint style="info" %}
**Kody wyjścia przyjazne dla skryptów.**`chloros-cli process` kończy działanie z kodem `0` w przypadku powodzenia oraz**z kodem niezerowym w przypadku niepowodzenia — w tym w przypadku uruchomienia, które zażądało produktów obrazowych, ale nie zapisało żadnego** (wyświetla kod `Processing finished but wrote no image products.` oraz podaje nazwę folderu projektu i typowe przyczyny). Pomyślne uruchomienia informują, ile produktów obrazowych zostało zapisanych (`Image products written: N`). Kody wyjścia: `0` — sukces, `1` — niepowodzenie, `2` — błąd argumentu, `130` — przerwanie.
{% endhint %}

### Przetwarzanie wielu zestawów danych

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Przetwarzanie z ustawieniami niestandardowymi

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Prawidłowych wartości `--format` są dokładnie cztery i zawierają spacje — należy je zawsze umieszczać w cudzysłowie:

| Wartość `--format` | Folder wyjściowy |
| --- | --- |
| `TIFF (16-bit)` *(domyślnie)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` akceptuje `standard` (domyślnie) lub `texture-aware` (Chloros+).

### Automatyczne przetwarzanie za pomocą Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Przykład Python SDK

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

### Nie znaleziono pliku CLI po instalacji

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Odmowa dostępu

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap failed” podczas instalacji

`.deb` stosuje `cap_net_bind_service` do `/usr/lib/chloros/chloros-backend`, aby umożliwić powiązanie portów PTP 319/320 bez uprawnień roota. Jeśli podczas instalacji brakowało pliku `libcap2-bin`, wywołanie jest pomijane. Zainstaluj go i ponownie zainstaluj pakiet:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP nie uruchamia się / nie można powiązać portu 319

Sprawdź, czy limit portów bez uprawnień został obniżony, a jeśli nie, zastosuj go ponownie dla bieżącego uruchomienia:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Następnie sprawdź serwer główny:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „Nie znaleziono sterowników kamer LATTICE”

Nie można rozpoznać środowiska uruchomieniowego Arena SDK. Sprawdź, czy konfiguracja modułu ładującego zapisywana przez pakiet jest obecna i zaktualizowana:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Nie udało się uruchomić modułu zaplecza

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Logi modułu zaplecza dotyczące nieudanego uruchomienia znajdują się w pliku `~/.cache/chloros/logs/`.

### Nie wykryto CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

Plik `chloros-cli selftest` zawiera tę samą informację w jednym wierszu: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Brakujące biblioteki współdzielone

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Powolne uruchamianie na systemach z kartą SD

Skompilowane pliki binarne rozpakowują się do katalogu tymczasowego przy każdym uruchomieniu. Jeśli plik `/mnt/ssd/tmp` istnieje, Chloros używa go automatycznie; w przeciwnym razie należy ustawić `TMPDIR` na szybki system plików:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualizacja Chloros na Linux

Polecenie `update` jest dostępne wyłącznie na Linux/Jetson. Sprawdza on wersję opublikowaną w kanale aktualizacji skonfigurowanym w `/etc/chloros/update.conf` i proponuje pobranie oraz zainstalowanie pasującej wersji `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Na Linux/Jetson CLI również przeprowadza nieblokujące sprawdzanie aktualizacji przy każdym uruchomieniu (wynik jest buforowany przez godzinę w `~/.chloros/update_cache.json`) i wyświetla komunikat `Update available: vX.Y.Z`, gdy dostępna jest nowsza wersja. Twoje ustawienia i projekty pozostają nienaruszone po aktualizacji; po jej zakończeniu konieczne będzie ponowne zalogowanie się.

## Odinstalowywanie

```bash
sudo apt remove chloros
```

Odinstalowanie zatrzymuje działanie `chloros-backend.service`, przywraca domyślny dolny limit portów bez uprawnień (1024), usuwa dowiązanie symboliczne do dołączonego narzędzia exiftool oraz konfigurację modułu ładującego Arena, a także czyści buforowane dane uwierzytelniające. Projekty i pliki danych `~/.chloros/` pozostają nienaruszone.

***

## Kolejne kroki

* [Przewodnik po NVIDIA Jetson](nvidia-jetson-guide.md) — optymalizacja i wdrażanie specyficzne dla Jetson
* [CLI : Wiersz poleceń](../CLI.md) — przewodnik CLI
* [API : Python SDK](../api-python-sdk.md) — przewodnik po SDK
* [CLI — dokumentacja](../reference/cli-reference.md) oraz [SDK — dokumentacja](../reference/sdk-reference.md) — wyczerpujące wykazy poleceń/API dla wersji 1.2.0
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — w jaki sposób Chloros dostosowuje się do posiadanego sprzętu
