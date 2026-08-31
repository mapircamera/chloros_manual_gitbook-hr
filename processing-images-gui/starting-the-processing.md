# Pokretanje obrade

Nakon što ste uvezli svoje slike, označili ciljne oznake i konfigurirali postavke projekta, spremni ste za početak obrade. Ova stranica vodi vas kroz pokretanje procesnog toka za Chloros.

## Kontrolna lista za predobradu

Prije nego što kliknete gumb Pokreni, provjerite je li sve spremno:

* [ ] **Datoteke uvezene** - Sve slike se prikazuju u pregledniku datoteka
* [ ] **Ciljne slike označene** - Provjerena kolona &quot;Target&quot; za slike za kalibraciju (ili uvezena snimka `.daq` za LATTICE)
* [ ] **Modeli kamera otkriveni** - stupac Model kamere prikazuje ispravne kamere
* [ ] **Postavke konfigurirane** - Postavke projekta pregledane i prilagođene
* [ ] **Indeksi odabrani** - Poželjni multispektralni indeksi dodani (ako je potrebno)
* [ ] **Odabran format izvoza** - Izlazni format prikladan za vaš radni tijek

{% hint style="info" %}
**Savjet**: Kliknite na nekoliko slika u pregledniku datoteka kako biste provjerili jesu li ispravno učitane prije obrade.
{% endhint %}

***

## Pokretanje obrade

### Pronalaženje gumba za pokretanje

Gumb za pokretanje/reprodukciju nalazi se u gornjoj zaglavnoj traci programa Chloros:

* Položaj: Gornji središnji dio prozora
* Ikona: **Gumb za reprodukciju/pokretanje** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stanje: Gumb je omogućen (svijetao) kada je spreman za obradu

### Kliknite za pokretanje

1. Kliknite **gumb za reprodukciju/pokretanje** u gornjoj zaglavnici
2. Obrada započinje odmah
3. Tijekom obrade gumb se pretvara u **gumb za zaustavljanje**

4. Traka napretka se ažurira i prikazuje status obrade

{% hint style="success" %}
**Početak obrade**: Nakon klika, Chloros automatski obavlja sve korake obrade – detekciju cilja, debayering, kalibraciju, izračun indeksa i izvoz. Automatski otkriva je li vaš projekt Survey3, LATTICE ili mješavina te primjenjuje odgovarajući pipeline na svaku kameru.
{% endhint %}

***

## Razumijevanje načina obrade

Chlorosradi u dva različita načina obrade ovisno o vašoj licenci:

### Besplatni način rada (sekvencijalna obrada)

**Dostupno svim korisnicima**

**Kako radi:**

* Obraduje slike jednu po jednu, sekvencijalno
* Rad s jednim nitom
* Manja potrošnja memorije

**Traka napretka prikazuje 2 faze:**

1.**Detekcija ciljeva** - Skeniranje kalibracijskih ciljeva
2. **Obrada** - Primjena kalibracije i izvoz slika**Vrijeme obrade:**

* Značajno sporije od paralelnog načina rada Chloros+
* Pogodno za male do srednje skupove podataka (&lt; 200 slika)

### Način rada Chloros+ (paralelna obrada)

**Zahtijeva licencu Chloros+**

**Kako radi:**

* Istovremeno obrađuje više slika koristeći [4-niti cjevovod za obradu](../processing-architecture/processing-pipeline.md)
* [Dinamička prilagodba izračuna](../processing-architecture/dynamic-compute-adaptation.md) automatski odabire optimalnu strategiju za vaš hardver na početku izvođenja
* Akceleracija GPU-a (CUDA) s NVIDIA grafičkim karticama (desktop i Jetson)
* **Broj radnika se prilagođava hardveru**: GPU strategije pokreću**1-4 istovremena radnika** (skalirano prema VRAM-u — Jetson s malo memorije pokreće 1, desktop GPU s 12 GB+ do 4); sustavi samo s CPU-om pokreću jednog radnika po fizičkom jezgri, umanjenog za jedan**Traka napretka prikazuje 4 faze** (koje odgovaraju 4 pipeline niti):

