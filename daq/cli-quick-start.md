# Szybki start CLI (pool-*)

Dostarczone sterowniki `chloros-cli` sterują czujnikami DAQ za pomocą rodziny poleceń **`daq pool-*`** — prostych klientów HTTP, które obsługują czujniki poprzez stałą pulę czujników backendu Chloros. Backend zarządza transportem, więc interfejs graficzny (GUI), interfejs CLI oraz skrypty SDK współdzielą jeden aktywny uchwyt zamiast rywalizować o port. Wszystko, czego potrzebuje klient, jest dostępne za pośrednictwem `pool-*`: nawiązywanie połączenia, przesyłanie strumieniowe, rejestrowanie skalibrowanych plików `.daq` oraz zamiana profili limitów.

`pool-*` to obsługiwany interfejs DAQ w wydanych kompilacjach. `chloros-cli daq --help` wyświetla listę podpoleceń `pool-*`, a `daq discover` / `daq list` uruchamiają to samo skanowanie co `pool-discover`. W przypadku braku dostępności podpolecenia DAQ sterującego bezpośrednio sprzętem program kończy działanie, wyświetlając wyraźny komunikat o błędzie wskazujący brakujący pakiet i odsyłający użytkownika do `pool-*` — żaden błąd nie przebiega w trybie cichym. (`pip install chloros-sdk` również nie udostępnia poleceń bezpośredniego dostępu do sprzętu.)

***

## Wymagania wstępne

