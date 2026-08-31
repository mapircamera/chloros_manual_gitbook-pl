# Zakładka „DAQ” w Chloros

Zakładka DAQ — oznaczona jako **Czujniki światła** na pasku bocznym Chloros — stanowi interfejs sterowania na żywo dla [czujników światła DAQ-U, DAQ-M i DAQ-E](README.md): podłącz czujniki za pomocą dowolnego protokołu, obserwuj skalibrowane widma w czasie rzeczywistym, obliczaj współczynnik odbicia na żywo na podstawie pary czujników oraz zapisuj pliki `.daq` bezpośrednio do projektu.

Zakładka staje się dostępna po zakończeniu uruchamiania zaplecza Chloros. Wykresy na tej karcie są zasilane przez usługę DAQ Chloros za pośrednictwem połączenia na żywo, które w razie przerwy łączy się ponownie automatycznie (czas oczekiwania 2–10 s); gdy usługa jest niedostępna, w wierszu „Status” czujnika wyświetla się komunikat **Brak serwera**.

Układ składa się z **paska bocznego czujników**(jeden wiersz na każdy podłączony czujnik) oraz**obszaru wykresów** (jedna kafelka wykresu na czujnik lub grupę).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Podłączanie czujnika

Kliknij **Połącz czujnik** u góry paska bocznego. Okno dialogowe połączenia otworzy się w obszarze głównym (lub jako nakładka podczas dodawania kolejnego czujnika — w takim przypadku pojawi się przycisk Anuluj).

| Element sterujący | Działanie |
| --- | --- |
| **Typ urządzenia** | `DAQ-U (USB)` (domyślnie), `DAQ-M (Bluetooth)` lub `DAQ-E (Ethernet)`. Zmiana typu powoduje ponowne uruchomienie skanowania w poszukiwaniu nowo wybranego protokołu transmisji. |
| **Port / Urządzenie BLE / Nazwa hosta / Adres IP** | Wyświetla listę wykrytych urządzeń jako `device - description`; automatycznie wybierany jest pierwszy wpis rozpoznany jako czujnik. Podczas skanowania wyświetla się `Scanning...` (USB), `Scanning (N)...` z 8-sekundowym odliczaniem (BLE) lub `Discovering ethernet sensors (N)...` z 5-sekundowym odliczaniem (Ethernet). Puste wyniki wyświetlane są jako `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Odśwież** | Natychmiast ponownie skanuje wybrany protokół (funkcja wyłączona w trakcie skanowania BLE/Ethernet). |
| **Połącz** | Opcja dostępna po wybraniu urządzenia; podczas nawiązywania połączenia nazwa zmienia się na `Connecting...`. |

Wykrywanie działa tylko **gdy na ekranie wyświetlany jest okno dialogowe połączenia** i powtarza się co 15 sekund wyłącznie dla wybranego protokołu — samo otwarcie karty nie powoduje skanowania. W przypadku niepowodzenia w oknie dialogowym pojawia się komunikat: *„Połączenie nie powiodło się. Spróbuj odłączyć i ponownie podłączyć czujnik, a następnie kliknij ponownie przycisk Połącz.”*

Pasek boczny otwiera się automatycznie po podłączeniu pierwszego czujnika.

{% hint style="info" %}
**DAQ-E nie wyświetla się?** Urządzenie DAQ-E nie posiada diody LED stanu — sprawdź wskaźnik PoE/połączenia na przełączniku lub porcie iniektora, do którego jest podłączone, i odczekaj kilka sekund po włączeniu zasilania, aż urządzenie się uruchomi. Urządzenie Chloros musi znajdować się w tej samej domenie rozgłoszeniowej (mDNS nie przechodzi przez routery). Na urządzeniu Windows zaakceptuj monit zapory Defender przy pierwszym połączeniu gniazd multiemisji przez Chloros (mDNS UDP 5353, dane DAQ-E UDP 5002, PTP UDP 319/320). Dwa urządzenia DAQ-E w tej samej sieci LAN są wykrywane oddzielnie, każde pod własną nazwą hosta `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Typ urządzenia obejmuje DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet)</figcaption></figure>***

## Pasek boczny czujników

