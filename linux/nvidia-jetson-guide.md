# Przewodnik po NVIDIA Jetson —

Chloros
na platformie NVIDIA Jetson umożliwia przetwarzanie obrazów wielospektralnych na urządzeniach brzegowych — w terenie, na bezzałogowych statkach powietrznych (UAV) oraz w odległych instalacjach. Chloros
1.3.0 rozpoznaje model Jetson podczas uruchamiania i optymalizuje strategię przetwarzania pod kątem wykrytego sprzętu. **Nie jest wymagane ręczne dostosowywanie.**
{% hint style="info" %}
**Pakiet Jetson 1.3.0 przetwarza dane na procesorze graficznym (GPU) urządzenia Jetson.** `chloros_1.3.0_arm64_jp6.deb` jest skompilowany dla JetPack 6 i wykorzystuje wersję PyTorch z obsługą CUDA z pakietu JetPack 6, więc przetwarzanie — w tym funkcja Texture Aware debayer — odbywa się na procesorze graficznym Orin. Pakiet nie zawiera samej biblioteki CUDA: korzysta z bibliotek CUDA i cuDNN instalowanych przez JetPack 6 (`sudo apt install nvidia-jetpack` na obrazie, na którym ich brakuje). W przypadku braku dostępnego procesora graficznego z obsługą CUDA przetwarzanie odbywa się na procesorze centralnym (CPU), a moduł Texture Aware wykorzystuje standardowy algorytm debayera.
{% endhint %}

***

## Obsługiwane modele Jetson

| Model                | Pamięć RAM            | Strategia przetwarzania                                     | Zalecane zastosowanie                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB współdzielonej pamięci | `GPU_PARALLEL` (2 procesy robocze)                              | Maksymalna wydajność, duże zbiory danych                      |
| **Jetson Orin NX**   | 8–16 GB współdzielonej pamięci  | `GPU_PARALLEL` (2 procesy robocze, 16 GB) / `GPU_SINGLE` (8 GB)   | Główne zalecenie do wdrożeń w terenie              |
| **Jetson Orin Nano** | 8 GB współdzielonej pamięci     | `GPU_SINGLE` (1 procesor roboczy, sekwencyjny)                     | Podstawowe obliczenia brzegowe                                 |

{% hint style="info" %}
Pakiet Linux
a dla architektury arm64 wymaga **JetPack 6**, który jest dostępny w rodzinie urządzeń Jetson Orin. Starsze modele (Nano, TX2, Xavier NX) nie obsługują JetPack 6 i nie są obsługiwane przez obecny pakiet.
{% endhint %}

***

## Wymagania

* **JetPack 6.x** (zalecana najnowsza wersja)
* **NVIDIA CUDA i cuDNN** z pakietu JetPack 6 (`sudo apt install nvidia-jetpack`, jeśli w obrazie ich brakuje)
* **Płatny plan Chloros
+** — poziom Copper lub wyższy (wymagany do wszelkiego dostępu do CLI
/SDK
; egzekwowany po stronie serwera)

## Instalacja

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

Plik `.deb` instaluje SDK
dla systemu `python3`, zgodny z tą kompilacją. Tylko w środowisku virtualenv należy zainstalować dołączony plik wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (w trybie całkowicie offline). Ogólne informacje dotyczące instalacji w systemie Linux
, lokalizacji plików oraz rozwiązywania problemów można znaleźć w sekcji [Instalacja w systemie Linux
](linux-installation.md).

{% hint style="info" %}
**Koniec z rozpakowywaniem przy uruchomieniu.** Od wersji 1.3.0 backend i interfejs CLI
są instalowane w postaci nieopakowanej w katalogu `/usr/lib/chloros/`, więc nie rozpakowują się już do katalogu tymczasowego przy każdym uruchomieniu. Pliki tymczasowe nadal są automatycznie zapisywane w katalogu `/mnt/ssd/tmp`, o ile katalog ten istnieje; w przeciwnym razie można ustawić `TMPDIR` na ścieżkę na dysku NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamiczna adaptacja obliczeniowa na Jetsonie

