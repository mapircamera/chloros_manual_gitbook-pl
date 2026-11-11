# Sieć i synchronizacja czasu w DAQ-E

> Informacje dotyczące fizycznej konfiguracji sieciowej czujnika — okablowania, zasilania PoE, przypisania adresów IP oraz ustawień sieciowych samego urządzenia — znajdują się w **[instrukcji obsługi DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Ta strona dotyczy strony Chloros: podłączania, synchronizacji czasu oraz postępowania w przypadku, gdy wykrywanie urządzeń nie przyniesie rezultatów.

DAQ-E to członek rodziny DAQ obsługujący protokół Ethernet: zasilany przez PoE, wykrywany za pomocą mDNS (usługa `_daq-e._tcp`) i adresowalny za pomocą nazwy hosta wywodzącej się z identyfikatora czujnika — `daq-e-<6 hex>.local`, np. `daq-e-def330.local`. Ta strona opisuje, w jaki sposób urządzenie przesyła dane w sieci oraz jak uczestniczy w synchronizacji czasu PTP.

## Tryby transmisji

| Tryb | Punkt końcowy | Odbiorcy | Uwagi |
| --- | --- | --- | --- |
| **Multicast** (domyślnie) | UDP `239.10.10.10:5002` | Dowolna liczba urządzeń w tej samej sieci LAN odbiera ten sam strumień | Każdy datagram jest weryfikowany za pomocą CRC-16/CCITT |
| **Raw** | Port TCP `5000` | Dokładnie jeden klient (wyłącznie) | Kompatybilność bajtowa z DAQ-U |

Chloros domyślnie korzysta z multiemisji, co pozwala interfejsowi graficznemu (GUI), interfejsowi CLI oraz SDK na jednoczesne monitorowanie jednego czujnika.

## Wymagania sieciowe

* **Ta sama domena rozgłoszeniowa.** Komputer, na którym działa Chloros, musi znajdować się w tym samym segmencie sieci L2 co czujnik — wykrywanie mDNS nie przechodzi przez routery.
* **Zapora sieciowa.** W systemie Windows instalator dodaje regułę zapory **Chloros Backend (UDP-In)**dla backendu do każdego profilu sieciowego, która obejmuje dane DAQ-E (UDP 5002), mDNS (UDP 5353) oraz PTP (UDP 319/320), a także regułę**„Chloros Backend (TCP-In)”**, ograniczoną do lokalnej podsieci, dotyczącą połączenia nawiązywanego przez DAQ-E z komputerem podczas aktualizacji oprogramowania układowego. W systemie Linux a moduł `.deb` otwiera porty UDP w modułach `ufw` lub `firewalld`, gdy ta zapora jest aktywna w momencie instalacji. W przypadku zapory sieciowej innej firmy lub zapory włączonej później należy zezwolić na te same porty.
* **Zasilanie PoE, brak diody LED stanu.** Urządzenie DAQ-E nie posiada własnej diody LED — zasilanie należy sprawdzić za pomocą wskaźnika link/PoE na przełączniku lub porcie iniektora i odczekać kilka sekund po włączeniu zasilania, aż urządzenie uruchomi się i dołączy do sieci.

## Podłączanie

**GUI:** Zakładka „Czujniki światła” → „Podłącz czujnik” → Typ urządzenia „DAQ-E (Ethernet)”. Wyszukiwanie przebiega wyłącznie wtedy, gdy na ekranie wyświetlane jest okno dialogowe połączenia (przeglądanie mDNS oraz skanowanie ARP w systemie Windows), powtarzając się co 15 sekund; przycisk „Odśwież” powoduje natychmiastowe ponowne skanowanie. Wykryte czujniki pojawiają się w menu rozwijanym; automatycznie wybierany jest pierwszy wykryty czujnik.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (uruchomiony backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Czujnik DAQ-E znajdujący się już w puli jest rozpoznawany zarówno po nazwie hosta, jak i po adresie IP: ponowne podłączenie go pod innym adresem powoduje zwrócenie istniejącego wpisu w puli zamiast otwierania urządzenia po raz drugi.

### Hosty z wieloma kartami sieciowymi i pierwsze połączenie po uruchomieniu systemu

Na hostach z więcej niż jednym aktywnym interfejsem sieciowym **pierwsze** połączenie `pool-connect --eth` po uruchomieniu systemu może zakończyć się niepowodzeniem, nawet jeśli czujnik działa prawidłowo — proces wykrywania może pominąć interfejs, na którym znajduje się czujnik, gdy pamięć podręczna ARP jest jeszcze pusta. Niezawodnym rozwiązaniem jest pominięcie procesu wykrywania i jawne podanie adresu:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akceptuje nazwę hosta mDNS lub adres IP, zawsze kieruje się do właściwego czujnika i jest zalecaną formą w przypadku skryptów oraz instalacji bezinterfejsowych. W interfejsie graficznym należy użyć przycisku Odśwież w oknie dialogowym połączenia i poczekać na zakończenie cyklu ponownego skanowania.

## Ustawienia urządzenia i oprogramowanie układowe

Sam czujnik przechowuje swoje ustawienia sieciowe: DHCP (ustawienie fabryczne, z awaryjnym przełączeniem na adres lokalny 169.254.x.x, gdy żaden serwer DHCP nie odpowiada) lub adres statyczny. Przechowuje również informację, czy rozpoczyna transmisję po włączeniu zasilania. Ustawienia te można zmienić w interfejsie graficznym (w ustawieniach czujnika, ikona koła zębatego, **Sieć**:**Zmień…**,**Przesyłaj strumień po włączeniu zasilania**,**Uruchom ponownie czujnik**; zobacz [zakładkę DAQ](gui.md)), lub za pomocą interfejsu CLI i SDK, używając poleceń `pool-set-network`, `pool-set-auto-stream` oraz `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; zobacz [Podręcznik CLI](../reference/cli-reference.md#daq-e-device-settings)). Czujnik uruchamia się ponownie w celu zastosowania zmiany sieciowej, a Chloros ponownie się z nim łączy. Adres statyczny musi znajdować się w sieci komputera, na którym działa Chloros: Chloros odrzuca adres, z którym nie mógł się połączyć po ponownym uruchomieniu. Identyfikator czujnika oraz jego nazwa hosta `daq-e-<id>.local` nigdy się nie zmieniają. Nowość w wersji 1.3.0.

**Aktualizacje oprogramowania układowego są wbudowane w interfejs graficzny (GUI).** Gdy podłączone urządzenie DAQ-E korzysta z oprogramowania układowego starszego niż obraz dołączony do kompilacji Chloros, w wierszu tego czujnika wyświetla się bursztynowa ikona **Dostępna aktualizacja**, a w oknie modalnym ustawień (ikona koła zębatego) pojawia się <version>przycisk</version>

„Zaktualizuj do<version>”. Aktualizacja jest przesyłana przez sieć w około 30 sekund; czujnik uruchamia się ponownie i automatycznie łączy się ponownie, a przerwany transfer pozostawia aktualne oprogramowanie układowe nienaruszone. Z poziomu interfejsu CLI lub SDK oraz w systemie Linux, który nie posiada interfejsu graficznego, należy uruchomić polecenie `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronizacja czasu PTP

Oprogramowanie układowe DAQ-E w wersji 1.2.0+ uczestniczy w standardzie IEEE 1588 PTPv2 jako zwykły zegar (tylko w trybie slave). **Backend hosta Chloros pełni rolę grandmastera PTP** — każde urządzenie DAQ-E i każda kamera LATTICE w sieci LAN są jego slave'ami w domenie 0, utrzymując wszystkie znaczniki czasu urządzeń z tolerancją ~1 ms. To właśnie ten wspólny zegar sprawia, że odczyty z DAQ można dopasować pod względem znaczników czasu do ekspozycji kamer (zobacz [Nagrywanie i format .daq](recording.md)).

Sprawdź synchronizację z poziomu interfejsu CLI:

| Polecenie | Wyświetla |
| --- | --- |
| `chloros-cli time-sync status` | Stan hosta grandmaster, priorytety BMCA, identyfikator zegara oraz `health`: `OK` lub `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` wraz z informacją `health_detail` wyjaśniającą, dlaczego jego pakiety nie docierają do sieci |
| `chloros-cli time-sync peers` | Wszystkie wykryte urządzenia podrzędne (czujniki DAQ-E + kamery LATTICE) |
| `chloros-cli time-sync cameras` | Stan PTP dla poszczególnych kamer (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Ponowne uruchomienie procesu grandmastera |

W interfejsie graficznym okno modalne ustawień DAQ-E wyświetla na żywo wiersz **PTP Sync** z aktualnym stanem PTP czujnika.**Urządzenie DAQ-E z wyłączonym protokołem PTP pozostaje w tym stanie.** Urządzenie przechowuje ustawienie włączenia/wyłączenia protokołu PTP we własnej pamięci, więc urządzenie, w którym protokół PTP został wyłączony, pozostaje w tym stanie po ponownym podłączeniu lub wyłączeniu i włączeniu zasilania, a jego sygnatury czasowe nie są synchronizowane za pomocą protokołu PTP. `chloros-cli daq pool-list` oznacza takie urządzenie (`!! PTP is switched OFF on this unit …`) i wyświetla polecenie, które to naprawia:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros nigdy samoczynnie nie włącza urządzenia z powrotem, ponieważ wyłączenie może być zamierzone.

Szczegóły dla użytkowników wymagających ścisłej synchronizacji:

* Każdy przesyłany datagram zawiera pole flag; **bit 2 jest ustawiony w ramkach, których sygnatura czasowa jest zsynchronizowana z PTP**. Potoki wymagające ścisłej synchronizacji z kamerą/urządzeniem DAQ powinny sprawdzać ten bit.
* Przed zsynchronizowanym przechwytywaniem należy sprawdzić, czy czujnik pojawia się w `chloros-cli time-sync peers` oraz czy `chloros-cli daq pool-list` nie sygnalizuje, że jego PTP jest wyłączone. (Wewnętrzne narzędzia MAPIR do bezpośredniej obsługi sprzętu mogą również filtrować nagrywanie na podstawie blokady PTP za pomocą flagi `--wait-ptp`, która czeka do 15 s, aż czujnik osiągnie stan SLAVE; narzędzia te nie są częścią dostarczanego interfejsu CLI.)
* Gdy PTP aktywnie działa w trybie slave, czujnik odrzuca ręczne wysyłanie sygnału zegara („PTP dostarcza sygnał zegara”). Jest to zgodne z założeniami — należy zaufać PTP.

## Uwagi dotyczące systemu Linux

* **Porty PTP bez uprawnień roota.** PTP wykorzystuje porty UDP 319/320. Plik `.deb` zapisuje plik `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), więc backend wiąże się z nimi, działając jako użytkownik. Od wersji 1.3.0 pakiet nie ustawia żadnej zdolności plikowej (`setcap`) na backendzie i nie wymaga `libcap2-bin`; nie dodawaj tej zdolności samodzielnie, ponieważ rozpakowany backend wtedy się nie uruchomi. Jeśli PTP nie może przywiązać portu 319, zapoznaj się z sekcją [Instalacja w systemie Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson bez monitora / host z systemem Linux:** pakiet instaluje usługi backendu, ale nie włącza żadnej z nich. Aby zapewnić ciągłą pracę PTP (i dostępność DAQ) bez konieczności uruchamiania polecenia przez użytkownika, włącz usługę jako swój użytkownik:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Bez tego PTP działa tylko wtedy, gdy coś uruchomiło backend — polecenie `chloros-cli` lub skrypt SDK. Zobacz [Instalacja w systemie Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts), aby uzyskać informacje na temat konta root `chloros-backend.service` oraz sposobu przełączenia się z niego.

## Rozwiązywanie problemów: „Nie znaleziono urządzeń DAQ-E”

| Sprawdź | Szczegóły |
| --- | --- |
| Zasilanie | Brak świecącej diody LED na czujniku — sprawdź wskaźniki PoE i połączenia na przełączniku/porcie iniektora; odczekaj kilka sekund po włączeniu zasilania |
| Domena rozgłoszeniowa | Host i czujnik znajdują się w tym samym segmencie L2; mDNS nie przekierowuje ruchu |
| Zapora sieciowa | Windows: reguła instalatora **Chloros Backend (UDP-In)** zezwala na przepływ strumienia DAQ-E (UDP 5002), mDNS (5353) i PTP (319/320); zapora sieciowa innej firmy musi mieć takie same ustawienia. Linux: `.deb` otwiera te porty w `ufw` / `firewalld` tylko wtedy, gdy zapora sieciowa była aktywna w momencie instalacji — należy je samodzielnie zezwolić w przypadku zapory włączonej później |
| Host z wieloma kartami sieciowymi | Funkcja wykrywania przeszukuje mDNS z rezerwowym wykorzystaniem ARP; `--eth-host <ip-or-hostname>` pomija wykrywanie i łączy się bezpośrednio |
| Ponowne skanowanie w GUI | Funkcja wykrywania działa tylko wtedy, gdy okno dialogowe połączenia jest otwarte; użyj opcji Odśwież |</version>
