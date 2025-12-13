# Početak obrade

Nakon što uvezete svoje slike, označite ciljeve kalibracije i konfigurirate postavke projekta, spremni ste za početak obrade. Ova stranica će vas voditi kroz početak procesa obrade Chlorosa.

## Kontrolni popis prije obrade

Prije nego što kliknete gumb Start, provjerite je li sve spremno:

* [ ] **Uvezene datoteke**: Sve slike pojavljuju se u pregledniku datoteka.
* [ ] **Označene ciljne slike**: Označeni ciljni stupac za kalibracijske slike.
* [ ] **Otkriveni modeli kamera** – stupac Model kamere prikazuje ispravne kamere
* [ ] **Konfigurirane postavke**: Postavke projekta pregledane i prilagođene
* [ ] **Odabrani indeksi**: Dodani željeni multispektralni indeksi (ako je potrebno)
* [ ] **Odabrani izvozni format**: Izlazni format prikladan za vaš tijek rada

{% hint style=&quot;info&quot; %}
**Savjet**: Kliknite na neke slike u pregledniku datoteka kako biste provjerili jesu li ispravno učitane prije obrade.
{% endhint %}

***

## Početak obrade

### Pronađite početni gumb

Gumb za početak/reprodukciju nalazi se u gornjoj traci Chlorosa:

* Položaj: gornji centar prozora
* Ikona: **play/home button** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Gumb je omogućen (svijetli) kada je spreman za renderiranje

### Kliknite za početak

1. Pritisnite gumb **Play/Start** na gornjoj traci
2. Obrada počinje odmah
3. Gumb je onemogućen (zasivljen) tijekom obrade
4. Traka napretka se ažurira i prikazuje status obrade

{% hint style=&quot;uspjeh&quot; %}
**Obrada započeta**: Jednom kada se klikne, Chloros automatski upravlja svim koracima obrade: otkrivanje cilja, debayering, kalibracija, izračun indeksa i izvoz.
{% endhint %}

***

## Razumijevanje načina obrade

Chloros radi u dva različita načina obrade, ovisno o vašoj licenci:

### Slobodan način (sekvencijalna obrada)

**Dostupno svim korisnicima**

**Kako radi:**

* Obrađuje slike jednu po jednu, uzastopno.
* Rad s jednim navojem.
* Manja upotreba memorije.

**Traka napretka prikazuje dvije faze:**

1. **Otkrivanje cilja**: Potražite ciljeve kalibracije.
2. **Obrada**: primjena kalibracije i izvoz slika.

**Vrijeme obrade:**

* Puno sporije od Chloros+ paralelnog načina rada.
* Prikladno za male i srednje skupove podataka (<200 slika).

### Chloros+ mod (paralelna obrada)

**Zahtijeva Chloros+ licencu.**

**Kako radi:**

* Obrada više slika istovremeno
* Višenitni rad (do 16 paralelnih radnika)
* Koristi više CPU jezgri
* Dodatno GPU ubrzanje (CUDA) s NVIDIA grafičkim karticama

**Traka napretka prikazuje 4 faze:**

1. **Detekcija**: traženje cilja kalibracije
2. **Analiza**: ispitivanje metapodataka slike i priprema procesa
3. **Kalibracija**: primjena korekcija i kalibracija
4. **Izvoz**: spremanje obrađenih slika i indeksa

**Interakcija s trakom napretka:**

* **Zadržite pokazivač** iznad trake da vidite detaljnu padajuću ploču s 4 stupnja
* **Kliknite** na traku napretka da zamrznete padajuću ploču na mjestu
* **Kliknite ponovno** za otključavanje i skrivanje ploče.

**Vrijeme obrade:**

* Znatno brže od besplatnog načina rada.
* Prilagođava se broju CPU jezgri.
* GPU ubrzanje dodatno poboljšava brzinu.

{% hint style=&quot;info&quot; %}
**Chloros+ Brzina**: Paralelna obrada može biti 5-10 puta brža od sekvencijalnog načina za velike skupove podataka. Projekt od 500 slika koji traje 2 sata u besplatnom načinu rada može se dovršiti za 15-20 minuta uz Chloros+.
{% endhint %}

***

## Što se događa tijekom obrade

### Faza 1: Otkrivanje cilja

**Što Chloros radi:**

* Skenira označene ciljane slike (ili sve slike ako nijedna nije označena).
*Identificira 4 kalibracijske ploče na svakom objektivu
* Izdvaja vrijednosti refleksije ciljanih ploča
* Bilježi ciljne vremenske oznake za planiranje kalibracije

**Trajanje:** 1-30 sekundi (s označenim metama), 5-30+ minuta (neoznačeno)

### Faza 2: Debayering (RAW konverzija)

**Što Chloros radi:**

