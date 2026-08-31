# Otwieranie obrazu na całym ekranie

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Obraz otwarty na pełnym ekranie, z selektorem warstw w prawym górnym rogu</p></figcaption></figure>

Przeglądarka obrazów Chloros to interfejs pełnoekranowy służący do przeglądania, analizowania i pomiaru obrazów. To właśnie w niej można odczytać **rzeczywiste wartości pikseli** — DN dla poszczególnych kanałów, procent odbicia lub natężenie promieniowania w W/m²/sr/nm — a nie rozciągnięty podgląd wyświetlany na ekranie.

## Dostęp do przeglądarki obrazów

### Z przeglądarki plików

1. Otwórz kartę **Przeglądarka plików** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknij dowolną **miniaturkę** w [siatce obrazów](image-grid.md)
3. Obraz otworzy się na pełnym ekranie w zakładce **Przeglądarka obrazów**

Obraz otworzy się w tym produkcie, który był wyświetlany w siatce. Jeśli siatka jest ustawiona na `RAW (Reflectance)`, to właśnie na tej warstwie się znajdziesz.

### Otwieranie paska bocznego przeglądarki obrazów

Kliknij ikonę **Przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na lewym pasku bocznym, aby rozsunąć panel analizy. Zawiera on, od góry do dołu:

* nazwę obrazu i model aparatu, z którego pochodzi
* przycisk **Eksportuj/Zapisz obraz(y)** (dostępny tylko wtedy, gdy aktywny jest indeks lub tabela LUT)
* pola wyboru **Indeks**i**LUT** oraz panel konfiguracji indeksu — zobacz [Środowisko testowe indeksu/LUT](index-lut-sandbox.md)
* panel **Wartości kursora**: odczyt dla poszczególnych kanałów, histogram warstwy oraz regulator GSD***

## Nawigacja i powiększanie

### Przeglądanie obrazów

* **Następny obraz**: przycisk → lub klawisz**→** (strzałka w prawo)
* **Poprzedni obraz**: przycisk ← lub klawisz**←** (strzałka w lewo)
* **Przejście do konkretnego obrazu**: wróć do siatki i kliknij jego miniaturę

Powiększenie i przesunięcie pozostają zachowane podczas przechodzenia między obrazami, dzięki czemu można przeglądać zestaw, pozostając w tej samej części kadru.

### Powiększenie

Powiększenie steruje się za pomocą **kółka myszy**, w krokach co 15%, z punktem odniesienia na kursorze — punkt pod wskaźnikiem pozostaje pod wskaźnikiem. Zakres jest ograniczony rozmiarem obrazu i okna: nie można pomniejszyć obrazu poniżej rozmiaru dopasowanego do okna, a górna granica jest ustalona przez natywną rozdzielczość obrazu.

W przeglądarce pełnoekranowej nie ma dedykowanych klawiszy do powiększania. (W siatce skróty **Ctrl + `+` / `−`** służą do zmiany rozmiaru miniatur — jest to inna funkcja.)

### Przesuwanie przy powiększeniu

Kliknij i przytrzymaj lewy przycisk myszy nad obrazem, a następnie przeciągnij. Przesuwanie jest ograniczone, więc obrazu nie da się przeciągnąć poza ekran.

### Analiza poszczególnych pikseli przy dużym powiększeniu

Gdy efektywne powiększenie przekroczy **60×**, Chloros rysuje ramkę podświetlającą wokół pojedynczego wyświetlanego piksela pod kursorem oraz wyświetla obok niego wartość.

Powiększenie „efektywne” uwzględnia rozmiar bloku GSD: przy rozmiarze bloku wynoszącym 8 podświetlenie pojawia się przy powiększeniu 7,5×, a nie 60×, ponieważ jeden wyświetlany piksel odpowiada już 8 × 8 pikseli źródłowych. Po zmniejszeniu powiększenia poniżej progu podświetlenie znika.

### Skróty klawiaturowe

