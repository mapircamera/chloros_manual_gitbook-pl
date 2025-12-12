# Monitorowanie przetwarzania

Po rozpoczęciu przetwarzania Chloros oferuje kilka sposobów monitorowania postępów, sprawdzania problemów i zrozumienia, co dzieje się z zestawem danych. Na tej stronie wyjaśniono, jak śledzić przetwarzanie i interpretować informacje dostarczane przez Chloros.

## Omówienie paska postępu

Pasek postępu w górnym nagłówku pokazuje status przetwarzania w czasie rzeczywistym oraz procent ukończenia.

### Pasek postępu w trybie darmowym

Dla użytkowników bez licencji Chloros+:

**Wyświetlanie postępu w 2 etapach:**

1. **Wykrywanie celu** — wyszukiwanie celów kalibracji na obrazach
2. **Przetwarzanie** — stosowanie korekt i eksportowanie

**Pasek postępu pokazuje:**

* Ogólny procent ukończenia (0-100%)
* Nazwę bieżącego etapu
* Proste wizualizacje w postaci poziomych pasków

### Pasek postępu Chloros+

Dla użytkowników posiadających licencję Chloros+:

**4-etapowe wyświetlanie postępu:**

1. **Wykrywanie** — wyszukiwanie celów kalibracji
2. **Analiza** — badanie obrazów i przygotowywanie potoku
3. **Kalibracja** — stosowanie korekt winietowania i odbicia
4. **Eksportowanie** — zapisywanie przetworzonych plików

**Funkcje interaktywne:**

* **Najedź kursorem** na pasek postępu, aby wyświetlić rozszerzony 4-etapowy panel
* **Kliknij** pasek postępu, aby zamrozić/przypiąć rozszerzony panel
* **Kliknij ponownie**, aby odmrozić i automatycznie ukryć po odsunięciu myszy
* Każdy etap pokazuje indywidualny postęp (0-100%)

***

## Zrozumienie każdego etapu przetwarzania

### Etap 1: Wykrywanie (wykrywanie celów)

**Co się dzieje:**

* Chloros skanuje obrazy oznaczone polem wyboru Cel
* Algorytmy wizji komputerowej identyfikują 4 panele kalibracyjne
* Wartości odbicia wyodrębnione z każdego panelu
* Znaczniki czasu celów zarejestrowane w celu prawidłowego planowania kalibracji

**Czas trwania:**

* Z zaznaczonymi celami: 10-60 sekund
* Bez zaznaczonych celów: 5-30+ minut (skanowanie wszystkich obrazów)

**Wskaźnik postępu:**

* Wykrywanie: 0% → 100%
* Liczba zeskanowanych obrazów
* Liczba znalezionych celów

**Na co należy zwrócić uwagę:**

* Jeśli cele są prawidłowo zaznaczone, proces powinien zakończyć się szybko.
* Jeśli trwa zbyt długo, cele mogą nie być zaznaczone.
* Sprawdź dziennik debugowania pod kątem komunikatów „Target found” (Znaleziono cel).

### Etap 2: Analiza

**Co się dzieje:**

* Odczyt metadanych EXIF obrazu (znaczniki czasu, ustawienia ekspozycji)
* Określenie strategii kalibracji na podstawie znaczników czasu celów
* Organizacja kolejki przetwarzania obrazów
* Przygotowanie równoległych procesów przetwarzania (tylko Chloros+)

**Czas trwania:** 5–30 sekund

**Wskaźnik postępu:**

* Analiza: 0% → 100%
* Szybki etap, zazwyczaj kończy się szybko

**Na co należy zwrócić uwagę:**

* Postęp powinien przebiegać płynnie, bez przerw
* Ostrzeżenia o brakujących metadanych pojawią się w dzienniku debugowania

### Etap 3: Kalibracja

**Co się dzieje:**

* **Debayering**: Konwersja wzorca RAW Bayer na 3 kanały
* **Korekcja winietowania**: usuwanie ciemnych krawędzi obiektywu
* **Kalibracja odbicia**: normalizacja za pomocą wartości docelowych
* **Obliczanie indeksu**: obliczanie indeksów wielospektralnych
* Przetwarzanie każdego obrazu w pełnym procesie

**Czas trwania:** większość całkowitego czasu przetwarzania (60–80%)

**Wskaźnik postępu:**

* Kalibracja: 0% → 100%
* Obecny obraz w trakcie przetwarzania
* Obrazy ukończone / Łączna liczba obrazów

**Zachowanie podczas przetwarzania:**

* **Tryb swobodny**: przetwarza po jednym obrazie na raz
* **Tryb Chloros+**: przetwarza do 16 obrazów jednocześnie
* **Przyspieszenie GPU**: Znacznie przyspiesza ten etap.

**Na co należy zwrócić uwagę:**

* Stały postęp w zakresie liczby obrazów.
* Sprawdź dziennik debugowania pod kątem komunikatów o zakończeniu przetwarzania poszczególnych obrazów.
* Ostrzeżenia dotyczące jakości obrazu lub problemów z kalibracją.

### Etap 4: Eksportowanie

**Co się dzieje:**

