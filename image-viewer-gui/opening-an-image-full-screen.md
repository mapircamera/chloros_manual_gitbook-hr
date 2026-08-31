# Otvaranje slike u punom zaslonu

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Slika otvorena u punom zaslonu, s odabiračem slojeva u gornjem desnom kutu</p></figcaption></figure>

Chloros preglednik slika je sučelje za pregled, ispitivanje i mjerenje vaših slika u punoj veličini. Tu čitate **stvarne vrijednosti piksela** — DN po kanalu, postotak reflektancije ili radijancu u W/m²/sr/nm — umjesto iskrivljenog pretpregleda koji zaslon prikazuje.

## Pristup pregledniku slika

### Iz preglednika datoteka

1. Otvorite karticu **Preglednik datoteka** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite na bilo koju **minijaturu** u [mreži slika](image-grid.md)
3. Slika se otvara na cijelom zaslonu u kartici **Preglednika slika**

Slika se otvara u proizvodu koji je prikazivao mrežu. Ako je mreža postavljena na `RAW (Reflectance)`, to je sloj na kojem se nalazite.

### Otvaranje bočne trake Prikaza slike

Kliknite ikonu **Prikaz slike** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> u lijevoj bočnoj traci kako biste izvukli panel za analizu. Sadrži, odozgo prema dolje:

* naziv slike i model kamere
* gumb **Izvoz/Spremi sliku(e)** (samo kada je indeks ili LUT aktivan)
* potvrdne okvire za **Indeks**i**LUT** te panel za konfiguraciju indeksa — pogledajte [Index/LUT Sandbox](index-lut-sandbox.md)
* panel **Vrijednosti pokazivača**: očitavanje po kanalu, histogram sloja i kontrola GSD-a***

## Kretanje i zumiranje

### Pregledavanje slika

* **Sljedeća slika**: gumb →, ili tipka**→** (desna strelica)
* **Prethodna slika**: gumb ←, ili tipka**←** (lijeva strelica)
* **Preskoči na određenu sliku**: vratite se na mrežu i kliknite na njezinu minijaturu

Zumiranje i pomicanje ostaju zadržani dok prelazite između slika, tako da možete prolaziti kroz skup slika i ostati na istom dijelu okvira.

### Zumiranje

Zoom se upravlja **kotačićem miša**, u koracima od 15 %, pri čemu se točka ispod pokazivača zadržava na istom mjestu. Opseg je ograničen veličinom slike i prozora: ne možete se odzoomirati više od veličine prilagođene prozoru, a gornju granicu određuje nativna rezolucija slike.

U pregledniku na cijelom zaslonu nema namjenskih tipki za zumiranje. (U mreži, **Ctrl + `+` / `−`** mijenja veličinu minijatura — to je druga kontrola.)

### Pomicanje pri uvećanju

Kliknite i držite lijevu tipku miša iznad slike i povucite. Pomicanje je ograničeno tako da se slika ne može povući s ekrana.

### Pregled piksela po piksela pri visokom uvećanju

Kada efektivno povećanje prijeđe **60×**, Chloros crta okvir za isticanje oko pojedinačnog prikazanog piksela ispod pokazivača i plutajuću vrijednost pored njega.

&quot;Učinkovito&quot; povećanje uzima u obzir veličinu GSD bloka: s veličinom bloka od 8, istaknuto područje se pojavljuje pri 7,5× povećanju, a ne pri 60×, jer je jedan prikazani piksel već 8 × 8 izvornih piksela. Smanjite li povećanje ispod praga, istaknuto područje će nestati.

### Prečaci na tipkovnici