| Klawisz                             | Gdzie       | Działanie                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Tryb pełnoekranowy | Następny obraz                          |
| **←**                           | Tryb pełnoekranowy | Poprzedni obraz                      |
| **Ctrl + R**                    | Tryb pełnoekranowy | Resetuj indeks/środowisko testowe LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Siatka        | Większe miniatury (4 piksele na naciśnięcie)  |
| **Ctrl + `−`**                  | Siatka        | Mniejsze miniatury (4 piksele na naciśnięcie) |***

## Wartości kursora

Po najechaniu kursorem na obraz panel **Wartości kursora** wyświetla wartości każdego kanału znajdującego się pod nim.

{% hint style="success" %}
**Są to rzeczywiste wartości pliku.** Obszar roboczy na ekranie jest 8-bitowym, rozciągniętym podglądem i nie może ich dostarczyć, więc Chloros pobiera próbki z rzeczywistego pliku produktu w celu wyświetlenia odczytu. Dlatego 12-bitowa klatka w formacie RAW wyświetla wartości powyżej 255, a warstwa promieniowania typu float32 wyświetla jednostki fizyczne.
{% endhint %}

### Znaczenie kolumn

Panel dostosowuje się do wyświetlanej warstwy:

| Wyświetlana warstwa              | Wyświetlane kolumny    | Uwagi                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Odbicie                        | **DN**i**%** | Procent jest obliczany na podstawie skali właściwej dla danego pliku — patrz poniżej                                      |
| Promieniowanie                        | **W/m²/sr/nm**   | Wartości fizyczne typu float; brak kolumny DN, ponieważ wartość DN nie ma tu znaczenia                           |
| Surowe / po usunięciu efektu bayera / podgląd / JPG    | **DN**           | Liczby całkowite                                                                         |
| Eksporty 32-bitowego procentowego współczynnika odbicia | **%** tylko       | Zapisana liczba zmiennoprzecinkowa nie jest wartością DN, więc zaokrąglenie jej do liczby całkowitej spowodowałoby wyświetlenie bezsensownego wyniku `0` lub `1` |

Każdy wiersz jest oznaczony nazwą kanału filtra aparatu — `Red / Green / NIR` dla RGN, `Orange / Cyan / NIR` dla OCN, `NIR / Green / Blue` dla NGB, `Red / Green / Blue` dla RGB oraz nazwą pojedynczego pasma dla kamer RE, NIR oraz kamery mono M3M. Każda etykieta posiada kolorową kropkę odpowiadającą okręgom kanałów używanym w edytorze formuł indeksu.

Zapisane obrazy **indeksu i LUT** stanowią przypadek szczególny: zawierają one składowe mapy kolorów zamiast pasm spektralnych, więc ich wiersze są oznaczone jako `Red / Green / Blue` (lub `Index` w przypadku pliku indeksowego jednokanałowego), a nie nazwami filtrów kamery.

Gdy indeks jest aktywny w piaskownicy, pod kanałami pojawia się dodatkowy wiersz pokazujący **wartość indeksu** w miejscu kursora, wraz z nazwą indeksu i białą kropką odpowiadającą jego znacznikowi na histogramie.

### Procent odbicia wykorzystuje własną skalę każdego pliku

{% hint style="warning" %}
**Nie należy zakładać, że 65535 = 100%.** Chloros przechowuje współczynnik odbicia w różnych skalach w zależności od aparatu, który go wygenerował, a przeglądarka dobiera właściwą skalę dla każdego pliku.
{% endhint %}

| Źródło                  | DN odpowiadający współczynnikowi odbicia 1,0 | Sposób identyfikacji                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Tag XMP `Chloros:PixelScale=32768` zapisywany w każdym eksporcie współczynnika odbicia LATTICE. 2-krotny zapas pozwala plikowi zawierać wartości ρ powyżej 1,0 bez obcięcia |
| **Survey3**|**65535**                      | Brak tagu skali XMP Chloros — kalibracja Survey3 zapisuje ρ × dtype-max i ogranicza wartość do 1,0                                                               |

