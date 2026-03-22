# Dodawanie plików do projektu

Po utworzeniu lub otwarciu projektu w Chloros kolejnym krokiem jest dodanie obrazów wielospektralnych w celu rozpoczęcia przetwarzania. Zakładka Przeglądarka plików<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ułatwia importowanie obrazów i zarządzanie zbiorem danych.

## Dostęp do przeglądarki plików

1. Otwórz lub utwórz projekt w Chloros
2. Kliknij ikonę **Przeglądarka plików** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. W panelu Przeglądarki plików wyświetli się lista plików projektu

{% hint style="info" %}
**Obsługiwane typy plików**: Chloros obsługuje pliki obrazów RAW+JPG i JPG z kamer MAPIR, Survey3W i Survey3N. Zalecane są wyłącznie pliki RAW+JPG.
{% endhint %}

***

## Dodawanie obrazów do projektu

Istnieją dwa podstawowe sposoby dodawania obrazów do projektu:

### Metoda 1: Dodaj pliki

Użyj tej opcji, aby zaimportować pojedyncze pliki obrazów lub niewielki zestaw plików.

1. Kliknij przycisk **„Dodaj pliki”** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> u góry panelu przeglądarki plików
2. Przejdź do folderu zawierającego obrazy
3. Wybierz jeden lub więcej plików obrazów (przytrzymaj klawisz **Ctrl**, aby zaznaczyć wiele plików)
4. Kliknij **„Otwórz”**, aby zaimportować wybrane pliki

### Metoda 2: Dodaj folder

Użyj tej opcji, aby zaimportować wszystkie obrazy z folderu jednocześnie.

1. Kliknij przycisk **„Dodaj folder”** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> w górnej części panelu Przeglądarki plików
2. Przejdź do folderu zawierającego zdjęcia z sesji i wybierz go
3. Kliknij **„Wybierz folder”**, aby zaimportować wszystkie obsługiwane zdjęcia z tego folderu***

## Omówienie tabeli przeglądarki plików

Po zaimportowaniu zdjęcia pojawiają się w tabeli zawierającej następujące kolumny:

### Nazwa pliku

* Oryginalna nazwa pliku z aparatu
* Zachowuje konwencję nazewniczą aparatu (np. IMG\_0001.RAW)

### Sygnatura czasowa

* Data i godzina wykonania zdjęcia
* Pobierana z metadanych EXIF zdjęcia
* Wykorzystywana do synchronizacji PPK i wykrywania celów kalibracyjnych

### Model aparatu

* Automatycznie wykryta konfiguracja aparatu i filtra
* Przykłady: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Służy do zastosowania odpowiednich profili przetwarzania

### Kolumna celów (pole wyboru)

* Zaznacz to pole wyboru dla obrazów zawierających cele kalibracyjne
* Znacznie przyspiesza wykrywanie celów podczas przetwarzania
* Szczegółowe informacje znajdziesz w sekcji [Wybieranie obrazów docelowych](choosing-target-images.md)

### Wyświetlanie metadanych obrazu

Kliknięcie przycisku przełączającego w prawym górnym rogu nad tabelą powoduje wyświetlenie metadanych wybranego obrazu w obszarze siatki obrazów.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Zarządzanie plikami w projekcie

### Usuwanie plików

Aby usunąć niepotrzebne obrazy z projektu:

1. Wybierz jeden lub więcej obrazów w tabeli przeglądarki plików
2. Kliknij przycisk **„Usuń wybrane”** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> 3. Potwierdź usunięcie (pliki nie są usuwane z dysku, a jedynie z projektu)

### Sortowanie i filtrowanie

* **Sortowanie według kolumny**: Kliknij dowolny nagłówek kolumny, aby posortować obrazy
* **Sortowanie według daty i godziny**: Przydatne do porządkowania sekwencji zdjęć w porządku chronologicznym
* **Filtr modelu aparatu**: Grupuj obrazy według typu aparatu, jeśli korzystasz z wielu aparatów***

## Podgląd obrazu

### Wyświetlanie pełnego obrazu

Kliknij dowolną miniaturę obrazu w przeglądarce plików, aby wyświetlić ją w głównym obszarze podglądu:

1. Obraz pojawia się w środkowym panelu podglądu
2. Użyj elementów sterujących powiększeniem, aby sprawdzić szczegóły obrazu
3. Przechodź między obrazami za pomocą klawiszy strzałek

### Szybka nawigacja

