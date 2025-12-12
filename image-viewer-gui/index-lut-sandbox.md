# Sandbox indeksu/LUT

Sandbox indeksu/LUT to interaktywna przestrzeń robocza w przeglądarce obrazów Chloros, która umożliwia eksperymentowanie z obliczeniami indeksów wielospektralnych i wizualizacjami kolorów w czasie rzeczywistym. To potężne narzędzie pomaga testować różne indeksy, udoskonalać zakresy wartości i tworzyć gotowe do publikacji wizualizacje bez konieczności ponownego przetwarzania całego zestawu danych.

## Czym jest piaskownica indeksów/LUT?

### Cel

Piaskownica zapewnia:

* **Obliczanie indeksów w czasie rzeczywistym** — natychmiastowe stosowanie dowolnego indeksu roślinności
* **Interaktywną regulację LUT** — precyzyjne dostosowywanie gradientów i zakresów kolorów
* **Optymalizację przepływu pracy** — określenie najlepszych ustawień przed przetwarzaniem wsadowym

### Sandbox a przetwarzanie projektu

**Sandbox indeksów/LUT (interaktywny):**

* Pojedyncze obrazy
* Natychmiastowa informacja zwrotna
* Eksperymentalny i iteracyjny
* Brak trwałych zmian w plikach
* Idealny do eksploracji i testowania

**Przetwarzanie projektu (wsadowe):**

* Cały zestaw danych naraz
* Wstępnie skonfigurowane ustawienia
* Trwałe pliki wyjściowe
* Czasochłonne
* Najlepsze, gdy ustawienia są ostateczne

{% hint style=&quot;success&quot; %}
**Najlepszy przepływ pracy**: Użyj piaskownicy, aby eksperymentować i znaleźć optymalne ustawienia indeksu i LUT, a następnie zastosuj te ustawienia podczas przetwarzania projektu dla całego zestawu danych.
{% endhint %}

***

## Praca z piaskownicą indeksu/LUT

### Zrozumienie wstępnie obliczonych indeksów

W Chloros indeksy można stosować podczas przetwarzania projektu. Aby określić, które ustawienia indeksu i LUT chcesz zastosować do eksportów, najłatwiej jest użyć piaskownicy przeglądarki obrazów.

Piaskownica umożliwia:

* **Zastosowanie nowych indeksów i gradientów kolorów (LUT)** w celu wizualizacji danych
* **Interaktywną regulację ustawień wizualizacji**
* **Wyświetlenie** już obliczonych obrazów indeksowych
* **Sprawdzenie** wartości pikseli na wszystkich poziomach powiększenia

### Otwieranie piaskownicy

Dostęp do piaskownicy indeksów/LUT można uzyskać w zakładce **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknij obraz w siatce obrazów przeglądarki plików, aby otworzyć go w zakładce **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Kliknij zakładkę **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby otworzyć lewy wysuwany pasek boczny, jeśli nie jest jeszcze otwarty

### Wybieranie obrazu, do którego ma zostać zastosowany indeks/LUT

Aby pracować z indeksem w przeglądarce obrazów <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Otwórz obraz** z głównej siatki obrazów, klikając na niego
2. Otworzy się zakładka **Przeglądarka obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Kliknij **menu rozwijane warstw** (w prawym górnym rogu przeglądarki).
4. Wybierz warstwę z menu rozwijanego:
   * RAW (odbicie)

### Stosowanie indeksu do obrazu

Gdy obraz zostanie wyświetlony na całym ekranie, a pasek boczny **Przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> jest otwarta:

1. Zaznacz pole Indeks u góry paska bocznego.
2. Wybierz filtr aparatu z menu rozwijanego po lewej stronie.
3. Wybierz żądaną formułę indeksu z menu rozwijanego po prawej stronie.
4. Przeciągnij kolorowe kółka kanałów filtra do odpowiednich miejsc w formule indeksu poniżej.
5. Gdy formuła będzie prawidłowa, obraz zostanie zaktualizowany i wyświetli wartości indeksu.
6. Przesuń kursor myszy, aby zobaczyć wartości w miejscu, w którym znajduje się kursor.
7. Powiększ, aby zobaczyć poszczególne piksele i związane z nimi wartości.

Każdy indeks ma określony zakres wartości i znaczenie:

#### NDVI Przykład

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

Pełna dokumentacja formuł indeksu znajduje się w sekcji [Formuły indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md).

***

## Praca z tabelami LUT (Look-Up Tables)

### Co to jest LUT?

**Tabela Look-Up (LUT)** mapuje numeryczne wartości indeksów na kolory w celu wizualizacji:

* **Wejście**: wartość pikseli indeksu (np. NDVI 0,65)
* **Wyjście**: kolor RGB (np. jasnozielony)
* **Cel**: ułatwienie dostrzegania i interpretacji wzorców

**Tabela LUT w skali szarości a tabela LUT w kolorze:**

* Skala szarości: naukowa i neutralna, pokazuje surowe dane
* Tabela LUT w kolorze: intuicyjna i efektowna, podkreśla wzorce i różnice

{% hint style=&quot;success&quot; %}
**Możliwości wizualizacji**: Zastosowanie kolorowej tabeli LUT do obrazu indeksowego w skali szarości znacznie ułatwia identyfikację wzorców, anomalii i obszarów zainteresowania na pierwszy rzut oka.
{% endhint %}

### Zastosowanie tabeli LUT do obrazu indeksowego

Po uzyskaniu obrazu indeksowego przedstawiającego

1. Kliknij przycisk <img src="../.gitbook/assets/image.png" alt="" data-size="line"> „+Dodaj LUT”
2. Wybierz gradient kolorów
3. Dostosuj minimalne/maksymalne punkty końcowe przycinania
4. Dostosuj tryb przycinania
5. Zaznacz pole Indeks w pasku bocznym **Przeglądarki obrazów** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby zastosować LUT.

### Wybór gradientu kolorów

**Wybór gradientu:**

1. W panelu LUT znajdź **kolorowy pasek gradientu**.
2. Najedź na niego myszką, aby wyświetlić dostępne ustawienia gradientu.
3. Wybierz żądany gradient.
4. Obraz **zostanie natychmiast zaktualizowany** o nowe kolory po zaznaczeniu pola Indeks.

{% hint style=&quot;success&quot; %}
**Najlepsza praktyka**: W przypadku wskaźników wegetacji, takich jak NDVI, gradient Red-Yellow-Green jest najbardziej intuicyjny, ponieważ odpowiada naturalnym skojarzeniom kolorystycznym (zielony = zdrowy, żółty = umiarkowany, czerwony = zestresowany).
{% endhint %}

### Dostosowywanie klas kolorów

**Kontrolka klas** określa, ile oddzielnych stopni kolorów pojawi się w gradiencie:

**Opcje liczby klas:**

* **2–5 klas**: bardzo szerokie kategorie, wyraźne strefy
* **6–10 klas**: zrównoważone, dobre do klasyfikacji
* **11–20 klas**: płynne gradienty, ciągły wygląd
* **Ponad 20 klas**: Prawie ciągłe, maksymalna płynność

**Jak dostosować:**

1. W panelu LUT znajdź **kwadraty próbek kolorów poniżej paska gradientu**
2. Dostosuj liczbę klas, dodając je za pomocą przycisku +
3. Usuń liczbę klas, klikając dwukrotnie próbkę koloru
4. Gradient aktualizuje się **w czasie rzeczywistym** na obrazie

**Wpływ na wizualizację:**

* **Mniejsza liczba klas** (3-5): Tworzy wyraźne strefy, uproszczoną klasyfikację, łatwiejsze do rozróżnienia kategorie
* **Średnia liczba klas** (6-10): Zrównoważone podejście, dobre dla większości zastosowań
* **Większa liczba klas** (15-20): Płynne przejścia, szczegółowe różnice, fotograficzny wygląd

**Kiedy stosować:**

* **Niewiele klas (3-5)**: slajdy prezentacji, mapy klasyfikacyjne, proste raporty
* **Średnia liczba klas (6-10)**: ogólna analiza, zrównoważone szczegóły, standardowe raporty
* **Wiele klas (15-20)**: analiza naukowa, szczegółowa kontrola, wyniki o jakości publikacyjnej

### Precyzyjne dostosowywanie zakresów wartości

**Elementy sterujące zakresem wartości** określają, które wartości indeksu są przypisane do poszczególnych kolorów w gradiencie:

**Elementy sterujące zakresem w panelu LUT:**

* **Wartość minimalna**: dolna granica skali kolorów
* **Wartość maksymalna**: górna granica skali kolorów
* **Wartości pośrednie**: automatycznie rozdzielane między wartością minimalną a maksymalną (na podstawie liczby klas)

#### Dostosowywanie wartości minimalnych/maksymalnych

**Aby dostosować zakresy wartości:**

1. W panelu LUT znajdź pola wprowadzania danych **Wartość minimalna** i **Wartość maksymalna**.
2. Kliknij pole **Wartość minimalna**.
3. Wpisz żądaną wartość minimalną (np. `0.2`).
4. Naciśnij **Enter** lub kliknij poza polem
5. Powtórz tę czynność dla pola **Wartość maksymalna** (np. `0.9`)
6. Wizualizacja **zostanie natychmiast zaktualizowana**