Przeglądarka, środowisko testowe indeksu/LUT oraz eksport indeksu wszystkie ustalają skalę za pomocą tej samej, pojedynczej implementacji, więc wartość odczytana w miejscu kursora jest tą samą wartością, której użyto w obliczeniach indeksu.

Dwie konsekwencje, o których warto wiedzieć:

* **32-bitowa wartość procentowa**TIFF przechowuje DN/65535 jako liczbę zmiennoprzecinkową, a**8-bitowa** PNG/JPG przechowuje wartość DN × 255/65535 — przeglądarka konwertuje obie wartości z powrotem przed wyświetleniem wartości procentowej.
* Jednego przypadku nie da się odtworzyć: **8-bitowy eksport TIFF z 8-bitowego źródła** jest przycinany do zakresu 0–255 zamiast być przeskalowany i celowo nie zawiera znacznika skali. W przypadku tych plików panel wyświetla wyłącznie wartości DN, bez kolumny procentowej. Jest to zgodne z rzeczywistością, a nie błąd.***

## Histogram warstwy

Poniżej wierszy kursora znajduje się aktualizowany na bieżąco histogram warstwy, którą przeglądasz, w **256 przedziałach**. Domyślnie rysuje on jedną połączoną krzywą, ważoną `(R + 2G + B) / 4` — w tej samej przestrzeni pomiarowej, której używają histogramy kamery LATTICE. Włączenie opcji**RGB**, zastępuje ją krzywymi dla poszczególnych kanałów w kolorach tych kanałów, mieszanymi addytywnie, dzięki czemu nakładające się obszary pozostają czytelne. Warstwy monochromatyczne zawsze wyświetlają pojedynczą krzywą.

Oś pozioma jest wyrażona w jednostkach właściwych dla danej warstwy:

| Warstwa       | Jednostka osi  | Maksymalna wartość osi                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Odbicie    | procent    | 125% — zakres dynamiczny produktu pozwala na wartość ρ powyżej 1,0           |
| Promieniowanie | W/m²/sr/nm | Szczytowa wartość klatki, zaokrąglona w górę do dwóch cyfr znaczących |
| Dane 8-bitowe  | DN         | 255                                                        |
| Dane 12-bitowe | DN         | 4095                                                       |
| Dane 16-bitowe | DN         | 65535                                                      |

Gdy oś jest ustawiona w skali DN i osiąga jeden z tych trzech pułapów, Chloros rozpoznaje również głębię bitową wyświetlanego obrazu.

Nad histogramem znajdują się trzy przyciski:

| Przycisk     | Ustawienie domyślne | Efekt                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURSOR** | Włączone | Rysuje linie znacznikowe na histogramie przy dokładnych wartościach pokazanych w wierszach powyżej, dzięki czemu można zobaczyć, gdzie w rozkładzie klatki znajduje się piksel pod kursorem. W trybie RGB dla każdego kanału wyświetlany jest jeden znacznik w osobnym kolorze; w pozostałych przypadkach wyświetlany jest pojedynczy biały znacznik przy wartości łącznej |
| **INDEX**| Włączone      | Pojawia się tylko wtedy, gdy indeks jest aktywny. Przełącza histogram z pasm źródłowych na**rozkład wartości indeksu**, przy czym dwa progi klipowania są zaznaczone pomarańczowymi liniami przerywanymi, a wartość indeksu kursora — białą linią                                                          |
| **RGB**| Wył.     | Przełącza z krzywej łącznej na krzywe dla poszczególnych kanałów. W przypadku czujnika monochromatycznego przycisk ten ma napis**MONO** i jest nieaktywny — do wyświetlenia jest tylko jeden kanał                                                                                                                                  |

Histogram jest obliczany na podstawie **widocznych bloków**, a nie pikseli źródłowych znajdujących się za nimi: zmiana rozmiaru bloku GSD powoduje ponowne obliczenie rozkładu, dzięki czemu histogram, znacznik kursora i wyświetlany obraz są zawsze zgodne.***

## Rozmiar bloku GSD

W dolnej części panelu znajduje się element sterujący **GSD (px)**: pole liczbowe, suwak w zakresie od**1 do 256**oraz przycisk**RESET**.

