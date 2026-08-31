# Podesavanje postavki projekta

Prije obrade slika važno je konfigurirati postavke projekta kako bi odgovarale zahtjevima vašeg radnog toka. Panel Postavke projekta (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) pruža sveobuhvatnu kontrolu nad kalibracijom, opcijama obrade, multispektralnim indeksima i formatima izvoza.

## Pristupanje postavkama projekta

1. Otvorite svoj projekt u Chloros
2. Kliknite ikonu **Postavke projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> u lijevoj bočnoj traci
3. Panel Postavke projekta prikazuje sve opcije konfiguracije

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Ploča Postavke projekta — Prikaz, Otkrivanje ciljeva i Obrada</p></figcaption></figure>

{% hint style="info" %}
**Postavke se automatski spremaju** s vašim projektom. Kada ponovno otvorite projekt, sve se postavke vraćaju.
{% endhint %}

***

## Brzo postavljanje za uobičajene radne procese

### Zadane postavke (preporučeno za većinu korisnika)

Zadane postavke dobro funkcioniraju za tipične radne proceseSurvey3
i LATTICE:

* ✅ **Korekcija vignete**: Omogućeno
* ✅ **Kalibracija refleksije / bilanca bijele boje**: Omogućeno (koristi ciljeve za kalibraciju reflektance i/ili podatke svjetlosnog senzora DAQ-a)
* ✅ **Metoda de-Bayera**: Standardna (Brzo, Srednja kvaliteta)
* ✅ **Format izvoza**:TIFF
(16-bitni)
* ✅ **Svi izvozni proizvodi**: Omogućeno (LATTICE automatski bilježi fan-out u debayered, pretpregled, radijancu i reflektancu)

Jednostavno uvezite svoje slike i započnite obradu s ovim zadanim postavkama.

***

## Pregled postavki projekta

Ploča Postavke projekta organizirana je u donje odjeljke. Dva dodatna odjeljka — **DAQ senzor svjetla**i**Poravnanje niza** — automatski se pojavljuju kada vaš projekt sadrži relevantne datoteke. Za potpunu dokumentaciju pogledajte [Postavke projekta](../project-settings/project-settings.md).

### Prikaz

* **Razlučivost sličice slike**: Razlučivost sličica mreže slika. Opcije:**Zadano (512 px)**,**1024 px**,**2048 px**,**Puna rezolucija**. Samo za prikaz — nikada ne utječe na obradu. Više vrijednosti izgledaju oštrije pri uvećanju, ali se sporije učitavaju.

### Detekcija ciljeva

Kontrolira kakoChloros
identificira ciljne oznake na vašim slikama.

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

**Ključne postavke:*** **Korekcija vinjetiranja**: Kompenzira zamračivanje objektiva na rubovima ✅ Preporučeno
* **Kalibracija reflektancije / balans bijele**: Kalibrira slike koristeći detektirane ciljeve (Survey3
) i/ili podatke DAQ senzora svjetlosti (LATTICE) ✅ Preporučeno
* **Debayer metoda**: Algoritam za pretvorbu RAW formata u troslojni multispektralni
* **Minimalni interval ponovne kalibracije**: Minimalno vrijeme u sekundama između upotrebe kalibracijskog cilja (zadano:**0** = koristi sve, raspon 0–3600)**Zamjenski proizvodi za nekalaribrane snimke:**Kada se kadar ne može kalibrirati reflektancijom (nema dostupnog cilja ili je kalibracija onemogućena), izvozi se kao jedan od dva zamjenska proizvoda —**točno jedan od para postoji po izvršenju**, a odabire se prekidačem za korekciju vinjete:

* **Izvoz odgovora senzora**: piše `Sensor_Response_Images` — koristi se kada je ispravak vinjete**isključen*** **Izvoz s korekcijom vignetiranja**: zapisuje `Vignette_Corrected_Images` — koristi se kada je korekcija vignetiranja**uključena**Kutijica koja nije odabrana je sivkasta. Deaktiviranje aktivne kutijice sprječava pisanje te datoteke.**LATTICE proizvodi za izvoz** (prikazani za svaki projekt; primjenjuju se na LATTICE snimke):

