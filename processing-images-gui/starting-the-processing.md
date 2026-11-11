# Pokretanje obrade

Nakon što ste uvezli svoje slike, označili kalibracijske ciljeve i konfigurirali postavke projekta, spremni ste za početak obrade. Ova stranica vodi vas kroz pokretanje procesne linije za obradu Chloros.

## Kontrolna lista za predobradu

Prije nego što kliknete gumb Pokreni, provjerite je li sve spremno:

* [ ] **Datoteke uvezene** - Sve slike se prikazuju u pregledniku datoteka
* [ ] **Slike ciljeva označene** - Provjerena kolona "Target" za slike za kalibraciju (ili uvezena snimka `.daq` za LATTICE)
* [ ] **Modeli kamera otkriveni** - stupac Modela kamere prikazuje ispravne kamere
* [ ] **Postavke konfigurirane** - Postavke projekta pregledane i prilagođene
* [ ] **Indeksi odabrani** - Poželjni multispektralni indeksi dodani (ako je potrebno)
* [ ] **Odabran format izvoza** - Izlazni format prikladan za vaš radni tijek

{% hint style="info" %}
**Savjet**: Kliknite na nekoliko slika u pregledniku datoteka kako biste provjerili jesu li ispravno učitane prije obrade.
{% endhint %}

***

## Pokretanje obrade

### Pronađite gumb za pokretanje

Gumb za pokretanje/reprodukciju nalazi se u gornjoj zaglavnoj traci programa Chloros:

* Položaj: Gornji središnji dio prozora
* Ikona: **Gumb za reprodukciju/pokretanje** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stanje: Gumb je omogućen (svijetao) kada je spreman za obradu

### Kliknite za pokretanje

1. Kliknite **gumb za reprodukciju/pokretanje** u gornjoj zaglavnici
2. Obrada započinje odmah
3. Tijekom obrade gumb postaje **gumb za zaustavljanje**

4. Traka napretka se ažurira i prikazuje status obrade

{% hint style="success" %}
**Početak obrade**: Nakon klika, Chloros automatski obavlja sve korake obrade – detekciju cilja, debayering, kalibraciju, izračun indeksa i izvoz. Automatski detektira je li vaš projekt Survey3, LATTICE ili mješavina te primjenjuje odgovarajući tijek obrade na svaku kameru.
{% endhint %}

***

## Razumijevanje načina obrade

Chloros radi u dva različita načina obrade ovisno o vašoj licenci:

### Besplatni način (sekvencijalna obrada)

**Dostupno svim korisnicima**

**Kako radi:**

* Prerađuje slike jednu po jednu, sekvencijalno
* Jednojezgrena obrada
* Manja potrošnja memorije

**Traka napretka prikazuje 2 faze:**

1.**Detekcija ciljeva** - Skeniranje za kalibracijske ciljeve
2. **Obrada** - Primjena kalibracije i izvoz slika**Vrijeme obrade:**

* Značajno sporije od paralelnog načina rada Chloros+
* Pogodno za male do srednje velike skupove podataka (< 200 slika)

### Način rada Chloros+ (paralelna obrada)

**Zahtijeva Chloros+ licencu**

**Kako radi:**

* Istovremeno obrađuje više slika koristeći [4-niti procesnu liniju](../processing-architecture/processing-pipeline.md)
* [Dinamička prilagodba izračuna](../processing-architecture/dynamic-compute-adaptation.md) automatski odabire optimalnu strategiju za vaš hardver na početku izvođenja
* Akceleracija GPU-om (CUDA) s NVIDIA grafičkim karticama
* **Broj radnika je hardverski prilagodljiv**: GPU strategije pokreću**2-4 istovremena radnika** (skalirano prema VRAM-u — 3 na GPU-u od 2-12 GB, 2 do 4 na GPU-u od 12+ GB); Sustavi s isključivo CPU-om pokreću jednog radnika po fizičkom jezgri, umanjen za jedan**Traka napretka prikazuje 4 faze** (koje odgovaraju 4 pipeline niti):

1. **Detektiranje** (Nit 1) - Pronalaženje ciljeva za kalibraciju
2. **Analiziranje** (Nit 2) - Pregledavanje metapodataka slike i izračunavanje kalibracije
3. **Kalibriranje** (Nit 3) - Debayering, korekcija vinjete, kalibracija, izračun indeksa
4. **Izvoz** (Nit 4) - Pohrana obrađenih slika i indeksa**Interakcija s trakom napretka:*** **Pređite mišem** preko trake za prikaz detaljnog padajućeg panela s 4 faze
* **Kliknite** na traku napretka kako biste zaključali padajući panel na mjestu
* **Ponovno kliknite** kako biste otključali i sakrili panel**Vrijeme obrade:**

* Značajno brže od besplatnog načina rada
* Akceleracija GPU-a dodatno poboljšava brzinu

