# Przewodnik po platformie NVIDIA Jetson

Chloros na platformie NVIDIA Jetson umożliwia przetwarzanie obrazów wielospektralnych w warunkach brzegowych — w terenie, na bezzałogowych statkach powietrznych (UAV) oraz w instalacjach zdalnych. Chloros automatycznie wykrywa model Jetson i optymalizuje strategię przetwarzania pod kątem posiadanego sprzętu.

***

## Obsługiwane modele Jetson

| Model                | Pamięć RAM            | Strategia przetwarzania                                   | Zalecane zastosowanie                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB współdzielonej pamięci | `GPU_PARALLEL` (4 procesy robocze)                            | Maksymalna wydajność, duże zbiory danych                      |
| **Jetson Orin NX**   | 8–16 GB współdzielonej pamięci  | `GPU_PARALLEL` (3 procesy robocze, 16 GB) / `GPU_SINGLE` (8 GB) | Główne zalecenie do zastosowań lotniczych i terenowych |
| **Jetson Orin Nano** | 8 GB współdzielonej pamięci     | `GPU_SINGLE` (1 procesor roboczy)                               | Podstawowe przetwarzanie brzegowe                                 |
| **Jetson Nano**      | 4–8 GB współdzielonej pamięci   | `GPU_SINGLE` (1 procesor roboczy)                               | Podstawowy, z ograniczoną pamięcią                          |

{% hint style="info" %}
**Starsze modele Jetson** (TX2, TX1, Xavier NX) mogą nie być obsługiwane. Wydajność będzie się różnić w zależności od dostępnej pamięci GPU i możliwości CUDA.
{% endhint %}

***

## Wymagania

* **JetPack 6.x** (zalecana najnowsza wersja)
* **NVIDIA CUDA** (dołączona do JetPack)
* **Licencja Chloros+** (wymagana do uzyskania dostępu do CLI/SDK)

## Instalacja

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Ogólne informacje na temat instalacji Linux można znaleźć w sekcji [Instalacja Linux](linux-installation.md).

***

## Dynamiczna adaptacja obliczeniowa na Jetson

Chloros automatycznie wykrywa model Jetson i wybiera optymalną strategię przetwarzania. **Nie jest wymagane ręczne dostrajanie.**

### Jak to działa

Podczas uruchamiania Chloros profiluje system:

1. **Wykrywa model Jetson** za pomocą `/proc/device-tree/model`
2. **Odczytuje dostępną pamięć GPU/współdzieloną**

3.**Wybiera strategię przetwarzania** (`GPU_PARALLEL`, `GPU_SINGLE` lub `CPU_PARALLEL`)
4. **Automatycznie ustawia liczbę procesów roboczych, typ potoku i alokację pamięci**

### Zachowanie poszczególnych modeli

| Model Jetson                | Strategia       | Pracownicy | Potok                       | Współbieżność |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (oszczędny pod względem pamięci) | Seryjny  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Seryjny  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializowane  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (pełna ścieżka GPU)    | Równoległe  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Równoległe  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** to idealne rozwiązanie do wdrożeń brzegowych — wykorzystuje strategię `GPU_PARALLEL` z 3 równoległymi procesami, zapewniając prawdziwe równoległe przetwarzanie na GPU w kompaktowej obudowie.
{% endhint %}

Kluczową różnicą między platformami jest **pamięć**. Jetson Nano z 8 GB pamięci współdzielonej musi przetwarzać obrazy pojedynczo, stosując oszczędzające pamięć podejście kafelkowe, podczas gdy Orin NX z 16 GB może przetwarzać 3 obrazy jednocześnie na GPU, korzystając z potokowego systemu o wyższej przepustowości.

Pełne informacje na temat adaptacji obliczeniowej można znaleźć w sekcji [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md).

***

## Zarządzanie temperaturą

Urządzenia Jetson mają ograniczony zapas mocy termicznej, zwłaszcza w zastosowaniach zamkniętych lub powietrznych. Chloros obejmuje automatyczne monitorowanie temperatury i dławienie:

| Temperatura         | Działanie                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Normalna praca — pełna prędkość przetwarzania          |
| **70°C** (Ostrzeżenie)  | Automatyczne zmniejszenie wielkości partii                   |
| **80°C** (Krytyczne) | Agresywne ograniczanie wydajności — mniejsza współbieżność         |
| **90°C** (Wyłączenie) | Całkowite zatrzymanie przetwarzania przez procesor graficzny — wymagane schłodzenie |

