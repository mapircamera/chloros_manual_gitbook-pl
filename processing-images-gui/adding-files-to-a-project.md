# Dodawanie plików do projektu

Po utworzeniu lub otwarciu projektu w Chloros, następnym krokiem jest dodanie obrazów wielospektralnych w celu rozpoczęcia przetwarzania. Przeglądarka plików<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ułatwia importowanie obrazów i zarządzanie zbiorem danych.

## Dostęp do przeglądarki plików

1. Otwórz lub utwórz projekt w Chloros
2. Kliknij ikonę **Przeglądarka plików** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na lewym pasku bocznym
3. Panel przeglądarki plików wyświetli listę plików projektu

{% hint style="info" %}
**Obsługiwane typy plików**: Chloros obsługuje pliki obrazów RAW+JPG i JPG z aparatów MAPIR Survey3W i Survey3N. Zalecane są tylko pliki RAW+JPG.
{% endhint %}

***

## Dodawanie obrazów do projektu

Istnieją dwa podstawowe sposoby dodawania obrazów do projektu:

### Metoda 1: Dodawanie plików

Użyj tej opcji, aby zaimportować pojedyncze pliki obrazów lub niewielką grupę plików.

1. Kliknij przycisk **„Dodaj pliki”** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> u góry panelu przeglądarki plików
2. Przejdź do folderu zawierającego obrazy
3. Wybierz jeden lub więcej plików obrazów (przytrzymaj klawisz **Ctrl**, aby wybrać wiele plików)
4. Kliknij **„Otwórz”**, aby zaimportować wybrane pliki

### Metoda 2: Dodaj folder

Użyj tej opcji, aby zaimportować wszystkie obrazy z folderu jednocześnie.

1. Kliknij przycisk **„Dodaj folder”** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> u góry panelu przeglądarki plików.
2. Przejdź do folderu zawierającego obrazy z sesji przechwytywania i wybierz go.
3. Kliknij przycisk **„Wybierz folder”**, aby zaimportować wszystkie obsługiwane obrazy z tego folderu.

***

## Informacje o tabeli przeglądarki plików

Po zaimportowaniu obrazy pojawiają się w tabeli zawierającej następujące kolumny:

### Nazwa pliku

* Oryginalna nazwa pliku z aparatu
* Zachowuje konwencję nazewniczą aparatu (np. IMG\_0001.RAW)

### Sygnatura czasowa

* Data i godzina wykonania zdjęcia
* Wyodrębniona z metadanych EXIF zdjęcia
* Używana do synchronizacji PPK i wykrywania celu kalibracji

### Model aparatu

* Automatycznie wykryta konfiguracja aparatu i filtra
* Przykłady: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Służy do zastosowania prawidłowych profili przetwarzania

### Kolumna celu (pole wyboru)

* Zaznacz to pole dla obrazów zawierających cele kalibracyjne
* Znacznie przyspiesza wykrywanie celów podczas przetwarzania
* Szczegółowe informacje można znaleźć w sekcji [Wybieranie obrazów docelowych](choosing-target-images.md)

### Wyświetlanie metadanych obrazu

Kliknięcie przycisku przełączającego w prawym górnym rogu nad tabelą powoduje wyświetlenie metadanych wybranego obrazu w obszarze siatki obrazów.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Zarządzanie plikami w projekcie

### Usuwanie plików

Aby usunąć niepotrzebne obrazy z projektu:

1. Wybierz jeden lub więcej obrazów w tabeli przeglądarki plików
2. Kliknij przycisk **„Usuń wybrane”** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Potwierdź usunięcie (pliki nie są usuwane z dysku, a jedynie z projektu).

### Sortowanie i filtrowanie

* **Sortuj według kolumny**: kliknij dowolny nagłówek kolumny, aby posortować obrazy.
* **Sortowanie według daty**: przydatne do organizowania sekwencji zdjęć w porządku chronologicznym.
* **Filtr modelu aparatu**: grupuj obrazy według typu aparatu, jeśli używasz wielu aparatów.

***

## Podgląd obrazu

### Wyświetlanie pełnego obrazu

Kliknij dowolną miniaturę obrazu w przeglądarce plików, aby wyświetlić go w głównym obszarze podglądu:

1. Obraz pojawi się w środkowym panelu podglądu.
2. Użyj elementów sterujących powiększaniem, aby sprawdzić szczegóły obrazu.
3. Przechodź między obrazami za pomocą klawiszy strzałek.

### Szybka nawigacja

