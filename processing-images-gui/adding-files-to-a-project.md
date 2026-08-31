# Dodawanie plików do projektu

Po utworzeniu lub otwarciu projektu w programie Chloros kolejnym krokiem jest dodanie obrazów wielospektralnych w celu rozpoczęcia przetwarzania. Zakładka „Przeglądarka plików” (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) ułatwia importowanie obrazów i zarządzanie zbiorem danych.

## Dostęp do przeglądarki plików

1. Otwórz lub utwórz projekt w programie Chloros
2. Kliknij ikonę **Przeglądarki plików** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. W panelu przeglądarki plików wyświetli się lista plików projektu

{% hint style="info" %}
**Obsługiwane typy plików**:

* **Survey3W / Survey3N**: pary plików RAW+JPG oraz obrazy JPG (zalecane: RAW+JPG)
* **LATTICE**: Nagrania `.tif` / `.tiff` — zapisane przez sterownik kamery Chloros lub przez hub LATTICE
* **Dane z czujników światła**: nagrania `.daq` (DAQ-U/M/E) oraz rejestry promieniowania padającego DAQ-M `.csv` — importowane wraz z obrazami w celu przeprowadzenia kalibracji odbicia
{% endhint %}

***

## Dodawanie obrazów do projektu

Istnieją dwa podstawowe sposoby dodawania obrazów do projektu:

### Metoda 1: Dodaj pliki

Użyj tej opcji, aby zaimportować pojedyncze pliki obrazów lub niewielki zestaw plików.

1. Kliknij przycisk **„Dodaj pliki”** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> u góry panelu przeglądarki plików
2. Przejdź do folderu zawierającego obrazy
3. Wybierz jeden lub więcej plików obrazów (przytrzymaj klawisz **Ctrl**, aby zaznaczyć wiele plików)
4. Kliknij **„Otwórz”**, aby zaimportować wybrane pliki

### Metoda 2: Dodaj folder

Użyj tej opcji, aby zaimportować wszystkie obrazy z folderu jednocześnie. W jednym oknie dialogowym możesz wybrać **wiele folderów**.

1. Kliknij przycisk **„Dodaj folder”** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> u góry panelu Przeglądarki plików
2. Przejdź do folderów zawierających obrazy z sesji przechwytywania i wybierz je
3. Kliknij **„Wybierz folder”**, aby zaimportować wszystkie obsługiwane obrazy

{% hint style="info" %}
**Pojawiają się komunikaty o plikach, których nie udało się załadować.** Jeśli folder zawiera pliki rozpoznawane przez Chloros, ale nie można ich załadować, pojawi się ostrzeżenie — obrazy nie znikają z siatki bez powiadomienia.
{% endhint %}

***

## Importowanie folderów z danymi z LATTICE

