# Wybór obrazów docelowych

Oznaczenie obrazów zawierających cele kalibracyjne jest kluczowym krokiem, który znacznie przyspiesza działanie potoku przetwarzania Chloros. Dzięki wstępnemu wyboru obrazów docelowych eliminujesz konieczność przeszukiwania przez Chloros każdego obrazu w zbiorze danych w poszukiwaniu celów kalibracyjnych.

## Dlaczego warto oznaczyć obrazy docelowe?

### Szybkość przetwarzania

Bez zaznaczenia obrazów docelowych program Chloros musi:

* Przeskanować każdy obraz w projekcie
* Uruchomić algorytmy wykrywania celów na każdym obrazie
* Niepotrzebnie sprawdzać setki lub tysiące obrazów

**Wynik**: Przetwarzanie może trwać znacznie dłużej, zwłaszcza w przypadku dużych zbiorów danych.

### Z zaznaczonymi obrazami docelowymi

Po zaznaczeniu kolumny „Target” dla określonych obrazów:

* Chloros skanuje tylko zaznaczone obrazy w poszukiwaniu obiektów docelowych
* Wykrywanie obiektów docelowych przebiega znacznie szybciej
* Całkowity czas przetwarzania ulega znacznemu skróceniu

{% hint style="success" %}
**Wzrost szybkości**: Zaznaczenie 2–3 obrazów docelowych w zbiorze danych zawierającym 500 obrazów może skrócić czas wykrywania obiektów docelowych z ponad 30 minut do mniej niż 1 minuty.
{% endhint %}

***

## Jak oznaczyć obrazy celów

### Krok 1: Zidentyfikuj swoje obrazy celów

Przejrzyj zaimportowane obrazy w przeglądarce plików i zidentyfikuj, które z nich zawierają cele kalibracyjne.

**Typowe scenariusze:*** **Cel przed przechwyceniem**: Przechwycony przed rozpoczęciem sesji
* **Cel po przechwyceniu**: Przechwycony po zakończeniu sesji
* **Cele w terenie**: Cele umieszczone w obszarze przechwytywania
* **Wiele celów**: 2–3 obrazy celów na sesję (zalecane)

### Krok 2: Sprawdź kolumnę „Target”

Dla każdego obrazu zawierającego cel kalibracyjny:

1. Znajdź obraz w tabeli przeglądarki plików
2. Znajdź kolumnę **Target** (kolumna po prawej stronie)
3. Zaznacz pole wyboru w kolumnie „Target” dla tego obrazu
4. Powtórz tę czynność dla wszystkich obrazów zawierających cele

### Krok 3: Sprawdź swój wybór

Przed przetworzeniem sprawdź dokładnie:

* [ ] Wszystkie obrazy z celami kalibracyjnymi są zaznaczone
* [ ] Żadne obrazy niebędące celami nie zostały przypadkowo zaznaczone
* [ ] Cele są wyraźnie widoczne na zaznaczonych obrazach

***

## Najlepsze praktyki dotyczące obrazów celów

### Wytyczne dotyczące rejestrowania celów

**Czas:**

* Rejestruj obrazy celów bezpośrednio przed sesją rejestracji i w jej trakcie
* W takich samych warunkach oświetleniowych, jak czujnik światła DAQ
* Najlepiej rejestruj obrazy celów tak często, jak to możliwe, aby uzyskać najlepsze wyniki. W przeciwnym razie dane z czujnika światła zostaną wykorzystane do dostosowania kalibracji w miarę upływu czasu.

**Pozycja aparatu:**

* Trzymaj aparat nad celem tak, aby był wyśrodkowany i zajmował około 40–60% środkowej części obrazu.
* Utrzymuj aparat równolegle do powierzchni celu lub w pozycji nadir

**Oświetlenie:**

* Takie samo oświetlenie otoczenia jak w przypadku czujnika światła DAQ
* Unikaj cieni na powierzchniach celów
* Nie zasłaniaj źródła światła swoim ciałem, pojazdem ani roślinnością
* Najbardziej spójne wyniki zapewniają warunki pochmurne

**Stan celu:**

* Utrzymuj panele celów w czystości i suchości
* Wszystkie 4 panele powinny być wyraźnie widoczne i niczym niezasłonięte
* Cele powinny być ustawione prostopadle/nadir do źródła światła, jeśli to możliwe

### Ile obrazów celów?

**Minimum:**1 zdjęcie celu na sesję.**Zalecane:** 3–5 zdjęć celu na sesję.**Harmonogram zgodny z najlepszymi praktykami:**

* 3–5 zdjęć wykonanych wkrótce po rozpoczęciu rejestracji przez czujnik światła
* Aby uzyskać najlepsze wyniki, należy obracać kamerę między kolejnymi ujęciami
* Opcjonalnie: okresowo w trakcie sesji, jeśli warunki oświetleniowe ulegają ciągłym zmianom

