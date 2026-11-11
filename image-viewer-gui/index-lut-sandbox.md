# Index/LUT Sandbox

Index/LUT Sandbox je interaktivni radni prostor u bočnoj traci preglednika slika Chloros. Odaberete formulu, povežete kanale svoje kamere s njom, obojite je gradijentom i podesite raspon vrijednosti — a slika se ažurira uživo dok to radite. Od verzije 1.2.0 možete i **spremiti ono što ste izradili**, za jednu sliku ili za cijeli projekt, bez ponovne obrade.

## Za što služi Sandbox

| Index/LUT Sandbox (interaktivni)        | Obrada projekta (serijska)       |
| -------------------------------------- | -------------------------------- |
| Jedna slika odjednom, trenutačna povratna informacija  | Cijeli skup podataka u jednom pokretanju     |
| Eksperimentalno i iterativno             | Prethodno konfigurirane postavke          |
| Izvodi se uživo; sprema samo na zahtjev  | Uvijek zapisuje datoteke proizvoda      |
| Savršeno za pronalaženje pravih postavki | Najbolje kada su postavke konačne |

{% hint style="success" %}
**Uobičajeni tijek rada**: podesite Sandbox dok vizualizacija ne bude onakva kakvu želite, zatim izvezite izravno iz Sandboxa ili kopirajte iste postavke indeksa i LUT-a u [Postavke projekta](../project-settings/project-settings.md) kako bi ih sljedeći tijek obrade primijenio na svaku sliku.
{% endhint %}

***

## Otvaranje Sandboxa

1. Kliknite na sliku u mreži — otvara se u punom zaslonu na kartici **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Kliknite ikonu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> kako biste izvukli lijevu bočnu traku ako već nije otvorena
3. Odaberite višebandni sloj iz padajućeg izbornika slojeva u gornjem desnom kutu — **RAW (Reflectance)** je uobičajen izbor, jer su indeksne vrijednosti izračunate na kalibriranoj refleksiji usporedive između slika

Bočna traka prikazuje, odozgo prema dolje:

* naziv slike i njezin model kamere
* gumb **Izvoz/Spremi sliku(e)** — pojavljuje se nakon što se označi Indeks ili LUT
* potvrdne okvire za **Indeks**i**LUT**
* panel za konfiguraciju indeksa
* panel **Vrijednosti pokazivača** s očitavanjem, histogramom i kontrolom GSD-a

{% hint style="warning" %}
**Nije dostupno za mono kamere.** Na jednobandnoj LATTICE M3M slici oba su potvrdna okvira onemogućena, s alatnom napomenom _"Nije dostupno za mono (M3M) senzore"_ — višebandni indeks nije definiran na jednoj traci. Za izračunavanje indeksa s M3M kamera, kombinirajte dvije ili više u poravnati višebandni sloj i upotrijebite LATTICE mehanizam za indekse.
{% endhint %}

***

## Primjena indeksa

