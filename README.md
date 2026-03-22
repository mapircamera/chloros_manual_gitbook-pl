---
metaLinks: {}
---

# Pierwsze kroki

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros to aplikacja firmy [MAPIR](https://www.mapir.camera) służąca do przetwarzania obrazów i innych danych z czujników.

***{% hint style="success" %}**Co nowego w Chloros 1.1.0**: Natywna obsługa Linux (amd64 i arm64), przetwarzanie brzegowe NVIDIA Jetson, dynamiczna adaptacja obliczeniowa, 4-wątkowy potok przetwarzania, nowe polecenia i opcje CLI. Pełną listę zmian można znaleźć w sekcji [Pobierz](download.md).
{% endhint %}

Chloros jest dostępny w 3 trybach aplikacji:

## Chloros: Aplikacja GUI na pulpit

Samodzielne, oddzielne okno z wszystkimi funkcjami. _Tylko dla systemu Windows._

## [Chloros CLI: Interfejs wiersza poleceń](CLI.md)

Przetwarzanie wsadowe z wiersza poleceń. Idealne do automatyzacji, tworzenia skryptów i pracy bez interfejsu graficznego. Dostępne na **Windows, Linux amd64 oraz Linux arm64 (NVIDIA Jetson)**. _Dostęp do CLI wymaga licencji Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Programowy interfejs Python do automatyzacji i niestandardowych przepływów pracy. Idealny do procesów badawczych, integracji z istniejącymi aplikacjami Python oraz tworzenia niestandardowych narzędzi. Dostępny na **wszystkich platformach** za pośrednictwem `pip install chloros-sdk`. _Dostęp do API wymaga licencji Chloros+._***

## Obsługiwane platformy

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Tak | Tak | Tak |
| **Linux amd64 (x86_64)** | Nie | Tak | Tak |
| **Linux arm64 (NVIDIA Jetson)** | Nie | Tak | Tak |

Instrukcje dotyczące instalacji Linux można znaleźć w sekcji [Linux i przetwarzanie brzegowe](linux/linux-overview.md).

***

## Chloros+

Chociaż Chloros jest darmowy do większości zadań, może się okazać, że potrzebujesz czegoś więcej. Wtedy przyda Ci się płatna licencja na Chloros+. Dzięki licencji Chloros+ możesz odblokować nowe funkcje, takie jak:

* **Przetwarzanie wielowątkowe**: znacznie przyspiesz przetwarzanie obrazów w większych projektach poprzez jednoczesne przetwarzanie obrazów w potoku.
* **Przyspieszenie GPU (CUDA)**: wykorzystaj dzisiejsze opcje większej pamięci GPU, aby jeszcze bardziej przyspieszyć potok przetwarzania obrazów. Aby uzyskać najlepsze wyniki, zalecamy 4 GB lub więcej pamięci VRAM.
* **Chloros+**[**CLI**](CLI.md)**Dostęp**: uruchom Chloros+ z wiersza poleceń, aby zautomatyzować i zintegrować go z własnym oprogramowaniem.
* **Chloros+**[**API**](api-python-sdk.md)**Dostęp:** uruchom Chloros+ z poziomu Python w celu sterowania programowego, umożliwiającego płynną integrację z procesami badawczymi, przepływami pracy związanymi z analizą danych oraz niestandardowymi aplikacjami.
* **Korzystanie z wielu urządzeń**: każda licencja Chloros+ pozwala na zarejestrowanie 2 lub więcej urządzeń. Użyj swojego konta MAPIR Cloud do zarządzania zarejestrowanymi urządzeniami. Dodaj obsługę większej liczby urządzeń, aktualizując swoją licencję Chloros+.
* **Zaawansowana metoda debayeringu uwzględniająca tekstury:** wysokiej jakości debayering uwzględniający krawędzie w połączeniu z modelem usuwania szumu opartym na AI/ML, który eliminuje niemal cały szum debayeringu. 
* **Niestandardowe formuły indeksów wielospektralnych:** wprowadzaj niestandardowe indeksy wielospektralne w kalkulatorach rastrowych Chloros, zarówno do przetwarzania, jak i do środowiska testowego przeglądania obrazów.
* **Linux i przetwarzanie brzegowe:** uruchom Chloros na platformach Linux x86\_64 i ARM64, w tym na NVIDIA Jetson, do przetwarzania w terenie i na obrzeżach sieci. Zobacz [Przegląd Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Ceny i rejestracja w usłudze Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
