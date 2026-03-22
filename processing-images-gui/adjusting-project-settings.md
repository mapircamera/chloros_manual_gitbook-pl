# Dostosowywanie ustawień projektu

Przed rozpoczęciem przetwarzania obrazów należy skonfigurować ustawienia projektu zgodnie z wymaganiami danego procesu pracy. Panel „Ustawienia projektu” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> zapewnia kompleksową kontrolę nad kalibracją, opcjami przetwarzania, wskaźnikami wielospektralnymi oraz formatami eksportu.

## Dostęp do ustawień projektu

1. Otwórz projekt w Chloros
2. Kliknij ikonę **Ustawienia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. Panel Ustawienia projektu wyświetla wszystkie opcje konfiguracyjne

{% hint style="info" %}
**Ustawienia są zapisywane automatycznie** wraz z projektem. Po ponownym otwarciu projektu wszystkie ustawienia zostaną przywrócone.
{% endhint %}

***

## Szybka konfiguracja dla typowych procesów pracy

### Ustawienia domyślne (zalecane dla większości użytkowników)

W przypadku typowych procesów pracy z kamerami MAPIR Survey3 ustawienia domyślne sprawdzają się dobrze:

* ✅ **Korekcja winietowania**: Włączona
* ✅ **Kalibracja odbicia**: Włączona (wymagane są zdjęcia celów MAPIR)
* ✅ **Metoda debayera**: Standardowa (szybka, średnia jakość)
* ✅ **Format eksportu**: TIFF (16-bitowy)

Wystarczy zaimportować zdjęcia i rozpocząć przetwarzanie przy użyciu tych ustawień domyślnych.

***

## Przegląd ustawień projektu

Panel ustawień projektu jest podzielony na kilka kategorii. Poniżej znajduje się podsumowanie każdej sekcji. Pełna dokumentacja znajduje się w [Ustawieniach projektu](../project-settings/project-settings.md).

### Wykrywanie celów

Kontroluje sposób, w jaki Chloros identyfikuje cele kalibracyjne na obrazach.

**Kluczowe ustawienia:*** **Minimalny obszar próbki kalibracji**: Próg wielkości dla wykrywania celów (domyślnie: 25 pikseli)
* **Minimalne grupowanie celów**: Próg podobieństwa dla grupowania obszarów docelowych (domyślnie: 60)**Kiedy dostosować:**

* Zwiększ obszar próbki, jeśli pojawiają się fałszywe wykrycia
* Zmniejsz, jeśli cele nie są wykrywane
* Dostosuj grupowanie, jeśli cele są rozdzielane na wiele wykryć

### Przetwarzanie

Główne opcje przetwarzania obrazu i kalibracji.

**Kluczowe ustawienia:*** **Korekcja winietowania**: Kompensuje zaciemnienie obiektywu na krawędziach ✅ Zalecane
* **Kalibracja odbicia**: Normalizuje wartości przy użyciu celów kalibracyjnych ✅ Zalecane
* **Metoda debayera**: Algorytm konwersji plików RAW na 3-kanałowe obrazy wielospektralne
* **Minimalny interwał ponownej kalibracji**: Czas między użyciem celów kalibracyjnych (0 = użyj wszystkich)**Ustawienia zaawansowane:*** **Przesunięcie strefy czasowej czujnika światła**: Do synchronizacji czasu PPK (domyślnie: 0)
* **Zastosuj korekty PPK**: Wykorzystuje dane GPS/pinów ekspozycji z plików .daq
* **Pin ekspozycji 1/2**: Przypisuje kamery do pinów ekspozycji w konfiguracjach z dwoma kamerami

### Metoda debayeringu

Obecnie w Chloros oferujemy 2 metody debayeringu:

#### Standardowa (szybka, średnia jakość)

Standardowy algorytm debayeringu działa szybko, ale powoduje szumy kolorowe, co skutkuje mniej dokładnymi i bardziej zaszumionymi obrazami.

#### Z uwzględnieniem tekstury (powolny, najwyższa jakość) \[Tylko Chloros+]

