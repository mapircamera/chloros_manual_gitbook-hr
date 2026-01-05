# Praćenje obrade

Nakon što obrada započne, Chloros pruža nekoliko načina za praćenje napretka, provjeru problema i razumijevanje što se događa s vašim skupom podataka. Ova stranica objašnjava kako pratiti vašu obradu i tumačiti informacije koje pruža Chloros.

## Pregled trake napretka

Traka napretka u gornjem zaglavlju prikazuje status obrade u stvarnom vremenu i postotak dovršenosti.

### Traka napretka u slobodnom načinu rada

Za korisnike bez Chloros+ licence:

**Prikaz napretka u 2 faze:**

1.**Target Detect** - Pronalaženje kalibracijskih ciljeva na slikama
2. **Obrada** - Primjena ispravaka i izvoz**Traka napretka prikazuje:**

* Ukupni postotak završetka (0-100%)
* Trenutno umjetničko ime
* Jednostavna vizualizacija vodoravne trake

### Chloros+ Traka napretka

Za korisnike s licencom Chloros+:

**Prikaz napretka u 4 faze:**

1.**Otkrivanje** - Pronalaženje ciljeva kalibracije
2. **Analiziranje** - Ispitivanje slika i pripremanje cjevovoda
3. **Kalibracija** - Primjena korekcija vinjete i refleksije
4. **Izvoz** - Spremanje obrađenih datoteka**Interaktivne značajke:*** **Zadržite pokazivač iznad** trake napretka da biste vidjeli proširenu ploču s 4 stupnja
* **Kliknite** traku napretka za zamrzavanje/pričvršćivanje proširene ploče
* **Kliknite ponovno** za odmrzavanje i automatsko skrivanje nakon odlaska miša
* Svaka faza pokazuje pojedinačni napredak (0-100%)

***

## Razumijevanje svake faze obrade

### Faza 1: Detekcija (otkrivanje cilja)

**Što se događa:**

* Chloros skenira slike označene potvrdnim okvirom Cilj
* Algoritmi računalnog vida identificiraju 4 kalibracijske ploče
* Vrijednosti refleksije izvučene iz svake ploče
* Zabilježene ciljne vremenske oznake za pravilno planiranje kalibracije

**Trajanje:**

* S označenim metama: 10-60 sekundi
* Bez označenih ciljeva: 5-30+ minuta (skenira sve slike)

**Indikator napretka:**

* Otkrivanje: 0% → 100%
* Broj skeniranih slika
* Broj pronađenih ciljeva

**Na što treba paziti:**

* Treba brzo završiti ako su mete ispravno označene
* Ako traje predugo, mete možda neće biti označene
* Provjerite zapisnik otklanjanja pogrešaka za poruke "Cilj je pronađen".

### Faza 2: Analiza

**Što se događa:**

* Čitanje EXIF metapodataka slike (vremenske oznake, postavke ekspozicije)
* Određivanje strategije kalibracije na temelju ciljnih vremenskih oznaka
* Organiziranje reda čekanja za obradu slike
* Priprema radnika za paralelnu obradu (samo Chloros+)

**Trajanje:** 5-30 sekundi**Indikator napretka:**

* Analiza: 0% → 100%
* Brza faza, obično se brzo završava

**Na što treba paziti:**

* Treba postojano napredovati bez pauza
* Upozorenja o metapodacima koji nedostaju pojavit će se u zapisniku otklanjanja pogrešaka

### Faza 3: Kalibracija

**Što se događa:*** **Debayering**: Pretvaranje RAW Bayerovog uzorka u 3 kanala
* **Korekcija vinjete**: Uklanjanje zatamnjenja ruba leće
* **Kalibracija refleksije**: normalizacija s ciljnim vrijednostima
* **Izračun indeksa**: Izračunavanje multispektralnih indeksa
* Obrada svake slike kroz cijeli cjevovod

**Trajanje:** Većina ukupnog vremena obrade (60-80%)**Indikator napretka:**

* Kalibracija: 0% → 100%
* Trenutna slika se obrađuje
* Slike dovršene / Ukupno slika

**Ponašanje obrade:*** **Slobodni način rada**: Obrađuje jednu po jednu sliku uzastopno
* **Chloros+ način rada**: Obrađuje do 16 slika istovremeno
* **GPU ubrzanje**: Značajno ubrzava ovu fazu**Na što treba paziti:**

* Stalni napredak kroz brojanje slika
* Provjerite zapisnik otklanjanja pogrešaka za poruke o dovršetku po slici
* Upozorenja o kvaliteti slike ili problemima s kalibracijom

