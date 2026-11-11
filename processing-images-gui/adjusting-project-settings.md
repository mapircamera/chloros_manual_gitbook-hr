# Podesavanje postavki projekta

Prije obrade vaših slika, važno je konfigurirati postavke projekta kako bi odgovarale zahtjevima vašeg radnog procesa. Panel **Postavke projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> pruža sveobuhvatnu kontrolu nad kalibracijom, opcijama obrade, multispektralnim indeksima i formatima izvoza.

## Pristupanje postavkama projekta

1. Otvorite svoj projekt u Chlorosu
2. Kliknite ikonu **Postavke projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> u lijevoj bočnoj traci
3. Panel Postavke projekta prikazuje sve opcije konfiguracije

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Ploča Postavke projekta — Prikaz, Otkrivanje ciljeva i Obrada</p></figcaption></figure>

{% hint style="info" %}
**Postavke se automatski spremaju** s vašim projektom. Kada ponovno otvorite projekt, sve se postavke vraćaju.
{% endhint %}

***

## Brzo postavljanje za uobičajene tijekove rada

### Zadane postavke (preporučeno za većinu korisnika)

Zadane postavke dobro funkcioniraju za tipične tijekove rada *Survey3
* i *LATTICE*:

* ✅ **Korekcija vinjete**: Omogućeno
* ✅ **Kalibracija reflektancije / balans bijele**: Omogućeno (koristi MAPIR ciljeve i/ili podatke DAQ senzora svjetlosti)
* ✅ **Metoda debayera**: Standardno (Brzo, Srednja kvaliteta)
* ✅ **Format izvoza**: TIFF (16-bitni) (TIFF (8-bitni) za projekt koji je u potpunosti 8-bitni)
* ✅ **Svi izvozni proizvodi**: Omogućeno (LATTICE automatski bilježi fan-out u debayered, prikaz, radijancu i reflektancu)

Jednostavno uvezite svoje slike i započnite obradu s ovim zadanim postavkama.

***

## Pregled postavki projekta

Ploča Postavke projekta organizirana je u donje odjeljke. Dva dodatna odjeljka — **DAQ senzor svjetla**i**Poravnanje niza** — pojavljuju se automatski kada vaš projekt sadrži relevantne datoteke. Za potpunu dokumentaciju pogledajte [Postavke projekta](../project-settings/project-settings.md).

### Prikaz

* **Rješenja sličica mreže slika**: Rješenja sličica mreže slika. Opcije:**Zadano (512 px)**,**1024 px**,**2048 px**,**Puna rezolucija**. Samo za prikaz — nikada ne utječe na obradu. Više vrijednosti izgledaju oštrije pri uvećanju, ali se sporije učitavaju.

### Detekcija ciljeva

Kontrolira kako Chloros identificira kalibracijske ciljeve na vašim slikama.

**Ključne postavke:*** **Minimalna površina uzorka za kalibraciju (px)**: Prag veličine za detekciju ciljeva (zadano:**25**, raspon 0–10000)
* **Minimalno grupiranje ciljeva (0-100)**: Prag sličnosti za grupiranje regija ciljeva (zadano:**60**)**Kada prilagoditi:**

* Povećajte površinu uzorka ako dolazi do lažnih detekcija
* Smanjite ako se ciljevi ne detektiraju
* Prilagodite grupiranje ako se ciljevi dijele na više detekcija

{% hint style="info" %}
Ove postavke su sivkaste kada je **Kalibracija reflektancije / balans bijele** isključena — s njom isključenom, detekcija ciljeva se uopće ne pokreće.
{% endhint %}

### Obrada

Glavne opcije za obradu slika i kalibraciju.

**Ključne postavke:*** **Korekcija vinjete**: Kompenzira zamračivanje objektiva na rubovima ✅ Preporučeno
* **Kalibracija reflektancije / balans bijele**: Kalibrira slike koristeći otkrivene ciljeve (Survey3) i/ili podatke DAQ senzora svjetla (LATTICE) ✅ Preporučeno
* **Metoda de-Bayera**: Algoritam za pretvorbu RAW formata u troslojni multispektralni
* **Minimalni interval ponovne kalibracije**: Minimalno vrijeme u sekundama između upotrebe kalibracijskog cilja (zadano:**0** = koristi sve, raspon 0–3600)**Zamjenski proizvodi za nekalaribrane slike:**Kada se kadar ne može kalibrirati reflektancijom (cilj nedostupan ili je kalibracija onemogućena), izvozi se kao jedan od dva zamjenska proizvoda —**točno jedan od para postoji po pokretanju**, a odabire se prekidačem za korekciju vinjete:

