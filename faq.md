---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# ČPP

<details>

<summary>Mogu li s Chloros-om obrađivati slike s kamera koje nisu marke MAPIR?</summary>

Ne, Chloros podržava obradu samo slika s kamera marke MAPIR — obitelji Survey3 i LATTICE. Za više informacija pogledajte popis [podržanih modela kamera](supported-cameras.md). Nudimo i obradu snimki drugih kamera na MAPIR Cloud, pogledajte cijeli popis [ovdje](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros li podržava LATTICE kamere?</summary>

Da. Chloros 1.3.0 podržava LATTICE M3C i M3M module kamera od početka do kraja: **uživo kontrola**— otkrijte, povežite, pregledajte i snimite s kartice Kamere u GUI-ju, `chloros-cli lattice`, ili s Pythona SDKa, uključujući sinkronizirane nizove više kamera s PTP vremenskom sinkronizacijom — i**potpunu radiometrijsku obradu** snimki (raw → debayered → sjaj → odraz → indeks). Pogledajte [Podržane kamere](supported-cameras.md) i [LATTICE vodič](lattice/README.md).

</details>

<details>

<summary>Mogu li kalibrirati svoje slike za reflektanciju bez kalibracijskog cilja?</summary>

**Survey3:** Ne. Bez slike kalibracijskog cilja snimljene u isto vrijeme kada se snimaju i ostale slike, nećete moći povezati vrijednosti piksela slike s poznatim postotkom reflektancije. Ako također ne uključite zapisnik s senzora svjetla [MAPIR], spektar ambijentalnog svjetla neće biti izmjeren i rezultati reflektancije neće biti točni.

**LATTICE:** Da. Reflektancija se može referencirati na zračnu radijanciju prema dolje, izmjerenu DAQ senzorom svjetla, umjesto na panel (ρ = π·L/E). Kada je prisutan cilj u kadru koji je prošao QA, on postaje apsolutna referenca prema zadanim postavkama (`--reflectance-source auto`), osim ako ne prođe s oznakom, kada se lanci cilja i DAQ-a natječu po mjerenom rezultatu. Jedan izuzetak: "F988 reflektancija kalibrira se pomoću reflektantne ploče unutar scene: pojas leži izvan kalibriranog raspona DAQ senzora svjetlosti, stoga Chloros primjenjuje vašu najnoviju snimku ploče i zadržava je između promatranja ploče." Vidi [Ciljevi kalibracije](calibration-targets.md).

</details>

<details>

<summary>Trebam li DAQ senzor svjetla?</summary>

Ne za radijancu: izvoz radijance LATTICE-a dolazi iz tvorničke radiometrijske kalibracije svake kamere i ne zahtijeva ni DAQ senzor ni metu. Za **reflektanciju**potrebna vam je referenca za ambijentalno svjetlo — ili mjerenje svjetla prema dolje DAQ senzora svjetla ili kalibracijska meta unutar kadra. DAQ senzor omogućuje vam izradu kalibrirane reflektancije**bez postavljanja ikakvih panela u scenu**. Snimljene `.daq` datoteke automatski se usklađuju s vašim slikama pomoću vremenskog žiga. Pogledajte [Referentne pločice za kalibraciju](calibration-targets.md) i [CLI referencu](reference/cli-reference.md).

</details>

<details>

<summary>Mogu li koristiti Chloros s AI asistentom (Claude, ChatGPT itd.)?</summary>

Da — ovaj priručnik i CLI/SDK izrađeni su za to:

* Cjeloviti indeks priručnika poslužen je na `https://mapir.gitbook.io/chloros/llms.txt` kako bi AI asistenti mogli otkriti svaku stranicu.
* Sirovi markdown svake stranice dostupan je na njezinom malom pismenom URLu s dodanim `.md` (na primjer `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI referenca](reference/cli-reference.md) i [SDK referenca](reference/sdk-reference.md) napisane su za upotrebu s velikim jezičnim modelima (LLM): točne zastavice, zadane vrijednosti, semantika izlaska i naredbe koje se mogu kopirati i zalijepiti.

Pogledajte [AI Assistants](ai-assistants.md) kako biste usmjerili svog asistenta na Chloros.

</details>

<details>

<summary>Kamo se spremaju moje obrađene izlazne datoteke?</summary>

Proizvodi se spremaju u mapu projekta, grupirani po kameri i zatim po formatu datoteke:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **mapa-kamere** — `LATT-<sensor>-<lens>-F<filter>` za LATTICE, `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3
* **format-folder** — `tiff16`, `tiff8`, `png8`, `jpg8` ili `tiff32`
* **folderi proizvoda** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (uvijek ispod `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — samo za LATTICE multispectral: maske po pikselu pored mapa s proizvodima, vidi dolje**Izvožene datoteke zadržavaju naziv izvornog datoteka — mapa identificira proizvod, a ne ekstenzija naziva datoteke.**Kod korištenja CLI-a, mapa projekta stvara se u vašoj zadanoj mapi projekta (`Documents/MAPIR/Chloros Projects`, osim ako je niste promijenili), osim ako ne navedete `-o`; Relativna putanja uzima se u odnosu na mapu iz koje pokrećete naredbu. Imajte na umu da izvršavanje naredbe `chloros-cli process` koje je zatražilo proizvode, ali nije zapisalo nijedan, ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nulovitim rezultatom**, tako da ga skripte mogu detektirati. Pogledajte [Output Image Formats](output-image-formats.md) i [CLI Reference](reference/cli-reference.md).

</details>

<details>

<summary>Što su mapu clamp_masks i datoteke *_clampmask.tif?</summary>

Novo u verziji 1.3.0. Radiance i reflectance proizvodi multispektralne kamere LATTICE M3C provode unmix s 3×3 pojasevima, a tamo gdje proizvedu negativnu vrijednost u pojasu, Chloros tu vrijednost skraćuje na nulu. Proizvod s barem jednim obrezanim pikselom dobiva jednobandnu `uint8` masku koja označava koje su piksele i pojaseve obrezane (bit 0 = R kanal proizvoda, bit 1 = G, bit 2 = B). Maske se spremaju u mapu `clamp_masks/` **pored** mapa proizvoda — na primjer `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — a XMP oznaka proizvoda `Chloros:UnmixClampMask` naziva njezinu masku (ili navodi `none`).

To nisu slike scene: vlastiti import programa Chloros ih preskače, a budući da se nikada ne nalaze unutar mape proizvoda, mapu proizvoda možete predati Pix4D-u ili Metashapeu u izvornom obliku. Pogledajte [Clamp masks](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Zašto Chloros prikazuje poruku "Mapa projekta je preduboka za ovo računalo"?</summary>

Ovo vrijedi samo za Windows. Windows dolazi s isključenim dugim putanjama, pa nijedna putanja do datoteke ne smije biti dulja od 260 znakova. Datoteke proizvoda Chloros smještene su do otprilike 130 znakova ispod mape projekta, stoga Chloros odbija dublju mapu projekta prije obrade — umjesto da pokrene proces i ne zapiše ništa, što se moglo dogoditi prije verzije 1.3.0. Poruka prikazuje najdužu mapu projekta koja odgovara.

Riješite to na bilo koji od sljedećih načina:

* Koristite kraći direktorij projekta (**Radni direktorij** u postavkama projekta, ili `-o` / `chloros-cli set-project-folder` za CLI).
* Uključite dug putanja sustava Windows i ponovno pokrenite Chloros. Opcija **Omogući dug putanja sustava Windows** u instalacijskom programu za Chloros (zadano označena) radi upravo to; ponovno pokrenite instalacijski program ako ste je odznačili.

Vidi [Pokretanje obrade](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Mogu li urediti svoje slike prije obrade u Chlorosu?</summary>

Ne. Chloros pretpostavlja da ulazni podaci nisu izmijenjeni. Nemojte mijenjati nazive datoteka.

</details>

<details>

<summary>Mogu li postaviti svoje kamere MAPIR Survey3 na automatsku ekspoziciju i obrađivati slike u Chloros?</summary>

Ne. Skupovi slika kamereSurvey3 moraju imati fiksnu/zaključanu ekspoziciju, stoga nema automatske brzine zatvarača ili automatskog ISO-a. Sve slike istog modela kamere moraju imati identičnu brzinu zatvarača i ISO (ekspoziciju).

Kamere LATTICE nemaju ovo ograničenje: Chloros upravlja njihovom ekspozicijom uživo (Smart AE) i svako snimanje bilježi ekspoziciju i pojačanje koji su zapravo korišteni, što radiometrijski pipeline uzima u obzir.

</details>

<details>

<summary>Može li Chloros obrađivati ili analizirati ortomosaicne slike?</summary>

Ne. Podržane su samo pojedinačne slike kamere MAPIR, a ne spojene slike poput orto-mosaika.

</details>

<details>

<summary>Kako mogu ubrzati korak detekcije ciljeva u programu Chloros?</summary>

U tablici preglednika datoteka, prethodno odabiranje ciljnih slika u desnoj stupci reći će Chlorosu da traži kalibracijske ciljeve samo u tim slikama, čime se znatno ubrzava obrada.

</details>

<details>

<summary>Ako ću svoje slike učitati na <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> trebam li ih prvo obraditi u Chloros prije učitavanja?</summary>

Ako planirate prenijeti slike na našu internetsku platformu za obradu [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nemojte uređivati slike prije prijenosa. Cloud će izvršiti svu istu obradu, a još i više.

</details>

<details>

<summary>Hoće li MAPIR ikada podržati značajku X? Stvarno bih želio da MAPIR nudi X.</summary>

Uvijek smo zainteresirani za povratne informacije o našim proizvodima. Ako pronađete problem s našim proizvodima ili imate prijedlog kako bismo ih mogli poboljšati, molimo vas da nas [KONTAKTIRATE](https://www.mapir.camera/community/contact) i podijelite svoja razmišljanja. Većinu našeg istraživanja i razvoja usmjeravamo slušajući najveće potrebe naših korisnika.

</details>

<details>

<summary>Je li Chloros dostupan za Linux?</summary>

Da! Chloros 1.3.0 podržava Linux amd64 (x86_64) i arm64 (NVIDIA Jetson JetPack 6) putem paketa `.deb`. CLI i Python SDK u potpunosti su podržani na Linux, uključujući upravljanje LATTICE kamerom uživo i DAQ senzorima. GUI za Linux ne postoji — sva interakcija odvija se putem [CLI](CLI.md) ili [Python SDK](api-python-sdk.md). Pogledajte [Linux pregled](linux/linux-overview.md) za detalje.

</details>

<details>

<summary>Mogu li pokrenuti Chloros na NVIDIA Jetsonu?</summary>

Da! Chloros podržava obitelj NVIDIA Jetson **Orin** — Orin Nano, Orin NX i AGX Orin — koji koriste JetPack 6. Stariji modeli (Jetson Nano, TX2, Xavier NX) ne mogu pokrenuti JetPack 6 i nisu podržani u trenutnom paketu. Chloros automatski otkriva vaš model Jetson i optimizira strategiju obrade. Jetson paket koristi CUDA izradu PyTorcha iz JetPacka 6, tako da se obrada odvija na GPU-u Jetsona. Pogledajte [Vodič za NVIDIA Jetson](linux/nvidia-jetson-guide.md) za upute o postavljanju i implementaciji.

</details>

<details>

<summary>Optimizira li Chloros automatski za moj hardver?</summary>

Da! Chloros uključuje [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) koja automatski otkriva vaš CPU, GPU, RAM i (na Jetsonu) termalne senzore. Zatim odabire optimalnu strategiju obrade — od `GPU_PARALLEL` na GPU-ima s velikom memorijom do `GPU_SINGLE` na manjim do `CPU_PARALLEL` na sustavima bez CUDA GPU-a koji PyTorch može koristiti. Nije potrebna ručna konfiguracija.

</details>

<details>

<summary>Što je 4-thread pipeline za obradu?</summary>

Chloros koristi pipelined arhitekturu s 4 niti za korisnike Chloros+: Nit 1 (Detekcija) učitava slike i otkriva ciljeve za kalibraciju, Nit 2 (Kalibracija) izračunava kalibraciju reflektivnosti, Nit 3 (Obrada) izvodi GPU-akcelerirano debayering i izračun indeksa, a Nit 4 (Izvoz) zapisuje izlazne datoteke. Više slika može se istovremeno obrađivati u različitim nitima radi maksimalne propusnosti. Pogledajte [Procesna cijev](processing-architecture/processing-pipeline.md) za detalje.

</details>

<details>

<summary>Kako mogu pokrenuti dijagnostiku na svojoj instalaciji Chloros?</summary>

Koristite naredbu `selftest` za pokretanje 7-koraknog smoke testa: verzija, dostupnost porta, pokretanje backenda, povezanost APIa (`/api/test`), informacije o sustavu (`/api/system-info` — GPU/CUDA/PyTorch), prisutnost modela za uklanjanje šuma i spremnost CUDA + modela za uklanjanje šuma:

```bash
chloros-cli selftest
```

Ovo je posebno korisno na sustavima Linux za provjeru postavke GPU-a i CUDA-e. Na Jetsonu, `CUDA: False` obično znači da nedostaju CUDA i cuDNN biblioteke iz JetPacka 6: instalirajte ih pomoću `sudo apt install nvidia-jetpack`.

</details>
