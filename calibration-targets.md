---
description: Panele pomiarowe laboratoryjne używane do kalibracji przechwyconych danych w przetwarzaniu końcowym
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cele kalibracji

MAPIR oferuje różne cele kalibracyjne dla szerokiego zakresu zastosowań. Przedstawiony poniżej kompaktowy model T4-R50 składa się z 4 paneli, dla których zmierzono współczynnik odbicia światła w zakresie 250–2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Rozproszone cele odniesienia T4 mają następujące krzywe odbicia: [pobierz dane tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Odbicie :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Odbicie :: 400-1000nm</p></figcaption></figure>

Patrząc na wykres współczynnika odbicia można zobaczyć, że wartościami są długość fali (oś x) w funkcji procentu odbicia (oś y). Po przechwyceniu obrazu obiektu kalibracyjnego tworzymy zależność między wartością pikseli a procentem odbicia w obrębie widma, na które wrażliwe jest każde z pasm czujnika kamery.

Oznacza to, że do każdego obrazu zarejestrowanego naszymi aparatami możesz użyć zdjęcia naszych obiektów docelowych o współczynniku odbicia, takich jak [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) lub [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), aby skalibrować obrazy pod kątem współczynnika odbicia. Po skalibrowaniu każdy piksel obrazu jest równy procentowi współczynnika odbicia.

Jeśli skalibrowane obrazy wyślesz w Chloros jako typowy JPG lub TIFF, wówczas procent odbicia zostanie obliczony poprzez podzielenie wartości pikseli przez głębię bitową formatu obrazu. Zatem w przypadku JPG podziel przez 255, a w przypadku TIFF podziel przez 65 535. Możesz także wybrać wyjście w formacie PROCENT w Chloros, a wtedy każdy piksel będzie miał wartość procentową od 0,0 do 1,0 (odbicie od 0% do 100%). Należy tylko pamiętać, że niektóre aplikacje graficzne nie obsługują obrazów procentowych (zmiennoprzecinkowych) i zajmują dużo miejsca w pamięci.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
