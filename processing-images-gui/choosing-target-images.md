# Wybór obrazów z celami

Zaznaczenie obrazów zawierających cele kalibracyjne pozwala programowi Chloros dokładnie określić, gdzie ma ich szukać. Gdy w kolumnie „Target” zaznaczony jest co najmniej jeden obraz, program Chloros skanuje **wyłącznie zaznaczone obrazy** — zatem zaznaczanie celów pozwala zarówno przyspieszyć przetwarzanie, jak i zapobiec pomyłkowemu rozpoznaniu zdjęć pomiarowych jako celów.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Dlaczego warto zaznaczać obrazy docelowe?

### Zaznaczenie kontroluje proces skanowania

Gdy zaznaczysz kolumnę „Docelowe” dla określonych obrazów:

* Chloros skanuje tylko zaznaczone obrazy w poszukiwaniu celów
* Wykrywanie celów przebiega znacznie szybciej
* Obrazy pomiarowe nie powodują fałszywych wykryć celów

Jeśli **żadne** zdjęcia nie są zaznaczone, Chloros przechodzi w tryb skanowania wszystkich zdjęć w projekcie:

* Algorytmy wykrywania celów działają na każdym zdjęciu
* Setki lub tysiące zdjęć są sprawdzane niepotrzebnie
* Przetwarzanie trwa znacznie dłużej, zwłaszcza w przypadku dużych zbiorów danych

{% hint style="success" %}
**Zwiększenie szybkości**: Oznaczenie 2–3 obrazów zawierających cele w zbiorze danych liczącym 500 obrazów może skrócić czas wykrywania celów z ponad 30 minut do mniej niż 1 minuty.
{% endhint %}

***

## Jak oznaczyć obrazy celów

### Krok 1: Zidentyfikuj obrazy celów

Przejrzyj zaimportowane obrazy w przeglądarce plików i zidentyfikuj, które z nich zawierają cele kalibracyjne.

**Typowe scenariusze:*** **Cel przed wykonaniem zdjęcia**: zarejestrowany przed rozpoczęciem sesji
* **Cel po zakończeniu sesji**: zarejestrowany po zakończeniu sesji
* **Cele w terenie**: cele umieszczone w obszarze rejestracji
* **Wiele celów**: 2–3 obrazy celów na sesję (zalecane)

### Krok 2: Zaznacz kolumnę „Target” (<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">)

Dla każdego zdjęcia zawierającego cel kalibracyjny:

1. Znajdź zdjęcie w tabeli przeglądarki plików
2. Znajdź kolumnę **Target** (kolumna po prawej stronie)
3. Zaznacz pole wyboru w kolumnie „Target” dla tego zdjęcia
4. Powtórz tę czynność dla wszystkich zdjęć zawierających cele

### Krok 3: Sprawdź swój wybór

Przed przetwarzaniem sprawdź jeszcze raz:

* [ ] Wszystkie zdjęcia z celami kalibracyjnymi są zaznaczone
* [ ] Żadne zdjęcia niebędące celami nie zostały przypadkowo zaznaczone
* [ ] Cele są wyraźnie widoczne na zaznaczonych zdjęciach

***

## LATTICE: Cele są opcjonalne podczas rejestracji przez moduł DAQ

W przypadku kamer wielospektralnych LATTICE cel kalibracyjny w kadrze jest **jednym z dwóch** możliwych punktów odniesienia dla współczynnika odbicia:

* **Cel w kadrze**: gdy obraz z zaznaczonym celem przejdzie kontrolę jakości (QA) w programie „Chloros”, cel staje się**absolutnym punktem odniesienia dla współczynnika odbicia** dla otaczających go obrazów.
* **Promieniowanie wpadające do DAQ**: gdy nie ma żadnego celu (lub nie spełnia on wymagań kontroli jakości), moduł Chloros oblicza współczynnik odbicia na podstawie natężenia promieniowania wpadającego do czujnika światła DAQ (ρ = π·L/E). Jeśli Twoje zdjęcia obejmują nagranie z `.daq` lub DAQ-M `.csv`, otrzymujesz skalibrowaną odbiciowość**bez żadnych obrazów celów**.

