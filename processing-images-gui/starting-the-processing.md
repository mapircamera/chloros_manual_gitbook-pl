# Rozpoczęcie przetwarzania

Po zaimportowaniu obrazów, zaznaczeniu punktów kalibracyjnych i skonfigurowaniu ustawień projektu można rozpocząć przetwarzanie. Ta strona zawiera instrukcje dotyczące uruchomienia procesu przetwarzania Chloros.

## Lista kontrolna przed przetwarzaniem

Przed kliknięciem przycisku „Start” sprawdź, czy wszystko jest gotowe:

* [ ] **Pliki zaimportowane** – wszystkie obrazy są widoczne w przeglądarce plików
* [ ] **Obrazy docelowe zaznaczone** – kolumna „Target” zaznaczona dla obrazów kalibracyjnych (lub zaimportowano zapis `.daq` dla LATTICE)
* [ ] **Wykryto modele kamer** – w kolumnie „Model kamery” wyświetlane są właściwe modele
* [ ] **Skonfigurowano ustawienia** – ustawienia projektu zostały sprawdzone i dostosowane
* [ ] **Wskaźniki wybrane** – Dodano żądane wskaźniki wielospektralne (w razie potrzeby)
* [ ] **Format eksportu wybrany** – Format wyjściowy odpowiedni dla danego przepływu pracy

{% hint style="info" %}
**Wskazówka**: Przed rozpoczęciem przetwarzania kliknij kilka obrazów w przeglądarce plików, aby sprawdzić, czy zostały poprawnie załadowane.
{% endhint %}

***

## Rozpoczęcie przetwarzania

### Znajdź przycisk Start

Przycisk „Start/Odtwórz” znajduje się na górnym pasku nagłówka programu Chloros:

* Położenie: u góry pośrodku okna
* Ikona: **Przycisk „Odtwórz/Start”** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stan: przycisk jest aktywny (podświetlony), gdy program jest gotowy do przetwarzania

### Kliknij, aby rozpocząć

1. Kliknij **przycisk Odtwarzaj/Start** w górnym pasku nagłówka
2. Przetwarzanie rozpocznie się natychmiast
3. Podczas przetwarzania przycisk zmienia się w przycisk **Stop**

4. Pasek postępu aktualizuje się, pokazując stan przetwarzania

{% hint style="success" %}
**Przetwarzanie rozpoczęte**: Po kliknięciu Chloros automatycznie wykonuje wszystkie etapy przetwarzania – wykrywanie obiektów, debayering, kalibrację, obliczanie indeksu oraz eksport. Automatycznie wykrywa, czy projekt jest w formacie Survey3, LATTICE, czy też ich kombinacją, i stosuje odpowiedni proces do każdej kamery.
{% endhint %}

***

## Omówienie trybów przetwarzania

Chloros działa w dwóch różnych trybach przetwarzania w zależności od posiadanej licencji:

### Tryb bezpłatny (przetwarzanie sekwencyjne)

**Dostępny dla wszystkich użytkowników**

**Jak to działa:**

* Przetwarza obrazy pojedynczo, sekwencyjnie
* Działanie jednowątkowe
* Mniejsze zużycie pamięci

**Pasek postępu pokazuje 2 etapy:**

1.**Wykrywanie celów** – skanowanie w poszukiwaniu celów kalibracyjnych
2. **Przetwarzanie** – stosowanie kalibracji i eksportowanie obrazów**Czas przetwarzania:**

* Znacznie wolniejszy niż w trybie równoległym Chloros+
* Odpowiedni dla małych i średnich zbiorów danych (&lt; 200 obrazów)

### Tryb Chloros+ (Przetwarzanie równoległe)

**Wymaga licencji Chloros+**

**Jak to działa:**

