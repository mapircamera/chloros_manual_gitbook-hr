# Odabir slika s ciljnim oznakama

Označavanje slika koje sadrže ciljne oznake govori Chlorosu točno gdje ih treba tražiti. Kada je u stupcu Cilj označena barem jedna slika, Chloros skenira **samo označene slike** — stoga je označavanje ciljeva način na koji ubrzavate obradu i sprječavate da se slike iz snimanja zamijene za cilj.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Zašto označavati ciljne slike?

### Označavanje kontrolira skeniranje

Kada označite stupac Cilj za određene slike:

* Chloros skenira samo označene slike u potrazi za ciljevima
* Otkrivanje ciljeva završava puno brže
* Slike iz snimanja ne mogu proizvesti lažna otkrivanja ciljeva

Ako **ni** jedna slika nije označena, Chloros se vraća na skeniranje svake slike u projektu:

* Algoritmi za detekciju ciljeva rade na svakoj slici
* Stotine ili tisuće slika se nepotrebno provjeravaju
* Obrada traje znatno duže, posebno za velike skupove podataka

{% hint style="success" %}
**Poboljšanje brzine**: Označavanje 2-3 ciljne slike u skupu od 500 slika može smanjiti vrijeme detekcije cilja s više od 30 minuta na manje od 1 minute.
{% endhint %}

***

## Kako označiti slike ciljeva

### Korak 1: Identificirajte svoje slike ciljeva

Pregledajte svoje uvezene slike u pregledniku datoteka i identificirajte koje slike sadrže ciljeve kalibracije.

**Uobičajeni scenariji:*** **Cilj prije snimanja**: Snimljen prije početka sesije
* **Cilj nakon snimanja**: Snimljen nakon završetka sesije
* **Ciljevi na terenu**: Ciljevi postavljeni unutar područja snimanja
* **Više ciljeva**: 2-3 slike ciljeva po sesiji (preporučeno)

### Korak 2: Provjerite stupac Cilj <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Za svaku sliku koja sadrži kalibracijski cilj:

1. Pronađite sliku u tablici Preglednika datoteka
2. Pronađite stupac **Cilj** (najdesniji stupac)
3. Kliknite na potvrdni okvir u stupcu Cilj za tu sliku
4. Ponovite postupak za sve slike koje sadrže ciljeve

### Korak 3: Provjerite svoj odabir

Prije obrade, dvaput provjerite:

* [ ] Sve slike s kalibracijskim ciljevima su označene
* [ ] Nijedna slika koja nije cilj nije slučajno označena
* [ ] Ciljevi su jasno vidljivi na označenim slikama

***

## LATTICE: Ciljevi su neobavezni kada DAQ snima

Za LATTICE multispektralne kamere, kalibracijski cilj unutar kadra **jedna je od dvije** moguće reference apsolutne reflektancije:

* **Cilj unutar kadra**: kada označena slika cilja prođe kontrole kvalitete (QA) alata Chloros, cilj postaje**apsolutna referenca odbijanja** za slike oko njega.
* **DAQ downwelling**: kada cilj nije prisutan (ili QA ne uspije), Chloros izračunava reflektanciju umjesto toga na temelju dolazne zračenosti DAQ senzora svjetlosti (ρ = π·L/E). Ako snimanje pomoću `.daq` ili DAQ-M `.csv` pokriva vaše snimke, dobivate kalibriranu reflektanciju**bez ikakvih ciljnih slika**.

