# Odabir ciljanih slika

Označavanje slika koje sadrže kalibracijske ciljeve ključni je korak koji značajno ubrzava proces obrade Chlorosa. Prethodnim odabirom ciljanih slika eliminirate potrebu da Chloros skenira svaku sliku u vašem skupu podataka radi kalibracijskih ciljeva.

## Zašto označiti ciljne slike?

### Brzina obrade

Bez označavanja ciljanih slika, Chloros mora:

* Skenirajte svaku sliku u svom projektu
* Pokrenite algoritme za otkrivanje cilja na svakoj slici
* Nepotrebno provjeravajte stotine ili tisuće slika

**Rezultat**: Obrada može trajati znatno dulje, posebno za velike skupove podataka.

### S označenim ciljnim slikama

Kada označite stupac Target za određene slike:

* Chloros skenira samo označene slike u potrazi za ciljevima
* Detekcija cilja je mnogo brža
* Sveukupno vrijeme obrade znatno je smanjeno

{% hint style="success" %}
**Poboljšanje brzine**: Označavanje 2-3 ciljane slike u skupu podataka od 500 slika može smanjiti vrijeme otkrivanja cilja s 30+ minuta na manje od 1 minute.
{% endhint %}***## Kako označiti ciljne slike

### Korak 1: Odredite svoje ciljane slike

Pregledajte svoje uvezene slike u pregledniku datoteka i identificirajte koje slike sadrže ciljeve kalibracije.

**Uobičajeni scenariji:**

* **Meta prije snimanja**: Uhvaćena prije početka sesije
* **Meta nakon snimanja**: Uhvaćena nakon završetka sesije
* **Mete unutar polja**: Mete postavljene unutar područja zarobljavanja
* **Više ciljeva**: 2-3 ciljane slike po sesiji (preporučeno)

### Korak 2: Provjerite ciljni stupac

Za svaku sliku koja sadrži kalibracijski cilj:

1. Pronađite sliku u tablici File Browser
2. Pronađite stupac**Cilj** (krajnji desni stupac)
3. Pritisnite potvrdni okvir u stupcu Cilj za tu sliku
4. Ponovite za sve slike koje sadrže ciljeve

### Korak 3: Potvrdite svoj odabir

Prije obrade još jednom provjerite:

* [ ] Provjeravaju se sve slike s ciljevima kalibracije
* [ ] Nijedna neciljana slika nije slučajno označena
* [ ] Mete su jasno vidljive na označenim slikama

***## Najbolji postupci za ciljane slike

### Smjernice za hvatanje meta**Vrijeme:**

* Snimite ciljane slike neposredno prije i tijekom sesije snimanja
* U istim uvjetima osvjetljenja kao i vaš DAQ svjetlosni senzor
* U idealnom slučaju snimajte ciljane slike što je češće moguće za najbolje rezultate. U suprotnom će se podaci svjetlosnog senzora koristiti za podešavanje kalibracije tijekom vremena.

**Položaj kamere:**

* Držite kameru iznad cilja tako da je centrirana i ispunjava oko 40-60% središta slike.
* Držite kameru paralelno/nadir s ciljanom površinom

**Rasvjeta:**

* Ista ambijentalna rasvjeta kao i vaš DAQ svjetlosni senzor
* Izbjegavajte sjene na ciljanim površinama
* Nemojte blokirati svoj izvor svjetlosti svojim tijelom, vozilom ili vegetacijom
* Oblačni uvjeti daju najdosljednije rezultate

**Ciljano stanje:**

* Održavajte ciljne ploče čistima i suhima
* Sve 4 ploče trebaju biti jasno vidljive i bez zapreka
* Ciljajte okomito/nadir na izvor svjetlosti ako je moguće

### Koliko ciljanih slika?

**Minimum:**1 target image per session.**Recommended:**3-5 target images per session.**Raspored najbolje prakse:**

* 3-5 slika snimljenih nedugo nakon snimanja svjetlosnog senzora
* Rotirajte kameru između snimanja za najbolje rezultate
* Izborno: povremeno usred sesije ako se uvjeti osvjetljenja stalno mijenjaju

***## Rad s više kamera

### Postavke s dvije kamere

Ako koristite dvije MAPIR kamere istovremeno (npr. Survey3W RGN + Survey3N OCN):

