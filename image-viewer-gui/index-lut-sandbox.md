# Index/LUT Sandbox

Index/LUT Sandbox to interaktywne środowisko robocze znajdujące się na pasku bocznym przeglądarki obrazów Chloros. Wybierasz formułę, przypisujesz do niej kanały z kamery, nadajesz jej kolor za pomocą gradientu i dostosowujesz zakres wartości — a obraz aktualizuje się na bieżąco podczas tych czynności. Od wersji 1.2.0 możesz również **zapisać to, co stworzyłeś**, zarówno dla pojedynczego obrazu, jak i dla całego projektu, bez konieczności ponownego przetwarzania.

## Do czego służy Sandbox

| Sandbox indeksu/LUT (interaktywny)        | Przetwarzanie projektu (partia)       |
| -------------------------------------- | -------------------------------- |
| Jeden obraz na raz, natychmiastowa informacja zwrotna  | Cały zbiór danych w jednym przebiegu     |
| Eksperymentalne i iteracyjne             | Wstępnie skonfigurowane ustawienia          |
| Renderowanie na żywo; zapis tylko na żądanie  | Zawsze zapisuje pliki końcowe      |
| Idealne do znalezienia odpowiednich ustawień | Najlepsze rozwiązanie, gdy ustawienia są już ostateczne |

{% hint style="success" %}
**Typowy przebieg pracy**: dostosowuj ustawienia w Sandboxie, aż wizualizacja będzie odpowiadać Twoim oczekiwaniom, a następnie albo wyeksportuj bezpośrednio z Sandboxa, albo skopiuj te same ustawienia indeksu i LUT do [Ustawień projektu](../project-settings/project-settings.md), aby podczas następnego przebiegu przetwarzania zostały one zastosowane do każdego obrazu.
{% endhint %}

***

## Otwieranie Sandboxa

1. Kliknij obraz w siatce — otworzy się on na pełnym ekranie w zakładce **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Kliknij ikonę **Przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, aby wysunąć lewy pasek boczny, jeśli nie jest jeszcze otwarty
3. Wybierz warstwę wielopasmową z menu rozwijanego warstw w prawym górnym rogu — zazwyczaj wybiera się opcję **RAW (odbicie)**, ponieważ wartości indeksu obliczone na podstawie skalibrowanego odbicia są porównywalne między obrazami

Pasek boczny wyświetla, od góry do dołu:

* nazwę obrazu i model aparatu
* przycisk **Eksportuj/Zapisz obraz(y)**— pojawia się po zaznaczeniu opcji**Indeks**lub**LUT*** pola wyboru **Indeks**i**LUT**
* panel konfiguracji wskaźnika
* panel **Wartości kursora** z odczytem, histogramem i kontrolką GSD

{% hint style="warning" %}
**Niedostępne dla kamer monochromatycznych.** W przypadku obrazu LATTICE M3M z jednym pasmem oba pola wyboru są nieaktywne, a podpowiedź brzmi: _„Niedostępne dla czujników monochromatycznych (M3M)”_ — indeks wielopasmowy nie jest zdefiniowany dla jednego pasma. Aby obliczyć indeksy z kamer M3M, należy połączyć co najmniej dwie z nich w wyrównany stos wielopasmowy i skorzystać z silnika indeksującego LATTICE.
{% endhint %}

***

## Stosowanie indeksu

