# Korzystanie z Chloros przy pomocy asystentów AI

Niniejsza instrukcja jest przeznaczona dla dwóch grup odbiorców: ludzi oraz asystentów AI, z których ludzie coraz częściej korzystają w pracy. Na każdej stronie podano dokładne wartości, ustawienia domyślne oraz polecenia, które można skopiować i wkleić, dzięki czemu asystent (Claude, ChatGPT, Copilot, agent programistyczny itp.) może od razu stworzyć działającą automatyzację Chloros.

Wersja Chloros: **

1.2.0**. Platformy CLI/SDK: Windows 10/11 x64 oraz Linux (x86_64 / Jetson aarch64).

## Co przekazać asystentowi

| Zasób | URL | Do czego służy |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Indeks wszystkich stron niniejszej instrukcji w formacie nadającym się do odczytu maszynowego. |
| **CLI Reference** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Kompletny zestaw poleceń `chloros-cli`: wszystkie polecenia, flagi, wartości domyślne, kody zakończenia oraz reguły dotyczące folderów wyjściowych. Opracowane z myślą o modelach LLM. |
| **SDK — Informacje o poleceniach** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Kompletny `chloros_sdk` Python API: klasy, sygnatury, wyjątki i przykłady z rozwiązaniami. Napisane z myślą o LLM. |
| **Dowolna strona w postaci surowego Markdownu** | dodaj `.md` do strony URL | np. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` zwraca stronę w postaci surowego Markdownu — idealne do wklejenia do okna kontekstowego lub pobrania z agenta. |

Linki w podręczniku: [CLI Odnośnik](reference/cli-reference.md) · [SDK Dokumentacja](reference/sdk-reference.md).

{% hint style="info" %}
Te dwie strony referencyjne są samowystarczalne: asystent, który zapoznał się z jedną z nich, nie potrzebuje reszty podręcznika, aby napisać poprawny skrypt.
{% endhint %}

## Gotowe skrypty

Skopiuj, uzupełnij `<placeholders>` i wklej do swojego asystenta.

### 1. Przetworzenie folderu lotu do NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Obserwacja w trybie wsadowym katalogu z danymi przechwyconymi

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Podłączenie matrycy LATTICE i przechwycenie danych

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Rejestruj widma z czujnika światła DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skrypty DAQ z wiersza poleceń zawsze przechodzą przez rodzinę `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Inne podpolecenia `daq`, które może wymyślić Twój asystent, nie są dostępne w dostarczonych kompilacjach i powodują zakończenie działania z błędem.
{% endhint %}

## Dlaczego skrypty napisane przez sztuczną inteligencję dobrze współpracują z Chloros

Każde z nich to rzeczywiste, zweryfikowane zachowanie Chloros 1.2.0 — eliminują one klasyczne przyczyny niepowodzeń automatyzacji tworzonej przez maszyny:

* **Brak skomplikowanych przygotowań.**Inteligentne pomocniki połączeń SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) oraz punkty wejścia przetwarzania (`ChlorosLocal`, `process_folder`)**automatycznie uruchamiają lokalny backend**. Wygenerowany skrypt nie wymaga otwartego interfejsu graficznego ani ręcznie uruchomionego serwera — potrzebuje jedynie zainstalowanego pakietu desktop/CLI.
* **Cały potok to jedno wywołanie.** `chloros_sdk.process_folder("path", indices=["NDVI"])` wykonuje import → kalibrację → pomiar odbicia → eksport indeksu od początku do końca. Mniejsza powierzchnia działania, mniej miejsc, w których wygenerowany skrypt może się nie powieść.
* **Przebiegi bez wyników przeprowadzają autodiagnostykę.** Po zakończeniu działania `process()` do wyniku dołączane jest podsumowanie przebiegu, a każda wskazówka dotycząca przetwarzania (np. *dlaczego* przebieg nie dał żadnych wyników) jest również ponownie wysyłana jako Python `UserWarning` — dzięki czemu nawet skrypt, który nigdy nie sprawdza słownika wyników, wyświetla diagnozę.
* **Pobieg CLI kończy się wyraźnym błędem.**Przebieg `chloros-cli process`, który zażądał wyników, ale żadnego nie wygenerował, wyświetla `Processing finished but wrote no image products.` i**kończy się z kodem wyjścia innym niż zero**, więc skrypty powłoki i CI wykrywają to poprzez zwykłą kontrolę kodu wyjścia. Pomyślne uruchomienia zgłaszają kod `Image products written: N`.

Jedna asymetria, o której asystent powinien wiedzieć: `process()` w SDK celowo **nie** generuje wyjątku w przypadku uruchomienia bez produktów — zamiast tego zgłasza to poprzez podsumowanie/wskazówki. Jeśli potok Python musi się zatrzymać w przypadku pustego przebiegu, sprawdź podsumowanie (tak jak robi to przepis nr 2).

## Uwagi

* **Wymagane logowanie Chloros+.**Usługi CLI i SDK wymagają**płatnego** poziomu Chloros+, egzekwowanego po stronie serwera: żądania kończą się niepowodzeniem z kodem `401 AUTH_REQUIRED` w przypadku braku logowania oraz z kodem `403 PLAN_UPGRADE_REQUIRED` na bezpłatnym poziomie. Przed uruchomieniem wygenerowanych skryptów należy raz na każdym komputerze uruchomić `chloros-cli login`. Zobacz [Chloros+ Logowanie](chloros+-login.md).
* **Polecenia przechwytywania sterują rzeczywistym sprzętem.** Polecenia `lattice` / `daq` / `project` oraz obiekty sesji SDK łączą się z fizycznymi kamerami i czujnikami, przesyłają strumień danych z nich oraz uruchamiają je. Przed pierwszym uruchomieniem należy przejrzeć wygenerowany skrypt i uruchomić go w obecności operatora sprzętu.
* **Wykonaj wyrywkową kontrolę wyników.** Przed opublikowaniem wyników sprawdź foldery produktów i kilka wartości pikseli. W szczególności pliki TIFF odzwierciedlenia są skalowane według źródła — zapoznaj się z tagiem XMP `Chloros:PixelScale` (LATTICE: 32768 = współczynnik odbicia 1,0; Survey3: 65535) zamiast zakładać dzielnik. Obie strony referencyjne opisują to w sekcji „Odczytywanie pikseli współczynnika odbicia”.
* **Drobne pułapki, które mogą zakłócić działanie wygenerowanego kodu:**`pool-record` zapisuje dane w systemie plików**hosta zaplecza** (domyślnie `~/Documents/DAQ Live View/`); na komputerach z kilkoma interfejsami sieciowymi należy preferować `daq pool-connect --eth-host <ip-or-hostname>` zamiast automatycznego wykrywania; oraz należy używać `http://127.0.0.1:5000` (nigdy `localhost`) wszędzie tam, gdzie pojawia się backend URL.
