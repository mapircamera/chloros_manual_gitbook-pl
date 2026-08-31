# Profile nasadek i skalibrowany zakres

> Same nasadki — informacje o tym, która nasadka jest dostarczana z danym czujnikiem, sposób ich montażu oraz ich właściwości optyczne — zostały opisane w **[instrukcji obsługi DAQ](https://mapir.gitbook.io/daq)**. Ta strona dotyczy *zgłaszania* zamontowanej nasadki w Chloros, co zapewnia prawidłowość korekcji.

Każda fabryczna kalibracja radiometryczna czujnika światła DAQ dotyczy *samego* czujnika. Fizyczna nasadka zamontowana na dyfuzorze zmienia rodzaj światła zbieranego przez czujnik, dlatego Chloros nakłada **profil korekcji nasadki** zmierzony fabrycznie na pakiet kalibracyjny. Zadeklarowanie właściwej nasadki jest częścią procesu uzyskiwania skalibrowanych danych — ta strona opisuje, jakie nasadki są dostępne dla poszczególnych modeli, jak je zadeklarować oraz jaki jest faktyczny skalibrowany zakres spektralny czujnika.

## Dostępność nasadek w zależności od modelu

| Profil nasadki (`cap_id`) | Nasadka fizyczna | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Nasadka Sunshine z korektorem cosinusowym (**domyślna we wszystkich modelach**) | Tak | Tak | Tak |
| `fov_15` / `fov_45` / `fov_90` | Stożki ograniczające pole widzenia (15° / 45° / 90°) | Tak | — | Tak |
| `fov_30` / `fov_60` | Stożki ograniczające pole widzenia (30° / 60°) | Tak | — | — |
| `none` | Bez zamontowanej nasadki | — | — | Tak |

Uwagi dotyczące poszczególnych modeli:

* **Model DAQ-M ma jeden profil nasadki: `sunshine_cosine`.** „Bare-plus-Sunshine-cap” to definicja tego produktu, a model DAQ-M w wersji „bare” nie wymaga profilu geometrycznego.
* **Model DAQ-U w wersji „bare” jest prawdziwie „bare”** — nie wymaga żadnego profilu geometrycznego, dlatego nie istnieje dla niego profil `none`.
* **Profil `none` w urządzeniu DAQ-E NIE jest operacją bezczynną.** Wgłębiony, pokryty szkłem dyfuzor urządzenia DAQ-E posiada własną rzeczywistą korekcję geometryczną, więc „brak osłony” sam w sobie stanowi profil pomiarowy w tym modelu.
* **Samo czujnik DAQ-E bez osłony nie może mierzyć bezpośredniego nasłonecznienia przy żadnym kącie elewacji** — osłona Sunshine stanowi konfigurację terenową. Nie należy planować prac na zewnątrz z wykorzystaniem czujnika DAQ-E bez osłony.

W ustawieniach poszczególnych czujników w interfejsie graficznym (ikona koła zębatego w zakładce „Czujniki światła”) menu rozwijane **Osłona** oferuje również opcję „Brak (czujnik bez osłony)” w modelach DAQ-U i DAQ-M — w tych dwóch modelach „bez osłony” oznacza po prostu, że nie stosuje się korekcji osłony, zgodnie z powyższymi uwagami. Opcję tę należy wybrać tylko wtedy, gdy osłona została fizycznie zdjęta.

## Deklarowanie osłony — i dlaczego ma to znaczenie

**Deklarowany kod `cap_id` musi odpowiadać osłonie fizycznie zamontowanej na czujniku.** Ani czujnik, ani oprogramowanie nie są w stanie wykryć zamontowanej nasadki. Deklaracja ta ma wpływ na dwie rzeczy:

1. **Korekcję na żywo** stosowaną do każdego widma.
2. **Oznaczenie nasadki zapisywane w każdym zapisie `.daq`**, na którym opiera się dalsze przetwarzanie współczynnika odbicia.

Nasadka Sunshine zapewnia **12-krotne tłumienie zgodnie z projektem**, więc rejestracja z podaniem niewłaściwej nasadki powoduje błędną skalę widm o mniej więcej ten współczynnik. Należy natychmiast zgłaszać zmiany nasadki.

### Ustawianie nasadki

Interfejs graficzny: zakładka „Czujniki światła” → ikona koła zębatego w wierszu czujnika → menu rozwijane **Nasadka**. Domyślnym ustawieniem dla każdego modelu jest `sunshine_cosine` (wszystkie czujniki DAQ są dostarczane z zainstalowanym korektorem cosinusowym), a wybór ten pozostaje zachowany w ramach projektu.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (muszą działać serwery zaplecza):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Model CLI akceptuje składniowo pełną listę `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); każdy profil jest weryfikowany względem modelu czujnika w momencie połączenia, więc niedostępny identyfikator czujnika (na przykład identyfikator typu „E-only” na urządzeniu DAQ-U) powoduje wyraźny błąd zamiast nieprawidłowej korekty. Domyślnym ustawieniem backendu, gdy nie przekazano żadnych danych, jest `sunshine_cosine`.

Python SDK uwaga: `cap_id` to **nie** jest pokrętło typu SDK — `connect_daq_sensor()` / `DAQSensorSession` nie udostępniają parametru cap. Wybierz limit za pomocą powyższych poleceń CLI lub z listy rozwijanej w interfejsie graficznym; zobacz [Podręcznik SDK](../reference/sdk-reference.md).

Zaawansowane: profile są dostarczane w ramach instalacji Chloros w `daq/cap_profiles/<u|m|e>/<cap_id>.json` i mogą być nadpisywane indywidualnie dla każdego użytkownika w `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Niezależnie od limitów, czujniki, które nigdy nie były ponownie kalibrowane, automatycznie otrzymują niewielką korektę offsetu ciemnego opartą na danych z floty — bez konieczności wykonywania jakichkolwiek czynności przez użytkownika.

## Wydajność limitu nasłonecznienia (konfiguracja zewnętrzna)

Dane, na podstawie których można opracować procedury:

| Właściwość | Wartość |
| --- | --- |
| Pole widzenia | 180° półkuliste |
| Błąd odpowiedzi cosinusowej | ≤ ±4 % przy kącie padania do 60°; ≤ ±4,5 % przy kącie padania do 70° |
| Granica niskiego położenia słońca | Nie zalecane poniżej ~15° elewacji słońca |
| Tłumienie | ~12× (zgodnie z projektem) |
| Powtarzalność ponownego montażu osłony | ≈ 1,5 % |
| Ilościowe natężenie promieniowania | Średnia z odczytów trwających **≥ 15 s** (cecha charakterystyczna przyrządu, a nie wada) |

W przypadku każdej wartości ilościowego natężenia promieniowania — w tym odniesień do współczynnika odbicia — należy stosować średnią z odczytów trwających co najmniej 15 sekund, a nie pojedynczą klatkę.

## Skalibrowany zakres spektralny

| Właściwość | Wartość |
| --- | --- |
| Próbkowanie spektralne | 340–1010 nm w krokach co 5 nm (135 punktów) |
| Zakres skalibrowany radiometrycznie | **~374–974 nm** (wymagane przez oprogramowanie) |

Czujnik przekazuje pełną siatkę w zakresie 340–1010 nm, ale wzmocnienie radiometryczne zgodne z normami NIST obejmuje zakres ~374–974 nm. Chloros **odrzuca podział na odbicie bezwzględne** dla każdego pasma kamery, którego mniej niż połowa wagi spektralnej mieści się w tym zakresie, zgłaszając powód pominięcia `dls-uncalibrated-band-<nm>` zamiast generować produkt nieskalibrowany. Spośród dostępnych w sprzedaży modeli kamer tylko filtr F988 wykracza poza ten zakres; w jego przypadku stosowany jest zamiast tego proces oparty na panelu odbicia — zobacz [Procesy oparte na odbiciu](reflectance.md).

Informacje na temat modeli czujników, transportów i identyfikatorów czujników można znaleźć w [przeglądzie DAQ](README.md). Informacje na temat sposobu wykorzystania limitu podczas przetwarzania można znaleźć w sekcji [Rejestracja i format .daq](recording.md).
