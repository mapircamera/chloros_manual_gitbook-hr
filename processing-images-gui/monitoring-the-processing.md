# Praćenje obrade

Nakon što obrada započne, Chloros nudi nekoliko načina za praćenje napretka, provjeru problema i razumijevanje što se događa s vašim skupom podataka. Ova stranica objašnjava kako pratiti obradu i tumačiti informacije koje pruža Chloros.

## Pregled trake napretka

Traka napretka na vrhu zaglavlja prikazuje status obrade u stvarnom vremenu i postotak dovršenosti.

### Traka napretka u slobodnom načinu rada

Za korisnike bez licence Chloros+:

**Prikaz napretka u dvije faze:**

1. **Detekcija cilja**: Potražite ciljeve kalibracije na slikama.
2. **Obrada**: primjena ispravaka i izvoz.

**Traka napretka prikazuje:**

* Ukupni postotak dovršenosti (0-100%)
* Naziv trenutne faze
* Jednostavan zaslon s vodoravnom trakom

### Chloros+ traka napretka

Za korisnike s licencom Chloros+:

**Vizualizacija napretka u 4 faze:**

1. **Otkrivanje**: traženje ciljeva kalibracije.
2. **Analiza**: pregled slika i priprema procesa.
3. **Kalibracija**: primjena vinjeta i korekcija refleksije.
4. **Izvoz**: spremanje obrađenih datoteka.

**Interaktivne značajke:**

* **Zadržite pokazivač iznad** trake napretka da vidite proširenu ploču s 4 stupnja.
* **Kliknite** na traku napretka da zamrznete/prikvačite proširenu ploču
* **Kliknite ponovno** za odmrzavanje i automatsko skrivanje kada uklonite miš
* Svaka faza pokazuje pojedinačni napredak (0-100%)

***

## Razumijevanje svake faze obrade

### Faza 1: Detekcija (otkrivanje cilja)

**Što se događa:**

* Chloros skenira slike označene poljem Cilj
* Algoritmi strojnog vida identificiraju 4 kalibracijske ploče
* Vrijednosti refleksije izdvojene su iz svake ploče
*Ciljne vremenske oznake se bilježe kako bi se pravilno rasporedila kalibracija

**Trajanje:**

* S označenim ciljevima: 10-60 sekundi
*Nema označenih ciljeva: 5-30+ minuta (skenirajte sve slike)

**Indikator napretka:**

* Detekcija: 0% → 100%.
* Broj skeniranih slika.
* Broj pronađenih ciljeva.

**Na što obratiti pozornost:**

*Treba brzo završiti ako su ciljevi točno označeni.
*Ako predugo traje, mete možda neće biti označene.
* Provjerite zapisnik za otklanjanje pogrešaka za poruke "Cilj je pronađen".

### Faza 2: Analiza

**Što se događa:**

* Čitanje EXIF ​​​​metapodataka sa slike (vremenske oznake, postavke ekspozicije).
* Određivanje strategije kalibracije na temelju ciljnih vremenskih oznaka.
* Organizacija čekanja za obradu slika.
* Priprema radnika za paralelnu obradu (samo Chloros+).

**Trajanje:** 5-30 sekundi.

**Indikator napretka:**

* Analiza: 0% → 100%
* Brza faza, obično se brzo završi

**Na što obratiti pozornost:**

* Mora postojano napredovati bez pauza
* Upozorenja o metapodacima koji nedostaju pojavit će se u zapisniku otklanjanja pogrešaka

### Faza 3: Kalibracija

**Što se događa:**

* **De-Bayerizacija**: konverzija Bayer RAW uzorka u 3 kanala
* **Korekcija vinjete**: Uklanjanje zatamnjenja ruba leće.
* **Kalibracija refleksije**: normalizacija s ciljnim vrijednostima.
* **Izračun indeksa**: izračun multispektralnih indeksa.
* Obrada svake slike tijekom cijelog procesa.

**Trajanje:** Većina ukupnog vremena obrade (60-80%).

**Indikator napretka:**

* Kalibracija: 0% → 100%
* Trenutna slika u tijeku
* Dovršene slike / Ukupno slika

**Ponašanje obrade:**

* **Slobodni način**: Obrađujte jednu po jednu sliku
* **Chloros Mode+**: Obrada do 16 slika istovremeno
* **GPU ubrzanje**: Značajno ubrzava ovu fazu.

**Na što obratiti pozornost:**

* Konstantan napredak kroz brojanje slika.
* Provjerite zapisnik otklanjanja pogrešaka za poruke o dovršetku po slikama.
* Upozorenja o kvaliteti slike ili problemima s kalibracijom.

