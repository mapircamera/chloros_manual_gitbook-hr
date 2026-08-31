---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibracijske mete

MAPIRnudi različite kalibracijske mete za pokrivanje niza primjena. Kompaktni T4-R50 prikazan ispod sadrži 4 panela koji su izmjereni za reflektanciju svjetla od 250 do 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 referentne mete za difuzno odbijanje imaju sljedeće krivulje odbijanja, [preuzmite podatke ovdje](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektancija :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflektancija :: 400-1000nm</p></figcaption></figure>T4P difuzne referentne mete imaju sljedeće krivulje reflektancije, [preuzmite podatke ovdje](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Reflektancija :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Reflektancija :: 400-1000nm</p></figcaption></figure>Gledajući graf reflektancije, možete vidjeti da su vrijednosti valna duljina (x-os) nasuprot postotku reflektancije (y-os). Kada snimimo sliku kalibracijskog cilja, tada stvaramo odnos između vrijednosti piksela i postotka reflektancije unutar spektra na koji je osjetljiv svaki od senzorskih pojaseva kamere.

To znači da za svaku snimku koju napravite našim kamerama možete koristiti fotografiju naših reflektivnih ciljeva, kao što je [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ili [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) za kalibraciju slika po odrazu. Nakon kalibracije, svaki piksel u slici jednak je postotku odraza.

Za izlaze **Survey3**, ako izvezete kalibrirane slike uChloros

u uobičajenom JPG iliTIFF

formatu, postotak reflektancije izračunava se dijeljenjem vrijednosti piksela s dubinom bita formata slike. Dakle, za JPG dijelite s 255, a zaTIFF

dijelite s 65.535. Možete također odabrati izlaz u PERCENT formatu uChloros

, a tada će se vrijednost svakog piksela kretati u rasponu od 0,0 do 1,0 (0 % do 100 % reflektancije). Samo imajte na umu da neki programi za rad s slikama ne mogu prihvatiti slike u postotnom (povratnom) formatu, a one su velike veličine po pitanju pohrane.

{% hint style="info" %}
**LATTICE reflektancija koristi drugačiju skalu piksela.** LATTICE reflektancija pohranjuje se s DN 32768 = 100 % reflektancije (a ne 65535), a svaka datoteka sadrži XMP oznaku `Chloros:PixelScale` koja navodi njezinu skalu. Pročitajte oznaku i podijelite s njom umjesto da pretpostavljate konstantu — pogledajte [Formati izlaznih slika](output-image-formats.md).
{% endhint %}

## Kalibracijske pločice s LATTICE kamerama

Kod LATTICE kamera kalibracijska pločica za reflektanciju je **neobavezna**:Chloros

može umjesto toga referencirati reflektanciju na zračenje prema dolje koje mjeri DAQ senzor svjetla (ρ = π·L/E). Referenca se odabire postavkom izvora reflektancije (Project Settings u GUI-ju; `--reflectance-source` uCLI

; `reflectance_source` uSDK

):

| Vrijednost | Ponašanje |
| --- | --- |
| `auto` *(zadano)* | Cilj u kadru koji prođe QA je **apsolutna referenca**; kada cilj nije prisutan ili QA ne uspije,Chloros

prelazi na DAQ downwelling divide. |
| `target` | Strogo samo cilj — bez DAQ supstitucije. |
| `daq` | DAQ-ovlašteno — mjerenje prema dolje je uvijek referenca. |

Dodatno ponašanje cilja za LATTICE:

* **Geometrije cilja** — podržani su ArUco-označeni paneli, paneli s fiksnim ROI-jem i ciljevi u traci; geometrija se uzima iz konfiguracije cilja projekta.
* **Podaci o reflektanciji cilja mjereni po jedinici** — `--target-reflectance-dir DIR` upućuje na direktorij skeniranja reflektancije cilja mjerenih po jedinici (`<serial>.csv`, dohvaćen pomoću serijskog/QR koda jedinice cilja). U slučaju promašaja,Chloros

se vraća na nominalna T3/T4P spektra.
* **Vremensko sidrenje** — otkriveni cilj kalibrira okvire oko sebe i zadržava se između uočavanja cilja.

Potpuna semantika zastavica i primjeri nalaze se u referentnoj dokumentaciji [CLI

](reference/cli-reference.md) (pogledajte &quot;Per-Product Export Toggles&quot;).

### F988

&quot;Refleksija F988 kalibrira se pomoću reflektirajuće ploče unutar scene: pojas leži izvan kalibriranog raspona DAQ senzora svjetla, stogaChloros

primjenjuje vašu najnoviju snimku ploče i zadržava je između snimanja ploče.&quot;

Ako se F988 pokrene s kalibracijom samo pomoću DAQ-a,Chloros

odbacuje reflektanciju temeljenu na DAQ-u za tu traku i navodi razlog (skip reason `dls-uncalibrated-band-988`); radni tijek s panelom je podržani put.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
