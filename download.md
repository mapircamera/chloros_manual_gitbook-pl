---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Pobierz

Pobierz najnowszą wersję programu Chloros, aby rozpocząć pracę z przetwarzaniem obrazów wielospektralnych.

### Wymagania systemowe

#### Windows

| Wymaganie          | Minimalne                                              | Zalecane                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **System operacyjny** | Windows 10 (64-bitowy)                                  | Windows 11 (64-bitowy)                                  |
| **Procesor**        | Intel Core i5 lub równoważny                          | Intel Core i7 lub lepszy                              |
| **Pamięć (RAM)**     | 8 GB                                                  | 16 GB lub więcej                                         |
| **Karta graficzna**    | Kompatybilna z DirectX 11                                | Procesor graficzny NVIDIA z co najmniej 4 GB pamięci VRAM                            |
| **Pamięć masowa**          | 6 GB wolnego miejsca                                       | Dysk SSD z co najmniej 10 GB wolnego miejsca                            |
| **Wyświetlacz**          | 1920x1080                                            | 2560x1440 lub wyższa                                  |
| **Internet**         | Wymagany do \[opcjonalnej] aktywacji licencji Chloros+ | Wymagany do \[opcjonalnej] aktywacji licencji Chloros+ |

#### Linux amd64 (x86\_64)

