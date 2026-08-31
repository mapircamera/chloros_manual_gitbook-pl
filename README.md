---
metaLinks: {}
---

# Pierwsze kroki

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

to aplikacja firmy [MAPIR

](https://www.mapir.camera) służąca do przetwarzania obrazów wielospektralnych, sterowania na żywo sprzętemMAPIR

oraz rejestrowania danych z czujników. AplikacjaChloros

w wersji 1.2.0 obsługuje całą rodzinę produktówMAPIR

:

* **KamerySurvey3** — przetwarzają zdjęcia w formacie RAW+JPG na skalibrowane mapy odbicia i wskaźników wegetacyjnych. Zobacz [Obsługiwane kamery](supported-cameras.md).
* **Kamery LATTICE** — podłączaj moduły kamer wielospektralnych GigE w trybie na żywo, pojedynczo lub jako zsynchronizowane układy wielokamerowe: podgląd, rejestracja i przetwarzanie na skalibrowane produkty promieniowania i odbicia. Zobacz [sekcję LATTICE](lattice/README.md).
* **Czujniki światła DAQ** — czujniki spektralne DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet): skalibrowane widma na żywo, nagrania `.daq` oraz oświetlenie padające w dół do przetwarzania współczynnika odbicia. Zobacz [sekcję DAQ](daq/README.md).

{% hint style="success" %}
**Co nowego wChloros

1.2.0**: sterowanie kamerami i matrycami LATTICE w czasie rzeczywistym, integracja czujników światła DAQ, tryby przechwytywania i rejestratory, pełny radiometryczny łańcuch przetwarzania LATTICE, automatyzacja projektów z poziomuCLI

/SDK

oraz wiele więcej. Zapoznaj się z listą nowości poniżej, a dziennik zmian znajdziesz po [pobraniu](download.md).
{% endhint %}

{% hint style="info" %}
**Korzystasz zChloros

z asystentem AI?** Ta instrukcja została stworzona właśnie z myślą o tym. Skieruj swojego asystenta na:

* `https://mapir.gitbook.io/chloros/llms.txt` — indeks każdej strony w formacie nadającym się do odczytu maszynowego.
* Dowolną stronę w formacie surowego Markdown — dodaj `.md` do jej adresuURL

(np. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI

Dokumentacja](reference/cli-reference.md) oraz [SDK

Dokumentacja](reference/sdk-reference.md) — kompletne strony referencyjne zawierające dokładne wartości, napisane z myślą o wykorzystaniu przez duże modele językowe (LLM).

Przykładowe polecenie: *„Przeczytaj https://mapir.gitbook.io/chloros/reference/cli-reference.md,, a następnie napisz skrypt, który zaloguje się i przetworzy folder ~/flights/flight_001 na pliki GeoTIFF w formacie reflectance orazNDVI

.”*

Pełny przewodnik: [Korzystanie zChloros

z asystentami AI](ai-assistants.md).
{% endhint %}

***

## Co nowego wChloros

1.2.0

* **Sterowanie kamerą na żywo — nowa zakładka „Kamery”.** Podłączaj kamery LATTICE pojedynczo lub jako zsynchronizowane układy wielokamerowe (synchronizacja czasu PTP, wyzwalanie przechwytywania przez sprzęt), z nakładkami podglądu na żywo, histogramami dla poszczególnych pasm, inteligentną automatyczną ekspozycją, kalkulatorem indeksu na żywo oraz aktualizacjami oprogramowania sprzętowego kamer w aplikacji.
* **Czujniki światła — nowa zakładka „Light Sensors”.** Podłącz czujniki DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet); przeglądaj skalibrowane widma na żywo (W/m²/nm), zapisuj pliki `.daq` w swoim projekcie, wybieraj profile korekcji kapilarnej oraz aktualizuj oprogramowanie sprzętowe DAQ-E przez sieć.
* **Tryby przechwytywania i rejestratory.** Przechwytywanie pojedyncze / ciągłe / interwałowe oraz tryb „Najszybsze przechwytywanie” wyłącznie dla danych surowych; wybór dla każdego projektu, które kamery i typy eksportu generuje opcja „Przechwyć wszystko”; rejestratory macierzowe do wideo indeksowego klasy monitorującej oraz serii danych surowych klasy analitycznej z możliwością tworzenia kompilacji wideo w trybie offline.
* **Potok przetwarzania LATTICE.** Importuj foldery przechwytywania LATTICE i przetwarzaj każdą surową klatkę na produkty: bez efektu Bayera, podgląd, natężenie promieniowania float32 (W/m²/sr/nm) oraz współczynnik odbicia, z przełącznikami dla poszczególnych produktów. Współczynnik odbicia może pochodzić z celu kalibracyjnego znajdującego się w klatce lub z danych DAQ dotyczących promieniowania padającego; do eksportowanych danych stosowane jest wyrównanie matrycy; brakująca kalibracja fabryczna jest pobierana automatycznie na podstawie numeru seryjnego kamery.
* **Projekty zapamiętują sprzęt.** Podłączone kamery i czujniki światła są zapisywane wraz z projektem (`cameras.json` / `sensors.json`) i ponownie łączą się z zachowanymi ustawieniami po ponownym otwarciu projektu. Zobacz [Interfejs graficzny: Projekty](projects.md).
* **Ulepszenia przeglądarki obrazów.** Odczyt pikseli/indeksu kursora z prawidłowym skalowaniem odbicia światła dla poszczególnych plików, histogramy warstw, suwak binningu GSD, tryby siatki „Per Trigger” / „Per Camera”, widoki produktów LATTICE oraz eksport indeksu/LUT do piaskownicy na dysk.
* **Znacznie rozbudowane funkcjeCLI