1. Snimite ciljane slike s**obje kamere**u isto vrijeme
2. Koristite**isti fizički cilj**za obje kamere
3. Označite ciljane slike za**obje vrste kamera**u pregledniku datoteka
4. Chloros će koristiti odgovarajuće mete za kalibraciju svake kamere

### Stupac modela kamere

Stupac**Model kamere** pomaže u prepoznavanju koje slike dolaze s kog fotoaparata:

* Anketa3W\_RGN
* Anketa3N\_OCN
* Anketa3W\_RGB
* itd.

Upotrijebite ovaj stupac kako biste potvrdili da ste označili ciljeve za svaku vrstu kamere u svom projektu.

***## Postavke otkrivanja cilja

### Podešavanje osjetljivosti detekcije

Ako Chloros ne otkriva ispravno vaše ciljeve, prilagodite ove postavke u [Postavke projekta](adjusting-project-settings.md):**Minimalna površina uzorka za kalibraciju:**

* **Zadano**: 25 piksela
* **Povećajte** ako dobivate lažna otkrivanja malih artefakata
* **Smanji**ako se ciljevi ne otkrivaju**Minimalno ciljno grupiranje:**

* **Zadano**: 60
* **Povećaj** ako se ciljevi dijele na više detekcija
* **Smanji**ako ciljevi s varijacijom boje nisu u potpunosti otkriveni***

## Uobičajeni problemi s ciljnom slikom

### Problem: Nisu otkriveni ciljevi

**Mogući uzroci:**

* Ciljajte slike koje nisu označene u pregledniku datoteka
* Cilj je premalen u okviru (< 30% slike)
* Loše osvjetljenje (sjene, odsjaj)
* Postavke detekcije cilja prestroge

**Rješenja:**1. Provjerite ima li u stupcu cilja točne slike
2. Pregledajte ciljnu kvalitetu slike u pregledu
3. Ponovno uhvatite ciljeve ako je kvaliteta loša
4. Po potrebi prilagodite postavke otkrivanja cilja

### Problem: lažne detekcije ciljeva**Mogući uzroci:**

* Bijele zgrade, vozila ili pokrivač tla zabunom se smatraju ciljevima
* Svijetle mrlje u vegetaciji
* Preniska osjetljivost detekcije

**Rješenja:**1. Označite samo stvarne ciljne slike kako biste ograničili opseg detekcije
2. Povećajte minimalno područje uzorka za kalibraciju
3. Povećajte minimalnu ciljnu vrijednost klasteriranja
4. Pobrinite se da ciljne slike prikazuju samo cilj (minimalni nered u pozadini)***

## Popis za provjeru

Prije početka obrade provjerite odabir ciljane slike:

* [ ] Najmanje 1 ciljana slika označena po sesiji
* [ ] Potvrdni okviri ciljnog stupca označeni su za sve ciljane slike
* [ ] Ciljajte slike snimljene unutar istog vremenskog okvira kao i istraživanje
* [ ] Ciljevi jasno vidljivi u pregledu kada se klikne
* [ ] Sve 4 kalibracijske ploče vidljive na svakoj ciljanoj slici
* [ ] Nema sjena ili prepreka na metama
* [ ] Za dvostruku kameru: Mete označene za obje vrste kamera

***## Obrada bez cilja

### Obrada bez kalibracijskih ciljeva

Iako se ne preporučuje za znanstveni rad, možete obraditi bez ciljeva:

1. Ostavite sve potvrdne okvire Target column neoznačene
2.**Onemogući**"Kalibraciju refleksije" u postavkama projekta
3. Korekcija vinjete i dalje će se primjenjivati
4. Izlaz neće biti kalibriran za apsolutnu refleksiju

{% hint style="warning" %}**Ne preporučuje se**: bez kalibracije refleksije, vrijednosti piksela predstavljaju samo relativnu svjetlinu, a ne znanstvena mjerenja refleksije. Koristite mete kalibracije za točne, ponovljive rezultate.
{% endhint %}***## Sljedeći koraci

Nakon što označite svoje ciljne slike:

1. **Pregledajte svoje postavke**- Pogledajte [Prilagodba postavki projekta](adjusting-project-settings.md)
2.**Pokreni obradu**- Vidi [Pokretanje obrade](starting-the-processing.md)
3.**Praćenje napretka** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)

Za više informacija o samim ciljevima kalibracije pogledajte [Calibration Targets](../calibration-targets.md).
