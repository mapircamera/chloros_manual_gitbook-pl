# Rozpoczęcie przetwarzania

Po zaimportowaniu obrazów, zaznaczeniu punktów kalibracyjnych i skonfigurowaniu ustawień projektu można przystąpić do przetwarzania. Niniejsza strona zawiera instrukcje dotyczące uruchomienia potoku przetwarzania Chloros.

## Lista kontrolna przed przetwarzaniem

Przed kliknięciem przycisku Start sprawdź, czy wszystko jest gotowe:

* [ ] **Pliki zaimportowane** – wszystkie obrazy pojawiają się w przeglądarce plików
* [ ] **Obrazy docelowe zaznaczone** – kolumna „Target” zaznaczona dla obrazów kalibracyjnych
* [ ] **Modele kamer wykryte** – kolumna „Camera Model” pokazuje prawidłowe kamery
* [ ] **Ustawienia skonfigurowane** – ustawienia projektu sprawdzone i dostosowane
* [ ] **Wskaźniki wybrane** — dodano żądane wskaźniki wielospektralne (w razie potrzeby)
* [ ] **Format eksportu wybrany** — format wyjściowy odpowiedni dla Twojego przepływu pracy

{% hint style="info" %}
**Wskazówka**: Przejrzyj kilka obrazów w przeglądarce plików, aby sprawdzić, czy zostały one poprawnie załadowane przed przetwarzaniem.
{% endhint %}

***

## Rozpoczęcie przetwarzania

### Znajdź przycisk Start

Przycisk Start/Odtwórz znajduje się w górnym pasku nagłówka Chloros:

* Położenie: górna środkowa część okna
* Ikona: **Przycisk Odtwórz/Start** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: Przycisk jest aktywny (podświetlony), gdy program jest gotowy do przetwarzania

### Kliknij, aby rozpocząć

1. Kliknij **przycisk Odtwarzaj/Start** w górnym pasku nagłówka
2. Przetwarzanie rozpocznie się natychmiast
3. Podczas przetwarzania przycisk staje się nieaktywny (wyszarzony)
4. Pasek postępu aktualizuje się, pokazując status przetwarzania

{% hint style="success" %}
**Rozpoczęto przetwarzanie**: Po kliknięciu Chloros automatycznie wykonuje wszystkie etapy przetwarzania — wykrywanie obiektów, debayering, kalibrację, obliczanie indeksu i eksport.
{% endhint %}

***

## Zrozumienie trybów przetwarzania

Chloros działa w dwóch różnych trybach przetwarzania w zależności od posiadanej licencji:

### Tryb darmowy (przetwarzanie sekwencyjne)

**Dostępny dla wszystkich użytkowników**

**Jak to działa:**

* Przetwarza obrazy pojedynczo, sekwencyjnie
* Działanie jednowątkowe
* Niższe zużycie pamięci

**Pasek postępu pokazuje 2 etapy:**

1.**Wykrywanie celów** – skanowanie w poszukiwaniu celów kalibracyjnych
2. **Przetwarzanie** – stosowanie kalibracji i eksportowanie obrazów**Czas przetwarzania:**

* Znacznie wolniejsze niż w trybie równoległym Chloros+
* Odpowiednie dla małych i średnich zbiorów danych (&lt; 200 obrazów)

### Tryb Chloros+ (przetwarzanie równoległe)

**Wymaga licencji Chloros+**

**Jak to działa:**

* Przetwarza wiele obrazów jednocześnie przy użyciu [4-wątkowego potoku przetwarzania](../processing-architecture/processing-pipeline.md)
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) automatycznie wybiera optymalną strategię dla posiadanego sprzętu
* Przyspieszenie GPU (CUDA) z kartami graficznymi NVIDIA (komputery stacjonarne i Jetson)
* Skalowalność od Jetson Nano (1 procesor roboczy) do komputera stacjonarnego z GPU o pojemności 12 GB+ (3–4 procesory robocze)

