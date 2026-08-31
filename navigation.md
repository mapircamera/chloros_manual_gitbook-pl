# Interfejs graficzny: Nawigacja

Po pierwszym uruchomieniu programu Chloros uruchamia się jego moduł przetwarzający. Gdy moduł ten będzie gotowy, w lewym górnym rogu pojawi się ikona menu głównego <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, a w lewym pasku bocznym odblokują się zakładki „Kamery” i „Czujniki światła” (do tego momentu są one wyszarzone).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

W górnym nagłówku, od lewej do prawej, znajdują się:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu główne

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Z poziomu menu głównego można:

* **Nowy projekt**— utworzyć nowy projekt. Jeśli masz zapisane szablony projektów, pojawi się lista rozwijana**Wybierz szablon**, dzięki czemu nowy projekt rozpocznie się od ustawień szablonu.
* **Otwórz projekt**— otworzyć istniejący projekt. Lista zawiera przycisk**Otwórz folder projektu**, który otwiera folder projektów w eksploratorze plików.
* **Duplikuj projekt** — skopiuj aktualnie otwarty projekt pod nową nazwą (sugerowana jest dowolna nazwa, np. „MójProjekt (2)”) i otwórz kopię. _(widoczne po otwarciu projektu)_
* **Dodaj pliki** — dodaj pojedyncze pliki obrazów do bieżącego projektu _(widoczne po otwarciu projektu)_
* **Dodaj folder** — dodaj jeden lub więcej folderów z obrazami do bieżącego projektu _(widoczne po otwarciu projektu)_
* **Rozpocznij przetwarzanie / Zatrzymaj przetwarzanie** — uruchom lub zatrzymaj proces przetwarzania obrazów _(opcja dostępna po dodaniu plików)_
* **Podłącz do kamery** — przejdź do [zakładki Kamery](lattice/), aby podłączyć kamerę lub matrycę LATTICE. Działa bez otwartego projektu.
* **Połącz z czujnikiem światła** — przejście do [zakładki „Czujniki światła”](daq/) w celu podłączenia czujnika światła DAQ. Działa bez otwartego projektu.

{% hint style="info" %}
**Tylko Windows**: Interfejs graficzny Chloros Desktop GUI jest dostępny na Windows. Użytkownicy modelu Linux powinni zapoznać się z dokumentacją dotyczącą modeli [CLI](CLI.md) oraz [Python SDK](api-python-sdk.md) dotyczącą przetwarzania bezinterfejsu graficznego.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Przycisk odtwarzania/startu

Po włączeniu przycisk rozpoczęcia przetwarzania uruchamia potok przetwarzania obrazów.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Pasek postępu<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

W bezpłatnym trybie Chloros, który przetwarza wszystkie pliki sekwencyjnie, pasek postępu pokazuje 2 etapy: wykrywanie obiektów i przetwarzanie.

W płatnym trybie licencjonowanym Chloros+, który przetwarza wszystkie pliki jednocześnie, pasek postępu pokazuje 4 etapy: wykrywanie, analizę, kalibrację i eksport. Po najechaniu kursorem myszy na pasek postępu Chloros+ rozwinie się rozszerzony panel z 4 paskami postępu, dzięki czemu można śledzić przebieg procesu. Kliknięcie górnego paska postępu spowoduje zamrożenie panelu rozwijanego, a kolejne kliknięcie – jego odblokowanie.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu boczne

Menu na lewym pasku bocznym zawiera różne ikony umożliwiające interakcję, w następującej kolejności od góry do dołu:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Ustawienia projektu](project-settings/project-settings.md)

Zakładka „Ustawienia projektu” pozwala dostosować globalne ustawienia projektu oraz ustawienia przetwarzania. Należy je dostosować przed rozpoczęciem przetwarzania plików.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Przeglądarka plików

Dodawaj pliki/foldery i usuwaj pliki z projektu. Pliki zduplikowane są ignorowane. Zaznacz pole w kolumnie „docelowe” obok dowolnego obrazu docelowego, a przetwarzanie będzie uwzględniało wyłącznie zaznaczone obrazy jako docelowe, co znacznie przyspieszy czas przetwarzania. Użyj przełącznika „Obraz/Metadane”, aby przełączać się między widokiem siatki miniatur wybranych obrazów a szczegółową tabelą metadanych.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Przeglądarka obrazów](image-viewer-gui/opening-an-image-full-screen.md)

Po kliknięciu obrazu w głównej przeglądarce obrazów zostanie on otwarty na pełnym ekranie w zakładce „Przeglądarka obrazów”.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Przeglądarka map](image-viewer-gui/map-markers.md)

Przeglądaj swoje zdjęcia na interaktywnej mapie 2D na podstawie ich współrzędnych GPS. Obsługuje dostawców kafelków Google Maps i ESRI, automatycznie wybierając najlepszą usługę dla danej lokalizacji. Najedź kursorem na znaczniki, aby wyświetlić podgląd miniatur zdjęć.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamery](lattice/)

Podłączaj i steruj kamerami LATTICE na żywo — pojedynczo lub jako zsynchronizowane układy wielokamerowe. Zakładka ta wyświetla kafelki podglądu na żywo z nakładkami i histogramami, ustawienia dla poszczególnych kamer i układów oraz ustawienia przechwytywania, które określają, które kamery i typy eksportu są generowane przez opcję „Capture All”. Funkcja dostępna po przygotowaniu zaplecza; pełny przewodnik znajdziesz w [sekcji LATTICE](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Czujniki światła](daq/)

Podłącz czujniki światła DAQ — DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet) — i przeglądaj ich skalibrowane wykresy widma w czasie rzeczywistym w jednostkach W/m²/nm. W tym miejscu można zapisywać pliki `.daq` do otwartego projektu, zmieniać nazwy czujników, wybierać profile korekcji nasłonecznienia oraz aktualizować oprogramowanie sprzętowe DAQ-E. Funkcja dostępna po uruchomieniu zaplecza; pełny opis znajdziesz w [sekcji DAQ](daq/).

#### Dziennik debugowania <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

W razie wystąpienia problemów przejrzyj dziennik pod kątem wpisów debugowania. Skopiuj lub pobierz dziennik i wyślij go do [pomocy technicznej MAPIR](https://www.mapir.camera/community/contact) w celu uzyskania pomocy.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Logowanie użytkownika](chloros+-login.md)

Pasek boczny logowania użytkownika umożliwia zalogowanie się na konto Chloros+, co pozwala odblokować zaawansowane funkcje. Możesz również sprawdzić aktualną wersję aplikacji, a także dostosować język wyświetlanego tekstu w interfejsie graficznym Chloros oraz CLI.
