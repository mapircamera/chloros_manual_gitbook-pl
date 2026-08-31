---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cele kalibracyjne

MAPIR oferuje różne cele kalibracyjne przeznaczone do szerokiego zakresu zastosowań. Przedstawiony poniżej kompaktowy model T4-R50 zawiera 4 panele, dla których zmierzono współczynnik odbicia światła w zakresie od 250 do 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Cele referencyjne rozpraszające T4 charakteryzują się następującymi krzywymi współczynnika odbicia, [dane do pobrania tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Odbicie T4 :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Odbicie T4 :: 400–1000 nm</p></figcaption></figure>Cele referencyjne rozpraszające T4P charakteryzują się następującymi krzywymi odbicia: [pobierz dane tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Odbicie T4P :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Odbicie T4P :: 400–1000 nm</p></figcaption></figure>Analizując wykres odbicia, można zauważyć, że wartości przedstawiają zależność między długością fali (oś x) a procentowym odbiciem (oś y). Po zarejestrowaniu obrazu wzorca kalibracyjnego ustalamy zależność między wartością piksela a procentowym odbiciem w zakresie widma, na które czułe są poszczególne pasma matrycy kamery.

Oznacza to, że do każdego zdjęcia wykonanego naszymi kamerami można wykorzystać zdjęcie naszych wzorców odbicia, takich jak [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) lub [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), do kalibracji zdjęć pod kątem współczynnika odbicia. Po kalibracji każdy piksel na zdjęciu odpowiada procentowemu współczynnikowi odbicia.

W przypadku **Survey3** , jeśli wyeksportujesz skalibrowane obrazy w formacie Chloros jako typowy plik JPG lub TIFF, wówczas procent odbicia oblicza się, dzieląc wartość piksela przez głębię bitową formatu obrazu. W przypadku formatu JPG należy więc podzielić przez 255, a w przypadku formatu TIFF – przez 65 535. Można również wybrać format wyjściowy PERCENT w pliku Chloros, a wtedy każdy piksel będzie miał wartość procentową w zakresie od 0,0 do 1,0 (od 0% do 100% współczynnika odbicia). Należy jednak pamiętać, że niektóre aplikacje graficzne nie obsługują obrazów w procentach (z liczbami zmiennoprzecinkowymi), a pliki takie zajmują dużo miejsca.

{% hint style="info" %}
**Współczynnik odbicia LATTICE wykorzystuje inną skalę pikseli.** Współczynnik odbicia LATTICE jest zapisywany przy DN 32768 = 100% współczynnika odbicia (a nie 65535), a każdy plik zawiera tag XMP `Chloros:PixelScale` określający jego skalę. Należy odczytać ten tag i podzielić przez tę wartość, zamiast zakładać stałą wartość — zobacz [Formaty obrazów wyjściowych](output-image-formats.md).
{% endhint %}

## Cele kalibracyjne w kamerach LATTICE

W przypadku kamer LATTICE cel kalibracyjny jest **opcjonalny** dla współczynnika odbicia: Chloros może zamiast tego odnosić współczynnik odbicia do natężenia promieniowania padającego zmierzonego przez czujnik światła DAQ (ρ = π·L/E). Odniesienie wybiera się za pomocą ustawienia źródła współczynnika odbicia (Ustawienia projektu w interfejsie graficznym; `--reflectance-source` w CLI; `reflectance_source` w SDK):

| Wartość | Zachowanie |
| --- | --- |
| `auto` *(domyślnie)* | Cel w kadrze, który przeszedł kontrolę jakości (QA), stanowi **odniesienie bezwzględne**; gdy nie ma celu lub kontrola jakości zakończy się niepowodzeniem, Chloros przechodzi na podział sygnału przychodzącego z DAQ. |
| `target` | Wyłącznie cel — brak substytucji DAQ. |
| `daq` | Autorytet DAQ — pomiar w kierunku w dół zawsze stanowi odniesienie. |

Dodatkowe zachowanie celów dla LATTICE:

* **Geometrie celów** — obsługiwane są panele oznaczone ArUco, panele ze stałym obszarem zainteresowania (ROI) oraz cele paskowe; geometria pochodzi z konfiguracji celów projektu.
* **Dane pomiarowe celów dla poszczególnych jednostek** — `--target-reflectance-dir DIR` wskazuje katalog skanów odbicia pomiarowego poszczególnych jednostek (`<serial>.csv`, wyszukiwanych na podstawie numeru seryjnego/kodu QR jednostki). W przypadku braku trafienia Chloros przechodzi na nominalne widma T3/T4P.
* **Kotwiczenie czasowe** — wykryty cel kalibruje otaczające go klatki i jest utrzymywany między kolejnymi wykryciami celu.

Pełna semantyka flag i przykłady znajdują się w [Dokumentacji CLI](reference/cli-reference.md) (patrz „Przełączniki eksportu dla poszczególnych produktów”).

### F988

„Odbicie F988 jest kalibrowane przy użyciu panelu odbicia umieszczonego w scenie: pasmo wykracza poza skalibrowany zakres czujnika światła DAQ, więc Chloros stosuje najnowszy zapis z panelu i utrzymuje go między kolejnymi obserwacjami panelu”.

Jeśli F988 jest uruchamiany z kalibracją opartą wyłącznie na DAQ, Chloros odrzuca współczynnik odbicia oparty na DAQ dla tego pasma i podaje powód (powód pominięcia `dls-uncalibrated-band-988`); zalecanym sposobem postępowania jest procedura z wykorzystaniem panelu.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
