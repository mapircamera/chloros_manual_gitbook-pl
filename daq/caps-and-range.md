# Profile osłon i skalibrowany zakres

> Same nasadki — które z nich są dostarczane z danym czujnikiem, jak się je montuje oraz jakie mają właściwości optyczne — zostały opisane w **[instrukcji obsługi DAQ](https://mapir.gitbook.io/daq)**. Ta strona dotyczy *zgłaszania* zamontowanej nasadki do systemu Chloros
, co jest niezbędne do prawidłowego przeprowadzenia korekcji.

Każda fabryczna kalibracja radiometryczna czujnika światła DAQ opisuje *sam* czujnik. Fizyczna osłona zamontowana na dyfuzorze zmienia rodzaj światła zbieranego przez czujnik, dlatego Chloros
stosuje zmierzony fabrycznie **profil korekcji osłony** jako dodatek do pakietu kalibracyjnego. Zgłoszenie właściwej nasadki jest częścią procesu uzyskiwania skalibrowanych danych — na tej stronie omówiono, jakie nasadki są dostępne dla poszczególnych modeli, jak je zgłaszać oraz jaki jest faktyczny skalibrowany zakres spektralny czujnika.

## Dostępność nasadek w zależności od modelu

| Profil nasadki (`cap_id`) | Nasadka fizyczna | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Nasadka Sunshine z korektorem cosinusowym (**domyślna w każdym modelu**) | Tak | Tak | Tak |
| `fov_15` / `fov_45` / `fov_90` | Stożki ograniczające pole widzenia (15° / 45° / 90°) | Tak | — | Tak |
| `fov_30` / `fov_60` | Stożki ograniczające pole widzenia (30° / 60°) | Tak | — | — |
| `none` | Bez zamontowanej nasadki | — | — | Tak |

Uwagi dotyczące poszczególnych modeli:

* **DAQ-M ma jeden profil nasadki: `sunshine_cosine`.** „Bare-plus-Sunshine-cap” to definicja tego produktu, a model DAQ-M w wersji podstawowej nie wymaga profilu geometrycznego.
* **Model DAQ-U w wersji podstawowej jest całkowicie „bare”** — nie wymaga żadnego profilu geometrycznego, dlatego nie istnieje dla niego profil `none`.
* **`none` na modelu DAQ-E NIE jest operacją bezczynną.** Wgłębiony, pokryty szkłem dyfuzor modelu DAQ-E posiada własną, rzeczywistą korekcję geometryczną, więc „brak nasadki” sam w sobie stanowi profil pomiarowy w tym modelu.
* **Sam czujnik DAQ-E bez osłony nie może mierzyć bezpośredniego nasłonecznienia przy żadnym kącie elewacji** — osłona Sunshine stanowi konfigurację terenową. Nie należy planować prac na zewnątrz z wykorzystaniem czujnika DAQ-E bez osłony.

W ustawieniach poszczególnych czujników w interfejsie graficznym (ikona koła zębatego w zakładce „Czujniki światła”) lista rozwijana **Osłona** oferuje również opcję „Brak (nagie urządzenie)” w modelach DAQ-U i DAQ-M — w tych dwóch modelach „nagie urządzenie” oznacza po prostu, że nie stosuje się korekcji osłony, zgodnie z powyższymi uwagami. Opcję tę należy wybrać tylko wtedy, gdy osłona została fizycznie zdjęta.

## Deklarowanie osłony — i dlaczego ma to znaczenie

**Deklarowany kod `cap_id` musi odpowiadać osłonie fizycznie zamontowanej na czujniku.** Ani czujnik, ani oprogramowanie nie są w stanie wykryć zamontowanej nasadki. Deklaracja ta ma wpływ na dwie rzeczy:

1. **Korekcję na żywo** stosowaną do każdego widma.
2. **Oznaczenie nasadki zapisywane w każdym zapisie `.daq`**, na którym opiera się dalsze przetwarzanie współczynnika odbicia.

Nasadka Sunshine zapewnia **12-krotne tłumienie zgodnie z projektem**, więc rejestracja z zadeklarowaną niewłaściwą nasadką powoduje nieprawidłowe skalowanie widm mniej więcej o ten współczynnik. Należy natychmiast zgłaszać zmiany nasadki.

### Ustawianie nasadki

Interfejs graficzny: zakładka „Czujniki światła” → ikona koła zębatego w wierszu czujnika → menu rozwijane **Nasadka**. Domyślnym ustawieniem dla każdego modelu jest `sunshine_cosine` – jest to wartość domyślna (modele DAQ-U, DAQ-M i DAQ-E są dostarczane bez nasadki, a zapis wskazuje, że nasadka została założona domyślnie, dopóki nie wybierzesz konkretnej), a wybór ten pozostaje zachowany w projekcie.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend musi być uruchomiony):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
akceptuje składniowo pełną listę `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); każdy profil jest weryfikowany względem modelu czujnika w momencie połączenia, więc niedostępny identyfikator czujnika (na przykład identyfikator przeznaczony wyłącznie dla modelu E w przypadku DAQ-U) powoduje wyraźny błąd zamiast nieprawidłowej korekty. Domyślną wartością serwera, gdy nie przekazano żadnych danych, jest `sunshine_cosine`.

Python
SDK
: nadaj nazwę czujnika w momencie nawiązywania połączenia za pomocą `connect_daq_sensor(cap_id="sunshine_cosine")` lub zmień ją w czasie wykonywania za pomocą `DAQSensorSession.set_cap("fov_45")`; zobacz [Dokumentację SDK
](../reference/sdk-reference.md#daq-sensor-sessions).

Zaawansowane: profile są dostarczane w ramach instalacji Chloros
pod nazwą `daq/cap_profiles/<u|m|e>/<cap_id>.json` i mogą być nadpisywane indywidualnie dla każdego użytkownika za pomocą `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Niezależnie od limitów, czujniki, które nigdy nie były ponownie kalibrowane, automatycznie otrzymują niewielką korektę ciemnego przesunięcia wywodzącą się z floty — bez konieczności podejmowania jakichkolwiek działań przez użytkownika.

