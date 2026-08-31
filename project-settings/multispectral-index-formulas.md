---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Wzory na wskaźniki wielospektralne

Poniższe wzory na wskaźniki wykorzystują kombinację zakresów średniej przepuszczalności filtra Survey3:

<table><thead><tr><th align="center">Kolor filtra Survey3</th><th width="196.199951171875" align="center">Nazwa filtra Survey3</th><th width="159.800048828125" align="center">Zakres przepuszczalności (FWHM)</th><th align="center">Średnia przepuszczalność</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB – Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Ponownie – RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>W przypadku stosowania tych wzorów nazwa może kończyć się na „\_1” lub „\_2”, co odpowiada temu, który filtr NIR, NIR1 czy NIR2 został użyty.

W przypadku kamer LATTICE M3C (potrójny filtr pasmowy typu Bayera) ten sam silnik indeksujący wykorzystuje pasma filtrów M3C:

| Filtr M3C | Pasmo 1 (środek/FWHM) | Pasmo 2 (środek/FWHM) | Pasmo 3 (środek/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamery LATTICE M3M są jednopasmowe (jeden filtr wąskopasmowy na kamerę), więc wskaźniki wielopasmowe nie są obliczane dla pojedynczego obrazu M3M. Aby obliczyć wskaźniki przy użyciu M3M, należy połączyć dwie lub więcej kamer w wyrównany stos wielopasmowy i skorzystać z silnika wskaźników LATTICE (`chloros-cli lattice index` lub kalkulatora wskaźników na żywo w interfejsie graficznym).

***

## Gdzie działa każda nazwa wskaźnika

Chloros posiada **trzy** powierzchnie wskaźników, a ich listy ustawień wstępnych nie są identyczne. Skorzystaj z tej sekcji, aby sprawdzić, czy dana nazwa będzie działać tam, gdzie planujesz ją zastosować.

| Gdzie się znajdujesz | Która lista ma zastosowanie | Liczba |
| --- | --- | --- |
| Ustawienia projektu → Indeks → Dodaj indeks (interfejs graficzny) | Powierzchnia 1 | 27 |
| Przeglądarka obrazów [Piaskownica indeksów/LUT](../image-viewer-gui/index-lut-sandbox.md) (interfejs graficzny) | Powierzchnia 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Powierzchnia 2 | 22 |
| SDK `process_folder(indices=[...])` | Powierzchnia 2 | 22 |
| `chloros-cli lattice index --preset` | Powierzchnia 3 | 22 (inna wartość 22) |
| Karta „Kamery” – kalkulator indeksu na żywo | Powierzchnia 3 | 22 (inna wartość 22) |

Surface 1 i 2 działają na **jednym obrazie naraz z jednej kamery**, wykorzystując sloty symboli `x`/`y`/`z`(/`a`), powiązanych z kanałami filtrującymi tej kamery. Surface 3 przetwarza**wyrównany stos wielopasmowy** — kilka kamer LATTICE współrejestrowanych w jedną kostkę — i odwołuje się do kanałów za pomocą nazw pisanych małymi literami.

### 1. Ustawienia projektu GUI / menu rozwijane „Sandbox” w przeglądarce obrazów — 27 formuł

Menu rozwijane wyświetla je w następującej kolejności (jest to kolejność wstawiania, a nie alfabetyczna):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

W interfejsie graficznym przeciągasz kanały filtrów swojej kamery na pola pasm formuł, dzięki czemu każdą formułę można zastosować z dowolnym przypisaniem pasm obsługiwanym przez kamerę. Zapisane przez Ciebie niestandardowe formuły są dodawane poniżej tej listy.

**Pięć formuł dostępnych wyłącznie w GUI** — tych, których lista CLI/SDK `--indices` nie akceptuje — zostało zaimplementowanych jako:

| Ustawienie dostępne wyłącznie w interfejsie graficznym | Formuła (w postaci zaimplementowanej) | Pola |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y | |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Przewidziane przyporządkowanie dla każdego z nich podano w osobnej sekcji w dalszej części tej strony (na przykład GARI oczekuje x=Green, y=NIR, z=Blue, a=Red). GARI jest jedyną formułą w Chloros, która wykorzystuje czwarte miejsce.

### 2. Rozszerzenie nazwy CLI / SDK `--indices` — 22 ustawienia wstępne

Opcja `chloros-cli process --indices` (oraz parametr SDK `indices`) akceptuje następujące nazwy ustawień wstępnych:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Nieznane nazwy indeksów są pomijane bez komunikatu.** Nazwa spoza tej listy (w tym pięć formuł dostępnych wyłącznie w interfejsie graficznym: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` oraz wszelkich formuł niestandardowych zapisanych w interfejsie graficznym) zostanie pominięta z jedynie informacją w dzienniku — uruchomienie będzie kontynuowane bez tego indeksu, a samo uruchomienie nadal zostanie zgłoszone jako zakończone sukcesem. Informacja ta zostanie wyświetlona w następującej postaci:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nazwy są porównywane bez uwzględniania wielkości liter po usunięciu spacji, więc `ndvi`, `NDVI` i ` NDVI ` to ten sam preset. Ustawienie wstępne jest również pomijane, jeśli wymaga pasma, którego filtr aparatu nie zapewnia.
{% endhint %}

Dokładne wzory w postaci zaimplementowanej (symbole `x`/`y`/`z` oznaczają sloty pasmowe; domyślne przyporządkowanie podano dla każdego ustawienia wstępnego):

| Ustawienie wstępne | Wzór (zgodnie z implementacją) | Filtr domyślny | Miejsca na pasma (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Jak nazwa presetu przekształca się w pozycje pasm

Gdy podajesz samą nazwę, np. `NDVI`, program Chloros musi ustalić, z którego kanału danego pliku odczytuje się każdy symbol. Wykorzystuje do tego tabelę, która przyporządkowuje kod filtru do pozycji w tablicy każdego kanału:

| Kod filtra | Kanał → indeks tablicy |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` jest akceptowany jako alias dla Orange, również 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Domyślny filtr** presetu (kolumna „Domyślny filtr” powyżej) jest używany, gdy projekt zawiera obrazy z tym filtrem. Jeśli nie zawiera, Chloros przeszukuje filtry faktycznie obecne w projekcie w kolejności `RGN, OCN, NGB, RGB, RE, NIR` i wybiera pierwszy, który może zapewnić wszystkie kanały wymagane przez preset. Jeśli żaden nie jest w stanie tego zrobić, preset jest pomijany w danym przebiegu. Dlatego zestaw danych `NDVI`, o który poproszono w przypadku zestawu danych OCN— wiąże się z pozycjami Orange i NIR w OCN.

Ciągi znaków modelu LATTICE M3C zawierają filtr z prefiksem `F` (`LATT-M3C-L41-FRGN`), ale prefiks ten jest pomijany podczas odczytu kodu filtra z obrazu, więc kamera FRGN rozpoznaje go poprzez wiersz `RGN` powyżej i nie wymaga specjalnego traktowania.

### 3. Silnik indeksujący LATTICE (`lattice index --preset`, kalkulator indeksu na żywo) — 22 ustawienia wstępne

Silnik LATTICE działa na wyrównanych stosach wielopasmowych (macierze na żywo lub wyeksportowane pliki TIFF wielopasmowe) i używa nazw kanałów pisanych małymi literami (`red`, `green`, `blue`, `red_edge`, `nir`). Jego lista ustawień wstępnych różni się od dwóch powyższych:

| Ustawienie wstępne | Wzór | Kanały |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | czerwony, nir |
| GNDVI | `(nir - green) / (nir + green)` | zielony, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | niebieski, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | czerwony\_krawędź, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | niebieski, zielony, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | czerwony, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | czerwony, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | czerwony, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | niebieski, czerwony, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | czerwony, NIR |
| CVI | `(nir / green) - (red / green)` | czerwony, zielony, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | czerwony, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | czerwony, nir |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | czerwony, zielony, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | czerwony, zielony, niebieski |
| NGRDI | `(green - red) / (green + red)` | czerwony, zielony |
| VARI | `(green - red) / (green + red - blue)` | czerwony, zielony, niebieski |
| TGI | `green - 0.39*red - 0.61*blue` | czerwony, zielony, niebieski |
| EXG | `2*green - red - blue` | czerwony, zielony, niebieski |
| CIRE | `(nir / red_edge) - 1` | czerwony\_krawędź, NIR |
| CIGREEN | `(nir / green) - 1` | zielony, NIR |
| NDWI | `(green - nir) / (green + nir)` | zielony, nir |

Uruchom `chloros-cli lattice index --list-presets`, aby wydrukować tę tabelę z zainstalowanej kompilacji, a polecenie `--list-gradients`, aby wyświetlić dostępne gradienty kolorów. W symbolach kanałów rozróżniana jest wielkość liter i muszą one odpowiadać nazwom presetów pisanym małymi literami (np. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Zgodnie z implementacją w interfejsie graficznym oraz liście ustawień wstępnych CLI/SDK, CVI jest wzorem opartym na stosunku współczynników:

$$
CVI = {(z / y) \over (x / y)}
$$

z domyślnym mapowaniem kanałów RGB: x=Red, y = Green, z = Blue. W interfejsie graficznym można przeciągnąć dowolny kanał z kamery do slotów x/y/z. Należy pamiętać, że ustawienie domyślne silnika indeksów LATTICE o nazwie `CVI` wykorzystuje inny wzór, `(NIR / Green) - (Red / Green)` — należy sprawdzić powyższe tabele dla danej powierzchni.

***

## ENDVI – ulepszony znormalizowany wskaźnik różnicy wegetacji

Wskaźnik ten wykorzystuje kanał niebieski oprócz NIR i zielonego i jest popularny w przypadku kamer z filtrem NGB, w których pasmo niebieskie zastępuje czerwone.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementacja polega na zastosowaniu wzoru symbolicznego `((x+y)-(2*z))/((x+y)+(2*z))` — przypisz kanały NIR i Green swojej kamery do pól x/y, a Blue do pola z (dla kamery NGB: x=NIR, y=Green, z=Blue).

***

## EVI – ulepszony wskaźnik wegetacji

Wskaźnik ten został pierwotnie opracowany do wykorzystania z danymi MODIS jako ulepszenie w stosunku do NDVI poprzez optymalizację sygnału roślinności na obszarach o wysokim wskaźniku powierzchni liści (LAI). Jest on najbardziej przydatny w regionach o wysokich wartościach LAI, gdzie wskaźnik NDVI może ulegać nasyceniu. Wykorzystuje on zakres odbicia światła niebieskiego do korekcji sygnałów tła pochodzących z gleby oraz do ograniczenia wpływów atmosferycznych, w tym rozpraszanie aerozoli.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Wartości EVI powinny mieścić się w przedziale od 0 do 1 dla pikseli roślinności. Jasne elementy, takie jak chmury i białe budynki, a także ciemne elementy, takie jak woda, mogą powodować anomalne wartości pikseli na obrazie EVI. Przed utworzeniem obrazu EVI należy zamaskować chmury i jasne obiekty na obrazie odbicia oraz, opcjonalnie, ustawić próg wartości pikseli w zakresie od 0 do 1.

_Źródło: Huete, A. i in. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices”. Remote Sensing of Environment 83 (2002): 195–213._

***

## FCI1 – Wskaźnik pokrycia leśnego 1

_Tylko w interfejsie graficznym — niedostępny jako ustawienie wstępne CLI/SDK `--indices`._

Wskaźnik ten odróżnia korony drzew od innych rodzajów roślinności przy użyciu wielospektralnych obrazów odbicia obejmujące pasmo „red edge”.

$$
FCI1 = Red * RedEdge
$$

Obszary zalesione będą miały niższe wartości FCI1 ze względu na niższy współczynnik odbicia drzew oraz obecność cieni w koronie drzew.

_Źródło: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. „Solidne wskaźniki pokrycia leśnego dla obrazów wielospektralnych”. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Wskaźnik pokrycia leśnego 2

_Tylko w interfejsie graficznym — niedostępny jako ustawienie wstępne CLI/SDK `--indices`._

Wskaźnik ten pozwala odróżnić korony drzew od innych rodzajów roślinności przy użyciu wielospektralnych obrazów odbicia, które nie zawierają pasma „red edge”.

$$
FCI2 = Red * NIR
$$

Obszary leśne będą miały niższe wartości FCI2 ze względu na niższy współczynnik odbicia drzew oraz obecność cieni w koronie drzew.

_Źródło: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. „Solidne wskaźniki pokrycia lasów dla obrazów wielospektralnych”. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI - Globalny wskaźnik monitorowania środowiska

_Tylko w GUI — niedostępny jako ustawienie wstępne CLI/SDK `--indices`._

Ten nieliniowy wskaźnik roślinności służy do globalnego monitorowania środowiska na podstawie zdjęć satelitarnych i ma na celu skorygowanie wpływu czynników atmosferycznych. Jest podobny do NDVI, ale jest mniej wrażliwy na czynniki atmosferyczne. Wpływa na niego odsłonięta gleba; dlatego nie zaleca się jego stosowania na obszarach o rzadkiej lub umiarkowanie gęstej roślinności.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Gdzie:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Źródło: Pinty, B. i M. Verstraete. GEMI: nieliniowy wskaźnik do monitorowania globalnej roślinności z satelitów. Vegetation 101 (1992): 15–20._

***

## GARI – Green – wskaźnik odporny na wpływ czynników atmosferycznych

_Tylko w GUI — niedostępny jako ustawienie wstępne CLI/SDK `--indices`._

Wskaźnik ten jest bardziej czuły na szeroki zakres stężeń chlorofilu i mniej wrażliwy na czynniki atmosferyczne niż NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Stała gamma jest funkcją ważenia zależną od warunków aerozolowych w atmosferze. Program ENVI stosuje wartość 1,7, która jest wartością zalecaną przez Gitelsona, Kaufmana i Merzylaka (1996, strona 296).

_Źródło: Gitelson, A., Y. Kaufman i M. Merzylak. „Wykorzystanie kanału Green w teledetekcji globalnej roślinności z EOS-MODIS”. Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Wskaźnik chlorofilu

Wskaźnik ten służy do szacowania zawartości chlorofilu w liściach szerokiej gamy gatunków roślin.

$$
GCI = {NIR \over Green} - 1
$$

Wykorzystanie szerokiego zakresu długości fal NIR oraz zielonych pozwala na lepsze przewidywanie zawartości chlorofilu, zapewniając jednocześnie większą czułość i wyższy stosunek sygnału do szumu.

_Źródło: Gitelson, A., Y. Gritz i M. Merzlyak. „Związki między zawartością chlorofilu w liściach a odbiciem spektralnym oraz algorytmy nieniszczącej oceny chlorofilu w liściach roślin wyższych”. Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green – wskaźnik liści

Wskaźnik ten został pierwotnie zaprojektowany do stosowania z cyfrową kamerą RGB w celu pomiaru pokrywy pszenicy, gdzie cyfrowe wartości (DN) dla kolorów czerwonego, zielonego i niebieskiego mieszczą się w zakresie od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Wartości GLI mieszczą się w przedziale od -1 do +1. Wartości ujemne odpowiadają glebie i elementom nieożywionym, natomiast wartości dodatnie odpowiadają zielonym liściom i łodygom.

_Źródło: Louhaichi, M., M. Borman i D. Johnson. „Platforma z lokalizacją przestrzenną i fotografia lotnicza do dokumentacji wpływu wypasu na uprawy pszenicy”. Geocarto International 16, nr 1 (2001): 65–70._

***

## GNDVI – Green – znormalizowany wskaźnik różnicy wegetacji

Wskaźnik ten jest podobny do NDVI, z tą różnicą, że mierzy on spektrum zielone w zakresie od 540 do 570 nm zamiast spektrum czerwonego. Wskaźnik ten jest bardziej czuły na stężenie chlorofilu niż NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Źródło: Gitelson, A. i M. Merzlyak. „Teledetekcja stężenia chlorofilu w liściach roślin wyższych”. Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Zoptymalizowany wskaźnik wegetacji skorygowany o właściwości gleby

Wskaźnik ten został pierwotnie opracowany z wykorzystaniem fotografii kolorowo-podczerwonej w celu prognozowania zapotrzebowania kukurydzy na azot. Jest on podobny do wskaźnika OSAVI, ale zastępuje pasmo zielone pasmem czerwonym.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Źródło: Sripada, R. i in. „Określanie zapotrzebowania kukurydzy na azot w trakcie sezonu wegetacyjnego przy użyciu lotnych zdjęć w kolorze i podczerwieni”. Rozprawa doktorska, Uniwersytet Stanowy Karoliny Północnej, 2005._

***

## Wskaźnik wegetacyjny GRVI – Green

Wskaźnik ten jest wrażliwy na tempo fotosyntezy w koronie drzew, ponieważ na odbicie światła w zakresie zielonym i czerwonym silny wpływ mają zmiany w pigmentacji liści.

$$
GRVI = {NIR \over Green }
$$

_Źródło: Sripada, R. i in. „Fotografia lotnicza w kolorze i podczerwieni do określania wczesnego zapotrzebowania na azot w uprawie kukurydzy”. Agronomy Journal 98 (2006): 968–977._

***

## GSAVI - Green Wskaźnik wegetacji skorygowany o właściwości gleby

Wskaźnik ten został pierwotnie opracowany z wykorzystaniem kolorowej fotografii w podczerwieni w celu prognozowania zapotrzebowania kukurydzy na azot. Jest podobny do wskaźnika SAVI, ale zastępuje pasmo zielone pasmem czerwonym.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Źródło: Sripada, R. i in. „Określanie zapotrzebowania kukurydzy na azot w trakcie sezonu wegetacyjnego przy użyciu lotnych zdjęć w kolorze i podczerwieni”. Rozprawa doktorska, Uniwersytet Stanowy Karoliny Północnej, 2005._

***

## LAI – wskaźnik powierzchni liści

Wskaźnik ten służy do oszacowania pokrycia liściowego oraz prognozowania wzrostu i plonu upraw. Program ENVI oblicza zielony LAI przy użyciu następującego wzoru empirycznego autorstwa Boegha i in. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Gdzie EVI oznacza:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Wysokie wartości LAI zazwyczaj mieszczą się w przedziale od około 0 do 3,5. Jednak gdy na obrazie występują chmury i inne jasne elementy powodujące nasycenie pikseli, wartości LAI mogą przekraczać 3,5. Najlepiej byłoby zamaskować chmury i jasne elementy sceny przed utworzeniem obrazu LAI.

_Źródło: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde i A. Thomsen. „Dane wielospektralne z lotu w celu ilościowego określenia wskaźnika powierzchni liści, stężenia azotu i wydajności fotosyntezy w rolnictwie”. Remote Sensing of Environment 81, nr 2-3 (2002): 179-193._

***

## LCI – Wskaźnik chlorofilu liści

_Tylko w interfejsie graficznym (GUI) — niedostępny jako ustawienie wstępne CLI/SDK `--indices`._

Wskaźnik ten służy do szacowania zawartości chlorofilu w roślinach wyższych, wrażliwych na zmiany współczynnika odbicia spowodowane absorpcją chlorofilu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Źródło: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves” [Teledetekcja zawartości wody w liściach eukaliptusa]. Journal of Plant Physiology 154, nr 1 (1999): 30–36._

***

## MNLI – zmodyfikowany indeks nieliniowy

Wskaźnik ten stanowi ulepszenie wskaźnika nieliniowego (NLI), który uwzględnia wskaźnik roślinności skorygowany o glebę (SAVI) w celu uwzględnienia tła glebowego. Program ENVI stosuje wartość współczynnika korekcji tła korony drzew (_L_) wynoszącą 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Źródło: Yang, Z., P. Willis i R. Mueller. „Wpływ obrazu AWIFS wzbogaconego o współczynniki pasmowe na dokładność klasyfikacji upraw”. Materiały z sympozjum Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – zmodyfikowany wskaźnik roślinności skorygowany o warunki glebowe 2

Wskaźnik ten jest uproszczoną wersją wskaźnika MSAVI zaproponowanego przez Qi i in. (1994), który stanowi ulepszenie wskaźnika roślinności skorygowanego o glebę (SAVI). Zmniejsza on zakłócenia związane z glebą i zwiększa zakres dynamiczny sygnału roślinności. MSAVI2 opiera się na metodzie indukcyjnej, która nie wykorzystuje stałej wartości _L_ (jak w przypadku SAVI) w celu uwydatnienia zdrowej roślinności.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Źródło: Qi, J., A. Chehbouni, A. Huete, Y. Kerr i S. Sorooshian. „A Modified Soil Adjusted Vegetation Index” [Zmodyfikowany wskaźnik wegetacji skorygowany o właściwości gleby]. Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – zmodyfikowany wskaźnik prosty

Wskaźnik ten stanowi modyfikację prostego współczynnika NIR/Red, mającą na celu linearyzację jego zależności od parametrów biofizycznych, i wykazuje większą czułość niż NDVI przy wyższych gęstościach roślinności.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Źródło: Chen, J. „Ocena wskaźników roślinności i zmodyfikowanego wskaźnika prostego do zastosowań w strefie borealnej”. Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – znormalizowana różnica RedEdge

Wskaźnik ten jest podobny do NDVI, ale porównuje kontrast między NIR a RedEdge zamiast z Red, co często pozwala wykryćstres roślinny.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – znormalizowany wskaźnik różnicy wegetacyjnej

Wskaźnik ten służy do pomiaru zdrowej, zielonej roślinności. Połączenie jego znormalizowanej różnicy oraz wykorzystania obszarów o najwyższej absorpcji i odbiciu światła przez chlorofil sprawia, że jest on odporny na szeroki zakres warunków. Może jednak ulegać nasyceniu w warunkach gęstej roślinności, gdy wartość LAI staje się wysoka.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Wartość tego wskaźnika mieści się w przedziale od -1 do 1. Typowy zakres wartości dla zielonej roślinności wynosi od 0,2 do 0,8.

_Źródło: Rouse, J., R. Haas, J. Schell i D. Deering. Monitoring systemów roślinnych na Wielkich Równinach za pomocą ERTS. Trzecie sympozjum ERTS, NASA (1973): 309–317._

***

## NLI – wskaźnik nieliniowy

Wskaźnik ten opiera się na założeniu, że zależność między wieloma wskaźnikami roślinności a powierzchniowymi parametrami biofizycznymi ma charakter nieliniowy. Linearyzuje on zależności z parametrami powierzchniowymi, które zazwyczaj są nieliniowe.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Źródło: Goel, N. i W. Qin. „Wpływ architektury korony drzew na zależności między różnymi indeksami roślinności a LAI i Fpar: symulacja komputerowa”. Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – zoptymalizowany wskaźnik wegetacji skorygowany o właściwości gleby

Wskaźnik ten opiera się na wskaźniku wegetacji skorygowanym o właściwości gleby (SAVI). Wykorzystuje on standardową wartość 0,16 jako współczynnik korekcji tła korony drzew. Rondeaux (1996) ustalił, że wartość ta zapewnia większą zmienność gleby niż SAVI w przypadku niskiego pokrycia roślinnego, wykazując jednocześnie zwiększoną czułość na pokrycie roślinne powyżej 50%. Wskaźnik ten najlepiej sprawdza się na obszarach o stosunkowo rzadkiej roślinności, gdzie gleba jest widoczna przez korony drzew.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Źródło: Rondeaux, G., M. Steven i F. Baret. „Optimization of Soil-Adjusted Vegetation Indices”. Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormalizowany wskaźnik różnicy wegetacji

Wskaźnik ten wykorzystuje różnicę między długościami fal w bliskiej podczerwieni i w zakresie czerwieni, wraz z NDVI, w celu podkreślenia zdrowej roślinności. Jest on niewrażliwy na wpływ gleby oraz geometrii padania promieni słonecznych.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Źródło: Roujean, J. i F. Breon. „Oszacowanie PAR pochłanianego przez roślinność na podstawie dwukierunkowych pomiarów odbicia”. Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – wskaźnik roślinności skorygowany o wpływ gleby

Wskaźnik ten jest podobny do NDVI, ale eliminuje wpływ pikseli gleby. Wykorzystuje on współczynnik korekcji tła korony drzew, _L_, który jest funkcją gęstości roślinności i często wymaga wcześniejszej wiedzy na temat ilości roślinności. Huete (1988) sugeruje optymalną wartość _L_ = 0,5 w celu uwzględnienia zmian tła glebowego pierwszego rzędu. Wskaźnik ten najlepiej stosować na obszarach o stosunkowo rzadkiej roślinności, gdzie gleba jest widoczna przez korony drzew.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Źródło: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI)”. Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – przekształcony wskaźnik różnicy wegetacji

Wskaźnik ten jest przydatny do monitorowania pokrywy roślinnej w środowiskach miejskich. Nie ulega nasyceniu, w przeciwieństwie do wskaźników NDVI i SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Źródło: Bannari, A., H. Asalhi i P. Teillet. „Przekształcony wskaźnik różnicy wegetacji (TDVI) do mapowania pokrywy roślinnej” w: Materiały z sympozjum poświęconego naukom o Ziemi i teledetekcji, IGARSS &#x27;02, IEEE International, tom 5 (2002)._

***

## VARI – Wskaźnik widzialny odporny na wpływ atmosfery

Wskaźnik ten opiera się na wskaźniku ARVI i służy do oszacowania udziału roślinności w obrazie przy niskiej wrażliwości na czynniki atmosferyczne.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Źródło: Gitelson, A. i in., „Linie roślinności i gleby w przestrzeni spektralnej światła widzialnego: koncepcja i technika zdalnego szacowania udziału roślinności”. International Journal of Remote Sensing 23 (2002): 2537–2562._

***

## WDRVI – Wskaźnik roślinności o szerokim zakresie dynamicznym

Wskaźnik ten jest podobny do NDVI, ale wykorzystuje współczynnik ważenia (_a_) w celu zmniejszenia rozbieżności między udziałem sygnałów z zakresu bliskiej podczerwieni i czerwieni w wskaźniku NDVI. Wskaźnik WDRVI jest szczególnie skuteczny w scenach o umiarkowanej lub wysokiej gęstości roślinności, gdy wartość wskaźnika NDVI przekracza 0,6. Wskaźnik NDVI ma tendencję do stabilizacji wraz ze wzrostem udziału roślinności i wskaźnika powierzchni liści (LAI) wzrasta, podczas gdy wskaźnik WDRVI jest bardziej wrażliwy na szerszy zakres udziału roślinności oraz na zmiany wskaźnika LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Współczynnik ważenia (_a_) może wynosić od 0,1 do 0,2. Wartość 0,2 jest zalecana przez Henebry&#x27;ego, Viñę i Gitelsona (2004).

_Bibliografia_

_Gitelson, A. „Wskaźnik roślinności o szerokim zakresie dynamicznym do zdalnej kwantyfikacji cech biofizycznych roślinności”. Journal of Plant Physiology 161, nr 2 (2004): 165–173._

_Henebry, G., A. Viña i A. Gitelson. „Wskaźnik wegetacji o szerokim zakresie dynamicznym i jego potencjalne zastosowanie w analizie luk”. Gap Analysis Bulletin 12: 50–56._