* **Backend Chloros musi być uruchomiony** — polecenia `pool-*` są klientami HTTP, a nie sterownikami sprzętowymi. W systemie Windows uruchom aplikację desktopową Chloros (uruchamia ona backend). W systemie Linux/Jetson bez interfejsu graficznego włącz usługę jako użytkownik: `sudo systemctl enable --now chloros-backend@$USER.service` (zobacz [Instalacja w systemie Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Logowanie do Chloros+ (plan płatny)**: najpierw uruchom `chloros-cli login`. Wymuszenie odbywa się po stronie serwera — bez logowania polecenia kończą się niepowodzeniem z kodem `401 AUTH_REQUIRED`; w bezpłatnym planie (Iron) kończą się niepowodzeniem z kodem `403 PLAN_UPGRADE_REQUIRED`.
* Polecenia domyślnie są skierowane do `http://127.0.0.1:5000`; rodzina `daq pool-*` uwzględnia zmienną środowiskową `CHLOROS_BACKEND_URL`, jeśli twój backend działa gdzie indziej.

***

## Pięciominutowa sesja

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — otwórz czujnik w puli

| Wariant | Znaczenie |
| --- | --- |
| `daq pool-connect` | Inteligentne wykrywanie: znajdź dowolny moduł DAQ na tym komputerze. |
| `daq pool-connect --port PORT` | Urządzenie DAQ-U na określonym porcie szeregowym (np. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | Urządzenie DAQ-M przez BLE, automatyczne skanowanie adresów MAC. |
| `daq pool-connect --mac MAC` | DAQ-M o znanym adresie MAC BLE (oznacza to `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E przy znanej nazwie hosta lub adresie IP — **ścieżka niezawodna**. |
| `daq pool-connect --eth` | DAQ-E z automatycznym wykrywaniem (mDNS, z rezerwą ARP). Zobacz zastrzeżenie poniżej. |

Flagi optymalizacyjne, wszystkie opcjonalne:

| Flaga | Znaczenie |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ręczny czas integracji w milisekundach. |
| `--frame-avg N` / `-f N` | Średnia liczba klatek na zgłoszone widmo. |
| `--no-ae` | Wyłącz automatyczną ekspozycję (AE jest domyślnie włączone). |
| `--no-stream` | Połącz się bez uruchamiania strumienia (wznowienie później za pomocą `pool-stream --start`). |
| `--cap-id CAP` | Profil korekcji Cap; domyślnym ustawieniem backendu jest `sunshine_cosine`. Zobacz [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Podłączenie urządzenia DAQ-E, które znajduje się już w puli pod innym adresem — nazwą hosta `daq-e-<id>.local` lub adresem IP — powoduje zwrócenie istniejącego wpisu w puli zamiast ponownego otwarcia urządzenia.

{% hint style="warning" %}
**Zastrzeżenie dotyczące automatycznego wykrywania `--eth`.** Na hoście z wieloma interfejsami sieciowymi (więcej niż jeden aktywny interfejs sieciowy) *pierwsze* `pool-connect --eth` po uruchomieniu systemu może zakończyć się bez wyników, nawet jeśli czujnik działa prawidłowo — proces wykrywania może pominąć interfejs czujnika, gdy pamięć podręczna ARP jest pusta. Jeśli polecenie `--eth` nie znajdzie niczego, należy spróbować ponownie lub całkowicie pominąć wykrywanie, używając polecenia `--eth-host <ip-or-hostname>`, co jest niezawodnym rozwiązaniem na komputerach z wieloma interfejsami sieciowymi. Nazwa hosta urządzenia DAQ-E to `daq-e-<id>.local` (np. `daq-e-def330.local`); działa również jego zwykły adres IP.
{% endhint %}

## `pool-discover` — skanowanie bez nawiązywania połączenia

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Wyszukuje czujniki, z którymi można się połączyć, i wyświetla adres, z którym należy się połączyć, bez nawiązywania połączenia. `--only` przyjmuje podzbiór wartości `usb,ble,eth` rozdzielonych przecinkami; `--timeout SEC` ustawia czas skanowania dla każdego protokołu (domyślnie 5 s, ograniczony do 1–20 s). Czujniki już otwarte w puli nie są wyświetlane (podłączony moduł DAQ-M przestaje nadawać sygnał) — w tym przypadku należy użyć `pool-list`. W dostarczonej kompilacji polecenia `chloros-cli daq discover` i `daq list` wykonują to samo skanowanie.

## `pool-list` — sprawdź, co jest podłączone

Wyświetla wszystkie czujniki w puli backendowej, w tym `sensor_id`, którego potrzebują wszystkie pozostałe polecenia:

| Model | Kształt `sensor_id` | Przykład |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktetowy z łącznikami | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Urządzenie DAQ-E, w którym wyłączono PTP (ustawienie jest zapisane w urządzeniu), jest oznaczone w `pool-list` za pomocą `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` oraz polecenia, które ponownie je włącza — zobacz [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — odczyt ramek widma

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Zwraca najnowszą ramkę lub najnowsze ramki `--recent N`; `--json` generuje dane wyjściowe w formacie nadającym się do odczytu maszynowego na potrzeby skryptów. Ramki przedstawiają kalibrowane radiometrycznie natężenie promieniowania spektralnego (W/m²/nm) w siatce 135 punktów w zakresie 340–1010 nm, z już zastosowanym profilem osłony czujnika. Aby uzyskać ilościowe wartości natężenia promieniowania, należy uśrednić co najmniej 15 sekund klatek — jest to cecha charakterystyczna urządzenia, a nie wada.

## `pool-stream` — wstrzymanie lub wznowienie transmisji strumieniowej

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — zapis pliku `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flaga | Domyślna | Znaczenie |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Długość nagrania w sekundach; `0` oznacza, że nagrywanie trwa do momentu wydania polecenia `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Katalog wyjściowy. W przypadku backendu na tym komputerze ścieżka względna jest rozpoznawana względem folderu, w którym uruchomiono polecenie; w przypadku backendu zdalnego ścieżka jest rozpoznawana **na komputerze, na którym działa backend**. |
| `--device-name NAME` | — | Etykieta zapisywana wraz z nagraniem. |
| `--stop` | — | Zatrzymanie trwającego nagrania. |
| `--confirm-after SEC` | `20` | Po uruchomieniu należy odczekać do tego czasu na zapisanie pierwszego wiersza i wyświetlić wyraźne ostrzeżenie (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), jeśli żaden nie nadejdzie. `0` wyłącza tę kontrolę. |
| `--heartbeat SEC` | `30` | Przy ustawieniu `--duration` wyświetlaj liczbę wierszy co SEC sekund i wyświetl ostrzeżenie, jeśli liczba ta przestanie rosnąć. `0` wyłącza tę funkcję. |

{% hint style="info" %}
Rejestracja odbywa się w backendzie, więc plik `.daq` trafia do systemu plików **maszyny backendowej** — domyślnie w `~/Documents/DAQ Live View/` tam, niekoniecznie tam, gdzie uruchomiono CLI. Nazwy plików zawierają identyfikator czujnika oraz sygnaturę czasową.
{% endhint %}

## `pool-set-cap` — zadeklaruj zamontowaną nasadkę

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Identyfikator nasadki wybiera zmierzony fabrycznie profil korekcyjny stosowany do każdego widma i **musi on odpowiadać nasadce fizycznie zamontowanej na czujniku** — ani czujnik, ani oprogramowanie nie są w stanie samodzielnie wykryć nasadki, a wybór ten jest zapisywany w każdym pliku `.daq`. Wszędzie domyślnym ustawieniem jest `sunshine_cosine` (każdy zestaw DAQ jest dostarczany z zainstalowaną nasadką korektora cosinusowego Sunshine, o tłumieniu ~12× zgodnie z projektem — niezgłoszona zmiana nasadki powoduje błędną korekcję widm mniej więcej o ten współczynnik).

| `--cap-id` | Dostępne w |
| --- | --- |
| `sunshine_cosine` (domyślnie) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Tylko DAQ-U |
| `none` | Tylko DAQ-E — patrz uwaga |

Identyfikator nasadki spoza zestawu czujnika jest odrzucany podczas łączenia z wyraźnym komunikatem o błędzie. `none` (DAQ-E) oznacza, że nasadka została fizycznie zdjęta — nadal stosowany jest fabryczny profil geometryczny dla wpuszczanego szklanego dyfuzora DAQ-E, więc nie jest to operacja bezczynna, a „nagi” DAQ-E stanowi konfigurację laboratoryjną, a nie obsługiwany w terenie. (Odsłonięty czujnik DAQ-U jest całkowicie odsłonięty i nie wymaga żadnego profilu korekcyjnego; czujnik DAQ-M jest używany wraz z nasadką Sunshine.)

## `pool-disconnect` — zwolnienie czujników

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` zwalnia czujnik niezależnie od tego, kto go otworzył, więc zwalnia również czujnik pozostawiony w puli przez skrypt, który uległ awarii (najpierw kończy się na nim rejestracja). Czujnik, który nie znajduje się w puli, nie ma czego zwolnić: polecenie wyświetla komunikat `<sensor-id>: already released (not in the pool)` i kończy działanie `0`. Zwolnienie, którego backend odmówił lub nie potwierdził, kończy działanie `1`.

## `pool-enable-ptp` — ponowne włączenie PTP w urządzeniu DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Urządzenie DAQ-E przechowuje ustawienie włączenia/wyłączenia PTP we własnej pamięci, więc urządzenie, którego PTP zostało wyłączone, pozostaje wyłączone — a jego sygnatury czasowe nie są synchronizowane z PTP — dopóki nie zostanie ponownie włączone. `pool-list` oznacza takie urządzenie i wyświetla dla niego to polecenie. Program Chloros nigdy samoczynnie nie włącza urządzenia ponownie, ponieważ wyłączenie może być zamierzone. Nowość w wersji 1.3.0.

## Ustawienia urządzenia DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` wyświetla oprogramowanie układowe czujnika (oraz informację, czy ta wersja Chlorosa zawiera nowszą wersję), jego adres i sposób jego uzyskania, zapisane ustawienia sieciowe, o ile różnią się one od aktualnie używanych, strumieniowanie po włączeniu zasilania oraz PTP. `pool-set-network` restartuje czujnik w celu zastosowania zmiany i ponownie łączy go z poprzednimi ustawieniami; `--no-reboot` jedynie zapisuje te ustawienia. Adres statyczny musi znajdować się w sieci tego komputera, w przeciwnym razie Chloros go odrzuci; `--force` pomija to ograniczenie, ale tylko w przypadku komputera, który zostanie ponownie zaadresowany lub przekierowany na ten adres. Ponowne uruchomienie jest odrzucane, gdy czujnik prowadzi rejestrację. `pool-update-firmware` jest odpowiednikiem opcji **Update to \<version\>** w interfejsie graficznym dla wiersza poleceń oraz sposobem na aktualizację urządzenia DAQ-E w systemie Linux, który nie posiada interfejsu graficznego. Nowość w wersji 1.3.0.

***

## Podsumowanie poleceń

| Polecenie | Przeznaczenie |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otwórz czujnik w puli backendowej. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Wyszukiwanie czujników bez nawiązywania połączenia (wykrywa adres MAC BLE urządzenia DAQ-M). |
| `daq pool-list` | Wyświetla wszystkie czujniki w puli wraz z ich identyfikatorami `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N najnowszych skalibrowanych ramek widma. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Wznowienie / wstrzymanie transmisji strumieniowej. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Rozpoczęcie / zatrzymanie nagrywania `.daq` (po stronie serwera). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmiana profilu korekcji limitu w trakcie działania. |
| `daq pool-disconnect --sensor-id ID [--all]` | Zwolnienie jednego czujnika lub wszystkich. |
| `daq pool-enable-ptp --sensor-id ID` | Ponowne włączenie PTP dla urządzenia DAQ-E, którego PTP jest wyłączone. |
| `daq pool-info --sensor-id ID [--json]` | Oprogramowanie sprzętowe urządzenia DAQ-E, adres, zapisane ustawienia sieciowe, strumieniowanie po włączeniu zasilania oraz PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP lub adres statyczny; powoduje ponowne uruchomienie i ponowne podłączenie czujnika. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Transmisja po włączeniu zasilania. |
| `daq pool-reboot --sensor-id ID` | Restart urządzenia DAQ-E i ponowne nawiązanie połączenia. |
| `daq pool-update-firmware --sensor-id ID` | Wgranie oprogramowania sprzętowego DAQ-E z tej wersji do starszego czujnika. |

***

## Rozwiązywanie problemów z pierwszym połączeniem DAQ-E

1. DAQ-E nie posiada diody LED stanu — sprawdź zasilanie za pomocą wskaźnika PoE/link na przełączniku lub porcie iniektora i odczekaj kilka sekund po włączeniu zasilania, aż urządzenie uruchomi się i dołączy do sieci.
2. Komputer serwerowy musi znajdować się w **tej samej domenie rozgłoszeniowej** co czujnik — protokół mDNS nie przechodzi przez routery.
3. Zapora sieciowa musi zezwalać na ruch mDNS (UDP 5353), dane DAQ-E (UDP 5002) oraz PTP (UDP 319/320). Instalator systemu Windows dodaje odpowiednią regułę, a w systemie Linux `.deb` otwiera je w `ufw` / `firewalld`, który był aktywny w momencie instalacji; zaporę sieciową innej firmy lub zaporę włączoną później należy skonfigurować tak, aby zezwalała na te połączenia.
4. Nadal brak odpowiedzi z `--eth`? Użyj adresu `--eth-host` wraz z nazwą hosta urządzenia (`daq-e-<id>.local`) lub adresem IP, co pominie proces wykrywania.

***
{% hint style="info" %}
**Wskazówka dla asystentów AI.** Każda strona tej instrukcji jest udostępniana w formacie surowego Markdown — dodaj `.md` do małej litery slugu URLu strony (ta strona: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); indeks czytelny dla maszyn to `https://mapir.gitbook.io/chloros/llms.txt`. Aby uzyskać pełną dokumentację na poziomie flag dla `chloros-cli daq` i każdej innej rodziny poleceń, pobierz [Podręcznik CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); ścieżka w języku Pythona to `chloros_sdk.connect_daq_sensor()` w [Podręczniku SDK](../reference/sdk-reference.md).
{% endhint %}
