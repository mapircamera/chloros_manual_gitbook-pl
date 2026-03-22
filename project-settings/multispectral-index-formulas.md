---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Wzory na wskaźniki wielospektralne

Poniższe wzory na wskaźniki wykorzystują kombinację średnich zakresów przepuszczalności filtra Survey3:

<table><thead><tr><th align="center">Kolor filtra Survey3</th><th width="196.199951171875" align="center">Survey3 Nazwa filtra</th><th width="159.800048828125" align="center">Zakres przepuszczalności (FWHM)</th><th align="center">Średnia przepuszczalność</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>W przypadku stosowania tych wzorów nazwa może kończyć się na „\_1” lub „\_2”, co odpowiada temu, który filtr NIR, NIR1 czy NIR2, został użyty.

***

## EVI – Ulepszony wskaźnik wegetacji

Wskaźnik ten został pierwotnie opracowany do stosowania z danymi MODIS jako ulepszenie w stosunku do NDVI poprzez optymalizację sygnału wegetacyjnego na obszarach o wysokim wskaźniku powierzchni liści (LAI). Jest on najbardziej przydatny w regionach o wysokim wskaźniku LAI, gdzie NDVI może ulegać nasyceniu. Wykorzystuje on zakres odbicia niebieskiego do korekcji sygnałów tła gleby oraz do redukcji wpływów atmosferycznych, w tym rozpraszania aerozoli.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Wartości EVI powinny mieścić się w zakresie od 0 do 1 dla pikseli roślinności. Jasne obiekty, takie jak chmury i białe budynki, wraz z ciemnymi obiektami, takimi jak woda, mogą powodować anomalne wartości pikseli na obrazie EVI. Przed utworzeniem obrazu EVI należy zamaskować chmury i jasne obiekty z obrazu odbicia oraz opcjonalnie ustawić próg wartości pikseli od 0 do 1.

_Źródło: Huete, A. i in. „Przegląd właściwości radiometrycznych i biofizycznych wskaźników roślinności MODIS”. Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Wskaźnik pokrycia lasów 1

Wskaźnik ten odróżnia korony drzew od innych rodzajów roślinności przy użyciu wielospektralnych obrazów odbicia, które zawierają pasmo czerwonej krawędzi.

$$
FCI1 = Red * RedEdge
$$

Obszary zalesione będą miały niższe wartości FCI1 ze względu na niższy współczynnik odbicia drzew oraz obecność cieni w koronie drzew.

_Źródło: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. „Solidne wskaźniki pokrycia lasów dla obrazów wielospektralnych”. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Wskaźnik pokrycia lasem 2

Wskaźnik ten odróżnia korony drzew od innych rodzajów roślinności przy użyciu wielospektralnych obrazów odbicia, które nie zawierają pasma czerwonej krawędzi.

$$
FCI2 = Red * NIR
$$

Obszary leśne będą miały niższe wartości FCI2 ze względu na niższy współczynnik odbicia drzew oraz obecność cieni w koronie drzew.

_Źródło: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. „Robust forest cover indices for multispectral images” („Solidne wskaźniki pokrycia lasów dla obrazów wielospektralnych”). Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Wskaźnik globalnego monitoringu środowiska

Ten nieliniowy wskaźnik roślinności służy do globalnego monitorowania środowiska na podstawie obrazów satelitarnych i ma na celu korektę wpływu czynników atmosferycznych. Jest podobny do NDVI, ale jest mniej wrażliwy na czynniki atmosferyczne. Wpływa na niego odsłonięta gleba, dlatego nie zaleca się jego stosowania na obszarach o rzadkiej lub umiarkowanie gęstej roślinności.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Gdzie:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Źródło: Pinty, B. i M. Verstraete. GEMI: nieliniowy wskaźnik do monitorowania globalnej roślinności z satelitów. Vegetation 101 (1992): 15-20._

***

## GARI – Green Wskaźnik odporny na wpływ atmosfery

