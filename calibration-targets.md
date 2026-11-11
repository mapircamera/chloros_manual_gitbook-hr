---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibracijske mete

MAPIR nudi različite kalibracijske mete za pokrivanje niza primjena. Kompaktni T4-R50 prikazan ispod sadrži 4 panela koji su izmjereni za reflektanciju svjetla od 250 do 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4 difuzne referentne mete imaju sljedeće krivulje odbijanja, [preuzmite podatke ovdje](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektancija :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectancija :: 400-1000 nm</p></figcaption></figure>T4P difuzne referentne mete imaju sljedeće krivulje reflektancije, [preuzmite podatke ovdje](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Reflectance :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Reflektancija :: 400-1000nm</p></figcaption></figure>Gledajući graf reflektancije, možete vidjeti da su vrijednosti valna duljina (x-os) nasuprot postotku reflektancije (y-os). Kada snimimo sliku kalibracijskog cilja, tada stvaramo odnos između vrijednosti piksela i postotka reflektancije unutar spektra na koji je osjetljiv svaki od senzorskih pojaseva kamere.

To znači da za svaku snimku koju napravite našim kamerama možete koristiti fotografiju naših reflektivnih ciljeva, kao što je [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ili [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) kako biste kalibrirali slike za reflektanciju. Nakon kalibracije, svaki piksel u slici jednak je postotku reflektancije.

Za izlaze **Survey3**, ako izvezete kalibrirane slike u Chloros kao uobičajeni JPG ili TIFF, tada se postotak reflektancije izračunava dijeljenjem vrijednosti piksela s dubinom bita formata slike. Dakle, za JPG dijelite s 255, a za TIFF dijelite s 65.535. Također možete odabrati izlazni PERCENT format u Chloros u, a tada će se vrijednost svakog piksela kretati u rasponu od 0,0 do 1,0 (0 % do 100 % reflektancije). Samo imajte na umu da neke aplikacije za rad s slikama ne mogu prihvatiti slike u postotnom (povratnom) formatu i da one zauzimaju mnogo prostora za pohranu.

{% hint style="info" %}
**LATTICE reflektancija koristi drugačiju skalu piksela.** LATTICE reflektancija pohranjuje se s DN 32768 = 100 % reflektancije (a ne 65535), a svaka datoteka sadrži XMP oznaku `Chloros:PixelScale` koja navodi njezinu skalu. Pročitajte oznaku i podijelite s njom umjesto da pretpostavljate konstantu — pogledajte [Formati izlaznih slika](output-image-formats.md).
{% endhint %}

## Kalibracijske pločice s LATTICE kamerama

Kod LATTICE kamera kalibracijska pločica za reflektanciju je **neobavezna**: Chloros umjesto toga može referencirati reflektanciju na zračenje prema dolje koje mjeri DAQ senzor svjetla (ρ = π·L/E). Referenca se odabire postavkom izvora odbijanja u projektu, postavljenom pomoću `chloros-cli process --reflectance-source` (grafičko sučelje nema kontrolu za to i prema zadanim postavkama koristi pohranjenu vrijednost projekta, `auto` — pogledajte [Postavke projekta](project-settings/project-settings.md)):

| Vrijednost | Ponašanje |
| `auto` *(zadano)* | Cilj u kadru koji prolazi QA je **apsolutna referenca**; kada cilj nije prisutan ili QA ne uspije, Chloros se vraća na DAQ-ovu podjelu prema dolje. Cilj koji prođe QA s oznakom (ispušten panel ili traka izvan redoslijeda) ne dobiva automatski prednost: lanci cilj, cilj+DAQ i samo DAQ isprobavaju se prema redoslijedu njihovih izmjerenih rezultata na sačuvanim panelima. |
| `target` | Strogo samo cilj — nema zamjene DAQ-om. |
| `daq` | Autoritativno DAQ — mjerenje prema dolje je uvijek referenca. |

Dodatno ponašanje cilja za LATTICE:

* **Geometrije cilja** — podržani su ArUco-označeni paneli, paneli s fiksnim ROI-jem i ciljevi u traci; geometrija se uzima iz konfiguracije cilja projekta.
* **Podaci o reflektanciji cilja mjereni po jedinici** — `--target-reflectance-dir DIR` upućuje na direktorij skeniranja reflektancije cilja mjerenih po jedinici (`<serial>.csv`, dohvaćen pomoću serijskog/QR koda jedinice cilja). U slučaju promašaja, Chloros se vraća na nominalna T3/T4P spektra.
* **Vremensko sidrenje** — otkriveni cilj kalibrira okvire oko sebe i zadržava se između uočavanja cilja.

Puna semantika zastavica i primjeri nalaze se u [Referenci za CLI](reference/cli-reference.md) (vidi "Prekidači za izvoz po proizvodu").

### F988

"Reflekcija pomoću F988 kalibrira se korištenjem panela za refleksiju unutar scene: pojas leži izvan kalibriranog raspona DAQ senzora svjetla, stoga Chloros primjenjuje vašu najnoviju snimku panela i zadržava je između očitavanja panela."

Ako se F988 pokrene s kalibracijom samo putem DAQ-a, Chloros odbacuje reflektanciju temeljenu na DAQ-u za tu traku i navodi razlog (skip reason `dls-uncalibrated-band-988`); radni tijek s panelom je podržani put.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
