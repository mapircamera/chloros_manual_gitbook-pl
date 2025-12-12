# Rozpoczęcie przetwarzania

Po zaimportowaniu obrazów, zaznaczeniu celów kalibracji i skonfigurowaniu ustawień projektu można rozpocząć przetwarzanie. Ta strona zawiera instrukcje dotyczące uruchomienia procesu przetwarzania Chloros.

## Lista kontrolna przed przetwarzaniem

Przed kliknięciem przycisku Start sprawdź, czy wszystko jest gotowe:

* [ ] **Pliki zaimportowane** — wszystkie obrazy pojawiają się w przeglądarce plików
* [ ] **Obrazy docelowe zaznaczone** — kolumna docelowa sprawdzona pod kątem obrazów kalibracyjnych
* [ ] **Wykryte modele aparatów** — kolumna „Model aparatu” pokazuje prawidłowe aparaty
* [ ] **Skonfigurowane ustawienia** — sprawdzone i dostosowane ustawienia projektu
* [ ] **Wybrane indeksy** — dodane żądane indeksy wielospektralne (w razie potrzeby)
* [ ] **Wybrany format eksportu** — format wyjściowy odpowiedni dla Twojego przepływu pracy

{% hint style=&quot;info&quot; %}
**Wskazówka**: Przed rozpoczęciem przetwarzania kliknij kilka obrazów w przeglądarce plików, aby sprawdzić, czy zostały one poprawnie załadowane.
{% endhint %}

***

## Rozpoczęcie przetwarzania

### Znajdź przycisk Start

Przycisk Start/Odtwórz znajduje się w górnym pasku nagłówkowym Chloros:

* Położenie: w górnej środkowej części okna
* Ikona: **Przycisk Odtwórz/Start** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Przycisk jest aktywny (jasny), gdy program jest gotowy do przetwarzania

### Kliknij, aby rozpocząć

1. Kliknij **przycisk Odtwórz/Start** w górnym pasku nagłówkowym
2. Przetwarzanie rozpocznie się natychmiast
3. Podczas przetwarzania przycisk staje się nieaktywny (szary)
4. Pasek postępu aktualizuje się, pokazując status przetwarzania

{% hint style=&quot;success&quot; %}
**Rozpoczęto przetwarzanie**: Po kliknięciu Chloros automatycznie obsługuje wszystkie etapy przetwarzania — wykrywanie celu, debayering, kalibrację, obliczanie indeksu i eksport.
{% endhint %}

***

## Informacje o trybach przetwarzania

Chloros działa w dwóch różnych trybach przetwarzania w zależności od posiadanej licencji:

### Tryb bezpłatny (przetwarzanie sekwencyjne)

**Dostępny dla wszystkich użytkowników**

**Jak to działa:**

* Przetwarza obrazy pojedynczo, sekwencyjnie
* Działanie jednowątkowe
* Niższe zużycie pamięci

**Pasek postępu pokazuje 2 etapy:**

1. **Wykrywanie celu** — skanowanie w poszukiwaniu celów kalibracji
2. **Przetwarzanie** — stosowanie kalibracji i eksportowanie obrazów

**Czas przetwarzania:**

* Znacznie wolniejszy niż tryb równoległy Chloros+
* Odpowiedni dla małych i średnich zbiorów danych (&lt; 200 obrazów)

### Tryb Chloros+ (przetwarzanie równoległe)

**Wymaga licencji Chloros+**

**Jak to działa:**

* Przetwarza wiele obrazów jednocześnie
* Działanie wielowątkowe (do 16 równoległych procesów)
* Wykorzystuje wiele rdzeni procesora
* Opcjonalne przyspieszenie GPU (CUDA) dzięki kartom graficznym NVIDIA

**Pasek postępu pokazuje 4 etapy:**

1. **Wykrywanie** — wyszukiwanie celów kalibracji
2. **Analiza** — badanie metadanych obrazu i przygotowanie potoku
3. **Kalibracja** — stosowanie korekt i kalibracji
4. **Eksportowanie** — zapisywanie przetworzonych obrazów i indeksów

