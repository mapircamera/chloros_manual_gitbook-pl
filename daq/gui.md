# Zakładka „DAQ” w programie Chloros

Zakładka „DAQ” — oznaczona jako **Czujniki światła** na pasku bocznym programu Chloros — stanowi interfejs sterowania na żywo dla [czujników światła DAQ-U, DAQ-M i DAQ-E](README.md): podłącz czujniki za pomocą dowolnego protokołu, obserwuj skalibrowane widma w czasie rzeczywistym, obliczaj współczynnik odbicia na żywo na podstawie pary czujników oraz zapisuj pliki `.daq` bezpośrednio do swojego projektu.

Zakładka staje się dostępna po zakończeniu uruchamiania zaplecza Chloros. Wykresy na tej zakładce są zasilane przez usługę DAQ Chloros za pośrednictwem połączenia na żywo, które w razie przerwy łączy się ponownie automatycznie (czas oczekiwania 2–10 s); gdy usługa jest niedostępna, w wierszu Status czujnika wyświetla się komunikat **Brak serwera**.

Układ składa się z **paska bocznego czujników**(jeden wiersz na każdy podłączony czujnik) oraz**obszaru wykresów** (jedna kafelka wykresu na czujnik lub grupę).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Podłączanie czujnika

Kliknij przycisk **Podłącz czujnik** u góry paska bocznego. Okno dialogowe podłączania otworzy się w obszarze głównym (lub jako nakładka podczas dodawania kolejnego czujnika — w takim przypadku pojawi się przycisk Anuluj).

| Element sterujący | Działanie |
| --- | --- |
| **Typ urządzenia** | `DAQ-U (USB)` (domyślnie), `DAQ-M (Bluetooth)` lub `DAQ-E (Ethernet)`. Zmiana powoduje ponowne uruchomienie skanowania w poszukiwaniu nowo wybranego protokołu transmisji. |
| **Port / Urządzenie BLE / Nazwa hosta / Adres IP** | Wyświetla listę wykrytych urządzeń jako `device - description`; automatycznie wybierany jest pierwszy wpis rozpoznany jako czujnik. Podczas skanowania wyświetla się `Scanning...` (USB), `Scanning (N)...` z 8-sekundowym odliczaniem (BLE) lub `Discovering ethernet sensors (N)...` z 5-sekundowym odliczaniem (Ethernet). Puste wyniki to: `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Odśwież** | Natychmiast ponownie skanuje wybrany protokół (funkcja wyłączona w trakcie skanowania BLE/Ethernet). |
| **Połącz czujnik** | Opcja dostępna po wybraniu urządzenia; podczas nawiązywania połączenia nazwa zmienia się na `Connecting...`. |

Wykrywanie działa tylko **gdy na ekranie wyświetlone jest okno dialogowe połączenia** i powtarza się co 15 sekund wyłącznie dla wybranego protokołu — samo otwarcie karty nie powoduje skanowania. W przypadku niepowodzenia w oknie dialogowym wyświetla się komunikat: *„Połączenie nie powiodło się. Spróbuj odłączyć i ponownie podłączyć czujnik, a następnie kliknij ponownie przycisk Połącz.”*

Pasek boczny otwiera się automatycznie po podłączeniu pierwszego czujnika.

{% hint style="info" %}
**DAQ-E nie wyświetla się?**Urządzenie DAQ-E nie posiada diody LED stanu — sprawdź wskaźnik PoE/połączenia na przełączniku lub porcie iniektora, do którego jest podłączone, i odczekaj kilka sekund po włączeniu zasilania, aż się uruchomi. Urządzenie Chloros musi znajdować się w tej samej domenie rozgłoszeniowej (mDNS nie przechodzi przez routery). Reguła zapory sieciowej**Chloros Backend (UDP-In)** w instalatorze zezwala na ruch mDNS (UDP 5353), dane z DAQ-E (UDP 5002) oraz PTP (UDP 319/320); w przypadku zapory sieciowej innego producenta należy zezwolić na te same porty. Dwa urządzenia DAQ-E w jednej sieci LAN są wykrywane oddzielnie, każde pod własną nazwą hosta `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Typ urządzenia obejmuje DAQ-U (USB), DAQ-M (Bluetooth) oraz DAQ-E (Ethernet)</figcaption></figure>