* **Izvoz odgovora senzora**: zapisuje `Sensor_Response_Images` — koristi se kada je korekcija vinjete**isključena*** **Izvoz s korekcijom vignete**: zapisuje `Vignette_Corrected_Images` — koristi se kada je korekcija vignete**uključena**Kutijica koja nije aktivna je sivkasta. Odabirom one aktivne sprječava se pisanje te datoteke.**Proizvodi za izvoz LATTICE-a** (prikazani za svaki projekt; primjenjuju se na snimke LATTICE-a):

* **Izvoz de- Bayerizirane slike**: linearna debayered slika (`Debayered_Images`): uklonjen je mračni offset kalibracijskog paketa, zatim se vrši demosaicing, bez flat polja ili unmixa — ista datoteka koju snimanje sprema na svojoj debayered razini — u nativnoj orijentaciji senzora. Primjenjivo na RGB i multispektralne module.
* **Izvoz pretpregleda**: pretpregled za prikaz (`Preview_Images`). RGB = balans bijele (DAQ-illuminant kada je dostupan, inače svijetlo sivo) + gamma; multispektralno = istezanje lažnih boja.
* **Izvoz radijance**: float32 spektralna radijanca (`Radiance_Images`, W/m²/sr/nm). Samo za multispektralne module — ne primjenjuje se na RGB master uređaje.
* ****Izvoz reflektancije**: uint16 reflektancija (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1.0) kada `.daq` nizvodna mjera ili cilj unutar kadra pokriva kadar. Samo za višespektralne module.

Sva četiri su **podrazumijevano uključena**— jedan uvezeno LATTICE sirovo okvirno snimanje raspoređuje se na svaki omogućen i primjenjiv proizvod u jednom procesu obrade. Kvadratica za**Izvoz reflektancije** je sivkasta kada je kalibracija reflektancije isključena. Postavke čije roditeljsko prekidače ih onemogućuju uvijek su zasivljene s alatnom napomenom koja navodi prekidač koji treba promijeniti.**Napredne postavke:*** **Offset vremenske zone senzora svjetla**: Sati od UTC-a za usklađivanje vremena senzora svjetla (zadano: 0, raspon od -12 do +12)
* **Primijeni PPK korekcije**: Koristi GPS/pin podatke o ekspoziciji iz datoteka `.daq` (podrazumijevano: isključeno)
* **Pin za ekspoziciju 1/2**: Dodjeljuje kamere pinovima za ekspoziciju za postavke s dvostrukom kamerom

{% hint style="info" %}
**Razina unosa za LATTICE je automatska.** Snimke za LATTICE nose svoju razinu obrade u XMP metapodacima, a obrada uvijek ulazi u proces na sirovom kadru — nema ništa za konfigurirati u GUI-ju. (Zastavica CLI `--input-level` postoji kao prečac za napredne korisnike za snimke s izgubljenim metapodacima; pogledajte [CLI referencu](../reference/cli-reference.md).)
{% endhint %}

### Metoda debayeringa

Trenutno nudimo 2 metode debayeringa u aplikaciji Chloros:

#### Standardna (Brza, prosječne kvalitete)

Standardni debayer obrađuje brzo, ali pokazuje šum u boji, što rezultira manje preciznim i bučnijim slikama.

#### S obzirom na teksturu (sporo, najviša kvaliteta) \[Samo za Chloros+]

