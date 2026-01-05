# Slojevi slike

Padajući izbornik Slojevi slike u pregledniku slika Chloros omogućuje vam brzo prebacivanje između različitih verzija iste slike - od izvornih snimaka do obrađenih rezultata refleksije i izračunatih indeksnih slika.

## Što su slojevi slike?

U Chloros, **slojevi** se odnose na različite izlazne slike dostupne za jednu izvornu sliku. Kada obrađujete slike, Chloros stvara više verzija:

* **Originalne slike** (JPG i RAW datoteke s vašeg fotoaparata)
* Izlazi **Kalibrirani za refleksiju** (ako je kalibracija za refleksiju bila omogućena)
* **Ciljne slike** (ako slika sadrži kalibracijske ciljeve)
* **Indeksne slike** (NDVI, NDRE, GNDVI, itd. ako su indeksi konfigurirani)**Padajući izbornik za odabir slojeva** u gornjem desnom kutu preglednika slika omogućuje vam trenutno prebacivanje između ovih verzija bez napuštanja preglednika.***

## Dostupne vrste slojeva

### JPG

* Izvorna JPG pretpregledna slika s vašeg fotoaparata
* Uvijek dostupno za sve slike
* Neobrađeno, kako je snimljeno kamerom
* Najbrži za učitavanje i prikaz

**Kada pogledati:**

* Brzi pregled izvorne snimke
* Provjera kompozicije i kadriranja slike
* Provjera kvalitete snimanja prije obrade

### RAW (Original)

* Izvorni RAW podaci senzora s vašeg fotoaparata
* Debayered bez naknadne obrade
* Veća dubina bita od JPG-a (obično 12-bitni ili 14-bitni podaci senzora)

**Kada pogledati:**

* Provjera kvalitete izvornih podataka senzora
* Provjera problema sa senzorom ili artefakata
* Usporedba rezultata obrade prije/poslije

### RAW (cilj)

* Pojavljuje se samo za slike za koje je identificirano da sadrže kalibracijske ciljeve
* Prikazuje izvornu RAW sliku s otkrivenim ciljem
* Koristi se za provjeru uspješnosti otkrivanja cilja

**Kada pogledati:**

* Potvrđujemo da su kalibracijski ciljevi ispravno otkriveni
* Provjera ciljne kvalitete slike
* Rješavanje problema s kalibracijom

{% hint style="info" %}
**Ciljni sloj**: Ovaj se sloj pojavljuje samo u padajućem izborniku za slike koje sadrže kalibracijske ciljeve. Uobičajene slike snimanja neće imati ovu opciju.
{% endhint %}

### RAW (refleksija)

* Kalibrirana izlazna slika refleksije
* Vinjeta ispravljena (ako je omogućena u obradi)
* Refleksija kalibrirana pomoću ciljanih podataka (ako je omogućeno)
* Višepojasni TIFF sa svim kanalima kamere
* Vrijednosti piksela predstavljaju postotak refleksije (kada se koristi način postotka)
* Spremno za rukovanje s [Index/LUT Sandbox] (index-lut-sandbox.md)

**Kada pogledati:**

* Provjera kalibriranih rezultata
* Provjera kvalitete kalibracije
* Provjera znanstvene točnosti vrijednosti piksela
* Usporedba s originalom kako bi se vidjeli učinci kalibracije

{% hint style="success" %}
**Preporučeno**: koristite sloj RAW (Reflectance) kada provjeravate vrijednosti piksela za znanstvena mjerenja i analize.
{% endhint %}

### RAW (NDVI Index)... i slično

* Slika izračunatog indeksa vegetacije (NDVI u ovom primjeru)
* Naziv indeksa mijenja se ovisno o tome koji je indeks konfiguriran tijekom obrade
* Primjeri: RAW (NDVI indeks), RAW (NDRE indeks), RAW (GNDVI indeks), itd.
* Jednopojasna slika u sivim tonovima koja prikazuje rezultate izračuna indeksa
* Jedan sloj se pojavljuje za svaki indeks konfiguriran u postavkama projekta

