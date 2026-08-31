# Znaczniki na mapie

Zakładka „Mapa” wyświetla zdjęcia na interaktywnej mapie 2D na podstawie ich współrzędnych GPS. Zapewnia to ogólny przegląd geograficzny sesji zdjęciowej i stanowi najszybszy sposób – zaraz po zaimportowaniu – na odrzucenie zdjęć, których nie chcesz przetwarzać.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Dostęp do zakładki „Mapa”

1. Otwórz lub utwórz projekt w Chloros
2. Zaimportuj zdjęcia zawierające metadane GPS
3. Kliknij zakładkę **Mapa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> na lewym pasku bocznym
4. Na mapie wyświetli się znacznik w lokalizacji GPS każdego zdjęcia

{% hint style="info" %}
**Wymagane dane GPS**: na mapie pojawiają się tylko zdjęcia z współrzędnymi GPS w metadanych EXIF. Zdjęcie bez współrzędnych nadal znajduje się w projekcie i jest przetwarzane normalnie — po prostu nie ma znacznika.
{% endhint %}

***

## Edytowanie zdjęć z zakładki „Mapa”

Zakładka **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> zawiera te same przyciski dodawania <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> i usuwania <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> plików, co zakładka [**Przeglądarka plików**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Wyświetla tę samą listę plików projektu wraz z kolumnami geograficznymi:

| Kolumna        | Zawartość                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nazwa**      | Nazwa pliku w postaci, w jakiej została zapisana przez aparat                             |
| **Szerokość geograficzna** | Stopnie dziesiętne, sześć miejsc po przecinku                                |
| **Długość geograficzna** | Stopnie w ułamku dziesiętnym, sześć miejsc po przecinku                                |
| **Wysokość nad poziomem morza** | Metry, jedno miejsce po przecinku — `-`, gdy zdjęcie nie zawiera informacji o wysokości |

{% hint style="info" %}
Kliknij dowolny nagłówek kolumny, aby posortować dane według tej kolumny; kliknij ponownie, aby odwrócić kolejność.
{% endhint %}

{% hint style="warning" %}
**Wysokość to wysokość nad poziomem morza, a nie wysokość nad ziemią.** Wartość pochodzi z tagu EXIF `GPSAltitude` obrazu, który odnosi się do średniego poziomu morza. Nie jest to wysokość lotu nad terenem, a Chloros nie wyliczy na jej podstawie odległości próbkowania naziemnego — nad polem położonym 300 m n.p.m. dron lecący na wysokości 100 m nad ziemią (AGL) zarejestruje tutaj wartość około 400 m. Kolumnę tę należy wykorzystywać do wykrywania wartości odstających i potwierdzania spójnej wysokości lotu, a nie jako pomiar wysokości nad ziemią (AGL).
{% endhint %}

***

## Znaczniki zdjęć

Każde zdjęcie z danymi GPS otrzymuje znacznik w swoich współrzędnych.

### Wyświetlanie znaczników

* Znaczniki znajdują się dokładnie w współrzędnych zarejestrowanych dla każdego zdjęcia
* Znaczniki położone blisko siebie mogą się wizualnie nakładać przy pomniejszeniu widoku — należy powiększyć widok, aby je rozdzielić
* Wybrane i podświetlone znaczniki są wyświetlane nad pozostałymi

### Podgląd po najechaniu kursorem

* **Najedź kursorem** na dowolny znacznik, aby wyświetlić miniaturę tego zdjęcia wraz z nazwą pliku
* **Kliknij**znacznik, aby wybrać zdjęcie i**przypiąć** okienko podglądu — pozostanie ono widoczne do momentu kliknięcia w inne miejsce. Gdy okienko podglądu jest przypięte, najechanie kursorem na inne znaczniki nie powoduje jego zniknięcia
* Jest to szybki sposób na znalezienie konkretnej klatki w dużej sesji bez opuszczania mapy

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Zakładka „Mapa” wyświetla wszystkie zdjęcia z geotagami w projekcie</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: po osiągnięciu maksymalnego powiększenia, dla którego dostawca kafelków udostępnia zdjęcia, dalsze powiększanie powoduje powiększanie samych kafelków zamiast zatrzymywania się, dzięki czemu można rozróżnić znaczniki, które znajdują się niemal jeden na drugim.
{% endhint %}

* Funkcja super-zoom włącza się tylko wtedy, gdy znajdujesz się **na** maksymalnym poziomie powiększenia dostawcy dla danej lokalizacji, a ładowanie kafelków zostało zakończone. Poniżej tego poziomu powiększanie działa normalnie
* Zakres wynosi **od 1× do 32×** ponad maksymalny poziom powiększenia ustalony przez dostawcę
* Wskaźnik w rogu pokazuje aktualny poziom super-powiększenia w procentach, a przycisk **×** obok niego pozwala jednym kliknięciem powrócić do normalnego powiększenia
* Pomniejszanie zawsze przenosi użytkownika z powrotem do samej mapy, więc nigdy nie utkniesz w trybie super-powiększenia
* Powiększanie i przesuwanie podczas superpowiększenia powoduje przeniesienie wynikowego przesunięcia z powrotem na mapę, dzięki czemu obszar poza centrum, do którego się przemieściłeś, nadal żąda kafelków zamiast stać się pusty
* Znaczniki są rysowane jako elementy wektorowe, a nie rastrowe, dzięki czemu pozostają ostre na każdym poziomie superpowiększenia

***

## Dostawcy kafelków mapy

{% hint style="success" %}
**Wybór automatyczny**: Chloros wybiera serwis kafelków, który oferuje najlepszy poziom powiększenia dla lokalizacji, w której znajdują się Twoje obrazy. W każdej chwili możesz ręcznie zmienić wybór.
{% endhint %}

| Dostawca        | Uwagi                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Szeroki zasięg na całym świecie; obsługuje wszystkie cztery typy kafelków                                                                                                            |
| **Esri ArcGIS**| Często zdjęcia lotnicze o wyższej rozdzielczości w określonych regionach. Typ kafelków**Teren** nie jest dostępny dla Esri, a jego przycisk jest nieaktywny, gdy wybrano Esri |***

## Rodzaje kafelków mapy

Wybierz rodzaj warstwy mapy za pomocą przycisków (od lewej do prawej):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Rodzaj                 | Wyświetla                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Teren**          | Cieniowanie wysokości z detalami mapy (drogi, etykiety). Dostępne tylko w Google       |
| **Mapa**              | Standardowe kafelki mapy ulicznej — opcja wymagająca najmniejszej przepustowości              |
| **Satelita**        | Szczegółowe zdjęcia satelitarne, bez etykiet — opcja wymagająca największej przepustowości |
| **Hybrydowa** (domyślna) | Zdjęcia satelitarne z naniesionymi drogami i etykietami                |

Karta „Mapa” otwiera się w trybie **Hybrydowym**. Wybrany tryb jest przenoszony przy zmianie dostawcy, o ile ten to obsługuje.***

## Nawigacja po mapie

* **Powiększanie**: kółko myszy lub przyciski powiększania na mapie
* **Przesuwanie**: kliknij i przeciągnij
* **Tryb pełnoekranowy**: przycisk trybu pełnoekranowego powiększa mapę do rozmiarów całego okna***

## Przykłady zastosowań

### Przegląd trasy lotu

* Szybki przegląd obszaru objętego sesją dronową
* Wykrywanie luk w pokryciu, gdzie pominięto przejście
* Sprawdzanie, czy lot przebiegał zgodnie z zaplanowanym schematem

### Przegląd pomiarów naziemnych

* Sprawdzanie rozmieszczenia zdjęć naziemnych
* Lokalizowanie ramek celów kalibracyjnych względem obszaru pomiarowego
* Decydowanie, gdzie potrzebne są dodatkowe zdjęcia

### Kontrola jakości

* Znajdź zdjęcia wykonane w nieoczekiwanych miejscach i usuń je przed przetwarzaniem
* Sortuj według wysokości, aby wykryć zdjęcie wykonane na niewłaściwej wysokości lub takie, na którym pozycjonowanie GPS było słabe
* Porównaj lokalizacje zdjęć z notatkami terenowymi

***

## Rozwiązywanie problemów

### Nie wyświetlają się znaczniki

**Możliwe przyczyny**

* Zdjęcia nie zawierają metadanych GPS
* Podczas robienia zdjęć w aparacie wyłączono GPS
* Dane EXIF zostały usunięte przez inne oprogramowanie przed importem

**Co zrobić**: upewnij się, że GPS jest włączony w aparacie, a następnie ponownie zaimportuj oryginalne pliki. Możesz sprawdzić, czy dany plik zawiera współrzędne, szukając go w tabeli plików w zakładce „Mapa” — zdjęcie bez współrzędnych nie ma tam swojego wiersza.

### Znaczniki znajdują się w niewłaściwym miejscu

**Możliwe przyczyny**: słaby sygnał satelitarny w momencie wykonania zdjęcia lub dryft GPS podczas sesji.**Co należy zrobić**: jest to problem związany z momentem wykonania zdjęcia, a nie coś, co Chloros może skorygować po fakcie. W celu uzyskania precyzji należy stosować procedurę z wykorzystaniem GPS PPK/RTK — zobacz ustawienie**Zastosuj korekty PPK** w [Ustawieniach projektu](../project-settings/project-settings.md).

### Mapa jest pusta lub przestały się ładować kafelki

Dostawcy kafelków to usługi internetowe. Jeśli kafelki przestaną się ładować, sprawdź połączenie sieciowe urządzenia, a następnie spróbuj zmienić dostawcę. Jeśli byłeś w trybie bardzo dużego powiększenia, naciśnij przycisk resetowania **×**, aby powrócić do normalnego poziomu powiększenia i pozwolić mapie ponownie zażądać kafelków.***

## Powiązane strony

* [**Siatka obrazów**](image-grid.md) — ten sam zestaw obrazów co miniatury
* [**Otwieranie obrazu na pełnym ekranie**](opening-an-image-full-screen.md) — szczegółowe przeglądanie jednego obrazu
* [**Dodawanie plików do projektu**](../processing-images-gui/adding-files-to-a-project.md) — przyciski dodawania/usuwania plików dostępne na tej zakładce
