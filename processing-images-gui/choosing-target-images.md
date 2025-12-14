# Wybór obrazów docelowych

Oznaczenie obrazów zawierających cele kalibracyjne jest kluczowym krokiem, który znacznie przyspiesza przetwarzanie w ramach procesu Chloros. Dzięki wstępnemu wybraniu obrazów docelowych eliminuje się konieczność skanowania przez Chloros każdego obrazu w zbiorze danych w celu znalezienia celów kalibracyjnych.

## Dlaczego warto oznaczać obrazy docelowe?

### Szybkość przetwarzania

Bez zaznaczania obrazów docelowych Chloros musi:

* Skanować każdy obraz w projekcie
* Uruchamiać algorytmy wykrywania celów na każdym obrazie
* Niepotrzebnie sprawdzać setki lub tysiące obrazów

**Rezultat**: Przetwarzanie może trwać znacznie dłużej, zwłaszcza w przypadku dużych zbiorów danych.

### Z zaznaczonymi obrazami docelowymi

Po zaznaczeniu kolumny Docelowe dla określonych obrazów:

* Chloros skanuje tylko zaznaczone obrazy w poszukiwaniu celów
* Wykrywanie celów przebiega znacznie szybciej
* Całkowity czas przetwarzania ulega znacznemu skróceniu

{% hint style=&quot;success&quot; %}
**Zwiększenie szybkości**: Zaznaczenie 2-3 obrazów docelowych w zbiorze danych zawierającym 500 obrazów może skrócić czas wykrywania docelowego z ponad 30 minut do mniej niż 1 minuty.
{% endhint %}

***

## Jak zaznaczyć obrazy docelowe

### Krok 1: Zidentyfikuj obrazy docelowe

Przejrzyj zaimportowane obrazy w przeglądarce plików i zidentyfikuj, które obrazy zawierają cele kalibracyjne.

**Typowe scenariusze:**

* **Cel przed przechwyceniem**: przechwycony przed rozpoczęciem sesji
* **Cel po przechwyceniu**: przechwycony po zakończeniu sesji
* **Cele w terenie**: cele umieszczone w obszarze przechwytywania
* **Wiele celów**: 2–3 obrazy docelowe na sesję (zalecane)

### Krok 2: Sprawdź kolumnę docelową

Dla każdego obrazu zawierającego cel kalibracyjny:

1. Znajdź obraz w tabeli przeglądarki plików.
2. Znajdź kolumnę **Docelowa** (kolumna po prawej stronie).
3. Kliknij pole wyboru w kolumnie Docelowa dla tego obrazu.
4. Powtórz tę czynność dla wszystkich obrazów zawierających cele.

### Krok 3: Sprawdź swój wybór

Przed przetworzeniem sprawdź:

* [ ] Wszystkie obrazy z celami kalibracyjnymi są zaznaczone
* [ ] Żadne obrazy niebędące celami nie są przypadkowo zaznaczone
* [ ] Cele są wyraźnie widoczne na zaznaczonych obrazach

***

## Najlepsze praktyki dotyczące obrazów docelowych

### Wytyczne dotyczące przechwytywania celów

**Czas:**

* Przechwytuj obrazy docelowe bezpośrednio przed sesją przechwytywania i w jej trakcie
* W takich samych warunkach oświetleniowych, jak czujnik światła DAQ
* Aby uzyskać najlepsze wyniki, najlepiej rejestruj obrazy celów tak często, jak to możliwe. W przeciwnym razie dane z czujnika światła zostaną wykorzystane do dostosowania kalibracji w miarę upływu czasu.

**Pozycja aparatu:**

* Trzymaj aparat nad celem tak, aby był wyśrodkowany i zajmował około 40–60% środka obrazu.
* Utrzymuj aparat równolegle/nadir do powierzchni celu

**Oświetlenie:**

* Takie samo oświetlenie otoczenia jak czujnik światła DAQ.
* Unikaj cieni na powierzchniach docelowych.
* Nie zasłaniaj źródła światła swoim ciałem, pojazdem lub roślinnością.
* Najbardziej spójne wyniki zapewniają warunki pochmurne.

**Stan docelowy:**

* Utrzymuj panele docelowe w czystości i suchości.
* Wszystkie 4 panele powinny być wyraźnie widoczne i niezasłonięte.
* Jeśli to możliwe, obiekty docelowe powinny być ustawione prostopadle/nadir względem źródła światła.

### Ile obrazów docelowych?

**Minimum:** 1 obraz docelowy na sesję. **Zalecane:** 3–5 obrazów docelowych na sesję.

**Najlepszy harmonogram:**

* 3–5 obrazów uchwyconych wkrótce po rozpoczęciu rejestracji przez czujnik światła
* Aby uzyskać najlepsze wyniki, należy obracać kamerę między kolejnymi ujęciami
* Opcjonalnie: okresowo w trakcie sesji, jeśli warunki oświetleniowe ulegają ciągłym zmianom