| Tipka                             | Gdje       | Radnja                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Cijeli zaslon | Sljedeća slika                        |
| **←**                           | Cijeli zaslon | Prethodna slika                      |
| **Ctrl + R**                    | Cijeli zaslon | Resetirajte indeks/LUT sandbox         |
| **Ctrl + `+`**/**Ctrl + `=`** | Rešetka        | Veće minijature (4 px po pritisku)  |
| **Ctrl + `−`**                  | Rešetka        | Manji sličice (4 px po pritisku) |***

## Vrijednosti pokazivača

Pomaknite pokazivač preko slike i panel **Vrijednosti pokazivača** prikazat će vrijednost svakog kanala ispod njega.

{% hint style="success" %}
**To su stvarne brojke datoteke.**Platno na zaslonu je 8-bitni rastegnuti pregled i ne može ih isporučiti, stoga**Chloros** uzorkuje stvarnu datoteku proizvoda za očitavanje. Zbog toga 12-bitni raw kadar izvještava o vrijednostima iznad 255, i zašto radijanska slojica float32 izvještava o fizičkim jedinicama.
{% endhint %}

### Što stupci znače

Ploča se prilagođava sloju koji gledate:

| Sloj koji gledate              | Prikazani stupci    | Napomene                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflektancija                        | **DN**i**%** | Postotak se izračunava prema vlastitoj skali te datoteke — pogledajte dolje                                      |
| Radijansa                           | **W/m²/sr/nm**   | Fiksne fizičke vrijednosti; nema stupca DN jer DN ovdje nema smisla                           |
| Sirov / Debayered / pretpregled / JPG | **DN**           | Cijeli digitalni brojevi                                                                         |
| Izvoz 32-bitne refleksije u postotke | samo **%**       | Pohranjeni float nije DN, pa bi njegovo zaokruživanje na cjelobrojni broj ispisalo besmisleni `0` ili `1` |

Svaki je redak označen nazivom kanala za filtar vaše kamere — `Red / Green / NIR` za RGN, `Orange / Cyan / NIR` za OCN, `NIR / Green / Blue` za NGB, `Red / Green / Blue` za RGB i jedinstveno ime pojasa za RE, NIR i mono M3M kamere. Svaka oznaka sadrži obojeni točak koji odgovara krugovima kanala korištenima u uređivaču formule indeksa.

Sačuvani **indeks i LUT** slike su poseban slučaj: one sadrže komponente bojne mape, a ne spektralne pojaseve, stoga su njihovi redovi označeni kao `Red / Green / Blue` (ili `Index` za datoteku indeksa s jednim kanalom), a ne nazivima filtara kamere.

Kada je indeks aktivan u pješčaniku, ispod kanala se pojavljuje dodatni redak koji prikazuje **vrijednost indeksa** na kursoru, s imenom indeksa i bijelim točkicama koje odgovaraju njegovim oznakama na histogramu.

### Postotak reflektancije koristi vlastitu skalu svake datoteke

{% hint style="warning" %}
**Ne pretpostavljajte da je 65535 = 100 %.** Chloros pohranjuje reflektanciju u različitim skalama ovisno o tome koja je kamera to proizvela, a preglednik određuje ispravnu skalu za svaku datoteku.
{% endhint %}

| Izvor                  | DN koji odgovara reflektanciji 1.0 | Kako se identificira                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP oznaka `Chloros:PixelScale=32768` upisana u svaki LATTICE izvoz reflektancije. Margina od 2× omogućuje datoteci da nosi ρ iznad 1,0 bez obrezivanja |
| **Survey3**|**65535**                      | Nema XMP oznake skale Chloros — kalibracija Survey3 zapisuje ρ × dtype-max i obrezuje na 1.0                                                               |

Preglednik, sandbox indeksa/LUT-a i izvoz indeksa sve rješavaju mjerilo putem iste jedinstvene implementacije, tako da je vrijednost koju pročitate na pokazivaču ista vrijednost koju je koristila matematika indeksa.

Dvije posljedice vrijedne spomena:

* **32-bitni postotak**TIFF pohranjuje DN/65535 kao decimalni broj, a**8-bitni** PNG/JPG izvoz pohranjuje DN × 255/65535 — preglednik oba pretvara natrag prije ispisivanja postotka.
* Jedan slučaj nije moguće oporaviti: **8-bitni izvoz TIFF zapisa snimljenog iz 8-bitnog izvora** ograničen je na raspon 0–255 umjesto da se ponovno skalira, i namjerno ne sadrži oznaku skale. Za te datoteke, panel ispisuje samo DN, bez stupca za postotak. To je iskren odgovor, a ne bug.***

## Histogram sloja

Ispod redaka pokazivača nalazi se histogram uživo sloja koji gledate, u **256 kutija**. Po zadanome se crta jedna kombinirana krivulja, `(R + 2G + B) / 4` — isti prostor mjerenja koji koriste histogrami kamere LATTICE. Uključivanjem**RGBa** on se zamjenjuje krivuljama za svaki kanal u bojama kanala, aditivno pomiješanim kako bi preklapanja ostala čitljiva. Mono slojevi uvijek crtaju jednu krivulju.

Hořizontalna os je u vlastitoj jedinici sloja:

| Sloj       | Jedinica osi  | Maksimum osi                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflektancija | postotak   | 125% — slobodni prostor proizvoda dopušta ρ iznad 1.0           |
| Zračenje    | W/m²/sr/nm | Vrhunac okvira, zaokružen na dvije značajne znamenke |
| 8-bitni podaci | DN         | 255                                                        |
| 12-bitni podaci | DN         | 4095                                                       |
| 16-bitni podaci | DN         | 65535                                                      |

Kada je os u DN i dosegne jednu od te tri gornje granice, Chloros također poznaje dubinu bita onoga što gledate.

Iznad histograma nalaze se tri gumba:

| Gumb     | Zadano | Učinak                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURSO** | Uključi | Crta linije markera na histogramu na točnim vrijednostima prikazanim u gornjim redovima, tako da možete vidjeti gdje se piksel ispod vašeg kursora nalazi u raspodjeli kadra. U načinu rada RGB postoji jedan marker po kanalu u vlastitoj boji; inače jedan bijeli marker na kombiniranoj vrijednosti |
| **INDEX**| Uključen | Pojavljuje se samo dok je indeks aktivan. Prelazi histogram sa izvorišnih pojaseva na**raspodjelu indeksnih vrijednosti**, pri čemu su dva praga za isrez iscrtana kao narančaste isprekidane linije, a indeksna vrijednost pokazivača kao bijela linija                                                          |
| **RGB**| Isključeno | Prelazi s kombinirane krivulje na krivulje po pojedinačnim kanalima. Na mono senzoru ovo gumb piše**MONO** i onemogućen je — postoji samo jedan kanal za prikaz.

Histogram se izračunava na **blokovima koje vidite**, a ne na izvornih pikselima iza njih: promijenite veličinu GSD bloka i distribucija se ponovno izračunava, tako da se histogram, pokazivač i prikazana slika uvijek podudaraju.***

## Veličina GSD bloka

Na dnu ploče nalazi se kontrola **GSD (px)**: brojčanik, klizač od**1 do 256**i gumb**RESET**.

Time se _prikazana_ slika zrnati prosječanjem N × N blokova izvornih piksela u jedan prikazani piksel. `1` je nativna rezolucija.

* Utječe na **prikaz cijelog zaslona, mrežne minijature, prikaz kursora i oba histograma** — sve što prikazuje sliku temelji se na istoj osnovnoj rezoluciji.
* To je **samo za prikaz**. Obrada i izvoz ostaju nepromijenjeni. Jedini iznimak je namjeran: an [Index/LUT Sandbox](index-lut-sandbox.md) izvoz sprema ono što gledate, pa prenosi trenutnu veličinu bloka, a izvozni panel upozorava vas kada je veličina bloka veća od 1.
* Vrijednost se pohranjuje **po projektu** kao `viewer_display.gsd_bin` u `project.json`, stoga se zadržava nakon zatvaranja i ponovnog otvaranja.
* Prikaz koordinata kursora izvještava o bloku, a ne o izvornoj pikseli, kad god je veličina bloka veća od 1 — prikazana vrijednost je prosjek bloka ispod vašeg kursora.

{% hint style="info" %}
**Zašto &quot;veličina bloka&quot;, a ne centimetri po pikselu?** Vrijednost cm/px zahtijeva visinu iznad tla. EXIF podatke jedne fotografije sadrže GPS nadmorsku visinu iznad razine mora, a ne nadmorsku visinu terena prema kojem je usmjerena, stoga Chloros neće ispisati udaljenost do tla koju iskreno ne može izračunati. Veličina bloka u izvornih piksela jednaka je zadanom rješenju koje koriste oblačni alati MAPIR kada je udaljenost uzorkovanja tla nepoznata.
{% endhint %}

***

## Vrste slika koje možete pregledavati

Padajući izbornik slojeva u gornjem desnom kutu preglednika navodi svaku verziju trenutne slike. Koji se unosi prikazuju ovisi o kameri i o tome što je obrađeno — pogledajte [Slojevi slike](image-layers.md) za potpuni popis i kako se padajući izbornik ponaša.

### Survey3

* **JPG** — datoteka s pregledom iz fotoaparata
* **RAW (Original)** — izvorni `.RAW`, de-bayeriziran za prikaz, bez ispravaka
* **RAW (Cilj)** — kadar identificiran kao onaj koji sadrži kalibracijski cilj
* **RAW (Reflectance)** — kalibrirani proizvod reflektancije (65535 = ρ 1.0)
* **Ispravljeno vignetiranje**/**Odziv senzora** — nekalibrirani proizvod kao rezervna opcija
* **Balansirano po bijeloj** — izlazni proizvod balansiran po bijeloj
* **RAW (indeks `<INDEX>`)**i**LUT `<INDEX>`** — izračunate indeksne slike

