# Zakończenie przetwarzania

Gdy program Chloros zakończy przetwarzanie, nadchodzi czas na przejrzenie wyników, sprawdzenie jakości obrazów oraz przygotowanie przetworzonych obrazów do wykorzystania w procesie roboczym. Niniejsza strona zawiera instrukcje dotyczące ostatnich kroków i dalszych działań.

## Wskazanie zakończenia przetwarzania

Po pomyślnym zakończeniu przetwarzania pojawi się kilka wskaźników:

* ✅ **Pasek postępu**: Osiąga 100% postępu
* ✅ **Dziennik debugowania**: Wyświetla ostatnie wiersze programu `[RUN-SUMMARY]` wraz z liczbami (obrazy, grupy kamer, cele, skalibrowane obrazy, zapisane pliki)
* ✅ **Przycisk Start**: Zostaje ponownie udostępniony (gotowy do kolejnego cyklu przetwarzania)
* ✅ **Pliki wyjściowe**: Wszystkie przetworzone obrazy zapisane w drzewie wyjściowym projektu (poniżej)

{% hint style="warning" %}
**Przebieg, w którym nie zapisano żadnych obrazów, kończy się niepowodzeniem.** Jeśli zażądano produktów obrazowych, a przebieg nie zapisał żadnego z nich, Chloros zgłasza to jako niepowodzenie — `[RUN-SUMMARY]` w nazwie dziennika sugeruje prawdopodobną przyczynę (nie zaimportowano niczego, nie wykryto celu lub pominięto wszystkie żądane produkty jako nieodpowiednie). Odpowiednik CLI kończy się wynikiem niezerowym. Celowe uruchomienie wyłącznie z metadanymi (wszystkie produkty eksportowe wyłączone, brak indeksów) nadal kończy się powodzeniem. Zobacz [Opis CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Lokalizowanie przetworzonych obrazów

### Otwieranie folderu wyjściowego

1. Kliknij ikonę **Menu głównego** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (w lewym górnym rogu)
2. Wybierz **„Otwórz folder projektu”**

3. Otworzy się eksplorator plików w katalogu projektu
4. Znajdź swój projekt według nazwy

### Drzewo wynikowe

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

* **Folder aparatu**: `LATT-<sensor>-<lens>-F<filter>` dla LATTICE (zgodny z danymi EXIF ujęcia `Model`), `<model>_<filter>` dla Survey3 (np. `Survey3N_RGN`). Dwie kamery, które mają ten sam czujnik i filtr, ale różnią się obiektywem, mają oddzielne drzewa — różnią się one winietowaniem, polem widzenia i zniekształceniami.
* **Folder formatu**: jest zgodny z ustawieniem formatu eksportu — `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32` dla TIFF (32-bitowe, procentowe). Radiance ma zawsze typ float32 i zawsze znajduje się pod `tiff32`.
* **Foldery produktów**:
  * `Reflectance_Calibrated_Images/` — skalibrowany współczynnik odbicia
  * `Debayered_Images/` — liniowe usunięcie efektu bayera (LATTICE)
  * `Preview_Images/` — podgląd na ekranie (LATTICE)
  * `Radiance_Images/` — promieniowanie spektralne typu float32, W/m²/sr/nm (LATTICE wielospektralne)
  * `Vignette_Corrected_Images/` **lub** `Sensor_Response_Images/` — nieskalibrowana wartość rezerwowa dla klatek bez odniesienia do współczynnika odbicia; w każdym przebiegu występuje dokładnie jedna z tych dwóch wartości, wybrana przez ustawienie korekcji winietowania
  * `<INDEX>_Index_Images/` — jeden folder na każdy wybrany indeks (np. `NDVI_Index_Images`)

{% hint style="info" %}
**Każdy wyeksportowany produkt zachowuje nazwę pliku ŹRÓDŁOWEGO.**Eksport radiancji o nazwie `capture_..._raw.tif` nadal nazywa się `capture_..._raw.tif` — po prostu znajduje się w folderze `tiff32/Radiance_Images/`.**To folder identyfikuje produkt, a nie nazwa pliku**, więc wyszukiwanie pliku `*radiance*.tif` nie daje żadnych wyników; zamiast tego należy wyszukiwać według katalogu.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Ile powinno być plików?

Nie licz według wzoru — liczba plików wyjściowych zależy od tego, które produkty zostały włączone i które mają zastosowanie do poszczególnych kamer (np. kamery RGB nie otrzymują danych dotyczących promieniowania/odbicia). Ostateczna liczba znajduje się w dzienniku: ostatni wiersz `[RUN-SUMMARY]` podaje dokładnie, ile plików zostało zapisanych, a wiersze z wskazówkami wyjaśniają, co zostało pominięte.

***

## Przeglądanie przetworzonych obrazów

### Szybki podgląd w Eksploratorze plików

**Wbudowany podgląd Windows:**

1. Przejdź do folderu produktu (np. `tiff16/Reflectance_Calibrated_Images/`)
2. Wybierz plik obrazu
3. Podgląd pojawi się w panelu podglądu Eksploratora Windows
4. Używaj klawiszy strzałek, aby przeglądać obrazy

### Podgląd w zewnętrznych przeglądarkach obrazów

**Zalecane przeglądarki:*** **QGIS** – bezpłatne oprogramowanie GIS (najlepsze do analizy wielospektralnej z georeferencjami)
* **IrfanView** – szybka, lekka przeglądarka obrazów (obsługuje TIFF)
* **Adobe Photoshop** – profesjonalna edycja (obsługa TIFF)
* **GIMP** – darmowa alternatywa dla programu Photoshop
* **Windows Photos** – podstawowe przeglądanie (może nie obsługiwać 16-bitowego formatu TIFF)

### Podgląd w przeglądarce obrazów Chloros

Aby uzyskać zaawansowany podgląd, skorzystaj z wbudowanej przeglądarki obrazów Chloros:

1. Kliknij miniaturę obrazu w przeglądarce plików
2. Obraz otworzy się w głównym obszarze podglądu
3. Kliknij kartę **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na lewym pasku bocznym
4. Skorzystaj z [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) w celu przeprowadzenia interaktywnej analizy

Szczegółowe instrukcje znajdziesz w sekcji [Przeglądarka obrazów](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Odczytywanie wartości odbicia pikseli (GIS / Pix4D / skrypty)

Odbicie jest przechowywane jako liczba całkowita DN, a **wartość DN odpowiadająca ρ = 1,0 zależy od kamery źródłowej**:

| Źródło          | ρ = 1,0 wynosi | Jak to sprawdzić                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (zakres do ρ 2,0) | W pliku umieszczono tag XMP `Chloros:PixelScale=32768` |
| Survey3         | **65535** (ograniczone do ρ 1,0)     | Brak tagów XMP typu `Chloros:*` — ten brak jest sygnałem |

**Odczytaj tag `Chloros:PixelScale` i podziel przez tę wartość** zamiast zakładać uniwersalną wartość 65535 — podzielenie współczynnika odbicia LATTICE przez 65535 w sposób niewidoczny zmniejsza każdą wartość o połowę. Jeden skrajny przypadek z założenia nie zawiera skali: przechwycenie ze źródła 8-bitowego zapisane jako dane wyjściowe 8-bitowe jest przycinane, a nie przeskalowywane, i celowo nie otrzymuje tagu skali — należy ponownie wyeksportować w formacie 16-bitowym lub 32-bitowym zamiast dzielić dane. Zobacz [Formaty obrazów wyjściowych](../output-image-formats.md).***

## Metadane przenoszone do eksportowanych plików

Każdy produkt zachowuje **blok GPS**z obrazu źródłowego oraz jego**podsekcję EXIF**, więc
eksport zawiera `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` oraz
`CameraSerialNumber`, a także georeferencje.

{% hint style="warning" %}
**Jeśli ortomosaika ma absurdalną skalę, należy najpierw sprawdzić `FocalLength`.**
Pix4D oblicza odległość próbkowania terenu na podstawie ogniskowej i wysokości. Bez tego tagu
program przyjmuje rażąco błędną skalę — w przypadku jednego lotu z 49 ujęciami gaj pomarańczowy o wymiarach 411 m × 160 m
gaj pomarańczowy został zrekonstruowany jako obszar o wymiarach 47,8 km × 13 km, co dało ortofotomapę o rozdzielczości 455 megapikseli, przedstawiającą głównie
pustą przestrzeń. Powolne wyświetlanie kafelków i nieoczekiwanie duży rozmiar pliku są objawami tego problemu, a nie odrębnymi
problemami.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nie *każdy* tag jest kopiowany. Tagi strukturalne IFD0 są celowo pomijane (ich kopiowanie
powoduje uszkodzenie wyników LATTICE), a `ExifImageWidth` / `ExifImageHeight` są wykluczone,
ponieważ opisują oryginalny zrzut — eksport o zmienionych wymiarach w przeciwnym razie
podawałby wymiary sprzeczne z wymiarami własnej siatki.

***

## Przeglądanie dziennika debugowania

### Sprawdzanie ostrzeżeń i błędów

1. Otwórz zakładkę **Dziennik debugowania** w programie <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Przewiń komunikaty
3. Poszukaj żółtych ostrzeżeń lub czerwonych błędów
4. Przeczytaj wiersze `[RUN-SUMMARY]` oraz wszelkie wskazówki
5. Skontaktuj się z pomocą techniczną MAPIR w celu uzyskania pomocy

### Zapisywanie dziennika

Aby zachować zapis przetwarzania lub wysłać go do pomocy technicznej MAPIR:

1. Kliknij przycisk **„Kopiuj”**lub**„Pobierz”**

2. Zapisz jako plik tekstowy w folderze projektu
3. Dołącz do dokumentacji projektu
4. Wyślij do pomocy technicznej MAPIR w razie wystąpienia problemów

***

## Typowe problemy z wynikami i ich rozwiązania

### Problem: Brakujące pliki wynikowe

**Możliwe przyczyny:**

* Produkt nie jest przeznaczony dla danej kamery (np. promieniowanie/odbicie dla kamer RGB — informacja ta znajduje się w dzienniku)
* Brakowało wymaganego odniesienia (np. współczynnik odbicia bez celu i bez promieniowania padającego typu `.daq`)
* Pole wyboru eksportu produktu było wyłączone w ustawieniach projektu
* Podczas eksportu zabrakło miejsca na dysku

**Rozwiązania:**

1. Sprawdź wskazówki `[RUN-SUMMARY]` i wiersze `[EXPORT-CHECK]` w dzienniku debugowania — wyjaśniają one pominięcia dla poszczególnych kamer
2. Sprawdź pola wyboru produktów eksportu w [Ustawieniach projektu](adjusting-project-settings.md)
3. Sprawdź, czy było wystarczająco dużo miejsca na dysku
4. Po usunięciu przyczyny uruchom przetwarzanie ponownie

### Problem: Ciemne lub jasne krawędzie (nadal widoczne winietowanie)

**Możliwe przyczyny:**

* Wyłączona korekcja winietowania
* Kamera/obiektyw nie figurują w bazie profili Chloros
* Ekstremalne winietowanie wykraczające poza możliwości korekcji

**Rozwiązania:**

1. Sprawdź, czy korekcja winietowania została włączona w ustawieniach projektu
2. Sprawdź, czy model kamery został poprawnie wykryty
3. Skontaktuj się z pomocą techniczną MAPIR, jeśli winietowanie nadal występuje

### Problem: Nieprawidłowe kolory lub wartości

**Możliwe przyczyny:**

* Nie wykryto celów kalibracyjnych
* Wybrano niewłaściwy model celu kalibracyjnego
* Wyłączona kalibracja odbicia
* Słaba jakość obrazów celów

**Rozwiązania:**

1. Sprawdź, czy kalibracja odbicia została włączona
2. Sprawdź komunikaty „Znaleziono cel” w dzienniku debugowania
3. Sprawdź jakość obrazów celów
4. Przeprowadź ponowne przetwarzanie z zaznaczonymi właściwymi celami

### Problem: Wartości NDVI wydają się nieprawidłowe

**Oczekiwane zakresy wartości NDVI:*** **Woda, skały, gleba**: od -0,1 do 0,2
* **Rzadka/niezdrowa roślinność**: od 0,2 do 0,4
* **Umiarkowana roślinność**: od 0,4 do 0,6
* **Zdrowa, gęsta roślinność**: od 0,6 do 0,9**Jeśli wartości wykraczają poza te zakresy:**

1. Sprawdź, czy zastosowano kalibrację współczynnika odbicia
2. Sprawdź, czy dołączono dziennik czujnika światła
3. Sprawdź, czy wykryto cele kalibracyjne
4. Upewnij się, że wykryto właściwy model kamery
5. Sprawdź czas i warunki wykonania zdjęć celów
6. Jeśli samodzielnie obliczasz wskaźniki na podstawie plików odbicia, upewnij się, że podzieliłeś przez wartość `Chloros:PixelScale` zawartą w pliku (patrz wyżej)

***

## Korzystanie z przetworzonych obrazów

### Do fotogrametrii / tworzenia ortomosaiki

**Zalecany przebieg pracy:**

1.**Zaimportuj skalibrowane obrazy odbicia** do oprogramowania fotogrametrycznego:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachowaj metadane EXIF**: Upewnij się, że dane GPS zostały zachowane w celu geotagowania
3. **Kalibrowane procedury**: Używaj obrazów odbicia światła w celu zapewnienia naukowej dokładności — odbicie światła LATTICE zawiera tagi kalibracyjne XMP odczytywane przez Pix4D
4. **Przetwarzaj mozaiki indeksowe**: Utwórz ortomozaiki NDVI z poszczególnych obrazów indeksowych
5. **Eksportuj georeferencjonowane pliki GeoTIFF**: Do wykorzystania w aplikacjach GIS

### Do analizy GIS

**Zalecany przebieg pracy:**

1.**Załaduj do programu QGIS, ArcGIS lub podobnego**

2.**Wykorzystaj 16-bitowe obrazy odbicia TIFF** do analizy wielopasmowej (podziel przez wartość `Chloros:PixelScale` podaną w pliku)
3. **Wykorzystaj obrazy indeksowe** (NDVI, NDRE) jako gotowe do użycia warstwy roślinności
4. **Kalkulator rastrowy**: Połącz pasma w celu przeprowadzenia niestandardowej analizy
5. **Eksport**: Twórz mapy klasyfikacyjne, wykrywanie zmian, mapy stanu roślinności

### Do bezpośredniej analizy / raportowania

**Zalecany przebieg pracy:**

1.**Wykorzystaj obrazy indeksowe z kolorami LUT** do raportów wizualnych
2. **Wyodrębnij statystyki**: Średnia wartość NDVI dla każdego pola/poletka
3. **Seria czasowa**: Porównaj wskaźniki z wielu sesji
4. **Generuj raporty**: Dołącz mapy, statystyki i wizualizacje***

## Archiwizacja i tworzenie kopii zapasowych

### Zalecana strategia tworzenia kopii zapasowych

**Co należy zapisać:*** ✅ **Oryginalne obrazy RAW/JPG lub surowe dane z LATTICE** – archiwizuj na oddzielnym dysku/w chmurze; dane surowe stanowią źródło procesu przetwarzania, a wszystko inne można z nich odtworzyć
* ✅ **Pliki z czujnika światła `.daq` / `.csv`** – potrzebne do późniejszego ponownego obliczenia współczynnika odbicia
* ✅ **Wyniki przetworzenia** – zachowaj skalibrowane obrazy i wskaźniki
* ✅ **Folder projektu** (`project.json` i pliki powiązane) – zawiera wszystkie ustawienia niezbędne do ponownego przetworzenia w razie potrzeby
* ✅ **Dziennik debugowania** – dokumentuje szczegóły przetwarzania
* ✅ **Obrazy wzorcowe do kalibracji** – Do weryfikacji i ponownego przetwarzania**Zalecenia dotyczące przechowywania:*** **Natychmiastowa kopia zapasowa**: Zewnętrzny dysk twardy
* **Archiwum długoterminowe**: Przechowywanie w chmurze (Google Drive, Dropbox itp.)
* **Dane krytyczne**: Należy przechowywać 2–3 kopie w różnych lokalizacjach***

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

**Opcja 2: Chloros CLI (tylko Chloros+)**

* Zautomatyzuj przetwarzanie wsadowe
* Przetwarzaj wiele folderów za pomocą skryptów
* Zobacz [Dokumentację CLI](../CLI.md) oraz [Podręcznik CLI](../reference/cli-reference.md)

**Opcja 3: Python SDK (tylko Chloros+)**

* Sterowanie programowe
* Integracja z potokami analitycznymi
* Zobacz [Dokumentacja API](../api-python-sdk.md) oraz [Podręcznik SDK](../reference/sdk-reference.md)

***

## Rozwiązywanie problemów związanych z przetwarzaniem końcowym

### Ponowne przetwarzanie z innymi ustawieniami

Jeśli wyniki nie są zadowalające:

1. Zachowaj oryginalne obrazy (nigdy ich nie usuwaj)
2. Otwórz ten sam projekt w Chloros
3. Dostosuj ustawienia w panelu „Ustawienia projektu”
4. Przetwórz ponownie — wyniki trafiają do tych samych folderów produktów, więc pliki o tej samej nazwie z poprzedniego przebiegu zostaną zastąpione

### Przetwarzanie podzbioru obrazów

Aby ponownie przetworzyć tylko wybrane obrazy:

1. Utwórz nowy projekt
2. Zaimportuj tylko te obrazy, które wymagają ponownego przetworzenia
3. Użyj tego samego szablonu ustawień
4. Przetwórz mniejszy zbiór danych

### Pomoc

Jeśli napotkasz problemy:

* 📧 **E-mail**: info@mapir.camera (dołącz dziennik debugowania)
* 🌐 **Pomoc techniczna**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Często zadawane pytania](../faq.md)
* 📖 **Dokumentacja**: [Podręcznik Chloros](../)***

## Podsumowanie: Pełny przebieg pracy

Właśnie zakończyłeś cały przebieg przetwarzania Chloros:

1. ✅ **Utworzono projekt** – zobacz [Projekty](../projects.md)
2. ✅ **Dodano pliki** — zobacz [Dodawanie plików](adding-files-to-a-project.md)
3. ✅ **Dostosowano ustawienia** — zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md)
4. ✅ **Oznaczono cele** – zobacz [Wybór obrazów docelowych](choosing-target-images.md)
5. ✅ **Rozpoczęto przetwarzanie** – zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
6. ✅ **Monitorowanie postępu** – zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)
7. ✅ **Przegląd wyników** – ta strona**Twoje skalibrowane, skorygowane pod kątem odbicia obrazy wielospektralne są gotowe do analizy!**

***

## Dodatkowe zasoby

### Funkcje zaawansowane

* [**Przeglądarka obrazów**](../image-viewer-gui/opening-an-image-full-screen.md) – interaktywna wizualizacja i analiza
* [**Środowisko testowe indeksów/tablic LUT**](../image-viewer-gui/index-lut-sandbox.md) – testowanie niestandardowych indeksów
* [**Wzory indeksów wielospektralnych**](../project-settings/multispectral-index-formulas.md) – kompletny zbiór indeksów

### Automatyzacja i integracja

* [**Dokumentacja CLI**](../CLI.md) – Przetwarzanie wsadowe z wiersza poleceń
* [**Python SDK**](../api-python-sdk.md) — automatyzacja programowa
* [**Chloros+ Funkcje**](../#chloros) — zaawansowane możliwości przetwarzania

### Pomoc techniczna i szkolenia

* [**Często zadawane pytania**](../faq.md) – odpowiedzi na najczęściej zadawane pytania
* [**Cele kalibracyjne**](../calibration-targets.md) – Informacje o kalibracji współczynnika odbicia
* [**Obsługiwane kamery**](../supported-cameras.md) – Kompatybilny sprzęt
