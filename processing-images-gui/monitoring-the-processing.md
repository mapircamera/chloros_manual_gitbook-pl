# Monitorowanie przetwarzania

Po rozpoczęciu przetwarzania Chloros oferuje kilka sposobów monitorowania postępów, sprawdzania ewentualnych problemów oraz zrozumienia, co dzieje się z danymi użytkownika. Na tej stronie wyjaśniono, jak śledzić przebieg przetwarzania i interpretować informacje dostarczane przez Chloros.

## Omówienie paska postępu

Pasek postępu w górnym nagłówku pokazuje stan przetwarzania w czasie rzeczywistym oraz procent zaawansowania. Postęp jest przesyłany na żywo z zaplecza za pośrednictwem Server-Sent Events (SSE), dzięki czemu pasek odzwierciedla aktualną aktywność potoku przetwarzania.

### Pasek postępu w trybie bezpłatnym

Dla użytkowników bez licencji Chloros+:

**2-etapowe wyświetlanie postępu:**

1.**Wykrywanie celów** – wyszukiwanie celów kalibracyjnych na obrazach
2. **Przetwarzanie** – stosowanie korekt i eksportowanie**Pasek postępu pokazuje:**

* Ogólny procent ukończenia (0–100%)
* Nazwę bieżącego etapu
* Prostą wizualizację w postaci poziomego paska

### Pasek postępu Chloros+

Dla użytkowników posiadających licencję Chloros+:

**4-etapowe wyświetlanie postępu:**

1.**Wykrywanie** – wyszukiwanie celów kalibracyjnych
2. **Analiza** – badanie obrazów i przygotowywanie potoku przetwarzania
3. **Kalibracja** – stosowanie korekcji winietowania i odbicia
4. **Eksport** – zapisywanie przetworzonych plików

**Funkcje interaktywne:**

* **Najedź kursorem** na pasek postępu, aby wyświetlić rozwijany panel z 4 etapami
* **Kliknij** pasek postępu, aby zamrozić/przypiąć rozwijany panel
* **Kliknij ponownie**, aby odblokować panel i przywrócić automatyczne ukrywanie po odsunięciu kursora
* Każdy etap pokazuje indywidualny postęp (0–100%)

{% hint style="info" %}
**Zgodność z CLI**: podczas działania `chloros-cli process` te same cztery wątki zgłaszają statusy „Wykrywanie”, „Analiza”, „Przetwarzanie” i „Eksportowanie”, a `chloros-cli export-status` pokazuje na żywo postęp eksportu w wątku 4 z innego terminala. Zobacz [Podręcznik CLI](../reference/cli-reference.md).
{% endhint %}

***

## Opis poszczególnych etapów przetwarzania

{% hint style="info" %}
**Architektura potoku**: Te 4 etapy interfejsu graficznego odpowiadają [4-wątkowemu potokowi przetwarzania](../processing-architecture/processing-pipeline.md). W systemach z akceleracją GPU wątek 3 (kalibracja) korzysta z [dynamicznej adaptacji obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md), która optymalizuje przetwarzanie pod kątem konkretnego sprzętu.
{% endhint %}

### Etap 1: Wykrywanie (wykrywanie obiektów docelowych)

**Co się dzieje:**

* Chloros skanuje obrazy zaznaczone polem wyboru „Cel” (wszystkie obrazy, jeśli żadne nie są zaznaczone)
* Algorytmy wizji komputerowej identyfikują panele kalibracyjne
* Wartości odbicia światła są wyodrębniane z każdego panelu
* Rejestrowane są znaczniki czasu celów w celu prawidłowego zaplanowania kalibracji

**Czas trwania:**

* Z zaznaczonymi celami: 10–60 sekund
* Bez zaznaczonych celów: 5–30+ minut (skanowanie wszystkich obrazów)

**Wskaźnik postępu:**

* Wykrywanie: 0% → 100%
* Liczba zeskanowanych zdjęć (liczy się tylko te zdjęcia, które faktycznie są skanowane)
* Liczba znalezionych celów

**Na co należy zwrócić uwagę:**

* Proces powinien zakończyć się szybko, jeśli cele są prawidłowo oznaczone
* Jeśli trwa zbyt długo, cele mogą nie być oznaczone
* Sprawdź dziennik debugowania pod kątem komunikatów „Target found”

