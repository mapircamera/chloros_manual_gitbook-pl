---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formaty obrazów wyjściowych

Chloros eksportuje przetworzone produkty w czterech formatach plików. Format należy wybrać w ustawieniach projektu (GUI), za pomocą polecenia `--format` (CLI) lub `export_format` (SDK). Interfejsy CLI i SDK akceptują dokładnie poniższe ciągi znaków.

| Ciąg formatujący | Rozszerzenie | Typ piksela | Zakres pikseli | Uwagi |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(domyślnie)* | `.tif` | liczba cyfrowa typu uint16 | 0 – 65535 | Zalecane do fotogrametrii / GIS. |
| `TIFF (8-bit)` | `.tif` | liczba cyfrowa typu uint8 | 0 – 255 | Dla wszystkich ujęć 8-bitowych, które oferują wyłącznie formaty 8-bitowe: Chloros nie zwiększa głębi bitowej. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% współczynnik odbicia. Niektóre aplikacje nie obsługują plików TIFF z liczbami zmiennoprzecinkowymi; pliki są większe. |
| `PNG (8-bit)` | `.png` | liczba cyfrowa typu uint8 | 0 – 255 | Bezstratna kompresja, odpowiednia do przeglądania w sieci i wizualizacji. |
| `JPG (8-bit)` | `.jpg` | liczba cyfrowa typu uint8 | 0 – 255 | Kompresja stratna, najmniejsze pliki. |

## Gdzie zapisywane są pliki wyjściowe

Pliki są zapisywane w folderze projektu, pogrupowane według kamery, a następnie według formatu pliku:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folder kamery to `LATT-<sensor>-<lens>-F<filter>` dla LATTICE oraz `<model>_<filter>` (np. `Survey3N_RGN`) dla Survey3. **Każdy wyeksportowany produkt zachowuje nazwę pliku źródłowego — to folder identyfikuje produkt, a nie rozszerzenie nazwy pliku.** Pełne zasady znajdziesz w sekcji [Gdzie zapisywane są pliki wyjściowe](reference/cli-reference.md) w dokumentacji CLI.

## Produkty LATTICE (poziomy przechwytywania i eksportu)

Jedna surowa klatka LATTICE jest rozdzielana na wszystkie żądane produkty w jednym przebiegu. Każdy typ produktu ma własny przełącznik (pola wyboru w GUI lub w CLI: `--debayered` / `--preview` / `--radiance` / `--reflectance`, domyślnie wszystkie włączone):

| Poziom | Zawartość | Typ danych |
| --- | --- | --- |
| `raw` | Dane Bayer prosto z czujnika (kamery monochromatyczne: pojedyncze pasmo). Przetwarzanie zawsze rozpoczyna się od danych surowych. | Tak jak zarejestrowano |
| `debayered` | Liniowe demosaikowanie — 3-kanałowe dla M3C, 1-kanałowe w skali szarości dla M3M: usunięto ciemne przesunięcie z pakietu kalibracyjnego, a następnie zastosowano demosaikowanie uwzględniające krawędzie, bez korekcji liniowości, płaskiego pola ani rozdzielania kolorów. Eksport `process` zapisuje ten sam plik, jaki zapisuje się na tym poziomie podczas rejestracji, w każdym trybie przetwarzania. | Liniowy DN |
| `radiance` | Bezwzględna promieniowanie spektralne z pełnego łańcucha radiometrycznego, w **W/m²/sr/nm**. Zawsze zapisywana jako 32-bitowy plik TIFF (`tiff32/Radiance_Images/`), niezależnie od wybranego formatu eksportu. | float32 |
| `reflectance` | Współczynnik odbicia ρ, gdzie **DN 32768 = ρ 1,0 (100%)** z rezerwą do ρ 2,0. Gotowe do użycia w Pix4D. | uint16 |
| `preview` | Renderowanie gotowe do wyświetlenia: RGB = balans bieli + gamma; wielospektralne = rozciągnięcie w fałszywych kolorach. | 8-bitowy format wyświetlania |

