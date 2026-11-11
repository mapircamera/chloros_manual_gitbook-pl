# Instalacja w systemie Linux

Chloros jest dystrybuowany dla systemu Linux w postaci pakietów o nazwach `.deb`, które instalują interfejs CLI, serwer zaplecza oraz SDK dla języka Python (w przypadku systemu `python3` dodatkowo zawiera plik wheel o dopasowanej wersji dla środowisk virtualenv).

Nazwy plików pakietów zawierają informację o wersji i architekturze: `chloros_1.3.0_amd64.deb` dla x86_64, a `chloros_1.3.0_arm64_jp6.deb` dla arm64 (NVIDIA Jetson, JetPack 6). W poniższych poleceniach należy zastąpić nazwę pliku nazwą faktycznie pobranego pliku.

{% hint style="info" %}
**Nowość w wersji 1.3.0: programy są dostarczane w postaci rozpakowanych plików.** Backend i interfejs CLI są instalowane jako foldery, `/usr/lib/chloros/backend/` i `/usr/lib/chloros/cli/`, zamiast pojedynczych plików samorozpakowujących się. Nie rozpakowują się już one do folderu tymczasowego przy każdym uruchomieniu, dzięki czemu uruchamiają się znacznie szybciej (na x86_64 backend zajmował około 100 sekund na rozpakowanie przy każdym starcie) i nie wymagają już kilku GB wolnego miejsca w folderze `/tmp`. Rozmiar folderu `.deb` dla architektury amd64 również zmniejszył się o około połowę.
{% endhint %}

***

## Linux, amd64 (x86_64)

### Wymagania systemowe

| Wymaganie | Minimalne | Zalecane |
| --- | --- | --- |
| **Dystrybucja** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 lub lepszy |
| **Pamięć (RAM)** | 8 GB | 16 GB lub więcej |
| **Karta graficzna** | Brak (przetwarzanie przez procesor) | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM (12 GB lub więcej odblokowuje `GPU_PARALLEL`, 7 GB lub więcej pozwala na wyłączenie funkcji Texture Aware w ścieżce pojedynczego obrazu) |
| **Pamięć masowa** | 2 GB wolnego miejsca | Dysk SSD z co najmniej 10 GB wolnego miejsca |
| **Python** | Python 3.7 lub nowszy (dla SDK) | Python 3.10 lub nowszy |

> **Systemy Ubuntu 20.04 i Debian 11 nie są obsługiwane.** Lista zależności `.deb`
> pochodzi z tego, z czym faktycznie łączy się backend Chloros, a obejmuje to
> `libc6` nowszą niż glibc 2.31 dostarczana w wydaniach focal i bullseye, więc `apt` od razu odrzuca
> instalację, zamiast dopuścić do jej niepowodzenia w późniejszym czasie podczas działania programu.

### Instalacja

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Należy użyć `apt install ./…`, a nie `dpkg -i`.** `apt` rozwiązuje zależności pakietu, a także instaluje zalecane elementy: wymagania Python SDK, `python3-numpy` oraz `python3-requests`. `dpkg -i` nie instaluje żadnego z nich. CLI i backend są samowystarczalne i działają bez nich, ale `import chloros_sdk` ich potrzebuje; wynik instalacji wymienia wszystkie brakujące elementy. Zachowaj `./`, aby `apt` zainstalował ten plik zamiast szukać pakietu o tej nazwie. Pakiet deklaruje również `libgl1`, który jest ładowany przez dołączony OpenCV i może go brakować w minimalnym obrazie serwera.
{% endhint %}

Sprawdź instalację:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux, arm64 (NVIDIA Jetson)

### Wymagania systemowe

| Wymaganie | Minimalne | Zalecane |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB lub AGX Orin |
| **JetPack** | JetPack 6.x | Najnowsza wersja JetPack 6 |
| **Pamięć (RAM)** | 8 GB (współdzielona przez GPU/CPU) | 16 GB+ współdzielonej pamięci (12 GB+ to próg wymagany do przetwarzania równoległego) |
| **Pamięć masowa** | 2 GB wolnego miejsca | Dysk SSD NVMe z co najmniej 10 GB wolnego miejsca |
| **Python** | Python 3.7+ (dla SDK) | Python 3.10+ |

