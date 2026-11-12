# Rozpoczęcie przetwarzania

Po zaimportowaniu obrazów, zaznaczeniu celów kalibracyjnych i skonfigurowaniu ustawień projektu można przystąpić do przetwarzania. Niniejsza strona zawiera instrukcje dotyczące uruchomienia procesu przetwarzania w programie Chloros.

## Lista kontrolna przed przetwarzaniem

Przed kliknięciem przycisku „Start” należy sprawdzić, czy wszystko jest gotowe:

* [ ] **Pliki zaimportowane** – wszystkie zdjęcia pojawiają się w przeglądarce plików
* [ ] **Zdjęcia docelowe zaznaczone** – w kolumnie „Target” zaznaczono zdjęcia kalibracyjne (lub zaimportowano zapis `.daq` dla LATTICE)
* [ ] **Wykryto modele kamer** — w kolumnie „Model kamery” wyświetlają się właściwe kamery
* [ ] **Skonfigurowano ustawienia** — ustawienia projektu zostały sprawdzone i dostosowane
* [ ] **Wybrano wskaźniki** — dodano żądane wskaźniki wielospektralne (w razie potrzeby)
* [ ] **Wybrano format eksportu** – format wyjściowy odpowiedni dla danego przepływu pracy

{% hint style="info" %}
**Wskazówka**: Przed rozpoczęciem przetwarzania kliknij kilka obrazów w przeglądarce plików, aby sprawdzić, czy zostały poprawnie załadowane.
{% endhint %}

***

## Rozpoczęcie przetwarzania

### Znajdź przycisk „Start”

Przycisk „Start/Odtwórz” znajduje się w górnym pasku nagłówkowym programu Chloros:

* Położenie: u góry pośrodku okna
* Ikona: **Przycisk „Odtwórz/Start”** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stan: Przycisk jest aktywny (podświetlony), gdy program jest gotowy do przetwarzania

### Kliknij, aby rozpocząć

1. Kliknij **przycisk Odtwarzaj/Start** w górnym pasku nagłówka
2. Przetwarzanie rozpocznie się natychmiast
3. Podczas przetwarzania przycisk zmienia się w przycisk **Stop**

4. Pasek postępu aktualizuje się, pokazując stan przetwarzania

{% hint style="success" %}
**Rozpoczęto przetwarzanie**: Po kliknięciu program Chloros automatycznie wykonuje wszystkie etapy przetwarzania – wykrywanie celów, usuwanie efektu bayera, kalibrację, obliczanie indeksu oraz eksport. Automatycznie wykrywa, czy projekt jest typu Survey3, LATTICE, czy mieszanym, i stosuje odpowiedni proces przetwarzania dla każdej kamery.
{% endhint %}

***

## Omówienie trybów przetwarzania Program

Chloros działa w dwóch różnych trybach przetwarzania w zależności od posiadanej licencji:

### Tryb bezpłatny (przetwarzanie sekwencyjne)

**Dostępny dla wszystkich użytkowników**

**Jak to działa:**

* Przetwarza obrazy pojedynczo, sekwencyjnie
* Działa w trybie jednowątkowym
* Mniejsze zużycie pamięci

**Pasek postępu pokazuje 2 etapy:**

1.**Wykrywanie celów** – skanowanie w poszukiwaniu celów kalibracyjnych
2. **Przetwarzanie** – stosowanie kalibracji i eksportowanie obrazów**Czas przetwarzania:**

* Znacznie wolniejszy niż w trybie równoległym Chloros+
* Odpowiedni dla małych i średnich zbiorów danych (< 200 obrazów)

### Tryb Chloros+ (przetwarzanie równoległe)

**Wymaga licencji Chloros+**

**Jak to działa:**