iSDK

.** Nowe rodziny poleceń `lattice`, `daq pool-*`, `project` oraz `time-sync`; nowe opcje `process` (`--input-level`, przełączniki dla poszczególnych produktów, `--reflectance-source`, flagi wyrównania tablic);SDK

uchwyty smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`), które automatycznie uruchamiają backend; automatyzacja `open_project()`; pakiet „SDK

” jest dołączony do instalatorów i opublikowany na PyPI jako `chloros-sdk`.
* **Rzetelna semantyka błędów.** Uruchomienie `chloros-cli process`, które zażądało produktów, ale nie zapisało żadnego, teraz kończy się wyraźnym błędem i zwraca wynik niezerowy; udane uruchomienia zgłaszają, ile produktów obrazowych zostało zapisanych.
* **Nowy układ wyników.** Produkty trafiają do folderów `<project>/<camera>/<format>/<Product>_Images/` i zachowują nazwę pliku źródłowego — to folder, a nie rozszerzenie nazwy pliku, identyfikuje produkt. Zobacz [Formaty obrazów wyjściowych](output-image-formats.md).
* **Więcej danych wejściowych, planów i języków.** Obsługa danych wejściowych `.dng`; pełna obsługa wszystkich 38 języków interfejsu; limity sprzętowe dla poszczególnych planów z bezpłatnym (bez logowania) dostępem do maksymalnie 4 kamer i 2 czujników światła.
* **Niezawodność.** Funkcja „Zatrzymaj przetwarzanie” kończy działanie w sposób uporządkowany, wyświetlając rzetelne podsumowanie przebiegu; w projektach z wieloma kamerami eksportowane są dane z każdej kamery, a aktualizacje instalatora nie powodują już wylogowania użytkownika.***

Chloros

jest dostępny w 3 wersjach aplikacji:

##Chloros

: Aplikacja graficzna na komputery stacjonarne

Samodzielne, oddzielne okno z wszystkimi funkcjami, w tym zakładkami „Kamery na żywo” i „Czujniki światła”. _Tylko dla systemu Windows._

## [Chloros

CLI

: Interfejs wiersza poleceń](CLI.md)

Przetwarzanie wsadowe z wiersza poleceń oraz polecenia na żywo `lattice`, `daq pool-*`, `project` i `time-sync`. Idealne rozwiązanie do automatyzacji, tworzenia skryptów i pracy w trybie bezinterfejsowym. Dostępne na **Windows

,Linux

amd64 orazLinux

arm64 (NVIDIA Jetson)**. _Aby uzyskać dostęp do CLI, wymagany jest płatny planChloros

+._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Programowy interfejsPython

do automatyzacji i niestandardowych przepływów pracy: przetwarzanie pełnego potoku, sesje z kamerami na żywo/matrycami, sesje z czujnikami DAQ oraz automatyzacja zapisanych projektów. Instalowany wraz z pakietem desktop/CLI

, a także opublikowany jako `pip install chloros-sdk`. _Dostęp do API wymaga płatnego planuChloros

+._

***

## Obsługiwane platformy

| Platforma | Interfejs graficzny |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Tak | Tak | Tak |
| **Linux

amd64 (x86_64)** | Nie | Tak | Tak |
| **Linux

arm64 (NVIDIA Jetson)** | Nie | Tak | Tak |

Instrukcje dotyczące instalacji w środowiskuLinux

można znaleźć w sekcji [Linux

i przetwarzanie brzegowe](linux/linux-overview.md).

***

## Rozpocznij pracę w trzech krokach

1. **Instalacja** — pobierz i uruchom instalator dla swojej platformy. Zobacz [Pobierz](download.md).
2. **Zaloguj się (opcjonalnie w przypadku interfejsu graficznego)** — interfejs graficzny przetwarza obrazy za darmo bez konieczności posiadania konta. [Chloros

+ zalogowanie się](chloros+-login.md) odblokowuje przetwarzanie równoległe, przyspieszenie GPU, wyższe limity urządzeń oraz dostęp doCLI

/SDK

.
3. **Utwórz swój pierwszy projekt** — otwórz stronęChloros

, utwórz [nowy projekt](projects.md), [dodaj swoje obrazy](processing-images-gui/adding-files-to-a-project.md) i [rozpocznij przetwarzanie](processing-images-gui/starting-the-processing.md). Aby zamiast tego sterować sprzętem na żywo, otwórz zakładkę „Kamery” lub „Czujniki światła” — zobacz [GUI: Nawigacja](navigation.md).

***

##Chloros

+

Chociaż program „Chloros

” jest bezpłatny w przypadku większości zadań, może się okazać, że potrzebujesz więcej. Właśnie wtedy przydatna może być płatna licencja na program „Chloros

+”. Dzięki licencji „Chloros

+” możesz odblokować nowe funkcje, takie jak:

* **Przetwarzanie wielowątkowe**: znacznie przyspiesza przetwarzanie obrazów w większych projektach poprzez jednoczesne przetwarzanie obrazów w ramach potoku.
* **Przyspieszenie za pomocą procesora graficznego (CUDA)**: wykorzystaj dzisiejsze opcje większej pamięci procesora graficznego, aby jeszcze bardziej przyspieszyć działanie potoku przetwarzania obrazów. Aby uzyskać najlepsze wyniki, zalecamy co najmniej 4 GB pamięci VRAM.
* **Dostęp doChloros

+**[**CLI**](CLI.md): uruchom polecenie `Chloros