## DAQ-E-S: stały dyfuzor i nachylenie

**DAQ-E-S** to urządzenie typu DAQ-E z dwiema różnicami, które mają znaczenie dla Chloros
. Wszystko inne (wykrywanie, łączenie, PTP, rejestrowanie, ustawienia urządzenia) działa dokładnie tak samo jak w przypadku DAQ-E.

* **Jego dyfuzor jest stały, więc nie wymaga korekcji za pomocą nasadki.** Dyfuzor znajdował się na urządzeniu podczas pomiaru jego kalibracji fabrycznej, więc kalibracja ta już odzwierciedla rzeczywiste natężenie promieniowania padającego w dół. Chloros
nie stosuje do niego żadnego profilu ograniczenia, niezależnie od tego, o co zostanie poproszony: profil o tej nazwie jest ignorowany z wyświetleniem komunikatu, jego zapisy wskazują, że ograniczenie pochodzi z modelu, a ostrzeżenie o założonym ograniczeniu nigdy się w tym przypadku nie uruchamia. W sekcji Ustawienia projektu → Czujnik światła DAQ w jego wierszu nie ma żadnego ograniczenia, a ogólne dla projektu nadpisanie ograniczenia nie ma na niego wpływu. Nie należy ręcznie nakładać ograniczenia na jego dane: korekcja nasłonecznienia na wierzchu wskazywałaby wartość około 11 razy wyższą.
* **Urządzenie jest wyposażone w 3-osiowy akcelerometr.** Charakterystyka cosinusowa dyfuzora jest ważna tylko wtedy, gdy jest on wypoziomowany, więc przechylony czujnik zgłasza błędne natężenie promieniowania bez żadnych innych objawów. Każdy odczyt z DAQ-E-S zawiera zatem informacje o jego przechyleniu, obrocie i pochyleniu, a plik `.daq` zarejestrowany z jednego z nich przechowuje je obok każdego widma (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), dzięki czemu można odróżnić chmurę od uderzenia w uchwyt.

Informacje na temat każdego pola położenia oraz sposobu zgłaszania błędów w strumieniu danych położenia można znaleźć w [Dokumentacji CLI
dotyczącej DAQ-E-S](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Wydajność osłony przeciwsłonecznej (konfiguracja zewnętrzna)

Wartości, na podstawie których można opracować procedury:

| Właściwość | Wartość |
| --- | --- |
| Pole widzenia | 180° półkuliste |
| Błąd odpowiedzi cosinusowej | ≤ ±4 % przy kącie padania do 60°; ≤ ±4,5 % przy kącie padania do 70° |
| Granica dla niskiego położenia słońca | Niezalecane poniżej ~15° elewacji słońca |
| Tłumienie | ~12× (zgodnie z projektem) |
| Powtarzalność ponownego montażu nasadki | ≈ 1,5 % |
| Ilościowe natężenie promieniowania | Średnia z **≥ 15 s** odczytów (cecha charakterystyczna przyrządu, nie jest to wada) |

W przypadku każdej wartości ilościowego natężenia promieniowania — w tym odniesień do współczynnika odbicia — należy stosować średnią z co najmniej 15 sekund odczytów, a nie pojedynczą klatkę.

## Skalibrowany zakres spektralny

| Właściwość | Wartość |
| --- | --- |
| Próbkowanie spektralne | 340–1010 nm w krokach co 5 nm (135 punktów) |
| Zakres skalibrowany radiometrycznie | **~374–974 nm** (wymuszony programowo) |

Czujnik przekazuje pełną siatkę w zakresie 340–1010 nm, ale wzmocnienie radiometryczne zgodne z normami NIST obejmuje zakres ~374–974 nm. Chloros
**odrzuca podział na odbicie bezwzględne** dla każdego pasma kamery, którego mniej niż połowa wagi spektralnej mieści się w tym zakresie, zgłaszając powód pominięcia `dls-uncalibrated-band-<nm>` zamiast generować produkt nieskalibrowany. Spośród dostępnych w sprzedaży modeli kamer tylko filtr F988 wykracza poza ten zakres; w jego przypadku stosowany jest zamiast tego proces oparty na panelu odbicia — zobacz [Procesy oparte na odbiciu](reflectance.md).

Informacje na temat modeli czujników, transportów i identyfikatorów czujników można znaleźć w [przeglądzie DAQ](README.md). Informacje na temat sposobu wykorzystania limitu podczas przetwarzania można znaleźć w sekcji [Rejestracja i format .daq](recording.md).