{% hint style=&quot;info&quot; %}
**Automatyczne skalowanie**: Po pierwszym zastosowaniu tablicy LUT, Chloros automatycznie ustawia minimalną/maksymalną wartość na rzeczywisty zakres danych w obrazie. Następnie można zawęzić ten zakres, aby skupić się na konkretnych zakresach wartości, które są przedmiotem zainteresowania.
{% endhint %}

**Przykładowe dostosowania zakresu NDVI:**

* **Pełny zakres**: od `-1.0` do `1.0` (wyświetl wszystkie możliwe wartości)
* **Skupienie się na roślinności**: od `0.2` do `0.9` (wykluczanie gołej gleby i wody)
* **Tylko zdrowa roślinność**: od `0.5` do `0.9` (podkreśl tylko bujne rośliny)
* **Wykrywanie stresu**: od `0.2` do `0.5` (podkreśl obszary problemowe)
* **Zakres niestandardowy**: dostosuj na podstawie obserwowanych wartości pikseli

**Dlaczego warto dostosowywać zakresy?**

* **Zwiększ kontrast** w obszarze zainteresowania
* **Wyklucz nieistotne wartości** (np. zbiorniki wodne, goła gleba)
* **Ujednolicaj wizualizację** dla wielu obrazów lub dat
* **Podkreślaj subtelne różnice** w wąskim zakresie wartości

### Wycinanie wartości poza zakresem

Gdy wartości pikseli wykraczają poza zdefiniowany zakres min/max, można kontrolować sposób ich wyświetlania za pomocą **trybów przycinania**.

#### **Dostępne opcje trybu przycinania:**

#### 1. Minimum i maksimum

* Piksele **poniżej minimum** → wyświetlanie przy użyciu **pierwszego koloru** w gradiencie (np. czerwony)
* Piksele **powyżej maksymalnej wartości** → wyświetlanie przy użyciu **ostatniego koloru** w gradiencie (np. zielonego)
* **Przykład zastosowania**: podkreślanie wartości skrajnych, pokazywanie pełnego zakresu danych przy użyciu nasyconych kolorów na granicach
* **Przykład**: wartości NDVI poniżej 0,2 są wyświetlane na czerwono, a wartości powyżej 0,9 są wyświetlane na zielono

#### 2. Przezroczyste tło

* Piksele **poza zakresem** stają się **całkowicie przezroczyste**
* Tylko piksele **w zakresie** pokazują gradient kolorów
* **Przykład zastosowania**: nakładka GIS, izolowanie określonych zakresów wartości, podkreślanie tylko obszarów zainteresowania
* **Przykład**: Pokaż tylko NDVI 0,4-0,7 w kolorze, wszystko inne przezroczyste

{% hint style=&quot;warning&quot; %}
**Ograniczenie przezroczystości**: Przezroczyste piksele będą wyświetlane jako kolor tła w przeglądarce. Podczas eksportowania w trakcie przetwarzania przezroczystość jest zachowana w formacie PNG, ale nie w formacie JPG.
{% endhint %}

#### 3. Tło indeksu

* Piksele **poza zakresem** wyświetlane są w **skali szarości** (pokazując surowe wartości indeksu)
* Piksele **w zakresie** pokazują **gradient kolorów**
* **Przykład zastosowania**: subtelne podświetlenie, zachowanie kontekstu przy jednoczesnym podkreśleniu obszarów zainteresowania
* **Przykład**: podświetlenie kolorami stresowanej roślinności (NDVI 0,3-0,5) przy jednoczesnym pokazaniu zdrowych obszarów w kolorze szarym

#### 4. Oryginalne tło

* Piksele **poza zakresem** wyświetlają **oryginalny obraz wielospektralny**
* Piksele **w zakresie** pokazują **gradient kolorów**
* **Przykład zastosowania**: Najbardziej intuicyjny — łączy naturalny kontekst obrazu z analityczną nakładką kolorów
* **Przykład**: Zobacz rzeczywisty wygląd pola/upraw z nałożonymi obszarami stresu oznaczonymi kolorami

### Wybór odpowiedniego trybu przycinania

