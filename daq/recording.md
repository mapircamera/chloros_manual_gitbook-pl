# Rejestracja i format .daq

Plik `.daq` to format zapisu danych z czujnika światła stosowany w programie Chloros: **baza danych SQLite** zawierająca skalibrowane klatki spektralne z jednego czujnika DAQ. Wystarczy zarejestrować jeden taki plik podczas sesji przechwytywania, a moduł przetwarzania odbicia światła będzie mógł później podzielić każdy obraz przez natężenie promieniowania padającego zmierzone dokładnie w tym momencie.

## Co zawiera plik .daq

| Właściwość | Wartość |
| --- | --- |
| Kontener | Baza danych SQLite, jeden plik na czujnik na nagranie |
| Nazwa pliku | Zawiera **identyfikator czujnika**i**znacznik czasu**, np. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Widmo na klatkę | 135 punktów, 340–1010 nm w krokach co 5 nm, plus trójbodźcowa koordynata CIE XYZ |
| Jednostki | Skalibrowane natężenie promieniowania spektralnego, **W/m²/nm** (zastosowano fabryczny pakiet kalibracyjny + profil osłony) |
| Metadane z sygnaturą | Identyfikator czujnika (klucz do pobrania fabrycznej kalibracji tego urządzenia) oraz aktualnie obowiązujący profil nasadki — zobacz [Profile nasadek i zakres kalibracji](caps-and-range.md) |

Format jest identyczny dla modeli DAQ-U, DAQ-M, DAQ-E i DAQ-E-S, więc dalsze przetwarzanie danych nie uwzględnia, które urządzenie pomiarowe dokonało rejestracji. Rejestracja z urządzenia DAQ-E-S przechowuje również dane dotyczące przechyłu, przechylenia bocznego i pochylenia urządzenia obok każdego widma.

**Zapisy są zachowywane w przypadku przerwy w zasilaniu (wersja 1.3.0).**Zapis na żywo jest zapisywany z dziennikiem i pełną synchronizacją dysku przy każdym zatwierdzeniu, więc przerwa w zasilaniu lub zakończenie procesu powoduje utratę danych z ostatnich około 3 sekund, a nie całego pliku. Zapis zatrzymany w trakcie zapisywania może pozostawić plik `<name>.daq-journal` obok pliku `.daq`:**należy zachować i skopiować oba pliki razem.** Program Chloros przywraca dziennik do poprzedniego stanu przy pierwszym odczycie pliku. Zajęty dysk opóźnia zapis pliku, ale nigdy nie wpływa na odczyty czujnika.

**Urządzenie DAQ-E, które przestaje przesyłać dane, zostaje ponownie podłączone (wersja 1.3.0).** Gdy strumień danych z urządzenia DAQ-E milknie (z powodu wyłączenia i ponownego włączenia zasilania przełącznika lub utraty połączenia), backend ponownie je podłącza z zachowaniem dotychczasowych ustawień, a nagrywanie na tym urządzeniu jest kontynuowane w tym samym pliku `.daq` z zaznaczoną w pliku przerwą.

Kalibrowane nagrywanie wymaga fabrycznego pakietu kalibracyjnego czujnika. W przypadku urządzeń DAQ-U i DAQ-M backend pobiera pakiet z chmury MAPIR na podstawie identyfikatora czujnika (jeśli nie jest to możliwe, nagrywanie zostaje odrzucone); urządzenia DAQ-E są wyłączone z tego wymogu, ponieważ przechowują kalibrację w pamięci urządzenia.

## Nagrywanie z poziomu GUI

Nagrywanie w interfejsie graficznym wymaga **otwartego projektu** (w przeciwnym razie przyciski nagrywania są nieaktywne):

* **Nagraj wszystko / Zatrzymaj wszystko** — u góry paska bocznego czujników światła; uruchamia lub zatrzymuje nagrywanie `.daq` na wszystkich podłączonych czujnikach jednocześnie.
* **Nagrywaj / Zatrzymaj nagrywanie** — dla poszczególnych czujników, w oknie modalnym ustawień (ikona koła zębatego). Podczas nagrywania w wierszach informacji na żywo czujnika wyświetla się czerwony wskaźnik „REC”.

