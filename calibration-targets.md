---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cele kalibracyjne Firma

MAPIR oferuje różnorodne cele kalibracyjne przeznaczone do szerokiego zakresu zastosowań. Przedstawiony poniżej kompaktowy model T4-R50 zawiera 4 panele, dla których zmierzono współczynnik odbicia światła w zakresie od 250 do 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Cele referencyjne rozproszone T4 charakteryzują się następującymi krzywymi współczynnika odbicia, [dane do pobrania tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>Odbicie światła MAPIR T4 :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>Odbicie światła MAPIRa T4 :: 400–1000 nm</p></figcaption></figure>Cele referencyjne rozproszone T4P mają następujące krzywe odbicia światła, [dane do pobrania tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>Odbicie MAPIRa T4P :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>Odbicie MAPIRa T4P :: 400–1000 nm</p></figcaption></figure>Patrząc na wykres współczynnika odbicia, widać, że wartości przedstawiają zależność między długością fali (oś x) a procentowym współczynnikiem odbicia (oś y). Po zarejestrowaniu obrazu wzorca kalibracyjnego tworzymy zależność między wartością piksela a procentowym współczynnikiem odbicia w zakresie widma, na które wrażliwe są poszczególne pasma czujnika kamery.

Oznacza to, że do każdego zdjęcia wykonanego naszymi kamerami można wykorzystać zdjęcie naszych wzorców odbicia, takich jak [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) lub [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), do kalibracji obrazów pod kątem współczynnika odbicia. Po kalibracji każdy piksel na obrazie odpowiada procentowemu współczynnikowi odbicia.

W przypadku wyników z programu **Survey3**, jeśli wyeksportujesz skalibrowane zdjęcia z programu Chloros

w typowym formacie JPG lub TIFF

, procent odbicia zostanie obliczony poprzez podzielenie wartości piksela przez głębię bitową formatu obrazu. W przypadku formatu JPG należy podzielić przez 255, a w przypadku formatu TIFF

– przez 65 535. Można również wybrać w programie Chloros

format wyjściowy PERCENT, a wtedy każdy piksel będzie miał wartość procentową w zakresie od 0,0 do 1,0 (od 0% do 100% współczynnika odbicia). Należy jednak pamiętać, że niektóre aplikacje graficzne nie obsługują obrazów w formacie procentowym (z ułamkami dziesiętnymi), a ponadto zajmują one dużo miejsca na dysku.

{% hint style="info" %}
**Odbicie LATTICE wykorzystuje inną skalę pikselową.** Odbicie LATTICE jest zapisywane przy DN 32768 = 100% odbicia (a nie 65535), a każdy plik zawiera tag XMP `Chloros:PixelScale` określający jego skalę. Należy odczytać ten tag i podzielić przez tę wartość, zamiast zakładać stałą wartość — zobacz [Formaty obrazów wyjściowych](output-image-formats.md).
{% endhint %}

## Cele kalibracyjne z kamerami LATTICE

W przypadku kamer LATTICE cel kalibracyjny jest **opcjonalny** dla współczynnika odbicia: program Chloros

może zamiast tego odnosić współczynnik odbicia do natężenia promieniowania padającego mierzonego przez czujnik światła DAQ (ρ = π·L/E). Odniesienie wybiera się za pomocą ustawienia źródła odbicia w projekcie, konfigurowanego za pomocą `chloros-cli process --reflectance-source` (interfejs graficzny nie posiada odpowiedniej opcji i wykorzystuje wartość zapisaną w projekcie, domyślnie `auto` — zobacz [Ustawienia projektu](project-settings/project-settings.md)):

| Wartość | Zachowanie |
| --- | --- |
| `auto` *(domyślnie)* | Cel w ramce, który przeszedł kontrolę jakości (QA), stanowi **odniesienie absolutne**; gdy nie ma żadnego celu lub kontrola jakości zakończyła się niepowodzeniem, Chloros

przechodzi na podział w dół strumienia danych DAQ. Cel, który przeszedł kontrolę jakości z flagą (upadły panel lub pasmo poza zakresem), nie jest automatycznie preferowany: łańcuchy typu „cel”, „cel+DAQ” oraz „tylko DAQ” są testowane w kolejności zgodnej z ich wynikiem zmierzonym na panelach testowych. |
| `target` | Wyłącznie cel — bez zastępowania przez DAQ. |
| `daq` | DAQ jako źródło ostateczne — pomiar w kierunku w dół zawsze stanowi punkt odniesienia. |

Dodatkowe zachowania celów dla LATTICE:

* **Geometrie celów** — obsługiwane są panele oznaczone ArUco, panele ze stałym obszarem zainteresowania (ROI) oraz cele paskowe; geometria pochodzi z konfiguracji celów projektu.
* **Dane pomiarowe celu dla poszczególnych jednostek** — `--target-reflectance-dir DIR` wskazuje katalog skanów odbicia pomiarowego celu dla poszczególnych jednostek (`<serial>.csv`, wyszukiwanych na podstawie numeru seryjnego/kodu QR jednostki celu). W przypadku braku trafienia program Chloros

przechodzi na nominalne widma T3/T4P.
* **Kotwiczenie czasowe** — wykryty cel kalibruje otaczające go klatki i jest utrzymywany między kolejnymi wykryciami celu.

Pełna semantyka flag i przykłady znajdują się w [Podręczniku CLI

](reference/cli-reference.md) (patrz „Przełączniki eksportu dla poszczególnych produktów”).

### F988

„Odbicie F988 jest kalibrowane przy użyciu panelu odbicia znajdującego się w scenie: pasmo to wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros

stosuje najnowszy zapis z panelu i utrzymuje go między kolejnymi obserwacjami panelu”.

Jeśli F988 jest uruchamiany z kalibracją wyłącznie za pomocą DAQ, Chloros

odrzuca współczynnik odbicia oparty na DAQ dla tego pasma i podaje powód (przyczyna pominięcia `dls-uncalibrated-band-988`); obsługiwanym sposobem postępowania jest procedura z wykorzystaniem panelu.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
