# Procedury pomiaru odbicia

Czujnik światła DAQ przekształca obrazy radiometryczne na wartości odbicia. Istnieją dwie odrębne procedury:

1. **Pojedynczy czujnik** — jeden czujnik DAQ mierzy natężenie promieniowania padającego, podczas gdy kamera rejestruje obraz, a program Chloros dzieli jasność zarejestrowaną przez kamerę przez tę wartość odniesienia.
2. **Dwa czujniki** — dwa czujniki DAQ, z których jeden monitoruje niebo, a drugi obiekt, generują na żywo krzywą spektralnego współczynnika odbicia bez udziału kamery.

## Pojedynczy czujnik + kamera (odniesienie promieniowania padającego)

Czujnik DAQ pełni rolę czujnika światła padającego (DLS): kamera mierzy promieniowanie wychodzące **L**(W/m²/sr/nm), DAQ mierzy natężenie promieniowania padającego**E** (W/m²/nm), a Chloros oblicza współczynnik odbicia dla każdego pasma według wzoru:

> ρ = π · L / E

Odczyt z modułu DAQ jest zawsze **zsynchronizowany czasowo z ekspozycją** — dlatego moduł DAQ i kamery korzystają ze wspólnego zegara regulowanego protokołem PTP (zobacz [Sieć i synchronizacja czasu modułu DAQ-E](ethernet-ptp.md)). W przypadku pracy na zewnątrz należy założyć osłonę Sunshine typu cosine cap i poprawnie ją zadeklarować; deklaracja osłony bezpośrednio skaluje wartość E (patrz [Profile osłon i skalibrowany zakres](caps-and-range.md)). W przypadku pomiarów ilościowych należy pamiętać o charakterystyce przyrządu: ilościowe natężenie promieniowania wynika ze średniej odczytów z co najmniej 15 sekund.

### Przechwytywanie na żywo

Powiąż moduł DAQ z kamerą w zakładce „Kamery”: panel ustawień każdej kamery zawiera menu rozwijane **Czujnik światła**, w którym wymienione są wszystkie podłączone moduły DAQ (DAQ-U/M/E) z zakładki „Czujniki światła”; w przypadku zsynchronizowanej matrycy wybór czujnika światła dla całej matrycy rozprzestrzenia się na każdy element (poszczególne kamery nadal mogą nadpisać te ustawienia). Po powiązaniu widma czujnika są przekazywane do gniazda DLS kamery, a eksportowane wartości odbicia są dzielone przez dopasowany odczyt. W macierzy element posiadający własny czujnik wykorzystuje własny odczyt — dla swoich pasm w kompozycji na żywo, eksportowanych danych oraz automatycznej ekspozycji — a pozostałe elementy korzystają z odczytu całej macierzy.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dwie cechy, o których warto wiedzieć:

* **Brak powiązania z DAQ → współczynnik odbicia jest odrzucany, a nie fałszowany.** Program Chloros odrzuca wynik współczynnika odbicia i rejestruje powód pominięcia, zamiast po cichu zwracać gorszy wynik.
* **Wykorzystany odczyt jest zachowywany.** Dla każdej klatki odbicia faktycznie zastosowany odczyt z DAQ jest zapisywany jako plik `.daq` w folderze `daq/` danego zapisu, dzięki czemu zapis można później ponownie przetworzyć ([Rejestracja i format .daq](recording.md)). Plik `.daq.cameras.json` znajdujący się obok określa kamery, których dotyczy, a ponowne przetwarzanie dopasowuje go wyłącznie do obrazów z tych kamer.

### Przetwarzanie zarejestrowanych obrazów

W celu przetwarzania po zakończeniu lotu należy podczas sesji nagrać plik `.daq` i zachować go wraz z materiałem obrazowym — system przetwarzania automatycznie dopasowuje dane o promieniowaniu padającym na podstawie znaczników czasu, pobierając przy pierwszym użyciu wszelkie brakujące kalibracje fabryczne z chmury MAPIR. Nagrania z interfejsu graficznego są automatycznie dodawane do otwartego projektu po ich zakończeniu.

Odniesienie odbicia można wybrać w momencie przetwarzania — `--reflectance-source` na `chloros-cli process` lub ustawienie źródła odbicia w sekcji „Ustawienia projektu” interfejsu graficznego:

| Wartość | Zachowanie |
| --- | --- |
| `auto` (domyślnie) | Cel kalibracyjny w ramce, który przeszedł kontrolę jakości (QA), stanowi odniesienie absolutne; promieniowanie wpadające z DAQ (ρ = π·L/E) jest rozwiązaniem rezerwowym. Cel, który przeszedł kontrolę jakości z flagą (opuszczony panel lub niesprawne pasmo), nie jest automatycznie preferowany: łańcuchy „cel”, „cel+DAQ” oraz „tylko DAQ” są testowane w kolejności zgodnej z ich wynikami pomiarowymi na panelach testowych. |
| `daq` | DAQ jako źródło ostateczne |
| `target` | Ścisły cel w ramce; brak zastąpienia przez DAQ |