{% hint style="info" %}
**Brzina programa Chloros+**: Paralelna obrada može biti 5-10 puta brža od sekvencijalnog načina rada za velike skupove podataka. Projekt od 500 slika koji u slobodnom načinu traje 2 sata može se dovršiti za 15-20 minuta s programom Chloros+.
{% endhint %}

***

## Što se događa tijekom obrade

### Faza 1: Detekcija cilja

**Što Chloros radi:**

* Skenira slike koje ste označili u stupcu Cilj (sve slike samo ako nijedna nije označena)
* Identificira kalibracijske ploče u svakom cilju
* Izvlači vrijednosti reflektancije iz panela cilja
* Bilježi vremenske oznake cilja za zakazivanje kalibracije

**Trajanje:** 1-30 sekundi (s označenim ciljevima), 5-30+ minuta (neoznačeno)

### Faza 2: Debayering (konverzija RAW)

**Što Chloros radi:**

* Pretvara podatke u RAW Bayerovom uzorku u potpune slike s 3 kanala (LATTICE mono moduli ostaju jednobandni — de-bayering za njih se preskače uz napomenu u zapisniku)
* Primjenjuje odabrani algoritam de-mosaicinga
* Održava maksimalnu kvalitetu i detalje slike

**Trajanje:** Ovisi o broju slika i brzini CPU-a/GPU-a

### Faza 3: Kalibracija

**Što Chloros radi:*** **Korekcija vinjete**: Uklanja zamračivanje na rubovima objektiva
* **Kalibracija reflektancije**: Normalizira pomoću ciljanih vrijednosti reflektancije i/ili DAQ podataka o dolaznom zračenju
* Primjenjuje korekcije na sve pojaseve/kanale
* Koristi odgovarajuću referencu za kalibraciju za svaku sliku na temelju vremenske oznake

**Trajanje:** Većina vremena obrade

### Faza 4: Izračun indeksa

**Što Chloros radi:**

* Izračunava konfigurirane multispektralne indekse (NDVI, NDRE itd.)
* Primjenjuje matematičke operacije na pojasima na kalibrirane slike
* Generira indeksne slike za svaki odabrani indeks

**Trajanje:** Nekoliko sekundi po slici

### Faza 5: Izvoz

**Što Chloros radi:**

* Pohranjuje obrađene slike u odabrani format
* **LATTICE fan-out**: svaki sirovi LATTICE kadar izvozi se kao svaki omogućen proizvod u jednom koraku — debayered, pretpregled, radijansa (uvijek float32), reflektancija
* Pohranjuje datoteke u stablo izlaznih datoteka projekta: `<project>/<camera>/<format>/<Product>_Images/`
* **Održava izvorni naziv datoteke** — mapa identificira proizvod, ne dodaje se sufiks**Trajanje:** Varira ovisno o formatu izvoza i veličini datoteke***

## Ponašanje tijekom obrade

### Automatska linija obrade

Nakon pokretanja, cijela linija obrade radi automatski:

* Nije potrebna interakcija korisnika
* Svi konfigurirani koraci izvršavaju se uzastopno
* Ažuriranja napretka prikazuju se u stvarnom vremenu
* Izvezene datoteke spremaju se na disk čim su dovršene — možete otvoriti dovršene rezultate dok se izvršavanje nastavlja

### Korištenje računala tijekom obrade

**Slobodni način rada:**

* Relativno niska iskoristivost CPU-a (jednojezgreno)
* Računalo ostaje responzivno za druge zadatke
* Sigurno je minimizirati Chloros i raditi u drugim aplikacijama

**Chloros+ paralelnog načina rada:**

* Visoka iskorištenost CPU-a na svim radnim procesorima strategije
* Uz GPU akceleraciju: Visoka iskorištenost GPU-a
* Računalo može biti manje odzivno tijekom obrade
* Izbjegavajte pokretanje drugih zadataka koji zahtijevaju veliku snagu procesora

{% hint style="warning" %}
**Savjet za bolje performanse**: Za najbolje performanse programa Chloros+, zatvorite druge aplikacije i dopustite programu Chloros da koristi sve resurse sustava.
{% endhint %}

### Obrada se ne može pauzirati (ali zaustavljanje je čisto)

* Jednom pokrenut, proces se ne može pauzirati i kasnije nastaviti
* Klikom na **Stop** zaustavlja se izvršavanje na siguran način već pri prvom kliku
* Proizvodi koji su već izvezeni prije zaustavljanja ostaju na disku
* Zaustavljeno izvršavanje iskreno izvještava o onome što je dovršeno (pogledajte retke `[RUN-SUMMARY]` u dnevniku)
* Novi pokretanje pokreće cijeli proces ispočetka

**Savjet za planiranje:** Za vrlo velike projekte razmislite o obradi u serijama ili korištenju CLI-ja za bolju kontrolu.***