### Faza 4: Izvoz

**Što se događa:**

* Zapišite kalibrirane slike na disk u odabranom formatu
* Izvoz multispektralnih indeksnih slika s LUT bojama
* Stvorite podmape modela fotoaparata
* Sačuvajte izvorne nazive datoteka s odgovarajućim sufiksima

**Trajanje:** 10-20% ukupnog vremena obrade

**Indikator napretka:**

* Izvoz: 0% → 100%
* Datoteke se zapisuju
* Format i odredište izvoza

**Na što obratiti pozornost:**

*Upozorenja o prostoru na disku
* Pogreške pri pisanju datoteke
* Završetak svih konfiguriranih izlaza

***

## Kartica dnevnika otklanjanja pogrešaka

Zapisnik ispravljanja pogrešaka pruža detaljne informacije o napretku obrade i svim problemima na koje se naiđe.

### Pristup zapisniku otklanjanja pogrešaka

1. Kliknite ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
2. Otvara se ploča dnevnika, prikazujući poruke obrade u stvarnom vremenu
3. Automatski se pomiče za prikaz najnovijih poruka

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

**Akcija:** Pregledajte upozorenja nakon obrade, ali ne prekidajte obradu.

#### Poruke o pogrešci (mreža)

Kritični problemi koji mogu uzrokovati neuspjeh obrade:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Radnja:** Zaustavite obradu, riješite pogrešku i ponovno pokrenite.

### Uobičajene poruke dnevnika

| Poruka | Značenje | Potrebna akcija |
| -------------------------------- | ------------------------------------- | ----------------------------------------------------- |
| «Cilj otkriven na \[ime datoteke]» | Cilj kalibracije ispravno pronađen | Ništa - normalno |
| «Obrada slike X od Y» | Trenutno ažuriranje napretka | Ništa - normalno |
| «Nisu pronađeni ciljevi» | Nisu otkriveni kalibracijski ciljevi | Označite ciljane slike ili onemogućite kalibraciju refleksije |
| “Nedovoljno prostora na disku” | Nedovoljno prostora za pohranu za izlaz | Oslobodite prostor na disku |
| «Preskakanje oštećene datoteke» | Datoteka slike je oštećena | Kopiraj datoteku natrag sa SD kartice |
| «Primijenjeni PPK podaci» | .daq datoteka GPS popravci primijenjeni | Ništa - normalno |

### Kopiraj podatke dnevnika

Da biste kopirali zapisnik u svrhu rješavanja problema ili tehničke podrške:

1. Otvorite ploču Debug Log.
2. Pritisnite gumb **«Kopiraj dnevnik»** (ili kliknite desnom tipkom miša → Odaberi sve).
3. Zalijepite sadržaj u tekstualnu datoteku ili e-poštu.
4. Pošaljite ga MAPIR tehničkoj podršci ako je potrebno.

***

## Praćenje resursa sustava

### Upotreba CPU-a

**Slobodni način rada:**

* 1 CPU jezgra na ~100%
* Ostale jezgre neaktivne ili dostupne
* Sustav još uvijek reagira

**Kloros+ Paralelni način rada:**

* Višejezgreni na 80-100% (do 16 jezgri)
* Visoka ukupna iskoristivost CPU-a
* Sustav se može činiti slabijim.

**Za praćenje:**

* Windows upravitelj zadataka (Ctrl+Shift+Esc)
* Kartica Performanse → odjeljak CPU
* Potražite procese "Chloros" ili "chloros-backend".

### Upotreba memorije (RAM)

**Uobičajena uporaba:**

* Mali projekti (<100 slika): 2-4 GB
* Srednji projekti (100-500 slika): 4-8 GB
* Veliki projekti (500+ slika): 8-16 GB
* Chloros+ paralelni način rada koristi više RAM-a

**Ako memorije nema dovoljno:**

* Obradite manje serije.
* Zatvorite ostale aplikacije.
*Nadogradite RAM ako redovito obrađujete velike skupove podataka.

### Upotreba GPU-a (Chloros+ s CUDA)

Kada je GPU ubrzanje omogućeno:

* NVIDIA GPU pokazuje visoku iskoristivost (60-90%).
* Povećava se upotreba VRAM-a (zahtijeva 4 GB+ VRAM-a).
* Faza kalibracije znatno je brža.

**Za praćenje:**

* Ikona NVIDIA programske trake.
* Upravitelj zadataka → Performanse → GPU.
* GPU-Z ili sličan alat za praćenje.

### Disk I/O

**Što očekivati:**

