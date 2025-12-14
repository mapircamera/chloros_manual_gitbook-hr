# Pokretanje obrade

Nakon što uvezete svoje slike, označite ciljeve kalibracije i konfigurirate postavke projekta, spremni ste za početak obrade. Ova vas stranica vodi kroz pokretanje cjevovoda za obradu Chlorosa.

## Kontrolni popis prije obrade

Prije nego što kliknete gumb Start, provjerite je li sve spremno:

* [ ] **Uvezene datoteke** - Sve slike pojavljuju se u pregledniku datoteka
* [ ] **Ciljne slike označene** - Ciljni stupac je provjeren za kalibracijske slike
* [ ] **Otkriveni modeli kamera** - Stupac Model kamera prikazuje ispravne kamere
* [ ] **Postavke konfigurirane** - Postavke projekta pregledane i prilagođene
* [ ] **Odabrani indeksi** - dodani željeni multispektralni indeksi (ako je potrebno)
* [ ] **Odabrani format izvoza**- Izlazni format prikladan za vaš tijek rada

{% hint style="info" %}**Savjet**: Kliknite na nekoliko slika u pregledniku datoteka kako biste provjerili jesu li ispravno učitane prije obrade.
{% endhint %}***

## Pokretanje obrade

### Pronađite gumb Start

Gumb Start/Play nalazi se u gornjoj traci zaglavlja Chlorosa:

* Položaj: Gornji središnji dio prozora
* Ikona: **gumb za reprodukciju/pokretanje**<img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Gumb je omogućen (svijetli) kada je spreman za obradu

### Pritisnite za početak

1. Kliknite gumb **Play/Start**u gornjem zaglavlju
2. Obrada počinje odmah
3. Gumb postaje onemogućen (zasivljen) tijekom obrade
4. Traka napretka se ažurira, prikazuje status obrade

{% hint style="success" %}**Obrada započeta**: Jednom kada se klikne, Chloros automatski upravlja svim koracima obrade - otkrivanje cilja, debayering, kalibracija, izračun indeksa i izvoz.
{% endhint %}***## Razumijevanje načina obrade

Chloros radi u dva različita načina obrade ovisno o vašoj licenci:

### Slobodan način rada (sekvencijalna obrada)

**Dostupno svim korisnicima**
**Kako radi:**

* Obrađuje slike jednu po jednu, uzastopno
* Rad s jednom niti
* Manja upotreba memorije

**Traka napretka prikazuje 2 faze:**1.**Otkrivanje cilja**- Skeniranje ciljeva za kalibraciju
2.**Obrada**- Primjena kalibracije i izvoz slika**Vrijeme obrade:**

* Mnogo sporije od Chloros+ paralelnog načina rada
* Prikladno za male do srednje skupove podataka (< 200 slika)

### Način Chloros+ (paralelna obrada)

**Zahtijeva Chloros+ licencu**
**Kako radi:**

* Obrađuje više slika istovremeno
* Višenitni rad (do 16 paralelnih radnika)
* Koristi više CPU jezgri
* Dodatno GPU (CUDA) ubrzanje s NVIDIA grafičkim karticama

**Traka napretka prikazuje 4 faze:**1.**Otkrivanje**- Pronalaženje ciljeva kalibracije
2.**Analiziranje**- Ispitivanje metapodataka slike i priprema cjevovoda
3.**Kalibracija**- Primjena korekcija i kalibracija
4.**Izvoz**- Spremanje obrađenih slika i indeksa**Interakcija trake napretka:**

* **Zadržite pokazivač miša** iznad trake da vidite detaljnu padajuću ploču u 4 stupnja
* **Kliknite** traku napretka da zamrznete padajuću ploču na mjestu
* **Kliknite ponovo**da odmrznete i sakrijete ploču**Vrijeme obrade:**

* Znatno brže od besplatnog načina rada
* Vage s brojem jezgri CPU-a
* GPU ubrzanje dodatno poboljšava brzinu

{% hint style="info" %}
**Chloros+ Brzina**: Paralelna obrada može biti 5-10x brža od sekvencijalnog načina za velike skupove podataka. Projekt od 500 slika koji traje 2 sata u besplatnom načinu rada može se završiti za 15-20 minuta uz Chloros+.
{% endhint %}***## Što se događa tijekom obrade

### Faza 1: Otkrivanje cilja

**Što Chloros radi:**

* Skenira označene ciljane slike (ili sve slike ako nijedna nije označena)
* Identificira 4 kalibracijske ploče u svakoj meti
* Izvlači vrijednosti refleksije iz ciljnih ploča
* Bilježi ciljne vremenske oznake za planiranje kalibracije

