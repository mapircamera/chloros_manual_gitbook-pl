# Środowisko testowe indeksów/tablic LUT

Środowisko testowe indeksów/tablic LUT to interaktywna przestrzeń robocza w przeglądarce obrazów Chloros, która umożliwia eksperymentowanie z obliczeniami indeksów wielospektralnych i wizualizacjami kolorów w czasie rzeczywistym. To potężne narzędzie pomaga testować różne indeksy, dopracowywać zakresy wartości oraz tworzyć wizualizacje gotowe do publikacji bez konieczności ponownego przetwarzania całego zbioru danych.

## Czym jest Index/LUT Sandbox?

### Cel

Sandbox zapewnia:

* **Obliczanie indeksów w czasie rzeczywistym** – natychmiastowe zastosowanie dowolnego indeksu wegetacyjnego
* **Interaktywną regulację LUT** – precyzyjne dostosowanie gradientów i zakresów kolorów
* **Optymalizację przepływu pracy** – określenie najlepszych ustawień przed przetwarzaniem wsadowym

### Sandbox a przetwarzanie projektowe

**Sandbox indeksów/LUT (interaktywny):**

* Pojedynczy obraz na raz
* Natychmiastowa informacja zwrotna
* Eksperymentalny i iteracyjny
* Brak trwałych zmian w plikach
* Idealny do eksploracji i testowania

**Przetwarzanie projektowe (partia):**

* Cały zbiór danych naraz
* Wstępnie skonfigurowane ustawienia
* Trwałe pliki wyjściowe
* Czasochłonne
* Najlepsze rozwiązanie, gdy ustawienia są już ostateczne

{% hint style="success" %}
**Najlepszy przebieg pracy**: Użyj środowiska testowego do eksperymentowania i znalezienia optymalnych ustawień indeksu i LUT, a następnie zastosuj te ustawienia podczas przetwarzania projektu dla całego zbioru danych.
{% endhint %}

***

## Praca z piaskownicą indeksów/LUT

### Zrozumienie wstępnie obliczonych indeksów

W Chloros indeksy można zastosować podczas przetwarzania projektu. Aby określić, które ustawienia indeksu i LUT chcesz zastosować do eksportów, najłatwiej jest użyć piaskownicy przeglądarki obrazów.

Piaskownica pozwala na:

* **Zastosować nowe indeksy i gradienty kolorów (LUT)** w celu wizualizacji danych
* **Dostosować ustawienia wizualizacji** w trybie interaktywnym
* **Wyświetlić** już obliczone obrazy indeksowe
* **Sprawdzić** wartości pikseli na wszystkich poziomach powiększenia

### Otwieranie środowiska testowego

Dostęp do środowiska testowego indeksów/LUT uzyskuje się w zakładce **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknij obraz w siatce obrazów przeglądarki plików, otworzy się on w zakładce **Przeglądarka obrazów**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Kliknij kartę**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby otworzyć wysuwany pasek boczny po lewej stronie, jeśli nie jest jeszcze otwarty

### Wybieranie obrazu, do którego chcesz zastosować indeks/LUT

Aby pracować z indeksem w <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Otwórz obraz** z głównej siatki obrazów, klikając na niego
2. Otworzy się wówczas zakładka **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> otworzy się
3. Kliknij **menu rozwijane warstw** (w prawym górnym rogu przeglądarki)
4. Wybierz warstwę z menu rozwijanego:
   * RAW (Odbicie)

### Stosowanie indeksu do obrazu

Gdy obraz jest wyświetlany na pełnym ekranie, a pasek boczny **Przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Zaznacz pole Indeks u góry paska bocznego
2. Wybierz filtr aparatu z menu rozwijanego po lewej stronie
3. Wybierz żądaną formułę indeksu z menu rozwijanego po prawej stronie
4. Przeciągnij kolorowe kółka kanałów filtra do odpowiednich miejsc w formule indeksu poniżej
5. Gdy formuła będzie poprawna, obraz zaktualizuje się i wyświetli wartości indeksu
6. Przesuń kursor myszy, aby zobaczyć wartości w miejscu, w którym znajduje się kursor
7. Powiększ obraz, aby zobaczyć poszczególne piksele i związane z nimi wartości

Każdy indeks ma określony zakres wartości i znaczenie:

#### Przykład NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Pełna dokumentacja formuł indeksów znajduje się w sekcji [Formuły indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md).

***

## Praca z tabelami LUT (Look-Up Tables)

### Czym jest tabela LUT?

**Tabela LUT (Look-Up Table)** mapuje numeryczne wartości indeksów na kolory w celu wizualizacji:

