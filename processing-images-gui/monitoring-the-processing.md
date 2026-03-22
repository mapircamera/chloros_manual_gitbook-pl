# Monitorowanie przetwarzania

Po rozpoczęciu przetwarzania Chloros oferuje kilka sposobów monitorowania postępów, sprawdzania ewentualnych problemów oraz śledzenia tego, co dzieje się z Twoim zbiorem danych. Na tej stronie wyjaśniono, jak śledzić przebieg przetwarzania i interpretować informacje dostarczane przez Chloros.

## Omówienie paska postępu

Pasek postępu w górnym nagłówku pokazuje status przetwarzania w czasie rzeczywistym oraz procent ukończenia.

### Pasek postępu w trybie bezpłatnym

Dla użytkowników bez licencji Chloros+:

**2-etapowe wyświetlanie postępu:**

1.**Wykrywanie celów** – wyszukiwanie celów kalibracyjnych na obrazach
2. **Przetwarzanie** – stosowanie korekt i eksportowanie**Pasek postępu pokazuje:**

* Ogólny procent ukończenia (0–100%)
* Nazwę bieżącego etapu
* Prostą wizualizację w postaci poziomego paska

### Pasek postępu Chloros+

Dla użytkowników z licencją Chloros+:

**4-etapowy wskaźnik postępu:**

1.**Wykrywanie** – wyszukiwanie celów kalibracyjnych
2. **Analiza** – badanie obrazów i przygotowywanie potoku
3. **Kalibracja** – stosowanie korekcji winietowania i odbicia
4. **Eksport** – zapisywanie przetworzonych plików**Funkcje interaktywne:*** **Najedź kursorem** na pasek postępu, aby wyświetlić rozbudowany panel 4-etapowy
* **Kliknij** pasek postępu, aby zatrzymać/przypiąć rozbudowany panel
* **Kliknij ponownie**, aby odblokować i automatycznie ukryć po odsunięciu myszy
* Każdy etap pokazuje indywidualny postęp (0–100%)

***

## Zrozumienie poszczególnych etapów przetwarzania

{% hint style="info" %}
**Architektura potoku**: Te 4 etapy GUI odpowiadają [4-wątkowemu potokowi przetwarzania](../processing-architecture/processing-pipeline.md). W systemach z akceleracją GPU wątek 3 (kalibracja) korzysta z [dynamicznej adaptacji obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md), która optymalizuje przetwarzanie pod kątem konkretnego sprzętu.
{% endhint %}

### Etap 1: Wykrywanie (wykrywanie celów)

**Co się dzieje:**

* Chloros skanuje obrazy oznaczone polem wyboru „Cel”
* Algorytmy wizji komputerowej identyfikują 4 panele kalibracyjne
* Wartości odbicia światła są wyodrębniane z każdego panelu
* Rejestrowane są znaczniki czasu celów w celu prawidłowego planowania kalibracji

**Czas trwania:**

* Z zaznaczonymi celami: 10–60 sekund
* Bez zaznaczonych celów: 5–30+ minut (skanuje wszystkie obrazy)

**Wskaźnik postępu:**

* Wykrywanie: 0% → 100%
* Liczba zeskanowanych obrazów
* Liczba znalezionych celów

**Na co należy zwrócić uwagę:**

* Proces powinien zakończyć się szybko, jeśli cele są prawidłowo oznaczone
* Jeśli trwa zbyt długo, cele mogą nie być oznaczone
* Sprawdź dziennik debugowania pod kątem komunikatów „Target found”

### Etap 2: Analiza

**Co się dzieje:**

* Odczyt metadanych EXIF obrazu (znaczniki czasu, ustawienia ekspozycji)
* Określanie strategii kalibracji na podstawie znaczników czasu celów
* Organizowanie kolejki przetwarzania obrazów
* Przygotowywanie procesów równoległych (tylko Chloros+)

**Czas trwania:** 5–30 sekund**Wskaźnik postępu:**