* Przetwarza wiele obrazów jednocześnie przy użyciu [4-wątkowego potoku przetwarzania](../processing-architecture/processing-pipeline.md)
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) automatycznie wybiera optymalną strategię dla danego sprzętu na początku działania
* Przyspieszenie GPU (CUDA) z wykorzystaniem kart graficznych NVIDIA
* **Liczba procesów roboczych dostosowuje się do sprzętu**: strategie oparte na GPU uruchamiają**2–4 równoległe procesy robocze** (skalowane w zależności od pamięci VRAM — 3 na kartach graficznych o pojemności 2–12 GB, 2–4 na kartach graficznych o pojemności 12 GB i większej); Systemy oparte wyłącznie na procesorze CPU uruchamiają jednego procesora roboczego na każdy fizyczny rdzeń, pomniejszając tę liczbę o jeden**Pasek postępu pokazuje 4 etapy** (odpowiadające 4 wątkom potoku):

1. **Wykrywanie** (wątek 1) – wyszukiwanie obiektów do kalibracji
2. **Analiza** (wątek 2) – badanie metadanych obrazu i obliczanie kalibracji
3. **Kalibracja** (wątek 3) – usuwanie efektu bayera, korekcja winietowania, kalibracja, obliczanie indeksu
4. **Eksport** (wątek 4) – zapisywanie przetworzonych obrazów i indeksów

**Obsługa paska postępu:**

* **Najedź kursorem myszy** na pasek, aby wyświetlić szczegółowy panel rozwijany z 4 etapami
* **Kliknij** pasek postępu, aby zablokować panel rozwijany w bieżącej pozycji
* **Kliknij ponownie**, aby odblokować i ukryć panel**Czas przetwarzania:**

* Znacznie szybszy niż w trybie darmowym
* Przyspieszenie GPU dodatkowo zwiększa szybkość

{% hint style="info" %}
**Szybkość Chloros+**: Przetwarzanie równoległe może być 5–10 razy szybsze niż w trybie sekwencyjnym w przypadku dużych zbiorów danych. Projekt obejmujący 500 obrazów, którego przetworzenie w trybie darmowym zajmuje 2 godziny, może zostać ukończony w 15–20 minut przy użyciu Chloros+.
{% endhint %}

***

## Co dzieje się podczas przetwarzania

### Etap 1: Wykrywanie celów

**Co robi Chloros:**

* Skanuje zdjęcia zaznaczone w kolumnie „Cel” (wszystkie zdjęcia, jeśli żadne nie jest zaznaczone)
* Identyfikuje panele kalibracyjne w każdym celu
* Wyodrębnia wartości odbicia światła z paneli celów
* Rejestruje znaczniki czasu celów na potrzeby planowania kalibracji

**Czas trwania:** 1–30 sekund (w przypadku zaznaczonych celów), 5–30+ minut (w przypadku niezaznaczonych)

### Etap 2: Usuwanie efektu bayera (konwersja plików RAW)

**Czym zajmuje się Chloros:**

* Konwertuje dane RAW w układzie Bayera na pełne obrazy 3-kanałowe (moduły LATTICE mono pozostają jednopasmowe — w ich przypadku proces debayeringu jest pomijany, co odnotowuje się w dzienniku)
* Stosuje wybrany algorytm demosaicingu
* Zachowuje maksymalną jakość obrazu i szczegółowość

**Czas trwania:** Zależy od liczby obrazów oraz szybkości procesora (CPU) lub karty graficznej (GPU)

### Etap 3: Kalibracja

**Czym zajmuje się Chloros:**

* **Korekcja winietowania**: Usuwa przyciemnienie na krawędziach spowodowane obiektywem
* **Kalibracja odbicia**: Normalizuje na podstawie docelowych wartości odbicia i/lub danych DAQ dotyczących promieniowania padającego
* Stosuje korekty we wszystkich pasmach/kanałach
* Wykorzystuje odpowiedni wzorzec kalibracyjny dla każdego obrazu na podstawie sygnatury czasowej

**Czas trwania:** Większość czasu przetwarzania

### Etap 4: Obliczanie wskaźników

**Czym zajmuje się Chloros:**

* Oblicza skonfigurowane wskaźniki wielospektralne (NDVI, NDRE itp.)
* Stosuje operacje matematyczne na pasmach do skalibrowanych obrazów
* Generuje obrazy wskaźnikowe dla każdego wybranego wskaźnika

**Czas trwania:** Kilka sekund na obraz

### Etap 5: Eksport

**Czym zajmuje się Chloros:**