Pliki są zapisywane w formacie `<project>/light_sensor/`, a po zatrzymaniu nagrywania — czy to poprzez przycisk „Zatrzymaj”, „Zatrzymaj wszystko”, czy też odłączenie czujnika nagrywającego — gotowy plik `.daq` jest **automatycznie dodawany do otwartego projektu**. Pojawia się on na liście plików projektu bez konieczności ręcznego dodawania, gotowy do przetwarzania odbicia.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Nagrywanie z poziomu CLI



CLI nagrywa za pośrednictwem puli czujników backendu (backend musi być uruchomiony — polecenia te są lekkimi klientami HTTP):

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

| Opcja | Wartość domyślna |
| --- | --- |
| `--duration` | `0` — rejestruj do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` w systemie plików **backendu**, a nie CLI |

Rozróżnienie katalogów wyjściowych ma znaczenie, gdy CLI kieruje dane do backendu na innym komputerze: plik trafia tam, gdzie działa backend. Gdy backend znajduje się na tym samym komputerze, ścieżka względna `-o` (taka jak `./out` powyżej) jest rozpoznawana względem folderu, w którym uruchomiono polecenie.

## Nagrywanie z języka Python

`DAQSensorSession` (zwracane przez `chloros_sdk.connect_daq_sensor()`) udostępnia to samo nagranie z puli: `record_start(output_dir=None, device_name=None)` zwraca ścieżkę do pliku, a `record_stop()` zwraca `{path, rows}`. Pełny opis API sesji znajduje się w [Dokumentacji SDK](../reference/sdk-reference.md). Klasy SDK obsługujące sprzęt bezpośrednio (tylko w instalacjach na komputerach stacjonarnych) domyślnie zapisują nagrania do `~/Documents/DAQ/`; w przypadku wydanych kompilacji obsługiwana jest powyższa ścieżka zbiorcza.

## Korzystanie z pliku .daq podczas przetwarzania

Aby obliczyć współczynnik odbicia na podstawie obrazów, program Chloros potrzebuje natężenia promieniowania padającego dopasowanego do każdej ekspozycji:

* **Należy zachować plik `.daq` wraz z obrazami.**Podczas przetwarzania potok automatycznie ustala**natężenie promieniowania padającego zgodne z sygnaturą czasową** na podstawie zarejestrowanego pliku `.daq` (dowolny model DAQ) — lub na podstawie natywnego pliku `.csv` — znajdującego się obok obrazów. Nagrania z interfejsu graficznego spełniają ten warunek automatycznie, ponieważ są dodawane do projektu w momencie ich zakończenia.
* **Kalibracja jest pobierana na żądanie.** Jeśli pakiet kalibracji fabrycznej dla danej kamery lub urządzenia DAQ nie został jeszcze zapisany w lokalnej pamięci podręcznej, program Chloros automatycznie pobiera go z chmury MAPIR przy pierwszym użyciu (wymagane jednorazowe połączenie z Internetem; zapisywany w pamięci podręcznej pod nazwą `~/.chloros/`).
* **Przechwytywane na żywo obrazy zapisują własny plik towarzyszący.** W przypadku każdej klatki odbicia światła zarejestrowanej na żywo faktycznie wykorzystany odczyt z modułu DAQ jest zapisywany jako plik `.daq` w folderze `daq/` danego zapisu, dzięki czemu zapis można później ponownie przetworzyć bez konieczności korzystania z oryginalnego nagrania.
* **Nagrania z rejestratorów `chloros_scripts` (wersja 1.3.0).**Plik `.daq` zapisany przez publiczne rejestratory [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) jest wymieniony jako model DAQ, który go utworzył, a jego limit można ustawić w**Ustawienia projektu → Czujnik światła DAQ**, tak jak w przypadku każdego innego nagrania. Rejestratory te zapisują surowe wartości zliczeń czujnika; Chloros pobiera fabryczną kalibrację czujnika przez port szeregowy i ją stosuje.
* **Czyje to odczyty (1.3.0).**Obok każdego zapisu `.daq` znajduje się `.daq.cameras.json`, który określa nazwę kamery (kamer), których klatki opisuje dany odczyt, a przetwarzanie dopasowuje go**wyłącznie do obrazów z tych kamer**. Samodzielna kamera bez własnego czujnika światła nie korzysta zatem już z odczytu innej kamery — należy przypisać do niej czujnik lub zarejestrować własny. Nagranie wykonane za pomocą samego DAQ (GUI Record, `daq pool-record`) oraz każdy plik `.daq` bez tego pliku (w tym starsze przechwyty) nadal obsługuje każdą kamerę.

## Odzyskiwanie natężenia promieniowania

Przetwarzanie projektu powoduje również wyeksportowanie wszystkich zapisów z czujników światła, jakie zawiera, do folderu
`Light Sensor/` obok produktów obrazowych. Nie wymaga to **obecności** obrazów:
samodzielny zapis z czujnika światła stanowi kompletny zapis, a folder zawierający wyłącznie pliki `.daq`
jest prawidłowym danymi wejściowymi. Projekt zawierający wyłącznie zapisy z czujników światła wyświetla ich listę i
przetwarza je zarówno w aplikacji, jak i w interfejsie CLI, a po zakończeniu przetwarzania pozostają one na liście. Raport z przetwarzania
podaje liczbę zapisanych produktów z czujników światła.

| Produkt | Co to jest |
| --- | --- |
| `<name>_calibrated.daq` | Archiwum nadające się do ponownego przetworzenia, zgodne ze schematem nagrania na żywo, zawierające teraz informacje o pakiecie kalibracyjnym, który je wygenerował. Ponowne zaimportowanie tego archiwum **nie** powoduje jego ponownej kalibracji. |
| `<name>_calibrated.csv` | Natężenie promieniowania spektralnego w W/m²/nm w siatce długości fal właściwej dla czujnika, jeden wiersz na odczyt, wraz z kolumnami fotometrycznymi: moc całkowita, luks fotopowy i skotopowy, PPFD z podziałem na niebieski/zielony/czerwony oraz długość fali szczytowej. |

Urządzenie DAQ-U lub DAQ-M, którego pakietu kalibracyjnego nie można pobrać — użytkownik jest w trybie offline lub
dla tego czujnika nie ma zapisanego pakietu kalibracyjnego — jest **pomijane z podaniem przyczyny** i nigdy nie jest zapisywane
jako plik „skalibrowany” zawierający surowe dane liczbowe. Należy połączyć się z Internetem i ponownie uruchomić proces. Urządzenie DAQ-E
posiada własną kalibrację, więc potrzebuje jej tylko wtedy, gdy urządzenie nie jest podłączone i
nic nie jest buforowane lokalnie.

### DAQ-A: surowe odczyty i dlaczego jest to właściwe rozwiązanie

**DAQ-A** powstał przed wprowadzeniem systemu pakietów kalibracyjnych dla poszczególnych numerów seryjnych i nie ma żadnego pakietu do
pobrania. Nie jest to przeoczenie: urządzenie DAQ-A jest kalibrowane w terenie względem
celu odbijającego, a kalibracja oparta na celu wymaga jedynie *względnej*
charakterystyki czujnika — a właśnie tym są jego surowe odczyty. Firma Chloros kalibruje urządzenia przy ich użyciu do dziś.

Zatem zapis z DAQ-A można wyeksportować, ale pod inną nazwą:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, a nie `_calibrated` — jest to inna nazwa pliku, a nie flaga wewnątrz pliku,
ponieważ informacja ta musi przetrwać wysłanie pliku pocztą elektroniczną jako sama nazwa. Nagłówek `.csv`
wskazuje `raw spectral sensor counts (NOT irradiance)` i ostrzega, że wartości są
porównywalne **w obrębie** pliku, a nie między czujnikami. Kolumny, które mają znaczenie
tylko w przypadku rzeczywistego natężenia promieniowania — moc całkowita, luks, PPFD — pozostają puste, a nie są
obliczane na podstawie liczb zliczeń.

Starsze zapisy DAQ-A-SD (schemat v1.01 / v1.02) rejestrują jedynie czas zapisu pliku, a nie
znacznik czasu dla każdego odczytu. Chloros nie będzie dopasowywać obrazów do tych danych — przyporządkowanie klatki do
czasu zapisu byłoby błędne, choć na pierwszy rzut oka nie wyglądałoby na błędne — ale eksport odczytuje je poprawnie, a
plik CSV wskazuje, na którym zegarze się znajdują.

Aby zapoznać się z pełnym opisem procesu pomiaru odbicia — z jednym czujnikiem i kamerą oraz z dwoma czujnikami (oświetlenie otoczenia/obiekt) — zobacz [Procesy pomiaru odbicia](reflectance.md).
