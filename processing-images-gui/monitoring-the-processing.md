# Monitorowanie przetwarzania

Po rozpoczęciu przetwarzania Chloros oferuje kilka sposobów monitorowania postępu, sprawdzania występujących problemów oraz zrozumienia, co dzieje się z danymi użytkownika. Na tej stronie wyjaśniono, jak śledzić przebieg przetwarzania oraz interpretować informacje dostarczane przez Chloros.

## Omówienie paska postępu

Pasek postępu w górnym nagłówku pokazuje stan przetwarzania w czasie rzeczywistym oraz procentowe zaawansowanie. Postęp jest przesyłany na żywo z zaplecza za pośrednictwem Server-Sent Events (SSE), dzięki czemu pasek odzwierciedla rzeczywisty stan działania potoku.

### Pasek postępu w trybie bezpłatnym

Dla użytkowników bez licencji Chloros+:

**Dwustopniowe wyświetlanie postępu:**

1.**Wykrywanie celów** – wyszukiwanie celów kalibracyjnych na obrazach
2. **Przetwarzanie** – stosowanie korekt i eksportowanie**Pasek postępu pokazuje:**

* Ogólny procent zaawansowania (0–100%)
* Nazwę bieżącego etapu
* Prostą wizualizację w postaci poziomego paska

### Pasek postępu Chloros+

Dla użytkowników posiadających licencję Chloros+:

**4-etapowe wyświetlanie postępu:**

1.**Wykrywanie** – wyszukiwanie celów kalibracyjnych
2. **Analiza** – badanie obrazów i przygotowanie potoku przetwarzania
3. **Kalibracja** – stosowanie korekcji winietowania i odbicia
4. **Eksport** – zapisywanie przetworzonych plików**Funkcje interaktywne:*** **Najedź kursorem** na pasek postępu, aby wyświetlić rozwijany panel z 4 etapami
* **Kliknij** pasek postępu, aby zamrozić/przypiąć rozwijany panel
* **Kliknij ponownie**, aby odblokować i automatycznie ukryć po odsunięciu myszy
* Każdy etap pokazuje indywidualny postęp (0–100%)

{% hint style="info" %}
**CLI parzystość**: podczas działania `chloros-cli process` te same cztery wątki zgłaszają statusy: Wykrywanie, Analiza, Przetwarzanie, Eksportowanie, a `chloros-cli export-status` pokazuje na żywo postęp eksportu wątku 4 z innego terminala. Zobacz [Podręcznik CLI](../reference/cli-reference.md).
{% endhint %}

***

## Omówienie poszczególnych etapów przetwarzania

{% hint style="info" %}
**Architektura potoku**: Te 4 etapy interfejsu graficznego odpowiadają [4-wątkowemu potokowi przetwarzania](../processing-architecture/processing-pipeline.md). W systemach z akceleracją GPU wątek 3 (kalibracja) korzysta z [dynamicznej adaptacji obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md), która optymalizuje przetwarzanie pod kątem konkretnego sprzętu.
{% endhint %}

### Etap 1: Wykrywanie (wykrywanie celów)

**Co się dzieje:**

* Chloros skanuje obrazy zaznaczone za pomocą pola wyboru „Cel” (wszystkie obrazy, jeśli żadne nie są zaznaczone)
* Algorytmy widzenia komputerowego identyfikują panele kalibracyjne
* Z każdego panelu wyodrębniane są wartości odbicia
* Rejestrowane są sygnatury czasowe celów w celu prawidłowego planowania kalibracji

**Czas trwania:**

* Z zaznaczonymi celami: 10–60 sekund
* Bez zaznaczonych celów: 5–30+ minut (skanowanie wszystkich obrazów)

**Wskaźnik postępu:**

* Wykrywanie: 0% → 100%
* Liczba zeskanowanych obrazów (liczone są tylko te, które faktycznie są skanowane)
* Liczba znalezionych celów

**Na co należy zwrócić uwagę:**

