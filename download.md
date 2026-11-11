---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Pobierz

Pobierz najnowszą wersję programu Chloros
, aby rozpocząć pracę z przetwarzaniem obrazów wielospektralnych.

### Wymagania systemowe

#### Windows


| Wymaganie          | Minimalne                                              | Zalecane                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **System operacyjny** | Windows
10 (64-bitowy)                                  | Windows
11 (64-bitowy)                                  |
| **Procesor**        | Intel Core i5 lub równoważny                          | Intel Core i7 lub lepszy                              |
| **Pamięć (RAM)**     | 8 GB                                                  | 16 GB lub więcej                                         |
| **Karta graficzna**    | Zgodna z DirectX 11                                | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM                            |
| **Miejsce na dysku**          | 6 GB wolnego miejsca                                       | Dysk SSD z co najmniej 10 GB wolnego miejsca                            |
| **Ekran**          | 1920x1080                                            | 2560x1440 lub wyższa rozdzielczość                                  |
| **Internet**         | Wymagany do [opcjonalnej] aktywacji licencji Chloros
+ | Wymagany do [opcjonalnej] aktywacji licencji Chloros
+ |

#### Linux
amd64 (x86_64)

| Wymagania       | Minimalne                    | Zalecane               |
| ----------------- | -------------------------- | ------------------------- |
| **Dystrybucja**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 lub lepszy   |
| **Pamięć (RAM)**  | 8 GB                        | 16 GB lub więcej              |
| **Karta graficzna** | Brak (przetwarzanie przez procesor)      | Karta graficzna NVIDIA z co najmniej 4 GB pamięci VRAM |
| **Pamięć masowa**       | 2 GB wolnego miejsca             | Dysk SSD z co najmniej 10 GB wolnego miejsca       |
| **Python**        | Python
3.7+ (dla SDK
)      | Python
3.10+              |

#### Linux
arm64 (NVIDIA Jetson)

| Wymagania      | Minimalne                      | Zalecane                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson z JetPack 6 | Jetson Orin NX 16 GB lub AGX Orin |
| **Pamięć (RAM)** | 8 GB (współdzielona przez GPU/CPU)         | 16 GB+ współdzielonej pamięci                    |
| **Pamięć masowa**      | 2 GB wolnego miejsca               | Dysk SSD NVMe z co najmniej 10 GB wolnego miejsca        |
| **Python**       | Python
3.7+ (dla SDK
)        | Python
3.10+                    |

{% hint style="info" %}
**Przyspieszenie GPU**: Użytkownicy Chloros
+ posiadający procesory graficzne NVIDIA mogą korzystać z przyspieszenia CUDA, co zapewnia znacznie szybsze przetwarzanie. Funkcja ta działa z procesorami graficznymi NVIDIA przeznaczonymi do komputerów stacjonarnych w systemach Windows
i Linux
amd64, a także na platformie NVIDIA Jetson z JetPack 6. Użytkownicy Chloros
+ zyskują również możliwość przetwarzania wielowątkowego, zapewniającego maksymalną prędkość.
{% endhint %}

***

## Pobierz Chloros


### Najnowsza stabilna wersja: wersja 1.3.0 (11 listopada 2026 r.)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Pobierz Chloros dla systemu Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Pobierz Chloros dla systemu Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Pobierz Chloros dla systemu Linux arm64 / Jetson (.deb)</a>


#### Instalator dla systemu Windows
(GUI + CLI
+ Backend)

* **Typ pliku**: .exe (instalator dla systemu Windows
)

**Kroki instalacji:**

