# Warstwy obrazu

**Menu rozwijane warstw** w prawym górnym rogu przeglądarki obrazów pozwala przełączać się między wszystkimi wersjami oglądanego obrazu — od zdjęcia źródłowego, przez każdy przetworzony produkt, aż po obliczone obrazy indeksowe — bez konieczności opuszczania przeglądarki.

## Czym są warstwy obrazów?

„Warstwa” w programie Chloros to jeden **plik produktu**przypisany do jednego obrazu źródłowego. Import zapewnia pliki źródłowe; przetwarzanie dodaje warstwę dla każdego produktu wygenerowanego w ramach danego przebiegu. Eksportowane pliki zachowują nazwę pliku źródłowego — to**folder** identyfikuje produkt, a nazwa warstwy jest etykietą nadaną temu folderowi przez Chloros.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Lista warstw

### Zawsze obecne

| Warstwa | Co to jest |
| --- | --- |
| **JPG**(lub**PNG**/**TIFF**) | Plik bazowy, który został zaimportowany wraz z przechwyconym obrazem. Program Survey3 importuje plik `.JPG` obok każdego pliku `.RAW`; przechwytywanie za pomocą LATTICE dostarcza podgląd wyświetlania w formacie PNG lub TIFF. Oznaczony zgodnie z tym, co faktycznie zostało zaimportowane |
| **RAW (oryginał)** | Źródłowa klatka w formacie RAW, poddana debayeringowi w celu wyświetlenia, bez zastosowania żadnych korekt. Dostępna od momentu importu — nie wymaga przetwarzania |

Zrzut z LATTICE, którego plikiem bazowym **jest** klatka surowa, nie ma oddzielnego wpisu bazowego: obejmuje go już `RAW (Original)`.

### Produkty przetwarzania w programie Survey3



| Warstwa | Zapisywana do | Istnieje, gdy |
| --- | --- | --- |
| **RAW (cel)** | — | Klatka została zidentyfikowana jako zawierająca cel kalibracyjny |
| **RAW (odbicie)** | `Reflectance_Calibrated_Images/` | Kalibracja odbicia przebiegła pomyślnie dla tej klatki |
| **Skorygowana winieta**| `Vignette_Corrected_Images/` | Nie można było przeprowadzić kalibracji odbicia dla tej klatki**i** *korekcja winiety* była włączona |
| **Reakcja czujnika**| `Sensor_Response_Images/` | Nie udało się przeprowadzić kalibracji odbicia światła dla tej klatki**i** *korekcja winietowania* była wyłączona |
| **Balans bieli** | `White_Balanced_Images/` | Utworzono produkt z balansem bieli |

{% hint style="info" %}
**Korekcja winietowania i czułość czujnika to opcje alternatywne, nigdy nie występują jednocześnie.** Dla każdego modelu aparatu istnieje dokładnie jeden nieskalibrowany produkt rezerwowy na każdy przebieg, a przełącznik *Korekcja winietowania* wybiera, który z nich zostanie użyty. Zobacz [Ustawienia projektu](../project-settings/project-settings.md).
{% endhint %}

### Poziomy LATTICE

LATTICE przechwytuje rozgałęzienia do tych poziomów w jednym przebiegu przetwarzania. To, które z nich istnieją, zależy od przełączników eksportu dla poszczególnych produktów w Ustawieniach projektu oraz od tego, co ma zastosowanie do danego aparatu.

| Warstwa | Zapisywana do | Dotyczy |
| --- | --- | --- |
| **RAW (po usunięciu efektu bayera)** | `Debayered_Images/` | RGB i wielospektralne |
| **RAW (podgląd)** | `Preview_Images/` | Multispektralne (rozciągnięcie w fałszywych kolorach) |
| **Zrównoważona biel** | `Preview_Images/` | Kamery główne RGB — podgląd RGB jest zarejestrowany pod tą nazwą, aby był zgodny z warstwą Survey3 o tej samej nazwie |
| **RAW (radiancja)** | `Radiance_Images/` | Tylko wielospektralne |
| **RAW (odbicie)** | `Reflectance_Calibrated_Images/` | Tylko wielospektralne i tylko wtedy, gdy pasujący zapis `.daq` z promieniowaniem w dół lub cel w kadrze, który przeszedł kontrolę jakości, pokrywa kadr |

Kamery główne RGB nie posiadają radiometrii dla poszczególnych pasm, więc promieniowanie i odbicie są w ich przypadku pomijane jako **nie mające zastosowania** — informacja ta pojawia się w dzienniku, a nie jest pomijana bez powiadomienia.

{% hint style="info" %}
**Warstwy surowe i po debayeringu są dopasowane do produktów (nowość w wersji 1.3.0).**Pliki surowe i po debayeringu LATTICE są zapisywane w natywnej orientacji czujnika, a orientacja ta jest zapisywana w pliku XMP; pozostałe produkty są zapisywane z już zastosowaną orientacją. Przeglądarka i siatka obrazu przełączają wyświetlanie między trybami**RAW (oryginalny)**i**RAW (po usunięciu efektu bayera)** zgodnie z tym zapisem, dzięki czemu klatka nie odwraca się już podczas przełączania między warstwą produktu a warstwą surową. Dotyczy to wyłącznie wyświetlania — pliki nigdy nie są przepisywane. Pliki bez zapisu orientacji (zarejestrowane przed wersją 1.3.0 lub zapisane przez inne oprogramowanie) są wyświetlane zgodnie z orientacją, w jakiej zostały zapisane. Zobacz [Formaty obrazów wyjściowych](../output-image-formats.md#orientation).
{% endhint %}

Maski ograniczające (`*_clampmask.tif` w folderze `clamp_masks/`) nie są warstwami: program Chloros pomija je podczas importu. Zobacz [Formaty obrazów wyjściowych](../output-image-formats.md#clamp-masks).

### Warstwy indeksu, LUT i piaskownicy

| Rodzaj warstwy | Przykład | Skąd pochodzi |
| --- | --- | --- |
| **RAW (indeks `<INDEX>`)** | `RAW (NDVI Index)` | Jedna na każdy indeks skonfigurowany w ustawieniach projektu, obliczana podczas przetwarzania |
| **`<INDEX>` LUT** | `NDVI LUT` | Wersja indeksu z mapowaniem kolorów |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jeden na każdy przebieg eksportu [Indeks/LUT Sandbox](index-lut-sandbox.md) |

Jeśli ta sama nazwa indeksu zostanie skonfigurowana więcej niż raz z różnymi ustawieniami, druga i kolejne otrzymują numer w nazwie (`RAW (NDVI2 Index)`), dzięki czemu warstwy pozostają rozróżnialne.

***

## Korzystanie z selektora warstw

1. Otwórz obraz na pełnym ekranie, klikając miniaturę w siatce
2. Kliknij **menu rozwijane warstw** w prawym górnym rogu przeglądarki
3. Wybierz warstwę — obraz zaktualizuje się natychmiast

W menu rozwijającym na pierwszym miejscu znajdują się kolejno opcje **JPG, RAW (oryginał), RAW (docelowy), RAW (odbicie)**, a pozostałe pozycje są wyświetlane w kolejności, w jakiej produkty zostały zarejestrowane.

### Preferencje dotyczące warstw podczas nawigacji

Naciśnięcie klawiszy **←**/**→** powoduje przejście do następnego obrazu i próbę utrzymania tej samej warstwy:

1. **Najpierw dokładne dopasowanie** — jeśli następny obraz ma warstwę o tej samej nazwie, zostanie ona wyświetlona. Dzięki temu podczas przeglądania całego zestawu pozostajesz na warstwie `RAW (NDVI Index)`
2. **Następnie dopasowanie według typu** — warstwa indeksowa szuka dowolnej warstwy indeksowej, LUT — dowolnej warstwy LUT, warstwa odbicia — warstwy odbicia, warstwa docelowa — warstwy docelowej, warstwa oryginalna — warstwy oryginalnej, warstwa bazowa — warstwy bazowej
3. **Następnie, wyłącznie w przypadku warstw eksportowych** — nazwa jest zachowywana, nawet jeśli lista warstw nie została jeszcze zaktualizowana, ponieważ plik już istnieje na dysku. To właśnie pozwala przeglądać produkty, podczas gdy proces ich zapisywania wciąż trwa
4. **W pozostałych przypadkach** — pierwsza dostępna warstwa, którą zazwyczaj jest obraz bazowy

Pliki sidecar `.daq` i `.csv` w projekcie są pomijane podczas nawigacji za pomocą klawiszy strzałek, dzięki czemu przechodzenie między obrazami nigdy nie prowadzi do zapisu z czujnika światła.

Powiększanie i przesuwanie obrazu działa również między obrazami, co ułatwia porównanie stanu „przed” i „po” dla tej samej pozycji pola.

***

## Zrozumienie wartości pikseli według warstw

[Panel wartości kursora](opening-an-image-full-screen.md#cursor-values) podaje rzeczywistą wartość dla każdego kanału pod kursorem, w jednostkach, w których dana warstwa jest zapisana. Jego kolumny zmieniają się w zależności od warstwy:

| Warstwa | Podawana jednostka | Uwagi |
| --- | --- | --- |
| Podstawowa (podgląd JPG / PNG / TIFF) | DN, 0–255 | Wartości wyświetlane, skorygowane pod kątem gamma dla RGB. Wyłącznie do oględzin |
| RAW (oryginał) | DN | Surowe wartości cyfrowe z czujnika. Oś histogramu wskazuje głębię: 255 (8-bitowa), 4095 (12-bitowa) lub 65535 (16-bitowa) |
| RAW (po usunięciu efektu bayera) | DN | Liniowy, bez rozciągania obrazu |
| RAW (podgląd) / z balansem bieli | DN | Wynik wyświetlania — rozciągnięty lub skorygowany pod kątem gamma. Nie służy do pomiarów |
| RAW (promieniowanie) | **W/m²/sr/nm** | Promieniowanie fizyczne typu Float32. Brak kolumny DN |
| RAW (odbicie) | DN **i %** | Procent obliczony według własnej skali pliku — patrz poniżej |
| Eksporty indeksu / LUT / sandbox | Wartość indeksu lub składowe RGB | Jednokanałowy plik indeksu podaje wartość indeksu; plik LUT z mapą kolorów podaje składowe Red /Green /Blue |

### Odbicie: skala jest określona dla danego pliku

{% hint style="warning" %}
**„Podzielenie przez 65 535” jest poprawne tylko w przypadku programu Survey3.** Odbicie LATTICE jest przechowywane w innej skali, a mieszanie tych dwóch dzielników jest najczęstszym sposobem uzyskania wartości odbicia, które są dokładnie o połowę mniejsze od tych, które powinny być.
{% endhint %}

| Źródło | DN odpowiadający współczynnikowi odbicia 1,0 | Identyfikator |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Tag XMP `Chloros:PixelScale=32768` umieszczany w każdym eksporcie współczynnika odbicia LATTICE. Dwukrotny zapas oznacza, że wartości ρ powyżej 1,0 mogą być przedstawione, a nie obcięte |
| **Survey3**| **65535** | Brak tagu skali XMP Chloros — kalibracja Survey3 zapisuje ρ × dtype-max i obcina wartość przy 1,0 |

W przypadku GIS i skryptów: odczytaj z pliku `Chloros:PixelScale` i podziel przez tę wartość. Jeśli tag nie występuje, plik ma skalę Survey3 (65535). Przeglądarka, środowisko testowe indeksu/LUT oraz eksport indeksu ustalają skalę w ten sam sposób, więc liczba odczytywana przy kursorze jest tą samą liczbą, której użyto w obliczeniach indeksu.

Przechowywanie specyficzne dla formatu w oparciu o tę skalę:

* **Format TIFF (32-bitowy, procentowy)** przechowuje wartość DN / 65535 jako liczbę zmiennoprzecinkową
* **Format PNG (8-bitowy)**i**JPG (8-bitowy)** przechowują wartość DN × 255 / 65535
* **8-bitowa wartość DN pochodząca z 8-bitowego zapisu** lub plik odbicia światła zapisany przed wersją 1.3.0 jest przycięty do zakresu 0–255 i nie zawiera znacznika skali. Panel wyświetla wartość DN tylko dla tych plików, bez kolumny procentowej

### Zakresy wartości indeksów

| Rodzaj indeksu | Typowy zakres | Odczyt |
| --- | --- | --- |
| Znormalizowana różnica (NDVI, GNDVI, NDRE, ENDVI…) | od −1 do +1 | Zdrowa roślinność zazwyczaj 0,4–0,9; goła gleba blisko 0; woda – wartości ujemne |
| Skorygowane o glebę (SAVI, OSAVI, MSAVI2…) | w przybliżeniu od −1 do +1,5 | Odczyt podobny do NDVI, z wyeliminowanym tłem glebowym |
| Stosunek (GRVI, GCI, MSR, CIRE…) | nieograniczony powyżej | Stosunki rosną bez ograniczeń, gdy pasmo mianownika dąży do zera |
| EVI / LAI | od 0 do ~1, od 0 do ~3,5 | Chmury i inne nasycone piksele powodują przekroczenie zakresu obu wskaźników — należy je najpierw zamaskować |

Dokładne wzory leżące u podstaw każdego ustawienia wstępnego można znaleźć w sekcji [Wzory wskaźników wielospektralnych](../project-settings/multispectral-index-formulas.md).

***

## Typowe procedury

### Porównanie przed i po

1. Wybierz **RAW (oryginał)** i zwróć uwagę na winietowanie oraz wartości nieskalibrowane
2. Przełącz na **RAW (odbicie)**

3. Porównaj — winietowanie usunięte, wartości skalibrowane. Powiększenie i przesunięcie pozostają niezmienne, więc patrzysz na ten sam obszar

### Przeglądaj jeden indeks w całym zestawie

1. Otwórz pierwsze przetworzone zdjęcie i wybierz warstwę indeksową
2. Naciskaj kilkakrotnie klawisz **→** — warstwa indeksowa podąża za tobą od zdjęcia do zdjęcia
3. Obserwuj histogram na pasku bocznym: klatka, w której rozkład wartości gwałtownie się zmienia, zasługuje na bliższe przyjrzenie się

### Sprawdź cele kalibracyjne

1. Wybierz **RAW (Target)** na klatce docelowej
2. Upewnij się, że cel jest wyraźnie widoczny i został wykryty
3. Przejdź do następnej klatki docelowej — warstwa docelowa podąża za tobą

### Sprawdź dokładność wartości odbicia

1. Wybierz **RAW (Reflectance)**

2. Odczytaj kolumnę**%** w panelu „Wartości kursora” — jest ona już prawidłowo przeskalowana dla tego pliku
3. Sprawdź poprawność na podstawie znanych materiałów w klatce: zdrowa roślinność charakteryzuje się wysokim odbiciem w zakresie NIR i niskim w zakresie czerwonym; wartość odbicia celu kalibracyjnego powinna być zbliżona do opublikowanej wartości

***

## Rozwiązywanie problemów

### Warstwa, której się spodziewałem, nie znajduje się na liście rozwijanej

**Możliwe przyczyny**

* Obraz nie został przetworzony — istnieją tylko warstwa bazowa i `RAW (Original)`
* Opcja eksportu produktu jest odznaczona w ustawieniach projektu
* Produkt nie ma zastosowania do tej kamery (promieniowanie i odbicie dla obrazu głównego w formacie RGB; dowolny wskaźnik dla jednopasmowej kamery monochromatycznej M3M)
* Kalibracja odbicia nie miała na czym działać — brak pokrycia promieniowania padającego `.daq` oraz brak celu w kadrze, który przeszedł kontrolę jakości — więc klatka została przywrócona do trybu „Vignette Corrected” lub „Sensor Response”

**Co należy zrobić**

1. Sprawdź dziennik przebiegu: Chloros informuje, kiedy żądany produkt eksportowy był niemożliwy do uzyskania i dlaczego
2. Sprawdź przełączniki eksportu dla poszczególnych produktów w [Ustawieniach projektu](../project-settings/project-settings.md)
3. Sprawdź, czy folder produktu istnieje w drzewie wyników projektu
4. Przeprowadź ponowne przetwarzanie z włączonym produktem

### Lista warstw wygląda na nieaktualną

Podczas trwania przebiegu program Chloros ponownie skanuje foldery produktów projektu i uzupełnia brakujące rejestracje warstw na podstawie danych faktycznie znajdujących się na dysku, więc warstwa, której eksport zakończył się pomyślnie, pojawia się samodzielnie w ramach aktualizacji. Przełączenie się z obrazu na inną stronę i powrót do niego wymusza wykonanie nowej aktualizacji.

### Wartości odbicia wydają się o połowę niższe od oczekiwanych

Prawie na pewno dzielisz plik LATTICE przez 65535. Użyj `Chloros:PixelScale` (32768) lub sprawdź kolumnę **%**, w której wartość ta została już zastosowana.

### Warstwa indeksowa istnieje, ale obraz jest pusty

Indeks wymaga pasm, których nie posiada dana warstwa — na przykład indeks odczytujący trzeci kanał zastosowany do pliku jedno- lub dwukanałowego. Przejdź na warstwę wielopasmową (odbiciową lub po usunięciu efektu Bayera) lub wybierz indeks pasujący do filtra aparatu.

***

## Kolejne kroki

* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — odczyt pozycji kursora, histogram i kontrola GSD
* [**Środowisko testowe indeksów/LUT**](index-lut-sandbox.md) — interaktywna wizualizacja indeksów i eksport
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — przewodnik po indeksach
* [**Zakończenie przetwarzania**](../processing-images-gui/finishing-the-processing.md) — drzewo folderów wyjściowych, do których prowadzą te warstwy