* Przetwarza wiele obrazów jednocześnie przy użyciu [4-wątkowego potoku przetwarzania](../processing-architecture/processing-pipeline.md)
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) automatycznie wybiera optymalną strategię dla posiadanego sprzętu na początku działania
* Przyspieszenie GPU (CUDA) z wykorzystaniem kart graficznych NVIDIA (komputery stacjonarne i Jetson)
* **Liczba procesów roboczych dostosowuje się do sprzętu**: strategie wykorzystujące GPU uruchamiają**1–4 równoległe procesy robocze** (skalowane w zależności od pamięci VRAM — Jetson o małej pamięci uruchamia 1, a karta graficzna komputera stacjonarnego o pojemności 12 GB lub większej uruchamia do 4); systemy oparte wyłącznie na procesorze CPU uruchamiają jeden proces roboczy na każdy fizyczny rdzeń, minus jeden**Pasek postępu pokazuje 4 etapy** (odpowiadające 4 wątkom potoku):

1. **Wykrywanie** (wątek 1) – Wyszukiwanie celów kalibracji
2. **Analiza** (wątek 2) – Badanie metadanych obrazu i obliczanie kalibracji
3. **Kalibracja** (wątek 3) — usuwanie efektu bayera, korekcja winietowania, kalibracja, obliczanie indeksu
4. **Eksport** (wątek 4) — zapisywanie przetworzonych obrazów i indeksów**Obsługa paska postępu:*** **Najedź kursorem myszy** na pasek, aby wyświetlić szczegółowy, czterostopniowy panel rozwijany
* **Kliknij** pasek postępu, aby unieruchomić panel rozwijany
* **Kliknij ponownie**, aby odblokować i ukryć panel**Czas przetwarzania:**

* Znacznie szybszy niż w trybie bezpłatnym
* Przyspieszenie GPU dodatkowo zwiększa szybkość

{% hint style="info" %}
**Chloros+ Szybkość**: Przetwarzanie równoległe może być 5–10 razy szybsze niż w trybie sekwencyjnym w przypadku dużych zbiorów danych. Projekt obejmujący 500 obrazów, którego przetworzenie w trybie bezpłatnym zajmuje 2 godziny, może zostać ukończony w 15–20 minut przy użyciu Chloros+.
{% endhint %}

***

## Co dzieje się podczas przetwarzania

### Etap 1: Wykrywanie obiektów docelowych

**Jak działa Chloros:**

* Skanuje zdjęcia zaznaczone w kolumnie „Cel” (wszystkie zdjęcia, jeśli żadne nie jest zaznaczone)
* Identyfikuje panele kalibracyjne w każdym celu
* Wyodrębnia wartości odbicia światła z paneli celowych
* Rejestruje znaczniki czasu celów na potrzeby planowania kalibracji

**Czas trwania:** 1–30 sekund (w przypadku zaznaczonych celów), 5–30+ minut (w przypadku niezaznaczonych)

### Etap 2: Debayering (konwersja plików RAW)

**Czym zajmuje się Chloros:**

* Konwertuje dane RAW w układzie Bayera na pełne obrazy 3-kanałowe (moduły LATTICE mono pozostają jednopasmowe — w ich przypadku proces demosaicingu jest pomijany, co odnotowuje się w dzienniku)
* Stosuje wybrany algorytm demosaicingu
* Zachowuje maksymalną jakość obrazu i szczegółowość

**Czas trwania:** Zależy od liczby obrazów oraz szybkości procesora (CPU) lub karty graficznej (GPU)

### Etap 3: Kalibracja

**Czym zajmuje się Chloros:*** **Korekcja winietowania**: usuwa przyciemnienie na krawędziach obrazu spowodowane obiektywem
* **Kalibracja odbicia**: normalizuje obraz przy użyciu docelowych wartości odbicia i/lub danych DAQ dotyczących promieniowania padającego
* Stosuje korekty we wszystkich pasmach/kanałach
* Wykorzystuje odpowiedni wzorzec kalibracyjny dla każdego obrazu na podstawie sygnatury czasowej

**Czas trwania:** Większość czasu przetwarzania

### Etap 4: Obliczanie indeksów

**Czym zajmuje się Chloros:**