| Wymagania       | Minimalne                    | Zalecane               |
| ----------------- | -------------------------- | ------------------------- |
| **Dystrybucja**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 lub lepszy   |
| **Pamięć (RAM)**  | 8 GB                        | 16 GB lub więcej              |
| **Karta graficzna** | Brak (przetwarzanie przez procesor)      | Karta graficzna NVIDIA z 4 GB+ pamięci VRAM |
| **Pamięć masowa**       | 2 GB wolnego miejsca             | Dysk SSD z 10 GB+ wolnego miejsca       |
| **Python**        | Python 3.7+ (dla SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Wymagania      | Minimalne                      | Zalecane                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB lub AGX Orin |
| **Pamięć (RAM)** | 8 GB (współdzielona przez GPU/CPU)         | 16 GB+ współdzielona                    |
| **Pamięć masowa**      | 2 GB wolnego miejsca               | Dysk SSD NVMe z co najmniej 10 GB wolnego miejsca        |
| **Python**       | Python 3.7+ (dla SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Przyspieszenie GPU**: Użytkownicy Chloros+ z kartami graficznymi NVIDIA mogą korzystać z przyspieszenia CUDA, co znacznie przyspiesza przetwarzanie. Działa to zarówno na Windows (karty graficzne do komputerów stacjonarnych), jak i Linux (karty graficzne do komputerów stacjonarnych i NVIDIA Jetson). Użytkownicy Chloros+ zyskują również przetwarzanie wielowątkowe, zapewniające maksymalną prędkość.
{% endhint %}

***

## Pobierz Chloros

### Najnowsza stabilna wersja (23 marca 2026 r.): Wersja 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Pobierz Chloros dla Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Pobierz Chloros dla Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Pobierz Chloros dla Linux arm64 / Jetson (.deb)</a>

#### Instalator Windows (GUI + CLI + Backend)

* **Typ pliku**: .exe (Instalator Windows)**Kroki instalacji:**

1. Pobierz powyższy plik .exe
2. Kliknij dwukrotnie instalator, aby rozpocząć instalację
3. Postępuj zgodnie z instrukcjami kreatora instalacji
4. Wybierz katalog instalacyjny (domyślny: `C:\Program Files\[USER]\Chloros\`)
5. Zakończ instalację i uruchom Chloros lub Chloros CLI
6. Zaloguj się na swoje [konto MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (lub kontynuuj korzystanie z wersji bezpłatnej)

{% hint style="success" %}
Instalator automatycznie dodaje `chloros-cli` do ścieżki systemowej PATH w celu uzyskania dostępu z wiersza poleceń.
{% endhint %}

#### Linux amd64 (pakiet .deb — CLI + Backend)

* **Typ pliku**: .deb (pakiet Debian/Ubuntu)
* **Architektura**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (pakiet .deb — CLI + Backend)

* **Typ pliku**: .deb (JetPack 6)
* **Architektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Szczegółowe instrukcje konfiguracji znajdziesz w sekcji [Instalacja Linux](linux/linux-installation.md), a wskazówki dotyczące konkretnie platformy Jetson — w [Przewodniku po NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (wszystkie platformy)

```bash
pip install chloros-sdk
```

Dokumentację można znaleźć w [API : Python SDK](api-python-sdk.md).

{% hint style="info" %}
**Użytkownicy Linux**: Pakiet `.deb` instaluje CLI oraz backend. Python SDK instaluje się osobno za pomocą pip. Nie ma interfejsu graficznego dla Linux — cała interakcja odbywa się za pośrednictwem CLI lub SDK.
{% endhint %}

***

## Dodatkowe zasoby

### Python SDK

Dla programistów i procesów automatyzacji zainstaluj Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacja**: [API: Python SDK](api-python-sdk.md)**Wymagania**: Musi być zainstalowany Chloros (instalator Windows lub pakiet Linux `.deb`), wymagane logowanie do licencji Chloros+***

## Co zawiera

### Instalator Windows

* ✅ **Chloros GUI** – w pełni funkcjonalny interfejs graficzny
* ✅ **Chloros CLI** – interfejs wiersza poleceń (wymagana licencja Chloros+)
* ✅ **Chloros Backend** – silnik przetwarzania
* ✅ **Profile kamer** – wstępnie skonfigurowane szablony kamer MAPIR

### Pakiet .deb Linux

* ✅ **Chloros CLI** – interfejs wiersza poleceń (wymaga licencji Chloros+)
* ✅ **Chloros Backend** – silnik przetwarzania
* ✅ **Profile kamer** – wstępnie skonfigurowane szablony kamer MAPIR
* ❌ Brak GUI — Linux jest wersją bezinterfejsową, dostępną wyłącznie dla CLI/SDK

### Python SDK (pip, wszystkie platformy)

* ✅ **Chloros SDK** — Python API (wymaga licencji Chloros+)***

## Aktualizacja do Chloros+

Odblokuj zaawansowane funkcje dzięki subskrypcji Chloros+:

* 🚀 **Przetwarzanie wielowątkowe** — przetwarzaj obrazy równolegle
* ⚡ **Przyspieszenie GPU (CUDA)** — wykorzystaj moc procesorów graficznych NVIDIA
* 💻 **Dostęp do CLI** — automatyzacja za pomocą narzędzi wiersza poleceń
* 🐍 **Python SDK** — programowy dostęp do API
* 📱 **Wiele urządzeń** — korzystaj na 2–10+ urządzeniach (w zależności od planu)
* **🐻 Zaawansowana metoda debayeringu z uwzględnieniem tekstur** — wysokiej jakości debayering z uwzględnieniem krawędzi w połączeniu z modelem usuwania szumów opartym na AI/ML, który eliminuje niemal cały szum debayeringu.
* 🧮 **Formuły niestandardowe** — twórz niestandardowe wskaźniki multispektralne

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobacz plany i ceny Chloros+</a></p>***

## Pomoc przy instalacji

### Rozwiązywanie problemów

**Instalacja kończy się niepowodzeniem z komunikatem o błędzie:**

* Upewnij się, że masz uprawnienia administratora
* Tymczasowo wyłącz oprogramowanie antywirusowe
* Sprawdź, czy spełniasz minimalne wymagania systemowe

**Aplikacja nie uruchamia się (Windows):**

* Sprawdź, czy zainstalowano Windows 10/11 (64-bit)
* Zaktualizuj sterowniki graficzne
* Sprawdź Windows Podgląd zdarzeń w celu uzyskania szczegółowych informacji o błędzie
* Skontaktuj się z pomocą techniczną, przekazując logi błędów

**CLI nie uruchamia się (Linux):**

* Sprawdź, czy pakiet `.deb` został poprawnie zainstalowany: `dpkg -l | grep chloros`
* Sprawdź uprawnienia: `sudo chmod +x /usr/bin/chloros-cli`
* Uruchom diagnostykę: `chloros-cli selftest`
* Sprawdź, czy nie brakuje bibliotek: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemy z aktywacją licencji:**

* Upewnij się, że połączenie internetowe jest aktywne
* Sprawdź dane uwierzytelniające pod adresem [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Sprawdź, czy zapora sieciowa nie blokuje Chloros
* Szczegółowe instrukcje znajdziesz w sekcji [Chloros+ Logowanie](chloros+-login.md)

### Pomoc techniczna

Potrzebujesz pomocy przy instalacji lub konfiguracji?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacja**: [Pierwsze kroki](./)
* ❓ **FAQ**: [Często zadawane pytania](faq.md)***

## Lista zmian

<details>

<summary>Wersja 1.1.0 (najnowsza)</summary>

**Data wydania: marzec 2026 r.**

**Nowe funkcje*** **Obsługa Linux** — Natywna obsługa CLI i SDK dla Linux amd64 (x86\_64) i arm64 (NVIDIA Jetson JetPack 6). Instalacja za pomocą pakietów `.deb`.
* **Obsługa NVIDIA Jetson** — Zoptymalizowane przetwarzanie dla urządzeń brzegowych Jetson Nano, Orin Nano, Orin NX i AGX Orin.
* **Dynamiczna adaptacja obliczeniowa** — Automatyczne wykrywanie sprzętu i optymalizacja strategii przetwarzania. Chloros dostosowuje się do sprzętu, od Jetson Nano po stację roboczą z wieloma procesorami graficznymi.
* **4-wątkowa potok przetwarzania** — Równoległe wątki wykrywania, kalibracji, przetwarzania i eksportu z dynamiczną alokacją pamięci GPU.
* **Nowe polecenia CLI** — `selftest` (diagnostyka systemu) i `update` (zarządzanie aktualizacjami Linux).
* **Nowe flagi procesu CLI** — `--debayer` (standardowe/z uwzględnieniem tekstur), `--indices` (określ indeksy), `--target` (najpierw przeszukaj docelowy podfolder w celu szybszego wykrywania).
* **Nowe pozycje menu GUI** — opcje Dodaj pliki, Dodaj folder oraz Rozpocznij/Zatrzymaj przetwarzanie są teraz dostępne w rozwijanym menu głównym.**Ulepszenia**

* Automatyczne wykrywanie backendu międzyplatformowego (ścieżki Windows i Linux)
* Ulepszono SDK `get_status()` poprzez śledzenie postępu dla każdego wątku
* Nowe wyjątki SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Zarządzanie temperaturą i adaptacyjne ograniczanie wydajności dla NVIDIA Jetson
* Automatyczne zarządzanie pamięcią z przełączeniem na przetwarzanie kafelkowe GPU w przypadku braku pamięci (OOM)

</details>

<details>

<summary>Wersja 1.0.5</summary>

**Data wydania: 10 lutego 2026 r.**

**Nowe funkcje*** **Metoda debayeringu z uwzględnieniem tekstury \[Tylko Chloros+] -** Metoda z uwzględnieniem tekstury wykorzystuje wysokiej jakości debayering z uwzględnieniem krawędzi w połączeniu z modelem odszumiania AI/ML, który usuwa prawie cały szum debayeringu.
* **Obsługa celów kalibracyjnych T4P*** **Szybsze przetwarzanie na GPU w Chloros+, lepsze zarządzanie pamięcią**

**Poprawki błędów*** Całkowicie nowy interfejs użytkownika (GUI), powinien teraz działać na wszystkich komputerach Windows.

</details>

<details>

<summary>Wersja 1.0.4</summary>

**Data wydania: 5 stycznia 2026 r.**

**Nowe funkcje*** **Przełącznik obraz/metadane**: Dodano przełącznik w przeglądarce plików, aby wyświetlać metadane wybranego obrazu w tabeli zamiast w siatce obrazów
* **Suwak powiększania siatki obrazów**: Nowy suwak interfejsu użytkownika do regulacji rozmiaru miniatur (obsługuje również skrót CTRL + kółko myszy)
* **Przyciski eksportu siatki obrazów**: Przyciski w górnym rzędzie do przełączania miniatur z formatu JPG na przetworzone pliki eksportowe (Targets, Reflectance, Index, LUT)
* **Zakładka Mapa**: Nowa interaktywna mapa 2D pokazująca znaczniki lokalizacji GPS obrazów
  * Obsługuje mapy Google Maps i kafelki map ESRI (automatycznie wybiera najlepszą usługę kafelkową w oparciu o dostępność poziomu powiększenia)
  * Podgląd miniatur po najechaniu myszką na znaczniki na mapie

**Poprawki błędów*** Ulepszona obsługa instalacji Chloros na komputerach z systemem w języku innym niż angielski

</details>

<details>

<summary>Wersja 1.0.3</summary>

**Data wydania: 20 grudnia 2025 r.**

**Nowe funkcje*** Pierwsze uruchomienie

**Ulepszenia*** Pierwsze uruchomienie

**Poprawki błędów*** Pierwsze uruchomienie

**Znane problemy*** Pierwsze uruchomienie

</details>***

## Umowa licencyjna**Oprogramowanie zastrzeżone** – Copyright (c) 2026 MAPIR Inc.

Niedozwolone jest nieautoryzowane używanie, rozpowszechnianie lub modyfikowanie.

**Wersja bezpłatna**: Dostępna do użytku osobistego i komercyjnego z ograniczeniami funkcji**Chloros+**: Licencja oparta na subskrypcji, zapewniająca zaawansowane funkcje i wdrożenia komercyjne