* Zapisuje przetworzone obrazy w wybranym formacie
* **LATTICE fan-out**: każda surowa klatka LATTICE jest eksportowana jako każdy włączony produkt w jednym przebiegu — bez efektu bayera, podgląd, promieniowanie (zawsze float32), odbicie
* Zapisuje pliki w drzewie wynikowym projektu: `<project>/<camera>/<format>/<Product>_Images/`
* **Zachowuje nazwę pliku źródłowego** — folder identyfikuje produkt, nie dodaje się żadnego rozszerzenia**Czas trwania:** Zależy od formatu eksportu i rozmiaru pliku***

## Zachowanie podczas przetwarzania

### Automatyczny potok przetwarzania

Po uruchomieniu cały potok działa automatycznie:

* Nie jest wymagana interakcja użytkownika
* Wszystkie skonfigurowane etapy są wykonywane po kolei
* Aktualizacje postępu są wyświetlane w czasie rzeczywistym
* Eksportowane pliki są zapisywane na dysku w miarę ich ukończenia — gotowe pliki wyjściowe można otwierać, podczas gdy przetwarzanie trwa

### Wykorzystanie zasobów komputera podczas przetwarzania

**Tryb swobodny:**

* Stosunkowo niskie obciążenie procesora (jednowątkowe)
* Komputer pozostaje responsywny dla innych zadań
* Można bezpiecznie zminimalizować okno programu Chloros i pracować w innych aplikacjach

**Tryb równoległy Chloros+:**

* Wysokie obciążenie procesora w całej puli procesorów strategii
* Przy przyspieszeniu GPU: wysokie obciążenie karty graficznej
* Komputer może reagować wolniej podczas przetwarzania
* Należy unikać uruchamiania innych zadań intensywnie obciążających procesor

{% hint style="warning" %}
**Wskazówka dotycząca wydajności**: Aby uzyskać najlepszą wydajność Chloros+, zamknij inne aplikacje i pozwól Chloros na pełne wykorzystanie zasobów systemowych.
{% endhint %}

### Przetwarzania nie można wstrzymać (ale zatrzymanie przebiega poprawnie)

* Po uruchomieniu przetwarzania nie można go wstrzymać, a następnie wznowić
* Kliknięcie przycisku **Stop** powoduje całkowite zatrzymanie przebiegu już przy pierwszym kliknięciu
* Produkty wyeksportowane przed zatrzymaniem pozostają na dysku
* Zatrzymany przebieg rzetelnie informuje o tym, co zostało ukończone (zobacz wiersze `[RUN-SUMMARY]` w dzienniku)
* Nowe uruchomienie rozpoczyna potok od początku

**Wskazówka dotycząca planowania:** W przypadku bardzo dużych projektów warto rozważyć przetwarzanie partiami lub korzystanie z interfejsu CLI w celu uzyskania lepszej kontroli.

***

## Monitorowanie przetwarzania

Podczas trwania przetwarzania można:

* **Obserwować pasek postępu** — sprawdzić ogólny procent zaawansowania
* **Wyświetlić bieżący etap** — wykrywanie, analiza, kalibracja lub eksport
* **Sprawdzić zakładkę dziennika** — wyświetlić szczegółowe komunikaty i ostrzeżenia dotyczące przetwarzania
* **Wyświetlić podgląd gotowych obrazów** — pliki eksportowane pojawiają się na dysku w trakcie przetwarzania

Szczegółowe informacje na temat monitorowania można znaleźć w artykule [Monitorowanie przetwarzania](monitoring-the-processing.md).

***

## Zatrzymywanie przetwarzania

Jeśli konieczne jest zatrzymanie przetwarzania:

### Jak zatrzymać

1. Znajdź **przycisk Zatrzymaj** (podczas przetwarzania zastępuje on przycisk „Start”)
2. Kliknij go raz — na pasku pojawi się komunikat **„Zatrzymywanie...”**, podczas gdy obraz w trakcie przetwarzania zostanie ukończony
3. Przetwarzanie zakończy się w stanie ostatecznego zatrzymania, a w dzienniku zostanie zapisany dokładny raport `[RUN-SUMMARY]` dotyczący tego, co zostało wykonane

