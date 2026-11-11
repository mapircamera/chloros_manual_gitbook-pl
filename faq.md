---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Często zadawane pytania

<details>

<summary>Czy za pomocą programu Chloros mogę przetwarzać zdjęcia z aparatów innych niż marki MAPIR?</summary>

Nie, Chloros obsługuje wyłącznie przetwarzanie obrazów z kamer MAPIR — z serii Survey3 i LATTICE. Więcej informacji można znaleźć na liście [obsługiwanych modeli kamer](supported-cameras.md). Oferujemy przetwarzanie obrazów z innych kamer w chmurze MAPIR; pełną listę można znaleźć [tutaj](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Czy program Chloros obsługuje kamery LATTICE?</summary>

Tak. Chloros 1.3.0 obsługuje moduły kamer LATTICE M3C i M3M w pełnym zakresie: **sterowanie na żywo**— wykrywanie, łączenie, podgląd i przechwytywanie obrazu za pomocą zakładki „Kamery” w interfejsie graficznym (`chloros-cli lattice`) lub biblioteki SDK SDK, w tym zsynchronizowane układy wielokamerowe z synchronizacją czasu PTP — oraz**pełne przetwarzanie radiometryczne** przechwyconych danych (surowe → po usunięciu efektu bayera → promieniowanie → odbicie → indeks). Zobacz [Obsługiwane kamery](supported-cameras.md) oraz [przewodnik LATTICE](lattice/README.md).

</details>

<details>

<summary>Czy mogę skalibrować moje obrazy pod kątem odbicia bez użycia tarczy kalibracyjnej?</summary>

**Survey3:** Nie. Bez zdjęcia tarczy kalibracyjnej zarejestrowanego w tym samym czasie, co zdjęcia bez tarczy, nie będzie można powiązać wartości pikseli obrazu ze znaną wartością procentową odbicia. Jeśli nie dołączysz również zapisu z czujnika światła MAPIR, spektrum światła otoczenia nie zostanie zmierzone, a wyniki odbicia nie będą dokładne.

**LATTICE:** Tak. Odbicie można odnieść do natężenia promieniowania padającego mierzonego przez czujnik światła DAQ zamiast panelu (ρ = π·L/E). Gdy w kadrze *znajduje się* cel, który przeszedł kontrolę jakości (QA), domyślnie staje się on punktem odniesienia absolutnego (`--reflectance-source auto`), chyba że przeszedł z flagą, w którym to przypadku cel i łańcuchy DAQ rywalizują o wynik pomiaru. Jedyny wyjątek: „Odbicie F988 jest kalibrowane przy użyciu panelu odbicia znajdującego się w kadrze: pasmo wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros stosuje najnowszy zapis z panelu i utrzymuje go między kolejnymi pomiarami panelu”. Zobacz [Cele kalibracyjne](calibration-targets.md).

</details>

<details>

<summary>Czy potrzebuję czujnika światła DAQ?</summary>

Nie w przypadku promieniowania: eksportowane dane promieniowania w formacie LATTICE pochodzą z fabrycznej kalibracji radiometrycznej każdej kamery i nie wymagają ani czujnika DAQ, ani celu. W przypadku **odbicia**potrzebny jest punkt odniesienia dla światła otoczenia — albo pomiar promieniowania padającego wykonany przez czujnik światła DAQ, albo cel kalibracyjny znajdujący się w kadrze. Czujnik DAQ pozwala uzyskać skalibrowane wartości odbicia**bez umieszczania jakichkolwiek paneli w kadrze**. Zarejestrowane pliki `.daq` są automatycznie dopasowywane do obrazów na podstawie znacznika czasu. Zobacz [Cele kalibracyjne](calibration-targets.md) oraz [Podręcznik CLI](reference/cli-reference.md).

</details>

<details>

<summary>Czy mogę korzystać z Chlorosa z asystentem AI (Claude, ChatGPT itp.)?</summary>

Tak — niniejsza instrukcja oraz CLI/SDK zostały stworzone właśnie z myślą o tym:

* Pełny spis treści instrukcji jest udostępniony pod adresem `https://mapir.gitbook.io/chloros/llms.txt`, dzięki czemu asystenci AI mogą znaleźć każdą stronę.
* Surowy kod Markdown każdej strony jest dostępny pod jej URL-em w małych literach z dodanym `.md` (na przykład `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Podręcznik CLI](reference/cli-reference.md) oraz [Podręcznik SDK](reference/sdk-reference.md) zostały napisane z myślą o modelach LLM: zawierają dokładne flagi, wartości domyślne, semantykę zakończenia oraz polecenia, które można skopiować i wkleić.

Zobacz [Asystenci AI](ai-assistants.md), aby dowiedzieć się, jak skonfigurować swojego asystenta do korzystania z Chloros.

</details>

<details>

<summary>Gdzie trafiają moje przetworzone pliki wyjściowe?</summary>

Pliki wyjściowe są zapisywane w folderze projektu, pogrupowane według kamery, a następnie według formatu pliku:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **folder-kamery** — `LATT-<sensor>-<lens>-F<filter>` dla LATTICE, `<model>_<filter>` (np. `Survey3N_RGN`) dla Survey3
* **folder-formatu** — `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32`
* **foldery produktów** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (zawsze w folderze `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — wyłącznie dane wielospektralne LATTICE: maski na poziomie pikseli znajdują się obok folderów produktów, patrz poniżej**Eksportowane pliki zachowują nazwę pliku źródłowego — to folder identyfikuje produkt, a nie rozszerzenie nazwy pliku.**W przypadku korzystania z interfejsu CLI folder projektu jest tworzony w domyślnym folderze projektów (`Documents/MAPIR/Chloros Projects`, o ile nie został zmieniony), chyba że podano parametr `-o`; względna ścieżka `-o` jest interpretowana względem folderu, z którego uruchamiane jest polecenie. Należy pamiętać, że uruchomienie `chloros-cli process`, które zażądało produktów, ale żadnego nie zapisało, wyświetla `Processing finished but wrote no image products.` i**kończy się wynikiem niezerowym**, dzięki czemu skrypty mogą to wykryć. Zobacz [Formaty obrazów wyjściowych](output-image-formats.md) oraz [Podręcznik CLI](reference/cli-reference.md).

</details>

<details>

<summary>Czym są foldery „clamp_masks” i pliki *_clampmask.tif?</summary>

Nowość w wersji 1.3.0. Produkty promieniowania i odbicia z wielospektralnej kamery LATTICE M3C przechodzą proces rozdzielania pasm 3×3, a tam, gdzie w danym paśmie powstaje wartość ujemna, program Chloros ogranicza tę wartość do zera. Produkt zawierający co najmniej jeden przycięty piksel otrzymuje jednopasmową maskę `uint8`, która wskazuje, które piksele i pasma zostały przycięte (bit 0 = kanał R produktu, bit 1 = G, bit 2 = B). Maski są zapisywane w folderze `clamp_masks/` **obok** folderów produktów — na przykład `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — a tag XMP produktu `Chloros:UnmixClampMask` określa nazwę jego maski (lub zawiera informację `none`).

Nie są to obrazy sceny: własna funkcja importu programu Chloros pomija je, a ponieważ nigdy nie znajdują się one w folderze produktu, można przekazać folder produktu do Pix4D lub Metashape w obecnej postaci. Zobacz [Maski zaciskowe](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Dlaczego program Chloros wyświetla komunikat „Folder projektu jest zbyt głęboki dla tego komputera”?</summary>

Dotyczy to wyłącznie systemu Windows. W systemie Windows domyślnie wyłączona jest obsługa długich ścieżek, a długość ścieżki do pliku nie może przekraczać 260 znaków. Produkty Chloros znajdują się maksymalnie około 130 znaków poniżej folderu projektu, więc Chloros odrzuca folder projektu znajdujący się głębiej przed rozpoczęciem przetwarzania — zamiast uruchamiać się i nie zapisywać niczego, jak mogło się zdarzyć przed wersją 1.3.0. Komunikat podaje najdłuższy folder projektu, który się mieści.

Można to naprawić na dwa sposoby:

* Użyj krótszego folderu projektu (opcja **Katalog roboczy** w Ustawieniach projektu lub `-o` / `chloros-cli set-project-folder` w przypadku interfejsu CLI).
* Włącz obsługę długich ścieżek w systemie Windows i uruchom ponownie program Chloros. Opcja **Włącz długie ścieżki w systemie Windows** w instalatorze Chloros (domyślnie zaznaczona) zapewnia to; uruchom ponownie instalator, jeśli ją odznaczyłeś.

Zobacz [Rozpoczynanie przetwarzania](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Czy mogę edytować moje obrazy przed przetworzeniem w programie Chloros?</summary>

Nie. Program Chloros zakłada, że dane wejściowe nie zostały zmodyfikowane. Nie należy zmieniać nazw plików.

</details>

<details>

<summary>Czy mogę ustawić moje kamery MAPIR Survey3 na automatyczną ekspozycję i przetwarzać zdjęcia w programie Chloros?</summary>

Nie. Zbiory danych obrazów z kamer Survey3 muszą mieć stałą/zablokowaną ekspozycję, więc nie można stosować automatycznego czasu naświetlania ani automatycznej czułości ISO. Wszystkie obrazy z tego samego modelu kamery muszą mieć identyczny czas naświetlania i czułość ISO (ekspozycję).

Kamery LATTICE nie mają tego ograniczenia: program Chloros steruje ich ekspozycją w czasie rzeczywistym (Smart AE), a każde ujęcie rejestruje faktycznie zastosowaną ekspozycję i wzmocnienie, co uwzględnia proces radiometryczny.

</details>

<details>

<summary>Czy program Chloros może przetwarzać lub analizować obrazy ortomosaiczne?</summary>

Nie. Obsługiwane są wyłącznie pojedyncze zdjęcia z aparatu MAPIR, a nie zdjęcia połączone w jedną całość, takie jak mapa ortomosaiczna.

</details>

<details>

<summary>Jak mogę przyspieszyć etap wykrywania celów w programie Chloros?</summary>

W tabeli przeglądarki plików wstępne zaznaczenie obrazów docelowych w prawej kolumnie spowoduje, że Chloros będzie szukał celów kalibracyjnych wyłącznie w tych obrazach, co znacznie przyspieszy przetwarzanie.

</details>

<details>

<summary>Jeśli zamierzam przesłać moje zdjęcia do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> czy powinienem przetworzyć je w Chlorosie przed przesłaniem?</summary>

Jeśli planujesz przesłać zdjęcia na naszą platformę przetwarzania online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nie edytuj ich przed przesłaniem. Chmura wykona wszystkie te same operacje przetwarzania i wiele więcej.

</details>

<details>

<summary>Czy MAPIR kiedykolwiek będzie obsługiwać funkcję X? Bardzo chciałbym, aby MAPIR oferował funkcję X.</summary>

Zawsze chętnie przyjmujemy opinie na temat naszych produktów. Jeśli zauważysz problem z naszymi produktami lub masz sugestię, jak możemy je ulepszyć, skontaktuj się z nami [SKONTAKTUJ SIĘ Z NAMI](https://www.mapir.camera/community/contact), aby podzielić się swoimi przemyśleniami. Większość naszych działań badawczo-rozwojowych opiera się na wsłuchiwaniu się w najważniejsze potrzeby naszych klientów.

</details>

<details>

<summary>Czy Chloros jest dostępny dla systemu Linux?</summary>

Tak! Chloros 1.3.0 obsługuje system Linux w architekturze amd64 (x86_64) oraz arm64 (NVIDIA Jetson JetPack 6) za pośrednictwem pakietów `.deb`. Interfejs wiersza poleceń (CLI) oraz zestaw SDK SDK są w pełni obsługiwane w systemie Linux, w tym sterowanie kamerą LATTICE na żywo oraz czujnikami DAQ. Nie ma graficznego interfejsu użytkownika (GUI) dla systemu Linux — cała interakcja odbywa się za pośrednictwem [CLI](CLI.md) lub [Python SDK](api-python-sdk.md). Szczegółowe informacje można znaleźć w [Przeglądzie systemu Linux](linux/linux-overview.md).

</details>

<details>

<summary>Czy mogę uruchomić Chloros na platformie NVIDIA Jetson?</summary>

Tak! Chloros obsługuje **rodzinę** NVIDIA Jetson Orin — Orin Nano, Orin NX i AGX Orin — z zainstalowanym JetPack 6. Starsze modele (Jetson Nano, TX2, Xavier NX) nie obsługują JetPack 6 i nie są obsługiwane przez obecny pakiet. Chloros automatycznie wykrywa model Jetson i optymalizuje strategię przetwarzania. Pakiet Jetson wykorzystuje wersję PyTorch dla CUDA z JetPack 6, więc przetwarzanie odbywa się na procesorze graficznym (GPU) urządzenia Jetson. Instrukcje dotyczące konfiguracji i wdrożenia znajdziesz w [Przewodniku po NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Czy Chloros automatycznie optymalizuje działanie pod kątem mojego sprzętu?</summary>

Tak! Chloros zawiera funkcję [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), która automatycznie wykrywa procesor (CPU), procesor graficzny (GPU), pamięć RAM oraz (w przypadku Jetson) czujniki temperatury. Następnie wybiera optymalną strategię przetwarzania — od `GPU_PARALLEL` na procesorach graficznych o dużej pamięci, przez `GPU_SINGLE` na mniejszych, aż po `CPU_PARALLEL` w systemach bez procesora graficznego obsługującego CUDA, z którego może korzystać PyTorch. Nie jest wymagana żadna ręczna konfiguracja.

</details>

<details>

<summary>Czym jest 4-wątkowy potok przetwarzania?</summary>

Chloros wykorzystuje architekturę potokową z 4 wątkami dla użytkowników Chloros+: Wątek 1 (Wykrywanie) ładuje obrazy i wykrywa cele kalibracji, Wątek 2 (Kalibracja) oblicza kalibrację współczynnika odbicia, Wątek 3 (Przetwarzanie) wykonuje przyspieszane przez procesor graficzny usuwanie efektu bayera oraz obliczanie indeksu, a Wątek 4 (Eksport) zapisuje pliki wyjściowe. W celu uzyskania maksymalnej przepustowości wiele obrazów może znajdować się jednocześnie w różnych wątkach. Szczegółowe informacje można znaleźć w sekcji [Potok przetwarzania](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Jak przeprowadzić diagnostykę instalacji programu Chloros?</summary>

Użyj polecenia `selftest`, aby przeprowadzić 7-etapowy test sprawdzający: wersja, dostępność portów, uruchomienie backendu, łączność z API (`/api/test`), informacje o systemie (`/api/system-info` — GPU/CUDA/PyTorch), obecność modelu odszumiającego oraz gotowość CUDA i odszumiającego:

```bash
chloros-cli selftest
```

Jest to szczególnie przydatne w systemach Linux do weryfikacji konfiguracji GPU i CUDA. Na urządzeniu Jetson kod `CUDA: False` zazwyczaj oznacza brak bibliotek CUDA i cuDNN w JetPack 6: zainstaluj je za pomocą polecenia `sudo apt install nvidia-jetpack`.

</details>
