# Praćenje obrade

Nakon što obrada započne, Chloros nudi nekoliko načina za praćenje napretka, provjeru problema i razumijevanje što se događa s vašim skupom podataka. Ova stranica objašnjava kako pratiti svoju obradu i tumačiti informacije koje Chloros pruža.

## Pregled trake napretka

Traka napretka u gornjem zaglavlju prikazuje status obrade u stvarnom vremenu i postotak dovršenosti.

### Traka napretka u slobodnom načinu rada

Za korisnike bez Chloros+ licence:

**Prikaz napretka u 2 stupnja:**1.**Otkrivanje cilja**- Pronalaženje ciljeva kalibracije na slikama
2.**Obrada**- Primjena ispravaka i izvoz**Prikazuje traka napretka:**

* Ukupni postotak dovršenosti (0-100%)
* Trenutno umjetničko ime
* Jednostavna vizualizacija vodoravne trake

### Chloros+ Traka napretka

Za korisnike s licencom Chloros+:

**Prikaz napretka u 4 stupnja:**1.**Otkrivanje**- Pronalaženje ciljeva kalibracije
2.**Analiza**- Ispitivanje slika i priprema cjevovoda
3.**Kalibracija**- Primjena korekcija vinjete i refleksije
4.**Izvoz**- Spremanje obrađenih datoteka**Interaktivne značajke:**

* **Zadržite pokazivač iznad** trake napretka da biste vidjeli proširenu ploču s 4 stupnja
* **Kliknite** traku napretka za zamrzavanje/prikvačivanje proširene ploče
* **Kliknite ponovno** za odmrzavanje i automatsko skrivanje nakon odlaska miša
* Svaka faza pokazuje pojedinačni napredak (0-100%)

***## Razumijevanje svake faze obrade

### Faza 1: otkrivanje (otkrivanje cilja)**Što se događa:**

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
* Broje se pronađeni ciljevi

**Na što treba paziti:**

* Treba brzo završiti ako su mete ispravno označene
* Ako traje predugo, mete možda neće biti označene
* Provjerite zapisnik otklanjanja pogrešaka za poruke "Cilj je pronađen".

### Faza 2: analiziranje

**Što se događa:**

* Čitanje EXIF ​​metapodataka slike (vremenske oznake, postavke ekspozicije)
* Određivanje strategije kalibracije na temelju ciljnih vremenskih oznaka
* Organiziranje reda čekanja za obradu slike
* Priprema radnika za paralelnu obradu (samo Chloros+)

**Trajanje:**5-30 seconds**Indikator napretka:**

* Analiziranje: 0% → 100%
* Brza faza, obično se brzo završi

**Na što treba paziti:**

* Treba napredovati ravnomjerno bez pauza
* Upozorenja o metapodacima koji nedostaju pojavit će se u zapisniku otklanjanja pogrešaka

### Faza 3: Kalibracija

**Što se događa:**

* **Debayering**: Pretvaranje RAW Bayerovog uzorka u 3 kanala
* **Korekcija vinjete**: Uklanjanje zatamnjenja ruba leće
* **Kalibracija refleksije**: normalizacija s ciljanim vrijednostima
* **Izračun indeksa**: Izračunavanje multispektralnih indeksa
* Obrada svake slike kroz cijeli cjevovod

**Trajanje:**Majority of total processing time (60-80%)**Indikator napretka:**

* Kalibracija: 0% → 100%
* Trenutna slika se obrađuje
* Slike dovršene / Ukupno slika

**Ponašanje obrade:**

* **Slobodni način rada**: Obrađuje jednu po jednu sliku uzastopno
* **Kloros+ način rada**: Obrađuje do 16 slika istovremeno
* **GPU ubrzanje**: Značajno ubrzava ovu fazu**Na što treba paziti:**

* Stabilan napredak kroz brojanje slika
* Provjerite zapisnik otklanjanja pogrešaka za poruke o dovršetku po slikama
* Upozorenja o kvaliteti slike ili problemima s kalibracijom

### Faza 4: Izvoz

**Što se događa:**

