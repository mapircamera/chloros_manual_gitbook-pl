---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Wzory indeksów wielospektralnych

Poniższe wzory indeksów wykorzystują kombinację średnich zakresów przepuszczalności filtrów Survey3:

<table><thead><tr><th align="center">Survey3 Kolor filtra</th><th width="196.199951171875" align="center">Nazwa filtra Survey3</th><th width="159.800048828125" align="center">Zakres transmisji (FWHM)</th><th align="center">Średnia transmisja</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

W przypadku zastosowania tych wzorów nazwa może kończyć się na „\_1” lub „\_2”, co odpowiada filtrowi NIR, NIR1 lub NIR2.

***

## EVI — ulepszony wskaźnik wegetacji

Wskaźnik ten został pierwotnie opracowany do użytku z danymi MODIS jako ulepszenie w stosunku do NDVI poprzez optymalizację sygnału wegetacji w obszarach o wysokim wskaźniku powierzchni liści (LAI). Jest on najbardziej przydatny w regionach o wysokim LAI, gdzie NDVI może ulec nasyceniu. Wykorzystuje on obszar odbicia niebieskiego do korekty sygnałów tła gleby i zmniejszenia wpływu atmosferycznego, w tym rozpraszania aerozoli.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Wartości EVI powinny mieścić się w zakresie od 0 do 1 dla pikseli roślinności. Jasne elementy, takie jak chmury i białe budynki, wraz z ciemnymi elementami, takimi jak woda, mogą powodować anomalie wartości pikseli w obrazie EVI. Przed utworzeniem obrazu EVI należy zamaskować chmury i jasne elementy z obrazu odbicia, a opcjonalnie ustawić próg wartości pikseli od 0 do 1.

_Źródło: Huete, A. i in. „Przegląd wydajności radiometrycznej i biofizycznej wskaźników roślinności MODIS”. Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 — wskaźnik pokrycia lasów 1

Wskaźnik ten odróżnia korony drzew od innych rodzajów roślinności za pomocą wielospektralnych obrazów odbicia, które zawierają pasmo czerwonej krawędzi.

$$
FCI1 = Red * RedEdge
$$

Obszary leśne będą miały niższe wartości FCI1 ze względu na niższe odbicie drzew i obecność cieni w koronie drzew.

_Źródło: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. „Solidne wskaźniki pokrycia lasów dla obrazów wielospektralnych”. Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – wskaźnik pokrycia lasów 2

Wskaźnik ten odróżnia korony drzew od innych rodzajów roślinności za pomocą obrazów wielospektralnych odbicia, które nie zawierają pasma czerwonej krawędzi.

$$
FCI2 = Red * NIR
$$

Obszary leśne będą miały niższe wartości FCI2 ze względu na niższy współczynnik odbicia drzew i obecność cieni w koronie drzew.

_Źródło: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. „Robust forest cover indices for multispectral images” (Solidne wskaźniki pokrycia lasów dla obrazów wielospektralnych). Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – globalny wskaźnik monitorowania środowiska

Ten nieliniowy wskaźnik roślinności jest wykorzystywany do globalnego monitorowania środowiska na podstawie zdjęć satelitarnych i ma na celu korektę wpływu atmosfery. Jest podobny do NDVI, ale jest mniej wrażliwy na wpływ atmosfery. Wpływa na niego goła gleba, dlatego nie zaleca się jego stosowania na obszarach o rzadkiej lub umiarkowanie gęstej roślinności.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Gdzie:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Źródło: Pinty, B. i M. Verstraete. GEMI: nieliniowy wskaźnik do monitorowania globalnej roślinności z satelitów. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indeks odporności na warunki atmosferyczne

Indeks ten jest bardziej wrażliwy na szeroki zakres stężeń chlorofilu i mniej wrażliwy na wpływ warunków atmosferycznych niż NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Stała gamma jest funkcją ważenia zależną od warunków aerozolowych w atmosferze. ENVI stosuje wartość 1,7, która jest wartością zalecaną przez Gitelsona, Kaufmana i Merzylaka (1996, strona 296).