**Moguća imena indeksa:**

* RAW (NDVI indeks)
* RAW (NDRE indeks)
* RAW (GNDVI indeks)
* RAW (OSAVI indeks)
* RAW (EVI indeks)
* RAW (SAVI indeks)
* I još mnogo toga... (pogledajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md))

**Kada pogledati:**

* Ispitivanje rezultata izračuna indeksa
* Provjera raspona vrijednosti indeksa
* Identificiranje područja interesa
* Provjera indeksnih slika prije korištenja u GIS-u ili analizi

***

## Korištenje birača slojeva

### Otvaranje padajućeg izbornika

1. Otvorite sliku na cijelom zaslonu (kliknite bilo koju sličicu u pregledniku slika)
2. Pronađite **padajući sloj** u gornjem desnom kutu preglednika
3. Padajući izbornik prikazuje trenutno odabrani sloj (npr. "JPG")
4. Pritisnite padajući izbornik kako biste vidjeli sve dostupne slojeve

### Promjena slojeva

1. Pritisnite padajući izbornik slojeva da biste otvorili popis
2. Prikazuju se svi dostupni slojevi za trenutnu sliku
3. Pritisnite bilo koji naziv sloja da biste se prebacili na tu verziju
4. Slika se odmah ažurira kako bi se prikazao odabrani sloj

**Brzo prebacivanje:**

* Padajući izbornik pamti vaš posljednji odabir
* Prilikom navigacije do sljedeće slike, Chloros pokušava prikazati istu vrstu sloja
* Ako taj sloj ne postoji na sljedećoj slici, zadani je JPG

### Dostupnost sloja

Nisu svi slojevi dostupni za svaku sliku:

**Uvijek dostupno:*** ✅ JPG (svaka slika ima JPG pregled)

**Uvjetno dostupno:**

* ⚠️ RAW (Original) - Samo ako je slika snimljena u RAW ili RAW+JPG načinu
* ⚠️ RAW (cilj) - Samo ako slika sadrži otkrivene ciljeve kalibracije
* ⚠️ RAW (refleksija) - Samo nakon obrade s omogućenom kalibracijom refleksije
* ⚠️ RAW (\[Index] Index) - Samo nakon obrade s konfiguriranim indeksima

***

## Postojanost sloja

### Kretanje između slika

Kada prijeđete na drugu sliku (pomoću tipki sa strelicama ili klikom na minijature):**Preference slojeva su sačuvane:**

* Ako gledate "RAW (reflektantnost)", sljedeća slika prikazuje "RAW (reflektantnost)" (ako je dostupno)
* Ako gledate "RAW (NDVI indeks)", sljedeća slika prikazuje "RAW (NDVI indeks)" (ako je dostupno)
* Ako isti sloj ne postoji, zadana je JPG

**Primjer tijeka rada:**

1. Otvorite sliku 1, prebacite se na RAW (NDVI indeks)
2. Pritisnite → za pregled slike 2
3. Slika 2 automatski prikazuje RAW (NDVI Index) sloj
4. Nastavite s navigacijom - sve slike prikazuju sloj NDVI
5. Vrlo učinkovit za pregled rezultata indeksa na mnogim slikama

***

## Uobičajeni tijek rada

### Tijek rada 1: Usporedba prije/poslije

**Cilj**: Usporedite izvornu i kalibriranu sliku

1. Otvorite obrađenu sliku u pregledniku slika
2. Odaberite **RAW (Original)** s padajućeg izbornika
3. Zabilježite vinjetiranje i nekalibrirane vrijednosti
4. Prebacite se na **RAW (Reflectance)** s padajućeg izbornika
5. Usporedite - vinjetiranje uklonjeno, vrijednosti kalibrirane

### Tijek rada 2: Pregled indeksa