* Zapisivanje kalibriranih slika na disk u odabranom formatu
* Izvoz multispektralnih indeksnih slika s LUT bojama
* Stvaranje podmapa modela kamere
* Očuvanje izvornih naziva datoteka s odgovarajućim sufiksima

**Trajanje:**10-20% of total processing time**Indikator napretka:**

* Izvoz: 0% → 100%
* Datoteke se pišu
* Format i odredište izvoza

**Na što treba paziti:**

* Upozorenja o prostoru na disku
* Pogreške pisanja datoteke
* Završetak svih konfiguriranih izlaza

***## Kartica dnevnika otklanjanja pogrešaka

Zapisnik otklanjanja pogrešaka pruža detaljne informacije o napretku obrade i svim problemima na koje ste naišli.

### Pristup zapisniku otklanjanja pogrešaka

1. Kliknite**Debug Log**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">ikonu na lijevoj bočnoj traci
2. Otvara se ploča zapisnika koja prikazuje poruke obrade u stvarnom vremenu
3. Automatski se pomiče za prikaz najnovijih poruka

### Razumijevanje poruka dnevnika

#### Informativne poruke (bijela/siva)

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
```**Akcijski:**Review warnings after processing, but don't interrupt

#### Poruke o pogreškama (crveno)

Kritični problemi koji mogu uzrokovati neuspjeh obrade:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```**Akcijski:**Stop processing, resolve error, restart

### Uobičajene poruke dnevnika

| Poruka                          | Značenje                                | Potrebna akcija                                         |
| -------------------------------- | ------------------------------------- | ----------------------------------------------------- |
| "Cilj otkriven u \[ime datoteke]" | Kalibracijski cilj je uspješno pronađen  | Nikakvo - normalno                                         |
| "Obrada slike X od Y"        | Trenutno ažuriranje napretka                | Nikakvo - normalno                                         |
| "Nisu pronađeni ciljevi"               | Nisu otkriveni kalibracijski ciljevi        | Označite ciljane slike ili onemogućite kalibraciju refleksije |
| "Nedovoljno prostora na disku"        | Nema dovoljno pohrane za ispis          | Oslobodite prostor na disku                                    |
| "Preskakanje oštećene datoteke"        | Datoteka slike je oštećena                  | Ponovno kopirajte datoteku sa SD kartice                             |
| "PPK podaci primijenjeni"               | Primijenjene su GPS korekcije iz .daq datoteke | Nikakvo - normalno                                         |

### Kopiranje podataka dnevnika

Za kopiranje dnevnika za rješavanje problema ili podršku:

1. Otvorite ploču Debug Log
2. Kliknite gumb**"Kopiraj dnevnik"**(ili kliknite desnom tipkom miša → Odaberi sve)
3. Zalijepite u tekstualnu datoteku ili e-poštu
4. Pošaljite MAPIR podršci ako je potrebno***

## Nadzor resursa sustava

### Upotreba CPU-a

**Slobodan način rada:**

* 1 CPU jezgra na \~100%
* Ostale jezgre neaktivne ili dostupne
* Sustav ostaje osjetljiv

**Chloros+ Paralelni način:**

* Više jezgri na 80-100% (do 16 jezgri)
* Visoka ukupna iskoristivost CPU-a
* Sustav može manje reagirati

**Za praćenje:**

* Windows upravitelj zadataka (Ctrl+Shift+Esc)
* Kartica Performanse → odjeljak CPU
* Potražite procese "Chloros" ili "chloros-backend".

### Korištenje memorije (RAM).

**Tipična uporaba:**

* Mali projekti (< 100 slika): 2-4 GB
* Srednji projekti (100-500 slika): 4-8 GB
* Veliki projekti (500+ slika): 8-16 GB
* Chloros+ paralelni način rada koristi više RAM-a

**Ako je memorije malo:**

* Obradite manje serije
* Zatvorite ostale aplikacije
* Nadogradite RAM ako redovito obrađujete velike skupove podataka

### Upotreba GPU-a (Chloros+ s CUDA-om)

Kada je GPU ubrzanje omogućeno:

* NVIDIA GPU pokazuje visoku iskoristivost (60-90%)
* Povećava se upotreba VRAM-a (zahtijeva 4 GB+ VRAM-a)
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

**Savjet za izvedbu:**