_Źródło: Gitelson, A., Y. Kaufman i M. Merzylak. „Wykorzystanie kanału Green w zdalnym wykrywaniu globalnej roślinności z EOS-MODIS”. Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Indeks chlorofilu

Indeks ten służy do szacowania zawartości chlorofilu w liściach wielu różnych gatunków roślin.

$$
GCI = {NIR \over Green} - 1
$$

Szerokie pasmo NIR i zielone długości fal zapewniają lepsze przewidywanie zawartości chlorofilu, jednocześnie umożliwiając większą czułość i wyższy stosunek sygnału do szumu.

_Źródło: Gitelson, A., Y. Gritz i M. Merzlyak. „Związki między zawartością chlorofilu w liściach a odbiciem spektralnym oraz algorytmy nieniszczącej oceny chlorofilu w liściach roślin wyższych”. Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI – Green Indeks liści

Indeks ten został pierwotnie zaprojektowany do użytku z cyfrową kamerą RGB do pomiaru pokrycia pszenicy, gdzie czerwone, zielone i niebieskie wartości cyfrowe (DN) mieszczą się w zakresie od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Wartości GLI mieszczą się w zakresie od -1 do +1. Wartości ujemne reprezentują glebę i elementy nieożywione, natomiast wartości dodatnie reprezentują zielone liście i łodygi.

_Źródło: Louhaichi, M., M. Borman i D. Johnson. „Platforma lokalizacji przestrzennej i fotografia lotnicza do dokumentacji wpływu wypasu na pszenicę”. Geocarto International 16, nr 1 (2001): 65-70._

***

## GNDVI - Green Znormalizowany wskaźnik różnicy wegetacji

Indeks ten jest podobny do NDVI, z tą różnicą, że mierzy spektrum zielone od 540 do 570 nm zamiast spektrum czerwonego. Indeks ten jest bardziej wrażliwy na stężenie chlorofilu niż NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Źródło: Gitelson, A. i M. Merzlyak. „Zdalne wykrywanie stężenia chlorofilu w liściach roślin wyższych”. Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Zoptymalizowany indeks wegetacji dostosowany do gleby

Indeks ten został pierwotnie opracowany z wykorzystaniem fotografii kolorowo-podczerwonej w celu przewidywania zapotrzebowania kukurydzy na azot. Jest podobny do OSAVI, ale zastępuje pasmo zielone pasmem czerwonym.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Źródło: Sripada, R. i in. „Określanie zapotrzebowania kukurydzy na azot w sezonie przy użyciu kolorowej fotografii w podczerwieni”. Rozprawa doktorska, North Carolina State University, 2005._

***

## GRVI – Green Wskaźnik wegetacji oparty na stosunku

Indeks ten jest wrażliwy na tempo fotosyntezy w koronach drzew, ponieważ odbicie światła zielonego i czerwonego jest silnie uzależnione od zmian w pigmentach liści.

$$
GRVI = {NIR \over Green }
$$

_Źródło: Sripada, R. i in. „Aerial Color Infrared Photography for Determining Early In-season Nitrogen Requirements in Corn” (Fotografia lotnicza w kolorze i podczerwieni do określania wczesnego zapotrzebowania na azot w uprawie kukurydzy). Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Wskaźnik wegetacji skorygowany o właściwości gleby

Indeks ten został pierwotnie opracowany z wykorzystaniem kolorowej fotografii w podczerwieni w celu przewidywania zapotrzebowania kukurydzy na azot. Jest podobny do SAVI, ale zastępuje pasmo zielone pasmem czerwonym.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Źródło: Sripada, R. i in. „Określanie zapotrzebowania na azot w sezonie wegetacyjnym dla kukurydzy przy użyciu kolorowej fotografii w podczerwieni”. Rozprawa doktorska, North Carolina State University, 2005 r._

***

## LAI – wskaźnik powierzchni liści