Powoduje to zgrubienie _wyświetlanego_ obrazu poprzez uśrednienie bloku pikseli źródłowych o wymiarach N × N w jeden piksel wyświetlany. `1` to rozdzielczość natywna.

* Wpływa to na **widok pełnoekranowy, miniatury siatki, odczyt kursora oraz oba histogramy** — wszystko, co wyświetla obraz, opiera się na tej samej rozdzielczości podstawowej.
* Dotyczy to **wyłącznie wyświetlania**. Przetwarzanie i eksport pozostają niezmienione. Jedyny wyjątek jest zamierzony: eksport z [Index/LUT Sandbox](index-lut-sandbox.md) zapisuje to, co aktualnie oglądasz, więc zachowuje bieżący rozmiar bloku, a panel eksportu wyświetla ostrzeżenie, gdy rozmiar bloku przekracza 1.
* Wartość ta jest przechowywana **dla każdego projektu** jako `viewer_display.gsd_bin` w `project.json`, dzięki czemu pozostaje zachowana po zamknięciu i ponownym otwarciu programu.
* Wskaźnik kursora podaje wartość bloku, a nie piksela źródłowego, ilekroć rozmiar bloku przekracza 1 — wyświetlana wartość jest średnią dla bloku znajdującego się pod kursorem.

{% hint style="info" %}
**Dlaczego „rozmiar bloku”, a nie centymetry na piksel?** Wartość cm/px wymaga podania wysokości nad ziemią. Dane EXIF pojedynczej klatki zawierają wysokość GPS nad średnim poziomem morza, a nie nad terenem, na który była skierowana kamera, więc Chloros nie wyświetli odległości od ziemi, której nie może rzetelnie obliczyć. Rozmiar bloku w pikselach źródłowych jest tym samym rozwiązaniem awaryjnym, z którego korzystają narzędzia chmurowe MAPIR, gdy odległość próbkowania naziemnego jest nieznana.
{% endhint %}

***

## Typy obrazów, które można wyświetlać

Lista rozwijana warstw w prawym górnym rogu przeglądarki zawiera wszystkie wersje bieżącego obrazu. To, które pozycje się pojawiają, zależy od aparatu i od tego, co zostało przetworzone — pełną listę oraz informacje o działaniu listy rozwijanej znajdziesz w sekcji [Warstwy obrazu](image-layers.md).

### Survey3