***

## Praca z wieloma kamerami

### Konfiguracje z dwoma kamerami

W przypadku jednoczesnego używania dwóch kamer MAPIR (np. Survey3W RGN + Survey3N OCN):

1. Zrób zdjęcia obiektów **obydwoma kamerami** jednocześnie.
2. Użyj **tego samego obiektu** dla obydwu kamer.
3. Oznacz zdjęcia obiektów dla **obydwu typów kamer** w przeglądarce plików.
4. Chloros użyje odpowiednich obiektów do kalibracji każdej kamery.

### Kolumna „Model kamery”

Kolumna **Model kamery** pomaga zidentyfikować, które obrazy pochodzą z której kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* itp.

Użyj tej kolumny, aby sprawdzić, czy oznaczyłeś cele dla każdego typu kamery w swoim projekcie.

***

## Ustawienia wykrywania celów

### Regulacja czułości wykrywania

Jeśli Chloros nie wykrywa prawidłowo celów, dostosuj te ustawienia w [Ustawieniach projektu](adjusting-project-settings.md):

**Minimalny obszar próbki kalibracji:**

* **Domyślnie**: 25 pikseli
* **Zwiększ**, jeśli występują fałszywe wykrycia małych artefaktów
* **Zmniejsz**, jeśli cele nie są wykrywane

**Minimalna klasteryzacja celów:**

* **Domyślnie**: 60
* **Zwiększ**, jeśli cele są dzielone na wiele wykryć
* **Zmniejsz**, jeśli cele o zróżnicowanej kolorystyce nie są w pełni wykrywane

***

## Typowe problemy z obrazami celów

### Problem: brak wykrytych celów

**Możliwe przyczyny:**

* Obrazy celów nie są zaznaczone w przeglądarce plików
* Cel jest zbyt mały w ramce (&lt; 30% obrazu)
* Słabe oświetlenie (cienie, odblaski)
* Zbyt rygorystyczne ustawienia wykrywania celów

**Rozwiązania:**

1. Sprawdź, czy kolumna Cel jest zaznaczona dla prawidłowych obrazów
2. Sprawdź jakość obrazu celu w podglądzie
3. Jeśli jakość jest słaba, ponownie uchwyć cele
4. W razie potrzeby dostosuj ustawienia wykrywania celów

### Problem: Fałszywe wykrycie celów

**Możliwe przyczyny:**

* Białe budynki, pojazdy lub pokrycie terenu mylone z celami
* Jasne plamy w roślinności
* Zbyt niska czułość wykrywania

**Rozwiązania:**

1. Zaznacz tylko rzeczywiste obrazy celów, aby ograniczyć zakres wykrywania
2. Zwiększ minimalny obszar próbki kalibracji
3. Zwiększ minimalną wartość klastrowania celów
4. Upewnij się, że obrazy celów pokazują tylko cel (minimalne zakłócenia tła)

***

## Lista kontrolna weryfikacji

Przed rozpoczęciem przetwarzania sprawdź wybór obrazów celów:

* [ ] Co najmniej 1 obraz celu zaznaczony na sesję
* [ ] Pola wyboru kolumny celów są zaznaczone dla wszystkich obrazów celów
* [ ] Obrazy celów zarejestrowane w tym samym przedziale czasowym co badanie
* [ ] Cele wyraźnie widoczne w podglądzie po kliknięciu
* [ ] Wszystkie 4 panele kalibracyjne widoczne na każdym obrazie celu
* [ ] Brak cieni lub przeszkód na celach
* [ ] W przypadku podwójnego aparatu: cele zaznaczone dla obu typów aparatów

***

## Przetwarzanie bez celów

### Przetwarzanie bez celów kalibracyjnych

Chociaż nie jest to zalecane w przypadku prac naukowych, można przetwarzać bez celów:

1. Pozostaw wszystkie pola wyboru w kolumnie „Cel” niezaznaczone
2. **Wyłącz** „Kalibrację odbicia” w ustawieniach projektu
3. Korekcja winietowania będzie nadal stosowana.
4. Wynik nie będzie skalibrowany pod kątem absolutnego współczynnika odbicia.

{% hint style=&quot;warning&quot; %}
**Nie zalecane**: Bez kalibracji współczynnika odbicia wartości pikseli reprezentują jedynie względną jasność, a nie naukowe pomiary współczynnika odbicia. Aby uzyskać dokładne, powtarzalne wyniki, należy używać celów kalibracyjnych.
{% endhint %}

***

## Kolejne kroki

Po oznaczeniu obrazów docelowych:

1. **Sprawdź ustawienia** — zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md)
2. **Rozpocznij przetwarzanie** — zobacz [Rozpoczęcie przetwarzania](starting-the-processing.md)
3. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)

Więcej informacji na temat samych celów kalibracyjnych można znaleźć w sekcji [Cele kalibracyjne](../calibration-targets.md).
