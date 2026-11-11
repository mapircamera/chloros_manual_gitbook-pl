# Czujniki światła DAQ

> **Szukasz informacji o sprzęcie?**Same czujniki — modele, sposób montażu, osłony, porty, zasilanie oraz aplikacja SCANNER — zostały opisane w**[instrukcji obsługi DAQ](https://mapir.gitbook.io/daq)**. W niniejszym rozdziale omówiono sposób korzystania z nich w programie Chloros. Czujniki światła**DAQ** firmy

MAPIR mierzą oświetlenie otoczenia w postaci widm skalibrowanych radiometrycznie. W programie Chloros pełnią one dwie funkcje:

* **Samodzielny przyrząd spektralny** — wykresy widma na żywo, dane kolorymetryczne oraz zapisy `.daq`, wszystko dostępne z [zakładki „Czujniki światła”](gui.md), [interfejsu CLI](cli-quick-start.md) lub SDK Python w języku Python.
* **Źródło natężenia promieniowania padającego na powierzchnię do obliczania współczynnika odbicia** — podczas przetwarzania Chloros interpoluje odczyty `.daq` do znacznika czasu ekspozycji każdego ujęcia i wykorzystuje zmierzone natężenie promieniowania padającego do przekształcenia radiancji z kamery na współczynnik odbicia (`--reflectance-source daq`); w przypadku skalibrowanych pasm nie jest wymagany panel w scenie.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Trzy modele, jeden format danych

| Model | Przesył | Wykrywanie |
| --- | --- | --- |
| **DAQ-U** | USB (szeregowy) | skanowanie portu szeregowego |
| **DAQ-M** | Bluetooth Low Energy | skanowanie BLE według nazwy |
| **DAQ-E** | Ethernet (IPv4, zasilanie PoE) | mDNS `_daq-e._tcp` (nazwa hosta `daq-e-<id>.local`) |**DAQ-E-S (nowość w wersji 1.3.0).** Urządzenie DAQ-E z stałym dyfuzorem i czujnikiem pochylenia. W Chlorosie figuruje ono jako `DAQ-E-S` i działa dokładnie tak samo jak DAQ-E, z tą różnicą, że nie wymaga korekcji nakładki, a każdy odczyt zawiera informacje o pochyleniu, przechylu i nachyleniu urządzenia. Zobacz [DAQ-E-S: stały dyfuzor i czujnik pochylenia](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Wszystkie trzy urządzenia wykorzystują ten sam protokół komunikacyjny i dostarczają identyczne dane:

* **135-punktowe widmo w zakresie od 340 do 1010 nm z krokiem co 5 nm** oraz wartości trójbodźcowe CIE XYZ w każdej ramce.
* **Kalibrowane radiometrycznie natężenie promieniowania spektralnego w W/m²/nm** — przed dostarczeniem danych do użytkownika stosowany jest fabryczny zestaw kalibracyjny każdego urządzenia (wraz z aktywnym profilem korekcji nasadki).
* Ten sam **format zapisu `.daq`** (plik SQLite). Dalsze przetwarzanie danych jest identyczne niezależnie od tego, który protokół transmisji wygenerował plik.

Stosy protokołów komunikacyjnych (USB szeregowy, BLE, mDNS/zeroconf) są zintegrowane z zapleczem Chloros — nie trzeba instalować niczego, aby komunikować się z którymkolwiek z trzech modeli za pomocą interfejsu graficznego lub poleceń `pool-*` w CLI.

***

## Zakres kalibracji: zgłaszany 340–1010 nm, skalibrowany ~374–974 nm

Czujnik zgłasza pełną siatkę 340–1010 nm, ale wzmocnienie radiometryczne zgodne z normami NIST obejmuje zakres około **374–974 nm**. Chloros odrzuca podział na odbicie bezwzględne dla każdego pasma kamery, którego mniej niż połowa wagi spektralnej mieści się w tym skalibrowanym zakresie; pominięte pasmo jest zgłaszane z przyczyną pominięcia `dls-uncalibrated-band-<nm>`.

Spośród dostępnych w sprzedaży filtrów LATTICE dotyczy to wyłącznie modelu **F988**:

Odbicie światła dla modelu F988 jest kalibrowane przy użyciu panelu odbicia światła umieszczonego w scenie: pasmo to wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros stosuje najnowszy zapis z panelu i utrzymuje tę wartość między kolejnymi pomiarami panelu.

Jeśli dane z modelu F988 są przetwarzane wyłącznie na podstawie danych z DAQ, Chloros odrzuca współczynnik odbicia oparty na DAQ dla tego pasma, podając powód pominięcia `dls-uncalibrated-band-988` — [procedura pracy z panelem odbicia](../calibration-targets.md) jest obsługiwanym sposobem postępowania dla F988.

***

## Identyfikatory czujników

Każdy moduł DAQ zgłasza stały identyfikator czujnika. Jego format różni się w zależności od modelu:

| Model | Format identyfikatora | Przykład |
| --- | --- | --- |
| DAQ-U | 5-oktetowy z łącznikami | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktetowy z łącznikami | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Identyfikator czujnika jest:

* umieszczony w każdym pliku `.daq`, który rejestruje,
* kluczem używanym przez Chloros do pobrania fabrycznego pakietu kalibracyjnego tego urządzenia,
* wartością, którą przekazujesz do `--sensor-id` w poleceniach CLI, oraz
* w przypadku urządzenia DAQ-E jest to również jego nazwa hosta mDNS (`daq-e-def330.local`) — wartość akceptowana przez `--eth-host`.

***

## Kalibracja fabryczna i chmura

Każde urządzenie DAQ jest indywidualnie kalibrowane fabrycznie przy użyciu łańcucha radiometrycznego zgodnego z normami NIST, a firma Chloros ładuje pakiet kalibracyjny dla każdego urządzenia, przypisany na podstawie identyfikatora czujnika. Raport kalibracyjny dla poszczególnych urządzeń (PDF) można pobrać z ustawień czujnika w [zakładce „Czujniki światła”](gui.md).

{% hint style="warning" %}
**Urządzenia DAQ-U i DAQ-M wymagają dostępu do chmury w celu kalibracji.**Żaden z tych modeli nie przechowuje żadnych danych lokalnie: ich fabryczne pakiety kalibracyjne znajdują się w chmurze MAPIR i są pobierane na podstawie identyfikatora czujnika (a następnie zapisywane w pamięci podręcznej lokalnie). Chloros potrzebuje połączenia z Internetem, aby dostarczać skalibrowane dane w jednostkach W/m²/nm z DAQ-U lub DAQ-M.**Wyjątkiem jest DAQ-E** — przechowuje on swoje dane kalibracyjne w pamięci urządzenia.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Gdzie trafiają zapisy

| Powierzchnia | Domyślne miejsce docelowe `.daq` |
| --- | --- |
| Interfejs graficzny — zakładka Czujniki światła | `<project folder>/light_sensor/` (gotowe zapisy są automatycznie dodawane do projektu) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na komputerze, na którym działa backend (względna ścieżka `-o` odnosi się do folderu, w którym uruchomiono polecenie, gdy backend znajduje się na tym samym komputerze) |

Każda nazwa pliku `.daq` zawiera identyfikator czujnika oraz sygnaturę czasową.

***

## W tym rozdziale

* [**Karta DAQ w programie Chloros**](gui.md) — pełny przewodnik po interfejsie graficznym: podłączanie poszczególnych modeli, ustawienia poszczególnych czujników, wykresy widmowe, dane kolorymetryczne na żywo, odbicie światła z dwóch czujników oraz rejestrowanie danych.
* [**Szybki start CLI (pool-\*)**](cli-quick-start.md) — sterowanie czujnikami DAQ z poziomu `chloros-cli daq pool-*`, obsługiwanej ścieżki wiersza poleceń.
* [**Profile ograniczeń i skalibrowany zakres**](caps-and-range.md) — jakie ograniczenia istnieją dla poszczególnych modeli, jak je zadeklarować oraz szczegółowy opis skalibrowanego zakresu spektralnego.
* [**Rejestracja i format .daq**](recording.md) — format SQLite `.daq` oraz procesy rejestracji.
* [**Sieć DAQ-E i synchronizacja czasu**](ethernet-ptp.md) — tryby transmisji DAQ-E oraz synchronizacja czasu PTP.
* [**Procesy związane z odbiciem**](reflectance.md) — wykorzystanie danych DAQ dotyczących promieniowania padającego do obliczenia odbicia.
* Pełna dokumentacja dotycząca flag znajduje się w [Podręczniku CLI](../reference/cli-reference.md) (sekcja `chloros-cli daq`) oraz w [Podręczniku SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), które zostały opracowane z myślą o bezpośrednim wykorzystaniu przez asystentów AI.