**Interakcja z paskiem postępu:**

* **Najedź myszką** na pasek, aby wyświetlić szczegółowy panel rozwijany z 4 etapami
* **Kliknij** pasek postępu, aby zamrozić panel rozwijany w miejscu
* **Kliknij ponownie**, aby odblokować i ukryć panel

**Czas przetwarzania:**

* Znacznie szybszy niż w trybie darmowym
* Skalowalny w zależności od liczby rdzeni procesora
* Przyspieszenie GPU dodatkowo poprawia szybkość

{% hint style=&quot;info&quot; %}
**Chloros+ Prędkość**: Przetwarzanie równoległe może być 5-10 razy szybsze niż tryb sekwencyjny w przypadku dużych zbiorów danych. Projekt obejmujący 500 obrazów, który w trybie darmowym zajmuje 2 godziny, może zostać ukończony w 15-20 minut dzięki Chloros+.
{% endhint %}

***

## Co dzieje się podczas przetwarzania

### Etap 1: Wykrywanie celu

**Co robi Chloros:**

* Skanuje zaznaczone obrazy docelowe (lub wszystkie obrazy, jeśli żadne nie są zaznaczone)
* Identyfikuje 4 panele kalibracyjne w każdym celu
* Wyodrębnia wartości odbicia od paneli docelowych
* Rejestruje sygnatury czasowe celów do planowania kalibracji

**Czas trwania:** 1–30 sekund (z zaznaczonymi celami), 5–30+ minut (bez zaznaczenia)

### Etap 2: Debayering (konwersja RAW)

**Co robi Chloros:**

* Konwertuje dane RAW w formacie Bayer na pełne obrazy RGB
* Stosuje wysokiej jakości algorytm demosaicingu
* Zachowuje maksymalną jakość obrazu i szczegóły

**Czas trwania:** Zależy od liczby obrazów i szybkości procesora

### Etap 3: Kalibracja

**Co robi Chloros:**

* **Korekcja winietowania**: usuwa zaciemnienie obiektywu na krawędziach
* **Kalibracja odbicia**: normalizuje przy użyciu docelowych wartości odbicia
* Stosuje korekty we wszystkich pasmach/kanałach
* Używa odpowiedniego celu kalibracji dla każdego obrazu na podstawie sygnatury czasowej

**Czas trwania:** większość czasu przetwarzania

### Etap 4: Obliczanie indeksu

**Czym zajmuje się Chloros:**

* Oblicza skonfigurowane indeksy wielospektralne (NDVI, NDRE itp.)
* Stosuje obliczenia pasmowe do skalibrowanych obrazów
* Generuje obrazy indeksowe dla każdego wybranego indeksu

**Czas trwania:** Kilka sekund na obraz

### Etap 5: Eksport

**Czynności wykonywane przez Chloros:**

* Zapisuje skalibrowane obrazy w wybranym formacie
* Eksportuje obrazy indeksowe z skonfigurowanymi kolorami LUT
* Zapisuje pliki w podfolderach modelu aparatu
* Zachowuje oryginalne nazwy plików z rozszerzeniami

**Czas trwania:** Zależy od formatu eksportu i rozmiaru pliku

***

## Zachowanie podczas przetwarzania

### Automatyczny proces przetwarzania

Po uruchomieniu cały proces przebiega automatycznie:

* Nie jest wymagana interakcja użytkownika
* Wszystkie skonfigurowane kroki są wykonywane sekwencyjnie
* Postęp jest wyświetlany w czasie rzeczywistym

### Wykorzystanie komputera podczas przetwarzania

**Tryb swobodny:**

* Stosunkowo niskie wykorzystanie procesora (jednowątkowe)
* Komputer pozostaje responsywny dla innych zadań
* Można bezpiecznie zminimalizować Chloros i pracować w innych aplikacjach

**Chloros+ Tryb równoległy:**

* Wysokie zużycie procesora (wielowątkowe, do 16 rdzeni)
* Z przyspieszeniem GPU: wysokie zużycie GPU
* Komputer może być mniej responsywny podczas przetwarzania
* Należy unikać uruchamiania innych zadań wymagających dużej mocy obliczeniowej procesora

