# Dostosowywanie ustawień projektu

Przed rozpoczęciem przetwarzania obrazów należy skonfigurować ustawienia projektu zgodnie z wymaganiami danego procesu pracy. Panel „Ustawienia projektu” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> zapewnia kompleksową kontrolę nad kalibracją, opcjami przetwarzania, wskaźnikami wielospektralnymi oraz formatami eksportu.

## Dostęp do ustawień projektu

1. Otwórz projekt w programie Chloros
2. Kliknij ikonę **Ustawienia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. W panelu „Ustawienia projektu” wyświetlą się wszystkie opcje konfiguracyjne

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panel „Ustawienia projektu” — wyświetlanie, wykrywanie obiektów i przetwarzanie</p></figcaption></figure>

{% hint style="info" %}
**Ustawienia są zapisywane automatycznie** wraz z projektem. Po ponownym otwarciu projektu wszystkie ustawienia zostaną przywrócone.
{% endhint %}

***

## Szybka konfiguracja dla typowych procesów roboczych

### Ustawienia domyślne (zalecane dla większości użytkowników)

Ustawienia domyślne sprawdzają się dobrze w typowych procesach roboczych Survey3 i LATTICE:

* ✅ **Korekcja winietowania**: Włączona
* ✅ **Kalibracja odbicia / balans bieli**: Włączona (wykorzystuje tarcze MAPIR i/lub dane z czujnika światła DAQ)
* ✅ **Metoda debayeringu**: Standardowa (szybka, średnia jakość)
* ✅ **Format eksportu**: TIFF (16-bitowy) (TIFF (8-bitowy) w przypadku projektu w całości 8-bitowego)
* ✅ **Wszystkie produkty eksportu**: Włączone (LATTICE automatycznie generuje rozgałęzienie na obrazy po usunięciu efektu bayera, podgląd, promieniowanie i odbicie)

Wystarczy zaimportować obrazy i rozpocząć przetwarzanie przy użyciu tych ustawień domyślnych.

***

## Przegląd ustawień projektu

Panel ustawień projektu jest podzielony na poniższe sekcje. Dwie dodatkowe sekcje — **Czujnik światła DAQ**i**Wyrównanie macierzy** — pojawiają się automatycznie, gdy projekt zawiera odpowiednie pliki. Pełna dokumentacja znajduje się w sekcji [Ustawienia projektu](../project-settings/project-settings.md).

### Wyświetlanie

* **Rozdzielczość miniatur obrazów**: Rozdzielczość miniatur w siatce obrazów. Opcje:**Domyślna (512 px)**,**1024 px**,**2048 px**,**Pełna rozdzielczość**. Ma wpływ wyłącznie na wyświetlanie — nigdy nie wpływa na przetwarzanie. Wyższe wartości zapewniają ostrzejszy obraz po powiększeniu, ale powodują wolniejsze ładowanie.

### Wykrywanie celów

Kontroluje sposób, w jaki Chloros identyfikuje cele kalibracyjne na obrazach.

**Kluczowe ustawienia:*** **Minimalny obszar próbki kalibracyjnej (px)**: Próg wielkości dla wykrywania celów (domyślnie:**25**, zakres 0–10000)
* **Minimalne skupienie celów (0–100)**: Próg podobieństwa dla grupowania obszarów celów (domyślnie:**60**)**Kiedy należy dostosować:**

* Zwiększ obszar próbkowania, jeśli pojawiają się fałszywe wykrycia
* Zmniejsz, jeśli cele nie są wykrywane
* Dostosuj skupianie, jeśli cele są rozdzielane na wiele wykryć

{% hint style="info" %}
Ustawienia te są wyszarzone, gdy opcja **Kalibracja odbicia / balans bieli** jest wyłączona — przy wyłączonej funkcji wykrywanie celów w ogóle nie działa.
{% endhint %}

### Przetwarzanie

Główne opcje przetwarzania obrazu i kalibracji.