Zobacz [Cele kalibracyjne](../calibration-targets.md), aby zapoznać się z procedurami dotyczącymi celów, oraz [rozdział LATTICE](../lattice/README.md) oraz [Podręcznik CLI](../reference/cli-reference.md), aby zapoznać się z pełnym procesem przetwarzania. Podczas odczytu wyeksportowanych pikseli odbicia należy używać skali oznaczonej (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — zobacz [Formaty obrazów wyjściowych](../output-image-formats.md).

### Pasma poza skalibrowanym zakresem modułu DAQ

Zakres kalibracji radiometrycznej modułu DAQ wynosi ~374–974 nm. Program Chloros odrzuca dane odbicia oparte na module DAQ dla każdego pasma kamery, którego mniej niż połowa wagi spektralnej mieści się w tym przedziale, zgłaszając powód pominięcia `dls-uncalibrated-band-<nm>`. Spośród dostępnych w sprzedaży modeli dotyczy to wyłącznie modelu F988: współczynnik odbicia kamery F988 jest kalibrowany za pomocą panelu odbicia umieszczonego w scenie; pasmo to wykracza poza skalibrowany zakres czujnika światła modułu DAQ, więc Chloros stosuje najnowszy zapis z panelu i utrzymuje tę wartość między kolejnymi pomiarami panelu. Jeśli kamera F988 działa wyłącznie w trybie DAQ, Chloros odrzuca współczynnik odbicia oparty na DAQ dla tego pasma, podając powód pominięcia `dls-uncalibrated-band-988` — obsługiwanym sposobem postępowania jest procedura z wykorzystaniem panelu.

## Podwójny czujnik (oświetlenie otoczenia + obiekt)

Dwa czujniki DAQ — dowolna para, niezależnie od sposobu transportu — zapewniają spektrum odbicia na żywo bez użycia kamery: jeden czujnik jest skierowany w stronę nieba (**źródło światła otoczenia**), a drugi w stronę obiektu (**skaner obiektu**), a program Chloros oblicza wartość dla każdej długości fali:

> R(λ) = obiekt(λ) / otoczenie(λ)

(wartość zerowa, gdy otoczenie ≤ 0).

### W interfejsie graficznym

Po podłączeniu obu czujników w zakładce „Czujniki światła” otwórz nakładkę dodawania czujnika (przycisk „+” na kafelku wykresu w widoku siatki) i wybierz opcję **Połącz światło otoczenia + obiekt**. Wybierz dwa czujniki z list rozwijanych**Źródło światła otoczenia**i**Skaner obiektu**, a następnie kliknij**Utwórz**. Grupa pojawi się jako osobny wykres oraz jako wiersz na pasku bocznym z zieloną plakietką**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Pod wykresem odbicia (widok listy) znajduje się aktualizowana na bieżąco **tabela wskaźników wegetacyjnych**, która oblicza wskaźniki na podstawie krzywej przy użyciu centrów pasm: niebieskiego 450 / zielonego 550 / czerwonego 670 / NIR 800 nm. Wskaźniki oparte na stosunku, które eliminują skalę bezwzględną (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) są zawsze wyświetlane; wskaźniki wymagające odbicia bezwzględnego (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) pojawiają się tylko wtedy, gdy oba czujniki są modelami skalibrowanymi pod kątem mocy.

### Pozorne a względne — zasada oznaczania

Chloros oznacza wyniki z dwóch czujników zgodnie z tym, co faktycznie może zapewnić dana para czujników:

| Para czujników | Oznaczenie |
| --- | --- |
| Oba czujniki skalibrowane — załadowany pakiet fabryczny | **Pozorne odbicie** |
| Jeden z czujników nieskalibrowany | **Odbicie względne** |

Wszystkie trzy modele są radiometryczne: po załadowaniu fabrycznego pakietu kalibracyjnego czujnika jego widma są wyrażone w wartościach bezwzględnych W/m²/nm, więc para skalibrowanych czujników określa bezwzględne odbicie pozorne — transport nie ma na to wpływu. Czujnik nadal przesyłający surowe dane liczbowe (pakiet niedostępny) obniża rangę wyniku do krzywej względnej (kształt widma pozostaje prawidłowy). Oba czujniki powinny posiadać poprawnie zadeklarowane ograniczenia ([Profile ograniczeń i zakres kalibracji](caps-and-range.md)).

### Z języka Python


W zbiorczej powierzchni SDK nie ma dedykowanej funkcji obsługującej dwa czujniki: należy otworzyć dwie sesje za pomocą `chloros_sdk.connect_daq_sensor()` i samodzielnie obliczyć stosunek ich widm `latest()`, stosując tę samą konwencję nazewniczą. (Narzędzie do rejestracji z wykorzystaniem dwóch czujników istnieje również w wewnętrznym interfejsie MAPIR umożliwiającym bezpośredni dostęp do sprzętu; dla kompletności zostało ono wymienione w [Podręczniku CLI](../reference/cli-reference.md) — nie jest ono jednak częścią dostarczanego interfejsu CLI; obsługiwanym sposobem postępowania jest opisany powyżej proces z wykorzystaniem interfejsu graficznego.)
