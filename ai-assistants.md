# Korzystanie z Chlorosa z asystentami AI

Niniejszy podręcznik jest przeznaczony dla dwóch grup odbiorców: ludzi oraz asystentów AI, z których ludzie coraz częściej korzystają. Każda strona zawiera dokładne wartości, ustawienia domyślne oraz polecenia, które można skopiować i wkleić, dzięki czemu asystent (Claude, ChatGPT, Copilot, agent programistyczny itp.) może od razu napisać działającą automatyzację w Chloros.

Wersja Chloros: **

1.3.0**. Platformy CLI/SDK: Windows 10/11 x64 oraz Linux (x86_64 / Jetson aarch64).

## Co przekazać asystentowi

| Zasób | URL | Do czego służy |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Indeks każdej strony niniejszej instrukcji w formacie nadającym się do odczytu maszynowego. |
| **Podręcznik CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Kompletny zestaw poleceń `chloros-cli`: wszystkie polecenia, flagi, wartości domyślne, kody wyjścia i reguły dotyczące folderów wyjściowych. Opracowany z myślą o wykorzystaniu przez LLM. |
| **Dokumentacja SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Kompletny Python API `chloros_sdk`: klasy, sygnatury, wyjątki i przykłady z rozwiązaniami. Opracowano z myślą o modelach LLM. |
| **Dowolna strona w formacie surowego Markdown** | dodaj `.md` do URLu strony | np. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` zwraca stronę w formacie surowego Markdown — idealne do wklejenia do okna kontekstowego lub pobrania z agenta. |

Linki w podręczniku: [Podręcznik CLI](reference/cli-reference.md) · [Podręcznik SDK](reference/sdk-reference.md).

{% hint style="info" %}
Obie strony referencyjne są samowystarczalne: asystent, który zapoznał się z jedną z nich, nie potrzebuje pozostałej części podręcznika, aby napisać poprawny skrypt.
{% endhint %}

## Pozwól asystentowi uruchamiać bezpośrednio program Chloros: `chloros-mcp`

Wszystkie powyższe przykłady wymagają od asystenta *napisania* skryptu lub polecenia. Dzięki `chloros-mcp` asystent obsługujący [Model Context Protocol](https://modelcontextprotocol.io) (MCP), taki jak Claude Desktop, Claude Code lub Cursor, sam steruje programem Chloros. Wystarczy poprosić prostym językiem, a asystent wywoła narzędzia serwera:

* **Przetwarzanie obrazów:** utworzenie projektu, import folderu z ujęciami, wybór ustawień, przetwarzanie oraz generowanie raportu z wyników operacji.
* **Obsługa sprzętu:** podłącza kamery LATTICE, matryce i czujniki światła DAQ, wyświetla podgląd, ustawia ekspozycję i kolory, wykonuje zdjęcia, nagrywa oraz aktualizuje oprogramowanie sprzętowe.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Narzędzia serwera obejmują funkcje [Python SDK](api-python-sdk.md). Instalują się one wraz z SDK, więc zazwyczaj nie trzeba instalować niczego więcej: w systemie Windows instalator Chloros umieszcza `chloros-mcp.exe` w folderze `Scripts` środowiska Python, a w systemie Linuxa `.deb` umieszcza plik `chloros-mcp` w katalogu `/usr/bin`.

**Kod Claude'a:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**dodaj to do pliku `claude_desktop_config.json` (można go otworzyć, wybierając**Ustawienia > Programista > Edytuj konfigurację**), a następnie uruchom ponownie Claude Desktop.**Kursor** przyjmuje ten sam wpis w pliku `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Jeśli klient zgłasza, że serwer nie uruchomił się, prawdopodobnie `chloros-mcp` nie znajduje się w `PATH`, który widzi: podaj zamiast tego pełną ścieżkę (`where chloros-mcp` w systemie Windows, `which chloros-mcp` w systemie Linux).

Konieczne jest zainstalowanie Chloros na tym samym komputerze oraz zalogowanie się na nim – w aplikacji lub za pomocą `chloros-cli login` – na dowolnym płatnym planie Chloros+ (Copper lub wyższym). Serwer nie może zalogować się w Twoim imieniu.