| Tryb przycinania              | Najlepszy dla                                   | Styl wizualizacji          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum i maksimum**    | Pełne wyświetlanie danych, analiza naukowa     | Wszystkie piksele pokolorowane           |
| **Przezroczyste tło** | Nakładki GIS, izolowanie określonych zakresów    | Kolor w zakresie, poza nim pusto |
| **Tło indeksowe**       | Subtelne podkreślenie, zachowanie kontekstu danych  | Kolor w zakresie, poza nim szarość  |
| **Oryginalne tło**    | Raporty, prezentacje, intuicyjna analiza | Kolor w zakresie, poza nim zdjęcie |

### Tworzenie niestandardowych kolorów LUT

Aby uzyskać pełną kontrolę nad wizualizacją, można tworzyć **niestandardowe gradienty kolorów**, edytując poszczególne punkty zatrzymania koloru.

**Aby utworzyć niestandardowy gradient:**

1. W panelu LUT zlokalizuj **pasek podglądu gradientu**
2. Poszukaj **kwadratów próbek kolorów** poniżej gradientu
3. **Kliknij punkt koloru**, aby go wybrać
4. Otworzy się **próbnik kolorów**
5. Wybierz nowy kolor za pomocą:
   * **Koła kolorów**: wizualny wybór koloru
   * **Suwaków RGB/HSV**: precyzyjna kontrola koloru
   * **Wprowadzenia kodu szesnastkowego**: dokładna specyfikacja koloru (np. `#FF0000` dla koloru czerwonego)
6. Kliknij poza selektorem kolorów, **aby zastosować nowy kolor**.
7. Gradient **zostanie natychmiast zaktualizowany** na obrazie.

**Dodawanie lub usuwanie punktów koloru:**

* **Dodaj punkt**: kliknij ikonę +, aby dodać nową próbkę na końcu.
* **Usuń punkt**: kliknij dwukrotnie kwadrat koloru, aby usunąć próbkę.

**Strategie dostosowywania:**

* **Odwróć gradient**: Odwróć kolejność kolorów, aby odwrócić znaczenie (np. zielony = niski, czerwony = wysoki)
* **Kolory marki**: Dopasuj paletę kolorów swojej organizacji do raportów
* **Przyjazne dla osób z daltonizmem**: Używaj kombinacji pomarańczowo-niebieskiej lub fioletowo-żółtej
* **Optymalizacja wydruku**: wybierz kolory, które sprawdzają się zarówno w druku kolorowym, jak i w skali szarości.
* **Wiele progów**: użyj różnych kolorów dla określonych progów wartości w celu klasyfikacji.

{% hint style=&quot;info&quot; %}
**Zapisywanie niestandardowych gradientów**: Niestandardowe gradienty można zapisać i ponownie wykorzystać. Kliknij ikonę zapisu w panelu LUT, aby zachować niestandardowe schematy kolorów do wykorzystania w przyszłości.
{% endhint %}

***

## Interaktywny przepływ pracy

### Aktualizacje w czasie rzeczywistym

Wszystkie zmiany LUT w piaskownicy aktualizują obraz **natychmiastowo i interaktywnie**:

* **Przełącz warstwę** → Obraz zmienia się natychmiast
* **Wybierz gradient** → Kolory aktualizują się natychmiast
* **Dostosuj zakres wartości** → Kontrast zmienia się w czasie rzeczywistym
* **Zmień klasy** → Płynność gradientu aktualizuje się natychmiast
* **Zmodyfikuj przycinanie** → Wyświetlanie tła zmienia się natychmiast
* **Edytuj kolory** → Niestandardowy gradient zostanie zastosowany natychmiast

**Nie jest potrzebny przycisk „Zastosuj”** — wszystkie zmiany są wprowadzane na żywo i interaktywnie!

{% hint style=&quot;success&quot; %}
**Informacje zwrotne na żywo**: Natychmiastowe informacje zwrotne pozwalają szybko eksperymentować z różnymi ustawieniami, aż do znalezienia optymalnej wizualizacji dla potrzeb analizy.
{% endhint %}

### Iteracyjny proces udoskonalania

**Typowy proces optymalizacji LUT:**

1. **Wybierz warstwę indeksową** (np. RAW (odbicie))
2. **Zastosuj indeks** — wybierz filtr aparatu i formułę indeksu, przeciągnij kolorowe kółka do odpowiedniego miejsca w formule indeksu
3. **Zastosuj gradient LUT** — zacznij od ustawienia wstępnego Red-Yellow-Green
4. **Sprawdź wartości pikseli** — przesuń kursor, zwróć uwagę na zakresy wartości
5. **Dostosuj min/max** — zawęź, aby skupić się na roślinności (np. 0,2 do 0,9)
6. **Wybierz przycinanie** — wypróbuj „Oryginalne tło” dla kontekstu
7. **Dopracuj kolory** — w razie potrzeby dostosuj gradient, aby uzyskać określony efekt
8. **Sfinalizuj ustawienia** — zapisz ustawienia i skopiuj je do ustawień projektu w celu przetworzenia eksportu

