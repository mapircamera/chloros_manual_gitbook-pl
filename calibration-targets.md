---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Cele kalibracyjne

MAPIR oferuje różne cele kalibracyjne, które obejmują szeroki zakres zastosowań. Kompaktowy model T4-R50 przedstawiony poniżej zawiera 4 panele, które zostały zmierzone pod kątem odbicia światła w zakresie od 250 do 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Rozproszone cele referencyjne T4 mają następujące krzywe odbicia, [pobierz dane tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Odbicie :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Odbicie :: 400–1000 nm</p></figcaption></figure>Rozproszone cele referencyjne T4P mają następujące krzywe odbicia, [pobierz dane tutaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Odbicie :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Odbicie :: 400-1000 nm</p></figcaption></figure>Patrząc na wykres odbicia, można zauważyć, że wartości są długością fali (oś x) w stosunku do procentowego odbicia (oś y). Kiedy rejestrujemy obraz celu kalibracyjnego, tworzymy relację między wartością pikseli a procentowym odbiciem w zakresie widma, na które wrażliwe są pasma czujnika aparatu.

Oznacza to, że do każdego obrazu zarejestrowanego naszymi kamerami można użyć zdjęcia naszych celów odbicia, takich jak [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) lub [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), aby skalibrować obrazy pod kątem odbicia. Po kalibracji każdy piksel obrazu jest równy procentowej wartości odbicia.

Jeśli wyświetlasz skalibrowane obrazy w Chloros jako typowy plik JPG lub TIFF, procent odbicia jest obliczany poprzez podzielenie wartości piksela przez głębię bitową formatu obrazu. W przypadku pliku JPG należy podzielić przez 255, a w przypadku TIFF przez 65 535. Można również wybrać format PERCENT w Chloros, a wtedy każdy piksel będzie miał wartość procentową od 0,0 do 1,0 (odbicie od 0% do 100%). Należy jednak pamiętać, że niektóre aplikacje graficzne nie akceptują obrazów procentowych (zmiennoprzecinkowych), a ponadto zajmują one dużo miejsca w pamięci.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