**Pasek postępu pokazuje 4 etapy** (odpowiadające 4 wątkom potoku):

1. **Wykrywanie** (Wątek 1) – Wyszukiwanie celów kalibracji
2. **Analiza** (Wątek 2) – Badanie metadanych obrazu i obliczanie kalibracji
3. **Kalibracja** (Wątek 3) – Debayering GPU, korekcja winietowania, obliczanie indeksu
4. **Eksportowanie** (wątek 4) – zapisywanie przetworzonych obrazów i indeksów**Interakcja z paskiem postępu:*** **Najedź myszką** na pasek, aby wyświetlić szczegółowy panel rozwijany z 4 etapami
* **Kliknij** pasek postępu, aby zatrzymać panel rozwijany w miejscu
* **Kliknij ponownie**, aby odblokować i ukryć panel**Czas przetwarzania:**

* Znacznie szybszy niż w trybie darmowym
* Skalowalny w zależności od liczby rdzeni procesora
* Przyspieszenie GPU dodatkowo zwiększa szybkość

{% hint style="info" %}
**Chloros+ Szybkość**: Przetwarzanie równoległe może być 5–10 razy szybsze niż tryb sekwencyjny w przypadku dużych zbiorów danych. Projekt obejmujący 500 obrazów, który w trybie darmowym zajmuje 2 godziny, może zostać ukończony w 15–20 minut przy użyciu Chloros+.
{% endhint %}

***

## Co dzieje się podczas przetwarzania

### Etap 1: Wykrywanie celów

**Co robi Chloros:**

* Skanuje zaznaczone obrazy celów (lub wszystkie obrazy, jeśli żadne nie są zaznaczone)
* Identyfikuje 4 panele kalibracyjne w każdym celu
* Wyodrębnia wartości odbicia z paneli celów
* Rejestruje znaczniki czasu celów do planowania kalibracji

**Czas trwania:** 1–30 sekund (z zaznaczonymi celami), 5–30+ minut (bez zaznaczenia)

### Etap 2: Debayering (konwersja RAW)

**Czym zajmuje się Chloros:**

* Konwertuje dane w formacie RAW z wzorem Bayera na pełne obrazy RGB
* Stosuje wysokiej jakości algorytm demosaicingu
* Zachowuje maksymalną jakość obrazu i szczegóły

**Czas trwania:** Zależy od liczby obrazów i szybkości procesora

### Etap 3: Kalibracja

**Czym zajmuje się Chloros:*** **Korekcja winietowania**: usuwa zaciemnienie obiektywu na krawędziach
* **Kalibracja odbicia**: normalizuje przy użyciu docelowych wartości odbicia
* Stosuje korekty we wszystkich pasmach/kanałach
* Wykorzystuje odpowiedni wzorzec kalibracyjny dla każdego obrazu na podstawie sygnatury czasowej

**Czas trwania:** większość czasu przetwarzania

### Etap 4: Obliczanie indeksów

**Co robi Chloros:**

* Oblicza skonfigurowane indeksy wielospektralne (NDVI, NDRE itp.)
* Stosuje operacje matematyczne na pasmach do skalibrowanych obrazów
* Generuje obrazy indeksowe dla każdego wybranego indeksu

**Czas trwania:** Kilka sekund na obraz

### Etap 5: Eksport

**Co robi Chloros:**

* Zapisuje skalibrowane obrazy w wybranym formacie
* Eksportuje obrazy indeksowe z skonfigurowanymi kolorami LUT
* Zapisuje pliki do podfolderów modeli kamer
* Zachowuje oryginalne nazwy plików z rozszerzeniami

**Czas trwania:** Zależy od formatu eksportu i rozmiaru pliku***

## Zachowanie podczas przetwarzania

### Automatyczny potok przetwarzania

Po uruchomieniu cały potok działa automatycznie:

* Nie jest wymagana interakcja użytkownika
* Wszystkie skonfigurowane kroki są wykonywane po kolei
* Aktualizacje postępu wyświetlane w czasie rzeczywistym

### Wykorzystanie komputera podczas przetwarzania

**Tryb swobodny:**

* Stosunkowo niskie obciążenie procesora (jednowątkowe)
* Komputer pozostaje responsywny dla innych zadań
* Można bezpiecznie zminimalizować Chloros i pracować w innych aplikacjach

**Chloros+ Tryb równoległy:**

* Wysokie obciążenie procesora (wielowątkowe, do 16 rdzeni)
* Z przyspieszeniem GPU: Wysokie obciążenie GPU
* Komputer może być mniej responsywny podczas przetwarzania
* Należy unikać uruchamiania innych zadań wymagających dużej mocy obliczeniowej procesora

{% hint style="warning" %}
**Wskazówka dotycząca wydajności**: Aby uzyskać najlepszą wydajność Chloros+, należy zamknąć inne aplikacje i pozwolić Chloros na wykorzystanie pełnych zasobów systemowych.
{% endhint %}

### Przetwarzania nie można wstrzymać

**Ważne ograniczenia:**

* Po uruchomieniu przetwarzania nie można go wstrzymać
* Można anulować przetwarzanie, ale postęp zostanie utracony
* Częściowe wyniki nie są zapisywane
* W przypadku anulowania należy rozpocząć od początku

**Wskazówka dotycząca planowania:** W przypadku bardzo dużych projektów rozważ przetwarzanie partiami lub użycie CLI w celu uzyskania lepszej kontroli.***

## Monitorowanie przetwarzania

Podczas przetwarzania można:

* **Obserwować pasek postępu** — sprawdzić ogólny procent ukończenia
* **Wyświetlić bieżący etap** — wykrywanie, analiza, kalibracja lub eksport
* **Sprawdzić kartę dziennika** — wyświetlić szczegółowe komunikaty i ostrzeżenia dotyczące przetwarzania
* **Wyświetlić podgląd ukończonych obrazów** — niektóre pliki eksportowe mogą pojawić się podczas przetwarzania

Szczegółowe informacje na temat monitorowania można znaleźć w sekcji [Monitorowanie przetwarzania](monitoring-the-processing.md).

***

## Anulowanie przetwarzania

Jeśli konieczne jest zatrzymanie przetwarzania:

### Jak anulować

1. Znajdź przycisk **Stop/Anuluj** (zastępuje przycisk Start podczas przetwarzania)
2. Kliknij przycisk Stop
3. Przetwarzanie zostanie natychmiast wstrzymane
4. Częściowe wyniki zostaną odrzucone

### Kiedy anulować

**Uzasadnione powody anulowania:**

* Zauważono, że użyto nieprawidłowych ustawień
* Zapomniano zaznaczyć obrazy docelowe
* Zaimportowano niewłaściwe obrazy
* System działa zbyt wolno lub nie reaguje

**Po anulowaniu:**

* Sprawdź i napraw wszelkie problemy
* Dostosuj ustawienia w razie potrzeby
* Uruchom przetwarzanie od początku
* Aby uzyskać najlepsze wrażenia, całkowicie zamknij Chloros i uruchom ponownie

{% hint style="warning" %}
**Brak częściowych wyników**: Anulowanie powoduje odrzucenie wszystkich dotychczasowych postępów. Chloros nie zapisuje częściowo przetworzonych obrazów.
{% endhint %}

***

## Szacunkowy czas przetwarzania

Rzeczywisty czas przetwarzania różni się znacznie w zależności od:

* Liczby obrazów
* Rozdzielczości obrazów
* Formatu wejściowego RAW lub JPG
* Trybu przetwarzania (Free lub Chloros+)
* Szybkości procesora i liczby rdzeni
* Dostępności procesora graficznego (tylko Chloros+)
* Liczba indeksów do obliczenia
* Złożoność formatu eksportu