{% hint style=&quot;warning&quot; %}
**Wskazówka dotycząca wydajności**: Aby uzyskać najlepszą wydajność Chloros+, zamknij inne aplikacje i pozwól Chloros korzystać z pełnych zasobów systemu.
{% endhint %}

### Przetwarzania nie można wstrzymać

**Ważne ograniczenia:**

* Po rozpoczęciu przetwarzania nie można go wstrzymać.
* Można anulować przetwarzanie, ale postęp zostanie utracony.
* Częściowe wyniki nie są zapisywane.
* W przypadku anulowania należy rozpocząć od początku.

**Wskazówka dotycząca planowania:** W przypadku bardzo dużych projektów należy rozważyć przetwarzanie w partiach lub użycie CLI w celu uzyskania lepszej kontroli.

***

## Monitorowanie przetwarzania

Podczas przetwarzania można:

* **Obserwować pasek postępu** — sprawdzić ogólny procent ukończenia.
* **Wyświetlić bieżący etap** — wykrywanie, analiza, kalibracja lub eksport.
* **Sprawdzić kartę dziennika** — wyświetlić szczegółowe komunikaty i ostrzeżenia dotyczące przetwarzania.
* **Wyświetlić podgląd ukończonych obrazów** — niektóre pliki eksportowe mogą pojawić się podczas przetwarzania.

Szczegółowe informacje na temat monitorowania można znaleźć w sekcji [Monitorowanie przetwarzania](monitoring-the-processing.md).

***

## Anulowanie przetwarzania

Jeśli chcesz zatrzymać przetwarzanie:

### Jak anulować

1. Znajdź **przycisk Zatrzymaj/Anuluj** (zastępuje przycisk Start podczas przetwarzania)
2. Kliknij przycisk Zatrzymaj
3. Przetwarzanie zostanie natychmiast zatrzymane
4. Częściowe wyniki zostaną odrzucone

### Kiedy anulować

**Uzasadnione powody anulowania:**

* Użyto nieprawidłowych ustawień
* Zapomniano o zaznaczeniu obrazów docelowych
* Zaimportowano nieprawidłowe obrazy
* System działa zbyt wolno lub nie odpowiada

**Po anulowaniu:**

* Sprawdź i napraw wszelkie problemy
* Dostosuj ustawienia zgodnie z potrzebami
* Uruchom przetwarzanie od początku
* Aby uzyskać najlepsze wyniki, całkowicie zamknij Chloros i uruchom ponownie

{% hint style=&quot;warning&quot; %}
**Brak częściowych wyników**: Anulowanie powoduje odrzucenie wszystkich postępów. Chloros nie zapisuje częściowo przetworzonych obrazów.
{% endhint %}

***

## Szacowany czas przetwarzania

Rzeczywisty czas przetwarzania różni się znacznie w zależności od:

* Liczby obrazów
* Rozdzielczości obrazów
* Formatu wejściowego RAW lub JPG
* Trybu przetwarzania (Free vs Chloros+)
* Szybkości procesora i liczby rdzeni
* Dostępności procesora graficznego (tylko Chloros+)
* Liczby indeksów do obliczenia
* Złożoności formatu eksportu

### Przybliżone szacunki (Chloros+, obrazy 12 MP, nowoczesny procesor)

| Liczba obrazów | Tryb Free | Chloros+ (procesor) | Chloros+ (procesor graficzny) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrazów   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrazów  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrazów  | 1–1,5 godz. | 20–30 min      | 10–15 min      |
| 500 zdjęć  | 2-3 godz.   | 45-60 min      | 20-30 min      |
| 1000 zdjęć | 4-6 godz.   | 1,5-2 godz.      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Pierwsze uruchomienie**: Początkowe przetwarzanie może potrwać dłużej, ponieważ Chloros tworzy pamięć podręczną i profile. Kolejne przetwarzanie podobnych zestawów danych będzie przebiegało szybciej.
{% endhint %}

***

