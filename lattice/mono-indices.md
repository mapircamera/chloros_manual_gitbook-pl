# Kamery monochromatyczne i wskaźniki wegetacji

## Jedna kamera = jedno pasmo

Kamera **M3M**jest monochromatycznym odpowiednikiem modelu Bayer**M3C**: zawiera monochromatyczny czujnik IMX265 z pojedynczym wąskopasmowym filtrem interferencyjnym. Nazwa modelu określa pasmo — `M3M-<lens>-F<wavelength>`, np. `M3M-L87-F685` (wyświetlane w programie Chloros

jako `LATT-M3M-L87-F685`). Czujnik dostarcza **pojedyncze pasmo w skali szarości** bez mozaiki Bayera: nie ma tu nic do demosaikowania, nie ma przesłuchu międzykanałowego do rozdzielenia ani balansu bieli do ustawienia.

Konsekwencje, o których warto wiedzieć przed zaplanowaniem systemu monochromatycznego:

* **Promieniowanie i odbicie są w pełni zdefiniowane dla każdego pasma.**Są to mapy radiometryczne dla poszczególnych pasm, więc jedna kamera M3M generuje skalibrowaną promieniowanie typu float32 (W/m²/sr/nm) oraz odbicie typu uint16 (`32768` = ρ 1,0) dokładnie tak samo, jak robi to pasmo M3C. Klatki monochromatyczne zawierają**identyczną** macierz odpowiedzi czujnika — nie jest wymagane ani stosowane rozdzielanie 3×3.
* **Pojedyncza kamera monochromatyczna nie może generować wskaźnika wegetacji.** Wskaźniki NDVI

, NDRE