* Analiza: 0% → 100%
* Szybki etap, zazwyczaj kończy się szybko

**Na co należy zwrócić uwagę:**

* Postęp powinien przebiegać płynnie, bez przerw
* Ostrzeżenia o brakujących metadanych pojawią się w dzienniku debugowania

### Etap 3: Kalibracja

**Co się dzieje:*** **Debayering**: Konwersja wzorca RAW Bayer na 3 kanały
* **Korekcja winietowania**: Usuwanie zaciemnienia na krawędziach obiektywu
* **Kalibracja odbicia**: Normalizacja względem wartości docelowych
* **Obliczanie wskaźników**: Obliczanie wskaźników wielospektralnych
* Przetwarzanie każdego obrazu w pełnym łańcuchu przetwarzania

**Czas trwania:** Większość całkowitego czasu przetwarzania (60–80%)**Wskaźnik postępu:**

* Kalibracja: 0% → 100%
* Aktualnie przetwarzany obraz
* Obrazy przetworzone / Łączna liczba obrazów

**Sposób przetwarzania:*** **Tryb swobodny**: Przetwarza po jednym obrazie na raz, sekwencyjnie
* **Tryb Chloros+**: Przetwarza do 16 obrazów jednocześnie
* **Przyspieszenie GPU**: Znacznie przyspiesza ten etap**Na co należy zwrócić uwagę:**

* Stały postęp w liczbie obrazów
* Sprawdź dziennik debugowania pod kątem komunikatów o zakończeniu przetwarzania poszczególnych obrazów
* Ostrzeżenia dotyczące jakości obrazu lub problemów z kalibracją

### Etap 4: Eksportowanie

**Co się dzieje:**

* Zapisywanie skalibrowanych obrazów na dysk w wybranym formacie
* Eksportowanie wielospektralnych obrazów indeksowych z kolorami LUT
* Tworzenie podfolderów modeli kamer
* Zachowanie oryginalnych nazw plików z odpowiednimi rozszerzeniami

**Czas trwania:** 10–20% całkowitego czasu przetwarzania**Wskaźnik postępu:**

* Eksportowanie: 0% → 100%
* Zapisywane pliki
* Format eksportu i miejsce docelowe

**Na co należy zwrócić uwagę:**

* Ostrzeżenia dotyczące miejsca na dysku
* Błędy zapisu plików
* Zakończenie wszystkich skonfigurowanych operacji wyjściowych

***

## Zakładka Dziennik debugowania

Dziennik debugowania zawiera szczegółowe informacje na temat postępu przetwarzania oraz wszelkich napotkanych problemów.

### Dostęp do dziennika debugowania

1. Kliknij ikonę **Dziennik debugowania** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na lewym pasku bocznym
2. Otworzy się panel dziennika z komunikatami przetwarzania w czasie rzeczywistym
3. Panel przewija się automatycznie, aby wyświetlić najnowsze komunikaty

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

**Działanie:** Sprawdź ostrzeżenia po zakończeniu przetwarzania, ale nie przerywaj go

#### Komunikaty o błędach (Red)

Krytyczne problemy, które mogą spowodować niepowodzenie przetwarzania:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Działanie:** Zatrzymaj przetwarzanie, usuń błąd, uruchom ponownie

### Typowe komunikaty dziennika

| Komunikat                          | Znaczenie                                | Wymagane działanie                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Wykryto cel w \[nazwa pliku]” | Cel kalibracji znaleziony pomyślnie  | Brak – normalnie                                         |
| „Przetwarzanie obrazu X z Y”        | Aktualizacja bieżącego postępu                | Brak – normalnie                                         |
| „Nie znaleziono celów”               | Nie wykryto celów kalibracji        | Oznacz obrazy celów lub wyłącz kalibrację odbicia |
| „Niewystarczająca ilość miejsca na dysku”        | Za mało miejsca na dane wyjściowe          | Zwolnij miejsce na dysku                                    |
| „Pomijanie uszkodzonego pliku”        | Plik obrazu jest uszkodzony                  | Skopiuj plik ponownie z karty SD                             |
| „Zastosowano dane PPK”               | Zastosowano poprawki GPS z pliku .daq | Brak – normalnie                                         |