* Proces powinien zakończyć się szybko, jeśli cele są prawidłowo zaznaczone
* Jeśli trwa zbyt długo, cele mogą nie być zaznaczone
* Sprawdź dziennik debugowania pod kątem komunikatów „Target found”

### Etap 2: Analiza

**Co się dzieje:**

* Odczyt metadanych EXIF obrazów (znaczniki czasu, ustawienia ekspozycji)
* Określanie strategii kalibracji na podstawie znaczników czasu celów i dostępnych danych DAQ dotyczących promieniowania w dół
* Organizowanie kolejki przetwarzania obrazów
* Przygotowywanie procesów równoległych (tylko Chloros+)

**Czas trwania:** 5–30 sekund**Wskaźnik postępu:**

* Analiza: 0% → 100%
* Szybki etap, zazwyczaj kończy się szybko

**Na co należy zwrócić uwagę:**

* Postęp powinien przebiegać płynnie, bez przerw
* Ostrzeżenia o brakujących metadanych pojawią się w dzienniku debugowania

### Etap 3: Kalibracja

**Co się dzieje:*** **Debayering**: Konwersja wzorca RAW Bayera na 3 kanały (pomijane w przypadku modułów monochromatycznych LATTICE, z odpowiednią adnotacją)
* **Korekcja winietowania**: Usuwanie przyciemnienia na krawędziach obiektywu
* **Kalibracja odbicia**: Normalizacja względem wartości docelowych i/lub natężenia promieniowania padającego z systemu DAQ
* **Obliczanie indeksów**: Obliczanie indeksów wielospektralnych
* Przetwarzanie każdego obrazu w pełnym łańcuchu przetwarzania

**Czas trwania:** Większość całkowitego czasu przetwarzania (60–80%)**Wskaźnik postępu:**

* Kalibracja: 0% → 100%
* Aktualnie przetwarzany obraz
* Obrazy przetworzone / Łączna liczba obrazów

**Sposób przetwarzania:*** **Tryb swobodny**: Przetwarza po jednym obrazie na raz, sekwencyjnie
* **Tryb Chloros+**: Uruchamia pulę procesów dostosowującą się do sprzętu — 1–4 procesy działające jednocześnie w systemach z kartami graficznymi (w zależności od pamięci VRAM), jeden proces na każdy fizyczny rdzeń (minus jeden) w systemach opartych wyłącznie na procesorze. Zobacz [Dynamiczne dostosowanie obliczeniowe](../processing-architecture/dynamic-compute-adaptation.md)
* **Przyspieszenie GPU**: Znacznie przyspiesza ten etap**Na co należy zwrócić uwagę:**

* Stały postęp w liczbie obrazów
* Sprawdź dziennik debugowania pod kątem komunikatów o zakończeniu przetwarzania poszczególnych obrazów
* Ostrzeżenia dotyczące jakości obrazu lub problemów z kalibracją

### Etap 4: Eksport

**Co się dzieje:**

* Zapisywanie przetworzonych obrazów na dysk w wybranym formacie w miarę ich ukończenia
* **LATTICE**: każda klatka jest rozdzielana na wszystkie włączone produkty (po usunięciu efektu debayeringu / podgląd / promieniowanie / odbicie)
* Eksportowanie wielospektralnych obrazów indeksowych z kolorami LUT
* Tworzenie drzewa wyjściowego `<project>/<camera>/<format>/<Product>_Images/` — eksportowane pliki zachowują nazwę pliku źródłowego; folder identyfikuje produkt

**Czas trwania:** 10–20% całkowitego czasu przetwarzania**Wskaźnik postępu:**

* Eksportowanie: 0% → 100%
* Trwa zapisywanie plików
* Format eksportu i miejsce docelowe

**Na co należy zwrócić uwagę:**

* Ostrzeżenia o braku miejsca na dysku
* Błędy zapisu plików
* Zakończenie wszystkich skonfigurowanych operacji wyjściowych

***