* Oblicza skonfigurowane indeksy wielospektralne (NDVI, NDRE itp.)
* Stosuje operacje matematyczne na pasmach do skalibrowanych obrazów
* Generuje obrazy indeksowe dla każdego wybranego indeksu

**Czas trwania:** Kilka sekund na obraz

### Etap 5: Eksport

**Czym zajmuje się Chloros:**

* Zapisuje przetworzone obrazy w wybranym formacie
* **Rozgałęzienie LATTICE**: każda surowa klatka LATTICE jest eksportowana jako każdy włączony produkt w jednym przebiegu — bez efektu bayera, podgląd, promieniowanie (zawsze float32), odbicie
* Zapisuje pliki do drzewa wyjściowego projektu: `<project>/<camera>/<format>/<Product>_Images/`
* **Zachowuje nazwę pliku źródłowego** — folder identyfikuje produkt, nie jest dodawany żaden sufiks**Czas trwania:** Zależy od formatu eksportu i rozmiaru pliku***

## Sposób przetwarzania

### Automatyczny potok przetwarzania

Po uruchomieniu cały potok działa automatycznie:

* Nie jest wymagana interakcja użytkownika
* Wszystkie skonfigurowane etapy są wykonywane po kolei
* Aktualizacje postępu są wyświetlane w czasie rzeczywistym
* Eksportowane pliki są zapisywane na dysku po zakończeniu poszczególnych etapów — gotowe pliki wyjściowe można otwierać, podczas gdy proces nadal trwa

### Wykorzystanie komputera podczas przetwarzania

**Tryb swobodny:**

* Stosunkowo niskie obciążenie procesora (jednowątkowe)
* Komputer pozostaje responsywny dla innych zadań
* Można bezpiecznie zminimalizować okno Chloros i pracować w innych aplikacjach

**Tryb równoległy Chloros+:**

* Wysokie obciążenie procesora w całej puli procesorów strategii
* Z przyspieszeniem GPU: wysokie obciążenie karty graficznej
* Komputer może reagować wolniej podczas przetwarzania
* Należy unikać uruchamiania innych zadań wymagających dużej mocy obliczeniowej procesora

{% hint style="warning" %}
**Wskazówka dotycząca wydajności**: Aby uzyskać najlepszą wydajność Chloros+, zamknij inne aplikacje i pozwól Chloros korzystać z pełnych zasobów systemowych.
{% endhint %}

### Przetwarzania nie można wstrzymać (ale zatrzymanie jest całkowite)

* Po uruchomieniu przetwarzania nie można go wstrzymać, a następnie wznowić
* Kliknięcie przycisku **Stop** całkowicie zatrzymuje przebieg już przy pierwszym kliknięciu
* Produkty wyeksportowane przed zatrzymaniem pozostają na dysku
* Zatrzymany przebieg rzetelnie informuje o tym, co zostało ukończone (zobacz wiersze `[RUN-SUMMARY]` w dzienniku)
* Nowe uruchomienie rozpoczyna potok od początku

**Wskazówka dotycząca planowania:** W przypadku bardzo dużych projektów warto rozważyć przetwarzanie partiami lub użycie CLI w celu uzyskania lepszej kontroli.***

## Monitorowanie przetwarzania

Podczas trwania przetwarzania można:

* **Obserwować pasek postępu** — sprawdzić ogólny procent zaawansowania
* **Wyświetlić bieżący etap** — wykrywanie, analiza, kalibracja lub eksport
* **Sprawdzić kartę dziennika** — wyświetlić szczegółowe komunikaty i ostrzeżenia dotyczące przetwarzania
* **Wyświetlić podgląd gotowych obrazów** — pliki eksportowe pojawiają się na dysku w trakcie przetwarzania

Szczegółowe informacje na temat monitorowania można znaleźć w sekcji [Monitorowanie przetwarzania](monitoring-the-processing.md).

***

## Zatrzymanie przetwarzania

Jeśli konieczne jest zatrzymanie przetwarzania:

### Jak zatrzymać

