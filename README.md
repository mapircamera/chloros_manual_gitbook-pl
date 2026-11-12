---
metaLinks: {}
---

# Pierwsze kroki

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros to aplikacja firmy [MAPIR](https://www.mapir.camera) służąca do przetwarzania obrazów wielospektralnych, sterowania sprzętem MAPIR w trybie rzeczywistym oraz rejestrowania danych z czujników. Chloros 1.3.0 obsługuje całą rodzinę produktów MAPIR:

* **Kamery Survey3** — przetwarzają zdjęcia w formacie RAW+JPG na skalibrowane mapy odbicia i wskaźników wegetacyjnych. Zobacz [Obsługiwane kamery](supported-cameras.md).
* **Kamery LATTICE** — podłączaj moduły kamer wielospektralnych GigE w czasie rzeczywistym, pojedynczo lub jako zsynchronizowane układy wielokamerowe: podglądaj, rejestruj i przetwarzaj na skalibrowane produkty promieniowania i odbicia. Zobacz [sekcję poświęconą LATTICE](lattice/README.md).
* **Czujniki światła DAQ** — czujniki spektralne DAQ-U (USB), DAQ-M (Bluetooth) oraz DAQ-E i DAQ-E-S (Ethernet): skalibrowane widma na żywo, `.daq` oraz oświetlenie padające w dół do przetwarzania współczynnika odbicia. Zobacz [sekcję DAQ](daq/README.md).

{% hint style="success" %}
**Nowości w Chloros 1.3.0**: Asystenci AI mogą sterować Chloros za pośrednictwem `chloros-mcp`, a CLI, SDK i `chloros-mcp` oferują te same funkcje. Ulepszono produkty z indeksu LATTICE, macierze pełnych czujników działają szybciej i obsługują dowolną sieć, a backend działa nieprzerwanie między poleceniami. Zapoznaj się z poniższą listą nowości i [pobierz](download.md) instalatory.
{% endhint %}

{% hint style="info" %}
**Korzystasz z Chloros a z asystentem AI?** Ta instrukcja została stworzona właśnie z myślą o tym. Skieruj swojego asystenta na:

* `https://mapir.gitbook.io/chloros/llms.txt` — indeks każdej strony w formacie nadającym się do odczytu maszynowego.
* Dowolną stronę w surowym formacie Markdown — dodaj `.md` do jej adresu URL (np. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Podręcznik CLI](reference/cli-reference.md) oraz [Podręcznik SDK](reference/sdk-reference.md) — kompletne strony referencyjne zawierające dokładne wartości, napisane z myślą o modelach LLM.

Przykładowe polecenie: *„Przeczytaj https://mapir.gitbook.io/chloros/reference/cli-reference.md,, a następnie napisz skrypt, który zaloguje się i przetworzy folder ~/flights/flight_001 na pliki GeoTIFF zawierające współczynniki odbicia i NDVI.”*

Można też uruchomić bezpośrednio Chloros: `chloros-mcp`, serwer MCP zainstalowany wraz z SDK, współpracuje z Claude Desktop, Claude Code, Cursor i innymi klientami MCP.

Pełny przewodnik: [Korzystanie z Chloros a z asystentami AI](ai-assistants.md).
{% endhint %}

***

## Co nowego w Chloros 1.3.0

* **Asystenci AI mogą sterować Chloros em.** `chloros-mcp`, serwer MCP zainstalowany wraz z Python SDK, umożliwia aplikacjom Claude Desktop, Claude Code, Cursor i innym klientom MCP przetwarzanie obrazów oraz sterowanie kamerami LATTICE, matrycami i czujnikami światła DAQ. Zobacz [Korzystanie z Chloros a z asystentami AI](ai-assistants.md).
* **CLI, SDK oraz `chloros-mcp` oferują te same funkcje.** Ze względów bezpieczeństwa w CLI pozostają jedynie logowanie, konfiguracja DAQ-E oraz bezpośrednie serwery DAQ. Niektóre skrypty z wersji 1.2.0 wymagają zmian: zobacz sekcję **Zmiany w wersji 1.3.0** na początku [Podręcznika CLI](reference/cli-reference.md) oraz [Podręcznika SDK](reference/sdk-reference.md).
* **Ulepszone produkty indeksowe LATTICE.** Wskaźnik NDVI oraz inne produkty indeksowe eksportowane z danych LATTICE zostały ulepszone we wszystkich trybach przetwarzania.
* **Opcje instalatora dla systemu Windows.**W sekcji**Ustawienia systemowe**instalator oferuje opcje**Włącz długie ścieżki w systemie Windows**oraz**Wyklucz backend Chloros ze skanowania przez Microsoft Defender**; obie są domyślnie zaznaczone. Pierwsza opcja pozwala na tworzenie folderów projektów o nazwach dłuższych niż 260 znaków; druga zapobiega spowolnieniu przechwytywania obrazu przez kamerę w wyniku skanowania w czasie rzeczywistym (około 30% szybkości przechwytywania przy wysokich częstotliwościach). Instalator dodaje teraz również folder CLI do `PATH` tylko raz i usuwa zduplikowane wpisy pozostawione przez wcześniejsze wersje instalatora. Zobacz [Pobierz](download.md).
* **Głębokie foldery projektowe są odrzucane z góry.** Przy wyłączonych długich ścieżkach w systemie Windows (ustawienie domyślne systemu Windows) folder projektu, którego ścieżka przekracza limit 260 znaków, jest odrzucany przed uruchomieniem jakichkolwiek operacji, przy czym wyświetlana jest informacja o najdłuższym folderze projektu, który mieści się w limicie, oraz o tym, jak włączyć długie ścieżki. Wcześniej takie uruchomienie nie tworzyło żadnych produktów. Przy włączonych długich ścieżkach produkty w bardzo głębokich folderach zachowują teraz swoje metadane (tagi GPS i XMP).
* **Maska przycinania.** Gdy rozdzielanie pasm produktu promieniowania lub odbicia LATTICE daje ujemną wartość pasma, wartość ta jest przycinana do zera, a maska na piksel w folderze `clamp_masks/` znajdującym się obok folderów produktów rejestruje, które piksele i pasma zostały objęte przycinaniem. Produkt nadaje nazwę swojej masce w tagu `Chloros:UnmixClampMask`. Widoki na żywo są teraz przycinane dokładnie tak samo jak podczas eksportu. Zobacz [Formaty obrazów wyjściowych](output-image-formats.md).
* **Produkty opisują się same.** Produkty indeksowe posiadają własne tagi przetwarzania (`Chloros:ProcessingLevel=index`, wyrażenie indeksowe oraz informację, czy zostały obliczone na podstawie promieniowania czy odbicia) zamiast tagów źródłowych plików surowych. Produkty Survey3 posiadają tagi przetwarzania Chloros we wszystkich trybach przetwarzania. Pliki surowe i po usunięciu matrycy LATTICE zachowują natywną orientację czujnika i zapisują ją w pliku XMP; przeglądarka obrazów obraca je wyłącznie w celu wyświetlenia.
* **Pakiety dla systemu Linux są dostarczane w postaci nierozpakowanej.** Moduł zaplecza i interfejs CLI są instalowane jako foldery w katalogu `/usr/lib/chloros/`, dzięki czemu nie rozpakowują się już przy każdym uruchomieniu. Na architekturze x86_64 backend nie potrzebuje już około 100 sekund na uruchomienie, a rozmiar katalogu `.deb` zmniejszył się o połowę w stosunku do poprzedniej wersji. Dołączone narzędzie ExifTool działa teraz na czystej instalacji systemu Ubuntu. Pakiet `.deb` instaluje teraz również Python owe SDK dla systemu `python3` (zainstaluj go za pomocą `sudo apt install ./chloros_1.3.0_<arch>.deb`), zwiększa rozmiar buforów odbiorczych gniazd jądra, których potrzebują strumienie z kamery, oraz dodaje usługę backendu działającą pod uprawnieniami użytkownika. Pakiet Jetson, `chloros_1.3.0_arm64_jp6.deb`, jest skompilowany dla JetPack 6 i przetwarza dane na procesorze graficznym Jetsona przy użyciu kompilacji PyTorch dla CUDA w ramach JetPack. W systemie Linux każdy wyeksportowany obraz trafia teraz na dysk tylko raz (jest zapisywany i oznaczany w pamięci, a następnie kopiowany na miejsce): na Jetsonie, gdzie tempo pracy wyznacza dysk, przebieg LATTICE obejmujący 288 obrazów zajmuje 14 minut zamiast 24. Zobacz [Instalacja w systemie Linux](linux/linux-installation.md).
* **Kamery LATTICE typu OCN odczytują dane dokładniej w świetle słonecznym.** Po rozdzieleniu pasm wartości promieniowania, odbicia i wskaźników są natychmiast korygowane, dzięki czemu szary cel wykazuje w słońcu wartość NDVI zbliżoną do neutralnej. Kamery typu RGN pozostają bez zmian.
* **Zgłaszanie błędów w aplikacji.** Ikona błędu u góry zakładki „Debug Log” (Dziennik debugowania) wysyła do serwisu MAPIR opis, maksymalnie 10 zdjęć, logi oraz szczegóły systemowe. Zobacz [GUI: Nawigacja](navigation.md).
* **Macierze LATTICE.**Macierze domyślnie zachowują pełną rozdzielczość czujnika, a skompresowany transport 12-bitowy (`--packed`, domyślnie `auto`) zwiększa liczbę klatek na sekundę przy pełnej rozdzielczości czujnika o około jedną trzecią; macierz działa w każdej sieci z częstotliwością klatek, na jaką pozwala dane łącze (wolne łącze obniża częstotliwość klatek zamiast odrzucać macierz lub zmniejszać liczbę klatek); macierz, której kamery zostały wyłączone i ponownie włączone, odzyskuje działanie samodzielnie, przy ponownym włączeniu PTP; automatyczna ekspozycja nie obniża już tej częstotliwości przy słabym oświetleniu (wypełnia okres, a następnie zwiększa wzmocnienie); Tryb ciągłego przechwytywania nadąża za szybkimi macierzami; kontrolka**Highlights** dla kamer RGB; a funkcje `ArraySession` i `grab()` w SDK `frame_stream()` oraz `continuous()`. Zobacz [sekcję LATTICE](lattice/README.md).
* **Bardziej stabilna automatyczna ekspozycja.** Ekspozycja utrzymuje się na stałym poziomie przy stałym oświetleniu i stabilizuje się w ciągu kilku sekund po zmianie; niewielkie jasne obszary nie przyciemniają już sceny; przy ręcznej ekspozycji wzmocnienie pozostaje w trybie automatycznym, a kamera, która przestaje reagować w jasnym słońcu, utrzymuje poziom dostosowany do oświetlenia odczytanego przez czujnik światła lub — w przypadku braku podłączonego czujnika — na podstawie własnych klatek kamery. Zobacz [Ustawienia kamery](lattice/camera-settings.md).
* **Czujniki światła.** **Obsługa DAQ-E-S:** jego stały dyfuzor nie wymaga korekcji nasadki, a każdy odczyt zawiera informacje o pochyleniu, obrocie i nachyleniu urządzenia. Nagranie wskazuje, czy wartość maksymalna została wybrana, czy założona, a podczas przetwarzania pojawia się ostrzeżenie w przypadku wartości założonej: modele DAQ-U, DAQ-M i DAQ-E są dostarczane bez wartości maksymalnej, więc należy podać nazwę zamontowanej. Aktywne zapisy `.daq` zachowują się po zaniku zasilania (należy zachować plik `.daq-journal` wraz z jego `.daq`), a urządzenie DAQ-E, które przestało przesyłać dane, zostanie ponownie podłączone z zaznaczeniem luki w jego zapisie. Projekt zawierający wyłącznie zapisy z czujników światła wyświetla ich listę i przetwarza je za pomocą aplikacji oraz interfejsu CLI, a zapis wykonany za pomocą `chloros_scripts` jest wyświetlany jako pochodzący z urządzenia DAQ, które go wykonało, z możliwością ustawienia limitu. Każde przechwycenie zapisuje odczyt z urządzenia DAQ, które zostało użyte. Zobacz [sekcję DAQ](daq/README.md).
* **Przetwarzanie.**Przetwarzanie klatek LATTICE rozpoczyna się podczas wykrywania celu. Odniesienia do celu i urządzenia DAQ są ustalane dla każdego pasma. Przeniesiony projekt ponownie łączy swoje pliki. Eksporty z 8-bitowego przechwycenia są w formacie 8-bitowym, w nowym formacie**TIFF (8-bit)**. Przebieg programu Survey3 bez celu kalibracyjnego jest zapisywany jako produkt rezerwowy (`Vignette_Corrected_Images` lub `Sensor_Response_Images`), a nie jako obraz odbicia. Tagi DLS dla poszczególnych pasm obrazu odbicia zawsze należą do jego własnej klatki. Opcja „Texture Aware” jest odrzucana w przypadku ujęć LATTICE, które zawsze przechodzą standardową deinterpolację.
* **CLI i SDK.** Względne ścieżki wyjściowe są teraz zapisywane w folderze, z którego uruchomiono polecenie lub skrypt (wcześniej zainstalowany backend próbował je rozpoznać w swoim własnym folderze instalacyjnym, co kończyło się niepowodzeniem). Moduł zaplecza działa w tle między poleceniami i skryptami, a po 30 minutach bezczynności zatrzymuje się samoczynnie; zarządzają tym `chloros-cli backend status` i `backend stop`. Polecenia przechwytywania podają nazwy wszystkich plików, których nie udało się zapisać, i kończą działanie z kodem 1, a powtarzające się operacje przechwytywania zatrzymują się w przypadku zapełnienia dysku. Pomyślne uruchomienie `process` generuje komunikat `Image products written: N (plus M clamp masks)`. W systemie Windows `ChlorosLocal` z SDK / `process_folder` z zestawu SDK wykrywa zainstalowany moduł zaplecza (i korzysta z tego, który już działa), a `read_image_audit_tags` działa na podstawie zwykłego pliku `pip install`.

***

## Co nowego w Chloros 1.2.0

* **Sterowanie kamerami na żywo — nowa zakładka „Kamery”.** Podłączaj kamery LATTICE pojedynczo lub jako zsynchronizowane układy wielokamerowe (synchronizacja czasu PTP, przechwytywanie wyzwalane sprzętowo), z nakładkami podglądu na żywo, histogramami dla poszczególnych pasm, inteligentną automatyczną ekspozycją, kalkulatorem indeksu na żywo oraz aktualizacjami oprogramowania sprzętowego kamer w aplikacji.
* **Czujniki światła — nowa zakładka „Czujniki światła”.** Podłączaj czujniki DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet); przeglądaj skalibrowane widma na żywo (W/m²/nm), zapisuj pliki `.daq` w swoim projekcie, wybieraj profile korekcji kapilarnej oraz aktualizuj oprogramowanie sprzętowe DAQ-E przez sieć.
* **Tryby przechwytywania i rejestratory.** Przechwytywanie pojedyncze / ciągłe / interwałowe oraz tryb „Fastest Capture” przeznaczony wyłącznie dla danych surowych; wybór dla każdego projektu, które kamery i typy eksportu generuje opcja „Capture All”; rejestratory macierzowe do wideo indeksowego klasy monitorującej oraz serii danych surowych klasy analitycznej z możliwością tworzenia kompilacji wideo w trybie offline.
* **Potok przetwarzania LATTICE.** Importuj foldery przechwytywania LATTICE i przetwarzaj każdą surową klatkę na produkty: po usunięciu matrycy Bayer, podgląd, natężenie promieniowania float32 (W/m²/sr/nm) oraz współczynnik odbicia, z przełącznikami dla poszczególnych produktów. Współczynnik odbicia może pochodzić z celu kalibracyjnego w klatce lub z danych DAQ dotyczących promieniowania padającego; do eksportowanych danych stosowane jest wyrównanie matrycy; brakująca kalibracja fabryczna jest pobierana automatycznie na podstawie numeru seryjnego kamery.
* **Projekty zapamiętują sprzęt.** Podłączone kamery i czujniki światła są zapisywane wraz z projektem (`cameras.json` / `sensors.json`) i ponownie łączą się z zapisanymi ustawieniami po ponownym otwarciu projektu. Zobacz [GUI: Projekty](projects.md).
* **Ulepszenia przeglądarki obrazów.** Odczyt pikseli/indeksu kursora z prawidłowym skalowaniem odbicia światła dla poszczególnych plików, histogramy warstw, suwak binningu GSD, tryby siatki „na wyzwolenie” / „na kamerę”, widoki produktów LATTICE oraz eksport indeksu/LUT do piaskownicy na dysk.
* **Znacznie rozbudowano interfejs CLI i SDK.** Nowe rodziny poleceń `lattice`, `daq pool-*`, `project` oraz `time-sync`; nowe opcje `process` (`--input-level`, przełączniki dla poszczególnych produktów, `--reflectance-source`, flagi wyrównania tablic); uchwyty inteligentnego połączenia SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`), które automatycznie uruchamiają backend; Automatyzacja `open_project()`; pakiet SDK jest dołączony do instalatorów i opublikowany w PyPI jako `chloros-sdk`.
* **Rzetelna semantyka błędów.** Uruchomienie `chloros-cli process`, które zażądało produktów, ale nie zapisało żadnego, teraz kończy się wyraźnym błędem i zwraca wynik niezerowy; udane uruchomienia zgłaszają, ile produktów obrazowych zostało zapisanych.
* **Nowy układ wyników.** Produkty trafiają do folderów `<project>/<camera>/<format>/<Product>_Images/` i zachowują nazwę pliku źródłowego — to folder, a nie rozszerzenie nazwy pliku, identyfikuje produkt. Zobacz [Formaty obrazów wyjściowych](output-image-formats.md).
* **Więcej danych wejściowych, planów i języków.** Obsługa danych wejściowych `.dng`; pełna obsługa wszystkich 38 języków interfejsu; limity sprzętowe dla poszczególnych planów z bezpłatnym (bez logowania) dostępem do maksymalnie 4 kamer i 2 czujników światła.
* **Niezawodność.** Funkcja „Zatrzymaj przetwarzanie” kończy działanie w sposób uporządkowany, wyświetlając rzetelne podsumowanie przebiegu; projekty z wieloma kamerami eksportują dane z każdej kamery, a aktualizacje instalatora nie powodują już wylogowania użytkownika.

***

Aplikacja Chloros jest dostępna w 3 wersjach:

## Chloros: aplikacja graficzna na komputery stacjonarne

Samodzielne, oddzielne okno z wszystkimi funkcjami, w tym zakładkami „Kamery na żywo” i „Czujniki światła”. _Tylko dla systemu Windows._

## [Chloros CLI: interfejs wiersza poleceń](CLI.md)

Przetwarzanie wsadowe z wiersza poleceń oraz polecenia na żywo `lattice`, `daq pool-*`, `project` i `time-sync`. Idealne rozwiązanie do automatyzacji, tworzenia skryptów i pracy w trybie bezinterfejsu. Dostępne na **systemach Windows, Linux amd64 oraz Linux arm64 (NVIDIA Jetson)**. _Dostęp do CLI wymaga płatnego planu Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Programowy interfejs w języku Python do automatyzacji i niestandardowych przepływów pracy: przetwarzanie pełnego potoku, sesje na żywo z kamer/matryc, sesje z czujników DAQ oraz automatyzacja zapisanych projektów. Instalowany wraz z pakietem desktopowym/CLI, a także opublikowany jako `pip install chloros-sdk`. _Dostęp do API wymaga płatnego planu Chloros+._

***

## Obsługiwane platformy

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Tak | Tak | Tak |
| **Linux amd64 (x86_64)** | Nie | Tak | Tak |
| **Linux arm64 (NVIDIA Jetson)** | Nie | Tak | Tak |

Instrukcje dotyczące instalacji w systemie Linux można znaleźć w sekcji [Linux i przetwarzanie brzegowe](linux/linux-overview.md).

***

## Rozpocznij pracę w trzech krokach

1. **Instalacja** — pobierz i uruchom instalator dla swojej platformy. Zobacz [Pobierz](download.md).
2. **Zaloguj się (opcjonalnie w przypadku interfejsu graficznego)** — interfejs graficzny przetwarza obrazy bezpłatnie, bez konieczności posiadania konta. [Logowanie do Chloros+](chloros+-login.md) odblokowuje przetwarzanie równoległe, przyspieszenie GPU, wyższe limity urządzeń oraz dostęp do CLI i SDK.
3. **Utwórz swój pierwszy projekt** — otwórz Chloros, utwórz [Nowy projekt](projects.md), [dodaj swoje obrazy](processing-images-gui/adding-files-to-a-project.md) i [rozpocznij przetwarzanie](processing-images-gui/starting-the-processing.md). Aby zamiast tego sterować sprzętem na żywo, otwórz kartę „Kamery” lub „Czujniki światła” — zobacz [GUI: Nawigacja](navigation.md).

***

## Chloros+

Chociaż Chloros jest darmowy w przypadku większości zadań, może się okazać, że potrzebujesz czegoś więcej. Właśnie wtedy przyda ci się płatna licencja na Chloros+. Dzięki licencji Chloros+ możesz odblokować nowe funkcje, takie jak:

* **Przetwarzanie wielowątkowe**: znacznie przyspiesz przetwarzanie obrazów w większych projektach dzięki jednoczesnemu przetwarzaniu obrazów w potoku.
* **Przyspieszenie GPU (CUDA)**: wykorzystaj dzisiejsze opcje większej pamięci GPU, aby jeszcze bardziej przyspieszyć potok przetwarzania obrazów. Aby uzyskać najlepsze wyniki, zalecamy co najmniej 4 GB pamięci VRAM.
* **Chloros+**[**CLI**](CLI.md) **Dostęp**: uruchamiaj Chloros+ z wiersza poleceń, aby zautomatyzować procesy i zintegrować program z własnym oprogramowaniem. Dostępne we wszystkich płatnych planach; egzekwowane po stronie serwera.
* **Chloros+**[**API**](api-python-sdk.md) **Dostęp:** uruchom Chloros+ z poziomu języka Python, aby uzyskać kontrolę programową, umożliwiającą płynną integrację z procesami badawczymi, przepływami pracy związanymi z analizą danych oraz niestandardowymi aplikacjami. Dostępne na każdym płatnym planie; egzekwowane po stronie serwera.
* **Wyższe limity sprzętowe**: podłącz jednocześnie więcej kamer i czujników światła. Bez logowania interfejs graficzny pozwala podłączyć maksymalnie 4 kamery i 2 czujniki światła DAQ; płatne plany zwiększają oba limity:

| Plan | Kamery | Czujniki światła DAQ |
| --- | --- | --- |
| Iron (bezpłatny, bez logowania) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Korzystanie z wielu urządzeń**: każda licencja Chloros+ pozwala na zarejestrowanie co najmniej 2 urządzeń. Używaj swojego konta MAPIR Cloud do zarządzania zarejestrowanymi urządzeniami. Dodaj obsługę kolejnych urządzeń, uaktualniając swoją licencję Chloros+.
* **Zaawansowana metoda debayeringu uwzględniająca teksturę:** wysokiej jakości debayering uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na AI/ML, który eliminuje niemal cały szum powstały podczas debayeringu.
* **Niestandardowe wzory indeksów wielospektralnych:** wprowadź niestandardowe indeksy wielospektralne w kalkulatorach rastrowych Chloros, zarówno do przetwarzania, jak i do środowiska testowego przeglądania obrazów.
* **Linux i przetwarzanie brzegowe:** uruchom Chloros na platformach Linux x86_64 i ARM64, w tym na NVIDIA Jetson, do przetwarzania w terenie i na urządzeniach brzegowych. Zobacz [Przegląd systemu Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Ceny i rejestracja w Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