* Pretvara podatke Bayer RAW uzorka u pune RGB slike.
* Primjenjuje visokokvalitetni demosaic algoritam.
* Održava maksimalnu kvalitetu slike i detalje.

**Trajanje:** ovisi o broju slika i brzini procesora.

### Faza 3: Kalibracija

**Što Chloros radi:**

* **Korekcija vinjete**: Uklanja zatamnjenje rubova leće.
* **Kalibracija refleksije**: Normalizira pomoću ciljanih vrijednosti refleksije.
* Primjenjuje ispravke na sve pojaseve/kanale.
* Koristi odgovarajući cilj kalibracije za svaku sliku na temelju vremenske oznake.

**Trajanje:** Većina vremena obrade

### Faza 4: Izračun indeksa

**Što Chloros radi:**

* Izračunava konfigurirane multispektralne indekse (NDVI, NDRE, itd.)
*Primjenjuje matematiku pojasa na kalibrirane slike
* Generirajte slike indeksa za svaki odabrani indeks

**Trajanje:** nekoliko sekundi po slici

### Faza 5: Izvoz

**Što Chloros radi:**

* Spremite kalibrirane slike u odabranom formatu
* Izvoz indeksnih slika s konfiguriranim LUT bojama
* Pišite datoteke u podmape modela fotoaparata
* Čuva izvorne nazive datoteka sa sufiksima.

**Trajanje:** ovisi o formatu izvoza i veličini datoteke.

***

## Ponašanje obrade

### Cjevovod automatske obrade

Nakon pokretanja, cijeli cjevovod se pokreće automatski:

* Nije potrebna interakcija korisnika.
* Svi konfigurirani koraci izvode se u nizu.
* Ažuriranja napretka prikazuju se u stvarnom vremenu.

### Korištenje računala tijekom obrade

**Slobodni način rada:**

* Relativno nisko korištenje procesora (jedna nit).
* Računalo nastavlja odgovarati na druge zadatke.
* Sigurno je minimizirati Chloros i raditi u drugim aplikacijama.

**Kloros+ Paralelni način rada:**

* Velika upotreba CPU-a (višenitni, do 16 jezgri).
* S GPU ubrzanjem: velika upotreba GPU-a.
*Računalo može slabije reagirati tijekom obrade.
* Izbjegavajte pokretanje drugih CPU-intenzivnih zadataka.

{% hint style=&quot;upozorenje&quot; %}
**Savjet za performanse**: Kako biste dobili najbolju izvedbu od Chloros+, zatvorite ostale aplikacije i dopustite Chlorosu da koristi sve resurse sustava.
{% endhint %}

### Obrada se ne može pauzirati

**Važna ograničenja:**

* Jednom započeta obrada ne može se pauzirati.
* Možete otkazati obradu, ali napredak će biti izgubljen.
* Djelomični rezultati se ne spremaju.
* Ako se otkaže, mora se ponovno pokrenuti iz početka.

**Savjet za planiranje:** Za vrlo velike projekte razmislite o grupiranju ili korištenju CLI-ja za bolju kontrolu.

***

## Praćenje obrade

Dok je obrada u tijeku, možete:

* **Pogledajte traku napretka** - Pogledajte ukupni postotak završetka.
* **Pogledajte trenutnu fazu**: otkrijte, analizirajte, kalibrirajte ili izvezite.
* **Provjeri karticu dnevnika** - Pogledajte detaljne poruke obrade i upozorenja.
* **Pregled dovršenih slika**: Neke datoteke za izvoz mogu se pojaviti tijekom obrade.

Za detalje o nadzoru pogledajte [Nadgledanje obrade](monitoring-the-processing.md).

***

## Otkaži obradu

Ako trebate zaustaviti obradu:

### Kako otkazati

1. Pronađite **gumb Stop/Cancel** (zamjenjuje gumb Start tijekom obrade).
2. Pritisnite gumb Stop.
3. Obrada se odmah zaustavlja.
4. Djelomični rezultati se odbacuju.

### Kada otkazati

**Valjani razlozi za otkazivanje:**

* Otkriveno je da su korištene netočne postavke.
* Zaboravili ste označiti odredišne ​​slike.
* Uvezene su netočne slike.
* Sustav radi presporo ili ne reagira.

**Nakon otkazivanja:**

* Pregledajte i ispravite sve probleme.
* Po potrebi prilagodite postavke.
* Ponovno pokrenite obradu od početka.
* Za najbolje iskustvo potpuno zatvorite Chloros i ponovno ga pokrenite.

{% hint style=&quot;upozorenje&quot; %}
**Nema djelomičnih rezultata**: Otkazivanjem se odbacuje sav napredak. Chloros ne sprema djelomično obrađene slike.
{% endhint %}

***

## Procjene vremena obrade