### Jak to działa

Podczas uruchamiania Chloros
tworzy profil systemu:

1. **Wykrywa model Jetson** za pomocą `/proc/device-tree/model`
2. **Odczytuje dostępną pamięć współdzieloną GPU/CPU** (Jetson korzysta z pamięci zunifikowanej)
3. **Pyta PyTorch, czy może korzystać z CUDA** — jeśli zainstalowano CUDA i cuDNN z pakietu JetPack 6, jest to możliwe
4. **Automatycznie wybiera strategię przetwarzania** (`GPU_PARALLEL`, `GPU_SINGLE` lub `CPU_PARALLEL`) oraz liczbę procesów roboczych

Decyzja zależy od **całkowitej ilości współdzielonej pamięci RAM**, a nie od nazwy modelu:

* **Przy całkowitej pamięci RAM poniżej 12 GB**(wszystkie urządzenia Jetson z 8 GB pamięci):**1 procesor roboczy — celowe przetwarzanie sekwencyjne**. Pamięć jest zbyt ograniczona dla równoległej pracy procesorów roboczych, więc obrazy są przetwarzane pojedynczo. Na urządzeniach Jetson z**8 GB lub mniej** wątek 3 całkowicie pomija pulę procesów roboczych i wykonuje operacje na poszczególnych obrazach w ramach tego samego procesu. W logach zaplecza strategia ta nosi nazwę `GPU_SINGLE`.
* **12 GB lub więcej**(Orin NX 16 GB, AGX Orin): pamięć zunifikowana kwalifikuje się do strategii `GPU_PARALLEL`, ale liczba procesów roboczych jest**ograniczona do 2 na Jetsonie** — procesor graficzny, pamięć RAM procesów roboczych oraz konteksty CUDA dla poszczególnych procesów czerpią z tej samej wspólnej puli, więc większa liczba procesów roboczych wiąże się z ryzykiem błędów spowodowanych brakiem pamięci.
* **Brak działającej biblioteki CUDA** (brak bibliotek CUDA w JetPacku): `CPU_PARALLEL` przy 12 GB lub więcej, poniżej jeden obraz na raz.