### Orientacja

Pliki `raw` i `debayered` zawsze zachowują **natywną** orientację czujnika; ustawienia odwrócenia/obrotu aparatu są stosowane wyłącznie do pozostałych produktów (podgląd, promieniowanie, odbicie, indeks). W przypadku serii zdjęć pliki wszystkich elementów zachowują orientację natywną, chyba że użytkownik zleci zmianę orientacji (`--orient` w interfejsie CLI, `orient=True` w SDK). Każdy plik rejestruje orientację odpowiadającą jego pikselom w formacie XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — a `ChlorosOrientationApplied` określa, czy orientacja ta jest zawarta w samych pikselach (`false` w plikach surowych i po usunięciu efektu Bayera). Przeglądarka obrazów Chloros przetwarza pliki surowe i po usunięciu efektu bayera zgodnie z tym rekordem wyłącznie na potrzeby wyświetlania; same pliki nigdy nie są przepisywane. Plik bez tego rekordu (utworzony przed wersją 1.3.0 lub za pomocą innego narzędzia) jest wyświetlany w stanie, w jakim został zapisany.

## Maski ograniczające

Wyniki pomiarów promieniowania i odbicia z wielospektralnej kamery M3C są poddawane rozdzielaniu pasmowemu 3×3. Tam, gdzie rozdzielanie pasmowe daje wartość ujemną w danym paśmie, wartość ta jest ograniczana do zera (podgląd na żywo działa dokładnie w ten sam sposób). Nowością w wersji 1.3.0 jest to, że produkt, w którym co najmniej jeden piksel został obcięty, otrzymuje towarzyszącą mu **maskę ograniczenia**, która dokładnie wskazuje, które piksele i pasma zostały obcięte:

* **Gdzie:**w folderze `clamp_masks/`**obok** folderów produktów, nigdy w ich wnętrzu: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` oraz `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Co:** jednopasmowy plik TIFF `uint8` na własnej siatce pikselowej produktu (wraz z orientacją, obszarem zainteresowania (ROI), binningiem i wyrównaniem tablicy). Bit 0 (`mask & 1`) jest ustawiony tam, gdzie pasmo w kanale R produktu zostało przycięte, bit 1 (`& 2`) – w kanale G, a bit 2 (`& 4`) – w kanale B. Udział pikseli z obciętymi wartościami dla poszczególnych pasm jest również oznaczony w produkcie jako `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` i `Chloros:BandBUnmixClampFraction`.
* **Jak jest to zaznaczone w produkcie:** tag XMP `Chloros:UnmixClampMask` zawiera ścieżkę do maski względem folderu produktu (`../clamp_masks/<product folder>/<name>_clampmask.tif`, zawsze oddzielone znakiem `/`), `none`, gdy żaden piksel nie został przycięty (maska nie została zapisana), lub `unavailable`, gdy nie można było zapisać maski. Produkt bez tagu nigdy nie przeszedł procesu rozdzielania kolorów (kamery RGB i monochromatyczne, poziomy surowe i po usunięciu efektu bayera).

Ponieważ żaden folder produktu nie zawiera maski, można przekazać folder produktu, taki jak `Reflectance_Calibrated_Images/`, do Pix4D, Metashape lub innego oprogramowania fotogrametrycznego w obecnej postaci, wraz z podfolderami — należy przekazać folder produktu, a nie folder formatu znajdujący się nad nim. Własna funkcja importu programu Chloros pomija maski ograniczające. Interfejs CLI zlicza je oddzielnie: `Image products written: N (plus M clamp masks)`.

## Produkty indeksowe

Każdy indeks trafia do własnego folderu `<INDEX>_Index_Images/` (np. `NDVI_Index_Images/`).