Wskaźnik ten służy do oszacowania pokrycia liśćmi oraz prognozowania wzrostu i plonów upraw. ENVI oblicza zielony LAI przy użyciu następującego wzoru empirycznego autorstwa Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Gdzie EVI jest:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Wysokie wartości LAI zazwyczaj mieszczą się w zakresie od około 0 do 3,5. Jednakże, gdy scena zawiera chmury i inne jasne elementy, które powodują nasycenie pikseli, wartości LAI mogą przekraczać 3,5. Najlepiej byłoby zamaskować chmury i jasne elementy ze sceny przed utworzeniem obrazu LAI.

_Źródło: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde i A. Thomsen. „Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture” (Dane wielospektralne z powietrza do ilościowego określania wskaźnika powierzchni liści, stężenia azotu i wydajności fotosyntezy w rolnictwie). Remote Sensing of Environment 81, nr 2-3 (2002): 179-193._

***

## LCI – wskaźnik chlorofilu liściowego

Wskaźnik ten służy do oszacowania zawartości chlorofilu w roślinach wyższych, wrażliwych na zmiany odbicia spowodowane absorpcją chlorofilu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Źródło: Datt, B. „Zdalne wykrywanie zawartości wody w liściach eukaliptusa”. Journal of Plant Physiology 154, nr 1 (1999): 30-36._

***

## MNLI – zmodyfikowany wskaźnik nieliniowy

Indeks ten stanowi ulepszenie indeksu nieliniowego (NLI), który uwzględnia indeks wegetacji dostosowany do gleby (SAVI) w celu uwzględnienia tła gleby. ENVI stosuje wartość współczynnika dostosowania tła korony drzew (_L_) wynoszącą 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Źródło: Yang, Z., P. Willis i R. Mueller. „Wpływ obrazu AWIFS o zwiększonym współczynniku pasma na dokładność klasyfikacji upraw”. Materiały z sympozjum Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – zmodyfikowany wskaźnik roślinności dostosowany do gleby 2

Indeks ten jest uproszczoną wersją indeksu MSAVI zaproponowanego przez Qi i in. (1994), który stanowi ulepszenie indeksu roślinności skorygowanego o glebę (SAVI). Zmniejsza on zakłócenia związane z glebą i zwiększa zakres dynamiczny sygnału roślinności. MSAVI2 opiera się na metodzie indukcyjnej, która nie wykorzystuje stałej wartości _L_ (jak w przypadku SAVI) w celu podkreślenia zdrowej roślinności.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Źródło: Qi, J., A. Chehbouni, A. Huete, Y. Kerr i S. Sorooshian. „A Modified Soil Adjusted Vegetation Index” (Zmodyfikowany wskaźnik roślinności dostosowany do warunków glebowych). Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – znormalizowana różnica RedEdge

Indeks ten jest podobny do NDVI, ale porównuje kontrast między NIR a RedEdge zamiast Red, co często pozwala szybciej wykryć stres roślinności.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – znormalizowany wskaźnik różnicy wegetacji

Wskaźnik ten jest miarą zdrowej, zielonej roślinności. Połączenie znormalizowanej formuły różnicy i wykorzystania regionów o najwyższej absorpcji i odbiciu chlorofilu sprawia, że jest on odporny na szeroki zakres warunków. Może jednak ulec nasyceniu w warunkach gęstej roślinności, gdy LAI osiąga wysoką wartość.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Wartość tego wskaźnika mieści się w zakresie od -1 do 1. Typowy zakres dla zielonej roślinności wynosi od 0,2 do 0,8.

_Źródło: Rouse, J., R. Haas, J. Schell i D. Deering. Monitorowanie systemów roślinności na Wielkich Równinach za pomocą ERTS. Trzecie sympozjum ERTS, NASA (1973): 309-317._

***

## NLI – wskaźnik nieliniowy

Wskaźnik ten zakłada, że zależność między wieloma wskaźnikami roślinności a parametrami biofizycznymi powierzchni jest nieliniowa. Liniuje on zależności z parametrami powierzchni, które mają tendencję do bycia nieliniowymi.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Źródło: Goel, N. i W. Qin. „Wpływ architektury korony drzew na relacje między różnymi wskaźnikami roślinności a LAI i Fpar: symulacja komputerowa”. Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – zoptymalizowany wskaźnik roślinności dostosowany do gleby