1. **Detekcija** (Nit 1) - Pronalaženje ciljeva za kalibraciju
2. **Analiza** (Nit 2) - Pregledavanje metapodataka slike i izračunavanje kalibracije
3. **Kalibracija** (Nit 3) - Debayering, korekcija vinjete, kalibracija, izračun indeksa
4. **Izvoz** (Nit 4) - Pohrana obrađenih slika i indeksa**Interakcija s trakom napretka:*** **Prijeđite mišem** preko trake da biste vidjeli detaljan padajući panel s 4 faze
* **Kliknite** na traku napretka kako biste fiksirali padajući panel
* **Ponovno kliknite** kako biste otkačili i sakrili panel**Vrijeme obrade:**

* Značajno brže od besplatnog načina rada
* Akceleracija GPU-a dodatno poboljšava brzinu

{% hint style="info" %}
**Brzina Chloros+**: Paralelna obrada može biti 5-10 puta brža od sekvencijalnog načina rada za velike skupove podataka. Projekt od 500 slika koji u slobodnom načinu traje 2 sata može se dovršiti za 15-20 minuta s Chloros+.
{% endhint %}

***

## Što se događa tijekom obrade

### Faza 1: Detekcija cilja

**Što Chloros radi:**

* Skenira slike koje ste označili u stupcu Cilj (sve slike samo ako nijedna nije označena)
* Identificira kalibracijske panele u svakom cilju
* Izvlači vrijednosti reflektancije iz ciljnih panela
* Bilježi vremenske oznake ciljeva za zakazivanje kalibracije

**Trajanje:** 1-30 sekundi (s označenim ciljevima), 5-30+ minuta (neoznačeni)

### Faza 2: Demosaicing (konverzija RAW formata)

**Što Chloros radi:**

* Pretvara podatke RAW Bayerovog uzorka u potpune slike s 3 kanala (LATTICE mono moduli ostaju jednobandni — za njih se preskače demosaicing uz napomenu u zapisniku)
* Primjenjuje odabrani algoritam demosaicinga
* Održava maksimalnu kvalitetu i detalje slike

**Trajanje:** Varira ovisno o broju slika i brzini CPU-a/GPU-a

### Faza 3: Kalibracija

**Što Chloros radi:*** **Korekcija vinjete**: Uklanja zamračivanje na rubovima objektiva
* **Kalibracija reflektancije**: Normalizira koristeći ciljne vrijednosti reflektancije i/ili DAQ podatke o dolaznom zračenju
* Primjenjuje korekcije na sve pojaseve/kanale
* Koristi odgovarajuću referencu za kalibraciju za svaku sliku na temelju vremenske oznake

**Trajanje:** Većina vremena obrade

### Faza 4: Izračun indeksa

**Što Chloros radi:**

* Izračunava konfigurirane multispektralne indekse (NDVI, NDRE itd.)
* Primjenjuje matematičke operacije na kalibrirane slike
* Generira indeksne slike za svaki odabrani indeks

**Trajanje:** Nekoliko sekundi po slici

### Faza 5: Izvoz

**Što Chloros radi:**

* Pohranjuje obrađene slike u odabrani format
* **LATTICE fan-out**: svaki sirovi LATTICE kadar izvozi se kao svaki omogućen proizvod u jednom koraku — debayered, pretpregled, radijansa (uvijek float32), reflektancija
* Sprema datoteke u stablo izlaznih datoteka projekta: `<project>/<camera>/<format>/<Product>_Images/`
* **Održava izvorni naziv datoteke** — mapa identificira proizvod, ne dodaje se nastavak**Trajanje:** Varira ovisno o formatu izvoza i veličini datoteke***

## Ponašanje obrade

### Automatski proces obrade

