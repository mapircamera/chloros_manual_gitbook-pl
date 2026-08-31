---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formaty obrazów wyjściowych Program

Chloroseksportuje przetworzone produkty w czterech formatach plików. Format należy wybrać w ustawieniach projektu (interfejs graficzny), za pomocą `--format` (CLI) lub `export_format` (SDK). Pliki CLI i SDK akceptują dokładnie poniższe ciągi znaków.

| Ciąg formatujący | Rozszerzenie | Typ piksela | Zakres pikseli | Uwagi |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(domyślnie)* | `.tif` | liczba cyfrowa typu uint16 | 0 – 65535 | Zalecane do fotogrametrii / GIS. |
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

Folder kamery to `LATT-<sensor>-<lens>-F<filter>` dla LATTICE oraz `<model>_<filter>` (np. `Survey3N_RGN`) dla Survey3. **Każdy wyeksportowany produkt zachowuje nazwę pliku źródłowego — to folder identyfikuje produkt, a nie rozszerzenie nazwy pliku.** Pełne zasady znajdziesz w sekcji [Gdzie trafiają pliki wyjściowe](reference/cli-reference.md) w podręczniku „CLI Reference”.

## Produkty LATTICE (poziomy przechwytywania i eksportu)

Jedna surowa klatka LATTICE jest rozdzielana na wszystkie żądane produkty w jednym przebiegu. Każdy typ produktu ma własny przełącznik (pola wyboru w interfejsie graficznym lub CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, domyślnie wszystkie włączone):

| Poziom | Zawartość | Typ danych |
| --- | --- | --- |
| `raw` | Dane Bayer prosto z czujnika (kamery monochromatyczne: pojedyncze pasmo). Przetwarzanie zawsze rozpoczyna się od danych surowych. | Tak jak zarejestrowano |
| `debayered` | Liniowe demosaikowanie — 3-kanałowe dla M3C, 1-kanałowe w skali szarości dla M3M. | Liniowa wartość DN |
| `radiance` | Bezwzględna promienność spektralna z pełnego łańcucha radiometrycznego, w **W/m²/sr/nm**. Zawsze zapisywana jako 32-bitowa liczba zmiennoprzecinkowa typu „TIFF” (`tiff32/Radiance_Images/`), niezależnie od wybranego formatu eksportu. | float32 |
| `reflectance` | Współczynnik odbicia ρ, gdzie **DN 32768 = ρ 1,0 (100%)** z rezerwą do ρ 2,0. Gotowe do użycia w Pix4D. | uint16 |
| `preview` | Renderowanie gotowe do wyświetlenia: RGB = balans bieli + gamma; multispektralne = rozciągnięcie w fałszywych kolorach. | 8-bitowy wyświetlacz |

## Odczytywanie wartości pikseli odbicia

Odbicie jest przechowywane jako całkowita liczba cyfrowa, a **wartość DN odpowiadająca ρ = 1,0 (100% odbicia) zależy od kamery źródłowej**:

| Kamera źródłowa | ρ = 1,0 to DN | Jak to rozpoznać |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (zakres do ρ 2,0) | W pliku umieszczono tag XMP o wartości `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (ograniczone do ρ 1,0) | Brak tagów XMP `Chloros:*` — ten brak jest sygnałem. |

**Odczytaj tag XMP `Chloros:PixelScale` i podziel przez jego wartość**, zamiast zakładać stałą wartość. Tag jest zdefiniowany w domenie uint16, więc pozostaje niezmieniony `32768` we wszystkich formatach wyjściowych, które zmieniają skalę — najpierw znormalizuj zapisany typ danych z powrotem do uint16 (×257 z 8-bitowego, ×65535 z float32).

{% hint style="warning" %}
**Jeden przypadek, zgodnie z założeniami projektowymi, nie podlega skalowaniu.** Gdy dane źródłowe 8-bitowe (BayerRG8) są zapisywane jako 8-bitowe TIFF, potok danych ogranicza zakres do 0–255 zamiast dokonywać przeskalowania, więc plik nie posiada skali — Chloros celowo pomija w tym przypadku `Chloros:PixelScale`. Jeśli tag ten nie występuje w pliku odbicia LATTICE, nie należy zakładać istnienia skali; zamiast tego należy ponownie wyeksportować plik w formacie 16-bitowym lub 32-bitowym.
{% endhint %}

Pełne zasady (w tym tagi zgodne ze standardem MicaSense) można znaleźć w sekcji **„Odczytywanie pikseli odbicia”** w [Podręczniku CLI](reference/cli-reference.md).
