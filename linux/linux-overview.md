# Linux – przegląd

Wersja Chloros 1.1.0 zapewnia natywną obsługę Linux dla **CLI**oraz**Python SDK**, umożliwiając przetwarzanie obrazów wielospektralnych w trybie bezmonitorowym na stacjach roboczych, serwerach i urządzeniach brzegowych NVIDIA Jetson.

{% hint style="info" %}
**Brak GUI na Linux.** GUI Chloros Desktop jest dostępne wyłącznie na Windows. Użytkownicy Linux korzystają z Chloros za pośrednictwem [CLI](../CLI.md) oraz [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Tabela zgodności platform

| Funkcja | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Interfejs graficzny na komputerze stacjonarnym** | Tak | Nie dotyczy | Nie | Nie |
| **CLI** | Tak | Tak | Tak | Tak |
| **Python SDK** | Tak | Tak | Tak | Tak |
| **Przyspieszenie GPU (CUDA)** | Tak | Tak | Tak | Tak (JetPack 6) |
| **Debayer uwzględniający tekstury** | Tak (Chloros+) | Tak (Chloros+) | Tak (Chloros+) | Tak (Chloros+) |
| **Dynamiczna adaptacja obliczeniowa** | Tak | Tak | Tak | Tak |***

## Obsługiwane architektury

| Architektura | Opis | Metoda instalacji |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardowe procesory do komputerów stacjonarnych/serwerów (Intel, AMD) | Pakiet `.deb` |
| **arm64 (aarch64)** | Procesory oparte na architekturze ARM, głównie NVIDIA Jetson | Pakiet `.deb` (JetPack 6) |

## Obsługiwane dystrybucje Linux

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson)***

## Co zyskują użytkownicy Linux

* **Chloros CLI** — Pełny interfejs wiersza poleceń do przetwarzania wsadowego, automatyzacji i tworzenia skryptów
* **Chloros Python SDK** — Programowy interfejs Python (`pip install chloros-sdk`) do integracji z procesami badawczymi i niestandardowymi narzędziami
* **Przyspieszenie GPU** — Przetwarzanie przyspieszane przez CUDA na procesorach graficznych NVIDIA (komputery stacjonarne i Jetson)
* **Dynamiczna adaptacja obliczeniowa** — Automatyczne wykrywanie sprzętu i optymalizacja strategii przetwarzania
* **Wszystkie funkcje przetwarzania** — Ten sam potok przetwarzania wielospektralnego co w Windows (kalibracja, korekcja winietowania, wskaźniki wegetacji, wszystkie formaty eksportu)
* **Funkcje Chloros+** — Przetwarzanie wielowątkowe, debayering z uwzględnieniem tekstury, niestandardowe wskaźniki (z licencją Chloros+)

## Czego nie otrzymują użytkownicy Linux

* **GUI na pulpicie** — Brak interfejsu graficznego; cała interakcja odbywa się za pośrednictwem CLI lub Python SDK
* **Przeglądarka obrazów** — Brak interaktywnej przeglądarki obrazów, widoku siatki ani znaczników na mapie
* **Wizualne zarządzanie projektami** — Projektami zarządza się za pomocą poleceń CLI i wywołań SDK***

## Pierwsze kroki z Linux

1. **Zainstaluj Chloros** — Zobacz [Instalacja Linux](linux-installation.md) w celu zainstalowania pakietu `.deb`
2. **Zainstaluj Python SDK** (opcjonalnie) — `pip install chloros-sdk`
3. **Aktywuj licencję** — `chloros-cli login your@email.com 'password'`
4. **Przetwórz swój pierwszy zestaw danych** — `chloros-cli process ~/datasets/flight001`

Użytkownicy NVIDIA Jetson powinni zapoznać się z dedykowanym [Przewodnikiem NVIDIA Jetson](nvidia-jetson-guide.md), zawierającym informacje dotyczące konfiguracji i optymalizacji specyficznej dla tej platformy.

***

## Kolejne kroki

* [Instalacja Linux](linux-installation.md) — Szczegółowe instrukcje instalacji dla amd64 i arm64
* [Przewodnik po NVIDIA Jetson](nvidia-jetson-guide.md) — konfiguracja specyficzna dla Jetson, zarządzanie temperaturą i wdrożenie w terenie
* [CLI : Wiersz poleceń](../CLI.md) — Pełna dokumentacja CLI
* [API : Python SDK](../api-python-sdk.md) — Pełna dokumentacja SDK
* [Dynamiczna adaptacja obliczeniowa](../processing-architecture/dynamic-compute-adaptation.md) — Jak Chloros dostosowuje się do sprzętu użytkownika