Można zastąpić automatyczny wybór za pomocą zmiennej środowiskowej `CHLOROS_STRATEGY` — zobacz [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Zachowanie w poszczególnych modelach

| Model Jetson                | Strategia       | Pracownicy | Wykonanie                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekwencyjna pętla wewnątrzprocesowa (`tiled_gpu` przy ograniczonej pamięci) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekwencyjna pętla wewnątrzprocesowa                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Równoległe procesy robocze, ścieżka `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Równoległe procesy robocze, ścieżka `fused_gpu`  |

Kluczową różnicą między platformami jest **pamięć**. Jetson z 8 GB pamięci musi przetwarzać obrazy pojedynczo, stosując oszczędzające pamięć podejście kafelkowe w warunkach dużego obciążenia, podczas gdy Orin z 16 GB lub więcej pamięci może przetwarzać jednocześnie 2 obrazy za pomocą procesora graficznego (GPU), korzystając z połączonego potoku o wyższej przepustowości.

### Budżet GPU na model

Każdy model Jetson posiada również profil sprzętowy, który określa, jaką część wspólnej puli pamięci może zająć przetwarzanie, oraz skaluje rozmiary partii:

| Model | Górna granica budżetu GPU | Współczynnik wielkości partii | Zarezerwowane dla systemu/wyświetlacza |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Wykryta pamięć RAM dostosowuje profil: w przypadku urządzenia Jetson zgłaszającego **16 GB lub więcej** mnożnik partii zostaje zwiększony o ×1,2. Podstawowy rozmiar partii przed zastosowaniem mnożników wynosi 8 obrazów.

Pełne informacje na temat adaptacji obliczeniowej można znaleźć w sekcji [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md).

***

## Ograniczenie częstotliwości GPU dla funkcji Texture Aware w modelach Nano i Orin Nano

Algorytm debayeringu z obsługą tekstur wykorzystuje inferencję sieci neuronowej na GPU, co może wywołać **ostrzeżenia o nadmiernym natężeniu prądu**w modelach Jetson o niskim poborze mocy (klasa 10–15 W) przy pełnej częstotliwości taktowania GPU. Po przekazaniu `--debayer texture-aware` na**Jetson Nano lub Orin Nano**, CLI
sprawdza maksymalną częstotliwość procesora graficznego i ogranicza ją do **510 MHz** (510000000), jeśli aktualna częstotliwość jest wyższa:

* Jeśli CLI
może zapisać węzeł sysfs częstotliwości procesora graficznego, ograniczenie jest **stosowane automatycznie**, a na ekranie wyświetla się potwierdzenie.
* Jeśli nie (wymagane uprawnienia roota), CLI
wyświetla dokładne polecenie `sudo` umożliwiające ręczne zastosowanie ograniczenia, czeka chwilę, abyś mógł je przeczytać, a następnie kontynuuje działanie — przetwarzanie nadal przebiega, ale mogą pojawić się ostrzeżenia o nadmiernym poborze prądu.

Aby samodzielnie zastosować ograniczenie przed przetwarzaniem:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modele o większej mocy (Orin NX 25 W, AGX Orin 60 W) działają z pełną prędkością procesora graficznego; ograniczenie nie jest stosowane. Standardowy algorytm debayeringu nigdy nie uruchamia ograniczenia na żadnym modelu.

{% hint style="info" %}
**Tryb „Texture Aware” na Jetsonie zawsze przetwarza jeden obraz na raz.** Każdy procesor roboczy wymagałby własnego kontekstu CUDA (~1 GB) oraz własnej kopii modelu odszumiającego, na co nie pozwala pamięć zunifikowana — dlatego na platformie Jetson ścieżka „Texture Aware” jest przypisana do jednego procesora roboczego, a dostęp do procesora graficznego jest zserializowany. Należy spodziewać się, że na dowolnym urządzeniu Jetson funkcja „Texture Aware” będzie znacznie wolniejsza niż tryb standardowy. Tryb „Texture Aware” ma zastosowanie do obrazów z Survey3
: dane z LATTICE są zawsze przetwarzane przy użyciu standardowego algorytmu debayeringu. W przypadku braku nadającego się do użytku procesora graficznego CUDA uruchomienie wykorzystuje tryb „Standard”, a backend rejestruje jeden wiersz `[DEBAYER] … needs a CUDA GPU …` wyjaśniający przyczynę (pokazany wraz z `chloros-cli --verbose`).
{% endhint %}

***

## Zarządzanie temperaturą

Urządzenia Jetson mają ograniczony zapas termiczny, zwłaszcza w instalacjach zamkniętych lub powietrznych. Chloros
monitoruje temperaturę układu SoC i automatycznie ogranicza rozmiary partii:

| Temperatura         | Działanie                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Normalna praca — pełna prędkość przetwarzania          |
| **70°C** (Ostrzeżenie) | Wielkość partii jest stopniowo zmniejszana (100% → 50% w zakresie od 70°C do 80°C) |
| **80°C** (Krytyczny) | Agresywne ograniczanie wydajności (z 50% do 0% w zakresie od 80°C do 90°C) |
| **90°C** (Wyłączenie) | Całkowite zatrzymanie przetwarzania przez procesor graficzny — wymagane schłodzenie |

{% hint style="warning" %}
**Należy zapewnić odpowiednią wentylację i odprowadzanie ciepła** w celu zapewnienia ciągłości przetwarzania, zwłaszcza w zamkniętych obudowach terenowych lub systemach lotniczych. Ograniczenie wydajności termicznej spowoduje zmniejszenie przepustowości przetwarzania w celu ochrony sprzętu.
{% endhint %}

***

## Zarządzanie pamięcią

Urządzenia Jetson wykorzystują **pamięć zunifikowaną** — procesor graficzny (GPU) i procesor centralny (CPU) współdzielą tę samą fizyczną pamięć RAM. Podawana pojemność pamięci VRAM (np. ~15,3 GB w przypadku Orin NX 16 GB) nie jest pamięcią dedykowaną dla procesora graficznego; jest to ta sama pamięć RAM, z której korzysta system operacyjny i wszystkie inne procesy.

### Ostrzeżenie dotyczące pamięci wymiany i zalecenia

Przed rozpoczęciem przetwarzania na urządzeniu Jetson interfejs CLI
zlicza obrazy RAW znajdujące się w folderze wejściowym (`.tif`, `.tiff`, `.raw`, `.dng` — podglądy JPG nie są wliczane), szacuje maksymalne zapotrzebowanie na pamięć dla danego zadania i **ostrzega przed rozpoczęciem**, jeśli pamięć RAM + pamięć wymiany mogą okazać się niewystarczające. Ostrzeżenie ma nagłówek `LOW MEMORY WARNING - Jetson Detected`, wyświetla liczbę obrazów, pamięć RAM, aktualną przestrzeń wymiany oraz szacowaną wartość szczytową, a następnie podaje dokładne polecenia `fallocate` / `chmod` / `mkswap` / `swapon` dostosowane do dla danego projektu (nigdy mniejsze niż 8 GB). Program zatrzymuje się na kilka sekund, aby komunikat nie zniknął w historii przewijania, a następnie kontynuuje przetwarzanie.**Szacunkowe wartości pamięci używane w ostrzeżeniu:**

| Tryb debayera | Podstawowy | Na obraz |
| --- | --- | --- |
| Standardowy | ~1,5 GB | ~10 MB |
| Z uwzględnieniem tekstur | ~2,5 GB (model + środowisko uruchomieniowe Python
a) | ~15 MB |

Ostrzeżenie pojawia się, gdy szacowana wartość szczytowa przekracza sumę pamięci RAM i pamięci wymiany pomniejszoną o 1 GB marginesu bezpieczeństwa; uwzględniana jest wyłącznie pamięć wymiany **oparta na plikach** — konfiguracja oparta wyłącznie na zram nadal będzie sygnalizowana.

Aby ręcznie dodać pamięć wymiany (przykład: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Obsługa błędu OOM (Out of Memory)

Podczas przetwarzania Chloros
monitoruje pamięć GPU i zamiast ulegać awarii, płynnie ogranicza wydajność:

1. Gdy wykorzystanie pamięci GPU przekroczy **85%**, rozmiary partii są prewencyjnie zmniejszane
2. Jeśli nadal wystąpi zdarzenie braku pamięci, rozmiar partii jest **zmniejszany o połowę**, a przy każdym kolejnym zdarzeniu OOM — ponownie o połowę; każda kolejna pomyślnie przetworzona partia cofa tę karę o jeden stopień
3. W warunkach długotrwałego obciążenia potok przetwarzania przechodzi ze ścieżki `fused_gpu` na ścieżkę `tiled_gpu`, charakteryzującą się oszczędnym wykorzystaniem pamięci, a w ostateczności na przetwarzanie na procesorze (CPU)

Ponieważ procesor graficzny współdzieli pamięć RAM urządzenia Jetson, pamięć RAM wraz z pamięcią wymiany nadal ograniczają czas działania — powyższe ostrzeżenie dotyczące pamięci wymiany jest sygnałem, na który należy zwrócić uwagę.

***

## Wdrożenie w terenie

### Kwestie związane z zasilaniem

| Model Jetson     | Typowy pobór mocy | Uwagi                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Gniazdo cylindryczne DC          |
| Jetson Orin NX   | 10–25 W             | Gniazdo cylindryczne DC          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD lub gniazdo cylindryczne |

Zaplanuj budżet energetyczny pod kątem długotrwałego przetwarzania — szczytowy pobór mocy występuje podczas intensywnego obciążenia procesora graficznego w wątku 3 (przetwarzanie).

### Zalecenia dotyczące pamięci masowej

* **Dysk SSD NVMe** jest zdecydowanie zalecany w przypadku wdrożeń opartych na architekturze arm64
* Karty SD są zbyt wolne do przetwarzania — należy ich używać wyłącznie jako nośników rozruchowych
* Należy zaplanować 2–3-krotność rozmiaru surowych danych obrazu na przetworzone dane wyjściowe

### Praca w trybie bezmonitorowym przez SSH
Interfejs Chloros

Chloros
jest idealny dla wdrożeń Jetson w trybie bezmonitorowym:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Zawsze aktywny backend do synchronizacji czasu LATTICE / DAQ-E

Jeśli Twój Jetson steruje kamerami LATTICE lub czujnikami światła DAQ-E w trybie bezmonitorowym, włącz usługę systemd backend **jako swój użytkownik**, aby serwer główny PTP działał nieprzerwanie, a skrypty SDK
mogły znaleźć uruchomiony backend (moduł jest zainstalowany, ale domyślnie nie jest włączony):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend działa wtedy jako użytkownik: zapisywane przez niego dane pochodzą od Ciebie i korzysta on z Twojego `chloros-cli login`. Zobacz [Instalacja w systemie Linux
](linux-installation.md#always-on-ptp-for-headless-hosts), aby uzyskać szczegółowe informacje, w tym na temat konta root `chloros-backend.service` oraz tego, w jaki sposób pakiet umożliwia przypisanie portów PTP 319/320 bez uprawnień roota.

### Zautomatyzowane przetwarzanie za pomocą systemd

Utwórz usługę systemd do automatycznego przetwarzania:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` zwraca wynik niezerowy, gdy przebieg, który zażądał produktów, nie zapisuje żadnych obrazów, więc status błędu systemd ma znaczenie dla monitorowania.

Połącz z timerem systemd w celu przetwarzania zaplanowanego:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Przykładowe przepływy pracy

### Podstawowe przetwarzanie na Jetsonie

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Zestaw PythonSDK
dla Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Przetwarzanie wsadowe wielu lotów

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Zalecane systemy Jetson do zastosowań terenowych

W przypadku wdrożeń terenowych i powietrznych warto rozważyć następujące opcje płyt nośnych Jetson Orin NX 16 GB:

* **Zastosowania powietrzne/drony**: Systemy o odporności na wibracje (MIL-STD), lekkie (poniżej 300 g), z chłodzeniem pasywnym
* **W trudnych warunkach terenowych**: obudowy wodoodporne o stopniu ochrony IP67/IP69K z obsługą kamer GigE z zasilaniem PoE
* **Minimalne wymagania/budżetowe**: zestawy deweloperskie z dodatkowymi obudowami

Skontaktuj się z [pomocą techniczną MAPIR
](https://www.mapir.camera/community/contact), aby uzyskać konkretne zalecenia dotyczące sprzętu dostosowanego do Twojego scenariusza wdrożenia.

***

## Kolejne kroki

* [Instalacja systemu Linux
](linux-installation.md) — Ogólne informacje dotyczące instalacji systemu Linux

* [Dynamiczna adaptacja mocy obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md) — Pełny przewodnik po strategiach obliczeniowych
* [Potok przetwarzania](../processing-architecture/processing-pipeline.md) — Omówienie potoku 4-wątkowego
* [CLI
: wiersz poleceń](../CLI.md) — Przewodnik po CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Przewodnik po SDK

* [Podręcznik CLI
](../reference/cli-reference.md) i [Podręcznik SDK
](../reference/sdk-reference.md) — Wyczerpujące zestawienia poleceń i API
dla wersji 1.3.0