Wskaźnik ten opiera się na wskaźniku roślinności dostosowanym do gleby (SAVI). Wykorzystuje on standardową wartość 0,16 dla współczynnika dostosowania tła korony drzew. Rondeaux (1996) ustalił, że wartość ta zapewnia większą zmienność gleby niż SAVI w przypadku niskiego pokrycia roślinnością, wykazując jednocześnie zwiększoną wrażliwość na pokrycie roślinnością powyżej 50%. Indeks ten najlepiej stosować na obszarach o stosunkowo rzadkiej roślinności, gdzie gleba jest widoczna przez korony drzew.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Źródło: Rondeaux, G., M. Steven i F. Baret. „Optimization of Soil-Adjusted Vegetation Indices” (Optymalizacja wskaźników roślinności dostosowanych do gleby). Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – renormalizowany wskaźnik różnicy roślinności

Indeks ten wykorzystuje różnicę między długościami fal bliskiej podczerwieni i czerwieni wraz z NDVI w celu podkreślenia zdrowej roślinności. Jest on niewrażliwy na wpływ gleby i geometrii widzenia słońca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Źródło: Roujean, J. i F. Breon. „Oszacowanie PAR pochłanianego przez roślinność na podstawie pomiarów dwukierunkowego współczynnika odbicia”. Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – indeks roślinności skorygowany o glebę

Indeks ten jest podobny do NDVI, ale tłumi wpływ pikseli gleby. Wykorzystuje współczynnik korekty tła korony drzew, _L_, który jest funkcją gęstości roślinności i często wymaga wcześniejszej wiedzy na temat ilości roślinności. Huete (1988) sugeruje optymalną wartość _L_=0,5, aby uwzględnić zmiany tła gleby pierwszego rzędu. Indeks ten najlepiej stosować na obszarach o stosunkowo rzadkiej roślinności, gdzie gleba jest widoczna przez korony drzew.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Źródło: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI)”. Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – przekształcony różnicowy wskaźnik wegetacji

Wskaźnik ten jest przydatny do monitorowania pokrywy roślinnej w środowisku miejskim. Nie ulega nasyceniu, jak NDVI i SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Źródło: Bannari, A., H. Asalhi i P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” [Przekształcony różnicowy wskaźnik wegetacji (TDVI) do mapowania pokrywy roślinnej] w: Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, tom 5 (2002)._

***

## VARI – indeks odporności na warunki atmosferyczne w zakresie widzialnym

Indeks ten opiera się na ARVI i służy do oszacowania udziału roślinności w scenie przy niskiej wrażliwości na warunki atmosferyczne.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Źródło: Gitelson, A. i in. „Linie roślinności i gleby w widzialnej przestrzeni spektralnej: koncepcja i technika zdalnego szacowania udziału roślinności”. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI — indeks roślinności o szerokim zakresie dynamicznym

Indeks ten jest podobny do NDVI, ale wykorzystuje współczynnik ważenia (_a_) w celu zmniejszenia rozbieżności między udziałem sygnałów bliskiej podczerwieni i czerwieni w NDVI. WDRVI jest szczególnie skuteczny w scenach o umiarkowanej lub dużej gęstości roślinności, gdy NDVI przekracza 0,6. Wartość NDVI ma tendencję do wyrównywania się wraz ze wzrostem udziału roślinności i wskaźnika powierzchni liści (LAI), natomiast wartość WDRVI jest bardziej wrażliwa na szerszy zakres udziałów roślinności i zmiany wartości LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Współczynnik ważenia (_a_) może wynosić od 0,1 do 0,2. Wartość 0,2 jest zalecana przez Henebry&#x27;ego, Viñę i Gitelsona (2004).

_Referencje_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation” [Indeks roślinności o szerokim zakresie dynamicznym do zdalnej kwantyfikacji właściwości biofizycznych roślinności]. Journal of Plant Physiology 161, nr 2 (2004): 165-173._

_Henebry, G., A. Viña i A. Gitelson. „Indeks roślinności o szerokim zakresie dynamicznym i jego potencjalne zastosowanie w analizie luk”. Gap Analysis Bulletin 12: 50-56._