**Trajanje:**1-30 seconds (with marked targets), 5-30+ minutes (unmarked)

### Faza 2: Debayering (RAW konverzija)**Što Chloros radi:**

* Pretvara RAW podatke Bayerovog uzorka u pune RGB slike
* Primjenjuje visokokvalitetni algoritam demosaicinga
* Čuva maksimalnu kvalitetu slike i detalje

**Trajanje:**Varies by image count and CPU speed

### Faza 3: Kalibracija**Što Chloros radi:**

* **Korekcija vinjete**: Uklanja zatamnjenje leće na rubovima
* **Kalibracija refleksije**: Normalizira pomoću ciljanih vrijednosti refleksije
* Primjenjuje ispravke na sve pojaseve/kanale
* Koristi odgovarajući cilj kalibracije za svaku sliku na temelju vremenske oznake

**Trajanje:**Majority of processing time

### Faza 4: Izračun indeksa**Što Chloros radi:**

* Izračunava konfigurirane multispektralne indekse (NDVI, NDRE, itd.)
* Primjenjuje matematiku pojasa na kalibrirane slike
* Generira slike indeksa za svaki odabrani indeks

**Trajanje:**A few seconds per image

### Faza 5: Izvoz**Što Chloros radi:**

* Sprema kalibrirane slike u odabranom formatu
* Izvozi indeksne slike s konfiguriranim LUT bojama
* Zapisuje datoteke u podmape modela fotoaparata
* Čuva izvorne nazive datoteka sa sufiksima

**Trajanje:**Varies by export format and file size***

## Ponašanje obrade

### Cjevovod za automatsku obradu

Nakon pokretanja, cijeli cjevovod se pokreće automatski:

* Nije potrebna interakcija korisnika
* Svi konfigurirani koraci izvršavaju se u nizu
* Ažuriranja napretka prikazana u stvarnom vremenu

### Upotreba računala tijekom obrade

**Slobodan način rada:**

* Relativno niska upotreba procesora (jednonit)
* Računalo ostaje osjetljivo na druge zadatke
* Sigurno za minimiziranje Chlorosa i rad u drugim aplikacijama

**Chloros+ Paralelni način:**

* Velika upotreba CPU-a (višenitni, do 16 jezgri)
* S GPU ubrzanjem: Visoka upotreba GPU-a
* Računalo može slabije reagirati tijekom obrade
* Izbjegavajte pokretanje drugih CPU-intenzivnih zadataka

{% hint style="warning" %}
**Savjet za performanse**: Za najbolju izvedbu Chloros+ zatvorite ostale aplikacije i dopustite Chlorosu da koristi sve resurse sustava.
{% endhint %}

### Obrada se ne može pauzirati**Važna ograničenja:**

* Jednom započeta obrada se ne može pauzirati
* Možete otkazati obradu, ali napredak se gubi
* Djelomični rezultati se ne spremaju
* Mora se ponovno pokrenuti od početka ako se otkaže

**Savjet za planiranje:**For very large projects, consider processing in batches or using CLI for better control.***

## Praćenje vaše obrade

Tijekom obrade možete:

* **Pogledajte traku napretka** - Pogledajte ukupni postotak završetka
* **Pogledajte trenutnu fazu** - Otkrijte, analizirajte, kalibrirajte ili izvezite
* **Provjeri karticu dnevnika** - Pogledajte detaljne poruke obrade i upozorenja
* **Pregled dovršenih slika**- Neke datoteke za izvoz mogu se pojaviti tijekom obrade

Za detaljne informacije o nadzoru pogledajte [Nadgledanje obrade](monitoring-the-processing.md).***

## Otkazivanje obrade

Ako trebate zaustaviti obradu:

### Kako otkazati

1. Pronađite **gumb Stop/Cancel**(zamjenjuje gumb Start tijekom obrade)
2. Pritisnite gumb Stop
3. Obrada se odmah zaustavlja
4. Djelomični rezultati se odbacuju

### Kada otkazati**Valjani razlozi za otkazivanje:**

* Uočene netočne postavke korištene su
* Zaboravili ste označiti ciljane slike
* Uvezene su pogrešne slike
* Sustav radi presporo ili ne reagira

**Nakon otkazivanja:**

* Pregledajte i riješite probleme
* Po potrebi prilagodite postavke
* Ponovno pokrenite obradu od početka
* Za najčišći doživljaj potpuno zatvorite Chloros i ponovno ga pokrenite

{% hint style="warning" %}
**Nema djelomičnih rezultata**: Otkazivanjem se odbacuje sav napredak. Chloros ne sprema djelomično obrađene slike.
{% endhint %}***