## Typowe problemy podczas uruchamiania

### Przycisk Start nieaktywny (wyszarzony)

**Możliwe przyczyny:**

* Brak zaimportowanych obrazów
* Backend nie został w pełni uruchomiony
* Poprzednie przetwarzanie nadal trwa
* Projekt nie został w pełni załadowany

**Rozwiązania:**

1. Poczekaj, aż backend zostanie w pełni zainicjowany (sprawdź ikonę menu głównego)
2. Sprawdź, czy obrazy zostały zaimportowane w przeglądarce plików
3. Uruchom ponownie Chloros, jeśli przycisk pozostaje nieaktywny
4. Sprawdź dziennik debugowania pod kątem komunikatów o błędach

### Przetwarzanie rozpoczyna się, ale natychmiast kończy się niepowodzeniem

**Możliwe przyczyny:**

* Brak prawidłowych obrazów w projekcie
* Uszkodzone pliki obrazów
* Niewystarczająca ilość miejsca na dysku
* Niewystarczająca ilość pamięci (RAM)

**Rozwiązania:**

1. Sprawdź dziennik debugowania <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> pod kątem komunikatów o błędach
2. Sprawdź dostępną przestrzeń dyskową
3. Spróbuj przetworzyć mniejszą podgrupę obrazów
4. Sprawdź, czy obrazy nie są uszkodzone

### Ostrzeżenie „Nie wykryto celów”

**Możliwe przyczyny:**

* Zapomniano oznaczyć obrazy docelowe
* Obrazy docelowe nie zawierają widocznych celów
* Ustawienia wykrywania celów są zbyt rygorystyczne

**Rozwiązania:**

1. Przejrzyj [Wybór obrazów docelowych](choosing-target-images.md)
2. Zaznacz odpowiednie obrazy w kolumnie Docelowe
3. Sprawdź, czy cele są widoczne na zaznaczonych obrazach
4. W razie potrzeby dostosuj ustawienia wykrywania celów

***

## Wskazówki dotyczące pomyślnego przetwarzania

### Przed rozpoczęciem

1. **Najpierw przetestuj na małej podgrupie** — przetwórz 10–20 obrazów, aby zweryfikować ustawienia.
2. **Sprawdź dostępną przestrzeń dyskową** — upewnij się, że masz 2–3 razy więcej wolnego miejsca niż rozmiar zbioru danych.
3. **Zamknij niepotrzebne aplikacje** — zwolnij zasoby systemowe.
4. **Zweryfikuj obrazy docelowe** — wyświetl podgląd zaznaczonych celów, aby upewnić się co do jakości.
5. **Zapisz projekt** — projekt jest zapisywany automatycznie, ale dobrą praktyką jest ręczne zapisywanie.

### Podczas przetwarzania

1. **Unikaj trybu uśpienia systemu** — wyłącz tryby oszczędzania energii.
2. **Utrzymuj Chloros na pierwszym planie** — lub przynajmniej widoczny na pasku zadań.
3. **Od czasu do czasu monitoruj postęp** — sprawdzaj, czy nie ma ostrzeżeń lub błędów.
4. **Nie ładuj innych ciężkich aplikacji** – zwłaszcza w trybie równoległym Chloros+.

### Chloros+ Przyspieszenie GPU

W przypadku korzystania z przyspieszenia GPU NVIDIA:

1. Zaktualizuj sterowniki NVIDIA do najnowszej wersji.
2. Upewnij się, że GPU ma co najmniej 4 GB pamięci VRAM.
3. Zamknij aplikacje intensywnie wykorzystujące GPU (gry, edycja wideo).
4. Monitoruj temperaturę GPU (zapewnij odpowiednie chłodzenie).

***

## Kolejne kroki

Po rozpoczęciu przetwarzania:

1. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md).
2. **Poczekaj na zakończenie** — przetwarzanie przebiega automatycznie.
3. **Sprawdź wyniki** — zobacz [Zakończenie przetwarzania](finishing-the-processing.md).

Aby uzyskać informacje na temat tego, co należy robić podczas przetwarzania, zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md).