{% hint style="warning" %}
**Należy zapewnić odpowiednią wentylację i odprowadzanie ciepła** w celu zapewnienia ciągłości przetwarzania, zwłaszcza w zamkniętych obudowach terenowych lub systemach powietrznych. Ograniczanie wydajności termicznej zmniejszy przepustowość przetwarzania w celu ochrony sprzętu.
{% endhint %}

***

## Zarządzanie pamięcią

Urządzenia Jetson wykorzystują **pamięć zunifikowaną** — procesor graficzny (GPU) i procesor centralny (CPU) współdzielą tę samą fizyczną pamięć RAM. Oznacza to, że podawana pojemność pamięci VRAM (np. 15,3 GB w przypadku Orin NX 16 GB) nie jest pamięcią dedykowaną dla procesora graficznego; jest ona współdzielona z systemem operacyjnym i innymi procesami.

### Zalecenia dotyczące pamięci wymiany

W przypadku dużych zbiorów danych lub przetwarzania debayera z uwzględnieniem tekstur, Chloros może zalecić utworzenie przestrzeni wymiany:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Szacunkowe zużycie pamięci na obraz:**

* Standardowe odszyfrowanie: ~10 MB na obraz
* Odszyfrowanie z uwzględnieniem tekstur: ~15 MB na obraz

Chloros automatycznie oblicza wymaganą pamięć na podstawie rozmiaru zbioru danych i ostrzega, jeśli zalecane jest użycie pamięci wymiany.

### Rezerwa na wypadek braku pamięci (OOM)

Jeśli podczas przetwarzania wykryty zostanie brak pamięci:

1. Chloros automatycznie zmniejsza liczbę procesorów graficznych
2. Przechodzi z potoku `fused_gpu` do potoku `tiled_gpu` (bardziej wydajnego pod względem pamięci)
3. Kontynuuje przetwarzanie przy zmniejszonej przepustowości zamiast ulegać awarii

***

## Wdrożenie w terenie

### Kwestie związane z zasilaniem

| Model Jetson     | Typowy pobór mocy | Uwagi                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C lub złącze cylindryczne    |
| Jetson Orin Nano | 7–15 W              | Gniazdo cylindryczne DC          |
| Jetson Orin NX   | 10–25 W             | Gniazdo cylindryczne DC          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD lub gniazdo cylindryczne |

Zaplanuj budżet energetyczny na ciągłe przetwarzanie — szczytowy pobór mocy występuje podczas intensywnego dla GPU wątku 3 (przetwarzanie).

### Zalecenia dotyczące pamięci masowej

* **Dysk SSD NVMe** jest zdecydowanie zalecany w przypadku wdrożeń arm64
* Karty SD są zbyt wolne do przetwarzania — używaj ich wyłącznie jako nośnika startowego
* Zaplanuj 2–3-krotność rozmiaru surowych danych obrazu na przetworzone dane wyjściowe

### Praca bez monitorów za pośrednictwem SSH

Chloros CLI idealnie nadaje się do wdrożeń Jetson bez monitorów:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Zautomatyzowane przetwarzanie za pomocą systemd

Utwórz usługę systemd do zautomatyzowanego przetwarzania:

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

Połącz z timerem systemd w celu zaplanowanego przetwarzania:

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

### Podstawowe przetwarzanie na Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Zalecane systemy Jetson do użytku w terenie

W przypadku wdrożeń terenowych i powietrznych warto rozważyć następujące opcje płyt nośnych Jetson Orin NX 16 GB:

* **Lotnicze/drony**: Systemy o odporności na wibracje (MIL-STD), lekkie (poniżej 300 g), z chłodzeniem pasywnym
* **W trudnych warunkach terenowych**: obudowy wodoodporne IP67/IP69K z obsługą kamer GigE z zasilaniem PoE
* **Minimalne/ekonomiczne**: zestawy deweloperskie z dodatkowymi obudowami

Skontaktuj się z [MAPIR Support](https://www.mapir.camera/community/contact), aby uzyskać konkretne zalecenia dotyczące sprzętu dla Twojego scenariusza wdrożenia.

***

## Kolejne kroki

* [Linux Instalacja](linux-installation.md) — Ogólne szczegóły dotyczące instalacji Linux
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — Pełne informacje dotyczące strategii obliczeniowej
* [Potok przetwarzania](../processing-architecture/processing-pipeline.md) — Omówienie potoku 4-wątkowego
* [CLI : Wiersz poleceń](../CLI.md) — Pełny opis CLI
* [API : Python SDK](../api-python-sdk.md) — Pełny opis SDK