## Zakładka „Dziennik debugowania”

Dziennik debugowania zawiera szczegółowe informacje na temat postępu przetwarzania oraz wszelkich napotkanych problemów. Komunikaty dotyczące uruchamiania zaplecza są również odtwarzane w konsoli dziennika, dzięki czemu dziennik przedstawia pełny obraz sytuacji, nawet jeśli otworzysz go z opóźnieniem.

### Dostęp do dziennika debugowania

1. Kliknij ikonę **Dziennik debugowania**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

na lewym pasku bocznym
2. Otworzy się panel dziennika wyświetlający komunikaty przetwarzania w czasie rzeczywistym
3. Dziennik przewija się automatycznie, aby wyświetlać najnowsze komunikaty

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Interpretacja komunikatów dziennika

Wiersze dziennika Chloros mają przedrostek w postaci tagów w nawiasach, określających nazwę podsystemu — na przykład `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najważniejszym elementem, o którym należy wiedzieć, jest **podsumowanie przebiegu**, wyświetlane na końcu każdego przebiegu (w tym przebiegów zatrzymanych):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Dodatkowe wiersze z wskazówkami `[RUN-SUMMARY]` pojawiają się zawsze, gdy coś wymaga wyjaśnienia — na przykład w przypadku przebiegu, który nie dał żadnych wyników, lub kamery, której żądany produkt został pominięty jako nieodpowiedni. Wiersze `[EXPORT-CHECK]` wyjaśniają pominięcia dla poszczególnych kamer (np. dlaczego kamera RGB nie otrzymała produktu promieniowania).

Ogólne poziomy ważności komunikatów (poniższe przykłady mają charakter ilustracyjny, a nie dosłowny):

#### Komunikaty informacyjne (białe/szare)

Normalne aktualizacje dotyczące przetwarzania: rozpoczęcie przetwarzania, wykrycie obiektów docelowych (wraz z liczbą paneli), postęp kalibracji poszczególnych obrazów, wyeksportowane pliki, zakończenie przetwarzania.

#### Komunikaty ostrzegawcze (żółte)

Problemy o charakterze niekrytycznym, które nie powodują zatrzymania przetwarzania — np. brak danych GPS w klatce, duża różnica w znacznikach czasu między obrazami obiektów docelowych lub niski kontrast w panelu kalibracyjnym.

**Działanie:** Należy przejrzeć ostrzeżenia po zakończeniu przetwarzania, ale nie należy przerywać przetwarzania

#### Komunikaty o błędach (Red)

Krytyczne problemy, które mogą spowodować niepowodzenie przetwarzania — np. zapełniony dysk, uszkodzony plik obrazu lub brak wykrytych obiektów podczas kalibracji odbicia.

**Działanie:** Zatrzymaj przetwarzanie, usuń błąd, uruchom ponownie

### Typowe sytuacje opisane w dzienniku

| Sytuacja                             | Znaczenie                                       | Wymagane działanie                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Obiekt wykryty w \[nazwa pliku]        | Obiekt kalibracyjny znaleziony pomyślnie         | Brak — stan normalny                                         |
| Linie postępu dla poszczególnych obrazów              | Aktualizacja bieżącego postępu                       | Brak — stan normalny                                         |
| Nie znaleziono celów                      | Nie wykryto celów kalibracyjnych               | Oznacz obrazy celów lub wyłącz kalibrację odbicia |
| Niewystarczająca ilość miejsca na dysku               | Za mało miejsca na dane wyjściowe                 | Zwolnij miejsce na dysku                                    |
| Pomijanie uszkodzonego pliku               | Plik obrazu jest uszkodzony                         | Skopiuj ponownie plik z karty SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Nie można przetworzyć zdjęcia bez klatki surowej | Wykonaj ponowne zdjęcie z klatką surową, lub użyj CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Przetwarzanie nie wygenerowało żadnych produktów obrazowych — zgłoszono jako niepowodzenie wraz z wskazówkami | Przeczytaj wiersze z wskazówkami; sprawdź, co zostało pominięte i dlaczego |

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
* Pozostałe rdzenie są w stanie bezczynności lub dostępne
* System nadal reaguje

**Tryb równoległy Chloros+:**

* Wiele rdzeni o wysokim obciążeniu — ich liczba zależy od strategii wybranej w ramach [Dynamicznej adaptacji obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md)
* System może wydawać się mniej responsywny

**Aby monitorować:**

* Windows Menedżer zadań (Ctrl+Shift+Esc)
* Zakładka „Wydajność” → sekcja „Procesor”
* Poszukaj procesów „Chloros” lub „chloros-backend”

### Zużycie pamięci (RAM)

**Typowe zużycie:**

* Małe projekty (&lt; 100 obrazów): 2–4 GB
* Średnie projekty (100–500 obrazów): 4–8 GB
* Duże projekty (ponad 500 obrazów): 8–16 GB
* Tryb równoległy Chloros+ zużywa więcej pamięci RAM

**W przypadku niedoboru pamięci:**

* Przetwarzaj mniejsze partie danych
* Zamknij inne aplikacje
* Zwiększ pojemność pamięci RAM, jeśli regularnie przetwarzasz duże zbiory danych

### Wykorzystanie procesora graficznego (GPU) (Chloros+ z CUDA)

Gdy przyspieszenie GPU jest włączone:

* Karta graficzna NVIDIA wykazuje wysokie wykorzystanie (60–90%)
* Wzrasta zużycie pamięci VRAM (wymagane co najmniej 4 GB pamięci VRAM; co najmniej 7 GB w przypadku równoczesnego debayeringu z uwzględnieniem tekstur)
* Etap kalibracji przebiega znacznie szybciej

**Aby monitorować:**

* Ikona NVIDIA w zasobniku systemowym
* Menedżer zadań → Wydajność → Karta graficzna
* GPU-Z lub podobne narzędzie do monitorowania

### Operacje wejścia/wyjścia na dysku

**Czego można się spodziewać:**

* Wysoki poziom odczytu z dysku podczas etapu analizy
* Wysoki poziom zapisu na dysku podczas etapu eksportu
* Dyski SSD są znacznie szybsze niż dyski HDD

**Wskazówka dotycząca wydajności:**

* Jeśli to możliwe, używaj dysku SSD jako lokalizacji folderu projektu
* Unikaj dysków sieciowych w przypadku dużych zbiorów danych
* Upewnij się, że dysk nie jest prawie pełny (wpływa to na prędkość zapisu)

***

## Wykrywanie problemów podczas przetwarzania

### Sygnały ostrzegawcze

**Postęp przetwarzania zatrzymuje się (brak zmian przez ponad 5 minut):**

* Sprawdź dziennik debugowania pod kątem błędów
* Sprawdź dostępną przestrzeń dyskową
* Sprawdź w Menedżerze zadań, czy proces Chloros jest uruchomiony

**Częste pojawianie się komunikatów o błędach:**

* Zatrzymaj przetwarzanie i przejrzyj błędy
* Typowe przyczyny: brak miejsca na dysku, uszkodzone pliki, problemy z pamięcią
* Zobacz sekcję „Rozwiązywanie problemów” poniżej

**System przestaje reagować:**

* Tryb równoległy Chloros+ zużywa zbyt wiele zasobów
* Rozważ zmniejszenie liczby równoległych zadań lub modernizację sprzętu
* Tryb swobodny zużywa mniej zasobów

### Kiedy należy zatrzymać przetwarzanie

Zatrzymaj przetwarzanie, jeśli zauważysz:

* ❌ Błędy „Dysk pełny” lub „Nie można zapisać pliku”
* ❌ Powtarzające się błędy uszkodzenia plików obrazów
* ❌ System całkowicie zawiesił się (nie odpowiada)
* ❌ Zorientowałeś się, że skonfigurowano nieprawidłowe ustawienia
* ❌ Zaimportowano niewłaściwe obrazy

**Jak zatrzymać:**

1. Kliknij**przycisk „Stop”** (zastępuje przycisk „Start”) — wystarczy raz
2. Pasek wyświetla komunikat „Zatrzymywanie...”, podczas gdy przetwarzany obraz jest kończony, a następnie proces kończy się w stanie zatrzymania
3. Produkty, które zostały już wyeksportowane, pozostają na dysku; w dzienniku wyświetla się dokładny komunikat `[RUN-SUMMARY]` opisujący, co zostało zakończone
4. Napraw problemy i uruchom ponownie — proces rozpocznie się od początku

***

## Rozwiązywanie problemów podczas przetwarzania

### Przetwarzanie przebiega bardzo wolno

**Możliwe przyczyny:**

* Nieoznaczone obrazy docelowe (skanowanie wszystkich obrazów)
* Pamięć HDD zamiast SSD
* Niewystarczające zasoby systemowe
* Skonfigurowano zbyt wiele indeksów
* Dostęp do dysku sieciowego

**Rozwiązania:**

1. Jeśli proces dopiero się rozpoczął i znajduje się w fazie wykrywania: zatrzymaj, oznacz obiekty docelowe, uruchom ponownie
2. Na przyszłość: użyj dysku SSD, zmniejsz liczbę indeksów, zmodernizuj sprzęt
3. Rozważ użycie CLI do przetwarzania wsadowego dużych zbiorów danych

### Ostrzeżenia dotyczące „miejsca na dysku”

**Rozwiązania:**

1. Natychmiast zwolnij miejsce na dysku
2. Przenieś projekt na dysk z większą ilością miejsca
3. Zmniejsz liczbę indeksów do eksportu
4. Wyłącz niepotrzebne produkty eksportowe LATTICE (Ustawienia projektu → Przetwarzanie)
5. Używaj formatu JPG zamiast TIFF (mniejsze pliki)

### Częste komunikaty o „uszkodzonych plikach”

**Rozwiązania:**

1. Skopiuj ponownie zdjęcia z karty SD, aby zapewnić ich integralność
2. Sprawdź kartę SD pod kątem błędów
3. Usuń uszkodzone pliki z projektu
4. Kontynuuj przetwarzanie pozostałych zdjęć

### Przegrzanie systemu / ograniczanie wydajności

**Rozwiązania:**

1. Zapewnij odpowiednią wentylację
2. Oczyść otwory wentylacyjne komputera z kurzu
3. Zmniejsz obciążenie przetwarzania (użyj trybu Free zamiast Chloros+)
4. Przeprowadzaj przetwarzanie w chłodniejszych porach dnia

***

## Powiadomienie o zakończeniu przetwarzania

Po zakończeniu przetwarzania:

* Pasek postępu osiąga 100%
* W dzienniku debugowania pojawiają się wiersze `[RUN-SUMMARY]` z ostatecznymi liczbami
* Przycisk „Start” staje się ponownie aktywny
* Wszystkie pliki wyjściowe znajdują się w drzewie wyjściowym projektu dla poszczególnych kamer: `<project>/<camera>/<format>/<Product>_Images/`

***

## Kolejne kroki

Po zakończeniu przetwarzania:

1. **Przejrzyj wyniki** – zobacz [Zakończenie przetwarzania](finishing-the-processing.md)
2. **Sprawdź folder wyjściowy** – upewnij się, że wszystkie pliki zostały poprawnie wyeksportowane
3. **Przejrzyj dziennik debugowania** — sprawdź, czy nie ma żadnych ostrzeżeń ani błędów
4. **Wyświetl podgląd przetworzonych obrazów** — użyj przeglądarki obrazów lub oprogramowania zewnętrznego

Aby uzyskać informacje na temat przeglądania i wykorzystywania przetworzonych wyników, zapoznaj się z sekcją [Zakończenie przetwarzania](finishing-the-processing.md).