### Faza 4: Izvoz

**Što se događa:**

* Zapisivanje kalibriranih slika na disk u odabranom formatu
* Izvoz multispektralnih indeksnih slika s LUT bojama
* Stvaranje podmapa modela fotoaparata
* Očuvanje izvornih naziva datoteka s odgovarajućim sufiksima

**Trajanje:** 10-20% ukupnog vremena obrade**Indikator napretka:**

* Izvoz: 0% → 100%
* Datoteke se zapisuju
* Format i odredište izvoza

**Na što treba paziti:**

* Upozorenja o prostoru na disku
* Pogreške pisanja datoteke
* Završetak svih konfiguriranih izlaza

***

## Kartica dnevnika otklanjanja pogrešaka

Zapisnik otklanjanja pogrešaka pruža detaljne informacije o napretku obrade i svim problemima na koje ste naišli.

### Pristup zapisniku otklanjanja pogrešaka

1. Kliknite ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
2. Otvara se ploča zapisnika koja prikazuje poruke obrade u stvarnom vremenu
3. Automatsko pomicanje za prikaz najnovijih poruka

### Razumijevanje poruka dnevnika

#### Informativne poruke (bijelo/sivo)

Ažuriranja normalne obrade:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Poruke upozorenja (žuto)

Nekritični problemi koji ne prestaju s obradom:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Akcija:** Pregledajte upozorenja nakon obrade, ali nemojte prekidati

#### Poruke o pogrešci (Red)

Kritični problemi koji mogu uzrokovati neuspjeh obrade:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Akcija:** Zaustavite obradu, riješite pogrešku, ponovno pokrenite

### Uobičajene poruke dnevnika

| Poruka | Značenje | Potrebna radnja |
| -------------------------------- | ------------------------------------- | ----------------------------------------------------- |
| "Cilj otkriven u \[ime datoteke]" | Cilj kalibracije uspješno pronađen | Ništa - normalno |
| "Obrada slike X od Y" | Trenutno ažuriranje napretka | Ništa - normalno |
| "Nisu pronađeni ciljevi" | Nisu otkriveni kalibracijski ciljevi | Označite ciljane slike ili onemogućite kalibraciju refleksije |
| "Nedovoljno prostora na disku" | Nema dovoljno pohrane za izlaz | Oslobodite prostor na disku |
| "Preskakanje oštećene datoteke" | Datoteka slike je oštećena | Ponovno kopirajte datoteku sa SD kartice |
| "PPK podaci primijenjeni" | GPS korekcije iz .daq datoteke primijenjene | Ništa - normalno |

### Kopiranje podataka dnevnika

Za kopiranje dnevnika za rješavanje problema ili podršku:

1. Otvorite ploču Debug Log
2. Kliknite gumb **"Kopiraj dnevnik"** (ili kliknite desnom tipkom miša → Odaberi sve)
3. Zalijepite u tekstualnu datoteku ili e-poštu
4. Pošaljite MAPIR podršci ako je potrebno

***

## Nadgledanje resursa sustava

### Upotreba CPU-a

**Slobodni način rada:**

* 1 CPU jezgra na \~100%
* Ostale jezgre neaktivne ili dostupne
* Sustav ostaje osjetljiv

**Chloros+ Paralelni način rada:**

* Više jezgri na 80-100% (do 16 jezgri)
* Visoka ukupna iskoristivost CPU-a
* Sustav može manje reagirati

**Za praćenje:**

* Windows Upravitelj zadataka (Ctrl+Shift+Esc)
* Kartica Performanse → odjeljak CPU
* Potražite procese "Chloros" ili "chloros-backend".

### Upotreba memorije (RAM).

**Uobičajena uporaba:**

* Mali projekti (< 100 slika): 2-4 GB
* Srednji projekti (100-500 slika): 4-8 GB
* Veliki projekti (500+ slika): 8-16 GB
* Chloros+ paralelni način rada koristi više RAM-a

**Ako je malo memorije:**

* Obradite manje serije
* Zatvorite ostale aplikacije
* Nadogradite RAM ako redovito obrađujete velike skupove podataka

### Upotreba GPU-a (Chloros+ sa CUDA)

Kada je GPU ubrzanje omogućeno:

* NVIDIA GPU pokazuje visoku iskoristivost (60-90%)
* Povećava se upotreba VRAM-a (potrebno je 4 GB+ VRAM-a)
* Faza kalibracije znatno je brža

**Za praćenje:**