* Zapisywanie skalibrowanych obrazów na dysku w wybranym formacie
* Eksportowanie obrazów indeksu wielospektralnego z kolorami LUT
* Tworzenie podfolderów modeli kamer
* Zachowanie oryginalnych nazw plików z odpowiednimi rozszerzeniami

**Czas trwania:** 10–20% całkowitego czasu przetwarzania

**Wskaźnik postępu:**

* Eksportowanie: 0% → 100%
* Zapisywane pliki
* Format eksportu i miejsce docelowe

**Na co należy zwrócić uwagę:**

* Ostrzeżenia dotyczące miejsca na dysku
* Błędy zapisu plików
* Zakończenie wszystkich skonfigurowanych operacji wyjściowych

***

## Zakładka dziennika debugowania

Dziennik debugowania zawiera szczegółowe informacje na temat postępu przetwarzania i wszelkich napotkanych problemów.

### Dostęp do dziennika debugowania

1. Kliknij ikonę **Dziennik debugowania** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na lewym pasku bocznym
2. Otworzy się panel dziennika z komunikatami dotyczącymi przetwarzania w czasie rzeczywistym
3. Automatyczne przewijanie w celu wyświetlenia najnowszych komunikatów

### Rozumienie komunikatów dziennika

#### Komunikaty informacyjne (białe/szare)

Normalne aktualizacje przetwarzania:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Komunikaty ostrzegawcze (żółte)

Problemy niekrytyczne, które nie zatrzymują przetwarzania:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Działanie:** Przejrzyj ostrzeżenia po przetworzeniu, ale nie przerywaj

#### Komunikaty o błędach (Red)

Krytyczne problemy, które mogą spowodować niepowodzenie przetwarzania:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Działanie:** Zatrzymaj przetwarzanie, usuń błąd, uruchom ponownie.

### Typowe komunikaty dziennika

| Komunikat                          | Znaczenie                                | Wymagane działanie                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Wykryto cel w \[nazwa pliku]” | Cel kalibracji znaleziony pomyślnie  | Brak — normalnie                                         |
| „Przetwarzanie obrazu X z Y”        | Aktualizacja bieżącego postępu                | Brak — normalnie                                         |
| „Nie znaleziono celów”               | Nie wykryto celów kalibracji        | Zaznacz obrazy celów lub wyłącz kalibrację odbicia |
| „Niewystarczająca ilość miejsca na dysku”        | Niewystarczająca ilość miejsca na dysku na dane wyjściowe          | Zwolnij miejsce na dysku                                    |
| „Pomijanie uszkodzonego pliku”        | Plik obrazu jest uszkodzony                  | Skopiuj ponownie plik z karty SD                             |
| „Zastosowano dane PPK”               | Zastosowano korekty GPS z pliku .daq | Brak — normalnie                                         |

### Kopiowanie danych dziennika

Aby skopiować dziennik w celu rozwiązania problemów lub uzyskania pomocy technicznej:

1. Otwórz panel dziennika debugowania.
2. Kliknij przycisk **„Kopiuj dziennik”** (lub kliknij prawym przyciskiem myszy → Wybierz wszystko).
3. Wklej do pliku tekstowego lub wiadomości e-mail.
4. W razie potrzeby wyślij do pomocy technicznej MAPIR.

***

## Monitorowanie zasobów systemowych

### Wykorzystanie procesora

**Tryb swobodny:**

* 1 rdzeń procesora przy ~100%
* Pozostałe rdzenie są bezczynne lub dostępne
* System pozostaje responsywny

**Chloros+ Tryb równoległy:**

* Wiele rdzeni przy 80-100% (do 16 rdzeni)
* Wysokie ogólne wykorzystanie procesora
* System może wydawać się mniej responsywny

**Aby monitorować:**

* Windows Menedżer zadań (Ctrl+Shift+Esc)
* Zakładka Wydajność → sekcja Procesor
* Poszukaj procesów „Chloros” lub „chloros-backend”

### Wykorzystanie pamięci (RAM)

**Typowe wykorzystanie:**

* Małe projekty (&lt; 100 obrazów): 2–4 GB
* Średnie projekty (100–500 obrazów): 4–8 GB
* Duże projekty (ponad 500 obrazów): 8–16 GB
* Tryb równoległy Chloros+ wykorzystuje więcej pamięci RAM

**Jeśli pamięć jest niewystarczająca:**

* Przetwarzaj mniejsze partie danych.
* Zamknij inne aplikacje.
* Zwiększ ilość pamięci RAM, jeśli regularnie przetwarzasz duże zbiory danych.

### Wykorzystanie GPU (Chloros+ z CUDA)

Gdy włączone jest przyspieszenie GPU:

* GPU NVIDIA wykazuje wysokie wykorzystanie (60–90%).
* Wzrasta wykorzystanie pamięci VRAM (wymagane 4 GB+ VRAM)
* Etap kalibracji przebiega znacznie szybciej

**Aby monitorować:**

* Ikona NVIDIA w zasobniku systemowym
* Menedżer zadań → Wydajność → GPU
* GPU-Z lub podobne narzędzie monitorujące