* Visoko čitanje diska tijekom faze analize.
* Visoko pisanje na disk tijekom faze izvoza.
* SSD znatno brži od HDD-a.

**Savjet za performanse:**

* Koristite SSD za mapu projekta kad god je to moguće.
* Izbjegavajte mrežne pogone za velike skupove podataka.
* Provjerite nije li pogon blizu svog maksimalnog kapaciteta (utječe na brzinu pisanja).

***

## Otkrivanje problema tijekom obrade

### Znakovi upozorenja

**Napredak se zaustavlja (bez promjena dulje od 5 minuta):**

* Provjerite ima li pogrešaka u zapisniku otklanjanja pogrešaka.
* Provjerite raspoloživi prostor na disku.
* Provjerite Upravitelj zadataka da provjerite radi li Chloros.

**Poruke o pogrešci pojavljuju se često:**

* Zaustavite obradu i provjerite pogreške.
* Uobičajeni uzroci: prostor na disku, oštećene datoteke, problemi s memorijom.
* Pogledajte odjeljak Rješavanje problema u nastavku.

**Sustav prestaje reagirati:**

* Chloros+ u paralelnom načinu rada koristi previše resursa.
* Razmotrite smanjenje istodobnih zadataka ili nadogradnju hardvera.
* Besplatni način rada troši manje resursa.

### Kada zaustaviti obradu

Zaustavite obradu ako vidite:

* ❌ Pogreške "Disk pun" ili "Ne mogu zapisati datoteku".
* ❌ Ponovljene pogreške oštećenja slikovne datoteke.
* ❌ Sustav se potpuno srušio (ne reagira).
* ❌ Otkriveno je da su konfigurirane netočne postavke.
* ❌ Uvezene su netočne slike.

**Kako to zaustaviti:**

1. Pritisnite gumb **Stop/Cancel** (zamjenjuje gumb Start).
2. Obrada se zaustavlja i napredak se gubi
3. Riješite probleme i ponovno pokrenite ispočetka

***

## Rješavanje problema tijekom obrade

### Obrada je vrlo spora

**Mogući uzroci:**

* Neoznačene ciljne slike (skenirajte sve slike)
* HDD pohrana umjesto SSD-a
*Nedovoljno resursa sustava
* Mnogi indeksi konfigurirani
* Pristup mrežnom pogonu

**Rješenja:**

1. Ako ste tek započeli iu fazi ste otkrivanja: otkažite, označite ciljeve i ponovno pokrenite.
2. Za budućnost: koristite SSD, smanjite indekse i nadogradite hardver.
3. Razmotrite korištenje CLI-ja za skupnu obradu velikih skupova podataka.

### Upozorenja o “prostoru na disku”.

**Rješenja:**

1. Odmah oslobodite prostor na disku.
2. Premjestite projekt na pogon s više prostora.
3. Smanjite broj indeksa za izvoz.
4. Koristite JPG format umjesto TIFF (manje datoteke).

### Česte poruke "oštećena datoteka".

**Rješenja:**

1. Ponovno kopirajte slike sa SD kartice kako biste osigurali njihov integritet.
2. Provjerite ima li na SD kartici pogrešaka.
3. Izbrišite oštećene datoteke iz projekta.
4. Nastavite s obradom preostalih slika.

### Pregrijavanje/usporavanje sustava

**Rješenja:**

1. Osigurajte odgovarajuću ventilaciju.
2. Očistite prašinu iz ventilacijskih otvora računala.
3. Smanjite opterećenje obrade (koristite Free mod umjesto Chloros+).
4. Obradite tijekom najhladnijih sati dana.

***

## Obavijest o dovršenoj obradi

Kada je obrada završena:

* Traka napretka doseže 100%.
* Poruka **"Obrada dovršena"** pojavljuje se u zapisniku otklanjanja pogrešaka.
* Početni gumb ponovno je omogućen.
* Sve izlazne datoteke nalaze se u podmapi modela fotoaparata.

***

## Sljedeći koraci

Nakon završetka obrade:

1. **Pregledajte rezultate**: Pogledajte [Završetak obrade](finishing-the-processing.md).
2. **Provjeri izlaznu mapu** – Provjerite jesu li sve datoteke uspješno izvezene.
3. **Provjerite zapisnik otklanjanja pogrešaka** - Provjerite postoje li upozorenja ili pogreške.
4. **Pregled obrađenih slika**: Koristite preglednik slika ili vanjski softver.

Za informacije o tome kako pregledati i koristiti obrađene rezultate, pogledajte [Završetak obrade](finishing-the-processing.md).