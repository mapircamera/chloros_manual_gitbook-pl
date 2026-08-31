# Interfejs graficzny: Projekty

Chloros umożliwia tworzenie projektów, które można ponownie otworzyć w przyszłości. Projekt to zwykły folder (znajdujący się w folderze projektów) zawierający:

* `project.json` — ustawienia projektu, listę plików i preferencje wyświetlania
* `cameras.json` — kamery i matryce podłączone w trakcie otwarcia projektu wraz z ich ustawieniami
* `sensors.json` — czujniki światła DAQ podłączone podczas otwarcia projektu, wraz z powiązaniami kamera↔czujnik
* Twoje zrzuty, nagrania `.daq` oraz foldery z przetworzonymi wynikami

Nie ma zastrzeżonego formatu plików projektów — projektem jest folder wraz z zawartymi w nim plikami JSON, co ułatwia również kopiowanie, archiwizowanie i przenoszenie projektów z [CLI](CLI.md) lub [Python SDK](api-python-sdk.md).

## Nowy projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Wybierz opcję „Nowy projekt” z menu głównego i wprowadź unikalną nazwę projektu.

Jeśli zapisałeś jakieś szablony projektów, pod polem nazwy pojawi się lista rozwijana **Wybierz szablon** — wybranie jednego z nich spowoduje rozpoczęcie nowego projektu na podstawie ustawień tego szablonu. Szablony zapisuje się w [Ustawieniach projektu](project-settings/project-settings.md): wpisz nazwę w polu „Nazwa szablonu projektu” i kliknij ikonę zapisu.

## Otwórz projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>W oknie „Otwórz projekt” wyświetlana jest lista wszystkich projektów znajdujących się w folderze projektów, a na dole znajduje się opcja <strong>„Otwórz folder projektów”</strong></p></figcaption></figure>Wybierz opcję „Otwórz projekt”, aby wyświetlić listę istniejących projektów w folderze projektów. Jeśli nie ma żadnych projektów, dodatkowe menu boczne nie zostanie otwarte. Na powyższym zdjęciu widać kilka projektów utworzonych za pomocą interfejsu graficznego (t1, t2, t3). Projekty o nazwach w formacie DATA\_CZAS zostały utworzone przez CLI przy użyciu domyślnego schematu nazewnictwa projektów. Kliknięcie dowolnej nazwy projektu spowoduje jego otwarcie.

Kliknięcie przycisku „Otwórz folder projektu” powoduje otwarcie eksploratora plików komputera w lokalizacji projektu. Ścieżkę do projektu można dostosować w [Ustawieniach projektu](project-settings/project-settings.md).

Jeśli którekolwiek z plików obrazów źródłowych projektu zostały przeniesione lub usunięte od czasu ostatniego otwarcia, program Chloros wyświetla okno dialogowe z dokładną listą brakujących plików zamiast otwierać pustą siatkę.

## Duplikowanie projektu

Opcja dostępna po otwarciu projektu. Wybierz „Duplikuj projekt”, aby skopiować bieżący projekt pod nową nazwą — program Chloros zasugeruje następną dostępną nazwę (np. „MójProjekt (2)”) — a duplikat zostanie natychmiast otwarty.

## Dodaj pliki

Po otwarciu projektu wybierz opcję „Dodaj pliki” z menu głównego, aby dodać pojedyncze pliki obrazów do bieżącego projektu. Funkcja ta odzwierciedla funkcję dodawania w przeglądarce plików, ale dla wygody jest dostępna bezpośrednio z menu głównego.

## Dodaj folder

Po otwarciu projektu wybierz opcję „Dodaj folder” z menu głównego, aby dodać foldery z obrazami do bieżącego projektu. Możesz wybrać wiele folderów za jednym razem. Duplikaty plików są ignorowane.

## Rozpocznij / Zatrzymaj przetwarzanie

