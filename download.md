---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Pobierz

Pobierz najnowszą wersję programu Chloros, aby rozpocząć przetwarzanie obrazów wielospektralnych.

### Wymagania systemowe

#### Windows

| Wymaganie          | Minimalne                                              | Zalecane                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **System operacyjny** | Windows 10 (64-bitowy)                                  | Windows 11 (64-bitowy)                                  |
| **Procesor**        | Intel Core i5 lub równoważny                          | Intel Core i7 lub lepszy                              |
| **Pamięć (RAM)**     | 8 GB                                                  | 16 GB lub więcej                                         |
| **Karta graficzna**    | Kompatybilna z DirectX 11                                | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM                            |
| **Miejsce na dysku**          | 6 GB wolnego miejsca                                       | Dysk SSD z co najmniej 10 GB wolnego miejsca                            |
| **Rozdzielczość ekranu**          | 1920x1080                                            | 2560x1440 lub wyższa                                  |
| **Internet**         | Wymagany do [opcjonalnej] aktywacji licencji Chloros+ | Wymagany do [opcjonalnej] aktywacji licencji Chloros+ |

#### Linux amd64 (x86\_64)

| Wymagania       | Minimalne                    | Zalecane               |
| ----------------- | -------------------------- | ------------------------- |
| **Dystrybucja**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 lub lepszy   |
| **Pamięć (RAM)**  | 8 GB                        | 16 GB lub więcej              |
| **Karta graficzna** | Brak (przetwarzanie przez procesor)      | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM |
| **Pamięć masowa**       | 2 GB wolnego miejsca             | Dysk SSD z co najmniej 10 GB wolnego miejsca       |
| **Python**        | Python 3.7+ (dla SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Wymagania      | Minimalne                      | Zalecane                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB lub AGX Orin |
| **Pamięć (RAM)** | 8 GB (współdzielona przez GPU/CPU)         | 16 GB+ współdzielona                    |
| **Pamięć masowa**      | 2 GB wolnego miejsca               | Dysk SSD NVMe z co najmniej 10 GB wolnego miejsca        |
| **Python**       | Python 3.7+ (dla SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Przyspieszenie GPU**: Użytkownicy Chloros+ z procesorami graficznymi NVIDIA mogą korzystać z przyspieszenia CUDA, co znacznie przyspiesza przetwarzanie. Działa to zarówno na Windows (karty graficzne do komputerów stacjonarnych), jak i na Linux (karty graficzne do komputerów stacjonarnych oraz NVIDIA Jetson). Użytkownicy Chloros+ zyskują również przetwarzanie wielowątkowe, zapewniające maksymalną prędkość.
{% endhint %}

***

## Pobierz Chloros

### Najnowsza stabilna wersja: wersja 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Pobierz Chloros dla Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Pobierz Chloros dla Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Pobierz Chloros dla Linux arm64 / Jetson (.deb)</a>

#### Instalator Windows (GUI + CLI + Backend)

* **Typ pliku**: .exe (Instalator Windows)**Kroki instalacji:**

1. Pobierz powyższy plik .exe
2. Kliknij dwukrotnie instalator, aby rozpocząć instalację
3. Postępuj zgodnie z instrukcjami kreatora instalacji
4. Wybierz katalog instalacyjny (domyślnie: `C:\Program Files\MAPIR\Chloros\`)
5. Zakończ instalację i uruchom program Chloros lub Chloros CLI
6. Zaloguj się na swoje [konto MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (lub kontynuuj korzystanie z wersji bezpłatnej)

{% hint style="success" %}
Instalator automatycznie dodaje `chloros-cli` do ścieżki systemowej PATH, umożliwiając dostęp z wiersza poleceń.
{% endhint %}

#### Linux amd64 (pakiet .deb — CLI + backend)

* **Typ pliku**: .deb (pakiet Debian/Ubuntu)
* **Architektura**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (pakiet .deb — CLI + backend)

* **Typ pliku**: .deb (JetPack 6)
* **Architektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Szczegółowe instrukcje konfiguracji znajdziesz w sekcji [Instalacja Linux](linux/linux-installation.md), a wskazówki dotyczące konkretnie platformy Jetson — w [Przewodniku po NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (wszystkie platformy)

Każdy instalator zawiera odpowiedni pakiet `chloros_sdk`, dzięki czemu wersja SDK zawsze pasuje do zainstalowanego GUI/CLI/backendu. W przypadku Windows instalator automatycznie instaluje go w systemie Python; w przypadku Linux `.deb` umieszcza pakiet typu „wheel” w `/usr/lib/chloros/sdk/` i wyświetla polecenie instalacyjne:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

W przypadku hostów obsługujących wyłącznie pip (bez zainstalowanego pakietu Chloros) SDK jest również dostępny na PyPI:

```bash
pip install chloros-sdk
```

Zobacz [API : Python SDK](api-python-sdk.md) oraz [Podręcznik SDK](reference/sdk-reference.md) w celu zapoznania się z dokumentacją.

Użytkownicy {% hint style="info" %}
**Użytkownicy Linux**: Pakiet `.deb` instaluje CLI oraz backend. Nie ma interfejsu graficznego dla Linux — cała obsługa odbywa się za pośrednictwem CLI lub SDK.
{% endhint %}

***

## Dodatkowe zasoby

### Python SDK

Dla programistów i procesów automatyzacji należy zainstalować Chloros, Python oraz SDK:

```bash
pip install chloros-sdk
```

**Dokumentacja**: [API: Python SDK](api-python-sdk.md)**Wymagania**: Musi być zainstalowany Chloros (instalator Windows lub pakiet Linux `.deb`), wymagane jest zalogowanie się na licencję Chloros+***

## Co zawiera pakiet

### Instalator Windows

* ✅ **Chloros GUI** – w pełni funkcjonalny interfejs graficzny
* ✅ **Chloros CLI** – Interfejs wiersza poleceń (wymaga licencji Chloros+)
* ✅ **Chloros Backend** – Silnik przetwarzania
* ✅ **Profile kamer** – wstępnie skonfigurowane szablony kamer MAPIR

### Pakiet .deb Linux

* ✅ **Chloros CLI** – interfejs wiersza poleceń (wymaga licencji Chloros+)
* ✅ **Chloros Backend** — silnik przetwarzania
* ✅ **Profile kamer** — wstępnie skonfigurowane szablony kamer MAPIR
* ❌ Brak GUI — Linux działa wyłącznie w trybie bezinterfejsu graficznego (headless) CLI/SDK

### Python SDK (pip, wszystkie platformy)

* ✅ **Chloros SDK** — Python API (wymagana licencja Chloros+)***

## Przejdź na wersję Chloros+

Odblokuj zaawansowane funkcje dzięki subskrypcji Chloros+:

* 🚀 **Przetwarzanie wielowątkowe** — równoległe przetwarzanie obrazów
* ⚡ **Przyspieszenie GPU (CUDA)** — wykorzystaj moc procesorów graficznych NVIDIA
* 💻 **Dostęp do CLI** – Automatyzacja za pomocą narzędzi wiersza poleceń
* 🐍 **Python SDK** – Programowy dostęp do API
* 📱 **Wiele urządzeń** – Korzystaj z 2–10+ urządzeń (w zależności od planu)
* **🐻 Zaawansowana metoda debayeringu uwzględniająca teksturę** – wysokiej jakości debayering uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na AI/ML, który eliminuje niemal cały szum powstający podczas debayeringu.
* 🧮 **Formuły niestandardowe** – Twórz własne wskaźniki wielospektralne

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobacz plany i ceny Chloros+</a></p>***

## Pomoc dotycząca instalacji

### Rozwiązywanie problemów

**Instalacja kończy się niepowodzeniem z komunikatem o błędzie:**

* Upewnij się, że masz uprawnienia administratora
* Tymczasowo wyłącz oprogramowanie antywirusowe
* Sprawdź, czy spełniasz minimalne wymagania systemowe

**Aplikacja nie uruchamia się (Windows):**

* Sprawdź, czy zainstalowany jest Windows 10/11 (64-bitowy)
* Zaktualizuj sterowniki karty graficznej
* Sprawdź Podgląd zdarzeń Windows w celu uzyskania szczegółów błędu
* Skontaktuj się z pomocą techniczną, przekazując logi błędów

**CLI nie uruchamia się (Linux):**

* Sprawdź, czy pakiet `.deb` został poprawnie zainstalowany: `dpkg -l | grep chloros`
* Sprawdź uprawnienia: `sudo chmod +x /usr/bin/chloros-cli`
* Uruchom diagnostykę: `chloros-cli selftest`
* Sprawdź, czy nie brakuje bibliotek: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemy z aktywacją licencji:**

* Upewnij się, że połączenie internetowe jest aktywne
* Zweryfikuj dane logowania pod adresem [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Sprawdź, czy zapora sieciowa nie blokuje Chloros
* Szczegółowe instrukcje znajdziesz w sekcji [Chloros+ Logowanie](chloros+-login.md)

### Pomoc techniczna

Potrzebujesz pomocy przy instalacji lub konfiguracji?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacja**: [Pierwsze kroki](./)
* ❓ **FAQ**: [Często zadawane pytania](faq.md)***

## Aktualizacje oprogramowania

Chloros sprawdza dostępność aktualizacji, informuje o pojawieniu się nowej wersji i przekierowuje na tę stronę pobierania — aktualizację przeprowadza się poprzez uruchomienie nowego, podpisanego instalatora. Twoje ustawienia i projekty pozostają nienaruszone po aktualizacji. W przypadku Linux i Jetson program `chloros-cli update` sprawdza dostępność nowszej wersji i proponuje pobranie oraz zainstalowanie odpowiedniego pliku `.deb` (polecenie to jest dostępne wyłącznie w wersji Linux).

***

## Dziennik zmian**Wersja 1.2.0 (najnowsza)**— pełną listę funkcji znajdziesz w sekcji**Co nowego w Chloros 1.2.0** na stronie [Pierwsze kroki](./).

<details>

<summary>Wersja 1.0.5</summary>

**Data wydania: 10 lutego 2026 r.**

**Nowe funkcje*** **Metoda debayeringu z uwzględnieniem tekstury \[tylko Chloros+] —** Metoda „Texture Aware” wykorzystuje wysokiej jakości algorytm debayeringu z uwzględnieniem krawędzi w połączeniu z modelem usuwania szumu opartym na sztucznej inteligencji (AI/ML), który eliminuje niemal cały szum powstały podczas debayeringu.
* **Obsługa celów kalibracyjnych T4P*** **Szybsze przetwarzanie na GPU w Chloros+, lepsze zarządzanie pamięcią**

**Poprawki błędów*** Całkowicie nowy interfejs użytkownika (GUI), powinien teraz działać na wszystkich komputerach Windows.

</details>

<details>

<summary>Wersja 1.0.4</summary>

**Data wydania: 5 stycznia 2026 r.**

**Nowe funkcje*** **Przełączanie między obrazem a metadanymi**: Dodano przełącznik w przeglądarce plików, umożliwiający wyświetlanie metadanych wybranego obrazu w tabeli zamiast w siatce obrazów
* **Suwak powiększenia siatki obrazów**: Nowy suwak w interfejsie użytkownika do regulacji rozmiaru miniatur (obsługuje również kombinację klawiszy CTRL + kółko myszy)
* **Przyciski eksportu siatki obrazów**: Przyciski w górnym rzędzie służące do przełączania miniatur z formatu JPG na przetworzone pliki eksportowe (Targets, Reflectance, Index, LUT)
* **Zakładka „Mapa”**: Nowa interaktywna mapa 2D pokazująca znaczniki lokalizacji GPS obrazów
  * Obsługa Google Maps i kafelków mapowych ESRI (automatyczny wybór najlepszej usługi kafelkowej w zależności od dostępności poziomu powiększenia)
  * Podgląd miniatur po najechaniu kursorem na znaczniki na mapie

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

Niedozwolone jest nieautoryzowane używanie, rozpowszechnianie lub modyfikowanie oprogramowania.

**Wersja bezpłatna**: Dostępna do użytku osobistego i komercyjnego z ograniczeniami funkcjonalności**Chloros+**: Licencja oparta na subskrypcji, zapewniająca dostęp do zaawansowanych funkcji i wdrożeń komercyjnych