**Kluczowe ustawienia:*** **Korekcja winietowania**: Kompensuje zaciemnienie obiektywu na krawędziach ✅ Zalecane
* **Kalibracja odbicia / balans bieli**: Kalibruje obrazy przy użyciu wykrytych celów (program Survey3) i/lub danych z czujnika światła DAQ (LATTICE) ✅ Zalecane
* **Metoda debayera**: Algorytm konwersji plików RAW na 3-kanałowe obrazy wielospektralne
* **Minimalny interwał ponownej kalibracji**: Minimalny czas w sekundach między kolejnymi użyciami celu kalibracyjnego (domyślnie:**0** = użyj wszystkich, zakres 0–3600)**Produkty rezerwowe bez kalibracji:**Gdy nie można przeprowadzić kalibracji odbicia dla klatki (brak dostępnego celu lub kalibracja wyłączona), jest ona eksportowana jako jeden z dwóch produktów rezerwowych —**na każdy przebieg istnieje dokładnie jeden z tej pary**, wybrany przez przełącznik korekcji winietowania:

* **Eksport charakterystyki czujnika**: zapisuje plik `Sensor_Response_Images` — stosowany, gdy korekcja winietowania jest**wyłączona*** **Eksport z korekcją winietowania**: zapisuje plik `Vignette_Corrected_Images` — stosowany, gdy korekcja winietowania jest**włączona**Pole wyboru, które nie jest aktywne, jest wyszarzone. Odznaczenie aktywnego pola powoduje całkowite wstrzymanie zapisu tego pliku.**Produkty eksportowe LATTICE** (wyświetlane dla każdego projektu; dotyczą one ujęć z LATTICE):

* **Eksport z usuniętym efektem bayera**: liniowy obraz po usunięciu efektu bayera (`Debayered_Images`): usunięto ciemne przesunięcie z pakietu kalibracyjnego, a następnie przeprowadzono demosaicing, bez płaskiego pola ani demiksowania — ten sam plik, jaki zapisuje się podczas przechwytywania na poziomie po usunięciu efektu bayera — w natywnej orientacji czujnika. Dotyczy modułów RGB i wielospektralnych.
* **Eksport podglądu**: podgląd wyświetlany na ekranie (`Preview_Images`). RGB = balans bieli (źródło światła DAQ, jeśli jest dostępne, w przeciwnym razie „szary świat”) + gamma; wielospektralny = rozciągnięcie w fałszywych kolorach.
* **Eksport promieniowania**: promieniowanie spektralne typu float32 (`Radiance_Images`, W/m²/sr/nm). Tylko moduły wielospektralne — nie dotyczy plików głównych RGB.
* **Eksport współczynnika odbicia**: współczynnik odbicia typu uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), gdy odczyt promieniowania padającego `.daq` lub cel w ramce pokrywa całą ramkę. Tylko moduły wielospektralne.

Wszystkie cztery opcje są **domyślnie włączone**— jedna zaimportowana surowa klatka LATTICE jest rozdzielana na wszystkie włączone i odpowiednie produkty w ramach jednego przebiegu przetwarzania. Pole wyboru**Eksportuj współczynnik odbicia** jest wyszarzone, gdy kalibracja współczynnika odbicia jest wyłączona. Ustawienia, których aktywacja uniemożliwia działanie nadrzędnego przełącznika, są zawsze wyszarzone i zawierają podpowiedź wskazującą przełącznik, który należy zmienić.**Ustawienia zaawansowane:*** **Przesunięcie strefy czasowej czujnika światła**: Liczba godzin względem czasu UTC służąca do synchronizacji czasu czujnika światła (domyślnie: 0, zakres od −12 do +12)
* **Zastosuj korekty PPK**: Wykorzystuje dane GPS/punktów ekspozycji z plików `.daq` (domyślnie: wyłączone)
* **Punkt ekspozycji 1/2**: Przypisuje kamery do punktów ekspozycji w konfiguracjach z dwoma kamerami