i podobne wymagają co najmniej dwóch pasm. Aby obliczyć wskaźniki przy użyciu sprzętu monochromatycznego, należy połączyć kilka kamer M3M — patrz poniżej.
* Kamery M3M przesyłają strumieniowo **Mono12** (12-bitowy: 2 bajty/piksel w transmisji lub 1,5 bajta/piksel, gdy macierz wykorzystuje [upakowany transport 12-bitowy](arrays.md#packed-12-bit-transport)), co ma znaczenie dla [planowania przepustowości macierzy](arrays.md#bandwidth-how-many-cameras-fit).

## Co program Chloros

pomija w trybie mono — i jak o tym informuje

Etapy potoku przetwarzania kolorów po prostu nie mają zastosowania w przypadku czujnika jednopasmowego. Chloros

**pomija je, wyświetlając jednowierszowy komunikat** zamiast zgłaszać błąd, a mimo to nadal wykonuje je normalnie dla każdej kamery M3C (Bayer) w tej samej sesji:

| Etap | Zachowanie w trybie monochromatycznym (M3M) | Zachowanie w trybie M3C |
| --- | --- | --- |
| Demosaic / debayer | Pominięte — poziom eksportu `debayered` to 1-kanałowy obraz w skali szarości. | 3-kanałowy demosaic. |
| Balans bieli (`lattice white-balance`) | Pominięto z jednowierszowym komunikatem. | Działa normalnie. |
| Profil kolorów (`lattice color-profile`) | Pominięto z jednowierszowym komunikatem. | Działa normalnie. |
| Nasycenie/kontrast (`lattice color`) | Pominięto z jednowierszowym komunikatem. | Działa normalnie. |
| Rozdzielanie przesłuchu spektralnego | Tożsamość (brak macierzy 3×3). | Zastosowano macierz 3×3 dla każdej kamery. |
| Promieniowanie / odbicie | **Działa** — dla każdego pasma, w pełni skalibrowane. | Działa dla każdego pasma. |

Interfejs graficzny stosuje te same ograniczenia: w przypadku kamery monochromatycznej panel ustawień dla poszczególnych kamer ukrywa wiersze dotyczące wyłącznie RGB

(balans bieli, gamma, profil kolorów, nasycenie, kontrast, podział kanałów), a histogram na żywo jest zablokowany na pojedynczym wykresie **MONO**. Czynnikiem rozróżniającym w całym stosie jest token `M3M` w ciągu znaków modelu, wyświetlany w GUI/SDK

jako `is_mono`.

## Indeksy wymagają ≥ 2 pasm: wyrównanie → stosowanie → indeksowanie

Przebieg indeksowania mono zawsze składa się z tych samych trzech etapów:

1. **Wyrównanie** — skieruj kilka kamer M3M na różne długości fal (np. F650 „Red

” i F850 „NIR

”), połącz je w [układ wielokamerowy](arrays.md), a następnie pozwól programowi Chloros

obliczyć transformację współrejestracji między kamerami.
2. **Stosowanie** — wyrównane klatki stają się jednym obrazem wielopasmowym (każda kamera dostarcza jedno nazwane pasmo).
3. **Indeks** — obliczanie wzoru indeksu dla pasm stosu, opcjonalnie z renderowaniem za pomocą tablicy LUT.

W interfejsie graficznym cały ten łańcuch stanowi tryb wyświetlania macierzy **Combined Cameras**: kompozycja na żywo jest już wyrównana, a Kalkulator indeksu macierzy (poniżej) definiuje wzór, na podstawie którego jest renderowana. Zarejestrowane eksporty można dopasować do tego samego wyrównania za pomocą opcji przechwytywania**Aligned**.

## Kalkulator indeksu

Kalkulator indeksu tworzy wyrażenie indeksowe używane w podglądzie na żywo oraz w eksportach indeksu dla poszczególnych kamer. Jest to jeden wspólny panel, otwierany z dwóch miejsc na pasku bocznym zakładki „Kamery”:

* **Dla poszczególnych kamer**— Podgląd na żywo → ikona koła zębatego**Indeks** (tylko kamery RGN

/OCN

/NGB

typu Bayer; pojedyncza kamera monochromatyczna nie ma kontroli indeksu, ponieważ jedno pasmo nie pozwala na utworzenie indeksu).
* **Dla matrycy**— ustawienia matrycy → Podgląd na żywo → ikona koła zębatego**Indeks**. Jest to ścieżka dla kamer monochromatycznych: lista pasm obejmuje**wszystkie kamery wchodzące w skład matrycy**, więc para kamer monochromatycznych wnosi tutaj swoje dwa pasma.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Elementy sterujące, od góry do dołu:

* **Chipy pasmowe** („Pasma — kliknij, aby dodać do wyrażenia”) — jeden przycisk na każde dostępne pasmo, oznaczony nazwą koloru + długością fali w nm (powtarzające się nazwy kolorów są rozróżniane, np. „Kolor 850”). Kliknięcie wstawia token pasma w miejscu kursora. Pasma z kamer, które nie mogą generować promieniowania dla poszczególnych pasm (RGB

/FRGB), są odfiltrowywane.
* **Przyciski operatorów i funkcji** — `+ - * / ( ) ^ ,` oraz `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Pole tekstowe wyrażenia** — dowolnie wpisywana formuła; symbol zastępczy pokazuje klasyczną postać NDVI

: `(NIR - Red) / (NIR + Red)`. Znajdujący się nad nim podgląd w trybie tylko do odczytu, podzielony na tokeny, renderuje fragmenty pasm, liczby i flagi jako nieznane tokeny.
* **Pasek ważności**— szary „Puste — nie zostanie zastosowany żaden indeks”; zielony „Prawidłowe wyrażenie”; czerwony z konkretnym błędem analizy (nieznane pasmo, niejednoznaczne pasmo wykrywane przez wiele kamer, brakujący nawias, …); lub bursztynowy, gdy wyrażenie jest poprawne, ale**stała** (np. `X/X` lub mianownik NDVI

wpisany jako `−` zamiast `+`) — stała mapuje całą klatkę na jeden kolor.
* Osobne pomarańczowe ostrzeżenie pojawia się, jeśli zastosowane wyrażenie jest poprawne, ale **klatka na żywo jest jednolita** (scena płaska lub nasycona) — wykryto za Ciebie zapadnięcie się histogramu.
* **Zastosuj LUT**(domyślnie włączone; wyłączone = rozciągnięcie skali szarości),**Poziom**2/3/5/7 stopni (domyślnie 7 stopni) oraz pola**Min / Max**po obu stronach paska gradientu. Min domyślnie wynosi**0,2**— powiększa to rampę kolorów do zakresu istotnego dla roślinności, podczas gdy wartości poniżej tego poziomu są traktowane jako skala szarości; ustaw Min na −1, aby uzyskać pełny zakres indeksu (przycisk**Reset** przywraca zakres −1…+1). Max domyślnie wynosi 1.
* **Histogram na żywo** przedstawiający rozkład indeksu — słupki w skali pierwiastkowej, bursztynowe linie 2. i 98. percentyla, biała linia mediany oraz odczyty wartości poza zakresem („◀ N% &lt; lo” / „hi &lt; N% ▶”), które zmieniają kolor na bursztynowy powyżej 1 %, co stanowi sygnał do poszerzenia okna Min/Max.
* Przycisk **Zastosuj** zatwierdza wyrażenie do strumienia na żywo; poprawki LUT są stosowane na bieżąco bez konieczności naciskania przycisku Zastosuj. Przy otwartym projekcie ustawienia wyrażenia i LUT są zapisywane wraz z projektem i przywracane po jego ponownym otwarciu; bez projektu pozostają aktywne tylko na czas trwania sesji.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Ścieżka CLI



Ten sam łańcuch: wyrównanie → stosowanie → indeksowanie, z możliwością skryptowania od początku do końca:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` wykorzystuje połączoną tablicę, która przechowuje kamery, lub otwiera je samodzielnie na czas jednego przebiegu. `index --live` zawsze samodzielnie otwiera kamery (za pomocą [tablicy tymczasowej](../reference/cli-reference.md#multi-camera-transient-sessions)) i nie posiada `--serials`: otwiera każdą kamerę w sieci. Należy więc zwolnić każdą kamerę, którą Chloros

trzyma przed nim — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): ponowne otwarcie zajętej kamery może zatrzymać strumień jej posiadacza lub spowodować awarię zaplecza. Kamera, która nie dostarcza klatek, powoduje niepowodzenie przebiegu, a stos ani indeks nie są zapisywane.

`--channel` mapuje symbole presetu na nazwy pasm stosu. Dwie zasady pozwolą uniknąć nieudanego przebiegu:

* **W symbolach rozróżniana jest wielkość liter** i muszą one dokładnie odpowiadać nazwom kanałów presetu — presety używają małych liter (dla NDVI

są to `red`, `nir`; sprawdź `--list-presets`). `--channel red=Red_660` działa; `--channel RED=660` kończy się niepowodzeniem z błędem `channel_map missing entries`.
* Strona pasma musi określać nazwę pasma w wyrównanym stosie (`lattice align-info --profile align.json` zawiera ich listę). Tryb offline akceptuje również indeksy pasm zaczynające się od 0, np. `--channel red=0 --channel nir=1`.

`lattice index` działa również w trybie offline na zapisanym, wyrównanym pliku TIFF

zawierającym wiele pasm. Każdy formularz działa na serwerze, więc musi on być uruchomiony i zalogowany na płatnym planie Chloros

+; względne `--input` lub `--output` odnoszą się do folderu, z którego uruchamiasz polecenie:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Ustawienia wstępne indeksu

`lattice index --preset` (oraz opcja [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) w zakładce „Obraz”, która korzysta z tego samego silnika) zawiera następujące **22 ustawienia wstępne**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Uruchom `chloros-cli lattice index --list-presets`, aby wyświetlić wzory i symbole kanałów dla każdego ustawienia wstępnego, oraz `--list-gradients`, aby wyświetlić dostępne gradienty kolorów. W przypadku wzorów niestandardowych należy użyć `--formula EXPR`, stosując tę samą składnię co w Kalkulatorze indeksów. Należy pamiętać, że ta lista ustawień wstępnych dotyczy wyłącznie silnika indeksów LATTICE — lista dostępna w menu rozwijanym „Przetwarzanie” w ustawieniach projektu dla importowanych obrazów jest inna (zobacz [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md)).

Kompletny zestaw flag (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, pokrętła wyrównywania i zniekształcania dla `--live` i inne) został opisany w [Podręczniku CLI

§ Indeksy / Obliczenia wegetacyjne](../reference/cli-reference.md#index--vegetation-maths); odpowiedniki w SDK

znajdują się w [Podręczniku SDK

](../reference/sdk-reference.md).

## Przechwytywanie produktów indeksowych z tablicy mono

Po podłączeniu tablicy i zastosowaniu wyrażenia indeksowego, `array-capture` (lub opcja GUI **Capture All**) zapisuje poziomy eksportu dla każdej kamery *oraz* render indeksu — `--index`/`--no-index` włącza tę opcję w CLI

, a domyślnie przechwytuje wszystkie odpowiednie poziomy. Wkład kamery mono do każdej grupy przechwytywania to jej jedno pasmo na poziomach surowych/po usunięciu debayeringu (skala szarości)/radiancji/odbicia, a także wspólny kompozyt z połączonym indeksem, gdy macierz działa w trybie połączonym. Zobacz [Układ wielokamerowy § Rejestracja](arrays.md#capturing-monitoring-vs-analysis).
