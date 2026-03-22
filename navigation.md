# Interfejs graficzny: Nawigacja

Po pierwszym uruchomieniu programów Chloros i Chloros (przeglądarka) uruchomi się ich moduł zaplecza. Gdy będzie gotowy, w lewym górnym rogu pojawi się ikona menu głównego <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Od lewej do prawej strony górny nagłówek zawiera:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu główne

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Z poziomu menu głównego można:

* **Nowy projekt** — utworzyć nowy projekt
* **Otwórz projekt** — otworzyć istniejący projekt
* **Otwórz folder projektu** — otworzyć folder projektu w eksploratorze plików
* **Dodaj pliki** — dodać pojedyncze pliki obrazów do bieżącego projektu _(widoczne po otwarciu projektu)_
* **Dodaj folder** — dodaj folder z obrazami do bieżącego projektu _(widoczne po otwarciu projektu)_
* **Rozpocznij przetwarzanie / Zatrzymaj przetwarzanie** — uruchom lub zatrzymaj proces przetwarzania obrazów _(dostępne po dodaniu plików)_

{% hint style="info" %}
**Tylko Windows**: Interfejs graficzny Chloros Desktop jest dostępny w Windows. Użytkownicy Linux powinni zobaczyć [CLI](CLI.md) oraz [Python SDK](api-python-sdk.md) dotyczącą przetwarzania bezinterfejsowego.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Przycisk odtwarzania/startu

Po włączeniu przycisk rozpoczęcia przetwarzania uruchamia potok przetwarzania obrazu.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Pasek postępu <img src=".gitbook/assets/image (5).png" alt="" data-size="line">W darmowym trybie Chloros, który przetwarza wszystkie pliki sekwencyjnie, pasek postępu pokazuje 2 etapy: Wykrywanie celu i Przetwarzanie.

W płatnym trybie Chloros+, który przetwarza wszystkie pliki jednocześnie, pasek postępu pokazuje 4 etapy: Wykrywanie, Analiza, Kalibracja, Eksport. Po najechaniu kursorem myszy na pasek postępu Chloros+ rozwinie się rozszerzony panel z 4 paskami postępu, dzięki czemu można śledzić postęp. Kliknięcie górnego paska postępu spowoduje zamrożenie panelu rozwijanego, a ponowne kliknięcie spowoduje jego odblokowanie.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu boczne

Menu na lewym pasku bocznym zawiera różne ikony, z którymi można wchodzić w interakcję:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Ustawienia projektu](project-settings/project-settings.md)

Zakładka Ustawienia projektu pozwala dostosować globalne ustawienia projektu oraz ustawienia przetwarzania. Dostosuj je przed rozpoczęciem przetwarzania plików.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Przeglądarka plików

Dodawaj pliki/foldery i usuwaj pliki z projektu. Pliki zduplikowane są ignorowane. Zaznacz pole w kolumnie docelowej dla dowolnego obrazu docelowego, a przetwarzanie będzie uwzględniało tylko zaznaczone obrazy jako cele, co znacznie przyspieszy czas przetwarzania. Użyj przełącznika Obraz/Metadane, aby przełączać się między wyświetlaniem siatki miniatur wybranego obrazu a szczegółową tabelą metadanych.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Przeglądarka obrazów](image-viewer-gui/opening-an-image-full-screen.md)

Po kliknięciu obrazu w głównej przeglądarce obrazów zostanie on otwarty na pełnym ekranie w zakładce Przeglądarka obrazów.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Przeglądaj swoje zdjęcia na interaktywnej mapie 2D na podstawie ich współrzędnych GPS. Obsługuje dostawców map Google Maps i ESRI, automatycznie wybierając najlepszą usługę dla Twojej lokalizacji. Najedź kursorem na znaczniki, aby wyświetlić podgląd miniatur zdjęć.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Dziennik debugowania

W razie wystąpienia problemów przejrzyj dziennik w poszukiwaniu wpisów debugowania. Skopiuj/pobierz dziennik i wyślij go do [MAPIR Pomoc techniczna](https://www.mapir.camera/community/contact) w celu uzyskania pomocy.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Logowanie użytkownika](chloros+-login.md)

Pasek boczny logowania użytkownika umożliwia zalogowanie się na konto Chloros+ w celu odblokowania zaawansowanych funkcji. Można również wyświetlić aktualną wersję aplikacji, a także dostosować język wyświetlanego tekstu w interfejsie graficznym Chloros i CLI.