{% hint style="info" %}
**Poziom wejściowy LATTICE jest automatyczny.** Nagrania z LATTICE zawierają poziom przetwarzania w metadanych XMP, a przetwarzanie zawsze rozpoczyna się od surowej klatki — nie ma nic do konfigurowania w interfejsie graficznym. (Flaga CLI `--input-level` służy jako opcja dla zaawansowanych użytkowników w przypadku nagrań z utraconymi metadanymi; zobacz [Podręcznik CLI](../reference/cli-reference.md).)
{% endhint %}

### Metoda usuwania efektu bayera

Obecnie w programie Chloros oferujemy 2 metody usuwania efektu bayera:

#### Standardowa (szybka, średnia jakość)

Standardowa metoda usuwania efektu bayera działa szybko, ale powoduje pojawienie się szumu kolorowego, co skutkuje mniej dokładnymi i bardziej zaszumionymi obrazami.

#### Z uwzględnieniem tekstury (powolna, najwyższa jakość) \[Tylko Chloros+]

Metoda z uwzględnieniem tekstury wykorzystuje wysokiej jakości algorytm usuwania matrycy z uwzględnieniem krawędzi w połączeniu z modelem usuwania szumu opartym na AI/ML, który eliminuje niemal cały szum wynikający z tego procesu. Model wymaga pamięci GPU (VRAM) do działania: przy **7 GB lub więcej pamięci VRAM** może przetwarzać wiele obrazów jednocześnie; poniżej 7 GB przetwarza jeden obraz na raz (znacznie wolniej). Zobacz [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**W przypadku zdjęć LATTICE zawsze stosowana jest standardowa demosaikacja.** Nie istnieje model Texture Aware wytrenowany dla LATTICE, więc opcja ta nie jest dostępna dla zdjęć LATTICE — zdjęcia z serwisu Survey3 w tym samym projekcie mogą jednak z niej korzystać.
{% endhint %}

### Indeksy (indeksy wielospektralne)

Skonfiguruj, które indeksy wegetacyjne mają być obliczane i eksportowane. Menu rozwijane w interfejsie graficznym oferuje **27 predefiniowanych wzorów indeksów**.**Jak dodać indeksy:**

1. Kliknij przycisk**„Dodaj indeks”**

2. Wybierz wskaźnik z menu rozwijanego (NDVI, NDRE, GNDVI itp.)
3. Skonfiguruj ustawienia wizualizacji (kolory LUT, zakresy wartości)
4. Dodaj dowolną liczbę wskaźników

**Popularne wskaźniki:*** **NDVI**: Ogólny stan zdrowia roślinności (najczęściej stosowany)
* **NDRE**: Wczesne wykrywanie stresu za pomocą RedEdge
* **GNDVI**: Wrażliwy na stężenie chlorofilu
* **OSAVI**: Działa dobrze w przypadku widocznej gleby
* **EVI**: Regiony o wysokim wskaźniku powierzchni liści (LAI)

**Formuły niestandardowe:**

* Twórz niestandardowe formuły indeksów wielospektralnych za pomocą operacji matematycznych na pasmach we wszystkich kanałach obrazu
* Zapisuj niestandardowe formuły w celu ponownego wykorzystania
* Formuły niestandardowe są funkcją Chloros+; ich dostępność zależy od poziomu planu

Aby zapoznać się ze wszystkimi dostępnymi wskaźnikami i formułami — w tym z informacją, które nazwy są dostępne wyłącznie w interfejsie graficznym (GUI), a które działają również w interfejsie CLI lub SDK — zobacz [Formuły wskaźników wielospektralnych](../project-settings/multispectral-index-formulas.md).

### Eksport

Steruje formatem pliku wyjściowego.

**Dostępne formaty**(ustawienie:**Format obrazu skalibrowanego**, domyślnie**TIFF (16-bitowy)**):

* **TIFF (16-bitowy)**: Zalecany do zastosowań GIS i analiz naukowych
* **TIFF (8-bit)**: 8-bitowy TIFF
* **TIFF (32-bit, procent)**: Wartości zmiennoprzecinkowe
* **PNG (8-bit)**: Bezstratna kompresja do wizualizacji
* **JPG (8-bit)**: Najmniejsze pliki, kompresja stratna

Lista pokazuje, jakie formaty mogą mieć obrazy w projekcie: Chloros nie zwiększa głębi bitowej. Obrazy wyłącznie 8-bitowe są dostępne tylko w formatach TIFF (8-bit), PNG (8-bit) i JPG (8-bit), a domyślnym formatem jest TIFF (8-bit). W projekcie mieszanym każde ujęcie jest eksportowane z własną głębią bitową lub niższą. Radiance pozostaje w formacie 32-bitowym zmiennoprzecinkowym.

Pliki wyjściowe są zapisywane w folderze projektu, pogrupowane według kamery i formatu: `<project>/<camera>/<format>/<Product>_Images/`. Dane o jasności są **zawsze** zapisywane jako float32 w folderze `tiff32`, niezależnie od tego ustawienia. Eksportowane pliki zachowują nazwę pliku źródłowego — folder identyfikuje produkt. Eksporty wielospektralne LATTICE mogą również zapisywać maski ograniczające dla poszczególnych pikseli w folderze `clamp_masks/` obok folderów produktów. Pełną strukturę drzewa wyników można znaleźć w sekcji [Zakończenie przetwarzania](finishing-the-processing.md).

{% hint style="warning" %}
**Odczytywanie wartości odbicia**: wartość DN odpowiadająca ρ = 1,0 zależy od kamery źródłowej — LATTICE używa wartości 32768 (oznaczonej w pliku XMP jako `Chloros:PixelScale`), natomiast Survey3 używa wartości 65535. Należy odczytać tę wartość, a nie zakładać stałej wartości. Zobacz [Formaty obrazów wyjściowych](../output-image-formats.md).
{% endhint %}

### Czujnik światła DAQ

W tej sekcji wymieniono wszystkie pliki DAQ typu „downwelling” (`.daq` / `.csv`) w projekcie, po jednym wierszu na plik, z podaniem modelu czujnika, nazwy pliku oraz **nakładki** dyfuzora obowiązującą dla tego pliku.

* **Zastąpienie wartości**cap**(wszystkie pliki)**: pojedyncza lista rozwijana dla całego projektu.**Auto** (domyślnie) wykorzystuje wartość nakładki zarejestrowaną w każdym pliku — w przypadku braku danych zakłada się nasłonecznienie, jednak modele DAQ-U, DAQ-M i DAQ-E są dostarczane bez nakładki, a program Chloros nie może rozpoznać dopasowanego modelu, więc podczas przetwarzania wyświetla ostrzeżenie o założonej wartości nakładki: należy wybrać nakładkę, która została dopasowana. Wybranie wartości granicznej zastępuje ustawienia we wszystkich plikach z wyjątkiem plików z urządzenia DAQ-E-S, którego dyfuzor stanowi część kalibracji: surowe zapisy są korygowane przy jego użyciu, a zapisy, które już zawierają wartość graniczną, są ponownie referencjonowane (zapisana korekta jest cofana, a wybrana wartość graniczna jest stosowana).
* Wiersze wyświetlają ostrzeżenie, gdy zarejestrowana nasadka była domyślną wartością zakładaną przez hub, a nie potwierdzoną przez operatora, oraz gdy wybrana nasadka nie ma profilu dla danego modelu urządzenia (w tym przypadku nadpisanie jest odrzucane dla tego pliku).

Zapisy DAQ wykonane w zakładce „Czujniki światła” są automatycznie dodawane do otwartego projektu, a zaimportowane pliki `.daq` / `.csv` pojawiają się tutaj natychmiast po ich dodaniu.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Dolne ustawienia projektu — Indeks, Format eksportu, sekcja DAQ „Czujniki światła” oraz elementy sterujące szablonem projektu/folderem</p></figcaption></figure>### Wyrównanie tablic

Ta sekcja pojawia się **tylko** wtedy, gdy co najmniej jeden obraz w projekcie zawiera transformację wyrównania między modułami, którą tablice LATTICE nanoszą podczas przechwytywania (`Chloros:Alignment*` XMP). Pokazuje, ile obrazów zawiera tagi i która kamera jest punktem odniesienia, wraz z następującymi opcjami:

* **Zastosuj wyrównanie matrycy** (domyślnie: włączone): dopasowuje każdy przetworzony produkt (po usunięciu pikseli / podgląd / promieniowanie / odbicie / indeks) do wspólnej geometrii odniesienia matrycy. Wyłączone = eksport w natywnej geometrii czujnika.
* **Przytnij do wspólnego nakładania się** (domyślnie: włączone): przycina wyrównane eksporty do obszaru wspólnego dla wszystkich modułów, dzięki czemu każde pasmo ma ten sam obszar pokrycia. Wyłączenie zachowuje pełny obszar czujnika (czarne wypełnienie poza źródłem).
* **Próbkowanie**:**Biliniowe (wygładzone, domyślne)**,**Najbliższe (zachowujące dokładne wartości)**— bez mieszania pikseli, do ścisłej analizy radiometrycznej — lub**Kubiczne (najostrzejsze)**.

***

## Zapisywanie i wczytywanie ustawień

### Zapisywanie szablonu projektu

Twórz szablony wielokrotnego użytku, aby zapewnić spójność procesów roboczych:

1. Skonfiguruj wszystkie żądane ustawienia w panelu Ustawienia projektu
2. Przewiń do sekcji **„Zapisz szablon projektu”** na dole
3. Wprowadź opisową nazwę szablonu (np. „Survey3N\_RGN\_Agriculture”)
4. Kliknij ikonę zapisywania

**Korzyści:**

* Stosowanie identycznych ustawień w wielu projektach
* Udostępnianie konfiguracji członkom zespołu
* Zachowanie spójności w przypadku powtarzających się badań

### Wczytanie szablonu do nowego projektu

Podczas tworzenia nowego projektu:

1. Wybierz **„Nowy projekt”** z menu głównego
2. Wybierz szablon projektu w opcjonalnym selektorze szablonów
3. Wszystkie ustawienia z szablonu zostaną automatycznie zastosowane

### Katalog roboczy

Ustawienie **„Katalog roboczy”** określa domyślną lokalizację tworzenia nowych projektów:

* **Lokalizacja domyślna**: `Documents\MAPIR\Chloros Projects` w folderze „Dokumenty” systemu Windows (zwykle `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; lokalizacja ta podąża za folderem „Dokumenty”, jeśli zostanie on przeniesiony przez OneDrive lub profil wędrujący)
* **Zmień lokalizację**: kliknij ikonę edycji i wybierz nowy folder
* **Udostępnione dla CLI**: `chloros-cli` korzysta z tego samego domyślnego ustawienia folderu projektu
* **Krótkie nazwy w systemie Windows**: o ile nie włączono obsługi długich ścieżek w systemie Windows, folder projektu o nazwie dłuższej niż około 130 znaków zostanie odrzucony przed przetwarzaniem — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Kiedy zmienić**:
  * Dysk sieciowy do współpracy zespołowej
  * Inny dysk z większą przestrzenią dyskową
  * Uporządkowana struktura folderów według roku/klienta

***

## Konfiguracja PPK (Post-Processed Kinematic)

W przypadku korzystania z rejestratorów danych MAPIR DAQ z GPS w celu precyzyjnej geolokalizacji:

### Wymagania wstępne

* Rejestrator MAPIR DAQ z modułem GPS (GNSS)
* Plik dziennika .daq zawierający wpisy dotyczące pinów ekspozycji
* Kamera podłączona do pinów ekspozycji rejestratora DAQ podczas sesji rejestracji

### Kroki konfiguracji

1. Umieść plik dziennika .daq w folderze projektu
2. W ustawieniach projektu zaznacz pole wyboru **„Zastosuj korekty PPK”**

3. W razie potrzeby ustaw**„Przesunięcie strefy czasowej czujnika światła”** (domyślnie: 0 dla UTC)
4. Przypisz kamery do pinów ekspozycji:
   * **Pojedyncza kamera**: automatycznie przypisana do pinu 1
   * **Dwie kamery**: ręcznie przypisz każdą kamerę do odpowiedniego pinu**Przypisanie pinów ekspozycji:*** **Pin ekspozycji 1**: Wybierz model kamery z listy rozwijanej
* **Pin ekspozycji 2**: Wybierz drugą kamerę lub opcję „Nie używaj”
* Nie można przypisać tej samej kamery do obu pinów

{% hint style="warning" %}
**Ważne**: Piny ekspozycji muszą być poprawnie przypisane do odpowiednich kamer. Nieprawidłowe przypisanie spowoduje uzyskanie błędnych danych geolokalizacyjnych.
{% endhint %}

***

## Zaawansowane scenariusze

### Projekty z wieloma kamerami

Podczas przetwarzania zdjęć z wielu kamer MAPIR w jednym projekcie:

1. Chloros automatycznie wykrywa model każdej kamery (zarówno Survey3, jak i LATTICE)
2. Każda kamera otrzymuje odpowiednie profile przetwarzania oraz własną strukturę folderów wyjściowych
3. PPK: Należy ręcznie przypisać każdą kamerę Survey3 do właściwego pinu ekspozycji
4. Wszystkie kamery używają tego samego formatu eksportu i indeksów

**Przykłady**: konfiguracja z dwoma kamerami Survey3W RGN + Survey3N OCN lub układ LATTICE łączący kamerę główną RGB z modułami wąskopasmowymi

### Pomiary w trybie poklatkowym lub z wielu dat

W przypadku powtarzających się pomiarów tego samego obszaru w czasie:

1. Utwórz szablon ze standardowymi ustawieniami
2. W każdej sesji stosuj spójną konfigurację celów kalibracyjnych
3. Przetwarzaj każdy dzień jako oddzielny projekt
4. Stosuj identyczne ustawienia, aby uzyskać porównywalne wyniki
5. Eksportuj w tym samym formacie w celu analizy czasowej

### Duże zbiory danych

W przypadku projektów zawierających wiele zdjęć (ponad 500):

* Rozważ podział na mniejsze projekty według daty lub obszaru
* Skorzystaj z przetwarzania równoległego w programie Chloros+, aby uzyskać szybsze wyniki
* Rozważ użycie interfejsu CLI lub API do automatyzacji przetwarzania wsadowego
* Dostosuj minimalny interwał ponownej kalibracji, aby skrócić czas wykrywania celów

***

## Sprawdzanie ustawień

Przed rozpoczęciem przetwarzania sprawdź następujące kluczowe ustawienia:

* [ ] Model kamery został poprawnie wykryty w przeglądarce plików
* [ ] Włączona korekcja winietowania
* [ ] Włączona kalibracja odbicia
* [ ] W przypadku Survey3: zaimportowano i sprawdzono co najmniej jeden obraz celu kalibracyjnego; w przypadku LATTICE: obecny jest cel i/lub zapis `.daq` w kierunku w dół
* [ ] Dodano żądane wskaźniki wielospektralne
* [ ] Format eksportu odpowiedni dla Twojego przepływu pracy
* [ ] Skonfigurowano ustawienia PPK (w przypadku korzystania z plików .daq z zdarzeniami ekspozycji)

***

## Kolejne kroki

Po skonfigurowaniu ustawień:

1. **Zaznacz obrazy wzorcowe** – zobacz [Wybór obrazów wzorcowych](choosing-target-images.md)
2. **Rozpocznij przetwarzanie** – zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
3. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)

Aby uzyskać szczegółowe informacje na temat wszystkich dostępnych ustawień, zapoznaj się z dokumentacją referencyjną [Ustawienia projektu](../project-settings/project-settings.md).
