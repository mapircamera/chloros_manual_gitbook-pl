# Kamery LATTICE

LATTICE to modułowy system kamer wielospektralnych firmMAPIR przeznaczony do obrazowania w rolnictwie i naukach ścisłych. Każda kamera LATTICE jest oparta na czujniku Sony IMX265 z globalną migawką (**3,1 MP, piksele 3,45 µm**) i łączy się przez sieć Ethernet jako urządzenie**GigE Vision**.

Chloros 1.3.0 steruje kamerami LATTICE w trybie na żywo — wykrywanie, podgląd na żywo, rejestracja oraz zsynchronizowane układy wielokamerowe — z trzech interfejsów:

| Interfejs    | Gdzie                                                          | Platformy                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Zakładka **Kamery** na pasku bocznym Chloros                         | Windows 10/11 x64                                        |
| CLI        | Rodzina poleceń `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Szukasz sprzętu?**Moduły kamer, obiektywy, filtry i pasma, ramki i elementy mocujące, kable, okablowanie PoE i wyzwalające zostały opisane w [**instrukcji obsługi LATTICE**](https://mapir.gitbook.io/lattice-camera). Niniejszy rozdział dotyczy sterowania kamerami za pomocą Chloros.

Pliki przechwycone przez LATTICE to standardowe pliki typu `.tif`/`.tiff`, a Chloros zawsze przetwarza je, zaczynając od surowych danych przechwyconych. Pełny zestaw poleceń i interfejs API można znaleźć w [Podręczniku CLI](../reference/cli-reference.md) oraz [Podręczniku SDK](../reference/sdk-reference.md).

## Dwie konfiguracje czujników

| Konfiguracja | Czujnik       | Filtr                                | Dane dostarczane przez jedną kamerę                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Kolorowy Bayer | potrójny filtr pasmowy                |**Trzy skalibrowane pasma z jednej ekspozycji**                 |
| **M3M**| Monochromatyczny   | pojedynczy wąskopasmowy filtr interferencyjny |**Jedno skalibrowane pasmo**; połączenie wielu kamer M3M w celu uzyskania wskaźników |

Ponieważ kamera M3M jest monochromatyczna i wyposażona w pojedynczy filtr, każde pasmo otrzymuje własną ekspozycję. Kamera M3C obejmuje wszystkie trzy pasma za pomocą jednej ekspozycji czujnika.

## Ciągi znaków modeli i nazewnictwo

Każda kamera przechowuje swoją tożsamość w GenICam `DeviceUserID` jako ciąg znaków modelu:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Program Chloros wyświetla go z prefiksem `LATT-` (na przykład `LATT-M3M-L87-F450`). Ten sam ciąg znaków `LATT-…` jest zapisywany w tagu EXIF `Model` każdego eksportowanego pliku i służy jako nazwa folderu wyjściowego aparatu w przetworzonych projektach.

| Element | Wartości                                                   | Znaczenie                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Matryca    | `M3C` / `M3M`                                            | Kolorowa matryca Bayera / monochromatyczna                                                                          |
| Obiektyw | `L41` / `L87`                                            | Liczba oznacza **poziome pole widzenia w stopniach**: L41 = wąskie (41°), L87 = szerokie (87°)    |
| Filtr    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) lub `F<nm>` (M3M) | Zobacz [Filtry i pasma spektralne](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Ciąg znaków modelu steruje wszystkimi dalszymi procesami: Chlorosa ustala profil czujnika, układ pasm oraz kalibrację fabryczną na podstawie `DeviceUserID` + `DeviceSerialNumber`. Nie ma potrzeby konfigurowania poszczególnych kamer — zobacz [Podłączanie kamer](connecting.md).

## Filtry i pasma

Środki pasm, szerokości FWHM oraz pełny katalog 23 pozycji M3M stanowią specyfikacje produktu, dlatego znajdują się w instrukcji sprzętu: [**Filtry i pasma spektralne**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Co ma znaczenie po stronie oprogramowania: kod filtra w ciągu znaków modelu określa, jakie produkty może generować Chloros. Kamery z filtrem RGB (`FRGB`) generują wyłącznie produkty po usunięciu efektu bayera i podglądowe — promieniowanie i odbicie w poszczególnych pasmach nie mają znaczenia w przypadku czujnika szerokopasmowego, więc Chloros je pomija i informuje o tym. Każdy inny filtr zapewnia pełny łańcuch: promieniowanie → odbicie → indeks.

## Kalibracja radiometryczna w skrócie

Każda kamera LATTICE jest indywidualnie kalibrowana w fabryce w oparciu o łańcuch kalibracyjny zgodny z normami NIST i dostarczana wraz z certyfikatem dla danego egzemplarza. Zakres kalibracji, sposób pomiaru oraz dokładność, którą można podać, opisano w instrukcji obsługi sprzętu: [**Fabryczna kalibracja radiometryczna**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Jeśli chodzi o oprogramowanie, istotne jest to, że Chloros ustala właściwą kalibrację po podłączeniu kamery i utrwala zastosowane współczynniki w każdym eksporcie — zobacz [Podłączanie kamer](connecting.md).

## W tym rozdziale

* [Podłączanie kamer](connecting.md) — automatyczne wykrywanie, okno dialogowe podłączania w interfejsie graficznym, odpowiedniki w CLI i SDK oraz sposób ustalania kalibracji fabrycznej (pakiet w kamerze vs. chmura) po podłączeniu kamery.
* [Ustawienia kamer](camera-settings.md) — wszystkie elementy sterujące w zakładce „Kamery”: lista kamer, panel ustawień poszczególnych kamer oraz panel ustawień macierzy (w tym edytor obszaru zainteresowania (ROI) i wyrównanie).
* [Ustawienia i tryby przechwytywania](capture.md) — opcja „Przechwyć wszystko”, przechwytywanie pojedyncze / ciągłe / interwałowe, typy eksportu oraz struktura folderów z przechwyconymi danymi.
* [Układ wielokamerowy](arrays.md) — kreator podłączania układu, sonda kabla synchronizacyjnego, przepustowość, PTP oraz podgląd układu na żywo.
* [Kamery monochromatyczne i wskaźniki wegetacji](mono-indices.md) — kamery M3M oraz proces wyrównywania → nakładania → tworzenia wskaźników.

Pełny zestaw poleceń znajduje się w [Podręczniku CLI](../reference/cli-reference.md) oraz [Podręczniku SDK](../reference/sdk-reference.md).