**Cilj**: Brzo pregledajte NDVI rezultate u skupu podataka

1. Otvorite prvu obrađenu sliku
2. Odaberite **RAW (NDVI indeks)** s padajućeg izbornika
3. Koristite tipku sa strelicom → za navigaciju do sljedeće slike
4. NDVI sloj ostaje automatski
5. Nastavite kroz sve slike, provjeravajući uzorke NDVI
6. Prijeđite na **RAW (NDRE indeks)** za usporedbu

### Tijek rada 3: Provjera cilja

**Cilj**: Provjerite jesu li sve ciljane slike ispravno otkrivene

1. Dođite do ciljane slike
2. Odaberite **RAW (cilj)** s padajućeg izbornika
3. Provjerite jesu li kalibracijski ciljevi jasno vidljivi i otkriveni
4. Prijeđite na sljedeću ciljnu sliku
5. Ponovite provjeru za sve ciljeve

### Tijek rada 4: Provjera vrijednosti piksela

**Cilj**: provjeriti znanstvenu točnost vrijednosti refleksije

1. Otvorite obrađenu sliku
2. Odaberite sloj **RAW (Reflectance)**

3. Omogućite način rada**Pixel Percent** (gumb u gornjoj desnoj alatnoj traci)
4. Pomaknite kursor preko vegetacijskih područja
5. Provjerite jesu li vrijednosti piksela u očekivanim rasponima (30-70% za NIR, 5-15% za Red)
6. Provjerite odgovarajuće vrijednosti u tlu i vodenim područjima

***

## Razumijevanje vrijednosti piksela prema sloju

Različiti slojevi pokazuju različite raspone vrijednosti piksela:

### JPG sloj

* **Raspon**: 0-255 (8-bitni)
* **Značenje**: Prikazane vrijednosti, gama-ispravljene
* **Uporaba**: Samo vizualni pregled, ne za znanstvena mjerenja

### RAW (Original)

* **Raspon**: 0-65535 (16-bitni)
* **Značenje**: Neobrađeni digitalni brojevi senzora
* **Uporaba**: Provjera rada senzora, nije kalibriran

### RAW (refleksija)

* **Raspon**: 0-65,535 (16-bitni TIFF) ili 0,0-1,0 (32-bitni postotak)
* **Značenje**: Kalibrirani postotak refleksije
* **Upotreba**: Znanstvena mjerenja i analize**Za 16-bitni TIFF:**Podijelite sa 65,535 da biste dobili postotak refleksije**Za 32-bitni Postotak:** Vrijednosti izravno predstavljaju postotak (0,5 = 50% refleksije)

### RAW (indeksne slike)

* **Raspon**: Varira ovisno o indeksu (obično -1,0 do +1,0 za normalizirane indekse)
* **Značenje**: Rezultat izračuna indeksa
* **Primjeri**:
  * NDVI: -1 do +1 (vegetacija obično 0,4 do 0,9)
  * NDRE: -1 do +1 (otkrivanje stresa)
  * EVI: 0 do 1 (pojačana vegetacija)

***

## Savjeti i najbolja praksa

### Učinkovito prebacivanje slojeva

* **Svijest o tipkovničkim prečacima**: Iako ne postoji tipkovnički prečac za slojeve, navigacijske strelice (←/→) rade na svim slojevima
* **Dosljedni tijek rada**: Odaberite jedan sloj (npr. NDVI) i pregledajte cijeli skup podataka prije prebacivanja na drugi
* **Brze usporedbe**: Prebacite se između Izvornika i Refleksije kako biste provjerili kvalitetu obrade

### Razmatranja izvedbe

* **JPG se učitava najbrže**: Koristite za brzu navigaciju kroz mnoge slike
* **RAW slojevi se učitavaju sporije**: veća razlučivost i dubina bitova
* **Indeksni slojevi**: Slična brzina kao slojevi refleksije
* **Prvo učitavanje je najsporije**: Sljedeći prikazi istog sloja pohranjuju se u predmemoriju i brži su