* **Dane wejściowe**: Wartość pikselowa indeksu (np. NDVI 0,65)
* **Wynik**: Kolor RGB (np. jasnozielony)
* **Cel**: Ułatwienie dostrzegania i interpretacji wzorów**Tabela LUT w skali szarości a tabela LUT kolorów:**

* Skala szarości: Naukowa i neutralna, pokazuje surowe dane
* Tabela LUT kolorów: Intuicyjna i wyrazista, podkreśla wzory i różnice

{% hint style="success" %}
**Możliwości wizualizacji**: Zastosowanie kolorowej tablicy LUT do obrazu indeksowego w skali szarości znacznie ułatwia identyfikację wzorów, anomalii i obszarów zainteresowania na pierwszy rzut oka.
{% endhint %}

### Stosowanie tablicy LUT do obrazu indeksowego

Gdy masz już obraz indeksowy przedstawiający

1. Kliknij przycisk <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> przycisk „+Dodaj LUT”
2. Wybierz gradient kolorów
3. Dostosuj minimalne i maksymalne punkty końcowe przycinania
4. Dostosuj tryb przycinania
5. Zaznacz pole Indeks w pasku bocznym **Przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby zastosować tabelę LUT

### Wybór gradientu kolorów

**Wybór gradientu:**

1. W panelu LUT znajdź**kolorowy pasek gradientu**

2. Najedź na niego myszką, aby wyświetlić dostępne ustawienia gradientu
3. Wybierz żądany gradient
4. Obraz **natychmiast się zaktualizuje** z nowymi kolorami, gdy pole Indeks zostanie zaznaczone

{% hint style="success" %}
**Najlepsza praktyka**: W przypadku wskaźników wegetacji, takich jak NDVI, gradient Red-żółty-Green jest najbardziej intuicyjny, ponieważ odpowiada naturalnym skojarzeniom kolorystycznym (zielony = zdrowy, żółty = umiarkowany, czerwony = zestresowany).
{% endhint %}

### Dostosowywanie klas kolorów

**Element sterujący „Klasy”**określa, ile odrębnych stopni koloru pojawi się w Twoim gradiencie:**Opcje liczby klas:*** **2–5 klas**: Bardzo szerokie kategorie, wyraźne strefy
* **6–10 klas**: Zrównoważone, dobre do klasyfikacji
* **11–20 klas**: Płynne gradienty, ciągły wygląd
* **20+ klas**: Prawie ciągłe, maksymalna płynność**Jak dostosować:**

1. W panelu LUT znajdź**kwadraty próbek kolorów poniżej paska gradientu**

2. Dostosuj liczbę klas, dodając je za pomocą przycisku +
3. Usuń liczbę klas, klikając dwukrotnie na próbkę koloru
4. Gradient aktualizuje się **w czasie rzeczywistym** na obrazie**Wpływ na wizualizację:*** **Mniej klas** (3–5): Tworzy wyraźne strefy, uproszczoną klasyfikację, łatwiejsze rozróżnienie kategorii
* **Średnia liczba klas** (6–10): Zrównoważone podejście, dobre dla większości zastosowań
* **Więcej klas** (15–20): Płynne przejścia, szczegółowe zróżnicowanie, wygląd fotograficzny**Kiedy stosować:*** **Niewiele klas (3–5)**: slajdy prezentacji, mapy klasyfikacyjne, proste raporty
* **Średnia liczba klas (6–10)**: ogólna analiza, zrównoważony poziom szczegółowości, standardowe raporty
* **Wiele klas (15–20)**: analiza naukowa, szczegółowa kontrola, wyniki o jakości publikacyjnej

### Precyzyjne dostosowywanie zakresów wartości

**Elementy sterujące zakresem wartości**określają, które wartości indeksu są przypisane do poszczególnych kolorów w gradiencie:**Elementy sterujące zakresem w panelu LUT:*** **Wartość minimalna**: Dolna granica skali kolorów
* **Wartość maksymalna**: Górna granica skali kolorów
* **Wartości pośrednie**: Automatycznie rozłożone między wartością minimalną a maksymalną (na podstawie liczby klas)

#### Dostosowywanie wartości minimalnych/maksymalnych

**Aby dostosować zakresy wartości:**

1. W panelu LUT znajdź pola wprowadzania**Wartość minimalna**i**Wartość maksymalna**

2. Kliknij pole**Wartość minimalna**