Wskaźnik ten jest bardziej czuły na szeroki zakres stężeń chlorofilu i mniej wrażliwy na wpływ atmosfery niż NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Stała gamma jest funkcją ważenia zależną od warunków aerozolowych w atmosferze. ENVI stosuje wartość 1,7, która jest wartością zalecaną przez Gitelsona, Kaufmana i Merzylaka (1996, strona 296).

_Źródło: Gitelson, A., Y. Kaufman i M. Merzylak. „Wykorzystanie kanału Green w teledetekcji globalnej roślinności z EOS-MODIS”. Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Wskaźnik chlorofilu

Wskaźnik ten służy do szacowania zawartości chlorofilu w liściach szerokiej gamy gatunków roślin.

$$
GCI = {NIR \over Green} - 1
$$

Zastosowanie szerokiego zakresu długości fal NIR i zielonych zapewnia lepsze przewidywanie zawartości chlorofilu, jednocześnie umożliwiając większą czułość i wyższy stosunek sygnału do szumu.

_Źródło: Gitelson, A., Y. Gritz i M. Merzlyak. „Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves” („Związki między zawartością chlorofilu w liściach a odbiciem spektralnym oraz algorytmy do nieniszczącej oceny chlorofilu w liściach roślin wyższych”). Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Indeks liści

Indeks ten został pierwotnie zaprojektowany do użytku z cyfrową kamerą RGB w celu pomiaru pokrywy pszenicy, gdzie cyfrowe wartości (DN) dla kolorów czerwonego, zielonego i niebieskiego mieszczą się w zakresie od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Wartości GLI mieszczą się w przedziale od -1 do +1. Wartości ujemne odpowiadają glebie i elementom nieożywionym, natomiast wartości dodatnie odpowiadają zielonym liściom i łodygom.

_Źródło: Louhaichi, M., M. Borman i D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.” Geocarto International 16, nr 1 (2001): 65-70._

***

## GNDVI – Green Znormalizowany wskaźnik różnicy wegetacji

Wskaźnik ten jest podobny do NDVI, z tą różnicą, że mierzy on spektrum zielone w zakresie od 540 do 570 nm zamiast spektrum czerwonego. Wskaźnik ten jest bardziej czuły na stężenie chlorofilu niż NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Źródło: Gitelson, A. i M. Merzlyak. „Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves” („Teledetekcja stężenia chlorofilu w liściach roślin wyższych”). Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Zoptymalizowany wskaźnik wegetacji skorygowany o właściwości gleby

Wskaźnik ten został pierwotnie opracowany przy użyciu fotografii kolorowo-podczerwonej w celu prognozowania zapotrzebowania kukurydzy na azot. Jest on podobny do OSAVI, ale zastępuje pasmo zielone pasmem czerwonym.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Źródło: Sripada, R. i in. „Określanie zapotrzebowania kukurydzy na azot w trakcie sezonu przy użyciu lotniczych zdjęć w kolorze i podczerwieni”. Rozprawa doktorska, Uniwersytet Stanowy Karoliny Północnej, 2005._

***

## GRVI – Wskaźnik wegetacji oparty na stosunku Green

Wskaźnik ten jest wrażliwy na tempo fotosyntezy w koronach drzew, ponieważ na odbicie światła zielonego i czerwonego silny wpływ mają zmiany w pigmentacji liści.

$$
GRVI = {NIR \over Green }
$$

_Źródło: Sripada, R. i in. „Fotografia lotnicza w kolorze i podczerwieni do określania wczesnego zapotrzebowania kukurydzy na azot w sezonie”. Agronomy Journal 98 (2006): 968-977._

***

## GSAVI – Green Wskaźnik wegetacji skorygowany o glebę

Wskaźnik ten został pierwotnie opracowany przy użyciu fotografii kolorowo-podczerwonej w celu prognozowania zapotrzebowania kukurydzy na azot. Jest on podobny do SAVI, ale zastępuje pasmo zielone pasmem czerwonym.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Źródło: Sripada, R. i in. „Określanie zapotrzebowania kukurydzy na azot w trakcie sezonu przy użyciu lotniczych zdjęć w kolorze i podczerwieni”. Rozprawa doktorska, North Carolina State University, 2005._

***

## LAI – Wskaźnik powierzchni liści

