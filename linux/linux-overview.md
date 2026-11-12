# Przegląd systemu Linux

Chloros 1.3.0 zapewnia natywną obsługę systemu Linux dla **CLI**oraz**Python SDK** — przetwarzanie obrazów wielospektralnych w trybie bezmonitorowym, a także sterowanie na żywo kamerami LATTICE i czujnikami światła DAQ — na stacjach roboczych z systemem Linux, serwerach oraz urządzeniach brzegowych NVIDIA Jetson.

{% hint style="info" %}
**Brak graficznego interfejsu użytkownika (GUI) w systemie Linux.**Graficzny interfejs użytkownika Chloros Desktop GUI jest dostępny wyłącznie w systemie Windows. Użytkownicy systemu Linux korzystają z Chloros za pośrednictwem [CLI](../CLI.md) oraz [Python SDK](../api-python-sdk.md). Program `.deb` dodaje wprawdzie pozycję**Chloros CLI** do menu aplikacji — po prostu otwiera emulator terminala z uruchomionym programem `chloros-cli`.
{% endhint %}

***

## Tabela zgodności z platformami

| Funkcja | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI na pulpicie** | Tak | Nie dotyczy | Nie | Nie |
| **Interfejs CLI** (`chloros-cli`) | Tak | Tak | Tak | Tak |
| **Python SDK** (`chloros-sdk`) | Tak | Tak | Tak | Tak |
| **Potok przetwarzania obrazu** | Tak | Tak | Tak | Tak |
| **Sterowanie kamerą LATTICE (na żywo)** | Tak (zakładka „Kamery”) | Tak (`chloros-cli lattice`, SDK) | Tak | Tak |
| **Czujniki światła DAQ (na żywo)** | Tak (zakładka „Czujniki światła”) | Tak (`chloros-cli daq pool-*`, SDK) | Tak | Tak |
| **Synchronizacja czasu PTP (host jest grandmasterem)** | Tak | Tak (`chloros-cli time-sync`) | Tak | Tak |
| **Przyspieszenie GPU (CUDA)** | Tak | Tak | Tak | Tak (JetPack 6) |
| **Debayering z uwzględnieniem tekstur** (wymaga procesora graficznego z obsługą CUDA) | Tak (Chloros+) | Tak (Chloros+) | Tak (Chloros+) | Tak (Chloros+) |
| **Dynamiczna adaptacja obliczeniowa** | Tak | Tak | Tak | Tak |
| **Backend jako usługa systemowa** (`chloros-backend@<user>.service`) | Nie | Nie | Tak (opcjonalnie) | Tak (opcjonalnie) |
| **Narzędzie do aktualizacji na miejscu** (`chloros-cli update`) | Nie (uruchom instalator) | Nie (uruchom instalator) | Tak | Tak |

***

## Obsługiwane architektury

| Architektura | Opis | Pakiet |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardowe procesory do komputerów stacjonarnych/serwerów (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesory ARM — rodzina NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Obsługiwane dystrybucje systemu Linux

* **Ubuntu 22.04 LTS lub nowszy** (amd64)
* **Debian 12 lub nowszy** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson Orin)

***

## Co zyskują użytkownicy systemu Linux