* Koristite SSD za mapu projekta kada je to moguće
* Izbjegavajte mrežne pogone za velike skupove podataka
* Osigurajte da disk nije blizu kapaciteta (utječe na brzinu pisanja)

***## Otkrivanje problema tijekom obrade

### Znakovi upozorenja**Napredak se zaustavlja (bez promjene 5+ minuta):**

* Provjerite zapisnik otklanjanja pogrešaka za pogreške
* Provjerite dostupan prostor na disku
* Provjerite Upravitelj zadataka kako biste bili sigurni da Chloros radi

**Često se pojavljuju poruke o pogrešci:**

* Zaustavite obradu i pregledajte pogreške
* Uobičajeni uzroci: prostor na disku, oštećene datoteke, problemi s memorijom
* Pogledajte odjeljak za rješavanje problema u nastavku

**Sustav ne reagira:**

* Chloros+ paralelni način rada koristi previše resursa
* Razmislite o smanjenju istodobnih zadataka ili nadogradnji hardvera
* Besplatni način rada zahtijeva manje resursa

### Kada zaustaviti obradu

Zaustavite obradu ako vidite:

* ❌ Pogreške "Disk pun" ili "Ne mogu zapisati datoteku".
* ❌ Ponovljene pogreške oštećenja slikovne datoteke
* ❌ Sustav potpuno zamrznut (ne reagira)
* ❌ Uočeno je da su konfigurirane pogrešne postavke
* ❌ Uvezene su pogrešne slike

**Kako prestati:**1. Kliknite**gumb Stop/Cancel**(zamjenjuje gumb Start)
2. Obrada se zaustavlja, napredak se gubi
3. Riješite probleme i ponovno pokrenite ispočetka***

## Rješavanje problema tijekom obrade

### Obrada je vrlo spora

**Mogući uzroci:**

* Neoznačene ciljne slike (skeniraju se sve slike)
* HDD umjesto SSD memorije
* Nedovoljno resursa sustava
* Mnogi indeksi konfigurirani
* Pristup mrežnom pogonu

**Rješenja:**1. Ako je upravo pokrenuto iu fazi otkrivanja: Odustani, označi ciljeve, ponovno pokreni
2. Za budućnost: Koristite SSD, smanjite indekse, nadogradite hardver
3. Razmotrite CLI za skupnu obradu velikih skupova podataka

### Upozorenja o "prostoru na disku".**Rješenja:**1. Odmah oslobodite prostor na disku
2. Premjestite projekt u vožnju s više prostora
3. Smanjite broj indeksa za izvoz
4. Koristite JPG format umjesto TIFF (manje datoteke)

### Česte poruke "Oštećena datoteka".**Rješenja:**1. Ponovno kopirajte slike sa SD kartice kako biste osigurali cjelovitost
2. Testirajte SD karticu na greške
3. Uklonite oštećene datoteke iz projekta
4. Nastavite s obradom preostalih slika

### Pregrijavanje sustava / prigušivanje**Rješenja:**1. Osigurajte odgovarajuću ventilaciju
2. Očistite prašinu iz ventilacijskih otvora računala
3. Smanjite opterećenje obrade (koristite Free mod umjesto Chloros+)
4. Obradite tijekom hladnijeg doba dana***

## Obrada kompletne obavijesti

Kada obrada završi:

* Traka napretka doseže 100%
* Poruka **"Obrada dovršena"** pojavljuje se u zapisniku otklanjanja pogrešaka
* Gumb Start ponovno postaje omogućen
* Sve izlazne datoteke nalaze se u podmapi modela fotoaparata

***## Sljedeći koraci

Nakon završetka obrade:

1.**Pregledajte rezultate**- Pogledajte [Završetak obrade](finishing-the-processing.md)
2.**Provjeri izlaznu mapu**- Provjerite sve izvezene datoteke ispravno
3.**Pregledajte zapisnik otklanjanja pogrešaka**- Provjerite postoje li upozorenja ili pogreške
4.**Pregled obrađenih slika** - Koristite Preglednik slika ili vanjski softver

Za informacije o pregledavanju i korištenju obrađenih rezultata pogledajte [Završetak obrade](finishing-the-processing.md).