{% hint style="info" %}
Konfiguracja klienta dla każdej aplikacji, opcji, każdego narzędzia oraz sposób działania procesu: [Serwer MCP (asystenci AI)](reference/sdk-reference.md#mcp-server-ai-assistants) w dokumentacji SDK.
{% endhint %}

## Przykłady poleceń

Skopiuj, uzupełnij `<placeholders>` i wklej do swojego asystenta.

### 1. Przetwórz folder lotu na NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Obserwacja zbiorcza katalogu z danymi przechwyconymi

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Podłączenie matrycy LATTICE i przechwytywanie danych

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Rejestrowanie widm czujników światła DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skrypty DAQ z wiersza poleceń zawsze przechodzą przez rodzinę `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Inne podpolecenia typu `daq`, które może wymyślić asystent, nie są dostępne w dostarczonych kompilacjach i powodują zakończenie działania z błędem.
{% endhint %}

## Dlaczego skrypty napisane przez sztuczną inteligencję dobrze współpracują z Chloros

Każda z tych cech to rzeczywiste, zweryfikowane zachowanie Chloros 1.3.0 — eliminują one klasyczne przyczyny niepowodzeń automatyzacji tworzonej przez maszyny:

* **Brak skomplikowanych czynności konfiguracyjnych.**Funkcje pomocnicze SDK do inteligentnego łączenia (`connect_camera`, `connect_array`, `connect_daq_sensor`) oraz punkty wejścia przetwarzania (`ChlorosLocal`, `process_folder`)**automatycznie uruchamiają lokalny backend**. Wygenerowany skrypt nie wymaga otwartego interfejsu graficznego ani ręcznie uruchomionego serwera — wystarczy zainstalowany pakiet na pulpit lub CLI.
* **Cały potok to jedno wywołanie.** `chloros_sdk.process_folder("path", indices=["NDVI"])` wykonuje import → kalibrację → pomiar odbicia → eksport indeksu od początku do końca. Mniejsza powierzchnia działania, mniej miejsc, w których wygenerowany skrypt może zawieść.
* **Przebiegi bez wyników poddają się samodiagnozie.** Po zakończeniu działania `process()` podsumowanie przebiegu jest dołączane do wyniku, a każda wskazówka dotycząca przetwarzania (np. *dlaczego* przebieg nie wygenerował wyników) jest ponownie wysyłana jako skrypt w języku Pythona `UserWarning` — dzięki czemu nawet skrypt, który nigdy nie sprawdza słownika wyników, wyświetla diagnozę.
* **CLI* zgłasza wyraźny błąd.**Przebieg `chloros-cli process`, który zażądał wyników, ale nie zapisał żadnego, wyświetla `Processing finished but wrote no image products.` i**kończy się wynikiem niezerowym**, dzięki czemu skrypty powłoki i CI wykrywają to poprzez zwykłą kontrolę kodu wyjścia. Pomyślne uruchomienia zgłaszają kod `Image products written: N`, a także `(plus M clamp masks)`, gdy zapisano maski. Polecenia przechwytujące również podają nazwy wszystkich plików, których nie udało się zapisać, i kończą działanie z kodem `1`.
* **Ścieżki względne mają znaczenie zgodne z oczekiwaniami skryptu.** W przypadku lokalnego backendu względna ścieżka wyjściowa (`-o out/` w interfejsie CLI, `output_dir="out/"` w SDK) jest rozpoznawana względem folderu, z którego uruchamiane jest polecenie lub skrypt.

Jedna asymetria, o której asystent powinien wiedzieć: `process()` w SDK celowo **nie** generuje wyjątku w przypadku uruchomienia bez wyników — zamiast tego zgłasza to w podsumowaniu/wskazówkach. Jeśli potok w języku Pythona musi się zatrzymać w przypadku pustego przebiegu, sprawdź podsumowanie (tak jak w przepisie 2).

## Zastrzeżenia

* **Wymagane logowanie do Chloros+.**CLI i SDK wymagają**płatnego** planu Chloros+, co jest egzekwowane po stronie serwera: żądania kończą się niepowodzeniem z kodem błędu `401 AUTH_REQUIRED` w przypadku braku logowania oraz `403 PLAN_UPGRADE_REQUIRED` na planie darmowym. Przed uruchomieniem wygenerowanych skryptów należy raz na każdej maszynie uruchomić polecenie `chloros-cli login`. Zobacz [Logowanie do Chloros+](chloros+-login.md).
* **Polecenia przechwytywania sterują rzeczywistym sprzętem.** Polecenia `lattice` / `daq` / `project`, obiekty sesji SDK oraz narzędzia `chloros-mcp` łączą się z fizycznymi kamerami i czujnikami, przesyłają strumieniowo dane z nich oraz uruchamiają je. Przed pierwszym uruchomieniem należy przejrzeć wygenerowany skrypt i uruchomić go w obecności operatora sprzętu.
* **Przeprowadź wyrywkową kontrolę wyników.** Przed opublikowaniem wyników sprawdź foldery produktów oraz kilka wartości pikseli. W szczególności pliki TIFF z danymi odbicia są skalowane według źródła — należy zapoznać się z tagiem XMP `Chloros:PixelScale` (pliki LATTICE zawierają ten tag: 32768 = 1,0 odbicia; pliki Survey3 go nie zawierają i stosują 65535 = 1,0) zamiast zakładać dzielnik. Obie strony referencyjne dokumentują to w sekcji „Odczytywanie pikseli odbicia”.
* **Drobne pułapki, które mogą zakłócić działanie wygenerowanego kodu:**`pool-record` zapisuje dane w systemie plików**hosta zaplecza** (domyślnie `~/Documents/DAQ Live View/`); na komputerach z kilkoma interfejsami sieciowymi lepiej wybrać `daq pool-connect --eth-host <ip-or-hostname>` zamiast automatycznego wykrywania; używaj `http://127.0.0.1:5000` (nigdy `localhost`) wszędzie tam, gdzie pojawia się URL serwera zaplecza; skrypt przeglądający drzewo wyników projektu powinien pomijać foldery `clamp_masks/` (znajdujące się w nich pliki `*_clampmask.tif` to maski, a nie obrazy — zobacz [Formaty obrazów wyjściowych](output-image-formats.md#clamp-masks)); oraz w systemie Windows, gdzie wyłączono obsługę długich ścieżek, należy stosować krótkie nazwy folderów projektowych — folder projektu o nazwie dłuższej niż około 130 znaków zostanie odrzucony przed rozpoczęciem przetwarzania.