* **Chloros CLI** — pełny interfejs wiersza poleceń do przetwarzania wsadowego, automatyzacji i tworzenia skryptów
* **Chloros Python SDK** — programowy interfejs w języku Python do tworzenia potoków badawczych i narzędzi niestandardowych, instalowany przez `.deb` dla systemu `python3` (pakiet wheel o dopasowanej wersji dla środowisk virtualenv znajduje się w `/usr/lib/chloros/sdk/`; PyPI jest przeznaczone dla maszyn bez pakietu Chloros)
* **Sterowanie kamerami LATTICE** — wykrywanie, podłączanie, konfigurowanie i rejestrowanie obrazu z kamer LATTICE oraz zsynchronizowanych układów wielokamerowych za pośrednictwem `chloros-cli lattice` i SDK; `.deb` zawiera środowisko uruchomieniowe Arena SDK wymagane przez kamery
* **Sterowanie czujnikami światła DAQ** — podłączaj czujniki DAQ-U/M/E, przesyłaj strumieniowo skalibrowane widma i rejestruj pliki `.daq` za pośrednictwem `chloros-cli daq pool-*` oraz SDK
* **Synchronizacja czasu PTP** — backend Chloros uruchamia serwer główny PTP, z którym synchronizują się kamery LATTICE i czujniki DAQ-E; sprawdź go za pomocą `chloros-cli time-sync` i utrzymuj jego działanie w trybie bezinterfejsowym za pomocą jednostki systemd `chloros-backend@<user>.service` (zobacz [Instalacja w systemie Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatyzacja projektów** — uruchamiaj zapisane projekty w trybie bezinterfejsowym za pomocą `chloros-cli project` oraz `open_project` z SDK
* **Przyspieszenie GPU** — przetwarzanie przyspieszane przez CUDA na procesorach graficznych NVIDIA (procesory graficzne komputerów stacjonarnych na platformie amd64 oraz procesor graficzny Jetson z JetPack 6)
* **Dynamiczna adaptacja obliczeniowa** — automatyczne wykrywanie sprzętu i wybór strategii przetwarzania, z opcją nadpisania `CHLOROS_STRATEGY` jako specjalistycznym rozwiązaniem awaryjnym
* **Wszystkie funkcje przetwarzania** — ten sam potok przetwarzania co w systemie Windows: kalibracja, korekcja winietowania, wskaźniki wegetacji oraz wszystkie formaty eksportu
* **Funkcje Chloros+** — przetwarzanie wielowątkowe (potokowe), debayering z uwzględnieniem tekstury (na procesorze graficznym CUDA) oraz niestandardowe wskaźniki, dostępne w ramach płatnego planu Chloros+

## Czego nie otrzymują użytkownicy systemu Linux

* **Graficzny interfejs użytkownika** — brak interfejsu graficznego; cała obsługa odbywa się za pośrednictwem CLI lub Python SDK
* **Przeglądarka obrazów** — brak interaktywnej przeglądarki obrazów, widoku siatki ani znaczników na mapie
* **Wizualne zarządzanie projektami** — projekty są tworzone i sterowane za pomocą poleceń CLI oraz wywołań SDK (sam sprzęt — kamery, czujniki, urządzenia przechwytujące — pozostaje w pełni sterowany z terminala)

***

## Wymagania licencyjne

Dostęp do CLI i SDK wymaga **płatnego planu Chloros+ — Copper lub wyższego**(Copper, Bronze, Silver, Gold). Bezpłatny plan**Iron** nie zapewnia dostępu do CLI/SDK. Ograniczenie to jest egzekwowane przez backend, a nie tylko przez CLI:

| Sytuacja | Odpowiedź backendu |
| --- | --- |
| Nie zalogowano się | `401` z `error_code: AUTH_REQUIRED` |
| Zalogowany na bezpłatnym poziomie Iron | `403` z `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` działa na każdym planie — jest to jedyna trasa zwolniona z kontroli — więc przyczyna odmowy jest zawsze widoczna.

***

## Pierwsze kroki w systemie Linuxie

1. **Zainstaluj Chloros** — zobacz [Instalacja na Linuxie](linux-installation.md) w celu zainstalowania `.deb`
2. **Sprawdź** — `chloros-cli --version` wyświetla `Chloros CLI 1.3.0`; `chloros-cli selftest` uruchamia 7-etapową diagnostykę
3. **Sprawdź Python SDK** — `.deb` instaluje go: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (w środowisku virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Zaloguj się** — `chloros-cli login your@email.com 'your-password'` (raz na komputer; aktualizacje pakietu zachowują dane logowania, a usunięcie pakietu powoduje ich wyczyszczenie)
5. **Przetwórz swój pierwszy zbiór danych** — `chloros-cli process ~/datasets/flight001`

W przypadku NVIDIA Jetson zapoznaj się z dedykowanym [Przewodnikiem po NVIDIA Jetson](nvidia-jetson-guide.md), zawierającym informacje dotyczące konfiguracji specyficznej dla platformy, zachowania termicznego oraz wdrożenia w terenie.

***

## Kolejne kroki

* [Instalacja w systemie Linux](linux-installation.md) — szczegółowa instrukcja instalacji, lokalizacje plików oraz rozwiązywanie problemów dla architektur amd64 i arm64
* [Przewodnik po platformie NVIDIA Jetson](nvidia-jetson-guide.md) — konfiguracja specyficzna dla Jetson, zachowanie pamięci i charakterystyka termiczna, wdrożenie w terenie
* [CLI: wiersz poleceń](../CLI.md) — przewodnik po CLI
* [API: Python SDK](../api-python-sdk.md) — przewodnik po SDK
* [Podręcznik CLI](../reference/cli-reference.md) oraz [Podręcznik SDK](../reference/sdk-reference.md) — wyczerpujące wykazy poleceń/API dla wersji 1.3.0
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — jak Chloros dostosowuje się do Twojego sprzętu

{% hint style="info" %}
**Odczytywanie niniejszego podręcznika programowo.** Każda strona jest również udostępniana w formacie surowego Markdown pod własnym adresem URL z dodatkiem `.md` (na przykład `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), a spis treści całej instrukcji jest opublikowany pod adresem [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
