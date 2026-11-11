# Podłączanie kamer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Zakładka „Kamery” przed podłączeniem jakiegokolwiek urządzenia</p></figcaption></figure>

Chloros
automatycznie wykrywa kamery LATTICE w sieci — z poziomu zakładki „Kamery” w interfejsie graficznym, z pliku `chloros-cli lattice` lub z Python
owego SDK
. Ciąg znaków określający model kamery steruje wszystkimi dalszymi działaniami: Chloros
ustala profil czujnika, układ pasm i kalibrację fabryczną na podstawie plików `DeviceUserID` i `DeviceSerialNumber` kamery, więc **nie ma potrzeby konfigurowania poszczególnych kamer**.

Przed podłączeniem upewnij się, że sieć hosta jest skonfigurowana — adresowanie lokalne, ramki jumbo oraz, w przypadku macierzy, ustawienia bufora odbiorczego karty sieciowej. Jest to konfiguracja po stronie sprzętowej, opisana w instrukcji LATTICE: [**Konfiguracja sieci**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Łączenie się za pomocą interfejsu graficznego

Otwórz zakładkę **Kamery** na pasku bocznym Chloros
(zakładki sprzętowe pojawiają się po zakończeniu uruchamiania zaplecza) lub skorzystaj z menu głównego → **Połącz z kamerą**. Oba sposoby otwierają okno dialogowe**Połącz kamery**.

### Okno dialogowe **Podłącz kamerę (kamery)**Okno dialogowe skanuje sieć w momencie otwarcia („Skanowanie sieci...”) i wyświetla listę wszystkich znalezionych kamer. Każdy wiersz zawiera**model**kamery (np. `LATT-M3M-L41-F550`),**numer seryjny**oraz**adres IP**.

* **Kliknij wiersz, aby go zaznaczyć**(zielone podświetlenie). Można zaznaczyć**wiele kamer** i podłączyć je jednocześnie — program Chloros
podłącza je kolejno.
* Wiersze z oznaczeniem **„Podłączone”** są już podłączone i nie można ich ponownie zaznaczyć.
* Wiersze z oznaczeniem **„W macierzy”** należą do aktualnie podłączonej macierzy kamer. Aby używać tej kamery w trybie autonomicznym, należy najpierw odłączyć macierz.
* **Połącz** — łączy wybrane kamery; przycisk wyświetla liczbę, np. „Połącz (3)”, gdy wybrano więcej niż jedną kamerę.
* **Skanuj ponownie** — ponownie uruchamia skanowanie.
* **Zamknij** — zamyka okno dialogowe.
* Jeśli skanowanie zakończy się bez wyników, w oknie dialogowym pojawi się komunikat **„Nie znaleziono kamer w sieci”** — zobacz [Rozwiązywanie problemów](connecting.md#troubleshooting) poniżej.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Okno dialogowe „Podłącz kamerę(-y)” — pokazane tutaj bez kamer w sieci</p></figcaption></figure>

### Pierwsze podłączenie: pobieranie pakietu kalibracyjnego

Gdy dana kamera jest **podłączana po raz pierwszy** do urządzenia, Chloros
pobiera fabryczny pakiet kalibracyjny kamery (\~3,8 MB) bezpośrednio z kamery przez GigE. Podczas tej operacji w oknie dialogowym wyświetla się zielony panel **„Pobieranie danych kalibracyjnych z kamery”**z paskiem postępu dla każdego numeru seryjnego — należy się spodziewać, że proces ten potrwa około**70 sekund** na każdą kamerę. W przypadku więcej niż sześciu kamer wiersze postępu przewijają się wewnątrz panelu. Pakiet jest buforowany na hoście, więc przy kolejnych podłączeniach tej samej kamery pobieranie jest całkowicie pomijane (a panel nigdy się nie wyświetla).

### Analiza systemu

Przycisk **Analiza systemu** w oknie dialogowym przeprowadza diagnostykę hosta i sieci (podczas działania wyświetla się komunikat „Analiza...”) i generuje raport diagnostyczny:

* **Host** — rdzenie procesora i pamięć RAM; nazwa i pamięć karty graficznej lub komunikat „Karta graficzna: nie wykryto”.
* **Interfejsy sieciowe** — nazwa każdej karty sieciowej, prędkość łącza, MTU (z oznaczeniem „jumbo”, jeśli jest aktywne), stan (aktywny/nieaktywny) oraz informacja, czy znajduje się na magistrali USB.
* **Kamery**— numer seryjny, model, adres IP oraz**informacja, do której karty sieciowej podłączona jest dana kamera**.
* **Wydajność** — aktualna i idealna liczba klatek na sekundę (fps) dla każdej kamery w danym formacie pikseli, z zielonym napisem „Potencjał: możliwa poprawa o N×”, gdy wartość idealna przewyższa aktualną.
* **Ostrzeżenia i ponumerowane zalecenia** — lub komunikat „System działa poprawnie przy obecnej liczbie kamer”, gdy nie ma nic do naprawienia.

Uruchom to narzędzie zawsze, gdy wykrywanie lub przesył strumieniowy zachowuje się nieoczekiwanie — pozwala ono zidentyfikować większość problemów po stronie kart sieciowych (nieprawidłowa wartość MTU, kamera podłączona do niewłaściwego interfejsu, ograniczenia adaptera USB) bez opuszczania okna dialogowego.

### Podłączanie macierzy

Aby podłączyć dwie lub więcej kamer jako **zsynchronizowaną macierz**, należy zamiast tego skorzystać z kreatora podłączania macierzy (**Podłącz macierz kamer**): przeprowadza on użytkownika przez wybór roli master/slave (wstępnie wypełniony przez sondę kabla synchronizacyjnego — każda podłączona kablem kamera może pełnić rolę master, a kamera RGB
`-FRGB` jest wstępnie wybrana po podłączeniu kabla), wybór trybu wyświetlania (kafelki oddzielne lub połączone) oraz scenę ustawień macierzy z podglądem na żywo możliwej liczby klatek na sekundę (fps) i przepustowości łącza przed zatwierdzeniem. Kreator i procedury pracy z układem kamer zostały omówione w sekcji poświęconej układom wielokamerowym niniejszej instrukcji; odpowiednikiem w CLI
jest „Procedura pierwszego podłączenia kamery LATTICE” w [Podręczniku CLI
](../reference/cli-reference.md).

## Łączenie się za pomocą CLI
i SDK
Dostęp do

CLI
i SDK
wymaga płatnego planu Chloros
+ oraz zalogowania się; wymóg ten jest egzekwowany po stronie serwera (`401 AUTH_REQUIRED` w przypadku braku logowania, `403 PLAN_UPGRADE_REQUIRED` w przypadku planu bezpłatnego).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Pełne sygnatury, opcje i procesy przechwytywania: [Dokumentacja CLI
](../reference/cli-reference.md) § `chloros-cli lattice`, [Dokumentacja SDK
](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Jak przebiega kalibracja po nawiązaniu połączenia

Każda kamera LATTICE posiada fabryczny pakiet kalibracyjny **wbudowany w kamerę**, a Chloros
sprawdza również chmurę MAPIR
po podłączeniu kamery:

| Sytuacja   | Co wykorzystuje Chloros
                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Najnowsza kalibracja opublikowana dla tego numeru seryjnego** — kopia w chmurze ma pierwszeństwo przed kopią w kamerze. Kamera, która została ponownie skalibrowana lub zaktualizowana przez MAPIR
, aktualizuje się zatem automatycznie; nie jest wymagana żadna interwencja użytkownika. |
| **Offline**|**Pakiet w aparacie** w obecnej postaci. Procesy pracy w trybie całkowicie offline nadal działają; po prostu nie pobierają nowszych kalibracji, dopóki aparat nie połączy się z internetem (lub nie zostanie ponownie zaktualizowany do ustawień fabrycznych).                                                  |

W momencie przechwytywania faktycznie zastosowane współczynniki są **utrwalane w metadanych XMP każdego obrazu**. Późniejsza aktualizacja kalibracji nigdy nie zmienia w sposób niewidoczny już zarejestrowanych obrazów — ponowne przetwarzanie starego ujęcia wykorzystuje współczynniki zapisane w jego metadanych XMP, a nie te, które są aktualnie najnowsze.

## Rozwiązywanie problemów

* **„Nie znaleziono kamer w sieci”**— sprawdź konfigurację link-local w [Konfiguracji sieci](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statyczny adres karty sieciowej hosta `169.254.x.x/16`, kamery na tym samym łączu, nie oczekuje się DHCP ani bramy. Następnie użyj opcji**Analiza systemu**w oknie dialogowym połączenia, aby sprawdzić, na której karcie sieciowej każda kamera jest (lub nie jest) widoczna. Po każdej zmianie okablowania lub karty sieciowej należy wykonać**ponowne skanowanie**.
* **Układ, który wcześniej działał, nie chce się połączyć** (panele macierzy z komunikatami `FRAMES WILL DROP` / `Reduce ROI to enable`) — aktualizacja sterownika karty sieciowej w sposób niewidoczny zresetowała ustawienia pierścienia odbiorczego. Zastosuj je ponownie lub uruchom polecenie `chloros-cli lattice network --fix` z terminala z uprawnieniami administratora; zobacz [Konfiguracja sieci](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera wyświetla komunikat „In Array”** — należy ona do podłączonej sesji macierzy. Odłącz macierz, aby korzystać z kamery w trybie autonomicznym.