1. Pobierz powyższy plik .exe
2. Kliknij dwukrotnie instalator, aby rozpocząć instalację
3. Postępuj zgodnie z instrukcjami kreatora instalacji
4. Wybierz katalog instalacyjny (domyślnie: `C:\Program Files\Chloros\`)
5. Na stronie **Wybierz dodatkowe zadania**pozostaw zaznaczone obie opcje**Ustawienia systemowe** (zalecane — patrz poniżej)
6. Zakończ instalację i uruchom Chloros
lub Chloros
CLI
7. Zaloguj się na swoje [konto MAPIR
Cloud Chloros
+](https://cloud.mapir.camera/pricing) (lub kontynuuj korzystanie z wersji bezpłatnej)

**Opcje ustawień systemowych** (obie zaznaczone domyślnie):

| Opcja, zgodnie z opisem w instalatorze | Co robi | Wyłączenie cichej instalacji |
| --- | --- | --- |
| **Włącz długie ścieżki w systemie Windows** (pozwala na umieszczanie folderów projektów w lokalizacjach dłuższych niż 260 znaków; zalecane) | Ustawia ogólnosystemowe ustawienie `LongPathsEnabled` w systemie Windows
na wartość 1. System Windows
jest fabrycznie dostarczany z wyłączoną tą opcją, co oznacza, że żadna ścieżka do pliku nie może być dłuższa niż 260 znaków; produkty Chloros
zajmują miejsce do około 130 znaków poniżej folderu projektu, więc przy wyłączonej opcji długich ścieżek głębszy folder projektu nie zostanie zaakceptowany. Zmiana zaczyna obowiązywać dla programów uruchomionych po instalacji. Pozostaje aktywna po odinstalowaniu, ponieważ inne programy mogą z niej korzystać. | `/MERGETASKS="!longpaths"` |
| **Wykluczenie modułu zaplecza Chloros
a ze skanowania przez Microsoft Defender** (znacznie szybsze przechwytywanie obrazu z kamer; zalecane) | Dodaje wykluczenie procesu Microsoft Defender dla `chloros-backend.exe` w folderze instalacyjnym, dzięki czemu ochrona w czasie rzeczywistym nie skanuje każdego pliku zapisywanego przez moduł zaplecza, niezależnie od lokalizacji zapisu. Przy wysokich częstotliwościach przechwytywania skanowanie pochłania około 30% częstotliwości przechwytywania (77,7 w porównaniu z 52,7 przechwyceń na sekundę podczas ciągłego przechwytywania z 9 kamer). Żaden inny program nie jest wykluczony, a wykluczenie jest usuwane podczas odinstalowywania. W przypadku braku programu Defender lub jego zablokowania przez zasady, ten krok kończy się niepowodzeniem bez komunikatu, a instalacja jest kontynuowana. | `/MERGETASKS="!defenderexclusion"` |

Instalacja w trybie cichym (`/SILENT` lub `/VERYSILENT`) stosuje obie opcje, chyba że zrezygnujesz z nich, podając wartość `/MERGETASKS`.

{% hint style="success" %}
Instalator dodaje folder CLI
(`C:\Program Files\Chloros\cli\`) do systemu `PATH` tylko raz — ponowna instalacja lub aktualizacja nie powoduje już dodania zduplikowanego wpisu i usuwa duplikaty dodane przez wcześniejsze instalatory. Po zakończeniu instalacji otwórz nowe okno terminala, aby korzystać z `chloros-cli`. Instalator dodaje również dwie reguły zapory systemu Windows
dla backendu w każdym profilu sieciowym: **Chloros
Backend (UDP-In)**, dla strumieni z kamer, PTP, mDNS oraz strumienia danych DAQ-E, oraz**Chloros
Backend (TCP-In)**, ograniczony do lokalnej podsieci, służący do aktualizacji oprogramowania układowego DAQ-E, podczas której czujnik łączy się z powrotem z tym komputerem w celu pobrania obrazu. Obie reguły są usuwane podczas odinstalowywania.
{% endhint %}

#### Linux
amd64 (pakiet .deb — CLI
+ backend)

* **Typ pliku**: .deb (pakiet Debian/Ubuntu)
* **Architektura**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux
arm64 — NVIDIA Jetson (pakiet .deb — CLI
+ backend)

* **Typ pliku**: .deb (JetPack 6)
* **Architektura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Użyj nazwy pliku, który faktycznie pobrałeś, i zachowaj `./`, aby `apt` zainstalował ten plik zamiast szukać pakietu po nazwie. Zainstaluj za pomocą `apt install`, a nie za pomocą `dpkg -i`: apt instaluje również wymagania Python
SDK
, które pakiet zaleca, a nie wymaga, a `dpkg -i` je pomija. Od wersji 1.3.0 backend i interfejs CLI
są instalowane w postaci rozpakowanej pod `/usr/lib/chloros/`, dzięki czemu uruchamiają się bez konieczności uprzedniego rozpakowywania.

Pakiet Jetson wykorzystuje wersję PyTorch z CUDA z JetPack 6, więc przetwarzanie, w tym zawarty w nim moduł Texture Aware debayer, odbywa się na procesorze graficznym Jetson. Wykorzystuje on biblioteki CUDA i cuDNN instalowane przez JetPack 6 (`sudo apt install nvidia-jetpack` na obrazie, na którym ich brakuje).

Szczegółowe instrukcje konfiguracji można znaleźć w sekcji [Instalacja w systemie Linux
](linux/linux-installation.md), a wskazówki dotyczące konkretnie platformy Jetson — w [Przewodniku po NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python
SDK
(wszystkie platformy)

Każdy instalator zawiera odpowiedni pakiet wheel `chloros_sdk`, dzięki czemu wersja SDK
zawsze jest zgodna z zainstalowanym interfejsem GUI/CLI
/backendem. W systemie Windows
instalator automatycznie instaluje go w systemowym Python
. W systemie Linux
pakiet `.deb` instaluje SDK
dla samego systemu `python3` (w katalogu `/usr/lib/python3/dist-packages`), dzięki czemu `import chloros_sdk` działa zaraz po instalacji, bez pip i bez połączenia z siecią. Jedynie środowisko virtualenv, które nie ma dostępu do pakietów systemowych, wymaga zainstalowania dołączonego pliku wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

W przypadku hostów korzystających wyłącznie z pip (bez zainstalowanego pakietu Chloros
; SDK
steruje wówczas backendem na innym komputerze), SDK
jest również dostępne na PyPI:

```bash
pip install chloros-sdk
```

Dokumentację można znaleźć w sekcji [API
: Python
SDK
](api-python-sdk.md) oraz w [Podręczniku SDK
](reference/sdk-reference.md).

{% hint style="info" %}
**Użytkownicy systemu Linux**: Pakiet `.deb` instaluje interfejs CLI
, moduł zaplecza oraz SDK
dla języka Python
. W systemie Linux
nie ma interfejsu graficznego — cała interakcja odbywa się za pośrednictwem CLI
lub SDK
.
{% endhint %}

***

## Dodatkowe zasoby

### SDK
Python


Dla programistów i procesów automatyzacji Chloros
Python
SDK
jest dołączony do każdego instalatora: instalator dla systemu Windows
instaluje go w Twoim środowisku Python
, a instalator `.deb` dla systemu Linux
instaluje go dla systemu `python3`. Na komputerze bez pakietu Chloros
(takim, który steruje backendem na innym komputerze) zainstaluj go z PyPI:

```bash
pip install chloros-sdk
```

**Dokumentacja**: [API
: Python
SDK
](api-python-sdk.md)

**Wymagania**: backend Chloros
do obsługi (instalator dla systemu Windows
lub pakiet `.deb` dla systemu Linux
instaluje taki backend) oraz konto Chloros
+ na płatnym planie

***

## Co zawiera pakiet

### Instalator dla systemu Windows


* ✅ **Interfejs graficzny Chloros** – w pełni funkcjonalny interfejs graficzny
* ✅ **Chloros
CLI
** – interfejs wiersza poleceń (wymaga licencji Chloros
+)
* ✅ **Backend Chloros** – silnik przetwarzania
* ✅ **Python
SDK
** — instalowany w środowisku Python
, jeśli jest ono obecne (wymaga licencji Chloros
+)
* ✅ **Profile kamer** — wstępnie skonfigurowane szablony kamer MAPIR


### Pakiet .deb dla systemu Linux


* ✅ **Chloros
CLI
** – interfejs wiersza poleceń (wymaga licencji Chloros
+)
* ✅ **Chloros
Backend** – silnik przetwarzania
* ✅ **Python
SDK
** – zainstalowany dla systemu `python3` (wymaga licencji Chloros
+)
* ✅ **Profile kamer** – wstępnie skonfigurowane szablony kamer MAPIR

* ❌ Brak GUI — wersja dla systemu Linux
jest bezinterfejsowa, dostępna wyłącznie w postaci CLI
/SDK


### Python
SDK
(PyPI, dla maszyn bez pakietu Chloros
)

* ✅ **Chloros
SDK
** – Python
API
(wymaga licencji Chloros
+)

***

## Przejdź na Chloros
+

Odblokuj zaawansowane funkcje dzięki subskrypcji Chloros
+:

* 🚀 **Przetwarzanie wielowątkowe** — równoległe przetwarzanie obrazów
* ⚡ **Przyspieszenie GPU (CUDA)** – Wykorzystaj moc procesorów graficznych NVIDIA
* 💻 **Dostęp przez CLI** – Automatyzacja za pomocą narzędzi wiersza poleceń
* 🐍 **Python
SDK
** – Programowy dostęp do API

* 📱 **Wiele urządzeń** – Korzystaj na 2–10+ urządzeniach (w zależności od planu)
* **🐻 Zaawansowana metoda debayeringu uwzględniająca teksturę** – wysokiej jakości debayering uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na AI/ML, który eliminuje niemal cały szum powstały podczas debayeringu.
* 🧮 **Formuły niestandardowe** – Twórz własne wskaźniki wielospektralne

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobacz plany i ceny Chloros+</a></p>***

## Pomoc dotycząca instalacji

### Rozwiązywanie problemów

**Instalacja kończy się niepowodzeniem z komunikatem o błędzie:**

* Upewnij się, że masz uprawnienia administratora
* Tymczasowo wyłącz oprogramowanie antywirusowe
* Sprawdź, czy spełniasz minimalne wymagania systemowe

**Aplikacja nie uruchamia się (system Windows

):**

* Sprawdź, czy zainstalowany jest system Windows

10/11 (64-bitowy)
* Zaktualizuj sterowniki karty graficznej
* Sprawdź Podgląd zdarzeń systemu Windows

w celu uzyskania szczegółów błędu
* Skontaktuj się z pomocą techniczną, przesyłając logi błędów

**Interfejs CLI

nie uruchamia się (Linux

):**

* Sprawdź, czy pakiet `.deb` został poprawnie zainstalowany: `dpkg -l | grep chloros`
* Sprawdź uprawnienia: `sudo chmod +x /usr/bin/chloros-cli`
* Uruchom diagnostykę: `chloros-cli selftest`
* Sprawdź, czy nie brakuje bibliotek: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (od wersji 1.3.0 plik `/usr/lib/chloros/chloros-backend` jest niewielkim skryptem uruchamiającym; sam program znajduje się w pliku `/usr/lib/chloros/backend/`)

**Problemy z aktywacją licencji:**

* Upewnij się, że połączenie internetowe jest aktywne
* Zweryfikuj dane uwierzytelniające pod adresem [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Sprawdź, czy zapora sieciowa nie blokuje serwisu Chloros

* Szczegółowe instrukcje znajdziesz w sekcji [Chloros

+ Login](chloros+-login.md)

### Pomoc techniczna

Potrzebujesz pomocy przy instalacji lub konfiguracji?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Strona internetowa**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacja**: [Pierwsze kroki](./)
* ❓ **FAQ**: [Często zadawane pytania](faq.md)***

## Aktualizacje oprogramowania

Chloros

sprawdza dostępność aktualizacji, powiadamia o pojawieniu się nowej wersji i udostępnia link do tej strony pobierania — aktualizację przeprowadza się poprzez uruchomienie nowego, podpisanego instalatora. Twoje ustawienia, projekty i dane logowania pozostają nienaruszone po aktualizacji. W systemie Linux

i na platformie Jetson program `chloros-cli update` sprawdza dostępność nowszej wersji i proponuje pobranie oraz zainstalowanie odpowiedniej wersji `.deb` (polecenie to działa wyłącznie w systemie Linux

).

***

## Dziennik zmian**Wersja 1.3.0 (najnowsza, 11 listopada 2026 r.)**— pełną listę zmian można znaleźć w sekcji**Co nowego w Chloros

ie 1.3.0** na stronie [Pierwsze kroki](./).**Wersja 1.2.0**— pełną listę zmian można znaleźć w sekcji**Co nowego w Chloros

ie 1.2.0** na stronie [Pierwsze kroki](./).

<details>

<summary>Wersja 1.0.5</summary>

**Data wydania: 10 lutego 2026 r.**

**Nowe funkcje*** **Metoda debayeringu uwzględniająca teksturę \[Tylko Chloros

+] —** Metoda „Texture Aware” wykorzystuje wysokiej jakości algorytm debayeringu uwzględniający krawędzie w połączeniu z modelem odszumiania opartym na sztucznej inteligencji i uczeniu maszynowym, który usuwa niemal cały szum powstały podczas debayeringu.
* **Obsługa celów kalibracyjnych T4P*** **Szybsze przetwarzanie Chloros

+ na GPU, lepsze zarządzanie pamięcią**

**Poprawki błędów*** Całkowicie nowy interfejs użytkownika (GUI), który powinien teraz działać na wszystkich komputerach z systemem Windows

.

</details>

<details>

<summary>Wersja 1.0.4</summary>

**Data wydania: 5 stycznia 2026 r.**

**Nowe funkcje*** **Przełącznik obraz/metadane**: Dodano przełącznik w przeglądarce plików, umożliwiający wyświetlanie metadanych wybranego obrazu w tabeli zamiast w siatce obrazów
* **Suwak powiększenia siatki obrazów**: Nowy suwak w interfejsie użytkownika do regulacji rozmiaru miniatur (obsługuje również skrót klawiszowy CTRL + kółko myszy)
* **Przyciski eksportu siatki obrazów**: Przyciski w górnym rzędzie służące do przełączania miniatur z formatu JPG na przetworzone pliki eksportowe (Targets, Reflectance, Index, LUT)
* **Zakładka „Mapa”**: Nowa interaktywna mapa 2D pokazująca znaczniki lokalizacji GPS na obrazach
  * Obsługuje Mapy Google i kafelki mapowe ESRI (automatycznie wybiera najlepszą usługę kafelkową w zależności od dostępności poziomu powiększenia)
  * Podgląd miniatur po najechaniu kursorem myszy na znaczniki na mapie

**Poprawki błędów*** Ulepszona obsługa instalacji programu Chloros na komputerach z systemem operacyjnym w języku innym niż angielski

</details>

<details>

<summary>Wersja 1.0.3</summary>

**Data wydania: 20 grudnia 2025 r.**

**Nowe funkcje*** Pierwsze uruchomienie

**Ulepszenia*** Pierwsze uruchomienie

**Poprawki błędów*** Pierwsze uruchomienie

**Znane problemy*** Pierwsze uruchomienie

</details>

***

## Umowa licencyjna**Oprogramowanie zastrzeżone** – Prawa autorskie (c) 2026 MAPIR Inc.

Niedozwolone jest nieautoryzowane używanie, rozpowszechnianie lub modyfikowanie oprogramowania.

**Wersja bezpłatna**: Dostępna do użytku osobistego i komercyjnego z ograniczeniami funkcjonalności**Chloros+**: Licencja oparta na subskrypcji, zapewniająca dostęp do zaawansowanych funkcji i wdrożeń komercyjnych