* Ikona NVIDIA System Tray
* Upravitelj zadataka → Performanse → GPU
* GPU-Z ili sličan alat za praćenje

### Disk I/O

**Što očekivati:**

* Visoko čitanje diska tijekom faze analize
* Visoko pisanje na disk tijekom faze izvoza
* SSD znatno brži od HDD-a

**Savjet za performanse:**

* Koristite SSD za mapu projekta kada je to moguće
* Izbjegavajte mrežne pogone za velike skupove podataka
* Osigurajte da disk nije blizu kapaciteta (utječe na brzinu pisanja)

***

## Otkrivanje problema tijekom obrade

### Znakovi upozorenja

**Napredak se zaustavlja (bez promjena 5+ minuta):**

* Provjerite zapisnik otklanjanja pogrešaka za pogreške
* Provjerite dostupan prostor na disku
* Provjerite Upravitelj zadataka kako biste osigurali da Chloros radi

**Poruke o pogrešci pojavljuju se često:**

* Zaustavite obradu i pregledajte pogreške
* Uobičajeni uzroci: prostor na disku, oštećene datoteke, problemi s memorijom
* Pogledajte odjeljak Rješavanje problema u nastavku

**Sustav ne reagira:**

* Chloros+ paralelni način rada koristi previše resursa
* Razmotrite smanjenje istodobnih zadataka ili nadogradnju hardvera
* Besplatni način rada zahtijeva manje resursa

### Kada zaustaviti obradu

Zaustavite obradu ako vidite:

* ❌ Pogreške "Disk pun" ili "Ne mogu zapisati datoteku".
* ❌ Ponovljene pogreške oštećenja slikovne datoteke
* ❌ Sustav potpuno zamrznut (ne reagira)
* ❌ Uočeno je da su konfigurirane pogrešne postavke
* ❌ Uvezene su pogrešne slike

**Kako prestati:**

1. Kliknite**gumb Stop/Cancel** (zamjenjuje gumb Start)
2. Obrada se zaustavlja, napredak je izgubljen
3. Riješite probleme i ponovno pokrenite ispočetka

***

## Rješavanje problema tijekom obrade

### Obrada je vrlo spora

**Mogući uzroci:**

* Neoznačene ciljne slike (skeniraju se sve slike)
* HDD umjesto SSD memorije
* Nedovoljno resursa sustava
* Mnogi indeksi konfigurirani
* Pristup mrežnom pogonu

**Rješenja:**

1. Ako je tek pokrenuto iu fazi otkrivanja: Odustani, označi ciljeve, ponovno pokreni
2. Za budućnost: Koristite SSD, smanjite indekse, nadogradite hardver
3. Razmotrite CLI za skupnu obradu velikih skupova podataka

### Upozorenja "Prostor na disku".

**Rješenja:**

1. Odmah oslobodite prostor na disku
2. Premjestite projekt u pogon s više prostora
3. Smanjite broj indeksa za izvoz
4. Koristite JPG format umjesto TIFF (manje datoteke)

### Česte poruke "Oštećena datoteka".

**Rješenja:**

1. Ponovno kopirajte slike sa SD kartice kako biste osigurali cjelovitost
2. Testirajte SD karticu na pogreške
3. Uklonite oštećene datoteke iz projekta
4. Nastavite s obradom preostalih slika

### Pregrijavanje sustava / prigušivanje

**Rješenja:**

1. Osigurajte odgovarajuću ventilaciju
2. Očistite prašinu iz ventilacijskih otvora računala
3. Smanjite opterećenje obrade (koristite Slobodan način rada umjesto Chloros+)
4. Obradite tijekom hladnijeg doba dana

***

## Obrada potpune obavijesti

Kada obrada završi:

* Traka napretka doseže 100%
* Poruka **"Obrada dovršena"** pojavljuje se u zapisniku otklanjanja pogrešaka
* Gumb Start ponovno postaje omogućen
* Sve izlazne datoteke nalaze se u podmapi modela fotoaparata

***

## Sljedeći koraci

Nakon završetka obrade:

1. **Rezultati pregleda** - Pogledajte [Završetak obrade](finishing-the-processing.md)
2. **Provjeri izlaznu mapu** - Provjerite sve datoteke koje su ispravno izvezene
3. **Pregledajte dnevnik otklanjanja pogrešaka** - Provjerite postoje li upozorenja ili pogreške
4. **Pregled obrađenih slika** - Koristite Image Viewer ili vanjski softver

Za informacije o pregledavanju i korištenju obrađenih rezultata pogledajte [Završetak obrade](finishing-the-processing.md).