### Etap 2: Analiza

**Co się dzieje:**

* Odczyt metadanych EXIF obrazów (znaczniki czasu, ustawienia ekspozycji)
* Określanie strategii kalibracji na podstawie znaczników czasu celów i dostępnych danych DAQ dotyczących promieniowania w dół
* Organizowanie kolejki przetwarzania obrazów
* Przygotowywanie procesów przetwarzania równoległego (tylko Chloros+)

**Czas trwania:** 5–30 sekund**Wskaźnik postępu:**

* Analiza: 0% → 100%
* Szybki etap, zazwyczaj kończy się szybko

**Na co należy zwrócić uwagę:**

* Postęp powinien przebiegać płynnie, bez przerw
* Ostrzeżenia o brakujących metadanych pojawią się w dzienniku debugowania

### Etap 3: Kalibracja

**Co się dzieje:**

* **Debayering**: Konwersja wzorca RAW Bayera na 3 kanały (pomijana w przypadku modułów monochromatycznych LATTICE, z odpowiednią adnotacją)
* **Korekcja winietowania**: Usuwanie przyciemnienia na krawędziach obiektywu
* **Kalibracja odbicia**: Normalizacja względem wartości docelowych i/lub sygnału DAQ
* **Obliczanie wskaźników**: Obliczanie wskaźników wielospektralnych
* Przetwarzanie każdego obrazu w ramach pełnego łańcucha przetwarzania

**Czas trwania:** Większość całkowitego czasu przetwarzania (60–80%)**Wskaźnik postępu:**

* Kalibracja: 0% → 100%
* Aktualnie przetwarzany obraz
* Obrazy przetworzone / Łączna liczba obrazów

**Sposób przetwarzania:**

* **Tryb swobodny**: Przetwarza po jednym obrazie na raz, sekwencyjnie
* **Tryb Chloros+**: Uruchamia pulę procesów dostosowującą się do sprzętu — 1–4 procesy działające równolegle w systemach z kartami graficznymi (w zależności od pamięci VRAM), jeden proces na każdy fizyczny rdzeń (minus jeden) w systemach opartych wyłącznie na procesorze. Zobacz [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md)
* **Przyspieszenie GPU**: Znacznie przyspiesza ten etap**Na co należy zwrócić uwagę:**

* Stały postęp w liczbie obrazów
* Sprawdź dziennik debugowania pod kątem komunikatów o zakończeniu przetwarzania poszczególnych obrazów
* Ostrzeżenia dotyczące jakości obrazów lub problemów z kalibracją

### Etap 4: Eksportowanie

**Co się dzieje:**

* Zapisywanie przetworzonych obrazów na dysk w wybranym formacie w miarę ich ukończenia
* **LATTICE**: każda klatka jest rozdzielana na wszystkie włączone produkty (debayering / podgląd / promieniowanie / odbicie)
* Eksportowanie wielospektralnych obrazów indeksowych z kolorami LUT
* Tworzenie drzewa wyjściowego `<project>/<camera>/<format>/<Product>_Images/` — eksportowane pliki zachowują nazwę pliku źródłowego; folder identyfikuje produkt

**Czas trwania:** 10–20% całkowitego czasu przetwarzania**Wskaźnik postępu:**

* Eksportowanie: 0% → 100%
* Trwa zapisywanie plików
* Format eksportu i miejsce docelowe

**Na co należy zwrócić uwagę:**

* Ostrzeżenia dotyczące miejsca na dysku
* Błędy zapisu plików
* Zakończenie wszystkich skonfigurowanych operacji wyjściowych

***

## Zakładka „Dziennik debugowania”

Dziennik debugowania zawiera szczegółowe informacje na temat postępu przetwarzania oraz wszelkich napotkanych problemów. Komunikaty dotyczące uruchamiania zaplecza są również odtwarzane w konsoli dziennika, dzięki czemu dziennik przedstawia pełny obraz sytuacji, nawet jeśli otworzysz go z opóźnieniem.

### Dostęp do dziennika debugowania

1. Kliknij ikonę **Dziennik debugowania** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

na lewym pasku bocznym
2. Otworzy się panel dziennika wyświetlający komunikaty przetwarzania w czasie rzeczywistym
3. Dziennik przewija się automatycznie, aby wyświetlać najnowsze komunikaty

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Interpretacja komunikatów dziennika Wiersze dziennika

