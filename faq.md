---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Często zadawane pytania

<details>

<summary>Czy za pomocą urządzenia Chloros mogę przetwarzać obrazy z kamer innych marek niż MAPIR?</summary>

Nie, Chloros obsługuje wyłącznie przetwarzanie obrazów z kamer MAPIR — z serii Survey3 i LATTICE. Więcej informacji znajdziesz na liście [obsługiwanych modeli kamer](supported-cameras.md). Oferujemy przetwarzanie obrazów z innych kamer w chmurze MAPIR — pełną listę można znaleźć [tutaj](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Czy Chloros obsługuje kamery LATTICE?</summary>

Tak. Chloros 1.2.0 obsługuje moduły kamer LATTICE M3C i M3M w pełnym zakresie: **sterowanie na żywo**— wykrywanie, łączenie, podgląd i rejestrowanie obrazu z poziomu zakładki „Kamery” w interfejsie graficznym, `chloros-cli lattice` lub Python SDK, w tym zsynchronizowane układy wielokamerowe z synchronizacją czasu PTP — oraz**pełne przetwarzanie radiometryczne** zarejestrowanych obrazów (surowe → bez efektu bayera → promieniowanie → odbicie → wskaźnik). Zobacz [Obsługiwane kamery](supported-cameras.md) oraz [przewodnik LATTICE](lattice/README.md).

</details>

<details>

<summary>Czy mogę skalibrować moje obrazy pod kątem współczynnika odbicia bez użycia tarczy kalibracyjnej?</summary>

**Survey3:** Nie. Bez zdjęcia wzorca kalibracyjnego zarejestrowanego w tym samym czasie, co zdjęcia bez wzorca, nie będzie można powiązać wartości pikseli obrazu ze znaną wartością procentową odbicia. Jeśli nie dołączysz również zapisu z czujnika światła MAPIR, spektrum światła otoczenia nie zostanie zmierzone, a wyniki odbicia nie będą dokładne.**LATTICE:** Tak. Odbicie można odnieść do natężenia promieniowania padającego zmierzonego przez czujnik światła DAQ zamiast panelu (ρ = π·L/E). Gdy w kadrze *znajduje się* cel, który przeszedł kontrolę jakości (QA), domyślnie staje się on punktem odniesienia absolutnego (`--reflectance-source auto`). Jedyny wyjątek: „F988 – współczynnik odbicia jest kalibrowany przy użyciu panelu odbicia znajdującego się w kadrze: pasmo wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros stosuje ostatni zapis z panelu i utrzymuje go między kolejnymi pomiarami panelu”. Zobacz [Cele kalibracyjne](calibration-targets.md).

</details>

<details>

<summary>Czy potrzebuję czujnika światła DAQ?</summary>

Nie w przypadku promieniowania: eksportowane wartości promieniowania w LATTICE pochodzą z fabrycznej kalibracji radiometrycznej każdej kamery i nie wymagają ani czujnika DAQ, ani tarczy kalibracyjnej. W przypadku **odbicia światła**potrzebne jest odniesienie dla światła otoczenia — albo pomiar promieniowania padającego wykonany przez czujnik światła DAQ, albo tarcza kalibracyjna umieszczona w kadrze. Czujnik DAQ pozwala uzyskać skalibrowane wartości odbicia**bez umieszczania jakichkolwiek paneli w kadrze**. Zarejestrowane pliki `.daq` są automatycznie dopasowywane do zdjęć na podstawie sygnatury czasowej. Zobacz [Cele kalibracyjne](calibration-targets.md) oraz [Podręcznik CLI](reference/cli-reference.md).

</details>

<details>

<summary>Czy mogę używać Chloros z asystentem AI (Claude, ChatGPT itp.)?</summary>

Tak — niniejsza instrukcja oraz pliki CLI/SDK zostały stworzone właśnie z myślą o tym:

* Pełny indeks instrukcji jest udostępniony pod adresem `https://mapir.gitbook.io/chloros/llms.txt`, aby asystenci AI mogli odnaleźć każdą stronę.
* Surowy kod Markdown każdej strony jest dostępny pod adresem odpowiadającym nazwie strony pisanej małymi literami URL z dodanym `.md` (na przykład `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Dokumentacja CLI](reference/cli-reference.md) oraz [SDK](reference/sdk-reference.md) zostały opracowane z myślą o modelach LLM: zawierają dokładne flagi, wartości domyślne, semantykę zakończenia oraz polecenia, które można skopiować i wkleić.

Zobacz [Asystenci AI](ai-assistants.md), aby dowiedzieć się, jak skonfigurować asystenta do korzystania z Chloros.

</details>

<details>

<summary>Gdzie trafiają moje przetworzone pliki wyjściowe?</summary>

Pliki wyjściowe są zapisywane w folderze projektu, pogrupowane według aparatu, a następnie według formatu pliku:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **folder-aparatu** — `LATT-<sensor>-<lens>-F<filter>` dla LATTICE, `<model>_<filter>` (np. `Survey3N_RGN`) dla Survey3
* **folder-formatu** — `tiff16`, `tiff8`, `png8`, `jpg8` lub `tiff32`
* **foldery produktów** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (zawsze w folderze `tiff32`), `<INDEX>_Index_Images/`**Eksportowane pliki zachowują nazwę pliku źródłowego — to folder identyfikuje produkt, a nie rozszerzenie nazwy pliku.**W przypadku CLI folder projektu jest tworzony obok folderu wejściowego, chyba że podasz `-o`. Należy pamiętać, że uruchomienie `chloros-cli process`, które zażądało produktów, ale żadnego nie zapisało, wyświetla `Processing finished but wrote no image products.` i**kończy się wynikiem niezerowym**, dzięki czemu skrypty mogą to wykryć. Zobacz [Formaty obrazów wyjściowych](output-image-formats.md) oraz [Podręcznik CLI](reference/cli-reference.md).

</details>

<details>

<summary>Czy mogę edytować moje obrazy przed przetworzeniem w Chloros?</summary>

Nie. Chloros zakłada, że dane wejściowe nie zostały zmodyfikowane. Nie należy zmieniać nazw plików.

</details>

<details>

<summary>Czy mogę ustawić moje aparaty MAPIR i Survey3 na automatyczną ekspozycję i przetwarzać zdjęcia w programie Chloros?</summary>

Nie. Zbiory danych obrazów Survey3 muszą mieć stałą/zablokowaną ekspozycję, więc nie można stosować automatycznego czasu otwarcia migawki ani automatycznego ISO. Wszystkie obrazy z tego samego modelu kamery muszą mieć identyczny czas otwarcia migawki i ISO (ekspozycję).

Kamery LATTICE nie mają tego ograniczenia: Chloros steruje ich ekspozycją na żywo (Smart AE), a każde ujęcie rejestruje faktycznie zastosowaną ekspozycję i wzmocnienie, co uwzględnia radiometryczny proces przetwarzania.

</details>

<details>

<summary>Czy Chloros może przetwarzać lub analizować obrazy ortomosaiczne?</summary>

Nie. Obsługiwane są wyłącznie pojedyncze zdjęcia z aparatu MAPIR, a nie zdjęcia połączone, takie jak mapa ortomosaiczna.

</details>

<details>

<summary>Jak mogę przyspieszyć etap wykrywania celów w programie Chloros?</summary>

W tabeli przeglądarki plików wstępny wybór obrazów docelowych w prawej kolumnie spowoduje, że Chloros będzie szukać celów kalibracyjnych wyłącznie w tych obrazach, co znacznie przyspieszy przetwarzanie.

</details>

<details>

<summary>Jeśli zamierzam przesłać moje zdjęcia do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">chmury MAPIR Cloud,</a> czy powinienem je przetworzyć w programie Chloros przed przesłaniem?</summary>

Jeśli planujesz przesłać zdjęcia na naszą platformę przetwarzania online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nie edytuj ich przed przesłaniem. Cloud wykona wszystkie te same operacje przetwarzania i wiele więcej.

</details>

<details>

<summary>Czy MAPIR kiedykolwiek będzie obsługiwać funkcję X? Bardzo chciałbym, aby MAPIR oferowało funkcję X.</summary>

Zawsze chętnie przyjmujemy opinie na temat naszych produktów. Jeśli zauważysz problem z naszymi produktami lub masz sugestię, jak możemy je ulepszyć, skontaktuj się z nami [SKONTAKTUJ SIĘ Z NAMI](https://www.mapir.camera/community/contact), aby podzielić się swoimi przemyśleniami. Większość naszych działań badawczo-rozwojowych opiera się na wsłuchiwaniu się w najważniejsze potrzeby naszych klientów.

</details>

<details>

<summary>Czy Chloros jest dostępne dla Linux?</summary>

Tak! Chloros 1.2.0 obsługuje Linux amd64 (x86_64) oraz arm64 (NVIDIA Jetson JetPack 6) za pośrednictwem pakietów `.deb`. Modele CLI oraz Python i SDK są w pełni obsługiwane na Linux, w tym sterowanie na żywo kamerą LATTICE i czujnikiem DAQ. Nie ma interfejsu graficznego dla Linux — cała interakcja odbywa się za pośrednictwem [CLI](CLI.md) lub [Python SDK](api-python-sdk.md). Szczegółowe informacje można znaleźć w [Linux Przegląd](linux/linux-overview.md).

</details>

<details>

<summary>Czy mogę uruchomić Chloros na platformie NVIDIA Jetson?</summary>

Tak! Chloros obsługuje platformy NVIDIA Jetson, w tym Jetson Nano, Orin Nano, Orin NX i AGX Orin z systemem JetPack 6. Chloros automatycznie wykrywa model urządzenia Jetson i optymalizuje strategię przetwarzania. Zobacz [Przewodnik po platformie NVIDIA Jetson](linux/nvidia-jetson-guide.md), aby uzyskać instrukcje dotyczące konfiguracji i wdrożenia.

</details>

<details>

<summary>Czy Chloros automatycznie dostosowuje się do mojego sprzętu?</summary>

Tak! Chloros zawiera funkcję [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), która automatycznie wykrywa procesor (CPU), procesor graficzny (GPU), pamięć RAM oraz (w przypadku Jetson) czujniki temperatury. Następnie wybiera optymalną strategię przetwarzania — od `GPU_PARALLEL` na systemach z dużą pamięcią, przez `GPU_SINGLE` na urządzeniach o ograniczonych zasobach, aż po `CPU_PARALLEL` na systemach bez procesora graficznego NVIDIA. Nie jest wymagana żadna ręczna konfiguracja.

</details>

<details>

<summary>Czym jest 4-wątkowy potok przetwarzania?</summary>

Chloros wykorzystuje architekturę potokową z 4 wątkami dla użytkowników Chloros+: Wątek 1 (Wykrywanie) ładuje obrazy i wykrywa cele kalibracyjne, wątek 2 (Kalibracja) oblicza kalibrację współczynnika odbicia, wątek 3 (Przetwarzanie) wykonuje przyspieszane przez procesor graficzny oddzielanie pikseli i obliczanie indeksu, a wątek 4 (Eksport) zapisuje pliki wyjściowe. W celu uzyskania maksymalnej przepustowości wiele obrazów może znajdować się jednocześnie w różnych wątkach. Szczegółowe informacje można znaleźć w sekcji [Potok przetwarzania](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Jak przeprowadzić diagnostykę mojej instalacji Chloros?</summary>

Użyj polecenia `selftest`, aby przeprowadzić 7-etapowy test sprawdzający: wersja, dostępność portów, uruchomienie backendu, łączność API (`/api/test`), informacje o systemie (`/api/system-info` — GPU/CUDA/PyTorch), obecność modelu odszumiającego oraz gotowość CUDA i odszumiającego:

```bash
chloros-cli selftest
```

Jest to szczególnie przydatne w systemach Linux/Jetson do weryfikacji konfiguracji GPU i CUDA.

</details>
