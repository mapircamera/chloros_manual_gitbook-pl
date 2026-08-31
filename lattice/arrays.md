# Układy wielokamerowe

**Układ**LATTICE to dwie lub więcej kamer LATTICE połączonych w jedną zsynchronizowaną całość. Jedna z kamer pełni rolę**głównej**: wysyła ona sprzętowy impuls wyzwalający GPIO na wspólnej linii synchronizacji (domyślnie**Line2**), dzięki czemu wszystkie kamery rejestrują ten sam moment. Chloros dodaje synchronizację czasową PTP, podgląd na żywo (kafelki z poszczególnych kamer lub pojedynczy, wyrównany kompozyt wielopasmowy) oraz zsynchronizowane przechwytywanie — każdy przebieg przechwytywania generuje jedną**grupę klatek**, w której wszystkie kamery mają ten sam znacznik czasu i identyfikator klatki (podawany jako `fid:N` na wyjściu przechwytywania).

Macierze służą do generowania wskaźników wegetacyjnych przez kamery monochromatyczne (M3M) — jedna kamera dostarcza jedno pasmo, a macierz wyrównuje je w stos wielopasmowy. Zobacz [Kamery monochromatyczne i wskaźniki wegetacyjne](mono-indices.md).

Istnieją trzy równoważne sposoby podłączenia macierzy, a wszystkie z nich uruchamiają ten sam przepływ „smart-prep”:

| Interfejs | Punkt wejścia |
| --- | --- |
| GUI | Zakładka „Kamery” → **Podłącz układ** (niebieski przycisk) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (pierwszy numer seryjny = urządzenie główne) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (pierwszy numer seryjny = urządzenie nadrzędne) |

Funkcja Smart-prep wykonuje kolejno: test dostępności sieci (ping ICMP DF + sonda GVSP), wybór warstwy synchronizacji, automatyczne zmniejszenie rozmiaru ramki w celu dostosowania do łącza, włączenie PTP, automatyczny wybór formatu pikseli dla każdej kamery, automatyczne ustawienie ekspozycji na podstawie zapisanego stanu każdej kamery oraz konfigurację wyzwalacza GPIO na linii Line2.

