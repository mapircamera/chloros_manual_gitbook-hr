---
opis: Laboratorijski mjerene ploče koje se koriste za kalibraciju snimljenih podataka u naknadnoj obradi
metaLinks:
zamjenici:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibracijski ciljevi

MAPIR nudi različite kalibracijske ciljeve za pokrivanje niza primjena. Kompaktni T4-R50 prikazan dolje sadrži 4 ploče kojima je izmjerena refleksija svjetlosti od 250 - 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 difuzne referentne mete imaju sljedeće krivulje refleksije, [podatke preuzmite ovdje](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 400-1000nm</p></figcaption></figure>

Gledajući grafikon refleksije možete vidjeti da su vrijednosti valne duljine (x-os) u odnosu na postotak refleksije (y-os). Kada snimimo sliku kalibracijske mete, tada stvaramo odnos između vrijednosti piksela i postotka refleksije, unutar spektra na koji je svaki od senzora kamere osjetljiv.

To znači da uz svaku sliku koju snimite našim fotoaparatima možete koristiti fotografiju naših ciljeva refleksije, kao što je [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ili [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) za kalibraciju slike za refleksiju. Nakon kalibracije svaki piksel na slici jednak je postotku refleksije.

Ako ispisujete kalibrirane slike u Chlorosu kao tipični JPG ili TIFF, tada se postotak refleksije izračunava dijeljenjem vrijednosti piksela s dubinom bita formata slike. Dakle, za JPG podijelite s 255, a za TIFF s 65,535. Također možete odabrati PERCENT izlaz formata u Chlorosu, a zatim će svaki piksel varirati od postotne vrijednosti od 0,0 do 1,0 (0% do 100% refleksije). Samo imajte na umu da neke aplikacije za slike ne mogu prihvatiti slike postotaka (pokretni zarez) i velike su veličine pohrane.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
