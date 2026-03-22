---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cele kalibracyjne

MAPIR oferuje różne cele kalibracyjne przeznaczone do szerokiego zakresu zastosowań. Przedstawiony poniżej kompaktowy model T4-R50 zawiera 4 panele, których współczynnik odbicia światła został zmierzony w zakresie od 250 do 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Cele referencyjne T4 o rozproszonym odbiciu mają następujące krzywe odbicia, [pobierz dane tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Odbicie T4 :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Odbicie T4 :: 400–1000 nm</p></figcaption></figure>Cele referencyjne T4P o rozproszonym odbiciu mają następujące krzywe odbicia, [pobierz dane tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Odbicie T4P :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Odbicie T4P :: 400–1000 nm</p></figcaption></figure>Patrząc na wykres odbicia, widać, że wartości są przedstawione jako długość fali (oś x) w stosunku do procentowego odbicia (oś y). Kiedy robimy zdjęcie tarczy kalibracyjnej, tworzymy zależność między wartością piksela a procentowym odbiciem w zakresie widma, na które wrażliwe są poszczególne pasma czujnika aparatu.

Oznacza to, że przy każdym zdjęciu wykonanym naszymi kamerami można użyć zdjęcia naszych celów odbicia, takich jak [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) lub [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), aby skalibrować zdjęcia pod kątem odbicia. Po kalibracji każdy piksel na zdjęciu odpowiada procentowej wartości odbicia.

Jeśli wyeksportujesz skalibrowane zdjęcia w formacie Chloros jako typowy plik JPG lub TIFF, wówczas procent odbicia oblicza się, dzieląc wartość piksela przez głębię bitową formatu obrazu. W przypadku formatu JPG należy podzielić przez 255, a w przypadku formatu TIFF przez 65 535. Można również wybrać format wyjściowy PERCENT w Chloros, a wtedy każdy piksel będzie miał wartość procentową w zakresie od 0,0 do 1,0 (od 0% do 100% odbicia). Należy jednak pamiętać, że niektóre aplikacje graficzne nie obsługują obrazów procentowych (z zmiennoprzecinkowymi wartościami), a ponadto zajmują one dużo miejsca w pamięci.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
