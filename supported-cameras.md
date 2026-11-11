---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Obsługiwane kamery —

Chloros
przetwarza obrazy z dwóch rodzin kamer MAPIR
na **wszystkich platformach** (Windows
, Linux
amd64 oraz Linux
arm64/Jetson):

* **Survey3** — kamery Survey3W
(szerokokątne) i Survey3N
(wąskokątne). Dane wejściowe: `RAW+JPG`.
* **LATTICE**— moduły kamer wielospektralnych M3C i M3M. Dane wejściowe: zdjęcia z `.tif`/`.tiff`. Kamerami LATTICE można również**sterować na żywo** z poziomu Chloros
— poprzez zakładkę „Kamer” w interfejsie graficznym (Windows
) lub `chloros-cli lattice` / Python
SDK
(Windows
i Linux
) — w tym zsynchronizowane układy wielokamerowe. Zobacz [przewodnik LATTICE](lattice/).

Potok przetwarzania akceptuje również pliki wejściowe `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Producent</th><th width="250">Model kamery</th><th width="138">Model filtra</th><th width="187">Typ obrazu</th></tr></thead><tbody><tr><td><strong>Producent</strong></td><td><strong>Model aparatu</strong></td><td><strong>Model filtra</strong></td><td><strong>Typ obrazu</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Seria LATTICE to modułowy system kamer wielospektralnych oparty na czujniku Sony IMX265 z globalną migawką (3,1 MP, piksele 3,45 µm). Każda kamera przechowuje swój identyfikator w postaci ciągu znaków modelu:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```. Program

Chloros

wyświetla ją z prefiksem `LATT-` (na przykład `LATT-M3M-L41-F550`), a ten ciąg znaków steruje wszystkimi dalszymi procesami — profil czujnika, układ pasm i kalibracja są ustalane automatycznie; nie ma potrzeby konfigurowania poszczególnych kamer. Numer obiektywu oznacza **poziome pole widzenia w stopniach**: `L41` = wąskie 41°, `L87` = szerokie 87°.

Istnieją dwie konfiguracje czujników:

| Konfiguracja | Czujnik      | Typ filtra                           | Pasma na kamerę                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Kolorowy filtr Bayera | Potrójny filtr pasmowy                       | 3 pasma spektralne z jednej ekspozycji                                 |
| **M3M**       | Monochromatyczny  | Pojedynczy wąskopasmowy filtr interferencyjny | 1 skalibrowane pasmo — połączenie wielu kamer M3M w celu uzyskania wskaźników wegetacyjnych |

### Opcje filtrów M3C (Bayer)

| Filtr | Pasma (nazwa @ środek nm / FWHM

nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue

475/30 · Green

550/30 · Red

625/30  |
| `FRGN` | Red

660/21 · Green

550/30 · NIR

850/30   |
| `FOCN` | Orange

618,4/46 · Cyan

494,1/40 · NIR

821,6/54 (zmierzone — patrz poniżej) |
| `FNGB` | Blue

475/30 · Green

550/30 · NIR

850/30  |

W przypadku `FOCN` wartości te odpowiadają **zmierzonemu** pasmu półmaksymalnemu filtra (program Chloros

1.3.0 i nowsze; wcześniejsze wersje wykorzystywały wartości z arkusza specyfikacji: 615/21 · 490/38 · 808/14). Nazwy pasm pozostają bez zmian: `Orange_615`, `Cyan_490` i `NIR_808`, a zmierzone położenie środka i szerokość pasma są danymi, które program Chloros

umieszcza w każdym skalibrowanym pliku eksportowym.

### Katalog filtrów M3M (mono) — 23 pozycje katalogowe

Liczba F stanowi oznaczenie pozycji katalogowej; zmierzone pasmo (wytłoczone na każdym skalibrowanym egzemplarzu) to skan filtra dla danej partii:

| Pozycja katalogowa | Środek (nm, zmierzony) | Krawędzie FWHM

(nm) | Szerokość (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominalna)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominalna)           | 867–893         | 26         |
| F905   | — (nominalna)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„Krawędzie pasm są mierzone jako wartości pełnej szerokości w połowie maksymalnej intensywności (FWHM) na podstawie skanów filtrów MAPIR

dla poszczególnych partii — te same wartości Chloros

umieszcza w każdym skalibrowanym eksporcie.”_ „— (nominalna)” = brak jeszcze skanów z partii; w przypadku tych jednostek SKU podanym środkiem jest numer SKU, a szerokością — wartość podana przez producenta.

† „Odbicie F988 jest kalibrowane przy użyciu panelu odbicia umieszczonego w scenie: pasmo wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros

stosuje najnowszy zapis z panelu i utrzymuje go między kolejnymi pomiarami panelu”. Zobacz [Cele kalibracyjne](calibration-targets.md).

Informacje na temat sterowania kamerą na żywo, matryc, konfiguracji sieci oraz łańcucha przetwarzania radiometrycznego można znaleźć w [przewodniku LATTICE](lattice/).
