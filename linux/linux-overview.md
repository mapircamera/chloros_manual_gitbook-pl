# Linux – przegląd

Chloros 1.2.0 zapewnia natywną obsługę Linux dla **CLI**i**Python SDK** — przetwarzanie obrazów wielospektralnych w trybie bezmonitorowym, a także sterowanie na żywo kamerami LATTICE i czujnikami światła DAQ — na stacjach roboczych, serwerach oraz urządzeniach brzegowych NVIDIA Jetson z systemem Linux.

{% hint style="info" %}
**Brak graficznego interfejsu użytkownika (GUI) na Linux.**Graficzny interfejs użytkownika Chloros jest dostępny wyłącznie na Windows. Użytkownicy Linux korzystają z Chloros za pośrednictwem [CLI](../CLI.md) oraz [Python SDK](../api-python-sdk.md). `.deb` dodaje wprawdzie pozycję**Chloros CLI** do menu aplikacji — po prostu otwiera emulator terminala z uruchomionym `chloros-cli`.
{% endhint %}

***

## Tabela zgodności z platformami

| Funkcja | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfejs graficzny na komputerze stacjonarnym** | Tak | Nie dotyczy | Nie | Nie |
| **CLI** (`chloros-cli`) | Tak | Tak | Tak | Tak |
| **Python SDK** (`chloros-sdk`) | Tak | Tak | Tak | Tak |
| **Potok przetwarzania obrazu** | Tak | Tak | Tak | Tak |
| **Sterowanie kamerami LATTICE (na żywo)** | Tak (zakładka „Kamery”) | Tak (`chloros-cli lattice`, SDK) | Tak | Tak |
| **Czujniki światła DAQ (na żywo)** | Tak (zakładka „Czujniki światła”) | Tak (`chloros-cli daq pool-*`, SDK) | Tak | Tak |
| **Synchronizacja czasu PTP (host jest grandmasterem)** | Tak | Tak (`chloros-cli time-sync`) | Tak | Tak |
| **Przyspieszenie GPU (CUDA)** | Tak | Tak | Tak | Tak (JetPack 6) |
| **Debayering z uwzględnieniem tekstur** | Tak (Chloros+) | Tak (Chloros+) | Tak (Chloros+) | Tak (Chloros+) |
| **Dynamiczna adaptacja obliczeniowa** | Tak | Tak | Tak | Tak |
| **Backend jako usługa systemowa** (`chloros-backend.service`) | Nie | Nie | Tak (opcjonalnie) | Tak (opcjonalnie) |
| **Aktualizator bezprzerwowy** (`chloros-cli update`) | Nie (uruchom instalator) | Nie (uruchom instalator) | Tak | Tak |***

## Obsługiwane architektury

| Architektura | Opis | Pakiet |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardowe procesory do komputerów stacjonarnych/serwerów (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesory ARM — rodzina NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (kompilacja JetPack 6) |

## Obsługiwane dystrybucje Linux

* **Ubuntu 22.04 LTS lub nowsze** (amd64)
* **Debian 12 lub nowszy** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson Orin)***

## Co zyskują użytkownicy Linux

