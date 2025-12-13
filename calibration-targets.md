---
description: Paneles medidos en laboratorio utilizados para calibrar los datos capturados en el posprocesamiento.
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Kalibracijski ciljevi

MAPIR nudi nekoliko kalibracijskih ciljeva za pokrivanje širokog raspona primjena. T4-R50 compact prikazan dolje sadrži 4 ploče čija je refleksija svjetlosti izmjerena između 250 i 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 difuzne referentne mete imaju sljedeće krivulje refleksije, [ovdje preuzmite podatke](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 400-1000 nm</p></figcaption></figure>

Ako pogledamo graf refleksije, možemo vidjeti da su vrijednosti valne duljine (x-os) u odnosu na postotak refleksije (y-os). Kada snimamo sliku kalibracijske mete, stvaramo odnos između vrijednosti piksela i postotka refleksije, unutar spektra na koji su trake senzora kamere osjetljive.

To znači da uz svaku sliku koju snimite našim fotoaparatima možete koristiti fotografiju naših ciljeva refleksije, kao što je [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ili [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) za kalibraciju slika za refleksiju. Nakon kalibracije, svaki piksel na slici jednak je postotku refleksije.

Ako se slike kalibrirane Chlorosom generiraju kao tipični JPG ili TIFF, postotak refleksije se izračunava dijeljenjem vrijednosti piksela s dubinom bita formata slike. Stoga se za JPG dijeli s 255, a za TIFF s 65 535. Također možete odabrati PERCENT format u Chlorosu, a tada će svaki piksel imati postotnu vrijednost između 0,0 i 1,0 (0% do 100% refleksije). Samo imajte na umu da neke aplikacije za slike ne prihvaćaju postotke (pokretni zarez) i da zauzimaju puno prostora za pohranu.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>