Wskaźnik ten służy do oszacowania pokrywy liściowej oraz prognozowania wzrostu i plonów upraw. ENVI oblicza zielony LAI przy użyciu następującego wzoru empirycznego autorstwa Boegha i in. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Gdzie EVI to:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Wysokie wartości LAI zazwyczaj mieszczą się w przedziale od około 0 do 3,5. Jednak gdy scena zawiera chmury i inne jasne elementy, które powodują nasycenie pikseli, wartości LAI mogą przekraczać 3,5. Najlepiej byłoby zamaskować chmury i jasne elementy ze sceny przed utworzeniem obrazu LAI.

_Źródło: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde i A. Thomsen. „Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.” Remote Sensing of Environment 81, nr 2-3 (2002): 179-193._

***

## LCI – Wskaźnik chlorofilu liści

Wskaźnik ten służy do szacowania zawartości chlorofilu w roślinach wyższych, wrażliwych na zmiany współczynnika odbicia spowodowane absorpcją chlorofilu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Źródło: Datt, B. „Zdalne wykrywanie zawartości wody w liściach eukaliptusa”. Journal of Plant Physiology 154, nr 1 (1999): 30-36._

***

## MNLI – zmodyfikowany wskaźnik nieliniowy

Wskaźnik ten stanowi ulepszenie wskaźnika nieliniowego (NLI), który uwzględnia wskaźnik wegetacji skorygowany o glebę (SAVI) w celu uwzględnienia tła glebowego. ENVI stosuje wartość współczynnika korekcji tła korony drzew (_L_) wynoszącą 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Źródło: Yang, Z., P. Willis i R. Mueller. „Wpływ obrazu AWIFS o ulepszonym stosunku pasm na dokładność klasyfikacji upraw”. Materiały z sympozjum Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – zmodyfikowany wskaźnik wegetacji skorygowany o glebę 2

Wskaźnik ten jest uproszczoną wersją wskaźnika MSAVI zaproponowanego przez Qi i in. (1994), który stanowi ulepszenie wskaźnika wegetacji skorygowanego o glebę (SAVI). Zmniejsza on szum gleby i zwiększa zakres dynamiczny sygnału wegetacyjnego. MSAVI2 opiera się na metodzie indukcyjnej, która nie wykorzystuje stałej wartości _L_ (jak w przypadku SAVI) do wyróżnienia zdrowej roślinności.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Źródło: Qi, J., A. Chehbouni, A. Huete, Y. Kerr i S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – znormalizowana różnica RedEdge

Wskaźnik ten jest podobny do NDVI, ale porównuje kontrast między NIR a RedEdge zamiast Red, co często pozwala wykryć stres roślinności wcześniej.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – znormalizowany wskaźnik różnicy wegetacji

Wskaźnik ten jest miarą zdrowej, zielonej roślinności. Połączenie jego znormalizowanej formuły różnicy oraz wykorzystania obszarów o najwyższej absorpcji i odbiciu chlorofilu sprawia, że jest on odporny na szeroki zakres warunków. Może jednak ulegać nasyceniu w warunkach gęstej roślinności, gdy wartość LAI staje się wysoka.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Wartość tego wskaźnika mieści się w przedziale od -1 do 1. Typowy zakres dla zielonej roślinności wynosi od 0,2 do 0,8.

_Źródło: Rouse, J., R. Haas, J. Schell i D. Deering. Monitoring systemów roślinności na Wielkich Równinach za pomocą ERTS. Trzecie sympozjum ERTS, NASA (1973): 309-317._

***

## NLI – wskaźnik nieliniowy

Wskaźnik ten zakłada, że zależność między wieloma wskaźnikami roślinności a powierzchniowymi parametrami biofizycznymi jest nieliniowa. Linearyzuje on zależności z parametrami powierzchniowymi, które zazwyczaj są nieliniowe.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Źródło: Goel, N. i W. Qin. „Wpływ architektury korony drzew na zależności między różnymi indeksami roślinności a LAI i Fpar: symulacja komputerowa”. Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Zoptymalizowany wskaźnik wegetacji skorygowany o glebę