* **Chloros CLI** — pełny interfejs wiersza poleceń do przetwarzania wsadowego, automatyzacji i tworzenia skryptów
* **Chloros Python SDK** — programowy interfejs Python przeznaczony do potoków badawczych i narzędzi niestandardowych (można go zainstalować z PyPI, a także jest dołączony do pakietu `.deb` jako plik wheel o dopasowanej wersji)
* **Sterowanie kamerami LATTICE** — wykrywanie, podłączanie, konfigurowanie i rejestrowanie obrazu z kamer LATTICE oraz zsynchronizowanych układów wielokamerowych za pośrednictwem `chloros-cli lattice` i SDK; pakiet `.deb` zawiera środowisko uruchomieniowe Arena SDK wymagane przez te kamery
* **Sterowanie czujnikami światła DAQ** — podłączaj czujniki DAQ-U/M/E, przesyłaj strumieniowo skalibrowane widma i rejestruj pliki `.daq` za pomocą `chloros-cli daq pool-*` oraz SDK
* **Synchronizacja czasu PTP** — backend Chloros uruchamia serwer PTP grandmaster, z którym synchronizują się kamery LATTICE i czujniki DAQ-E; sprawdź go za pomocą `chloros-cli time-sync`, i utrzymuj go w trybie bezinterfejsowym za pomocą jednostki systemd `chloros-backend.service` (zobacz [Instalacja Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatyzacja projektów** — uruchamiaj zapisane projekty w trybie bezinterfejsowym za pomocą `chloros-cli project` oraz `open_project` z modułu SDK
* **Przyspieszenie GPU** — przetwarzanie przyspieszane przez CUDA na procesorach graficznych NVIDIA (komputery stacjonarne i Jetson)
* **Dynamiczna adaptacja obliczeniowa** — automatyczne wykrywanie sprzętu i wybór strategii przetwarzania, z opcją nadpisania `CHLOROS_STRATEGY` jako ekspertową opcją awaryjną
* **Wszystkie funkcje przetwarzania** — ten sam potok przetwarzania co w Windows: kalibracja, korekcja winietowania, wskaźniki wegetacji oraz wszystkie formaty eksportu
* **Funkcje Chloros+** — przetwarzanie wielowątkowe (potokowe), debayering z uwzględnieniem tekstur oraz indeksy niestandardowe, dostępne w ramach płatnego planu Chloros+

## Czego nie otrzymują użytkownicy Linux

* **Interfejs graficzny na komputerze stacjonarnym** — brak interfejsu graficznego; cała obsługa odbywa się za pośrednictwem CLI lub Python SDK
* **Przeglądarka obrazów** — brak interaktywnej przeglądarki obrazów, widoku siatki ani znaczników na mapie
* **Wizualne zarządzanie projektami** — projekty są tworzone i sterowane za pomocą poleceń CLI oraz wywołań SDK (sam sprzęt — kamery, czujniki, urządzenia rejestrujące — pozostaje w pełni sterowany z terminala)***

## Wymagania licencyjne

Dostęp do CLI i SDK wymaga **płatnego planu Chloros+ — Copper lub wyższego**(Copper, Bronze, Silver, Gold). Bezpłatny poziom**Iron** nie zapewnia dostępu do funkcji CLI/SDK. Ograniczenie to jest egzekwowane przez zaplecze, a nie tylko przez CLI:

| Sytuacja | Odpowiedź serwera |
| --- | --- |
| Nie zalogowany | `401` z `error_code: AUTH_REQUIRED` |
| Zalogowany na bezpłatnym poziomie Iron | `403` z `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` działa na każdym planie — jest to jedyna trasa zwolniona z bramki — więc przyczyna odmowy jest zawsze widoczna.

***

## Pierwsze kroki z Linux

1. **Zainstaluj Chloros** — zobacz [Instalacja Linux](linux-installation.md) w celu zainstalowania `.deb`
2. **Sprawdź** — `chloros-cli --version` wyświetla komunikat `Chloros CLI 1.2.0`; `chloros-cli selftest` uruchamia 7-etapową diagnostykę
3. **Zainstaluj Python i SDK** (opcjonalnie) — `pip install chloros-sdk`
4. **Zaloguj się** — `chloros-cli login your@email.com 'your-password'` (raz na urządzenie oraz ponownie po każdej aktualizacji pakietu)
5. **Przetwórz swój pierwszy zestaw danych** — `chloros-cli process ~/datasets/flight001`

W przypadku NVIDIA Jetson zapoznaj się z dedykowanym [Przewodnikiem po NVIDIA Jetson](nvidia-jetson-guide.md), zawierającym informacje dotyczące konfiguracji specyficznej dla platformy, charakterystyki termicznej oraz wdrożenia w terenie.

***

## Kolejne kroki

* [Instalacja Linux](linux-installation.md) — szczegółowa instrukcja instalacji, lokalizacje plików oraz rozwiązywanie problemów dla architektur amd64 i arm64
* [Przewodnik po platformie NVIDIA Jetson](nvidia-jetson-guide.md) — konfiguracja specyficzna dla platformy Jetson, zachowanie pamięci i charakterystyka termiczna, wdrożenie w terenie
* [CLI : Wiersz poleceń](../CLI.md) — przewodnik CLI
* [API : Python SDK](../api-python-sdk.md) — przewodnik SDK
* [CLI — dokumentacja](../reference/cli-reference.md) oraz [SDK — dokumentacja](../reference/sdk-reference.md) — wyczerpujące wykazy poleceń/API dla wersji 1.2.0
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — jak Chloros dostosowuje się do posiadanego sprzętu

{% hint style="info" %}
**Odczytywanie tej instrukcji programowo.** Każda strona jest również udostępniana w postaci surowego kodu Markdown pod własnym adresem URL oraz `.md` (na przykład `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), a spis treści całej instrukcji jest opublikowany pod adresem [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