### Kopiowanie danych dziennika

Aby skopiować dziennik w celu rozwiązywania problemów lub uzyskania pomocy technicznej:

1. Otwórz panel dziennika debugowania
2. Kliknij przycisk **„Kopiuj dziennik”** (lub kliknij prawym przyciskiem myszy → Zaznacz wszystko)
3. Wklej do pliku tekstowego lub wiadomości e-mail
4. W razie potrzeby wyślij do pomocy technicznej MAPIR

***

## Monitorowanie zasobów systemowych

### Wykorzystanie procesora

**Tryb swobodny:**

* 1 rdzeń procesora na poziomie ~100%
* Pozostałe rdzenie w stanie bezczynności lub dostępne
* System pozostaje responsywny

**Tryb równoległy Chloros+:**

* Wiele rdzeni na poziomie 80–100% (do 16 rdzeni)
* Wysokie ogólne wykorzystanie procesora
* System może wydawać się mniej responsywny

**Aby monitorować:**

* Menedżer zadań Windows (Ctrl+Shift+Esc)
* Zakładka Wydajność → sekcja Procesor
* Poszukaj procesów „Chloros” lub „chloros-backend”

### Wykorzystanie pamięci (RAM)

**Typowe zużycie:**

* Małe projekty (&lt; 100 obrazów): 2–4 GB
* Średnie projekty (100–500 obrazów): 4–8 GB
* Duże projekty (ponad 500 obrazów): 8–16 GB
* Tryb równoległy Chloros+ zużywa więcej pamięci RAM

**Jeśli pamięć jest na wyczerpaniu:**

* Przetwarzaj mniejsze partie
* Zamknij inne aplikacje
* Zwiększ pamięć RAM, jeśli regularnie przetwarzasz duże zbiory danych

### Wykorzystanie GPU (Chloros+ z CUDA)

Gdy przyspieszenie GPU jest włączone:

* Procesor graficzny NVIDIA wykazuje wysokie wykorzystanie (60–90%)
* Wzrasta zużycie pamięci VRAM (wymagane co najmniej 4 GB pamięci VRAM)
* Etap kalibracji przebiega znacznie szybciej

**Aby monitorować:**

* Ikona NVIDIA w zasobniku systemowym
* Menedżer zadań → Wydajność → GPU
* GPU-Z lub podobne narzędzie do monitorowania

### Operacje wejścia/wyjścia na dysku

**Czego można się spodziewać:**

* Wysokie obciążenie odczytu dysku podczas etapu analizy
* Wysokie obciążenie zapisu dysku podczas etapu eksportu
* Dyski SSD są znacznie szybsze niż dyski HDD

**Wskazówka dotycząca wydajności:**

* Jeśli to możliwe, używaj dysku SSD dla folderu projektu
* Unikaj dysków sieciowych w przypadku dużych zbiorów danych
* Upewnij się, że dysk nie jest prawie zapełniony (wpływa to na prędkość zapisu)

***

## Wykrywanie problemów podczas przetwarzania

### Sygnały ostrzegawcze

**Postęp przetwarzania zatrzymuje się (brak zmian przez ponad 5 minut):**

* Sprawdź dziennik debugowania pod kątem błędów
* Sprawdź dostępną przestrzeń dyskową
* Sprawdź w Menedżerze zadań, czy proces Chloros jest uruchomiony

**Często pojawiają się komunikaty o błędach:**

* Zatrzymaj przetwarzanie i przejrzyj błędy
* Typowe przyczyny: brak miejsca na dysku, uszkodzone pliki, problemy z pamięcią
* Zobacz sekcję Rozwiązywanie problemów poniżej

**System przestaje reagować:**