* **Izvoz debayered slike**: linearna debayered slika (`Debayered_Images`). Primjenjuje se naRGB
i multispektralne module.
* **Export pretpregleda**: prikaz pretpregleda (`Preview_Images`).RGB
= bijela točka (DAQ-illuminant kada je dostupan, inače svijetlo sive) + gamma; multispektralno = istezanje lažnih boja.
* **Izvoz radijance**: float32 spektralna radijanca (`Radiance_Images`, W/m²/sr/nm). Samo za multispektralne module — ne primjenjuje se naRGB
master uređaje.
* ****Izvoz reflektancije**: uint16 reflektancija (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1.0) kada `.daq` mjerenje usmjereno prema dolje ili cilj unutar kadra pokriva kadar. Samo za multispektralne module.

Sva četiri su **podrazumijevano uključena**— jedan uvezeno LATTICE sirovo okvirno snimanje raspoređuje se na svaki omogućen i primjenjiv proizvod u jednom koraku obrade. Kvadratica za potvrdu**Izvoz reflektancije** je sivkasta kada je kalibracija reflektancije isključena. Postavke čije roditeljsko prekidače ih onemogućuju uvijek su isključene i sive, s obavijesnim okvirom koji navodi prekidač koji treba promijeniti.**Napredne postavke:*** **Offset vremenske zone senzora svjetla**: Sati od UTC-a za usklađivanje vremena senzora svjetla (zadano: 0, raspon od -12 do +12)
* **Primijeni PPK korekcije**: Koristi GPS/pin podatke o ekspoziciji iz datoteka `.daq` (podrazumijevano: isključeno)
* **Pin za ekspoziciju 1/2**: Dodjeljuje kamere pinovima za ekspoziciju za postavke s dvostrukom kamerom

{% hint style="info" %}
**Razina unosa za LATTICE je automatska.** Snimke za LATTICE nose svoju razinu obrade u XMP metapodacima, a obrada uvijek započinje s sirovim okvirom — nema ništa za konfigurirati u GUI-ju. (ZastavicaCLI
`--input-level` postoji kao napredna opcija za snimke s izgubljenim metapodacima; pogledajte [CLI
referencu](../reference/cli-reference.md).)
{% endhint %}

### Metoda debayeringa

Trenutno nudimo 2 metode debayeringa uChloros
:

#### Standardno (Brzo, Srednja kvaliteta)

Standardni debayer obrađuje brzo, ali prikazuje šum u boji uzrokovan debayeringom, što rezultira manje preciznim i bučnijim slikama.

#### S obzirom na teksturu (sporo, najviša kvaliteta) \[Samo zaChloros
+]

