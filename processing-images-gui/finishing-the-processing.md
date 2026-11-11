# Zakończenie przetwarzania

Gdy program Chloros
zakończy przetwarzanie, nadchodzi czas na przejrzenie wyników, sprawdzenie jakości wyjściowej oraz przygotowanie przetworzonych obrazów do wykorzystania w procesie pracy. Ta strona zawiera instrukcje dotyczące ostatnich kroków i dalszych działań.

## Wskazania zakończenia przetwarzania

Po pomyślnym zakończeniu przetwarzania pojawi się kilka wskaźników:

* ✅ **Pasek postępu**: Osiąga 100% realizacji
* ✅ **Dziennik debugowania**: Wyświetla końcowe wiersze `[RUN-SUMMARY]` wraz z liczbami (obrazy, grupy kamer, cele, skalibrowane obrazy, zapisane pliki)
* ✅ **Przycisk Start**: Zostaje ponownie udostępniony (gotowy do kolejnego przebiegu przetwarzania)
* ✅ **Pliki wyjściowe**: Wszystkie przetworzone obrazy zapisane w drzewie wyjściowym projektu (poniżej)

{% hint style="warning" %}
**Cykl, w którym nie zapisano żadnych obrazów, kończy się niepowodzeniem.** Jeśli zażądano produktów obrazowych, a przebieg nie zapisał żadnego, Chloros
zgłasza to jako niepowodzenie — nazwa logu `[RUN-SUMMARY]` wskazuje prawdopodobną przyczynę (nie zaimportowano niczego, nie wykryto żadnego celu lub pominięto wszystkie żądane produkty jako nieodpowiednie). Odpowiednik w CLI
kończy działanie z wynikiem niezerowym. Celowe uruchomienie obejmujące wyłącznie metadane (wszystkie produkty eksportu wyłączone, brak indeksów) nadal jest traktowane jako sukces. Zobacz [Podręcznik CLI
](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Lokalizowanie przetworzonych obrazów

### Otwieranie folderu wyjściowego

1. Kliknij ikonę **Menu głównego** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(w lewym górnym rogu)
2. Wybierz **„Otwórz folder projektu”**

3. Otworzy się eksplorator plików w katalogu projektu
4. Znajdź projekt według nazwy

### Struktura katalogów wyjściowych

Pliki są zapisywane **w folderze projektu, pogrupowane według aparatu, a następnie według formatu pliku**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Folder aparatu**: `LATT-<sensor>-<lens>-F<filter>` dla LATTICE (zgodne z danymi EXIF ujęcia `Model`), `<model>_<filter>` dla Survey3
(np. `Survey3N_RGN`). Dwie kamery, które mają ten sam czujnik i filtr, ale różnią się obiektywem, mają oddzielne drzewa — różnią się one winietowaniem, polem widzenia i zniekształceniami.
* **Format folderu**: zgodny z ustawieniem formatu eksportu — `tiff16`, `tiff8`, `png8`, `jpg8`, lub `tiff32` dla formatu TIFF
(32-bitowy, procentowy). Radiance ma zawsze typ float32 i zawsze znajduje się w pliku `tiff32`.
* **Foldery produktów**:
  * `Reflectance_Calibrated_Images/` — skalibrowana odbiciowość
  * `Debayered_Images/` — dane po liniowej demosaikacji (LATTICE): usunięto ciemne przesunięcie i przeprowadzono demosaikację, bez pola płaskiego ani demiksowania, w natywnej orientacji czujnika
  * `Preview_Images/` — podgląd na ekranie (LATTICE)
  * `Radiance_Images/` — promieniowanie spektralne typu float32, W/m²/sr/nm (LATTICE wielospektralne)
  * `Vignette_Corrected_Images/` **lub** `Sensor_Response_Images/` — nieskalibrowany plik rezerwowy dla klatek bez odniesienia odbicia; na każdy przebieg istnieje dokładnie jeden z tych dwóch plików, wybrany na podstawie ustawienia korekcji winietowania
  * `<INDEX>_Index_Images/` — jeden folder na każdy wybrany indeks (np. `NDVI_Index_Images`)
* **`clamp_masks/`** (nowość w wersji 1.3.0, wyłącznie dla wielospektralnych kamer LATTICE M3C): pojawia się w folderze formatów, obok folderów produktów, gdy proces rozdzielania pasm przycina ujemne wartości do zera w produkcie promieniowania lub odbicia. Zawiera jedną maskę na piksel dla każdego produktu, którego to dotyczy, np. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Maski nigdy nie znajdują się wewnątrz folderu produktu, więc folder produktu można przekazać do oprogramowania fotogrametrycznego w niezmienionej postaci. Zobacz [Maska Clamp](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Każdy wyeksportowany produkt zachowuje nazwę pliku ŹRÓDŁOWEGO.**Eksport promieniowania o nazwie `capture_..._raw.tif` nadal nazywa się `capture_..._raw.tif` — po prostu znajduje się w folderze `tiff32/Radiance_Images/`.**Produkt identyfikuje folder, a nie nazwa pliku**, więc wyszukiwanie pliku `*radiance*.tif` nie da żadnego wyniku; zamiast tego należy wyszukiwać według katalogu.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Ile plików powinno być?

Nie licz według wzoru — liczba plików wyjściowych zależy od tego, które produkty zostały włączone i które mają zastosowanie do poszczególnych kamer (np. kamery RGB
nie otrzymują danych dotyczących promieniowania/odbicia). Ostateczna liczba znajduje się w dzienniku: ostatni wiersz `[RUN-SUMMARY]` podaje dokładnie, ile plików zostało zapisanych, a wiersze z podpowiedziami wyjaśniają, co zostało pominięte.

***

## Przeglądanie przetworzonych obrazów

### Szybki podgląd w Eksploratorze plików

**Wbudowany podgląd systemu Windows
:**

1. Przejdź do folderu produktu (np. `tiff16/Reflectance_Calibrated_Images/`)
2. Wybierz plik obrazu
3. Podgląd pojawi się w panelu podglądu Eksploratora Windows

4. Używaj klawiszy strzałek, aby przeglądać obrazy

### Podgląd w zewnętrznych przeglądarkach obrazów

**Zalecane przeglądarki:*** **QGIS** – darmowe oprogramowanie GIS (najlepsze do analizy wielospektralnej z georeferencjami)
* **IrfanView** – szybka, lekka przeglądarka obrazów (obsługuje format TIFF
)
* **Adobe Photoshop** – profesjonalna edycja (obsługa formatu TIFF
)
* **GIMP** – bezpłatna alternatywa dla programu Photoshop
* **Zdjęcia w systemie Windows** – podstawowe przeglądanie (może nie obsługiwać 16-bitowego formatu TIFF
)

### Podgląd w przeglądarce obrazów Chloros


Użyj wbudowanej przeglądarki obrazów Chloros
do zaawansowanej wizualizacji:

1. Kliknij miniaturę obrazu w przeglądarce plików
2. Obraz otworzy się w głównym obszarze podglądu
3. Kliknij zakładkę **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
na lewym pasku bocznym
4. Użyj [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) do interaktywnej analizy

Szczegółowe instrukcje znajdziesz w sekcji [Przeglądarka obrazów](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Odczytywanie wartości pikseli odbicia (GIS / Pix4D / Skrypty)

Odbicie jest przechowywane jako liczba całkowita DN, a **wartość DN odpowiadająca ρ = 1,0 zależy od kamery źródłowej**:

| Źródło          | ρ = 1,0 wynosi | Jak to sprawdzić                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (zakres do ρ 2,0) | W pliku umieszczono tag XMP `Chloros:PixelScale=32768` |
| Survey3
         | **65535** (ograniczone do ρ 1,0)     | Brak tagu `Chloros:PixelScale` — ten brak jest sygnałem (produkty Survey3
zawierają inne tagi przetwarzania typu `Chloros:*`) |

**Odczytaj tag `Chloros:PixelScale` i podziel przez tę wartość** zamiast zakładać uniwersalną wartość 65535 — podzielenie odbicia LATTICE przez 65535 w sposób niewidoczny zmniejsza każdą wartość o połowę. Plik współczynników odbicia sprzed wersji 1.3.0 zapisany w formacie 8-bitowym został przycięty, a nie przeskalowany, i nie zawiera tagu skali: należy go ponownie wyeksportować w formacie 16-bitowym lub 32-bitowym zamiast go dzielić. Pełne wyjaśnienie znajduje się w sekcji [Formaty obrazów wyjściowych](../output-image-formats.md).***

## Metadane przenoszone do eksportowanych plików

Każdy produkt zachowuje **blok GPS**z oryginalnego zdjęcia oraz jego**podsekcję EXIF**, więc
eksport zawiera pliki `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` i
`CameraSerialNumber`, a także georeferencje.

{% hint style="warning" %}
**Jeśli ortomosaika ma absurdalną skalę, najpierw sprawdź `FocalLength`.**
Pix4D oblicza odległość próbkowania naziemnego na podstawie ogniskowej i wysokości. Bez tego tagu
program przyjmuje całkowicie błędną skalę — w przypadku jednego lotu z 49 ujęciami gaj pomarańczowy o wymiarach 411 m × 160 m
został zrekonstruowany jako 47,8 km × 13 km, co dało ortofotomapę o rozdzielczości 455 megapikseli, przedstawiającą głównie
pustą przestrzeń. Powolne dzielenie na kafelki i nieoczekiwanie ogromny rozmiar pliku są objawami tego problemu, a nie odrębnymi
problemami.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nie *każdy* tag jest kopiowany. Tagi strukturalne IFD0 są celowo pomijane (ich kopiowanie
powoduje uszkodzenie danych wyjściowych LATTICE), a pliki `ExifImageWidth` / `ExifImageHeight` są wykluczone,
ponieważ opisują one oryginalny zrzut — eksport o zmienionych wymiarach w przeciwnym razie
podawałby wymiary sprzeczne z jego własną siatką rastrową.

Chloros
zapisuje również własne tagi XMP typu `Chloros:*` opisujące, jakie operacje zostały wykonane na każdym produkcie
(na przykład `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Od wersji 1.3.0 produkty programu Survey3
zawierają je w każdym trybie przetwarzania
trybie przetwarzania, produkty indeksowe opisują sam indeks (`Chloros:ProcessingLevel=index`,
wzór oraz to, czy został obliczony na podstawie odbicia czy promieniowania), zamiast powtarzać
tagi surowych danych źródłowych, a produkty w folderach o nazwach dłuższych niż 260 znaków również zachowują swoje metadane
. Zobacz [Formaty obrazów wyjściowych](../output-image-formats.md#index-products).

***

## Przeglądanie dziennika debugowania

### Sprawdzanie ostrzeżeń i błędów

1. Otwórz kartę **Dziennik debugowania** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Przewiń komunikaty
3. Poszukaj żółtych ostrzeżeń lub czerwonych błędów
4. Przeczytaj wiersze `[RUN-SUMMARY]` oraz wszelkie wskazówki
5. Skontaktuj się z pomocą techniczną MAPIR
, aby uzyskać pomoc

### Zapisywanie dziennika

Aby zachować zapis przebiegu przetwarzania lub przesłać go do działu pomocy technicznej MAPIR
:

1. Kliknij przycisk **„Kopiuj”**lub**„Pobierz”**

2. Zapisz jako plik tekstowy w folderze projektu
3. Dołącz do dokumentacji projektu
4. W razie napotkanych problemów wyślij go do pomocy technicznej MAPIR


Aby zgłosić problem bezpośrednio do MAPIR
, użyj ikony błędu (**Zgłoś błąd**) u góry zakładki „Dziennik debugowania” — zobacz [Zgłaszanie błędu z zakładki dziennika](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).***

## Typowe problemy z wynikami i ich rozwiązania

### Problem: Brakujące pliki wynikowe

**Możliwe przyczyny:**

* Produkt nie ma zastosowania do tej kamery (np. promieniowanie/odbicie dla kamer RGB
— informacja ta znajduje się w dzienniku)
* Brakowało wymaganego odniesienia (np. współczynnik odbicia bez celu i bez promieniowania padającego `.daq`)
* Pole wyboru eksportu produktu było wyłączone w ustawieniach projektu
* Podczas eksportu zabrakło miejsca na dysku

**Rozwiązania:**

1. Sprawdź wskazówki `[RUN-SUMMARY]` i wiersze `[EXPORT-CHECK]` w dzienniku debugowania — wyjaśniają one pominięcia dla poszczególnych kamer
2. Sprawdź pola wyboru eksportu produktu w [Ustawieniach projektu](adjusting-project-settings.md)
3. Sprawdź, czy było wystarczająco dużo miejsca na dysku
4. Po usunięciu przyczyny przeprowadź przetwarzanie ponownie

### Problem: Ciemne lub jasne krawędzie (wciąż widoczne winietowanie)

**Możliwe przyczyny:**

* Wyłączona korekcja winietowania
* Kamera/obiektyw nie figurują w bazie danych profili Chloros

* Ekstremalne winietowanie wykraczające poza możliwości korekcji

**Rozwiązania:**

1. Sprawdź, czy korekcja winietowania została włączona w ustawieniach projektu
2. Sprawdź, czy model kamery został poprawnie wykryty
3. Skontaktuj się z pomocą techniczną MAPIR
, jeśli winietowanie nadal występuje

### Problem: Nieprawidłowe kolory lub wartości

**Możliwe przyczyny:**

* Nie wykryto celów kalibracyjnych
* Wybrano niewłaściwy model celu kalibracyjnego
* Wyłączona kalibracja odbicia
* Niska jakość obrazów celów

**Rozwiązania:**

1. Sprawdź, czy kalibracja odbicia została włączona
2. Sprawdź komunikaty „Target found” w dzienniku debugowania
3. Sprawdź jakość obrazów celów
4. Przeprowadź ponowne przetwarzanie z zaznaczonymi właściwymi celami

### Problem: Wartości NDVI
wydają się nieprawidłowe

**Oczekiwane zakresy wartości NDVI
:*** **Woda, skały, gleba**: od -0,1 do 0,2
* **Rzadka/niezdrowa roślinność**: od 0,2 do 0,4
* **Umiarkowana roślinność**: od 0,4 do 0,6
* **Zdrowa, gęsta roślinność**: od 0,6 do 0,9**Jeśli wartości wykraczają poza te zakresy:**

1. Sprawdź, czy zastosowano kalibrację odbicia
2. Sprawdź, czy dołączono dziennik czujnika światła
3. Sprawdź, czy wykryto cele kalibracyjne
4. Upewnij się, że wykryto właściwy model kamery
5. Sprawdź czas i warunki wykonania zdjęć celów
6. Jeśli samodzielnie obliczasz wskaźniki na podstawie plików odbicia, upewnij się, że podzieliłeś je przez wartość `Chloros:PixelScale` zawartą w pliku (patrz wyżej)

***

## Korzystanie z przetworzonych zdjęć

### Do fotogrametrii / tworzenia ortomosaiki

**Zalecany przebieg pracy:**

1.**Zaimportuj skalibrowane obrazy odbicia** do oprogramowania fotogrametrycznego:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachowaj metadane EXIF**: Upewnij się, że dane GPS zostały zachowane w celu geotagowania
3. **Kalibrowane procedury**: Używaj obrazów odbicia światła w celu uzyskania naukowej dokładności — odbicie światła LATTICE zawiera tagi kalibracyjne XMP odczytywane przez Pix4D
4. **Przetwarzaj mozaiki indeksowe**: Twórz ortomosaiki NDVI
z poszczególnych obrazów indeksowych
5. **Eksportuj pliki GeoTIFF
z georeferencjami**: Do wykorzystania w aplikacjach GIS

### Do analizy GIS

**Zalecany przebieg pracy:**

1.**Załaduj do QGIS, ArcGIS lub podobnego programu**

2.**Wykorzystaj 16-bitowe obrazy odbicia w formacie TIFF
** do analizy wielopasmowej (podziel przez wartość `Chloros:PixelScale` zawartą w pliku)
3. **Wykorzystaj obrazy indeksowe** (NDVI
, NDRE
) jako gotowe do użycia warstwy roślinności
4. **Kalkulator rastrowy**: Łączenie pasm w celu przeprowadzenia niestandardowej analizy
5. **Eksport**: Tworzenie map klasyfikacyjnych, wykrywanie zmian, map stanu roślinności

### Do bezpośredniej analizy / raportowania

**Zalecany przebieg pracy:**

1.**Wykorzystaj obrazy indeksowe z kolorami LUT** do tworzenia raportów wizualnych
2. **Wyodrębnij statystyki**: średnia wartość NDVI
dla każdego pola/działki
3. **Seria czasowa**: porównaj wskaźniki z wielu sesji
4. **Generuj raporty**: dołącz mapy, statystyki i wizualizacje***

## Archiwizacja i tworzenie kopii zapasowych

### Zalecana strategia tworzenia kopii zapasowych

**Co należy zapisać:*** ✅ **Oryginalne obrazy RAW/JPG lub surowe ujęcia LATTICE** – archiwizuj na oddzielnym dysku/w chmurze; pliki surowe stanowią źródło procesu przetwarzania, a wszystko inne można z nich odtworzyć
* ✅ **Pliki z czujników światła `.daq` / `.csv`** – potrzebne do późniejszego ponownego obliczenia współczynnika odbicia
* ✅ **Wyniki przetworzenia** – zachowaj skalibrowane obrazy i wskaźniki
* ✅ **Folder projektu** (`project.json` i pliki towarzyszące) – zawiera wszystkie ustawienia niezbędne do ponownego przetwarzania w razie potrzeby
* ✅ **Dziennik debugowania** – dokumentuje szczegóły przetwarzania
* ✅ **Obrazy wzorców kalibracyjnych** – do weryfikacji i ponownego przetwarzania**Zalecenia dotyczące przechowywania:*** **Natychmiastowa kopia zapasowa**: Zewnętrzny dysk twardy
* **Archiwum długoterminowe**: Przechowywanie w chmurze (Google Drive, Dropbox itp.)
* **Dane krytyczne**: Zachowaj 2–3 kopie w różnych lokalizacjach***

## Kolejne przebiegi przetwarzania

### Ponowne wykorzystanie ustawień projektu

W przypadku przetwarzania podobnych zbiorów danych w przyszłości:

1. **Zapisz szablon projektu** (jeśli jeszcze tego nie zrobiono)
2. **Utwórz nowy projekt** przy użyciu zapisanego szablonu
3. **Zaimportuj nowe obrazy**

4.**Przetwórz**przy użyciu identycznych ustawień w celu zachowania spójności

### Przetwarzanie wsadowe wielu sesji

W przypadku wielu sesji/zbiorów danych:**Opcja 1: Interfejs graficzny (GUI) – wiele projektów**

* Utwórz oddzielny projekt dla każdej sesji
* Stosuj spójne ustawienia szablonu
* Przetwarzaj po jednym na raz

**Opcja 2: Interfejs CLI
Chloros
+ (Chloros
+ tylko)**

* Zautomatyzuj przetwarzanie wsadowe
* Przetwarzaj wiele folderów za pomocą skryptów
* Zobacz [Dokumentację CLI
](../CLI.md) oraz [Podręcznik CLI
](../reference/cli-reference.md)

**Opcja 3: Python
SDK
(tylko Chloros
+)**

* Sterowanie programowe
* Integracja z potokami analitycznymi
* Zobacz [Dokumentację API
](../api-python-sdk.md) oraz [Podręcznik SDK
](../reference/sdk-reference.md)

***

## Rozwiązywanie problemów związanych z przetwarzaniem końcowym

### Ponowne przetwarzanie z innymi ustawieniami

Jeśli wyniki nie są zadowalające:

1. Zachowaj oryginalne obrazy (nigdy ich nie usuwaj)
2. Otwórz ten sam projekt w Chloros
3. Dostosuj ustawienia w panelu „Ustawienia projektu”
4. Przetwórz ponownie — wyniki trafiają do tych samych folderów produktów, więc pliki o tej samej nazwie z poprzedniego przebiegu zostaną zastąpione

### Przetwarzanie podzbioru obrazów

Aby ponownie przetworzyć tylko określone obrazy:

1. Utwórz nowy projekt
2. Zaimportuj tylko te obrazy, które wymagają ponownego przetworzenia
3. Użyj tego samego szablonu ustawień
4. Przetwórz mniejszy zbiór danych

### Pomoc

Jeśli napotkasz problemy:

* 📧 **E-mail**: info@mapir.camera (dołącz dziennik debugowania)
* 🌐 **Pomoc techniczna**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Często zadawane pytania](../faq.md)
* 📖 **Dokumentacja**: [Podręcznik Chloros
](../)

***

## Podsumowanie: Pełny przebieg procesu

Właśnie zakończyłeś cały proces przetwarzania w Chloros
:

1. ✅ **Utworzono projekt** – zobacz [Projekty](../projects.md)
2. ✅ **Dodano pliki** – zobacz [Dodawanie plików](adding-files-to-a-project.md)
3. ✅ **Dostosowano ustawienia** – zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md)
4. ✅ **Oznaczono cele** – zobacz [Wybieranie obrazów docelowych](choosing-target-images.md)
5. ✅ **Rozpoczęto przetwarzanie** – zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
6. ✅ **Monitorowano postęp** – zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)
7. ✅ **Przejrzano wyniki** — ta strona**Twoje skalibrowane, skorygowane pod kątem odbicia obrazy wielospektralne są gotowe do analizy!**

***

## Dodatkowe zasoby

### Funkcje zaawansowane

* [**Przeglądarka obrazów**](../image-viewer-gui/opening-an-image-full-screen.md) – interaktywna wizualizacja i analiza
* [**Środowisko testowe indeksów/LUT**](../image-viewer-gui/index-lut-sandbox.md) — testowanie niestandardowych indeksów
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) — kompletny zbiór indeksów

### Automatyzacja i integracja

* [**Dokumentacja CLI**](../CLI.md) – Przetwarzanie wsadowe z wiersza poleceń
* [**Python
SDK
**](../api-python-sdk.md) – Automatyzacja programowa
* [**Funkcje Chloros
+**](../#chloros) – zaawansowane możliwości przetwarzania

### Wsparcie i szkolenia

* [**Często zadawane pytania**](../faq.md) – odpowiedzi na typowe pytania
* [**Cele kalibracyjne**](../calibration-targets.md) – Omówienie kalibracji współczynnika odbicia
* [**Obsługiwane kamery**](../supported-cameras.md) – Kompatybilny sprzęt