{% hint style="info" %}
Aby którekolwiek z tych działań zadziałało, kamery muszą być dostępne w sieci — zobacz [Podłączanie kamer](connecting.md) w celu wykrywania, adresowania oraz pobrania kalibracji przy pierwszym połączeniu. W przypadku zestawów wielokamerowych ustawienia pierścienia odbiorczego karty sieciowej hosta mają równie duże znaczenie jak prędkość łącza; pełna tabela objawów i rozwiązań znajduje się w [CLI Materiały referencyjne § Konfiguracja i dostrajanie karty sieciowej hosta](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Okno dialogowe „Podłącz macierz”

Zakładka „Kamery” → **Podłącz macierz**otwiera trzyetapowego kreatora:**Wybierz → Tryb wyświetlania → Ustawienia**.

### Krok 1 — Wybór kamery głównej i kamer

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

podrzędnych Okno dialogowe skanuje sieć zaraz po otwarciu („Skanowanie sieci...”), a następnie sprawdza okablowanie wyzwalacza GPIO („Sprawdzanie okablowania GPIO...”). Do utworzenia macierzy potrzebne są co najmniej **2 kamery**.

Sprawdzanie okablowania wstępnie wypełnia wybór ról, jeśli to możliwe, i wyświetla jeden z trzech komunikatów:

| Komunikat | Znaczenie |
| --- | --- |
| „Wykryto urządzenie główne GPIO — wybory wstępnie wypełnione” (zielony) | Test wykrył topologię wyzwalania; pola wyboru dla urządzenia głównego i podrzędnych są już zaznaczone. |
| „Nie wykryto urządzenia nadrzędnego — sprawdź kabel GPIO” (pomarańczowy) | Żadna kamera nie wykryła impulsu wyzwalającego; sprawdź okablowanie synchronizacyjne. Nadal możesz ręcznie wybrać role. |
| „Brak kabla synchronizacyjnego: {numery seryjne}” (pomarańczowy) | Wymienione kamery nie mają podłączonego kabla synchronizacyjnego. |

Tabela kamer zawiera kolumny **Kamera / Numer seryjny / IP / Master (radio) / Slave (pole wyboru)**:

* Wybierz dokładnie **jednego mastera**i**jednego lub więcej slave&#x27;ów**. Ponowne kliknięcie pola wyboru obecnego mastera spowoduje jego wyczyszczenie.
* Kamery oznaczone jako **„Brak kabla synchronizacji”** nigdy nie mogą zostać wybrane jako urządzenia podrzędne — urządzenie podrzędne bez okablowania wyzwalającego czekałoby w nieskończoność na sygnał z linii synchronizacji i dostarczałoby martwy sygnał. Zamiast tego podłącz tę kamerę jako kamerę autonomiczną.
* Kamery już podłączone w trybie samodzielnym *nie* są wyłączane: podłączenie do macierzy zwalnia sesję samodzielną i ponownie otwiera kamerę w ramach macierzy.

**Dalej: Tryb wyświetlania →**staje się aktywne po wybraniu urządzenia nadrzędnego i co najmniej jednego urządzenia podrzędnego.**Ponowne skanowanie** ponownie uruchamia wykrywanie i sondę okablowania.

{% hint style="warning" %}
**Anuluj** jest wyłączone podczas trwania skanowania lub testu okablowania — anulowanie w trakcie testu może spowodować awarię kamery SDK z oprogramowaniem sprzętowym LATTICE. Należy poczekać na zakończenie działania ikony obrotowej.
{% endhint %}

### Krok 2 — Tryb

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

wyświetlania | Tryb | Co otrzymujesz |
| --- | --- |
| **Oddzielne kamery** | Jedna kafelka na żywo na każdą kamerę, wszystkie uruchamiane jednocześnie, dzięki czemu klatki pozostają zsynchronizowane. Każda kamera zachowuje własny kolor i ustawienia. |
| **Kamery połączone** *(domyślnie)* | Pojedynczy kafelek renderujący wyrównany, wielopasmowy kompozyt NDVI/index. Kamery mają wspólny kolor macierzy. |

Tryb wyświetlania zmienia jedynie sposób prezentacji podglądu na żywo — sposób przechwytywania obrazu jest taki sam w obu trybach.

### Krok 3 — Ustawienia matrycy i przewidywany wynik

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Po wejściu do tej sceny Chloros zwraca się do zaplecza z prośbą o **rekomendację**i automatycznie stosuje kombinację obszaru zainteresowania (ROI) oraz binningu, która pasuje do pierścienia odbiorczego karty sieciowej (NIC) (preferuje binning zamiast kadrowania obszaru zainteresowania (ROI), ponieważ binning zachowuje pełne pole widzenia). Każda wprowadzona zmiana powoduje ponowne przeprowadzenie analizy na żywo i aktualizację panelu**Przewidywany wynik** po prawej stronie.

Lewa kolumna — ustawienia:

| Element sterujący | Opcje | Domyślne | Uwagi |
| --- | --- | --- | --- |
| **ROI (pole widzenia)** | Pełne (2048×1536) / Połowa (1024×768) / Ćwierć (512×384) | Pełne | Kadrowanie czujnika: kadrowanie do połowy/ćwiartki w mniejszy obszar przy natywnym rozstawie pikseli. |
| **Łączenie pikseli** | 1× / 2× (suma 2×2) / 4× (suma 4×4) | 1× | Binning sprzętowy: 2×2 = pełne pole widzenia przy jednej czwartej kosztu przesyłu; 4×4 = pełne pole widzenia przy 1/16. Ukryte, jeśli kamery nie obsługują binningu. |
| **Obraz przesyłany** (odczyt) | — | — | Szerokość × wysokość po binningu faktycznie przesyłana, zaokrąglona do wielokrotności 16 (minimum 64). |
| **Rozdzielczość pinowa**| pole wyboru | wyłączone | Chloros zazwyczaj automatycznie zwiększa binning przy połączeniu, gdy prognozowana szybkość spadnie poniżej**1,5 klatki na sekundę**. Zablokowanie rozdzielczości pozwala zachować wybrany rozmiar klatki i akceptuje niższą częstotliwość — a także sprawia, że konfiguracja z nadmiernymi wymaganiami skutkuje twardym odrzuceniem połączenia zamiast automatycznego obniżenia częstotliwości. |
| **Częstotliwość wyzwalania** | 0,5–60 klatek na sekundę, krok 0,1 | puste = auto | Częstotliwość wyzwalania urządzenia nadrzędnego. Pozostaw puste, aby Chloros sam ją obliczył. |
| **Budżet przepustowości**| 20–2000 MB/s, krok co 10 | puste = auto | Ile host może faktycznie przyjąć, w MB/s —**jedyna wartość, od której zależy cała alokacja macierzy.** Wykrywane automatycznie przez kartę sieciową. Należy obniżyć tę wartość, jeśli macierz zgłasza uszkodzone ramki: wykryta wartość zawyża możliwości kart USB i współdzielonych przełączników. Zmiana tej wartości powoduje ponowne uruchomienie prognozy w czasie rzeczywistym. |

Prawa kolumna — **Prognozowany wynik**:

* **Poziom synchronizacji** — „Jednoczesne przechwytywanie” (zielony), „Jednoczesne przechwytywanie (FTD – rozłożone w czasie wysyłanie)” (zielony), „Przechwytywanie rozłożone w czasie (odchylenie 100 ms)” (pomarańczowy) lub „Konfiguracja zbyt duża” (czerwony).
* **Prognoza fps** — pokazana jako zakres („słabe → jasne”), ponieważ częstotliwość zsynchronizowanej macierzy jest ograniczona czasem naświetlania najwolniejszej kamery.
* **Linia karty sieciowej** — prędkość łącza i stała przepustowość („karta sieciowa {mbps} Mbps · stała {N} MB/s”).
* **Sprawdzenie serii symulowanych emisji** — czy pierścień odbiorczy karty sieciowej hosta może przyjąć jedną równoczesną serię danych ze wszystkich kamer („Seria symulowanych emisji: X MB · dostępny pierścień karty sieciowej: Y MB ✓/✗”).
* **Sprawdzenie budżetu łącza** — zagregowane zapotrzebowanie w stanie ustalonym w porównaniu z bezpiecznym pułapem łącza chroniącym przed kolizjami („Budżet łącza: {demand} MB/s wymagane przez {n} kamer · pułap {ceiling} MB/s ✓/✗ przekroczenie limitu”).
* **„Maksymalna liczba kamer na tym łączu: {n} — ustalona przez minimalną przepustowość na kamerę, więc grupowanie nie zwiększa tej liczby.”** — wyświetla się, gdy zbliżasz się do (lub przekraczasz) limit liczby kamer.
* **„Przy tych ustawieniach BĘDĄ WYSTĘPOWAŁY UTRATY KLATEK”.**— czerwone ostrzeżenie z podaniem przyczyny ze strony zaplecza, wraz z listą przeszkód i niebieskimi**sugestiami rozwiązań** („Aby zmieścić tę matrycę w sieci” / „Aby odblokować jednoczesne przechwytywanie”).

Opcja **Zastosuj i połącz** jest zablokowana do momentu pojawienia się prognozy, a jej etykieta informuje, dlaczego jest odrzucana:

| Etykieta przycisku | Znaczenie | Co faktycznie pomaga |
| --- | --- | --- |
| „Analizowanie...” | Analiza wciąż trwa. | Poczekaj. |
| **„Zbyt wiele kamer dla tej sieci”**| Macierz przeciąża łącze (niepowodzenie kontroli agregacji). | Mniejsza liczba kamer, ramki typu jumbo na całej długości łącza lub szybsza karta sieciowa.**Mniejszy obszar zainteresowania (ROI) NIE pomoże** — patrz poniżej. |
| **„Zmniejsz ROI, aby włączyć”** | Przy tych ustawieniach ramki byłyby utracane (nie powiodła się kontrola burst/ring). | Zmniejsz obszar ROI, zwiększ binning lub napraw pierścień odbiorczy karty sieciowej. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Podczas podłączania może pojawić się zielony **panel pobierania kalibracji** z paskiem postępu dla każdego portu szeregowego: gdy kamera jest podłączana do urządzenia po raz pierwszy, Chloros pobiera z kamery fabryczny pakiet kalibracyjny o wielkości ~3,8 MB przez GigE (około 70 sekund na kamerę). Kamery zapisane w pamięci podręcznej nigdy nie wyświetlają tego panelu. Zobacz [Podłączanie kamer](connecting.md).

## Przepustowość: ile kamer można podłączyć

To, ile kamer może obsłużyć macierz, zależy od właściwości łącza, a nie od Chloros, więc dane dotyczące planowania znajdują się w instrukcji sprzętu: **[Planowanie przepustowości macierzy](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Jak Chloros je wykorzystuje: okno dialogowe połączenia uruchamia sondę sieciową, prognozuje osiągalną liczbę klatek na sekundę i wybiera odpowiedni poziom. Jeśli macierz nadmiernie obciąża łącze, odmawia połączenia zamiast cicho odrzucać pakiety — zobacz opisany powyżej panel z prognozowanymi wynikami.

## Gdy brakuje ramek

Kamera może nie figurować w opublikowanej grupie z dwóch zupełnie różnych powodów,
a każda z tych sytuacji wymaga odwrotnego rozwiązania. Chloros zlicza je oddzielnie, zamiast zgłaszać jedną
„niekompletną” liczbę, która nie określa żadnej z tych przyczyn:

| Co się stało | Co to oznacza | Gdzie szukać |
| --- | --- | --- |
| **Uszkodzona**— ramka dotarła, ale miała błędy strukturalne | Utrata pakietów GVSP na ścieżce sieciowej |**Przepustowość łącza**, pierścień odbiorczy karty sieciowej, ramki jumbo, przełącznik |
| **Nigdy nie dotarła**— żadna ramka w ogóle nie dotarła | Kamera nie uruchomiła się lub nic z niej nie wyszło |**Kabel synchronizacyjny M8**, linia synchronizacji, czy wszystkie urządzenia są uzbrojone |

Podział jest ponownie oceniany co 10 sekund podczas transmisji strumienia z macierzy. Powyżej 5% jest to
rejestrowane z podaniem obu wartości, a każdy uszkodzony bufor jest zgłaszany po raz pierwszy, gdy
wystąpi dla danej kamery, a następnie sumowany co minutę, aby długa sesja pozostała czytelna.

**Uszkodzone klatki z zerową liczbą „nigdy nie dotarło” oznaczają, że wyzwalanie i synchronizacja kablowa działają idealnie**,
a każda utracona klatka znajduje się na ścieżce sieciowej. Rozwiązaniem jest obniżenie **Wire Budget** i
ponowne podłączenie.

{% hint style="warning" %}
**Obniżenie częstotliwości wyzwalania nie pomaga w przypadku uszkodzonych klatek.** Tempo przesyłania pakietów
przez kamerę jest ustalane jednorazowo, w momencie nawiązania połączenia. Obniżenie częstotliwości wyzwalania zmienia częstotliwość występowania serii
pakietów, a nie szybkość, z jaką sama seria trafia do sieci. W przypadku zmierzonego zestawu z 4 kamerami
5-krotne zmniejszenie częstotliwości wyzwalania nie przyniosło żadnej zmiany, natomiast obniżenie budżetu przepustowości z 240 do
200 MB/s spowodowało, że w tej samej konfiguracji odsetek uszkodzonych ramek spadł z 10,4% do zera.
{% endhint %}

Działająca macierz nie może samodzielnie zmienić planu — należy ją odłączyć i podłączyć ponownie, aby moduł wybierający czas połączenia
mógł działać zgodnie z nowym limitem przepustowości.

### Karty sieciowe USB mają ograniczenie do 200 MB/s

Karta sieciowa USB podaje swoją prędkość łącza *Ethernet*, ale to, co faktycznie
może utrzymać, jest ograniczone przez magistralę USB i jej sterownik. Klatce USB 10GbE przypisywano
przepustowość rzędu około 1000 MB/s — wartość, której nikt nigdy nie zmierzył — a synchronizacja
czterech kamer z tym fikcyjnym rezerwowym pasmem powodowała uszkodzenie 6–18% klatek, podczas gdy macierz
nadal zgłaszała prawidłową docelową liczbę klatek na sekundę. Adaptery podłączane przez USB mają obecnie ograniczenie do
**200 MB/s**. Ograniczenie to jest wartością bezwzględną, a nie procentową, ponieważ limitem jest
magistrala: adapter USB 1 GbE osiąga około 80 MB/s i nie ma na niego wpływu.

Jeśli komputer hosta jest rzeczywiście szybszy niż limit, należy zwiększyć wartość **Wire Budget**, aby to odzwierciedlić.

## Synchronizacja czasu PTP

*Synchronizacja* klatek odbywa się za pomocą wyzwalacza sprzętowego; **PTP** (IEEE 1588 PTPv2) zapewnia porównywalne *znaczniki czasu* na wszystkich urządzeniach. Jest ona domyślnie włączona po podłączeniu macierzy:

* Backend hosta **Chloros pełni rolę serwera nadrzędnego PTP**. Kamery LATTICE i czujniki światła DAQ-E działają jako urządzenia podrzędne w domenie 0, dzięki czemu znaczniki czasu obrazów i widma DAQ są zsynchronizowane z jednym zegarem (~1 ms).
* `--no-ptp` (CLI) wyłącza tę funkcję na potrzeby prac laboratoryjnych — wówczas znaczniki czasu między kamerami **nie** są porównywalne.
* Sprawdź stan synchronizacji za pomocą CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Sama zakładka „Kamery” nie zawiera wskaźnika PTP; dostępne tam informacje dotyczące synchronizacji poszczególnych kamer to tylko do odczytu: **Rola**(Master/Slave),**Linia synchronizacji** oraz poziom możliwości macierzy. Stan PTP modułu DAQ-E jest wyświetlany w szczegółach czujnika na zakładce „Czujniki światła”.

## Widok

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

macierzy na żywo Główny obszar podglądu oferuje dwa układy (przełączanie w górnym pasku): **widok siatki**(każdy kafelek to komórka; można zmieniać kolejność przez przeciągnięcie, gdy kłódka siatki jest odblokowana) oraz**widok listy**(matryce na całej szerokości u góry, jedna aktywna kamera poniżej). Suwak**Powiększenie podglądu** zmienia rozmiar kafelków; przy szerokości komórki poniżej 200 pikseli nakładki z nazwą i liczbą klatek na sekundę ukrywają się automatycznie.**Tryb oddzielny** wyświetla jeden kafelek na kamerę. Każdy kafelek zawiera:

* nazwę kamery (w lewym górnym rogu),
* **odczyt liczby klatek na sekundę** (w prawym górnym rogu) — jest to *rzeczywista częstotliwość przechwytywania* kamery zgłaszana przez serwer, a nie częstotliwość odświeżania podglądu (podgląd na żywo jest ograniczony do 30 klatek na sekundę niezależnie od częstotliwości przechwytywania),
* kropkę stanu — zieloną (transmisja) / bursztynową (ładowanie) / czerwoną (błąd),
* **ikonkę wskazującą nieaktualną klatkę**, gdy przez 2 s nie nadeszła żadna nowa klatka — jest to normalne zjawisko trwające około 5 s po każdym podłączeniu/odłączeniu, podczas gdy serwer rozdziela przepustowość między kamery.**Tryb połączony**wyświetla pojedynczą kafelkę złożoną: backend przeprowadza odbayering, skalowanie, wyrównywanie, usuwanie szumów, konwersję do promieniowania na pasmo (plus odbicie DLS, gdy podłączony jest czujnik światła), ocenia wyrażenie indeksowe tablicy, stosuje tabelę LUT i przesyła wynik w formacie MJPEG. Do momentu wyrenderowania pierwszej wyrównanej klatki kafelek informuje o swoim stanie: „Przygotowywanie tablicy…”, „Kalibrowanie wyrównania…”, „Oczekiwanie na pierwszą klatkę…” lub — jeśli wyczerpie się limit ponownych prób automatycznego wyrównania (~30 s) — „Wymagane wyrównanie” wraz z przyciskiem**Kalibruj wyrównanie**.

Przydatne informacje dotyczące trybu połączonego:

* Obraz złożony jest zsynchronizowany z klatką kamery **głównej**. Celowanie AE-ROI i pomiar punktowy na obrazie złożonym są dokładne dla kamery głównej, a przybliżone dla kamer podrzędnych; użyj opcji**Podzielony widok** (ustawienia macierzy → „Pokaż kamery składowe”), aby uzyskać kafelki z dokładnością co do piksela dla poszczególnych kamer bez otwierania dodatkowych połączeń z kamerami.
* Opcja **Warstwy wyświetlania**(ustawienia macierzy; domyślnie wyłączona) pozwala wybrać warstwę pierwszego planu i tła — dowolną kamerę należącą do macierzy lub**Indeks**. Gdy warstwa pierwszego planu = Indeks, piksele poza zakresem minimalnym/maksymalnym LUT wyświetlają warstwę tła.
* **Rozdzielczość renderowania** (domyślnie 720p) ustawia wysokość strumienia na żywo *oraz* rozmiar eksportowanego obrazu kompozytowego. Obrazy z poszczególnych kamer są zawsze eksportowane w pełnej rozdzielczości.
* Wyrównanie jest obliczane dla każdej sesji i nigdy nie jest zapisywane — w sekcji wyrównania w panelu ustawień macierzy znajdziesz resztki RMS oraz przycisk „Rekalibruj”.

## Przechwytywanie: monitorowanie a analiza

Powierzchnie przechwytywania macierzy dzielą się wyraźnie na **klasa monitorowania**(rejestruje to, co widać) i**klasa analizy** (rejestruje surowe dane, kalibruje później):

| Przebieg pracy | Klasa | Co zapisuje | Interfejs użytkownika | CLI |
| --- | --- | --- | --- | --- |
| **Rejestracja**(zdjęcia) | Analiza | Jedna zsynchronizowana grupa klatek na przejście; pliki z poszczególnych kamer na każdym wybranym poziomie eksportu (surowe/po usunięciu efektu debayeringu/radiancja/odbicie/podgląd/indeks) + plik sidecar `.daq` |**Przycisk **Przechwyć wszystko** + Ustawienia przechwytywania | `lattice array-capture` |
| **Nagraj wideo indeksowe** | Monitorowanie | Wyświetlany na żywo złożony obraz indeksowy — 8-bitowy, rozdzielczość podglądu, wbudowana tablica LUT; wymaga otwartego strumienia na żywo | ● Nagraj wideo indeksowe (połączone tablice) | `lattice array-record` |
| **Seria surowych klatek → tworzenie filmu**| Analiza | Surowe klatki z czujnika przy pełnej częstotliwości przechwytywania + manifest + `.daq`, a następnie rekonstrukcja offline w skalibrowany film przedstawiający promieniowanie / odbicie / indeks, zsynchronizowany czasowo z odczytami DAQ | ⦿ Nagraj serię surowych klatek →**Utwórz film** | `lattice array-burst` → `lattice array-build-video` |

Ogólna zasada: jeśli piksele mają dostarczać *pomiary*, użyj trybu przechwytywania lub serii zdjęć (klasa analityczna); jeśli chcesz tylko *obejrzeć lub zademonstrować* to, co zarejestrowała matryca, nagraj film indeksowy (klasa monitorowania).

### Ustawienia przechwytywania (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Ikona koła zębatego obok opcji **Przechwytuj wszystko** otwiera okienko Ustawień przechwytywania (wymaga otwartego projektu — przechwycone dane są w nim zapisywane):

* **Tryb przechwytywania**:**Pojedynczy**(jedno przejście) /**Ciągły**(kolejne ujęcia; ograniczone liczbą przechwyceń (domyślnie 1) lub czasem trwania (domyślnie 10 s)) /**Interwał** (timelapse: N przechwyceń co X interwałów, łącznie Y; domyślnie 1 co 5 s przez 1 minutę).
* **Typy eksportu dla każdej kamery**: Raw, Debayered, Radiance, Reflectance, Preview, Index — domyślnie włączone są wszystkie dostępne opcje. Opcje Radiance/Reflectance są ukryte dla kamer z filtrem RGB;**opcja Reflectance pojawia się tylko wtedy, gdy kamera posiada czujnik światła DAQ** (własny lub odziedziczony po macierzy); opcja „Index” wymaga skonfigurowanego wyrażenia indeksowego.
* **Wyrównane**(dla każdej macierzy, domyślnie**włączone**): dopasowuje eksporty elementów do profilu wyrównania macierzy, dzięki czemu eksporty są zarejestrowane pikselowo. Dane surowe pozostają zawsze nieprzekształcone, ale zawierają transformację w metadanych.
* **Najszybsze przechwytywanie** (przełącznik): wyłącznie dane surowe + przypisany odczyt DAQ + bezpłatna kompozycja z indeksem łącznym, pomijająca obliczenia kalibracyjne w momencie przechwytywania w celu uzyskania maksymalnej szybkości — odtworzenie promieniowania/odbicia/indeksu nastąpi później na podstawie zapisanego `.daq`.
* Zaznaczenia pozostają zachowane wraz z projektem. Ukryte lub wstrzymane kamery są pomijane.

Odpowiednik CLI (ten sam punkt końcowy zaplecza, ta sama semantyka):

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

Kompresja TIFF dla nagrań to `deflate` (bezstratna, domyślna) lub `none` — pełne tabele flag, układ folderu przechwytywania oraz reguły ponownego przetwarzania znajdują się w [CLI Dokumentacja](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Parowanie czujnika światła DAQ

Podglądy z korekcją odbicia i oświetlenia wymagają danych dotyczących światła padającego z czujnika DAQ (podłączonego w zakładce **Czujniki światła**):

* W **wierszu tablicy**na pasku bocznym wyświetla się przycisk**„DAQ · włączony/wyłączony”** — *włączony*, gdy czujnik światła na poziomie macierzy jest ustawiony **lub** gdy dowolna kamera w macierzy posiada własny czujnik; w podpowiedzi podano dokładnie, który czujnik zasila daną kamerę.
* Przypisz ustawienie dla całej matrycy w ustawieniach matrycy → **Czujnik światła otoczenia**→ menu rozwijane**Czujnik światła**. Wybór ten pozostaje zachowany w projekcie, rozprzestrzenia się na każdą kamerę w matrycy, a poszczególne kamery mogą nadal nadpisać to ustawienie własnym czujnikiem.
* Linia stanu poniżej informuje o aktualnym stanie: **Wyłączony**→ „Oczekiwanie na pierwsze widmo…” →**„Aktywne — wszystkie kamery w macierzy mają skorygowane oświetlenie”** → lub, jeśli w ciągu ostatnich 3 s nie nadeszło żadne nowe widmo, komunikat o nieaktualności — nadal wykorzystywany jest ostatni odczyt (odczyty nigdy nie tracą ważności na ścieżce przechwytywania).

Po przypisaniu czujnika: typ eksportu „Odbicie” staje się dostępny, podglądy na żywo są skorygowane pod kątem oświetlenia, predykcyjna automatyczna ekspozycja może korzystać ze spektrum, a każde przechwycenie odbicia zapisuje odczyt DAQ faktycznie użyty jako **`.daq`** obok obrazu, dzięki czemu przechwycenie można później ponownie przetworzyć.

## Opcje `array-connect` CLI

| Flaga | Domyślnie | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatyczne wykrywanie wszystkich kamer LATTICE (wymagane ≥2) | **Pierwsza kamera szeregowa jest MASTEREM.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linia synchronizacji GPIO. |
| `--target-fps F` | auto | Częstotliwość wyzwalania urządzenia głównym. |
| `--binning {1,2,4}` | auto | Binning sprzętowy. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ręczne nadpisanie wyboru poziomu synchronizacji przez eksperta. |
| `--wire-ceiling-mbps MB_PER_S` | wykrywane automatycznie | Limit przepustowości hosta w MB/s — forma pola **Wire Budget** w CLI. Należy ją zmniejszyć, jeśli macierz zgłasza uszkodzone ramki. Jest zapisywana wraz z projektem, więc późniejsze ponowne połączenie przywraca tę wartość. |
| `--no-recommend` | wyłączone | Pomiń etap analizy sieci. |
| `--no-ptp` | wyłączone | Wyłącz PTP (znaczniki czasu z różnych kamer nie będą wtedy porównywalne). |

`lattice array-list`, `array-status` i `array-disconnect` zarządzają sesją trwałą. Pełny opis podpoleceń, w tym wyrównywanie (`align-calibrate` / `align-apply`) oraz narzędzia sieciowe, znajduje się w [CLI Reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); odpowiedniki SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) znajdują się w [Odniesieniu SDK](../reference/sdk-reference.md). Od Python limit przewodów wynosi `connect_array(..., wire_ceiling_mbps=120)`, a podział na uszkodzone/nigdy nie dotarłe znajduje się w [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
