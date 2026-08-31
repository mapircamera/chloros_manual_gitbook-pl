# Czujniki światła DAQ

> **Szukasz informacji o sprzęcie?**Same czujniki — modele, sposób montażu, osłony, porty, zasilanie oraz aplikacja SCANNER — zostały opisane w**[instrukcji obsługi DAQ](https://mapir.gitbook.io/daq)**. W niniejszym rozdziale omówiono sposób korzystania z nich, począwszy od modelu Chloros.

Czujniki światła **DAQ** z MAPIR mierzą światło otoczenia w postaci spektrum skalibrowanego radiometrycznie. W Chloros pełnią one dwie role:

* **Samodzielny przyrząd spektralny** — wykresy widma na żywo, dane kolorymetryczne oraz zapisy `.daq` — wszystko to dostępne z [zakładki „Czujniki światła”](gui.md), [CLI](cli-quick-start.md) lub z Python SDK.
* **Źródło natężenia promieniowania padającego na powierzchnię do obliczania współczynnika odbicia** — podczas przetwarzania moduł Chloros interpoluje odczyty z modułu `.daq` do znacznika czasu ekspozycji każdego ujęciaznacznika czasu ekspozycji i wykorzystuje zmierzone natężenie promieniowania padającego do przekształcenia radiancji kamery w współczynnik odbicia (`--reflectance-source daq`); w przypadku skalibrowanych pasm nie jest wymagany panel w scenie.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trzy modele, jeden format danych

| Model | Transport | Wykrywanie |
| --- | --- | --- |
| **DAQ-U** | USB (szeregowy) | skanowanie portu szeregowego |
| **DAQ-M** | Bluetooth Low Energy | skanowanie BLE według nazwy |
| **DAQ-E** | Ethernet (IPv4, zasilanie PoE) | mDNS `_daq-e._tcp` (nazwa hosta `daq-e-<id>.local`) |

Wszystkie trzy urządzenia wykorzystują ten sam protokół komunikacyjny i dostarczają identyczne dane:

* **135-punktowe widmo w zakresie od 340 do 1010 nm z krokiem co 5 nm** oraz wartości trójbodźcowe CIE XYZ w każdej ramce.
* **Kalibrowane radiometrycznie natężenie promieniowania spektralnego w W/m²/nm** — pakiet kalibracji fabrycznej każdego urządzenia (wraz z aktywnym profilem korekcji nasadki) jest stosowany, zanim dane dotrą do użytkownika.
* Ten sam **format zapisu `.daq`** (plik SQLite). Dalsze przetwarzanie danych jest identyczne niezależnie od tego, który protokół transmisji wygenerował plik.

Stosy protokołów transmisji (szeregowy USB, BLE, mDNS/zeroconf) są wbudowane w backend Chloros — nie trzeba niczego instalować, aby komunikować się z którymkolwiek z trzech modeli za pomocą interfejsu graficznego lub poleceń `pool-*` w CLI.

***

## Zakres kalibracji: zgłaszany 340–1010 nm, skalibrowany ~374–974 nm

Czujnik zgłasza pełną siatkę 340–1010 nm, ale wzmocnienie radiometryczne zgodne z normami NIST obejmuje zakres około **374–974 nm**. Chloros odrzuca podział na odbicie bezwzględne dla każdego pasma kamery, którego mniej niż połowa wagi spektralnej mieści się w tym skalibrowanym zakresie; pominięte pasmo jest zgłaszane z przyczyną pominięcia `dls-uncalibrated-band-<nm>`.

Spośród dostępnych w sprzedaży filtrów LATTICE problem dotyczy wyłącznie modelu **F988**:

Odbicie F988 jest kalibrowane przy użyciu panelu odbicia umieszczonego w scenie: pasmo to wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros stosuje najnowsze dane z panelu i utrzymuje je pomiędzy kolejnymi pomiarami panelu.

Jeśli rejestracja F988 jest przetwarzana wyłącznie na podstawie dostępnych danych z DAQ, kod Chloros odrzuca współczynnik odbicia oparty na danych z DAQ dla tego pasma z przyczyną pominięcia `dls-uncalibrated-band-988` — [procedura pracy z panelem odbicia](../calibration-targets.md) jest obsługiwanym schematem dla F988.

***

## Identyfikatory czujników

Każdy moduł DAQ zgłasza stały identyfikator czujnika. Jego format różni się w zależności od modelu:

| Model | Format identyfikatora | Przykład |
| --- | --- | --- |
| DAQ-U | 5-oktetowy z łącznikami | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktetowy z łącznikami | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Identyfikator czujnika to:

* wartość umieszczona w każdym pliku `.daq`, który rejestruje,
* klucz używany przez Chloros do pobrania pakietu kalibracji fabrycznej tego urządzenia,
* wartością przekazaną do `--sensor-id` w poleceniach CLI `pool-*`, oraz
* w przypadku urządzenia DAQ-E również jego nazwę hosta mDNS (`daq-e-def330.local`) — wartość akceptowaną przez `--eth-host`.

***

## Kalibracja fabryczna i chmura

Każde urządzenie DAQ jest indywidualnie kalibrowane fabrycznie przy użyciu łańcucha radiometrycznego zgodnego z normami NIST, a Chloros ładuje pakiet kalibracyjny każdego urządzenia, przypisany na podstawie identyfikatora czujnika. Raport kalibracyjny dla poszczególnych urządzeń (PDF) można pobrać z ustawień czujnika w [zakładce „Czujniki światła”](gui.md).

{% hint style="warning" %}
**Modele DAQ-U i DAQ-M wymagają dostępu do chmury w celu kalibracji.**Żaden z tych modeli nie przechowuje żadnych danych lokalnie: ich fabryczne pakiety kalibracyjne znajdują się w chmurze MAPIR i są pobierane na podstawie identyfikatora czujnika (a następnie zapisywane w pamięci podręcznej lokalnie). Chloros wymaga połączenia z Internetem, aby dostarczać skalibrowane dane w W/m²/nm z urządzenia DAQ-U lub DAQ-M.**Wyjątkiem jest DAQ-E** — przechowuje ono swoje dane kalibracyjne na samym urządzeniu.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Gdzie trafiają zapisy

| Powierzchnia | Domyślne miejsce docelowe dla `.daq` |
| --- | --- |
| Interfejs użytkownika — zakładka „Czujniki światła” | `<project folder>/light_sensor/` (gotowe zapisy są automatycznie dodawane do projektu) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na komputerze, na którym działa backend |

Każda nazwa pliku `.daq` zawiera identyfikator czujnika oraz sygnaturę czasową.

***

## W tym rozdziale

* [**Karta DAQ w Chloros**](gui.md) — pełny przewodnik po interfejsie graficznym: podłączanie poszczególnych modeli, ustawienia poszczególnych czujników, wykresy widmowe, dane kolorymetryczne na żywo, odbicie dwuczujnikowe oraz rejestracja.
* [**CLI Szybki start (pool-\*)**](cli-quick-start.md) — sterowanie czujnikami DAQ z poziomu `chloros-cli daq pool-*`, obsługiwana ścieżka wiersza poleceń.
* [**Profile ograniczeń i skalibrowany zakres**](caps-and-range.md) — jakie ograniczenia istnieją dla poszczególnych modeli, jak je zadeklarować oraz szczegółowy opis skalibrowanego zakresu spektralnego.
* [**Rejestracja i format .daq**](recording.md) — format SQLite `.daq` oraz procedury rejestracji.
* [**Sieć DAQ-E i synchronizacja czasu**](ethernet-ptp.md) — tryby transmisji DAQ-E oraz synchronizacja czasu PTP.
* [**Procesy pomiaru odbicia**](reflectance.md) — wykorzystanie danych DAQ dotyczących promieniowania padającego do obliczenia odbicia.
* Pełna dokumentacja na poziomie flag znajduje się w [Podręczniku CLI](../reference/cli-reference.md) (sekcja `chloros-cli daq`) oraz w [Podręcznika SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), które zostały napisane tak, aby mogły być bezpośrednio wykorzystywane przez asystentów AI.
