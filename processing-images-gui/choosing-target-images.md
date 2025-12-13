# Odabir ciljne slike

Označavanje slika koje sadrže kalibracijske ciljeve ključni je korak koji značajno ubrzava Chloros proces. Prethodnim odabirom ciljanih slika eliminirate potrebu da Chloros skenira sve slike u skupu podataka za ciljeve kalibracije.

## Zašto označavati ciljne slike?

### Brzina obrade

Bez označavanja ciljanih slika, Chloros mora:

* Skenirajte svaku sliku u svom projektu.
* Pokrenite algoritme za otkrivanje cilja na svakoj slici.
* Nepotrebno provjeravajte stotine ili tisuće slika.

**Rezultat**: Obrada može trajati mnogo dulje, posebno za velike skupove podataka.

### S označenim ciljanim slikama

Kada označite stupac Target za određene slike:

* Chloros samo skenira označene slike u potrazi za ciljevima.
* Otkrivanje mete je puno brže.
* Ukupno vrijeme obrade znatno je smanjeno.

{% hint style=&quot;uspjeh&quot; %}
**Poboljšanje brzine**: Označavanje 2-3 ciljane slike u skupu podataka od 500 slika može smanjiti vrijeme otkrivanja cilja s više od 30 minuta na manje od 1 minute.
{% endhint %}

***

## Kako označiti ciljne slike

### Korak 1: Odredite svoje ciljane slike

Pregledajte uvezene slike u pregledniku datoteka i pronađite koje sadrže kalibracijske ciljeve.

**Uobičajeni scenariji:**

* **Meta prije snimanja**: Uhvaćena prije početka sesije.
* **Cilj nakon snimanja**: Snimljeno nakon završetka sesije.
* **Mete na terenu**: Mete postavljene unutar područja zarobljavanja.
* **Više ciljeva**: 2-3 ciljane slike po sesiji (preporučeno).

### Korak 2: Provjerite stupac Cilj.

Za svaku sliku koja sadrži kalibracijski cilj:

1. Pronađite sliku u tablici preglednika datoteka.
2. Pronađite stupac **Cilj** (krajnji desni stupac).
3. Pritisnite potvrdni okvir u stupcu Target za tu sliku.
4. Ponovite postupak za sve slike koje sadrže ciljeve.

### Korak 3: Provjerite svoj odabir.

Prije obrade provjerite sljedeće:

* [ ] Sve slike s ciljevima kalibracije su označene.
* [ ] Nema slučajno označenih neciljanih slika.
* [ ] Ciljevi su jasno vidljivi na označenim slikama.

***

## Najbolji postupci za ciljane slike

### Smjernice za hvatanje meta

**Trenutak:**

* Snimite ciljane slike neposredno prije i tijekom sesije snimanja.
* Pod istim svjetlosnim uvjetima kao i DAQ svjetlosni senzor.
* Idealno bi bilo da snimate slike meta što je češće moguće za najbolje rezultate. U suprotnom će se podaci svjetlosnog senzora koristiti za podešavanje kalibracije tijekom vremena.

**Pozicija kamere:**

*Držite kameru iznad leće tako da bude centrirana i da zauzima 40-60% središta slike.
* Držite kameru paralelno/nadirno s površinom leće.

**Munja:**

* Ista ambijentalna rasvjeta kao i vaš DAQ svjetlosni senzor.
* Izbjegavajte sjene na ciljnim površinama.
*Nemojte blokirati izvor svjetlosti svojim tijelom, vozilom ili vegetacijom.
*Oblačni uvjeti daju najdosljednije rezultate.

**Stanje objektiva:**

* Održavajte ciljne ploče čistima i suhima.
* Sve 4 ploče moraju biti jasno vidljive i bez prepreka.
*Mete trebaju biti okomite/nadir na izvor svjetlosti, ako je moguće.

### Koliko ciljnih slika?

**Minimum:** 1 ciljna slika po sesiji. **Preporučeno:** 3-5 ciljanih slika po sesiji.

**Kalendar dobrih praksi:**

* Snimite 3-5 slika ubrzo nakon što svjetlosni senzor počne snimati.
* Rotirajte kameru između snimanja za najbolje rezultate.
*Izborno: Povremeno usred sesije ako se uvjeti osvjetljenja stalno mijenjaju.

***

## Rad s više kamera

### Postavke dvostruke kamere

Ako koristite dvije MAPIR kamere istovremeno (npr. Survey3W RGN + Survey3N OCN):