## Praćenje obrade

Dok se obrada odvija, možete:

* **Pratite traku napretka** - Vidite ukupni postotak dovršenosti
* **Pregledajte trenutnu fazu** - Otkrivanje, Analiza, Kalibracija ili Izvoz
* **Provjerite karticu dnevnika** - Pogledajte detaljne poruke o obradi i upozorenja
* **Pregledajte dovršene slike** - Datoteke za izvoz pojavljuju se na disku tijekom obrade

Za detaljne informacije o nadzoru pogledajte [Nadzor obrade](monitoring-the-processing.md).

***

## Zaustavljanje obrade

Ako trebate zaustaviti obradu:

### Kako zaustaviti

1. Pronađite **gumb Zaustavi** (zamjenjuje gumb Pokreni tijekom obrade)
2. Kliknite ga jednom — traka prikazuje **"Zaustavljanje..."** dok se trenutna slika dovršava
3. Pokretanje završava u konačnom zaustavljenom stanju i zapisnik ispisuje izvješće `[RUN-SUMMARY]` o onome što je dovršeno

### Kada zaustaviti

**Valjani razlozi za zaustavljanje:**

* Otkrili ste da su korištene netočne postavke
* Zaboravili ste označiti ciljne slike
* Uvezeni su pogrešni podaci
* Sustav radi presporo ili ne reagira

**Nakon zaustavljanja:**

* Proizvodi izvezeni prije zaustavljanja ostaju na disku
* Pregledajte i ispravite sve probleme, prilagodite postavke prema potrebi
* Ponovno pokrenite obradu — izvršavanje započinje ispočetka

***

## Procjene vremena obrade

Stvarno vrijeme obrade uvelike varira ovisno o:

* Broju slika
* Rezoluciji slika
* Ulaznom formatu RAW vs JPG
* Načinu obrade (Besplatni nasuprot Chloros+)
* Brzina i broj jezgri CPU-a
* Dostupnost GPU-a (samo za Chloros+)
* Broj indeksa za izračunavanje
* Broj omogućenih proizvoda za izvoz (LATTICE)

### Grube procjene (Chloros+, slike od 12 MP, suvremeni CPU)

| Broj slika | Besplatni način | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 slika   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 slika  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 slika   | 1-1,5 h | 20-30 min      | 10-15 min      |
| 500 slika  | 2-3 h    | 45-60 min      | 20-30 min      |
| 1000 slika | 4-6 sati | 1,5-2 sata | 40-60 min |

{% hint style="info" %}
**Prvo pokretanje**: Početna obrada može potrajati dulje jer Chloros gradi predmemorije i profile. Naknadna obrada sličnih skupova podataka bit će brža.
{% endhint %}

***

## Uobičajeni problemi pri pokretanju

### Gumb za pokretanje je onemogućen (sivo obojen)

**Mogući uzroci:**

* Nije uvezeno nijedno slike
* Backend nije u potpunosti pokrenut
* Prethodna obrada još uvijek traje
* Projekt nije u potpunosti učitano

**Rješenja:**

1. Pričekajte da se pozadinski sustav (backend) u potpunosti inicijalizira (provjerite ikonu u glavnom izborniku).
2. Provjerite jesu li slike uvezene u pregledniku datoteka (File Browser).
3. Ponovno pokrenite Chloros ako gumb ostane onemogućen.
4. Provjerite zapisnik za otklanjanje pogrešaka (Debug Log) radi poruka o pogreškama.

### Obrada započinje, a zatim odmah završava s pogreškom

**Mogući uzroci:**

* Nema valjanih slika u projektu
* Oštećene datoteke sa slikama
* Nedovoljno prostora na disku
* Nedovoljno memorije (RAM)

**Rješenja:**

1. Provjerite zapisnik za otklanjanje pogrešaka (Debug Log) <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> za poruke o pogreškama
2. Provjerite dostupni prostor na disku
3. Pokušajte obraditi manji podskup slika
4. Provjerite jesu li slike neoštećene

### Pokretanje završava, ali ne pišu se slike

Pokretanje koje je zatražilo proizvode sa slikama, ali nije napisalo nijedan, smatra se **neuspjehom, a ne uspjehom** — Chloros to glasno prijavljuje:

* U GUI dnevniku ispisuju se `[RUN-SUMMARY]` naznake koje navode vjerojatni uzrok — nije uvezeno nijedno slike, nije detektiran cilj ili je svaki zatraženi proizvod preskočen kao neprimjenjiv (npr. zahtjev za radijancijom/reflektancijom s kamera koje podržavaju samo RGB)
* Ekvivalent u CLI-u (`chloros-cli process`) ispisuje `Processing finished but wrote no image products.` i **izlazi s ne-nuljevim rezultatom**, tako da ga skripte mogu detektirati
* Namjerno pokretanje samo za metapodatke (svaki proizvod za izvoz je onemogućen, nema indeksa) i dalje se računa kao uspjeh