Metoda z uwzględnieniem tekstury wykorzystuje wysokiej jakości algorytm debayeringu z uwzględnieniem krawędzi w połączeniu z modelem usuwania szumu opartym na sztucznej inteligencji (AI/ML), który eliminuje prawie cały szum debayeringu. Model z uwzględnieniem tekstury wymaga pamięci GPU (VRAM) do działania. Zalecamy korzystanie z niego, gdy masz do dyspozycji &gt;4 GB pamięci VRAM, aby zapewnić szybsze przetwarzanie.

### Indeksy (indeksy wielospektralne)

Skonfiguruj, które indeksy roślinności mają być obliczane i eksportowane.

**Jak dodać wskaźniki:**

1. Kliknij przycisk**„Dodaj wskaźnik”**

2. Wybierz wskaźnik z menu rozwijanego (NDVI, NDRE, GNDVI itp.)
3. Skonfiguruj ustawienia wizualizacji (kolory LUT, zakresy wartości)
4. Dodaj wiele wskaźników w razie potrzeby

**Popularne wskaźniki:*** **NDVI**: Ogólny stan zdrowia roślinności (najczęściej stosowany)
* **NDRE**: Wczesne wykrywanie stresu wraz z RedEdge
* **GNDVI**: Wrażliwy na stężenie chlorofilu
* **OSAVI**: Działa dobrze w przypadku widocznej gleby
* **EVI**: Regiony o wysokim wskaźniku powierzchni liści (LAI)**Formuły niestandardowe (tylko Chloros+):**

* Twórz niestandardowe formuły indeksów wielospektralnych
* Korzystaj z operacji matematycznych na pasmach dla wszystkich kanałów obrazu
* Zapisuj niestandardowe formuły do ponownego wykorzystania

Wszystkie dostępne indeksy i formuły można znaleźć w sekcji [Formuły indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md).

### Eksport

Kontroluje format i jakość pliku wyjściowego.

**Dostępne formaty:*** **TIFF (16-bitowy)**: Zalecany do GIS i analiz naukowych (zakres 0–65 535)
* **TIFF (32-bitowy, procent)**: Wartości odbicia w formacie zmiennoprzecinkowym (zakres 0,0–1,0)
* **PNG (8-bitowy)**: Bezstratna kompresja do wizualizacji (zakres 0–255)
* **JPG (8-bitowy)**: Najmniejsze pliki, kompresja stratna (zakres 0–255)***

## Zapisywanie i wczytywanie ustawień

### Zapisywanie szablonu projektu

Twórz szablony wielokrotnego użytku, aby zapewnić spójność procesów:

1. Skonfiguruj wszystkie żądane ustawienia w panelu Ustawienia projektu
2. Przewiń do sekcji **„Zapisz szablon projektu”** na dole
3. Wprowadź opisową nazwę szablonu (np. „Survey3N\_RGN\_Rolnictwo”)
4. Kliknij ikonę zapisu

**Korzyści:**

* Stosuj identyczne ustawienia w wielu projektach
* Udostępniaj konfiguracje członkom zespołu
* Zachowaj spójność w przypadku powtarzających się ankiet

### Wczytaj szablon do nowego projektu

Podczas tworzenia nowego projektu:

1. Wybierz **„Nowy projekt”** z menu głównego
2. Wybierz opcję **„Wczytaj z szablonu”**

3. Wybierz zapisany szablon
4. Wszystkie ustawienia zostaną automatycznie zastosowane

### Katalog roboczy

Ustawienie **„Katalog zapisu projektu”** określa domyślną lokalizację tworzenia nowych projektów:

* **Lokalizacja domyślna**: `C:\Users\[Username]\Chloros Projects`
* **Zmień lokalizację**: Kliknij ikonę edycji i wybierz nowy folder
* **Kiedy zmienić**:
  * Dysk sieciowy do współpracy zespołowej
  * Inny dysk z większą przestrzenią dyskową
  * Uporządkowana struktura folderów według roku/klienta

***

## Konfiguracja PPK (Post-Processed Kinematic)