Texture Aware koristi visokokvalitetni debayer s obzirom na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji/strojnom učenju (AI/ML), koji uklanja gotovo sav šum pri detekciji boja. Model zahtijeva GPU memoriju (VRAM) za rad: s **7 GB ili više VRAM-a** može obrađivati više slika istovremeno; ispod 7 GB radi s jednom slikom odjednom (značajno sporije). Pogledajte [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Uhvaćanja s mrežom (LATTICE) uvijek koriste standardnu demosaikizaciju.** Ne postoji model osjetljiv na teksturu (Texture Aware) treniran za LATTICE, stoga se ta opcija ne nudi za LATTICE slike — slikeSurvey3
u istom projektu i dalje je mogu koristiti.
{% endhint %}

### Indeks (Multispektralni indeksi)

Konfigurirajte koje indekse vegetacije izračunati i izvesti. Spuštajući izbornik u GUI-ju nudi **27 unaprijed definiranih formula indeksa**.**Kako dodati indekse:**

1. Kliknite gumb**&quot;Dodaj indeks&quot;**

2. Odaberite indeks iz padajućeg izbornika (NDVI
,NDRE
,GNDVI
itd.)
3. Konfigurirajte postavke vizualizacije (LUT boje, rasponi vrijednosti)
4. Dodajte više indeksa prema potrebi

**Popularni indeksi:*** **NDVI**: Opće stanje vegetacije (najčešći)
* **NDRE**: Rano otkrivanje stresa pomoćuRedEdge
* **GNDVI**: Osjetljiv na koncentraciju klorofila
* **OSAVI**: Dobro funkcionira s vidljivim tlom
* **EVI**: Područja s visokim indeksom površine lišća (LAI
)

**Prilagođene formule:**

* Kreirajte prilagođene formule multispektralnih indeksa pomoću matematičkih operacija na svim kanalima slike
* Spremite prilagođene formule za ponovnu upotrebu
* Prilagođene formule su značajkaChloros
+; dostupnost ovisi o vašem paketu usluge

Za sve dostupne indekse i formule — uključujući koja su imena dostupna samo u GUI-ju, a koja također rade uCLI
/SDK
— pogledajte [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

### Izvoz

Kontrole formata izlazne datoteke.

**Dostupni formati**(podešavanje:**Kalibrirani format slike**, zadano**TIFF
(16-bitni)**):

* **TIFF
(16-bitni)**: Preporučuje se za GIS i znanstvenu analizu
* **TIFF
(32-bita, postotak)**: Pokretne zarezne vrijednosti
* **PNG
(8-bita)**: Kompresija bez gubitka za vizualizaciju
* **JPG (8-bita)**: Najmanje datoteke, kompresija s gubitkom

Izlazne datoteke spremaju se u podmapu projekta, grupirane po kameri i formatu: `<project>/<camera>/<format>/<Product>_Images/`. Radiance se **uvijek** sprema kao float32 u mapu `tiff32`, bez obzira na ovu postavku. Izvozne datoteke zadržavaju izvorni naziv datoteke — mapa identificira proizvod. Pogledajte [Završavanje obrade](finishing-the-processing.md) za potpuno stablo izlaznih datoteka.

{% hint style="warning" %}
**Čitanje vrijednosti reflektancije**: DN koji znači ρ = 1,0 ovisi o izvornoj kameri — LATTICE koristi 32768 (zabilježeno kao XMP `Chloros:PixelScale`), aSurvey3
koristi 65535. Pročitajte oznaku umjesto da pretpostavljate konstantu. Pogledajte [Formate izlaznih slika](../output-image-formats.md).
{% endhint %}

### DAQ senzor svjetla

Ovaj odjeljak navodi svaku DAQ downwelling datoteku (`.daq` / `.csv`) u vašem projektu, po jedna red po datoteci, prikazujući model senzora, naziv datoteke i korekciju **kapice** difuzora koja je na snazi za tu datoteku.

* **Preuzimanje korekcije (sve datoteke)**: jedinstven padajući izbornik na razini cijelog projekta.**Auto** (zadano) koristi snimljenu korekciju svake datoteke — pretpostavlja se sunčano vrijeme ako ništa nije snimljeno, budući da svi DAQ-oviMAPIR
dolaze sa korektorom za sunčano vrijeme. Odabir kapa nadjačava svaku datoteku: sirovi zapisi se korigiraju njome, a zapisi koji već sadrže kap se ponovno referenciraju (uklonjena je prethodno snimljena korekcija i primijenjena je odabrana kapa).
* Redovi upozoravaju kada je zabilježena granica bila pretpostavljena zadana vrijednost čvorišta umjesto potvrđena od strane operatera i kada odabrana granica nema profil za taj model uređaja (nadjačavanje se odbija za tu datoteku).

Snimke DAQ-a napravljene na kartici Light Sensors automatski se dodaju otvorenom projektu, a uvezene datoteke `.daq` / `.csv` pojavljuju se ovdje čim budu dodane.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Donji dio postavki projekta — indeks, format izvoza, odjeljak DAQ senzora svjetla i kontrole predloška/mape projekta</p></figcaption></figure>### Poravnanje niza

Ovaj odjeljak se prikazuje **samo** kada barem jedna slika u projektu sadrži transformaciju poravnanja između modula koju nizovi LATTICE utiskuju pri snimanju (`Chloros:Alignment*` XMP). Prikazuje koliko slika ima oznake i koja je kamera referentna, s ovim kontrolama:

* **Primijeni poravnanje niza** (zadano: uključeno): deformira svaki obrađeni proizvod (debayereni / pretpregled / radijanca / reflektancija / indeks) u zajedničku referentnu geometriju niza. Isključeno = izvoz u nativnoj geometriji senzora.
* **Orezivanje na zajednički preklap** (zadano: uključeno): orezuje poravnate izvoze na regiju koju dijele svi moduli, tako da svaki pojas ima isti otisak. Isključeno zadržava cijelo platno senzora (crno popunjavanje izvan izvora).
* **Resampling**:**Bilinear (glatko, zadano)**,**Najbliže (očuva točne vrijednosti)**— bez miješanja između piksela, za strogu radiometrijsku analizu — ili**Kubično (najostreće)**.***

## Spremanje i učitavanje postavki

### Spremi predložak projekta

Stvorite ponovno upotrebljive predloške za dosljedne tijekove rada:

1. Konfigurirajte sve željene postavke na ploči Postavke projekta
2. Pomaknite se do odjeljka **&quot;Spremi predložak projekta&quot;** na dnu
3. Unesite opisni naziv predloška (npr. &quot;Survey3N

\_RGN\_Agriculture&quot;)
4. Kliknite na ikonu spremanja

**Prednosti:**

* Primijenite identične postavke na više projekata
* Dijelite konfiguracije s članovima tima
* Održavajte dosljednost za ponovljena istraživanja

### Učitavanje predloška za novi projekt

Prilikom izrade novog projekta:

1. Odaberite **&quot;Novi projekt&quot;** iz glavnog izbornika
2. Odaberite predložak projekta u opcionalnom odabiraču predložaka
3. Svi se postavke iz predloška automatski primjenjuju

### Radni direktorij

Postavka **&quot;Radni direktorij&quot;** određuje gdje se novi projekti zadano stvaraju:

* **Zadana lokacija**: `C:\Users\[Username]\Chloros Projects`
* **Promijeni lokaciju**: Kliknite ikonu za uređivanje i odaberite novu mapu
* **Zajednički sCLI

-om**: `chloros-cli` koristi istu zadanu postavku mape projekta
* **Kada promijeniti**:
  * Mrežni pogon za timsku suradnju
  * Drugi pogon s više prostora za pohranu
  * Organizirana struktura mapa po godini/klijentu

***

## Postavljanje PPK (Post-obrada kinematike)

Ako koristite DAQ snimačeMAPIR

s GPS-om za preciznu geolokaciju:

### Preduvjeti

* DAQMAPIR

s GPS (GNSS) modulom
* .daq zapisnik s unosima na pinovima za ekspoziciju
* Kamera povezana na pinove za ekspoziciju DAQ-a tijekom snimanja

### Koraci konfiguracije

1. Stavite .daq log datoteku u mapu projekta
2. U postavkama projekta omogućite potvrdni okvir **&quot;Primijeni PPK korekcije&quot;**

3. Po potrebi postavite**&quot;Offset vremenske zone senzora svjetla&quot;** (zadano: 0 za UTC)
4. Dodijelite kamere ekspozicijskim pinovima:
   * **Jedna kamera**: Automatski dodijeljena na Pin 1
   * **Dvije kamere**: Ručno dodijelite svaku kameru na odgovarajući pin**Dodjela ekspozicijskog pina:*** **Izlaganje Pin 1**: Odaberite model kamere iz padajućeg izbornika
* **Izlaganje Pin 2**: Odaberite drugu kameru ili &quot;Ne koristi&quot;
* Ista kamera ne može biti dodijeljena oba pina

{% hint style="warning" %}
**Važno**: Pinovi za ekspoziciju moraju biti ispravno dodijeljeni svojim odgovarajućim kamerama. Neispravna dodjela rezultirat će pogrešnim podacima o geolokaciji.
{% endhint %}

***

## Napredni scenariji

### Projekti s više kamera

Prilikom obrade slika s višeMAPIR

kamera u jednom projektu:

1.Chloros

automatski detektira svaki model kamere (isto zaSurvey3

i LATTICE)
2. Svaka kamera dobiva odgovarajuće profile obrade i vlastito stablo izlaznih mapa
3. PPK: Ručno dodijelite svakuSurvey3

kameru na odgovarajući pin za ekspoziciju
4. Sve kamere koriste isti format izvoza i indekse

**Primjeri**:Survey3W

RGN

+Survey3N

OCN

konfiguracija s dvostrukom kamerom, ili LATTICE niz koji miješaRGB

master s uskopojasnim modulima

### Time-Lapse ili višedatumska snimanja

Za ponovljena snimanja istog područja tijekom vremena:

1. Izradite predložak sa svojim standardnim postavkama
2. Koristite dosljedan raspored kalibracijskog cilja svake sesije
3. Obradite svaki datum kao zaseban projekt
4. Koristite identične postavke za usporedive rezultate
5. Izvezite u istom formatu za vremensku analizu

### Veliki skupovi podataka

Za projekte s mnogo slika (500+):

* Razmislite o podjeli na manje projekte prema datumu ili području
* KoristiteChloros

+ paralelnu obradu za brže rezultate
* Razmislite oCLI

iliAPI

za automatizaciju serije
* Podesite minimalni interval ponovne kalibracije kako biste smanjili vrijeme detekcije cilja

***

## Provjera vaših postavki

Prije početka obrade, provjerite ove ključne postavke:

* [ ] Model kamere ispravno prepoznat u pregledniku datoteka
* [ ] Omogućeno ispravljanje vinjetiranja
* [ ] Omogućena kalibracija reflektancije
* [ ] ZaSurvey3

: uvezena i provjerena barem jedna slika kalibracijskog cilja; za LATTICE: prisutan cilj i/ili snimka usmjerene emisije `.daq`
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