1. Znajdź **przycisk Zatrzymaj** (podczas przetwarzania zastępuje przycisk Start)
2. Kliknij go raz — na pasku pojawi się komunikat **„Zatrzymywanie...”**, podczas gdy obraz w trakcie przetwarzania zostanie ukończony
3. Przebieg kończy się w stanie ostatecznego zatrzymania, a w dzienniku wyświetla się rzetelny raport `[RUN-SUMMARY]` dotyczący tego, co zostało wykonane

### Kiedy należy zatrzymać

**Uzasadnione powody zatrzymania:**

* Stwierdzono, że zastosowano nieprawidłowe ustawienia
* Zapomniano oznaczyć docelowe obrazy
* Zaimportowano niewłaściwe obrazy
* System działa zbyt wolno lub nie reaguje

**Po zatrzymaniu:**

* Produkty wyeksportowane przed zatrzymaniem pozostają na dysku
* Sprawdź i napraw wszelkie problemy, dostosuj ustawienia w razie potrzeby
* Uruchom przetwarzanie ponownie — proces rozpocznie się od początku

***

## Szacunkowy czas przetwarzania

Rzeczywisty czas przetwarzania różni się znacznie w zależności od:

* Liczby obrazów
* Rozdzielczości obrazów
* Formatu wejściowego (RAW lub JPG)
* Trybu przetwarzania (Free lub Chloros+)
* Szybkości procesora i liczby rdzeni
* Dostępność procesora graficznego (tylko Chloros+)
* Liczba indeksów do obliczenia
* Liczba włączonych produktów eksportowych (LATTICE)

### Przybliżone szacunki (Chloros+, zdjęcia 12 MP, nowoczesny procesor)

| Liczba obrazów | Tryb Free | Chloros+ (procesor) | Chloros+ (karta graficzna) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrazów   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 zdjęć  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 zdjęć  | 1–1,5 godz. | 20–30 min      | 10–15 min      |
| 500 zdjęć  | 2–3 godz.   | 45–60 min      | 20–30 min      |
| 1000 zdjęć | 4–6 godz.   | 1,5–2 godz.      | 40–60 min      |

{% hint style="info" %}
**Pierwsze uruchomienie**: Wstępne przetwarzanie może potrwać dłużej, ponieważ Chloros tworzy pamięć podręczną i profile. Kolejne przetwarzanie podobnych zestawów danych będzie przebiegało szybciej.
{% endhint %}

***

## Typowe problemy przy uruchomieniu

### Przycisk „Start” jest nieaktywny (wyszarzony)

**Możliwe przyczyny:**

* Nie zaimportowano żadnych obrazów
* Moduł zaplecza nie został w pełni uruchomiony
* Trwa przetwarzanie poprzedniego zadania
* Projekt nie został w pełni załadowany

**Rozwiązania:**

1. Poczekaj, aż backend w pełni się zainicjuje (sprawdź ikonę w menu głównym)
2. Sprawdź, czy obrazy zostały zaimportowane w przeglądarce plików
3. Uruchom ponownie Chloros, jeśli przycisk pozostaje nieaktywny
4. Sprawdź dziennik debugowania pod kątem komunikatów o błędach

### Przetwarzanie rozpoczyna się, a następnie natychmiast kończy się niepowodzeniem

**Możliwe przyczyny:**

* Brak prawidłowych obrazów w projekcie
* Uszkodzone pliki obrazów
* Niewystarczająca ilość miejsca na dysku
* Niewystarczająca ilość pamięci (RAM)

**Rozwiązania:**

1. Sprawdź dziennik debugowania <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pod kątem komunikatów o błędach
2. Sprawdź, czy jest wystarczająca ilość miejsca na dysku
3. Spróbuj przetworzyć mniejszy podzbiór obrazów
4. Sprawdź, czy obrazy nie są uszkodzone

### Przetwarzanie kończy się, ale nie zapisuje żadnych obrazów