***

## Pasek boczny czujników

Każdy podłączony czujnik otrzymuje jeden wiersz (plus jeden wiersz na każdą grupę „Otoczenie+Obiekt”). Wiersze można zmieniać kolejność poprzez przeciąganie, a ich kolejność wpływa również na kolejność kafelków wykresów. Kliknij wiersz, aby ustawić ten czujnik/grupę jako aktywny wykres w widoku listy.

| Element | Znaczenie |
| --- | --- |
| Kolorowa lewa ramka | Kolor wykresu czujnika. |
| Ikona transportu | `DAQ-U` / `DAQ-M` / `DAQ-E` lub zielona plakietka `REF` dla grupy odbicia Ambient+Object. |
| Nazwa urządzenia | Domyślnie jest to numer seryjny czujnika (jego stały identyfikator do celów kalibracji, nazw plików `.daq` oraz dopasowywania podczas importu); nazwy niestandardowe są zachowywane dla poszczególnych projektów. |
| Piktogram **Skalibrowano** (zielony) | Wyświetlany, gdy załadowany jest fabryczny pakiet kalibracyjny czujnika, tj. widma mają rzeczywistą wartość w W/m²/nm. |
| Ikona **Dostępna aktualizacja** (bursztynowa, tylko DAQ-E) | Bieżące oprogramowanie sprzętowe jest starsze niż obraz dołączony do tej kompilacji Chloros

. Podczas aktualizacji wyświetla bieżący postęp (`Flashing… N%`, `Restarting sensor…`, a następnie `Updated X → Y` lub `Failed`). |
| Oko | Włącza lub wyłącza widoczność tego czujnika na wykresie. |
| Koło zębate | Otwiera okno modalne ustawień danego czujnika (poniżej). |
| ✕ (czerwone) | Odłącza czujnik lub usuwa grupę „Otoczenie+Obiekt”. |

Nad wierszami znajdują się dwa przyciski:

* **Podłącz czujnik** — otwiera okno dialogowe podłączania (podczas działania zmienia nazwę na `Connecting...`).
* **Nagrywaj wszystko / Zatrzymaj wszystko**— uruchamia lub zatrzymuje nagrywanie `.daq` na**każdym**podłączonym czujniku. Wymaga co najmniej jednego czujnika**oraz otwartego projektu** (podpowiedź: „Otwórz projekt, aby rozpocząć nagrywanie”); przycisk zmienia kolor na czerwony, gdy trwa jakiekolwiek nagrywanie.

W stanie pustym wyświetla się komunikat „Brak podłączonych czujników”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Ustawienia poszczególnych czujników (okno modalne z ikoną koła zębatego)

Otwierają się po kliknięciu ikony koła zębatego w wierszu czujnika. Zawartość w kolejności:

* **Wiersze informacyjne** — Typ urządzenia (DAQ-U/M/E), Połączenie (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (port COM, adres BLE lub host) oraz numer seryjny.
* **Raport kalibracji: Pobierz** — pobiera certyfikat kalibracji tego urządzenia zgodny z normami NIST (PDF) i otwiera go w przeglądarce plików PDF. Opcja dostępna po podaniu numeru seryjnego; certyfikat jest zapisywany w pamięci podręcznej przy pierwszym połączeniu.
* **Nazwa urządzenia** — kliknij ikonę ołówka, aby zmienić nazwę; nazwa jest zachowywana dla danego projektu.
* **Kolor linii wykresu** — próbka koloru; zachowywana dla danego projektu.
* **Czas integracji (ms)**— suwak + liczba,**1–500 ms**, domyślnie**32 ms**. Opcja wyłączona, gdy funkcja AE jest włączona.
* **Średnia klatek**— suwak + liczba,**1–50 klatek**, domyślnie**20**.
* **AE: WŁ./WYŁ.**— przełącznik automatycznej ekspozycji;**domyślnie WŁ.** po podłączeniu. Wyłącz, aby ręcznie ustawić czas integracji.
* **Zatrzymaj transmisję / Rozpocznij transmisję** — wstrzymaj lub wznow transmisję na żywo.
* **Nagrywaj / Zatrzymaj nagrywanie** — nagrywanie z czujnika `.daq` (wymaga otwartego projektu).
* **Cap** — profil korekcji cap (następna sekcja).
* **Wiersze informacji na żywo** — czas integracji (ms), liczba klatek na sekundę (FPS), próbki, nagrywanie (czerwone `REC` lub `Off`) oraz stan (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Tylko DAQ-E: wiersze dotyczące sieci, oprogramowania układowego i PTP

* **Nazwa hosta / adres IP** — aktualny adres urządzenia.
* **Oprogramowanie układowe** — aktualna wersja oprogramowania układowego wraz z komórką akcji:<version\>

przycisk</version\>

**Zaktualizuj do \<version\>** pojawia się</version\>

,<version\>

gdy ta kompilacja Chloros

zawiera nowszy obraz oprogramowania układowego DAQ-E. Aktualizacja jest przesyłana przez sieć w około 30 sekund; czujnik uruchamia się ponownie i automatycznie ponownie łączy się, a przerwany transfer pozostawia aktualne oprogramowanie układowe nienaruszone. Postęp aktualizacji jest wyświetlany na bieżąco (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), a komórka wyświetla wartość `Up to date`, gdy aktualizacja jest aktualna.
* **Synchronizacja PTP** — aktualny stan PTP (w razie awarii powraca do `unknown`). Oprogramowanie sprzętowe DAQ-E w wersji 1.2.0+ uczestniczy w standardzie IEEE 1588 PTPv2 wyłącznie jako zegar podrzędny; serwer Chloros

pełni rolę grandmastera PTP, a każda kamera DAQ-E i LATTICE w sieci LAN działa jako urządzenie podrzędne w domenie 0, utrzymując rozbieżność znaczników czasu w granicach około 1 ms.
* **Sieć**— adres, pod którym urządzenie działa, oraz sposób jego uzyskania (DHCP, adres lokalny lub statyczny), a także zapisane ustawienie, które zostanie zastosowane po ponownym uruchomieniu.**Zmień…**otwiera formularz:**Automatycznie (DHCP)**lub**Adres IP statyczny**z adresem IP, maską podsieci i opcjonalną bramą.**Zastosuj i uruchom ponownie** zapisuje ustawienia, restartuje czujnik i ponownie go podłącza. Adres statyczny spoza sieci tego komputera jest odrzucany, ponieważ Chloros

nie mógł połączyć się z czujnikiem po ponownym uruchomieniu; pojawia się wówczas okienko pozwalające na dopuszczenie tego mimo wszystko, w przypadku komputera, który zostanie ponownie zaadresowany lub przekierowany do tej sieci. **Transmisja po włączeniu zasilania**określa, czy czujnik rozpocznie transmisję natychmiast po włączeniu zasilania, zanim jakiekolwiek oprogramowanie nawiąże połączenie.**Restartuj czujnik** powoduje jego ponowne uruchomienie i ponowne połączenie z zachowaniem bieżących ustawień. Restart jest odrzucany, gdy czujnik prowadzi rejestrację. Nowość w wersji 1.3.0.

W przypadku grupy „Ambient+Object” okno modalne z ikoną koła zębatego pokazuje tylko czujniki źródłowe grupy, nazwę urządzenia oraz kolor linii wykresu.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Wybór nasadki

Menu rozwijane **Nasadka** informuje aplikację Chloros

, która nasadka fizyczna jest założona na dyfuzor czujnika, i stosuje do każdego widma profil korekcyjny tej nasadki, zmierzony fabrycznie. Dostępne opcje zależą od modelu:

| Model | Dostępne nasadki |
| --- | --- |
| DAQ-U | Brak (nagie czujniki), pole widzenia 15°, pole widzenia 30°, pole widzenia 45°, pole widzenia 60°, pole widzenia 90°, Sunshine (korektor cosinusowy) |
| DAQ-M | Brak (nagie czujniki), Sunshine (korektor cosinusowy) |
| DAQ-E | Brak (nagie czujniki), pole widzenia 15°, pole widzenia 45°, pole widzenia 90°, Sunshine (korektor cosinusowy) |

**Ustawieniem domyślnym dla każdego modelu jest Sunshine (korektor cosinusowy)** — firma MAPIR

dostarcza każdy moduł DAQ z zamontowaną nasadką Sunshine, co stanowi standardową konfigurację do użytku na zewnątrz: widok półkulisty 180° z błędem cosinusowym ≤ ±4 % do 60° i ≤ ±4,5 % do 70° (niezalecane przy elewacji Słońca poniżej ~15°), z konstrukcyjnym tłumieniem (~12×). Wybrany ustawienie pozostaje zachowane w projekcie.

{% hint style="warning" %}
**Wybór nasadki musi odpowiadać fizycznej nasadce.**Ani czujnik, ani oprogramowanie nie są w stanie wykryć, która nasadka jest zamontowana. Wybór ten determinuje zarówno korekcję na żywo, jak i znacznik zapisywany w każdym pliku `.daq` — przy ~12-krotnym tłumieniu nasadki Sunshine niezgłoszona zmiana nasadki powoduje błędną korekcję widm mniej więcej o ten współczynnik. (Zdejmowanie i ponowne zamontowanie tej samej nasadki powtarza się z dokładnością do około 1,5 %). Wybierz opcję**Brak (nagi czujnik)** tylko wtedy, gdy nasadka jest fizycznie zdjęta; w przypadku DAQ-E opcja „None” nadal stosuje fabryczny profil geometryczny dla wpuszczanego szklanego dyfuzora — nie jest to operacja bezczynna — a goły DAQ-E stanowi konfigurację laboratoryjną, a nie obsługiwany w terenie.
{% endhint %}

{% hint style="info" %}
Aktualizacja w stosunku do wcześniejszej instrukcji: przełącznik „Zainstalowany dyfuzor słoneczny” po stronie przeglądarki, obecny w wersji 1.1.0, został usunięty. Obsługa osłon odbywa się teraz za pomocą profilu osłony dla poszczególnych czujników, stosowanego po stronie serwera.
{% endhint %}

***

## Obszar wykresu

Przyklejony górny pasek zawiera **przełącznik między widokiem listy a siatką**oraz suwak**powiększenia wykresu** (rozmiar kafelka 200–2000 pikseli). Widok automatycznie przełącza się na siatkę, gdy istnieje więcej niż jedna grupa wykresów, a wraca do widoku listy, gdy jest jedna grupa lub mniej. Tryb wyświetlania i rozmiar wykresu są zachowywane dla każdego projektu.**Wykres widma** dla każdego czujnika pokazuje:

* **Oś X** — długość fali (nm). Siatka czujnika obejmuje zakres 340–1010 nm z krokiem 5 nm (135 punktów), interpolowanym do 1 nm na potrzeby wyświetlania.
* **Oś Y**— dla skalibrowanego czujnika (zielona pigułka**Skalibrowany**),**Moc (W/m²/nm)**— radiometrycznie skalibrowane natężenie promieniowania spektralnego we wszystkich trzech systemach jednostek — z automatycznym prefiksem SI (m/µ/n) wybranym na podstawie wartości szczytowej. Czujnik, którego pakiet kalibracyjny nie został załadowany, wyświetla swoje surowe odczyty jako**Intensywność (liczba impulsów)**, bez skalowania. Gdy wykres nakłada na siebie oba rodzaje danych, moc jest wyświetlana na lewej osi, a liczby zliczeń na prawej osi.
* Tęczowe wypełnienie spektralne pod pojedynczą krzywą; wiele czujników na jednym wykresie nakłada się jako kolorowe linie z przyciemnionym wypełnieniem.
* **Najechanie kursorem**— pionowy kursor z długością fali i wartością dla danego czujnika (W/m²/nm dla czujnika skalibrowanego, w pozostałych przypadkach liczba zliczeń);**przeciągnięcie** służy do powiększania (podczas powiększenia pojawia się przycisk pomniejszania).
* Przycisk **+** (tylko w widoku siatki) służy do dodania czujnika do tego wykresu lub utworzenia grupy (poniżej).
* Nazwa urządzenia wyśrodkowana u góry oraz ikona ładowania do momentu pojawienia się pierwszej klatki.

**Nasycenie** nie jest zaznaczone na samym wykresie: nasycony czujnik wyświetla czerwony tekst statusu `SATURATED` oraz czerwony wiersz `Saturated: Yes` w tabeli danych na żywo. Aby to usunąć, należy skrócić czas integracji lub ponownie włączyć AE.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabela danych na żywo (widok listy)

Poniżej wykresu w widoku listy, odświeżana co 500 ms:

* **Wszystkie modele**: Próbka koloru światła (sRGB z CIE XYZ), Nasycony (Tak/Nie), CIE 1931 X/Y/Z, Chromatyczność x/y, CIE u′/v′, CCT (K), CRI (Ra), dominująca długość fali (nm), szczytowa długość fali (nm), czystość wzbudzenia, Duv, CIE L\*/a\*/b\* oraz Munsell H/V/C.
* **Tylko czujniki skalibrowane**(dowolny model DAQ-U / DAQ-M / DAQ-E po załadowaniu fabrycznego pakietu kalibracyjnego — rozpoznaje się to po zielonej plakietce**Skalibrowany** w wierszu czujnika): Moc całkowita (W/m²), luks fotopowy (lx), luks skotopowy (lx), stosunek S/P, PPFD oraz PPFD Red

/Green

/Blue

(µmol/m²/s), a także natężenia promieniowania opicznego — S-cone, Melanopic, Rhodopic, M-cone, L-cone (wszystkie w W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupy odbicia (Światło otoczenia + obiekt)

Dwa połączone czujniki można połączyć w celu uzyskania podglądu odbicia na żywo — bez użycia kamery:

1. W widoku siatki kliknij **+**na kafelku wykresu i wybierz**Połącz światło otoczenia + obiekt**.
2. Wybierz czujnik **Źródło światła otoczenia**oraz czujnik**Skaner obiektu**(dwa różne czujniki), a następnie**Utwórz**.

Chloros

oblicza R(λ) = obiekt(λ) / otoczenie(λ) dla każdej długości fali na podstawie dwóch strumieni danych na żywo (0, gdy otoczenie ≤ 0). Etykieta grupy zależy od klasy kalibracji czujników:

* Oba czujniki skalibrowane (pakiet załadowany) → **„Zdanowa odbiciowość”**.
* Jeden z czujników nie jest skalibrowany → **„Współczynnik odbicia względnego”**.

Grupa pojawia się jako zielony wiersz `REF` na pasku bocznym oraz na własnym wykresie (wypełnienie tęczowe, wartości wyświetlane po najechaniu myszką z dokładnością do 4 miejsc po przecinku, powiększanie przez przeciągnięcie).

Menu **+**oferuje również opcję**Dodaj nowy czujnik** z trzema możliwościami umieszczenia: *Połącz nowy czujnik* (dołącz do tego wykresu), *Przenieś tutaj istniejący czujnik* lub *Wyświetl nowy czujnik* (w osobnym wykresie).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabela wskaźników wegetacyjnych

W widoku listy poniżej wykresu grupy odbicia znajduje się tabela wskaźników wegetacyjnych, obliczona na podstawie aktualnych wartości odbicia w centrach pasm **niebieski 450 / zielony 550 / czerwony 670 / NIR

800 nm** (wartości z dokładnością do 4 miejsc po przecinku, `---`, gdy nie da się obliczyć; najedź kursorem na nazwę wskaźnika, aby wyświetlić jego pełną nazwę):

* **Zawsze wyświetlane** (niezależne od skali, dowolna kombinacja czujników): NDVI

, GNDVI

, ENDVI, WDRVI

, GRVI

, CVI

, GCI

, MSR.
* **Tylko gdy oba czujniki są skalibrowane pod kątem mocy** (załadowane oba zestawy): EVI

, SAVI

, OSAVI

, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI

, NLI, MNLI, FCI, GEMI

.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Zapisywanie plików `.daq`

* Zapisywanie wymaga **otwartego projektu** — w przeciwnym razie zarówno opcja „Zapisz wszystko” (pasek boczny), jak i przycisk „Zapisz” dla poszczególnych czujników są nieaktywne.
* Pliki są zapisywane w katalogu **`<project folder>/light_sensor/`**; nazwy plików zawierają identyfikator czujnika i sygnaturę czasową, a nazwa urządzenia jest zapisywana wraz z zapisem.
* Po zakończeniu nagrywania (przycisk „Stop”, „Stop All” lub rozłączenie w trakcie nagrywania) gotowy plik `.daq` jest **automatycznie dodawany do otwartego projektu** — pojawia się on na liście plików projektu bez konieczności ręcznego dodawania, gotowy do wykorzystania jako dane natężenia promieniowania padającego do [przetwarzania współczynnika odbicia](README.md).
* Podczas nagrywania w wierszach na żywo okna modalnego ustawień wyświetla się czerwony wskaźnik `REC`.

W przypadku ilościowych wartości natężenia promieniowania należy uśrednić dane z co najmniej 15 sekund — jest to cecha charakterystyczna urządzenia, a nie wada.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Układy wieloczujnikowe i trwałość projektu

* Łącz kilka czujników na jednym wykresie (wspólne osie), zachowuj oddzielne wykresy (automatyczny układ siatki), przenoś czujniki między wykresami, zmieniaj kolejność wierszy/kafelków poprzez przeciąganie oraz ukrywaj poszczególne czujniki za pomocą przełącznika w postaci ikony oka.
* W ramach każdego projektu Chloros

zachowuje: nazwy urządzeń, kolory wykresów, rozmiar wykresu, tryb wyświetlania oraz ustawienia każdego czujnika (czas integracji, uśrednianie klatek, stan AE, wybór nasadki).
* **Ponowne otwarcie projektu powoduje automatyczne ponowne połączenie czujników** na podstawie adresu — portu COM dla DAQ-U, urządzenia BLE dla DAQ-M, nazwę hosta mDNS dla DAQ-E (rozpoznawaną nawet po zmianie adresu IP urządzenia) — oraz ponownie stosuje zapisany profil nakładki każdego czujnika, uśrednianie klatek, stan AE oraz ręcznie ustawiony czas integracji.***

## Parowanie kamery (DLS)

Nie ma potrzeby parowania. W przeciwieństwie do procesów DLS stosowanych w dronach, które z góry wiążą czujnik światła z kamerą, Chloros

dopasowuje dane DAQ do obrazów na dalszym etapie: podczas importu/przetwarzania odczyty `.daq` są interpolowane do znacznika czasu ekspozycji każdego ujęcia. Nagrywaj przy użyciu dowolnego podłączonego czujnika (`.daq` trafia do projektu automatycznie), a przetwarzanie odbicia znajdzie odpowiednie odczyty według czasu — zobacz [Czujniki światła DAQ](README.md), aby dowiedzieć się, w jaki sposób wykorzystywane są dane dotyczące promieniowania padającego. W przypadku **na żywo**ujęć LATTICE należy zamiast tego wybrać czujnik z menu rozwijanego**Czujnik światła** w zakładce Kamery; każdy pomiar zapisuje wówczas wykorzystany odczyt, dopasowany wyłącznie do kamer tego pomiaru (zobacz [Przepływy pracy z odbiciem światła](reflectance.md)).</version\>