Ovo automatsko ponašanje je zadano. U CLI / SDK to odgovara `--reflectance-source auto`; također možete prisiliti `target` (strogo — bez zamjene DAQ-om) ili `daq` (autoritativno-DAQ). Pogledajte [CLI reference](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrije ciljeva LATTICE**: osim klasične detekcije panela koja se koristi za Survey3, obrada LATTICE podržava**ciljeve označene ArUco oznakama**,**ciljeve s fiksnim ROI-jem**i**ciljeve u obliku trake**, konfigurirane za svaki projekt. Pojedinačni**izmjereni** skenovi reflektancije cilja mogu se isporučiti prema serijskom broju (CLI: `--target-reflectance-dir`, jedan `<serial>.csv` po jedinici cilja), s nominalnim T3/T4P spektrima kao rezervnom opcijom.

{% hint style="info" %}
**Modul F988**: Reflektancija F988 kalibrira se pomoću panela za reflektanciju unutar scene: pojas leži izvan kalibriranog raspona DAQ senzora svjetla, stoga Chloros primjenjuje vašu najnoviju snimku panela i zadržava je između promatranja panela. Ako se modul F988 obrađuje samo putem DAQ-a, Chloros odbacuje reflektanciju temeljenu na DAQ-u za tu traku (razlog preskakanja `dls-uncalibrated-band-988`) — radni postupak s panelom je podržani put.
{% endhint %}

***

## Najbolje prakse za slike cilja

### Smjernice za snimanje cilja

**Vremensko određivanje:**

* Snimite slike cilja odmah prije i tijekom cijele sesije snimanja
* U istim uvjetima osvjetljenja kao i vaš DAQ senzor svjetla
* Idealno je snimati slike cilja što je češće moguće za najbolje rezultate. U suprotnom će se podaci sa senzora svjetla koristiti za prilagodbu kalibracije tijekom vremena.

**Pozicija kamere:**

* Držite kameru iznad cilja tako da bude centrirana i ispunjava oko 40-60 % središnjeg dijela slike.
* Držite kameru paralelno/u nadiru s površinom cilja

**Rasvjeta:**

* Ista ambijentalna rasvjeta kao i kod vašeg DAQ senzora svjetla
* Izbjegavajte sjene na površinama cilja
* Ne blokirajte izvor svjetla svojim tijelom, vozilom ili vegetacijom
* Uvjeti s naoblakom pružaju najkonzistentnije rezultate

**Stanje cilja:**

* Držite ciljne ploče čistima i suhima
* Sve ploče vašeg cilja (npr. sve 4 na T4) trebaju biti jasno vidljive i neometane
* Ako je moguće, cilj postavite okomito/u nadirnom smjeru u odnosu na izvor svjetla

### Koliko slika cilja?

**Minimalno:**1 slika mete po sesiji.**Preporučeno:** 3-5 slika mete po sesiji.**Najbolja praksa rasporeda:**

* 3-5 slika snimite ubrzo nakon što senzor svjetla zabilježi
* Okrenite kameru između snimanja za najbolje rezultate
* Po želji: povremeno usred sesije ako se uvjeti osvjetljenja stalno mijenjaju

***

## Rad s više kamera

### Postavke s dvije kamere

Ako istovremeno koristite dvije kamere MAPIR (npr. Survey3W RGN + Survey3N OCN):

1. Snimite slike cilja s **obje kamere** u isto vrijeme
2. Koristite **isti fizički cilj** za obje kamere
3. Označite slike cilja za **obje vrste kamera** u pregledniku datoteka
4. Chloros će koristiti odgovarajuće ciljeve za kalibraciju svake kamere

### stupac Modela kamere

Stupac **Modela kamere** pomaže identificirati koje su slike došle iz koje kamere:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* itd.

Koristite ovu stupac kako biste provjerili jeste li označili ciljeve za svaku vrstu kamere u svom projektu.

***

## Postavke detekcije ciljeva

### Podesite osjetljivost detekcije

Ako Chloros ne detektira vaše ciljeve ispravno, prilagodite ove postavke u [Postavke projekta](adjusting-project-settings.md):**Minimalna površina kalibracijske uzorke (px):*** **Zadano**: 25 piksela
* **Povećajte** ako se javljaju lažne detekcije na malim artefaktima
* **Smanjite** ako se ciljevi ne detektiraju**Minimalno grupiranje ciljeva (0-100):*** **Zadano**: 60
* **Povećajte** ako se ciljevi dijele na više detekcija
* **Smanjite** ako se ciljevi s varijacijom boje ne detektiraju u potpunosti

{% hint style="info" %}
**Savjet za CLI**: `chloros-cli process` prihvaća iste kontrole (`--min-target-size`, `--target-clustering`), a njezina zastavica `--target`/`--targets` označava cijelu ulaznu mapu kao samo za ciljni panel. Pogledajte [CLI Referencu](../reference/cli-reference.md).
{% endhint %}

***

## Uobičajeni problemi s ciljnim slikama

### Problem: Nisu otkriveni ciljevi

**Mogući uzroci:**

* Ciljne slike nisu označene u pregledniku datoteka
* Cilj je premalen u kadru (&lt; 30 % slike)
* Loše osvjetljenje (sjene, odbljesak)
* Postavke za otkrivanje ciljeva su previše stroge

**Rješenja:**

1. Provjerite je li stupac Cilj označen za ispravne slike
2. Provjerite kvalitetu slike cilja u pretpregledu
3. Ponovno snimite ciljeve ako je kvaliteta loša
4. Po potrebi prilagodite postavke detekcije cilja

### Problem: Lažne detekcije cilja

**Mogući uzroci:**

* Bijele zgrade, vozila ili pokrov tla pogrešno se smatraju ciljevima
* Svetli mrlji u vegetaciji
* Preosjetljivost detekcije je preniska

**Rješenja:**

1. Označite samo stvarne slike ciljeva — samo označene slike se skeniraju
2. Povećajte minimalnu površinu uzorka za kalibraciju
3. Povećajte minimalnu vrijednost grupiranja ciljeva
4. Osigurajte da slike ciljeva prikazuju samo cilj (minimalna pozadinska zbrka)

***

## Kontrolna lista za provjeru

Prije početka obrade provjerite odabir slika ciljeva:

* [ ] Označeno najmanje 1 slika cilja po sesiji (ili, za LATTICE, snimka `.daq`/`.csv` koja pokriva sesiju)
* [ ] Za sve ciljne slike označene su potvrdne okvire za stupac cilja
* [ ] Ciljne slike snimljene u istom vremenskom razdoblju kao i pregled
* [ ] Ciljevi su jasno vidljivi u pretpregledu nakon klika
* [ ] Svi kalibracijski paneli vidljivi su na svakoj ciljnoj slici
* [ ] Na ciljevima nema sjena ni prepreka
* [ ] Za dvostruku kameru: Ciljevi su označeni za oba tipa kamera

***

## Obrada bez ciljeva

### LATTICE: S DAQ snimkom

Ako je DAQ senzor svjetla snimao zračenje prema dolje tijekom vaših LATTICE snimanja, cilj nije potreban:

1. Uveзите datoteku `.daq` (ili DAQ-M `.csv`) sa slikama
2. Ostavite stupac Cilj (Target) neoznačenim
3. Reflektancija se automatski izračunava iz referentne vrijednosti zračenja prema dolje (downwelling) iz DAQ-a
4. Radijansa nikada ne treba referentnu ploču niti DAQ — dobiva se isključivo iz tvorničke radiometrijske kalibracije kamere

### Obrada bez ikakve referentne ploče

Također možete obraditi bez referentnih ploča i bez DAQ-a:

1. Ostavite sve potvrdne okvire u stupcu &quot;Target&quot; neoznačene
2. **Onemogućite** &quot;Kalibraciju reflektancije / balans bijele&quot; u Postavkama projekta — detekcija cilja tada se u potpunosti preskače
3. Korekcija vinjetiranja i dalje će se primijeniti
4. Izlazni proizvod neće biti kalibriran za apsolutnu reflektanciju (LATTICE multispectral i dalje izvozi debayered, pretpregledne i radijancijske proizvode)

{% hint style="warning" %}
**Ne preporučuje se za znanstveni rad s Survey3**: Bez kalibracije reflektancije, vrijednosti piksela u Survey3 predstavljaju samo relativnu svjetlinu, a ne znanstvena mjerenja reflektancije. Koristite kalibracijske mete (ili, za LATTICE, DAQ senzor svjetla) za točne, ponovljive rezultate.
{% endhint %}

***

## Sljedeći koraci

Nakon što ste označili ciljne slike:

1. **Provjerite postavke** - Pogledajte [Podešavanje postavki projekta](adjusting-project-settings.md)
2. **Pokrenite obradu** - Vidi [Pokretanje obrade](starting-the-processing.md)
3. **Pratite napredak** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)

Za više informacija o samim kalibracijskim targetima pogledajte [Kalibracijski targeći](../calibration-targets.md).