Przebieg, w którym zażądano produktów obrazowych, ale nie zapisano żadnego, jest traktowany jako **niepowodzenie, a nie sukces** — Chloros wyraźnie to sygnalizuje:

* W dzienniku GUI pojawia się komunikat `[RUN-SUMMARY]` wskazujący prawdopodobną przyczynę — brak zaimportowanych obrazów, brak wykrytego celu lub pominięcie wszystkich żądanych produktów jako nieodpowiednich (np. żądanie danych dotyczących promieniowania/odbicia z kamer obsługujących wyłącznie RGB)
* Odpowiednik CLI (`chloros-cli process`) wyświetla komunikat `Processing finished but wrote no image products.` i **kończy działanie z wynikiem niezerowym**, dzięki czemu skrypty mogą to wykryć
* Celowe uruchomienie wyłącznie w celu uzyskania metadanych (wszystkie produkty eksportu wyłączone, brak indeksów) nadal jest traktowane jako sukces

Pełną semantykę można znaleźć w [dokumencie referencyjnym CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Ostrzeżenie „Nie wykryto celów”

**Możliwe przyczyny:**

* Zapomniano oznaczyć obrazy docelowe
* Obrazy docelowe nie zawierają widocznych celów
* Ustawienia wykrywania celów są zbyt rygorystyczne

**Rozwiązania:**

1. Zapoznaj się z sekcją [Wybór obrazów docelowych](choosing-target-images.md)
2. Zaznacz odpowiednie obrazy w kolumnie „Cel”
3. Sprawdź, czy cele są widoczne na zaznaczonych obrazach
4. W razie potrzeby dostosuj ustawienia wykrywania obiektów

***

## Wskazówki dotyczące pomyślnego przetwarzania

### Przed rozpoczęciem

1. **Najpierw przetestuj na małym podzbiorze** – przetwórz 10–20 obrazów, aby zweryfikować ustawienia
2. **Sprawdź dostępną przestrzeń dyskową** – upewnij się, że masz 2–3 razy więcej wolnego miejsca niż rozmiar zbioru danych (więcej, jeśli włączone są wszystkie produkty LATTICE)
3. **Zamknij zbędne aplikacje** – zwolnij zasoby systemowe
4. **Sprawdź obrazy celów** – wyświetl podgląd zaznaczonych celów, aby upewnić się co do jakości
5. **Zapisz projekt** – projekt zapisuje się automatycznie, ale dobrą praktyką jest ręczne zapisywanie

### Podczas przetwarzania

1. **Unikaj uśpienia systemu** – Wyłącz tryby oszczędzania energii
2. **Utrzymuj program Chloros na pierwszym planie** – Lub przynajmniej widoczny na pasku zadań
3. **Od czasu do czasu sprawdzaj postęp** – Sprawdź, czy nie ma ostrzeżeń lub błędów
4. **Nie uruchamiaj innych aplikacji wymagających dużej mocy obliczeniowej** – zwłaszcza w trybie równoległym Chloros+

### Przyspieszenie GPU w Chloros+

W przypadku korzystania z przyspieszenia GPU firmy NVIDIA:

1. Zaktualizuj sterowniki NVIDIA do najnowszej wersji
2. Upewnij się, że karta graficzna ma co najmniej 4 GB pamięci VRAM (co najmniej 7 GB w przypadku równoczesnego odtwarzania tekstur z wykorzystaniem algorytmu Texture Aware)
3. Zamknij aplikacje intensywnie obciążające kartę graficzną (gry, edycja wideo)
4. Monitoruj temperaturę karty graficznej (zapewnij odpowiednie chłodzenie)

***

## Kolejne kroki

Po rozpoczęciu przetwarzania:

1. **Monitoruj postęp** – zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)
2. **Poczekaj na zakończenie** – przetwarzanie przebiega automatycznie
3. **Sprawdź wyniki** - Zobacz [Zakończenie przetwarzania](finishing-the-processing.md)

Aby uzyskać informacje na temat tego, co należy robić podczas przetwarzania, zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md).
