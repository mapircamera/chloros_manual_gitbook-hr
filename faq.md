---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<detalji>

<summary>Mogu li slike s kamera koje nisu MAPIR obraditi s Chloros?</summary>

Ne, Chloros podržava samo obradu slika MAPIR kamere. Za više informacija pogledajte popis [podržanih modela kamera] (supported-cameras.md). Nudimo obradu drugih kamera na MAPIR Cloudu, pogledajte cijeli popis [ovdje](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<detalji>

<summary>Mogu li kalibrirati svoje slike za refleksiju bez kalibracijske mete?</summary>

Ne. Bez slike kalibracijske mete snimljene oko trenutka kada se snimaju neciljne slike, nećete moći povezati vrijednosti piksela slike s poznatim postotkom refleksije. Ako također ne uključite evidenciju svjetlosnog senzora MAPIR, tada se spektar ambijentalnog svjetla neće mjeriti, a rezultati refleksije neće biti točni.

</details>

<detalji>

<summary>Mogu li urediti svoje slike prije obrade u Chloros?</summary>

Br. Chloros pretpostavlja da ulazni podaci nisu izmijenjeni. Nemojte mijenjati nazive datoteka.

</details>

<detalji>

<summary>Mogu li postaviti svoje MAPIR Survey3 kamere na automatsku ekspoziciju i obradu slika u Chloros?</summary>

Br. Survey3 skupovi slikovnih podataka moraju imati fiksnu/zaključanu ekspoziciju, dakle bez automatske brzine zatvarača ili automatskog ISO-a. Sve slike istog modela fotoaparata moraju imati identičnu brzinu zatvarača i ISO (ekspoziciju).

</details>

<detalji>

<sažetak>Može li Chloros obraditi ili analizirati ortomozaične slike?</sažetak>

Ne. Podržane su samo pojedinačne slike MAPIR kamere, a ne spojene slike poput ortomozične karte.

</details>

<detalji>

<summary>Kako mogu ubrzati korak otkrivanja cilja za Chloros?</summary>

U tablici preglednika datoteka prethodni odabir ciljanih slika u desnom stupcu reći će Chloros da u tim slikama traži samo ciljeve kalibracije, što znatno ubrzava obradu.

</details>

<detalji>

<sažetak>Ako ću učitati svoje slike u <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>, trebam li obraditi u Chloros prije učitavanja?</summary>

Ako planirate prenijeti na našu platformu za online obradu [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nemojte uređivati ​​slike prije učitavanja. Cloud će izvršiti istu obradu i više.

</details>

<detalji>

<sažetak>Hoće li MAPIR ikada podržavati X značajku? Stvarno bih volio da MAPIR nudi X.</summary>

Uvijek smo zainteresirani za povratne informacije o našim proizvodima. Ako pronađete problem s našim proizvodima ili imate prijedlog o tome kako možemo poboljšati naše proizvode, [KONTAKTIRAJTE NAS] (https://www.mapir.camera/community/contact) da podijelite svoje mišljenje. Većina našeg istraživanja i razvoja vođena je slušanjem najvećih potreba naših kupaca.

</details>

<detalji>

<summary>Je li Chloros dostupan za Linux?</summary>

Da! Chloros 1.1.0 podržava Linux amd64 (x86\_64) i arm64 (NVIDIA Jetson JetPack 6) preko `.deb` paketa. CLI i Python SDK u potpunosti su podržani na Linux. Ne postoji GUI za Linux — sva interakcija je putem [CLI](CLI.md) ili [Python SDK](api-python-sdk.md). Za detalje pogledajte [Linux Pregled](linux/linux-overview.md).

</details>

<detalji>

<summary>Mogu li pokrenuti Chloros na NVIDIA Jetson?</summary>

Da! Chloros 1.1.0 podržava NVIDIA Jetson platforme uključujući Jetson Nano, Orin Nano, Orin NX i AGX Orin koji pokreće JetPack 6. Chloros automatski otkriva vaš Jetson model i optimizira njegovu strategiju obrade. Pogledajte [Vodič za NVIDIA Jetson](linux/nvidia-jetson-guide.md) za upute za postavljanje i implementaciju.

</details>

<detalji>

<sažetak>Optimizira li se Chloros automatski za moj hardver?</sažetak>

Da! Chloros 1.1.0 uključuje [Dynamic Compute Adaptation] (processing-architecture/dynamic-compute-adaptation.md) koja automatski detektira vaš CPU, GPU, RAM i (na Jetson) toplinske senzore. Zatim odabire optimalnu strategiju obrade — od `GPU_PARALLEL` na sustavima s velikom količinom memorije do `GPU_SINGLE` na ograničenim uređajima do `CPU_PARALLEL` na sustavima bez NVIDIA GPU-a. Nije potrebna ručna konfiguracija.

</details>

<detalji>

<summary>Što je cjevovod za obradu u 4 niti?</summary>

Chloros 1.1.0 koristi cjevovodnu arhitekturu od 4 niti za korisnike Chloros+: nit 1 (detekcija) učitava slike i otkriva ciljeve kalibracije, nit 2 (kalibracija) izračunava kalibraciju refleksije, nit 3 (obrada) izvodi debayering ubrzano GPU-om i izračun indeksa, a Thread 4 (Export) piše izlazne datoteke. Više slika može biti u različitim nitima istovremeno za maksimalnu propusnost. Pogledajte [Cjevovod za obradu] (processing-architecture/processing-pipeline.md) za detalje.

</details>

<detalji>

<sažetak>Kako mogu pokrenuti dijagnostiku na mojoj instalaciji Chloros?</sažetak>

Upotrijebite naredbu `selftest` za pokretanje 7 dijagnostika sustava uključujući provjeru verzije, dostupnost porta, pokretanje pozadine, API povezivost, informacije o sustavu, modele denoisera i dostupnost CUDA-e:

```bash
chloros-cli selftest
```

Ovo je posebno korisno na sustavima Linux/Jetson za provjeru GPU i CUDA postavki.

</details>