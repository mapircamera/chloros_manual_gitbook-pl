# Przewodnik po platformie NVIDIA Jetson

Chloros na platformie NVIDIA Jetson umożliwia przetwarzanie obrazów wielospektralnych na urządzeniach brzegowych — w terenie, na bezzałogowych statkach powietrznych (UAV) oraz w odległych instalacjach. Chloros 1.2.0 wykrywa model Jetson podczas uruchamiania i optymalizuje strategię przetwarzania pod kątem wykrytego sprzętu. **Nie jest wymagane ręczne dostosowywanie.**

***

## Obsługiwane modele Jetson

| Model                | Pamięć RAM            | Strategia przetwarzania                                     | Zalecane zastosowanie                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB współdzielonej pamięci | `GPU_PARALLEL` (2 procesy robocze)                              | Maksymalna wydajność, duże zbiory danych                      |
| **Jetson Orin NX**   | 8–16 GB współdzielonej pamięci  | `GPU_PARALLEL` (2 procesy robocze, 16 GB) / `GPU_SINGLE` (8 GB)   | Główne zalecenie do zastosowań lotniczych i terenowych |
| **Jetson Orin Nano** | 8 GB współdzielonej pamięci     | `GPU_SINGLE` (1 procesor roboczy, sekwencyjny)                     | Podstawowe rozwiązania do przetwarzania brzegowego                                 |

{% hint style="info" %}
Pakiet Linux dla architektury arm64 wymaga **JetPack 6**, który jest dostępny w rodzinie urządzeń Jetson Orin. Starsze modele (Nano, TX2, Xavier NX) nie obsługują JetPack 6 i nie są obsługiwane przez obecny pakiet.
{% endhint %}

***

## Wymagania

* **JetPack 6.x** (zalecana najnowsza wersja)
* **NVIDIA CUDA** (dołączona do JetPack)
* **Płatny plan Chloros+** — poziom Copper lub wyższy (wymagany dla wszystkich dostępów CLI/SDK; egzekwowany po stronie serwera)

## Instalacja

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Ogólne informacje dotyczące instalacji Linux, lokalizacji plików i rozwiązywania problemów można znaleźć w sekcji [Instalacja Linux](linux-installation.md).

{% hint style="info" %}
**Umieść katalog rozpakowywania na szybkim nośniku danych.** Skompilowane pliki binarne rozpakowują się do katalogu tymczasowego przy każdym uruchomieniu — co z karty SD przebiega niezwykle wolno. Chloros automatycznie korzysta z `/mnt/ssd/tmp`, jeśli ten istnieje; w przeciwnym razie należy ustawić `TMPDIR` na ścieżkę na dysku NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamiczna adaptacja obliczeniowa na Jetsonie

### Jak to działa

Podczas uruchamiania Chloros profiluje system:

1. **Wykrywa model Jetson** za pomocą `/proc/device-tree/model`
2. **Odczytuje dostępną pamięć współdzieloną GPU/CPU** (Jetson wykorzystuje pamięć zunifikowaną)
3. **Wybiera strategię przetwarzania** (`GPU_PARALLEL`, `GPU_SINGLE` lub `CPU_PARALLEL`)
4. **Automatycznie ustawia liczbę procesów roboczych, typ potoku oraz alokację pamięci**Decyzja zależy od**całkowitej wielkości współdzielonej pamięci RAM**, a nie od nazwy modelu:

* **Poniżej 12 GB całkowitej pamięci RAM**(wszystkie urządzenia Jetson o pojemności 8 GB): `GPU_SINGLE` z**1 procesem roboczym — celowe przetwarzanie sekwencyjne**. Pamięć jest zbyt ograniczona dla równoczesnych procesów roboczych, więc obrazy są przetwarzane pojedynczo. Na urządzeniach Jetson z**8 GB lub mniej** wątek 3 całkowicie pomija pulę procesów roboczych i wykonuje operacje dla poszczególnych obrazów w ramach tego samego procesu.
* **12 GB lub więcej**(Orin NX 16 GB, AGX Orin): pamięć zunifikowana kwalifikuje się do `GPU_PARALLEL`, ale liczba procesów roboczych jest**ograniczona do 2 na Jetsonie** — procesor graficzny, pamięć RAM procesów roboczych oraz konteksty CUDA poszczególnych procesów czerpią zasoby z tej samej wspólnej puli, więc większa liczba procesów roboczych wiąże się z ryzykiem błędów spowodowanych brakiem pamięci.