### Instalacja

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ten sam układ co w przypadku amd64 `.deb` (w zestawie Python SDK, bufory odbiorcze i usługi zaplecza), dla Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Zastosowany PyTorch to kompilacja CUDA z JetPack 6 dla Orin, więc przetwarzanie na GPU, w tym algorytm debayeringu z uwzględnieniem tekstur (Texture Aware), odbywa się na procesorze graficznym Jetson. Pakiet nie zawiera samego CUDA: korzysta z bibliotek CUDA i cuDNN zainstalowanych przez JetPack 6 (`sudo apt install nvidia-jetpack` na obrazie, na którym ich brakuje). W przypadku braku dostępnego procesora graficznego z obsługą CUDA przetwarzanie odbywa się na procesorze centralnym, a `--debayer texture-aware` wykorzystuje algorytm debayera Standard (backend rejestruje jeden wiersz `[DEBAYER] … needs a CUDA GPU …` wyjaśniający powód). Informacje na temat pamięci, temperatury i zachowania urządzenia Jetson w warunkach terenowych można znaleźć w [Przewodniku po NVIDIA Jetson](nvidia-jetson-guide.md).***

## Instalacja Python SDK (wszystkie systemy Linux)**`.deb` automatycznie instaluje SDK.** Oba pakiety (amd64 i Jetson) instalują go dla systemu `python3`, więc `import chloros_sdk` działa zaraz po instalacji, bez pip i bez połączenia z siecią. Jego pakiety (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` oraz moduły ładujące projekt) trafiają do `/usr/lib/python3/dist-packages`, należącego do pakietu: aktualizacja je zastępuje, a usunięcie pakietu powoduje ich usunięcie. Wersja zawsze odpowiada zainstalowanemu interfejsowi CLI i backendowi.