### Kiedy należy zatrzymać przetwarzanie

**Uzasadnione powody zatrzymania:**

* Stwierdzono, że zastosowano nieprawidłowe ustawienia
* Zapomniano oznaczyć obrazy docelowe
* Zaimportowano niewłaściwe obrazy
* System działa zbyt wolno lub nie reaguje

**Po zatrzymaniu:**

* Produkty wyeksportowane przed zatrzymaniem pozostają na dysku
* Sprawdź i napraw ewentualne problemy, dostosuj ustawienia w razie potrzeby
* Uruchom przetwarzanie ponownie — proces rozpocznie się od początku

***

## Szacunkowy czas przetwarzania

Rzeczywisty czas przetwarzania różni się znacznie w zależności od:

* Liczby obrazów
* Rozdzielczości obrazów
* Formatu wejściowego (RAW lub JPG)
* Trybu przetwarzania (Free vs Chloros+)
* Szybkość procesora i liczba rdzeni
* Dostępność karty graficznej (tylko Chloros+)
* Liczba indeksów do obliczenia
* Liczba włączonych produktów eksportowych (LATTICE)

### Przybliżone szacunki (Chloros+, zdjęcia 12 MP, nowoczesny procesor)

| Liczba zdjęć | Tryb bezpłatny | Chloros+ (procesor) | Chloros+ (karta graficzna) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrazów   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrazów  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrazów  | 1–1,5 godz. | 20–30 min      | 10–15 min      |
| 500 obrazów  | 2–3 godz.   | 45–60 min      | 20–30 min      |
| 1000 obrazów | 4–6 godz.   | 1,5–2 godz.      | 40–60 min      |

{% hint style="info" %}
**Pierwsze uruchomienie**: Wstępne przetwarzanie może potrwać dłużej, ponieważ Chloros tworzy pamięć podręczną i profile. Kolejne przetwarzanie podobnych zestawów danych będzie przebiegało szybciej.
{% endhint %}

***

## Typowe problemy przy uruchomieniu

### Przycisk „Start” jest nieaktywny (wyszarzony)

**Możliwe przyczyny:**

* Nie zaimportowano żadnych obrazów
* Moduł zaplecza nie został w pełni uruchomiony
* Poprzednie przetwarzanie nadal trwa
* Projekt nie został w pełni załadowany

**Rozwiązania:**

1. Poczekaj, aż serwer w tle w pełni się zainicjuje (sprawdź ikonę w menu głównym)
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
3. Spróbuj przetworzyć mniejszy podzbiór obrazów
4. Sprawdź, czy obrazy nie są uszkodzone

### Przetwarzanie kończy się, ale nie zapisuje żadnych obrazów

Przetwarzanie, które zażądało produktów obrazowych, ale nie zapisało żadnego, jest traktowane jako **niepowodzenie, a nie sukces** — program Chloros wyraźnie to sygnalizuje:

* W dzienniku GUI wyświetlane są komunikaty `[RUN-SUMMARY]` wskazujące prawdopodobną przyczynę — brak zaimportowanych obrazów, brak wykrytego celu lub pominięcie wszystkich żądanych produktów jako nieadekwatnych (np. żądanie danych dotyczących promieniowania/odbicia z kamer obsługujących wyłącznie RGB)
* Odpowiednik w CLI (`chloros-cli process`) wyświetla komunikat `Processing finished but wrote no image products.` i **kończy działanie z wynikiem niezerowym**, dzięki czemu skrypty mogą to wykryć
* Celowe uruchomienie ograniczone wyłącznie do metadanych (wszystkie produkty eksportowe wyłączone, brak indeksów) nadal liczy się jako sukces