Można zastąpić automatyczny wybór za pomocą zmiennej środowiskowej `CHLOROS_STRATEGY` — zobacz [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Zachowanie w poszczególnych modelach

| Model Jetson                | Strategia       | Procesy robocze | Wykonanie                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekwencyjna pętla wewnątrzprocesowa (`tiled_gpu` w przypadku niedoboru pamięci) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekwencyjna pętla wewnątrzprocesowa                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Równoległe procesy robocze, ścieżka `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Równoległe procesy robocze, ścieżka `fused_gpu`  |

Kluczową różnicą między platformami jest **pamięć**. Jetson z 8 GB pamięci musi przetwarzać obrazy pojedynczo, stosując oszczędzające pamięć podejście kafelkowe w warunkach dużego obciążenia, podczas gdy Orin z 16 GB lub więcej pamięci może przetwarzać jednocześnie 2 obrazy za pomocą procesora graficznego (GPU), korzystając z połączonego potoku o wyższej przepustowości.

### Budżet GPU na model

Każdy model Jetson posiada również profil sprzętowy, który określa, jaką część wspólnej puli pamięci może zająć przetwarzanie, oraz skaluje rozmiary partii:

| Model | Górna granica budżetu GPU | Współczynnik wielokrotności rozmiaru partii | Rezerwa dla systemu/wyświetlacza |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Wykryta pamięć RAM dostosowuje profil: w przypadku urządzenia Jetson zgłaszającego **16 GB lub więcej** mnożnik partii zostaje zwiększony o ×1,2. Podstawowy rozmiar partii przed zastosowaniem mnożników wynosi 8 obrazów.

Pełne informacje na temat adaptacji obliczeniowej można znaleźć w sekcji [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md).

***

## Ograniczenie częstotliwości GPU dla funkcji Texture Aware w modelach Nano i Orin Nano

Algorytm debayeringu z obsługą tekstur wykorzystuje inferencję sieci neuronowej na GPU, co może wywołać **ostrzeżenia o nadmiernym natężeniu prądu**w modelach Jetson o niskim poborze mocy (klasa 10–15 W) przy pełnej częstotliwości taktowania GPU. Przed rozpoczęciem przetwarzania przez algorytm Texture Aware na**Jetson Nano lub Orin Nano**moduł Chloros sprawdza maksymalną częstotliwość procesora graficznego i ogranicza ją do**510 MHz** (510000000), jeśli aktualna częstotliwość jest wyższa:

* Jeśli skrypt CLI może zapisać wartość w węźle sysfs częstotliwości procesora graficznego, ograniczenie jest **stosowane automatycznie**, a na ekranie wyświetlany jest komunikat potwierdzający.
* Jeśli nie (wymagane uprawnienia roota), CLI wyświetla dokładne polecenie `sudo` służące do ręcznego zastosowania ograniczenia, czeka chwilę, abyś mógł je przeczytać, a następnie kontynuuje — przetwarzanie nadal trwa, ale mogą pojawić się ostrzeżenia o nadmiernym natężeniu prądu.

Aby samodzielnie zastosować ograniczenie przed przetwarzaniem:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modele o wyższej mocy (Orin NX 25 W, AGX Orin 60 W) działają z pełną prędkością procesora graficznego; ograniczenie nie jest stosowane. Standardowy algorytm debayeringu nigdy nie uruchamia ograniczenia w żadnym modelu.

{% hint style="info" %}
**Tryb „Texture Aware” na Jetsonie zawsze przetwarza po jednym obrazie na raz.** Każdy procesor roboczy potrzebowałby własnego kontekstu CUDA (~1 GB) oraz własnej kopii modelu odszumiającego, na co nie pozwala pamięć zunifikowana — dlatego na Jetsonie ścieżka „Texture Aware” jest przypisana do jednego procesora roboczego, a dostęp do procesora graficznego jest zserializowany. Należy spodziewać się, że tryb „Texture Aware” będzie znacznie wolniejszy niż tryb „Standard” na każdym urządzeniu Jetson.
{% endhint %}

***

## Zarządzanie temperaturą

Urządzenia Jetson mają ograniczony zapas termiczny, zwłaszcza w instalacjach zamkniętych lub pokładowych. Chloros monitoruje temperaturę układu SoC i automatycznie ogranicza rozmiary partii:

| Temperatura         | Działanie                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Normalna praca — pełna prędkość przetwarzania          |
| **70°C** (Ostrzeżenie)  | Wielkość partii jest stopniowo zmniejszana (100% → 50% w zakresie od 70°C do 80°C) |
| **80°C** (Stan krytyczny) | Agresywne ograniczanie wydajności (z 50% do 0% w zakresie od 80°C do 90°C) |
| **90°C** (Wyłączenie) | Całkowite zatrzymanie przetwarzania przez procesor graficzny — wymagane schłodzenie |

{% hint style="warning" %}
**Należy zapewnić odpowiednią wentylację i odprowadzanie ciepła** w celu zapewnienia ciągłości przetwarzania, zwłaszcza w zamkniętych obudowach terenowych lub systemach lotniczych. Ograniczenie wydajności termicznej spowoduje zmniejszenie przepustowości przetwarzania w celu ochrony sprzętu.
{% endhint %}

***

## Zarządzanie pamięcią

Urządzenia Jetson wykorzystują **pamięć zunifikowaną** — procesor graficzny (GPU) i procesor centralny (CPU) współdzielą tę samą fizyczną pamięć RAM. Podawana pojemność pamięci VRAM (np. ~15,3 GB w przypadku Orin NX 16 GB) nie jest pamięcią dedykowaną dla procesora graficznego; jest to ta sama pamięć RAM, z której korzysta system operacyjny i wszystkie inne procesy.

### Ostrzeżenie dotyczące pamięci wymiany i zalecenia

Przed rozpoczęciem przetwarzania na Jetsonie narzędzie CLI zlicza obrazy RAW w folderze wejściowym (`.tif`, `.tiff`, `.raw`, `.dng` — podglądy w formacie JPG nie są liczone), szacuje szczytowe zapotrzebowanie na pamięć dla danego zadania i **ostrzega przed rozpoczęciem**, jeśli pamięć RAM + pamięć wymiany mogą okazać się niewystarczające. Ostrzeżenie ma nagłówek `LOW MEMORY WARNING - Jetson Detected`, wyświetla liczbę obrazów, ilość pamięci RAM, aktualny rozmiar przestrzeni wymiany oraz szacowane maksymalne zapotrzebowanie, a następnie podaje dokładne polecenia `fallocate` / `chmod` / `mkswap` / `swapon` dostosowane do rozmiaru projektu (nigdy mniejsze niż 8 GB). Program wstrzymuje się na kilka sekund, aby komunikat nie zniknął w historii przewijania, a następnie kontynuuje przetwarzanie.**Szacunki dotyczące pamięci wykorzystywane przez ostrzeżenie:**

| Tryb debayera | Podstawowe | Na obraz |
| --- | --- | --- |
| Standardowy | ~1,5 GB | ~10 MB |
| Z uwzględnieniem tekstur | ~2,5 GB (model + środowisko uruchomieniowe Python) | ~15 MB |

Ostrzeżenie pojawia się, gdy szacowana wartość szczytowa przekracza sumę pamięci RAM i pamięci wymiany pomniejszoną o 1 GB marginesu bezpieczeństwa; uwzględniana jest wyłącznie pamięć wymiany **oparta na plikach** — konfiguracja oparta wyłącznie na zram nadal będzie sygnalizowana.

Aby ręcznie dodać pamięć wymiany (przykład: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### Obsługa błędu OOM (brak pamięci)

Podczas przetwarzania Chloros monitoruje pamięć GPU i zamiast ulegać awarii, płynnie ogranicza wydajność:

1. Gdy wykorzystanie pamięci GPU przekroczy **85%**, rozmiary partii są prewencyjnie zmniejszane
2. Jeśli nadal wystąpi zdarzenie braku pamięci, rozmiar partii jest **zmniejszany o połowę**, a przy każdym kolejnym zdarzeniu OOM — ponownie o połowę; każda kolejna pomyślnie przetworzona partia cofa tę karę o jeden krok
3. W warunkach długotrwałego obciążenia potok przetwarzania przechodzi z ścieżki `fused_gpu` na ścieżkę `tiled_gpu`, charakteryzującą się oszczędnym wykorzystaniem pamięci, a w ostateczności na przetwarzanie na procesorze (CPU)

***

## Wdrożenie w terenie

### Kwestie związane z zasilaniem

| Model Jetson     | Typowy pobór mocy | Uwagi                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Gniazdo cylindryczne DC          |
| Jetson Orin NX   | 10–25 W             | Gniazdo cylindryczne DC          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD lub gniazdo cylindryczne |

Zaplanuj budżet energetyczny pod kątem długotrwałego przetwarzania — szczytowy pobór mocy występuje podczas intensywnie obciążającego procesor graficzny wątku 3 (przetwarzanie).

### Zalecenia dotyczące pamięci masowej

* **Dysk SSD NVMe** jest zdecydowanie zalecany w przypadku wdrożeń na platformie arm64
* Karty SD są zbyt wolne do przetwarzania — używaj ich wyłącznie jako nośnika rozruchowego
* Zaplanuj 2–3-krotność rozmiaru surowych danych obrazu na przetworzone dane wyjściowe

### Praca bez monitora za pomocą SSH

Chloros CLI idealnie nadaje się do wdrożeń Jetson w trybie bezmonitorowym:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Zawsze aktywny backend do synchronizacji czasu LATTICE / DAQ-E

Jeśli urządzenie Jetson steruje kamerami LATTICE lub czujnikami światła DAQ-E w trybie bez monitoru, należy włączyć usługę systemd backend, aby serwer główny PTP działał w trybie ciągłym (moduł jest zainstalowany, ale domyślnie nie jest włączony):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Szczegółowe informacje, w tym sposób, w jaki pakiet umożliwia przypisanie portów PTP 319/320 bez uprawnień roota, znajdziesz w sekcji [Instalacja Linux](linux-installation.md#always-on-ptp-for-headless-hosts).

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
    --indices NDVI NDRE GNDVI
```

### Python SDK na Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
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
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Zalecane systemy Jetson do zastosowań terenowych

W przypadku zastosowań terenowych i powietrznych warto rozważyć następujące opcje płyt nośnych Jetson Orin NX 16 GB:

* **Zastosowania lotnicze/drony**: Systemy o odporności na wibracje (MIL-STD), lekkie (poniżej 300 g), z chłodzeniem pasywnym
* **Wymagające warunki terenowe**: Obudowy wodoodporne IP67/IP69K z obsługą kamer GigE z zasilaniem PoE
* **Minimalne/ekonomiczne**: zestawy deweloperskie z dodatkowymi obudowami

Skontaktuj się z [pomocą techniczną MAPIR](https://www.mapir.camera/community/contact), aby uzyskać konkretne zalecenia dotyczące sprzętu dostosowanego do Twojego scenariusza wdrożenia.

***

## Kolejne kroki

* [Instalacja Linux](linux-installation.md) — Ogólne informacje dotyczące instalacji Linux
* [Dynamiczna adaptacja mocy obliczeniowej](../processing-architecture/dynamic-compute-adaptation.md) — Pełny opis strategii obliczeniowych
* [Potok przetwarzania](../processing-architecture/processing-pipeline.md) — Omówienie potoku 4-wątkowego
* [CLI: Wiersz poleceń](../CLI.md) — Przewodnik po CLI
* [API : Python SDK](../api-python-sdk.md) — Podręcznik SDK
* [CLI — Informacje](../reference/cli-reference.md) oraz [SDK — Informacje](../reference/sdk-reference.md) — Wyczerpujące wykazy poleceń/API dla wersji 1.2.0