### Przybliżone szacunki (Chloros+, zdjęcia 12 MP, nowoczesny procesor)

| Liczba zdjęć | Tryb Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrazów   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 zdjęć  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 zdjęć  | 1–1,5 godz. | 20–30 min      | 10–15 min      |
| 500 obrazów  | 2–3 godz.   | 45–60 min      | 20–30 min      |
| 1000 obrazów | 4–6 godz.   | 1,5–2 godz.      | 40–60 min      |

{% hint style="info" %}
**Pierwsze uruchomienie**: Wstępne przetwarzanie może potrwać dłużej, ponieważ Chloros tworzy pamięć podręczną i profile. Kolejne przetwarzanie podobnych zestawów danych będzie przebiegało szybciej.
{% endhint %}

***

## Typowe problemy przy uruchamianiu

### Przycisk Start jest nieaktywny (wyszarzony)

**Możliwe przyczyny:**

* Brak zaimportowanych obrazów
* Backend nie został w pełni uruchomiony
* Poprzednie przetwarzanie nadal trwa
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
2. Sprawdź dostępną przestrzeń dyskową
3. Spróbuj przetworzyć mniejszy podzbiór obrazów
4. Sprawdź, czy obrazy nie są uszkodzone

### Ostrzeżenie „Nie wykryto celów”

**Możliwe przyczyny:**

* Zapomniano zaznaczyć obrazy docelowe
* Obrazy docelowe nie zawierają widocznych celów
* Ustawienia wykrywania celów są zbyt rygorystyczne

**Rozwiązania:**

1. Zapoznaj się z sekcją [Wybór obrazów docelowych](choosing-target-images.md)
2. Zaznacz odpowiednie obrazy w kolumnie „Cel”
3. Sprawdź, czy cele są widoczne na zaznaczonych obrazach
4. W razie potrzeby dostosuj ustawienia wykrywania celów

***

## Wskazówki dotyczące pomyślnego przetwarzania

### Przed rozpoczęciem

1. **Najpierw przetestuj na małym podzbiorze** — przetwórz 10–20 obrazów, aby zweryfikować ustawienia
2. **Sprawdź dostępną przestrzeń dyskową** — upewnij się, że masz 2–3 razy więcej wolnego miejsca niż rozmiar zbioru danych
3. **Zamknij niepotrzebne aplikacje** — zwolnij zasoby systemowe
4. **Zweryfikuj obrazy docelowe** — wyświetl podgląd zaznaczonych celów, aby upewnić się co do jakości
5. **Zapisz projekt** – projekt zapisuje się automatycznie, ale warto zapisać go ręcznie

### Podczas przetwarzania

1. **Unikaj uśpienia systemu** – wyłącz tryby oszczędzania energii
2. **Trzymaj Chloros na pierwszym planie** – lub przynajmniej widoczny na pasku zadań
3. **Od czasu do czasu monitoruj postęp** – sprawdzaj, czy nie ma ostrzeżeń lub błędów
4. **Nie uruchamiaj innych wymagających aplikacji** – zwłaszcza w trybie równoległym Chloros+

### Przyspieszenie GPU Chloros+

Jeśli korzystasz z przyspieszenia GPU NVIDIA:

1. Zaktualizuj sterowniki NVIDIA do najnowszej wersji
2. Upewnij się, że GPU ma co najmniej 4 GB pamięci VRAM
3. Zamknij aplikacje intensywnie wykorzystujące procesor graficzny (gry, edycja wideo)
4. Monitoruj temperaturę procesora graficznego (zapewnij odpowiednie chłodzenie)

***

## Kolejne kroki

Po rozpoczęciu przetwarzania:

1. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)
2. **Poczekaj na zakończenie** — przetwarzanie przebiega automatycznie
3. **Sprawdź wyniki** — zobacz [Zakończenie przetwarzania](finishing-the-processing.md)

Aby uzyskać informacje na temat tego, co należy robić podczas przetwarzania, zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md).