3. Wpisz żądaną wartość minimalną (np. `0.2`)
4. Naciśnij klawisz **Enter** lub kliknij poza polem
5. Powtórz tę czynność dla pola **Wartość maksymalna** (np. `0.9`)
6. Wizualizacja **zaktualizuje się natychmiast**{% hint style="info" %}**Automatyczne skalowanie**: Po pierwszym zastosowaniu tabeli LUT, Chloros automatycznie ustawia wartości minimalne i maksymalne zgodnie z rzeczywistym zakresem danych na obrazie. Następnie można zawęzić ten zakres, aby skupić się na konkretnych zakresach wartości, które nas interesują.
{% endhint %}

**Przykładowe dostosowania zakresu NDVI:*** **Pełny zakres**: od `-1.0` do `1.0` (pokaż wszystkie możliwe wartości)
* **Skupienie na roślinności**: od `0.2` do `0.9` (z wyłączeniem gołej gleby i wody)
* **Tylko zdrowa roślinność**: od `0.5` do `0.9` (podkreśl tylko bujne rośliny)
* **Wykrywanie stresu**: od `0.2` do `0.5` (podkreśl obszary problemowe)
* **Zakres niestandardowy**: dostosuj na podstawie obserwowanych wartości pikseli**Dlaczego warto dostosowywać zakresy?*** **Zwiększ kontrast** w obszarze, który Cię interesuje
* **Wyklucz nieistotne wartości** (np. zbiorniki wodne, goła gleba)
* **Ujednolic wizualizację** na wielu obrazach lub w różnych datach
* **Podkreśl subtelne różnice** w wąskim zakresie wartości

### Wycinanie wartości poza zakresem

Gdy wartości pikseli wykraczają poza zdefiniowany zakres min/max, możesz kontrolować sposób ich wyświetlania za pomocą **trybów przycinania**.

#### **Dostępne opcje trybów przycinania:**

#### 1. Minimum i maksimum

* Piksele **poniżej minimum**→ wyświetlane przy użyciu**pierwszego koloru** w gradiencie (np. czerwonego)
* Piksele **powyżej maksimum**→ wyświetlane przy użyciu**ostatniego koloru** w gradiencie (np. zielonego)
* **Przykład zastosowania**: Podkreślenie skrajnych wartości, pokazanie pełnego zakresu danych z nasyconymi kolorami na granicach
* **Przykład**: Wartości NDVI poniżej 0,2 są wyświetlane na czerwono, a wartości powyżej 0,9 na zielono

#### 2. Przezroczyste tło

* Piksele **poza zakresem**stają się**całkowicie przezroczyste*** Tylko piksele **w zakresie** pokazują gradient kolorów
* **Przykład zastosowania**: nakładka GIS, izolowanie określonych zakresów wartości, podkreślanie tylko obszarów zainteresowania
* **Przykład**: Pokaż tylko NDVI 0,4–0,7 w kolorze, wszystko inne przezroczyste

{% hint style="warning" %}
**Ograniczenie przezroczystości**: Przezroczyste piksele będą wyświetlane w przeglądarce jako kolor tła. Podczas eksportu w trakcie przetwarzania przezroczystość jest zachowana w formacie PNG, ale nie w formacie JPG.
{% endhint %}

#### 3. Tło indeksowe

* Piksele **poza zakresem**wyświetlane są w**skali szarości** (pokazując surowe wartości indeksu)
* Piksele **w zakresie**pokazują**gradient kolorów*** **Przykład zastosowania**: Subtelne podświetlenie, zachowanie kontekstu przy jednoczesnym podkreśleniu obszarów zainteresowania
* **Przykład**: Podświetlenie kolorami roślinności poddanej stresowi (NDVI 0,3–0,5), przy jednoczesnym pokazaniu zdrowych obszarów w kolorze szarym

#### 4. Tło oryginalne

* Piksele **poza zakresem**wyświetlają**oryginalny obraz wielospektralny*** Piksele **w zakresie**pokazują**gradient kolorów*** **Przykład zastosowania**: Najbardziej intuicyjny — łączy naturalny kontekst obrazu z analityczną nakładką kolorową
* **Przykład**: Zobacz rzeczywisty wygląd pola/uprawy z nałożonymi obszarami stresu oznaczonymi kolorami

### Wybór odpowiedniego trybu przycinania

| Tryb przycinania              | Najlepsze zastosowanie                                   | Styl wizualizacji          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum i maksimum**    | Pełne wyświetlanie danych, analiza naukowa     | Wszystkie piksele kolorowe           |
| **Przezroczyste tło** | Nakładki GIS, izolowanie określonych zakresów    | Kolor w zakresie, poza nim pusto |
| **Tło indeksowe**       | Subtelne podkreślenie, zachowanie kontekstu danych  | Kolor w zakresie, szarość poza nim  |
| **Tło oryginalne**    | Raporty, prezentacje, intuicyjna analiza | Kolor w zakresie, zdjęcie poza nim |