* **Poprzedni obraz**: Kliknij strzałkę w lewo lub naciśnij klawisz ←
* **Następny obraz**: Kliknij strzałkę w prawo lub naciśnij klawisz →
* **Powiększanie/pomniejszanie**: Użyj kółka myszy lub przycisków powiększania
* **Przesuwanie**: Kliknij i przeciągnij obraz po powiększeniu***

## Obsługa duplikatów plików

Chloros automatycznie wykrywa i ignoruje duplikaty plików:

* Pliki o identycznych nazwach są pomijane
* Zapobiega przypadkowemu podwójnemu przetwarzaniu
* Po wykryciu duplikatów wyświetlany jest komunikat ostrzegawczy

{% hint style="warning" %}
**Ważne**: Nie zmieniaj nazw ani nie modyfikuj oryginalnych plików obrazów przed importem. Chloros opiera się na oryginalnych nazwach plików i metadanych w celu prawidłowego przetwarzania.
{% endhint %}

***

## Zestawy danych z różnych kamer

Jeśli projekt zawiera obrazy z wielu kamer MAPIR:

1. Chloros automatycznie wykrywa model każdej kamery
2. Każdy typ kamery jest przetwarzany przy użyciu odpowiedniego profilu kalibracji
3. Przeglądarka plików wyświetla model kamery w kolumnie „Model kamery”
4. Przetwarzanie stosuje odpowiednie ustawienia dla każdego typu kamery

**Przykładowy scenariusz**: Konfiguracja z dwoma kamerami: Survey3W + RGN + Survey3N + OCN***

## Najlepsze praktyki

### Porządkowanie przed importem

* Przechowuj obrazy wzorców kalibracyjnych w tym samym folderze co obrazy pomiarowe
* Zachowaj oryginalną strukturę folderów z aparatu/karty SD
* Nie mieszaj zestawów danych z różnych sesji w jednym projekcie

### Nazewnictwo plików

* Zachowaj oryginalne nazwy plików z aparatu (IMG\_0001.RAW itp.)
* Nie zmieniaj nazw plików przed importem
* Oryginalne nazwy zawierają ważne metadane

### Obrazy celów kalibracyjnych

* Zawsze dołączaj 1–2 obrazy celów kalibracyjnych na sesję
* Zrób zdjęcia celów przed i po sesji fotografowania
* Umieść cele w takich samych warunkach oświetleniowych jak obszar fotografowania
* Oznacz obrazy celów za pomocą pola wyboru „Target”, aby przyspieszyć przetwarzanie

***

## Typowe problemy i rozwiązania

### Obrazy nie pojawiają się po zaimportowaniu

**Możliwe przyczyny:**

* Format pliku nie jest obsługiwany (tylko RAW+JPG i JPG z aparatów MAPIR)
* Obrazy pochodzą z aparatów innych niż MAPIR (zobacz [Obsługiwane aparaty](../supported-cameras.md))
* Uszkodzenie pliku lub niekompletny transfer z karty SD

**Rozwiązanie**: Sprawdź zgodność formatu pliku i modelu aparatu

### Model aparatu nie został wykryty

**Możliwe przyczyny:**

* Zmodyfikowane metadane EXIF
* Zdjęcia edytowane w oprogramowaniu zewnętrznym
* Niekompletny transfer plików

**Rozwiązanie**: Ponownie zaimportuj oryginalne, niezmodyfikowane pliki z aparatu/karty SD

### Brakujące znaczniki czasu

**Możliwe przyczyny:**

* Nieprawidłowo ustawiony zegar aparatu
* Dane EXIF usunięte przez oprogramowanie zewnętrzne

**Rozwiązanie**: Sprawdź, czy ustawienia czasu w aparacie były prawidłowe podczas robienia zdjęć***

## Kolejne kroki

Po zaimportowaniu plików:

1. **Przejrzyj listę plików** – upewnij się, że wszystkie zdjęcia zostały poprawnie załadowane
2. **Sprawdź modele aparatów** — upewnij się, że aparaty zostały poprawnie wykryte
3. **Zaznacz docelowe zdjęcia** — zobacz [Wybieranie docelowych zdjęć](choosing-target-images.md)
4. **Dostosuj ustawienia** — skonfiguruj opcje przetwarzania w [Ustawieniach projektu](adjusting-project-settings.md)
5. **Rozpocznij przetwarzanie** — zobacz [Rozpoczynanie przetwarzania](starting-the-processing.md)

Szczegółowe informacje na temat konfiguracji projektu znajdziesz w sekcji [Dostosowywanie ustawień projektu](adjusting-project-settings.md).
