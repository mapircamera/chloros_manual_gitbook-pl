# Układy wielokamerowe

**Układ**LATTICE to dwie lub więcej kamer LATTICE połączonych w jedną zsynchronizowaną całość. Jedna z kamer pełni rolę**głównej**: wysyła ona sprzętowy impuls wyzwalający GPIO na wspólnej linii synchronizacji (domyślnie**Line2**), dzięki czemu wszystkie kamery rejestrują ten sam moment. Chloros

zapewnia synchronizację czasową PTP, podgląd na żywo (każda kamera wyświetla osobny kadr lub pojedynczy, wyrównany obraz złożony z wielu pasm) oraz zsynchronizowane przechwytywanie — każda seria zdjęć tworzy jedną **grupę klatek**, w której wszystkie kamery mają ten sam znacznik czasu i identyfikator klatki (podawany jako `fid:N` na wyjściu przechwytywania).

Macierze to sposób, w jaki kamery monochromatyczne (M3M) generują wskaźniki wegetacji — jedna kamera dostarcza jedno pasmo, a macierz wyrównuje je w stos wielopasmowy. Zobacz [Kamery mono i wskaźniki wegetacyjne](mono-indices.md).

Istnieją trzy równoważne sposoby podłączenia macierzy, a wszystkie z nich wykorzystują ten sam proces „smart-prep”:

| Interfejs | Punkt wejścia |
| --- | --- |
| GUI | Zakładka „Kamery” → **Podłącz macierz** (niebieski przycisk) |
| CLI

| `chloros-cli lattice array-connect --serials SN1,SN2,…` (pierwszy numer seryjny = urządzenie główne) |
| Python

SDK

| `connect_array(serials=[…])` → `ArraySession` (pierwszy numer seryjny = urządzenie nadrzędne) |