Pełną semantykę można znaleźć w [podręczniku CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Folder projektu jest zbyt głęboko osadzony na tym komputerze

**Tylko Windows, nowość w wersji 1.3.0.** System Windows jest dostarczany z wyłączoną obsługą długich ścieżek, co oznacza, że żadna ścieżka do pliku nie może być dłuższa niż 260 znaków. Produkty Chloros znajdują się około 130 znaków poniżej folderu projektu, więc folder projektu położony głębiej nie może otrzymać swoich produktów. Zamiast uruchamiać się i nie zapisywać niczego, Chloros sprawdza teraz przed rozpoczęciem przetwarzania:

* Kliknięcie przycisku „Start” nie uruchamia procesu: pasek postępu pokazuje przyczynę, a w dzienniku debugowania pojawia się kod `[PROCESS] refused: The project folder is too deep for this computer…`. Komunikat podaje długość folderu projektu oraz najdłuższy folder projektu, który się mieści.
* Tworzenie lub otwieranie projektu w folderze zbyt głębokim nawet dla własnych plików projektu również jest odrzucane. W oknie komunikatu podawany jest powód wraz z informacją o najdłuższym dopuszczalnym folderze projektu oraz instrukcją włączenia długich ścieżek; nic nie jest tworzone ani otwierane; w dzienniku debugowania jest to rejestrowane jako `[PROJECT] refused: …`. Powielanie projektu pod dłuższą nazwą również może spowodować przekroczenie limitu: kopia zostaje utworzona, ale nie jest otwierana, a ten sam komunikat wyjaśnia przyczynę.

**Rozwiązania:**

1. Przenieś projekty do folderu o krótszej nazwie — zmień**Katalog roboczy** w [Ustawieniach projektu](../project-settings/project-settings.md) i utwórz projekt ponownie, lub
2. Włącz obsługę długich ścieżek w systemie Windows i uruchom ponownie program Chloros: uruchom ponownie instalator Chloros z zaznaczoną opcją **Włącz długie ścieżki w systemie Windows** (jest ona zaznaczona domyślnie) lub skorzystaj z zasad grupy: *Konfiguracja komputera > Szablony administracyjne > System > System plików > Włącz długie ścieżki Win32*.

### Ostrzeżenie „Nie wykryto celów”

**Możliwe przyczyny:**

* Zapomniano oznaczyć obrazy docelowe
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

1. **Najpierw przetestuj na małym podzbiorze** – przetwórz 10–20 obrazów, aby zweryfikować ustawienia
2. **Sprawdź dostępną przestrzeń dyskową** – upewnij się, że masz 2–3 razy więcej wolnego miejsca niż rozmiar zbioru danych (więcej, jeśli włączone są wszystkie produkty LATTICE)
3. **Zamknij zbędne aplikacje** – zwolnij zasoby systemowe
4. **Sprawdź obrazy z celami** – wyświetl podgląd zaznaczonych celów, aby upewnić się co do jakości
5. **Zapisz projekt** – projekt zapisuje się automatycznie, ale dobrą praktyką jest ręczne zapisywanie

### Podczas przetwarzania

1. **Unikaj uśpienia systemu** – wyłącz tryby oszczędzania energii
2. **Utrzymuj program Chloros na pierwszym planie** – lub przynajmniej widoczny na pasku zadań
3. **Od czasu do czasu monitoruj postęp** – sprawdzaj, czy nie ma ostrzeżeń lub błędów
4. **Nie uruchamiaj innych wymagających aplikacji** – zwłaszcza w trybie równoległym Chloros+

### Przyspieszenie Chloros za pomocą procesora graficznego (GPU)

W przypadku korzystania z przyspieszenia za pomocą procesora graficznego NVIDIA:

1. Zaktualizuj sterowniki NVIDIA do najnowszej wersji
2. Upewnij się, że procesor graficzny ma co najmniej 4 GB pamięci VRAM (co najmniej 7 GB w przypadku równoczesnego debayeringu z obsługą tekstur)
3. Zamknij aplikacje intensywnie obciążające procesor graficzny (gry, edycja wideo)
4. Monitoruj temperaturę procesora graficznego (zapewnij odpowiednie chłodzenie)

***

## Kolejne kroki

Po rozpoczęciu przetwarzania:

1. **Monitoruj postęp** – zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)
2. **Poczekaj na zakończenie** – przetwarzanie przebiega automatycznie
3. **Sprawdź wyniki** — zobacz [Zakończenie przetwarzania](finishing-the-processing.md)

Aby uzyskać informacje na temat tego, co należy robić podczas przetwarzania, zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md).