* **Formaty całkowitoliczbowe**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) przechowują**liniowe rozciągnięcie dla poszczególnych obrazów**, a nie odwzorowanie w zakresie od −1 do 1: DN 0 to najniższa wartość indeksu tej klatki, a pełna skala – najwyższa. Plik zawiera `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` i `Chloros:IndexMax`, a więc również `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Ponieważ każda klatka ma własne punkty końcowe, należy porównywać klatki na podstawie wartości, a nie na podstawie DN.
* **`TIFF (32-bit, Percent)`** przechowuje same wartości indeksowe jako float32, bez przeskalowania — jest to najlepszy wybór do analiz ilościowych w ramach całego lotu.

Nowość w wersji 1.3.0: produkt indeksowy opisuje **sam siebie**, zamiast powtarzać tagi surowych danych źródłowych. Jego tagi GPS i kamery są kopiowane z produktu, na podstawie którego został obliczony (produktu odbicia, jeśli taki istnieje), a następnie:

| Tag | Wartość |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (dzięki czemu plik nigdy nie jest ponownie importowany jako dane z rejestracji) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / zastosowany wzór |
| `Chloros:IndexLevel` | `reflectance` lub `radiance` — na podstawie czego obliczono wskaźnik (brak, jeśli nie ma żadnego z nich) |
| `Chloros:IlluminationInvariant` | `True` dla wskaźnika obliczonego na podstawie współczynnika odbicia, `False` na podstawie promieniowania lub wartości z czujnika |
| `Chloros:ChlorosPipelineVersion` | wersja programu Chloros, która utworzyła indeks |

`Chloros:PixelScale` został usunięty z produktów indeksowych — należy użyć powyższych wartości `IndexMin` / `IndexMax`. Pełna instrukcja odczytu znajduje się w sekcji „Raster indeksu całkowitoliczbowego to rozciągnięcie kontrastu dla poszczególnych obrazów” w [Podręczniku CLI](reference/cli-reference.md).

## Odczytywanie wartości odbicia pikseli

Odbicie jest przechowywane jako liczbowa wartość całkowita, a **wartość DN odpowiadająca ρ = 1,0 (100% odbicia) zależy od kamery źródłowej**:

| Kamera źródłowa | ρ = 1,0 to DN | Jak to ustalić |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (zakres do ρ 2,0) | W pliku umieszczono tag XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (ograniczone do ρ 1,0) | Brak tagu `Chloros:PixelScale` — ten brak jest sygnałem. (Produkty Survey3 zawierają inne tagi przetwarzania `Chloros:*`, takie jak `Chloros:CalibrationApplied`, we wszystkich trybach przetwarzania od wersji 1.3.0.) |

**Odczytaj tag XMP `Chloros:PixelScale` i podziel przez jego wartość**, zamiast zakładać stałą wartość. Tag ten jest zdefiniowany w domenie uint16, więc pozostaje na poziomie `32768` we wszystkich formatach wyjściowych, które dokonują przeskalowania — najpierw znormalizuj zapisany typ danych z powrotem do uint16 (×257 z 8-bitowego, ×65535 z float32).

{% hint style="warning" %}
**Pomiary 8-bitowe.** Dane eksportowane z pomiaru 8-bitowego są 8-bitowe. Ich współczynnik odbicia jest przeskalowywany i oznaczany jako `Chloros:PixelScale=32768`, dzięki czemu skala pozostaje zachowana; 8-bitowe produkty DN oraz pliki odbicia zapisane przed wersją 1.3.0 są przycięte do zakresu 0–255 i nie zawierają znacznika skali. Jeśli w pliku odbicia LATTICE brakuje tego znacznika, nie należy zakładać istnienia skali; zamiast tego należy ponownie wyeksportować dane w formacie 16-bitowym lub 32-bitowym.
{% endhint %}

Pełne zasady (w tym znaczniki zgodne z MicaSense) można znaleźć w sekcji **„Odczytywanie pikseli odbicia”** w [Podręczniku CLI](reference/cli-reference.md).
