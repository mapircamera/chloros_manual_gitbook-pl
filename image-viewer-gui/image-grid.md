# Siatka obrazów

Po zaimportowaniu obrazów do projektu zobaczysz je ułożone w siatce w głównym obszarze. W tej siatce wybierasz, **którą wersję każdego obrazu chcesz wyświetlić** — przyciski nad nią pozwalają jednocześnie przełączać wszystkie miniatury między plikami źródłowymi a poszczególnymi przetworzonymi produktami.

## Rozmiar miniatur

Użyj suwaka powiększenia w prawym górnym rogu, aby dostosować rozmiar miniatur obrazów. Zakres suwaka wynosi od **64 px do 1200 px**.

* **Ctrl + kółko myszy** również pozwala skalować miniatury.
* **Ctrl + `+`**/**Ctrl + `=`**oraz**Ctrl + `−`** zwiększają rozmiar o 4 px przy każdym naciśnięciu. Zakres skali klawiaturowej kończy się na 64 px po stronie mniejszej, a po stronie większej – na rozmiarze, który dokładnie mieści dwie miniatury w każdym wierszu w bieżącym oknie.
* Wybrany rozmiar jest zapisywany wraz z projektem (`UI → Grid thumbnail size` w `project.json`, domyślnie `160`), więc ponowne otwarcie projektu powoduje jego przywrócenie.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Rozdzielczość* miniatur to ustawienie niezależne od *rozmiaru* miniatur: zobacz **Wyświetlanie → Rozdzielczość miniatur obrazów** w [Ustawieniach projektu](../project-settings/project-settings.md) (domyślnie 512 pikseli na dłuższym boku). Rozmiar określa, jak duża jest rysowana kafelka; rozdzielczość określa, ile szczegółów jest pobieranych w celu jej wypełnienia.***

## Pasek narzędzi siatki

Rząd przycisków nad siatką składa się z maksymalnie trzech grup, od lewej do prawej:

1. **Na wyzwalacz / Na kamerę** — tryb grupowania. Pojawia się tylko w projektach zawierających nagrania z LATTICE.
2. **Przyciski filtrowania kamer** — po jednym na każdą kamerę LATTICE. Pojawiają się tylko w trybie „Na kamerę”.
3. **Przyciski trybu eksportu/podglądu** — określają, który produkt wyświetla każda miniatura.

Gdy okno jest zbyt wąskie, by pomieścić wszystkie przyciski, grupy zwijają się od prawej do lewej, tworząc rozwijane menu po najechaniu kursorem: najpierw zwijają się przyciski eksportu/podglądu, a następnie przyciski kamer. Zwinięta grupa pozostawia jeden przycisk oznaczony aktualnie aktywną opcją, a najechanie kursorem na niego powoduje wysunięcie pełnego zestawu w dół. **Tryby „Na wyzwalacz” / „Na kamerę” nigdy nie są zwijane.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Przyciski eksportu i podglądu

Przyciski te przełączają miniatury w siatce między typami obrazów. **Przycisk pojawia się, gdy tylko istnieje produkt, którego nazwa jest na nim podana** — co w przypadku plików źródłowych oznacza natychmiast po zaimportowaniu, a nie po przetworzeniu. Chloros ponownie skanuje produkty projektu w trakcie trwania przebiegu, więc przyciski pojawiają się podczas przetwarzania, gdy każdy produkt zaczyna być zapisywany na dysku.

### Przycisk podstawowy

Najbardziej po lewej stronie przycisk eksportu jest opatrzony etykietą **odzwierciedlającą to, co faktycznie zaimportowano**:

| Co zaimportowano | Nazwa przycisku |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Ujęcia LATTICE z podglądem wyświetlania obok klatki RAW | `PNG` lub `TIFF`, w zależności od tego, które podglądy są wyświetlane |
| Zrzuty LATTICE, w których plikiem bazowym **jest** klatka w formacie RAW | *brak przycisku* — `RAW (Original)` już pokazuje ten plik |

W projekcie mieszanym etykieta odpowiada rozszerzeniu, którego używa większość obrazów.

### Przyciski produktów

| Przycisk | Wyświetla | Kiedy się pojawia |
| --- | --- | --- |
| **Cele** | Obrazy z wykrytym celem kalibracyjnym | Po przebiegu, w którym wykryto cele |
| **Odbicie** | Skalibrowane obrazy odbicia | Tylko w projektach Survey3 — w projektach LATTICE używa się zamiast tego `RAW (Reflectance)`, więc siatka nigdy nie wyświetla dwóch przycisków odbicia |
| **Zrównoważona biel** | Obraz z wyrównaną bielą (kamery RGB) | Po przetworzeniu |
| **Skorygowane winietowanie** | Nieskalibrowany obraz rezerwowy ze skorygowanym winietowaniem | Po przebiegu, w którym nie można było zastosować kalibracji odbicia, a *korekcja winietowania* była włączona |
| **Reakcja czujnika** | Nieskalibrowany wariant awaryjny oparty na reakcji czujnika | To samo, ale z wyłączoną *korekcją winietowania* |
| **`RAW (<INDEX> Index)`** | Jeden przycisk na każdy obliczony wskaźnik | Po przebiegu z skonfigurowanymi wskaźnikami |
| **`<INDEX> LUT`** | Jeden przycisk na indeks z mapą kolorów | Po przebiegu z skonfigurowaną tabelą LUT |
| **`<Index> <Index\|LUT> <NNN>`** | Jeden przycisk na każdy przebieg eksportu [Index/LUT Sandbox](index-lut-sandbox.md) | W momencie zakończenia eksportu do piaskownicy |

### Przyciski poziomów LATTICE

Projekty zawierające ujęcia LATTICE zawierają te przyciski, oznaczone nazwą poziomu zamiast nazwą produktu:

| Przycisk | Poziom |
| --- | --- |
| **RAW (Oryginalny)** | Źródłowa klatka w formacie RAW, w postaci zaimportowanej |
| **RAW (Radiance)** | Promieniowanie spektralne typu Float32, W/m²/sr/nm |
| **RAW (Reflectance)** | Współczynnik odbicia typu uint16, 32768 = ρ 1,0 |

Poziom `RAW (Original)` jest dostępny od momentu importu — nie wymaga żadnego przetwarzania. Gdy import z programu LATTICE nie ma w ogóle przycisku bazowego (plikiem bazowym każdego ujęcia jest jego surowa klatka), siatka przesuwa się sama do pierwszego dostępnego przycisku poziomu, tak aby podświetlenie paska narzędzi odpowiadało temu, co widzisz.

Eksporty Chloros z dwoma poziomami **nie mają własnego przycisku siatki**:

* **Debayered** — widok `RAW (Original)` jest już renderowany po usunięciu efektu bayera, więc drugi przycisk na wizualnie identycznym obrazie byłby zbędny. Produkt `RAW (Debayered)` jest nadal zapisywany na dysku i nadal można go wybrać z rozwijanego menu warstw pełnoekranowych.
* **Podgląd** — w kamerach typu RGB podgląd jest zarejestrowany jako warstwa `White Balanced`, która posiada przycisk. W kamerach wielospektralnych jest on zarejestrowany jako `RAW (Preview)` i jest dostępny z rozwijanego menu warstw pełnoekranowych.

{% hint style="info" %}
Te przyciski poziomów są renderowane tylko w projektach, które faktycznie zawierają klatki LATTICE. Projekty Survey3 rejestrują niektóre z tych samych wewnętrznych nazw warstw, a przyciski są dla nich odfiltrowane, dzięki czemu siatka Survey3 zachowuje swój znany zestaw `JPG / Targets / Reflectance`.
{% endhint %}

Kliknięcie miniatury siatki otwiera pełnoekranową [Przeglądarkę obrazów](opening-an-image-full-screen.md) dla **tego samego produktu, który przedstawia siatka** — jeśli siatka jest ustawiona na `Targets`, miniatura otwiera wyeksportowany obraz docelowy.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Grupowanie projektu LATTICE: według wyzwalacza vs według kamery

Zrzuty matrycowe generują kilka obrazów tej samej chwili z różnych modułów kamer. Grupowanie określa, w jaki sposób siatka je układa. W obu trybach wyświetlane są paski nagłówkowe o pełnej szerokości, które można zwinąć; **każda grupa jest początkowo rozwinięta**, a Chloros zapamiętuje te, które użytkownik zamknął. Stan zwinięcia jest śledzony oddzielnie dla każdego trybu, więc zamknięcie grupy w trybie „Na kamerę” nie powoduje zamknięcia niczego w trybie „Na wyzwalacz”.

### Na kamerę (domyślnie)

Jedna grupa na moduł kamery. Nagłówek pokazuje model kamery i numer seryjny (`LATT-M3M — <serial>`) oraz liczbę zdjęć. Kafelki w grupie są uporządkowane chronologicznie według momentu wykonania zdjęcia.

W tym trybie pasek narzędzi zawiera również jeden **przycisk filtrowania kamer na każdą kamerę**, oznaczony jako `MODEL (SERIAL)`. Wszystkie kamery są domyślnie zaznaczone; kliknięcie przycisku powoduje odznaczenie tej kamery i usunięcie jej grupy z siatki. Jest to szybki sposób na przeglądanie jednego pasma z całego lotu.

### Na wyzwalacz

Jedna grupa na zdarzenie przechwytywania — zbiór klatek zarejestrowanych przez wszystkie moduły przy tym samym wyzwalaczu. Nagłówek pokazuje czas przechwycenia, liczbę kamer, które wzięły udział w rejestracji, oraz ikonę dla każdego modelu kamery w grupie. Kafelki w obrębie grupy są uporządkowane według numeru seryjnego kamery, dzięki czemu to samo pasmo znajduje się w tej samej kolumnie dla każdego wyzwolenia.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Obrazy niepochodzące z systemu LATTICE w projekcie mieszanym nie są grupowane — są wyświetlane jako zwykłe kafelki po grupach.

***

## Miniatury w siatce są dostosowane do rozmiaru bloku GSD

Jeśli w pasku bocznym zakładki obrazu ustawiono rozmiar bloku **GSD (px)**, miniatury w siatce są wyświetlane w tej samej rozdzielczości naziemnej — nie tylko w widoku pełnoekranowym. Rozmiar bloku równy 8 oznacza, że każdy wyświetlany piksel jest średnią z bloku 8 × 8 pikseli źródłowych, w każdym miejscu aplikacji, w którym wyświetlany jest obraz.

Ponieważ szerokość kafelka wynosi zaledwie kilkaset pikseli, grube rozmiary bloków przestają mieć widoczną różnicę w siatce znacznie wcześniej niż w widoku pełnoekranowym: ramka o szerokości 4000 px narysowana na kafelku o szerokości 160 px ma już około 25 pikseli źródłowych na każdy wyświetlany piksel. Zobacz [Otwieranie obrazu na całym ekranie](opening-an-image-full-screen.md#gsd-block-size), aby zapoznać się z samym elementem sterującym.

***

## Strony powiązane

* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — przeglądarka pełnoekranowa, wartości kursora i histogram
* [**Warstwy obrazu**](image-layers.md) — menu rozwijane warstw w przeglądarce pełnoekranowej
* [**Piaskownica indeksów/LUT**](index-lut-sandbox.md) — tworzenie i eksportowanie wizualizacji indeksów
* [**Ustawienia projektu**](../project-settings/project-settings.md) — przełączniki eksportu, które decydują o tym, które produkty w ogóle istnieją