Chloros a mają przedrostek w postaci tagów w nawiasach, określających nazwę podsystemu — na przykład `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najważniejsze do zapamiętania jest **podsumowanie przebiegu**, wyświetlane na końcu każdego przebiegu (w tym przebiegów zatrzymanych):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Dodatkowe wiersze z wskazówkami `[RUN-SUMMARY]` pojawiają się zawsze, gdy coś wymaga wyjaśnienia — na przykład w przypadku przebiegu, który nie dał żadnych wyników, lub kamera, dla której żądany produkt został pominięty jako nieodpowiedni. Wiersze `[EXPORT-CHECK]` wyjaśniają pominięcia dla poszczególnych kamer (np. dlaczego kamera RGB nie otrzymała produktu promieniowania).

Ogólne poziomy ważności komunikatów (poniższe przykłady mają charakter ilustracyjny, a nie dosłowny):

#### Komunikaty informacyjne (białe/szare)

Normalne aktualizacje dotyczące przetwarzania: rozpoczęcie przetwarzania, wykrycie celów (z liczbą paneli), postęp kalibracji poszczególnych obrazów, wyeksportowane pliki, zakończenie przetwarzania.

#### Komunikaty ostrzegawcze (żółte)

Problemy o charakterze niekrytycznym, które nie zatrzymują przetwarzania — np. brak danych GPS w klatce, duża luka w znacznikach czasu między obrazami obiektów docelowych lub niski kontrast w panelu kalibracyjnym.

**Działanie:** Sprawdź ostrzeżenia po zakończeniu przetwarzania, ale nie przerywaj go

#### Komunikaty o błędach (kolor Red)

Krytyczne problemy, które mogą spowodować niepowodzenie przetwarzania — np. zapełniony dysk, uszkodzony plik obrazu lub brak wykrytych obiektów podczas kalibracji odbicia.

**Działanie:** Zatrzymaj przetwarzanie, usuń błąd, uruchom ponownie

### Typowe sytuacje opisane w dzienniku

| Sytuacja                             | Znaczenie                                       | Wymagane działanie                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Obiekt wykryty w \[nazwa pliku]        | Obiekt kalibracyjny znaleziony pomyślnie         | Brak — normalnie                                         |
| Linie postępu dla poszczególnych obrazów              | Aktualizacja bieżącego postępu                       | Brak — normalnie                                         |
| Nie znaleziono celów                      | Nie wykryto celów kalibracyjnych               | Oznacz obrazy celów lub wyłącz kalibrację odbicia |
| Niewystarczająca ilość miejsca na dysku               | Za mało miejsca na dane wyjściowe                 | Zwolnij miejsce na dysku                                    |
| Pomijanie uszkodzonego pliku               | Plik obrazu jest uszkodzony                         | Skopiuj plik ponownie z karty SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Nie można przetworzyć przechwycenia bez klatki surowej | Przechwyć ponownie z klatką surową, lub skorzystaj z interfejsu CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Przetwarzanie nie wygenerowało żadnych produktów obrazowych — zgłoszono jako błąd z wskazówkami | Przeczytaj wiersze z wskazówkami; sprawdź, co zostało pominięte i dlaczego |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Długie ścieżki w systemie Windows są wyłączone, a folder projektu jest zbyt głęboko zagnieżdżony, co przekracza limit 260 znaków dla ścieżki; nic nie zostało przetworzone | Użyj krótszego folderu projektu (komunikat podaje najdłuższą dopuszczalną ścieżkę) lub włącz obsługę długich ścieżek w systemie Windows i uruchom ponownie program Chloros — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Nie udało się zapisać maski ograniczeń produktu LATTICE; sam produkt działa poprawnie i wyświetla komunikat `Chloros:UnmixClampMask=unavailable` | Sprawdź ilość miejsca na dysku i uprawnienia do folderu, a następnie ponownie przetwórz dane, jeśli potrzebujesz tej maski |

### Kopiowanie danych dziennika

Aby skopiować dziennik w celu rozwiązywania problemów lub uzyskania pomocy technicznej:

1. Otwórz panel „Dziennik debugowania”
2. Kliknij przycisk **Kopiuj**(kopiuje dziennik do schowka) lub**Pobierz**, aby zapisać go jako plik tekstowy
3. Wklej do pliku tekstowego lub wiadomości e-mail
4. W razie potrzeby wyślij do działu pomocy technicznej MAPIR



### Zgłaszanie błędu z karty dziennika

Nowość w wersji 1.3.0: ikona błędu (**Zgłoś błąd**) u góry karty „Dziennik debugowania” wysyła zgłoszenie bezpośrednio do MAPIR.

1. Kliknij ikonę błędu u góry karty „Dziennik debugowania”
2. Opisz problem: co robiłeś, czego się spodziewałeś i co się faktycznie stało (wymagane)
3. Opcjonalnie zaznacz **Dodaj kilka obrazów (maksymalnie 10)**i wybierz obrazy z otwartego projektu lub użyj opcji**Przeglądaj...**, aby wybrać dowolny inny plik obrazu
4. Kliknij **Wyślij**

Raport zawiera Twój opis, wszelkie załączone zdjęcia, logi Chloros oraz szczegóły systemowe (wersja Chloros, system operacyjny, karta graficzna i ustawienia projektu). Aby wysłać raport, musisz być zalogowany do Chloros (zakładka „Użytkownik”). Podczas wysyłania możesz zamknąć okno; po zakończeniu do dziennika zostanie zapisany numer referencyjny, np. `Bug report sent. Reference: BR-…` — podaj go, kontaktując się z MAPIR. Jeśli nie da się przesłać raportu (na przykład z powodu braku połączenia z Internetem), Chloros zapisuje go na komputerze i wyświetla ścieżkę dostępu, dzięki czemu można go wysłać pocztą elektroniczną na adres support@mapir.camera.

***

## Monitorowanie zasobów systemowych

### Wykorzystanie procesora

**Tryb swobodny:**

* 1 rdzeń procesora przy ~100%
* Pozostałe rdzenie są w stanie bezczynności lub dostępne
* System pozostaje responsywny

**Tryb równoległy Chloros+:**

* Wiele rdzeni o wysokim stopniu wykorzystania — ich liczba zależy od strategii wybranej w ramach [Dynamicznej adaptacji obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md)
* System może wydawać się mniej responsywny

**Aby monitorować:**

* Menedżer zadań systemu Windows (Ctrl+Shift+Esc)
* Zakładka „Wydajność” → sekcja „Procesor”
* Poszukaj procesów „Chloros” lub „chloros-backend”

### Wykorzystanie pamięci (RAM)

**Typowe wykorzystanie:**

* Małe projekty (< 100 obrazów): 2–4 GB
* Średnie projekty (100–500 obrazów): 4–8 GB
* Duże projekty (ponad 500 obrazów): 8–16 GB
* Tryb równoległy Chloros+ zużywa więcej pamięci RAM

**Jeśli brakuje pamięci:**

* Przetwarzaj mniejsze partie
* Zamknij inne aplikacje
* Zwiększ pojemność pamięci RAM, jeśli regularnie przetwarzasz duże zbiory danych

### Wykorzystanie procesora graficznego (GPU) (Chloros+ z CUDA)

Gdy przyspieszenie GPU jest włączone:

* Procesor graficzny NVIDIA wykazuje wysokie wykorzystanie (60–90%)
* Wzrasta zużycie pamięci VRAM (wymagane co najmniej 4 GB pamięci VRAM; co najmniej 7 GB w przypadku równoczesnego debayeringu z uwzględnieniem tekstur)
* Etap kalibracji przebiega znacznie szybciej

**Aby monitorować:**

* Ikona NVIDIA w zasobniku systemowym
* Menedżer zadań → Wydajność → Karta graficzna
* GPU-Z lub podobne narzędzie do monitorowania

### Operacje wejścia/wyjścia na dysku

**Czego można się spodziewać:**

* Wysokie obciążenie odczytu dysku na etapie analizy
* Wysokie obciążenie zapisu dysku na etapie eksportu
* Dyski SSD działają znacznie szybciej niż dyski HDD

**Wskazówka dotycząca wydajności:**

* Jeśli to możliwe, używaj dysku SSD jako lokalizacji folderu projektu
* Unikaj dysków sieciowych w przypadku dużych zbiorów danych
* Upewnij się, że dysk nie jest prawie zapełniony (wpływa to na prędkość zapisu)

***

## Wykrywanie problemów podczas przetwarzania

### Sygnały ostrzegawcze

**Postęp przetwarzania zatrzymuje się (brak zmian przez ponad 5 minut):**

* Sprawdź dziennik debugowania pod kątem błędów
* Sprawdź dostępną przestrzeń dyskową
* Sprawdź w Menedżerze zadań, czy program Chloros działa

**Często pojawiają się komunikaty o błędach:**

* Zatrzymaj przetwarzanie i przeanalizuj błędy
* Typowe przyczyny: brak miejsca na dysku, uszkodzone pliki, problemy z pamięcią
* Zobacz sekcję „Rozwiązywanie problemów” poniżej

**System przestaje odpowiadać:**

* Tryb równoległy Chloros+ zużywa zbyt wiele zasobów
* Rozważ zmniejszenie liczby równoległych zadań lub modernizację sprzętu
* Tryb swobodny jest mniej wymagający pod względem zasobów

### Kiedy przerwać przetwarzanie

Przerwij przetwarzanie, jeśli zauważysz:

* ❌ Błędy „Dysk pełny” lub „Nie można zapisać pliku”
* ❌ Powtarzające się błędy uszkodzenia plików obrazów
* ❌ System całkowicie zawiesił się (nie odpowiada)
* ❌ Zorientowałeś się, że skonfigurowano nieprawidłowe ustawienia
* ❌ Zaimportowano niewłaściwe obrazy

**Jak zatrzymać:**

1. Kliknij**przycisk „Zatrzymaj”** (zastępuje przycisk „Start”) — wystarczy raz
2. Pasek wyświetla komunikat „Zatrzymywanie...”, dopóki trwający obraz nie zostanie zakończony, po czym proces kończy się w stanie zatrzymania
3. Produkty, które zostały już wyeksportowane, pozostają na dysku; w dzienniku pojawia się dokładny komunikat `[RUN-SUMMARY]` opisujący, co zostało zakończone
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

1. Jeśli proces dopiero się rozpoczął i znajduje się w fazie wykrywania: zatrzymaj, zaznacz obiekty docelowe, uruchom ponownie
2. Na przyszłość: używaj dysku SSD, zmniejsz liczbę indeksów, zmodernizuj sprzęt
3. Rozważ użycie interfejsu CLI do przetwarzania wsadowego dużych zbiorów danych

### Ostrzeżenia dotyczące „miejsca na dysku”

**Rozwiązania:**

1. Natychmiast zwolnij miejsce na dysku
2. Przenieś projekt na dysk z większą ilością miejsca
3. Zmniejsz liczbę indeksów do eksportu
4. Wyłącz niepotrzebne produkty eksportowe LATTICE (Ustawienia projektu → Przetwarzanie)
5. Używaj formatu JPG zamiast TIFF a (mniejsze pliki)

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
3. Zmniejsz obciążenie przetwarzania (użyj trybu „Free” zamiast „Chloros+”)
4. Przetwarzaj w chłodniejszych porach dnia

***

## Powiadomienie o zakończeniu przetwarzania

Po zakończeniu przetwarzania:

* Pasek postępu osiąga 100%
* W dzienniku debugowania pojawiają się wiersze `[RUN-SUMMARY]` wraz z ostatecznymi liczbami
* Przycisk „Start” staje się ponownie aktywny
* Wszystkie pliki wyjściowe znajdują się w drzewie wyjściowym projektu dla poszczególnych kamer: `<project>/<camera>/<format>/<Product>_Images/`

***

## Kolejne kroki

Po zakończeniu przetwarzania:

1. **Sprawdź wyniki** – zobacz [Zakończenie przetwarzania](finishing-the-processing.md)
2. **Sprawdź folder wyjściowy** – upewnij się, że wszystkie pliki zostały poprawnie wyeksportowane
3. **Przejrzyj dziennik debugowania** — sprawdź, czy nie ma żadnych ostrzeżeń lub błędów
4. **Wyświetl podgląd przetworzonych obrazów** — użyj przeglądarki obrazów lub oprogramowania zewnętrznego

Aby uzyskać informacje na temat przeglądania i wykorzystywania przetworzonych wyników, zapoznaj się z sekcją [Zakończenie przetwarzania](finishing-the-processing.md).