Własne wymagania SDK, `python3-numpy` i `python3-requests`, są **zalecane, a nie wymagane** przez pakiet: `sudo apt install ./chloros_<version>_<arch>.deb` instaluje je, a `dpkg -i` nie. Wynik instalacji zawiera informację o wersji SDK, którą importuje `python3`, o wszelkich brakujących wymaganiach, oraz wszelkie pliki `chloros_sdk` znajdujące się w innym miejscu na ścieżce `python3`, które zastępują kopię zawartą w pakiecie (np. zainstalowane wcześniej za pomocą `pip install --user`).**W środowisku virtualenv**, które nie ma dostępu do pakietów systemowych, zainstaluj w nim dołączony plik wheel (całkowicie w trybie offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Z PyPI** — tylko dla komputera, na którym nie jest zainstalowany pakiet Chloros, a którego SDK steruje backendem na innym komputerze (zobacz [Tryb zdalnego backendu](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Opcjonalne dodatki do instalacji za pomocą pip (w środowisku virtualenv lub z PyPI):

| Dodatek | Polecenie | Dodaje |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` do strumieniowego przesyłania postępu na żywo |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` do transportu BLE (DAQ-M) |

Sprawdź zestaw SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pythonowe SDK komunikuje się z zapleczem, które `.deb` instaluje za pośrednictwem lokalnego HTTP API (`http://127.0.0.1:5000`). `ChlorosLocal` oraz pomocnicze funkcje łączenia uruchamiają go automatycznie w razie potrzeby; zapisany projekt `connect_all()` nigdy tego nie robi, więc na komputerze bez monitora uruchom backend jako [usługę](#always-on-ptp-for-headless-hosts). Zawsze używaj dosłownego adresu IPv4 zamiast `localhost` — `localhost` może rozdzielać się na `::1` i kosztować około dwóch sekund na żądanie.
{% endhint %}

***

## Konfiguracja początkowa

### 1. Zaloguj się

Dostęp do CLI i SDK wymaga płatnego planu Chloros+ (**Copper** lub wyższego), co jest egzekwowane po stronie serwera: wylogowany użytkownik otrzymuje adres `401 AUTH_REQUIRED`, a użytkownik korzystający z bezpłatnego planu (Iron) otrzymuje adres `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Dane uwierzytelniające są buforowane w `~/.chloros/user_session.json`.

{% hint style="info" %}
**Aktualizacja zachowuje dane logowania; usunięcie powoduje ich wyczyszczenie.** Zainstalowanie nowszej wersji `.deb` na istniejącej instalacji (w tym poprzez `chloros-cli update`) zachowuje `~/.chloros/user_session.json` oraz buforowaną licencję, dzięki czemu zadania CLI i SDK bez interfejsu graficznego nadal działają. Usunięcie pakietu (`sudo apt remove chloros`) powoduje wyczyszczenie zapisanych danych logowania i pamięci podręcznej licencji dla każdego użytkownika na komputerze, więc nowa instalacja wymaga ponownego zainstalowania `chloros-cli login`.
{% endhint %}

### 2. Sprawdź stan swojej licencji

```bash
chloros-cli status
```

`chloros-cli status` działa na każdym poziomie (w tym bezpłatnym), dzięki czemu zawsze możesz sprawdzić, dlaczego dostęp jest lub nie jest dostępny.

### 3. Uruchom diagnostykę systemu

```bash
chloros-cli selftest
```

Siedem testów jest wykonywanych po kolei, a polecenie zwraca wynik niezerowy, jeśli którykolwiek z nich zakończy się niepowodzeniem:

| # | Test | Co sprawdza |
| --- | --- | --- |
| 1 | **Wersja** | Interfejs CLI podaje swoją wersję (`v1.3.0`). |
| 2 | **Port dostępny** | Port 5000 jest wolny *lub* otrzymał już odpowiedź od sprawnego backendu Chlorosa (co liczy się jako wynik pozytywny). |
| 3 | **Uruchomienie backendu** | Plik binarny backendu uruchamia się. |
| 4 | **Test API (`/api/test`)** | Serwer odpowiada `status: ok`. |
| 5 | **Informacje o systemie** | Wyświetla `GPU: <name>, CUDA: <bool>, PyTorch: <version>` z `/api/system-info`. |
| 6 | **Modele odszumiania** | Znajduje modele `*.pth.enc` (w systemie Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + filtr odszumiający**| Funkcja Texture Aware jest faktycznie użyteczna — wymaga CUDA**oraz** co najmniej jednego pliku modelu. |

Przebieg kończy się na `N/7 checks passed`, wyświetlając listę wszystkich niepowodzeń wraz z nazwami.

### 4. Przetwarzanie pierwszego zbioru danych

```bash
chloros-cli process ~/datasets/flight001
```

***

## Pliki i katalogi

### Dane użytkownika

Chloros przechowuje swoje dane uwierzytelniające oraz konfigurację CLI w jednym katalogu działającym na różnych platformach, **`~/.chloros/`** (w systemie Windows — `%USERPROFILE%\.chloros\`). Dwie pamięci podręczne specyficzne dla systemu Linux są natomiast zgodne z konwencjami XDG — po ustawieniu stosują się one do katalogów `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`.

| Ścieżka | Przeznaczenie |
| --- | --- |
| `~/.chloros/user_session.json` | Pamięć podręczna sesji logowania zapisywana przez `chloros-cli login` (zachowywana po aktualizacjach pakietu; czyszczona po usunięciu pakietu) |
| `~/.chloros/working_directory.txt` | Domyślne nadpisanie folderu projektu (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferencje językowe interfejsu CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Ustawienie języka współdzielone z interfejsem graficznym systemu Windows — wartość `language` ma tutaj pierwszeństwo przed wartością `cli_language.json` |
| `~/.chloros/update_cache.json` | Godzinna pamięć podręczna dla sprawdzania aktualizacji podczas uruchamiania systemu Linux/Jetson |
| `~/.chloros/backend.log` | Log zaplecza w momencie uruchomienia zaplecza przez CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Buforowane pakiety kalibracyjne LATTICE dla poszczególnych kamer, indeksowane według numeru seryjnego i skrótu pakietu |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Opcjonalne nadpisania użytkownika dla profili korekcji ograniczeń DAQ |
| `~/.config/chloros/system_config.json` | Profil sprzętowy z Dynamic Compute Adaptation zapisany w pamięci podręcznej — usunięcie go wymusi ponowne wykrycie sprzętu |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Logi serwera backendowego, jeden plik na każde uruchomienie |
| `~/Documents/MAPIR/Chloros Projects/` | Domyślny folder projektu, gdy nie ustawiono nadpisania |

### Ogólnosystemowe

| Ścieżka | Przeznaczenie |
| --- | --- |
| `/usr/bin/chloros-cli` | Skrypt opakowujący — ustawia `LD_LIBRARY_PATH` dla dołączonych bibliotek natywnych (oraz `TMPDIR` na `/mnt/ssd/tmp`, jeśli ten folder istnieje), a następnie uruchamia rzeczywisty plik binarny |
| `/usr/bin/chloros-backend` | Skrypt opakowujący — to samo dla backendu, który oznacza jako zainstalowaną kompilację produkcyjną |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Skompilowane CLI i backend, zainstalowane w postaci rozpakowanej (nowość w wersji 1.3.0); programy to `cli/chloros-cli` i `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Dwuwierszowe skrypty uruchamiające, które uruchamiają powyższe rzeczywiste programy — przechowywane w tych ścieżkach, ponieważ SDK, CLI i starsze skrypty szukają ich właśnie tutaj |
| `/usr/lib/chloros/arena_runtime/` | Środowisko uruchomieniowe Arena SDK wymagane przez kamery LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Zaszyfrowane modele odszumiania używane przez moduł Texture Aware debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (oraz `lattice_sdk`, `lattice_cam`, `arena_api` — moduły ładujące projekt) | Zestaw SDK SDK, zainstalowany dla systemu `python3` i należący do pakietu |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Plik wheel zestawu Python SDK odpowiadający dokładnie tej kompilacji, przeznaczony dla środowiska virtualenv |
| `/usr/lib/chloros/exiftool` | Dołączony program uruchamiający ExifTool (z dowiązaniem symbolicznym do `/usr/local/bin/exiftool` tylko wtedy, gdy nie istnieje systemowy exiftool). Od wersji 1.3.0 skrypt ExifTool jest dostarczany z własną biblioteką Perla w `/usr/lib/chloros/exiftool-dist/`, więc działa bez systemowego pakietu `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfiguracja kanału aktualizacji odczytywana przez `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Ustawia `net.ipv4.ip_unprivileged_port_start = 319` tak, aby backend mógł przywiązać porty PTP bez uprawnień roota |
| `/etc/sysctl.d/60-chloros-gige.conf` | Zwiększa bufory odbiorcze gniazd jądra (`net.core.rmem_max`, `net.core.rmem_default`) do 32 MB dla strumieni z kamer LATTICE i stosuje je podczas instalacji (jeśli wartość jest już wyższa, pozostaje bez zmian). Przy domyślnych ustawieniach jądra pakiety z kamer są odrzucane w jądrze, a macierz traci klatki i częstotliwość odświeżania; `chloros-cli lattice network` zgłasza niski poziom buforów. Zobacz [Hosty Linux — bufory odbiorcze gniazd](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Kieruje dynamiczny moduł ładujący do `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Przyznaje zalogowanemu użytkownikowi dostęp do mostka szeregowego USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Usługa backendowa typu „always-on” z opcją włączenia, która uruchamia backend jako użytkownik o podanej nazwie (zainstalowana, **nie włączona**) — tę należy włączyć, zobacz [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Ta sama usługa uruchamiająca backend jako **root**(zainstalowana,**nieaktywna**), zachowana po to, by komputer, na którym została włączona, nadal działał |
| `/var/log/chloros/` | Szczegółowy dziennik usługi root, dostępny dla grupy `adm` (usuwany po usunięciu pakietu) |
| `/usr/share/applications/chloros-cli.desktop` | Pozycja w menu aplikacji „Chloros CLI”, która otwiera terminal |

**Zapora sieciowa (nowość w wersji 1.3.0).** Jeśli w momencie instalacji aktywne są usługi `ufw` lub `firewalld`, pakiet otwiera porty UDP `5002` (dane DAQ-E), `5353` (wykrywanie mDNS), `319` oraz `320` (PTP), a po usunięciu pakietu usuwa te reguły. Jeśli zapora sieciowa jest zainstalowana, ale nieaktywna, pakiet nie wprowadza żadnych zmian i wyświetla listę portów, które należy zezwolić w przypadku jej późniejszego włączenia. Nie otwiera żadnych portów TCP: aktualizacja oprogramowania sprzętowego DAQ-E, podczas której czujnik łączy się ponownie z tym komputerem przez protokół TCP w celu pobrania obrazu, kończy się niepowodzeniem z kodem błędu `… device never connected back on TCP <port> …`, jeśli aktywna zapora sieciowa blokuje to połączenie.

## Lokalizacja pliku wykonywalnego modułu zaplecza

CLI i SDK automatycznie wykrywają moduł zaplecza:

| Komponent | Ścieżka |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (program uruchamiający dla `/usr/lib/chloros/backend/chloros-backend`) |

Ścieżkę backendu można nadpisać za pomocą flagi CLI’a `--backend-exe` lub parametru konstruktora SDK’a `backend_exe`, a port za pomocą `--port` (domyślnie `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` kieruje rodziny poleceń **`lattice`**,**`project`**oraz**`daq pool-*`** na zdalnym backendzie. Polecenia podstawowe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) celowo ją ignorują i zawsze kierują się do `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamery LATTICE i czujniki światła DAQ w systemie Linux

Wszystkie rodziny poleceń live-hardware działają w systemie Linux (amd64 i Jetson):

* **`chloros-cli lattice`** — wykrywanie, łączenie, konfigurowanie i przechwytywanie obrazu z kamer LATTICE oraz zsynchronizowanych macierzy. Polecenie `.deb` dołącza wymagane środowisko uruchomieniowe Arena SDK i rejestruje je w dynamicznym modułach ładującym.
* **`chloros-cli daq pool-*`** — podłącz czujniki światła DAQ-U/M/E poprzez pulę backendową, przesyłaj strumieniowo skalibrowane widma, rejestruj pliki `.daq` i wymieniaj profile czapek, a w przypadku DAQ-E zmieniaj jego ustawienia sieciowe, uruchamiaj go ponownie i aktualizuj oprogramowanie układowe. `chloros-cli daq discover` skanuje w poszukiwaniu czujników, które można podłączyć, a `chloros-cli daq --help` wyświetla listę podpoleceń `pool-*`.
* **`chloros-cli project`** — uruchamia zapisany projekt (obejmujący kamery, czujniki i ustawienia przetwarzania) w trybie bezinterfejsowym.
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

`--sensor-id` jest wymagane przez `pool-latest`, `pool-stream`, `pool-record` oraz `pool-set-cap`; `pool-list` wyświetla identyfikatory znajdujące się obecnie w puli.

{% hint style="info" %}
**`--eth-host` pomija fazę wykrywania.** `pool-connect --eth` wykrywa urządzenie DAQ-E za pomocą mDNS, z rezerwowym mechanizmem ARP działającym na podstawie pustej pamięci podręcznej ARP; przekazanie adresu IP lub nazwy hosta czujnika za pomocą `--eth-host` powoduje bezpośrednie połączenie z nim.
{% endhint %}

**Uprawnienia szeregowe DAQ-U** są obsługiwane przez zainstalowaną regułę udev (`uaccess` + grupa `dialout`). Jeśli czujnik, który był już podłączony, pozostaje niedostępny, należy ponownie załadować reguły lub podłączyć go ponownie:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Pełny zestaw poleceń znajduje się w [podręczniku CLI](../CLI.md).

### PTP zawsze włączony dla hostów bez monitorów

Pakiet instaluje dwie usługi systemd dla zaplecza, z których żadna nie jest domyślnie włączona. Na komputerze Jetson bez monitora lub serwerze, który powinien utrzymywać ciągłą synchronizację czasu PTP dla czujników DAQ-E i kamer LATTICE, lub w przypadku gdy skrypty SDK łączą zapisane projekty (`connect_all()` nigdy nie uruchamia zaplecza), włącz usługę **jako swój użytkownik**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` uruchamia backend jako ten użytkownik: zapisywane przez niego dane pochodzą od niego, jego szczegółowy dziennik znajduje się w pliku `~/.cache/chloros/logs`, a korzysta on z konta `chloros-cli login` (`~/.chloros`), więc najpierw zaloguj się raz jako ten użytkownik. Instalacja pakietu wyświetla to polecenie z wpisanym Twoim imieniem użytkownika.

Bez usługi backend (a wraz z nim grandmaster PTP) działa tylko wtedy, gdy coś go uruchomiło — polecenie `chloros-cli` lub skrypt SDK.

Moduł wiąże backend z ustawieniami środowiskowymi `127.0.0.1:5000` (ustawienia środowiskowe `CHLOROS_HOST` / `CHLOROS_PORT` wewnątrz modułu; można je nadpisać za pomocą `sudo systemctl edit chloros-backend@.service`) i uruchamia go ponownie po 5 sekundach w przypadku awarii.

**Usługa root.**`chloros-backend.service` uruchamia backend jako**root**. Jest ona nadal zainstalowana, dzięki czemu maszyna, na której została włączona, nadal działa, ale wszystko, co zapisuje backend jako root, należy do roota, więc użytkownik nie może usunąć własnych przechwyceń; jego dziennik znajduje się w pliku `/var/log/chloros`, do którego dostęp ma grupa `adm`. Aby przełączyć się na swojego użytkownika:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend usługi jest współdzielony i nigdy nie jest zatrzymywany przez klienta.** Skrypty `chloros-cli` i SDK korzystają z niego tak, jak go zastaną: `process` uruchamia się na nim, ładując swój własny projekt w miejsce ostatniego. W sytuacji, gdy interfejs CLI, w innym przypadku, zrestartowałby backend (`--restart` lub `process`, którego import nie znalazł niczego), podaje nazwę jednostki, wyświetla komunikat `sudo systemctl restart <unit>` i kończy działanie `1`, ponieważ backend zatrzymany w ten sposób pozostałby wyłączony. Należy go ponownie uruchomić samodzielnie za pomocą polecenia `sudo systemctl restart chloros-backend@$USER.service`. Backend usługi nigdy nie zatrzymuje się samoczynnie w stanie bezczynności, a `chloros-cli backend stop` odrzuca to, podając nazwę polecenia `systemctl stop`.**W jaki sposób PTP uzyskuje swoje porty.** PTP wykorzystuje porty UDP 319/320, oba poniżej standardowego progu 1024 dla portów uprzywilejowanych. Plik `postinst` w pakiecie zapisuje `/etc/sysctl.d/60-chloros-ptp.conf` wraz z `net.ipv4.ip_unprivileged_port_start = 319`, co pozwala backendowi powiązać je podczas działania pod uprawnieniami użytkownika. Od wersji 1.3.0 pakiet nie ustawia żadnych uprawnień plików (`setcap`) w backendzie, a `libcap2-bin` nie jest już zależnością: w rozpakowanym backendzie uprawnienie pliku uniemożliwiłoby jego uruchomienie, więc nie dodawaj jej samodzielnie.***

## Przykłady skryptów w języku Bash

{% hint style="info" %}
**Kody wyjścia przyjazne dla skryptów.**`chloros-cli process` kończy działanie z kodem `0` w przypadku powodzenia oraz**kodem niezerowym w przypadku niepowodzenia — w tym w przypadku uruchomienia, które zażądało produktów obrazowych, ale nie zapisało żadnego** (wyświetla `Processing finished but wrote no image products.` oraz podaje nazwę folderu projektu i typowe przyczyny). Pomyślne uruchomienia informują, ile produktów obrazowych zostało zapisanych (`Image products written: N`, a także `(plus M clamp masks)`, gdy zapisano maski). Kody wyjścia: `0` — sukces, `1` — niepowodzenie, `2` — błąd argumentu, `130` — przerwanie.
{% endhint %}

### Przetwarzanie wielu zbiorów danych

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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Prawidłowych wartości `--format` są dokładnie cztery i zawierają spacje — należy je zawsze umieszczać w cudzysłowie:

| Wartość `--format` | Folder wyjściowy |
| --- | --- |
| `TIFF (16-bit)` *(domyślnie)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` akceptuje `standard` (domyślnie) lub `texture-aware` (Chloros+).

### Zautomatyzowane przetwarzanie za pomocą Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Przykład SDK’a dla Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Rozwiązywanie problemów

### Nie znaleziono interfejsu CLI po instalacji

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend nie uruchamia się po dodaniu uprawnień do plików

Nie uruchamiaj `setcap` na backendzie. Od wersji 1.3.0 pakiet nie ustawia żadnych uprawnień plików, a porty PTP 319/320 wynikają z ustawienia `sysctl` opisanego w sekcji [Jak PTP uzyskuje swoje porty](#always-on-ptp-for-headless-hosts). Uprawnienie pliku powoduje, że rozpakowany backend przechodzi w tryb bezpiecznego wykonywania, w którym nie może odnaleźć własnych bibliotek, przez co nie uruchamia się. Jeśli dodałeś takie uprawnienie, usuń je:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP nie uruchamia się / nie może powiązać portu 319

Sprawdź, czy dolna granica portów bez uprawnień została obniżona, a jeśli nie, zastosuj ją ponownie dla bieżącego uruchomienia:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Następnie sprawdź serwer główny:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Polecenia `lattice` kończą się niepowodzeniem z komunikatem „Nie znaleziono sterowników kamer LATTICE”

Backend nie mógł załadować środowiska uruchomieniowego kamery LATTICE, z którym został zainstalowany. Każde polecenie `lattice` jest wykonywane na backendzie; sam interfejs CLI nie zawiera żadnych sterowników kamer. Zainstaluj ponownie Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): polecenie `.deb` instaluje środowisko uruchomieniowe obok modułu zaplecza, w katalogu `/usr/lib/chloros/arena_runtime`, i rejestruje je w dynamicznym modułu ładującym. Aby sprawdzić tę rejestrację:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Nie udało się uruchomić backendu

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Dzienniki backendu dotyczące nieudanego uruchomienia znajdują się w pliku `~/.cache/chloros/logs/`. Jeśli port 5000 należy do usługi backendu (`systemctl status chloros-backend@$USER.service`), należy ponownie uruchomić usługę zamiast zabijać jej proces.

### Nie wykryto CUDA

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

Plik `chloros-cli selftest` zawiera tę samą informację w jednym wierszu: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Brakujące biblioteki współdzielone

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Powolne uruchamianie na systemach z kartą SD

Od wersji 1.3.0 backend i interfejs CLI są instalowane w postaci rozpakowanej, więc nie są już rozpakowywane do katalogu tymczasowego przy każdym uruchomieniu. Pliki tymczasowe są automatycznie zapisywane w katalogu `/mnt/ssd/tmp`, o ile katalog ten istnieje; w przeciwnym razie można wskazać im szybki system plików:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualizacja Chloros na Linux

Polecenie `update` jest dostępne wyłącznie w systemie Linux/Jetson. Sprawdza ono wersję opublikowaną w kanale aktualizacji skonfigurowanym w `/etc/chloros/update.conf` i proponuje pobranie oraz zainstalowanie pasującej wersji `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

W systemie Linux/Jetson interfejs CLI przeprowadza również nieblokujące sprawdzanie dostępności aktualizacji przy każdym uruchomieniu (wynik jest buforowany przez godzinę w `~/.chloros/update_cache.json`) i wyświetla komunikat `Update available: vX.Y.Z`, gdy dostępna jest nowsza wersja. Twoje ustawienia, projekty i dane logowania pozostają nienaruszone po aktualizacji.

Aktualizacja pakietu zatrzymuje wszystkie usługi zaplecza na czas instalacji, więc uruchomiony skrypt traci dostęp do kamer i czujników, a następnie ponownie uruchamia każdą z nich, która jest włączona (skrypt uruchomiony ręcznie, ale nigdy nie włączony, pozostaje zatrzymany, tak jak po ponownym uruchomieniu systemu).

## Odinstalowywanie

```bash
sudo apt remove chloros
```

Odinstalowanie zatrzymuje i wyłącza wszystkie usługi zaplecza, przywraca domyślną dolną granicę portów bez uprawnień (1024), usuwa plik ustawień bufora odbiorczego (bieżące wartości pozostają do momentu ponownego uruchomienia systemu), dowiązanie symboliczne dołączonego narzędzia exiftool, konfigurację programu ładującego Arena, reguły zapory dodane przez pakiet oraz zainstalowany przez niego zestaw PythonSDK, a także czyści buforowane poświadczenia. Twoje projekty i pliki danych `~/.chloros/` pozostają nienaruszone. `sudo apt purge chloros` usuwa również `/var/log/chloros`.

***

## Kolejne kroki

* [Przewodnik po NVIDIA Jetson](nvidia-jetson-guide.md) — Optymalizacja i wdrażanie specyficzne dla Jetson
* [CLI: wiersz poleceń](../CLI.md) — przewodnik po CLI
* [API: Python SDK](../api-python-sdk.md) — przewodnik po SDK
* [Podręcznik CLI](../reference/cli-reference.md) i [Podręcznik SDK](../reference/sdk-reference.md) — wyczerpujące wykazy poleceń/API dla wersji 1.3.0
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — jak Chlorosdostosowuje się do posiadanego sprzętu