1. Zaznacz pole **Indeks** u góry paska bocznego
2. Wybierz filtr swojej kamery z rozwijanego menu po lewej stronie (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Wybierz formułę indeksu z rozwijanego menu po prawej stronie — 27 wbudowanych formuł oraz dowolne własne formuły, które zapisałeś
4. Formuła wyświetla się poniżej w postaci wyrażenia matematycznego, z pustym kółkiem w każdym polu pasma. **Przeciągnij kolorowe kółko kanału na pole**, aby je przypisać
5. Gdy wszystkie pola używane w formule zostaną przypisane, obraz zostanie zaktualizowany i wyświetli wartości indeksu
6. Najedź kursorem na obraz, aby odczytać wartości; panel **Wartości kursora** dodaje wiersz indeksu z wartością znajdującą się pod kursorem

Kliknij dwukrotnie przypisane miejsce, aby je wyczyścić. Niekompletna formuła to normalny stan podczas przeciągania, a nie błąd — obraz po prostu nie aktualizuje się, dopóki formuła nie zostanie uzupełniona.

Kółka kanałów są oznaczone kolorami: czerwony = Red, zielony = Green, niebieski = Blue, pomarańczowy = Orange, cyjan = Cyan, fioletowy = NIR, magenta = RE. Te same kolory są używane dla kropek kanałów i krzywych histogramu w panelu „Wartości kursora”.

### Przykład NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Pełny opis wzorów — wszystkie trzy listy ustawień wstępnych oraz informacje o tym, które nazwy działają w poszczególnych miejscach — znajduje się w sekcji [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md).

### Z zaznaczoną opcją „Indeks”, ale bez tabeli LUT

Obraz jest rysowany w **skali szarości**, rozciągnięty między dwiema wartościami progowymi. Jest to zamierzone: obraz indeksowy stanowi dane skalarne, a skala szarości jest jego wiernym odwzorowaniem. Aby uzyskać kolor, należy dodać tabelę LUT.***

## Praca z tabelami LUT (Look-Up Tables)**Tabela odnośników** przyporządkowuje wartości indeksowe do kolorów: na wejściu NDVI 0,65, na wyjściu określony odcień zieleni. Nie zmienia to danych — zmienia jedynie sposób ich odczytu.

### Dodawanie tabeli LUT

1. Kliknij przycisk **„+ Dodaj LUT”** w sekcji „<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">” poniżej wzoru
2. Wybierz gradient kolorów
3. Ustaw minimalną i maksymalną wartość przycinania
4. Wybierz tryb przycinania
5. Zaznacz pole wyboru **LUT** na pasku bocznym, aby ją wyrenderować

Pole wyboru LUT pozostaje nieaktywne, dopóki tablica LUT nie zostanie faktycznie skonfigurowana w indeksie.

### Wybór gradientu koloru

Najedź kursorem na **pasek gradientu**, aby otworzyć listę ustawień wstępnych — Chloros zawiera**siedem** gotowych ustawień gradientów:

| # | Gradient                            | Kształt                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Żółty → Green (**domyślny**)  | Rozchodzący się — odpowiada typowemu postrzeganiu roślinności: zielony = zdrowy |
| 2 | Fioletowy → Żółty → Green             | Rozbieżny, z wyraźnie zaznaczonym dolnym zakresem                                  |
| 3 | Brązowy → Biały → Blue                | Rozbieżny wokół jasnego punktu środkowego                                   |
| 4 | Czarny → Fioletowy → Różowy → Jasnożółty | Sekwencyjny, od ciemnego do jasnego                                           |
| 5 | Red → Żółty → Blue                 | Rozbieżny wokół jasnego punktu środkowego                                   |
| 6 | Fioletowy → Blue → Green → Żółty      | Sekwencyjny, od ciemnego do jasnego                                           |
| 7 | Orange → Biały → Fioletowy             | Rozchodzące się wokół jasnego punktu środkowego                                   |

Gradient **rozbieżny**umieszcza neutralny kolor w środku okna, co sprawdza się dobrze, gdy punkt środkowy ma określone znaczenie (próg, data odniesienia). Gradient**sekwencyjny** przebiega monotonicznie od ciemnego do jasnego, co sprawdza się dobrze w przypadku wielkości, które mają tylko wartości „więcej” i „mniej”.

Każdy preset zawiera siedem punktów kolorów. Kliknij preset, a obraz natychmiast się zaktualizuje (gdy pole LUT jest zaznaczone).

### Edycja punktów kolorów

Pod paskiem gradientu znajduje się rząd próbek kolorów, po jednej na każdy punkt:

* **Zmiana koloru**: kliknij próbkę, aby otworzyć selektor kolorów (koło kolorów, suwaki RGB/HSV lub kod szesnastkowy, np. `#FF0000`)
* **Dodaj punkt**: kliknij przycisk**+** na końcu rzędu — zostanie dodany biały punkt
* **Usuń punkt**:**kliknij dwukrotnie** próbkę
* **Zachowaj edytowany gradient**: kliknij ikonę zapisu obok paska gradientu, aby dodać edytowany gradient do listy ustawień wstępnych, dzięki czemu będziesz mógł go ponownie wybrać

Gradient skonfigurowany dla indeksu jest zapisywany wraz z tym indeksem w ustawieniach projektu, dzięki czemu pozostaje zachowany po zamknięciu i ponownym otwarciu projektu.

**Mniejsza liczba punktów**tworzy wyraźne strefy, które odczytuje się jako klasyfikację;**większa liczba punktów** zapewnia płynne, niemal fotograficzne przejścia. Trzy do pięciu punktów nadaje się do slajdów prezentacyjnych i map klasyfikacyjnych; sześć do dziesięciu — do ogólnej analizy; piętnaście lub więcej — do szczegółowej analizy i rysunków publikacyjnych.

### Ustawianie zakresu wartości

Element sterujący progiem to **suwak z dwoma uchwytami**o zakresie od −1 do +1, z edytowalnym polem tekstowym na każdym końcu do wpisania dokładnych wartości oraz przyciskiem**AUTO**.

* Przeciągnij dowolny suwak lub wpisz liczbę w odpowiednim polu i naciśnij Enter
* **AUTO**ustawia zakres na**

2. i 98. percentyl** prawidłowych wartości indeksu obrazu — jest to dobry punkt wyjścia, który pomija wartości odstające. Chloros adaptacyjnie zaokrągla wynik do 4 miejsc po przecinku dla bardzo wąskiego zakresu, do 3 dla wąskiego zakresu, a w pozostałych przypadkach do 2
* Każda ręczna regulacja ma pierwszeństwo przed trybem AUTO, dopóki nie naciśniesz ponownie przycisku AUTO

Przykładowe okna NDVI:

| Cel                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Pokaż wszystko                         | −1,0 | 1,0 |
| Tylko roślinność, z wyłączeniem gleby i wody | 0,2  | 0,9 |
| Tylko zdrowa roślinność                 | 0,5  | 0,9 |
| Podkreśl stres                        | 0,2  | 0,5 |

Zawężenie przedziału zwiększa kontrast w obszarze zainteresowania i powoduje, że wszystko inne znajduje się poza zakresem — gdzie **tryb przycinania** decyduje o tym, co się z tym stanie.***

## Tryby przycinania

Gdy wartość indeksu piksela wykracza poza okno min/max, tryb przycinania decyduje o tym, jak zostanie on narysowany.

| Etykieta listy rozwijanej                  | Zapisana wartość      | Piksele poza zakresem są rysowane jako                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum i maksimum** (domyślnie) | `clip`            | Najbliższy kolor końcowy gradientu — wartości poniżej minimum przyjmują pierwszy kolor, wartości powyżej maksimum przyjmują ostatni |
| **Przezroczyste tło**      | `transparent`     | Całkowicie przezroczyste (prawdziwa wartość alfa)                                                                                                  |
| **Tło indeksowe**| `indexColor`      | Skala szarości, rozciągnięta na**cały** zakres indeksu obrazu, dzięki czemu struktura poza zakresem jest nadal widoczna w odcieniach szarości                |
| **Oryginalne tło**         | `backgroundColor` | Sam obraz bazowy, więc nakładka kolorystyczna znajduje się na prawdziwej scenie                                                |

| Tryb                       | Najlepsze zastosowanie                               | Wygląd                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum i maksimum**      | Pełne wyświetlanie danych, analiza naukowa | Każdy piksel jest pokolorowany                      |
| **Przezroczyste tło** | Nakładki GIS, izolowanie przedziału wartości   | Kolor wewnątrz okna, nic poza nim |
| **Tło indeksowe**       | Podkreślenie przy zachowaniu kontekstu danych    | Kolor wewnątrz, szarość na zewnątrz               |
| **Tło oryginalne**    | Raporty i prezentacje              | Kolor wewnątrz, zdjęcie na zewnątrz         |

{% hint style="info" %}
**Piksele bez danych są zawsze przezroczyste, w każdym trybie.** Piksel, którego indeks nie jest skończony (dzielenie przez 0/0) lub wynosi dokładnie −1,0 lub +1,0 (wskaźniki nasycenia, wynikające z sytuacji, gdy jedno pasmo wskazuje zero, a drugie nie), jest traktowany jako brak danych, a nie jako wartość skrajna. Dzięki temu prześwietlone jasne obszary i martwe cienie nie pojawiają się w skali kolorów, zamiast tego nie są one przedstawiane jako najbardziej skrajne odczyty w kadrze. Ta sama zasada określa, które piksele zasilają progi AUTO oraz histogram indeksów, dzięki czemu wszystkie trzy są ze sobą zgodne.
{% endhint %}

Przezroczystość jest zachowana, gdy eksport jest zapisywany w formacie PNG. Nie można jej przedstawić w formacie JPG.

***

## Odczytywanie wartości podczas regulacji

Panel **Wartości kursora** poniżej panelu konfiguracyjnego służy jako przyrząd pomiarowy dla środowiska testowego:

* Przesuń kursor nad obrazem i odczytaj wartości źródłowe dla poszczególnych kanałów oraz wartość indeksu w osobnym wierszu
* Włącz przycisk **INDEX** nad histogramem, aby zobaczyć rozkład wartości indeksu w klatce, z dwoma progami przycięcia zaznaczonymi pomarańczowymi liniami przerywanymi oraz wartością kursora przedstawioną białą linią — jest to najszybszy sposób na wybranie okna, które faktycznie zawiera dane
* Włącz opcję **CURSOR**, aby wyświetlić linie znaczników przy wartościach pod wskaźnikiem
* Powiększ obraz powyżej 60× (mniej, jeśli ustawiono rozmiar bloku GSD), aby podświetlić poszczególne wyświetlane piksele z wartością zmienną

Praktyczna procedura:

1. Zanotuj wartości nad zdrową roślinnością, roślinnością zestresowaną, nagą glebą i wodą
2. Sprawdź, gdzie te skupiska znajdują się na histogramie indeksu
3. Ustaw wartości min/max tak, aby objąć interesujący Cię skupisko
4. Wybierz tryb przycinania — _Original Background_ (Oryginalne tło) sprawia, że otoczenie sceny pozostaje widoczne

***

## Eksportowanie z Sandboxa

Wszystko powyżej jest podglądem na żywo, dopóki nie zostanie zapisane. Przycisk **Eksportuj/Zapisz obraz(y)** u góry paska bocznego otwiera panel, który przesuwa się nad paskiem bocznym (zamiast zakrywać obraz, dzięki czemu nadal widać to, co się ocenia).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opcje

| Opcja                          | Efekt                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Zastosuj do bieżącego obrazu**      | Zapisuje dokładnie ten obraz, który jest wyświetlany, z tymi ustawieniami                                                                                                |
| **Zastosuj do wszystkich obrazów w projekcie** | Ponownie stosuje identyczną konfigurację do każdego obrazu w projekcie. Obrazy bez pasm wymaganych przez ten indeks są pomijane, a nie traktowane jako błędy |
| **Pasek gradientu indeksu/LUT**      | Zapisuje również osobny obraz legendy dla każdego eksportu, z oznaczonym zakresem wartości                                                                     |
| **Histogram indeksu**             | Zapisuje również osobny obraz histogramu dla każdego eksportu, pokazujący minimalne i maksymalne wartości danych oraz progi przycięcia                                               |

Jeśli wartość **rozmiaru bloku GSD** na karcie obrazu jest większa niż 1, w panelu pojawi się odpowiedni komunikat przed zatwierdzeniem: eksport zapisuje to, co widzisz, łącznie z uśrednieniem blokowym. Jeśli chcesz uzyskać pełną rozdzielczość, najpierw ustaw kontrolkę GSD z powrotem na 1.

### Gdzie trafiają pliki

Każde kliknięcie przycisku **Eksportuj**powoduje utworzenie**nowego, nigdy wcześniej nieużywanego folderu**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Przykłady: `Sandbox_Exports/NDVI_LUT_001/`, a następnie `Sandbox_Exports/NDVI_LUT_002/` dla kolejnego przebiegu. Numeracja jest generowana na podstawie skanowania zawartości dysku, dzięki czemu pozostaje zachowana nawet po ponownym uruchomieniu programu i ręcznym usunięciu folderów. Nic nigdy nie jest nadpisywane — głównym celem środowiska Sandbox jest porównywanie bieżącej próby z poprzednią.

Wewnątrz folderu, dla każdego obrazu:

| Plik                                                   | Zawartość                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Wyrenderowany obraz, piksel po pikselu taki, jak wyświetlała go przeglądarka |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Plik pomocniczy z paskiem gradientowym, jeśli został zamówiony                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Plik pomocniczy z histogramem indeksów, jeśli został zamówiony                  |

Oba pliki towarzyszące są zawsze zapisywane w **pełnej rozdzielczości**, nawet gdy główny obraz jest uśredniony blokowo: rozmiar bloku odpowiada rozdzielczości wyświetlacza, a oba pliki towarzyszące odczytują rzeczywiste wartości indeksu dla każdego piksela. Zawierają one również więcej informacji niż wersje wyświetlane na ekranie — oba pliki zawierają oznaczenie okna rozciągania _oraz_ rzeczywiste wartości minimalne i maksymalne danych, dzięki czemu zapisana legenda jest nadal czytelna nawet po upływie miesięcy, bez konieczności otwierania projektu.

### Postęp i wyniki

Eksport całego projektu zajmuje kilka minut, więc proces ten informuje o postępie na kanale na żywo, zamiast blokować działanie programu:

* Pasek postępu wyświetla `current / total` oraz plik, który jest zapisywany
* Po zakończeniu w panelu wyświetlane są informacje o liczbie wyeksportowanych obrazów, liczbie pominiętych oraz ścieżka do folderu wyjściowego
* Pominięte obrazy są wymienione wraz z przyczyną (wyświetla się maksymalnie pięć, a następnie wiersz „+N więcej”). Najczęstszą przyczyną jest warstwa, która nie posiada kanałów wymaganych przez ten indeks
* Jeśli **żaden** obraz w projekcie nie mógł skorzystać z indeksu, operacja zgłasza niepowodzenie zamiast pozostawić pusty folder

W danym momencie może być uruchomiony tylko jeden eksport w piaskownicy. Próba uruchomienia drugiego eksportu w trakcie trwania pierwszego jest odrzucana z wyraźnym komunikatem, zamiast dopuszczać do konfliktu dwóch operacji walczących o ten sam plik projektu.

### Siatka wyświetla przebieg

Każdy zakończony przebieg pojawia się jako osobny przycisk na pasku narzędzi [siatki obrazów](image-grid.md), oznaczony jako `<IndexName> <Index|LUT> <NNN>`. W ten sposób można porównywać przebiegi: należy wykonać dwa eksporty z różnymi gradientami lub progami, a następnie przełączać się między dwoma przyciskami w siatce.

***

## Niestandardowe formuły indeksu (Chloros+)

{% hint style="info" %}
**Gdzie je tworzyć**: na pasku bocznym Sandbox lub w**Ustawieniach projektu** przed przetwarzaniem. Oba zapisują się do tej samej listy na poziomie projektu.
{% endhint %}

1. Otwórz kalkulator formuł niestandardowych z menu rozwijanego formuł indeksowych (wymagane jest zalogowanie się przy użyciu kwalifikującej się subskrypcji Chloros+)
2. Wpisz formułę, używając **symboli przedziałów pasmowych** `x`, `y`, `z`, `a`, `b`, `c` — nie są to nazwy pasm
3. Dostępne operatory: `+`, `-`, `*`, `/`, `^` oraz `()` do grupowania
4. Dostępne funkcje: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nadaj nazwę i zapisz — pojawi się ona na dole listy rozwijanej formuł, a jej sloty można przypisać, przeciągając kółka kanałów, dokładnie tak samo jak w przypadku wbudowanych ustawień wstępnych

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Formuły niestandardowe są dostępne wyłącznie w interfejsie graficznym.** Opcja CLI/SDK `--indices` rozszerza listę 22 wbudowanych nazw presetów i pomija w tle wszystko inne, w tym formuł niestandardowych. Aby zastosować formułę niestandardową w trybie wsadowym, skonfiguruj ją w ustawieniach projektu i uruchom przetwarzanie lub skorzystaj z opcji eksportu „Zastosuj do wszystkich obrazów projektu” w środowisku Sandbox.
{% endhint %}

***

## Rozwiązywanie problemów

### „Ta warstwa nie posiada kanałów wymaganych przez ten indeks”

Formuła odczytuje pozycję kanału, której bieżąca warstwa nie posiada — na przykład indeks z trzema slotami w pliku jedno- lub dwukanałowym. Przełącz się na warstwę wielopasmową (odbiciową lub po usunięciu matrycy) lub wybierz indeks pasujący do filtra Twojej kamery.

### „Nie udało się połączyć z modułem przetwarzania obrazu”

Moduł nie odpowiada. Sprawdź zakładkę „Log”; jeśli moduł przetwarzający jest ponownie uruchamiany, środowisko testowe (Sandbox) przywróci działanie samodzielnie po jego ponownym uruchomieniu.

### Obraz nie zmienił się po przeciągnięciu okręgu

Formuła nie jest jeszcze kompletna. Niekompletna formuła jest traktowana jako normalny stan w trakcie przeciągania — nic nie jest renderowane i nie zgłaszany jest żaden błąd. Wypełnij wszystkie pola używane przez formułę.

### Cały obraz ma jeden kolor

Twoje okno klipu prawdopodobnie znajduje się daleko poza zakresem danych. Naciśnij **AUTO**, aby przyciągnąć je do 2. lub 98. percentyla, lub włącz histogram**INDEX**, aby zobaczyć, gdzie faktycznie znajdują się dane.

### Wyeksportowane kolory nie odpowiadają temu, co widziałem

Powinny — ścieżka eksportu jest celowym odzwierciedleniem podglądu na żywo, łącznie z alfa w trybie przycinania, a uśrednianie blokowe jest stosowane _po_ pokolorowaniu, dokładnie tak jak robi to przeglądarka. Jeśli występują różnice, sprawdź, czy rozmiar bloku GSD nie uległ zmianie między przeglądaniem a eksportem.

***

## Kolejne kroki

* [**Warstwy obrazu**](image-layers.md) — na której warstwie uruchomić indeks i co oznaczają jej wartości
* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — szczegółowe informacje na temat odczytu kursora, histogramu i kontroli GSD
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — wszystkie ustawienia wstępne dla każdej powierzchni
* [**Ustawienia projektu**](../project-settings/project-settings.md) — zapisanie znalezionych ustawień w cyklu przetwarzania
