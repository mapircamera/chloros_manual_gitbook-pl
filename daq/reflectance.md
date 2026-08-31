# Procedury pomiaru odbicia

Czujnik światła systemu DAQ przekształca obrazy radiometryczne na wartości odbicia. Istnieją dwie odrębne procedury:

1. **Pojedynczy czujnik** — jeden czujnik DAQ mierzy natężenie promieniowania padającego w dół, podczas gdy kamera rejestruje obraz, a Chloros dzieli jasność obrazu z kamery przez tę wartość odniesienia.
2. **Konfiguracja z dwoma czujnikami** — dwa czujniki DAQ, z których jeden monitoruje niebo, a drugi obiekt, generują na żywo spektralną krzywą odbicia bez udziału kamery.

## Pojedynczy czujnik + kamera (odniesienie promieniowania padającego)

Moduł DAQ pełni rolę czujnika światła padającego (DLS): kamera mierzy promieniowanie w górę **L**(W/m²/sr/nm), DAQ mierzy natężenie promieniowania padającego**E** (W/m²/nm), a Chloros oblicza współczynnik odbicia dla każdego pasma według wzoru:

> ρ = π · L / E

Odczyt z modułu DAQ jest zawsze **zsynchronizowany czasowo z ekspozycją** — dlatego moduł DAQ i kamery korzystają z zegara regulowanego protokołem PTP (zobacz [Sieć i synchronizacja czasu modułu DAQ-E](ethernet-ptp.md)). W przypadku pracy na zewnątrz należy założyć czapkę Sunshine z filtrami cosinusowymi i poprawnie ją zadeklarować; deklaracja czapki bezpośrednio skaluje wartość E (patrz [Profile czapek i skalibrowany zakres](caps-and-range.md)). W przypadku pomiarów ilościowych pamiętaj o charakterystyce przyrządu: natężenie promieniowania mierzone ilościowo wynika ze średniej odczytów z co najmniej 15 sekund.

### Przechwytywanie na żywo

Powiąż moduł DAQ z kamerą w zakładce „Kamery”: panel ustawień każdej kamery zawiera menu rozwijane **Czujnik światła**, w którym wyświetlane są wszystkie podłączone moduły DAQ (DAQ-U/M/E) z zakładki „Czujniki światła”; w przypadku zsynchronizowanej matrycy wybór czujnika światła dla całej matrycy jest propagowany do każdego elementu (poszczególne kamery nadal mogą nadpisać to ustawienie). Po powiązaniu widma czujnika są przekazywane do gniazda DLS kamery, a eksportowane wartości odbicia są dzielone przez dopasowany odczyt.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dwie cechy, o których warto wiedzieć:

* **Brak powiązanego urządzenia DAQ → dane o odbiciu są odrzucane, a nie fałszowane.** Chloros odrzuca wynik pomiaru odbicia i rejestruje powód pominięcia, zamiast po cichu zwracać wynik o niższej jakości.
* **Wykorzystany odczyt jest zachowywany.** Dla każdej klatki odbicia faktycznie zastosowany odczyt DAQ jest zapisywany jako plik towarzyszący `.daq` obok obrazu, dzięki czemu dane można później ponownie przetworzyć ([Rejestracja i format .daq](recording.md)).

### Przetwarzanie zarejestrowanych obrazów

W celu przetwarzania po zakończeniu lotu należy podczas sesji zarejestrować plik `.daq` i zachować go wraz z obrazami — potok przetwarzania automatycznie rozpoznaje dane o promieniowaniu padającym o dopasowanym znaczniku czasu, pobierając przy pierwszym użyciu wszelkie brakujące kalibracje fabryczne z chmury pliku MAPIR. Nagrania z interfejsu graficznego są automatycznie dodawane do otwartego projektu po ich zakończeniu.

Odniesienie odbicia można wybrać w trakcie przetwarzania — `--reflectance-source` na `chloros-cli process` lub ustawienie źródła odbicia w sekcji Ustawienia projektu interfejsu graficznego:

| Wartość | Zachowanie |
| --- | --- |
| `auto` (domyślnie) | Cel kalibracyjny w ramce, który przeszedł kontrolę jakości, stanowi odniesienie bezwzględne; sygnał DAQ w kierunku w dół (ρ = π·L/E) jest wartością rezerwową |
| `daq` | Wymagana autorytatywność danych z DAQ |
| `target` | Ścisły cel w ramce; brak zastępowania danymi z DAQ |