+` z wiersza poleceń, aby zautomatyzować proces i zintegrować go z własnym oprogramowaniem. Dostępne we wszystkich płatnych planach; egzekwowane po stronie serwera.
* **Chloros

+**[**API**](api-python-sdk.md) **Dostęp:** uruchom polecenieChloros

+ zPython

w celu sterowania programowego, co umożliwia płynną integrację z procesami badawczymi, przepływami pracy związanymi z analizą danych oraz niestandardowymi aplikacjami. Dostępne na każdym płatnym planie; egzekwowane po stronie serwera.
* **Wyższe limity sprzętowe**: podłącz jednocześnie więcej kamer i czujników światła. Bez logowania interfejs graficzny pozwala podłączyć maksymalnie 4 kamery i 2 czujniki światła DAQ; płatne plany zwiększają oba limity:

| Plan | Kamery | Czujniki światła DAQ |
| --- | --- | --- |
| Iron (bezpłatny, bez logowania) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Korzystanie z wielu urządzeń**: każda licencja typu „Chloros

” pozwala na zarejestrowanie co najmniej 2 urządzeń. Do zarządzania zarejestrowanymi urządzeniami użyj konta w chmurzeMAPIR

. Dodaj obsługę kolejnych urządzeń, uaktualniając licencję typu „Chloros

”.
* **Zaawansowana metoda debayeringu uwzględniająca teksturę:** wysokiej jakości debayering uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na AI/ML, który eliminuje niemal cały szum wynikający z debayeringu.
* **Niestandardowe wzory indeksów wielospektralnych:** wprowadzaj niestandardowe indeksy wielospektralne w kalkulatorach rastrowychChloros

, zarówno do przetwarzania, jak i do środowiska testowego przeglądania obrazów.
* **TechnologiaLinux

oraz przetwarzanie brzegowe:** uruchamiaj oprogramowanieChloros

na platformachLinux

x86_64 i ARM64, w tym na NVIDIA Jetson, do przetwarzania w terenie i na urządzeniach brzegowych. Zobacz [Linux

Przegląd](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Ceny i rejestracja</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