### We/wy dysku

**Czego można się spodziewać:**

* Wysoki odczyt dysku podczas etapu analizy
* Wysoki zapis dysku podczas etapu eksportowania
* SSD znacznie szybszy niż HDD

**Wskazówka dotycząca wydajności:**

* Jeśli to możliwe, używaj dysku SSD dla folderu projektu
* Unikaj dysków sieciowych dla dużych zbiorów danych
* Upewnij się, że dysk nie jest prawie pełny (wpływa to na szybkość zapisu)

***

## Wykrywanie problemów podczas przetwarzania

### Ostrzegawcze sygnały

**Postęp zatrzymuje się (brak zmian przez ponad 5 minut):**

* Sprawdź dziennik debugowania pod kątem błędów
* Sprawdź dostępną przestrzeń dyskową
* Sprawdź w Menedżerze zadań, czy proces Chloros jest uruchomiony

**Często pojawiają się komunikaty o błędach:**

* Zatrzymaj przetwarzanie i sprawdź błędy
* Typowe przyczyny: przestrzeń dyskowa, uszkodzone pliki, problemy z pamięcią
* Zobacz sekcję Rozwiązywanie problemów poniżej

**System nie odpowiada:**

* Tryb równoległy Chloros+ zużywa zbyt wiele zasobów
* Rozważ zmniejszenie liczby równoczesnych zadań lub modernizację sprzętu
* Tryb swobodny zużywa mniej zasobów

### Kiedy przerwać przetwarzanie

Przerwij przetwarzanie, jeśli widzisz:

* ❌ Błędy „Dysk pełny” lub „Nie można zapisać pliku”
* ❌ Powtarzające się błędy uszkodzenia plików obrazów
* ❌ Całkowite zawieszenie systemu (brak reakcji)
* ❌ Stwierdzenie nieprawidłowych ustawień
* ❌ Importowanie nieprawidłowych obrazów

**Jak zatrzymać:**

1. Kliknij **przycisk Zatrzymaj/Anuluj** (zastępuje przycisk Start)
2. Przetwarzanie zostanie zatrzymane, a postęp zostanie utracony
3. Napraw problemy i zacznij od początku

***

## Rozwiązywanie problemów podczas przetwarzania

### Przetwarzanie przebiega bardzo wolno

**Możliwe przyczyny:**

* Nieoznaczone obrazy docelowe (skanowanie wszystkich obrazów)
* Dysk twardy zamiast dysku SSD
* Niewystarczające zasoby systemowe
* Skonfigurowano wiele indeksów
* Dostęp do dysku sieciowego

**Rozwiązania:**

1. Jeśli proces właśnie się rozpoczął i znajduje się w fazie wykrywania: anuluj, zaznacz cele, uruchom ponownie
2. Na przyszłość: użyj dysku SSD, zmniejsz liczbę indeksów, zmodernizuj sprzęt
3. Rozważ użycie CLI do przetwarzania wsadowego dużych zbiorów danych

### Ostrzeżenia dotyczące „miejsca na dysku”

**Rozwiązania:**

1. Natychmiast zwolnij miejsce na dysku
2. Przenieś projekt na dysk z większą ilością miejsca
3. Zmniejsz liczbę indeksów do wyeksportowania.
4. Użyj formatu JPG zamiast TIFF (mniejsze pliki).

### Częste komunikaty „Uszkodzony plik”

**Rozwiązania:**

1. Skopiuj ponownie obrazy z karty SD, aby zapewnić ich integralność.
2. Sprawdź kartę SD pod kątem błędów.
3. Usuń uszkodzone pliki z projektu.
4. Kontynuuj przetwarzanie pozostałych obrazów.

### Przegrzanie systemu / ograniczanie wydajności

**Rozwiązania:**

1. Zapewnij odpowiednią wentylację.
2. Oczyść otwory wentylacyjne komputera z kurzu.
3. Zmniejsz obciążenie przetwarzania (użyj trybu Free zamiast Chloros+).
4. Przetwarzaj w chłodniejszych porach dnia.

***

## Powiadomienie o zakończeniu przetwarzania

Po zakończeniu przetwarzania:

* Pasek postępu osiąga 100%
* W dzienniku debugowania pojawia się komunikat **„Przetwarzanie zakończone”**
* Przycisk Start ponownie staje się aktywny
* Wszystkie pliki wyjściowe znajdują się w podfolderze modelu aparatu

***

## Kolejne kroki

Po zakończeniu przetwarzania:

1. **Sprawdź wyniki** — zobacz [Zakończenie przetwarzania](finishing-the-processing.md)
2. **Sprawdź folder wyjściowy** — sprawdź, czy wszystkie pliki zostały poprawnie wyeksportowane
3. **Przejrzyj dziennik debugowania** — sprawdź, czy nie ma żadnych ostrzeżeń lub błędów
4. **Wyświetl podgląd przetworzonych obrazów** — użyj przeglądarki obrazów lub oprogramowania zewnętrznego

Aby uzyskać informacje na temat przeglądania i korzystania z przetworzonych wyników, zobacz [Zakończenie przetwarzania](finishing-the-processing.md).