W przypadku korzystania z rejestratorów DAQ MAPIR z GPS do precyzyjnej geolokalizacji:

### Wymagania wstępne

* Rejestrator MAPIR DAQ z modułem GPS (GNSS)
* Plik dziennika .daq z wpisami dotyczącymi pinów ekspozycji
* Kamera podłączona do pinów ekspozycji rejestratora DAQ podczas sesji przechwytywania

### Kroki konfiguracji

1. Umieść plik dziennika .daq w folderze projektu
2. W ustawieniach projektu zaznacz pole wyboru **„Zastosuj korekty PPK”**

3. W razie potrzeby ustaw**„Przesunięcie strefy czasowej czujnika światła”** (domyślnie: 0 dla UTC)
4. Przypisz kamery do pinów ekspozycji:
   * **Pojedyncza kamera**: automatycznie przypisana do pinu 1
   * **Dwie kamery**: ręcznie przypisz każdą kamerę do odpowiedniego pinu**Przypisanie pinów ekspozycji:*** **Pin ekspozycji 1**: Wybierz model kamery z listy rozwijanej
* **Pin ekspozycji 2**: Wybierz drugą kamerę lub opcję „Nie używaj”
* Ta sama kamera nie może być przypisana do obu pinów

{% hint style="warning" %}
**Ważne**: Piny ekspozycji muszą być poprawnie przypisane do odpowiednich kamer. Nieprawidłowe przypisanie spowoduje błędne dane geolokalizacyjne.
{% endhint %}

***

## Zaawansowane scenariusze

### Projekty z wieloma kamerami

Podczas przetwarzania obrazów z wielu kamer MAPIR w jednym projekcie:

1. Chloros automatycznie wykrywa model każdej kamery
2. Każda kamera otrzymuje odpowiedni profil przetwarzania
3. PPK: Ręcznie przypisz każdą kamerę do właściwego pinu ekspozycji
4. Wszystkie kamery używają tego samego formatu eksportu i indeksów

**Przykład**: Survey3W RGN + Survey3N OCN zestaw z dwiema kamerami

### Badania poklatkowe lub wielodatowe

W przypadku powtarzających się badań tego samego obszaru w czasie:

1. Utwórz szablon ze standardowymi ustawieniami
2. Używaj spójnej konfiguracji celów kalibracyjnych w każdej sesji
3. Przetwarzaj każdą datę jako oddzielny projekt
4. Używaj identycznych ustawień, aby uzyskać porównywalne wyniki
5. Eksportuj w tym samym formacie w celu analizy czasowej

### Duże zbiory danych

W przypadku projektów zawierających wiele obrazów (ponad 500):

* Rozważ podzielenie na mniejsze projekty według daty lub obszaru
* Użyj przetwarzania równoległego Chloros+ w celu uzyskania szybszych wyników
* Rozważ użycie CLI lub API do automatyzacji przetwarzania wsadowego
* Dostosuj minimalny interwał ponownej kalibracji, aby skrócić czas wykrywania celów

***

## Sprawdzanie ustawień

Przed rozpoczęciem przetwarzania sprawdź następujące kluczowe ustawienia:

* [ ] Model kamery został poprawnie wykryty w przeglądarce plików
* [ ] Włączona korekcja winietowania
* [ ] Włączona kalibracja odbicia
* [ ] Zaimportowano co najmniej jeden obraz celu kalibracyjnego
* [ ] Dodano żądane wskaźniki wielospektralne
* [ ] Format eksportu odpowiedni dla Twojego przepływu pracy
* [ ] Skonfigurowano ustawienia PPK (w przypadku korzystania z plików .daq z wydarzeniami ekspozycji)

***

## Kolejne kroki

Po skonfigurowaniu ustawień:

1. **Oznacz obrazy wzorcowe** — zobacz [Wybieranie obrazów wzorcowych](choosing-target-images.md)
2. **Rozpocznij przetwarzanie** — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
3. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)

Aby uzyskać szczegółowe informacje na temat wszystkich dostępnych ustawień, zapoznaj się z dokumentacją referencyjną [Ustawienia projektu](../project-settings/project-settings.md).