Pogledajte [referencu za CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) za potpunu semantik.

### Projektna mapa je preduboka za ovo računalo

**Samo za Windows, novo u verziji 1.3.0.**Windows dolazi s isključenim dugim putanjama, pri čemu nijedna putanja do datoteke ne smije biti dulja od 260 znakova. Proizvodi tvrtke Chloros smješteni su do otprilike 130 znakova ispod mape projekta, stoga mapa projekta dublja od toga ne bi mogla primiti svoje proizvode. Umjesto da se pokrene i ne zaprimi nikakve podatke, Chloros sada provjerava prije početka obrade:

* Klikom na Pokreni se ne pokreće izvršavanje: traka napretka prikazuje razlog, a zapisnik pogrešaka (Debug Log) prikazuje `[PROCESS] refused: The project folder is too deep for this computer…`. Poruka navodi duljinu mape projekta i najdužu mapu projekta koja stane.
* Također je odbijeno stvaranje ili otvaranje projekta u mapi koja je pre duboka čak i za vlastite datoteke projekta. Poruka objašnjava zašto, s najdužom projektnom mapom koja stane i uputama kako omogućiti duge putanje, te ništa nije stvoreno niti otvoreno; Datoteka za otklanjanje pogrešaka bilježi to kao `[PROJECT] refused: …`. Dupliciranje projekta pod dužim nazivom također može premašiti ograničenje: kopija se stvara, ali se ne otvara, a ista poruka objašnjava zašto.

**Rješenja:**

1. Pohranite projekte u kraći direktorij — promijenite**Radni direktorij** u [Postavke projekta](../project-settings/project-settings.md) i ponovno kreirajte projekt, ili
2. Uključite dugine staze sustava Windows i ponovno pokrenite Chloros: ponovno pokrenite instalacijski program programa Chloros s potvrđenom opcijom **Omogući dugine staze sustava Windows** (što je potvrđeno prema zadanim postavkama) ili upotrijebite grupnu politiku: *Konfiguracija računala > Upravljački predlošci > Sustav > Datotečni sustav > Omogući dugine Win32 staze*.

### Upozorenje "Nije otkriveno ciljeva"

**Mogući uzroci:**

* Zaboravili ste označiti slike ciljeva
* Slike ciljeva ne sadrže vidljive ciljeve
* Postavke za otkrivanje ciljeva su previše stroge

**Rješenja:**

1. Pregledajte odjeljak [Odabir slika ciljeva](choosing-target-images.md)
2. Označite odgovarajuće slike u stupcu Cilj
3. Provjerite jesu li ciljevi vidljivi na označenim slikama
4. Po potrebi prilagodite postavke za otkrivanje ciljeva

***

## Savjeti za uspješnu obradu

### Prije početka

1. **Prvo testirajte s malim podskupom** - Obradite 10-20 slika kako biste provjerili postavke
2. **Provjerite dostupni prostor na disku** - Osigurajte 2-3x veličinu skupa podataka slobodnog prostora (više ako su omogućeni svi LATTICE proizvodi)
3. **Zatvorite nepotrebne aplikacije** - Oslobodite sistemske resurse
4. **Provjerite slike s ciljevima** - Pregledajte označene ciljeve kako biste osigurali kvalitetu
5. **Spremite projekt** - Projekt se automatski sprema, ali je dobra praksa spremiti ga ručno

### Tijekom obrade

1. **Izbjegavajte hibernaciju sustava** - Onemogućite načine štednje energije
2. **Držite Chloros u prvom planu** - Ili barem vidljivim u traci zadataka
3. **Povremeno provjeravajte napredak** - Provjerite postoje li upozorenja ili pogreške
4. **Ne pokretajte druge zahtjevne aplikacije** - Posebno u paralelnom načinu rada Chloros+

### Chloros+ GPU akceleracija

Ako koristite NVIDIA GPU akceleraciju:

1. Ažurirajte NVIDIA upravljačke programe na najnoviju verziju
2. Provjerite ima li GPU 4 GB+ VRAM-a (7 GB+ za istovremeno Texture Aware debayering)
3. Zatvorite aplikacije koje intenzivno koriste GPU (igre, uređivanje videa)
4. Pratite temperaturu GPU-a (osigurajte adekvatno hlađenje)

***

## Sljedeći koraci

Nakon što je obrada započela:

1. **Pratite napredak** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)
2. **Pričekajte dovršetak** - Obrada se odvija automatski
3. **Pregledajte rezultate** - Pogledajte [Završetak obrade](finishing-the-processing.md)

Za informacije o tome što učiniti tijekom obrade pogledajte [Praćenje obrade](monitoring-the-processing.md).