### Sprawdzanie wartości pikseli

Zrozumienie rzeczywistych wartości pikseli ma kluczowe znaczenie dla ustawienia skutecznych zakresów LUT:

**Jak sprawdzić wartości:**

1. Wartości pikseli są wyświetlane, gdy obraz ma zaznaczone pole Indeks lub zarówno pole Indeks, jak i LUT.
2. **Przesuń kursor** nad różne obszary obrazu.
3. **Obserwuj wartości pikseli** wyświetlane w legendzie po najechaniu kursorem.
4. Powiększ, aby zobaczyć poszczególne piksele podświetlone wartością zmienną.
5. **Zrób notatki** dotyczące zakresów wartości dla różnych cech:
   * **Zdrowa roślinność**: np. NDVI 0,55-0,85
   * **Roślinność zestresowana**: np. NDVI 0,30-0,50
   * **Nagie gleby**: np. NDVI 0,05–0,25
   * **Woda** (jeśli występuje): np. NDVI -0,05 do 0,10

**Używanie wartości pikseli do ustawiania zakresów LUT:**

Po sprawdzeniu wartości pikseli dostosuj odpowiednio minimalną/maksymalną wartość LUT:

**Przykładowy scenariusz:**

* **Obserwacja**: wartości gleby = 0,05–0,25, stres = 0,25–0,50, zdrowie = 0,50–0,85
* **Cel**: wizualizacja wyłącznie stanu zdrowia roślin (z wyłączeniem gleby)
* **Ustawienia LUT**: Min = `0.25`, Max = `0.85`
* **Przycinanie**: „Oryginalne tło”, aby zobaczyć glebę w naturalnym kolorze
* **Wynik**: Gradient kolorów dotyczy tylko roślinności, gleba jest wyświetlana jako oryginalny obraz

{% hint style=&quot;info&quot; %}
**Zakres dynamiczny**: różne uprawy, pory roku i etapy wzrostu będą miały różne zakresy wartości. Przed ustawieniem zakresów LUT należy zawsze sprawdzić wartości pikseli w konkretnym zestawie danych.
{% endhint %}

***

## Indeksy niestandardowe (Chloros+)

### Tworzenie niestandardowych formuł indeksów

{% hint style=&quot;info&quot; %}
**Gdzie tworzyć**: Niestandardowe indeksy można skonfigurować w **Ustawieniach projektu** przed przetwarzaniem, a także w pasku bocznym piaskownicy przeglądarki obrazów.
{% endhint %}

**Aby utworzyć indeks niestandardowy:**

1. **Otwórz ustawienia projektu** (przed przetwarzaniem) lub pasek boczny piaskownicy przeglądarki obrazów
2. Przejdź do **menu rozwijanego formuły indeksu**
3. Znajdź opcję **„Niestandardowe”** (konieczne jest zalogowanie się z licencją Chloros+)
4. **Zdefiniuj formułę**, używając zmiennych pasma:
   * Nazwy pasm: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` itp.
   * Operatory: `+`, `-`, `*`, `/`, `^` (wykładnik)
   * Funkcje: `sqrt()`, `abs()` itp. (jeśli są obsługiwane)
   * Nawiasy: `()` dla kolejności operacji
5. **Nazwij swój indeks** (np. „MyIndex” lub „CustomNDVI”)
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

{% hint style=&quot;warning&quot; %}
**Walidacja formuły**: Upewnij się, że formuła wykorzystuje pasma dostępne w Twojej kamerze. Na przykład RedEdge jest dostępne tylko w kamerach z filtrem RedEdge.
{% endhint %}

***

## Kolejne kroki

Teraz, gdy rozumiesz już działanie piaskownicy indeksów/tablic LUT:

* **Zastosuj do przetwarzania**: użyj odkrytych ustawień w [Ustawieniach projektu](../project-settings/project-settings.md)
* **Przetwarzanie wsadowe**: zastosuj zoptymalizowane indeksy do pełnych zestawów danych
* **Dowiedz się więcej**: przeczytaj [Wzory indeksów wielospektralnych](../project-settings/multispectral-index-formulas.md)

Powiązana dokumentacja:

* [**Warstwy obrazu**](image-layers.md) — zarządzanie warstwami i wizualizacja
* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — podstawy przeglądarki obrazów
* [**Przetwarzanie obrazów (GUI)**](../processing-images-gui/adding-files-to-a-project.md) — pełny przebieg przetwarzania