* Tryb równoległy Chloros+ zużywa zbyt dużo zasobów
* Rozważ zmniejszenie liczby zadań wykonywanych jednocześnie lub modernizację sprzętu
* Tryb swobodny zużywa mniej zasobów

### Kiedy należy zatrzymać przetwarzanie

Zatrzymaj przetwarzanie, jeśli zauważysz:

* ❌ Błędy „Dysk pełny” lub „Nie można zapisać pliku”
* ❌ Powtarzające się błędy uszkodzenia plików obrazów
* ❌ System całkowicie zawiesił się (nie odpowiada)
* ❌ Zorientowałeś się, że skonfigurowano nieprawidłowe ustawienia
* ❌ Zaimportowano nieprawidłowe obrazy

**Jak zatrzymać:**

1. Kliknij**przycisk Zatrzymaj/Anuluj** (zastępuje przycisk Start)
2. Przetwarzanie zostanie zatrzymane, postęp zostanie utracony
3. Napraw problemy i zacznij od początku

***

## Rozwiązywanie problemów podczas przetwarzania

### Przetwarzanie przebiega bardzo wolno

**Możliwe przyczyny:**

* Nieoznaczone obrazy docelowe (skanowanie wszystkich obrazów)
* Dysk HDD zamiast pamięci SSD
* Niewystarczające zasoby systemowe
* Skonfigurowano zbyt wiele indeksów
* Dostęp do dysku sieciowego

**Rozwiązania:**

1. Jeśli proces właśnie się rozpoczął i jest na etapie wykrywania: Anuluj, zaznacz obiekty docelowe, uruchom ponownie
2. Na przyszłość: Użyj dysku SSD, zmniejsz liczbę indeksów, zmodernizuj sprzęt
3. Rozważ użycie CLI do przetwarzania wsadowego dużych zbiorów danych

### Ostrzeżenia o „miejscu na dysku”

**Rozwiązania:**

1. Natychmiast zwolnij miejsce na dysku
2. Przenieś projekt na dysk z większą ilością miejsca
3. Zmniejsz liczbę indeksów do eksportu
4. Użyj formatu JPG zamiast TIFF (mniejsze pliki)

### Częste komunikaty o „uszkodzonych plikach”

**Rozwiązania:**

1. Skopiuj ponownie zdjęcia z karty SD, aby zapewnić ich integralność
2. Sprawdź kartę SD pod kątem błędów
3. Usuń uszkodzone pliki z projektu
4. Kontynuuj przetwarzanie pozostałych zdjęć

### Przegrzanie systemu / Ograniczanie wydajności

**Rozwiązania:**

1. Zapewnij odpowiednią wentylację
2. Oczyść otwory wentylacyjne komputera z kurzu
3. Zmniejsz obciążenie przetwarzania (użyj trybu Free zamiast Chloros+)
4. Przetwarzaj w chłodniejszych porach dnia

***

## Powiadomienie o zakończeniu przetwarzania

Po zakończeniu przetwarzania:

* Pasek postępu osiąga 100%
* W dzienniku debugowania pojawia się komunikat **„Przetwarzanie zakończone”**
* Przycisk Start staje się ponownie aktywny
* Wszystkie pliki wyjściowe znajdują się w podfolderze modelu aparatu

***

## Kolejne kroki

Po zakończeniu przetwarzania:

1. **Sprawdź wyniki** — zobacz [Zakończenie przetwarzania](finishing-the-processing.md)
2. **Sprawdź folder wyjściowy** — upewnij się, że wszystkie pliki zostały poprawnie wyeksportowane
3. **Przejrzyj dziennik debugowania** — sprawdź, czy nie ma żadnych ostrzeżeń lub błędów
4. **Wyświetl podgląd przetworzonych obrazów** — użyj przeglądarki obrazów lub oprogramowania zewnętrznego

Aby uzyskać informacje na temat przeglądania i korzystania z przetworzonych wyników, zobacz [Zakończenie przetwarzania](finishing-the-processing.md).
