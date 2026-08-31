# Sieć i synchronizacja czasu w DAQ-E

> Informacje dotyczące fizycznej konfiguracji sieciowej czujnika — okablowania, zasilania PoE, przypisania adresów IP oraz ustawień sieciowych samego urządzenia — znajdują się w **[instrukcji obsługi DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Ta strona dotyczy urządzenia Chloros: podłączania, synchronizacji czasu oraz postępowania w przypadku, gdy wykrywanie nie przyniesie wyników.

DAQ-E to urządzenie z rodziny DAQ obsługujące protokół Ethernet: zasilane przez PoE, wykrywane za pomocą mDNS (usługa `_daq-e._tcp`) i adresowalne za pomocą nazwy hosta wywodzącej się z identyfikatora czujnika — `daq-e-<6 hex>.local`, np. `daq-e-def330.local`. Ta strona opisuje, w jaki sposób urządzenie przesyła dane w sieci oraz jak uczestniczy w synchronizacji czasu PTP.

## Tryby transmisji

| Tryb | Punkt końcowy | Odbiorcy | Uwagi |
| --- | --- | --- | --- |
| **Multicast** (domyślnie) | UDP `239.10.10.10:5002` | Dowolna liczba urządzeń w tej samej sieci LAN odbiera ten sam strumień | Każdy datagram jest sprawdzany za pomocą CRC-16/CCITT |
| **Raw** | Port TCP `5000` | Dokładnie jeden klient (wyłącznie) | Kompatybilny bajtowo z DAQ-U |

Chloros domyślnie korzysta z multiemisji, co pozwala interfejsowi graficznemu, CLI oraz SDK na jednoczesne monitorowanie jednego czujnika.

## Wymagania sieciowe

* **Ta sama domena rozgłoszeniowa.** Komputer, na którym działa Chloros, musi znajdować się w tym samym segmencie sieci L2 co czujnik — wykrywanie mDNS nie przechodzi przez routery.
* **Monit zapory Windows: zaakceptuj go.** Gdy Chloros po raz pierwszy wiąże gniazda multiemisji, program Windows Defender wyświetla jednorazowy monit. Zezwolenie na to obejmuje dane DAQ-E (UDP 5002), mDNS (UDP 5353) oraz PTP (UDP 319/320). W przypadku Linux proces ten przebiega bez komunikatów.
* **Zasilanie PoE, brak diody LED stanu.** Urządzenie DAQ-E nie posiada własnej diody LED — sprawdź zasilanie za pomocą wskaźnika link/PoE na przełączniku lub porcie iniektora i odczekaj kilka sekund po włączeniu zasilania, aż urządzenie uruchomi się i dołączy do sieci.

## Podłączanie

**Interfejs graficzny:** Zakładka „Czujniki światła” → „Podłącz czujnik” → Typ urządzenia „DAQ-E (Ethernet)”. Wykrywanie przebiega wyłącznie wtedy, gdy na ekranie wyświetlane jest okno dialogowe połączenia (przeglądanie mDNS oraz skanowanie ARP na Windows), powtarzając się co 15 sekund; przycisk Odśwież powoduje natychmiastowe ponowne skanowanie. Wykryte czujniki pojawiają się w menu rozwijanym; automatycznie wybierany jest pierwszy wykryty czujnik.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (działający backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hosty z wieloma kartami sieciowymi i pierwsze połączenie po uruchomieniu systemu

Na hostach z więcej niż jedną aktywną kartą sieciową **pierwsze** `pool-connect --eth` po uruchomieniu systemu może nie zawierać żadnych danych, nawet jeśli czujnik działa poprawnie — proces wykrywania może pominąć interfejs, na którym znajduje się czujnik, gdy pamięć podręczna ARP jest jeszcze niezaładowana. Niezawodnym rozwiązaniem jest pominięcie procesu wykrywania i jawne podanie adresu:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akceptuje nazwę hosta mDNS lub adres IP, zawsze kieruje się do właściwego czujnika i jest zalecaną formą w przypadku skryptów oraz instalacji bezinterfejsowych. W interfejsie graficznym należy użyć przycisku „Odśwież” w oknie dialogowym połączenia i poczekać na zakończenie cyklu ponownego skanowania.

## Ustawienia urządzenia i oprogramowanie układowe

Sam czujnik przechowuje ustawienia sieciowe — statyczny adres IP lub DHCP wraz z adresowaniem lokalnym łącza, nazwę urządzenia, automatyczne uruchamianie strumienia podczas startu systemu oraz hasło OTA. Te ustawienia po stronie urządzenia nie są udostępniane jako polecenia w dostarczonym CLI; są one zarządzane za pośrednictwem interfejsu graficznego Chloros, w którym są wyświetlane, lub przy pomocy pomocy technicznej MAPIR.

**Aktualizacje oprogramowania układowego są wbudowane w interfejs GUI.**Gdy podłączony moduł DAQ-E korzysta z oprogramowania układowego starszego niż obraz dołączony do kompilacji Chloros, w rzędzie czujnika pojawia się bursztynowa ikona**Dostępna aktualizacja**, a w oknie modalnym ustawień w postaci koła zębatego dostępny jest<version>

przycisk</version> „Zaktualizuj do<version>

”. Aktualizacja jest przesyłana przez sieć w ciągu około 30 sekund; czujnik uruchamia się ponownie i automatycznie łączy się ponownie, a przerwany transfer pozostawia aktualne oprogramowanie układowe nienaruszone.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronizacja czasu PTP

Oprogramowanie sprzętowe DAQ-E w wersji 1.2.0+ uczestniczy w standardzie IEEE 1588 PTPv2 jako zwykły zegar (tylko w trybie slave). **Backend hosta Chloros pełni rolę grandmastera PTP** — każde urządzenie DAQ-E i każda kamera LATTICE w sieci LAN są jego slave’ami w domenie 0, utrzymując wszystkie sygnatury czasowe urządzeń w granicach tolerancji ~1 ms. To właśnie ten wspólny zegar pozwala na dopasowanie znaczników czasu odczytów DAQ do ekspozycji kamer (zobacz [Nagrywanie i format .daq](recording.md)).

Sprawdź synchronizację z pliku CLI:

| Polecenie | Wyświetla |
| --- | --- |
| `chloros-cli time-sync status` | Stan hosta grandmaster, priorytety BMCA, identyfikator zegara |
| `chloros-cli time-sync peers` | Wszystkie wykryte urządzenia podrzędne (czujniki DAQ-E + kamery LATTICE) |
| `chloros-cli time-sync cameras` | Stan PTP dla poszczególnych kamer (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Ponowne uruchomienie procesu grandmastera |

W interfejsie graficznym okno modalne ustawień DAQ-E wyświetla na żywo wiersz **PTP Sync** z aktualnym stanem PTP czujnika.

Szczegóły dla odbiorców wymagających ścisłej synchronizacji:

* Każdy przesyłany datagram zawiera pole flag; **bit 2 jest ustawiony w ramkach, których znacznik czasu jest zsynchronizowany z PTP**. Potoki wymagające ścisłej synchronizacji kamery z DAQ powinny filtrować dane na podstawie tego bitu.
* Przed rozpoczęciem zsynchronizowanego przechwytywania należy sprawdzić, czy czujnik pojawia się w `chloros-cli time-sync peers`. (Wewnętrzne narzędzia sprzętowe MAPIR mogą również sterować nagrywaniem na podstawie blokady PTP za pomocą flagi `--wait-ptp`, która czeka do 15 s, aż czujnik osiągnie stan SLAVE; narzędzie to nie wchodzi w skład dostarczanego CLI.)
* Gdy PTP aktywnie działa w trybie slave, czujnik odrzuca ręczne sygnały zegara („PTP dostarcza zegar”). Jest to zgodne z założeniami — należy zaufać PTP.

## Uwagi dotyczące Linux

* **PTP wymaga `libcap2-bin` podczas instalacji.** Skrypt postinst `.deb` przyznaje uprawnienia `cap_net_bind_service=+ep` na `/usr/lib/chloros/chloros-backend`, dzięki czemu może on przywiązać porty PTP 319/320 bez uprawnień roota. Jeśli brakuje pliku `libcap2-bin`, ten krok zostanie pominięty, a PTP nie uruchomi się. Rozwiązanie:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Urządzenia Jetson / Raspberry Pi bez wyświetlacza:** podczas pierwszej instalacji generowana jest jednostka systemd `chloros-backend.service`, ale nie jest ona włączona. Aby zapewnić stałą dostępność PTP (i DAQ) bez interfejsu graficznego:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Bez tego PTP działa tylko wtedy, gdy otwarty jest interfejs graficzny Chloros.

## Rozwiązywanie problemów: „Nie znaleziono urządzeń DAQ-E”

| Sprawdź | Szczegóły |
| --- | --- |
| Zasilanie | Brak świecącej diody LED na czujniku — sprawdź wskaźniki PoE i połączenia na przełączniku/porcie iniektora; odczekaj kilka sekund po włączeniu zasilania |
| Domena rozgłoszeniowa | Host i czujnik znajdują się w tym samym segmencie L2; mDNS nie przekierowuje ruchu |
| Zapora Windows | Zaakceptuj monit programu Defender przy pierwszym uruchomieniu (UDP 5002, 5353, 319/320) |
| Host z wieloma kartami sieciowymi | Pierwsze wykrywanie po uruchomieniu może pominąć czujnik — połącz się za pomocą `--eth-host <ip-or-hostname>` |
| Ponowne skanowanie w GUI | Wykrywanie przebiega tylko wtedy, gdy okno dialogowe połączenia jest otwarte; użyj przycisku Odśwież |</version>