Stvarno vrijeme obrade uvelike varira ovisno o:

* Broj slika
* Rezolucija slike
* RAW u odnosu na JPG ulazni format
* Način obrade (Free vs. Chloros+)
* CPU brzina i broj jezgri
* Dostupnost GPU-a (samo Chloros+)
* Broj indeksa za izračunavanje
* Složenost izvoznog formata

### Grube procjene (Chloros+, slike od 12 MP, moderni CPU)

| Broj slika | Slobodan način | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 slika | 15-20 min | 5-8 min | 3-5 min |
| 100 slika | 30-40 min | 10-15 min | 5-8 min |
| 200 slika | 1-1,5 sati | 20-30 min | 10-15 min |
| 500 slika | 2-3 sata | 45-60 min | 20-30 min |
| 1000 slika | 4-6 sati | 1,5-2 sata | 40-60 min |

{% hint style=&quot;info&quot; %}
**Prvo pokretanje**: početna obrada može potrajati dulje jer Chloros stvara predmemorije i profile. Naknadna obrada sličnih skupova podataka bit će brža.
{% endhint %}

***

## Uobičajeni problemi pri pokretanju

### Gumb Početna onemogućen (zasivljen)

**Mogući uzroci:**

*Nijedna slika nije uvezena
* Pozadina nije u potpunosti pokrenuta
* Prethodna obrada još traje
* Projekt nije do kraja učitan

**Rješenja:**

1. Pričekajte da se pozadina u potpunosti inicijalizira (provjerite ikonu glavnog izbornika).
2. Provjerite jesu li slike uvezene u file explorer.
3. Ponovno pokrenite Chloros ako je gumb još uvijek onemogućen.
4. Provjerite zapisnik za otklanjanje pogrešaka za poruke o greškama.

### Obrada počinje i ne uspijeva odmah

**Mogući uzroci:**

*U projektu nema valjanih slika
* Oštećene slikovne datoteke
* Nedovoljno prostora na disku
* Nedovoljno memorije (RAM)

**Rješenja:**

1. Provjerite zapisnik otklanjanja pogrešaka <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> za poruke o pogreškama.
2. Provjerite raspoloživi prostor na disku.
3. Pokušajte obraditi manji podskup slika.
4. Provjerite ima li slika oštećenih.

### Upozorenje "Nisu otkriveni ciljevi"

**Mogući uzroci:**

* Zaboravili ste označiti ciljane slike.
*Slike meta ne sadrže vidljive mete.
* Postavke detekcije cilja su prestroge.

**Rješenja:**

1. Označite [Odabir ciljnih slika](choosing-target-images.md).
2. Provjerite odgovarajuće slike u stupcu Cilj.
3. Provjerite jesu li mete vidljive na označenim slikama.
4. Po potrebi prilagodite postavke otkrivanja cilja.

***

## Savjeti za uspješnu obradu

### Prije nego počnete

1. **Prvo testirajte s malim podskupom**: Obradite 10-20 slika za provjeru postavki.
2. **Provjerite raspoloživi prostor na disku** - Provjerite imate li 2-3 puta veću veličinu skupa podataka.
3. **Zatvorite nepotrebne aplikacije** – Oslobodite resurse sustava.
4. **Provjerite slike ciljeva** – Pregledajte označene ciljeve kako biste osigurali kvalitetu.
5. **Spremi projekt**: Projekt se automatski sprema, no preporučuje se ručno spremanje.

### Tijekom obrade

1. **Spriječite prelazak sustava u stanje mirovanja**: Onemogućite načine rada za uštedu energije.
2. **Zadrži Chloros u prvom planu**: Ili barem vidljiv na programskoj traci.
3. **Provjerite napredak s vremena na vrijeme**: Provjerite postoje li upozorenja ili pogreške.
4. **Ne opterećujte druge teške aplikacije**: posebno s Chloros+ paralelnim načinom rada.

### Chloros+ GPU ubrzanje

Ako koristite NVIDIA GPU ubrzanje:

1. Ažurirajte NVIDIA upravljačke programe na najnoviju verziju.
2. Provjerite ima li GPU više od 4 GB VRAM-a.
3. Zatvorite aplikacije koje troše mnogo GPU resursa (igre, uređivanje videa).
4. Pratite temperaturu GPU-a (provjerite je li hlađenje odgovarajuće).

***

## Sljedeći korak

Nakon što obrada započne:

1. **Praćenje napretka**: Pogledajte [Praćenje obrade](monitoring-the-processing.md).
2. **Pričekajte završetak**: Obrada se pokreće automatski.
3. **Pregledajte rezultate**: Pogledajte [Završetak obrade](finishing-the-processing.md).

Za informacije o tome što učiniti tijekom obrade, pogledajte [Nadgledanje obrade](monitoring-the-processing.md).