### REŠETKA

Kod snimanja rešetke koristi se ista padajuća lista, s nazivima razina procesa:

| Sloj                 | Što sadrži                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (originalno)**    | Izvorni sirovi kadar snimljen                                     |
| **RAW (dekonvoluirano)**   | Linearna dekonvoluirana slika                                    |
| **RAW (pregled)**     | Pregled prikaza — istezanje lažnih boja za multispektralne kamere |
| **Balančirano po bijeloj**    | Pregled prikaza za kamere RGB (balančiranje po bijeloj + gama)   |
| **RAW (Zračenje)**    | Spektralno zračenje Float32 u W/m²/sr/nm                              |
| **RAW (Odbijanje)**    | Odbijanje uint16, 32768 = ρ 1.0                                    |

Radijansa i reflektancija su dostupni samo za višespektralne snimke: glavna kamera RGB nema radiometriju po pojasu, stoga se ti slojevi za nju ne izrađuju.

***

## Primjena indeksa i LUT-a

Primijenite multispektralne indekse i tablice boja (LUT) s bočne trake:

1. Otvorite bočnu traku **Preglednika slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Označite **Indeks**

3. Odaberite filter svoje kamere i formulu indeksa, zatim povucite krugove kanala na prazna mjesta u formuli
4. Dodajte LUT i odaberite gradijent, prage i način obrezivanja
5. Očitajte vrijednosti na pokazivaču i spremite rezultat pomoću opcije **Izvoz/Spremljanje slike(a)**Pogledajte [Index/LUT Sandbox](index-lut-sandbox.md) za potpuni vodič.***