To automatyczne zachowanie jest ustawieniem domyślnym. W pliku CLI / SDK odpowiada to opcji `--reflectance-source auto`; można również wymusić opcję `target` (ścisła — bez zastępowania przez DAQ) lub `daq` (nadrzędność DAQ). Zobacz [CLI Dokumentacja](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrie celów LATTICE**: oprócz klasycznego wykrywania paneli stosowanego w trybie Survey3, przetwarzanie LATTICE obsługuje**cele oznaczone ArUco**,**celów ze stałym obszarem zainteresowania (ROI)**oraz**celów paskowych**, konfigurowanych indywidualnie dla każdego projektu. Skany**zmierzonej** odbicia celu dla poszczególnych jednostek można dostarczyć według numeru seryjnego (CLI: `--target-reflectance-dir`, po jednym `<serial>.csv` na każdą jednostkę docelową), przy czym jako rozwiązanie awaryjne stosuje się nominalne widma T3/T4P.

{% hint style="info" %}
**Moduł F988**: Odbicie modułu F988 jest kalibrowane przy użyciu panelu odbicia umieszczonego w scenie: pasmo to wykracza poza skalibrowany zakres czujnika światła DAQ, dlatego Chloros stosuje najnowszy zapis z panelu i zachowuje go między kolejnymi pomiarami panelu. Jeśli moduł F988 jest przetwarzany wyłącznie na podstawie danych z DAQ, program „Chloros” odrzuca współczynnik odbicia oparty na danych z DAQ dla tego pasma (przyczyna pominięcia `dls-uncalibrated-band-988`) — obsługiwanym sposobem postępowania jest wykorzystanie panelu.
{% endhint %}

***

## Najlepsze praktyki dotyczące obrazów docelowych

### Wytyczne dotyczące rejestrowania obrazów docelowych

**Czas:**

* Rejestruj obrazy docelowe bezpośrednio przed sesją rejestracji oraz w jej trakcie
* W takich samych warunkach oświetleniowych, jak czujnik światła DAQ
* Najlepiej rejestrować obrazy obiektów tak często, jak to możliwe, aby uzyskać najlepsze wyniki. W przeciwnym razie dane z czujnika światła zostaną wykorzystane do dostosowania kalibracji w miarę upływu czasu.

**Pozycja aparatu:**

* Trzymaj aparat nad obiektem tak, aby był wyśrodkowany i zajmował około 40–60% środkowej części obrazu.
* Utrzymuj aparat równolegle do powierzchni obiektu lub w pozycji nadir

**Oświetlenie:**

* Takie samo oświetlenie otoczenia jak w przypadku czujnika światła DAQ
* Unikaj cieni na powierzchniach obiektu
* Nie zasłaniaj źródła światła własnym ciałem, pojazdem ani roślinnością
* Najbardziej spójne wyniki zapewniają warunki zachmurzenia

**Stan obiektu:**

* Panele celu powinny być czyste i suche
* Wszystkie panele celu (np. wszystkie 4 w przypadku T4) powinny być wyraźnie widoczne i nic nie powinno zasłaniać ich widoku
* Jeśli to możliwe, cel powinien znajdować się prostopadle lub nad źródłem światła

### Ile zdjęć celu?

**Minimum:**1 zdjęcie celu na sesję.**Zalecane:** 3–5 zdjęć celu na sesję.**Zalecany harmonogram:**

* 3–5 zdjęć wykonanych wkrótce po rozpoczęciu rejestracji przez czujnik światła
* Aby uzyskać najlepsze wyniki, zmieniaj położenie kamery między kolejnymi ujęciami
* Opcjonalnie: okresowo w trakcie sesji, jeśli warunki oświetleniowe ulegają ciągłym zmianom

***

## Praca z wieloma kamerami

### Konfiguracje z dwoma kamerami

W przypadku jednoczesnego używania dwóch kamer typu „MAPIR” (np. Survey3W RGN + Survey3N OCN):

1. Zrób zdjęcia celów **obiema kamerami** jednocześnie
2. Użyj **tego samego fizycznego obiektu** dla obu kamer
3. Zaznacz zdjęcia obiektu dla **obu typów kamer** w przeglądarce plików
4. Program „Chloros” wykorzysta odpowiednie obiekty do kalibracji każdej z kamer

### Kolumna „Model kamery”

Kolumna **„Model kamery”** pomaga zidentyfikować, które zdjęcia pochodzą z której kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* itp.

Użyj tej kolumny, aby sprawdzić, czy zaznaczyłeś cele dla każdego typu kamery w swoim projekcie.

***

## Ustawienia wykrywania obiektów

### Regulacja czułości wykrywania

Jeśli funkcja „Chloros” nie wykrywa obiektów poprawnie, dostosuj następujące ustawienia w [Ustawieniach projektu](adjusting-project-settings.md):**Minimalny obszar próbki kalibracyjnej (px):*** **Domyślnie**: 25 pikseli
* **Zwiększ**, jeśli występują fałszywe wykrycia na małych artefaktach
* **Zmniejsz**, jeśli cele nie są wykrywane**Minimalne skupienie celów (0–100):*** **Domyślnie**: 60
* **Zwiększ**, jeśli cele są rozdzielane na wiele wykrytych obiektów
* **Zmniejsz**, jeśli cele o zróżnicowanej kolorystyce nie są w pełni wykrywane

{% hint style="info" %}
**Wskazówka dotycząca CLI**: `chloros-cli process` akceptuje te same parametry (`--min-target-size`, `--target-clustering`), a jego flaga `--target`/`--targets` oznacza cały folder wejściowy jako przeznaczony wyłącznie dla panelu celów. Zobacz [Podręcznik CLI](../reference/cli-reference.md).
{% endhint %}

***

## Typowe problemy z obrazami docelowymi

### Problem: Nie wykryto żadnych celów

**Możliwe przyczyny:**

* Obrazy celów nie są zaznaczone w przeglądarce plików
* Cel jest zbyt mały w kadrze (&lt; 30% obrazu)
* Słabe oświetlenie (cienie, odblaski)
* Zbyt rygorystyczne ustawienia wykrywania celów

**Rozwiązania:**

1. Sprawdź, czy kolumna „Cel” jest zaznaczona przy odpowiednich obrazach
2. Sprawdź jakość obrazu celu w podglądzie
3. Jeśli jakość jest niska, ponownie zarejestruj cele
4. W razie potrzeby dostosuj ustawienia wykrywania celów

### Problem: Fałszywe wykrycia celów

**Możliwe przyczyny:**

* Białe budynki, pojazdy lub pokrycie terenu mylone z celami
* Jasne plamy w roślinności
* Zbyt niska czułość wykrywania

**Rozwiązania:**

1. Zaznaczaj tylko rzeczywiste obrazy celów — skanowane są wyłącznie zaznaczone obrazy
2. Zwiększ minimalny obszar próbki kalibracyjnej
3. Zwiększ minimalną wartość skupiania celów
4. Upewnij się, że obrazy celów przedstawiają wyłącznie cel (minimalne zakłócenia tła)

***

## Lista kontrolna weryfikacji

Przed rozpoczęciem przetwarzania sprawdź wybór obrazów celów:

* [ ] Co najmniej 1 obraz celu zaznaczony na sesję (lub, w przypadku LATTICE, nagranie `.daq`/`.csv` obejmujące sesję)
* [ ] Pola wyboru w kolumnie „Cel” są zaznaczone dla wszystkich obrazów celów
* [ ] Obrazy celów zostały zarejestrowane w tym samym przedziale czasowym co badanie
* [ ] Cele są wyraźnie widoczne w podglądzie po kliknięciu
* [ ] Wszystkie panele kalibracyjne są widoczne na każdym obrazie celu
* [ ] Brak cieni lub przeszkód na celach
* [ ] W przypadku dwóch kamer: cele zaznaczone dla obu typów kamer

***

## Przetwarzanie bez celów

### LATTICE: z zapisem DAQ

Jeśli czujnik światła DAQ zarejestrował natężenie promieniowania padającego podczas sesji LATTICE, cele nie są potrzebne:

1. Zaimportuj plik `.daq` (lub DAQ-M `.csv`) zawierający obrazy
2. Pozostaw kolumnę „Cel” niezaznaczoną
3. Odbicie światła jest obliczane automatycznie na podstawie referencji promieniowania padającego z czujnika DAQ
4. Promieniowanie nigdy nie wymaga celu ani danych DAQ — pochodzi wyłącznie z fabrycznej kalibracji radiometrycznej kamery

### Przetwarzanie bez żadnego odniesienia

Można również przeprowadzić przetwarzanie bez celów i bez danych DAQ:

1. Pozostaw wszystkie pola wyboru w kolumnie „Cel” niezaznaczone
2. **Wyłącz** opcję „Kalibracja odbicia / balans bieli” w ustawieniach projektu — wykrywanie celów zostanie wówczas całkowicie pominięte
3. Korekcja winietowania nadal będzie stosowana
4. Wynik nie zostanie skalibrowany pod kątem odbicia bezwzględnego (program LATTICE multispectral nadal eksportuje produkty po usunięciu efektu bayera, podgląd oraz produkty radiancji)

{% hint style="warning" %}
**Niezalecane do prac naukowych w systemie „Survey3”**: Bez kalibracji odbicia wartości pikseli w formacie „Survey3” reprezentują jedynie względną jasność, a nie naukowe pomiary odbicia. Aby uzyskać dokładne i powtarzalne wyniki, należy używać celów kalibracyjnych (lub, w przypadku LATTICE, czujnika światła DAQ).
{% endhint %}

***

## Kolejne kroki

Po zaznaczeniu obrazów docelowych:

1. **Sprawdź ustawienia** – zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md)
2. **Rozpocznij przetwarzanie** — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)
3. **Monitoruj postęp** — zobacz [Monitorowanie przetwarzania](monitoring-the-processing.md)

Więcej informacji na temat samych tarcz kalibracyjnych można znaleźć w sekcji [Tarcze kalibracyjne](../calibration-targets.md).