***

## Praca z wieloma kamerami

### Konfiguracje z dwoma kamerami

W przypadku jednoczesnego używania dwóch kamer MAPIR (np. Survey3W RGN + Survey3N OCN):

1. Zrób zdjęcia obiektu **obiema kamerami** w tym samym czasie
2. Użyj **tego samego obiektu** dla obu kamer
3. Zaznacz zdjęcia obiektu dla **obu typów kamer** w przeglądarce plików
4. Chloros użyje odpowiednich obiektów do kalibracji każdej z kamer

### Kolumna „Model kamery”

Kolumna **„Model kamery”** pomaga zidentyfikować, które obrazy pochodzą z której kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* itd.

Użyj tej kolumny, aby sprawdzić, czy w projekcie zaznaczyłeś cele dla każdego typu kamery.

***

## Ustawienia wykrywania celów

### Regulacja czułości wykrywania

Jeśli Chloros nie wykrywa celów poprawnie, dostosuj te ustawienia w [Ustawieniach projektu](adjusting-project-settings.md):**Minimalny obszar próbki kalibracji:*** **Domyślnie**: 25 pikseli
* **Zwiększ**, jeśli występują fałszywe wykrycia na małych artefaktach
* **Zmniejsz**, jeśli cele nie są wykrywane**Minimalne grupowanie celów:*** **Domyślnie**: 60
* **Zwiększ**, jeśli cele są dzielone na wiele wykryć
* **Zmniejsz**, jeśli cele o zróżnicowanej kolorystyce nie są w pełni wykrywane***

## Typowe problemy z obrazami celów

### Problem: Brak wykrytych celów

**Możliwe przyczyny:**

* Obrazy celów nie są zaznaczone w przeglądarce plików
* Cel jest zbyt mały w kadrze (&lt; 30% obrazu)
* Słabe oświetlenie (cienie, odblaski)
* Zbyt rygorystyczne ustawienia wykrywania celów

**Rozwiązania:**

1. Sprawdź, czy kolumna „Cel” jest zaznaczona dla właściwych obrazów
2. Sprawdź jakość obrazu celu w podglądzie
3. Ponownie uchwyć cele, jeśli jakość jest słaba
4. W razie potrzeby dostosuj ustawienia wykrywania celów

### Problem: Fałszywe wykrycia celów

**Możliwe przyczyny:**

* Białe budynki, pojazdy lub pokrycie terenu mylone z celami
* Jasne plamy w roślinności
* Zbyt niska czułość wykrywania

**Rozwiązania:**

1. Zaznacz tylko rzeczywiste obrazy celów, aby ograniczyć zakres wykrywania
2. Zwiększ minimalny obszar próbki kalibracji
3. Zwiększ minimalną wartość skupiania celów
4. Upewnij się, że obrazy celów pokazują tylko cel (minimalne zakłócenia tła)

***

## Lista kontrolna weryfikacji

Przed rozpoczęciem przetwarzania sprawdź wybór obrazów celów:

* [ ] Co najmniej 1 obraz celu zaznaczony na sesję
* [ ] Pola wyboru w kolumnie „Cel” są zaznaczone dla wszystkich obrazów celów
* [ ] Obrazy celów zarejestrowane w tym samym przedziale czasowym co badanie
* [ ] Cele wyraźnie widoczne w podglądzie po kliknięciu
* [ ] Wszystkie 4 panele kalibracyjne widoczne na każdym obrazie celu
* [ ] Brak cieni lub przeszkód na celach
* [ ] W przypadku dwóch kamer: cele zaznaczone dla obu typów kamer

***

## Przetwarzanie bez celów

### Przetwarzanie bez celów kalibracyjnych

Chociaż nie jest to zalecane do prac naukowych, można przetwarzać dane bez celów:

1. Pozostaw wszystkie pola wyboru w kolumnie „Cel” niezaznaczone
2. **Wyłącz** opcję „Kalibracja odbicia” w ustawieniach projektu
3. Korekcja winietowania nadal będzie stosowana
4. Wynik nie będzie skalibrowany pod kątem odbicia bezwzględnego

{% hint style="warning" %}
**Nie zalecane**: Bez kalibracji odbicia wartości pikseli reprezentują jedynie względną jasność, a nie naukowe pomiary odbicia. Aby uzyskać dokładne i powtarzalne wyniki, należy używać celów kalibracyjnych.
{% endhint %}

***

## Kolejne kroki

Po zaznaczeniu obrazów docelowych:

1. **Sprawdź ustawienia** — zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md)
2. **Rozpocznij przetwarzanie** — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
3. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)

Więcej informacji na temat samych wzorców kalibracyjnych znajdziesz w sekcji [Wzorce kalibracyjne](../calibration-targets.md).