Funkcja „smart-prep” wykonuje: test możliwości sieciowych (ping ICMP DF + sonda GVSP), wybór warstwy synchronizacji, prognozę liczby klatek przy żądanym rozmiarze klatki, decyzję o zastosowaniu spakowanego transportu 12-bitowego (zobacz [Transport spakowany 12-bitowy](#packed-12-bit-transport)), włączenie PTP, automatyczny wybór formatu pikseli dla każdej kamery, automatyczne ustawienie ekspozycji na podstawie zapisanego stanu każdej kamery oraz konfigurację wyzwalacza GPIO na linii Line2.

**Domyślnie pełny czujnik (nowość w wersji 1.3.0).** Connect zachowuje żądany rozmiar klatki — pełny czujnik 2048×1536, bez binningu i bez kadrowania, chyba że zdecydujesz inaczej — i pozwala, by liczba klatek na sekundę spadła do poziomu, jaki zapewniają host i sieć. Nigdy nie łączy pikseli ani nie przycina obrazu z własnej inicjatywy (w wersji 1.2.0 łączenie pikseli uruchamiało się automatycznie poniżej 1,5 kl./s). Decyzja o binningu lub mniejszym obszarze zainteresowania (ROI) należy do Ciebie: w oknie dialogowym połączenia wybierz jedną z opcji, a sekcja „Przewidywany wynik” pokaże wynikającą z tego liczbę klatek na sekundę; `array-connect` wyświetla szybsze opcje wraz z ich przewidywanymi wartościami.

{% hint style="info" %}
Aby cokolwiek z tego działało, kamery muszą być dostępne w sieci — zobacz [Podłączanie kamer](connecting.md) w celu wykrycia, adresowania oraz pobrania kalibracji przy pierwszym połączeniu. W przypadku zestawów wielokamerowych ustawienia pierścienia odbiorczego karty sieciowej hosta mają równie duże znaczenie jak prędkość łącza; pełna tabela „objaw→rozwiązanie” znajduje się w [Podręcznik CLI

§ Konfiguracja i dostrajanie karty sieciowej hosta](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Okno dialogowe „Połącz układ”

Zakładka „Kamery” → **Połącz układ**otwiera trzyetapowego kreatora:**Wybierz → Tryb wyświetlania → Ustawienia**.

### Krok 1 — Wybór kamery głównej i kamer podrzędnych

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Okno dialogowe skanuje sieć zaraz po otwarciu („Skanowanie sieci...”), a następnie sprawdza kable synchronizacyjne („Sprawdzanie kabli synchronizacyjnych...”), aby znaleźć kamery podłączone do linii synchronizacyjnej M8. Do utworzenia macierzy potrzebne są co najmniej **2 kamery**.

Kable synchronizacyjne M8 łączą się w jednym punkcie, więc **każda podłączona kamera może pełnić rolę kamery głównej**. Funkcja sprawdzania wstępnie przypisuje role — kamera RGB

(`-FRGB`) jako kamera główna, jeśli jest podłączona, w przeciwnym razie pierwsza podłączona kamera, a pozostałe podłączone kamery jako kamery podrzędne — i wyświetla jeden z trzech komunikatów:

| Komunikat | Znaczenie |
| --- | --- |
| „Znaleziono kabel synchronizacyjny w {n} kamerach. Każda z nich może pełnić rolę urządzenia nadrzędnego; kamera RGB

(w przeciwnym razie pierwsza) jest wstępnie wybrana.” (zielony) | Sonda wykryła kamery na linii synchronizacyjnej; pola wyboru urządzenia nadrzędnego i podrzędnego są już zaznaczone. |
| „Nie wykryto urządzenia nadrzędnego — sprawdź kabel GPIO” (pomarańczowy) | Żadna kamera nie wykryła impulsu wyzwalającego; sprawdź okablowanie synchronizacyjne. Role są nadal wstępnie wypełnione (kamera RGB

, w przeciwnym razie pierwsza kamera na liście, jako urządzenie nadrzędne) i można je zmienić. |
| „Brak kabla synchronizacji: {numery szeregowe}” (pomarańczowy) | Wymienione kamery nie mają podłączonego kabla synchronizacji. |

Tabela kamer zawiera kolumny **Kamera / Numer seryjny / IP / Master (radio) / Slave (pole wyboru)**. Wiersze są wyświetlane w**kolejności kamer** — kamery M3C (Bayer) przed kamerami M3M (mono); wśród kamer M3C kolejność to FRGB, FRGN, FOCN, a następnie FNGB; kamery M3M według długości fali filtra, najpierw najniższe (F450 przed F808). Lista kamer na pasku bocznym oraz domyślny układ siatki stosują tę samą kolejność.

* Wybierz dokładnie **jedną kamerę główną**i**jedną lub więcej kamer podrzędnych**. Ponowne kliknięcie pola wyboru bieżącej kamery głównej powoduje jej odznaczenie. Wybór innej kamery głównej wymaga jednego kliknięcia: nowa kamera główna opuszcza grupę kamer podrzędnych, a poprzednia kamera główna staje się kamerą podrzędną (chyba że nie ma kabla synchronizacyjnego).
* Kamera oznaczona jako **„Brak kabla synchronizacyjnego”** nigdy nie może zostać wybrana jako kamera podrzędna — kamera podrzędna bez okablowania wyzwalającego czekałaby w nieskończoność na sygnał z linii synchronizacji i dostarczałaby martwy sygnał. Zamiast tego podłącz tę kamerę jako kamerę autonomiczną.
* Kamery już podłączone w trybie samodzielnym *nie* są wyłączane: podłączenie do macierzy zwalnia sesję samodzielną i ponownie otwiera kamerę w ramach macierzy.

**Dalej: Tryb wyświetlania →**staje się dostępny po wybraniu kamery głównej i co najmniej jednej kamery podrzędnej.**Ponowne skanowanie** ponownie uruchamia wykrywanie i sondę okablowania.

{% hint style="warning" %}
**Anuluj** jest wyłączone podczas trwania skanowania lub testu okablowania — anulowanie w trakcie testu może spowodować awarię biblioteki SDK

kamery w oprogramowaniu sprzętowym kamery LATTICE. Należy poczekać, aż ikona ładowania zakończy działanie.
{% endhint %}

### Krok 2 — Tryb wyświetlania



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Tryb | Co otrzymujesz |
| --- | --- |
| **Oddzielne kamery** | Jedna kafelka na żywo na każdą kamerę, wszystkie uruchamiane jednocześnie, dzięki czemu klatki pozostają zsynchronizowane. Każda kamera zachowuje własny kolor i ustawienia. |
| **Kamery połączone** *(domyślnie)* | Pojedyncza kafelka wyświetlająca wyrównany, wielopasmowy kompozyt NDVI

/indeksowy. Kamery mają wspólną kolorystykę matrycy. |

Tryb wyświetlania zmienia jedynie sposób prezentacji podglądu na żywo — sposób przechwytywania obrazu jest taki sam w obu trybach.

### Krok 3 — Ustawienia matrycy i przewidywany wynik

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Po wejściu do tej sceny Chloros

zwraca się do serwera z prośbą o **rekomendację**i stosuje ją:**pełny czujnik — bez binningu, bez kadrowania ROI — o ile pozwala na to dowolna częstotliwość klatek**, niezależnie od szybkości przesyłu zapewnianej przez hosta i łącze. Dopiero gdy żadna częstotliwość nie pozwala na wykorzystanie pełnego czujnika, rekomendacja zmniejsza kadr, a wtedy preferuje binning zamiast kadrowania obszaru zainteresowania (ROI), ponieważ binning zachowuje pełne pole widzenia. Od tego momentu dokonuje się kompromisu między rozdzielczością a liczbą klatek na sekundę. Każda wprowadzona zmiana powoduje ponowne przeprowadzenie analizy na żywo i aktualizację panelu**Przewidywany wynik** po prawej stronie.

Lewa kolumna — ustawienia:

| Element sterujący | Opcje | Domyślne | Uwagi |
| --- | --- | --- | --- |
| **ROI (pole widzenia)** | Pełne (2048×1536) / Połowa (1024×768) / Ćwierć (512×384) | Pełne | Kadrowanie czujnika: kadrowanie do połowy lub ćwierci w mniejszym obszarze przy natywnej gęstości pikseli. |
| **Binning** | 1× / 2× (suma 2×2) / 4× (suma 4×4) | 1× | Binning sprzętowy: 2×2 = pełne pole widzenia przy jednej czwartej kosztu transmisji; 4×4 = pełne pole widzenia przy 1/16. Ukryte, jeśli kamery nie obsługują binningu. |
| **Obraz przesyłany po przewodzie** (odczyt) | — | — | Szerokość × wysokość po binningu faktycznie przesyłana po przewodzie, zaokrąglona do wielokrotności 16 (minimum 64). |
| **Częstotliwość wyzwalania** | 0,5–60 klatek na sekundę, krok 0,1 | puste = auto | Częstotliwość wyzwalania urządzenia nadrzędnego. Pozostaw puste, aby program Chloros

sam ją obliczył. |
| **Przepustowość łącza**| 20–2000 MB/s, krok co 10 | puste = auto | Ilość danych, jaką host może faktycznie przyjąć, w MB/s —**jedyna wartość, od której zależy cała alokacja macierzy.** Wykrywana automatycznie przez kartę sieciową. Zmniejsz ją, jeśli macierz zgłasza uszkodzone ramki: wykryta wartość zawyża wydajność adapterów USB i przełączników współdzielonych. Zmiana tej wartości powoduje ponowne uruchomienie prognozy na żywo. Pole otwiera się z wartością przepustowości, z jaką podłączona była ostatnia macierz (zapisaną wraz z projektem), więc raz skorygowana wartość przepustowości jest zachowywana. |

Zniknęło pole wyboru **Rozdzielczość pinów** z wersji 1.2.0: funkcja „Połącz” zawsze zachowuje wybrany rozmiar ramki i zamiast tego obniża częstotliwość odświeżania.

Prawa kolumna — **Prognozowany wynik**:

* **Poziom synchronizacji** — „Jednoczesne przechwytywanie” (zielony), „Jednoczesne przechwytywanie (emisja rozłożona w czasie FTD)” (zielony), „Przechwytywanie rozłożone w czasie (odchylenie 100 ms)” (bursztynowy) lub „Konfiguracja zbyt duża” (czerwony). Chloros

wybiera poziom: przechwytywanie równoczesne, gdy tylko łącze może być współdzielone z prędkością, do której można dostosować tempo pracy kamer (liczba klatek na sekundę spada wraz ze zmniejszaniem się udziału każdej z kamer), a przechwytywanie z przesunięciem tylko wtedy, gdy nie jest to możliwe — dzięki czemu macierz łączy się z dowolną siecią, z liczbą klatek na sekundę, na jaką pozwala ta sieć.
* **Prognoza fps** — oczekiwana częstotliwość klatek, z jaką ma działać macierz. W przypadku przechwytywania równoczesnego jest to jedna liczba: na pierwszym miejscu jest częstotliwość, a automatyczna ekspozycja wypełnia wyznaczony przez nią okres, a następnie zwiększa wzmocnienie, dzięki czemu słabo oświetlona scena nie powoduje jej obniżenia (zobacz [Jak automatyczna ekspozycja rozdziela czas naświetlania i wzmocnienie](camera-settings.md)). W przypadku przechwytywania rozłożonego w czasie jest to przedstawione jako zakres („słabe oświetlenie → jasne”), ponieważ ekspozycja każdej kamery wydłuża rozłożenie w scenie o słabym oświetleniu.
* **Linia karty sieciowej** — prędkość łącza i stała przepustowość („karta sieciowa {mbps} Mbps · stała {N} MB/s”).
* **Sprawdzenie symulowanej emisji serii danych** — czy pierścień odbiorczy karty sieciowej hosta może przyjąć jedną równoczesną serię danych ze wszystkich kamer („Symulowana emisja serii danych: X MB · dostępny pierścień karty sieciowej: Y MB ✓/✗”).
* **Sprawdzenie budżetu łącza** — zagregowane zapotrzebowanie w stanie ustalonym w porównaniu z bezpiecznym dla kolizji limitem łącza („Budżet łącza: {demand} MB/s wymagane przez {n} kamer · limit {ceiling} MB/s ✓/✗ przekroczenie limitu”).
* **„Maksymalna liczba kamer na tym łączu: {n} — ustalona przez minimalną przepustowość na kamerę, więc grupowanie nie zwiększa tej wartości.”** — wyświetla się, gdy zbliżasz się do (lub przekraczasz) limit liczby kamer.
* **„Przy tych ustawieniach NAGRYWANIE ZOSTANIE PRZERWANE.”**— czerwone ostrzeżenie z uzasadnieniem serwera, wraz z listą przeszkód i niebieskimi**sugestiami rozwiązań** („Aby zmieścić tę matrycę w sieci” / „Aby odblokować jednoczesne przechwytywanie”).

Opcja **Zastosuj i połącz** jest zablokowana, dopóki nie pojawi się prognoza, a jej etykieta informuje, dlaczego jest odrzucana:

| Etykieta przycisku | Znaczenie | Co faktycznie pomaga |
| --- | --- | --- |
| „Analizowanie...” | Analiza wciąż trwa. | Poczekaj. |
| **„Zbyt wiele kamer dla tej sieci”**| Łącze nie jest w stanie obsłużyć tej matrycy nawet po jednej kamerze na raz (niepowodzenie kontroli agregacji): znacznie więcej kamer niż może obsłużyć lub inne kamery przesyłające strumień w tej sieci. | Mniejsza liczba kamer, ramki typu jumbo w całym łańcuchu lub szybsza karta sieciowa.**Mniejszy obszar zainteresowania (ROI) NIE pomoże** — patrz poniżej. |
| **„Zmniejsz obszar ROI, aby włączyć”** | Przy tych ustawieniach dochodziłoby do utraty ramek (niepowodzenie kontroli burst/ring). | Zmniejsz obszar ROI, zwiększ binning lub napraw pierścień odbiorczy karty sieciowej. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Podczas łączenia może pojawić się zielony **panel pobierania kalibracji** z paskiem postępu dla każdego portu szeregowego: przy pierwszym podłączeniu kamery do urządzenia Chloros

pobiera z kamery fabryczny pakiet kalibracyjny o wielkości ~3,8 MB przez GigE (około 70 sekund na kamerę). Kamery zapisane w pamięci podręcznej nigdy nie wyświetlają tego panelu; przy ponad sześciu kamerach jego wiersze przewijają się. Zobacz [Podłączanie kamer](connecting.md).

### Pakowany transport 12-bitowy

Formaty 12-bitowe (`BayerRG12`, `Mono12`) są zazwyczaj przesyłane jako słowa 16-bitowe. **Transport**skompresowany**wysyła te same 12 bitów w 1,5 bajta na piksel i dekompresuje je na hoście, dzięki czemu wysłanie każdej klatki zajmuje o 25% mniej czasu — przy pełnej rozdzielczości czujnika daje to około jednej trzeciej więcej klatek na sekundę.**Zapisane dane są identyczne.** Funkcja „Array connect” decyduje o tym automatycznie (`auto`): kompresuje tylko wtedy, gdy jest to korzystne, a host dysponuje wystarczającą mocą obliczeniową procesora do dekompresji. Interfejs graficzny nie posiada przełącznika do tej funkcji; w interfejsie CLI

należy użyć `array-connect --packed {auto,on,off}` (domyślnie `auto`). Formaty 8-bitowe i 16-bitowe nigdy nie są kompresowane.

## Przepustowość: ile kamer zmieści się w sieci

To, ile kamer może obsłużyć macierz, zależy od łącza, a nie od Chloros

a, więc dane dotyczące planowania znajdują się w instrukcji sprzętu: **[Planowanie przepustowości macierzy](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Jak Chloros

je wykorzystuje: okno dialogowe połączenia uruchamia sondę sieciową, prognozuje osiągalną liczbę klatek na sekundę i wybiera odpowiedni poziom. Macierz łączy się w każdej sieci: w przypadku wolnego łącza każda kamera jest dostosowywana do swojej części przepustowości, a liczba klatek na sekundę spada, zamiast odrzucania macierzy lub zmniejszania jej klatek. Tylko macierz, której łącze nie jest w stanie obsłużyć nawet jednej kamery naraz, zostaje odrzucona (**Zbyt wiele kamer dla tej sieci**), zamiast cichego odrzucania pakietów — zobacz panel z prognozowanymi wynikami opisany powyżej.

## Gdy brakuje klatek

Kamera może nie figurować w opublikowanej grupie z dwóch zupełnie różnych powodów,
a każda z tych sytuacji wymaga odmiennego rozwiązania. Chloros

zlicza je oddzielnie, zamiast podawać jedną
„niekompletną” liczbę, która nie określa żadnej z tych przyczyn:

| Co się stało | Co to oznacza | Gdzie szukać |
| --- | --- | --- |
| **Uszkodzona**— ramka dotarła, ale miała błędy strukturalne | Utrata pakietów GVSP na ścieżce sieciowej |**Przepustowość łącza**, pierścień odbiorczy karty sieciowej, ramki jumbo, przełącznik |
| **Nigdy nie dotarła**— żadna ramka w ogóle nie dotarła | Kamera nie uruchomiła się lub nic z niej nie wyszło |**Kabel synchronizacyjny M8**, linia synchronizacji, czy wszystkie urządzenia są włączone |

Podział jest ponownie oceniany co 10 sekund podczas transmisji strumienia z macierzy. Powyżej 5% jest to
rejestrowane z podaniem obu wartości, a każdy uszkodzony bufor jest zgłaszany przy pierwszym wystąpieniu
dla danej kamery, a następnie sumowany co minutę, aby długa sesja pozostała czytelna.

**Uszkodzone klatki z zerową liczbą „nigdy nie dotarły” oznaczają, że wyzwalanie i synchronizacja kablowa działają idealnie**i każda utracona klatka znajduje się na ścieżce sieciowej. Rozwiązaniem jest obniżenie**Wire Budget** i
ponowne podłączenie.

{% hint style="warning" %}
**Obniżenie częstotliwości wyzwalania nie pomaga w przypadku uszkodzonych klatek.** Tempo przesyłania pakietów
przez kamerę jest ustalane jednorazowo, podczas nawiązywania połączenia. Obniżenie częstotliwości wyzwalania zmienia częstotliwość występowania serii
pakietów, a nie szybkość, z jaką sama seria trafia do sieci. W przypadku zmierzonej konfiguracji z 4 kamerami
5-krotne zmniejszenie częstotliwości wyzwalania nie zmieniło niczego, natomiast obniżenie budżetu przepustowości z 240 do
200 MB/s spowodowało spadek odsetka uszkodzonych klatek w tej samej konfiguracji z 10,4% do zera.
{% endhint %}

Działająca macierz nie może samodzielnie zmienić planu — należy ją odłączyć i podłączyć ponownie, aby moduł wybierający czas połączenia
mógł działać zgodnie z nowym limitem przepustowości.

### Karty sieciowe USB mają ograniczenia

Karta sieciowa USB Ethernet podaje swoją prędkość łącza *Ethernet*, ale to, co faktycznie
może utrzymać, jest ograniczone przez magistralę USB i jej sterownik. Klucz USB 10GbE był kiedyś uznawany
za zapewniający przepustowość około 1000 MB/s — wartość, której nikt nigdy nie zmierzył — a dostosowanie
pracy czterech kamer do tego fikcyjnego rezerwy przepustowości powodowało uszkodzenie 6–18 % klatek, podczas gdy macierz
nadal zgłaszała prawidłową docelową liczbę klatek na sekundę. Karty podłączane przez USB mają obecnie ograniczenie do
**200 MB/s** w systemie Windows

i na platformie NVIDIA Jetson oraz **400 MB/s** w systemie Linux

x86_64. Ograniczenie to jest wartością bezwzględną, a nie procentową, ponieważ limitem jest
magistrala: adapter USB 1 GbE osiąga prędkość około 80 MB/s i nie ma na niego wpływu.

Jeśli Twój host jest rzeczywiście szybszy niż limit, zwiększ wartość **Wire Budget**, aby to odzwierciedlić.

## Synchronizacja czasu PTP

*Synchronizacja* klatek pochodzi z wyzwalacza sprzętowego; **PTP** (IEEE 1588 PTPv2) zapewnia porównywalne *znaczniki czasu* na każdym urządzeniu. Jest ona domyślnie włączona po podłączeniu macierzy:

* **Backend hosta Chloros

pełni rolę serwera nadrzędnego PTP**. Kamery LATTICE i czujniki światła DAQ-E działają jako urządzenia podrzędne w domenie 0, dzięki czemu znaczniki czasu obrazów i widma DAQ są zsynchronizowane z jednym zegarem (~1 ms).
* `--no-ptp` (CLI

) wyłącza tę funkcję na potrzeby pracy laboratoryjnej — wówczas znaczniki czasu między kamerami **nie** są porównywalne.
* Sprawdź stan synchronizacji za pomocą CLI

:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Sama zakładka „Kamery” nie zawiera wskaźnika PTP; dostępne tam parametry synchronizacji dla poszczególnych kamer to tylko do odczytu: **Rola**(Master/Slave),**Linia synchronizacji** oraz poziom możliwości macierzy. Stan PTP czujników DAQ-E jest wyświetlany w szczegółach czujników w zakładce „Czujniki światła”.

## Widok

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

macierzy na żywo Główny obszar wyświetlania oferuje dwa układy (przełączanie w górnym pasku): **widok siatki**(każda kafelka to komórka; można zmieniać kolejność przez przeciągnięcie, gdy kłódka siatki jest odblokowana) oraz**widok listy**(wiersze o pełnej szerokości:**połączona**matryca zajmuje jeden wiersz — jest to jej złożona kafelka — natomiast**oddzielna**matryca zapewnia**każdemu elementowi własny wiersz**, z własnym podglądem na żywo i własną tabelą celów kalibracyjnych; poniżej układów znajduje się jedna samodzielna kamera — ta aktywna). Suwak**Powiększenie podglądu** zmienia rozmiar kafelków; przy szerokości komórki poniżej 200 pikseli nakładki z nazwą i liczbą klatek na sekundę ukrywają się automatycznie. Kamery wchodzące w skład układu pojawiają się na pasku bocznym oraz w domyślnym układzie siatki w kolejności kamer (M3C przed M3M, najpierw FRGB, kamery monochromatyczne według długości fali); odblokowanie siatki i przeciągnięcie nadal zastępuje kolejność w siatce.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Tryb oddzielny** wyświetla jedną płytkę na kamerę. Każda płytka zawiera:

* nazwę kamery (w lewym górnym rogu),
* **odczyt liczby klatek na sekundę** (w prawym górnym rogu) — jest to *rzeczywista częstotliwość przechwytywania* kamery zgłaszana przez serwer, a nie częstotliwość odświeżania podglądu (podgląd na żywo jest ograniczony do 30 klatek na sekundę niezależnie od częstotliwości przechwytywania),
* kropkę stanu — zieloną (transmisja strumieniowa) / bursztynową (ładowanie) / czerwoną (błąd),
* **ikona obrotowa wskazująca nieaktualną klatkę**, gdy przez 2 s nie nadeszła żadna nowa klatka — jest to normalne przez około 5 s po każdym połączeniu lub rozłączeniu, podczas gdy backend ponownie równoważy budżet przepustowości między kamerami.**Tryb połączony**wyświetla pojedynczą kafelkę złożoną: backend przeprowadza debayering, skalowanie, wyrównywanie, usuwanie szumów, konwersję na promieniowanie w poszczególnych pasmach (plus odbicie DLS, gdy podłączony jest czujnik światła), ocenia wyrażenie indeksowe tablicy, stosuje tabelę LUT oraz przesyła wynik w formacie MJPEG. Do momentu wyrenderowania pierwszej wyrównanej klatki kafelek informuje o swoim stanie: „Przygotowywanie macierzy…”, „Kalibruje się wyrównanie…”, „Oczekiwanie na pierwszą klatkę…” lub — jeśli wyczerpią się próby automatycznego wyrównania kafelka (około minuty) — „Wymagane wyrównanie” wraz z przyciskiem**Kalibruj wyrównanie**.

Odczyt liczby klatek na sekundę (fps) na kafelku matrycy informuje również o stanie strumienia:**„Stosowanie ustawień…”**podczas zmiany ustawień (format, binning, ROI), co powoduje ponowne uruchomienie kamer,**„Zawieszenie”**, gdy matryca działa, ale przestała dostarczać grupy klatek, oraz**„Zatrzymano”**, gdy matryca przestała przesyłać strumień — w takim przypadku należy ponownie podłączyć matrycę. Gdy podczas rozdzielania spektralnego konieczne było ograniczenie pasma widocznego na obrazie złożonym na dużej części jego pikseli (25% lub więcej), na kafelku pojawia się komunikat**„Dane pasma o niskim poziomie pewności · N%”**: wartości indeksowe oparte na tym paśmie są wątpliwe (zazwyczaj jedno pasmo otrzymało zbyt mało światła).

Przydatne informacje dotyczące trybu połączonego:

* Obraz złożony jest zsynchronizowany z klatką kamery **głównej**. Celowanie AE-ROI i pomiar punktowy na obrazie złożonym są dokładne dla kamery głównej, a przybliżone dla kamer podrzędnych; użyj opcji**Podzielony widok** (ustawienia macierzy → „Pokaż kamery składowe”), aby uzyskać kafelki dla poszczególnych kamer z dokładnością do piksela bez otwierania dodatkowych połączeń z kamerami.
* **Warstwy wyświetlania**(ustawienia macierzy; domyślnie wyłączone) pozwalają wybrać warstwę pierwszego planu i tła — dowolną kamerę członkowską lub**Indeks**. Gdy pierwszym planem jest**Indeks**, piksele poza zakresem minimalnym/maksymalnym LUT wyświetlają warstwę tła.
* **Rozdzielczość renderowania** (domyślnie 720p) określa wysokość transmisji na żywo *oraz* rozmiar eksportowanego obrazu złożonego. Obrazy z poszczególnych kamer są zawsze eksportowane w pełnej rozdzielczości.
* Wyrównanie jest obliczane dla każdej sesji i nigdy nie jest zapisywane. Chloros

automatycznie wyrównuje **każdy układ po połączeniu, w obu trybach wyświetlania**, gdy tylko ustabilizuje się automatyczna ekspozycja kamer składowych (próby trwają do 5 minut). Sekcja**Wyrównanie**w panelu ustawień matrycy pokazuje odchylenia RMS oraz przycisk**Ponowna kalibracja wyrównania** — dotyczy to również oddzielnych matryc, które nie mają kompozytowej mozaiki.

## Przechwytywanie: monitorowanie a analiza

Powierzchnie przechwytywania matryc dzielą się wyraźnie na **klasy monitorowania**(rejestrują to, co widać) i**klasy analizy** (rejestrują surowe dane, kalibrowane później):

| Przebieg pracy | Klasa | Co zapisuje | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **Przechwytywanie**(zdjęcia) | Analiza | Jedna zsynchronizowana grupa klatek na przejście; pliki z poszczególnych kamer na każdym wybranym poziomie eksportu (surowe/po usunięciu matrycy/radiancja/odbicie/podgląd/indeks) + plik sidecar `.daq` | Przycisk**Przycisk **Przechwyć wszystko** + Ustawienia przechwytywania | `lattice array-capture` |
| **Nagraj wideo indeksowe** | Monitorowanie | Wyświetlany na żywo złożony obraz indeksowy — 8-bitowy, rozdzielczość podglądu, wbudowana tablica LUT. Z poziomu CLI

, bez otwartego okna GUI, backend renderuje NDVI

(lub `--expr`) tablicy po jej wyrównaniu; nagrywanie, które nie zapisuje żadnej klatki, kończy się niepowodzeniem | ● Nagrywanie wideo indeksowego (połączone macierze) | `lattice array-record` |
| **Seria surowych klatek → utworzenie filmu**| Analiza | Surowe klatki z czujnika przy pełnej częstotliwości przechwytywania + manifest + `.daq`, a następnie rekonstrukcja offline do skalibrowanego filmu przedstawiającego promieniowanie / odbicie / indeks, zsynchronizowanego czasowo z odczytami DAQ | ⦿ Nagranie serii surowych klatek →**Utworzenie filmu** | `lattice array-burst` → `lattice array-build-video` |

Ogólna zasada: jeśli piksele mają dostarczać *pomiarów*, użyj trybu przechwytywania lub serii zdjęć (klasa analityczna); jeśli chcesz tylko *obejrzeć lub zademonstrować* to, co zarejestrowała matryca, nagraj film indeksowy (klasa monitorowania).

### Ustawienia przechwytywania (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Ikona koła zębatego obok opcji **Capture All** otwiera panel ustawień przechwytywania (wymaga otwartego projektu — przechwycone dane są zapisywane w tym projekcie):

* **Tryb przechwytywania**:**Pojedynczy**(jedno przejście) /**Ciągły**(kolejne ujęcia; ograniczony liczbą ujęć, domyślnie 1, lub czasem trwania, domyślnie 10 s) /**Interwał** (timelapse: N ujęć co X interwałów, łącznie Y; domyślnie 1 co 5 s przez 1 minutę).
* **Typy eksportu na kamerę**: Raw, Debayered, Radiance, Reflectance, Preview, Index — domyślnie włączone są wszystkie dostępne opcje. Opcje Radiance/Reflectance są ukryte dla kamer z filtrem RGB

; **Reflectance pojawia się tylko wtedy, gdy kamera posiada czujnik światła DAQ** (własny lub odziedziczony po macierzy); opcja Index wymaga skonfigurowanego wyrażenia indeksującego.
* **Wyrównane**(dla całej macierzy, domyślnie**włączone**): dopasowuje eksportowane pliki elementów do profilu wyrównania macierzy, dzięki czemu eksporty są zarejestrowane pikselowo. Dane surowe pozostają zawsze nieprzekształcone, ale zawierają transformację w metadanych. Pliki elementów macierzy są zapisywane w natywnej orientacji każdej kamery (wszelkie ustawienia odbicia lustrzanego/obrotu są zapisywane w pliku, ale nie są stosowane).
* **Najszybsze przechwytywanie** (przełącznik): wyłącznie dane surowe + przypisany odczyt DAQ + bezpłatna kompozycja z indeksem łącznym, z pominięciem obliczeń kalibracyjnych w momencie przechwytywania w celu uzyskania maksymalnej szybkości — odbudowa promieniowania/odbicia/indeksu nastąpi później na podstawie zapisanego pliku `.daq`.
* Wybory pozostają zachowane wraz z projektem. Ukryte lub wstrzymane kamery są pomijane.
* Tryb **Ciągły** dla macierzy działa w ramach backendu Chloros

(nowość w wersji 1.3.0): gdy każda kamera z listy należy do macierzy, backend sam uruchamia pętlę przechwytywania, zapisując jedną grupę podczas wybierania kolejnej, bez wysyłania żądania dla każdego przechwycenia. Pełny dysk zatrzymuje działanie („Zatrzymano: dysk pełny. Zapisano N, F nie powiodło się”). Zobacz [Ustawienia i tryby przechwytywania](capture.md#capture-modes), aby zapoznać się z uwagą dotyczącą programu Microsoft Defender w zakresie wysokich częstotliwości przechwytywania.

Odpowiednik w interfejsie CLI

(ten sam punkt końcowy zaplecza, ta sama semantyka):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Kompresja TIFF

dla przechwytywanych obrazów to `deflate` (bezstratna, domyślna) lub `none` — pełne tabele flag, układ folderów przechwytywania oraz reguły ponownego przetwarzania znajdują się w [Podręczniku CLI

](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Parowanie czujnika światła DAQ

Podglądy z korekcją odbicia i oświetlenia wymagają danych dotyczących światła padającego z czujnika DAQ (podłączonego w zakładce **Czujniki światła**):

* W **wierszu macierzy**na pasku bocznym widoczna jest**pigułka „DAQ · wł./wył.”** — *włączona*, gdy ustawiono czujnik światła na poziomie macierzy **lub** gdy dowolna kamera wchodząca w skład macierzy posiada własny czujnik; w podpowiedzi podano dokładnie, który czujnik zasila daną kamerę.
* Przypisz ustawienie dla całej matrycy w ustawieniach matrycy → **Czujnik światła otoczenia**→ menu rozwijane**Czujnik światła**. Wybór ten pozostaje zachowany wraz z projektem, jest propagowany do każdej kamery wchodzącej w skład matrycy, a poszczególne kamery mogą nadal nadpisać to ustawienie, korzystając z własnego czujnika.
* Linia stanu poniżej pokazuje aktualny stan: **Wyłączony**→ „Oczekiwanie na pierwsze widmo…” →**„Aktywny — wszystkie kamery w macierzy mają skorygowane oświetlenie”** → lub, jeśli w ciągu ostatnich 3 s nie nadeszło żadne nowe widmo, komunikat o nieaktualności — nadal wykorzystywany jest ostatni odczyt (odczyty nigdy nie tracą ważności na ścieżce przechwytywania).

Po przypisaniu czujnika: udostępnia się typ eksportu „Odbicie”, podglądy na żywo są skorygowane pod kątem oświetlenia, predykcyjna automatyczna ekspozycja może wykorzystywać widmo, a każde przechwycenie odbicia zapisuje odczyt DAQ faktycznie użyty jako **plik `.daq`** w folderze przechwyceń `daq/`, dzięki czemu przechwycenie można później ponownie przetworzyć. Obok niego plik `.daq.cameras.json` określa kamery, których klatki opisuje dany odczyt; podczas ponownego przetwarzania program Chloros

dopasowuje odczyt wyłącznie do obrazów z tych kamer.

## Opcje CLI

dla `array-connect`

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatyczne wykrywanie wszystkich kamer LATTICE (wymagane ≥2) | **Pierwszy numer seryjny oznacza kamerę MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linia synchronizacji GPIO. |
| `--target-fps F` | auto | Częstotliwość wyzwalania urządzenia głównego. |
| `--binning {1,2,4}` | auto | Binning sprzętowy. Ustawienie „auto” oznacza pełny czujnik (bez binningu) zawsze, gdy pozwala na to częstotliwość klatek; opcja „connect never bins” służy wyłącznie do utrzymania wysokiej częstotliwości. |
| `--packed {auto,on,off}` | `auto` | [Pakowany transport 12-bitowy](#packed-12-bit-transport): `auto` stosuje kompresję tylko wtedy, gdy jest to korzystne, a host jest w stanie poradzić sobie z dekompresją. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ekspertowe nadpisanie wyboru warstwy synchronizacji. |
| `--wire-ceiling-mbps MB_PER_S` | wykrywane automatycznie | Limit przepustowości hosta w MB/s — forma pola **Wire Budget** w interfejsie CLI

. Należy go zmniejszyć, jeśli macierz zgłasza uszkodzone ramki. Jest zapisywany wraz z projektem, więc przy późniejszym ponownym połączeniu zostanie przywrócony. |
| `--no-recommend` | wyłączone | Pomiń etap analizy sieci. |
| `--no-ptp` | wyłączone | Wyłącz PTP (znaczniki czasu między kamerami nie będą wtedy porównywalne). |

`lattice array-list`, `array-status` i `array-disconnect` zarządzają sesją trwałą. Pełny opis podpoleceń, w tym wyrównanie (`align-calibrate` / `align-apply`) oraz narzędzia sieciowe, znajduje się w [Podręczniku CLI

§ chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); odpowiedniki w SDK

(`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) znajdują się w [Podręczniku SDK

](../reference/sdk-reference.md). W języku Python

budżet transmisji to `connect_array(..., wire_ceiling_mbps=120)`, a podział na dane uszkodzone na żywo i nigdy nieotrzymane znajduje się w [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