## Procjena vremena obrade

Stvarno vrijeme obrade uvelike varira ovisno o:

* Broj slika
* Rezolucija slike
* RAW u odnosu na JPG ulazni format
* Način obrade (Free vs Chloros+)
* CPU brzina i broj jezgri
* Dostupnost GPU-a (samo Chloros+)
* Broj indeksa za izračunavanje
* Složenost izvoznog formata

### Grube procjene (Chloros+, slike od 12 MP, moderni CPU)

| Broj slika | Slobodan način rada | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 slika   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 slika  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 slika  | 1-1,5 sati | 20-30 min      | 10-15 min      |
| 500 slika  | 2-3 sata   | 45-60 min      | 20-30 min      |
| 1000 slika | 4-6 sati   | 1,5-2 sata      | 40-60 min      |

{% hint style="info" %}
**Prvo pokretanje**: početna obrada može potrajati dulje jer Chloros gradi predmemorije i profile. Naknadna obrada sličnih skupova podataka bit će brža.
{% endhint %}***## Uobičajeni problemi na početku

### Gumb Start onemogućen (zasivljen)

**Mogući uzroci:**

* Nema uvezenih slika
* Pozadina nije u potpunosti pokrenuta
* Prethodna obrada još traje
* Projekt nije u potpunosti učitan

**Rješenja:**1. Pričekajte da se pozadina potpuno inicijalizira (provjerite ikonu glavnog izbornika)
2. Provjerite jesu li slike uvezene u preglednik datoteka
3. Ponovno pokrenite Chloros ako gumb ostane onemogućen
4. Provjerite zapisnik otklanjanja pogrešaka za poruke o pogrešci

### Obrada počinje, a zatim odmah ne uspijeva**Mogući uzroci:**

* Nema valjanih slika u projektu
* Oštećene slikovne datoteke
* Nedovoljno prostora na disku
* Nedovoljno memorije (RAM)

**Rješenja:**1. Provjerite zapisnik otklanjanja pogrešaka<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">za poruke o greškama
2. Provjerite dostupan prostor na disku
3. Pokušajte obraditi manji podskup slika
4. Provjerite jesu li slike oštećene

### Upozorenje "Nisu otkriveni ciljevi".**Mogući uzroci:**

* Zaboravili ste označiti ciljane slike
* Slike meta ne sadrže vidljive mete
* Postavke detekcije cilja prestroge

**Rješenja:**1. Pregled [Odabir ciljanih slika](choosing-target-images.md)
2. Označite odgovarajuće slike u stupcu Target
3. Provjerite jesu li ciljevi vidljivi na označenim slikama
4. Po potrebi prilagodite postavke otkrivanja cilja***

## Savjeti za uspješnu obradu

### Prije početka

1. **Prvo testirajte s malim podskupom**- Obradite 10-20 slika za provjeru postavki
2.**Provjerite raspoloživi prostor na disku**- Osigurajte 2-3 puta veću veličinu skupa podataka
3.**Zatvorite nepotrebne aplikacije**- Oslobodite resurse sustava
4.**Provjerite ciljne slike**- Pregledajte označene ciljeve kako biste osigurali kvalitetu
5.**Spremi projekt**- Projekt se automatski sprema, ali dobra praksa je ručno spremanje

### Tijekom obrade

1.**Izbjegavajte stanje mirovanja sustava**- Onemogućite načine rada za uštedu energije
2.**Kloros neka bude u prvom planu**- Ili barem vidljiv na programskoj traci
3.**Povremeno pratite napredak**- Provjerite postoje li upozorenja ili pogreške
4.**Ne opterećujte druge teške aplikacije**- Pogotovo s Chloros+ paralelnim načinom rada

### Chloros+ GPU ubrzanje

Ako koristite NVIDIA GPU ubrzanje:

1. Ažurirajte NVIDIA upravljačke programe na najnoviju verziju
2. Osigurajte da GPU ima 4 GB+ VRAM-a
3. Zatvorite aplikacije koje zahtijevaju GPU (igre, uređivanje videa)
4. Pratite temperaturu GPU-a (osigurajte odgovarajuće hlađenje)***

## Sljedeći koraci

Nakon što obrada započne:

1. **Pratite napredak**- Pogledajte [Nadgledanje obrade](monitoring-the-processing.md)
2.**Pričekajte završetak**- Obrada se pokreće automatski
3.**Pregledajte rezultate** - Pogledajte [Završetak obrade](finishing-the-processing.md)

Za informacije o tome što učiniti tijekom obrade, pogledajte [Nadgledanje obrade](monitoring-the-processing.md).