### Tworzenie niestandardowych kolorów LUT

Aby uzyskać pełną kontrolę nad wizualizacją, możesz tworzyć **niestandardowe gradienty kolorów**, edytując poszczególne punkty kolorów.**Aby utworzyć niestandardowy gradient:**

1. W panelu LUT znajdź**pasek podglądu gradientu**

2. Poszukaj**kwadratów próbek kolorów** poniżej gradientu
3. **Kliknij punkt koloru**, aby go zaznaczyć
4. Otworzy się **próbnik kolorów**

5. Wybierz nowy kolor za pomocą:
   * **Koła kolorów**: Wizualny wybór koloru
   * **Suwaki RGB/HSV**: Precyzyjna kontrola koloru
   * **Wprowadzenie kodu szesnastkowego**: Dokładna specyfikacja koloru (np. `#FF0000` dla czerwonego)
6. Kliknij poza okienkiem wyboru kolorów, **aby zastosować nowy kolor**

7. Gradient**natychmiast się zaktualizuje** na obrazie**Dodawanie lub usuwanie punktów kolorystycznych:*** **Dodaj punkt**: Kliknij ikonę +, aby dodać nową próbkę na końcu
* **Usuń punkt**: Kliknij dwukrotnie kwadrat koloru, aby usunąć próbkę**Strategie dostosowywania:*** **Odwróć gradient**: Odwróć kolejność kolorów, aby zmienić znaczenie (np. zielony = niski, czerwony = wysoki)
* **Kolory marki**: Dopasuj paletę kolorów do palety swojej organizacji w raportach
* **Przyjazne dla osób z daltonizmem**: Używaj kombinacji pomarańczowo-niebieskich lub fioletowo-żółtych
* **Optymalizacja druku**: Wybierz kolory, które sprawdzają się zarówno w druku kolorowym, jak i w skali szarości
* **Wiele progów**: Użyj różnych kolorów przy określonych progach wartości do klasyfikacji

{% hint style="info" %}
**Zapisywanie niestandardowych gradientów**: Niestandardowe gradienty można zapisać i ponownie wykorzystać. Kliknij ikonę zapisu w panelu LUT, aby zachować niestandardowe schematy kolorów do wykorzystania w przyszłości.
{% endhint %}

***

## Interaktywny przebieg pracy

### Aktualizacje w czasie rzeczywistym

Wszystkie zmiany LUT w piaskownicy aktualizują obraz **natychmiastowo i interaktywnie**:

* **Zmiana warstwy** → Obraz zmienia się natychmiast
* **Wybór gradientu** → Kolory aktualizują się natychmiast
* **Dostosowanie zakresu wartości** → Kontrast zmienia się w czasie rzeczywistym
* **Zmiana klas** → Gładkość gradientu aktualizuje się natychmiast
* **Modyfikacja przycinania** → Wyświetlanie tła zmienia się natychmiast
* **Edycja kolorów** → Niestandardowy gradient stosuje się natychmiast**Nie jest potrzebny przycisk „Zastosuj”** — wszystkie zmiany są widoczne na żywo i interaktywne!

{% hint style="success" %}
**Informacja zwrotna na żywo**: Natychmiastowa informacja zwrotna pozwala szybko eksperymentować z różnymi ustawieniami, aż do znalezienia optymalnej wizualizacji dostosowanej do potrzeb analizy.
{% endhint %}

### Iteracyjny proces udoskonalania

**Typowy proces optymalizacji LUT:**

1.**Wybierz warstwę indeksową** (np. RAW (Odbicie))
2. **Zastosuj indeks** – wybierz filtr kamery i wzór indeksu, przeciągnij kolorowe kółka w odpowiednie miejsce we wzorze indeksu
3. **Zastosuj gradient LUT** – zacznij od ustawienia wstępnego Red-Yellow-Green
4. **Sprawdź wartości pikseli** – przesuwaj kursor, zwracaj uwagę na zakresy wartości
5. **Dostosuj min/maks** – zawęź zakres, aby skupić się na roślinności (np. od 0,2 do 0,9)
6. **Wybierz przycinanie** – wypróbuj opcję „Original Background” dla zachowania kontekstu
7. **Dopracuj kolory** – dostosuj gradient, jeśli to konieczne, aby uzyskać konkretny efekt
8. **Sfinalizuj ustawienia**– zapisz ustawienia i skopiuj je do ustawień projektu w celu przetworzenia eksportu