Zobacz [Cele kalibracyjne](../calibration-targets.md), aby zapoznać się z procedurami dotyczącymi celów, oraz [rozdział LATTICE](../lattice/README.md) oraz [CLI Reference](../reference/cli-reference.md), aby zapoznać się z pełnym procesem przetwarzania. Podczas odczytu wyeksportowanych pikseli odbicia należy stosować skalę podaną w pliku (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — patrz [Formaty obrazów wyjściowych](../output-image-formats.md).

### Pasma poza skalibrowanym zakresem urządzenia DAQ

Zakres radiometrycznie skalibrowany urządzenia DAQ wynosi ~374–974 nm. Chloros odrzuca wartości odbicia oparte na DAQ dla każdego pasma kamery, którego mniej niż połowa wagi spektralnej mieści się w tym przedziale, zgłaszając powód pominięcia `dls-uncalibrated-band-<nm>`. Spośród dostępnych w sprzedaży modeli dotyczy to wyłącznie modelu F988: współczynnik odbicia kamery F988 jest kalibrowany za pomocą panelu odbicia umieszczonego w scenie; pasmo to wykracza poza skalibrowany zakres czujnika światła modułu DAQ, dlatego Chloros stosuje najnowszy zapis z panelu i zachowuje go między kolejnymi pomiarami panelu. Jeśli kamera F988 działa wyłącznie w trybie DAQ, kod Chloros odrzuca odbicie oparte na DAQ dla tego pasma z powodem pominięcia `dls-uncalibrated-band-988` — obsługiwanym rozwiązaniem jest procedura z wykorzystaniem panelu.

## Podwójny czujnik (oświetlenie otoczenia + obiekt)

Dwa czujniki DAQ — dowolna para, w dowolnym układzie — zapewniają spektrum odbicia na żywo bez kamery: jeden czujnik skierowany jest w stronę nieba (**źródło światła otoczenia**), a drugi w stronę obiektu (**skaner obiektu**), a Chloros oblicza dla każdej długości fali:

> R(λ) = obiekt(λ) / otoczenie(λ)

(wartość zerowa, gdy otoczenie ≤ 0).

### W interfejsie graficznym

Po podłączeniu obu czujników w zakładce „Czujniki światła” otwórz nakładkę dodawania czujnika (przycisk „+” na kafelku wykresu w widoku siatki) i wybierz opcję **Połącz światło otoczenia + obiekt**. Wybierz dwa czujniki z list rozwijanych Źródło światła otoczenia i Skaner obiektu, a następnie kliknij Utwórz. Grupa pojawi się jako osobny wykres oraz jako wiersz na pasku bocznym z zieloną plakietką**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Pod wykresem odbicia (widok listy) znajduje się aktualizowana na bieżąco **tabela wskaźników wegetacyjnych**, która oblicza wskaźniki na podstawie krzywej przy użyciu centrów pasm: niebieski 450 / zielony 550 / czerwony 670 / NIR 800 nm. Wskaźniki oparte na stosunku, które eliminują skalę bezwzględną (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) są zawsze wyświetlane; wskaźniki wymagające współczynnika odbicia bezwzględnego (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) pojawiają się tylko wtedy, gdy oba czujniki są modelami z kalibracją mocy.

### Pozorne a względne — zasada nazewnictwa

Chloros oznacza dane wyjściowe z dwóch czujników zgodnie z tym, co faktycznie może zapewnić dana para czujników:

| Para czujników | Oznaczenie |
| --- | --- |
| Oba czujniki skalibrowane — wczytany pakiet fabryczny | **Odbicie pozorne** |
| Jeden z czujników nieskalibrowany | **Odbicie względne** |

Wszystkie trzy modele są radiometryczne: po załadowaniu fabrycznego pakietu kalibracyjnego czujnika jego widma są wyrażone w wartościach bezwzględnych W/m²/nm, więc para skalibrowanych czujników określa bezwzględne odbicie pozorne — transport nie ma na to wpływu. Czujnik nadal przesyłający surowe wartości liczbowe (brak dostępnego zestawu) obniża rangę wyniku do krzywej względnej (kształt widma pozostaje ważny). Oba czujniki powinny posiadać poprawnie zadeklarowane limity ([Profile limitów i zakres kalibracji](caps-and-range.md)).

### Z Python

W połączonej powierzchni SDK nie ma dedykowanego wywołania dla dwóch czujników: należy otworzyć dwie sesje za pomocą `chloros_sdk.connect_daq_sensor()` i samodzielnie porównać ich widma `latest()`, stosując tę samą konwencję nazewniczą. (Narzędzie do rejestracji z dwoma czujnikami istnieje również na wewnętrznej powierzchni sprzętowej MAPIR, wymienionej w [Podręczniku CLI](../reference/cli-reference.md) dla kompletności — nie jest ono częścią dostarczanego urządzenia CLI; obsługiwanym sposobem postępowania jest przedstawiony powyżej przebieg pracy w interfejsie graficznym.)