* **Poprzedni obraz**: kliknij strzałkę w lewo lub naciśnij klawisz ←.
* **Następny obraz**: kliknij strzałkę w prawo lub naciśnij klawisz →.
* **Powiększ/pomniejsz**: użyj kółka myszy lub przycisków powiększania.
* **Przesuwanie**: kliknij i przeciągnij obraz po powiększeniu.

***

## Obsługa zduplikowanych plików

Chloros automatycznie wykrywa i ignoruje zduplikowane pliki:

* Pliki o identycznych nazwach są pomijane
* Zapobiega przypadkowemu podwójnemu przetwarzaniu
* W przypadku wykrycia duplikatów wyświetlany jest komunikat ostrzegawczy

{% hint style="warning" %}
**Ważne**: Nie zmieniaj nazw ani nie modyfikuj oryginalnych plików obrazów przed importem. Chloros wykorzystuje oryginalne nazwy plików i metadane do prawidłowego przetwarzania.
{% endhint %}

***

## Zestawy danych z różnych kamer

Jeśli projekt zawiera obrazy z wielu kamer MAPIR:

1. Chloros automatycznie wykrywa każdy model kamery
2. Każdy typ kamery jest przetwarzany przy użyciu odpowiedniego profilu kalibracji
3. Przeglądarka plików wyświetla model aparatu w kolumnie Model aparatu.
4. Przetwarzanie stosuje prawidłowe ustawienia dla każdego typu aparatu.

**Przykładowy scenariusz**: Survey3W RGN + Survey3N OCN konfiguracja z dwoma aparatami.

***

## Najlepsze praktyki

### Porządkowanie przed importem

* Przechowuj obrazy docelowe kalibracji w tym samym folderze, co obrazy pomiarowe.
* Zachowaj oryginalną strukturę folderów z aparatu/karty SD.
* Nie mieszaj zestawów danych z różnych sesji w jednym projekcie.

### Nazewnictwo plików

* Zachowaj oryginalne nazwy plików aparatu (IMG\_0001.RAW itp.).
* Nie zmieniaj nazw plików przed importem.
* Oryginalne nazwy zawierają ważne metadane.

### Obrazy kalibracyjne

* Zawsze dołączaj 1-2 obrazy kalibracyjne na sesję.
* Przechwytuj obrazy kalibracyjne przed i po sesji przechwytywania.
* Umieść obrazy kalibracyjne w takich samych warunkach oświetleniowych jak obszar przechwytywania.
* Zaznacz obrazy kalibracyjne za pomocą pola wyboru Target, aby przyspieszyć przetwarzanie.

***

## Typowe problemy i rozwiązania

### Obrazy nie pojawiają się po imporcie

**Możliwe przyczyny:**

* Nieobsługiwany format pliku (tylko RAW+JPG i JPG z aparatów MAPIR)
* Obrazy pochodzą z aparatów innych niż MAPIR (zobacz [Obsługiwane aparaty](../supported-cameras.md))
* Uszkodzenie pliku lub niekompletny transfer z karty SD

**Rozwiązanie**: Sprawdź format pliku i zgodność modelu aparatu

### Model aparatu nie został wykryty

**Możliwe przyczyny:**

* Zmodyfikowane metadane EXIF
* Obrazy edytowane w zewnętrznym oprogramowaniu
* Niekompletny transfer plików

**Rozwiązanie**: Ponownie zaimportuj oryginalne, niezmodyfikowane pliki z aparatu/karty SD

### Brakujące znaczniki czasu

**Możliwe przyczyny:**

* Nieprawidłowe ustawienie zegara aparatu
* Usunięcie danych EXIF przez oprogramowanie zewnętrzne

**Rozwiązanie**: Sprawdź, czy ustawienia czasu aparatu były prawidłowe podczas robienia zdjęć

***

## Kolejne kroki

Po zaimportowaniu plików:

1. **Przejrzyj listę plików** — upewnij się, że wszystkie zdjęcia zostały załadowane poprawnie
2. **Sprawdź modele aparatów** — upewnij się, że aparaty zostały poprawnie wykryte
3. **Zaznacz docelowe zdjęcia** — zobacz [Wybieranie docelowych zdjęć](choosing-target-images.md)
4. **Dostosuj ustawienia** — skonfiguruj opcje przetwarzania w [Ustawieniach projektu](adjusting-project-settings.md)
5. **Rozpocznij przetwarzanie** — zobacz [Rozpoczęcie przetwarzania](starting-the-processing.md)

Szczegółowe informacje na temat konfiguracji projektu można znaleźć w sekcji [Dostosowywanie ustawień projektu](adjusting-project-settings.md).