### Sprawdzanie wartości pikseli

Zrozumienie rzeczywistych wartości pikseli ma kluczowe znaczenie dla ustawienia skutecznych zakresów LUT:**Jak sprawdzać wartości:**

1. Wartości pikseli są widoczne, gdy na obrazku jest**zaznaczone** pole „Indeks” lub zarówno „Indeks”, jak i „LUT”.
2. **Przesuń kursor** nad różne obszary obrazu
3. **Obserwuj wartości pikseli** wyświetlane w legendzie po najechaniu kursorem
4. Powiększ obraz, aby zobaczyć poszczególne piksele podświetlone z wyświetlaną wartością
5. **Zanotuj** zakresy wartości dla różnych elementów:
   * **Zdrowa roślinność**: np. NDVI 0,55–0,85
   * **Roślinność w stanie stresu**: np. NDVI 0,30–0,50
   * **Odsłonięta gleba**: np. NDVI 0,05–0,25
   * **Woda** (jeśli występuje): np. NDVI od -0,05 do 0,10**Wykorzystanie wartości pikseli do ustawienia zakresów LUT:**Po sprawdzeniu wartości pikseli dostosuj odpowiednio minimalne i maksymalne wartości LUT:**Przykładowy scenariusz:*** **Obserwacja**: Wartości gleby = 0,05–0,25, Stres = 0,25–0,50, Zdrowie = 0,50–0,85
* **Cel**: Wizualizacja wyłącznie stanu zdrowia roślin (z wyłączeniem gleby)
* **Ustawienia LUT**: Min = `0.25`, Max = `0.85`
* **Ograniczanie**: „Oryginalne tło”, aby zobaczyć glebę w naturalnym kolorze
* **Wynik**: Gradient kolorów dotyczy wyłącznie roślinności, gleba jest wyświetlana jako na oryginalnym zdjęciu

{% hint style="info" %}
**Zakres dynamiczny**: Różne uprawy, pory roku i etapy wzrostu będą miały różne zakresy wartości. Zawsze sprawdzaj wartości pikseli w konkretnym zbiorze danych przed ustawieniem zakresów LUT.
{% endhint %}

***

## Indeksy niestandardowe (Chloros+)

### Tworzenie formuł indeksów niestandardowych

{% hint style="info" %}
**Gdzie tworzyć**: Indeksy niestandardowe można skonfigurować w**Ustawieniach projektu** przed przetwarzaniem, a także na pasku bocznym środowiska testowego przeglądarki obrazów.
{% endhint %}

**Aby utworzyć indeks niestandardowy:**

1.**Otwórz Ustawienia projektu** (przed przetwarzaniem) lub pasek boczny środowiska testowego przeglądarki obrazów
2. Przejdź do **menu rozwijanego Formuła indeksu**

3. Znajdź opcję**„Niestandardowy”** (wymagane jest zalogowanie się przy użyciu licencji Chloros+)
4. **Zdefiniuj formułę**, używając zmiennych pasm:
   * Nazwy pasm: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` itp.
   * Operatory: `+`, `-`, `*`, `/`, `^` (wykładnik)
   * Funkcje: `sqrt()`, `abs()` itp. (jeśli są obsługiwane)
   * Nawiasy: `()` dla kolejności operacji
5. **Nadaj nazwę indeksowi** (np. „MyIndex” lub „CustomNDVI”)
6. **Zapisz konfigurację**

**Przykładowe formuły niestandardowe:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Walidacja formuły**: Upewnij się, że formuła wykorzystuje pasma dostępne w Twojej kamerze. Na przykład RedEdge jest dostępna tylko w kamerach z filtrem RedEdge.
{% endhint %}

***

## Kolejne kroki

Teraz, gdy rozumiesz już działanie środowiska testowego Index/LUT:

* **Zastosuj do przetwarzania**: Użyj odkrytych ustawień w [Ustawieniach projektu](../project-settings/project-settings.md)
* **Przetwarzanie wsadowe**: Zastosuj zoptymalizowane indeksy do pełnych zestawów danych
* **Dowiedz się więcej**: Przeczytaj [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md)

Powiązana dokumentacja:

* [**Warstwy obrazu**](image-layers.md) – Zarządzanie warstwami i wizualizacja
* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — podstawy przeglądarki obrazów
* [**Przetwarzanie obrazów (GUI)**](../processing-images-gui/adding-files-to-a-project.md) — pełny przebieg przetwarzania