Metoda s obzirom na teksturu koristi visokokvalitetni debayer osjetljiv na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji/strojnom učenju (AI/ML), koji uklanja gotovo sav šum pri debayeranju. Model zahtijeva GPU memoriju (VRAM) za rad: s **7 GB ili više VRAM-a** može obrađivati više slika istovremeno; ispod 7 GB radi sa jednom slikom odjednom (značajno sporije). Pogledajte [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Uvijek se koristi standardna demosaika.**Ne postoji model osjetljiv na teksturu (Texture Aware) treniran za LATTICE, stoga ta opcija nije dostupna za LATTICE snimke — slike iz**Survey3 a** u istom projektu i dalje je mogu koristiti.
{% endhint %}

### Indeks (Višespektralni indeksi)

Konfigurirajte koje će se indekse vegetacije izračunati i izvesti. Spuštajući izbornik korisničkog sučelja nudi **27 unaprijed definiranih formula indeksa**.**Kako dodati indekse:**

1. Kliknite gumb**"Dodaj indeks"**

2. Odaberite indeks iz padajućeg izbornika (NDVI, NDRE, GNDVI itd.)
3. Konfigurirajte postavke vizualizacije (LUT boje, rasponi vrijednosti)
4. Dodajte više indeksa prema potrebi

**Popularni indeksi:*** **NDVI**: Opće zdravlje vegetacije (najčešći)
* **NDRE**: Rana detekcija stresa pomoću **GNDVI*** **GNDVI**: Osjetljiv na koncentraciju klorofila
* **OSAVI**: Dobro funkcionira s vidljivim tlom
* **EVI**: Područja s visokim indeksom površine lista (LAI)

**Prilagođene formule:**

* Kreirajte prilagođene multispektralne formule indeksa pomoću matematičkih operacija na svim kanalima slike
* Spremite prilagođene formule za ponovnu upotrebu
* Prilagođene formule su značajka usluge Chloros+; dostupnost ovisi o vašem paketu usluge

Za sve dostupne indekse i formule — uključujući koja su imena dostupna samo u GUI-ju, a koja također rade u CLI/SDK — pogledajte [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

### Izvoz

Upravlja formatom izlazne datoteke.

**Dostupni formati**(podešavanje:**Kalibrirani format slike**, zadano**TIFF (16-bitni)**):

* **TIFF (16-bitni)**: Preporučuje se za GIS i znanstvenu analizu
* **TIFF (8-bit)**: 8-bitni TIFF
* **TIFF (32-bit, Postotak)**: Pokretne zarezne vrijednosti
* **PNG (8-bit)**: Kompresija bez gubitka za vizualizaciju
* **JPG (8-bit)**: Najmanje datoteke, kompresija s gubitkom

Popis prikazuje što slike projekta mogu sadržavati: Chloros ne povećava dubinu bita. Sve 8-bitne slike nude samo TIFF (8-bit), PNG (8-bit) i JPG (8-bit), a zadano su TIFF (8-bit). U mješovitom projektu svako snimanje izvozi se u svojoj dubini ili nižoj. Radiance ostaje 32-bitni float.

Izlazne datoteke spremaju se u podmapu projekta, grupirane po kameri i formatu: `<project>/<camera>/<format>/<Product>_Images/`. Radiance se **uvijek** sprema kao float32 u mapu `tiff32`, bez obzira na ovu postavku. Izvožene datoteke zadržavaju izvorni naziv datoteke — mapa identificira proizvod. Multispektralni izvoz LATTICE-a također može zapisati maske za ograničavanje po pikselu u mapu `clamp_masks/` pokraj mapa s proizvodima. Pogledajte [Završavanje obrade](finishing-the-processing.md) za potpuno stablo izlaza.

{% hint style="warning" %}
**Čitanje vrijednosti reflektancije**: DN koji znači ρ = 1.0 ovisi o izvornoj kameri — LATTICE koristi 32768 (označeno kao XMP `Chloros:PixelScale`), a Survey3 koristi 65535. Pročitajte oznaku umjesto da pretpostavljate konstantu. Pogledajte [Formati izlaznih slika](../output-image-formats.md).
{% endhint %}

### DAQ senzor svjetla

Ovaj odjeljak navodi svaku DAQ downwelling datoteku (`.daq` / `.csv`) u vašem projektu, po jedan redak za svaku datoteku, prikazujući model senzora, naziv datoteke i **kapu** difuzora korekcija koja je na snazi za tu datoteku.

* **Preuzimanje kapice (sve datoteke)**: jedinstven padajući izbornik za cijeli projekt.**Auto** (zadano) koristi snimljeni poklopac svake datoteke — pretpostavlja se sunčano vrijeme ako ništa nije snimljeno, ali DAQ-U, DAQ-M i DAQ-E isporučuju se bez poklopca, a Chloros ne može vidjeti što je postavljeno, stoga procesiranje upozorava na pretpostavljeni poklopac: odaberite poklopac koji je bio postavljen. Odabir korekcije poništava svaku datoteku osim DAQ-E-S, čiji je difuzor dio njegove kalibracije: sirovi zapisi se korigiraju njime, a zapisi koji već sadrže korekciju ponovno se referenciraju (korekcija snimljena vraća se na početno stanje, a primjenjuje se odabrana korekcija).
* Redovi upozoravaju kada je snimljeni kap bio pretpostavljeni zadani kap čvora, a ne onaj koji je potvrdio operater, te kada odabrani kap nema profil za taj model uređaja (nadjačavanje se odbija za tu datoteku).

Zapisi DAQ-a napravljeni na kartici Light Sensors automatski se dodaju otvorenom projektu, a uvezene datoteke `.daq` / `.csv` pojavljuju se ovdje čim budu dodane.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Donji dio postavki projekta — indeks, format izvoza, odjeljak DAQ senzora svjetla i kontrole predloška/mape projekta</p></figcaption></figure>### Poravnanje niza

Ovaj odjeljak se prikazuje **samo** kada barem jedna slika u projektu sadrži transformaciju poravnanja između modula koju nizovi LATTICE utiskuju pri snimanju (`Chloros:Alignment*` XMP). Prikazuje koliko slika ima oznake i koja je kamera referentna, s ovim kontrolama:

* **Primijeni poravnanje niza** (zadano: uključeno): deformira svaki obrađeni proizvod (debayereni / pretpregled / radijanca / reflektancija / indeks) u zajedničku referentnu geometriju niza. Isključeno = izvoz u nativnoj geometriji senzora.
* **Orezivanje na zajednički preklop** (zadano: uključeno): orezuje poravnate izvoze na regiju koju dijele svi moduli, tako da svaki pojas ima isti otisak. Isključeno zadržava cijelo platno senzora (crno popunjavanje izvan izvora).
* **Resampling**:**Bilinear (glatko, zadano)**,**Najbliže (očuvaj točne vrijednosti)**— bez miješanja između piksela, za strogu radiometrijsku analizu — ili**Kubično (najostreće)**.***

## Spremanje i učitavanje postavki

### Spremi predložak projekta

Stvorite ponovno upotrebljive predloške za dosljedne tijekove rada:

1. Konfigurirajte sve željene postavke na ploči Postavke projekta
2. Pomaknite se do odjeljka **"Spremi predložak projekta"** na dnu
3. Unesite opisni naziv predloška (npr. "Survey3N_RGN_Agriculture")
4. Kliknite na ikonu spremanja

**Prednosti:**

* Primijenite identične postavke na više projekata
* Dijelite konfiguracije s članovima tima
* Održavajte dosljednost za ponovljena istraživanja

### Učitavanje predloška za novi projekt

Prilikom izrade novog projekta:

1. Odaberite **"Novi projekt"** iz glavnog izbornika
2. Odaberite predložak projekta u opcionalnom odabiraču predložaka
3. Svi se postavke iz predloška automatski primjenjuju

### Radni direktorij

Postavka **"Radni direktorij"** određuje gdje se novi projekti zadano stvaraju:

* **Zadana lokacija**: `Documents\MAPIR\Chloros Projects` u vašem mapi Dokumenti sustava Windows (obično `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; slijedi mapu Dokumenti ako je OneDrive ili putujući profil premjesti)
* **Promijeni lokaciju**: Kliknite ikonu uređivanja i odaberite novu mapu
* **Podijeljeno s CLI-om**: `chloros-cli` koristi istu zadanu postavku mape projekta
* **Držite putanje kratkima u Windows u**: ako su isključene dugim putanjama u Windows u, obrada će odbiti projektnu mapu dublju od otprilike 130 znakova — pogledajte [Pokretanje obrade](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Kada promijeniti**:
  * Mrežni pogon za timsku suradnju
  * Drugi pogon s više prostora za pohranu
  * Organizirana struktura mapa po godini/klijentu

***

## Postavljanje PPK (Post-Processed Kinematic)

Ako koristite MAPIR DAQ snimače s GPS-om za preciznu geolokaciju:

### Preduvjeti

* MAPIR DAQ s GPS (GNSS) modulom
* .daq zapisnik s unosima za pune ekspozicije
* Kamera povezana s DAQ pinovima za ekspoziciju tijekom snimanja

### Koraci konfiguracije

1. Stavite .daq zapisnik u mapu projekta
2. U postavkama projekta omogućite potvrdni okvir **"Primijeni PPK korekcije"**

3. Postavite**"Offset vremenske zone senzora svjetla"** ako je potrebno (zadano: 0 za UTC)
4. Dodijelite kamere pinovima za ekspoziciju:
   * **Jedna kamera**: Automatski dodijeljena na Pin 1
   * **Dvije kamere**: Ručno dodijelite svaku kameru na odgovarajući pin**Dodjela pina za ekspoziciju:*** **Pin za ekspoziciju 1**: Odaberite model kamere iz padajućeg izbornika
* **Pin za ekspoziciju 2**: Odaberite drugu kameru ili "Ne koristi"
* Ista kamera ne može biti dodijeljena oba pina

{% hint style="warning" %}
**Važno**: Pinovi za ekspoziciju moraju biti ispravno dodijeljeni svojim odgovarajućim kamerama. Neispravna dodjela rezultirat će pogrešnim podacima o geolokaciji.
{% endhint %}

***

## Napredni scenariji

### Projekti s više kamera

Prilikom obrade slika s više MAPIR kamera u jednom projektu:

1. Chloros automatski detektira svaki model kamere (i Survey3 i LATTICE)
2. Svaka kamera dobiva odgovarajuće profile obrade i vlastito stablo izlaznih mapa
3. PPK: Ručno dodijelite svaku kameru Survey3 ispravnom pinu ekspozicije
4. Sve kamere koriste isti format izvoza i indekse

**Primjeri**: konfiguracija s dvostrukom kamerom Survey3W RGN + Survey3N OCN, ili LATTICE niz koji miješa RGB glavnu kameru sa širokopojasnim modulima

### Vremenski snimci ili snimanja s više datuma

Za ponovljena snimanja istog područja tijekom vremena:

1. Izradite predložak sa svojim standardnim postavkama
2. Koristite dosljedan postavljanje kalibracijskog cilja svake sesije
3. Obradite svaki datum kao zaseban projekt
4. Koristite identične postavke za usporedive rezultate
5. Izvezite u istom formatu za vremensku analizu

### Veliki skupovi podataka

Za projekte s mnogo slika (500+):

* Razmislite o podjeli na manje projekte prema datumu ili području
* Koristite paralelnu obradu pomoću Chloros+ za brže rezultate
* Razmislite o korištenju CLI ili API za automatizaciju u serijama
* Podesite minimalni interval ponovne kalibracije kako biste smanjili vrijeme detekcije cilja

***

## Provjera vaših postavki

Prije početka obrade, provjerite ove ključne postavke:

* [ ] Model kamere ispravno prepoznat u pregledniku datoteka
* [ ] Omogućeno ispravljanje vinjete
* [ ] Omogućena kalibracija reflektancije
* [ ] Za Survey3: uvezena i provjerena barem jedna slika kalibracijskog cilja; za LATTICE: prisutan cilj i/ili snimka usmjerena prema dolje `.daq`
* [ ] Dodani željeni multispektralni indeksi
* [ ] Format izvoza prikladan za vaš radni tijek
* [ ] PPK postavke konfigurirane (ako koristite .daq s događajima ekspozicije)

***

## Sljedeći koraci

Nakon što su vaše postavke konfigurirane:

1. **Označite slike kalibracijskih ciljeva** - Pogledajte [Odabir slika ciljeva](choosing-target-images.md)
2. **Pokrenite obradu** - Pogledajte [Pokretanje obrade](starting-the-processing.md)
3. **Pratite napredak** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)

Za potpune detalje o svim dostupnim postavkama pogledajte referentnu dokumentaciju [Postavke projekta](../project-settings/project-settings.md).