1. Snimite ciljane slike s **obje kamere** u isto vrijeme.
2. Koristite **isti fizički objektiv** za obje kamere.
3. Provjerite ciljane slike za **obje vrste kamera** u pregledniku datoteka.
4. Chloros će koristiti odgovarajuće objektive za kalibraciju svake kamere.

### Stupac modela kamere

Stupac **Model fotoaparata** pomaže u prepoznavanju koje slike dolaze s kojeg fotoaparata:

* Anketa3W\_RGN
* Anketa3N\_OCN
* Anketa3W\_RGB
* itd.

Pomoću ovog stupca provjerite jeste li označili ciljeve za svaku vrstu kamere u svom projektu.

***

## Postavke detekcije cilja

### Podesite osjetljivost detekcije

Ako Chloros ne otkrije točno vaše ciljeve, prilagodite ove parametre u [Postavke projekta](adjusting-project-settings.md):

**Minimalno područje uzorka za kalibraciju:**

* **Zadano**: 25 piksela
* **Povećajte** ako dobijete lažna otkrivanja malih artefakata
* **Smanjenje** ako ciljevi nisu otkriveni

**Minimalno ciljano grupiranje:**

* **Zadano**: 60
* **Povećaj** ako su mete podijeljene u više detekcija.
* **Smanjenje** ako ciljevi s varijacijom boje nisu u potpunosti otkriveni.

***

## Uobičajeni problemi sa slikama objektiva

### Problem: Nijedan cilj nije otkriven.

**Mogući uzroci:**

*Ciljne slike nisu označene u pregledniku datoteka.
* Objektiv je premalen u okviru (< 30% slike).
* Loše osvjetljenje (sjene, refleksije)
* Postavke detekcije cilja prestroge

**Rješenja:**

1. Provjerite je li stupac Cilj označen za ispravne slike
2. Provjerite kvalitetu slike cilja u pregledu
3. Ponovno uhvatite ciljeve ako je kvaliteta loša
4. Po potrebi prilagodite postavke otkrivanja cilja

### Problem: lažne detekcije ciljeva

**Mogući uzroci:**

* Bijele zgrade, vozila ili vegetacijski pokrov zabunom se smatraju ciljevima.
* Svijetle točke na vegetaciji.
* Preniska osjetljivost detekcije.

**Rješenja:**

1. Označite samo prave ciljne slike kako biste ograničili raspon detekcije.
2. Povećajte minimalno područje uzorka za kalibraciju.
3. Povećajte minimalnu vrijednost ciljanog grupiranja.
4. Pobrinite se da slike mete prikazuju samo metu (minimalna pozadinska buka).

***

## Kontrolni popis

Prije početka obrade provjerite odabir ciljane slike:

* [ ] Najmanje 1 ciljana slika označena po sesiji.
* [ ] Potvrdni okviri stupca Target označeni su za sve ciljne slike.
* [ ] Ciljane slike snimljene u istom vremenskom intervalu kao i studija.
* [ ] Ciljevi jasno vidljivi u pregledu kada se klikne.
* [ ] Sve 4 kalibracijske ploče vidljive na svakoj ciljanoj slici.
* [ ] Na lećama nema sjena ili prepreka.
* [ ] Za dvostruke kamere: objektivi označeni za obje vrste kamera.

***

## Obrada bez cilja

### Obrada bez ciljeva kalibracije

Iako se ne preporučuje za znanstveni rad, možete obraditi bez ciljeva:

1. Ostavite neoznačene sve okvire u stupcu Cilj.
2. **Onemogućite** “Kalibraciju refleksije” u postavkama projekta.
3. Korekcija vinjete će se i dalje primjenjivati.
4. Izlaz neće biti kalibriran za apsolutnu refleksiju.

{% hint style=&quot;upozorenje&quot; %}
**Ne preporučuje se**: bez kalibracije refleksije, vrijednosti piksela predstavljaju samo relativnu svjetlinu, a ne znanstvena mjerenja refleksije. Koristite mete kalibracije za točne, ponovljive rezultate.
{% endhint %}

***

## Sljedeći koraci

Nakon što ste označili ciljne slike:

1. **Pregledajte postavke**: Pogledajte [Prilagodba postavki projekta](adjusting-project-settings.md)
2. **Pokrenite obradu** - Pogledajte [Pokretanje obrade](starting-the-processing.md)
3. **Nadgledajte napredak** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)

Za više informacija o ciljevima kalibracije pogledajte [Calibration Targets](../calibration-targets.md).