## Rješavanje problema

### Slika se ne otvara

**Mogući uzroci**: datoteka je premještena ili izbrisana nakon uvoza; proizvod nikada nije napisan; nema dovoljno memorije za vrlo veliku sliku.**Što učiniti**:

1. Provjerite postoji li datoteka sloja još uvijek u izlaznom stablu projekta
2. Otvorite datoteku u vanjskom pregledniku kako biste potvrdili da je netaknuta
3. Zatvorite druge aplikacije kako biste oslobodili memoriju

### Slika je crna, bijela ili neobično obojena

**Mogući uzroci**: istezanje prikaza nema s čime raditi (gotovo nepromjenjiva slika); sloj float32 s neuobičajenim vrijednostima; indeks koji nije proizveo valjane podatke.**Što učiniti**:

1. Očitajte vrijednosti pokazivača — ako je svaki kanal na nuli ili blizu nule, problem je u podacima, a ne u prikazu
2. Provjerite histogram: pojedinačni vrh na jednom kraju govori vam da je kadar obrezan ili prazan
3. Provjerite zapisnik obrade za pokretanje koje je proizvelo sloj

### Vrijednosti izgledaju pogrešno

**Mogući uzroci**: nalazite se na drugom sloju nego što mislite; uspoređujete postotak s neobrađenim DN-om (raw DN); uspoređujete LATTICE datoteku s Survey3 datotekom koristeći isti dijelitelj.**Što učiniti**:

1. Potvrdite odabranu sloj u padajućem izborniku — jedinice na ploči prate sloj
2. Za reflektanciju upotrijebite stupac **%** umjesto da sami dijelite DN; ako morate dijeliti, upotrijebite `Chloros:PixelScale` datoteku (32768 za LATTICE, a u slučaju izostanka to znači 65535 za Survey3)
3. Vratite veličinu GSD bloka na 1 — iznad 1 čitate prosjek bloka, a ne piksela
4. Provjerite je li kalibracija reflektancije zaista izvršena za taj kadar; nekalibrirani alternativni proizvod (Sensor Response / Vignette Corrected) nije reflektancija

***

## Sljedeći koraci

* [**Slojevi slike**](image-layers.md) — naziv svakog sloja, kada postoji, i što njegove vrijednosti znače
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — izrada, podešavanje i izvoz vizualizacija indeksa
* [**Oznake na karti**](map-markers.md) — isti skup slika na karti
* [**Multispektralne formule indeksa**](../project-settings/multispectral-index-formulas.md) — referenca za indeks

Za tijek obrade pogledajte [Obrada slika (GUI)](../processing-images-gui/adding-files-to-a-project.md).
