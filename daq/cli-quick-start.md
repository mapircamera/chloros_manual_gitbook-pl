# CLI — Szybki start (pool-*)

Dostarczone napędy `chloros-cli` sterują czujnikami DAQ za pomocą rodziny poleceń **`daq pool-*`** — są to „cienkie” klienci HTTP, którzy obsługują czujniki za pośrednictwem trwałej puli czujników backendu Chloros. Backend zarządza transportem, więc interfejs graficzny, skrypty CLI i SDK współdzielą jeden aktywny uchwyt zamiast rywalizować o port. Wszystko, czego potrzebuje klient, jest dostępne za pośrednictwem `pool-*`: nawiązywanie połączenia, przesyłanie strumieniowe, rejestrowanie skalibrowanych plików `.daq` oraz wymiana profili czapek.

`pool-*` jest również **jedyną** powierzchnią DAQ w wydanych kompilacjach. `chloros-cli daq --help` wyświetla listę podpoleceń `pool-*`, a wywołanie podpolecenia DAQ sterującego bezpośrednio sprzętem w wydanej kompilacji kończy się wyraźnym błędem wskazującym brakujący pakiet i odsyłającym do `pool-*` — nic nie kończy się bez komunikatu o błędzie. (Polecenia bezpośredniego dostępu do sprzętu działają wyłącznie po pobraniu kodu źródłowego z gałęzi MAPIR; polecenie `pip install chloros-sdk` również ich nie udostępnia.)

***

## Wymagania wstępne

* **Backend Chloros musi być uruchomiony** — polecenia `pool-*` są klientami HTTP, a nie sterownikami sprzętowymi. W systemie Windows uruchom aplikację desktopową Chloros (uruchamia ona moduł zaplecza). Na urządzeniu bez monitoru Linux/Jetson włącz usługę: `sudo systemctl enable --now chloros-backend.service`.
* **Logowanie na Chloros+ (poziom płatny)**: najpierw uruchom `chloros-cli login`. Wymuszanie odbywa się po stronie serwera — bez logowania polecenia kończą się niepowodzeniem z kodem `401 AUTH_REQUIRED`; w bezpłatnym planie (Iron) kończą się niepowodzeniem z kodem `403 PLAN_UPGRADE_REQUIRED`.
* Polecenia domyślnie są skierowane do `http://127.0.0.1:5000`; rodzina `daq pool-*` uwzględnia zmienną środowiskową `CHLOROS_BACKEND_URL`, jeśli twój backend działa gdzie indziej.

***

## Pięciominutowa sesja

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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

Flagi dostrajające, wszystkie opcjonalne:

| Flaga | Znaczenie |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ręczny czas integracji w milisekundach. |
| `--frame-avg N` / `-f N` | Średnia liczba klatek na zgłoszone widmo. |
| `--no-ae` | Wyłącz automatyczną ekspozycję (domyślnie włączona). |
| `--no-stream` | Połącz się bez uruchamiania strumienia (wznowienie później za pomocą `pool-stream --start`). |
| `--cap-id CAP` | Profil korekcji Cap; domyślnym ustawieniem backendu jest `sunshine_cosine`. Zobacz [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Zastrzeżenie dotyczące automatycznego wykrywania `--eth`.** Na hoście z wieloma interfejsami sieciowymi (więcej niż jeden aktywny interfejs sieciowy) *pierwsze* wywołanie `pool-connect --eth` po uruchomieniu systemu może zakończyć się bez wyników, nawet jeśli czujnik jest sprawny — proces wykrywania może pominąć interfejs czujnika, gdy pamięć podręczna ARP jest pusta. Jeśli polecenie `--eth` nie znajdzie niczego, należy spróbować ponownie lub całkowicie pominąć wykrywanie, używając polecenia `--eth-host <ip-or-hostname>`, co jest niezawodnym rozwiązaniem na komputerach z wieloma interfejsami sieciowymi. Nazwa hosta urządzenia DAQ-E to `daq-e-<id>.local` (np. `daq-e-def330.local`); działa również jego zwykły adres IP.
{% endhint %}

## `pool-list` — sprawdź, co jest podłączone

Wyświetla wszystkie czujniki w puli zaplecza, w tym `sensor_id`, którego wymagają wszystkie pozostałe polecenia:

| Model | Format `sensor_id` | Przykład |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktetowy z łącznikami | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — odczyt ramek widma

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Zwraca najnowszą ramkę lub najnowsze ramki `--recent N`; `--json` generuje dane wyjściowe w formacie nadającym się do odczytu maszynowego na potrzeby skryptów. Ramki przedstawiają radiometrycznie skalibrowane natężenie promieniowania spektralnego (W/m²/nm) w siatce 135 punktów w zakresie 340–1010 nm, z już zastosowanym profilem osłony czujnika. Aby uzyskać ilościowe wartości natężenia promieniowania, należy uśrednić co najmniej 15 sekund klatek — jest to cecha charakterystyczna urządzenia, a nie wada.

## `pool-stream` — wstrzymanie lub wznowienie przesyłania strumieniowego

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — nagranie pliku `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flaga | Domyślna | Znaczenie |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Czas nagrywania w sekundach; `0` oznacza, że proces będzie trwał do momentu wydania polecenia `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Katalog wyjściowy, ustalany **na komputerze, na którym działa backend**. |
| `--device-name NAME` | — | Etykieta zapisana wraz z nagraniem. |
| `--stop` | — | Zatrzymanie trwającego nagrania. |

{% hint style="info" %}
Nagrywanie odbywa się w backendzie, więc plik `.daq` trafia do systemu plików **komputera z serwerem** — domyślnie do katalogu `~/Documents/DAQ Live View/`, niekoniecznie do miejsca, w którym uruchomiono program CLI. Nazwy plików zawierają identyfikator czujnika oraz sygnaturę czasową.
{% endhint %}

## `pool-set-cap` — zadeklaruj zamontowaną nasadkę

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Identyfikator nasadki wybiera zmierzony fabrycznie profil korekcyjny stosowany do każdego widma i **musi on odpowiadać nasadce fizycznie zamontowanej na czujniku** — ani czujnik, ani oprogramowanie nie są w stanie samodzielnie wykryć nasadki, a wybór ten jest zapisywany w każdym pliku `.daq`. Wszędzie domyślnym ustawieniem jest `sunshine_cosine` (każdy zestaw DAQ jest dostarczany z zainstalowaną nasadką korektora cosinusowego Sunshine, o tłumieniu około 12× zgodnie z projektem — niezgłoszona zmiana nasadki powoduje nieprawidłową korekcję widm mniej więcej o ten współczynnik).

| `--cap-id` | Dostępne w |
| --- | --- |
| `sunshine_cosine` (domyślnie) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Tylko DAQ-U |
| `none` | Tylko DAQ-E — patrz uwaga |

Identyfikator nasadki spoza zestawu czujnika powoduje odrzucenie połączenia z wyraźnym komunikatem o błędzie. `none` (DAQ-E) oznacza, że nasadka została fizycznie zdjęta — nadal stosowany jest fabryczny profil geometryczny dla wpuszczanego szklanego dyfuzora DAQ-E, więc nie jest to operacja bezczynna, a gołe urządzenie DAQ-E stanowi konfigurację laboratoryjną, a nie obsługiwany w terenie. (Odsłonięty czujnik DAQ-U jest w pełni odsłonięty i nie wymaga żadnego profilu korekcyjnego; czujnik DAQ-M jest używany wraz z nasadką Sunshine.)

## `pool-disconnect` — zwolnienie czujników

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Podsumowanie poleceń

| Polecenie | Cel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otwórz czujnik w puli backendowej. |
| `daq pool-list` | Wyświetla wszystkie czujniki w puli wraz z ich identyfikatorami `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Najnowsze N skalibrowanych ramek widma. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Wznowienie / wstrzymanie strumieniowania. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Rozpoczęcie / zatrzymanie nagrywania `.daq` (po stronie zaplecza). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmiana profilu korekcji ograniczeń w trakcie działania. |
| `daq pool-disconnect --sensor-id ID [--all]` | Zwolnienie jednego czujnika lub wszystkich. |

***

## Rozwiązywanie problemów przy pierwszym podłączeniu DAQ-E

1. Urządzenie DAQ-E nie posiada diody LED stanu — sprawdź zasilanie za pomocą wskaźnika PoE/link na przełączniku lub porcie iniektora i odczekaj kilka sekund po włączeniu zasilania, aby urządzenie się uruchomiło i dołączyło do sieci.
2. Komputer serwerowy musi znajdować się w **tej samej domenie rozgłoszeniowej** co czujnik — protokół mDNS nie przechodzi przez routery.
3. W przypadku modelu Windows zaakceptuj monit zapory Defender przy pierwszym uruchomieniu (mDNS UDP 5353, dane DAQ-E UDP 5002, PTP UDP 319/320).
4. Nadal brak odpowiedzi z `--eth`? Użyj `--eth-host` z nazwą hosta urządzenia (`daq-e-<id>.local`) lub adresem IP — jest to niezawodna ścieżka, zwłaszcza w przypadku hostów z wieloma adresami IP.

***{% hint style="info" %}**Wskazówka dla asystentów AI.** Każda strona niniejszej instrukcji jest udostępniana w formacie surowego Markdown — dodaj `.md` do małej litery slugu strony URL (ta strona: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); indeks czytelny dla maszyn to `https://mapir.gitbook.io/chloros/llms.txt`. Aby uzyskać pełną dokumentację na poziomie flag dla `chloros-cli daq` i każdej innej rodziny poleceń, pobierz [Podręcznik CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); ścieżka Python to `chloros_sdk.connect_daq_sensor()` w [Podręczniku SDK](../reference/sdk-reference.md).
{% endhint %}