Dane z LATTICE są zapisywane w **jednym podfolderze na każdy poziom eksportu** — na przykład `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — wraz z odpowiadającym im plikiem `.daq` w katalogu głównym:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Wskaż folder w katalogu głównym plików przechwyconych** (`output/` powyżej). Gdy wybrany folder sam w sobie nie zawiera żadnych obrazów, ale ma podfoldery, Chloros automatycznie przechodzi do nich — podfoldery tego poziomu oraz folder główny `.daq` są pobierane za jednym razem.**Sposób importowania ujęć:*** Każde ujęcie jest importowane jako **pojedynczy obraz**, pogrupowany według ujęcia (nie jako osobny wpis dla każdego poziomu). Pozostałe poziomy tego samego ujęcia pojawiają się jako tryby wyświetlania tego jednego obrazu.
* **Przetwarzanie zawsze rozpoczyna się od surowej klatki.** Pozostałe poziomy są widoczne, ale przez potok danych przechodzi wyłącznie `raw` — ponowne przetwarzanie już przetworzonego produktu spowodowałoby podwójne zastosowanie korekt, więc Chloros jest odrzucany. Ponownie zaimportowany eksport nigdy nie może zająć miejsca przeznaczonego na surowy plik ujęcia.
* Folder ujęć zapisany **bez** surowych plików importuje się i wyświetla normalnie, ale przetwarzanie pomija go i odnotowuje to w dzienniku. (Flaga CLI `--input-level` może wymusić punkt wejścia w tym przypadku — zobacz [Podręcznik CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**Sesje hubu LATTICE** importuje się w ten sam sposób: należy wskazać opcję „Dodaj folder” i wskazać folder sesji skopiowany z huba (zawiera on pliki `raw/` oraz `previews/`), wraz z dowolnym dziennikiem DAQ-M `.csv` typu downwelling. Jeśli kalibracja kamery lub urządzenia DAQ nie została jeszcze zapisana w pamięci podręcznej na Twoim komputerze, Chloros pobierze ją automatycznie na podstawie numeru seryjnego podczas importu (wymaga jednorazowego połączenia z Internetem).***

## Opis tabeli przeglądarki plików

Po zaimportowaniu obrazów pojawiają się one w tabeli zawierającej następujące kolumny:

### Nazwa pliku

* Oryginalna nazwa pliku z aparatu
* Zachowuje konwencję nazewniczą aparatu (np. IMG\_0001.RAW lub capture\_20260816\_101500\_SN213800234\_raw.tif)

### Sygnatura czasowa

* Data i godzina wykonania zdjęcia
* Pobierana z metadanych EXIF zdjęcia
* Wykorzystywana do dopasowywania czujników światła, synchronizacji PPK oraz planowania celów kalibracyjnych

### Model aparatu

* Automatycznie wykryta konfiguracja aparatu i filtra
* Przykłady Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Przykłady LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Służy do stosowania prawidłowych profili przetwarzania

### Kolumna celów (pole wyboru)

* Zaznacz to pole wyboru dla obrazów zawierających cele kalibracyjne
* Gdy zaznaczono co najmniej jeden obraz, **tylko zaznaczone obrazy są skanowane** pod kątem obecności celów
* Szczegółowe informacje znajdziesz w sekcji [Wybieranie obrazów docelowych](choosing-target-images.md)

### Wyświetlanie metadanych obrazu

Kliknięcie przycisku przełączającego w prawym górnym rogu nad tabelą powoduje wyświetlenie metadanych wybranego obrazu w obszarze siatki obrazów.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Pliki czujników światła w projekcie

* Pliki `.daq` i `.csv` pojawiają się na liście przeglądarki plików, ale nie są obrazami, które można kliknąć — dostarczają one natężenie promieniowania padającego do kalibracji współczynnika odbicia.
* Każdy zaimportowany plik typu `.daq`/`.csv` jest wymieniony w sekcji **Ustawienia projektu → Czujnik światła DAQ**, gdzie można sprawdzić korekcję nasadki rozpraszającej obowiązującą dla każdego pliku. Zobacz [Dostosowywanie ustawień projektu](adjusting-project-settings.md).
* Zapisy wykonane w zakładce **Czujniki światła** są automatycznie dodawane do otwartego projektu — nie jest wymagany ręczny import.***

## Zarządzanie plikami w projekcie

### Usuwanie plików

Aby usunąć niepotrzebne obrazy z projektu:

1. Zaznacz jeden lub więcej obrazów w tabeli przeglądarki plików
2. Kliknij przycisk **„Usuń zaznaczone”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Potwierdź usunięcie (pliki nie są usuwane z dysku, a jedynie z projektu)

### Sortowanie i filtrowanie

* **Sortowanie według kolumny**: Kliknij dowolny nagłówek kolumny, aby posortować obrazy
* **Sortowanie według daty i godziny**: Przydatne do porządkowania sekwencji zdjęć w porządku chronologicznym
* **Filtr według modelu aparatu**: Grupuj obrazy według typu aparatu, jeśli używasz wielu aparatów***

## Podgląd zdjęć

### Wyświetlanie pełnego zdjęcia

Kliknij dowolną miniaturę zdjęcia w przeglądarce plików, aby wyświetlić je w głównym obszarze podglądu:

1. Zdjęcie pojawia się w środkowym panelu podglądu
2. Użyj elementów sterujących powiększeniem, aby obejrzeć szczegóły zdjęcia
3. Przechodź między zdjęciami za pomocą klawiszy strzałek

### Szybka nawigacja

* **Poprzedni obraz**: Kliknij strzałkę w lewo lub naciśnij klawisz ←
* **Następny obraz**: Kliknij strzałkę w prawo lub naciśnij klawisz →
* **Powiększanie/pomniejszanie**: Użyj kółka myszy lub przycisków powiększania
* **Przesuwanie**: Kliknij i przeciągnij na obrazie po powiększeniu***

## Obsługa zduplikowanych plików

Chloros automatycznie wykrywa i ignoruje zduplikowane pliki:

* Pliki o identycznych nazwach są pomijane
* Zapobiega przypadkowemu podwójnemu przetworzeniu
* Po wykryciu duplikatów wyświetlany jest komunikat ostrzegawczy

{% hint style="warning" %}
**Ważne**: Nie zmieniaj nazw ani nie modyfikuj oryginalnych plików obrazów przed importem. Chloros opiera się na oryginalnych nazwach plików i metadanych w celu prawidłowego przetwarzania.
{% endhint %}

***

## Zestawy danych z różnych aparatów

Jeśli projekt zawiera obrazy z wielu kamer MAPIR:

1. Chloros automatycznie wykrywa każdy model kamery — Survey3, LATTICE lub kombinację
2. Każdy typ kamery jest przetwarzany przy użyciu odpowiedniego profilu kalibracji
3. Przeglądarka plików wyświetla model kamery w kolumnie „Model kamery”
4. Po przetworzeniu każda kamera otrzymuje własną strukturę folderów wyjściowych

**Przykładowe scenariusze**: Survey3W RGN + Survey3N OCN — konfiguracja z dwiema kamerami, lub układ LATTICE z modułem głównym RGB i kilkoma modułami wąskopasmowymi***

## Najlepsze praktyki

### Porządkowanie przed importem

* Przechowuj obrazy celów kalibracyjnych w tym samym folderze co obrazy pomiarowe
* Pliki z czujników światła `.daq` / `.csv` z każdej sesji przechowuj razem z obrazami z tej sesji
* Zachowaj oryginalną strukturę folderów z aparatu/karty SD/koncentratora
* Nie mieszaj zbiorów danych z różnych sesji w jednym projekcie

### Nazewnictwo plików

* Zachowaj oryginalne nazwy plików z aparatu (IMG\_0001.RAW, capture\_..., itp.)
* Nie zmieniaj nazw plików przed importem
* Oryginalne nazwy zawierają ważne metadane

### Obrazy wzorców kalibracyjnych

* Zawsze dołączaj 1–2 obrazy wzorców kalibracyjnych na sesję (Survey3; w przypadku LATTICE można je zastąpić zapisem DAQ — zobacz [Wybór obrazów wzorców](choosing-target-images.md))
* Należy wykonać zdjęcia obiektów kalibracyjnych przed i po sesji rejestracji
* Obiekty kalibracyjne należy umieścić w takich samych warunkach oświetleniowych jak obszar rejestracji
* Obrazy obiektów kalibracyjnych należy oznaczyć, zaznaczając pole wyboru „Target”

***

## Typowe problemy i rozwiązania

### Obrazy nie pojawiają się po zaimportowaniu

**Możliwe przyczyny:**

* Format pliku nie jest obsługiwany (zobacz listę obsługiwanych typów na górze tej strony)
* Obrazy pochodzą z aparatów innych niż MAPIR (zobacz [Obsługiwane aparaty](../supported-cameras.md))
* Uszkodzenie pliku lub niekompletne przesłanie z karty SD

**Rozwiązanie**: Sprawdź zgodność formatu pliku i modelu aparatu oraz zapoznaj się z ostrzeżeniem dotyczącym wczytywania plików, aby zidentyfikować konkretne pliki, których import się nie powiódł

### Nie wykryto modelu aparatu

**Możliwe przyczyny:**

* Zmodyfikowane metadane EXIF
* Zdjęcia edytowane w zewnętrznym oprogramowaniu
* Niekompletny transfer plików

**Rozwiązanie**: Ponownie zaimportuj oryginalne, niezmodyfikowane pliki z aparatu/karty SD

### Brakujące sygnatury czasowe

**Możliwe przyczyny:**

* Nieprawidłowo ustawiony zegar aparatu
* Dane EXIF usunięte przez oprogramowanie zewnętrzne

**Rozwiązanie**: Sprawdź, czy ustawienia czasu w aparacie były prawidłowe podczas robienia zdjęć

### Ponowne otwarcie projektu zgłasza brakujące pliki

Jeśli pliki źródłowe zostały przeniesione lub usunięte od czasu ostatniego otwarcia projektu, komunikat Chloros wskaże Ci, **które** pliki zniknęły, zamiast otwierać pustą siatkę. Przywróć pliki do ich pierwotnych ścieżek lub usuń brakujące wpisy i zaimportuj je ponownie.***

## Kolejne kroki

Po zaimportowaniu plików:

1. **Przejrzyj listę plików** – upewnij się, że wszystkie obrazy zostały poprawnie załadowane
2. **Sprawdź modele aparatów** – zweryfikuj poprawność rozpoznania aparatu
3. **Zaznacz obrazy docelowe** — zobacz [Wybieranie obrazów docelowych](choosing-target-images.md)
4. **Dostosuj ustawienia** — skonfiguruj opcje przetwarzania w [Ustawieniach projektu](adjusting-project-settings.md)
5. **Rozpocznij przetwarzanie** — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)

Szczegółowe informacje na temat konfiguracji projektu można znaleźć w sekcji [Dostosowywanie ustawień projektu](adjusting-project-settings.md).
