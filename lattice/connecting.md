# Podłączanie kamer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Zakładka „Kamery” przed podłączeniem jakiegokolwiek urządzenia</p></figcaption></figure>Chloros automatycznie wykrywa kamery LATTICE w sieci — z poziomu zakładki „Kamery” w interfejsie graficznym, z `chloros-cli lattice` lub z Python SDK. Ciąg znaków określający model kamery determinuje wszystkie dalsze działania: Chloros ustala profil czujnika, układ pasm i kalibrację fabryczną na podstawie parametrów kamery `DeviceUserID` + `DeviceSerialNumber`, więc **nie ma potrzeby konfigurowania poszczególnych kamer**.

Przed podłączeniem upewnij się, że sieć hosta jest skonfigurowana — adresowanie lokalne, ramki jumbo oraz, w przypadku macierzy, ustawienia bufora odbiorczego karty sieciowej. Jest to konfiguracja po stronie sprzętowej, opisana w instrukcji obsługi LATTICE: [**Konfiguracja sieci**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Łączenie się z poziomu GUI

Otwórz zakładkę **Kamery**na pasku bocznym Chloros (zakładki sprzętowe pojawiają się po zakończeniu uruchamiania serwisu) lub skorzystaj z menu głównego →**Połącz z kamerą**. Oba sposoby otwierają okno dialogowe**Połącz kamerę(-y)**.

### Okno dialogowe **Podłącz kamerę(-y)**Okno dialogowe skanuje sieć w momencie otwarcia („Skanowanie sieci...”) i wyświetla listę wszystkich znalezionych kamer. Każdy wiersz zawiera**model**kamery (np. `LATT-M3M-L41-F550`),**numer seryjny**oraz**adres IP**.

* **Kliknij wiersz, aby go zaznaczyć**(zielone podświetlenie). Możesz zaznaczyć**wiele kamer** i połączyć je jednocześnie — Chloros łączy je po kolei.
* Wiersze z oznaczeniem **„Podłączone”** są już podłączone i nie można ich ponownie zaznaczyć.
* Wiersze z oznaczeniem **„W macierzy”** należą do aktualnie podłączonej macierzy kamer. Aby używać tej kamery samodzielnie, należy najpierw odłączyć macierz.
* **Podłącz** — podłącza zaznaczone kamery; przycisk wyświetla liczbę, np. „Podłącz (3)”, gdy zaznaczono więcej niż jedną kamerę.
* **Skanuj ponownie** — ponownie uruchamia skanowanie.
* **Zamknij** — zamyka okno dialogowe.
* Jeśli skanowanie zakończy się bez wyników, w oknie dialogowym pojawi się komunikat **„Nie znaleziono kamer w sieci”** — zobacz [Rozwiązywanie problemów](connecting.md#troubleshooting) poniżej.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Okno dialogowe „Podłącz kamerę(-y)” — pokazane tutaj w sytuacji, gdy w sieci nie ma żadnych kamer</p></figcaption></figure>### Pierwsze podłączenie: pobieranie pakietu kalibracyjnego

Gdy dana kamera jest **podłączana po raz pierwszy**do urządzenia, Chloros pobiera fabryczny pakiet kalibracyjny kamery (\~3,8 MB) bezpośrednio z samej kamery przez GigE. Podczas tego procesu w oknie dialogowym wyświetla się zielony panel**„Pobieranie danych kalibracyjnych z kamery”**z paskiem postępu dla każdego numeru seryjnego — należy spodziewać się około**70 sekund** na każdą kamerę. Pakiet jest buforowany na hoście, więc przy kolejnych podłączeniach tej samej kamery pobieranie jest całkowicie pomijane (a panel nigdy nie jest wyświetlany).

### Analiza systemu

Przycisk **Analiza systemu** w oknie dialogowym przeprowadza diagnostykę hosta i sieci (podczas działania wyświetla się komunikat „Analiza w toku...”) i generuje raport diagnostyczny:

* **Host** — rdzenie procesora i pamięć RAM; nazwa i pamięć karty graficznej (GPU) lub komunikat „GPU: Nie wykryto”.
* **Interfejsy sieciowe** — nazwa każdej karty sieciowej, prędkość łącza, MTU (z oznaczeniem „jumbo”, jeśli jest aktywne), stan aktywny/nieaktywny oraz informacja, czy znajduje się na magistrali USB.
* **Kamery**— numer seryjny, model, adres IP oraz**na której karcie sieciowej podłączona jest każda kamera**.
* **Wydajność** — aktualna liczba klatek na sekundę (fps) w porównaniu z idealną dla danego formatu pikseli dla każdej kamery, z zieloną linią „Potencjał: możliwe N-krotne przyspieszenie”, gdy wartość idealna przewyższa aktualną.
* **Ostrzeżenia i ponumerowane zalecenia** — lub komunikat „System działa poprawnie przy bieżącej liczbie kamer”, gdy nie ma nic do naprawienia.

Uruchom to narzędzie zawsze, gdy wykrywanie lub przesył strumieniowy zachowuje się nieoczekiwanie — identyfikuje ono większość problemów po stronie kart sieciowych (nieprawidłowa wartość MTU, kamera podłączona do niewłaściwego interfejsu, ograniczenia adaptera USB) bez konieczności opuszczania okna dialogowego.

### Podłączanie macierzy

Aby podłączyć dwie lub więcej kamer jako **zsynchronizowaną macierz**, należy zamiast tego użyć kreatora podłączania macierzy (**Podłącz macierz kamer**): kroczy on przez wybór konfiguracji master/slave (wstępnie wypełniony przez sondę okablowania GPIO), wybór trybu wyświetlania (kafelki oddzielne vs. połączone) oraz scenę ustawień macierzy z podglądem na żywo możliwej liczby klatek na sekundę (fps) i przepustowości łącza przed zatwierdzeniem. Kreator i procedury pracy z układem kamer zostały omówione w sekcji poświęconej układom wielokamerowym w niniejszej instrukcji; odpowiednikiem dla modelu CLI jest „Procedura pierwszego podłączenia kamery LATTICE” w [Podręczniku CLI](../reference/cli-reference.md).

## Łączenie się z CLI i SDK

Dostęp do CLI i SDK wymaga płatnego planu Chloros+ oraz zalogowania się; jest to egzekwowane po stronie serwera (`401 AUTH_REQUIRED` w przypadku braku zalogowania, `403 PLAN_UPGRADE_REQUIRED` w ramach bezpłatnego planu).

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

Pełne sygnatury, opcje i przepływy pracy związane z przechwytywaniem: [CLI Dokumentacja](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK Odniesienie](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Jak przebiega kalibracja po podłączeniu

Każda kamera LATTICE posiada fabryczny pakiet kalibracyjny **wbudowany w kamerę**, a model Chloros sprawdza również chmurę modelu MAPIR po podłączeniu kamery:

| Sytuacja   | Co wykorzystuje model Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Najnowsza kalibracja opublikowana dla tego numeru seryjnego** — kopia w chmurze ma pierwszeństwo przed kopią w aparacie. Aparat, który został ponownie skalibrowany lub zaktualizowany przez MAPIR, aktualizuje się zatem automatycznie; nie jest wymagana żadna interwencja użytkownika. |
| **Offline**|**Pakiet w aparacie** pozostaje bez zmian. Procesy pracy w trybie całkowicie offline nadal działają; po prostu nie pobierają nowszych kalibracji, dopóki aparat nie połączy się z internetem (lub nie zostanie przywrócony do ustawień fabrycznych).                                                  |

W momencie przechwytywania faktycznie zastosowane współczynniki są **utrwalane w metadanych XMP każdego obrazu**. Późniejsza aktualizacja kalibracji nigdy nie zmienia w sposób niezauważalny już zarejestrowanych zdjęć — ponowne przetwarzanie starego zdjęcia wykorzystuje współczynniki zapisane w jego metadanych XMP, a nie te, które są aktualnie najnowsze.

## Rozwiązywanie problemów

* **„Nie znaleziono kamer w sieci”**— sprawdź konfigurację lokalnego łącza w [Konfiguracji sieci](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statyczny adres karty sieciowej hosta `169.254.x.x/16`, kamery na tym samym łączu, nie oczekuje się DHCP ani bramy. Następnie użyj opcji**Analiza systemu**w oknie dialogowym połączenia, aby sprawdzić, na której karcie sieciowej każda kamera jest (lub nie jest) widoczna. Po każdej zmianie okablowania lub karty sieciowej należy wykonać**ponowne skanowanie**.
* **Układ, który wcześniej działał, nie chce się połączyć** (bramki panelu macierzy z kodami `FRAMES WILL DROP` / `Reduce ROI to enable`) — aktualizacja sterownika karty sieciowej w sposób niewidoczny zresetowała ustawienia pierścienia odbiorczego. Zastosuj je ponownie lub uruchom polecenie `chloros-cli lattice network --fix` z terminala z uprawnieniami administratora; zobacz [Konfiguracja sieci](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera wyświetla komunikat „In Array”** — należy ona do podłączonej sesji macierzy. Odłącz macierz, aby korzystać z kamery w trybie autonomicznym.