* **JPG** — plik podglądu z samej kamery
* **RAW (oryginał)** — plik źródłowy `.RAW`, poddany debayeringowi w celu wyświetlenia, bez korekcji
* **RAW (cel)** — klatka zidentyfikowana jako zawierająca cel kalibracyjny
* **RAW (Odbicie)** — skalibrowany produkt odbicia (65535 = ρ 1,0)
* **Skorygowane winietowanie**/**Reakcja czujnika** — nieskalibrowany produkt rezerwowy
* **Zrównoważona biel** — produkt z wyrównaniem balansu bieli
* **RAW (indeks `<INDEX>`)**i**LUT `<INDEX>`** — obliczone obrazy indeksowe

### LATTICE

W przypadku ujęć LATTICE stosuje się tę samą listę rozwijaną, zawierającą nazwy poziomów potoku:

| Warstwa                 | Co zawiera                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (oryginalny)**    | Źródłowa klatka w formacie RAW w stanie po przechwyceniu                                     |
| **RAW (po usunięciu efektu bayera)**   | Obraz po liniowym usunięciu efektu bayera                                           |
| **RAW (podgląd)**     | Podgląd wyświetlany na ekranie — rozciągnięcie w fałszywych kolorach dla kamer wielospektralnych |
| **Z balansem bieli**    | Podgląd na ekranie dla kamer głównych RGB (balans bieli + gamma)   |
| **RAW (promieniowanie)**    | Promieniowanie spektralne w formacie Float32 w W/m²/sr/nm                              |
| **RAW (odbicie)** | Odbicie w formacie uint16, 32768 = ρ 1,0                                    |

Promieniowanie i współczynnik odbicia są dostępne wyłącznie w trybie wielospektralnym: kamera główna typu RGB nie posiada radiometrii dla poszczególnych pasm, więc warstwy te nie są dla niej generowane.

***

## Stosowanie indeksów i tabel LUT

Zastosuj indeksy wielospektralne i kolorowe tabele przeglądowe (LUT) z paska bocznego:

1. Otwórz okno **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na pasku bocznym
2. Zaznacz opcję **Index**

3. Wybierz filtr swojej kamery oraz wzór indeksu, a następnie przeciągnij koła kanałów na pola wzoru
4. Dodaj tabelę LUT i wybierz gradient, progi oraz tryb przycinania
5. Odczytaj wartości w miejscu kursora i zapisz wynik za pomocą opcji **Eksportuj/Zapisz obraz(y)**Pełny przewodnik znajdziesz w [Index/LUT Sandbox](index-lut-sandbox.md).***

## Rozwiązywanie problemów

### Obraz się nie otwiera

**Możliwe przyczyny**: plik został przeniesiony lub usunięty po zaimportowaniu; produkt nigdy nie został zapisany; za mało pamięci dla bardzo dużego obrazu.**Co zrobić**:

1. Sprawdź, czy plik warstwy nadal istnieje w drzewie wynikowym projektu
2. Otwórz plik w zewnętrznej przeglądarce, aby sprawdzić, czy jest nienaruszony
3. Zamknij inne aplikacje, aby zwolnić pamięć

### Obraz jest czarny, biały lub ma bardzo dziwne kolory

**Możliwe przyczyny**: funkcja rozciągania obrazu nie ma na czym pracować (klatka o niemal stałej wartości); warstwa typu float32 z nietypowymi wartościami; indeks, który nie wygenerował żadnych prawidłowych danych.**Co należy zrobić**:

1. Odczytaj wartości kursora — jeśli każdy kanał ma wartość równą zero lub bliską zeru, problem leży po stronie danych, a nie wyświetlacza
2. Sprawdź histogram: pojedynczy skok na jednym końcu wskazuje, że klatka jest przycięta lub pusta
3. Sprawdź dziennik przetwarzania dla przebiegu, który wygenerował warstwę

### Wartości wydają się nieprawidłowe

**Możliwe przyczyny**: znajdujesz się na innej warstwie niż sądzisz; porównujesz wartość procentową z surową wartością DN; porównujesz plik LATTICE z plikiem Survey3, używając tego samego dzielnika.**Co należy zrobić**:

1. Sprawdź wybraną warstwę w menu rozwijanym — jednostki w panelu są dostosowane do warstwy
2. W przypadku współczynnika odbicia użyj kolumny **%** zamiast samodzielnego dzielenia wartości DN; jeśli musisz dokonać podziału, użyj wartości `Chloros:PixelScale` z tego pliku (32768 dla LATTICE, brak wartości oznacza 65535 dla Survey3)
3. Ustaw rozmiar bloku GSD z powrotem na 1 — przy wartości powyżej 1 odczytujesz średnią blokową, a nie wartość pikselową
4. Sprawdź, czy kalibracja współczynnika odbicia faktycznie przebiegła dla tej klatki; nieskalibrowany produkt rezerwowy (Sensor Response / Vignette Corrected) nie jest współczynnikiem odbicia

***

## Kolejne kroki

* [**Warstwy obrazu**](image-layers.md) — nazwy wszystkich warstw (o ile istnieją) oraz znaczenie ich wartości
* [**Piaskownica indeksów/tablic LUT**](index-lut-sandbox.md) — tworzenie, dostosowywanie i eksportowanie wizualizacji indeksów
* [**Znaczniki na mapie**](map-markers.md) — ten sam zestaw obrazów na mapie
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — przewodnik po indeksach

Informacje na temat procesu przetwarzania można znaleźć w sekcji [Przetwarzanie obrazów (GUI)](../processing-images-gui/adding-files-to-a-project.md).
