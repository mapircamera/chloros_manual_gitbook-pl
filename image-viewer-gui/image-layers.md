# Warstwy obrazu

**Menu rozwijane warstw** w prawym górnym rogu przeglądarki obrazów pozwala przełączać się między wszystkimi wersjami oglądanego obrazu — od ujęcia źródłowego, przez poszczególne przetworzone wersje, aż po obliczone obrazy indeksowe — bez konieczności opuszczania przeglądarki.

## Czym są warstwy obrazów?

„Warstwa” w programie Chloros to jeden **plik produktu**przypisany do jednego obrazu źródłowego. Importowanie zapewnia pliki źródłowe; przetwarzanie dodaje warstwę dla każdego produktu wygenerowanego w trakcie przebiegu. Eksportowane pliki zachowują nazwę pliku źródłowego — to**folder** identyfikuje produkt, a nazwa warstwy to etykieta nadana temu folderowi przez Chloros.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Lista warstw

### Zawsze obecne

| Warstwa | Co to jest |
| --- | --- |
| **JPG**(lub**PNG**/**TIFF**) | Plik bazowy, który został przesłany wraz z przechwyceniem. Survey3 importuje plik `.JPG` obok każdego pliku `.RAW`; przechwytywania LATTICE dostarczają plik PNG lub TIFF. Oznaczony zgodnie z tym, co faktycznie zostało zaimportowane |
| **RAW (oryginał)** | Źródłowa klatka w formacie RAW, poddana debayeringowi w celu wyświetlenia, bez zastosowanych korekcji. Dostępna od momentu importu — nie wymaga przetwarzania |

Zrzut z LATTICE, którego plikiem bazowym **jest** klatka surowa, nie ma oddzielnego wpisu bazowego: obejmuje go już `RAW (Original)`.

### Produkty przetwarzania Survey3

| Warstwa | Zapisana do | Istnieje, gdy |
| --- | --- | --- |
| **RAW (Cel)** | — | Klatka została zidentyfikowana jako zawierająca cel kalibracyjny |
| **RAW (odbicie)** | `Reflectance_Calibrated_Images/` | Kalibracja odbicia przebiegła pomyślnie dla tej klatki |
| **Skorygowana winieta**| `Vignette_Corrected_Images/` | Nie można było przeprowadzić kalibracji odbicia dla tej klatki,**a** *korekcja winiety* była włączona |
| **Reakcja czujnika**| `Sensor_Response_Images/` | Nie udało się przeprowadzić kalibracji odbicia światła dla tej klatki**i** *korekcja winietowania* była wyłączona |
| **Balans bieli** | `White_Balanced_Images/` | Utworzono produkt z balansem bieli |

{% hint style="info" %}
**Korekcja winietowania i reakcja czujnika to opcje alternatywne, nigdy nie występują jednocześnie.** W każdym przebiegu istnieje dokładnie jeden nieskalibrowany produkt rezerwowy dla każdego modelu aparatu, a przełącznik *Korekcja winietowania* wybiera, który z nich zostanie użyty. Zobacz [Ustawienia projektu](../project-settings/project-settings.md).
{% endhint %}

### Poziomy LATTICE

LATTICE przechwytuje rozgałęzienia do tych poziomów w jednym cyklu przetwarzania. To, które z nich istnieją, zależy od przełączników eksportu dla poszczególnych produktów w Ustawieniach projektu oraz od tego, co ma zastosowanie do danego aparatu.

| Warstwa | Zapisywana do | Dotyczy |
| --- | --- | --- |
| **RAW (po usunięciu efektu bayera)** | `Debayered_Images/` | RGB oraz dane wielospektralne |
| **RAW (podgląd)** | `Preview_Images/` | Multispektralne (rozciągnięcie w fałszywych kolorach) |
| **Z balansem bieli** | `Preview_Images/` | Kamery główne RGB — podgląd RGB jest zarejestrowany pod tą nazwą, aby pokrywał się z warstwą Survey3 o tej samej nazwie |
| **RAW (promieniowanie)** | `Radiance_Images/` | Tylko wielospektralne |
| **RAW (odbicie)** | `Reflectance_Calibrated_Images/` | Tylko wielospektralne i tylko wtedy, gdy pasujący rekord promieniowania w dół `.daq` lub cel w kadrze, który przeszedł kontrolę jakości, pokrywa kadr |

Kamery główne RGB nie posiadają radiometrii dla poszczególnych pasm, więc promieniowanie i odbicie są w ich przypadku pomijane jako **nie mające zastosowania** — informacja ta pojawia się w dzienniku, a nie jest pomijana bez powiadomienia.

### Warstwy indeksu, LUT i piaskownicy

| Schemat warstwy | Przykład | Skąd pochodzi |
| --- | --- | --- |
| **RAW (`<INDEX>` Indeks)** | `RAW (NDVI Index)` | Jedna na każdy indeks skonfigurowany w ustawieniach projektu, obliczana podczas przetwarzania |
| **`<INDEX>` LUT** | `NDVI LUT` | Wersja indeksu z mapą kolorów |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Po jednym na każdy przebieg eksportu [Indeksu/LUT Sandbox](index-lut-sandbox.md) |

Jeśli ta sama nazwa indeksu zostanie skonfigurowana więcej niż raz z różnymi ustawieniami, druga i kolejne otrzymują numer w nazwie (`RAW (NDVI2 Index)`), dzięki czemu warstwy pozostają rozróżnialne.

***

## Korzystanie z selektora warstw

1. Otwórz obraz na pełnym ekranie, klikając miniaturę w siatce
2. Kliknij **menu rozwijane warstw** w prawym górnym rogu przeglądarki
3. Wybierz warstwę — obraz zaktualizuje się natychmiast

W menu rozwijanym na początku znajdują się opcje **JPG, RAW (oryginał), RAW (docelowy), RAW (odbicie)** w tej kolejności, a pozostałe pozycje są wyświetlane zgodnie z kolejnością rejestracji produktów.

### Preferencje warstw podczas nawigacji

Naciśnięcie klawiszy **←**/**→** powoduje przejście do następnego obrazu i próbę utrzymania tej samej warstwy:

1. **Najpierw dokładne dopasowanie** — jeśli następny obraz ma warstwę o tej samej nazwie, zostanie ona wyświetlona. Dzięki temu podczas przeglądania całego zestawu pozostajesz na warstwie `RAW (NDVI Index)`
2. **Następnie dopasowanie według typu** — warstwa indeksowa szuka dowolnej warstwy indeksowej, LUT — dowolnej warstwy LUT, warstwa odbicia — warstwy odbicia, warstwa docelowa — warstwy docelowej, warstwa oryginalna — warstwy oryginalnej, warstwa bazowa — warstwy bazowej
3. **Następnie, wyłącznie w przypadku warstw eksportowych** — nazwa jest zachowywana, nawet jeśli lista warstw nie została jeszcze zaktualizowana, ponieważ plik już istnieje na dysku. To właśnie pozwala przeglądać produkty, podczas gdy proces ich zapisywania wciąż trwa
4. **W pozostałych przypadkach** — pierwsza dostępna warstwa, którą zazwyczaj jest obraz bazowy

Pliki sidecar `.daq` i `.csv` w projekcie są pomijane podczas nawigacji za pomocą klawiszy strzałek, dzięki czemu przechodzenie między obrazami nigdy nie prowadzi do zapisu z czujnika światła.

Powiększanie i przesuwanie obrazu działa również między obrazami, co ułatwia porównanie stanu „przed” i „po” dla tej samej pozycji pola.

***

## Zrozumienie wartości pikseli według warstw

[Panel wartości kursora](opening-an-image-full-screen.md#cursor-values) podaje rzeczywistą wartość dla poszczególnych kanałów pod kursorem, w jednostkach, w których dana warstwa jest zapisana. Kolumny panelu zmieniają się w zależności od warstwy:

| Warstwa | Podawana jednostka | Uwagi |
| --- | --- | --- |
| Podstawowa (JPG / podgląd PNG / TIFF) | DN, 0–255 | Wartości wyświetlane, skorygowane pod kątem gamma w RGB. Tylko do oceny wizualnej |
| RAW (oryginał) | DN | Surowe wartości cyfrowe z czujnika. Oś histogramu wskazuje głębię: 255 (8-bitowa), 4095 (12-bitowa) lub 65535 (16-bitowa) |
| RAW (po usunięciu efektu bayera) | DN | Liniowy, bez rozciągania obrazu |
| RAW (podgląd) / z balansem bieli | DN | Obraz wyświetlany — rozciągnięty lub z korekcją gamma. Nie służy do pomiarów |
| RAW (promieniowanie) | **W/m²/sr/nm** | Promieniowanie fizyczne typu Float32. Brak kolumny DN |
| RAW (odbicie) | DN **i %** | Procent obliczony według własnej skali pliku — patrz poniżej |
| Eksport indeksu / LUT / sandbox | Wartość indeksu lub składowe RGB | Jednokanałowy plik indeksu podaje wartość indeksu; plik LUT z mapą kolorów podaje składowe Red/Green/Blue |

### Odbicie: skala jest określana indywidualnie dla każdego pliku

{% hint style="warning" %}
**„Podzielenie przez 65 535” jest poprawne tylko dla Survey3.** Odbicie LATTICE jest przechowywane w innej skali, a pomylenie tych dwóch dzielników jest najczęstszym sposobem uzyskania wartości odbicia, które są dokładnie o połowę mniejsze od tych, które powinny być.
{% endhint %}

| Źródło | DN odpowiadający współczynnikowi odbicia 1,0 | Identyfikator |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Tag XMP `Chloros:PixelScale=32768` umieszczany w każdym eksporcie współczynnika odbicia LATTICE. Dwukrotny zapas oznacza, że wartości ρ powyżej 1,0 mogą być reprezentowane, a nie obcinane |
| **Survey3**|**65535** | Brak tagu skali XMP Chloros — kalibracja Survey3 zapisuje ρ × dtype-max i obcina wartość przy 1,0 |

W przypadku GIS i skryptów: odczytaj `Chloros:PixelScale` z pliku i podziel przez tę wartość. Jeśli tag nie występuje, plik jest w skali Survey3 (65535). Przeglądarka, środowisko testowe indeksu/LUT oraz eksport indeksu ustalają skalę w ten sam sposób, więc liczba odczytana w miejscu kursora jest liczbą wykorzystaną w obliczeniach indeksu.

Dodatkowe zapisywanie specyficzne dla formatu w oparciu o tę skalę:

* **TIFF (32-bitowe, procent)** przechowuje wartość DN / 65535 jako liczbę zmiennoprzecinkową
* **PNG (8-bitowe)**oraz**JPG (8-bitowe)** przechowują wartość DN × 255 / 65535
* **Eksport 8-bitowy TIFF z przechwycenia źródła 8-bitowego** jest przycinany do zakresu 0–255 zamiast przeskalowywany i celowo nie zawiera znacznika skali. Panel wyświetla wartość DN tylko dla tych plików, bez kolumny procentowej

### Zakresy wartości indeksowych

| Rodzaj indeksu | Typowy zakres | Odczyt |
| --- | --- | --- |
| Znormalizowana różnica (NDVI, GNDVI, NDRE, ENDVI…) | od −1 do +1 | Zdrowa roślinność zazwyczaj 0,4–0,9; goła gleba blisko 0; woda – wartości ujemne |
| Skorygowana o glebę (SAVI, OSAVI, MSAVI2…) | w przybliżeniu od −1 do +1,5 | Odczyt podobny do NDVI z wyeliminowanym tłem glebowym |
| Stosunek (GRVI, GCI, MSR, CIRE…) | nieograniczony w górę | Stosunki rosną bez ograniczeń, gdy pasmo mianownika dąży do zera |
| EVI / LAI | od 0 do ~1, od 0 do ~3,5 | Chmury i inne nasycone piksele powodują, że obie wartości wykraczają poza zakres — należy je najpierw zamaskować |

Dokładne wzory dla każdego z gotowych ustawień znajdziesz w sekcji [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md).

***

## Typowe procedury

### Porównanie przed i po

1. Wybierz **RAW (oryginał)** i zwróć uwagę na winietowanie oraz nieskalibrowane wartości
2. Przełącz na **RAW (odbicie)**

3. Porównaj — winietowanie usunięte, wartości skalibrowane. Powiększenie i przesuwanie pozostają niezmienne, więc patrzysz na ten sam obszar terenu

### Przeglądaj jeden wskaźnik w całym zestawie

1. Otwórz pierwsze przetworzone zdjęcie i wybierz warstwę wskaźnika
2. Naciskaj wielokrotnie klawisz **→** — warstwa wskaźnika podąża za tobą od zdjęcia do zdjęcia
3. Obserwuj histogram na pasku bocznym: klatka, w której rozkład wartości gwałtownie się zmienia, zasługuje na bliższe przyjrzenie się

### Sprawdź cele kalibracyjne

1. Wybierz **RAW (Target)** na klatce docelowej
2. Upewnij się, że cel jest wyraźnie widoczny i został wykryty
3. Przejdź do następnej klatki docelowej — warstwa docelowa podąża za tobą

### Sprawdź dokładność wartości odbicia

1. Wybierz **RAW (Reflectance)**

2. Odczytaj kolumnę**%** w panelu „Wartości kursora” — jest ona już prawidłowo przeskalowana dla tego pliku
3. Sprawdź poprawność na podstawie znanych materiałów w klatce: zdrowa roślinność ma wysoką wartość NIR i niską wartość czerwieni; cel kalibracyjny powinien wykazywać wartość zbliżoną do opublikowanej wartości odbicia

***

## Rozwiązywanie problemów

### Warstwa, której się spodziewałem, nie znajduje się na liście rozwijanej

**Możliwe przyczyny**

* Obraz nie został nigdy przetworzony — istnieją tylko warstwa bazowa i `RAW (Original)`
* W ustawieniach projektu nie zaznaczono opcji eksportu tego produktu
* Produkt nie ma zastosowania do tej kamery (promieniowanie i odbicie w przypadku kamery głównej RGB; dowolny wskaźnik w przypadku jednopasmowej kamery monochromatycznej M3M)
* Kalibracja odbicia nie miała na czym się oprzeć — brak zasięgu promieniowania opadającego `.daq` oraz braku celu w kadrze, który przeszedł kontrolę jakości — więc dla tej klatki zastosowano domyślnie korekcję winietowania lub charakterystykę czujnika

**Co należy zrobić**

1. Sprawdź dziennik przebiegu: Chloros podaje, kiedy żądany produkt eksportowy był niemożliwy do uzyskania oraz dlaczego
2. Sprawdź przełączniki eksportu dla poszczególnych produktów w [Ustawieniach projektu](../project-settings/project-settings.md)
3. Sprawdź, czy folder produktu istnieje w drzewie wyników projektu
4. Przeprowadź ponowne przetwarzanie z włączonym produktem

### Lista warstw wygląda na nieaktualną

Chloros ponownie skanuje foldery produktów projektu w trakcie trwania operacji i uzupełnia brakujące rejestracje warstw na podstawie danych faktycznie znajdujących się na dysku, więc warstwa, której eksport zakończył się pomyślnie, pojawia się samodzielnie w wyniku ponownego skanowania. Przełączenie się na inny obraz i powrót do niego wymusza nowe skanowanie.

### Wartości odbicia wydają się o połowę niższe od oczekiwanych

Prawie na pewno dzielisz plik LATTICE przez 65535. Użyj `Chloros:PixelScale` (32768) lub odczytaj kolumnę **%**, w której wartość ta została już zastosowana.

### Warstwa indeksowa istnieje, ale obraz jest pusty

Indeks wymaga pasm, których nie posiada dana warstwa — na przykład indeks odczytujący trzeci kanał zastosowany do pliku jedno- lub dwukanałowego. Przejdź na warstwę wielopasmową (odbicie lub bez bayera) lub wybierz indeks pasujący do filtra kamery.

***

## Kolejne kroki

* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — odczyt kursora, histogram i kontrola GSD
* [**Piaskownica indeksów/LUT**](index-lut-sandbox.md) — interaktywna wizualizacja indeksów i eksport
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — przewodnik po indeksach
* [**Zakończenie przetwarzania**](../processing-images-gui/finishing-the-processing.md) — drzewo folderów wyjściowych, do których odwołują się te warstwy