1. Označite okvir **Index** na vrhu bočne trake
2. Odaberite filtar vaše kamere iz padajućeg izbornika s lijeve strane (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Odaberite formulu indeksa iz padajućeg izbornika s desne strane — 27 ugrađenih formula, plus sve prilagođene formule koje ste spremili
4. Formula se ispod prikazuje kao matematička formula, s praznim krugom na svakom slotu trake. **Povucite obojeni krug kanala na slot** kako biste ga povezali
5. Kada su svi slotovi koje formula koristi povezani, slika se ažurira i prikazuje vrijednosti indeksa
6. Pomaknite pokazivač miša preko slike kako biste pročitali vrijednosti; panel **Vrijednosti pokazivača** dodaje redak indeksa s vrijednošću ispod pokazivača

Dvostruko kliknite na vezanu utičnicu kako biste je očistili. Nedovršena formula je normalno stanje tijekom povlačenja, a ne greška — slika se jednostavno ne ažurira dok formula nije dovršena.

Krugovi kanala su obojeni: crveno = Red, zeleno = Green, plavo = Blue, narančasto = Orange, cijan = Cyan, ljubičasto = NIR, magenta = RE. Ista se boja koristi za točkice kanala i krivulje histograma na ploči **Vrijednosti pokazivača**.

### Primjer indeksa s LUT-om

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Za potpunu referencu formule — sve tri liste prednastavki i koja imena gdje funkcioniraju — pogledajte [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

### Sa označenim indeksom, ali bez LUT-a

Slika se iscrtava u **sivoj skali**, rastegnuta između dviju vrijednosti praga. To je namjerno: indeksna slika su skalarni podaci, a siva skala je njihovo vjerno prikazivanje. Dodajte LUT kada želite boju.***

## Rad s LUT-ovima (tablicama za traženje)**Tablica za traženje** mapira indeksne vrijednosti na boje: ulaz NDVI 0,65, izlaz određena zelena boja. Ona ne mijenja podatke — mijenja način na koji ih čitate.

### Dodavanje LUT-a

1. Kliknite gumb <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">**"+ Dodaj LUT"** ispod formule
2. Odaberite gradijent boje
3. Postavite minimum i maksimum obrezivanja
4. Odaberite način obrezivanja
5. Označite okvir **LUT** u bočnoj traci za iscrtavanje

Kutija za potvrdu **LUT** ostaje onemogućena dok se LUT zapravo ne konfigurira na indeksu.

### Odabir gradijenta boja

Pređite mišem preko **trake gradijenta**kako biste otvorili popis unaprijed postavljenih vrijednosti — Chloros dolazi s**sedam** unaprijed postavljenih gradijenata:

| # | Gradijent                            | Oblik                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Žuta → Green (**zadano**)  | Divergentno — odgovara uobičajenoj intuiciji o vegetaciji, zelena = zdrava |
| 2 | Ljubičasta → Žuta → Green             | Divergentna, s izraženim donjim krajem                                  |
| 3 | Smeđa → Bijela → Blue                | Divergentna oko svijetle sredine                                   |
| 4 | Crna → Ljubičasta → Ružičasta → Blago žuta | Sekvencijalna, od tamne do svijetle                                           |
| 5 | Red → Žuta → Blue                 | Divergentni, oko svijetlog središnje tonusa                                   |
| 6 | Ljubičasta → Blue → Green → Žuta      | Sekvencijalni, od tamnog prema svijetlom                                           |
| 7 | Orange → Bijela → Ljubičasta            | Divergentni, oko svijetlog središnje tonusa                                   |

**Divergentni**gradijent postavlja neutralnu boju u sredinu vašeg prozora, što je dobro čitljivo kada srednja točka ima značenje (prag, referentni datum).**Sekvencijalni** gradijent monotono teče od tamne do svijetle boje, što je dobro čitljivo za količinu koja ima samo "više" i "manje".

Svaki preset ima sedam bojenih stanica. Kliknite na preset i slika se odmah ažurira (kada je potvrđen okvir za LUT).

### Uređivanje bojenih stanica

Ispod trake gradijenta nalazi se red uzoraka boja, po jedan za svaku stanicu:

* **Promijenite boju**: kliknite na uzorak boje kako biste otvorili odabirač boja (kolo boja, klizači za odabir boje /HSV ili heksadecimalni kod kao što je `#FF0000`)
* **Dodaj stanku**: kliknite gumb**+** na kraju retka — dodaje se bijela stanka
* **Ukloni stanku**:**dvaput kliknite** na uzorak boje
* **Sačuvajte uređeni gradijent**: kliknite ikonu spremanja pokraj trake s gradijentom kako biste dodali svoj uređeni gradijent na popis unaprijed postavljenih vrijednosti i mogli ga ponovno odabrati

Gradijent koji ste konfigurirali na indeksu pohranjuje se s tim indeksom u postavkama projekta, tako da ostaje sačuvan nakon zatvaranja i ponovnog otvaranja projekta.

**Manje stanica**stvara oštre zone koje se tumače kao klasifikacija;**više stanica** stvara glatke, gotovo fotografske prijelaze. Tri do pet stanica pogodno je za prezentacijske slajdove i karte klasifikacije; šest do deset za opću analizu; petnaest ili više za detaljni pregled i publikacijske grafike.

### Postavljanje raspona vrijednosti

Kontrola praga je **klizač s dvostrukom ručkom**koji se proteže od −1 do +1, s uređivim tekstualnim poljem na svakom kraju za točne vrijednosti i gumbom**AUTO**.

* Povucite bilo koju ručku ili upišite broj u njezino polje i pritisnite Enter
* **AUTO**postavlja raspon na**

2. i 98. percentil** vrijednosti indeksa slike — dobra polazna točka koja zanemaruje odstupajuće vrijednosti. Chloros prilagodljivo zaokružuje rezultat, na 4 decimalna mjesta za vrlo uzak raspon, 3 za uzak raspon i 2 u ostalim slučajevima
* Bilo koja ručna prilagodba ima prednost nad postavkom AUTO dok ponovno ne pritisnete AUTO

Primjer prozora za NDVI:

| Cilj                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Prikaži sve                            | −1.0 | 1.0 |
| Samo vegetacija, isključi tlo i vodu | 0.2  | 0.9 |
| Samo zdrava vegetacija                | 0.5  | 0.9 |
| Naglasiti stres                        | 0.2  | 0.5 |

Sužavanje prozora povećava kontrast unutar vašeg područja interesa i sve ostalo izbacuje izvan dosega — gdje **način obrezivanja** odlučuje što se s tim događa.***

## Načini obrezivanja

Kada vrijednost indeksa piksela izađe izvan prozora min/max, način obrezivanja odlučuje kako će biti iscrtan.

| Naziv padajućeg izbornika | Pohranjena vrijednost | Piksele izvan raspona iscrtava se kao                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum i maksimum** (zadano) | `clip`            | Najbliža boja na kraju gradijenta — vrijednosti ispod minimuma uzimaju prvu boju, vrijednosti iznad maksimuma uzimaju posljednju |
| **Prozirna pozadina**      | `transparent`     | Potpuno prozirna (prava alfa)                                                                                                  |
| **Indeksna pozadina**| `indexColor`      | Siva skala, rastegnuta preko**cjelokupnog** indeksnog raspona slike, tako da je struktura izvan raspona i dalje vidljiva u sivoj boji                |
| **Izvorna pozadina**         | `backgroundColor` | Sama podloga slike, tako da se boja preklapa na vrh stvarne scene                                                |

| Način                       | Najbolje za                               | Izgled                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimalno i maksimalno**      | Potpuni prikaz podataka, znanstvena analiza | Svaki piksel obojen                      |
| **Prozirna pozadina**        | GIS preklopi, izoliranje vrijednosnog pojasa  | Boja unutar prozora, ništa izvan |
| **Pozadina s indeksom**       | Naglašavanje uz očuvanje konteksta podataka | Boja unutra, sivo vani               |
| **Izvorna pozadina**       | Izvještaji i prezentacije              | Boja unutra, fotografija vani         |

{% hint style="info" %}
**Pikseli bez podataka uvijek su prozirni, u svakom načinu rada.** Piksel čiji je indeks nefinit (dijeljenje s 0/0) ili je točno −1,0 ili +1,0 (indikatori zasićenosti, kada jedna traka mjeri nulu dok druga ne) tretira se kao piksel bez podataka, a ne kao ekstremna vrijednost. Na taj se način izbjegava da preeksponirane svjetline i potpuno zasjenjena područja budu prikazana na vašoj skali boja kao najekstremnije vrijednosti u kadru. Isti pravilo definira koje piksele napajaju AUTO prage i histogram indeksa, pa su sva tri usklađena.
{% endhint %}

Prozirnost se zadržava kada se izvoz zapisa kao PNG. Ne može se predstaviti u JPG formatu.

***

## Čitanje vrijednosti tijekom podešavanja

Ploča **Vrijednosti pokazivača** ispod konfiguracijske ploče mjerni je instrument za Sandbox:

* Pomaknite pokazivač preko slike i pročitajte izvorišne vrijednosti po kanalu, plus vrijednost indeksa u vlastitom retku
* Uključite gumb **INDEKS** iznad histograma kako biste vidjeli raspodjelu indeksnih vrijednosti u kadru, pri čemu su vaša dva praga klipa iscrtana kao narančaste isprekidane linije, a vrijednost kursora kao bijela linija — ovo je najbrži način odabira prozora koji doista sadrži vaše podatke
* Uključite **KURSOR** kako biste vidjeli linije markera na vrijednostima ispod pokazivača
* Zumirajte više od 60× (manje ako je postavljena veličina GSD bloka) kako biste istaknuli pojedinačne prikazane piksele lebdećom vrijednošću

Praktična rutina:

1. Zabilježite vrijednosti iznad zdravog raslinja, pod stresom, gole zemlje i vode
2. Pogledajte gdje se ti klasteri nalaze na histogramu indeksa
3. Postavite min/max kako biste obuhvatili klaster koji vas zanima
4. Odaberite način obrezivanja — _Original Background_ zadržava scenu vidljivom oko njega

***

## Izvoz iz Sandboxa

Sve navedeno gore je prikaz uživo dok ga ne spremite. Gumb **Izvoz/Spremanje slike(a)** na vrhu bočne trake otvara prozor koji se klizi preko bočne trake (umjesto da prekriva sliku, tako da i dalje možete vidjeti o čemu odlučujete).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opcije

| Opcija                          | Učinak                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Primijeni na trenutnu sliku**      | Sačuvajte točno sliku koja se prikazuje, s ovim postavkama                                                                                                |
| **Primijeni na sve slike u projektu** | Ponovno pokreće istu konfiguraciju na svakoj slici u projektu. Slike bez pojaseva koje ovaj indeks treba se preskaču i ne tretiraju se kao neuspjesi |
| **Indeksna/LUT gradijentna traka**      | Također zapisuje zasebnu sliku legende za svaki izvoz, s označenim rasponom vrijednosti                                                                     |
| **Indeksni histogram**             | Također zapisuje zasebnu sliku histograma za svaki izvoz, koja prikazuje min/max podataka i pragove za obrezivanje (clip)                                               |

Ako je **veličina GSD bloka** na kartici slike veća od 1, prozor će vas na to upozoriti prije potvrde: izvoz spremava ono što gledate, uključujući prosječno izračunavanje po blokovima. Prvo vratite kontrolu GSD na 1 ako želite punu rezoluciju.

### Kamo se datoteke spremaju

Svaki klik na **Izvezi**stvara**novi, nikad neupotrebljavani mapu**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Primjeri: `Sandbox_Exports/NDVI_LUT_001/`, a zatim `Sandbox_Exports/NDVI_LUT_002/` za sljedeće pokretanje. Brojanje se temelji na skeniranju sadržaja diska, stoga ostaje nepromijenjeno nakon ponovnog pokretanja i brisanja mapa ručno. Ništa se nikada ne prepišava — cijela je svrha Sandboxa uspoređivanje jednog pokušaja s prethodnim.

Unutar mape, za svaku sliku:

| Datoteka                                                   | Sadržaj                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Renderirana slika, piksel za piksel ono što je prikazivač pokazao |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Sporedna datoteka s gradijentnom trkom, ako je zatraženo                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Sporedna datoteka s histogramom indeksa, ako je zatraženo                  |

Dva sporedna zapisa uvijek se spremaju u **punoj rezoluciji**, čak i kada je glavna slika prosječno obrađena po blokovima: veličina bloka je rezolucija zaslona, a oba sporedna zapisa čitaju stvarne indeksne vrijednosti po pikselu. Također ispisuju više podataka nego verzije na zaslonu — obje ispisuju prozor za istezanje _i_ stvarne min/max vrijednosti podataka, pa je spremljena legenda i dalje čitljiva mjesecima kasnije bez otvorenog projekta.

### Napredak i rezultati

Izvoz cijelog projekta traje nekoliko minuta, pa tijekom izvoza primate povratne informacije putem kanala za praćenje napretka u stvarnom vremenu, umjesto da program blokira:

* Traka napretka prikazuje `current / total` i datoteku koja se zapisuje
* Kada završi, okno izvještava o tome koliko je slika izvezeno, koliko je preskočeno i o putanji izlazne mape
* Preskočene slike su navedene s razlogom (prikazuje se do pet razloga, a zatim redak "+N više"). Uobičajeni razlog je sloj koji nema kanale potrebne za ovaj indeks
* Ako **ni** jedna slika u projektu ne može koristiti indeks, izvršavanje se prijavljuje kao neuspješno umjesto da vam ostavi praznu mapu

U jednom trenutku može se izvršavati samo jedno izvoženje u sandboxu. Pokretanje drugog dok je prvo u tijeku odbijeno je jasnom porukom umjesto da se dva izvršavanja natječu za istu datoteku projekta.

### Mreža preuzima pokretanje

Svako dovršeno pokretanje pojavljuje se kao vlastito gumb na alatnoj traci [mreže slika](image-grid.md), s oznakom `<IndexName> <Index|LUT> <NNN>`. Tako uspoređujete pokrete: izvezite dva puta s različitim gradijentima ili pragovima, a zatim prebacujte između dva gumba na mreži.

***

## Prilagođene formule indeksa (Chloros+)

{% hint style="info" %}
**Gdje ih stvoriti**: u bočnoj traci Sandboxa ili u**Postavkama projekta** prije obrade. Oba se upisuju u istu listu na razini projekta.
{% endhint %}

1. Otvorite kalkulator prilagođenih formula iz padajućeg izbornika za formulu indeksa (zahtijeva prijavu s kvalificiranom pretplatomChloros

+)
2. Napišite formulu koristeći **simbole traka** `x`, `y`, `z`, `a`, `b`, `c` — nisu nazivi bendova
3. Dostupni operateri: `+`, `-`, `*`, `/`, `^` i `()` za grupiranje
4. Dostupne funkcije: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nazovite ga i spremite — pojavljuje se na dnu padajućeg izbornika formula i njegove utore povezujete povlačenjem krugova kanala, točno kao kod ugrađenog preseta

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Prilagođene formule su dostupne samo putem GUI-ja.** OpcijaCLI

/SDK `--indices` proširuje 22 ugrađena naziva prednastavki i tiho preskače sve ostalo, uključujući vaše prilagođene formule. Da biste grupno primijenili prilagođenu formulu, konfigurirajte je u Postavkama projekta i pokrenite obradu ili upotrijebite izvoz "Primijeni na sve slike projekta" u Sandboxu.
{% endhint %}

***

## Rješavanje problema

### "Ovaj sloj nema kanale koji su potrebni ovom indeksu"

Formula čita položaj kanala koji trenutni sloj nema — na primjer, indeks s tri utora na datoteci s jednim ili dva kanala. Prelazite na višebandni sloj (reflektancije ili debayered), ili odaberite indeks koji odgovara filtru vaše kamere.

### "Nije moguće dohvatiti pozadinski sustav za obradu slika"

Pozadinski sustav ne odgovara. Provjerite karticu Zapisnik (Log); ako se pozadinski proces (backend) ponovno pokreće, Sandbox se sam oporavi čim se on vrati.

### Slika se nije promijenila kada sam povukao krug

Formula još nije dovršena. Nedovršena formula tretira se kao normalno stanje tijekom povlačenja — ništa se ne iscrtava i ništa se ne prijavljuje kao pogreška. Ispunite svako mjesto koje formula koristi.

### Cijela slika je jedne boje

Vaš prozor za obrezivanje vjerojatno je daleko izvan podataka. Pritisnite **AUTO**kako biste ga pomaknuli na 2. / 98. percentil, ili uključite histogram**INDEX** kako biste vidjeli gdje se podaci zapravo nalaze.

### Izvezene boje ne odgovaraju onome što sam vidio

Trebale bi — put izvoza je namjerno ogledalo uživo pretpregleda, uključujući alfu u načinu obrezivanja, a prosječenje po blokovima primjenjuje se _nakon_ obojavanja točno onako kako to radi preglednik. Ako se razlikuju, provjerite je li se veličina GSD bloka promijenila između pregledavanja i izvoza.

***

## Sljedeći koraci

* [**Slojevi slike**](image-layers.md) — na kojem sloju izvršiti indeks i što znače njegove vrijednosti
* [**Otvaranje slike u punom zaslonu**](opening-an-image-full-screen.md) — detaljno o prikazu kursora, histogramu i GSD kontroli
* [**Multispektralne formule indeksa**](../project-settings/multispectral-index-formulas.md) — svaki preddefinirani indeks, na svakoj površini
* [**Postavke projekta**](../project-settings/project-settings.md) — spremanje pronađenih postavki u izvršavanje obrade
