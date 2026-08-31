---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# ČPP

<details>

<summary>Mogu li obrađivati slike s kamera koje nisu marke MAPIR pomoću Chloros?</summary>

Ne, Chloros podržava obradu slika samo s kamera obitelji MAPIR — Survey3 i LATTICE. Za više informacija pogledajte popis [podržanih modela kamera](supported-cameras.md). Također nudimo obradu slika s drugih kamera na MAPIR Cloud, pogledajte cijeli popis [ovdje](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Podržava li Chloros LATTICE kamere?</summary>

Da. Chloros 1.2.0 podržava LATTICE M3C i M3M module kamera od početka do kraja: **uživo kontrola**— otkrijte, povežite se, pregledajte i snimite s kartice Kamere u GUI-ju, `chloros-cli lattice`, ili s PythonSDK, uključujući sinkronizirane nizove više kamera s PTP vremenskom sinkronizacijom — i**potpunu radiometrijsku obradu** snimki (raw → debayered → sjaj → odraz → indeks). Pogledajte [Podržane kamere](supported-cameras.md) i [LATTICE vodič](lattice/README.md).

</details>

<details>

<summary>Mogu li kalibrirati svoje slike za reflektanciju bez kalibracijskog cilja?</summary>

**Survey3

:** Ne. Bez slike kalibracijskog cilja snimljene otprilike u isto vrijeme kada su snimljene i slike bez cilja, nećete moći povezati vrijednosti piksela slike s poznatim postotkom reflektancije. Ako također ne uključite zapisnik s senzora svjetlaMAPIR

, spektar ambijentalnog svjetla neće biti izmjeren i rezultati reflektancije neće biti točni.

**LATTICE:** Da. Reflektancija se može referencirati na zračnu radijanciju mjerenu DAQ senzorom svjetlosti umjesto na panel (ρ = π·L/E). Kada je prisutan cilj u kadru koji je prošao QA, on postaje apsolutna referenca prema zadanim postavkama (`--reflectance-source auto`). Jedan izuzetak: &quot;F988 reflektancija kalibrira se pomoću reflektantne ploče unutar scene: pojas leži izvan kalibriranog raspona DAQ senzora svjetlosti, stoga seChloros

primjenjuje vaša najnovija snimka ploče i zadržava je između promatranja ploče.&quot; Vidi [Ciljevi kalibracije](calibration-targets.md).

</details>

<details>

<summary>Trebam li DAQ senzor svjetla?</summary>

Ne za radijancu: Izvoz radijance u LATTICE formatu temelji se na tvorničkoj radiometrijskoj kalibraciji svake kamere i ne zahtijeva ni DAQ senzor ni metu. Za **reflektancu**potrebna vam je referenca za ambijentalno svjetlo — ili mjerenje svjetlosti prema dolje DAQ senzora za svjetlo ili kalibracijska meta unutar kadra. DAQ senzor vam omogućuje izradu kalibrirane reflektancije**bez postavljanja ikakvih panela u scenu**. Snimljene `.daq` datoteke automatski se usklađuju s vašim slikama prema vremenu snimanja. Pogledajte [Referentne ploče za kalibraciju](calibration-targets.md) i [Referentni materijal za kalibraciju](reference/cli-reference.md).

</details>

<details>

<summary>Mogu li koristiti Chloros s AI asistentom (Claude, ChatGPT itd.)?</summary>

Da — ovaj priručnik i CLI / SDK izrađeni su za to:

* Cjeloviti indeks priručnika dostupan je na `https://mapir.gitbook.io/chloros/llms.txt` kako bi AI asistenti mogli otkriti svaku stranicu.
* Sirovi markdown svake stranice dostupan je na njezinoj malom pismenom adresi URL s dodanim `.md` (na primjer `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI referenca](reference/cli-reference.md) i [SDK referenca](reference/sdk-reference.md) napisane su za upotrebu s LLM-ovima: točne zastavice, zadane vrijednosti, semantika izlaska i naredbe koje se mogu kopirati i zalijepiti.

Pogledajte [AI Assistants](ai-assistants.md) kako biste usmjerili svog asistenta na Chloros.

</details>

<details>

<summary>Kamo se spremaju moje obrađene izlazne datoteke?</summary>

Proizvodi se spremaju u mapu projekta, grupirani po kameri i zatim po formatu datoteke:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **folder-kamere** — `LATT-<sensor>-<lens>-F<filter>` za LATTICE, `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3
* **format-folder** — `tiff16`, `tiff8`, `png8`, `jpg8` ili `tiff32`
* **folderi proizvoda** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (uvijek ispod `tiff32`), `<INDEX>_Index_Images/`**Izvozene datoteke zadržavaju naziv izvornog datoteka — mapa identificira proizvod, a ne nastavak naziva datoteke.**Uz CLI, projektna mapa se stvara pored ulazne mape, osim ako ne navedete `-o`. Imajte na umu da izvršavanje naredbe `chloros-cli process` koje je zatražilo proizvode, ali nije zapisalo nijedan, ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nulovitim rezultatom**, pa ga skripte mogu otkriti. Pogledajte [Output Image Formats](output-image-formats.md) i [CLI Reference](reference/cli-reference.md).

</details>

<details>

<summary>Mogu li urediti svoje slike prije obrade u Chloros?</summary>

Ne. Chloros pretpostavlja da ulazni podaci nisu izmijenjeni. Nemojte mijenjati nazive datoteka.

</details>

<details>

<summary>Mogu li postaviti svoje MAPIR Survey3 kamere na automatsku ekspoziciju i obraditi slike u Chloros?</summary>

Ne. Skupovi slika za Survey3 moraju imati fiksnu/zaključanu ekspoziciju, stoga nema automatske brzine zatvarača ili automatskog ISO-a. Sve slike istog modela kamere moraju imati identičnu brzinu zatvarača i ISO (ekspoziciju).

Kamere LATTICE nemaju ovo ograničenje: Chloros upravlja ekspozicijom u stvarnom vremenu (Smart AE), a svako snimanje bilježi stvarnu upotrijebljenu ekspoziciju i pojačanje, što radiometrijski pipeline uzima u obzir.

</details>

<details>

<summary>Može li Chloros obrađivati ili analizirati ortomosaicne slike?</summary>

Ne. Podržane su samo pojedinačne slike iz kamere MAPIR, a ne spojene slike poput ortomosaika.

</details>

<details>

<summary>Kako mogu ubrzati korak detekcije ciljeva u programu Chloros?</summary>

U tablici preglednika datoteka, prethodno odabiranje ciljnih slika u desnoj stupci reći će Chlorosu da traži kalibracijske ciljeve samo u tim slikama, čime se znatno ubrzava obrada.

</details>

<details>

<summary>Ako ću svoje slike učitati na <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> trebam li ih prvo obraditi u Chloros prije učitavanja?</summary>

Ako planirate prenijeti slike na našu internetsku platformu za obradu [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nemojte uređivati slike prije prijenosa. Cloud će izvršiti svu istu obradu, a još mnogo više.

</details>

<details>

<summary>Hoće li MAPIR ikada podržati značajku X? Stvarno bih volio da MAPIR nudi X.</summary>

Uvijek smo zainteresirani za povratne informacije o našim proizvodima. Ako pronađete problem s našim proizvodima ili imate prijedlog kako bismo ih mogli poboljšati, molimo vas da nas [KONTAKTIRATE](https://www.mapir.camera/community/contact) i podijelite svoja razmišljanja. Većinu našeg istraživanja i razvoja (R&amp;D) usmjeravamo slušajući najveće potrebe naših kupaca.

</details>

<details>

<summary>Je li Chloros dostupan za Linux?</summary>

Da! Chloros 1.2.0 podržava Linux amd64 (x86_64) i arm64 (NVIDIA Jetson JetPack 6) putem paketa `.deb`. CLI i Python SDK u potpunosti su podržani na Linux, uključujući upravljanje LATTICE kamerom uživo i DAQ senzorima. Ne postoji GUI za Linux — sva interakcija odvija se putem [CLI](CLI.md) ili [Python SDK](api-python-sdk.md). Pogledajte [Linux Overview](linux/linux-overview.md) za detalje.

</details>

<details>

<summary>Mogu li pokrenuti Chloros na NVIDIA Jetson?</summary>

Da! Chloros podržava NVIDIA Jetson platforme uključujući Jetson Nano, Orin Nano, Orin NX i AGX Orin koje koriste JetPack 6. Chloros automatski otkriva vaš model Jetson i optimizira strategiju obrade. Pogledajte [NVIDIA Jetson vodič](linux/nvidia-jetson-guide.md) za upute za postavljanje i implementaciju.

</details>

<details>

<summary>Optimizira li Chloros automatski za moj hardver?</summary>

Da! Chloros uključuje [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) koja automatski otkriva vaš CPU, GPU, RAM i (na Jetsonu) termalne senzore. Zatim odabire optimalnu strategiju obrade — od `GPU_PARALLEL` na sustavima s velikom memorijom do `GPU_SINGLE` na ograničenim uređajima do `CPU_PARALLEL` na sustavima bez NVIDIA GPU-a. Nije potrebna ručna konfiguracija.

</details>

<details>

<summary>Što je 4-thread pipeline za obradu?</summary>

Chloros

koristi pipelined arhitekturu s 4 niti za korisnike s više jezgri: Nit 1 (Detekcija) učitava slike i otkriva ciljeve za kalibraciju, Nit 2 (Kalibracija) izračunava kalibraciju reflektivnosti, Nit 3 (Obrada) izvodi GPU-akcelerirano debayering i izračun indeksa, a Nit 4 (Izvoz) zapisuje izlazne datoteke. Više slika može istovremeno biti u različitim nitima radi maksimalne propusnosti. Pogledajte [Processing Pipeline](processing-architecture/processing-pipeline.md) za detalje.

</details>

<details>

<summary>Kako pokrenuti dijagnostiku na mojoj instalaciji Chloros?</summary>

Koristite naredbu `selftest` za pokretanje 7-koraknog smoke testa: verzija, dostupnost porta, pokretanje backenda, povezanost s API (`/api/test`), informacije o sustavu (`/api/system-info` — GPU/CUDA/PyTorch), prisutnost modela za uklanjanje šuma i spremnost CUDA + uklanjanja šuma:

```bash
chloros-cli selftest
```

Ovo je posebno korisno na sustavima Linux/Jetson za provjeru postavke GPU-a i CUDA-e.

</details>
