# Dostosowywanie ustawień projektu

Przed przetworzeniem obrazów należy skonfigurować ustawienia projektu zgodnie z wymaganiami przepływu pracy. Panel ustawień projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> zapewnia kompleksową kontrolę nad kalibracją, opcjami przetwarzania, indeksami wielospektralnymi i formatami eksportu.

## Dostęp do ustawień projektu

1. Otwórz projekt w Chloros
2. Kliknij ikonę **Ustawienia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. Panel Ustawienia projektu wyświetla wszystkie opcje konfiguracyjne

{% hint style=&quot;info&quot; %}
**Ustawienia są zapisywane automatycznie** wraz z projektem. Po ponownym otwarciu projektu wszystkie ustawienia zostaną przywrócone.
{% endhint %}

***

## Szybka konfiguracja dla typowych procesów roboczych

### Ustawienia domyślne (zalecane dla większości użytkowników)

W przypadku typowych procesów pracy z kamerą MAPIR Survey3 dobrze sprawdzają się ustawienia domyślne:

* ✅ **Korekcja winietowania**: włączona
* ✅ **Kalibracja odbicia**: włączona (wymaga obrazów celów MAPIR)
* ✅ **Metoda debayeringu**: Wysoka jakość (szybsza)
* ✅ **Format eksportu**: TIFF (16-bitowy)

Wystarczy zaimportować obrazy i rozpocząć przetwarzanie przy użyciu tych ustawień domyślnych.

***

## Omówienie ustawień projektu

Panel ustawień projektu jest podzielony na kilka kategorii. Poniżej znajduje się podsumowanie każdej sekcji. Pełna dokumentacja znajduje się w sekcji [Ustawienia projektu](../project-settings/project-settings.md).

### Wykrywanie obiektów docelowych

Kontroluje sposób, w jaki Chloros identyfikuje obiekty kalibracyjne na obrazach.

**Kluczowe ustawienia:**

* **Minimalny obszar próbki kalibracji**: próg rozmiaru dla wykrywania celów (domyślnie: 25 pikseli)
* **Minimalne grupowanie celów**: próg podobieństwa dla grupowania obszarów docelowych (domyślnie: 60)

**Kiedy dostosować:**

* Zwiększ obszar próbki, jeśli pojawiają się fałszywe wykrycia.
* Zmniejsz, jeśli cele nie są wykrywane.
* Dostosuj grupowanie, jeśli cele są dzielone na wiele wykryć.

### Przetwarzanie

Główne opcje przetwarzania obrazu i kalibracji.

**Kluczowe ustawienia:**

* **Korekcja winietowania**: kompensuje zaciemnienie obiektywu na krawędziach ✅ Zalecane
* **Kalibracja odbicia**: normalizuje wartości przy użyciu celów kalibracyjnych ✅ Zalecane
* **Metoda Debayera**: algorytm konwersji RAW do 3-kanałowego obrazu wielospektralnego
* **Minimalny interwał ponownej kalibracji**: czas między użyciem celów kalibracyjnych (0 = użyj wszystkich)

**Ustawienia zaawansowane:**

* **Przesunięcie strefy czasowej czujnika światła**: do synchronizacji czasu PPK (domyślnie: 0)
* **Zastosuj korekty PPK**: wykorzystuje dane GPS/pin ekspozycji z plików .daq
* **Pin ekspozycji 1/2**: przypisuje kamery do pinów ekspozycji w przypadku konfiguracji z dwoma kamerami

### Indeks (indeksy wielospektralne)

Skonfiguruj, które indeksy roślinności mają być obliczane i eksportowane.

**Jak dodać indeksy:**

1. Kliknij przycisk **„Dodaj indeks”**
2. Wybierz indeks z menu rozwijanego (NDVI, NDRE, GNDVI itp.)
3. Skonfiguruj ustawienia wizualizacji (kolory LUT, zakresy wartości)
4. Dodaj wiele indeksów w razie potrzeby

**Popularne indeksy:**

* **NDVI**: Ogólny stan zdrowia roślinności (najczęściej stosowany)
* **NDRE**: Wczesne wykrywanie stresu za pomocą RedEdge
* **GNDVI**: Wrażliwy na stężenie chlorofilu
* **OSAVI**: Działa dobrze w przypadku widocznej gleby
* **EVI**: Regiony o wysokim wskaźniku powierzchni liści (LAI)

**Formuły niestandardowe (tylko Chloros+):**

* Tworzenie niestandardowych formuł indeksów wielospektralnych
* Wykorzystanie matematyki pasmowej we wszystkich kanałach obrazu
* Zapisywanie niestandardowych formuł do ponownego wykorzystania

Wszystkie dostępne indeksy i formuły można znaleźć w sekcji [Formuły indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md).

### Eksport

Kontroluje format i jakość pliku wyjściowego.

**Dostępne formaty:**

* **TIFF (16-bitowy)**: zalecany do GIS i analiz naukowych (zakres 0–65 535)
* **TIFF (32-bitowy, procentowy)**: wartości odbicia w systemie zmiennoprzecinkowym (zakres 0,0–1,0)
* **PNG (8-bitowy)**: bezstratna kompresja do wizualizacji (zakres 0–255)
* **JPG (8-bitowy)**: najmniejsze pliki, kompresja stratna (zakres 0–255)

***

## Zapisywanie i wczytywanie ustawień

### Zapisywanie szablonu projektu

Utwórz szablony wielokrotnego użytku, aby zapewnić spójność przepływu pracy:

1. Skonfiguruj wszystkie żądane ustawienia w panelu Ustawienia projektu.
2. Przewiń do sekcji **„Zapisz szablon projektu”** u dołu.
3. Wprowadź opisową nazwę szablonu (np. „Survey3N\_RGN\_Agriculture”).
4. Kliknij ikonę zapisu.

**Korzyści:**

* Zastosuj identyczne ustawienia w wielu projektach.
* Udostępniaj konfiguracje członkom zespołu.
* Zachowaj spójność w przypadku powtarzających się ankiet.

### Wczytaj szablon do nowego projektu

Podczas tworzenia nowego projektu:

1. Wybierz **„Nowy projekt”** z menu głównego.
2. Wybierz opcję **„Wczytaj z szablonu”**.
3. Wybierz zapisany szablon.
4. Wszystkie ustawienia zostaną automatycznie zastosowane.

### Katalog roboczy

Ustawienie **„Zapisz folder projektu”** określa domyślną lokalizację tworzenia nowych projektów:

* **Domyślna lokalizacja**: `C:\Users\[Username]\Chloros Projects`
* **Zmień lokalizację**: kliknij ikonę edycji i wybierz nowy folder
* **Kiedy zmienić**:
  * Dysk sieciowy do współpracy zespołowej
  * Inny dysk z większą przestrzenią dyskową
  * Uporządkowana struktura folderów według roku/klienta

***

## Konfiguracja PPK (Post-Processed Kinematic)

W przypadku korzystania z rejestratorów MAPIR DAQ z GPS w celu precyzyjnej geolokalizacji:

### Wymagania wstępne

* MAPIR DAQ z modułem GPS (GNSS)
* Plik dziennika .daq z wpisami pinów ekspozycji
* Kamera podłączona do pinów ekspozycji DAQ podczas sesji przechwytywania

### Kroki konfiguracji

1. Umieść plik dziennika .daq w folderze projektu.
2. W ustawieniach projektu zaznacz pole wyboru **„Zastosuj korekty PPK”**.
3. W razie potrzeby ustaw **„Przesunięcie strefy czasowej czujnika światła”** (domyślnie: 0 dla UTC).
4. Przypisz kamery do pinów ekspozycji:
   * **Pojedyncza kamera**: Automatycznie przypisana do pinu 1
   * **Dwie kamery**: Ręcznie przypisz każdą kamerę do odpowiedniego pinu

**Przypisanie pinów ekspozycji:**

* **Pin ekspozycji 1**: Wybierz model kamery z listy rozwijanej
* **Pin ekspozycji 2**: Wybierz drugą kamerę lub „Nie używaj”
* Ta sama kamera nie może być przypisana do obu pinów

{% hint style=&quot;warning&quot; %}
**Ważne**: Piny ekspozycji muszą być prawidłowo przypisane do odpowiednich kamer. Nieprawidłowe przypisanie spowoduje wygenerowanie błędnych danych geolokalizacyjnych.
{% endhint %}

***

## Zaawansowane scenariusze

### Projekty z wieloma kamerami

Podczas przetwarzania obrazów z wielu kamer MAPIR w jednym projekcie:

1. Chloros automatycznie wykrywa każdy model kamery
2. Każda kamera otrzymuje odpowiedni profil przetwarzania
3. PPK: ręcznie przypisz każdą kamerę do odpowiedniego pinów ekspozycji
4. Wszystkie kamery używają tego samego formatu eksportu i indeksów

**Przykład**: Survey3W RGN + Survey3N OCN zestaw dwóch kamer

### Badania poklatkowe lub wielodniowe

W przypadku powtarzających się badań tego samego obszaru w czasie:

1. Utwórz szablon ze standardowymi ustawieniami.
2. Używaj spójnej konfiguracji celu kalibracji w każdej sesji.
3. Przetwarzaj każdą datę jako oddzielny projekt.
4. Używaj identycznych ustawień, aby uzyskać porównywalne wyniki.
5. Eksportuj w tym samym formacie do analizy czasowej.

### Duże zbiory danych

W przypadku projektów zawierających wiele obrazów (ponad 500):

* Rozważ podzielenie projektu na mniejsze projekty według daty lub obszaru.
* Użyj przetwarzania równoległego Chloros+, aby uzyskać szybsze wyniki.
* Rozważ użycie CLI lub API do automatyzacji przetwarzania wsadowego.
* Dostosuj minimalny interwał ponownej kalibracji, aby skrócić czas wykrywania celu.

***

## Sprawdzanie ustawień

Przed rozpoczęciem przetwarzania sprawdź następujące kluczowe ustawienia:

* [ ] Model aparatu prawidłowo wykryty w przeglądarce plików
* [ ] Włączona korekcja winietowania
* [ ] Włączona kalibracja odbicia
* [ ] Zaimportowano co najmniej jeden obraz kalibracyjny
* [ ] Dodano żądane indeksy wielospektralne
* [ ] Format eksportu odpowiedni dla Twojego przepływu pracy
* [ ] Skonfigurowano ustawienia PPK (w przypadku korzystania z .daq z wydarzeniami ekspozycji)

***

## Kolejne kroki

Po skonfigurowaniu ustawień:

1. **Zaznacz obrazy docelowe kalibracji** — zobacz [Wybór obrazów docelowych](choosing-target-images.md)
2. **Rozpocznij przetwarzanie** — zobacz [Rozpoczęcie przetwarzania](starting-the-processing.md)
3. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)

Pełne informacje na temat wszystkich dostępnych ustawień można znaleźć w dokumentacji referencyjnej [Ustawienia projektu](../project-settings/project-settings.md).