Jednom pokrenut, cijeli proces se automatski izvršava:

* Nije potrebna korisnička interakcija
* Svi konfigurirani koraci izvršavaju se uzastopno
* Ažuriranja napretka prikazuju se u stvarnom vremenu
* Izvezeni se datoteke spremaju na disk čim se dovrše — gotove rezultate možete otvoriti dok se izvršavanje nastavlja

### Korištenje računala tijekom obrade

**Slobodni način rada:**

* Relativno niska iskorištenost CPU-a (jednojezgreno)
* Računalo ostaje odzivno za druge zadatke
* Sigurno je minimizirati Chloros i raditi u drugim aplikacijama

**Paralelni način rada Chloros+:**

* Visoka iskorištenost CPU-a u radnoj skupini strategije
* S GPU akceleracijom: Visoka iskorištenost GPU-a
* Računalo može biti manje odzivno tijekom obrade
* Izbjegavajte pokretanje drugih zadataka koji intenzivno koriste CPU

{% hint style="warning" %}
**Savjet za bolje performanse**: Za najbolje performanse Chloros+, zatvorite druge aplikacije i dopustite Chlorosu da koristi sve resurse sustava.
{% endhint %}

### Obrada se ne može pauzirati (ali zaustavljanje je čisto)

* Jednom pokrenut, proces se ne može pauzirati i kasnije nastaviti
* Klikom na **Stop** (Zaustavi) pokretanje se čisto zaustavlja već pri prvom kliku
* Proizvodi koji su već izvezeni prije zaustavljanja ostaju na disku
* Zaustavljeno pokretanje iskreno izvještava o onome što je dovršilo (pogledajte retke `[RUN-SUMMARY]` u dnevniku)
* Novi pokretanje pokreće cijeli proces ispočetka

**Savjet za planiranje:** Za vrlo velike projekte razmislite o obradi u serijama ili korištenju CLI-ja za bolju kontrolu.***

## Praćenje obrade

Dok se obrada odvija, možete:

* **Pratiti traku napretka** - Vidjeti ukupni postotak dovršenosti
* **Pregledajte trenutnu fazu** - Otkrivanje, Analiza, Kalibracija ili Izvoz
* **Provjerite karticu dnevnika** - Pogledajte detaljne poruke o obradi i upozorenja
* **Pregledajte dovršene slike** - Datoteke za izvoz pojavljuju se na disku tijekom obrade

Za detaljne informacije o nadzoru pogledajte [Nadzor obrade](monitoring-the-processing.md).

***

## Zaustavljanje obrade

Ako trebate zaustaviti obradu:

### Kako zaustaviti

1. Pronađite **gumb Stop** (zamjenjuje gumb Pokreni tijekom obrade)
2. Kliknite ga jednom — traka prikazuje **&quot;Zaustavljanje...&quot;** dok se dovršava obrada slike
3. Pokretanje završava u konačnom zaustavljenom stanju i zapisnik ispisuje iskren `[RUN-SUMMARY]` o onome što je dovršeno

### Kada zaustaviti

**Valjani razlozi za zaustavljanje:**

* Otkrili ste da su korišteni netočni postavke
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

* Nema uvezanih slika
* Backend nije u potpunosti pokrenut
* Prethodna obrada još uvijek traje
* Projekt nije u potpunosti učitan

**Rješenja:**

1. Pričekajte da se backend u potpunosti inicijalizira (provjerite ikonu u glavnom izborniku)
2. Provjerite jesu li slike uvezene u pregledniku datoteka
3. Ponovno pokrenite Chloros ako gumb ostane onemogućen
4. Provjerite zapisnik za greške (Debug Log) radi poruka o pogreškama

### Obrada započinje, a zatim odmah završava s pogreškom

**Mogući uzroci:**

* Nema valjanih slika u projektu
* Oštećene datoteke sa slikama
* Nedovoljno prostora na disku
* Nedovoljno memorije (RAM)