### Provjera kvalitete

* **Uvijek provjerite RAW (Original)**: Provjerite kvalitetu izvornih podataka prije nego što povjerujete obrađenim izlazima
* **Usporedite slojeve**: upotrijebite promjenu slojeva za provjeru ispravnosti obrade
* **Provjerite raspone indeksa**: koristite način rada Postotak piksela sa slojevima indeksa da provjerite jesu li vrijednosti razumne***

## Rješavanje problema

### Sloj nije dostupan

**Problem**: Očekivani sloj se ne pojavljuje u padajućem izborniku**Mogući uzroci:**

* Slika nije obrađena (dostupni su samo JPG i RAW (Original))
* Kalibracija refleksije bila je onemogućena tijekom obrade
* Određeni indeks nije konfiguriran u postavkama projekta
* Slika je samo ciljna slika (nema generiranih indeksa za ciljeve)

**Rješenja:**

1. Provjerite je li slika obrađena (provjerite izlaznu mapu za obrađene datoteke)
2. Provjerite postavke projekta kako biste potvrdili da su indeksi konfigurirani
3. Ponovno obradite s omogućenim željenim indeksima

### Prikazan je pogrešan sloj

**Problem**: slika se otvara u neočekivanom sloju**Uzrok**: Postavke sloja iz prethodne slike prenesene su naprijed, ali taj sloj ne postoji na trenutnoj slici**Rješenje**: Chloros automatski se vraća na JPG kada preferirani sloj nije dostupan - to je normalno ponašanje

### Ne mogu vidjeti kalibracijske ciljeve

**Problem**: RAW (Target) sloj ne prikazuje detekciju cilja**Mogući uzroci:**

* Ciljevi nisu otkriveni tijekom obrade
* Slika zapravo ne sadrži ciljeve
* Postavke detekcije cilja prestroge

**Rješenja:**

1. Provjerite zapisnik otklanjanja pogrešaka za poruke "Cilj je pronađen".
2. Provjerite sadrži li slika stvarno vidljive kalibracijske ciljeve
3. Podesite postavke otkrivanja cilja u postavkama projekta
4. Pogledajte [Odabir ciljnih slika](../processing-images-gui/choosing-target-images.md)

***

## Povezane značajke

### Alati za preglednik slika

Kada gledate bilo koji sloj, možete koristiti:

* **Kontrole zumiranja**: Povećajte za pregled detalja
* **Pomicanje**: kliknite i povucite za pomicanje po zumiranoj slici
* **Provjera vrijednosti piksela**: Pogledajte vrijednosti na mjestu kursora
* **Navigacijske strelice**: Krećite se između slika uz zadržavanje sloja
* **Postotni način piksela**: Prebacivanje između DN i postotnog prikaza

Pogledajte [Otvaranje slike preko cijelog zaslona](opening-an-image-full-screen.md) za potpunu dokumentaciju o pregledniku slika.

### Indeks/LUT Sandbox

Za interaktivno testiranje indeksa i vizualizaciju:

* **Izračun indeksa u stvarnom vremenu**: testirajte različite formule indeksa
* **LUT mapiranje boja**: Primijenite gradijente boja na indekse u sivim tonovima
* **Izvoz vizualizacija**: Spremite indeksne slike u boji

Za detalje pogledajte [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Sljedeći koraci

Sada kada razumijete slojeve slike:

* [**Otvaranje slike preko cijelog zaslona**](opening-an-image-full-screen.md) - Potpuni vodič za preglednik slika
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Interaktivna vizualizacija indeksa
* [**Formule multispektralnog indeksa**](../project-settings/multispectral-index-formulas.md) - Referenca dostupnih indeksa
* [**Završetak obrade**](../processing-images-gui/finishing-the-processing.md) - Razumijevanje obrađenih izlaza