Po dodaniu plików do projektu w menu głównym pojawia się opcja „Rozpocznij przetwarzanie”. Jest to ta sama czynność, co kliknięcie przycisku Odtwórz/Rozpocznij w górnym nagłówku. Podczas przetwarzania pozycja menu zmienia się na „Zatrzymaj przetwarzanie”, co pozwala zatrzymać proces.

## Podłącz kamerę / Podłącz czujnik światła

W dolnej części menu głównego znajdują się dwa skróty sprzętowe, dostępne niezależnie od tego, czy projekt jest otwarty, czy nie:

* **Podłącz kamerę** — otwiera [kartę „Kamery”](lattice/) w celu podłączenia kamery lub matrycy LATTICE.
* **Podłącz czujnik światła** — otwiera [kartę „Czujniki światła”](daq/), umożliwiającą podłączenie czujnika światła DAQ.

Podłączenie sprzętu przy otwartym projekcie powoduje zapisanie go w projekcie (patrz poniżej). Bez projektu połączenia są dostępne wyłącznie w ramach bieżącej sesji.

{% hint style="info" %}
Pozycje menu Dodaj pliki, Dodaj folder oraz Rozpocznij/Zatrzymaj przetwarzanie są widoczne lub aktywne tylko wtedy, gdy projekt jest otwarty, a pliki zostały dodane. Zapewniają one szybki dostęp do czynności dostępnych również za pośrednictwem paska bocznego Przeglądarki plików oraz przycisków w nagłówku.
{% endhint %}

## Projekty zapamiętują podłączony sprzęt

Nowość w wersji 1.2.0: projekt zachowuje informacje o sprzęcie podłączonym w czasie, gdy jest otwarty. Kamery i matryce (wraz z ustawieniami poszczególnych kamer, nazwami, kolorami i układem siatki) są zapisywane w pliku `cameras.json`, a czujniki światła (wraz z nazwami, kolorami i powiązaniami z kamerami) w pliku `sensors.json` — automatycznie, w trakcie pracy.

Gdy **ponownie otworzysz** projekt, plik Chloros nie nawiązuje od razu połączenia z żadnym sprzętem. Każda połowa ponownie łączy się przy pierwszym przejściu do zakładki, do której należy:

* Otwarcie zakładki **Kamery** powoduje ponowne podłączenie zapisanych kamer i macierzy oraz ponowne zastosowanie ich zapisanych ustawień.
* Otwarcie zakładki **Czujniki światła** powoduje ponowne podłączenie zapisanych czujników DAQ.

W ten sposób otwarcie projektu wyłącznie w celu przeglądania lub eksportowania obrazów nigdy nie powoduje uruchomienia transmisji strumieniowej z kamer. Jeśli podczas otwierania zakładki nie można znaleźć zapisanego urządzenia, pojawi się okno dialogowe informujące, które urządzenia są niedostępne, aby można było je ponownie podłączyć lub usunąć.

## Nagrania DAQ i pliki .daq w projekcie

* Nagrania `.daq` wykonane podczas otwartego projektu (z zakładki **Czujniki światła**lub podczas przechwytywania) są**automatycznie dodawane do projektu**.
* Zaimportowane pliki `.daq` oraz wszystkie nagrania projektu są wymienione w sekcji **Czujnik światła DAQ** w [Ustawieniach projektu](project-settings/project-settings.md), a każdy z nich posiada swój profil korekcji nasłonecznienia.
* Podczas przetwarzania pliki `.daq` projektu dostarczają dane o oświetleniu padającym dla produktów odbicia — zobacz [Formaty obrazów wyjściowych](output-image-formats.md).

## Uruchamianie zapisanego projektu bez interfejsu graficznego

Zapisany projekt można uruchomić bez interfejsu graficznego:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` działa na ścieżce do folderu projektu — zobacz [Podręcznik CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` zwraca uchwyt projektu; `connect_all()` uruchamia wszystkie zapisane kamery i czujniki wraz z ich zapisanymi ustawieniami — zobacz [Opis funkcji SDK](reference/sdk-reference.md).
