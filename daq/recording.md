# Rejestracja i format .daq

Plik `.daq` to format zapisu danych z czujnika światła Chloros: **baza danych SQLite** zawierająca skalibrowane klatki spektralne z jednego czujnika DAQ. Wystarczy zarejestrować taki plik podczas sesji przechwytywania, a następnie moduł przetwarzania współczynnika odbicia może podzielić każdy obraz przez natężenie promieniowania padającego zmierzone dokładnie w tym momencie.

## Co zawiera plik .daq

| Właściwość | Wartość |
| --- | --- |
| Kontener | Baza danych SQLite, jeden plik na czujnik na nagranie |
| Nazwa pliku | Zawiera **identyfikator czujnika**i**znacznik czasu**, np. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Widmo na klatkę | 135 punktów, 340–1010 nm w krokach co 5 nm, plus trójbodźcowe współrzędne CIE XYZ |
| Jednostki | Skalibrowane natężenie promieniowania spektralnego, **W/m²/nm** (zastosowano fabryczny pakiet kalibracyjny + profil osłony) |
| Metadane z datą i godziną | Identyfikator czujnika (klucz do pobrania fabrycznej kalibracji tego urządzenia) oraz aktualnie obowiązujący profil nakładki — zob. [Profile nakładek i zakres kalibracji](caps-and-range.md) |

Format jest identyczny dla modeli DAQ-U, DAQ-M i DAQ-E, więc dalsze przetwarzanie danych nie uwzględnia, które urządzenie transmisyjne dokonało rejestracji.

Rejestracja skalibrowana wymaga pakietu kalibracji fabrycznej czujnika. W przypadku modeli DAQ-U i DAQ-M moduł zaplecza pobiera pakiet z chmury MAPIR na podstawie identyfikatora czujnika (jeśli nie jest to możliwe, rejestracja zostanie odrzucona); urządzenia DAQ-E są wyłączone z tego wymogu, ponieważ przechowują dane kalibracyjne w pamięci urządzenia.

## Rejestracja z poziomu interfejsu graficznego

Rejestracja w interfejsie graficznym wymaga **otwartego projektu** (w przeciwnym razie przyciski „Record” są nieaktywne):

* **Record All / Stop All** — u góry paska bocznego „Light Sensors”; uruchamia lub zatrzymuje rejestrację `.daq` na wszystkich podłączonych czujnikach jednocześnie.
* **Nagrywaj / Zatrzymaj nagrywanie** — dla poszczególnych czujników, w oknie modalnym ustawień (ikona koła zębatego). Podczas nagrywania w wierszach informacji na żywo czujnika wyświetla się czerwony wskaźnik „REC”.

Pliki są zapisywane w `<project>/light_sensor/`, a po zatrzymaniu nagrywania — czy to poprzez przycisk „Zatrzymaj”, „Zatrzymaj wszystko”, czy też odłączenie czujnika nagrywającego — gotowy plik `.daq` jest **automatycznie dodawany do otwartego projektu**. Pojawia się on na liście plików projektu bez konieczności ręcznego dodawania, gotowy do przetwarzania współczynnika odbicia.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Nagrywanie z CLI

Urządzenie CLI rejestruje dane za pośrednictwem puli czujników modułu zaplecza (moduł zaplecza musi być uruchomiony — polecenia te są lekkimi klientami HTTP):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Pobierz wartość `--sensor-id` z `chloros-cli daq pool-list`. Dwie wartości domyślne, o których warto wiedzieć:

| Opcja | Domyślna |
| --- | --- |
| `--duration` | `0` — rejestruj do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` w systemie plików **backendu**, a nie w CLI |

Rozróżnienie katalogów wyjściowych ma znaczenie, gdy plik CLI jest skierowany do backendu na innym komputerze: plik trafia tam, gdzie działa backend.

## Nagrywanie z Python

`DAQSensorSession` (zwracany przez `chloros_sdk.connect_daq_sensor()`) udostępnia to samo nagranie z puli: `record_start(output_dir=None, device_name=None)` zwraca ścieżkę do pliku, a `record_stop()` zwraca `{path, rows}`. Pełną sesję API można znaleźć w [Opisie funkcji SDK](../reference/sdk-reference.md). Klasy sprzętowe SDK (tylko w instalacjach na komputerach stacjonarnych) domyślnie zapisują nagrania do `~/Documents/DAQ/`; w przypadku wydanych kompilacji obsługiwana jest powyższa ścieżka zbiorcza.

## Korzystanie z pliku .daq podczas przetwarzania

Aby obliczyć współczynnik odbicia na podstawie obrazów, Chloros wymaga natężenia promieniowania padającego dopasowanego do każdej ekspozycji:

* **Należy zachować plik `.daq` wraz z obrazami.**Podczas przetwarzania potok automatycznie ustala**natężenie promieniowania opadającego dopasowane czasowo** na podstawie zarejestrowanego pliku `.daq` (dowolny model DAQ) — lub na podstawie natywnego pliku `.csv` — znalezionego obok obrazów. Nagrania z interfejsu GUI spełniają ten warunek automatycznie, ponieważ są dodawane do projektu natychmiast po zakończeniu.
* **Kalibracja jest pobierana na żądanie.** Jeśli pakiet kalibracji fabrycznej dla danej kamery lub urządzenia DAQ nie został jeszcze zapisany w lokalnej pamięci podręcznej, plik Chloros automatycznie pobiera go z chmury MAPIR przy pierwszym użyciu (wymagane jednorazowe połączenie z internetem; zapisywane w pamięci podręcznej pod nazwą `~/.chloros/`).
* **Przechwytywane na żywo obrazy tworzą własny plik towarzyszący.** W przypadku każdej klatki odbicia światła przechwyconej na żywo faktycznie wykorzystany odczyt z modułu DAQ jest zapisywany jako plik towarzyszący `.daq` obok obrazu, dzięki czemu przechwycony materiał można później ponownie przetworzyć bez konieczności korzystania z oryginalnego nagrania.

## Odzyskiwanie danych o natężeniu promieniowania

Przetwarzanie projektu powoduje również wyeksportowanie wszystkich zawartych w nim zapisów z czujników światła do
folderu `Light Sensor/` znajdującego się obok produktów obrazowych. Nie jest do tego **potrzebny** materiał obrazowy:
samodzielnie przelatujący czujnik światła stanowi kompletny zapis, a folder zawierający wyłącznie pliki `.daq`
jest prawidłowym materiałem wejściowym. Raport z przebiegu wskazuje, ile produktów z czujnika światła zostało zapisanych.

| Produkt | Co to jest |
| --- | --- |
| `<name>_calibrated.daq` | Archiwum nadające się do ponownego przetworzenia, zgodne ze schematem nagrania na żywo, zawierające teraz informacje o pakiecie kalibracyjnym, który je wygenerował. Ponowne zaimportowanie tego archiwum **nie** powoduje jego drugiej kalibracji. |
| `<name>_calibrated.csv` | Natężenie promieniowania spektralnego w W/m²/nm w siatce długości fal właściwej dla czujnika, jeden wiersz na odczyt, wraz z kolumnami fotometrycznymi: moc całkowita, luks fotopowy i skotopowy, PPFD z podziałem na niebieski/zielony/czerwony oraz długość fali szczytowej. |

Urządzenie DAQ-U lub DAQ-M, którego pakietu kalibracyjnego nie można pobrać — użytkownik jest offline lub
dla tego czujnika nie ma kalibracji w archiwum — jest **pomijane z podaniem przyczyny** i nigdy nie jest zapisywane
jako plik „skalibrowany” zawierający surowe odczyty. Należy połączyć się z Internetem i uruchomić proces ponownie. Urządzenie DAQ-E
posiada własną kalibrację, więc potrzebuje jej tylko wtedy, gdy urządzenie nie jest podłączone i
nic nie jest buforowane lokalnie.

### DAQ-A: surowe odczyty i dlaczego jest to właściwe rozwiązanie

**DAQ-A** powstał przed wprowadzeniem systemu pakietów kalibracyjnych dla poszczególnych numerów seryjnych i nie ma żadnego pakietu do
pobrania. Nie jest to przeoczenie: urządzenie DAQ-A jest kalibrowane w terenie względem
celu odbiciowego, a kalibracja oparta na celu wymaga jedynie *względnej*
charakterystyki czujnika — czyli dokładnie tego, czym są surowe odczyty. Urządzenie Chloros jest obecnie kalibrowane przy ich użyciu.

Zatem zapis z czujnika DAQ-A można wyeksportować, ale pod inną nazwą:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, a nie `_calibrated` — chodzi o inną nazwę pliku, a nie o flagę wewnątrz pliku,
ponieważ informacja ta musi przetrwać wysłanie pliku pocztą elektroniczną jako sama nazwa. Nagłówek `.csv`
wskazuje `raw spectral sensor counts (NOT irradiance)` i ostrzega, że wartości są
porównywalne **w obrębie** pliku, a nie między czujnikami. Kolumny, które mają znaczenie
tylko w przypadku rzeczywistego natężenia promieniowania — moc całkowita, luks, PPFD — pozostają puste, a nie są
obliczane na podstawie zliczeń.

Starsze zapisy DAQ-A-SD (schemat v1.01 / v1.02) rejestrują jedynie czas zapisu pliku, a nie
znacznik czasu dla każdego odczytu. Chloros nie będzie porównywać obrazów z tymi danymi — przyporządkowanie klatki do
czasu zapisu byłoby błędne, choć na pierwszy rzut oka nie wyglądałoby na błędne — ale eksport odczytuje je poprawnie, a
CSV określa, na którym zegarze się znajdują.

Pełny opis procesu pomiaru odbicia — z jednym czujnikiem z kamerą oraz z dwoma czujnikami (otoczenie/obiekt) — znajduje się w [Procedurach pomiaru odbicia](reflectance.md).