**Rješenja:**

1. Provjerite zapisnik za otklanjanje pogrešaka (Debug Log) <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> radi poruka o pogreškama
2. Provjerite dostupni prostor na disku
3. Pokušajte obraditi manji podskup slika
4. Provjerite jesu li slike oštećene

### Pokretanje je završeno, ali slike nisu napisane

Pokretanje koje je zatražilo slikovne proizvode, ali nije napisalo nijedan, smatra se **neuspjehom, a ne uspjehom** — Chloros to glasno prijavljuje:

* U GUI dnevniku ispisuje se `[RUN-SUMMARY]` naznake koje navode vjerojatni uzrok — nije uvezeno nijedno slike, nije detektirana nijedna meta ili je svaki zatraženi proizvod preskočen kao neprimjenjiv (npr. zahtjev za radijanciju/reflektanciju s kamera koje podržavaju samo RGB)
* Ekvivalent za CLI (`chloros-cli process`) ispisuje `Processing finished but wrote no image products.` i **izlazi s ne-nulovnom vrijednošću**, tako da ga skripte mogu detektirati
* Namjerno pokretanje samo za metapodatke (svaki proizvod za izvoz je onemogućen, bez indeksa) i dalje se računa kao uspjeh

Pogledajte [referencu za CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) za potpunu semantiku.

### Upozorenje &quot;Nije otkriveno ciljeva&quot;

**Mogući uzroci:**

* Zaboravili ste označiti slike s ciljevima
* Slike s ciljevima ne sadrže vidljive ciljeve
* Postavke za otkrivanje ciljeva su previše stroge

**Rješenja:**

1. Pregledajte odjeljak [Odabir slika s ciljevima](choosing-target-images.md)
2. Označite odgovarajuće slike u stupcu Cilj
3. Provjerite jesu li ciljevi vidljivi na označenim slikama
4. Podesite postavke za detekciju ciljeva ako je potrebno

***

## Savjeti za uspješnu obradu

### Prije početka

1. **Prvo testirajte s malom podskupinom** - Obradite 10-20 slika kako biste provjerili postavke
2. **Provjerite raspoloživi prostor na disku** - Osigurajte 2-3 puta više slobodnog prostora od veličine skupa podataka (više ako su omogućeni svi LATTICE proizvodi)
3. **Zatvorite nepotrebne aplikacije** - Oslobodite sistemske resurse
4. **Provjerite slike s ciljevima** - Pregledajte označene ciljeve kako biste osigurali kvalitetu
5. **Spremite projekt** - Projekt se automatski sprema, ali je dobra praksa spremiti ga ručno

### Tijekom obrade

1. **Izbjegavajte hibernaciju sustava** - Onemogućite načine štednje energije
2. **Držite Chloros u prvom planu** - Ili barem vidljivim u traci zadataka
3. **Povremeno provjeravajte napredak** - Provjerite postoje li upozorenja ili pogreške
4. **Ne pokretajte druge zahtjevne aplikacije** - Posebno u načinu rada s više procesora i GPU-a

### Chloros+ akceleracija GPU-a

Ako koristite akceleraciju NVIDIA GPU-a:

1. Ažurirajte NVIDIA upravljačke programe na najnoviju verziju
2. Provjerite ima li GPU 4 GB+ VRAM-a (7 GB+ za istovremeni Texture Aware debayering)
3. Zatvorite aplikacije koje intenzivno koriste GPU (igre, uređivanje videa)
4. Pratite temperaturu GPU-a (osigurajte adekvatno hlađenje)

***

## Sljedeći koraci

Nakon što je obrada započela:

1. **Pratite napredak** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)
2. **Pričekajte dovršetak** - Obrada se odvija automatski
3. **Pregledajte rezultate** - Pogledajte [Završetak obrade](finishing-the-processing.md)

Za informacije o tome što raditi tijekom obrade, pogledajte [Praćenje obrade](monitoring-the-processing.md).