Wskaźnik ten opiera się na wskaźniku wegetacji skorygowanym o glebę (SAVI). Wykorzystuje on standardową wartość 0,16 jako współczynnik korekcji tła korony drzew. Rondeaux (1996) ustalił, że wartość ta zapewnia większą zmienność gleby niż SAVI w przypadku niskiego pokrycia roślinnością, wykazując jednocześnie zwiększoną czułość na pokrycie roślinnością powyżej 50%. Wskaźnik ten najlepiej sprawdza się na obszarach o stosunkowo rzadkiej roślinności, gdzie gleba jest widoczna przez koronę drzew.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Źródło: Rondeaux, G., M. Steven i F. Baret. „Optimization of Soil-Adjusted Vegetation Indices”. Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalized Difference Vegetation Index

Wskaźnik ten wykorzystuje różnicę między długościami fal bliskiej podczerwieni i czerwieni, wraz z NDVI, w celu wyróżnienia zdrowej roślinności. Jest on niewrażliwy na wpływ gleby i geometrii widzenia Słońca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Źródło: Roujean, J. i F. Breon. „Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements” („Oszacowanie PAR pochłanianego przez roślinność na podstawie pomiarów odbicia dwukierunkowego”). Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Wskaźnik roślinności skorygowany o glebę

Wskaźnik ten jest podobny do NDVI, ale eliminuje wpływ pikseli gleby. Wykorzystuje on współczynnik korekcji tła korony drzew, _L_, który jest funkcją gęstości roślinności i często wymaga wcześniejszej wiedzy na temat ilości roślinności. Huete (1988) sugeruje optymalną wartość _L_=0,5 w celu uwzględnienia zmian tła gleby pierwszego rzędu. Wskaźnik ten najlepiej sprawdza się na obszarach o stosunkowo rzadkiej roślinności, gdzie gleba jest widoczna przez korony drzew.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Źródło: Huete, A. „Wskaźnik roślinności skorygowany o glebę (SAVI)”. Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformed Difference Vegetation Index (Przekształcony wskaźnik różnicowy roślinności)

Wskaźnik ten jest przydatny do monitorowania pokrywy roślinnej w środowiskach miejskich. Nie ulega nasyceniu, jak NDVI i SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Źródło: Bannari, A., H. Asalhi i P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” [Przekształcony wskaźnik różnicowy roślinności (TDVI) do mapowania pokrywy roślinnej] w: Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, tom 5 (2002)._

***

## VARI – Wskaźnik widzialny odporny na wpływ atmosfery

Wskaźnik ten opiera się na ARVI i służy do oszacowania udziału roślinności w scenie przy niskiej wrażliwości na czynniki atmosferyczne.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Źródło: Gitelson, A. i in. „Linie roślinności i gleby w przestrzeni widmowej widzialnej: koncepcja i technika zdalnego szacowania udziału roślinności”. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Wskaźnik roślinności o szerokim zakresie dynamicznym

Wskaźnik ten jest podobny do NDVI, ale wykorzystuje współczynnik ważenia (_a_) w celu zmniejszenia dysproporcji między udziałem sygnałów bliskiej podczerwieni i czerwonych w NDVI. Wskaźnik WDRVI jest szczególnie skuteczny w scenach o umiarkowanej lub wysokiej gęstości roślinności, gdy wartość NDVI przekracza 0,6. NDVI ma tendencję do wyrównywania się wraz ze wzrostem udziału roślinności i wskaźnika powierzchni liści (LAI), podczas gdy WDRVI jest bardziej wrażliwy na szerszy zakres udziału roślinności oraz na zmiany w LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Współczynnik ważenia (_a_) może wynosić od 0,1 do 0,2. Wartość 0,2 jest zalecana przez Henebry&#x27;ego, Viñę i Gitelsona (2004).

_Bibliografia_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, nr 2 (2004): 165-173._

_Henebry, G., A. Viña i A. Gitelson. „Wskaźnik wegetacji o szerokim zakresie dynamicznym i jego potencjalna przydatność w analizie luk”. Gap Analysis Bulletin 12: 50–56._
