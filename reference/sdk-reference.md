# Dokumentacja Chloros Python SDK

**Wersja:**

1.3.0**Data utworzenia:**

29.07.2026, godz. 19:19 ·**Data aktualizacji:**

06.11.2026**Pakiet:** `chloros-sdk` (PyPI)**Grupa docelowa:** Zoptymalizowane pod kątem wykorzystania przez modele LLM; czytelne dla ludzi.**Zakres:** Wszystkie publiczne klasy, funkcje i funkcje pomocnicze udostępniane przez `import chloros_sdk`, wraz z przykładami, które można skopiować i wkleić, obejmującymi przetwarzanie obrazu, sterowanie pojedynczą kamerą, zsynchronizowane tablice, czujników DAQ oraz automatyzacji projektów, a także serwer `chloros-mcp` dla asystentów AI.

Jeśli interesują Cię tylko najważniejsze informacje, przejdź do:
- [Instalacja i szybki start](#instalacja)
- [Smart-Connect dla układów LATTICE](#smart-connect-for-lattice-cameras)
- [Sesje czujników DAQ](#daq-sensor-sessions)
- [Narzędzia do kamer](#narzędzia-do-kamer)
- [Macierze przejściowe, profile wyrównania i indeksy](#macierze-przejściowe-profile-wyrównania--indeksy)
- [Folder projektu i receptury](#folder-projektu--receptury)
- [Automatyzacja projektów](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Klatki na żywo we własnym potoku przetwarzania](#live-frames-in-your-own-pipeline)
- [Serwer MCP (asystenci AI)](#mcp-server-ai-assistants)

> **Zmiany w wersji 1.3.0** (co wykrywa istniejący skrypt w wersji 1.2.0).
>
> **Zmiany, z którymi skrypt w wersji 1.2.0 może musieć sobie poradzić:**
>
> - **Występuje błąd, którego backend obecnie nie potwierdza.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` w tablicy otwartej przez sesję, `DAQSensorSession.disconnect()` / `force_disconnect()` na czujniku, który otworzyła, oraz `ChlorosProject.disconnect_all()` generują błąd `ChlorosConnectError`, gdy backend odrzuca zwolnienie lub nie potwierdza go na czas, ponieważ urządzenie może być nadal zajęte. W wersji 1.2.0 powracały one bez komunikatu. Wyjście z bloku `with` nadal nigdy nie generuje wyjątku.
> - **Wypożyczone urządzenie nigdy nie jest zwalniane.** `with attach_array(...)` oraz jawne wywołanie `disconnect()` na nim pozostawiają tablicę aktywną (w wersji 1.2.0 opuszczenie bloku powodowało jej zwolnienie, nawet jeśli należała do GUI); wywołaj `force_disconnect()`, jeśli chcesz zakończyć jego działanie. Podobnie `connect_daq_sensor()`, który wykrył, że czujnik znajduje się już w puli (w tym z włączoną funkcją inteligentnego wykrywania z flagą zerową), pozostawia go otwartym w `disconnect()` oraz po opuszczeniu bloku `with`.
> - **Formaty PNG i JPEG obsługują wyłącznie produkty `display` i indeksowane.** `capture()` wraz z `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) oraz na poziomie `radiance` lub `reflectance`, lub `processing="all"`, generuje `ValueError` przed każdym żądaniem; 12-bitowy `raw` / `debayered` jest odrzucany przez backend. Zobacz [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` otwiera ten czujnik lub generuje** `ChlorosConnectError` z listą znalezionych elementów; nigdy nie przechodzi na inny czujnik. Zobacz [Transport przypisany](#pinned-transport).
> - **`frame_stream()` generuje komunikat zamiast przejść w stan ciszy:** `FrameFetchError` po odrzuceniu w następnym takcie powtarza (odczyt współczynnika odbicia bez żadnego DAQ), po 60 sekundach bez ramki oraz po 60 sekundach bez nowego wyzwalacza. Zobacz [Jeden wyzwalacz na odczyt](#one-trigger-per-read--group_id).
> - **Przekroczenie limitu czasu odczytu lub utrata połączenia powoduje wygenerowanie błędu `ChlorosConnectError`** przez każde wywołanie funkcji smart-connect oraz błędu `analyze_array_network()`, gdzie w wersji 1.2.0 generowano podstawowy błąd przekroczenia limitu czasu lub błąd połączenia.
> - **`connect_array()` domyślnie czeka 600 s (`timeout`, 180 s w wersji 1.2.0), a połączenie, z którego rezygnuje, zostaje cofnięte.** Wywołanie wysyła swój kod błędu `timeout` do zaplecza, które przywraca każdą kamerę do stanu, w jakim została wykryta, i nie nawiązuje żadnego połączenia, gdy połączenie nie zostanie nawiązane w odpowiednim czasie; wywołanie generuje kod błędu `ChlorosConnectError` (kod HTTP 504 `CLIENT_DEADLINE_PASSED`). W wersji 1.2.0 nawiązanie połączenia bezpośrednio po włączeniu zasilania kamer mogło trwać dłużej niż wywołanie i doprowadzić do połączenia tablicy po jej wygenerowaniu. `connect_all()` oczekuje na każdą zapisaną tablicę w ten sam sposób (120 s w wersji 1.2.0). Zobacz [`connect_array()` Signature](#connect_array-signature).
> - **`connect_array()` zachowuje pełny czujnik.** Bez `width` / `height` / `binning` każda kamera jest ustawiona na pełną matrycę, a liczba klatek na sekundę spada do poziomu zapewnianego przez host i sieć; w wersji 1.2.0 zmniejsza lub łączy klatki, aby utrzymać częstotliwość. Zobacz [Synchronized Array](#synchronized-array--arraysession-smart-prep).
> - **Format pikseli Bayera w kamerze M3M (mono) to jej głębia.** Kamera M3M oferuje wyłącznie formaty mono, a nazwy ustawień wstępnych odnoszą się do formatów Bayera: w wersji 1.2.0 kamera odrzucała ten format i zachowywała swój własny, podczas gdy `settings.pixel_format` nazywał format Bayera. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` oraz tablicawspólny format ustawiają teraz format Mono o tej samej głębokości na pliku M3M: `"BayerRG8"` i `high_speed` dają `Mono8`; `"BayerRG12"`, `high_quality` oraz `triggered` dają `Mono12`; `cam.settings.pixel_format` nadaje mu nazwę. To samo działa w drugą stronę: M3C nigdy nie jest zapisywany w formacie Mono, więc `"Mono12"` — z dowolnego z nich, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` czy zapisany format macierzy — daje `BayerRG12` na M3C, niezależnie od tego, która kamera jest urządzeniem nadrzędnym w macierzy. Przekazany obiekt `CameraSettings` zachowuje swoją nazwę, więc ten sam obiekt nadal żąda od każdej kamery tej głębokości w swojej własnej rodzinie. `connect_camera(preset=...)` zachowuje format presetu (wersja 1.2.0 ustawia kamerę na tryb 12-bitowy po zastosowaniu presetu), a kamera otwarta na presecie nie zapisuje już swoich danych do `PRESETS`. Każdy inny format, którego kamera nie obsługuje (którego brakuje na jej liście `PixelFormat`, np. `RGB8` w M3M lub `BayerRG16` w oprogramowaniu sprzętowym, które go nie obsługuje), również nie jest nazywany jako osobny: po połączeniu z `LatticeCamera`, `apply_settings()` lub `set_format()`, `cam.settings.pixel_format` oraz `packed_transport` określają format strumienia z kamery, a jeden wyświetlony wiersz wskazuje format, którego nie zaakceptowano.
> - **`analyze_array_network()` określa, które połączenie ma zastosowanie:** `pixel_format=None` (poprzednio `"BayerRG8"`) oznacza formaty ustawione przez połączenie (Mono12 / BayerRG12), a `packed="auto"` określa sposób pakowania danych podczas przesyłania, więc podana przez niego liczba klatek na sekundę może różnić się od wartości podanej w wersji 1.2.0&#x27;. Kod `pixel_format`, który przekazujesz, generuje plik M3M o tej głębokości. Zobacz [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` zwraca `CaptureResult`** (nadal lista słowników ramek) zawierającą `.skipped`, `.failed`, `.daq_files` oraz `.daq_matches`. Wynik `ArraySession.capture()` zawiera również `.alignment` (informację, czy zastosowano wyrównanie, a jeśli nie, to dlaczego) oraz `.oriented`.
> - **Pobrane tablice są domyślnie wyrównane, podobnie jak w przypadku aplikacji.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` oraz `ArrayHandle.capture()` / `capture_smart()` dopasowują pliki `debayered`, `display`, `radiance`, `reflectance` oraz pliki indeksowe do klatki kamery głównej i przycina je do obszaru nakładania się członków (mniejszy od matrycy o przesunięcia kamer, `XMP-mapir:AlignmentJSON` `"applied": true`), dzięki czemu są to te same pliki, co te zarejestrowane przez aplikację Chloros dla tej samej sceny; `raw` pozostaje w rozmiarze natywnym. Wcześniej podczas przechwytywania bez `aligned=True` każdy plik był zapisywany w rozmiarze natywnym, bez korekcji zniekształceń. `aligned=False` zachowuje tę właściwość. Pliki `chloros-mcp``capture_array` i `start_capture_run` przyjmują tę samą wartość domyślną. Zobacz [Metody `ArraySession`](#arraysession-methods).
> - **`connect_all()` wskazuje backend, który odrzuca SDK**. Działający backend, który jest wylogowany lub korzysta z planu poniżej Copper, jest zgłaszany pod `errors` z własnym powodem (HTTP 401 / 403), a nie jako „żaden backend Chloros nie działa”, a każde wywołanie `connect_all()` ponownie sprawdza backend, więc wywołanie go po zalogowaniu działa na tym samym obiekcie projektu.
> - **Względna ścieżka `output_dir` działa w przypadku lokalnego backendu.** Ścieżki względne są przekształcane na ścieżki bezwzględne względem katalogu roboczego skryptu przed ich wysłaniem. W wersji 1.2.0 backend rozpoznawał je w swoim własnym folderze instalacyjnym, co kończyło się błędem „Odmowa dostępu”.
> - **`numpy` jest teraz wymagane** przez SDK (wywołania funkcji live-frame zwracają tablice numpy).
> - **Nazwa na poziomie eksportu, która nie jest liczbą, zostanie odrzucona.** `processing=` oraz `levels=` przyjmują `raw`, `debayered`, `radiance`, `reflectance` lub `display` (lub alias, taki jak `preview`; `processing=`, a także `"all"`). Wszelkie inne nazwy są odrzucane przed dotknięciem aparatu (kod błędu HTTP 400 `UNKNOWN_LEVEL`: `capture()` generuje `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); wersja 1.2.0 zapisała lub zwróciła dla tego `raw`.
> - **Funkcja Texture Aware jest odrzucana w przypadku przechwyceń LATTICE.** W projekcie zawierającym ujęcia LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` generuje błąd `ChlorosConfigurationError`, podobnie jak `process()` po błędzie `configure()`, który wymagał o to przed importem. Wartość opcji „Texture Aware” zapisana wcześniej w projekcie zostaje ustawiona na „Standard” wraz z błędem `UserWarning`. Ujęcia LATTICE zawsze podlegają standardowej demosaice, więc wersja 1.2.0 zaakceptowała to ustawienie i je zignorowała; aplikacja Chloros również nie oferuje tej opcji dla tych ujęć. Zobacz [Obsługiwane wartości](#supported-values).
> - **Kamery powracają do stanu, w jakim zostały znalezione — chyba że wystąpi błąd `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, na końcu każdej funkcji przejściowej (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) oraz zatrzymanie funkcji `start_camera_continuous()` przywracają każdą kamerę do stanu, w jakim została znaleziona: okablowanie wyzwalacza i linii synchronizacji, połączenie PTP oraz wszystkie ustawienia kamery zmienione podczas sesji (ekspozycja i wzmocnienie, cel AE, limity i tłumienie, balans bieli, poziom czerni, format pikseli, binning i ROI, odwrócenia, częstotliwość klatek). Z kolei `keep_settings=True` zapisuje ustawienia każdej kamery w jej własnej pamięci, dzięki czemu po włączeniu kamery uruchamiają się one z tymi ustawieniami; `clear_settings=True` resetuje tę pamięć do ustawień fabrycznych. W obu przypadkach kamery pozostają w trybie swobodnej pracy. Projekt ustawia kamery na podstawie własnych zapisanych ustawień po podłączeniu ich. Zobacz [Pamięć kamery](#camera-memory--keep_settings--clear_settings).
> - **Każdy tryb zapisuje projekt w ten sam sposób.** Każda zmiana dotycząca kamery, matrycy lub czujnika światła podłączonego w ramach projektu — wprowadzona za pomocą SDK, aplikacji, CLI lub serwera MCP — jest zapisywana w pliku `cameras.json` / `sensors.json`, a otwarcie projektu w dowolnym trybie powoduje skonfigurowanie urządzeń zgodnie z tymi ustawieniami; plik `ChlorosProject` określa nazwę własnego folderu w backendzie. Wcześniej tylko aplikacja zapisywała projekt. Zobacz [Jeden rekord projektu, każdy tryb](#one-project-record-every-mode).
> - **Pliki utworzone z obrazów z kilku kamer zawierają nazwy każdej z nich.** Stos wielopasmowy, indeks na żywo utworzony z pasm kilku kamer oraz obraz z przechwytywania tablicowego `composite/` zawierają numer seryjny, model i pakiet kalibracyjny każdej kamery źródłowej w postaci list (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), a listy EXIF `SerialNumber` zawierają numery seryjne połączone przecinkami. Wcześniej nie zawierały one informacji o kamerze lub zawierały wyłącznie numer seryjny pierwszej kamery. Zobacz [Pliki utworzone z danych z kilku kamer](#files-made-from-several-cameras).
> - **Tylko plik `raw` zawiera tagi kalibracyjne kamery.** Zestaw zgodny z MicaSense (`BlackLevel`, `RadiometricCalibration`, pliki dla poszczególnych`Band{R,G,B}RadiometricCalibration` oraz zestawy `MicaSenseVignetting*`) oraz ładunek `CalibrationJSON` opisują surowe wartości czujnika, więc są one dołączone wyłącznie do pliku przechwytywania `raw`. Wszystkie pozostałe pliki — pliki przechwytywania`debayered`, `display`, `radiance`, `reflectance` oraz pliki indeksowe, ramki z nagrań ciągłych i z wielu kameroraz każdy plik produktu `process()` — zawiera identyfikator kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...), a podczas przechwytywania — własny zapis klatki (`XMP-mapir:FrameJSON`: ekspozycja, wzmocnienie, temperatura, format pikseli, czas przechwycenia). W wersji 1.2.0 zawierały one również zestaw danych surowych, który narzędzie je wykorzystujące (Pix4D, Metashape) stosowało do pikseli niebędących wartościami surowymi. Program Processing ustala kalibrację takiego pliku na podstawie jego numeru seryjnego oraz `CalBundleSha256`.
>
> **Inne zmiany:**
>
> - **Automatyczna częstotliwość klatek tablicy dostosowuje się do tego, co dostarcza tablica.** Częstotliwość, która traci ponad 1% swoich wyzwalaczy, zostaje obniżona – ocena opiera się na kilku długich obserwacjach, a nie na jednej – a częstotliwość, z której została obniżona, jest od czasu do czasu ponownie testowana. Częstotliwość ustawiona ręcznie pozostaje niezmieniona.
> - **Automatyczna ekspozycja działa stabilniej.** Utrzymuje stały poziom naświetlenia, ustabilizuje się w ciągu kilku sekund po zmianie, ignoruje niewielkie prześwietlenia i przywraca działanie aparatu, który przestaje reagować w jasnym słońcu. Zobacz [Jak działa automatyczna ekspozycja](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` zlicza wyłącznie Twoje zdjęcia i nagrania.** W folderze zawierającym plik `camera_cal/` projektu pliki kalibracyjne nie są już zaliczane do zdjęć i nagrań, a projekt, który je już uwzględnił, usuwa je.
> - **`configure(radiometric_output=...)` ma zastosowanie.** Teraz ustawia produkty projektu `Export radiance` / `Export reflectance`; wcześniej ustawienia produktów każdego projektu miały pierwszeństwo, więc `radiance`, `sensor-response` i `all` eksportowały zarówno promieniowanie, jak i współczynnik odbicia.
> - **`test_array_sync()` przechodzi ze stanu włączenia kamer.** Włącza protokół PTP kamer na czas trwania zadania, czeka na zsynchronizowanie i ustabilizowanie się ich zegarów przed pierwszym wyzwoleniem (`ptp_lock`) — każda kamera pozostaje zsynchronizowana przez 120 s, czyli około 2 minut od włączenia zasilania (`ptp_settle_s=` ustawia ten parametr), ponieważ zegar, który właśnie się zsynchronizował, wykazuje odchylenie do ~200 us przez około 2 minuty — i przywraca PTP każdej kamery do stanu, w jakim je zastał. Nie czeka się ponownie na kamerę, która pozostała zsynchronizowana z wcześniejszego testu. Wcześniej pozostawiał PTP wyłączony w sieci z jednym przełącznikiem, w której żaden moduł DAQ-E nie odpowiadał, i odczytywał zegar swobodny każdej kamery jako zegar PTP, który nie zsynchronizował się („PTP nie jest zsynchronizowane”, przy każdym uruchomieniu).
> - **Zapis `.daq` wskazuje, czy nasadka została wybrana, czy też została przypisana automatycznie.** Urządzenia DAQ-U, DAQ-M i DAQ-E są dostarczane bez nasadki, a Chloros nie jest w stanie stwierdzić, czy jest ona zamontowana, więc czujnik podłączony bez nazwanej nasadki otrzymuje domyślnie nazwę `sunshine_cosine`, co jest *przyjęciem*. Każdy zapis Chloros (schemat 1.30) przechowuje teraz `cap_id_source` obok `cap_id`: `operator`, gdy nadałeś nazwę nasadce (w tym `none`), `auto_default`, gdy był to parametr domyślny. Przetwarzanie zapisu, w którym wartość kondensatora została przyjęta domyślnie, przy ustawieniu kondensatora DAQ projektu na „Auto”, powoduje wyświetlenie `[daq-dls] WARNING` z nazwą pliku: sam DAQ-U bez korekcji nasłonecznienia wskazuje wartość około 30 razy wyższą. **Należy nadać nazwę zamontowanemu kondensatorowi** podczas podłączania (`connect_daq_sensor(cap_id=...)` lub `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap` lub z listy rozwijanej kondensatorów w GUI), albo ustaw kondensator DAQ w projekcie, a ostrzeżenie zniknie.
> - **Koniec z oknami konsoli pip w systemie Windows.** Na komputerze z zainstalowanym Pythonem backend i każdy z jego modułów przetwarzania obrazów uruchamiał `pip` raz podczas startu systemu, a każdy z nich otwierał okno konsoli. Teraz już tego nie robią.
> - **Nagrania na żywo `.daq` przetrwają przerwę w zasilaniu.**Nagranie jest zapisywane z dziennikiem i pełną synchronizacją dysku przy każdym zatwierdzeniu, za czujnikiem, więc wolny dysk opóźnia zapis pliku, ale nigdy nie opóźnia odczytów. Nagranie zatrzymane w trakcie zatwierdzania (przerwa w zasilaniu, siłowe zakończenie działania) może pozostawić obok siebie plik `<name>.daq-journal`:**należy skopiować oba pliki razem.**Chloros przywraca dziennik przy pierwszym odczycie pliku, więc zapis otwiera się z wszystkimi zmianami zatwierdzonymi przed zatrzymaniem (utracone są około ostatnich 3 sekund); w wersji 1.2.0 przerwanie zasilania w nieodpowiednim momencie mogło spowodować, że cały plik stał się nieczytelny („obraz dysku bazy danych jest uszkodzony”).
> - **Plik `DAQSensorSession.record_start()` na obciążonym dysku nie powoduje już zawieszenia czujnika**: odczyty są umieszczane w kolejce i zapisywane w osobnym wątku, a te, których nie udało się zapisać, są uwzględniane w `status()` (wraz z `record_errors` i `record_last_error`).
> - **Bezpośredni `LatticeCamera`:**klatka bez ekspozycji/wzmocnienia fragmentu zgłasza teraz**rzeczywistą** ekspozycję i wzmocnienie kamery (w trybie awaryjnym odczytywana była wartość z pamięci podręcznej, która mogła pozostać niezmieniona przez cały cykl automatycznej ekspozycji), a `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` stosują to samo 5-mikrosekundowe filtrowanie zakłóceń wejściowych, jakie otrzymują podłączone kablem urządzenia Chloros (niefiltrowane – urządzenie wyzwalane wysyłało klatki bez wysłanego sygnału wyzwalającego).
> - **Kamery wyzwalane programowo nie pomijają już co drugiego sygnału wyzwalającego.** Kamera w trybie wyzwalania programowego umieszcza teraz w kolejce sygnał wyzwalający, który nadejdzie podczas jej odczytu (lub Chloros czeka, aż będzie gotowa), tak jak robiły to już kamery synchronizowane kablem.
> - **`discover_daq_sensors()` nadaje nazwę DAQ-E-S `DAQ-E-S`.** Jego `model` to `DAQ-E` dla każdego czujnika Ethernet; czujnik, który nie odpowie na sprawdzenie tożsamości w ciągu około 2 s, nadal figuruje na liście jako `DAQ-E`.
> - **Nowość: `chloros-mcp`, serwer MCP dla asystentów AI.** Claude Desktop, Claude Code, Cursor i inne klienty MCP mogą przetwarzać obrazy (tworzyć projekty, importować zdjęcia, wybierać ustawienia, przetwarzać, generować raporty z wyników przetwarzania) oraz sterować kamerami LATTICE, matrycami i czujnikami światła DAQ: łączyć się, wyświetlać podgląd, ustawiać ekspozycję i kolory, robić zdjęcia, rejestrować. Jego narzędzia obejmują funkcje tego SDK. Instaluje się wraz z SDK, a pakiet .deb dla systemu Linux umieszcza go w katalogu `/usr/bin`. Zobacz [Serwer MCP (asystenci AI)](#mcp-server-ai-assistants).
> - **Nowość: narzędzia do obsługi kamer** – polecenia `chloros-cli lattice` wywoływane przez SDK: profil kolorów, nasycenie / kontrast, światła, balans bieli (jednorazowy, automatyczny, stałe współczynniki), ustawienia wstępne, modele filtrów, stan i aktualizacja oprogramowania sprzętowego, zasilanie (sprawdzanie, uśpienie, wybudzanie), kalibracja obiektywu, karty sieciowe (sprawdzanie, szacowanie, naprawa), korekcja winietowania w plikach, odczyt DLS oraz sprawdzanie stanu hosta. Zobacz [Narzędzia do obsługi kamery](#camera-utilities).
> - **Nowość: pozostała część interfejsu CLI jako wywołania SDK**, dzięki czemu interfejs CLI, SDK oraz `chloros-mcp` oferują te same funkcje (interfejs CLI zachowuje wyłącznie logowanie, konfigurację urządzenia DAQ-E oraz bezpośrednie serwery DAQ ze względów bezpieczeństwa): kamery otwierane za pomocą jednego polecenia (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), pliki profili wyrównania (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indeksy (`index_catalog()`, `compute_index()`, `capture_live_index()`), plik główny PTP (`time_sync_status()` i podobne), folder projektu (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) oraz receptury (`run_project_recipe()`, własny program uruchamiający CLI). Dla czujników światła: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` i `light_sensor_reflectance()` (odbicie światła otoczenia i obiektu oraz wskaźniki dla dwóch czujników); dla kamery: `CameraSession.status()` oraz `capture_repeated()`; dla matrycy: `ArraySession.capability()` oraz `set_member_settings()` (ekspozycja, wzmocnienie itp. elementu matrycy). Zobacz [Macierze przejściowe, profile wyrównania i wskaźniki](#transient-arrays-alignment-profiles--indices), [Folder projektu i receptury](#project-folder--recipes) oraz [Sesje czujników DAQ](#daq-sensor-sessions).
> - **Ulepszone produkty indeksowe LATTICE** (NDVI i inne indeksy) z `process()`, we wszystkich trybach przetwarzania.
> - **Kamery LATTICE OCN: ulepszone produkty dotyczące promieniowania, odbicia i wskaźników w świetle słonecznym.** Korekcja floty jest stosowana po rozdzieleniu pasm — w trybie na żywo, w przechwyconych danych oraz w `process()`; kamery RGN pozostają bez zmian. Zobacz [Wyniki radiometryczne](#radiometric-output-lattice-multispectral-pipeline).
> - **Każdy zrzut zapisuje odczyt z modułu DAQ, z którego korzystał, podobnie jak robi to interfejs graficzny.** `CameraSession.capture()` oraz `ArraySession.capture()` / `continuous()` zapisują plik `.daq` oraz jego `.match.json` za każdym razem, gdy moduł DAQ jest powiązany lub umieszczony w puli, w tym również w przypadku przechwytywania wyłącznie w formacie surowym; w wersji 1.2.0 przechwytywanie surowe lub po usunięciu matrycy Bayera było zapisywane wyłącznie za pomocą `force_daq=True`. `daq=False` wyłącza tę funkcję, a `daq_tolerance_ms=` ustawia tolerancję dopasowania; oba wyniki wskazują, co zostało zapisane w `.daq_files` i `.daq_matches`. Przechwytywanie czeka teraz na następny odczyt z modułu DAQ, aby móc połączyć odczyty po obu stronach ekspozycji. Zobacz [Odczyt z modułu DAQ zapisywany podczas przechwytywania](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` zawiera klatki na żywo oraz przechwytywanie ciągłe po stronie serwera.** `grab()` i `frame_stream()` zwracają klatki numpy z jednego wyzwalacza (tak samo jak w przypadku API projektu `ArrayHandle.grab()` / `frame_stream()`), a `continuous()` zapisuje każdą zsynchronizowaną grupę w backendzie i zwraca [`ContinuousHandle`](#continuoushandle). Zobacz [Ramki na żywo we własnym potoku](#live-frames-in-your-own-pipeline).
> - **Tablica `grab()` jest przypisana do jednego wyzwalacza** (`ArrayHandle.grab()` oraz nowy `ArraySession.grab()`). W wersji 1.2.0 12 z 20 czterechkamery `ArrayHandle.grab()` obejmowały dwa wyzwalacze. Każda klatka zawiera teraz `group_id`, a tablica `frame_stream()` generuje jeden wynik dla każdego wyzwalacza. Zobacz [Jeden wyzwalacz na odczyt](#one-trigger-per-read--group_id).
> - **Klatki dla poszczególnych pasm i wyrównane:** `grab()` / `frame_stream()` pobierają `band=`, `align=`, `bit_depth=` oraz `refl_max=` na uchwytach kamery i macierzy. Zobacz [Eksportowanie pojedynczego pasma](#exporting-a-single-band--band).
> - **Nowe metody i argumenty `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (wraz z `CaptureResult.group_seq` i `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` łączy kolejne grupy i zatrzymuje się po zapełnieniu dysku.
> - **`ArraySession.record()` rejestruje dane przy zamkniętym interfejsie graficznym.** Gdy połączony widok interfejsu graficznego jest zamknięty, backend sam renderuje widok: tablicęNDVI lub indeks przekazany jako `expr=`. `RecorderHandle.stop()` generuje wyjątek `ChlorosConnectError` w przypadku nagrania, które zakończyło się niepowodzeniem (nie zapisano żadnej klatki) i zachowuje wynik zatrzymania w `.result` (opuszczenie bloku `with` nie powoduje wygenerowania wyjątku: sprawdź `rec.result`). Zobacz [`RecorderHandle`](#recorderhandle).
> - **Nowe metody `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` oraz, w przypadku ustawień własnych urządzenia DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` oraz `update_firmware()`. `record_stop()` zwraca `warning` oraz `gaps`. Ponowne uruchomienie, które zamknęło nagranie, nadaje mu nazwę w swojej odpowiedzi lub, gdy ponowne uruchomienie nie powiedzie się, w `ChlorosConnectError` (`recording_closed`; zob. [Ustawienia urządzenia DAQ-E](#daq-e-device-settings)). Zobacz [Metody `DAQSensorSession`](#daqsensorsession-methods).
> - **Urządzenie DAQ-E z puli, które przestaje przesyłać dane, jest ponownie podłączane przez backend** z zachowaniem dotychczasowych ustawień, a nagrywanie na nim jest kontynuowane w tym samym `.daq` z luką zaznaczoną w pliku; `record_stop()` zwraca luki jako `gaps`. W wersji 1.2.0 czujnik pozostawał nieaktywny, dopóki nie został odłączony i ponownie podłączony, co kończyło rejestrację. Zobacz sekcję [DAQ-E, który przestaje przesyłać dane](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` działa po zainstalowaniu SDK.** Koło (instalatori PyPI) zawiera teraz niezbędne moduły ładujące projektu; wcześniej `open_project()` generował błąd `ImportError`, wskazując na `project_cameras` / `project_sensors` w każdym SDK zainstalowanym za pomocą pip. `connect_all()` łączy zapisane kamery, tablice i czujniki światła poprzez uruchomiony backend (czujniki światła poprzez swoją pulę DAQ), a zapisane urządzenie, którego nie ma możliwości odnaleźć, jest wymienione w `errors` wraz z uzasadnieniem, zamiast być pomijane bez słowa. Zapisana tablica, która jest już uruchomiona w backendzie (w interfejsie graficznym, lub `chloros-cli project connect`) jest podłączana i pozostawiana w stanie działania, podczas gdy wcześniej zgłaszano to jako błąd („Kamera  znajduje się<sn> już w macierzy<id>”). Zobacz [Metody `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` i `process_lattice_capture()` wyszukują Chloros w systemie Windows.** Sprawdzają one, gdzie instalator umieścił backend (domyślnie `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, niestandardowy folder z własnego zapisu instalatora) i łączą się z już uruchomionym backendem, nie szukając go w ogóle. W wersji 1.2.0 generowały komunikat „Nie znaleziono backendu Chloros” na zainstalowanej maszynie, chyba że przekazano `backend_exe=`.
> - **Backend, który ulega awarii podczas uruchamiania, jest natychmiast zgłaszany.** Gdy `ChlorosLocal()` lub wywołanie smart-connect uruchamia backend, który kończy działanie przed udzieleniem odpowiedzi, wezwanie generuje w ciągu kilku sekund błąd `ChlorosBackendError` z `ChlorosLocal` oraz `ChlorosConnectError` z wywołań typu smart-connect, wraz z kodem zakończenia, wykonanym poleceniem oraz ostatnimi wierszami wyświetlonymi przez backend. Wersja 1.2.0 czekała do upływu 60-sekundowego limitu czasu uruchamiania i nie podawała przyczyny. Zobacz [Automatyczne uruchamianie backendu](#backend-auto-start).
> - **Backend nadal działa pomiędzy skryptami.** Opuszczenie bloku `with ChlorosLocal(...)`, powrót z `process_folder()` / `process_lattice_capture()` oraz zakończenie działania `chloros-mcp` nie zatrzymują już backendu uruchomionego przez SDK; kolejny skrypt ponownie z niego korzysta. Zatrzymuje się on na `logout()` (chyba że aplikacja Chloros jest otwarta lub trwa uruchomienie), na `shutdown_backend()` lub samoczynnie po 30 minutach, jeśli nic z niego nie korzysta (`CHLOROS_BACKEND_IDLE_MIN` w środowisku modułu zaplecza zmienia to zachowanie; `0`: nigdy).
> - **Moduł zaplecza uruchomiony przez SDK otrzymuje polecenie zamknięcia, a nie jest zabijany.** `shutdown_backend()` pozwala backendowi zwolnić kamery i czujniki światła oraz zakończyć wszelkie nagrania; jest on zamykany tylko wtedy, gdy nadal działa po upływie 10 s. W wersji 1.2.0 był on zamykany natychmiast (w systemie Windows – przymusowe zakończenie działania), co mogło spowodować, że czujnik światła DAQ podłączony do tego backendu przestał reagować. Backend, którego uruchomienie wykryło SDK, pozostaje nadal nienaruszony.
> - **W systemie Linux funkcja automatycznego uruchamiania nigdy nie uruchamia `backend_server.py` z bieżącego folderu.** W przypadku braku zainstalowanego modułu zaplecza skrypt uruchomiony w folderze, w którym znajdował się taki moduł, uruchamiał go; obecnie uruchamiany jest wyłącznie zainstalowany moduł zaplecza (lub, w przypadku `ChlorosLocal`, ten wskazany przez `backend_exe=`).
> - **`configure(indices=...)`**dodaje każdy preset do projektu dla pasm danej kamery, w tym do nowego projektu, i wyświetla ostrzeżenie (`UserWarning`) w przypadku nazwy, której nie może dodać.**`import_images()`** zgłasza nagrania `.daq` jako `scan_count` i nie wyświetla już ostrzeżenia o folderze przechwytywania LATTICE.
> - **`read_image_audit_tags()` działa po instalacji za pomocą pip.** Samodzielnie odczytuje metadane obrazu; w wersji 1.2.0 zwracał `{}` bez elementów wewnętrznych instalacji desktopowej znajdujących się w ścieżce Python.
> - **`capture()` bezpośrednio po `connect_array()`** czeka do 20 s na pierwszą grupę klatek macierzy zamiast zgłaszać błąd HTTP503 „Macierz wciąż się rozgrzewa”.
> - **`status()` zgłasza zatrzymanie.** Nowe pole `stalled` ma wartość True podczas działania pętli przechwytywania, ale nie publikuje żadnych danych, a `actual_fps` ma wówczas wartość 0. W wersji 1.2.0 `actual_fps` zachowywało swoją ostatnią wartość podczas zablokowania, więc zablokowana tablica wyświetlała `actual_fps > 0`, podczas gdy przechwytywanie kończyło się niepowodzeniem z kodem 503 „Brak nowej zsynchronizowanej grupy klatek”.
> - **Mniej niekompletnych zestawów klatek i błędów 503 podczas przechwytywania:** sygnał wyzwalający każdej kamery podrzędnej jest teraz oczyszczony z zakłóceń.
> - **Kompresowany transport 12-bitowy** (`connect_array(packed="auto")`) zwiększa częstotliwość klatek macierzy z pełnym czujnikiem o około jedną trzecią. Zobacz [Kompresowany transport 12-bitowy](#packed-12-bit-transport-packed).
> - **Ekspozycja matrycy w obszarze ROI:** mały obszar ROI nie ogranicza już ekspozycji elementu matrycy do okresu swobodnej pracy czujnika (4,9 ms przy rozdzielczości 768 × 304). Zobacz [Ekspozycja elementów matrycy](#exposure-on-array-members).
> - **Ekspozycja automatyczna nie obniża już automatycznej częstotliwości klatek matrycy.** Matryca zsynchronizowana kablem działa z częstotliwością, na jaką pozwala sieć i odczyt kamer; automatyczna ekspozycja wypełnia okres wynikający z tej częstotliwości, a następnie zwiększa wzmocnienie. Ręczna ekspozycja dłuższa niż ten okres spowalnia wyzwalanie, aby je dopasować, zamiast być odrzucona. Zobacz [Ekspozycja na elementach matrycy](#exposure-on-array-members).
> - **`read_nodes()`**dodaje `frame_values` i `notes` podczas działania automatycznej ekspozycji (zobacz [`read_nodes()` w trybie automatycznej ekspozycji](#read_nodes-under-auto-exposure--frame_values-and-notes)). Gdy kamera odrzuca ekspozycję lub wzmocnienie,**`set_settings()`** podaje zakres, który akceptuje, i przywraca tryb automatyczny, który został wyłączony dla tej wartości; `set_settings(target_brightness_auto=True)` to nowy kod.
> - **`orient=`** w `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` oraz `ArrayHandle.capture()` / `capture_smart()` obraca produkty zgodnie z orientacją aparatu. Zobacz [Metody `ArraySession`](#arraysession-methods).
> - **`DLS()` wymaga własnego adresu `mac_address` czujnika.** Nie ma już domyślnego adresu MAC (był to adres jednej jednostki); funkcja `DLS()` bez tego adresu generuje błąd `DLSError`. Zobacz [Wielkości pasma DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Instalator dla systemu Windows** oferuje opcje „Włącz długie ścieżki w systemie Windows” oraz „Wyklucz backend Chloros ze skanowania przez Microsoft Defender”, przy czym obie są domyślnie zaznaczone. Zobacz [Krok 1](#step-1--install-the-chloros-platform-package).
> - **Plik .deb dla systemu Linux** instaluje Python SDK dla systemu `python3` (wcześniej umieszczał on jedynie pakiet wheel na dysku), zwiększa rozmiar buforów odbiorczych gniazd jądra, których potrzebują strumienie z kamer, oraz dodaje usługę backendową działającą pod uprawnieniami użytkownika. Pakiet Jetson to `chloros_1.3.0_arm64_jp6.deb`, przeznaczony dla JetPack 6, wraz z biblioteką CUDA PyTorch dla JetPack. Zobacz [Krok 1](#step-1--install-the-chloros-platform-package) oraz [Krok 2](#linux-deb).

---

## Architektura w 60 sekund

SDK to cienka warstwa Pythona nakładająca się na backend Chloros (ten sam serwer Flask, z którego korzystają graficzny interfejs użytkownika i CLI). W celu automatyzacji należy zaimportować `chloros_sdk` i wywołać metody wysokiego poziomu; w tle każde wywołanie staje się żądaniem HTTP do lokalnego backendu na porcie 5000 — `http://127.0.0.1:5000/api/...` (celowo nie `localhost`, który w systemie Windows najpierw przekierowuje do `::1` i kosztuje około 2 s na żądanie w przypadku backendu obsługującego wyłącznie IPv4). Backend zarządza pulą sprzętu — kamerami, czujnikami DAQ, profilami wyrównania, buforami klatek — dzięki czemu skrypty SDK mogą współistnieć z interfejsem graficznym bez rywalizacji o porty szeregowe lub przepustowość karty sieciowej.

Dostępne są trzy interfejsy, z których będziesz korzystać:

1. **`ChlorosLocal` + funkcje swobodne** (`process_folder`, `process_lattice_capture`) — Potok przetwarzania obrazów. Przetwarzaj cały folder poprzez kalibrację / debayering / eksport indeksu za pomocą jednego wywołania w języku Python.
2. **Uchwyty Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otwórz trwałą sesję zaplecza dla sprzętu działającego na żywo. Ten sam przebieg „smart-prep” co w interfejsie graficznym: sonda sieciowa, automatyczny wybór warstwy, PTP, inicjowanie AE, konfiguracja wyzwalacza GPIO.
3. **`ChlorosProject` / `open_project`** — Wczytaj zapisany projekt (folder zawierający pliki `cameras.json` + `sensors.json` + `project.json`), podłącz jednocześnie kamery, matryce i czujniki światła, a następnie steruj przechwytywaniem za pomocą nazwanych uchwytów.

Powierzchnie 1 i 2 **automatycznie uruchamiają lokalny backend**, jeśli żaden jeszcze nie nasłuchuje (ten sam dołączony plik binarny, który uruchamia GUI/CLI) — dzięki czemu sam skrypt działa w nowej powłoce bez konieczności wcześniejszego uruchamiania backendu: `ChlorosLocal` (oraz `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` oraz `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` oraz `analyze_array_network()` nigdy się nie uruchamiają: wymagają działającego backendu. Należy przekazać `auto_start_backend=False`, aby wyłączyć tę funkcję (np. gdy wskazują na zdalny backend, który nigdy nie jest uruchamiany). Zobacz [Automatyczne uruchamianie backendu](#backend-auto-start). Surface 3 zachowuje się inaczej: `open_project()` nie przyjmuje parametru `auto_start_backend`, a `connect_all()` nigdy nie uruchamia backendu — raz sprawdza dostępność `http://127.0.0.1:5000`, a jeśli nic nie odpowiada, przechodzi na bezpośrednie (bez backendu) sterowanie urządzeniem `lattice_sdk`. Wymaga to środowiska uruchomieniowego Arena SDK oraz biblioteki OpenCV (`cv2`), które można zaimportować do Pythona, a których pakiet SDK wheel nie instaluje; bez nich każda zapisana kamera i tablica są zgłaszane jako błąd wraz z podaniem przyczyny. Czujniki światła w projekcie nie mają ścieżki bez modułu zaplecza: bez uruchomionego modułu zaplecza każdy zapisany czujnik jest zgłaszany jako błąd wraz z podaniem przyczyny (patrz [`SensorHandle`](#sensorhandle)). Tylko `proj.process()` i `stream(..., overlays=True)` leniwie tworzą obiekt `ChlorosLocal()` (który uruchamia się automatycznie).

Wszystkie trzy wymagają uwierzytelnienia: uruchom `chloros-cli login` raz na komputerze lub zaloguj się za pomocą graficznego interfejsu użytkownika na pulpicie. To, jaki błąd wywoła wywołanie bez ważnej sesji, zależy od interfejsu (zobacz [Wyjątki](#exceptions)): `ChlorosAuthenticationError` z `ChlorosLocal` (oraz `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` z wywołań typu smart-connect i ich sesji, `status: "error"` z `error_code` z `analyze_array_network()` oraz wpis pod `errors` w raporcie `connect_all()`.

Wymagania:
- Python 3.7+ (zgodnie z informacją w pakiecie; opracowano i przetestowano na wersji 3.10)
- Lokalnie zainstalowany Chloros Desktop (plik binarny backendu znajduje się w instalatorze)
- Aktywne konto Chloros+ na **dowolnym płatnym planie Chloros+ (Copper lub wyższym)**; bezpłatny plan**Iron**nie zapewnia dostępu do SDK ani CLI. Zasada ta jest egzekwowana**po stronie serwera****:** każde żądanie z flagą SDK/CLI musi zawierać zarówno aktywną sesję, jak i płatny plan. Plan niższy niż Copper zwraca błąd `403` wraz z `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` z `ChlorosLocal`, a `ChlorosConnectError` – z wywołań smart-connect). Wylogowany użytkownik otrzymuje zamiast tego kod `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` z `ChlorosLocal`, `ChlorosConnectError` z połączeń smart-connect) — te dwa kody są różne, ponieważ ponowne uruchomienie `chloros-cli login` naprawia błąd `401`, ale nie może naprawić błędu `403`, który wymaga aktualizacji planu.
- Korzystanie w trybie offline jest obsługiwane w ramach okresu karencji planu: poziom dostępu jest odczytywany z pamięci podręcznej walidacji serwerowej (5 minut) lub z pamięci podręcznej podpisanej licencji przypisanej do urządzenia (30 dni w przypadku planów miesięcznych, do wygaśnięcia subskrypcji w przypadku planów rocznych). Po upływie tego okresu karencji plan przechodzi w tryb bezpłatny, a dostęp do SDK i CLI zostaje wstrzymany do momentu, gdy urządzenie nawiąże połączenie z serwerem. Błąd `chloros-cli status` (`GET /api/license-status`) pozostaje dostępny w ramach bezpłatnego poziomu, dzięki czemu przyczyna jest widoczna.
- Windows 10/11 64-bitowy, **Ubuntu 22.04 LTS lub nowszy**lub Jetson (JetPack 6). Ubuntu 20.04**nie** jest obsługiwany: zależności `.deb` wynikają z bibliotek, z którymi łączy się backend, w tym `libc6 (>= 2.35)`, a system Focal zawiera bibliotekę glibc 2.31.

---

## Instalacja

Pythonowe SDK to cienka warstwa Pythona nakładająca się na backend Chloros. O ile nie korzystasz z backendu na innym komputerze (zobacz [Tryb zdalnego backendu](#remote-backend-mode-pip-only-host-via-tunnel)), musisz mieć **lokalnie zainstalowany pakiet Chloros na komputerze stacjonarnym** (instalator dla systemu Windows lub Linux `.deb`), który zawiera czujniki światła DAQ. Pakiet zawiera plik binarny backendu, środowisko uruchomieniowe SDK Arena dla kamer LATTICE oraz pakiety kalibracyjne.

Najnowsze pliki do pobrania: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Krok 1 — Zainstaluj pakiet platformy Chloros

#### Windows (.exe)

1. Pobierz plik `Chloros Installer 1.3.0.exe` ze strony pobierania.
2. Uruchom instalator i postępuj zgodnie z instrukcjami kreatora. Domyślna ścieżka instalacji to `C:\Program Files\Chloros\`.
3. Uruchom Chloros przynajmniej raz i zaloguj się na swoje konto Chloros+.

Instalator zezwala na przepuszczanie przez zaporę sieciową Windows w każdym profilu sieciowym wyłącznie dla backendu (`chloros-backend.exe`, nic więcej) na wszystkie profile sieciowe przez zaporę systemu Windows: przychodzący ruch UDP dla strumieni z kamer, PTP, mDNS oraz strumienia DAQ-E, a także przychodzący ruch TCP wyłącznie z lokalnej podsieci, dla połączenia, które urządzenie DAQ-E nawiązuje z powrotem do komputera podczas aktualizacji oprogramowania układowego ([`update_firmware()`](#daq-e-device-settings)). Program do odinstalowywania usuwa obie reguły. HTTP API serwisu pozostaje na `127.0.0.1` niezależnie od tego.

Grupa **Ustawienia systemowe** kreatora zawiera dwie opcje, z których obie są domyślnie zaznaczone:

| Opcja | Opis | Wyłączenie cichej instalacji |
| --- | --- | --- |
| **Włącz długie ścieżki w systemie Windows** | Pozwala na umieszczanie folderów projektów głębiej niż wynosi limit 260 znaków ścieżki w systemie Windows. Gdy opcja długich ścieżek jest wyłączona (ustawienie domyślne systemu Windows), folder projektu, który jest zbyt głęboko umieszczony dla danego produktu, zostanie odrzucony przed rozpoczęciem jakiegokolwiek przetwarzania — zobacz [Rozwiązywanie problemów](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Wyklucz backend Chloros z skanowania przez Microsoft Defender** | Zatrzymuje skanowanie w czasie rzeczywistymskanowanie w czasie rzeczywistym każdego pliku zapisanego przez kamerę. Przy wysokich częstotliwościach przechwytywania skanowanie pochłania około 30% przepustowości: zmierzono to dla obszaru zainteresowania (ROI) 512 w układzie 9 kamer — ciągłe przechwytywanie w formacie RAW osiągało 52,7 klatek/s ze skanowaniem i 77,7 klatek/s z wyłączonym backendem. | `/MERGETASKS="!defenderexclusion"` |

Ciągłe przechwytywanie zgłasza obciążenie programu Defender jako `defender_cores` (zobacz [`ContinuousHandle`](#continuoushandle)).

#### Linux, amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Zainstaluj za pomocą `apt install ./…`, a nie `dpkg -i`: apt instaluje również wymagania Python SDK, które pakiet zaleca, a nie wymaga (zobacz [Krok 2](#linux-deb)).

#### Linux, arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Dla Jetson Orin / Orin NX / Orin Nano z JetPack 6 (L4T R36). PyTorch jest tu kompilacją CUDA dla JetPack 6, więc przetwarzanie na GPU odbywa się na procesorze graficznym Orina; wykorzystuje biblioteki CUDA i cuDNN instalowane przez JetPack 6 (`sudo apt install nvidia-jetpack` na obrazie, na którym ich brakuje).

#### Linux: bufory odbiorcze i usługa zaplecza

Oba pakiety .deb zwiększają bufory odbiorcze gniazd jądra, których potrzebują strumienie z kamer LATTICE, do 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Przy domyślnych ustawieniach jądra pakiety z kamery są odrzucane w jądrze, a tablica traci klatki i częstotliwość odświeżania; `chloros-cli lattice network` zgłasza niski poziom buforów. Zobacz sekcję [Hosty Linux — bufory odbiorcze gniazd](cli-reference.md#linux-hosts--socket-receive-buffers) w dokumentacji CLI.

Na komputerze bez monitora uruchom backend jako usługę **pod swoim kontem użytkownika**, aby skrypt mógł wykryć uruchomiony proces (`connect_all()` nigdy nie uruchamia backendu):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend działa wtedy jako użytkownik: zapisywane przez niego dane przechwytywania należą do użytkownika i korzysta on z pliku `chloros-cli login`. Plik root `chloros-backend.service` jest nadal zainstalowany na maszynach, na których został włączony, ale każdy plik zapisywany przez backend działający jako root należy do użytkownika root. Zobacz sekcję [Uruchamianie modułu zaplecza jako usługi](cli-reference.md#running-the-backend-as-a-service-headless) w dokumentacji CLI.

Moduł zaplecza uruchamiany przez usługę jest współdzielony i nigdy nie jest zatrzymywany przez klienta: skrypty SDK oraz `chloros-cli` korzystają z niego tak, jak go zastaną (`ChlorosLocal.shutdown_backend()` zatrzymuje tylko backend, który sam uruchomił `ChlorosLocal`), a `GET /api/status` nazywa tę jednostkę jako `service`. Uruchom go ponownie za pomocą `sudo systemctl restart chloros-backend@$USER.service`. Aktualizacja pakietu zatrzymuje usługi backendowe na czas instalacji, więc uruchomiony skrypt traci dostęp do kamer i czujników, a następnie ponownie uruchamia wszystkie włączone urządzenia; usunięcie pakietu zatrzymuje je i wyłącza te, które były włączone. W danym momencie działa tylko jeden backend, więc włącz tylko jedno urządzenie: drugie nie uruchomi się i pozostaje w stanie `failed` (kod wyjścia 98, nie uruchamia się ponownie), a jego `systemctl status` wyjaśnia dlaczego. Aby przełączyć się między jednostkami, wyłącz tę, której nie używasz.</id></sn>

### Krok 2 — Zainstaluj Python SDK

**Instalator Chloros zawiera odpowiedni pakiet SDK w formacie wheel.** Każdy instalator dla systemu Windows oraz pakiet .deb dla systemu Linux umieszcza na dysku plik `chloros_sdk-X.Y.Z-py3-none-any.whl`, który dokładnie odpowiada wersji interfejsu graficznego, CLI oraz zaplecza. Nie musisz śledzić zmian na PyPI, aby zachować synchronizację.

#### Windows

Instalator automatycznie uruchamia plik `pip install` w oparciu o dołączony plik wheel, korzystając z systemowej wersji Pythona (preferowany jest launcher `py.exe`, w razie potrzeby używany jest `python -m pip`). Nie trzeba nic robić — po pomyślnej instalacji `import chloros_sdk` działa w Twoim środowisku Pythona. Jeśli na komputerze nie ma Pythona, instalator po cichu pomija ten krok, a GUI i CLI nadal działają.

#### Linux (.deb)

Oba pliki .deb (x86_64 i Jetson) instalują SDK dla systemu `python3`, więc `import chloros_sdk` działa po instalacji, bez pip i bez połączenia z siecią. Pakiety (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` oraz programy ładujące projekty) trafiają do `/usr/lib/python3/dist-packages`, a polecenie SDK `chloros-mcp` ([serwer MCP](#mcp-server-ai-assistants)) do `/usr/bin`; wszystkie te elementy należą do pakietu: aktualizacja je zastępuje, a usunięcie pakietu powoduje ich usunięcie.

Własne wymagania SDK, `python3-numpy` i `python3-requests`, są **zalecane, a nie wymagane** (interfejs graficzny, CLI i zaplecze są samowystarczalne): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` na Jetsonie) instaluje je, natomiast `dpkg -i` tego nie robi. Wynik instalacji zawiera informację o wersji SDK, którą importuje `python3`, o wszelkich brakujących wymaganiach, oraz o pliku `chloros_sdk` znajdującym się gdzie indziej w ścieżce `python3`, który zastępuje kopię pakietu (np. zainstalowaną wcześniej za pomocą `pip install --user`).

W środowisku virtualenv, które nie ma dostępu do pakietów systemowych, zainstaluj w nim dołączony plik wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

W przypadku wdrożeń Jetson w trybie air-gapped proces ten przebiega całkowicie w trybie offline — SDK i plik wheel znajdują się już na dysku.

#### Publiczne repozytorium PyPI

W przypadku hostów korzystających wyłącznie z pip (bez zainstalowanego pakietu Chloros; SDK steruje wówczas backendem na innym komputerze, zobacz [Tryb zdalnego backendu](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI udostępnia najnowszą stabilną wersję. Należy utrzymywać tę samą wersję SDK, co backend Chloros, który jest przez niego sterowany (zobacz [Wersjonowanie i kompatybilność](#versioning--compatibility)). Narzędzie pomocnicze DAQ-M `DLS` wymaga również `bleak`: `pip install "chloros-sdk[camera]"` (lub `pip install bleak`).

#### Sprawdź

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Wymagana subskrypcja Chloros+.** Wszystkie wywołania SDK wymagają aktywnego loginu do Chloros+. Uruchom `chloros-cli login user@example.com 'YourPassword'` raz na każdym komputerze; dane uwierzytelniające są buforowane w `~/.chloros/`.

### Czy potrzebuję pakietu Desktop?

Sam pakiet pip **nie** wystarcza w przypadku większości procesów roboczych. Oto, czego wymaga każda platforma SDK:

| Platforma SDK | Czy wymaga pakietu Desktop? | Dlaczego |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Tak**(aby uruchomić jeden)**/ Nie** (backend jest już dostępny) | Wykorzystaj uruchomiony backend (aplikacji desktopowej lub innego uruchomionego skryptu); w przeciwnym razie automatycznie uruchom zainstalowany: `/usr/lib/chloros/chloros-backend` (system Linux) lub `<install folder>\backend\chloros-backend.exe` (system Windows; domyślnie `C:\Program Files\Chloros\backend\`, niestandardowy folder instalacyjny jest odczytywany z danych instalatora). Przekaż `backend_exe=`, aby nadać mu własną nazwę. |
| `chloros-mcp` ([serwer MCP](#mcp-server-ai-assistants)) | **Tak**(aby uruchomić serwer)**/ Nie** (serwer zaplecza jest już dostępny) | Oparty na `ChlorosLocal`, więc tak samo jak w wierszu powyżej. |
| `read_image_audit_tags` | **Nie** | Odczytuje metadane przetworzonego obrazu (tylko biblioteka standardowa Pythona). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Tak**(lokalnie)**/ Nie**(zdalnie) | Czyste klienty HTTP działające przez backend. Backend lokalny → wymagany pakiet na komputerze stacjonarnym. Backend zdalny → `backend_url=`**przez tunel** (zobacz Tryb zdalnego backendu — dostarczone backendy wiążą się wyłącznie z pętlą zwrotną). |
| `ChlorosProject` / `open_project` | **Tak** | Obsługuje zapisane projekty za pośrednictwem backendu. |
| Bezpośrednie klasy LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Tak** | Wymaga środowiska uruchomieniowego Arena SDK, które jest dostarczane w pakiecie na komputery stacjonarne, oraz biblioteki OpenCV (`cv2`) w środowisku Python, której nie instaluje ani pakiet na komputery stacjonarne, ani SDK (`pip install opencv-python`). W przeciwnym razie `CAMERA_AVAILABLE` jest równe `False` podczas importu. `DLS` wymaga również `bleak`. |

### Tryb zdalnego backendu (host obsługujący wyłącznie pip, poprzez tunel)

> **Zainstalowany backend nie jest dostępny w sieci LAN.** Nasłuchuje on
> wyłącznie na pętli zwrotnej (`127.0.0.1` / `::1`), więc `backend_url="http://<lan-ip>:5000"`
> **nie może połączyć się z zainstalowanym Chlorosem**. Aby sterować backendem na innym
> komputerze, należy samodzielnie przekierować jego port pętli zwrotnej i skierować SDK na
> tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Hosty bez monitorów / CI / robotyki mogą utrzymywać jedną maszynę z pełną instalacją pulpitu jako „serwer Chloros”, a `pip install chloros-sdk` wszędzie indziej — jednak transport między nimi odbywa się za pośrednictwem wspomnianego powyżej tunelu skonfigurowanego przez użytkownika, a nie bezpośredniego URL w sieci LAN.

> **`ChlorosLocal` na hoście obsługującym wyłącznie pip.** `ChlorosLocal(backend_url=BACKEND)` łączy się z serwerem zaplecza, do którego ma dostęp bez instalacji środowiska graficznego: szuka lokalnego pliku binarnego serwera zaplecza tylko wtedy, gdy musi go URUCHOMIĆ (brak odpowiedzi i `auto_start_backend=True`). Do wersji 1.2.0 program najpierw szukał pliku binarnego i generował błąd `ChlorosBackendError` („Nie znaleziono backendu Chloros…”) na hoście obsługującym wyłącznie pip, nawet jeśli backend był dostępny. Foldery przekazane przez użytkownika (`process_folder`, `import_images`) są sprawdzane na tym komputerze i odczytywane przez backend, więc w przypadku zdalnego backendu muszą one wskazywać ten sam folder na obu maszynach, np. na dysku współdzielonym.

### Przebieg pracy wyłącznie z DAQ (host obsługujący wyłącznie protokół pip)

Sam program `pip install chloros-sdk` nie może sterować czujnikiem DAQ: czujnik DAQ jest zawsze sterowany za pośrednictwem backendu Chlorosa przy użyciu programu `connect_daq_sensor()`:

- **Czujnik na tym komputerze:** zainstaluj tutaj pakiet Chlorosa na komputery stacjonarne. `connect_daq_sensor()` uruchamia swój backend, jeśli żaden nie jest uruchomiony.
- **Czujnik na innym komputerze:** ten komputer wymaga pakietu desktopowego; dostęp do jego backendu uzyskuje się poprzez tunel w [trybie zdalnego backendu](#remote-backend-mode-pip-only-host-via-tunnel). Backend otwiera czujnik na swoim własnym komputerze, więc urządzenie DAQ-U jest podłączone do tego komputera, urządzenie DAQ-M znajduje się w zasięgu Bluetooth tego komputera, a urządzenie DAQ-E znajduje się w sieci, do której ma dostęp.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

W obu przypadkach backend egzekwuje plan „Chloros+”: zaloguj się na komputerze, na którym działa backend (`chloros-cli login` lub aplikacja desktopowa) przy użyciu dowolnego płatnego planu (Copper lub wyższego). Wylogowany backend oraz plan niższy niż Copper powodują wygenerowanie błędu `ChlorosConnectError` zawierającego przyczynę związaną z backendem (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Szybki start

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indeks API najwyższego poziomu

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Przetwarzanie obrazów — `ChlorosLocal`

Główna klasa potoku. Uruchamia backend podczas konstruowania (jeśli żaden nie odpowiada i `auto_start_backend=True`), tworzy i konfiguruje projekty, monitoruje postęp, zwraca podsumowania po zakończeniu działania.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metody

| Metoda | Opis |
| --- | --- |
| `create_project(project_name, camera=None)` | Utwórz nowy projekt (opcjonalnie z szablonem kamery, takim jak `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Otwiera zapisany projekt (jego folder lub nazwę w folderze projektów) i ustawia go jako otwarty projekt. Ścieżki przeniesionego projektu są przekierowywane do folderu, w którym się obecnie znajduje; `relink` określa nazwę folderu, w którym znajdują się obecnie pozostałe brakujące pliki. Zwraca odpowiedź backendu za pomocą `source_files`; wywołuje wyjątek `ChlorosProjectFilesMissingError`, jeśli nie można znaleźć żadnego z nich. Zobacz **Przeniesiony projekt** poniżej. |
| `import_images(folder_path, recursive=False)` | Importuje obrazy w formatach RAW/TIF/JPG/DNG **oraz `.daq` zapisy z czujnika światła**. Zwraca `count` (obrazy) oraz `scan_count` (zapisy). Wyświetla ostrzeżenie tylko wtedy, gdy w folderze nie ma ani jednego, ani drugiego. |
| `export_light_sensor(daq=True, csv=True)` | Zapisuje skalibrowane pliki `.daq` + `.csv` dla każdego zapisu z czujnika światła w projekcie, do pliku `<project>/Light Sensor/`. Zobacz [Zapisy z czujnika światła](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Ustaw pokrętła przetwarzania. `indices` to nazwy ustawień wstępnych (`NDVI`, `GNDVI`, `OSAVI`, …) – każda z nich została dodana do projektu dla pasm danej kamery; nazwa, która nie jest ustawieniem wstępnym lub której pasma nie mogą utworzyć (`NDRE` wymaga pasm RE i NIR, których żadna pojedyncza kamera nie zapewnia), otrzymuje nazwę `UserWarning` i nie ma obrazu indeksowego. Nazwa `daq_log_path` odnosi się do zapisu z czujnika światła (plik `.daq` lub `.csv` albo folder zawierający takie pliki), z którym porównywana jest każda klatka – zarówno w LATTICE, jak i w Survey3 – zamiast nagrań zaimportowanych wraz z obrazami; jeśli ścieżka nie istnieje lub nie zawiera żadnego zapisu, wykorzystywane są zapisy zaimportowane. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Uruchom potok. Zwraca plik `{"status": "complete", "async": False}`, oraz klucz `summary`, jeśli backend go udostępnia — zobacz [Podsumowanie po uruchomieniu i wskazówki](#post-run-summary--hints) — `export_check_failure`, gdy nie udało się zweryfikować eksportów z przebiegu, oraz `image_failure`, gdy utracono obraz. `wait=False` zwraca jednocześnie `{"status": "started", "async": True}` jednocześnie. |
| `get_config()` / `get_status()` / `status()` | Sprawdź stan zaplecza. |
| `logout()` | Wylogowanie: czyści buforowane dane uwierzytelniające i zwalnia gniazdo urządzenia tego komputera (`device_deregistration` w wyniku). Trwające przetwarzanie nie zostaje zatrzymane: kończy się w trybie, w którym zostało uruchomione (`run_in_progress: true` i `run_mode` w wyniku), a kolejne przetwarzanie odbywa się w trybie standardowym. |
| `shutdown_backend()` | Zatrzymaj backend, jeśli ta instancja go uruchomiła; backend, który został wykryty jako uruchomiony, pozostaje bez zmian. Pozostawienie bloku `with` lub poddanie instancji czyszczeniu pamięci nie powoduje jego zatrzymania: backend nadal działa dla następnego skryptu i zatrzymuje się samoczynnie po 30 minutach, jeśli nic z niego nie korzysta. Backend otrzymuje polecenie zamknięcia, więc najpierw zwalnia swoje kamery i czujniki światła oraz kończy wszelkie nagrania; zostaje zakończony tylko wtedy, gdy nadal działa po upływie 10 sekund. |
| `discover_cameras()` | Wykryj kamery LATTICE **za pośrednictwem backendu tej instancji** (`/api/camera/discover`). Zwraca listę słowników (`serial`, `model`, `ip`, …) — o takim samym kształcie, jaki widzą GUI i CLI. Pusta lista, jeśli nie znaleziono żadnych kamer lub backend jest niedostępny. |
| `camera_capture(output_dir, format="tiff", **settings)` | Przechwytuje pojedynczą klatkę**za pośrednictwem zaplecza**(uruchamianego automatycznie przez ten uchwyt), tak aby została ona przygotowana w taki sam sposób jak w GUI lub CLI (domyślnie 12 bitów, ponowne wykorzystanie puli, osadzone metadane kalibracyjne). Określ cel za pomocą `serial=` lub `device_index=`; przekaż `exposure`/`gain`/`pixel_format`/`preset` jako `**settings`. Zwraca słownik metadanych starszego typu (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Generuje klatki podglądu złożone z nakładek z połączonej kamery — lekki klient MJPEG działający na trasie `/api/camera/<serial>/stream-annotated` serwera (zebra / siatka / krzyżyk / histogram / peaking / punkt rysowany po stronie serwera). `decode=True` generuje tablice BGR i wymaga biblioteki OpenCV (`cv2`), którego SDK nie instaluje; `False` generuje surowe bajty JPEG. Dostępne również na poziomie projektu jako `ChlorosProject.stream(overlays=True)`. |

**Uruchomienie w trakcie działania zachowuje swój projekt.** Podczas trwania operacji przetwarzania — tego skryptu, aplikacji, `chloros-cli process` lub `chloros-mcp` — backend odmawia tworzenia, otwierania, importowania do projektu lub zmiany jego ustawień przetwarzania (kod błędu HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` i `import_images()` generują błąd `ChlorosProcessingError`, a `configure()` generuje błąd `ChlorosConfigurationError`, a `process()` z trybem innym niż aktualnie uruchomiony generuje błąd `ChlorosProcessingError` (uruchomienie zachowuje tryb, w którym zostało rozpoczęte), każde z komunikatem backendu. Zapisany w projekcie zapis dołącza się do niego po zakończeniu uruchomienia (wskazuje na to odpowiedź `project_add`). Należy poczekać na zakończenie uruchomienia (`get_status()`), a następnie kontynuować.**Przeniesiony projekt.** Plik `project.json` projektu nazywa każdy plik źródłowy (każdy obraz i zapis z czujnika światła) według jego pełnej ścieżki, więc przeniesienie folderu projektu — lub całego folderu „projects” na inny dysk — powoduje, że pliki te nadal odwołują się do starej lokalizacji. Każde otwarcie projektu naprawia to, co może samodzielnie: plik, który znajdował się w folderze projektu, jest wyszukiwany w folderze, w którym projekt się obecnie znajduje, a każda znaleziona tam ścieżka jest przekierowywana i zapisywana (`project.json` jest najpierw archiwizowany jako `project.json.before-relink-<date>-<time>`). Ścieżka jest przekierowywana wyłącznie do pliku, który faktycznie się tam znajduje. Pliki, które znajdowały się poza folderem projektu — sesja hubu, import z opcji „Dodaj folder” — są wykrywane wyłącznie w `relink=<the folder they are in now>` (lub w dowolnym folderze nadrzędnym). `open_project()` zwraca `source_files`: `total`, `found`, `missing`, `missing_roots` (foldery, w których znajdowały się brakujące pliki), `missing_outside_project` oraz `relink` (co zostało przekierowane, skąd i dokąd). Projekt, w którym brakuje niektórych plików, otwiera się wraz z pozostałymi; projekt, w którym nie znaleziono żadnych plików, nie zostaje otwarty, a `open_project()` generuje błąd `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). `project open --relink` i `chloros-mcp` w CLI`open_project` (`relink_folder`) działają w ten sam sposób.

Użyj jako menedżera kontekstu w celu zagwarantowanego oczyszczenia:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Rejestracje czujnika światła — skalibrowane `.daq` + `.csv`

Dane z DAQ-U / DAQ-M / DAQ-E można rejestrować **bez** pakietu kalibracyjnego. Tak właśnie
działają domyślnie publiczne rejestratory [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): zapisują surowe odczyty czujników i oznaczają
plik tak, aby Chloros pobrał fabryczną kalibrację danego czujnika **na podstawie numeru seryjnego** — najpierw z lokalnej pamięci podręcznej
, a następnie z chmury MAPIR — i stosuje ją podczas importu.

Chloros zapisuje wynik z powrotem jako dwa produkty na każdy zapis, pod
`<project>/Light Sensor/`:

| Produkt | Co to jest |
| --- | --- |
| `<name>_calibrated.daq` | Archiwum nadające się do ponownego przetworzenia — ten sam schemat co w przypadku nagrania na żywo, z tym że teraz zawiera informację o pakiecie, który je wygenerował. Ponowny import **nie** powoduje jego drugiej kalibracji. |
| `<name>_calibrated.csv` | Natężenie promieniowania spektralnego w W/m²/nm w siatce długości fal czujnika, jeden wiersz na odczyt, plus kolumny fotometryczne (moc całkowita, luks fotopowy/skotopowy, PPFD i jego podział na niebieski/zielony/czerwony, długość fali szczytowej). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Tylko czujniki bez pakietu (DAQ-A).** Surowe wartości liczbowe czujnika spektralnego — *nie* natężenie promieniowania. Zobacz poniżej. |

`<name>` to nazwa pliku danego zapisu. Dwa zapisy w różnych folderach mogą mieć wspólną
jedną — odczyt z danego ujęcia (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) zapisany w
tej samej milisekundzie przez dwie kamery — a każda z nich nadal otrzymuje własne wyniki: pierwsza zachowuje
`<name>`, a następny dodaje foldery, które go odróżniają **po** nazwie
(`<name>__<capture>_daq_calibrated.daq`), więc data i godzina nagrania nadal znajdują się na początku.

`process()` wykonuje ten eksport jako jeden ze swoich etapów. Nie wymaga on **żadnych** obrazów:
samodzielnie latający czujnik światła stanowi pełnoprawny proces roboczy, a taki projekt z założenia nie zawiera
żadnych obrazów.

**Zapisy z DAQ-A są eksportowane jako surowe zliczenia.** Rodzina DAQ-A powstała przed wprowadzeniem systemu
pakietów przypisanych do poszczególnych numerów seryjnych i nie ma żadnego pakietu do pobrania — jest ona kalibrowana w terenie względem
celu odbicia, dlatego nigdy nie potrzebowała takiego pakietu. Te zapisy są eksportowane
z przedrostkiem `_raw` zamiast `_calibrated`: jest to inna nazwa pliku, a nie flaga
wewnątrz pliku, ponieważ nazwa musi pozostać nienaruszona podczas wysyłania e-mailem jako sama nazwa. Nagłówek
Nagłówek `.csv` zawiera wartość `raw spectral sensor counts (NOT irradiance)` i ostrzega, że
wartości są porównywalne **w obrębie** pliku — dokładnie to, do czego służy kalibracja oparta na wzorcu
— a nie między czujnikami. Kolumny fotometryczne zależne od mocy (moc całkowita,
luks fotopowy/skotopowy, PPFD) zwracają wartość **NULL** zamiast być zintegrowane na podstawie liczb zliczeń.

Urządzenie DAQ-U / DAQ-M / DAQ-E, którego pakietu po prostu nie udało się pobrać, jest nadal **pomijane**,
a nie zapisywane w postaci surowej: w tym przypadku pakiet istnieje, a „ponowne połączenie i ponowne przetworzenie” to realna rada.

Starsze zapisy **v1.01 / v1.02** (zapisywane przez DAQ-A-SD) nie zawierają epoki dla poszczególnych odczytów,
a jedynie czas zapisu pliku. Moduł dopasowujący obraz↔promieniowanie padające nadal je odrzuca — dopasowanie
klatki z czasem zapisu byłoby niewidocznym błędem — jednak eksportator je odczytuje, a
plik CSV wyświetla `clock=daq_created_on`, dzięki czemu produkt wskazuje, na jakim zegarze się znajduje.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Zapis, którego pakiet kalibracyjny nie może zostać pobrany (tryb offline lub czujnik bez
kalibracji w pliku), jest zgłaszany pod numerem `skipped` **wraz z przyczyną**. Nigdy nie jest
zapisywany jako „skalibrowany” plik zawierający surowe dane liczbowe — należy połączyć się z Internetem i
uruchomić proces ponownie, a eksport zostanie zakończony.

### Wywołania zwrotne dotyczące postępu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Podsumowanie po zakończeniu i wskazówki

Po zakończeniu `process()` pobiera `GET /api/processing-summary` i dołącza treść jako `result["summary"]`. Pobieranie odbywa się na zasadzie „best-effort” i nigdy nie blokuje pomyślnego zwrotu — jeśli podsumowanie jest niedostępne, `process()` przechodzi na zwykły format `{"status": "complete", "async": False}`. Każdy wpis w `summary["hints"]` — pełne zdania zawierające sugerowane działania naprawcze, np. dlaczego uruchomienie dało wynik zerowy — jest również ponownie generowany jako `UserWarning` w języku Python, więc uruchomienia z wynikiem zerowym są samodiagnostyczne, nawet jeśli nigdy nie sprawdzisz słownika:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` to część nadająca się do odczytu maszynowego:

| Klucz | Co liczy |
| --- | --- |
| `models` | Grupy kamer w przebiegu. |
| `images_in_groups` | Obrazy źródłowe z tych grup. |
| `targets_found` | Wykryte cele odbicia. |
| `images_calibrated` | Obrazy, dla których w przebiegu wygenerowano produkt odbicia. |
| `images_failed` | Obrazy, które nie przeszły pomyślnie i nie zostały wyeksportowane. |
| `exported_files` | **Pliki produktów obrazowych utworzone przez przebieg**: każdy produkt (po usunięciu matrycy, podgląd, promieniowanie, współczynnik odbicia, indeks, rendery celów) w każdym formacie. |
| `reflectance_files` | Produkty współczynnika odbicia spośród nich (`Reflectance_Calibrated_Images`), niezależnie od tego, czy zostały skalibrowane na podstawie natężenia promieniowania docelowego, czy natężenia promieniowania z czujnika światła (DAQ). Przebieg bez obiektu docelowego, który mimo to zapisał odbicie, zawiera odpowiednią informację w podpowiedzi; tylko w przypadku przebiegu, który nie zapisał żadnych danych, pojawia się komunikat o braku wyników dotyczących odbicia. |
| `clamp_masks_written` | [Maski ograniczenia rozdzielania](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) zapisane obok nich — są to elementy towarzyszące, a nie produkty, więc nie znajdują się w `exported_files` (CLI liczy je oddzielnie w ten sam sposób). |
| `captures_left_out` | Surowe klatki w folderze `captures` projektu, których to uruchomienie nie zarejestrowało. Przebieg czeka na początku na dane zarejestrowane przed nim, więc wartość ta wynosi 0, chyba że podczas przebiegu zapisano dane lub wygasł czas oczekiwania. `summary["captures_left_out_note"]` określa, co z nimi zrobić. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Zapisy z czujników światła, celowo zliczane oddzielnie — pochodzą one z innego etapu i występują w przebiegach, w których w ogóle nie ma obrazów, więc uwzględnienie ich sprawiłoby, że przebieg zawierający wyłącznie dane DAQ wyglądałby tak, jakby eksportował obrazy. |

Oprócz nich: `summary["output_dirs"]` (każdy katalog, do którego zapisano dane),
`summary["light_sensor_export"]`, `summary["stopped"]` (ma wartość true, gdy użytkownik przerwał
przebieg, dzięki czemu częściowe wyniki nie są interpretowane jako zakończony przebieg, który dał zbyt małą liczbę wyników) oraz
`summary["groups"]` (podział na grupy).

`exported_files` jest rejestrowany przez potok **w trakcie zapisu**, a nie skanowany później z
obiektów obrazu projektu. Strategie równoległe i GPU tworzą własne obiekty obrazu
(w podprocesach roboczych dla ścieżek GPU), więc stare skanowanie zgłaszało
`0 file(s) written` dla każdego takiego przebiegu, a następnie generował wskazówkę o zerowej liczbie eksportów — w przypadku przebiegów,
w których wszystko działało poprawnie. Jeśli tworzysz skrypt oparty na tej liczbie, prawidłowo działający przebieg równoległy będzie teraz
zwracał liczbę niezerową.

Pominięcia wykrywane przez czujnik podają przyczynę faktycznie ustaloną przez czytnik dla każdego pliku —
nieczytelny schemat, brakujący pakiet, błąd zapisu — **po deduplikacji**, więc dwadzieścia plików
pominiętych z jednej przyczyny jest odczytywanych jako jedna przyczyna, a nie dwadzieścia jej powtórzeń.

> **Błąd `process()` nie jest generowany, gdy przebieg nie tworzy żadnych obrazów.** Jest to jedno z dwóch miejsc, w których SDK i
> CLI celowo się różnią: `chloros-cli process` traktuje sytuację „zaprošono o produkty, ale żadne nie zostały
> zapisane” jako błąd i kończy działanie z wynikiem niezerowym, podczas gdy SDK wraca normalnie i zgłasza
> ten stan za pośrednictwem `summary` / wskazówek. Jeśli Twój potok powinien zatrzymać się w przypadku pustego przebiegu, sprawdź to
> samodzielnie — przeanalizuj `summary` (lub policz pliki w folderze projektu), zamiast polegać na
> braku wyjątku. Typowymi przyczynami są: folder wejściowy, który nie został rozpoznany jako
> obszar przechwytywania, oraz produkty pominięte jako nieodpowiednie dla obecnych kamer (np. promieniowanie z kamer obsługujących wyłącznie RGB
>).
>
> `process()` nie jest generowany również wtedy, gdy nie można zweryfikować eksportów danego przebiegu: wyświetla ostrzeżenie i ustawia
> `result["export_check_failure"]`. Produkty są zapisywane. Aby zatrzymać proces w tym momencie, sprawdź ten klucz. Tak samo jest w przypadku przebiegu, w którym
> utracono obraz: wyświetla ostrzeżenie i ustawia `result["image_failure"]`, przy czym `chloros-cli process`
> zwraca wynik niezerowy.

### Funkcje pomocnicze

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Każdy przebieg działa w ramach własnego `with ChlorosLocal(...)`; backend, który musiał uruchomić, nadal działa po jego zakończeniu, na następne wywołanie, a zatrzymuje się samoczynnie po 30 minutach, jeśli nic z niego nie korzysta. Żadna z nich nie przyjmuje `auto_start_backend`, `radiometric_output` ani `daq_log_path`; w tych przypadkach należy użyć `ChlorosLocal` oraz `configure()`. `process_lattice_capture()` nie akceptuje `camera`, `debayer` ani `ppk`: pomija wykrywanie panelu i wykorzystuje standardowy algorytm debayera.

### Obsługiwane wartości

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Wynik radiometryczny (potok wielospektralny LATTICE)

Poziom eksportu wielospektralnego (M3C/M3M) potoku `process` — `reflectance` (domyślnie), `radiance`, `sensor-response` lub `all` (każdy odpowiedni tryb dla każdego obrazu) — określa produkty projektu **„Eksport promieniowania”**i**„Eksport odbicia”****: `radiance` eksportuje wyłącznie promieniowanie, `reflectance` wyłącznie odbicie, `sensor-response` żadnego z nich (tylko produkty po usunięciu efektu Bayera i podgląd), `all` — oba. Produktów po usunięciu efektu Bayera i podglądu nie można zmieniać na tym poziomie. `configure()` ma do tego dedykowane słowo kluczowe:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Zaawansowana metoda awaryjna — zapisywanie produktów za pomocą `custom_settings` — nadal działa, ale należy pamiętać, że zastępuje ona cały blok ustawień (zobacz ostrzeżenie poniżej). Zapisz klucze `"Export <product>"`: ustawienia własnego projektu `"Export radiance"` / `"Export reflectance"` mają pierwszeństwo przed starszym kluczem `"Radiometric output"`, więc sam ten klucz niczego nie zmienia.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (ustawienie domyślne) dzieli promieniowanie kamery przez **dane DAQ o dopasowanym znaczniku czasu**, ustalane automatycznie na podstawie zarejestrowanego klucza `.daq` (DAQ-U/M/E)**lub natywnego dla DAQ-M pliku `.csv`**znalezionego wraz z obrazami; wszelkie brakujące lokalnie pakiety kalibracyjne dla poszczególnych kamer lub urządzeń DAQ są**automatycznie pobierane z AWS** przy pierwszym użyciu. Interfejs CLI udostępnia to zgodnie zprzełączników typu produktu w `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Czyj to odczyt?**Odczyt zapisany w elemencie `.daq` to odczyt z kamery (kamer), którą (które) opisuje — wymienionej w pliku towarzyszącym `<file>.daq.cameras.json` obok niego (przechwyty pojedyncze, zsynchronizowane i tablicowe; surowa seria podaje odczyt każdego elementu w swoim manifeście) — i jest dopasowany**wyłącznie do obrazów z tych kamer**. Samodzielna kamera bez własnego odczytu nie korzysta zatem już z DAQ innej kamery; należy przypisać do niej DAQ lub zarejestrować własny. Własny zapis DAQ (`chloros-cli daq pool-record` lub nagranie rozpoczęte w interfejsie graficznym) lub dowolny plik `.daq` bez tego pliku towarzyszącego (w tym starsze przechwyty) nadal obsługuje każdą kamerę.**Kamery OCN w świetle słonecznym.** Każdy pakiet kalibracyjny OCN jest tworzony w świetle wąskopasmowym, z jedną diodą LED na pasmo, podczas gdy światło słoneczne miesza pasma. Chloros 1.3.0 stosuje zbiorczą korektę opartą na świetle słonecznym po rozdzieleniu pasm, co poprawia produkty OCN dotyczące radiancji, odbicia i indeksu w świetle słonecznym. Działa wszędzie tam, gdzie odbywa się rozdzielanie pasm — w `process()`, podczas przechwytywania, w `grab()` / `frame_stream()` oraz w podglądzie na żywo — dla pakietów OCN, które to obsługują; kamery RGN pozostają bez zmian. Obrazy OCN zarejestrowane przy użyciu wcześniejszej wersji również otrzymują tę korektę, jeśli są przetwarzane na komputerze, na którym zachowano kalibrację używaną podczas ich wykonania (Chloros zachowuje kalibrację kamery po jej podłączeniu); w przeciwnym razie są przetwarzane tak jak dotychczas. Wartości promieniowania, odbicia i wskaźnika OCN różnią się zatem od tych z wersji 1.2.0.

> `custom_settings` **zastępuje** cały blok obliczonych ustawień (zgodnie z założeniami pomija inne słowa kluczowe oraz walidację `configure()`). Korzystając z niego, należy uwzględnić wszystkie kluczowe słowa `Project Settings`, które są istotne, tak jak w powyższym przykładzie.

#### Odczytywanie pikseli indeksowych (wskaźnik NDVI i inne wskaźniki wegetacji)

**Raster indeksu całkowitoliczbowego to rozciągnięcie kontrastu dla poszczególnych obrazów, a nie mapa w zakresie od −1 do 1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` oraz `JPG (8-bit)` — w tym 8-bitowy plik TIFF, w którym zapisane jest 8-bitowe ujęcie — przechowują

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

gdzie `IndexMin` / `IndexMax` to **najniższa i najwyższa skończona wartość indeksu tej klatki**. DN 0 nie jest równe −1, a pełna skala nie jest równa +1.

Program Chloros oznacza oba punkty końcowe, dzięki czemu odwzorowanie pozostaje odwracalne:

| Tag | Znaczenie |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — zgodnie z poniższą konwencją |
| `Chloros:IndexMin` | wartość indeksu zapisana jako DN 0 |
| `Chloros:IndexMax` | wartość indeksu zapisana jako pełna skala |

Należy je odczytać i odwrócić, normalizując zapisany DN do zakresu 0…1 najpierw względem maksymalnej wartości typu danych (dtype), tak aby jedno wyrażenie obejmowało każdy format liczb całkowitych:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Odczyt nie wymaga `-config pix4d.config` — exiftool odczytuje te właściwości XMP samodzielnie. Konfiguracja jest wymagana tylko do ich *zapisu*.

> **`IndexMin == IndexMax`** oznacza, że każdy piksel tej klatki miał tę samą wartość indeksu. Raster jest wypełniony w połowie skali i nie ma nic, przez co można by go podzielić — należy przyjąć wartość końcową.

> **Brak tagów `Chloros:Index*` oznacza brak określonego mapowania.** Eksport LUT to render w kolorze, którego piksele mają kolory gradientowe przypisane do okna progowego LUT, a klatka bez pikseli o skończonym indeksie nie ma zakresu, który można by określić. W żadnym z tych przypadków nie należy zakładać istnienia mapowania — obowiązuje ta sama zasada, co w przypadku brakującego tagu `PixelScale`.**Ponieważ rozciągnięcie dotyczy poszczególnych obrazów, dwie klatki z jednego przelotu nie są porównywalne pod względem DN**— każda z nich ma własne punkty końcowe, więc ten sam wskaźnik NDVI odpowiada różnym wartościom DN na różnych obrazach. W przypadku analiz ilościowych obejmujących cały przelot (statystyki strefowe, stałe progi, szeregi czasowe) należy wyeksportować**`TIFF (32-bit, Percent)`**: rastrowe dane indeksowe są tam zapisywane jako wartości typu float32**samych wartości indeksowych**, bez konieczności przeskalowywania i bez tagów. Pomimo nazwy formatu produkty indeksowe nie są pomnożone przez 100 — skalowanie to dotyczy wyłącznie odbicia o typie danych całkowitoliczbowym.

#### Maska ograniczenia rozdzielania — które piksele zostały ograniczone w danym paśmie

Rozdzielanie spektralne 3×3 może spowodować, że pasmo przyjmie wartość ujemną — tam, gdzie światło ledwo zawiera to pasmo lub gdzie scena wykracza poza zakres opisany przez macierz rozdzielania kamery. Nie jest to rzeczywista wartość, więc podczas eksportu jest ona ustalana na 0 — a podgląd na żywo pokazuje tę samą wartość 0. Pasmo może zostać ograniczone na kilku pikselach lub na prawie wszystkich; maska rejestruje obie te sytuacje. Plik `Chloros:Band{R,G,B}UnmixClampFraction` określa, jaka część klatki została objęta tym zjawiskiem; maska ograniczenia wskazuje **które piksele**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— w folderze `clamp_masks/`**obok** folderu produktu, nigdy w nim (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — dla każdego produktu promieniowania, odbicia i indeksu, w którym co najmniej jeden piksel został ograniczony. Jednopasmowy `uint8`, na własnej siatce pikseli produktu (z uwzględnieniem orientacji, obszar zainteresowania (ROI), binning oraz uwzględniona korekcja wyrównania tablicy). Każda maska zawiera nazwę kamery, z której pochodzi dany produkt (`Make` / `Model` / numer seryjny), czas wykonania zdjęcia oraz tagi obiektywu, a także listę pasm, które reprezentują jej bity (`XMP-Camera:BandName`, najpierw bit 0); jej `ImageDescription` i `Software` oznaczają ją jako maskę z ograniczeniem, a nie zawiera ona kalibracji.
- **Bity**: bit 0 = pasmo w kanale R produktu (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Ustawiony bit oznacza, że wartość piksela w tym paśmie została ograniczona do 0 — należy traktować go oraz wszelkie wskaźniki na nim oparte jako mało wiarygodne.
- **`Chloros:UnmixClampMask`** jest umieszczany na każdym produkcie, na którym przeprowadzono proces rozdzielania: ścieżka maski względem folderu produktu (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, zawsze `/`— połącz ją z folderem obrazu, aby otworzyć maskę), lub `none`, gdy żadna piksel nie został ograniczony. Brak oznaczenia oznacza, że produkt nigdy nie przeszedł procesu unmix (kamery RGB lub monochromatyczne, poziomy surowe/po usunięciu debayeringu).
- Wyniki przechwytywania zawierają go jako **`clamp_mask`** klatki (ścieżka do maski lub `None`, gdy nic nie zostało ograniczone) — w wynikach `CameraSession.capture()`, klatkach `ArraySession.capture()`w klatkach `CaptureResult` oraz w odpowiedziach HTTP. Kompilacja offline wymienia te maski jako `clamp_masks`.
- Stos wielopasmowy indeksu na żywo (`/api/camera/cli-array-index` wraz z `save_multiband`) otrzymuje jedną maskę na każdą kamerę źródłową, w folderze wyjściowym o nazwie `clamp_masks/` — zobacz dokumentację CLI`lattice index`. Plik wideo nie zawiera maski: wynik kompilacji offline podaje zamiast tego współczynnik ograniczenia dla każdej klatki.

Ponieważ żaden folder produktu nie zawiera maski na żadnym poziomie hierarchii, folder produktu można przekazać do oprogramowania fotogrametrycznego w obecnej postaci — wraz z podfolderami — a oprogramowanie to pobiera wyłącznie obrazy. Należy przekazać mu folder produktu, a nie folderu nadrzędnego. Własna funkcja importu programu Chloros pomija maski.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect dla kamer LATTICE

Trwałe sesje backendowe dla sprzętu działającego w czasie rzeczywistym. Te same punkty końcowe, z których korzysta GUI, więc zachowanie jest identyczne w SDK, CLI i GUI.

### Pojedyncza kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Ustawione tutaj ekspozycja, wzmocnienie i cel są zachowywane jako Twoje: automatyczna kontrola ekspozycji Chloros (w tym ekspozycja mierzona przez DAQ) nie ingeruje w nie. `cam.set_settings(exposure_auto="Continuous")` (lub `gain_auto=`) zwraca kontrolę nad ekspozycją (wzmocnieniem), a `cam.set_settings(target_brightness_auto=True)` przywraca wartość docelową do automatycznego ustawiania przez Chloros.

#### `connect_camera()` Sygnatura

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Powraca po uruchomieniu strumienia.** Backend czeka w ograniczonym czasie (około 15 s na uruchomienie strumienia, a następnie 10 s na jego pierwszy kompletny bufor — nie na pierwszą klatkę podglądu, która pojawia się około 1,5 s później), więc `status()` pojawia się zaraz po tym, jak pojawia się `streaming`. `connect_stream` sesji to blok `stream` odpowiedzi: `state` `streaming`, `no_frames` (strumień został uruchomiony, ale nie dostarczył żadnych danych — wskazuje na to `note` oraz, w przypadku kamery obsługującej pakiety typu jumbo, należy ponownie nawiązać połączenie przy rozmiarze pakietu wynoszącym 1500 bajtów), `starting` lub `start_failed`, gdzie `waited_s`. `None` dla kamery, która była już podłączona. Podgląd (`get_camera_preview`, `/api/camera/jpeg/<serial>`) żądany zaraz po nawiązaniu połączenia oczekuje przez określony czas (4 s) na pierwszą klatkę podglądu z kamery zamiast odpowiadać „Brak dostępnej klatki”; kamera, której strumień nie dostarczył jeszcze niczego (`no_frames` lub uzbrojona na wyzwalacz sprzętowy), nadal odpowiada natychmiast.
- **Rozmiar pakietu można ustawić samodzielnie.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` otwiera kamerę przy pakietach o rozmiarze 1500 bajtów, bez sprawdzania (`chloros-cli lattice cam-connect --packet-size standard` wysyła to samo).
- **W danym momencie może być nawiązane tylko jedno połączenie z pulą kamer**: otwarcie, sprawdzenie rozmiaru pakietu, pierwsze pobranie danych kalibracyjnych (około 70 s dla kamery po raz pierwszy podłączonej do tego komputera), rozpoczęcie transmisji. Żądanie, które wymaga w międzyczasie kamery (`read_nodes`, `set_settings`, przechwytywanie, drugie połączenie) czeka na nią maksymalnie 5 s, a następnie kończy się niepowodzeniem z kodem HTTP 503 `CAMERA_POOL_BUSY`, podając nazwę łączącej się kamery (`ChlorosConnectError`); zapytanie ponownie za chwilę. Listy nigdy nie czekają: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` i `CameraSession.status()` odpowiadają natychmiast. Kamera w trakcie łączenia jest wymieniona w puli odpowiedzi pod numerem `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — oraz `elapsed_s`), a jego `status()` wskazuje na `connecting: True` wraz z jego `phase`; pozostałe kamery są odczytywane na żywo, gdy pula jest wolna, w przeciwnym razie zwracane są dane z ostatniego odczytu (`live: False`).

#### Metody `CameraSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=15.0)` | Stan kamery na żywo, odczytywany przez `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` lub `stopped`), `device` (model, numer seryjny, oprogramowanie sprzętowe, temperatura, `is_mono`) oraz jego aktualne ustawienia. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Odczytuje węzły GenICam; zwraca `{nodes, errors, enums, device, was_in_pool, in_array_id}`, oraz `frame_values` i `notes` podczas działania automatycznej ekspozycji (patrz poniżej). |
| `set_settings(**kwargs)` | Zapisuje węzły według nazw przyjaznych (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Kod `exposure_time` (`gain`) oznacza ręczną ekspozycję (wzmocnienie): wpisany bez kodu `exposure_auto` (`gain_auto`), należy najpierw wyłączyć tryb automatyczny, aby wartość nie została odrzucona przez działający tryb automatyczny. Wynik `applied` zawiera ustawienia, z którymi aparat wykonał zdjęcie; zapis odrzucony przez aparat znajduje się w `errors` i nie jest zachowywany. Błąd określa zakres, jaki aparat akceptuje w danej chwili (wartość maksymalna `ExposureTime` zależy od częstotliwości klatek aparatu), a tryb automatyczny, który został wyłączony wyłącznie dla odrzuconej wartości, zostaje przywrócony; `restored` zawiera jego listę (np. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Przechwytuje **pojedynczą** klatki. Zwraca `CaptureResult`: listę słowników metadanych klatek (po jednym na każdy zapisany poziom), wraz z `.skipped`, `.failed`, `.daq_files` oraz `.daq_matches`. (Funkcja przechwytywania seryjnego/wieloklatowego została usunięta — jeśli potrzebujesz serii, wywołaj `capture()` w pętli.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Przechwytywanie kolejno (`interval_s=0`) lub w odstępach czasu z tej kamery, dla przechwytywań typu `count` i/lub `duration_s` (przynajmniej jednego, aby proces się zakończył), tak jak robi to `ArraySession.capture_repeated()` w przypadku tablicy. Zwraca listę elementów typu `CaptureResult`; zatrzymuje się po przechwyceniu sygnalizującym zapełnienie dysku. `on_capture(i, result)` uruchamia się po każdym z nich; pozostałe słowa kluczowe trafiają do `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Zwolnienie z puli, przywrócenie kamery do stanu, w jakim została znaleziona podczas łączenia (okablowanie wyzwalacza, PTP, ustawienia); `keep_settings=True` zapisuje swoje ustawienia w własnej pamięci, natomiast `clear_settings=True` resetuje tę pamięć (patrz [Pamięć kamery](#camera-memory--keep_settings--clear_settings)); zwraca `camera_memory`, gdy zostanie wywołana dowolna z tych funkcji. Kamera będąca elementem tablicy zwalnia całą tablicę. Nie wykonuje żadnej operacji, jeśli dołączono ją do już otwartej sesji. Zwraca wynik, gdy backend potwierdzi zwolnienie lub gdy nie ma nic do zwolnienia (kamera już nie istnieje lub żaden backend nie działa); zwolnienie, które zakończyło się przekroczeniem limitu czasu lub zostało odrzucone przez backend, generuje błąd `ChlorosConnectError`, ponieważ kamera może nadal być zajęta, a kolejne połączenie kolidowałoby z nią. Wyjście z bloku `with` nigdy nie generuje błędu. |

Atrybuty: `serial`, `device` (rekord wykrycia kamery) oraz `backend_url`.

Kontrola eksportu `capture()` (ten sam model co tablica + GUI):

- `processing` / `levels` — `processing="all"` zapisuje każdy odpowiedni typ eksportu; `levels=["raw","radiance"]` zapisuje tylko te (zastępuje `processing`). Aby użyć domyślnych ustawień backendu, należy pominąć oba.
- `daq` — odczyt z modułu DAQ użytego do przechwycenia jest domyślnie zapisywany jako plik towarzyszący `.daq`, zawierający wyłącznie surowe dane, dzięki czemu klatka może zostać później przetworzona na odbicie/wskaźnik. `daq=False` pomija ten plik, jeśli żaden poziom go nie wykorzystuje. `force_daq=True` to starsza pisownia wartości domyślnej. Brak DAQ, brak `.daq`.
- `daq_tolerance_ms` — jak daleko od ekspozycji może znajdować się odczyt, aby nadal mieścił się w tolerancji (domyślnie 500). Zmienia się jedynie wynik oceny.

#### `read_nodes()` przy automatycznej ekspozycji — `frame_values` i `notes`

Podczas działania `ExposureAuto` (`GainAuto`) węzeł `ExposureTime` (`Gain`) odczytuje **ostatnią zapisaną wartość**, a nie aktualny stan kamery. `nodes` nadal przechowuje tę wartość węzła. Gdy zapytasz o `ExposureTime` lub `Gain` w podłączonej kamerze, której tryb automatyczny nie jest ustawiony na `Off`, w odpowiedzi dodawane jest:

- `frame_values` — wartość zawartą w najnowszej klatce z kamery, pod nazwą węzła (`ExposureTime` w µs, `Gain` w dB);
- `notes` — jedno zdanie na każdy taki węzeł, zawierające obie wartości.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Oba klucze są nieobecne, gdy tryb automatyczny to `Off` (węzeł jest wówczasrzeczywistą wartością kamery) oraz w przypadku kamery, która nie jest podłączona do zaplecza. W przypadku tablicy `ArraySession.status()["member_ae"]` podaje wartości klatek wszystkich elementów jednocześnie.

#### Limit ekspozycji mierzonej przez DAQ (HTTP)

Po podłączeniu czujnika światła DAQ program Chloros ogranicza ekspozycję × wzmocnienie kamery na podstawie zmierzonego oświetlenia dziennego, tak aby biały panel referencyjny nie uległ przesterowaniu. Ograniczenie to obowiązuje tylko wtedy, gdy w polu widzenia kamery znajduje się cel kalibracyjny (widoczny w jej kadrze w ciągu ostatnich 10 s); gdy cel nie znajduje się w polu widzenia, zamiast tego działa zwykła automatyczna ekspozycja kamery wraz z jej własną ochroną przed prześwietleniem, dzięki czemu pozostała część sceny nie jest ciemniejsza niż to konieczne. Nie ma opakowania SDK; wartość tę można odczytać lub zmienić dla poszczególnych kamer za pomocą `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (treść żądania POST `{"enabled": false}` wyłącza tę funkcję dla danej kamery). Odpowiedź zawiera `enabled`, `cap_us` (aktualna wartość górna, `None` gdy funkcja jest nieaktywna, w tym gdy w polu widzenia nie ma celu kalibracyjnego), `k_exp_g` (stała, na podstawie której obliczany jest pułap) oraz:

| Pole | Znaczenie |
| --- | --- |
| `k_exp_g_source` | Która stała jest używana: `camera` — własna dla tej kamery, obliczona na podstawie punktu laboratoryjnego w jej pakiecie kalibracyjnym (kamery RGB); `fleet` — stała floty, stosowana w przypadku braku punktu laboratoryjnego w pakiecie; `override` — nadpisanie pola ustawione na zapleczu. |
| `k_exp_g_fleet` | Stała floty, dla porównania. |

### Zsynchronizowany układ — `ArraySession` (Smart-Prep)

`connect_array` stanowi **zalecany punkt wyjścia** dla konfiguracji z wieloma kamerami. W tle uruchamia pełny proces Smart-Prep z graficznym interfejsem użytkownika:

1. **Analiza sieci**(`/api/camera/array/recommend`) — ustawia**pełne parametry czujnika** (bez binningu, bez kadrowania ROI) i pozwala na obniżenie częstotliwości klatek do poziomu, jaki zapewnia dany host i sieć, przy uwzględnieniu formatów pikseli oraz sposobu pakowania, z których będzie korzystać Connect. Klatki nigdy nie są zmniejszane w celu utrzymania częstotliwości: należy przekazać mniejszy `width` / `height` lub `binning=2` / `4`, aby zamienić rozdzielczość na liczbę klatek na sekundę (punkt końcowy`alternatives` zawiera listę tych opcji wraz z przewidywanymi wartościami fps).
2. **Automatyczny wybór poziomu** — `sim-capture-sim-emit` (każda kamera wykonuje ekspozycję przy tym samym wyzwalaczu, a sygnały są wysyłane w taki sposób, aby współdzielić łącze; liczba klatek na sekundę spada wraz ze zmniejszaniem się przepustowości) w dowolnej sieci, w której kamery mogą być zsynchronizowane w celu współdzielenia łącza; `slip-emit-and-capture` (kamery korzystają z łącza na zmianę, nie są eksponowane jednocześnie) tylko wtedy, gdy nie jest to możliwe.
3. **PTP włączone**domyślnie — sygnatury czasowe między kamerami są zsynchronizowane z jednym wspólnym zegarem z dokładnością do**~1 ms**, a [zegar nadrzędny PTP](#ptp-grandmaster--time_sync_status). Jednoczesna ekspozycja wynika z wyzwalacza sprzętowego M8 (**&lt; 100 µs** między modułami), a nie z PTP: PTP synchronizuje *znaczniki czasu*, a nie ekspozycje.
4. **Automatyczny wybór formatu pikseli dla poszczególnych kamer** — monochromatyczny → `Mono12`, kolorowy → `BayerRG12`.
5. **Inicjowanie AE** — rejestruje aktualny stan AE każdej kamery, dzięki czemu połączenie nie resetuje ekspozycji w trakcie pracy.
6. **Konfiguracja wyzwalacza GPIO** — `connect_array` uzbraja każdą kamerę (`TriggerMode=On`, `TriggerSource=Line2`), dzięki czemu impuls z urządzenia nadrzędnegoimpuls steruje urządzeniami podrzędnymi przez kabel M8. Jest to krok dotyczący wyłącznie macierzy: pojedyncza kamera uruchomiona za pomocą `LatticeCamera` działa zamiast tego w trybie swobodnym. Zwolnienie macierzy (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) powoduje przywrócenie wszystkich kamer do trybu swobodnej pracy (`TriggerMode=Off`), przy czym pozostałe ustawienia pozostają takie, jak wykryto podczas nawiązywania połączenia: ustawienia wyzwalacza i linii synchronizacji (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; linia synchronizacji urządzenia nadrzędnego przywrócona do trybu wejściowego), PTP oraz ustawienia aparatu zmienione podczas sesji — chyba że `keep_settings=True` / `clear_settings=True` (patrz [Pamięć kamery](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atrybuty: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` oraz `packed_reason` (ostatnie cztery to `None` na uchwycie od `attach_array`), `connect_stream` oraz `backend_url`. `connect_stream` to blok `stream` odpowiedzi na połączenie, kształt pojedynczego połączenia plus `members` (`{serial: {state, why?}}`): `streaming` po nadejściu pierwszej zsynchronizowanej grupy tablicy (backend czeka na nią, w ograniczonym czasie, około 10 s), lub `no_frames`, gdzie ostatnią nieudaną próbą pobrania danych przez każdego członka jest `why`, a w przypadku pakietów jumbo rezerwę 1500 bajtów w `note`. `None` na uchwycie `attach_array`.

#### Sygnatura `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` określa, jak długo wywołanie czeka na nawiązanie połączenia, i jest wysyłane do
backendu jako termin ostateczny. Jeśli do tego czasu połączenie nie zostanie nawiązane, zostaje ono cofnięte: każda kamera, którą
otworzono, zostaje przywrócona do stanu, w jakim została znaleziona, i zamknięta, żadna tablica nie jest podłączona, a
wywołanie generuje błąd `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, wskazując
etap, do którego dotarło połączenie). Tak więc wywołanie, które wygenerowało błąd, nigdy nie pozostawia za sobą podłączonej macierzy
połączonego za sobą. Macierz składająca się z 9 kamer łączy się w ciągu około minuty; zaraz po
włączeniu kamer może to zająć kilka minut. Macierz, która jest uruchomiona, ale nie ma jeszcze
zsynchronizowanej grupy klatek, jest zwracana z kodem błędu `connect_stream["state"]`
`no_frames`. Dziennik zaplecza zawiera jeden wiersz czasowy na każdy etap połączenia
(`[array-connect <id>] <step>: <s> s …`) oraz ostatni wiersz określający najwolniejszy etap.

Wartości `force_tier`:
- `"sim-capture-sim-emit"` — prawdziwa synchronizacja (wszystkie kamery wyzwalają się na tym samym zboczu zegara).
- `"sim-capture-ftd-stagger"` — elastyczne rozłożenie w dziedzinie czasu (kamery emitują sygnały w nieznacznie przesuniętych momentach, dzięki czemu pakiety są szeregowane w sieci).
- `"slip-emit-and-capture"` — sekwencyjne przechwytywanie dla każdej kamery (bez synchronizacji czasowej). Wybierane automatycznie tylko wtedy, gdy nie można wystarczająco spowolnić pracy kamer, aby mogły współdzielić magistralę jednocześnie.

`wire_ceiling_mbps` zastępuje **stały limit przepustowości magistrali hosta** w MB/s — jest to jedyna
wartość, od której zależy cała alokacja macierzy. Pozostaw wartość `None`, aby korzystać z automatycznie wykrytej
wartości. Zmniejsz ją, gdy macierz zgłasza uszkodzone ramki GVSP: wartość automatyczna jest wyliczana
na podstawie ogłaszanej przez kartę sieciową prędkości łącza, która zawyża prędkość adapterów USB, wąskich pasm PCIe oraz
obciążonych, współdzielonych struktur — a to zawyżenie objawia się uszkodzonymi ramkami, a nie
widocznym spowolnieniem łącza. Wartość ta jest zapisywana w bloku przechwytywania macierzy projektu, więc
ponowne otwarcie lub późniejsze ustawienie `connect_array` przywraca ją tak samo jak każde inne ustawienie macierzy.
Zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames).

#### Transport spakowany 12-bitowy (`packed=`)

Przy pełnym obciążeniu czujników częstotliwość ramek macierzy zależy od tego, jak szybko każda kamera może wysyłać
swoje ramki. Formaty 12-bitowe (`BayerRG12`, `Mono12`) są zazwyczaj przesyłane jako 16-bitowe
słowa; transport spakowany wysyła te same 12 bitów w 1,5 bajta na piksel i
rozpakowuje je na hoście, dzięki czemu wysłanie każdej klatki zajmuje o 25% mniej czasu — około
**33% więcej klatek na sekundę przy pełnym wykorzystaniu czujnika** (wynik pomiaru: 10,7 → 14,0 kl./s w
układzie 9 kamer). **Zapisane dane są identyczne**: te same wartości, te same 16-bitowe
pliki, te same metadane.

- `packed="auto"` (domyślnie) kompresuje dane tylko wtedy, gdy jest to korzystne **i** komputer główny jest w stanie
  sobie pozwolić na dekompresję: podczas nawiązywania połączenia moduł zaplecza mierzy własną prędkość dekompresji,
  liczbę rdzeni procesora, aktualne obciążenie oraz przepustowość sieci. Komputer stacjonarny z wolnymi rdzeniami
  kompresuje przy pełnej liczbie czujników; zajęty lub energooszczędny host (komputery jednopłytkowe,
  kontrolery wbudowane) przesyła dane w postaci niekompresowanej. Małe obszary zainteresowania (ROI) i klatki z grupowaniem pikseli przynoszą niewielkie korzyści.
- `packed="off"` (lub `False`) nigdy nie kompresuje — aby wykluczyć tę opcję podczas diagnozowania
  lub na hoście, który musi zachować moc procesora dla innych zadań.
- `packed="on"` (lub `True`) zawsze kompresuje formaty 12-bitowe.
- Każda inna wartość powoduje wygenerowanie `ValueError`. Formaty 8-bitowe i 16-bitowe nigdy nie są
  kompresowane.

`analyze_array_network(..., packed="auto")` podejmuje tę samą decyzję,
więc jego fps to częstotliwość, z jaką będzie działać tablica (odpowiedź zawiera
`packed_transport` i `packed_reason`); jego wartość domyślna `pixel_format` to to, co
stosuje connect (Mono12 / BayerRG12), a nie 8-bit.

Decyzja i jej uzasadnienie znajdują się w sesji:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` stosuje to samo nadpisanie do każdej
tablicy w projekcie (domyślnie `None`, co oznacza `"auto"`: projekt nie zapisuje
ustawień upakowania dla poszczególnych tablic).
Aby ustawić domyślną wartość dla całego komputera, należy ustawić `CHLOROS_ARRAY_PACKED12=off` (lub `on`) w
środowisku backendu; jednak jawnie określone wartości `packed="on"` / `"off"` nadal mają pierwszeństwo.
`/api/camera/array/status` zgłasza wartości `packed_transport` i `packed_decision`.

#### Oczekiwane częstotliwości klatek

Zmierzona częstotliwość klatek matrycy (synchronizacja sprzętowa, 12 bitów; pełna rozdzielczość matrycy = 2048 × 1536). Wykorzystaj je do weryfikacji poprawności `status()["actual_fps"]`; `analyze_array_network()` prognozuje liczbę klatek dla Twojego komputera i sieci.

| Konfiguracja | Liczba klatek matrycy |
| --- | --- |
| Pełny czujnik, karta sieciowa hosta PCIe 10 GbE, transport spakowany (`packed="auto"` na komputerze stacjonarnym) | ~14 klatek na sekundę |
| Pełny czujnik, karta sieciowa hosta PCIe 10 GbE, bez kompresji (`packed="off"`) | ~10,5 klatek na sekundę |
| Pełny czujnik, 8 kamer podłączonych do adaptera USB 10 GbE | ~3,8 klatek na sekundę |
| Pełna rozdzielczość czujnika, łącze hosta 1 GbE | 1,2–1,7 kl./s |
| Binning 2 × 2 (`binning=2`), karta sieciowa hosta 10 GbE | ~30–33 kl./s |

- W przypadku łącza PCIe 10 GbE to **kamery** stanowią ograniczenie przy pełnym wykorzystaniu matrycy, a nie sieć: 4 i 8 kamer działają z tą samą szybkością.
- **`target_fps` to częstotliwość wyzwalania, a macierz nigdy nie działa szybciej.** `target_fps=1` uruchamia układ z prędkością 1 klatki na sekundę; `None` (ustawienie domyślne) uruchamia go z najwyższą prędkością, jaką obsługują host i sieć. Prędkość przekraczająca tę wartość jest obniżana do tego poziomu (patrz [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` z `4294` w `status()` nie jest prędkością łącza.** Oznacza to, że system Windows nie mógł odczytać prędkości karty (sterownik zgłosił największą wartość 32-bitową), co jest typowe dla kart sieciowych USB Ethernet. Prędkość użytkowa karty USB w systemie Windows jest ograniczona do 200 MB/s niezależnie od wartości (zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames)); aby go dostroić, zapoznaj się z sekcją „Karty sieciowe USB 10 GbE” w dokumentacji CLI w rozdziale [Konfiguracja i dostrajanie karty sieciowej hosta](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Ograniczenie USB obowiązuje również w systemie Linux: 400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson.** Karta jest identyfikowana na podstawie sysfs (w razie braku danych – na podstawie `ethtool -i`), więc karta USB 5 GbE lub 10 GbE jest traktowana zgodnie z tym limitem, a nie zgodnie z podaną na tabliczce znamionowej prędkością 500 / ~1060 MB/s. Wartość dla x86_64 została zmierzona w systemie Linux: ta sama karta, która w systemie Windows powoduje uszkodzenia ramek przy prędkościach powyżej 200 MB/s, obsługiwała bezbłędnie układ czterech kamer przy każdej prędkości do 480 MB/s. Na platformie NVIDIA Jetson ta sama karta działała bezbłędnie tylko przy prędkości 200 MB/s, a powyżej tej wartości sporadycznie traciła ramki. Jeśli Twój host przetwarza więcej danych bez utraty ramek przy użyciu `gvsp_corrupt`, zwiększ go za pomocą `wire_ceiling_mbps`. W systemie Linux adapter podłączony do portu USB 2.0 ma limit 30 MB/s; podłącz go do portu USB 3.

#### Nadmierna subskrypcja (minimalna prędkość na kamerę)

Macierz działa w każdej sieci; wszystko zależy od liczby klatek na sekundę. Algorytm Sim-emit przydziela każdej kamerze równy udział w bezpiecznym pod względem kolizji budżecie przepustowości, a wszystkie z nich nadal eksponują obraz przy tym samym wyzwalaczu, więc w przypadku wolnego łącza udział ten — a wraz z nim liczba klatek na sekundę — jest niewielki: 8 pełnychkamery o pełnej rozdzielczości i 12-bitowej głębi kolorów w sieci 1 GbE bez ramek jumbo otrzymują po 6,5 MB/s, co daje około **1 klatki na sekundę**(około 1,3 klatki na sekundę po kompresji). Ta część jest ograniczona do najniższej częstotliwości, z jaką kamery mogą być sterowane (`per_cam_floor_bps`: odczyt z zakresu GevSCPD każdej kamery podczas łączenia, nigdy poniżej 1 MB/s). Jeśli łącze wymaga wolniejszej transmisji, połączenie przechodzi w tryb**szeregowy**(`slip-emit-and-capture`): kamery działają na zmianę z pełną prędkością łącza, więc każda ramka dociera, ale elementy**nie są udostępniane jednocześnie**— poziom odpowiedzi i blokady (`SERIAL: ...`) to potwierdzają; należy przekazać `force_tier="sim-capture-sim-emit"`, aby odrzucić ten tryb awaryjny. Rozmiar klatki nie ma wpływu na współdzielenie, więc**binning i ROI zwiększają częstotliwość klatek, ale nie zmieniają tego, czy tablica się mieści**. Tylko łącze, które nie jest w stanie przesyłać tablicy przy żadnym tempie, ani jednocześnie, ani na zmianę,**jest przeciążone** — trybem awarii jest utrata pakietów GVSP, a nie niższa częstotliwość klatek. Rozwiązania: mniej kamer, ramki typu jumbo w całym łańcuchu lub szybsza karta sieciowa.

- Odpowiedzi `analyze_array_network()` i `/api/camera/array/connect` zawierają `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` oraz `per_cam_floor_bps`. Gdy wartość `oversubscribed` ma wartość true, projekcja **zeruje pola fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) zamiast zgłaszać mylącą wartość wskazującą na niską, ale działającą częstotliwość.
- `POST /api/camera/array/connect` akceptuje parametr treści `pin_resolution` (**tylko HTTP — nie jest to argument kluczowy SDK**; `connect_array` nie udostępnia go). Jest to tryb ścisły: połączenie z nadmierną liczbą subskrypcji i ustawionym parametrem `pin_resolution` jest**kategorycznie odrzucane** wraz z błędem wskazującym wszystkie możliwe rozwiązania. Bez tego połączenie jest nawiązywane z ostrzeżeniem, że macierz utraci pakiety. W obu przypadkach połączenie zachowuje żądany rozmiar ramki — nigdy nie grupuje ani nie przycina jej samodzielnie.
- Wyłącznie do celów testowych: ustaw `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` w środowisku backendu, aby zmienić odrzucenie na wyraźne ostrzeżenie — połączenie zostanie nawiązane mimo to, a utrata pakietów zostanie zaakceptowana.

#### Stan macierzy — który podsystem traci ramki

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) zawiera aktywny blok `health` w
podłączonej macierzy, ponownie oceniany w ramach ruchomego **10-sekundowego** . Dzieli on utratę ramek
na dwie przyczyny wymagające przeciwnych rozwiązań, zamiast jednego wskaźnika „niekompletności”, który
nie określa żadnej z nich:

| Pole | Co to oznacza | Który podsystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na numer seryjny) | Klatka **dotarła, ale była uszkodzona strukturalnie**— utrata pakietów GVSP. |**Sieć**: przepustowość łącza, taktowanie, pierścień odbiorczy karty sieciowej, MTU |
| `never_arrived_rate_pct` (na numer seryjny) | Ramka **w ogóle nie dotarła**— kamera nie zadziałała lub nic z niej nie wyszło. |**Wyzwalacz / synchronizacja**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najgorsza częstotliwość transmisji dla każdy z nich. | — |
| `per_cam_rate_pct` | Łączny wskaźnik niekompletności na aparat (obie przyczyny łącznie). | — |
| `stable_for_seconds` | Jak długo każdy aparat pozostawał poniżej 0,01 %. | — |

Oprócz `health` ten sam rekord podaje liczbę, od której zależy cała alokacja:

| Pole | Znaczenie |
| --- | --- |
| `wire_ceiling_mbps` | Obowiązujący budżet przepustowości hosta, MB/s. |
| `wire_ceiling_source` | Źródło tej wartości, opisane słowami — np. `USB-capped 200 MB/s (was theoretical 1062; …)` lub `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, gdy ustawił ją `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` dla adaptera Ethernet USB. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretacja:** wartość `gvsp_corrupt_rate_pct` różna od zera przy wartości `never_arrived_rate_pct` równej 0 oznacza,
że wyzwalanie i synchronizacja kabla są idealne, a 100% strat występuje na ścieżce sieciowej — należy obniżyć
`wire_ceiling_mbps` i ponownie podłącz. Odwrotny wzór wskazuje raczej na kabel synchronizacji lub
linię wyzwalającą.

> **Wartość `target_fps` nie jest czynnikiem decydującym o uszkodzonych ramkach.** Tempo GevSCPD jest ustalane jednorazowo przy
> nawiązaniu połączenia, więc obniżenie częstotliwości wyzwalania zmienia cykl pracy, a nie
> częstotliwość wysyłania serii impulsów. Zmniejszenie zapotrzebowania o 5× nie przyniosło poprawy, natomiast
> obniżenie limitu przepustowości z 240 do 200 MB/s spowodowało spadek odsetka uszkodzonych ramek w tym samym zestawie z 10,4 % do
> 0,00 %.

> **Działająca macierz nie może samodzielnie przeplanować się w oprogramowaniu układowym LATTICE (M3C / M3M).** Nie może
> samodzielnie tego naprawić; należy odłączyć i ponownie podłączyć urządzenie, aby moduł wyboru czasu połączenia ponownie zaplanował działanie w oparciu o
> nową wartość maksymalną.

**Karta sieciowa USB Ethernet jest ograniczona do 200 MB/s** przez sondę, niezależnie od jej
nazwy: tabela wydajności, która przekształca prędkość łącza na wartość ciągłą, jest
oparta na standardzie PCIe, a karta sieciowa USB podaje swoją prędkość łącza Ethernet, będąc jednocześnie ograniczona przez
magistralę USB i jej sterownik. Ograniczenie to jest wartością bezwzględną, a nie ułamkową — adapter USB 1 GbE
osiąga prędkość ~80 MB/s i nie ma na niego wpływu. Dotyczy to systemów Windows i Linux (w tym Jetson);
zobacz [Oczekiwane częstotliwości odświeżania](#expected-frame-rates), aby dowiedzieć się, gdzie zostało to zmierzone.

#### Metody `ArraySession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Aktualny stan działającej tablicy: `target_fps` / `target_fps_auto` (częstotliwość wyzwalania oraz informacja, czy kontroler częstotliwości ją kontroluje), `target_fps_clamped_reason` (powód, dla którego ustalona częstotliwość jest utrzymywana na niższym poziomie; w przeciwnym razie `None`), `actual_fps` (wartość zmierzona; 0, gdy pętla przechwytywania zrezygnowała lub utknęła), `recommended_fps`, `streaming` (False, gdy pętla przechwytywania zrezygnowała — sprawdź to, a nie tylko `last_error`; pozostaje True po ponownym uruchomieniu ustawień, co sygnalizuje `restarting`), `stalled` (True, gdy pętla pobierania działa, ale nie opublikowała żadnej grupy klatek przez około dziesięć okresów klatek, 2–30 s; `last_error` sygnalizuje to dopiero po 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` tylko wtedy, gdy macierz była połączona z PTP **i** każdy element ostatnio odczytał `PtpEnable` jako włączony; nie to samo, co zablokowany, co mierzy `ptp_epoch`), `ptp_configured` (to, o co prosiło połączenie), `ptp_off_members` / `ptp_unread_members` (elementy, które ostatnio odczytały `PtpEnable` jako wyłączone / nie odpowiedziały), `member_recovery` (`None` lub węzeł, który stracił zasilanie lub połączenie: `state`, `waiting`, mimo że nie odpowiada, `reopening`, następnie `recovered`, wraz z `members` i `why` — macierz ponownie otwiera go, gdy ten ponownie odpowie, i przywraca mu czas połączenia PTP, okablowanie wyzwalacza i linii synchronizacji, ustawienia tempa pracy i kamery, a następnie ponownie przesyła strumień bez nowych `connect_array`) oraz `ptp_epoch` (zmierzone), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` oznacza, że system Windows nie mógł odczytać prędkości karty — zobacz [Oczekiwane częstotliwości odświeżania](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (czy łańcuch radiancji dla poszczególnych pasm nadaje się do użytku oraz który element go przerwał) oraz `band_clamp` — podczas gdy podgląd na żywo renderuje tablicę: udział oświetlonych pikseli, który proces rozdzielania kolorów ograniczył do 0, dla każdego elementu (`members`: `{serial: [band0, band1, band2]}` w kolejności pasm R, G, B kamery), a dla każdego pasma obraz złożony pokazuje (`composite`: `{band name: share}`), przy czym `warn` to odsetek, który interfejs użytkownika oznacza jako „Dane pasma o niskim poziomie pewności” (mierzone mniej więcej raz na sekundę; `None`, gdy nie jest uruchomiony podgląd na żywo), oraz `member_ae` — dla każdego elementu: ekspozycja (`exposure_us`) i wzmocnienie (`gain_db`) zawarte w ostatniej opublikowanej klatce, pobrane z samej klatki (`read_nodes` z `ExposureTime` / `Gain` zwraca ostatnią wartość zapisaną podczas działania automatycznej ekspozycji, a nie bieżącą wartość). |
| `set_member_settings(serial, **kwargs)` | Zapisuje ustawienia jednej kamery należącej do grupy, zgodnie z tym, jak `CameraSession.set_settings()` je pobiera (ekspozycja, wzmocnienie, docelowa jasność, ...); element pozostaje w tablicy. Wywołuje błąd `ChlorosConnectError` w przypadku elementu szeregowego, który nie jest członkiem tablicy. Zobacz [Ekspozycja na elementach tablicy](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Ustalone przez funkcję smart-prep parametry (poziom przechwytywania, rozmiar klatki, `pixel_format`), aktywny blok `health` oraz budżet przepustowości hosta. Zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Zmiana częstotliwości wyzwalania podczas pracy macierzy, bez konieczności ponownego podłączania. Liczba ustala tę wartość (kontroler częstotliwości nie ingeruje); `None` przywraca ustawienie automatyczne. Zwraca `{requested, target_fps, target_fps_auto, clamped}` — `clamped` ma wartość True, gdy częstotliwość została obniżona do poziomu, jaki mogą obsłużyć host i sieć. Zobacz [Przechwytywanie interwałowe w pomiarach dronowych](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Jedna klatka z każdej kamery, **wszystkie z jednego wyzwalacza**, w postaci tablic numpy (bez zapisywania plików). Zwraca `GrabResult` (lub `dict`): `{serial: array}` lub `{serial: (array, metadata)}` z `with_metadata=True`. `.group_id` to sygnał wyzwalający wspólny dla wszystkich klatek; `.errors` przypisuje każdą pominiętą kamerę do przyczyny (kamera, której nie można obsłużyć przy danym sygnale wyzwalającym, jest pomijana i nigdy nie jest uzupełniana danymi z innej kamery). `processing` to `raw` / `debayered` / `radiance` / `reflectance` / `display`. Odczyt ten jest taki sam jak w przypadku [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) w API projektu — jedna wspólna implementacja — więc każde słowo kluczowe tam (`band`, `align`, `bit_depth`, `refl_max`) działa tutaj. Wymaga `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator zwracający jeden wynik `grab()` na takt (tylko argumenty słowne), z synchronizacją do `fps`, ograniczony przez `duration` (w sekundach) i/lub `count` (`None` = nieograniczony). **Każdy zwrócony zbiór stanowi nowy wyzwalacz:** tik, którego najnowszym wyzwalaczem jest już zwrócony zbiór, czeka na następny wyzwalacz zamiast go powtarzać, więc strumień działa z prędkością mniejszą z wartości `fps` i własnej częstotliwości odświeżania tablicy. Generuje `FrameFetchError` zamiast milczeć: natychmiast, gdy każda kamera z jakiegoś powodu odmówi wykonania kolejnego taktu, po 60 sekundach taktów bez klatki oraz po 60 sekundach bez nowego wyzwalacza („brak nowego wyzwalacza przez 60 s”). Ta sama implementacja co `ArrayHandle.frame_stream()`. Zobacz [Klatki na żywo we własnym potoku](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Uruchom własny tryb ciągłego przechwytywania backendu (tryb ciągły w interfejsie graficznym) → [`ContinuousHandle`](#continuoushandle). Zapisuje **każdą** zsynchronizowaną grupę publikowaną przez tablicę, tak jak zapisałaby ją funkcja `capture()` (te same ograniczenia eksportu), aż do zatrzymania lub do momentu uruchomienia funkcji `max_captures` / `duration_s` (`None` = do momentu zatrzymania). Generuje błąd `ChlorosConnectError`, jeśli dla tej tablicy już działa inne zadanie. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Jedna zsynchronizowana grupa przechwytywania. Zwraca `CaptureResult` (lista słowników ramek + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Przekaż poprzedni wynik`group_seq` jako `after_seq`, aby uzyskać następną zsynchronizowaną grupę po nim, zamiast czekać na nową. Poniżej znajdują się opcje eksportu. |
| `capture(..., smart=True)` | **Inteligentne przechwytywanie** — czeka na ustabilizowanie się AE we wszystkich kamerach, a następnie uruchamia się. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Najszybsze przechwytywanie: tylko dane surowe + przypisany odczyt DAQ (+ swobodny indeks łączny). Odzwierciedla przycisk „Najszybsze przechwytywanie” w interfejsie graficznym. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Pojedyncze / ciągłe / interwałowe w jednej pętli ograniczonej. Zwraca `list[CaptureResult]`.**Wymaga `count` i/lub `duration_s`**, aby zakończyć działanie (SDK nie obsługuje skrótu Ctrl+C). Kończy się również przedwcześnie, po zakończeniu przebiegu, gdy przejście zgłasza zapełnienie dysku (wpis `.failed` z `disk_full`); to przejście jest ostatnim wynikiem. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Rozpocznij nagrywanie podglądu indeksu połączonego na żywo do pliku wideo/GIF → `RecorderHandle`. Plik wideo jest w formacie Motion JPEG `.avi` (nie jest wymagany ffmpeg). Jeśli nie jest otwarty żaden widok GUI, backend renderuje go: `expr` (wyrażenie indeksowe w nazwach pasm macierzy, np. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) lub domyślnie NDVI macierzy. Jeden rejestrator kompozytowy na każdą matrycę. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Rozpocznij serię zdjęć w formacie surowym Bayer z dużą liczbą klatek na sekundę → `RecorderHandle`. Przetwórz offline za pomocą `build_video()`. `orient=True`: produkty tworzone przez `build_video()` są obracane zgodnie z orientacją kamery (produkty poszczególnych kamer zgodnie z orientacją ich własnych kamer, a połączony indeks zgodnie z orientacją kamery głównej); klatki w formacie surowym pozostają w formacie natywnym, a manifest serii rejestruje orientację każdego elementu. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Przeprowadź przetwarzanie offline zapisanej serii zdjęć w formacie RAW, tworząc skalibrowane pliki wideo. Blokuje działanie do zakończenia (`wait=True`) i zwraca wynik kompilacji: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — zobacz [wynik kompilacji](#the-build_video-result) poniżej. |
| `build_video_status(job_id, timeout=15.0)` | Sprawdzanie statusu zadania kompilacji w trybie offline: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Zwolnij całą matrycę — **jeśli ta sesja ją otworzyła** (`connect_array`): wówczas jest to `force_disconnect()` i generuje wyjątek `ChlorosConnectError`, gdy zwolnienie nie zostanie potwierdzone (blok `with` nigdy nie generuje wyjątku przy wyjściu). W przypadku uchwytu z `attach_array` pozostawia tablicę działającą i wyświetla jeden wiersz na stderr z informacją o tym: tablica może należeć do GUI lub innego skryptu. Każda kamera pozostaje w trybie swobodnej pracy i zachowuje stan, w jakim została znaleziona podczas nawiązywania połączenia; `keep_settings=True` zapisuje jej ustawienia w pamięci kamery, a `clear_settings=True` resetuje tę pamięć do ustawień fabrycznych (patrz [Pamięć kamery](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Zwolnij całą macierz, niezależnie od tego, kto ją otworzył. Użyj tej funkcji na przypisanym uchwycie, jeśli chcesz zakończyć działanie macierzy (np. w celu ponownego podłączenia jej z innymi ustawieniami). Zwraca wynik po potwierdzeniu zwolnienia przez backend lub gdy nie ma nic do zwolnienia (tablica już nie istnieje lub żaden backend nie działa); operacja zwolnienia, która zakończyła się przekroczeniem limitu czasu lub została odrzucona przez backend, generuje błąd `ChlorosConnectError`. Niezależnie od użytej flagi zwraca stan pamięci każdej kamery (`camera_memory`), w przeciwnym razie zwraca `None`. |

`capture()` — funkcje eksportu (ten sam punkt końcowy, z którego korzystają GUI i CLI):

- `processing` / `levels` — `processing="all"` (lub `levels=["raw","radiance",…]`) zapisuje każdy odpowiedni typ eksportu dla każdej kamery; pojedyncza wartość `processing` zapisuje tylko ten poziom.
- `aligned` — **domyślnie włączone (`None`), ponieważ przechwytywanie tej samej matrycy przez aplikację Chloros**(jej ustawienie przechwytywania**Wyrównane** dla danej matrycy). Eksport danych niebędących danymi surowymi każdego elementu jest dopasowywany do klatki kamery głównej za pomocą [profilu wyrównania](#array-alignment) i przycinane do wspólnego obszaru nakładania się elementów, dzięki czemu pliki są zarejestrowane pikselowo i mają taki sam rozmiar, kadrowanie oraz tagi `Alignment*`, jak w aplikacji (w obrębie obszaru zainteresowania (ROI) transformacja jest ponownie dostosowywana do okna); `XMP-mapir:AlignmentJSON` określa `"applied": true` wraz z jego `crop_roi`. Pliki surowe pozostają nieprzekształcone w rozmiarze natywnym, ale zawierają transformację w metadanych. `aligned=False` zapisuje każdy plik w rozmiarze natywnym, nieprzekształconym; `aligned=True` to domyślne ustawienie. `continuous()`, `capture(smart=True)`, `capture_repeated()` oraz pliki na poziomie projektu `ArrayHandle.capture()` / `capture_smart()` przyjmują tę samą wartość domyślną. Niezależnie od tego, czy zastosowano zniekształcenie, każdy plik elementu rejestruje transformację `Alignment*` swojej kamery, gdy tablica posiada profil wyrównania — z wyjątkiem kamery, której profil nie jest wyrównany (pominięta lub przymusowo ustawiona na tożsamość z powodu nieudanego rozwiązania) oraz kamera referencyjna, gdy żadna inna kamera nie jest wyrównana: te nie zawierają tagów `Alignment*`. Jeśli w tablicy nie ma profilu, nagranie jest zapisywane jako niewyrównane, a wynik to odzwierciedla: `result.alignment` to `{"requested": True, "applied": False, "warnings": [...]}`, po jednym ostrzeżeniu na każdą przyczynę, dla której nie można było zastosować zniekształcenia (brak profilu, elementy o różnych rozdzielczościach, tablica, której nie da się, klatka, na którą nie można nałożyć profilu). Kod `applied` to `True` tylko wtedy, gdy każdy eksport, który zażądał zniekształcenia, je otrzymał. Automatyczne wyrównanie matrycy tworzy profil po ustabilizowaniu się ekspozycji; `ArrayHandle.calibrate_alignment()` / `align-calibrate` w CLI tworzą go na żądanie. `result.oriented` wskazuje, czy `orient=True` obrócił produkty.
- `render_index=False` — pomija nakładkę wskaźnika wegetacji dla poszczególnych kamer; domyślnie renderuje ją tam, gdzie została skonfigurowana.
- `daq` / `daq_tolerance_ms` — odczyt DAQ jest domyślnie zapisywany jako plik towarzyszący `.daq`, nawet jeśli żaden z wybranych poziomów tego nie wymaga; `daq=False` pomija go w takim przypadku, a `daq_tolerance_ms` ustawia tolerancję dopasowania (domyślnie 500). `force_daq=True` to starsza pisownia wartości domyślnej. Zobacz [Odczyt z DAQ zapisywany podczas przechwytywania](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — włącza **produkty**każdego elementu (`display`, `radiance`, `reflectance`, indeks dla poszczególnych kamer oraz obraz złożony) o orientację kamery po zniekształceniu wynikającym z wyrównania: zgodnie z orientacją**kamery głównej**w momencie ich dopasowania do jej układu odniesienia (domyślnie `aligned` — elementy pozostają współzarejestrowane), w przeciwnym razie zgodnie z orientacją ich własnych kamer.**Domyślnie: każdy plik elementu w formacie natywnym**, tak jak zawsze ma to miejsce w przypadku ujęć tablicowych; pliki `raw` i `debayered` pozostają w formacie natywnym niezależnie od wyboru. Każdy plik rejestruje orientację ramki, w której znajdują się jego piksele (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — plik surowy, z własnego aparatu), a plik XMP `Alignment*` zachowuje transformację ramki natywnej, zastosowaną przed orientacją; plik RAW zawierający tę transformację rejestruje również `ChlorosAligned*` (układ odniesienia głównego), czyli orientację produktu przekształconego do układu odniesienia głównego. Przetwarzanie obraca każdy produkt o wartość odpowiadającą jego pikselom: o wartość wyrównania, gdy przekształca plik RAW (włączone wyrównanie tablicy), lub o orientację własną aparatu, gdy nie jest. `capture_repeated()` i `capture_fastest()` przepuszczają ją bez zmian; `ArrayHandle.capture()` / `capture_smart()` na poziomie projektu również ją przyjmują.

**Kompresja TIFF (opcja dostępna HTTP):** `ArraySession.capture()` nie wysyła klucza `compression`, więc obowiązuje domyślne ustawienie zaplecza — `POST /api/camera/array/capture` odczytuje parametr treści `compression`, domyślnie: nieskompresowany, piksele z czujnika dokładnie tak, jak zostały odczytane (~6,3 MB na klatkę w pełnej rozdzielczości). `"deflate"` (bezstratna kompresja zlib L1 + predyktor poziomy, ~4,1 MB/klatka) zapisuje mniejsze pliki wolniej — obie metody są bezstratne i po zaimportowaniu odczytują się identycznie. SDK nie udostępnia dla tego parametru żadnego argumentu kwarg; rozwiązaniem awaryjnym jest `chloros-cli lattice array-capture --compression deflate` lub surowe HTTP. DEFLATE blokuje GIL w Python, więc zapis skompresowanych danych nie jest równoległy w wątkach zapisu poszczególnych kamer — do ciągłego przechwytywania w pełnej rozdzielczości z częstotliwością czujnika konieczne jest użycie domyślnego zapisu nieskompresowanego. Szczegóły: [Podręcznik CLI → array-capture](cli-reference.md).**Kolejne grupy (protokół HTTP):** punkt końcowy zwraca `group_seq`; pętla, która odsyła ten kod jako `after_seq` w kolejnym żądaniu, otrzymuje następną zsynchronizowaną grupę — w tym tę opublikowaną podczas zapisywania poprzedniego przechwycenia — zamiast czekać na grupę opublikowaną po wysłaniu żądania, a `skipped_groups` zgłasza wszystkie grupy pominięte przez pętlę. `capture_repeated`, `--continuous` w CLI oraz tryb ciągły w GUI działają w ten sposób; interwały pozostają aktualne w każdym zaplanowanym momencie.**Nadpisywanie eksportu dla poszczególnych członków (tylko HTTP):**ten sam punkt końcowy akceptuje również `exclude_serials` (lista — usunięcie elementów z zapisanego zestawu; tablica nadal działa jako jedna zsynchronizowana grupa, a wykluczone elementy są zwracane w `excluded`), `serial_levels` (nadpisania na poziomie poszczególnych kamer `{serial: [level tokens]}`) oraz `serial_index` (nadpisania nakładki indeksowej dla poszczególnych kamer `{serial: bool}`). Są to parametry korpusu zgodne z GUI i**nie są to jeszcze argumenty kluczowe SDK**; elementy nieobecne w mapach są zastępowane wartościami dla całej tablicy `levels` / `render_index`.

##### Sprawdzanie pominiętych kamer — `CaptureResult.skipped`

`ArraySession.capture()` zwraca `CaptureResult`, który jest podklasą `list`: iteruj po nim, indeksuj go, `len()` — każdy istniejący wzorzec nadal działa. Nowy kod może sprawdzić atrybut `.skipped`, aby zobaczyć, które kamery zostały wykluczone i dlaczego. Najczęstszym przypadkiem są kamery RGB w matrycy z mieszanymi filtrami, gdy żądane jest `processing="radiance"` lub `"reflectance"` — promieniowanie na piksel w układzie Bayera nie ma znaczenia dla czujnika szerokopasmowego, więc moduł zaplecza pomija te kamery, zamiast generować bezsensowne dane.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeny przyczyny mają wzór `<level>-not-applicable-to-rgb-cam` (jeden wpis na każdy pominięty poziom, z których każdy zawiera `level`). Kamera, której pakiet kalibracyjny nie został załadowany, pomija `radiance` / `reflectance`, zastępując je `<level>-needs-calibration (connect the camera so its cal pack loads)`, a w przypadku braku użytecznego odczytu odbicia światła z modułu DAQ pomija je, zastępując je wartością `reflectance-skipped-no-dls`. Pozostałe pominięcia związane z odbiciem światła to: `reflectance-skipped-no-fresh-dls` (brak dostępnego aktualnego odczytu promieniowania padającego), `reflectance-skipped-bound-daq-unavailable (…)` (nie udało się uzyskać dostępu do powiązanego urządzenia DAQ) oraz `dls-uncalibrated-band-<nm>` — pasmo leży w większości poza radiometrycznie skalibrowanym zakresem czujnika światła DAQ (~374–974 nm), więc absolutny podział oparty na danych z modułu DAQ zostaje odrzucony, a klatka zostaje wyraźnie zdegradowana do poziomu odpowiedzi czujnika. Spośród dostępnych w sprzedaży modeli tylko F988 wywołuje ten błąd; obsługiwanym trybem pracy tej kamery jest proces z wykorzystaniem panelu odbicia.

##### Pliki, których nie udało się zapisać — `CaptureResult.failed`

Przechwytywanie, w ramach którego udało się zapisać tylko część plików, nadal zwraca: lista zawiera wszystkie pliki, które **zostały** zapisane, a `.failed` zawiera pozostałe, po jednym słowniku na plik wraz z `serial` (`None` dla połączonego pliku kompozytowego oraz pliku pomocniczego tablicy `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, lub `match` — pochodzenie `.daq` dla `.match.json`, gdy `.daq` został zapisany, ale jego plik sidecar nie) oraz `error`. W przypadku zapełnionego dysku wpis zawiera również `disk_full: True`; każda późniejsza operacja przechwytywania na tym dysku zakończyłaby się niepowodzeniem w ten sam sposób, dlatego `capture_repeated()` kończy się w tym miejscu. W przypadku częściowego przechwytywania nie jest zgłaszany żaden wyjątek, a plik o nazwie `.failed` nie pozostał na dysku w stanie częściowo zapisanym. `.failed` jest pusty w przypadku czystego przechwytywania.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Odczyt z modułu DAQ podczas zapisywania przechwycenia — `.daq_files` i `.daq_matches`

Za każdym razem, gdy moduł DAQ jest powiązany z kamerą lub matrycą albo znajduje się w puli, każde przechwycenie zapisuje wykorzystany odczyt promieniowania padającego: plik `.daq` w podfolderze `daq/` folderu przechwytywania, wraz z plikami `.match.json` i `.cameras.json` (którego odczyt to jest). Dotyczy to również przechwyceń w formacie surowym lub po usunięciu matrycy Bayera, a plik jest ten sam niezależnie od tego, czy przechwycenie pochodzi z GUI, CLI czy SDK. `daq=False` pomija to w przypadku przechwycenia, którego poziomy nie wykorzystują DAQ (poziom `radiance`, `reflectance` lub `display` nadal to zapisuje); `daq_tolerance_ms` przesuwa linię tolerancji dla tego przechwycenia, a nigdy od użytego odczytu. Który moduł DAQ: ten powiązany z kamerą lub matrycą (wybór czujnika światła w GUI, powiązanie projektu lub `chloros-cli daq pool-bind`), w przeciwnym razie najświeższy, jaki posiada backend. Interfejs CLI określa polecenia `--no-daq` i `--daq-tolerance-ms` ([Dokumentacja CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Przechwycenie zapisuje odczyt czujnika światła, który został użyty, jako `.daq` wraz z wartością `<file>.daq.match.json` obok niego, określającą sposób powiązania odczytu z ekspozycją: `method`, `gap_ms` (od ekspozycji do najbliższego odczytu DAQ) oraz `in_tolerance` (w ciągu 500 ms lub `daq_tolerance_ms` danego zapisu). Plik XMP pliku odbicia zawiera tę samą informację, co `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` i `Chloros:DlsInTolerance`.

Zrzut czeka na pierwszy odczyt z modułu DAQ po ekspozycji, aby móc połączyć odczyty po obu jego stronach (`method` `interpolated`). Urządzenie DAQ-E wysyła odczyt co 0,4 s w świetle dziennym i co 1–2 s w pomieszczeniach, więc `capture()` zwraca wynik maksymalnie około półtora takiego interwału później; sama ekspozycja nie jest opóźniana. Nie czeka się na moduł DAQ, który przestał wysyłać odczyty. Odczyt z modułu DAQ-E stanowi średnią natężenia światła w przedziale czasu od poprzedniego odczytu, więc jest umieszczany w połowie tego przedziału: `gap_ms` jest mierzony względem tego środka, a `.match.json`&#x27;`bracket_stamps_ns` podaje własne znaczniki czasu odczytów. `ArraySession.continuous()` czeka bez spowalniania przechwytywania. `capture_repeated()` wod tyłu nie czekają, więc zachowują swoją częstotliwość i są dopasowywane do najbliższego odczytu przed ekspozycją.

Odczyt nigdy nie traci ważności, więc żadna klatka nie jest tracona z powodu spóźnionego odczytu — zamiast tego jest oznaczana:

- **Strumień z modułu DAQ utknął** — wykorzystywany jest jego najnowszy odczyt, a `gap_ms` podaje, jak długo jest on nieaktualny (`in_tolerance: false` — ponad 500 ms).
- **Brak urządzenia DAQ do odczytu**— rozłączenie (`disconnect()`, `chloros-cli daq pool-disconnect`) lub brak dostępu do powiązanego urządzenia DAQ — ponownie wykorzystywany jest ostatni odczyt z kamery i opisywany w odniesieniu do**tej** ekspozycji: `method`, `retained`, `gap_ms` z tej ekspozycji, `in_tolerance` jest przeliczana, a `retained_from_epoch_ns` wskazuje ekspozycję, z którą została po raz pierwszy dopasowana. Kamera, która nigdy nie miała odczytu, nadal nie otrzymuje współczynnika odbicia, a wynik wyjaśnia dlaczego (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, lub odrzucenie ujęcia).

Pliki zapisane przez aplikację w folderze projektu otwartego w Chloros dołączają do listy plików tego projektu, tak jak ma to miejsce w przypadku zrzutów wykonanych w samej aplikacji (pliki zapisane gdzie indziej pozostają bez zmian): `CameraSession.capture()`, `ArraySession.capture()` (a także `smart=True`) oraz `capture_cameras_transient()`. Wynik potwierdza to w pliku `.project_add` (`capture_cameras_transient()`: słownik `project_add`) — `held`, `files` oraz `message` z backendu; `held` to `True` w trakcie trwania przebiegu przetwarzania, a pliki dołączają się po jego zakończeniu. `None`, gdy żaden plik nie trafił do projektu.

`CameraSession.capture()` i `ArraySession.capture()` zwracają je jako wyniki `.daq_files` i `.daq_matches`, jako odpowiedzi przechwycone w protokole HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) zawierają `daq_files` i `daq_matches`, po jednym na `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` oraz `stream_fault` — błąd strumienia właściwy dla urządzenia DAQ, sprawdzany wyłącznie w przypadku odczytu wykraczającego poza tolerancję. Przed oparciem się na współczynniku odbicia z rejestracji należy sprawdzić `in_tolerance` w jego `.match.json` lub XMP, albo w [`status()`](#daqsensorsession-methods).

Poziomy `processing`:

| Poziom | Wyjście |
| --- | --- |
| `"raw"` | Jednokanałowy układ Bayera (kamery monochromatyczne: pojedyncze pasmo) bezpośrednio z czujnika. |
| `"debayered"` *(ustawienie domyślne SDK)* | 3-kanałowy kolor: odjęcie ciemnego przesunięcia (DSNU) z pakietu kalibracyjnego od danych surowych, a następnie demosaikowanie z uwzględnieniem krawędzi za pomocą OpenCV — bez tablicy LUT liniowości, płaskiego pola ani demiksowania (kamery monochromatyczne: 1-kanałowa skala szarości). Zapisany plik TIFF ma kolejność kanałów **RGB**(fotometryczne RGB, więc każda przeglądarka obrazów wyświetla prawdziwe kolory); tablice w pamięci, zwracane przez `grab()` / `frame_stream()`, mają kolejność**BGR** — należy odwrócić kolejność kanałów (`frame[..., ::-1]`), aby porównać wynik z zapisanym plikiem. `chloros-cli process` zapisuje ten sam wynik co `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm poprzez pełny łańcuch radiometryczny. Tylko tryb wielospektralny — kamery RGB są pomijane. |
| `"reflectance"` | uint16 0..32768 (kompatybilny z Pix4D); wymaga sparowania z aktywnym systemem DAQ w celu uzyskania odniesienia bezwzględnego. Tylko tryb wielospektralny. |
| `"display"` | Pełny łańcuch odpowiadający podglądowi w interfejsie graficznym (CCM + WB + gamma zgodnie z profilem kamery). |
| `"all"` | **Jeden plik na każdy odpowiedni poziom** dla każdej kamery (zgodny z opcją „Capture All” w interfejsie graficznym / ustawieniem domyślnym w CLI). Zwrócony plik `CaptureResult` zawiera wówczas jeden słownik klatek na każdy plik `(cam, level)`, z poziomem w każdym słowniku; poziomy, które nie mają zastosowania, pojawiają się w pliku `.skipped`. Odczyt z urządzenia DAQ użytego do przechwytywania jest zapisywany jako plik towarzyszący `.daq`. |

> **Uwaga — ustawienia domyślne różnią się od tych w interfejsie CLI.** Domyślnym ustawieniem dla `ArraySession.capture()` jest `processing="debayered"`; domyślnym ustawieniem dla polecenia `chloros-cli lattice array-capture` jest `processing="all"`. Aby odzwierciedlić wielopoziomowe zapisywanie w interfejsie SDK/GUI, należy jawnie przekazać `processing="all"` z SDK.

##### Nazwy plików przechwytujących tablicę

Każdy poziom jest zapisywany we własnym podfolderze `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` dla `display`, `index/`, `composite/`). Każdy plik zawiera nazwę kamery w formacie `SN<serial>-<filter>`: numer seryjny, a następnie część nazwy modelu kamery odpowiadająca filtrowi — `SN214000533-FOCN` dla `M3C-L87-FOCN`, `SN261902142-F450` dla modelu `M3M-L87-F450` — połączone z `-`, więc nazwa ma te same pola oddzielone znakiem `_`, co poprzednio. Kamera, której modelu nie da się odczytać, to sam `SN<serial>`, a nagrania z wcześniejszych wersji nie mają filtra w nazwie. Nazwa pliku zależy od tego, o ile poziomów poprosił użytkownik:

| Liczba poziomów, o którą poproszono | Nazwa pliku | Przykład (`reflectance`, domyślnie `format="tiff"`) |
| --- | --- | --- |
| **Jeden poziom** (`processing="reflectance"` lub `levels` zawierający jedną pozycję) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — brak przyrostka poziomu, tak jak w poprzednich wersjach, dzięki czemu istniejące skrypty nadal działają | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Kilka poziomów** (`processing="all"` lub `levels` z więcej niż jedną pozycją) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Maska ograniczenia](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) następuje po nazwie produktu, `clamp_masks/<product folder>/<product name>_clampmask.tif`: zapis odbicia pojedynczegopoziomowy zapis odbicia generuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, natomiast `processing="all"` generuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Reguła dotycząca sufiksów jest ustalana indywidualnie dla każdego elementu na podstawie poziomów, o które poproszono, więc pominięty poziom (np. odbicie w kamerze RGB) nie powoduje jej zmiany. Odczytaj ścieżki z zwróconych klatek, zamiast je tworzyć. Rejestracja z jednej kamery (`CameraSession.capture()`) zawsze dodaje poziom: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### Pliki PNG/JPEG: pobierane są wyłącznie produkty wyświetlane i indeksowane

Pliki PNG lub JPEG przechowują 8 bitów na kanał. Żądanie pliku o poziomie, którego nie może on pomieścić, zostanie odrzucone przed rozpoczęciem przechwytywania, wraz z komunikatem wskazującym poziom oraz format TIFF jako ten, który go obsługuje:

- **`radiance`**(float32) oraz**`reflectance`**(uint16, `32768` = 1,0) — a także `processing="all"`, który je obejmuje. SDK sprawdza to na podstawie argumentów: `capture()` generuje**`ValueError`** bez żądania (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` z kamery przesyłającej strumień o rozdzielczości większej niż 8 bitów** (12-bitowy M3C, Mono12 M3M) — moduł zaplecza odrzuca te dane, znając głębię bitową każdej kamery: kod stanu HTTP 400 z kodami `error_code` i `FORMAT_CANNOT_HOLD_LEVEL`, generowany jako `ChlorosConnectError` (`CameraSession` / `ArraySession`) lub `RuntimeError` (`CameraHandle` / `ArrayHandle`). Jednopoziomowe przechwytywanie `debayered` z jednej kamery pobiera 8-bitową klatkę dla formatów PNG/JPEG i nie jest odrzucane.

Zapisz dane radiometryczne jako `format="tiff"` (`ext=".tiff"`); użyj formatów PNG/JPEG dla `display` i renderów indeksowych.

#### Jak działa automatyczna ekspozycja

- **Najpierw ekspozycja, potem wzmocnienie.** Gdy wzmocnienie jest ustawione na tryb automatyczny, pozostaje na poziomie 0 dB, dopóki ekspozycja nie osiągnie maksymalnej dozwolonej wartości.
- **Ręczna ekspozycja pozostawia wzmocnienie w trybie automatycznym**, dopóki nie zmienisz wzmocnienia lub nie ustawisz go na tryb ręczny. Po powrocie do trybu automatycznego limit wzmocnienia jest taki sam jak poprzednio.
- **Niewielkie jasne obszary nie przyciemniają sceny.** Odbicie światła zajmujące do 1% kadru jest ignorowane, a w przypadku paneli znanego wzorca kalibracyjnego – do 0,5% powierzchni panelu.
- **Górna granica ekspozycji DAQ obowiązuje tylko wtedy, gdy wzorzec kalibracyjny znajduje się w polu widzenia.** Zapobiega to przesterowaniu białego panelu; gdy panel nie jest widoczny, kamera ustawia ekspozycję odpowiednio do sceny.
- **Stałe oświetlenie pozostaje niezmienne, a po zmianie oświetlenia obraz ustabilizuje się w ciągu kilku sekund.** Przy stałym oświetleniu ekspozycja i jej limit pozostają niezmienne, a po znacznej zmianie oświetlenia obraz ustabilizuje się w ciągu kilku sekund. Przeciążenie nie ogranicza już ekspozycji na stałe, a zmiana obszaru zainteresowania (ROI) powoduje ponowne ustalenie ekspozycji.
- **Docelowa jasność pozostaje w zasięgu.** Nie wzrasta, gdy kamera osiąga limit ekspozycji, a jej wzmocnienie nie może wzrosnąć, więc powracające światło nie powoduje prześwietlenia kadru.
- **Kamera, której automatyczna ekspozycja przestaje reagować, jest utrzymywana na poziomie odpowiednim dla aktualnego oświetlenia.** W jasnym słońcu kamera może spaść do najkrótszego czasu naświetlania i pozostać w tym stanie z ciemnym obrazem, przy czym wszystkie jej ustawienia wskazują wartości w normie. Chloros monitoruje obraz, utrzymuje tę kamerę na poziomie dostosowanym do danego oświetlenia i zwalnia ją, gdy światło się zmienia. Nie zakłóca tego zaciemniony lub zakryty czujnik światła.

#### Ekspozycja w elementach macierzy

Kamera LATTICE ogranicza wartość `ExposureTime` do jednego okresu częstotliwości odświeżania. Podczas gdy macierz przesyła strumień danych, Chloros uruchamia każdy element z włączoną wartością `AcquisitionFrameRateEnable`, z częstotliwością nieco wyższą od częstotliwości wyzwalania, tak więc górna granica ekspozycji elementu znajduje się nieco poniżej okresu wyzwalania, **w dowolnym obszarze zainteresowania (ROI)**:

- Okres klatki elementu wynosi okres wyzwalania pomniejszony o margines 15% tego okresu (co najmniej 3 ms, maksymalnie połowa okresu), a jego górna granica ekspozycji to ten okres klatki pomniejszony o około 0,1 ms. Przykład: wyzwalacz 10 klatek na sekundę (100 ms) daje częstotliwość elementu wynoszącą około 11,8 klatek na sekundę oraz górną granicę ekspozycji wynoszącą około 85 ms.
- **Ręczne ustawienie `ExposureTime` dłuższe niż ten limit spowalnia wyzwalanie, aby się do niego dostosować.** Chloros najpierw otwiera limit tego elementu na tę wartość i pozwala, by wolniejsza częstotliwość działała przez kilka okresów wyzwalania (zapis może potrwać kilka sekund dłużej), a następnie zapisuje ją, a wyzwalacz nie uruchamia się szybciej niż pozwala na to ekspozycja. W przypadku tablicy z automatyczną częstotliwością poprzednia częstotliwość powraca, gdy każdy element, którego ręczna ekspozycja spowolniła działanie, powróci do ekspozycji automatycznej. Wartość, której sama kamera nie może przyjąć, jest nadal odrzucana, a zakres, który obecnie akceptuje, jest podawany w komunikacie o błędzie, a automatyczna ekspozycja elementu jest przywracana, jeśli została wyłączona tylko na czas tego zapisu. Przed wersją 1.3.0 mały obszar zainteresowania (ROI) ograniczał każdy element do własnego okresu swobodnej pracy czujnika, niezależnie od częstotliwości wyzwalania: 4,9 ms przy rozdzielczości 768 × 304, więc żaden element nie mógł zostać rozjaśniony.
- Program Chloros nigdy nie skraca ręcznej ekspozycji, aby dostosować ją do szybszego wyzwalania. Częstotliwość automatyczna pozostaje w granicach dozwolonych przez ręczne ekspozycje; zablokowana częstotliwość szybsza niż pozwala na to ekspozycja pozostawia dla danego elementu limit czasu wystarczająco długi, a wyzwalanie odbywa się nie szybciej niż pozwala na to ekspozycja. Należy zmniejszyć ekspozycję lub wartość zablokowaną, gdy każdy element musi wykonać każde wyzwolenie.
- **Odczyt:** odczyt węzłów elementu pokazuje wartości `AcquisitionFrameRateEnable` True oraz `AcquisitionFrameRate` z częstotliwością elementu; zezwolenie wyłącza się, gdy tablica się zatrzymuje.
- **Macierz jest właścicielem tych dwóch węzłów. Nie zapisuj `AcquisitionFrameRateEnable` ani `AcquisitionFrameRate` na elemencie macierzy** — zamiast tego ustaw częstotliwość macierzyza pomocą `target_fps` / `set_target_fps()`.

Zapisz ekspozycję elementu za pomocą `arr.set_member_settings(serial, exposure_time=20000)` (słowa kluczowe `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` działa tak samo). Element pozostaje w tablicy. Nie należy otwierać elementu tablicy za pomocą `connect_camera()`: funkcja connect odrzuca kamerę, która już znajduje się w tablicy, a funkcja `CameraSession`, której wynik zwraca, usuwa kamerę z tablicy po jej odłączeniu. Funkcja `ArraySession.status()["member_ae"]` zgłasza ekspozycję każdego elementuekspozycję i wzmocnienie z najnowszej klatki.

### Jeden rekord projektu, każdy tryb

Folder projektu zawiera ustawienia każdego urządzenia: `cameras.json` — dla każdej kamery (`cameras[].settings`, dla pojedynczej kamery i dla każdego elementu macierzy) oraz dla całej macierzy (`arrays[].capture`: format, obszar zainteresowania (ROI), binning, częstotliwość wyzwalania, limit przepustowości) — oraz `sensors.json`, dotyczące każdego czujnika światła (czas integracji, średnia klatki, limit). Aplikacja, interfejs CLI, zestaw SDK oraz serwer MCP odczytują i zapisują te same pliki w ten sam sposób:

- **Urządzenie podłączone w ramach projektu jest w nim zapisywane i rejestruje w nim każdą zmianę.** „W ramach projektu” oznacza, że projekt jest otwarty w zapleczu (aplikacji) lub — poprzez `ChlorosProject` (`open_project()`), który wskazuje własny folder w zapleczu (`project_path`). Kamera, matryca lub czujnik światła, których projekt jeszcze nie zawiera, są dodawane do jego plików po podłączeniu (`connect_camera()`, `connect_array()`, podłączenie czujnika światła, CLI, aplikacja, `chloros-mcp`) wraz z ustawieniami. Zmiana z dowolnego trybu (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, ustawienia macierzy, `set_cap()`, CLI, elementy sterujące w aplikacji) zostaje zapisana w plikach tego projektu chwilę później. Ustawienia ekspozycji, wzmocnienia lub wartości docelowej AE w kamerze są zapisywane jako zachowane, więc ponowne otwarcie projektu powoduje ich przywrócenie.
- **Otwarcie projektu powoduje ustawienie urządzeń zgodnie z nim**, w każdym trybie: aparat, matryca lub czujnik światła podłączone bez własnych ustawień otrzymujązapisane w projekcie (`lattice cam-connect` / `array-connect` przy otwartym projekcie, `chloros-cli project connect`, `open_project().connect_all()`, ponowne otwarcie aplikacji). Macierz podłączona pod nowym identyfikatorem aktualizuje macierz zapisaną w projekcie z tymi samymi kamerami; obok niej dodawana jest macierz z innymi kamerami.
- **Własne zapisywanie aplikacji nigdy nie nadpisuje bieżących ustawień urządzenia** starszą ich kopią i zachowuje nadpisania poszczególnych elementów zapisane przez wywołującego z poziomu CLI lub SDK.
- **Format pikseli, który ustawisz, jest tym, który projekt zachowuje.** Format ustawiony w ustawieniach tablicy, za pomocą `--pixel-format` lub `connect_array(pixel_format=)`, w tym 8-bitowy, jest zapisywany i odtwarzany dokładnie tak, jak został ustawiony. Jeśli nie ustawiono żadnego formatu, każda kamera łączy się przy domyślnym ustawieniu 12-bitowym (`BayerRG12`, `Mono12` w przypadku czujnika monochromatycznego). Nazwa formatu oznacza jego głębię bitową; rodzina odnosi się do czujnika, więc kamera kolorowa nigdy nie jest ustawiana na format monochromatyczny, a kamera monochromatyczna na format Bayera, a wiersz `[PixelFormat <serial>] ... written` wskazuje na to, gdy format został zmapowany. Pliki eksportowane z 8-bitowego przechwytywania mają rozdzielczość 8 bitów, łącznie z odbiciem: są przeskalowane i opatrzone oznaczeniem `PixelScale`.
- Urządzenie podłączone bez projektu nie rejestruje nic.
- **Nagranie podlega projektowi utworzonemu po nim; otwarcie pozostawia je w jego projekcie.**Gdy tworzony jest projekt — `create_project()`, `chloros-mcp`, `create_project` lub opcja**Nowy projekt**w aplikacji — lub zduplikowany (funkcja**Duplikuj**w aplikacji, która otwiera kopię), podczas gdy nagranie nadal zapisuje dane do projektu, który był otwarty wcześniej, nagranie jest kontynuowane, a zapisane dane trafiają do nowego projektu, w tym samym miejscu w jego obrębie (poniżej). Otwarcie istniejącego projektu — `ChlorosProject.process()` (co powoduje otwarcie projektu w tle), `chloros-cli process`, opcja**Otwórz projekt** w aplikacji — powoduje, że nagrywanie nadal trwa w ramach projektu, do którego jest zapisywane: kończy się tam, a pliki dołączają do listy plików tego projektu (aplikacja zatrzymuje każde nagrywanie przed otwarciem projektu).
  - `ArraySession.record()` i `ArraySession.burst()`: plik wideo lub folder serii zawierający wszystkie klatki zostaje tam przeniesiony w całości po zatrzymaniu nagrywania. Wynik zatrzymania podaje nową ścieżkę (`out_dir`, `video_path`), a na jej podstawie tworzona jest kompilacja serii klatek. Jeśli dysk odmawia przeniesienia, nagranie pozostaje w miejscu, w którym zostało zapisane, a wynik to odzwierciedla (`relocation_error`). Surowe klatki zatrzymanej serii oraz plik `.daq` dołączają do listy plików projektu, do którego trafiają; plik wideo nie jest dodawany do listy plików projektu.
  - `ArraySession.continuous()`: każde przechwycenie po wprowadzeniu zmiany jest zapisywane w nowym projekcie; te zapisane wcześniej pozostają na swoich miejscach.
  - `DAQSensorSession.record_start()` (oraz nagranie z wbudowanego czujnika światła aplikacji): po zakończeniu nagrywania plik `.daq` zostaje przeniesiony do folderu `light_sensor` nowego projektu; `record_stop()` zwraca nową ścieżkę, a plik dołącza do listy plików tego projektu.

  Nagranie w folderze poza otwartym projektem pozostaje tam, gdzie jest, a zadania `start_camera_continuous()` / wielo strumieniowe zachowują folder, który im przypisano.
- **W aplikacji nagrania wykonane po utworzeniu nowego projektu trafiają do niego.** Utworzenie lub skopiowanie projektu nie przerywa trwających sesji przechwytywania ani nagrywania, a każde kolejne przechwycenie (opcja „Przechwyć wszystko” lub „Rozpocznij przechwytywanie”) jest zapisywane w nowym projekcie i dołączane do jego listy plików. Otwarcie istniejącego projektu w aplikacji powoduje najpierw zatrzymanie wszystkich z nich.
- **Projekt nie jest przenoszony podczas nagrywania.**Przeniesienie otwartego projektu do innego katalogu roboczego (w aplikacji**Ustawienia → Katalog roboczy**; żadne wywołanie SDK nie przenosi projektu, klient HTTP wysyłający dane do `/api/move-project` to robi) jest odrzucane z kodem HTTP 409 `PROJECT_RECORDING`, gdy do katalogu zapisuje się nagranie macierzy lub seria zdjęć, ciągłe przechwytywanie macierzy, zadanie ciągłego / wielostrumieniowego przechwytywania z kamery lub nagranie z czujnika światła: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Jest to kwestia odrębna od tego, w jakim stanie pozostają kamery po rozłączeniu (zobacz [Pamięć kamery](#camera-memory--keep_settings--clear_settings)): projekt sam ustawia swoje kamery podczas otwierania.

### Pamięć kamery — `keep_settings` / `clear_settings`

Po podłączeniu macierzy kamery przejmują ustawienia macierzy, a następnie można je zmienić. Gdy program Chloros zwalnia kamery — `ArraySession.disconnect()` / `force_disconnect()`, co następuje po zakończeniu funkcji przejściowej lub zatrzymaniu funkcji `start_camera_continuous()` — przywraca każdą z nich do stanu, w jakim znajdowała się przed podłączeniem: okablowanie wyzwalacza i linii synchronizacji, ustawienia PTP i kamery — `ExposureAuto` / `GainAuto` (oraz ręczne ustawienia `ExposureTime` / `Gain`), `TargetBrightness` oraz ograniczenia i tłumienie AE, balans bieli, poziom czerni, `PixelFormat`, binning oraz obszar zainteresowania (ROI), a także potok przetwarzania kolorów (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP: `PtpSlaveOnly` / `PtpDomainNumber`, rozmiar pakietu (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), częstotliwość stymulacji (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), dane fragmentu (`ChunkModeActive`, każdy `ChunkEnable` pod swoim `ChunkSelector`), usuwanie zakłóceń na wejściu wyzwalającym – tablica podaje linię synchronizacji urządzenia podrzędnego (`LineFilterSelector` / `LineFilterWidth`) oraz częstotliwość pozyskiwania klatek (`AcquisitionFrameRate` tylko wtedy, gdy jego włączenie było aktywne: gdy jest wyłączone, częstotliwość odczytuje maksymalną wartość dozwoloną przez taktowanie, a nie ustawienie). Zapisywane są tylko różnice: moment zatrzymania kamery, taktowanie oraz częstotliwość klatek przed ekspozycją, która wymaga określonego okresu klatki.

W przypadku korzystania z kamer poza projektem dwa argumenty zapisują **własną pamięć** kamery — to, z czym się uruchamia, w dowolnym oprogramowaniu (w `disconnect()` / `force_disconnect()`, każdej funkcji przejściowej, `start_camera_continuous()` i `ChlorosProject.disconnect_all()`; MCP: te same argumenty co w `disconnect_device` i narzędziach przejściowych; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** zapisuje ustawienia każdej kamery w pamięci w stanie, w jakim pozostały po zakończeniu sesji (UserSet2, utworzony zestaw startowy). Kamery nadal działają w trybie swobodnym (`TriggerMode` `Off`), a okablowanie linii synchronizacji pozostaje bez zmian. Kamera bez zestawu UserSet2 zapisuje ustawienia w UserSet1; komputer to zapamiętuje, więc mechanizm samonaprawy macierzy Chloros — który resetuje zestaw UserSet1, który odbiega od wartości bazowej Chloros — pozostawia ją bez zmian.
- **`clear_settings=True`** resetuje pamięć każdej kamery do ustawień fabrycznych: wartości bazowej Chlorosa, tej, którą uwzględnia funkcja samonaprawy macierzy — domyślne ustawienia fabryczne z `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatyczną ekspozycją oraz skalibrowanym balansem bieli kamery RGB zachowany — w UserSet1, utworzono zestaw startowy.
- Nie oba (`ValueError`; backend zwraca kod 400). Odpowiedź zawiera `camera_memory`, dla każdej kamery `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` lub `{"cleared": bool, "errors": [...]}` — `force_disconnect()` zwraca tę wartość; w przypadku zadania towarzyszy mu `stop()`.

Aplikacja nie udostępnia tych wartości.

**Każde wyjście powoduje przywrócenie kamer.**Przywracanie stanu pierwotnego odbywa się na każdej ścieżce wyjściowej: `CameraSession.disconnect()` (kamera**X**aplikacji),**X** w tablicy lub dowolny element tablicy (cała tablica jest zwalniana), zamykanie aplikacji (backend ma do 20 s na przywrócenie każdej kamery, tablicy i uruchomionego ciągłego nagrywania; funkcja przejściowa lub jednorazowe wywołanie, które nadal działa, zatrzymuje się w następnym bezpiecznym punkcie, najpierw przywraca swoje kamery, przez maksymalnie 8 s, a następnie odpowiada kodem HTTP 503 `SHUTTING_DOWN`; nadal działająca funkcja `connect_array()` zatrzymuje się przed kolejną kamerą i przywraca otwarte przez siebie kamery, przez maksymalnie 10 s, a następnie generuje błąd `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); a ta, która pojawia się podczas wyłączania, niczego nie otwiera: HTTP 503 `SHUTTING_DOWN`)oraz polecenie `chloros-cli`, które zostało przerwane lub którego terminal został zamknięty (jego zadanie zostaje zatrzymane, a kamery są najpierw przywracane). Backend, który ulega awarii lub zostaje przymusowo zamknięty, nie może niczego przywrócić; wyłączenie i ponowne włączenie zasilania powoduje przywrócenie każdej kamery do jej pamięci.**Projekt sam ustawia swoje kamery.** Gdy macierz łączy się poprzez projekt — otwarty projekt aplikacji lub `ChlorosProject`, który zgłasza się do backendu (`project_path`) — każdy element otrzymuje ustawienia zapisane dla niego przez projekt: ekspozycję, wzmocnienie lub docelową wartość AE utrzymywane dla niego (traktowane jako własne, więc automatyczna ekspozycja nie ingeruje w nie) oraz tryb balansu bieli (korekcja Chlorosa; własny balans bieli aparatu pozostaje wyłączony), niezależnie od zawartości pamięci aparatu.

### Tryby przechwytywania i rejestratory

Powierzchnia tablicy odzwierciedla panel przechwytywania w interfejsie graficznym: tryby pojedynczego zdjęcia / seryjnego / interwałowego / najszybszej migawki, a także dwa rejestratory (kompozytowe wideo na żywo oraz seria zdjęć w formacie RAW → przetwarzanie offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**to pętla trybów ciągłego/interwałowego w SDK. Ponieważ nie ma `Ctrl+C`, który pozwoliłby przerwać ją ze skryptu,**należy** przekazać `count` i/lub `duration_s` (pętla zatrzymuje się po osiągnięciu dowolnej z tych wartości). `interval_s` jest mierzony od początku każdego przebiegu (zgodnie z interfejsem graficznym). Przejścia ciągłe (`interval_s=0`) łączą się w łańcuch `group_seq` → `after_seq`, więc są to kolejne zsynchronizowane grupy o. Pozostałe argumenty kwargs są przekazywane bezpośrednio do `capture()`.
- **`continuous`**to własny tryb ciągłego przechwytywania backendu — tryb ciągły w interfejsie graficznym.**Należy go używać zamiast `capture_repeated()`, gdy chcesz, aby każdy wyzwalacz był zapisywany na dysku z wysoką częstotliwością klatek:** `capture_repeated()` wysyła jedno żądanie HTTP na grupę z twojego skryptu, podczas gdy `continuous()` utrzymuje dwa procesy przechwytywania w toku wewnątrz backendu bez wymiany danych między grupami. Wyniki pomiarów dla obszaru zainteresowania (ROI) 512 w układzie 9 kamer: 73–77 przechwyceń/s, w porównaniu z układem publikującym około 78. Zachowaj `capture_repeated()` do przechwytywania interwałowego (`interval_s`) oraz do serii, które sprawdzasz pojedynczo. W systemie Windows należy wykluczyć backend z programu Microsoft Defender, aby uzyskać wysoką częstotliwość (patrz [Krok 1](#step-1--install-the-chloros-platform-package)).
- **`record`** jest przeznaczony do *monitoringu*: rejestruje renderowany na żywo złożony indeks połączony. Gdy widok połączony w GUI jest otwarty, rejestruje ten widok tak, jak jest wyświetlany. Gdy żaden nie jest otwarty, backend sam renderuje widok: `expr`, jeśli podasz jeden, w przeciwnym razie NDVI macierzy (pasma czerwone i NIR najbliższe 660 nm i 850 nm, tak jak są one wybierane w widoku na żywo), po wyrównaniu tablicy. Nazwy pasm w `expr` powinny odpowiadać nazwom nadanym przez tablicę: `<band>_<filter>` dla kamery M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` dla kamery M3M (`Mono_850`); krótka nazwa koloru (`NIR`, `Red`) działa tylko wtedy, gdy posiada ją jeden pasek tablicy. Widok GUI otwarty podczas nagrywania jest rejestrowany tak długo, jak długo pozostaje otwarty. Nagrywanie, które nie zapisuje żadnej klatki, kończy się niepowodzeniem z kodami błędów `error`, `"no-frames: …"` (tablica może nadal być w trakcie wyrównywania lub ustalania ekspozycji), a `stop()` generuje wyjątek (zobacz [`RecorderHandle`](#recorderhandle)). Jeden rejestrator złożony na matrycę (generuje wyjątek, jeśli jeden już działa).
- **`burst` → `build_video`** to tryb *analizy*: `burst` zapisuje surowe klatki + manifest dla każdej klatki + jeden `.daq` na każdy odrębny odczyt DLS w ramach `<output>/bursts/<base>/` z pełną prędkością pętli przechwytywania (bez łańcucha, bez exiftool, bez podglądu na żywo). `build_video` dopasowuje czasowo każdą klatkę do najbliższego `.daq` i ponownie uruchamia łańcuch radiancji/odbicia/wskaźnika. `products` to lista `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (domyślnie: połączony wskaźnik). `burst().stop()` automatycznie uruchamia również tworzenie połączonego, zwaną jako `build_job` w wyniku zatrzymania.

##### Wynik `build_video`

| Klucz | Znaczenie |
| --- | --- |
| `success` | `True` tylko wtedy, gdy każdy żądany produkt został zbudowany zgodnie z żądaniem: co najmniej jeden wynik, brak wpisów w `skipped`, i brak wpisu w `degraded`. |
| `outputs` | Ścieżki zapisanych filmów (i plików GIF). |
| `errors` | Problemy w formie tekstowej. |
| `skipped` | Jeden wpis `{"serial", "level", "reason"}` na każdy żądany produkt, który **nie** wygenerował pliku wideo — (brak pakietu kalibracyjnego dla tej kamery, brak odczytu DLS dla współczynnika odbicia, awaria łańcucha przetwarzania) lub indeksu łącznego, który zawiera `"kind": "combined"` oraz `serial` `None` (brak pary pasm czerwonego i NIR wśród kamer, brak grupy z promieniowaniem dla obu pasm). Każdy wpis powoduje `success` `False`; pozostałe filmy są nadal zapisywane i wyświetlane na liście. |
| `substitutions` | Kamery renderowane na innym poziomie niż żądany oraz przyczyna — kamera RGB, dla której żądano promieniowania/odbicia, otrzymuje swoje wideo wyświetlające, z wyrównaniem balansu bieli, jeśli dostępny jest pakiet kalibracyjny kamery. Nie jest to błąd. |
| `notes` | Tekst informacyjny, a nie błąd — np. kamera bez wyrażenia indeksu dla danej kamery nie ma obrazu indeksowego lub obraz wyświetlany przez kamerę RGB nie jest skorygowany pod kątem balansu bieli (brak pakietu kalibracyjnego lub pakiet bez fabrycznego balansu bieli), z podaniem nazwy kamery. Nie jest to błąd. |
| `degraded` | `True`, gdy produkt został zapisany, ale nie jest tym, o który proszono: połączonym indeksem offline jest wbudowana znormalizowana różnica czerwonych/NIR, więc skonfigurowane wyrażenie indeksu nie jest oceniane. Powoduje to `success` `False`. |
| `combined` | Podsumowanie indeksu połączonego: użyte pasma, zakres LUT, utworzone klatki. |
| `clamp_masks` | Ścieżki [masek ograniczających mieszanie](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) dla-klatkowych plików TIFF (`save_tiffs=True`), w `clamp_masks/<level>/` obok folderu `<level>/` plików TIFF, po jednej na każdy plik TIFF, w którym nastąpiło ograniczenie piksela. Plik wideo nie zawiera żadnej. |
| `clamp_fractions` | Co zawiera plik wideo zamiast maski: jeden plik `{"kind", "serial", "level", "frames"}` na każdy produkt wideo (`kind` `"combined"` dla kamer należących do połączonego indeksu), `frames` lista `[frame, R, G, B]` — udział pikseli tej klatki, które proces oddzielania kolorów ograniczył do pasma, jakie zajmuje jej kanał R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Zwracane przez `ArraySession.record()` i `ArraySession.burst()`. Należy używać go jako menedżera kontekstu w celu automatycznego zatrzymania po zamknięciu środowiska lub sterować nim ręcznie.

| Element | Opis |
| --- | --- |
| `job_id` | Identyfikator zadania backendowego (str). |
| `kind` | `"composite"` (z `record`) lub `"raw"` (z `burst`). |
| `start_stats` | Słownik zwrócony przez wywołanie `start`. |
| `result` | `None` podczas działania; końcowy słownik wyników zatrzymania po zakończeniu. |
| `stats(timeout=10.0)` | Aktualne statystyki zadania (zapisane klatki, rzeczywista liczba klatek na sekundę, czas, który upłynął). |
| `stop(timeout=60.0)` | Zatrzymanie rejestratora; zwraca i buforuje wynik końcowy (`.result` pozostaje `None`, jeśli samo żądanie zatrzymania zakończyło się niepowodzeniem). W przypadku nagrania `record()`, które zakończyło się niepowodzeniem — takiego, które nie zapisało żadnej klatki (`error` `"no-frames: …"`) — buforuje wynik w `.result`, a następnie generuje wyjątek `ChlorosConnectError`. Idempotentne (drugie wywołanie zwraca wynik z pamięci podręcznej, bez generowania wyjątku). Opuszczenie bloku `with` zatrzymuje rejestrator bez generowania błędu, więc należy sprawdzić `rec.result.get("error")` po nim. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Zwracany przez `ArraySession.continuous()`. Należy go używać jako menedżera kontekstu — opuszczenie bloku `with` zatrzymuje przechwytywanie, jeśli nadal trwa — lub steruj nim za pomocą `status()` / `wait()` / `stop()`. W danym momencie dla każdej tablicy może być uruchomiony tylko jeden ciągły proces przechwytywania.

| Element | Opis |
| --- | --- |
| `start_status` | Słownik stanu zwracany przez wywołanie startowe. |
| `result` | `None` podczas działania; końcowy słownik stanu po zakończeniu przechwytywania (ustawiony przez `stop()`, lub przez `status()` / `wait()`, które wykryły jego zakończenie). |
| `status(timeout=10.0)` | Słownik stanu na żywo (pola poniżej). |
| `wait(timeout=None, poll_s=0.5)` | Blokuj do momentu samoczynnego zakończenia przechwytywania — osiągnięcie wartości `max_captures` lub `duration_s`, wyczerpanie tablicy, zapełnienie dysku, błąd — i zwróć status końcowy. W przypadku `timeout` zwraca najnowszy stan po upływie tej liczby sekund, nawet jeśli operacja nadal trwa: sprawdź `running`. |
| `stop(timeout=30.0)` | Zatrzymaj przechwytywanie i zwróć jego końcowy status. Idempotentne (drugie wywołanie zwraca wynik z pamięci podręcznej). |

Pola statusu:

| Pole | Znaczenie |
| --- | --- |
| `running` | Prawda, dopóki trwa przechwytywanie. |
| `captures` | Zsynchronizowane grupy zapisane. |
| `files` | Pliki zapisane. |
| `failed_files` | Pliki, których nie udało się zapisać (jak w [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Próby przechwytywania, które zakończyły się niepowodzeniem jako całość, w tym te, w przypadku których przechwytywanie zostało powtórzone (błąd HTTP 503, gdy macierz nie miała na czas nowej grupy). |
| `skipped_groups` | Grupy opublikowane przez macierz, które nie zostały zapisane (przechwytywanie nie nadążało). |
| `captures_per_s` | Średni wskaźnik zapisywania od początku. |
| `elapsed_s` / `started_at` | Liczba sekund od rozpoczęcia; czas rozpoczęcia (sekundy epoki Unix). |
| `last_group_seq` | Najnowsza zsynchronizowana grupa, która została zapisana. |
| `last_error` | Najnowszy błąd lub `None`. |
| `stop_reason` | Po zakończeniu: `count` (osiągnięto `max_captures`), `duration` (osiągnięto `duration_s`), `stopped` (wywołano `stop()`), `array_gone` (macierz została odłączona), `error` (20 kolejnych niepowodzeń lub żądanie odrzucone przez backend), `disk_full` (plik, na który na dysku zabrakło miejsca). `None` podczas działania. |
| `max_captures` / `duration_s` | Granice, z którymi został uruchomiony (`0` = brak). |
| `defender_cores` | Rdzenie procesora wykorzystane przez program Microsoft Defender w ciągu ostatnich 10 sekund; `None` do momentu zmierzenia wartości oraz zawsze `None` na hoście bez programu Defender. |
| `defender_warned` | Ma wartość „True”, gdy rejestracja wykryje, że program Defender spowalnia działanie (przy wykorzystaniu co najmniej 1 rdzenia). Wyklucz backend ze skanowania programu Defender — zobacz [Krok 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Ramki na żywo we własnym potoku przetwarzania

Aby przekazać zsynchronizowane klatki do własnego kodu — uczenia maszynowego, niestandardowej przeglądarki, własnego rejestratora — użyj `ArraySession.frame_stream()`. Każdy takt generuje `GrabResult`: `{serial: numpy array}` dla każdej kamery, **wszystko z jednego wyzwalacza**, z identyfikatorem tego wyzwalacza w `.group_id`. Jest to ta sama implementacja, co `ArrayHandle.frame_stream()` w API projektu, więc wszystko w sekcjach [Jeden wyzwalacz na odczyt](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) oraz [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) ma zastosowanie do obu.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Każdy zwrócony zestaw stanowi nowy sygnał wyzwalający.** `frames.group_id` nigdy nie powtarza się w obrębie strumienia; gdy żądasz więcej `fps` niż wynosi prędkość tablicy, strumień czeka na następny sygnał wyzwalający, więc działa z prędkością wolniejszą z tych dwóch.
- **Brakujące kamery są wymienione z nazwy, a nie uzupełniane.** Kamera, której nie można było obsłużyć przy danym wyzwalaczu, jest nieobecna w słowniku i wymieniona w `frames.errors`.
- **Kończy się głośno.** W przypadku odmowy `FrameFetchError` powtórzy się w następnym takcie, po 60 sekundach bez klatki lub po 60 sekundach bez nowego wyzwalacza.**Co zapewnia.**`frame_stream()` przesyła całe ramki numpy przez lokalne połączenie HTTP, osiągając prędkość maksymalną bliską**170 MB/s**. To wystarcza do przetwarzania na żywo surowych klatek, tablic z grupowanymi danymi lub tablic ROI, ale nie do obsługi każdego wyzwalacza w pełnej rozdzielczości — aby zachować**każdy** wyzwalacz, należy zapisać go na dysku za pomocą [`continuous()`](#continuoushandle). Pomiar przeprowadzono na układzie czterech kamer (FOCN, FRGN, FRGB, F450) z transportem spakowanym; każdy takt stanowił kompletny zestaw jednego wyzwalacza:

| | Pełny czujnik (układ 14,2–14,4 kl./s) | Binning 2 × 2 (układ ~33 kl./s) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 zestawów/s | 22,2 zestawów/s |
| `frame_stream(processing="debayered")` | 2,3 zestawów/s | 9,3 zestawów/s |
| `frame_stream(processing="display")` | 0,8 zestawów/s | 3,0 zestawów/s |
| MJPEG dla każdej kamery, 4 strumienie w rozdzielczości 720p | po 13,5 klatek na sekundę każdy; każda klatka dopasowana do wyzwalacza | po ~25 klatek na sekundę każdy; ~45% wyzwalaczy obecnych we wszystkich 4 |

Użyj `processing="raw"`, aby uzyskać przepustowość i demosaik po swojej stronie; poziom `debayered` usuwa również ciemne przesunięcie z pakietu kalibracyjnego, czego nie robi demosaikowanie na poziomie `raw`.

#### Strumienie MJPEG dla każdej kamery (bez RTSP)

**Protokół RTSP nie jest obsługiwany.** Interfejs strumieniowania to MJPEG przez HTTP (`multipart/x-mixed-replace`, jeden plik JPEG na część, każda część z `Content-Length`), jeden adres URL na kamerę:

| URL | Co udostępnia |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bitowy podgląd, do ~30 klatek na sekundę. W przypadku elementu macierzy, `max_h` (domyślnie 720, 64–2160) oraz `jpeg_q` (domyślnie 80) określają rozmiar i jakość. |
| `GET /api/camera/<serial>/stream-annotated` | Ten sam podgląd z narysowanymi na nim nakładkami GUI (zebra, siatka, krzyżyk, histogram, peaking, pomiar punktowy). `ChlorosLocal.camera_stream()` odczytuje to za Ciebie. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bitowa reflektancja na pasmo — zobacz [Strumień reflektancji na żywo](#live-reflectance-stream-mjpeg). |

- **Nagłówki wyzwalające.** W przypadku elementu tablicy każda część wszystkich trzech strumieni zawiera `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` oraz `X-Chloros-Timestamp-Ns`. Fragmenty z różnych kamer o tym samym `X-Chloros-Group-Id` stanowią ten sam wyzwalacz: zbierz części i pogrupuj je według tego nagłówka, aby ponownie złożyć zestaw. Samodzielna kamera nie ma grupy: jej części podglądu nie zawierają żadnego z tych nagłówków (jej części dotyczące odbicia światła mają pusty nagłówek `X-Chloros-Group-Id`).
- **Strumień podglądu pomija klatki pod obciążeniem.** Przesyła najnowszą klatkę i pomija te, których nie udało się wysłać na czas, więc przy wysokich częstotliwościach tablicy nie każdy sygnał wyzwalający dociera do strumienia każdej kamery (patrz tabela powyżej). Dla każdego sygnału wyzwalającego należy używać `frame_stream()` lub `continuous()`.
- **`/api/camera/stream/<serial>` wstrzymuje się, gdy połączony strumień na żywo macierzy jest otwarty** (`/api/camera/array/stream/<array_id>` — otwiera go widok macierzy w GUI). Strumień każdej kamery wysyła wówczas jedną klatkę zastępczą „Wstrzymane (strumień macierzy aktywny)”, bez nagłówków `X-Chloros-*`, i nic więcej, dopóki połączony strumień nie zostanie zamknięty; następnie wznawia się samoczynnie. `stream-annotated` i `stream-reflectance` nie wstrzymują się. Zamknij widok macierzy w GUI, gdy skrypt odczytuje strumienie podglądu poszczególnych kamer.
- **W przypadku danych liczbowych, a nie obrazów, należy użyć `frame_stream()`.**Format JPEG jest stratny i 8-bitowy; `frame_stream()` zwraca te same wyniki bez kodeka w ścieżce.

### Przechwytywanie w odstępach czasowych dla pomiarów z dronów

Lot pomiarowy wymaga jednej zsynchronizowanej grupy obrazów co kilka metrów przebytej odległości, a nie każdej klatki, jaką może wygenerować tablica. Kontrolują to dwa ustawienia:**`target_fps`**określa częstotliwość rejestrowania klatek przez kamery, a**`capture_repeated(interval_s=…)`** określa częstotliwość zapisywania klatek.**Kamery kontynuują strumieniowanie danych między zapisami.** W każdym interwałowym zapisie zapisywana jest najnowsza zsynchronizowana grupa; klatki pomiędzy nimi nie są zapisywane. Automatyczna ekspozycja dostosowuje się przy każdej klatce, więc w momencie zapisu ekspozycja jest już ustabilizowana. Nie ma trybu, w którym kamery zatrzymywałyby się między ujęciami i byłyby budzone przy każdym z nich: każde ujęcie musiałoby wtedy czekać na rozpoczęcie transmisji strumieniowej i ustabilizowanie się ekspozycji.**Ustaw częstotliwość klatek zgodnie z wymaganiami interwału.**Przy ustawieniu automatycznym tablica działa z najwyższą częstotliwością, jaką obsługują host i sieć. Ustawienie niższej częstotliwości zmniejsza nagrzewanie się kamer, obciążenie sieci i procesora hosta bez zmiany zawartości zapisu. Wybierz częstotliwość zapewniającą**2–4 klatki na interwał**:

- Każde zapisywanie czeka na następną klatkę po zgłoszeniu żądania, więc zapisana grupa klatek trafia do pamięci z opóźnieniem do jednego okresu klatki w stosunku do zaplanowanego momentu. Wyższa częstotliwość pozwala zminimalizować tę lukę.
- Automatyczna ekspozycja dostosowuje się raz na klatkę, więc bardzo niska częstotliwość spowalnia tempo, z jakim ekspozycja nadąża za przemieszczającą się chmurą.

**Oblicz interwał na podstawie planu lotu:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Na przykład obszar pokrycia wynoszący 60 m przy 80% nakładania się do przodu oznacza odstęp 12 m; przy prędkości 10 m/s daje to `interval_s=1.2`, a zablokowana wartość `target_fps=3` daje od 3 do 4 klatek na interwał. Przechwytywanie odbywa się na podstawie czasu: zestaw SDK nie posiada trybu wyzwalania na podstawie odległości ani sygnału GPS, dlatego należy utrzymywać stałą prędkość względem ziemi na liniach pomiarowych.

**Zapisuj dane surowe na pokładzie statku powietrznego i kalibruj je później.** `interval_s` jest mierzony od początku każdego zapisu, więc zapis trwający dłużej niż interwał powoduje opóźnienie rozpoczęcia następnego zapisu i dryft odstępu. Zapisanie danych surowych wraz z odczytem czujnika światła (`processing="raw"`; odczyt jest zapisywany domyślnie) sprawia, że każdy zapis jest krótki, a każdą klatkę można później przekształcić na odbicie.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**ustala częstotliwość od samego początku;**`arr.set_target_fps(fps)`**zmienia ją w trakcie lotu bez konieczności ponownego łączenia, a**`arr.set_target_fps(None)`** przywraca automatyczną regulację. Szybkość przekraczająca możliwości hosta i sieci jest obniżana do tego limitu — sprawdź `clamped` w wynikach — a najniższa szybkość wynosi 0,1 klatki na sekundę.
- **Nie używaj `burst()` do rzadzenia klatek.** Zapisuje on każdą klatkę generowaną przez tablicę; działa on jednak zgodnie z ustaloną wartością `target_fps`, więc jest to właściwe narzędzie tylko wtedy, gdy potrzebne są wszystkie klatki.
- **`target_fps` nie naprawia uszkodzonych klatek.** Zmniejszenie tej wartości ogranicza liczbę generowanych klatek, a nie szybkość wysyłania poszczególnych klatek; w przypadku klatek uszkodzonych przez GVSP należy zamiast tego zmniejszyć wartość `wire_ceiling_mbps` (zobacz [Stan macierzy](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Dołączanie do już podłączonej macierzy — `attach_array`

Jeśli macierz jest już uruchomiona (została otwarta przez GUI lub poprzednia sesja SDK wywołała `connect_array`), należy użyć `attach_array`, aby pobrać do niej uchwyt zamiast ponownie się łączyć. W takiej sytuacji `connect_array` zawsze zwraca błąd „Kamera  <sn>znajduje się już w macierzy<id>”, ponieważ wysłanie żądania POST o numerze `/array/connect` dla elementu należącego do puli nie jest idempotentne; `attach_array` odczytuje `/api/camera/array/list` i dopasowuje na podstawie identyfikatora tablicy (array_id) lub numerów seryjnych.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Wzorzec: skrypty SDK, które współdziałają z graficznym interfejsem użytkownika (GUI) na komputerze stacjonarnym, powinny najpierw spróbować `attach_array`, a jeśli w puli nie ma jeszcze żadnej tablicy, przejść na `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Podłączona macierz jest wypożyczona, a nie własnością.** Uchwyt z `attach_array` nigdy nie zwalnia macierzy: ani wyjście z bloku `with`, ani jawne wywołanie `disconnect()` nie zwalniają jej (jawne wywołanie wyświetla jeden wiersz na stderr informujący, że tablica nadal działa). Ta sama zasada dotyczy `CameraSession` / `DAQSensorSession`. Współdzielenie z GUI jest więc bezpieczne zarówno z `with`, jak i bez niego:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Jeśli chcesz faktycznie zakończyć działanie — na przykład, aby ponownie połączyć się z innym `wire_ceiling_mbps` — wywołaj `arr.force_disconnect()`. Ponowne połączenie po zwykłym wywołaniu `disconnect()` na podłączonym uchwycie kończy się niepowodzeniem z komunikatem „Kamera  znajduje się<sn> już w tablicy<id>”, ponieważ nic nie zostało zwolnione.

### Narzędzie do analizy sieci

Przydatne przed otwarciem tablicy — pozwala sprawdzić, czy proponowane ustawienia będą pasować:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nigdy nie uruchamia backendu: ten, który nie odpowiada, generuje błąd `ChlorosConnectError`, podczas gdy odrzucone żądanie zwraca wynik. Nieprawidłowy błąd `packed` generuje błąd `ValueError`.

`status` jest jednym z: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (w przeciwnym razie `error` — w tym odrzucone żądanie: brak logowania, `error_code: AUTH_REQUIRED` lub plan poniżej progu minimalnego określonego w SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` oznacza, że żądana rozdzielczość pasuje do pierścienia RX tylko przy ograniczonej częstotliwości wyzwalania — zachowaj rozdzielczość i przejdź od `target_fps=result["recommended"]["recommended_target_fps"]` do `connect_array` (zobacz [Przykład 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` pojawia się tylko wtedy, gdy żądanej klatki nie da się dostarczyć przy żadnej częstotliwości klatek; program connect nigdy sam nie zmniejsza klatek, więc należy samodzielnie zastosować `recommended`.

**Jak odczytywać projekcję** (ten sam model co w panelu ustawień macierzy GUI):

- **Seria (`frame_bytes_total`) jest sumowana dla każdej kamery w rzeczywistym formacie pikseli danej kamery.**Kamery Mono**M3M**przesyłają strumień w formacie Mono12 (2 b/px) lub, przy zastosowaniu `pixel_format`, w formacie Mono swojej głębokości (`"BayerRG8"` to `Mono8`, 1 bit/piksel), więc w przypadku braku tego formatu klatka w pełnej rozdzielczości z czterech kamer ma rozmiar**~25 MB** przy trzech kamerach mono, a nie ~12,6 MB, jak wynikałoby z założenia, że wszystkie są 8-bitowe. Backend rozpoznaje format każdej kamery na podstawie jej modelu.
- **Admittance (`burst_fits_nic_ring`) uwzględnia obciążenie**, a nie na zasadzie „cała seria vs. pierścień”: tryb symulacji emisji (sim-emit) jest stosowany, gdy host opróżnia pierścień odbiorczy (RX ring) szybciej niż kamery go wypełniają. Host 10G + kamery 1 GbE**przesyłają** obraz w pełnej rozdzielczości, nawet gdy seria danych przekracza pojemność pierścienia; natomiast host 1 GbE blokuje (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` to konserwatywny pułap pobierania szeregowego** — `max(readout+emit, N×emit)` z emisją na kamerę ograniczoną do łącza 1 GbE, niezależnie od ekspozycji. Np. ~2,8 klatek na sekundę dla matrycy 4-kamerowej o pełnej rozdzielczości i 12 bitach (zgodne z wartościami ~2,7–3,0 zmierzonymi w środowisku uruchomieniowym). Pełny model: [Dokumentacja CLI→ Model klatek na sekundę i serii zdjęć dla macierzy](cli-reference.md#array-fps--burst-model).
- **Po zakończeniu ciągłego przechwytywania projekcja podaje wartość, na której się ustabilizowała.** Przechwytywanie trwające 30 s lub dłużej przy automatycznej częstotliwości rejestruje częstotliwość, na której ustabilizowała się przy danej wielkości macierzy na tym komputerze; pomijane są krótkie próby kontrolera częstotliwości z zastosowaniem szybszej częstotliwości. Od tego momentu w wierszu `array-connect`, analiza `connect_array()` oraz panel ustawień tablicy podają tę częstotliwość, gdy jest ona niższa od wartości modelowej, wraz z informacją o tym, co ją ograniczyło w `cycle_limiter`: `"clock"` (tablica działała z częstotliwością, na której się ustabilizowała, a pętla przechwytywania czekała na zegar wyzwalający), `"loop"` (pętla przechwytywania nie miała już czasu), `"capture"` (przechwytywanie pozostawało w tyle i pomijało grupy) lub `"measured"` (nie ma informacji, która z tych sytuacji miała miejsce). Wartość modelowa znajduje się w `wire_fps_max`. Tablica nadal rośnie do wartości przekazywanej przez łącze.
- **Nadmierna subskrypcja (`oversubscribed: true`) oznacza, że łącze nie jest w stanie przesyłać tablicy przy żadnej częstotliwości, jaką osiągają kamery** — pola fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) przyjmują wartość 0, a binning nie jest w stanie tego naprawić (zmniejsza ona liczbę bajtów na klatkę, a nie liczbę bajtów na sekundę przy danej częstotliwości). Samo wolne łącze nie powoduje tego problemu: zamiast tego spada liczba klatek na sekundę, a jeśli nie da się wystarczająco spowolnić pracy kamer, poziom staje się `slip-emit-and-capture` (szeregowy). Rozwiązaniem jest zmniejszenie liczby kamer, użycie ramek typu jumbo lub szybsza karta sieciowa; `max_cams_collision_safe` wskazuje maksymalną przepustowość posiadanego łącza i kamer. Odpowiedź zawiera również kody `aggregate_demand_bps`, `collision_safe_ceiling_bps` i `per_cam_floor_bps`. Zobacz [Nadmierna subskrypcja](#over-subscription-the-per-cam-floor).

### Wykrywanie i wyświetlanie listy

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Każda z nich przyjmuje wyłącznie słowa kluczowe `backend_url=` i `timeout=30.0` oraz zwraca listę słowników. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) przyjmuje również `auto_start_backend=True`; wywołania `list_*` nigdy nie uruchamiają backendu.

---

## Smart-AE / Smart-Capture

Macierze LATTICE uruchamiają ciągłą ekspozycję automatyczną (AE) w tle zaraz po podłączeniu, ale nowo wycelowana scena potrzebuje chwili, aby osiągnąć stabilność. **Smart-capture** to gotowe, wygodne rozwiązanie: sprawdza ekspozycję każdej kamery, czeka, aż macierz ustabilizuje się w całym oknie, a następnie uruchamia przechwytywanie. Działa to analogicznie do interfejsu graficznego: przycisk „smart” capture w aplikacji komputerowej wywołuje ten sam punkt końcowy backendu.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

W przypadku sterowania za pomocą `ChlorosProject` (następna sekcja) dostępnych jest więcej opcji regulacji:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Polityka inteligentnej ekspozycji automatycznej (smart-AE) jest domyślnie konserwatywna. Zmniejsz wartość `exposure_tolerance_pct` w przypadku wymagających prac radiometrycznych; zwiększ ją w przypadku szybko zmieniających się scen, gdzie wystarczy, że wynik będzie „wystarczająco bliski”.

---

## Sesje czujników DAQ

Trwała pula backendów dla czujników spektralnych (DAQ-U przez USB, DAQ-M przez BLE, DAQ-E przez Ethernet). Odzwierciedla strukturę kamery: inteligentne wykrywanie, ponowne wykorzystanie puli, idempotentne podłączanie.

### Inteligentne wykrywanie (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorytet: Ethernet → BLE → USB. Przekaż dowolną wyraźną wskazówkę, aby przypiąć protokół transportowy.

### Przypięty protokół transportowy

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` określa jeden czujnik i tylko ten czujnik jest kiedykolwiek otwierany. Jest on porównywany z czujnikami już otwartymi, a następnie nazwa hosta lub adres IP urządzenia DAQ-E (urządzenie DAQ-E, które skanowanie sieci wykrywa wyłącznie na podstawie adresu IP, jest pytane — w trybie tylko do odczytu — o swoją nazwę, adres MAC i numer seryjny, więc każdy z nich również go wykrywa), adres MAC BLE urządzenia DAQ-M oraz identyfikator zgłaszany przez urządzenie DAQ-U lub DAQ-M po otwarciu (każde inne otwarte w celu sprawdzenia urządzenie jest ponownie zamykane). Jeśli nie zostanie znalezione, `connect_daq_sensor` generuje komunikat `ChlorosConnectError` zawierający listę znalezionych elementów wraz z adresami (`port=` / `mac=` / `eth_host=`), na którym znajduje się inny czujnik, również generuje komunikat. Nigdy nie przechodzi na pierwszy lepszy widoczny czujnik — do tego służy wywołanie z flagą zerową.</id></sn></id></sn>

### `connect_daq_sensor()` Sygnatura

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Czujnik, który już znajduje się w puli, jest zwracany w niezmienionej postaci (pożyczony): nie są do niego stosowane: `integration_time`, `frame_avg` i `enable_ae`. Najpierw należy go odłączyć, aby otworzyć go z własnymi ustawieniami. Wyjątkiem jest `cap_id`: na wypożyczonym czujniku jest on ustawiany za pomocą `set_cap()`. W przypadku pominięcia obowiązują domyślne ustawienia zaplecza (patrz uwaga pod tabelą metod).

### Metody `DAQSensorSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Podsumowanie wpisu do puli (stan transmisji/nagrywania, zakres długości fal, klucze kalibracyjne, czas integracji, frame_avg, stan AE). W przypadku urządzenia DAQ-E, którego strumień przestał nadawać: `reconnect` (gdzie działa automatyczne ponowne łączenie się modułu zaplecza: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` podczas przesyłania), `stream_lost_s`, `reconnects` oraz `last_reconnect`, a także `record_gaps` dla trwającego nagrywania; `rx_stream_fault` wskazuje stan ponownego połączenia do momentu nadejścia pierwszego widma. |
| `latest(*, n=1, timeout=10.0)` | Zwraca do N najnowszych ramek widma (każda `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Wznowienie / wstrzymanie transmisji strumieniowej (uchwyt pozostaje otwarty). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Rozpocznij nagrywanie pliku .daq. Zwraca ścieżkę do pliku. Nie działa w przypadku DAQ-U/M bez pakietu kalibracyjnego AWS (z wyjątkiem DAQ-E). |
| `record_stop()` | Zatrzymuje nagrywanie. Zwraca `{path, rows, warning, gaps}`; `warning` to `None` w przypadku poprawnego nagrania, w przeciwnym razie zwraca komunikat diagnostyczny modułu zaplecza (na przykład pusty plik lub lukę w strumieniu), który jest również wyświetlany na stderr. `gaps` wyświetla każdy odcinek, w którym strumień nie dostarczył żadnych danych: `start_utc`, `end_utc` (`None`: nadal utracone przy zatrzymaniu), `duration_s`, `reconnected`, `marked_in_file`. W przypadku ciągłego zapisu jest to puste. |
| `set_cap(cap_id, timeout=15.0)` | Ustaw profil korekcji cap stosowany do odczytów począwszy od następnego: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` lub `"none"` (lub `None`) w przypadku czujnika bez osłony. Zwraca podsumowanie czujnikapodsumowanie czujnika, które odczytuje `cap_id`. Urządzenie DAQ-E-S utrzymuje wartość `as_recorded` niezależnie od zapytania (posiada wbudowany dyfuzor). Generuje wartość `ChlorosConnectError` dla profilu, który się nie ładuje. CLIA: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Tylko DAQ-E: ponowne włączenie PTP dla urządzenia, którego PTP jest wyłączone (`status()["rx_ptp_enabled"]` to `False`; ustawienie jest zapisywane w urządzeniu, więc pozostaje wyłączone do momentu ponownego włączenia). Zwraca zaktualizowany stan. Generuje błąd `ChlorosConnectError` w przypadku DAQ-U / DAQ-M lub gdy urządzenie nie potwierdzi. Program Chloros nigdy samoczynnie nie włącza urządzenia ponownie, ponieważ wyłączenie może być zamierzone. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Tylko DAQ-E: własne ustawienia urządzenia. `firmware` (`running`, `bundled`, `update_available`), `ip` oraz `ip_mode` (`dhcp` / `linklocal` / `static`: to, co działa obecnie), `network` (zapisane ustawienie: `mode` oraz `ip` / `mask` / `gateway` w trybie statycznym), `network_change_pending` (prawda, dopóki urządzenie nie uruchomi zapisanego ustawienia, takiego, które zawiera `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` to `None`, gdynie udało się odczytać wersji urządzenia. Gdy strumień danych z urządzenia nie dociera do tego komputera, jest on odczytywany przy nowym połączeniu, a słownik dodaje `stream_silent` (`True`), `stream_fault` i `reconnect`; urządzenie, które nie odpowiada, generuje komunikat. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | Tylko DAQ-E tylko: DHCP (`"auto"`) lub adres statyczny. Urządzenie zostanie ponownie uruchomione w celu zastosowania zmian i ponownego połączenia; bieżąca sesja działa dalej, a `address` śledzi urządzenie. `reboot=False` jedynie zapisuje ustawienia. Zobacz [Ustawienia urządzenia DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Tylko DAQ-E: przesył strumieniowy po włączeniu zasilania bez podłączonego oprogramowania (zapisane w urządzeniu). Zwraca potwierdzone ustawienie. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Tylko DAQ-E: ponowne uruchomienie urządzenia i ponowne podłączenie go z zachowaniem poprzednich ustawień. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Tylko DAQ-E: zaktualizuj oprogramowanie sprzętowe dołączone do tej wersji Chloros na urządzeniu z starszą wersją, a następnie podłącz je ponownie (trwa to około minuty; nie wyłączaj zasilania). Zwraca `updated`, `fw_before`, `fw_after`, `bundled` oraz `warning`, jeśli urządzenie powróciło z opóźnieniem lub z inną wersją; `updated` to `False` dla urządzenia, które ma już aktualną wersję. Generuje błąd `ChlorosConnectError` dla urządzenia, którego wersji nie można odczytać. CLI: `pool-update-firmware`. |
| `disconnect()` | Zwolnienie z puli. Brakdziałania dla uchwytów podłączonych, ale niebędących własnością. W przypadku czujnika otwartego przez tę sesję jest to `force_disconnect()`, a generuje błąd `ChlorosConnectError`, gdy zwolnienie nie zostanie potwierdzone (wyjście z bloku `with` nigdy nie generuje wyjątku). |
| `force_disconnect(timeout=30.0)` | Zwolnij czujnik niezależnie od tego, kto go otworzył — na przykład czujnik pozostawiony w puli przez skrypt, który uległ awarii, a do którego nowy skrypt ma dostęp jedynie jako do dołączonego uchwytu, którego `disconnect()` nie wykonuje żadnej czynności. Zakończenie działania czujnika innego klienta kończy wszelkie prowadzone przez niego rejestrowanie. Zwraca wynik, gdy backend potwierdzi zwolnienie lub gdy nie ma czego zwolnić (czujnik nie znajduje się w puli lub nie działa żaden backend); zwolnienie, które przekroczyło limit czasu lub zostało odrzucone przez backend, generuje błąd `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atrybuty: `sensor_id`, `model`, `transport`, `address` (następuje po DAQ-E, którego adres został zmieniony przez `set_network()`), `info` oraz `backend_url`.

> **Podaj nazwę nasadki.** Modele DAQ-U, DAQ-M i DAQ-E są dostarczane bez kondensatora, a Chloros nie jest w stanie stwierdzić, czy jest on zamontowany: czujnik podłączony bez `cap_id` otrzymuje wartość `sunshine_cosine`, co zgodnie z jego zapisami oznacza, że został wykonany (`cap_id_source=auto_default`). Nieosłonięty DAQ-U po zastosowaniu tej korekty wskazuje wartości około 30 razy wyższe, więc należy przekazać `cap_id="none"` dla nieosłoniętego czujnika (lub `set_cap("none")`), a w przeciwnym razie profil nasadki, którą zamontowałeś.

### DAQ-E-S — stały dyfuzor i czujnik nachylenia

Model **DAQ-E-S** to urządzenie typu DAQ-E z dwiema różnicami istotnymi dla oprogramowania. Wszystko inne — wykrywanie, łączenie, PTP, rejestrowanie oraz poniższe ustawienia urządzenia — działa dokładnie tak samo jak w przypadku DAQ-E.**Jego dyfuzor jest stały, więc nie wymaga korekcji nakładki.** Dyfuzor typu „Sunshine” jest wbudowany i znajdował się na urządzeniu podczas pomiaru kalibracji fabrycznej, dzięki czemu kalibracja ta już odzwierciedla rzeczywiste natężenie promieniowania padającego w dół. Chloros ustawia swój limit na `as_recorded` (bez profilu) niezależnie od tego, o co zostanie poproszony: limit o tej samej nazwie (`sunshine_cosine`, `fov_*`, `none`) jest ignorowany z wyświetleniem komunikatu, a w zapisach przechowywane są wartości `cap_id_source=model`, więc ostrzeżenie o domniemanym limicie nigdy się nie uruchamia. Nie należy też ręcznie nakładać ograniczenia na jego dane: `sunshine_cosine` ustawiony w pozycji pionowej wskazywałby wartość około 11 razy wyższą.

**Urządzenie jest wyposażone w 3-osiowy akcelerometr.** Charakterystyka cosinusowa dyfuzora światła słonecznego jest ważna tylko wtedy, gdy jest on wypoziomowany, więc przechylony czujnik zgłasza błędne natężenie promieniowania — a współczynnik odbicia dziedziczy ten błąd — bez żadnych innych objawów. Każdy odczyt z DAQ-E-S zawiera zatem informacje o jego położeniu:

| Pole | Znaczenie |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Kąty w stopniach. `None`, gdy oprogramowanie sprzętowe oznacza kąt jako nieokreślony (swobodny spadek, martwa oś). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Przyspieszenie w mili-g. `mag_mg` znacznie odbiegające od ~1000 oznacza, że urządzenie było w ruchu, a kąty dla tej próbki nie mają znaczenia. |
| `cal_applied` | Ustawione: osie i kąty są w układzie odniesienia dyfuzora (zastosowano kalibrację montażową urządzenia). Wyczyść: surowy układ odniesienia płytki, który może wykazywać nachylenie do ~2,3°, które nie jest rzeczywiste. |
| `sample_age_ms` | Wiek próbki z akcelerometru w momencie wysłania widma. |
| `temp_raw` | Nieskalibrowany **trend** temperatury matrycy — wyłącznie różnice, nigdy odczyt termometru. |

`DAQSensorSession.latest()` zwraca je pod kluczem `imu` każdej ramki (`None` w przypadku DAQ-U, DAQ-M lub zwykłego DAQ-E). Dane `.daq` zarejestrowane z urządzenia DAQ-E-S przechowują te same wartości w kolumnach `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) obok każdego widma, dzięki czemu czytelnik może odróżnić chmurę od przewróconego statywu. W przypadku wszystkich pozostałych modeli kolumny te są puste.

Strumień danych o położeniu przekazuje informacje o własnych polach w `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), oddzielonych od pól spektralnych, dzięki czemu awaria akcelerometru nigdy nie jest odczytywana jako luka w zapisie. `discover_daq_sensors()` oraz `model` sesji wskazują `DAQ-E-S`.

### Ustawienia urządzenia DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **W sieci tego komputera musi znajdować się adres statyczny.** `set_network` generuje błąd `ChlorosConnectError` w przypadku urządzenia, którego podsieć nie obejmuje adresu tego komputeraw sieci czujnika: po ponownym uruchomieniu żadne urządzenie nie będzie mogło połączyć się z czujnikiem w celu przywrócenia poprzednich ustawień. Ustawienie `force=True` należy zastosować tylko wtedy, gdy ten komputer zostanie przypisany do nowej podsieci lub przekierowany do niej.
- **`set_network`, `reboot` i `update_firmware` powodują ponowne uruchomienie czujnika** i generują komunikat `ChlorosConnectError` podczas rejestracji, oraz gdy nie powróci w ciągu `comeback_timeout` sekund. Identyfikator czujnika i nazwa hosta `daq-e-<id>.local` nigdy się nie zmieniają.
- **Nagranie rozpoczęte tuż przed ponownym uruchomieniem jest przez nie zamykane i nazywane.** Gdy czujnik wraca do działania, zwracany słownik zawiera `recording_closed` (ścieżkę do pliku) oraz `warning` potwierdzający ten fakt. Gdy czujnik nie wraca lub transfer oprogramowania układowego kończy się niepowodzeniem, informację tę przekazuje zamiast tego `ChlorosConnectError`: jego komunikat kończy się ostrzeżeniem, a ustawiane są wartości `e.recording_closed` / `e.warning` są ustawione (oba `None`, jeśli żadne nagranie nie zostało zamknięte). Uruchom nagrywanie ponownie, gdy czujnik będzie z powrotem.
- **Podłączany jest wyłącznie sam czujnik.** Po ponownym uruchomieniu urządzenie musi odpowiedzieć, gdzie się znajdowało — a po `set_network("auto")` również pod swoją nazwą `.local` — podając swoją nazwę, nazwę hosta i model, na podstawie posiadanego adresu MAC karty sieciowej; `address` staje się adresem IP, który udzielił odpowiedzi. Odpowiedź pod daną nazwą jest uznawana wyłącznie przy tym adresie MAC, więc z komputera, który łączy się z czujnikiem przez router, lub z domyślnej sieci NAT WSL2, czujnik przeniesiony przez DHCP nie zostanie ponownie podłączony (`ChlorosConnectError`; należy ponownie połączyć się z nim pod nowym adresem).
- **Aktualizacja oprogramowania sprzętowego wymaga ponownego połączenia się czujnika.** `update_firmware()` wysyła zaproszenie do urządzenia, a urządzenie otwiera połączenie TCP z powrotem do tego komputera w celu pobrania obrazu, na porcie wybranym przez system operacyjny. Serwer backend nasłuchuje na karcie sieciowej, która łączy się z urządzeniem (na każdej karcie, tylko wtedy, gdy nie można określić, która to jest) i obsługuje wyłącznie to urządzenie: wszelkie inne połączenia są odrzucane (`[DAQ-FW] … refused a firmware connection from <ip>` w wyjściu serwera zaplecza), podczas gdy serwer nadal czeka. Zapora sieciowa blokująca połączenie powoduje niepowodzenie aktualizacji z komunikatem `device never connected back on TCP <port>`. Instalator systemu Windows dodaje odpowiednią regułę (patrz [Krok 1](#windows-exe)); w przypadku innej zapory sieciowej należy zezwolić na przychodzący ruch TCP do serwera zaplecza z podsieci czujnika.
- **W środowisku WSL2 należy użyć sieci lustrzanej** (`networkingMode=mirrored` w `.wslconfig`): w domyślnym trybie NAT każdy adres statyczny jest odrzucany (własny adres WSL nie znajduje się w sieci czujnika), strumień nie dociera do WSL, a aktualizacja oprogramowania układowego nie może zostać zakończona.
- Interfejs graficzny utrzymuje własne połączenie z czujnikiem: ponowne uruchomienie za pomocą skryptu kończy sesję interfejsu graficznego z tym czujnikiem. Interfejs graficzny ma te same ustawienia (zakładka „Czujniki światła”, ustawienia czujnika, „Sieć”).

### Wykrywanie — znalezienie adresu do połączenia

`discover_daq_sensors()` skanuje porty USB / BLE / ETH w poszukiwaniu czujników, które *można* otworzyć. Jest to odpowiednik `discover_lattice_cameras()` dla modułów DAQ i jedyny sposób na uzyskanie **adresu MAC BLE modułu DAQ-M** — urządzenie DAQ-E ma nazwę hosta, a DAQ-U port COM, ale adres MAC nie jest ani wydrukowany na urządzeniu, ani wyświetlany przez system operacyjny.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Pole | Opis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / adres MAC BLE / nazwa hosta — przekazać do `connect_daq_sensor` jako `port=` / `mac=` / `eth_host=`. |
| `display` | Etykieta czytelna dla człowieka. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, lub `None`. Czujnik Ethernet to `DAQ-E-S` lub `DAQ-E`, ponieważ zgłasza się na żądanie, w trybie tylko do odczytu, podczas skanowania (`DAQ-E`, gdy nie odpowiada na czas). Port szeregowy USB wyświetla wartość `DAQ-U`, gdy jego układ mostkowy jest jednym z tych używanych przez DAQ-U (FTDI, CH340, CP210x, Prolific) — wskazówka z opisu portu, a nie sondy — oraz `None` dla każdego innego urządzenia USB, które nadal znajduje się na liście, więc można je przekazać jako `port=`. Port szeregowy bez identyfikatora producenta USB i bez takiego układu mostkującego (UART na płycie głównej, taki jak `/dev/ttyS*` w systemie Linux, szeregowe łącze Bluetooth, Intel AMT) nie może być urządzeniem typu DAQ-U i nie jest wymieniony na liście. |
| `extra` | Szczegóły dotyczące poszczególnych protokołów (nazwa reklamowana BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; adres IP/oprogramowanie sprzętowe DAQ-E/…). Puste wartości są pomijane. |

| Parametr | Wartość domyślna | Opis |
| --- | --- | --- |
| `transports` | wszystkie trzy | Sekwencja (lub ciąg znaków w formacie CSV) ograniczająca skanowanie. Warto podać tę wartość, gdy wiesz, czego chcesz — BLE jest najwolniejszym ogniwem. |
| `scan_timeout` | 5 | Okno skanowania na transport w sekundach; backend ogranicza wartość do zakresu 1–20. |
| `timeout` | 60,0 | Limit HTTP dla całego wywołania (tak jak w pozostałych częściach SDK). |
| `auto_start_backend` | `True` | Uruchamia lokalny backend, jeśli żaden nie jest uruchomiony. Nigdy nie uruchamia się dla zdalnego `backend_url`. |

> **Czujniki już otwarte w puli nie są wyświetlane.** Podłączone urządzenie peryferyjne BLE przestaje nadawać sygnał, a otwartego portu COM nie da się wykryć, więc lista wykrytych urządzeń zawiera tylko te, które są *dostępne do połączenia*. Pusty wynik zaraz po podłączeniu urządzenia jest oczekiwany — użyj `list_daq_sensors()` dla urządzeń, które już posiadasz. Transporty, których skanowanie nie może zostać uruchomione (brak zainstalowanego bleak / zeroconf), są pomijane zamiast generowania błędu, więc komputer bez Bluetooth nadal otrzymuje odpowiedzi dotyczące USB i ETH.

### Lista

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Który moduł DAQ obsługuje którą kamerę — `daq_bindings()`

Odbicie światła z danego ujęcia oraz zapisywany plik `.daq` wykorzystują moduł DAQ powiązany z tą kamerą lub matrycą, a w przeciwnym razie najnowszy moduł DAQ dostępny w backendzie. Wybór czujnika światła w interfejsie graficznym oraz plik `chloros-cli daq pool-bind` ustalają to; podobnie jak:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` zwraca `{"cameras": {serial: b}, "arrays": {array_id: b}}`: każdy `b` zawiera `binding` (ustaloną wartość), `source` (`project`, z `sensors.json` projektu lub `runtime`), `known` / `known_as` (czy nazwa klucza kamery lub tablicy jest znana Chlorosowi i w jaki sposób: `connected`, `project` — `cameras.json` otwartego projektu — lub, w przypadku kamery, `seen`: otwierany wcześniej na tym komputerze; `known: False` dla klucza, który niczego nie określa) oraz `resolved` (czujnik puli, do którego jest obecnie przypisany: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Powiązanie w czasie wykonywania ma pierwszeństwo przed plikiem projektuna czas trwania sesji; plik projektu nie ulega zmianie. Odłączenie kamery, macierzy lub urządzenia DAQ powoduje usunięcie wszystkich powiązań z tym sprzętem, niezależnie od tego, kto je ustawił (elementy macierzy są powiązane z samą macierzą).

`set_daq_bindings()` wiąże wyłącznie kamerę, która jest podłączona, znajduje się w otwartym projekcie lub była wcześniej otwarta na tym komputerze, oraz tablicę, która jest podłączona lub znajduje się w otwartym projekcie (lub klucz, który został już powiązany). Każdy inny klucz wywołuje błąd `ChlorosConnectError`, podając jego nazwę (`400`, `code`, `UNKNOWN_BINDING_TARGET` oraz `unknown`), a **nic** w tym wywołaniu nie jest przypisane. `None` usuwa dowolny klucz.

### Odbicie otoczenia + obiektu — `light_sensor_reflectance()`

Dwa połączone urządzenia DAQ, z których jedno skierowane jest w górę, w stronę nieba (środowisko), a drugie w dół, w stronę obiektu (obiekt), dostarczają współczynnik odbicia obiektu oraz wskaźniki wegetacji na podstawie swoich najnowszych odczytów: para „Środowisko + Obiekt” w interfejsie graficznym oraz `chloros-cli daq reflectance` bez konieczności ponownego uruchamiania czujników. Oba muszą być podłączone i przesyłać dane.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Pasma.** Refleksja każdego pasma to zintegrowane natężenie promieniowania obiektunad promieniowaniem otoczenia. `daq_band_presets(filter_code=None)` zawiera listę zestawów pasm: po jednym na filtr LATTICE M3C (`RGN`, `OCN`, `NGB`, model pasmowy właściwy dla kamery, więc NDVI z systemu DAQ i NDVI z kamery to ta sama wielkość) oraz `GENERIC` szerokopasmowy. Jego `suggested` to ustawienie wstępne odpowiadające `filter_code`, w przeciwnym razie filtr pierwszej podłączonej kamery (`suggested_from` określa, który).
- **Zwraca** `bands` (dla każdego pasma: `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (wbudowanewbudowane), `custom` (każdy z `value` lub `valid: False` i `error`), `invalid_bands`, `reflectance_type`, `preset` oraz odczytów `timestamps` (`ts_ns`). Zakres, którego czujnik otoczenia nie był w stanie zmierzyć (poza zakresem pomiarowym lub na poziomie szumu tła), jest uznawany za nieważny wraz z podaniem przyczyny, a każdy wskaźnik, który tego wymaga, jest pomijany i nigdy nie jest obliczany względem zera.
- **Pozorne lub względne.** Wskaźniki zależne od skali (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) wymagają stosunku bezwzględnego, więc są generowane tylko wtedy, gdy oba czujniki są skalibrowane pod kątem mocy (każdy posiada `calibration_bundle_sha`): `"Apparent Reflectance"`. W przeciwnym razie wynikiem jest `"Relative Reflectance"`, zawierający wyłącznie wskaźniki niezmienne względem skali.
- **Dwie różne siatki długości fal** są porównywane pod kątem ich nakładania się: punkty czujnika otoczenia znajdują się w zasięgu obiektu, a widmo obiektu jest na nie interpolowane.
- **Dwa czujniki, nigdy jeden.** Nazwanie jednego czujnika jako obu (ten sam identyfikator, w dowolnej pisowni akceptowanej przez pulę) powoduje błąd `ChlorosConnectError`: odczyt czujnika na samym sobie wynosi 1,0 w każdym paśmie, co oznacza brak pomiaru. Backend również to odrzuca (`POST /api/daq/indices` odpowiada na `400`, `code` na `SAME_SENSOR`, podczas gdy `ambient_sensor_id` i `object_sensor_id` wysyłane przez SDK dotyczą jednego czujnika w puli, np. jego identyfikator i port).

### Współdzielenie z interfejsem CLI i innymi skryptami

Pula czujników DAQ backendu jest współdzielona przez wszystkich klientów CLI i SDK na danym komputerze. Jeśli czujnik znajduje się już w puli — otwarty przez `chloros-cli daq pool-connect` lub inny skrypt — wywołanie `connect_daq_sensor(port="COM3")` z języka Python zwraca pożyczony uchwyt: jego `disconnect()` nie wykonuje żadnej operacji, więc skrypt nie odbiera czujnika klientowi, który go otworzył, przy zamknięciu okna. To samo dotyczy funkcji `sensor_id=` (wpisanej jako `:` lub `-` — obie formy są dopuszczalne) w przypadku urządzenia DAQ-E, którego nazwa pochodzi od adresu IP, gdy został otwarty przez nazwę hosta (lub odwrotnie), oraz w przypadku inteligentnego wykrywania z flagą zerową, gdy backend odpowiada czujnikiem, który już posiadał — blok `with` zwalnia wyłącznie czujnik, który sam otworzył.

Interfejs graficzny Chloros **nie** jest częścią tej puli: jego panel DAQ utrzymuje własne połączenie z czujnikiem, więc `connect_daq_sensor` nigdy nie pożycza czujnika otwartego przez interfejs graficzny. Nie należy korzystać z tego samego czujnika jednocześnie z poziomu GUI i skryptu. Urządzenie typu DAQ-U lub DAQ-M, które jest obecnie zajęte przez GUI, zazwyczaj po prostu nie da się otworzyć, ale moduł DAQ-E akceptuje drugą sesję: połączenie nawiązane przez skrypt zmienia ustawienia akwizycji urządzenia w interfejsie GUI, a jego `disconnect()` zatrzymuje strumień danych z urządzenia, co powoduje zakończenie podglądu na żywo w interfejsie GUI oraz wszelkich prowadzonych przez niego nagrań. Najpierw należy zamknąć czujnik w interfejsie GUI.

### PTP Grandmaster — `time_sync_status()`

Backend obsługuje serwer PTP Grandmaster, z którego kamery LATTICE i urządzenia DAQ-E pobierają swoje znaczniki czasu. Uruchamia się on na żądanie: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` oraz `connect_all()` uruchamiają go, jeśli nie działa — w tym w przypadku połączenia, które następnie kończy się niepowodzeniem, np. gdy `connect_all()` próbuje połączyć się z nieistniejącym modułem DAQ-E — podobnie jak `test_array_sync()` oraz każde wywołanie, które otwiera kamery w celu wykonania jednego polecenia (`capture_cameras_transient()`, `start_multi_camera_stream()`, wywołania związane z wyrównywaniem, `capture_live_index()`).

- **Gdzie to ogłasza.** Na przewodowych kartach sieciowych podłączonych do kamer (w tym adres link-local `169.254.x.x`), w przeciwnym razie na karcie sieciowej trasy domyślnej, jeśli jest ona podłączona przewodowo. Nigdy na Wi-Fi ani na karcie wirtualnej (Tailscale, WSL): za żadną z nich nie znajduje się kamera ani moduł DAQ-E. Jeśli żadna taka karta nie jest aktywna, ogłoszenia są wysyłane na pustą kartę (`health` `NO_SOCKETS`), a po podłączeniu kolejnej kamery lub urządzenia DAQ następuje przypisanie karty do tego urządzenia.
- **Po utracie połączenia.** Gdy karta sieciowa kamery traci połączenie, a następnie je odzyskuje (w wyniku wyrwania kabla, wymiany przełącznika lub ponownej identyfikacji karty USB), serwer główny samodzielnie odbudowuje swoje gniazda bez konieczności ponownego uruchamiania, i ponownie sprawdza wszystkie podłączone urządzenia DAQ-E, ponownie aktywując te, które utknęły w stanie zawieszenia. Odbudowuje się również, gdy wszystkie obsługiwane przez niego kamery i urządzenia DAQ-E pozostają nieaktywne przez 15 sekund.
- **Czy dochodzi do przewodu?** Odczytaj `time_sync_status()` (`GET /api/time-sync/status`). `state` informuje jedynie, że wątek urządzenia nadrzędnego działa; `health` wskazuje, czy jego pakiety są wysyłane: `OK` (nie wykryto usterki), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (niektóre karty sieciowe nie działają) lub `PEERS_SILENT` (kamery i urządzenia DAQ-E, które obsługiwał, przestały odpowiadać), wraz z komunikatem w `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` zliczają tylko wiadomości, które zostały wysłane; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` i `last_delay_req_age_s` opisują te, które nie zostały wysłane, a `rebinds` / `last_rebind` – operacje odbudowy. `sync_sends_slow` zlicza operacje synchronizacji, których wysyłanie trwało dłużej niż 200 us, a `sync_send_max_us` to najdłuższa z nich; czas rozpoczęcia każdego Follow_Up liczy się maksymalnie jako 200 us od momentu jego wysłania. Grandmaster działa w osobnym procesie (`own_process`, `process_pid`), z wysokim priorytetem w systemie Windows (`process_priority`), dzięki czemu obciążony backend nie może opóźniać czasów, które stempluje, ani synchronizować zegarów wszystkich kamer jednocześnie; jeśli proces ten nie może zostać uruchomiony, działa on wewnątrz backendu tak jak poprzednio (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funkcja | Opis działania |
| --- | --- |
| `time_sync_status()` | Stan, kondycja, identyfikator zegara, priorytety i liczniki komunikatów serwera nadrzędnego (powyżej). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Obserwowane przez niego urządzenia podrzędne PTP (kamery, urządzenia DAQ-E): `peers`, każdy z `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Każda podłączona kamera matrycowa: `PtpStatus`, `PtpOffsetFromMaster` oraz `PtpMeanPathDelay` (kamera, której oprogramowanie sprzętowe nie zawiera węzłów PTP, posiada wyłącznie numer seryjny). `time-sync cameras`. |
| `restart_time_sync()` | Zatrzymaj i uruchom ponownie serwer nadrzędny; po tym PTP każdego urządzenia DAQ-E w puli jest ponownie sprawdzane (`daq_ptp_rechecked`). Generuje sygnał `ChlorosConnectError`, gdy nie powróci. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Jego priorytety PTP (0–255; niższa wartość wygrywa w wyborach na najlepszego mistrza) dla sieci z innym mistrzem PTP. Wymaga, aby ten działał. `time-sync set-priority`. |

Każda z nich pobiera `backend_url=` i `timeout=` oraz uruchamia lokalny backend, który nie działa.

---

## Narzędzia do obsługi kamer

Czym zajmuje się `chloros-cli lattice` poza łączeniem się i przechwytywaniem, jako zwykłe funkcje w `chloros_sdk` (moduł `chloros_sdk.camera_utils`). Każda z nich wywołuje trasę backendu używaną przez polecenie CLI, zgodnie z regułami CLI, więc obie zachowują się podobnie. Przyjmują one **numer seryjny** kamery, a nie sesję, oraz te same słowa kluczowe `backend_url=` / `timeout=`, co wywołania funkcji smart-connect; jeśli backend nie działa, zostaje uruchomiony, tak jak w przypadku `connect_camera()`. Odmowa powoduje wygenerowanie kodu `ChlorosConnectError` wraz z przyczyną podaną przez backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funkcja | Opis działania |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Profil kolorów wyświetlacza: `raw`, `linear`, `natural`, `enhanced` lub `custom_temp` oraz jego temperatura barwowa (ograniczona do zakresu 2000–10 000 K; odczytywana, gdy profil to `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Nasycenie i kontrast wyświetlacza, po 0–200 % (100 = bez zmian); `reset=True` przywraca obie wartości do 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Zmniejsz jasność najjaśniejszych, najbielszych pikseli renderowanego obrazu: -100..0 (wartości są obcinane w kierunku zera, a następnie ograniczane; 0 = wyłączone). |
| `camera_white_balance_once(serial, settle_s=None)` | Jednorazowy balans bieli: kamera przesyła strumień na krótko, dokonuje konwergencji i zwraca `red_ratio` / `blue_ratio` (`ratios_read` False, jeśli nie może ich zgłosić). Zachowuje ustalony balans bieli; wszystkie pozostałe ustawienia zapisane przez otwarty strumień są przywracane w stanie, w jakim zostały znalezione. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (tryb ciągły) lub `"off"`; lub stałe współczynniki balansu `red` / `blue`, które jako pierwsze wyłączają tryb automatyczny. Współczynnik, którego kamera nie akceptuje, jest oznaczony jako `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Ustawienia wstępne kamery; po zastosowaniu jednego z nich kamera zachowuje ustawienia o nazwach tych ustawień wstępnych (odpowiedź to `settings`, ponieważ kamera odczytuje je później, np. częstotliwość klatek, z jaką pracuje; `None` dla pola, którego profil nie określa), a wszystkie pozostałe ustawienia zapisane w pliku otwartym zostają przywrócone do stanu, w jakim zostały znalezione. `"default"` resetuje kamerę do domyślnych ustawień fabrycznych (ustawienia fabryczne z automatyczną ekspozycją w trybie swobodnym, wyłączonym limitem przepustowości łącza i wyłączoną korekcją defektów, przy zachowaniu skalibrowanego balansu bieli kamery RGB), które nie są zapisywane w jej pamięci: odpowiedź zawiera `factory_defaults: True` oraz wszystkie pola z `settings`. W `camera_presets()`, `default` to `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Modele filtrów oraz szczegóły jednego modelu (pasma, współczynnik warunkowy odpowiedzi spektralnej). Model filtra kamery jest odczytywany z jej ciągu modelowego; nie ustawia się go ręcznie. |
| `camera_firmware_status(serial=None)` | Oprogramowanie układowe każdej kamery: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` oraz `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Wgranie oprogramowania układowego tej kompilacji Chloros do jednej kamery: przywrócenie ustawień fabrycznych, wgranie, ponowne uruchomienie, ponowne zapisanie podstawowych ustawień Chloros; kalibracja kamery zostaje zachowana. Trwa to kilka minut; kamera musi pozostać podłączona do zasilania. `force=True` ponownie wgrywa oprogramowanie do aktualnej kamery. |
| `camera_power_probe(serial)` | Węzły zasilania i temperatury kamery oraz jej temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Tryb czuwania o niskim poborze mocy kamery (pozostaje w stanie uśpienia do momentu wybudzenia: limit przepustowości łącza włączony (`DeviceLinkThroughputLimitMode` `On`) na minimalnym poziomie, włączony tryb oszczędzania energii, jeśli kamera go posiada), wraz z temperaturą przed i po; wybudzenie przywraca połączenie do stanu, w jakim było przed przejściem w tryb uśpienia (limit, tryb pracy oraz opóźnienie pakietów `GevSCPD`, które kamera utrzymuje na poziomie 0, gdy limit jest włączony; zapis jest przechowywany na tym komputerze, więc wybudzenie z późniejszego serwera również go odnajdzie), lub, w przypadku kamery, której ten komputer nie przełączył w tryb uśpienia, a która wygląda na uśpioną (limit włączony na poziomie minimalnym), z wyłączonym limitem i wartością `GevSCPD` pozostawioną na 0 przy 80 (ustawienia fabryczne); limit ustawiony powyżej wartości minimalnej pozostaje bez zmian. Zachowywany jest wyłącznie stan zasilania; wszystkie pozostałe ustawienia zapisane w pliku „open” są przywracane do stanu, w jakim zostały znalezione. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Przechwytuj klatki, dopóki w polu widzenia nie pojawi się cel kalibracyjny MAPIR, a następnie dopasuj współczynniki dla poszczególnych kanałów na podstawie jego paneli: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Karty sieciowe tego komputera obsługujące GigE Vision (MTU, prędkość łącza, ramki jumbo) oraz wszelkie `issues`; `estimate=True` dodaje liczbę klatek na sekundę i przepustowość, jakie uzyskałyby kamery `cameras`. |
| `fix_network_settings()` | Zastosuj ustawienia karty GigE Vision do adaptera kamery tego komputera. Zmienia to ustawienia adaptera i działa tylko wtedy, gdy program Chloros jest uruchomiony z uprawnieniami administratora; w przeciwnym razie wyświetla komunikat z dziennikiem wykrytych ustawień. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Skoryguj winietowanie obrazów w folderze, przy czym każdy z nich ma własną mapę płaskiego pola obiektywu: `processed`, `lens_models` (obiektyw, dla którego skorygowano każdy plik), `skipped` (wraz z uzasadnieniem). Bez `lens_model` obiektyw każdego pliku pochodzi z pola „Model” w danych EXIF; plik `lens_model` dotyczy plików, których model nie określa konkretnego aparatu, a plik, którego model jest z nim sprzeczny, jest pomijany. Tylko pliki. |
| `read_dls(mac, filter=None, camera_serial=None)` | Jeden odczyt z czujnika DLS (czujnik światła padającego z góry, Bluetooth): widmo, natężenie promieniowania w poszczególnych pasmach oraz wielkości pasmowe na podstawie charakterystyki własnej aparatu, gdy podano `camera_serial`. `discover_daq_sensors(transports=["ble"])` wyszukuje adres MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Sprzęt hosta przeznaczony dla kamer oraz zalecana przez niego konfiguracja (`report_text`), a także procesor / karta graficzna / środowisko uruchomieniowe PyTorch systemu. |

- **Niektóre wymagają wyłącznego dostępu do kamery.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` i `update_camera_firmware()` otwierają kamerę. Gdy Chloros ją zajmuje (podłączona w aplikacji, z `connect_camera()` jako elementem macierzy lub aktualnie używana przez inne polecenie, np. test tablicy lub funkcja przechwytywania otwierająca swoje kamery), backend odrzuca żądanie z kodem HTTP 409 `CAMERA_HELD` (zarówno w `code`, jak i `error_code`), wskazując, jak go zwolnić. Ustawienia kolorów, podświetlenia, `camera_power_probe()` oraz odczyty działają na podłączonej kamerze.
- **Kolor, podświetlenia i balans bieli dotyczą kamer RGB.** W przypadku kamery jednopasmowej monochromatycznej (M3M) funkcje ustawiające nie zmieniają niczego i zwracają wartość `{"skipped": True, "reason": ...}`, tak jak robi to CLI. Są to ustawienia wyświetlania: podgląd na żywo i zrzuty na poziomie wyświetlacza, nigdy natężenie promieniowania ani współczynnik odbicia.

---

## Tablice przejściowe, profile wyrównania i indeksy

Co robi `chloros-cli lattice` bez stałego połączenia, podobnie jak funkcje w `chloros_sdk` (moduł `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` oraz `index`. Każde z nich wysyła żądanie wysyłane przez polecenie CLI, z domyślnymi ustawieniami CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Same otwierają kamery.** Każde polecenie otwiera swoje kamery, wykonuje swoje zadanie i je zamyka, więc potrzebuje kamer, których Chloros nie obsługuje (podłączonych w aplikacji, z `connect_camera()` lub jako element tablicy); backend odrzuca żądanie z kodem HTTP 409 `CAMERA_HELD` i informuje, jak zwolnić kamerę. Tymczasowe otwarcie zajmuje trochę czasu (około 40 s dla kilku kamer). W przypadku powtarzających się przechwyceń `connect_array()` raz jest najszybszym rozwiązaniem i zapisuje każdy typ eksportu, podczas gdy przechwytywanie przejściowe zapisuje obraz po usunięciu efektu Bayer. `calibrate_alignment_profile()` stanowi wyjątek: w przypadku podłączonej macierzy tych kamer rozwiązuje problem na podstawie własnych klatek macierzy (`used_live_array`).
- **`serials`** to kamery, które należy otworzyć, przy czym pierwsza z nich uruchamia wyzwalacz; `None` otwiera wszystkie kamery w sieci.
- **`settings`** to słownik nadpisujących ustawień kamer, pola ustawiane przez flagi CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` oraz `preset` (ustawienie wstępne kamery o tej nazwie). Podanie wartości ekspozycji lub wzmocnienia wyłącza tę automatyczną kontrolę, podobnie jak flagi.
- **`gpu`**: `True` wymusza przetwarzanie warp na GPU, a `False` na CPU (`--gpu` / `--no-gpu`); `None` pozwala backendowi na wybór.
- **`keep_settings`**/**`clear_settings`** (każda funkcja otwierająca kamery, domyślnie `False`): na końcu każda kamera jest przywracana do stanu, w jakim została znaleziona; `keep_settings=True` zapisuje natomiast ustawienia przebiegu w pamięci samej kamery (tryb swobodny), `clear_settings=True` resetuje tę pamięć do ustawień fabrycznych — zobacz [Pamięć kamery](#camera-memory--keep_settings--clear_settings). Odpowiedź (dla zadania, `stop()`) zawiera `camera_memory`.
- **Ścieżki są ścieżkami backendu**: ścieżki względne są przekształcane na bezwzględne względem katalogu roboczego, gdy backend znajduje się na tym komputerze. Funkcje pliku profilu wyrównania (`alignment_profile_info()`, `reorder_alignment_profile()`) odczytują plik na tym komputerze, tak jak robi to interfejs CLI.

| Funkcja | Opis działania |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Prowadzi ciągłą rejestrację z jednej kamery do momentu zatrzymania. Zwraca `CliJob`. Klatki z przebiegu `tiff` stanowią plik `raw` — tablicę zapisanych danych z tej kamery (jedna płaszczyzna mozaiki przy, nieskompresowane); `png` i `jpg` zawierają 8 bitów, więc te serie to klatki po demosaikowaniu przeznaczone do przeglądania. Związany lub zgrupowany moduł DAQ zapisuje jeden plik `.daq` na klatkę w folderze serii(`daq_files` w statystykach). Każda klatka jest zapisywana jednokrotnie wraz z pełnymi metadanymi (model, czas przechwycenia, obiektyw, pasma, kalibracja na klatce surowej); `stop()` zwraca wynik po zapisaniu każdej już przechwyconej klatki (do 25 s) i dopiero wtedy kamera zostaje zamknięta. Kamera pozostawiona w trybie gotowości do wyzwalania sprzętowego (`TriggerMode` `On`) jest ustawiona na tryb swobodny na czas serii (odpowiedź startowa `trigger` informuje o tym, co zostało wykryte). Po zakończeniu kamera zostaje przywrócona do stanu, w jakim została znaleziona — z jej ustawieniami wyzwalania oraz wszystkimi ustawieniami zmienionymi podczas przebiegu — chyba że zostanie użyte `keep_settings=True`, które zapisuje ustawienia przebiegu w pamięci własnej kamery (tryb swobodny), lub `clear_settings=True`, który resetuje tę pamięć. Cykl, w którym nie zarejestrowano żadnej klatki, kończy się samoczynnie po 10 s z kodami `stop_reason` i `no_frames`, a przyczyna jest podana w `last_error`, a jego `stop()` zwraca ten powód jako `error`. Klatki pominięte z powodu niemożności nadążenia przez moduły zapisu są zliczane, a `drop_reason` podaje przyczynę. W przypadku kamery, którą trzyma Chloros, użyj `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Zsynchronizowane strumieniowo grupy klatek w `fps`, dla grup `count` lub do momentu zatrzymania. Zwraca `CliJob` (jego `start_reply` zawiera `n_cams` oraz przesyłane dane `pacing`); jego końcowe statystyki obejmują liczbę pobranych, zapisanych i odrzuconych grup, plików, których moduł zapisu nie mógł zapisać, oraz grup, dla których kamera nie dostarczyła żadnych danych. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Jedno zsynchronizowane przechwycenie, jeden obraz po usunięciu efektu Bayera na kamerę: `results` (`filepath`, `serial`, `frame_id`) oraz `pacing`. Pojawia się komunikat o brakującej kamerze, podający liczbę zapisanych plików. `lattice multi-capture` bez podłączonej macierzy. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` zsynchronizowane zdjęcia przy stałej ekspozycji: `sync_groups`, `master`, jeden wiersz `group_timestamps` na każdy sygnał wyzwalający (kamera, która opuściła jedno ujęcie, nie występuje w swoim wierszu) oraz `sync`: przesunięcie co drugiej kamery względem kamery głównej, dla każdego wyzwolenia (`offsets_us`: dla każdej kamery `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), w ramach JEDNEGO taktu (`clock_domain`: `ptp` lub `device` — PTP wyłączone, własny zegar czasu pracy każdej kamery, więc jej przesunięcie czasowe wynika z momentu uruchomienia, a synchronizację mierzy wyłącznie jitter). Wyzwalacz, którego znaczniki czasu łączą zegar urządzenia z PTP (PTP wyłączone lub wciąż w fazie synchronizacji), jest pomijany i zliczany (`groups_mixed_domain`), podobnie jak ten, w którym zegar PTP nadal odbiega o więcej niż `agree_within_us` od zegara głównego (`groups_clocks_disagree`); potwierdzają to wartości `notes`. `clock_offset_mean_us` / `clock_offset_jitter_us` to największe średnie i odchylenia dla poszczególnych kamer. `success` ma wartość fałszywą, podobnie jak `error`, gdy pominięto sygnał wyzwalający, gdy mniej niż dwa sygnały wyzwalające z każdej kamery są zsynchronizowane z jednym zegarem lub gdy zegary nigdy nie są zsynchronizowane (`sync.problems`); wiersze są zwracane niezależnie od sytuacji. Uruchamia serwer główny PTP, jeśli nie działa (`time_sync` określa `grandmaster_running` i `grandmaster_started_by_this_test`), włącza PTP w każdej kamerze (tak jak robi to `connect_array()`) i przed pierwszym sygnałem wyzwalającym czeka, aż zegar PTP każdej kamery zsynchronizuje się z zegarem nadrzędnym i ustabilizuje się: `PtpStatus` Slave na jednym zegarze nadrzędnym, z odchyleniem nie większym niż 100 us i stabilny przez 3 s, a każda kamera zsynchronizowana w ten sposób, bez przerwy, przez 120 s — około 2 minut od włączenia zasilania. Zegar, który właśnie się zsynchronizował, wykazuje odchylenie do ~200 us przez około 2 minuty po odczytaniu statusu „Slave”, a własny `PtpOffsetFromMaster` kamery tego nie pokazuje; 120 s doprowadza kamery do odchylenia ~10 us od ustabilizowanego zegara. Kamera, która pozostaje zsynchronizowana od czasu wcześniejszego testu, w którym pozostawiono włączony PTP (z punktu widzenia tego backendu), liczy od momentu tej synchronizacji, więc test przeprowadzany bezpośrednio po innym nie czeka ponownie; kamera włączona przez test lub taka, która została wykryta z włączonym PTP, a której ten backend nie widział w stanie zsynchronizowania (włączona gdzie indziej lub przed ponownym uruchomieniem backendu), czeka swoje 120 s, a w tablicy obu kamer, które właśnie się zsynchronizowały, nadal czekają na swoje. Kamera, która opuszcza tryb Slave, przekracza 100 us lub podąża za innym grandmasterem przez 3 odczyty z rzędu (1,5 s) w trakcie oczekiwania, rozpoczyna odliczanie od nowa; jeden lub dwa takie odczyty są rejestrowane (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), a odliczanie trwa dalej, a identyfikator grandmastera, którego nie da się odczytać, jest traktowany jako nieznany, a nie jako zmiana. Jednoczesne ponowne rozpoczęcie liczenia przez wszystkie kamery (w ciągu sekundy) oznacza zmianę czasu „grandmastera”, a nie kamer:logu backendowego widnieje `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` wymienia to w `grandmaster_steps`, a wyczerpanie się czasu oczekiwania powoduje błąd zaczynający się od `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limit nie jest w tym przypadku przedłużany. `ptp_settle_s` (0–600; `0` w przypadku braku oczekiwania na ustabilizowanie się) ustawia czas ustabilizowania się dla danego przebiegu; inna wartość jest odrzucana (kod HTTP 400). Czas oczekiwania jest ograniczony do czasu ustabilizowania plus 90 s (domyślnie 210 s, co najmniej 90 s), a wywołanie czeka na backend przez `timeout` s: domyślnie 240 plus czas ustabilizowania (360 s), podobnie jak w przypadku `chloros-mcp` i `test_array_sync`. `ptp_lock` posiada `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (jak długo każda kamera pozostawała zablokowana i stabilna), `found_settled` (kamery, które były już zablokowane przez czas ustabilizowania w momencie wykrycia) oraz, dla każdej kamery, `status`, `offsets_ns`, `slave_s` (jak długo jest zsynchronizowana; „Brak”: nie jest zsynchronizowana) oraz `remaining_s` (wartość „Brak”, gdy PTP żadnej kamery nie zostało włączone). Dziennik serwera zapisu zawiera co 5 s wiersz z informacją o stanie każdej kamery, czasie trwania synchronizacji oraz pozostałym czasie. Każda kamera włącza się z wyłączonym PTP; od tego momentu zegary synchronizują się w ciągu około 10 s. Następnie każda kamera jest przywracana do stanu, w jakim została znaleziona podczas testu — jej PTP, automatyczna ekspozycja i wzmocnienie pozostają wyłączone na czas trwania testu (`ExposureAuto` / `GainAuto` oraz ręczne `ExposureTime` / `Gain`), a pozostałe, o ile nie występuje `keep_settings=True` — czyli kamera pozostawiona z włączonym PTP (`PtpEnable=True` lub przebieg z `keep_settings=True`) pozostaje zsynchronizowana, a gdy ten backend stwierdzi, że jest zsynchronizowana przez czas ustabilizowania, późniejszy test nie czeka na nią. Jeśli zegary się nie ustabilizują, `success` ma wartość fałszywą przed jakimkolwiek wyzwoleniem, a `error` określa, co należy zrobić: kamera, która nigdy nie została zsynchronizowana (`not_slave`), oznacza, że serwer główny nie ma do niej dostępu (`time_sync_status()`); kamera, która ciągle się rozłączała, jest oznaczona czasem, przez jaki pozostawała zsynchronizowana (`remaining_s`); zegary wciąż się zsynchronizowują (`far`): sprawdź `time_sync_status()`, a aby zablokować je przed testem, włącz PTP i pozostaw włączony (`PtpEnable=True` na każdej kamerze lub `keep_settings=True`; test przywraca znalezione ustawienia, więc pozostaje włączony), a następnie uruchom go. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamery w linii synchronizacji i domyślny serwer główny: `master_serial`, `slave_serials`, sprawdzone przez `line`. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Cóż za bogaty wybór modeli tych kamer (nazwy produktów, `"LATT-M3C-L87-FOCN"`) mogłaby pojawić się w sieci tego komputera: `tier`, `achievable_fps_max`, limit przepustowości (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` oraz `fix_suggestions`. Domyślnie są to modele wykrytych kamer; żadna kamera nie jest otwarta. Informacje o posiadanych kamerach wraz z ustawieniami, których używałoby połączenie, można znaleźć w [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Oblicz profil dopasowania na podstawie klatek na żywo i zapisz go. `method`: `orb`, `akaze`, `phase`, `checkerboard` (wraz z `checkerboard="9x6"`, narożniki wewnętrzne) lub `manual` (wraz z `control_points={serial: [[x, y], ...]}`, również z kamery referencyjnej). `model`: `translation`, `rigid`, `affine` lub `homography`. Bardziej szczegółowe opcje to flagi CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` to `{serial: {option: value}}`. `rms_threshold_px` odmawia zapisania gorszego rozwiązania. Zwraca `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (dla każdej kamery `reprojection_error`, `num_matches`) oraz `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Jeden wyrównany obraz wielopasmowy w ramach profilu: `files` (wielopasmowy plik TIFF wraz z `bands` i rozmiarem, plus po jednym na pasmo z `per_band` oraz po jednym na kamerę z `per_camera`) oraz `vignette_corrected`. `bands` wybiera, a `order` porządkuje pasma według nazwy (lista lub ciąg znaków rozdzielonych przecinkami). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Obrazy wielopasmowe wyrównane ze strumieniem (te same opcje dopasowania co `capture_aligned_multiband()`), dla `count` lub do momentu zatrzymania. Zwraca `CliJob`; jego końcowe statystyki obejmują `frames_written` oraz wszelkie `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Plik profilu podsumowany jako `align-info` wyświetla następujące informacje: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (na każdą kamerę: `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (dla każdej kamery: `band_names`, `wavelengths_nm`, `enabled`); `profile` to cały plik. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Zmiana pliku profilu na miejscu: `order` to nazwy pasm (każda z nich przesuwa warstwę swojej kamery w górę, w tej kolejności), `enable` / `disable` to numery seryjne. Zwraca nowy plik `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Ustawienia indeksowe (`presets`: kolejno `name`, `channels`, `formula`) oraz kolorowe `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Indeks dla wyrównanego, wielopasmowego pliku TIFF: `preset` (`NDVI`, `NDRE`, ...) lub `formula` nad symbolami `channel_map` mapuje na pasma (`{"NIR": "NIR_850", "Red": 0}`, według nazwy lub indeksu). Symbol presetusymbolu `channel_map` pomija i wybiera jedno pasmo z pliku, którego nazwa zaczyna się od niego (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), nazwane zgodnie z samym plikiem (jego lista pasm lub `Chloros:Band{R,G,B}Name` z przechwycenia/produktu Chloros), więc NDVI’ na zdjęciu reflektancyjnym Chloros’ nie wymaga `channel_map`; dwa pasma odpowiadające jednemu symbolowi są odrzucane, a nie domyślane. `capture_level` określa zawartość pasm: `reflectance` lub `radiance` nadają wartościom charakter ilościowy; niekalibrowany stos (`align-apply`) dostarcza wartości jakościowe; `"unknown"` odczytuje to z pliku, w tym z pliku Chloros: `Chloros:ProcessingLevel`. Zwraca `stats` (kształt, zakres od 1. do 99. percentyla, zakres LUT, użyty `capture_level`) oraz `files` zapisany obok danych wejściowych (`<input stem>_<preset>.tif`, `..._LUT.png`) lub w `output`, ścieżkę: w przypadku rozszerzeń obrazów nazwy plików są tworzone na podstawie ich rdzenia (`output="D:/out/ndvi_FRGN.png"` zapisuje `ndvi_FRGN.tif` i `ndvi_FRGN_LUT.png`), a bez niej jest to prefiks (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); odczytaj ścieżki od `files`. Zawierają one dane rejestru aparatu źródłowego (marka / model / numer seryjny, GPS, obiektyw, czas wykonania zdjęcia w UTC, identyfikator kalibracji, ale nigdy kalibrację raw-DN), `Chloros:ProcessingLevel=index` oraz indeks jako swoje jedyne pasmo (`XMP-Camera:BandName`, zarówno plik TIFF, jak i PNG z tabelą LUT). Tylko pliki; bez danych z kamery. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Przechwyć, skalibruj (`reflectance` lub `radiance`; `raw` pomija ten etap), wyrównanie i obliczenie indeksu w jednym kroku; `save_multiband=True` zachowuje skalibrowany stos, który `compute_index()` może pobrać później. Zwraca `stats` (wraz z `serials`, którego pasma zostały wykorzystane) oraz `files`. Otwiera `serials` (pierwsza uruchamia wyzwalacz), w przeciwnym razie kamery profilu, których wpis `layer_order` jest włączony, najpierw kamera referencyjna (otwarta nawet przy wyłączonych pasmach); kamera z wyłączonymi pasmami nigdy nie jest kalibrowana. `channel_map` jest potrzebny tylko dla symbolu, na który stos nie może odpowiedzieć, jak w `compute_index()`: symbol presetu wybiera jedno włączone pasmo, którego nazwa zaczyna się od niego, sprawdzane przed otwarciem jakiejkolwiek kamery (odrzucane są odpowiedzi braku lub wielu kamer, z podaniem ich nazw); niestandardowy `formula` jest wymagany dla każdego symbolu. `index.tif` i LUT zawierają informacje o kamerze ze stosu, czasie przechwycenia (ekspozycja kamery referencyjnej, UTC) oraz obiektywie, a oba określają indeks jako swoje jedyne pasmo (`XMP-Camera:BandName`, np. `NDVI`: plik PNG w kolorze określa indeks, a nie kanały kolorów). `lattice index --live`. |

### `CliJob`

Uchwyt zwracany przez trzy strumienie. Zadanie działa w tle i zachowuje kontrolę nad kamerami do momentu jego zatrzymania.

| Element | Opis |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (klatki zapisane / zapisane, odrzucone, `fps`, `elapsed`, błędy zapisu, `stop_reason`; w przypadku zadania ciągłego również `frames_grabbed` oraz `frames_queued` — każda przechwycona klatka jest zapisywana, umieszczana w kolejce, odrzucana lub kończy się niepowodzeniem — oraz `drop_reason`, `writers` oraz `write_ms`, średni czas zapisu jednej klatki) oraz `output_dir`. `state` to `running`, `ended` (zatrzymał się samoczynnie i nie został jeszcze zatrzymany), `stopping` (zatrzymanie zapisuje swoją kolejkę: statystyki na żywo oraz `stopped_by`) albo `finished` (jego końcowe statystyki, `error` i `stopped_by` procesu zatrzymania). Rekord zatrzymanego zadania jest przechowywany przez 10 minut (maksymalnie ostatnie 32), dla każdego klienta: inny skrypt lub `chloros-mcp` odczytuje ten sam stan i statystyki końcowe. Zadanie `chloros-cli` rozpoczęło się (`lattice continuous`, `multi-stream`, `align-stream`) jest zatrzymywane przez backend, gdy interfejs CLI przestaje się zgłaszać przez 30 s — zabite, jego maszyna w trybie uśpienia — przy `stop_reason`, `client_gone` i `stopped_by` po stronie backendu; odczytanie jego statystyk z tego miejsca nie utrzymuje go przy życiu. Zadanie uruchomione z poziomu SDK działa do momentu jego zatrzymania. Każde uruchomienie wysyła swój `timeout` jako termin dla backendu (`client_deadline_s`): kamera, która otwiera się po nim (wciąż się uruchamia), jest przywracana i zamykana, a żadne zadanie nie jest uruchamiane, a wywołanie generuje `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kamera zajmowana przez zadanie jest odrzucana przy każdym kolejnym otwarciu (`connect_camera()`, kolejne uruchomienie, przechwytywanie przejściowe) z kodem HTTP 409 `CAMERA_HELD`, podając nazwę zadania i osobę, która je uruchomiła. |
| `stop(timeout=30.0)` | Zatrzymaj je i zamknij jego kamery: `final_stats`, `output_dir` oraz `error`, gdy zatrzymanie nie przebiegło poprawnie. Drugie wywołanie zwraca tę samą odpowiedź, podobnie jak zatrzymanie z innego klienta: wysłane, gdy pierwszy nadal zapisuje swoją kolejkę, czeka na jego zakończenie (do 25 s; po upływie tego czasu generuje wyjątek, a ostateczne statystyki pojawiają się w `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Należy poczekać, aż proces zakończy się samoczynnie (po osiągnięciu `count`, zapełnieniu dysku lub wystąpieniu błędu), a następnie `stop()` go, aby zebrać ostateczne wyniki. `timeout` generuje wyjątek `TimeoutError`, gdy proces nadal działa; zadanie bez `count` kończy się dopiero po zatrzymaniu. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Jego identyfikator i rodzaj (`continuous`, `multi-stream`, `align-stream`), miejsce zapisu, odpowiedź trasy początkowej oraz informację, czy `stop()` został uruchomiony. |

Użyj go jako menedżera kontekstu, aby zatrzymać go przy wyjściu z zakresu.

---

## Folder projektu i przepisy

**Folder projektu** to folder, w którym tworzone są nowe projekty, współdzielony przez aplikację Chloros, interfejs CLI oraz SDK (moduł `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Jeśli skonfigurowany folder już nie istnieje, stosowana jest wartość domyślna (`Documents/MAPIR/Chloros Projects`), tak samo jak w przypadku aplikacji. Ścieżka względna podana dla `set_project_folder()` jest pobierana z katalogu roboczego, gdy backend znajduje się na tym komputerze.

**Receptura** to lista akcji w formacie YAML / JSON, którą `chloros-cli project run PATH RECIPE` uruchamia na zapisanym projekcie. `run_project_recipe()` uruchamia ją za pomocą CLIwłasnym modułem uruchamiającym (zainstalowanym wraz z SDK), więc jedna receptura zachowuje się tak samo niezależnie od tego, czy jest uruchamiana przez:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** to plik `.yaml` / `.yml` / `.json`, jak i samą recepturą: słownikiem zawierającym `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), a opcjonalnie `overrides` i `stop_on_error` (domyślnie true: działanie zatrzymuje się przy pierwszej akcji, która zakończyła się niepowodzeniem). Akcje są zgodne z opisem w dokumentacji CLI [Recipe DSL](cli-reference.md#recipe-dsl); względna ścieżka `output` jest rozpoznawana względem katalogu roboczego skryptu.
- **Zwraca** `ok` (wszystkie akcje zakończyły się powodzeniem), `actions` (ile), `log` (wynik działania programu uruchamiającego; `echo=True` również wyświetla go podczas działania) oraz `validation`. Brakujący lub niemożliwy do przeanalizowania plik receptury powoduje wygenerowanie błędu `ChlorosConnectError`.
- **Sprawdzane przed uruchomieniem.**`validation` to `{"ok", "errors", "warnings", "steps"}`: każda akcja musi mieć jako klucz dokładnie jeden znany czasownik (`{"capture": {"name": ...}}`, a nie `{"action": "capture"}` lub `{"type": "capture"}`), określający nazwę kamery, tablicy lub czujnika posiadanego przez projekt, odpowiedniego typu. Przepis zawierający błąd**nie** wykonuje żadnej akcji (`ok` false, błędy w `log`), niezależnie od tego, co mówi `stop_on_error`. Nieużywane klucze, oraz każda tablica lub czujnik światła sterowany przez recepturę, który nie jest obecnie podłączony, powodują błąd `warnings`. Receptura nie podłącza urządzeń: należy podłączyć je za pomocą pierwszego narzędzia projektu (`ChlorosProject.connect_all()`, `chloros-cli project connect`, lub narzędzie MCP `connect_project_devices`). `dry_run=True` niczego nie uruchamia i zwraca `{"recipe": ..., "validation": ...}`.
- W procesie uruchamia się tylko jedna receptura na raz. Receptura YAML wymaga biblioteki PyYAML.

---

## Automatyzacja projektu — `ChlorosProject`

Zapisany projekt Chloros to folder zawierający pliki `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` ładuje manifest (`relink`: folder, w którym obecnie znajdują się brakujące pliki źródłowe przeniesionego projektu, używany podczas ładowania go przez backend — `process()` — zobacz **Przeniesiony projekt** w sekcji `ChlorosLocal`), a `connect_all` uruchamia wszystkie zapisane kamery, matryce i czujniki światła wraz z ich zapisanymi ustawieniami — w takim samym stanie sprzętowym, jaki uzyskałby interfejs graficzny. Czujniki światła łączą się poprzez [puli czujników DAQ](#daq-sensor-sessions), dlatego wymagają uruchomionego backendu (zobacz [`SensorHandle`](#sensorhandle)).

### Minimalny przykład

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Lub jako menedżer kontekstu:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metody `ChlorosProject`

| Metoda | Opis |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Wykrywa i łączy się z każdą zapisaną kamerą, tablicą i czujnikiem światła. Zwraca raport połączeń dla poszczególnych klas (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, z których każda zawiera `connected`, `missing` i `errors`; `arrays` obsługuje również `alignment`). Jeśli backend nasłuchuje na `127.0.0.1:5000`, zapisane kamery i tablice są wykrywane i podłączane za jego pośrednictwem. W przypadku braku takiego backendu proces ten samodzielnie je wykrywa i steruje nimi poprzez `lattice_sdk`, co wymaga środowiska uruchomieniowego SDK Arena oraz OpenCV (`cv2`) — pakiet SDK nie instaluje żadnego z nich; gdy instalacja nie jest możliwa, każda zapisana kamera i macierz jest wymieniona pod `errors` wraz z przyczyną, a `missing` pozostaje pusty. Backend, który działa, ale odrzuca SDK (wylogowany: HTTP 401; plan poniżej Copper: 403), nie oznacza „braku backendu”: każda zapisana kamera, tablica i czujnik światła są wymienione w `errors` wraz z przyczyną podaną przez backend, i w tym procesie nic nie jest otwarte. Każde wywołanie ponownie sprawdza serwer zaplecza, więc `connect_all()` po zalogowaniu się lub po uruchomieniu Chloros łączy się z tym samym obiektem projektu. Zapisana matryca, która już istnieje w backendzie z tymi samymi kamerami (z interfejsu graficznego lub z `chloros-cli project connect`), jest dołączana w obecnej postaci, bez ponownego stosowania jej zapisanych ustawień, a `disconnect_all()` pozostawia go uruchomionym. Czujniki światła zawsze przechodzą przez backend (zobacz [`SensorHandle`](#sensorhandle)). Nigdy nie uruchamia backendu. |
| `disconnect_all()` | Zlikwiduj wszystko. Najpierw zwolnione zostają wszystkie urządzenia; następnie, jeśli backend nie potwierdził zwolnienia kamery, tablicy lub czujnika światła (likwidacja przekroczyła limit czasu lub została odrzucona), `ChlorosConnectError` nadaje imionę każdemu z nich. Urządzenie, które już zostało zwolnione, lub dla którego nie działa backend, pozostaje ciche. Matryca lub czujnik światła, które były otwarte przez innego klienta, pozostają uruchomione. Zakończenie działania projektu `with` nigdy nie jest zgłaszane. |
| `capture_all(output_dir=".")` | Jedna klatka z każdej podłączonej kamery + matrycy, oraz jedno widmo (`SensorHandle.read()`) z każdego podłączonego czujnika światła, pobierane po kolei (kamery, następnie macierze, a potem czujniki), a nie w jednym momencie. |
| `stream(camera, overlays=False, fps=10.0)` | Generator generujący klatki BGR `numpy` z określonej kamery (lub matrycy). `overlays=False` odczytuje klatki o identyfikatorze`grab()` na poziomie `debayered`: przez backend, gdy projekt jest z nim połączony, w przeciwnym razie poprzez bezpośrednią pętlę pobierania `lattice_sdk` (tablice generują słowniki `{serial: frame}`). `overlays=True` przekierowuje przez `ChlorosLocal.camera_stream()` → strumień MJPEG `/api/camera/<serial>/stream-annotated` backendu, przy czym zapisany blok kamery `ui.overlay` jest przekazywany jako parametry zapytania. Wymaga trybu backendowego, biblioteki OpenCV (`cv2`) oraz **kamery autonomicznej**: kamera w trybie bezpośrednim generuje błąd `RuntimeError` (backend nie może przejąć kamery należącej do tego procesu), a tablica generuje błąd `NotImplementedError` (nakłada kompozycję dla każdej kamery — przesyła strumień elementu według nazwy). Odpowiednik jednorazowy: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Uruchom wyrównywanie dla każdej aktualnie podłączonej tablicy. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Uruchom potok kalibracji / indeksowania na obrazach projektu (zawiera `ChlorosLocal.process`; te cztery są **jedynymi** akceptowanymi argumentami kluczowymi — `indices=` itp. wywołuje wyjątek `TypeError`; ustawia indeksy za pomocą `ChlorosLocal.configure()`). Leniwie tworzy obiekt `ChlorosLocal()`, który automatycznie uruchamia backend. |

Atrybuty:
- `proj.cameras` — `Dict[str, CameraHandle]` indeksowane według nazwy ORAZ numeru seryjnego.
- `proj.arrays` — `Dict[str, ArrayHandle]` indeksowany według nazwy ORAZ identyfikatora tablicy (array_id).
- `proj.sensors` — `Dict[str, SensorHandle]` indeksowany według nazwy ORAZ identyfikatora slotu (slot_id).
- `proj.config` — słownik `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Poziomy przetwarzania.** `capture()`, `grab()` i `frame_stream()` przyjmują ten sam token `processing`
, a łańcuch ma charakter kumulacyjny — każdy poziom uruchamia wszystkie elementy znajdujące się powyżej niego:

| Poziom | Wynik | Uwagi |
| --- | --- | --- |
| `raw` | 1-kanałowy Bayer, natywny dla czujnika | Brak demosaic. Na tym poziomie nakładki nie są dostępne. |
| `debayered` | 3-kanałowy kolor (**domyślny** dla `CameraHandle` oraz wszystkich `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` i `stream()` domyślnie przyjmują wartość `raw`): BGR w pamięci (`grab()` / `frame_stream()`), zapisywane jako RGB przez `capture()` | W trybie backendowym, jako przechwycenie: odchylenie ciemności (DSNU) z pakietu kalibracyjnego od danych surowych, a następnie demosaikowanie z uwzględnieniem krawędzi w OpenCV — bez tablicy LUT liniowości, wyrównania pola ani demiksowania. Jedyny poziom, który działa bez trybu backendowego, to zwykłe demosaikowanie biliniowe biblioteki kamery (bez ciemnego przesunięcia). Zapisany plik TIFF jest w formacie fotometrycznym RGB, więc należy odwrócić kanały klatki w pamięci (`frame[..., ::-1]`), aby porównać ją z plikiem. |
| `radiance` | float32, W/m²/sr/nm, **jedna płaszczyzna na pasmo** | Pełny łańcuch radiometryczny: demosaikowanie + rozdzielanie 3×3 (multispektralne) + DSNU + pole płaskie + skala NIST, z wyeliminowaną ekspozycją i wzmocnieniem, dzięki czemu wartości są bezwzględne. |
| `reflectance` | uint16, 32768 = 1,0 (lub uint8, 255 = 1,0 — patrz `bit_depth`), **jedna płaszczyzna na pasmo** | Promieniowanie podzielone przez natężenie promieniowania padającego (ρ = π·L/E). Wymaga odczytu z DLS/DAQ — patrz uwaga poniżej. |
| `display` | 8-bitowy, zbliżony do sRGB | Renderowanie odpowiadające interfejsowi graficznemu: CCM + balans bieli + gamma poprzez aktywny profil kolorów kamery. |

Tylko w trybie bezpośrednim (bez zaplecza) dostępny jest `debayered`: `grab()` na innym poziomie powoduje, że
`NotImplementedError`, `capture()` ignoruje `processing`, a `frame_stream()` wywołuje
`RuntimeError` na każdym poziomie. `reflectance` wymaga użytecznego odczytu promieniowania padającego — punkt końcowy ramki automatycznie wciąga
zgromadzone dane DAQ do gniazda DLS kamery, ale bez powiązanego DAQ łańcuch odrzuca
wyjście odbicia i uczciwie zaznacza obniżenie klasy w zwracanych metadanych, zamiast po cichu
zwracając produkt o niższej jakości.

> **Skala DN odbicia — nie należy jej zakodować na stałe.** Reflektancja LATTICE wykorzystuje `32768` = ρ 1,0 i oznacza
> XMP `Chloros:PixelScale=32768`; reflektancja Survey3 wykorzystuje `65535` = ρ 1,0 i nie zawiera żadnych
> tagów `Chloros:*`. Odczytaj tag i podziel przez niego. Jest on zdefiniowany w domenie uint16, więc pozostaje
> `32768` dla każdego formatu, który zmienia skalę (16-bitowy TIFF, 8-bitowy TIFF, 8-bitowy PNG/JPG, 32-bitowy procent)
> — najpierw znormalizuj zapisany typ danych z powrotem do uint16 (×257 z 8-bitowego, ×65535 z typu float). Jedynym
> wyjątkiem jest plik z kompilacji sprzed wersji 1.3.0: 8-bitowy plik TIFF był wówczas *przycinany*, a nie przeskalowywany, więc
> nie opisuje go żadna skala i nie zawiera on tuplu `PixelScale` ani MicaSense. Brakujący tag w
> pliku odbicia LATTICE należy traktować jako „brak prawidłowej skali”, a nie jako wartość domyślną.

**Tagi kalibracyjne: tylko w przypadku `raw`.** Plik `raw` z danego ujęcia zawiera pakiet kalibracyjny kamery: dane użytkowe `CalibrationJSON` oraz tagi zgodne ze standardem MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), które opisują surowe wartości czujnika. Pliki każdego kolejnego poziomu (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) oraz każda klatka z nagrania ciągłego / strumienia z wielu kamer zawiera identyfikator kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) oraz własny zapis klatki, `XMP-mapir:FrameJSON` (ekspozycja, wzmocnienie, temperatura, format pikseli, czas przechwycenia, dopasowanie DAQ), ponieważ każdy produkt `process()` posiada swój identyfikator. Program przetwarzający wyszukuje kalibrację takiego pliku na podstawie jego numeru seryjnego oraz `CalBundleSha256` w pamięci podręcznej kalibracji programu Chloros, pobierając pakiet tylko raz, jeśli na danym komputerze go brakuje.

#### Pliki utworzone z danych z kilku kamer

Plik utworzony z pasm z więcej niż jednej kamery — stos wielopasmowy (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), indeks na żywo utworzony z obrazów z kilku kamer (`index.tif`, `index_LUT.png`), obraz `composite/` z przechwytywania macierzowego — wymienia każdą kamerę, z której pochodzą dane:

- **Wkamery, w kolejności pasm** (kompozycja: warstwa tła, następnie pierwszy plan; warstwa indeksowa, kamery, z których pochodzą pasma): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (pakiet kalibracyjny, przez który przeszły pasma każdej kamery, w przeciwnym razie — stos `capture_aligned_multiband()` / `start_aligned_stream()`, którego pasma nie przechodzą przez żaden — pakiet kamery, który Chloros zapisał w pamięci podręcznej, ten, którego nazwę noszą jej pliki przechwyconych obrazów), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listy równoległe, przy czym pozycja *n* każdej z nich opisuje tę samą kamerę; `-`, gdzie kamera nie ma żadnej wartości (brak pakietu kalibracyjnego, nieznany obiektyw). Każda kamera to **jedna** pozycja: jeśli jej pakiet zawiera kilka wartości dla danego pola — certyfikaty lamp halogenowych pakietu — są one połączone za pomocą `;` (`F-1479-1;F-1480-1;F-1481-1`), nigdy przecinkiem, więc lista zawiera jedną pozycję na aparat, niezależnie od tego, jak jest wydrukowana.
- **EXIF `SerialNumber`** zawiera te same numery seryjne, połączone przecinkami (`214000535,214000551`).
- Nie zawiera **żadnego** identyfikatora pojedynczej kamery (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): te identyfikatory określają jedną kamerę i jej zestaw, a przetwarzanie pozwala na ustalenie kalibracji pliku na ich podstawie. Indeks obliczony na podstawie pasm z kilku kamer (`index.tif`, `index_LUT.png`) zawiera w metadanych EXIF wartości `Model` i `LATTICE` (konfiguracja), a nie model `LATT-…` jednej kamery.

Plik zawierający pasma z **jednej** kamery — plik TIFF dla każdego pasma, klatkę `capture_aligned_multiband(per_camera=True)`, stos lub indeks na żywo z pojedynczej kamery — zawiera własne tagi identyfikacyjne tej kamery, tak jak każdy inny plik z ujęciami. Stos `capture_aligned_multiband()`, pliki poszczególnych pasm i poszczególnych kamer, a także każdy stos `start_aligned_stream()` i jego pliki poszczególnych pasm zawierają czas wykonania ujęcia dla całej grupy (ekspozycja kamery referencyjnej, UTC), a klatka `per_camera` z kamery monochromatycznej stanowi jedną płaszczyznę, podobnie jak stos `bit_depth`.**Orientacja kamery (odwrócenie / obrót).** Klatki z plików `grab()` / `frame_stream()` są zawsze w
natywnej orientacji kamery na każdym poziomie. Zapisane ujęcia są zgodne z ustawieniami orientacji kamery
(opcje GUI: Odbicie w poziomie / Odbicie w pionie / Obrót):

- **Pliki `raw` i `debayered` zawierają dokładnie to, co odczytał czujnik**, niezależnie od ustawień — mozaika Bayera
  obrócona lub odwrócona stanowiłaby inny wzór filtrów kolorowych, a mapy kalibracyjne są
  na własnej siatce czujnika.
- **Pliki `display`, `radiance`, `reflectance` oraz pliki indeksowe są zorientowane** — obrócone o jeden raz, po zakończeniu wszystkich
  obliczeń kalibracyjnych. Ich maski ograniczające znajdują się na tej samej siatce.
- **Każdy plik przechwytywania rejestruje ustawienie** w XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, zgodnie z ruchem wskazówek zegara, po
  odwrócenia) oraz `ChlorosOrientationApplied` — `false` na danych surowych i po usunięciu efektu Bayera, a `true` na zorientowanych
  produktach. `ChlorosProcessing` zawiera listę operacji zastosowanych do pikseli tego pliku (`none` w przypadku
  formatu natywnego). Obejmuje to każdą klatkę nagrania ciągłego (własne ustawienie kamery,
  niezastosowane) oraz strumienia z wielu kamer (tożsamość: klatki wchodzące w jego skład są w formacie natywnym, podobnie jak w przypadku
  przechwytywania macierzowego).
- **Przetwarzanie projektu powoduje zastosowanie zarejestrowanej orientacji** do produktów tworzonych na podstawie danych surowych
  (najpierw kalibracja we współrzędnych czujnika, następnie korekcja wyrównania matrycy, a na końcu orientacja),
  dzięki czemu odpowiadają one produktom z przechwytywania na żywo — w tym niekalibrowanej odpowiedzi czujnika oraz
  produktom rezerwowym z korekcją winietowania; produkt po usunięciu efektu bayera pozostaje natywny.
- **Przeglądarka obrazów Chloros pokazuje plik surowy lub po usunięciu bayera obrócony zgodnie z danymi rejestracji** — dotyczy to wyłącznie wyświetlania,
  plik nie ulega zmianie; plik bez danych rejestracji jest wyświetlany w postaci, w jakiej został zapisany.
- **Ujęcia bez danych rejestracji są przetwarzane jako natywne.** Przed wersją 1.3.0 przechwytywanie z jednej kamery
  samodzielnie obracało pliki surowe i po usunięciu matrycy; Chloros nie potrafi odróżnić takiego pliku surowego od natywnego i
  nie podejmuje prób odgadnięcia.
- **Przechwyty z macierzy domyślnie zachowują każdy plik składowy w formacie natywnym** i rejestrują tożsamość;
  `ArraySession.capture(orient=True)` przetwarza ich wyniki (zobacz opcje obsługi serii zdjęć).

#### Dane wyjściowe dla poszczególnych pasm i macierze mieszane

Na poziomach `radiance` i `reflectance` kamera zwraca **jedną płaszczyznę na pasmo tej
kamery** — trzy dla M3C (jej trzy niezmieszane pasma, kolejność kanałów BGR), jedną dla M3M (jej
pojedynczy filtr wąskopasmowy). Powielone płaszczyzny kamery monochromatycznej są zwinięte, więc **mieszana
M3C/M3M daje różne kształty macierzy dla każdego numeru seryjnego**. Należy odczytać klucz `bands` z metadanych klatki,
aby dowiedzieć się, czym jest każda płaszczyzna; nie należy wyciągać wniosków na podstawie liczby kanałów:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Zawsze sprawdź `hdr["processing_level"] == "reflectance"` przed użyciem tych wartości. Obniżenie rangi
(brak odczytu DAQ, brak pakietu kalibracyjnego, nieznany układ pasm) jest zgłaszane w `processing_level` i
`radiometric_refused`, a nie podwyższone — otrzymana matryca nadal jest prawidłową ramką, tylko
nie jest to produkt, o który prosiłeś. Zapisane przechwycenie, które zostało zdegradowane, zawiera w swoim pliku XMP ten sam kod
co `Chloros:RadiometricRefused` (np. `no-band-layout`, `no-dls-snapshot`).

#### Jeden sygnał wyzwalający na odczyt — `group_id`

**Elementy macierzy są dostarczane z zsynchronizowanej grupy klatek macierzy, a nie z własnego
bufora przechwytywania każdej kamery.** Kamery w macierzy zsynchronizowanej sprzętowo eksponują obraz razem, ale N odczytów z każdej kamery
nie są *próbkowane* razem — odczyt najnowszej zbuforowanej klatki z każdej kamery wykracza poza granice wyzwalacza
, a pojedyncza utracona klatka desynchronizuje zestaw, co pozostaje niezauważalne.

Dostarczanie danych z grupy jest konieczne, ale samo w sobie niewystarczające: każde żądanie nadal
przechwytuje *najnowszą* grupę w swoim własnym momencie, a pętla przechwytywania może publikować klatki w międzyczasie.
W pomiarach przeprowadzonych na działającej macierzy z 4 kamerami **12 z 20 równoległych odczytów obejmowało dwie grupy** — zawsze
dokładnie sąsiadujące ze sobą. Odczyt jest więc **przypisany**:

1. Każda klatka zawiera `group_id`, monotoniczny licznik publikacji układu.
2. `grab()` odczytuje kamerę główną (lub, jeśli nie można jej obsłużyć, następną kamerę, która jest dostępna),
   a następnie przypisuje pozostałe kamery do identyfikatora grupy, na której się zatrzymał (`?group_id=N` w punkcie końcowym HTTP
  ).
3. Backend przechowuje ostatnie grupy przez **co najmniej pół sekundy**, więc element równorzędny, który został już
   wyprzedzony przez późniejsze publikacje, nadal obsługuje ten sygnał wyzwalający — wystarczająco długo, aby umożliwić
   eksportu współczynnika odbicia, nawet w przypadku pełnej matrycy składającej się z 9 kamer. Przypisanie, którego backend już
   nie utrzymuje, jest odrzucane za pomocą kodów `409` i `group_aged_out: true`, a nie odpowiadane w milczeniu
   z innym wyzwalaczem — `grab()` ponownie odczytuje zestaw.

Ponowny pomiar z przypisaniem: **20 z 20 odczytów zwróciło jeden sygnał wyzwalający.** `grab()` robi to za Ciebie;
potrzebujesz `group_id=` tylko w przypadku bezpośredniego sterowania punktem końcowym HTTP.

- Kamera, której nie można obsłużyć w przypisanej grupie — brakuje jej w tej grupie lub odrzuca żądanie
  (nie posiada `band`, nie może wygenerować danego poziomu) — jest **pomijana, a nie uzupełniana**
  z innego wyzwalacza, a pozostałe kamery są nadal zwracane.
- Słownik zwracany przez `grab()` jest podklasą `GrabResult` (zwykła podklasa `dict`;
  `from chloros_sdk import GrabResult`): `.errors` przyporządkowuje każdy
  pominięty numer seryjny do wyjątku, który spowodował jego odczyt, a `.group_id` to grupa wspólna. Pusty
  `.errors` oznacza, że każda kamera weszła w skład tej grupy.
- Kamera autonomiczna nie należy do żadnej grupy, zgłasza `group_id: None` i **odrzuca** pin `group_id=`
  zamiast go ignorować.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Ten sam licznik jest publikowany w **każdym strumieniu MJPEG dla każdej kamery** należącej do macierzy — w
podglądzie (`/api/camera/stream/<serial>`), podglądzie z adnotacjami (`/stream-annotated`) oraz
strumień odbicia (`/stream-reflectance`) — jako `X-Chloros-Group-Id` w każdej części, oprócz
`X-Chloros-Frame-Id` i `X-Chloros-Timestamp-Ns`. Strumienie mają charakter ciągły, a nie
punktowy w czasie, więc są dopasowywane poprzez zbieranie części i grupowanie na podstawie tego nagłówka — nie ma
żadnego pinu do ustawienia. Kamera autonomiczna nie ma grupy. Zobacz
[Strumienie MJPEG dla poszczególnych kamer](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nigdy nie milknie.** Tyk, który kończy się niepowodzeniem z powodu, który może ustąpić samoczynnie,
jest pomijany — strumień z kamery nie zwraca dla niego żadnych danych, a strumień tablicowy zwraca kamery, które
odpowiedziały (być może żadna). Przyczyny te to: brak klatki; wątek pobierania obrazu z kamery został wstrzymany na
sekundę lub dwie (`404 not streaming`, na co backend odpowiada za każdym razem, gdy inna kamera łączy się
lub rozłącza, lub gdy zmiana ustawień powoduje zrównoważenie puli); ponowne uruchomienie zaplecza lub `5xx`; wyrównanie
wciąż się ustabilizuje (`409 … retry shortly`); oraz odrzucenie odbicia, ponieważ moduł DAQ nie ma jeszcze odczytu
(`no-dls-snapshot`, nieaktualny lub pusty odczyt). Odmowa w następnym takcie tylko by się powtórzyła —
nieznany `band`, `bit_depth=8` dotyczący promieniowania, pakiet kalibracyjny bez skali promieniowania, odmowa odzwierciedlenia
przy **całkowitym braku modułu DAQ** (żadnego powiązanego z kamerą lub jej matrycą, żadnego podłączonego do
backendu: `dls_source` odrzucenia to `"none"`), każdy inny `4xx` —
powoduje `chloros_sdk.FrameFetchError` (`RuntimeError`, zawierający `.status` i `.body`)
natychmiast. Podobnie dzieje się w przypadku **60 sekund** pominiętych taktów bez żadnej klatki pomiędzy nimi, niezależnie od tego, czym jest `fps` — wystarczająco
długo, by przetrwać ustabilizowanie się wyrównania i ponowne uruchomienie zaplecza, a wystarczająco krótko, by kamera, która
naprawdę przestała działać, zakończy strumień. Ten błąd rezygnacji to również `FrameFetchError`, zarówno w przypadku pojedynczej kamery, jak i
tablicy, z wyjątkiem ostatniego odczytu na `.errors` (a w przypadku kamery jako `__cause__`). W przypadku
macierzy „odrzucone” oznacza, że każda kamera odmówiła; wyjątek `.errors` zawiera
przyczynę dla każdej kamery.

**Tablica `frame_stream()` generuje każdy sygnał wyzwalający tylko raz** (podobnie jak `ArrayHandle` i `ArraySession`).
Tik, którego najnowszy zestaw jest wyzwalaczem, który już zwrócił, czeka na następny wyzwalacz zamiast
zwracać ten sam zestaw ponownie, więc strumień działa z prędkością mniejszą od `fps` lub własnej częstotliwości ramki tablicy
, a kolejne zestawy nigdy nie dzielą tego samego `group_id`. Tablica, która przestaje publikować, generuje
`FrameFetchError` („brak nowego wyzwalacza przez 60 s”) zamiast powtarzać swój ostatni zestaw. Bez tego
strumień, który zażądał więcej niż opublikowała tablica — lub uruchomił się zaraz po połączeniu — generował
powtórzenia: w jednym z pomiarów 20 zestawów zawierało 11 różnych identyfikatorów grup.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Eksportowanie pojedynczego pasma — `band`

`grab()` / `frame_stream()` i `band="<name>"` służą do eksportowania tylko jednego pasma z kamery jako
jednokanałowej tablicy `(H, W, 1)` — o takim samym kształcie, jaki generuje kamera monochromatyczna, więc dane wyjściowe z jednego pasma są
przetwarzane identycznie, niezależnie od tego, czy pochodzą z kamery M3M, czy z podziału kamery M3C. Pasmo czerwone kamery FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Dopuszczalne zapisy to pełna nazwa układu (`Red_660`, `NIR_850`, `Green_550`) lub jego skrócona
nazwa (`red`, `nir`, `green`), bez rozróżniania wielkości liter. Nieznana nazwa jest odrzucana wraz z listą dostępnych
nazw.

> **Wybór odbywa się według nazwy, a nie numeru kanału — celowo.** „Kanał 1” to `Green_550` w
> kolejności kanałów BGR matrycy, ale *czerwony* w kodzie filtra(`RGN`), a te dwie
> wartości są ze sobą niezgodne. Wybranie niewłaściwej wartości daje całkowicie wiarygodną ramkę z niewłaściwym pasmem, której
> żadne dalsze elementy łańcucha nie są w stanie wykryć. Z tego powodu nie ma indeksu numerycznego.

#### Wyrównywanie strumieni — `align`

`align=True` przekształca klatkę do wspólnej klatki odniesienia macierzy. Każda kamera jest przekształcana
przy użyciu **tej samej** rozdzielczości profilu i obszaru ROI, więc strumienie poszczególnych kamer, otwarte niezależnie,
są **wzajemnie zarejestrowane** — nie trzeba ich żądać razem:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Wymagania i zachowanie:

- Kamera musi znajdować się w **zsynchronizowanej sprzętowo** macierzy (kabel synchronizacyjny M8), która posiada profil wyrównania
  — współrejestracja zakłada, że elementy macierzy są eksponowane jednocześnie, czego nie zapewnia macierz działająca w trybie
  macierz działająca w trybie swobodnym. W przeciwnym razie zostanie to odrzucone z kodem `409`.
- **Odmowa, nigdy nie dopuszcza się cichego braku wyrównania.** Ścieżka przechwytywania może zapisać dane niewyrównane z ostrzeżeniem,
  ponieważ plik zachowuje swoje tagi pochodzenia i może zostać ponownie zarejestrowany później; strumień na żywo nie ma
  takiej drugiej szansy, więc brak profilu jest błędem, a nie rozwiązaniem awaryjnym.
- **Żądanie wyrównania uruchamia ten proces.** Żądanie `align` dla macierzy bez profilu uruchamia
  kalibrację i zwraca `409` z informacją o tym, na co czeka (np. *„oczekiwanie na ustabilizowanie się ekspozycji
  na poziomie 214000551 (1/3)”*) — ponów próbę, a operacja zakończy się powodzeniem, zazwyczaj w ciągu kilkudziesięciu sekund.
  Ma to znaczenie w trybie bezinterfejsu graficznego: automatyczne wyrównywanie w momencie połączenia kończy się po 30 sekundach, a każda próba
  w tym przedziale jest pomijana, gdy inteligentna ekspozycja (smart-AE) wciąż się ustabilizuje, więc tablica, której ekspozycja
  zbiega się powoli, nie miała wcześniej profilu dla całej sesji i nie było możliwości uzyskania
  go bez otwarcia widoku połączonego na pulpicie. `alignment_status` zgłasza również
  `ae_align_holdouts`, podając nazwy kamer wciąż się stabilizujących oraz ich postęp.
- Należy nadać każdemu strumieniowi jednej matrycy **ten sam kod `max_h`**, w przeciwnym razie ich kadry znajdą się na różnych siatkach pikselowych
 .
- Wyrównanie jest stosowane *przed* kodowaniem głębi bitowej, więc interpolacja zniekształceń odbywa się z pełną precyzją
  wartości.

#### `bit_depth` — zamiana precyzji na przepustowość

`grab()` i `frame_stream()` akceptują `bit_depth=16` (domyślnie) lub `bit_depth=8`:

| `bit_depth` | Kodowanie odbicia | Koszt |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, zapas do ρ 2,0 | Pełna precyzja. |
| `8` | uint8 obejmujący `0..refl_max` (domyślnie ρ 1,0) | Połowa liczby bajtów; kwantyzacja ~0,004 ρ przy domyślnym pułapie. Wartości powyżej pułapu ulegają obcięciu — patrz poniżej. |

##### 8-bitowy pułap — `refl_max`

`refl_max` określa wartość ρ, na którą rozciąga się zakres 8-bitowy, więc w przypadku `pixel_scale = 255 / refl_max` wartość ρ pozostaje
`pixel / pixel_scale`. `grab()` i `frame_stream()` traktują to jako słowo kluczowe zarówno w przypadku kamery, jak i
uchwytów tablic (`?refl_max=` w punkcie końcowym HTTP). Zwiększ tę wartość, aby uzyskać rezerwę dla lustrzanych
refleksów, przy proporcjonalnie grubszych krokach:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | krok |
| --- | --- | --- |
| `1.0` (domyślnie) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatyczna ekspozycja nie jest w stanie naprawić przesterowania 8-bitowego, a próba jej zastosowania w tym celu pogarsza jakość danych.**
> ρ = π·L/E po wyeliminowaniu ekspozycji i wzmocnienia, więc ρ jest właściwością sceny, a nie
> ustawień aparatu — zmierzono na matrycy na żywo, że zmniejszenie ekspozycji o połowę spowodowało przesunięcie średniej wartości ρ o 1,3%. Zwiększenie
> ekspozycji w celu „dopasowania” zakresu powoduje natomiast nasycenie pikseli **matrycy**, co
> prowadzi do zaniżenia wartości L i **zaniżenia** wartości ρ bez zgłoszenia błędu (wynik pomiaru: maks. ρ 1,203 → 0,985 przy
> 2-krotnej ekspozycji, średnia −23%). Wybierz zakres za pomocą `refl_max`; zapobieganie
> nasyceniu matrycy jest już zadaniem automatycznej ekspozycji (AE), co chroni *wiarygodność* wartości ρ, a nie jej zakres.

**Przeczytaj `reflectance_clipped_fraction`, a nie flagę.** `reflectance_clipped_at_1_0` wyzwala się na
pojedynczym pikselu — w rzeczywistej scenie zgłaszał `True`, podczas gdy ułamek wynosił `0.000001`, czyli jeden piksel
na milion. To właśnie ułamek wskazuje, czy należy podnieść górną granicę.

**`bit_depth=8` dotyczy wyłącznie współczynnika odbicia.** Żądanie dotyczące `radiance` jest odrzucane, a nie
realizowane: odbicie ma stały zakres 0..1, który zachowuje się przy kodowaniu 8-bitowym, podczas gdy promieniowanie nie ma
stałego zakresu, więc dopasowanie go do 8 bitów wymaga rozciągnięcia percentylowego dla każdej klatki, którego odwzorowanie
zmienia się w każdej klatce — jest to obraz, a nie pomiar.

#### Strumień odbicia na żywo (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` dostarcza 8-bitową reflektancję dla każdego pasma jako
`multipart/x-mixed-replace` — plik M3C jako 3-kanałowy JPEG, a plik M3M jako obraz w skali szarości. Parametry zapytania:
`fps` (domyślnie 10), `jpeg_q` (domyślnie 95), `max_h` (0 = natywny), `band` (jedno nazwane pasmo, jak
powyżej), `align` (`1` do współrejestracji — te same zasady co w przypadku `align` z SDK), `refl_max` (8-bitowy
limit kodowania, jak powyżej). Każda część zawiera
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` oraz `X-Chloros-Reflectance-Clipped`.
**Części z różnych kamer zawierające ten sam identyfikator `X-Chloros-Group-Id` stanowią ten sam wyzwalacz** —
w ten sposób ponownie łączone są poszczególne strumienie z poszczególnych kamer w ramach jednej matrycy, a jest to gwarantowane
po stronie serwera, ponieważ elementy matrycy są dostarczane z zsynchronizowanej grupy klatek (zobacz
[Jeden wyzwalacz na odczyt](#one-trigger-per-read--group_id)). Zwraca `409`, gdy współczynnik odbicia jest
niedostępny w momencie nawiązania połączenia, i kończy strumień, jeśli stanie się on niedostępny w trakcie działania, więc zablokowany
strumień oznacza „brak współczynnika odbicia”, a nie „wciąż w porządku, tylko statyczny obraz”.

> **Format JPEG jest stratny — te piksele stanowią pomiar.** Błąd kompresji przekłada się bezpośrednio na
> wartości odbicia i jest najpoważniejszy na krawędziach, gdzie znajdują się granice materiałów. Z tego powodu
> domyślna jakość wynosi 95. Jeśli potrzebujesz nienaruszonych wartości liczbowych, użyj
> `grab(processing="reflectance")` / `frame_stream(...)`, które zwracają ten sam wynik bez
> kodeka w ścieżce (a `bit_depth=8`, jeśli ograniczeniem jest koszt przesyłu, a nie kodek).

> **Dane EXIF są przenoszone do eksportowanego pliku.** `process()` kopiuje blok GPS źródłowego nagrania
> **oraz jego ExifIFD** do każdego produktu, więc eksporty zawierają `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` i `CameraSerialNumber`, a także
> georeferencje. Na podstawie `FocalLength` program Pix4D oblicza odległość między punktami na ziemi — bez tego
> rekonstrukcja przyjmuje rażąco błędną skalę (w jednym z przypadków zmierzonego terenu o powierzchni 411 m
> przekształciła się w teren o powierzchni 47,8 km). Plik ten celowo nie nosi nazwy `-all:all`: tagi strukturalne IFD0 zakłócają
> dane wyjściowe LATTICE, a pliki `ExifImageWidth`/`Height` są wykluczone, ponieważ opisują źródłowe
> dane z rejestracji, a nie wyeksportowaną siatkę.

Podflagi etapu przechwytywania (dotyczą poziomów radiometrycznych — `radiance`, `reflectance`, `display`):

| Flaga | Domyślna | Znaczenie |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + pole płaskie + demiksowanie 3x3 + skala radiometryczna NIST. |
| `apply_white_balance` | `True` | Tabela LUT dla balansu bieli. Uwzględnia DLS, gdy do kamery podłączony jest moduł DAQ. |
| `apply_index` | `False` | Ocena wskaźnika wegetacji. |
| `index_expression` | `None` | Nadpisanie wzoru. Pustepuste → automatycznie włącza wskaźnik. |
| `annotated` | `False` | Nakładanie elementów dekoracyjnych GUI (zebra/siatka/peaking). Niedostępne dla `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Wartością zwracaną jest `chloros_sdk.CapturePaths`, typu `CapturePathMap`, a nie `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` to `Dict[str, Union[str, List[str]]]`: jednowarstwowa
> `processing` przypisuje każdej serii jedną ścieżkę, podczas gdy `processing="all"` przypisuje jej
> **uporządkowaną listę** wszystkich produktów zapisanych dla tej kamery. `ArrayHandle.capture()` nie przyjmuje
> żadnego `levels=` i ignoruje argumenty słowne, których nie rozpoznaje (`levels`, `daq`,
> `render_index`, `force_daq`), więc należy przekazywać tylko te, które są tutaj pokazane (`aligned=False`
> zapisuje pliki natywne, tak jak w przypadku `ArraySession.capture()`); dla tych elementów sterujących należy używać
> `ArraySession.capture()`. Aktywny kompozyt połączony, jeśli był przesyłany strumieniowo, trafia pod dodatkowy
> `"combined"`, a nie pod kluczem seryjnym. Kod zakładający `str` ulega awarii w
> postaci listy, mimo że żaden moduł sprawdzający typy nie zgłasza zastrzeżeń — adnotacja wskazywała `Dict[str, str]`
> przez pewien czas po wydaniu formy listowej, dlatego właśnie istnieje ten alias. Należy normalizować
> w przypadku, gdy wymagana jest forma płaska:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Czego nie można było zapisać: `.failed`.** Mapa zwracana przez `capture()` to
> `chloros_sdk.CapturePaths` — zwykłą podklasą `dict`, więc wszystko powyżej działa
> bez zmian — która zawiera również `.failed`: pliki, których backend nie mógł zapisać,
> te same wpisy co [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` oraz `disk_full: True` na zapełnionym dysku). Częściowe
> przechwycenie nadal zwraca wszystkie ścieżki, które zostały zapisane; na zapełnionym dysku mapa jest
> pusta, a plik `.failed` wyjaśnia dlaczego. `.skipped` wyświetla pominięcia dla poszczególnych kamer, a
> `.group_seq` — zsynchronizowaną grupę, a `.alignment` / `.oriented` to
> dane zaplecza, podobnie jak w przypadku `CaptureResult`. `capture_smart()` zwraca je jako
> `result["frames"].failed` oraz `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Wyrównanie tablic

`ArrayHandle` udostępnia pełną powierzchnię wyrównania. Profile są domyślnie przechowywane tylko w ramach sesji — aby je zapisać trwale, należy jawnie wywołać funkcję `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Wyrównanie w momencie połączenia

`connect_all(align=...)` może automatycznie wyrównać każdą tablicę w momencie połączenia:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

W przypadku braku określenia wartości stosowana jest domyślna opcja `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Zapisany czujnik światła (DAQ-U, DAQ-M lub DAQ-E), otwarty przez `connect_all()` w [puli czujników DAQ](#daq-sensor-sessions) — tej samej puli, z której korzystają `connect_daq_sensor()` i `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` oczekuje na **kolejne** widmo czujnika — to, które było już zbuforowane w momencie wywołania, nie jest brane pod uwagę — i zwraca `(spectrum, is_saturated, integration_time, x, y, z)` lub `None`, jeśli żadne nie nadejdzie w ciągu 5 s. Wstrzymany strumień zostaje wznowiony. `spectrum` jest wyrażony w W/m²/nm, gdy czujnik jest skalibrowany.
- `.sensor` to [`DAQSensorSession`](#sesje-czujnika-daq), w pozostałych przypadkach: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` oraz `.settings` to zapisane wpisy z `sensors.json`.

**Co przywraca `connect_all`.** Każdy czujnik jest otwierany pod swoim zapisanym adresem — portem COM, adresem Bluetooth lub hostem DAQ-E — a jeśli projekt zarejestrował identyfikator czujnika, musi to być właśnie ten czujnik: inne urządzenie pod tym adresem jest odrzucane jako niepodłączone. Stosowane są zapisane ustawienia czasu integracji, uśredniania klatek i automatycznej ekspozycji, a także korekcja czapki:

| Zapisany profil | Połączony z |
| --- | --- |
| Profil (np. `fov_30`) | Ten profil. |
| Nie wybrano | `sunshine_cosine`, domyślna opcja połączenia, zarejestrowana jako zakładana (`cap_id_source=auto_default`), więc przetwarzanie wyświetla ostrzeżenie. Czujniki są dostarczane bez osłony: należy wybrać zamontowaną osłonę. |
| `none` — wybrany jako **Brak (czujnik bez nasadki)** w interfejsie graficznym | Brak korekcji nasadki. |

DAQ-E-S nie przyjmuje żadnego profilu nasadki, niezależnie od tego, co zostało zapisane: jego dyfuzor stanowi część kalibracji (zobacz [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Projekt zapisany przez wcześniejszy model Chloros zarejestrował czujnik bez nasadki jako brak wyboru, więc ponownie łączy się z modelem `sunshine_cosine`; wybierz ponownie opcję **Brak (czujnik bez nasadki)** w interfejsie graficznym i zapisz projekt.

Czujnik, który inny klient ma już otwarty w puli (CLI, inny skrypt), jest podłączany w obecnym stanie, bez ponownego stosowania ustawień, a `disconnect_all()` pozostawia go uruchomionym; czujnik otwarty przez `connect_all()` zostaje zwolniony. Podobnie jak w przypadku `connect_daq_sensor()`, interfejs graficzny Chloros **nie** jest częścią puli: należy zamknąć czujnik w interfejsie graficznym przed podłączeniem projektu ze skryptu.**Wymagania i raport.** Czujniki światła wymagają uruchomionego backendu Chloros zalogowanego na płatnym planie Chloros+ (Copper lub wyższym), a `connect_all()` nigdy go nie uruchamia. Gdy żaden nie działa, każdy zapisany czujnik jest wymieniony w `report["sensors"]["errors"]` z odpowiednią informacją. Czujnik, którego backend nie może otworzyć, jest tam również wymieniony wraz z przyczyną podaną przez backend; `report["sensors"]["missing"]` pozostaje pusty dla czujników światła.

---

## Sprzęt bezpośredni (bez backendu)

Jeśli chcesz całkowicie uniezależnić się od modułu zaplecza w przypadku kamer LATTICE (CI, roboty bez głowicy, urządzenia wbudowane), zaimportuj bezpośrednio `lattice_sdk` — jest on ponownie eksportowany przez `chloros_sdk`. Uwaga dotycząca `CAMERA_AVAILABLE`: `lattice_sdk` znajduje się w pakiecie SDK wheel, ale wymaga obecności środowiska uruchomieniowego Arena SDK oraz biblioteki OpenCV (`cv2`), a pakiet wheel nie instaluje żadnego z nich. Czujniki światła przechodzą przez backend — [`connect_daq_sensor()`](#daq-sensor-sessions) lub `proj.sensors` danego projektu — z jednym wyjątkiem, pomocnikiem DAQ-M do obliczania wielkości pasmowych [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), który również wymaga `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Ustawienia wstępne i wyzwalacz

Trzy z czterech ustawień wstępnych działają w trybie **free-run**: kamera naświetla obraz w sposób ciągły, a
`capture()` zwraca kolejną klatkę. Wyjątkiem jest `triggered` — uzbraja on
aparat na sygnał zbocza sprzętowego na linii 2, więc nie rejestruje niczego, dopóki taki sygnał nie nadejdzie.

| Ustawienie | Wyzwalacz | Użyj, gdy |
| --- | --- | --- |
| `default` | tryb swobodny | ogólne zastosowanie |
| `high_speed` | tryb swobodny | 8-bitowy, ograniczenie do 60 klatek na sekundę, krótka ekspozycja |
| `high_quality` | tryb swobodny | 12-bitowy, bez ograniczenia klatek na sekundę — typowy wybór do zdjęć |
| `triggered` | **w stanie gotowości, linia 2** | kamera jest podłączona kablem synchronizacyjnym M8, a wyzwalanie odbywa się za pomocą innego urządzenia |

Jeśli wybierzesz `triggered` (lub samodzielnie ustawisz `trigger_mode="On"`) bez żadnego
sygnału sterującego na linii 2, każdy `capture()` zakończy się przekroczeniem limitu czasu — zgodnie z oczekiwaniami, ponieważ poprosiłeś
kamerę o oczekiwanie. SDK wyjaśnia to, gdy tak się dzieje; zobacz
[SC_ERR_TIMEOUT podczas przechwytywania](#direct-hardware-backend-free).

> **Uwaga — komunikaty „GVSP probe” / `SC_ERR_TIMEOUT -1011` pojawiające się podczas nawiązywania połączenia nie są błędami.**
> Podczas nawiązywania połączenia SDK próbuje wynegocjować**ramki jumbo** (pakiety GVSP o rozmiarze 9000 bajtów) w celu uzyskania większej przepustowości. W przypadku bezpośredniego połączenia punkt-punkt za pomocą karty sieciowej (np. adres lokalny typu link-local `169.254.x.x`) sieć zazwyczaj nie obsługuje ramek typu jumbo, więc próba ta kończy się przekroczeniem limitu czasu i w dzienniku pojawiają się wpisy takie jak:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Jest to **przewidziane rozwiązanie awaryjne**: SDK automatycznie powraca do standardowych pakietów o rozmiarze 1500 bajtów, a kamera kontynuuje nawiązywanie połączenia w normalny sposób (kolejne wiersze `[chunk-enable …]` stanowią część standardowej sekwencji nawiązywania połączenia). Przechwytywanie nadal działa.
>
> Można pominąć tę sondę, ale **nie służy ona jedynie wyciszeniu logów — wyłącza ona obsługę ramek typu jumbo.** Kamera odpowiada na pingi z opcją „Don&#x27;t-Fragment” tylko do 1500 bajtów, niezależnie od tego, jak dobra jest Twoja sieć, więc sam test pingowy nigdy nie wykryje ramek typu jumbo; tylko ten test jest w stanie to zrobić. Wyłącz go, a kamera będzie na zawsze wysyłać standardowe pakiety o rozmiarze 1500 bajtów, w każdej sieci:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Opłaca się to tylko w sieci, o której *wiesz*, że nie obsługuje ramek typu jumbo, gdzie pozwala to zaoszczędzić około sekundy czasu połączenia na każdą kamerę. Ponieważ jest to rzeczywista zmiana, a nie tylko kosmetyczna, SDK informuje o tym podczas korzystania z tej funkcji:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nie zmieniaj tego ustawienia, chyba że masz ku temu powód.**Jeśli opcja pozostaje włączona, każda kamera jest mierzona raz na sesję backendu oraz ponownie w przypadku zmiany połączenia (inna karta sieciowa, inna prędkość łącza lub utrata połączenia, np. w wyniku zmiany przełącznika); przy ponownym połączeniu wykorzystywany jest ten sam wynik.**Po zmianie za przełącznikiem, który pozostał aktywny, należy ponownie uruchomić backend, aby przeprowadzić pomiar od nowa.** Pierwsze połączenie może czasami zająć kilka minut podczas tej kontroli, a dziennik informuje o tym, gdy tak się stanie.
>
> Jeśli *chcesz* uzyskać przepustowość pakietów jumbo, włącz obsługę pakietów jumbo w całym łańcuchu (MTU karty sieciowej 9000 + przełącznik, który je przepuszcza) lub zablokuj go za pomocą `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, gdy wiesz, że łącze je obsługuje — choć lepiej jest używać `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` dla poszczególnych poleceń niż ustawiać to na stałe, ponieważ stały rozmiar pomija testy i uniemożliwia dostosowywanie się do sieci znajdującej się przed nim. **Każde** urządzenie na ścieżce musi przepuszczać pakiety jumbo — w tym wszelkie rozgałęźniki lub iniektory PoE, co jest typową przyczyną, dla której konfiguracja, która w innym przypadku obsługiwałaby pakiety jumbo, nie może ich przesyłać.

> **`SC_ERR_TIMEOUT -1011` podczas `capture()` / `grab*()` to inny problem — ten jest prawdziwym błędem.**
> Powyższa uwaga dotyczy wyłącznie błędu `-1011` zarejestrowanego przez**sondę czasu połączenia**. Ten sam błąd zgłoszony w wyniku**przechwytywania** oznacza, że kamera połączyła się poprawnie, ale nie wysyła żadnych obrazów:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Cechą charakterystyczną jest sytuacja, w której kanał *sterujący* kamery działa poprawnie — wykrywanie przebiega pomyślnie, ustawienia oraz zapisy `[chunk-enable …]` kończą się sukcesem — podczas gdy *każda* klatka przekracza limit czasu.
>
> **Najczęstszą przyczyną jest to, że kamera jest ustawiona na wyzwalanie sprzętowe.** W przypadku błędów `trigger_mode="On"` i `trigger_source="Line2"` kamera nie wysyła żadnych danych, dopóki na kablu synchronizacyjnym M8 nie pojawi się zbocze sygnału. Jeśli nie ma kabla sterującego tą linią, każde pobranie danych trwa w nieskończoność. Kamera nie jest uszkodzona, a sieć działa prawidłowo — działa dokładnie tak, jak została zaprogramowana.
>
> Modele `CameraSettings()` oraz `default` / `high_speed` / `high_quality` ustawia tryb swobodnej pracy, a przechwycenie, które przekroczy limit czasu podczas uzbrojenia, wyświetla odpowiedni komunikat zamiast samego kodu `-1011`. `PRESETS["triggered"]` uzbraja linię 2 zgodnie z założeniami.
>
> Aby wymusić tryb swobodnej pracy dowolnej kamery:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Jeśli nadal występuje przekroczenie limitu czasu przy ustawieniu `trigger_mode="Off"`, oznacza to, że kamera faktycznie nie przesyła danych — prześlij nam dziennik oraz `ip link show`.

#### Profile kolorów (podgląd na żywo RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` określa profil kolorów wyświetlacza dla **podglądu na żywo** w kamerach RGB (kamery wielospektralne ignorują to ustawienie):

| Profil | Znaczenie |
| --- | --- |
| `raw` | Całkowite pominięcie łańcucha radiometrycznego. |
| `linear` | DSNU + flat + WB, bez CCM, bez gamma. |
| `natural` | Liniowy + zmierzony CCM + gamma sRGB, wyłącznie z tanim wykończeniem (wygładzanie chrominancji + desaturacja świateł) — realistyczne ustawienie domyślne. |
| `enhanced` | `natural` plus pełne wykończenie typu „hub-parity” (usuwanie fringingu, intensywność, lokalny kontrast CLAHE). Bogatszy wygląd przy mniej więcej **dwukrotnie wyższym koszcie wykończenia na klatkę**, a więc niższa liczba klatek na sekundę w trybie LIVE. |
| `custom_temp` | `natural`, ale balans bieli zablokowany na `custom_cct_k` kelwinach (DLS ignorowane; ograniczone do 2000–10000 K po stronie serwera). |

Profil ten jest **przeznaczony wyłącznie do podglądu na żywo** i służy do regulacji szybkości/wyglądu: zapisane zrzuty zawsze otrzymują pełne, bogate wykończenie niezależnie od wybranego profilu, więc wybór `natural` w celu odzyskania czasu klatki nie obniża jakości tego, co trafia na dysk. Nieznany profil powoduje wywołanie `ValueError`; gdy backend chloros jest dostępny, zmiana jest również wysyłana do niego za pomocą POST, dzięki czemu następna klatka podglądu ją odzwierciedla (użytkownicy korzystający bezpośrednio z SDK’a bez backendu nadal otrzymują aktualizację ustawień).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Najjaśniejsze obszary (wyświetlacz RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` wybiera najjaśniejsze, najbardziej białe piksele **wyświetlacza** RGB — poprawka dotycząca jasnej białej powierzchni, która renderuje się jako płaska biała plama. `value` to `-100`..`0` (z ograniczeniem); `0` jest wyłączony, a renderowanie wygląda wtedy dokładnie tak, jak bez tego ustawienia. Silniejsze ustawienia obniżają biel wyświetlacza (do około 60% przy `-100`) i poszerzają krzywą tonów, dzięki czemu powierzchnia, którą automatyczna regulacja jasności w podglądzie „wybieliła” do czystej bieli, odzyskuje swoje szczegóły.

Jest to ustawienie wyświetlacza, ale w przeciwieństwie do nasycenia i kontrastu (które zmieniają jedynie podgląd na żywo) wpływa ono również na zapisywane dane: podgląd na żywo oraz zdjęcia na **poziomie wyświetlacza** dostosowują się do niego, więc pliki te również wychodzą ciemniejsze (mają ustawienie `Chloros:DisplayHighlights`); eksporty promieniowania i odbicia nigdy tego nie robią, a opcja ta nie może odtworzyć danych z matrycy, które uległy przesterowaniu — w tym celu należy zmniejszyć ekspozycję. Kamery wielospektralne i monochromatyczne ignorują tę opcję. Wartość jest zapisywana w `settings.highlights` i wysyłana metodą POST do dostępnego backendu chloros, dokładnie tak jak `set_color_profile`; odpowiednikiem w CLI jest `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Kamery monochromatyczne (M3M) i `Calibration`

Kamera monochromatyczna **M3M** (`M3M-<lens>-F<wavelength>`) jest jednopasmowa: jedna płaszczyzna skali szarości, brak mozaiki Bayera, brak macierzy przesłuchu spektralnego 3×3. `Calibration` rozpoznaje ją i udostępnia flagę `is_mono`. Odbicie nadal ma zastosowanie jako mapa radiometryczna dla poszczególnych pasm (matryca rozdzielająca to macierz tożsamościowa), ale obliczenia wielopasmowe na pojedynczej kamerze dają sensowne wyniki, a nie bezsensowne:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Aby zbudować indeks wegetacji przy użyciu sprzętu monochromatycznego, należy połączyć kilka kamer M3M o różnych długościach fal w wyrównany stos wielopasmowy (zobacz [Wyrównanie macierzy](#array-alignment)) i obliczyć indeks dla całego stosu zamiast dla jednej kamery.

#### Wielkości pasmowe DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` łączy się z DAQ-M przez Bluetooth za pomocą własnego adresu MAC BLE. Jest to jedyna klasa czujników światła, która komunikuje się bezpośrednio z czujnikiem, a nie poprzez zaplecze, więc wymaga tego, czego potrzebują klasy kamer bezpośrednich (`CAMERA_AVAILABLE`: środowisko uruchomieniowe Arena SDK oraz OpenCV), a dodatkowo `bleak` dla Bluetooth (`pip install "chloros-sdk[camera]"` lub `pip install bleak`; bez tego `DLS` generuje błąd `DLSError`). Urządzenie DAQ-M podłączone gdzie indziej (w puli backendu, w projekcie, w GUI) przestaje nadawać sygnał, więc najpierw należy go tam zwolnić. **Wymagany jest `mac_address`**: nie ma wartości domyślnej, ponieważ każda jednostka ma swoją własną, a `DLS()` bez niej wywołuje `DLSError` przed podjęciem jakiejkolwiek próby połączenia. Znajdź adres MAC za pomocą `discover_daq_sensors(transports=["ble"])` (zobacz [Wykrywanie](#discovery--finding-an-address-to-connect-with)) lub `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integruje widmo DAQ-M (najnowszy odczyt, gdy `spectrum` wynosi `None`; `DLSError`, jeśli jeszcze go nie ma) z wykorzystaniem odpowiedzi trzech kanałów kamery i zwraca `{"red": …, "green": …, "nir": …}` — nazwy kluczy odpowiadają kolejności kanałów; w przypadku krzywych kamery RGB trzeci (`nir`) to kanał niebieski.

- Bez `spectral_response` wykorzystuje krzywą flotową modelu filtra (`filter_model`, na podstawie której zbudowano `DLS`), która została zmierzona na jednym egzemplarzu.
- `spectral_response` wykorzystuje natomiast **własne** krzywe pomiarowe kamery: słownik `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, zapisany w pakiecie kalibracyjnym tej kamery (`color_correction.spectral_response_smoothed` — kamery RGB; pakiety wielospektralne nie zawierają takiej krzywej). W przypadku słownika, którego nie można wykorzystać, stosowana jest krzywa floty.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

Funkcja `chloros-cli lattice dls --camera SERIAL` w interfejsie CLI działa w ten sam sposób poprzez backend, wyszukując krzywą kamery w załadowanym lub zapisanym w pamięci podręcznej pakiecie kalibracyjnym i podając, której krzywej użyto. W protokole HTTP jest to `POST /api/dls/cli-read` wraz z `mac` (wymagane: bez tego odpowiedź to HTTP 400, `code` `MAC_REQUIRED`) oraz `camera_serial` w treści; `band_magnitudes_source` w odpowiedzi to `camera <serial>`, gdy użyto krzywej właściwej dla tej kamery, lub `model <filter>` dla krzywej floty (kamera bez krzywej: nie jest to RGB lub nie załadowano pakietu ani w pamięci podręcznej).

---

## Wyjątki

Przechwytuj klasę bazową, aby obsłużyć „wszelkie nieprawidłowości w Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` i `ChlorosConfigurationError` są atrybutami `chloros_sdk` (oraz `chloros_sdk.exceptions`), ale nie występują w `__all__`, więc `from chloros_sdk import *` ich nie importuje: należy je zaimportować według nazwy. Rodzina `ChlorosError` jest rozszerzana przez `ChlorosLocal` oraz funkcje oparte na niej (`process_folder()`, `process_lattice_capture()`); powierzchnia Smart-Connect dziedziczy po `ChlorosConnectError`.

Hierarchia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Przykłady kompleksowe

### 1. Przetwarzanie folderu z niestandardowym paskiem postępu

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Tablica LATTICE na żywo → odbicie + odniesienie DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Kampania przechwytywania danych w ramach projektu

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Strumień klatek z wielu kamer → potok NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skrypt przechwytywania bez interfejsu graficznego, bezpośrednio na sprzęcie (bez zaplecza)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Sprawdzanie możliwości przed podłączeniem macierzy 4 kamer

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Odpowiednik receptury przechwytywania (w czystym Pythonie)

`run_project_recipe()` uruchamia recepturę CLI bez zmian (zobacz [Folder projektu i receptury](#project-folder--recipes)). Te same kroki, napisane w języku Python, wyglądają następująco:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Serwer MCP (asystenci AI)

`chloros-mcp` umożliwia asystentowi AI obsługującemu [Model Context Protocol](https://modelcontextprotocol.io) (MCP), takiemu jak Claude Desktop, Claude Code lub Cursor, sterowanie systemem Chloros: przetwarzanie obrazów (tworzenie projektu, importowanie folderu z ujęciami, wybierać ustawienia, przetwarzać dane oraz generować raporty dotyczące wyników przetwarzania), a także sterować kamerami LATTICE, matrycami i czujnikami światła DAQ (łączyć, wyświetlać podgląd, ustawiać ekspozycję i kolory, rejestrować, nagrywać, aktualizować oprogramowanie sprzętowe). Jego narzędzia obejmują funkcje tego SDK. Wystarczy sformułować polecenie prostym językiem, a asystent uruchomi odpowiednie narzędzia serwera:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Serwer jest zbudowany w oparciu o ten SDK: narzędzia do przetwarzania wywołują [`ChlorosLocal`](#image-processing--chloroslocal), a narzędzia sprzętowe [wywołują smart-connect](#smart-connect-for-lattice-cameras), [sesje DAQ](#daq-sensor-sessions), `ChlorosProject` oraz [narzędzia kamery](#camera-utilities), a wszystkie one komunikują się z tym samym zapleczem, z którego korzysta aplikacja Chloros. Żądania te zawierają `X-Chloros-Client: sdk`, więc backend stosuje sprawdzanie logowania i planu SDK, a wraz z tym obowiązują reguły SDK (na przykład funkcja Texture Aware jest odrzucana w przypadku przechwytywania obrazu przez LATTICE).

### Instalacja

Serwer jest dostarczany w pakiecie `chloros-sdk`; nie trzeba instalować niczego więcej. SDK instaluje polecenie `chloros-mcp`, które jest takie samo jak `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** instalator Chloros instaluje SDK za pomocą pip w środowisku Python, umieszczając plik `chloros-mcp.exe` w folderze `Scripts` tego środowiska Python.
- **System Linux (.deb):** pakiet instaluje SDK dla systemu `python3` i umieszcza polecenie `chloros-mcp` w `/usr/bin`, dzięki czemu jest ono dostępne w katalogu `PATH`. W środowisku virtualenv `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` dodaje `chloros-mcp` do katalogu `bin` tego środowiska.

Serwer korzysta wyłącznie z biblioteki standardowej w ramach SDK (Python 3.7+). Uruchamia go klient MCP i komunikuje się z nim poprzez stdin/stdout; nie uruchamia się go samodzielnie, z wyjątkiem przypadku `--help`.

### Konfiguracja klienta

**Claude Desktop:**dodaj serwer do `claude_desktop_config.json` (w systemie Windows: `%APPDATA%\Claude\claude_desktop_config.json`; otwiera się to poprzez**Ustawienia &gt; Programista &gt; Edytuj konfigurację**), a następnie uruchom ponownie Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Kod Claude’a:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kursor:** ten sam wpis `mcpServers` co w Claude Desktop, w pliku `~/.cursor/mcp.json` (dla każdego projektu; `%USERPROFILE%\.cursor\mcp.json` w systemie Windows) lub `.cursor/mcp.json` w jednym projekcie.

Opcje należy umieścić w `args` lub po poleceniu dla Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Aby użyć konkretnego SDK Python zamiast `chloros-mcp` w `PATH`, należy uruchomić moduł z tym Pythonem (podając jego pełną ścieżkę jako `command`, jeśli nie jest to pierwszy `python3` na `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Klient uruchamia serwer bez Twojej powłoki, więc jeśli zgłasza, że serwer nie uruchomił się, prawdopodobnie `chloros-mcp` nie znajduje się na `PATH` widocznym dla klienta. Zamiast tego podaj pełną ścieżkę (`where chloros-mcp` w systemie Windows, `which chloros-mcp` w systemie Linux, gdzie plik .deb to `/usr/bin/chloros-mcp`) lub uruchom moduł przy użyciu środowiska Python z pakietem SDK.

### Opcje

| Opcja | Domyślnie | Opis |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend Chloros, który ma być używany. |
| `--no-auto-start` | wyłączone | Nigdy nie uruchamiaj backendu: użyj już działającego, takiego jak aplikacja Chloros, usługa backendu w systemie Linux lub tunel do folderu na innym komputerze (zobacz [Tryb zdalnego backendu](#remote-backend-mode-pip-only-host-via-tunnel); ścieżki do folderów muszą wówczas wskazywać ten sam folder na obu maszynach). |
| `--backend-exe PATH` | tam, gdzie instaluje go Chloros | Backend, który należy uruchomić, gdy żaden nie działa, w przypadku instalacji Chloros w miejscu, którego SDK nie przeszukuje. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` lub `ERROR`. Logi są kierowane do stderr, co większość klientów pokazuje w swoim logu MCP; stdout przenosi wyłącznie komunikaty protokołu. |

### Narzędzia

Argumenty każdego narzędzia są zgodne z wywołaniem SDK, które ono obsługuje; klient wymienia każdy z nich wraz z pełnym schematem. Ścieżki są bezwzględne.

**Przetwarzanie**

| Narzędzie | Wywołanie SDK | Co robi |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Tworzy projekt i ustawia go jako otwarty projekt, na którym działają narzędzia przetwarzające. Podanie samej nazwy powoduje utworzenie folderu w domyślnym folderze projektów; podanie ścieżki bezwzględnej powoduje utworzenie go w tej lokalizacji. |
| `open_project` | `ChlorosLocal.open_project` | Otwiera zapisany projekt i ustawia go jako projekt otwarty. Ścieżki przeniesionego projektu są przekierowywane do folderu, w którym się on obecnie znajduje; nazwy `relink_folder` (absolutne) wskazują folder, w którym znajdują się obecnie pozostałe brakujące pliki. Zwraca `source_files` i `image_count`. Nie znaleziono żadnych: wyświetla komunikat o błędzie z informacją o ich lokalizacji i nazywa `relink_folder`; nic nie jest otwarte. |
| `import_images` | `import_images` | Importuje pary plików `.RAW`+`.JPG` z programu Survey3, a program LATTICE `.tif` przechwytuje, a DAQ `.daq` rejestruje; zwraca `count` (obrazy) i `scan_count` (nagrania). |
| `configure_processing` | `configure` | Zmienia przekazane ustawienia i żadne inne (wartości: [Obsługiwane wartości](#supported-values)); `custom_settings` wysyła całą konfigurację w podanej postaci. |
| `get_project_settings` | `get_config` | Odczytuje zapisane ustawienia przetwarzania, eksportu i indeksowania otwartego projektu. |
| `start_processing` | `process(wait=False)` | Rozpoczyna przetwarzanie otwartego projektu i natychmiast kończy działanie. |
| `get_processing_status` | `get_status` | Zgłasza `state` (`idle`, `starting`, `processing` lub `finished`) oraz postęp; po zakończeniu – w przypadku `summary`, `export_check_failure`, gdy nie udało się zweryfikować eksportów, oraz `image_failure`, gdy utracono obrazy: pozostałe produkty są zapisywane; zgłoś przebieg jako nieudany. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Tworzy projekt dla folderu, importuje go, konfiguruje i uruchamia sesję w ramach jednego wywołania (`lattice: true` dla przechwytywań LATTICE). Gdy przebieg jest w toku, operacja zostaje odrzucona, zanim cokolwiek zostanie utworzone, a backend odrzuca wówczas również zmianę projektu z dowolnego narzędzia. |
| `export_light_sensor` | `export_light_sensor` | Zapisuje skalibrowane pliki `.daq` i `.csv` dla nagrań z czujnika światła w projekcie bez przetwarzania obrazów, pobierając kalibrację fabryczną z chmury MAPIR, jeśli nie jest ona zapisana w pamięci podręcznej (patrz [Zapisy czujnika światła](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Odczytuje metadane audytu Chlorosa przetworzonego obrazu (źródło, kalibracja, zastosowane etapy). |

**Kamery i matryce**

| Narzędzie | Wywołanie SDK | Co robi |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Co jest obecnie podłączone w dowolnym kliencie, każdy z `connected_here`; oraz zadania uruchomione na serwerze. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamery i czujniki światła, które można podłączyć. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Stan na żywo jednej kamery, macierzy (wraz z jej możliwościami i stanem technicznym) lub czujnika światła. |
| `analyze_array_network` | `analyze_array_network` | Jakie obciążenie może obsłużyć ten host i sieć dla proponowanej macierzy. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Podłącz kamerę lub zsynchronizowaną matrycę (najpierw urządzenie nadrzędne). |
| `connect_project_devices` | `open_project(...).connect_all` | Podłącz wszystkie urządzenia zapamiętane w zapisanym projekcie wraz z ich ustawieniami. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Odłącz kamerę, układ lub czujnik światła (zobacz sekcję „Zachowanie”). `keep_settings: true` na macierzy zapisuje ustawienia każdej kamery do jej własnej pamięci, a `clear_settings: true` resetuje tę pamięć (wynik tej operacji to `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Urządzenia projektu razem. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Odczyt lub zapis ustawień kamery na podstawie nazwy przyjaznej lub nazwy węzła GenICnazwy węzła GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Ustaw stałą częstotliwość tablicy lub przywróć częstotliwość automatyczną. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Pojedyncze przechwycenie (`fastest` i `wait_for_exposure` to `capture_fastest` oraz zdjęcie o ustalonej ekspozycji). W przypadku braku `output_dir` zapisuje się je w otwartym projekciefolderze `captures`, a pliki dołączają do projektu (zatrzymane do zakończenia cyklu przetwarzania); informuje o tym odpowiedź `project_add`. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Przechwytywanie ciągłe lub interwałowe, jako zadanie. |
| `start_array_recording` | `ArraySession.record` / `burst` | Film z indeksem łącznym lub surowa seria zdjęć, jako zadanie. |
| `build_burst_video` | `ArraySession.build_video` | Ponowne przetworzenie serii zdjęć na skalibrowany materiał wideo w ramach zadania. |
| `get_job_status` / `stop_job` | | Śledzenie lub zatrzymanie zadania. |
| `get_camera_preview` | | Podgląd na żywo z każdej kamery, zwracany jako obraz. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Jedna klatka (lub jeden zsynchronizowany zestaw) na poziomie przetwarzania, bez zapisywania: statystyki dla poszczególnych płaszczyzn oraz plik PNG dla każdej z nich. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Wyrównanie tablicy. |

**Czujniki światła**

| Narzędzie | Wywołanie SDK | Co robi |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Podłącz moduł DAQ-U, DAQ-M lub DAQ-E (`cap_id` ustawia korekcję nasilenia). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Najnowsze odczyty: sygnatura czasowa, nasycenie, zakres widma i wartość szczytowa oraz widmo uśrednione w przedziałach długości fal (`full_spectrum` dla każdego punktu). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Rozpoczęcie lub zatrzymanie strumienia. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Nagrywanie pliku `.daq` jako zadania; `duration_s` samoczynnie go zatrzymuje. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Korekcja ogranicznika (`none` dla gołego czujnika). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Który czujnik dostarcza dane o odbiciu światła dla poszczególnych kamer i matryc. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Zestawy pasm oraz współczynniki odbicia otoczenia i obiektu oraz indeksy dla dwóch czujników strumieniowych. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Własne ustawienia urządzenia DAQ-E, zapisane w urządzeniu. |

**Narzędzia kamery** ([Narzędzia kamery](#camera-utilities))

| Narzędzie | Wywołanie SDK | Opis działania |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` oraz ich funkcje ustawiające | Kolor wyświetlacza kamery RGB: profil i temperatura barwowa, nasycenie / kontrast, światła. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Jednorazowy balans bieli lub tryb automatyczny / wyłączony / stałe współczynniki. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Ustawienia wstępne kamery. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modele filtrów lub pojedynczy filtr według nazwy. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Wersje oprogramowania sprzętowego oraz błysk (domyślnie zadanie). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stan zasilania i temperatury; tryb uśpienia lub czuwania. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Współczynniki z wyświetlanego celu kalibracyjnego. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Karty sieciowe kamery, szacunkowe dane i poprawka (tylko dla administratora). |
| `apply_vignette_correction` | `apply_vignette_correction` | Korekcja winietowania folderu obrazów. |
| `read_dls` | `read_dls` | Jeden odczyt DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Sprzęt i środowisko uruchomieniowe hosta. |

**Kamery przejściowe, profile wyrównania i indeksy** ([Tablice przejściowe, profile wyrównania i indeksy](#transient-arrays-alignment-profiles--indices)). Każde narzędzie w tym miejscu, które otwiera kamery, przyjmuje `keep_settings` / `clear_settings` (domyślnie false): zobacz [Pamięć kamery](#camera-memory--keep_settings--clear_settings).

| Narzędzie | Wywołanie SDK | Co robi |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Nagrywanie z jednej kamery lub synchroniczna transmisja strumieniowa z kilku kamer w ramach jednego zadania (`duration_s` samoczynnie ją zatrzymuje). |
| `capture_cameras_transient` | `capture_cameras_transient` | Jedno zsynchronizowane przechwytywanie bez podłączania macierzy. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Sprawdź synchronizację wyzwalacza oraz które kamery znajdują się na linii synchronizacji. |
| `analyze_array_models` | `analyze_array_models` | Jakie wyniki uzyskałaby macierz tych modeli w tej sieci. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Rozwiązanie i zapisanie pliku profilu wyrównania (`options` zawiera bardziej szczegółowe opcje solwera). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Wyrównany obraz wielopasmowy lub strumień takich obrazów w ramach zadania. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Zawartość pliku profilu; kolejność pasm i włączone kamery. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Ustawienia wstępne indeksu i gradienty; indeks do wielo pasmowego pliku TIFF; przechwytywanie do indeksu w jednym kroku. |

**Synchronizacja czasu, folder projektu i receptury** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Folder projektu i receptury](#project-folder--recipes))

| Narzędzie | Wywołanie SDK | Co robi |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Stan zdrowia grandmastera PTP, jego węzłów równorzędnych oraz stan PTP każdej kamery macierzowej. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Uruchom go ponownie; ustaw jego priorytety. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Folder, w którym tworzone są nowe projekty. |
| `run_project_recipe` | `run_project_recipe` | Uruchom recepturę zapisanego projektu jako zadanie wraz z dziennikiem. Najpierw sprawdzane: receptura zawierająca błąd jest odrzucana i nic nie jest uruchamiane; pojawia się ostrzeżenie o niepodłączonych tablicach / czujnikach (nie podłącza ich: najpierw `connect_project_devices`). `dry_run` kończy sprawdzanie i nic nie uruchamia. |

Nie ma narzędzia do otwierania zapisanego projektu w celu przetwarzania: każde zadanie przetwarzania rozpoczyna się od `create_project` (lub `process_folder`).

### Wymagania

- **Chloros zainstalowany na tym samym komputerze.** Przetwarzanie odbywa się w backendzie Chloros. Serwer może zamiast tego korzystać z backendu dostępnego przez tunel (`--backend-url` z `--no-auto-start`, zobacz [Opcje](#options)).
- **Zalogowanie się na tym komputerze** w aplikacji Chloros lub za pomocą `chloros-cli login`. Serwer nie może zalogować się za Ciebie: narzędzie wywołane przy wylogowanym użytkowniku zwraca komunikat „Nie jesteś zalogowany do Chloros” wraz z przyczyną podaną przez backend, a asystent poinformuje Cię o tym.
- **Dowolny płatny plan Chloros+ (Copper lub wyższy).** Serwer wymaga dostępu do SDK, a bezpłatny plan Iron go nie zapewnia: narzędzie wywołane w ramach bezpłatnego planu zwraca komunikat „Niedostępne w tym planie Chloros” wraz z przyczyną podaną przez backend.
- Python 3.7 lub nowszy z SDK oraz klient MCP, który uruchamia lokalne serwery (stdio).

### Działanie

- **Backend.** Uruchomienie serwera lub wyświetlenie listy jego narzędzi nie powoduje uruchomienia niczego; pierwsze wywołanie narzędzia nawiązuje połączenie. Jeśli aplikacja Chloros (lub usługa backendowa w systemie Linux) jest uruchomiona, serwer korzysta z jej backendu i pozostawia go uruchomionym. Jeśli żaden backend nie jest uruchomiony, serwer uruchamia zainstalowany backend w trybie bezokienkowym. Gdy serwer się zamyka (użytkownik zamyka klienta lub serwer zostaje ponownie uruchomiony), zatrzymuje on rozpoczęte zadania przechwytywania i nagrania z czujników światła oraz zwalnia podłączone urządzenia; backend nadal działa, dzięki czemu niedokończony proces przetwarzania jest kontynuowany do końca, a sam backend wyłącza się po 30 minutach bezczynności. W przypadku `--no-auto-start` serwer nigdy nie uruchamia backendu: wywołanie narzędzia przy braku uruchomionego backendu wyświetla prośbę o otwarcie aplikacji Chloros, a kolejne wywołanie po jej uruchomieniu nawiązuje połączenie.
- **Jeden otwarty projekt.** Backend ma jeden otwarty projekt, współdzielony z aplikacją Chloros, gdy aplikacja jest otwarta, więc nie należy pracować nad innym projektem w aplikacji, gdy asystent jest aktywny.
- **Zmieniają się tylko przekazane ustawienia.** `configure_processing` pozostawia wszystkie ustawienia, które nie zostały przekazane, na ich zapisanych wartościach (SDK `configure()` resetuje debayer, korekcję winietowania, kalibrację odbicia, PPK i format eksportu do wartości domyślnych, gdy nie są one przekazane; serwer uzupełnia je na podstawie projektu). Jeśli nie może odczytać zapisanych ustawień projektu, nie wprowadza żadnych zmian i informuje o tym, chyba że podano wszystkie pięć parametrów. `indices` zastępuje listę indeksów projektu. Wynik zawiera listę elementów wysłanych jako `settings_sent`, a pod `warnings` wymienia indeksy, których pasma kamery nie są w stanie utworzyć.
- **Przetwarzanie odbywa się w trybie „start-then-poll”.** `start_processing` zwraca wynik zaraz po rozpoczęciu przetwarzania, które trwa od kilku minut do kilku godzin, a asystent kontynuuje proces za pomocą `get_processing_status`. `state` to `idle` (brak przetwarzania), `starting` (backend nie zgłosił jeszcze przebiegu, co może trwać do 20 s), `processing` lub `finished`, wraz z [podsumowanie](#post-run-summary--hints): co zostało wyeksportowane oraz wskazówki, gdy wyeksportowano mniej niż oczekiwano (brak obrazów, nie znaleziono celu kalibracji). Wyniki są zapisywane w folderze projektu, tak jak w aplikacji. `start_processing` jest odrzucany, gdy przebieg jest w toku, a backend, który przestaje odpowiadać, jest zgłaszany jako błąd, a nie jako zakończony przebieg.
- **Co widzi asystent.** Ostrzeżenia z SDK pojawiają się w wynikach narzędzia pod kodem `warnings`, a niepowodzenia (wylogowanie, plan, nieprawidłowa ścieżka) są wyświetlane jako tekst błędu, który asystent może odczytać i przekazać użytkownikowi. Lista zawierająca więcej niż 20 pozycji jest skracana do pierwszych 20, a jej pełna długość jest podawana jako `<key>_total`. Ścieżki muszą być bezwzględne.
- **Urządzenia są współdzielone.** Kamery, matryce i czujniki światła znajdują się w zapleczu, więc asystent może sterować jednym z nich za pośrednictwem połączonej aplikacji Chloros, a wszelkie zmiany są widoczne w aplikacji. `disconnect_device` udostępnia informacje o tym, z czym połączył się serwer; urządzenie, z którym połączył się inny klient (aplikacja, `chloros-cli`, skrypt, inny `chloros-mcp`) jest zwalniane wyłącznie za pomocą `force: true`, czyli `force_disconnect` z SDK, a w przypadku odmowy podaje się nazwę podmiotu, który je podłączył, o ile backend posiada tę informację; urządzenie, którego backend w ogóle nie obsługuje, jest zgłaszane jako niepodłączone (`released: false`, `connected: false`, nie ma czego zwolnić). Nagranie lub strumień uruchamiany przez backend generuje widok `get_job_status` wraz z `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (grupy klatek dla strumienia z wielu kamer) — oraz jednowierszowy wpis `summary` („220 klatek zapisanych, 55 odrzuconych, 0 nieudanych: ...”) podczas działania oraz, po zakończeniu, końcowe wyniki; lista tego wpisu u innego klienta (`other_clients_jobs`) zawiera te same dane (`list_cameras()`, `list_arrays()` i `list_daq_sensors()` zawierają dane każdego urządzenia`connected_by`: `client`, `label`, `name`, `at`). Każde urządzenie `chloros-mcp` identyfikuje się w backendzie jako `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, wysyłane jako `X-Chloros-Client-Label`; każdy skrypt SDK może to ustawić). Zatrzymanie nagrywania z czujnika światła innego klienta również wymaga `force`.
- **Długotrwała praca to zadanie.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, a każde żądanie przechwycenia, rozwiązania lub indeksu dotyczące `background` zwraca natychmiast `job_id`; następuje po nim `get_job_status`, a kończy je `stop_job` . Przebieg trwa do momentu zatrzymania, chyba że podasz `max_captures`, `duration_s` (lub, w przypadku serii, `max_frames`); `min_free_gb` odmawia uruchomienia, gdy na dysku jest za mało wolnego miejsca. Nie obowiązuje żadne ograniczenie, chyba że je ustawisz. Nagranie lub strumień uruchamiany przez backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) zachowuje identyfikator zadania serwera, więc kolejne zadanie `chloros-mcp` — uruchomione później — odnajduje je na podstawie tego identyfikatora: `get_job_status` kontynuuje go (i wyświetla takie zadania jak `other_clients_jobs`, od `GET /api/camera/cli-jobs`, z których każde ma `started_by`), a `stop_job` zatrzymuje je dopiero za pomocą `force: true`. Każde inne zadanie (przebieg przechwytywania, zapis tablicowy lub serię, przechwytywanie w tle) działa w ramach zadania `chloros-mcp`, które je uruchomiło, i tylko ten serwer o tym wie: inny serwer, który o to poprosił, tak informuje. Zatrzymane zadanie odczytuje `stopping` podczas zapisywania swojej kolejki, a następnie `finished` z końcowymi liczbami, przez 10 minut, a z dowolnego serwera `chloros-mcp`: ten, który to uruchomił, widzi wymuszone zatrzymanie innego serwera w ten sam sposób. `get_job_status` nigdy nie czeka na zatrzymanie: podczas zatrzymania (własnego `stop_job`, innego klientalub zatrzymania, które zwalnia kamery z zadania, które zakończyło się samoistnie) odpowiada natychmiast za pomocą `stopping` i aktualnych liczb, a `stop_job` zadania już się zatrzymującego informuje o tym. Zadanie, które się zakończyło, podaje powód w `stop_reason`: własną przyczynę (`no_frames`, `disk_full`, `member_missing`, lub w przypadku ciągłego przebiegu tablicy: `count`, `duration`, `array_gone`), a `limit reached` tylko wtedy, gdy jego wartości wskazują na `count` (lub `max_frames` dla serii, `duration_s` dla nagrania)`duration_s`) zostało osiągnięte, `error`, `stopped` (z `stopped_by`), gdy klient go zatrzymał, w przeciwnym razie `finished`.
- **Gdzie trafiają przechwyty.** Do folderu `captures` otwartego projektu, gdzie aplikacja je zapisuje, chyba że podano `output_dir`.
- **Obrazy.** `get_camera_preview` i `grab_frames` zwracają obrazy, które asystent może przeglądać, oprócz ich danych w formacie JSON. Obrazy `grab_frames` są poddawane korekcji kontrastu w celu ułatwienia przeglądania; ich statystyki odzwierciedlają rzeczywiste wartości.
- **Gdy serwer się zamyka**, uruchomione przez niego zadania zostają zatrzymane, a rozpoczęte przez niego nagrywanie z czujnika światła zostaje zamknięte. Jeśli serwer uruchomił backend, przed jego zatrzymaniem zwalnia również podłączone urządzenia; w backendzie aplikacji pozostają one podłączone.
- **Protokół.** Wyłącznie stdio (bez punktu końcowego HTTP), wyłącznie narzędzia (bez zasobów ani monitów), wyniki tekstowe i graficzne, wersje protokołu MCP od 2024-11-05 do 2025-11-25.**Czego brakuje.** Logowanie się lub wylogowywanie (konto należy do użytkownika; serwer nie może się zalogować) oraz zatrzymywanie backendu, którego serwer nie uruchomił. Wszystkie pozostałe operacje związane z przetwarzaniem i sprzętem, które wykonuje ten SDK, mają swoje narzędzia. (Konfiguracja urządzenia DAQ-E — jego nazwa, token uwierzytelniający, hasło OTA, wymóg podpisu, przywrócenie ustawień fabrycznych — oraz bezpośrednie serwery DAQ należą wyłącznie do `chloros-cli` i są również dostępne w SDK: błędna nazwa lub token blokują dostęp każdej instalacji Chloros do urządzenia.)

---

## Automatyczne uruchamianie modułów zaplecza

Punkty wejścia smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` oraz `discover_daq_sensors` — to lekkie klienty HTTP, które zakładają, że backend nasłuchuje na adresie `127.0.0.1:5000` (domyślny adres URL interfejsu smart-connect). Jeśli interfejs graficzny (GUI) lub interfejs wiersza poleceń (CLI) już działa, to serwer ten jest uruchomiony. W przypadku uruchomienia z samego skryptu może go nie być — dlatego funkcje te **automatycznie uruchamiają dołączony plik binarny serwera** (w trybie bezokiennym, tak samo jak robi to `ChlorosLocal`) przed pierwszym wywołaniem, a następnie czekają do 60 s na jego uruchomienie. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` oraz `analyze_array_network()` nigdy go nie uruchamiają.

Zasady:

- **Uruchamiany jest wyłącznie lokalny adres URL.** Kwalifikuje się proces `backend_url` wskazujący na `localhost` / `127.0.0.1` / `[::1]` jest dopuszczalny; każdy inny host jest traktowany jako komputer należący do kogoś innego i nigdy nie jest uruchamiany.
- **Jeśli backend zakończy działanie przed udzieleniem odpowiedzi, oczekiwanie zostaje przerwane.** Wywołanie zwraca natychmiast kod wyjścia, polecenie, które zostało wykonane, oraz ostatnie linijki wyświetlone przez backend, zamiast czekać przez 60 s. Wyjątkiem jest kod wyjścia 98: inny backend już zajmuje port lub plik blokady (aplikacji Chloros, usługi backendowej lub innego skryptu, który wciąż się uruchamia), więc wywołanie nadal czeka na odpowiedź tego backendu. `ChlorosLocal` zachowuje się tak samo.
- **Backend pozostaje uruchomiony w celu ponownego wykorzystania** (tak samo jak w CLI) — po zakończeniu działania skryptu nie następuje domyślne wyłączenie. Ponowne uruchomienie skryptu powoduje ponowne wykorzystanie aktywnego backendu.
- **Można zrezygnować z tej opcji za pomocą `auto_start_backend=False`** w przypadku dowolnego z tych wywołań (np. gdy wskazano zdalny backend lub samodzielnie zarządza się cyklem życia backendu).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Jeśli nie można zlokalizować lub uruchomić dołączonego pliku binarnego, kolejne wywołanie HTTP generuje nadający się do działania, **zależny od platformy** błąd `ChlorosConnectError` zamiast zwykłego komunikatu o odmowie połączenia — w systemie Windows wskazuje ona aplikację desktopową lub polecenie `chloros-cli`; w systemie Linux (bez GUI) wskazuje ona polecenie `chloros-cli` lub `.deb`.

---

## Środowisko i nagłówki

SDK oznacza każde wywołanie HTTP w backendzie kodem `X-Chloros-Client: sdk`. Backend stosuje zasady licencjonowania SDK/CLI (wymagane jest zalogowanie się **oraz** płatny plan Chloros+), a nie ścieżkę bezpłatnego poziomu dostępu w GUI. Jest to ustawiane automatycznie w momencie importu — nie musisz nic robić.

Nagłówek jest dodawany do każdego wywołania `requests` / `urllib`, którego adres URL zawiera `localhost` lub `127.0.0.1`, na dowolnym porcie — w tym na własnych usługach lokalnych — i zastępuje wartość ustawioną przez użytkownika. Wywołania do innych hostów pozostają niezmienione.

Aby zastąpić URL zaplecza, należy przekazać `backend_url=` (`ChlorosLocal` przyjmuje `backend_url=` lub `api_url=`; `process_folder()` i `process_lattice_capture()` akceptują wyłącznie `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Zainstalowany backend nasłuchuje wyłącznie na pętli zwrotnej, więc `backend_url` z sieci LAN nie może się z nim połączyć; zobacz [Tryb zdalnego backendu](#remote-backend-mode-pip-only-host-via-tunnel) w celu zapoznania się ze schematem tunelowania.)

---

## Wersjonowanie i kompatybilność

- Wersja SDK’a jest udostępniana jako `chloros_sdk.__version__`.
- SDK wiąże zachowanie pinów z wersją dołączonego backendu. Łączenie starszego SDK z nowszym backendem zazwyczaj działa (punkty końcowe zgodne z przyszłymi wersjami), ale połączenie nowszego SDK ze starszym backendem może powodować błędy `404` w nowych punktach końcowych — należy zaktualizować aplikację desktopową, aby była zgodna.
- Interfejs smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) oraz punkt końcowy analizy sieci zwracają stabilne schematy JSON; nowe pola są dodawane.

---

## Wskazówki dotyczące rozwiązywania problemów

- **`ChlorosAuthenticationError: Login required`** (z `ChlorosLocal`; wywołania funkcji smart-connect generują błąd `ChlorosConnectError: Login required. …` z tej samej przyczyny) → Uruchom raz polecenie `chloros-cli login EMAIL PASSWORD` na tym komputerze lub zaloguj się za pomocą aplikacji Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (lub `Failed to create project: …` wynikający z `create_project()`) → Na tym komputerze wyłączono obsługę długich ścieżek w systemie Windows (ustawienie domyślne systemu Windows), więc żadna ścieżka nie może przekraczać 260 znaków, a produkty znajdują się maksymalnie około 130 znaków poniżej folderu projektu. Nic nie zostało przetworzone. Komunikat podaje najdłuższy folder projektu, który się mieści: użyj krótszego lub włącz długie ścieżki w systemie Windows i uruchom ponownie Chloros. Instalator Windowsa oferuje tę opcję jako „Włącz długie ścieżki Windowsa”, zaznaczoną domyślnie; w przeciwnym razie należy skorzystać z zasad grupy: *Konfiguracja komputera &gt; Szablony administracyjne &gt; System &gt; System plików &gt; Włącz długie ścieżki Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkcja Smart-Connect wywołuje automatyczne uruchomienie lokalnego backendu, więc ten komunikat pojawia się tylko wtedy, gdy nie można znaleźć/uruchomić dołączonego pliku binarnego (np. host działający wyłącznie w trybie pipbez pakietu pulpitu). Komunikat jest zależny od platformy: w systemie Windows otwórz aplikację pulpitu lub uruchom dowolne polecenie `chloros-cli`; w systemie Linux uruchom polecenie `chloros-cli` (nie ma interfejsu graficznego) lub zainstaluj `.deb`. W przypadku zdalnego backendu przekaż `backend_url=` (oraz `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** podczas importu → Nie udało się załadować `lattice_sdk`: środowisko uruchomieniowe Arena SDK lub biblioteka OpenCV (`cv2`) nie można zaimportować w tej wersji Pythona. Powierzchnia sterowana przez backend (`connect_*`, wyświetlana przez backend, `ChlorosLocal`) nadal działa.
- **Tablica łączy się przy pełnej rozdzielczości czujnika, ale przy niskiej częstotliwości klatek**→ Zgodnie z założeniami: tryb automatyczny zachowuje pełną rozdzielczość czujnika i pozwala, by częstotliwość spadła do wartości, jaką pozwala na to łącze. Przekaż `binning=2` / `4` lub mniejsze `width` / `height`, aby uzyskać większą liczbę klatek na sekundę, albo włącz ramki jumbo / zmodernizuj łącze. Binning i ROI**nie** rozwiązują problemu zagregowanego nadmiernego obciążenia (`oversubscribed: true`, pola fps 0): zbyt dużej liczby kamer dla danego łącza nie da się rozwiązać za pomocą binningu/ROI — należy zmniejszyć liczbę kamer, włączyć ramki typu jumbo lub przejść na szybszą kartę sieciową (zobacz [Nadmierna subskrypcja](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` zgłasza, że pierścień odbiorczy karty sieciowej jest bardzo mały (~0,26 MB) / bramki połączeń wyświetlają komunikat „FRAMES WILL DROP”** → Pierścień odbiorczy karty sieciowej hosta ma wartość domyślną (często resetowaną do 32 po aktualizacji sterownika karty). W przypadku karty Realtek USB 10GbE należy ustawić `ReceiveBufferLen=256` i `PendingReceives=64` (z podwyższonymi uprawnieniami), a następnie ponownie uruchomić backend, aby ponownie odczytał pierścień. Pełna procedura: [Podręcznik CLI → Konfiguracja i optymalizacja karty sieciowej hosta](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host zawiesza się podczas restartu/wyłączania, później pojawiają się błędy WMI `Invalid class` / karta sieciowa nie włącza się** → Przestarzały sterownik USB 10GbE powodujący błąd `DRIVER_POWER_STATE_FAILURE` (niebieski ekran śmierci `0x9F`). Zaktualizuj sterownik karty sieciowej do najnowszej wersji (≥ 2026) i ponownie zastosuj ustawienia pierścienia odbiorczego. Zobacz [Podręcznik CLI → Konfiguracja i optymalizacja karty sieciowej hosta](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Przechwytywanie macierzy kończy się niepowodzeniem z kodem HTTP 503 „Brak nowej zsynchronizowanej grupy ramek”** → Macierz nie opublikowała żadnej nowej grupy ramek w trakcie oczekiwania. Sprawdź `arr.status()`: `stalled: True` (lub `streaming: False`), przy czym wartość `actual_fps` równa 0 oznacza, że pętla przechwytywania przestała publikować — należy odłączyć i ponownie podłączyć macierz. Jeśli `stalled` ma wartość False, a `actual_fps` jest większe od 0, należy spróbować ponownie. Zaraz po `connect_array()` przechwytywanie czeka już do 20 s na pierwszą grupę.
- **`status()` pokazuje `link_speed_mbps: 4294`** → System Windows nie mógł odczytać prędkości karty sieciowej; jest to typowe dla kart sieciowych USB i nie stanowi usterki. Zobacz [Oczekiwane częstotliwości ramek](#expected-frame-rates).
- **`update_firmware()` kończy się niepowodzeniem z kodem `device never connected back on TCP <port>`** → Zapora sieciowa zablokowała połączenie urządzenia DAQ-E z tym komputerem. Instalator systemu Windows dodaje odpowiednią regułę (zobacz [Ustawienia urządzenia DAQ-E](#daq-e-device-settings)); w przypadku innych zapór sieciowych należy zezwolić na przychodzący ruch TCP do serwera zaplecza z podsieci czujnika.
- **Odmowa pomiaru współczynnika odbicia** → Aby uzyskać współczynnik odbicia w skali bezwzględnej, do kamery (lub matrycy) musi być przypisany aktywny moduł DAQ. Przypisz go za pomocą interfejsu graficznego lub użyj `processing="radiance"` (W/m²/sr/nm), który nie wymaga sparowanego czujnika.
- **Rejestracja za pomocą `smart=True` trwa dłużej niż oczekiwano** → Zbieżność AE zależy od dynamiki sceny. W przypadku `ArrayHandle.capture_smart()` należy poszerzyć `exposure_tolerance_pct`, skróć `stability_window_s` lub zmniejsz wartość `settle_timeout_s`, aby uzyskać szybsze, mniej ustabilizowane wyzwalanie.

---

## Zobacz też

- [Podręcznik CLI](cli-reference.md) — każde polecenie podrzędne CLI odpowiada wywołaniu SDK.
- [Czujniki światła DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) w instrukcji obsługi Chloros — czujniki, ich ograniczenia i skalibrowany zakres, rejestrowanie, sieć DAQ-E.
- [Kamery LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) w instrukcji obsługi Chloros — podłączanie, ustawienia kamer, tryby przechwytywania, macierze.
- Dokumentacja online: `https://mapir.gitbook.io/chloros/api-python-sdk`