Każdy podłączony czujnik ma swój wiersz (plus jeden wiersz na grupę „Otoczenie+Obiekt”). Wiersze można zmieniać kolejno poprzez przeciąganie, a ich kolejność wpływa również na kolejność kafelków wykresów. Kliknij wiersz, aby ustawić ten czujnik/grupę jako aktywny wykres w widoku listy.

| Element | Znaczenie |
| --- | --- |
| Kolorowa lewa ramka | Kolor wykresu czujnika. |
| Ikona transportu | `DAQ-U` / `DAQ-M` / `DAQ-E` lub zielona plakietka `REF` dla grupy współczynnika odbicia Ambient+Object. |
| Nazwa urządzenia | Domyślnie jest to numer seryjny czujnika (jego stały identyfikator do celów kalibracji, nazw plików `.daq` oraz dopasowywania podczas importu); nazwy niestandardowe są zachowywane dla poszczególnych projektów. |
| Piktogram **Skalibrowano** (zielony) | Wyświetlany, gdy załadowany jest fabryczny pakiet kalibracyjny czujnika, tzn. widma są podane w jednostkach W/m²/nm. |
| Ikona **Dostępna aktualizacja** (bursztynowa, tylko DAQ-E) | Bieżące oprogramowanie sprzętowe jest starsze niż obraz dołączony do tej kompilacji Chloros. Podczas aktualizacji wyświetla bieżący postęp (`Flashing… N%`, `Restarting sensor…`, a następnie `Updated X → Y` lub `Failed`). |
| Oko | Włącza lub wyłącza widoczność tego czujnika na wykresie. |
| Koło zębate | Otwiera okno modalne ustawień poszczególnych czujników (poniżej). |
| ✕ (czerwone) | Odłącza czujnik lub usuwa grupę „Ambient+Object”. |

Nad wierszami znajdują się dwa przyciski:

* **Podłącz czujnik** — otwiera okno dialogowe podłączania (podczas działania nazwa zmienia się na `Connecting...`).
* **Nagraj wszystko / Zatrzymaj wszystko**— uruchamia lub zatrzymuje nagrywanie `.daq` na**każdym**podłączonym czujniku. Wymaga co najmniej jednego czujnika**oraz otwartego projektu** (podpowiedź: „Otwórz projekt, aby nagrywać”); podczas trwania nagrywania przycisk zmienia kolor na czerwony.

W stanie pustym wyświetla się komunikat „Brak podłączonych czujników”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Ustawienia poszczególnych czujników (okno modalne z ikoną koła zębatego)

Otwiera się po kliknięciu ikony koła zębatego w wierszu czujnika. Zawartość w kolejności:

* **Wiersze informacyjne** — Typ urządzenia (DAQ-U/M/E), Połączenie (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (port COM, adres BLE lub host) oraz numer seryjny.
* **Raport kalibracji: Pobierz** — pobiera certyfikat kalibracji tego urządzenia zgodny z normami NIST (PDF) i otwiera go w przeglądarce plików PDF. Opcja dostępna po podaniu numeru seryjnego; certyfikat jest zapisywany w pamięci podręcznej przy pierwszym połączeniu.
* **Nazwa urządzenia** — kliknij ołówek, aby zmienić nazwę; nazwa jest zachowywana dla danego projektu.
* **Kolor linii wykresu** — próbka koloru; ustawienie zachowuje się dla danego projektu.
* **Czas integracji (ms)**— suwak + liczba,**1–500 ms**, domyślnie**32 ms**. Wyłączone, gdy funkcja AE jest włączona.
* **Średnia klatek**— suwak + liczba,**1–50 klatek**, domyślnie**20**.
* **AE: WŁ./WYŁ.**— przełącznik automatycznej ekspozycji;**domyślnie włączony** po podłączeniu. Wyłącz, aby ręcznie ustawić czas integracji.
* **Zatrzymaj transmisję / Rozpocznij transmisję** — wstrzymaj lub wznow transmisję na żywo.
* **Nagrywaj / Zatrzymaj nagrywanie** — nagrywanie z czujnika `.daq` (wymaga otwartego projektu).
* **Cap** — profil korekcji cap (następna sekcja).
* **Wiersze informacji na żywo** — czas integracji (ms), liczba klatek na sekundę (FPS), próbki, nagrywanie (czerwony `REC` lub `Off`) oraz stan (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Tylko DAQ-E: wiersze dotyczące sieci, oprogramowania sprzętowego i PTP

* **Nazwa hosta / adres IP** — aktualny adres urządzenia.
* **Oprogramowanie układowe** — aktualna wersja oprogramowania układowego wraz z komórką akcji:<version\>

przycisk</version\>

**Zaktualizuj do \<version\>** pojawia się</version\>

,<version\>

gdy ta kompilacja Chloros zawiera nowszy obraz oprogramowania układowego DAQ-E. Aktualizacja jest instalowana przez sieć w około 30 sekund; czujnik uruchamia się ponownie i automatycznie łączy się ponownie, a przerwany transfer pozostawia aktualne oprogramowanie układowe bez zmian. Postęp aktualizacji jest wyświetlany na bieżąco (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), a komórka wyświetla wartość `Up to date`, gdy jest aktualna.
* **Synchronizacja PTP** — aktualny stan PTP (przywraca się do `unknown`). Oprogramowanie sprzętowe DAQ-E w wersji 1.2.0+ uczestniczy w standardzie IEEE 1588 PTPv2 jako zegar wyłącznie w trybie slave; serwer Chloros pełni rolę grandmastera PTP, a każde urządzenie DAQ-E i kamera LATTICE w sieci LAN są jego urządzeniami podrzędnymi w domenie 0, utrzymując zbieżność znaczników czasu z dokładnością do około 1 ms.

W przypadku grupy „Ambient+Object” okno modalne sprzętu wyświetla wyłącznie czujniki źródłowe tej grupy, nazwę urządzenia oraz kolor linii wykresu.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Wybór nasadki

Lista rozwijana **Osłona** informuje Chloros, która fizyczna osłona jest zamontowana na dyfuzorze czujnika, i stosuje do każdego widma profil korekcyjny tej osłony zmierzony fabrycznie. Dostępne opcje zależą od modelu:

| Model | Opcje osłon |
| --- | --- |
| DAQ-U | Brak (niedotknięty czujnik), pole widzenia 15°, pole widzenia 30°, pole widzenia 45°, pole widzenia 60°, pole widzenia 90°, Sunshine (korektor cosinusowy) |
| DAQ-M | Brak (nagie czujniki), Sunshine (korektor cosinusowy) |
| DAQ-E | Brak (nagie czujniki), pole widzenia 15°, pole widzenia 45°, pole widzenia 90°, Sunshine (korektor cosinusowy) |

**Domyślnym ustawieniem dla każdego modelu jest Sunshine (korektor cosinusowy)** — firma MAPIR dostarcza każdy moduł DAQ z zamontowaną nasadką Sunshine, co stanowi standardową konfigurację do użytku na zewnątrz: polowe widzenie półkuliste o kącie 180°, z błędem cosinusowym ≤ ±4 % do 60° i ≤ ±4,5 % do 70° (niezalecane przy elewacji Słońca poniżej ~15°), z konstrukcyjnym tłumieniem (~12×). Wybrany przez użytkownika parametr pozostaje zachowany w projekcie.

{% hint style="warning" %}
**Wybór nasadki musi odpowiadać fizycznej nasadce.**Ani czujnik, ani oprogramowanie nie są w stanie wykryć, która nasadka jest zamontowana. Wybór ten determinuje zarówno korekcję na żywo, jak i sygnaturę zapisywanej w każdym pliku `.daq` — przy ~12-krotnym tłumieniu nasadki Sunshine niezgłoszona zmiana nasadki powoduje błędną korekcję widm o mniej więcej ten współczynnik. (Zdejmowanie i ponowne zamontowanie tej samej nasadki powoduje powtórzenie błędu na poziomie około 1,5 %.) Opcję**Brak (nagi czujnik)** należy wybrać tylko wtedy, gdy nasadka jest fizycznie zdjęta; w przypadku urządzenia DAQ-E opcja „Brak” nadal stosuje fabryczny profil geometryczny dla jego wpuszczanego szklanego dyfuzora — nie jest to operacja bezczynna — a nagi czujnik DAQ-E stanowi konfigurację laboratoryjną, a nie obsługiwany w terenie.
{% endhint %}

{% hint style="info" %}
Aktualizacja z wcześniejszej instrukcji: przełącznik „Sunshine Diffuser Installed” po stronie przeglądarki, obecny w wersji 1.1.0, został usunięty. Obsługa osłon odbywa się teraz poprzez profil osłony dla poszczególnych czujników, stosowany po stronie serwera.
{% endhint %}

***

## Obszar wykresów

Przyklejony górny pasek zawiera **przełącznik widoku listy ⇄ siatki**oraz suwak**powiększenia wykresu** (rozmiar kafelka 200–2000 pikseli). Widok automatycznie przełącza się na siatkę, gdy istnieje więcej niż jedna grupa wykresów, a wraca do listy, gdy jest jedna lub mniej. Tryb wyświetlania i rozmiar wykresu są zachowywane dla każdego projektu.**Wykres widma** dla każdego czujnika pokazuje:

* **Oś X** — długość fali (nm). Siatka czujnika obejmuje zakres 340–1010 nm z krokiem 5 nm (135 punktów), interpolowanym do 1 nm na potrzeby wyświetlania.
* **Oś Y** — moc (W/m²), z automatycznym przedrostkiem SI (m/µ/n) wybranym na podstawie wartości szczytowej. Widma są kalibrowane radiometrycznie jako natężenie promieniowania spektralnego (W/m²/nm) we wszystkich trzech systemach transportu danych.
* Tło widma w kolorach tęczy pod pojedynczą krzywą; wiele czujników na jednym wykresie nakłada się jako kolorowe linie z przyciemnionym tłem.
* **Najechanie kursorem**— pionowy kursor z długością fali i wartością dla danego czujnika;**przeciągnij**, aby powiększyć (podczas powiększenia pojawia się przycisk pomniejszenia).
* Przycisk **+** (tylko w widoku siatki) służy do dodania czujnika do tego wykresu lub utworzenia grupy (poniżej).
* Nazwa urządzenia wyśrodkowana u góry oraz ikona ładowania do momentu pojawienia się pierwszej klatki.

**Nasycenie** nie jest zaznaczone na samym wykresie: nasycony czujnik wyświetla czerwony tekst statusu `SATURATED` oraz czerwony wiersz `Saturated: Yes` w tabeli danych na żywo. Aby to skasować, skróć czas integracji lub ponownie włącz AE.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabela danych na żywo (widok listy)

Poniżej wykresu w widoku listy, odświeżana co 500 ms:

* **Wszystkie modele**: Próbka koloru światła (sRGB z CIE XYZ), Nasycony (Tak/Nie), CIE 1931 X/Y/Z, Chromatyczność x/y, CIE u′/v′, CCT (K), CRI (Ra), Dominująca długość fali (nm), długość fali szczytowej (nm), czystość wzbudzenia, Duv, CIE L\*/a\*/b\* oraz Munsell H/V/C.
* **Tylko czujniki skalibrowane**(dowolny model DAQ-U / DAQ-M / DAQ-E po załadowaniu fabrycznego pakietu kalibracyjnego — rozpoznaje się to po zielonej plakietce**Skalibrowany** w wierszu czujnika): Moc całkowita (W/m²), luks fotopowy (lx), luks skotopowy (lx), stosunek S/P, PPFD oraz PPFD Red/Green/Blue (µmol/m²/s) oraz natężenia promieniowania opicznego — czopki S, melanopowe, rodopowe, czopki M, czopki L (wszystkie w W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupy odbicia (otoczenie + obiekt)

Dwa połączone czujniki można połączyć w celu wyświetlania odbicia na żywo — bez użycia kamery:

1. W widoku siatki kliknij **+**na kafelku wykresu i wybierz**Połącz otoczenie + obiekt**.
2. Wybierz czujnik **Źródło światła otoczenia**oraz czujnik**Skaner obiektu**(dwa różne czujniki), a następnie**Utwórz**.

Chloros oblicza R(λ) = obiekt(λ) / otoczenie(λ) dla każdej długości fali na podstawie dwóch strumieni danych na żywo (0, gdy oświetlenie otoczenia ≤ 0). Etykieta grupy zależy od klasy kalibracji czujników:

* Oba czujniki skalibrowane (pakiet załadowany) → **„Odbicie pozorne”**.
* Chociażby jeden czujnik niekalibrowany → **„Współczynnik odbicia względnego”**.

Grupa pojawia się jako zielony wiersz `REF` na pasku bocznym oraz na własnym wykresie (wypełnienie tęczowe, wartości wyświetlane po najechaniu kursorem z dokładnością do 4 miejsc po przecinku, powiększanie przez przeciągnięcie).

Menu **+**oferuje również opcję**Dodaj nowy czujnik** z trzema możliwościami umieszczenia: *Połącz nowy czujnik* (dołącz do tego wykresu), *Przenieś tutaj istniejący czujnik* lub *Wyświetl nowy czujnik* (własny wykres).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabela wskaźników wegetacyjnych

W widoku listy poniżej wykresu grupy odbicia znajduje się tabela wskaźników wegetacyjnych, obliczona na podstawie aktualnego odbicia w centrach pasm **niebieski 450 / zielony 550 / czerwony 670 / NIR 800 nm** (wartości z dokładnością do 4 miejsc po przecinku, `---`, gdy nie da się obliczyć; najedź kursorem na nazwę wskaźnika, aby zobaczyć jego pełną nazwę):

* **Zawsze wyświetlane** (niezależne od skali, dowolna kombinacja czujników): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Tylko wtedy, gdy oba czujniki są skalibrowane pod kątem mocy** (oba zestawy załadowane): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Nagrywanie plików `.daq`

* Nagrywanie wymaga **otwartego projektu** — w przeciwnym razie zarówno opcja „Nagraj wszystko” (na pasku bocznym), jak i przycisk nagrywania dla poszczególnych czujników są nieaktywne.
* Pliki są zapisywane w formacie **`<project folder>/light_sensor/`**; nazwy plików zawierają identyfikator czujnika i sygnaturę czasową, a nazwa urządzenia jest zapisywana wraz z nagraniem.
* Po zakończeniu nagrywania (przycisk „Stop”, „Stop All” lub rozłączenie w trakcie nagrywania) gotowy plik `.daq` jest **automatycznie dodawany do otwartego projektu** — pojawia się on na liście plików projektu bez konieczności ręcznego dodawania, gotowy do wykorzystania jako dane natężenia promieniowania padającego do [przetwarzania współczynnika odbicia](README.md).
* Podczas nagrywania w wierszach na żywo okna modalnego ustawień wyświetla się czerwony wskaźnik `REC`.

W przypadku ilościowych wartości natężenia promieniowania należy uśrednić dane z co najmniej 15 sekund — jest to cecha charakterystyczna urządzenia, a nie wada.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Układy wieloczujnikowe i trwałość projektu

* Łącz kilka czujników na jednym wykresie (wspólne osie), utrzymuj oddzielne wykresy (automatyczny układ siatki), przenoś czujniki między wykresami, zmieniaj kolejność wierszy/kafelków poprzez przeciąganie oraz ukrywaj poszczególne czujniki za pomocą przełącznika w postaci ikony oka.
* W ramach każdego projektu zachowywane są następujące ustawienia Chloros: nazwy urządzeń, kolory wykresów, rozmiar wykresu, tryb wyświetlania oraz ustawienia każdego czujnika (czas integracji, uśrednianie klatek, stan AE, wybór ograniczenia).
* **Ponowne otwarcie projektu powoduje automatyczne ponowne połączenie czujników** na podstawie adresu — portu COM dla DAQ-U, urządzenia BLE dla DAQ-M, nazwy hosta mDNS dla DAQ-E (rozpoznawane nawet w przypadku zmiany adresu IP urządzenia) — oraz ponowne zastosowanie zapisanego profilu nakładki, uśredniania klatek, stan AE oraz ręczny czas integracji.***

## Parowanie kamery (DLS)

Nie ma potrzeby parowania. W przeciwieństwie do procesów DLS w dronach, które z góry przypisują czujnik światła do kamery, Chloros dopasowuje dane DAQ do obrazów na dalszym etapie: podczas importu/przetwarzania odczyty `.daq` są interpolowane do znacznika czasu ekspozycji każdego ujęcia. Nagrywaj za pomocą dowolnego podłączonego czujnika (`.daq` automatycznie trafia do projektu), a przetwarzanie współczynnika odbicia znajdzie odpowiednie odczyty według czasu — zobacz [Czujniki światła DAQ](README.md), aby dowiedzieć się, w jaki sposób wykorzystywane są dane dotyczące promieniowania padającego.</version\>
