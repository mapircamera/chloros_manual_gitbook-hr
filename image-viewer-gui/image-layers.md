# Slojevi slike

Padajući izbornik Slojevi slike u pregledniku slika Chloros omogućuje vam brzo prebacivanje između različitih verzija iste slike, od originalnih snimaka do obrađenih rezultata refleksije i izračunatih indeksnih slika.

## Što su slojevi slike?

U Chlorosu, **slojevi** se odnose na različite rezultate slike dostupne za jednu izvornu sliku. Kada obrađujete slike, Chloros stvara nekoliko verzija:

* **Originalne slike** (JPG i RAW datoteke s vašeg fotoaparata)
* Rezultati **refleksija kalibrirana** (ako je omogućena kalibracija refleksije)
* **Ciljne slike** (ako slika sadrži kalibracijske ciljeve)
* **Indeksne slike** (NDVI, NDRE, GNDVI, itd., ako su indeksi konfigurirani)

**Padajući izbornik za odabir slojeva** koji se nalazi u gornjem desnom kutu preglednika slika omogućuje vam trenutno prebacivanje između ovih verzija bez napuštanja preglednika.

***

## Dostupne vrste slojeva

###JPG

* Izvorna JPG pretpregledna slika s vašeg fotoaparata
* Uvijek dostupno za sve slike
* Sirovo, kako je snimljeno kamerom
* Najbrže za učitavanje i prikaz

**Kada vidjeti:**

* Brzi pregled izvorne snimke
* Provjera kompozicije i kadriranja slike
* Provjera kvalitete snimanja prije obrade

### RAW (original)

* Izvorni podaci iz RAW senzora vašeg fotoaparata.
*Bijeljeno bez naknadne obrade.
* Veća bitna dubina od JPG-a (obično 12 ili 14-bitni podaci senzora).

**Kada vidjeti:**

* Provjerite kvalitetu izvornih podataka senzora.
* Provjerite ima li problema sa senzorom ili artefaktima.
* Usporedite rezultate prije i poslije obrade.

### RAW (objektiv)

*Pojavljuje se samo za slike za koje je identificirano da sadrže kalibracijske ciljeve
* Prikazuje izvornu RAW sliku s otkrivenom metom
* Koristi se za provjeru je li otkrivanje cilja bilo uspješno

**Kada vidjeti:**

* Potvrdite da su kalibracijski ciljevi ispravno otkriveni
* Provjerite kvalitetu slike objektiva
* Riješite probleme s kalibracijom

{% hint style=&quot;info&quot; %}
**Ciljni sloj** – Ovaj se sloj pojavljuje samo u padajućem izborniku za slike koje sadrže kalibracijske ciljeve. Normalne slike neće imati ovu opciju.
{% endhint %}

### RAW (refleksija)

* Refleksija izlazne slike kalibrirana.
* Vinjeta ispravljena (ako je omogućena u renderiranju).
* Refleksija kalibrirana pomoću ciljanih podataka (ako je omogućeno)
* Višepojasni TIFF sa svim kanalima kamere
*Vrijednosti piksela predstavljaju postotak refleksije (kada se koristi način postotka)
* Spremno za rukovanje s [Index/LUT Sandbox](index-lut-sandbox.md)

**Kada gledati:**

* Provjerite kalibrirane rezultate
* Provjerite kvalitetu kalibracije
* Provjerite vrijednosti piksela kako biste osigurali znanstvenu točnost
* Usporedite s originalom kako biste vidjeli učinke kalibracije

{% hint style=&quot;uspjeh&quot; %}
**Preporučeno**: koristite sloj RAW (refleksija) kada provjeravate vrijednosti piksela za znanstvena mjerenja i analize.
{% endhint %}

### RAW (NDVI Index)... i slično

*Slika izračunatog indeksa vegetacije (NDVI u ovom primjeru)
* Naziv indeksa mijenja se ovisno o indeksu koji je konfiguriran tijekom obrade
* Primjeri: RAW (NDVI indeks), RAW (NDRE indeks), RAW (GNDVI indeks), itd.
*Slika u sivim tonovima s jednim pojasom prikazuje rezultate izračuna indeksa
* Jedan sloj se pojavljuje za svaki indeks konfiguriran u postavkama projekta.

**Moguća imena indeksa:**

* RAW (NDVI indeks)
* RAW (NDRE indeks)
* RAW (GNDVI indeks)
* RAW (OSAVI indeks)
* RAW (EVI indeks)
* RAW (SAVI indeks)
* I još mnogo toga... (pogledajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md))

**Kada pogledati:**

* Ispitajte rezultate izračuna indeksa.
* Provjerite raspone vrijednosti indeksa.
* Identificirajte područja interesa.
* Provjerite indeksne slike prije korištenja u GIS-u ili analizi.

***

## Korištenje birača slojeva

### Otvorite padajući izbornik

1. Otvorite sliku na cijelom zaslonu (kliknite bilo koju sličicu u pregledniku slika).
2. Pronađite **padajući izbornik slojeva** u gornjem desnom kutu preglednika.
3. Padajući izbornik prikazuje trenutno odabrani sloj (na primjer, “JPG”).
4. Pritisnite padajući izbornik kako biste vidjeli sve dostupne slojeve.

### Promijeni sloj

1. Pritisnite padajući izbornik slojeva da biste otvorili popis.
2. Prikazuju se svi dostupni slojevi za trenutnu sliku.
3. Pritisnite bilo koji naziv sloja da biste se prebacili na tu verziju.
4. Slika se odmah ažurira kako bi se prikazao odabrani sloj.

**Brza promjena:**

* Padajući izbornik pamti vaš posljednji odabir.
* Prilikom navigacije na sljedeću sliku, Chloros pokušava prikazati istu vrstu sloja.
*Ako taj sloj ne postoji na slici ispod, zadani je JPG.

### Dostupnost sloja

Nisu svi slojevi dostupni za sve slike:

**Uvijek dostupno:**

* ✅ JPG (sve slike su pregledane u JPG).

**Uvjetno dostupno:**

* ⚠️ RAW (original): samo ako je slika snimljena u RAW ili RAW+JPG modu.
* ⚠️ RAW (cilj) - Samo ako slika sadrži otkrivene ciljeve kalibracije.
* ⚠️ RAW (refleksija): samo nakon obrade s omogućenom kalibracijom refleksije.
* ⚠️ RAW (\[Index] Index): samo nakon obrade s konfiguriranim indeksima.

***

## Postojanost sloja

### Navigacija između slika

Kada prijeđete na drugu sliku (pomoću tipki sa strelicama ili klikom na minijature):

**Preference slojeva su sačuvane:**

* Ako je prikazano "RAW (refleksija)", sljedeća slika prikazuje "RAW (refleksija)" (ako je dostupno).
* Ako se prikaže "RAW (NDVI indeks)", sljedeća slika prikazuje "RAW (NDVI indeks)" (ako je dostupno).
* Ako isti sloj ne postoji, zadani je JPG.

**Primjer tijeka rada:**

1. Otvorite sliku 1, prebacite se na RAW (NDVI indeks).
2. Pritisnite → za pregled slike 2.
3. Slika 2 automatski prikazuje RAW (NDVI Index) sloj.
4. Nastavite pregledavati: Sve slike prikazuju NDVI sloj.
5. Vrlo učinkovit za pregled rezultata indeksa na mnogim slikama.

***

## Uobičajeni tijek rada

### Tijek rada 1: Usporedba prije/poslije

**Cilj**: Usporedite izvornu sliku s kalibriranom slikom.

1. Otvorite obrađenu sliku u pregledniku slika.
2. Odaberite **RAW (Original)** iz padajućeg izbornika.
3. Zabilježite vinjetiranje i nekalibrirane vrijednosti.
4. Prebacite se na **RAW (Reflectance)** u padajućem izborniku.
5. Usporedite: Vinjetiranje je uklonjeno i vrijednosti su kalibrirane.

### Tijek rada 2: Pregledajte indeks

**Cilj**: Brzi pregled NDVI rezultata u cijelom skupu podataka.

1. Otvorite prvu obrađenu sliku.
2. Odaberite **RAW (NDVI indeks)** iz padajućeg izbornika.
3. Koristite tipku sa strelicom → za navigaciju do sljedeće slike.
4. NDVI sloj se automatski održava.
5. Nastavite sa svim slikama, provjeravajući NDVI uzorke.
6. Prijeđite na **RAW (NDRE indeks)** za usporedbu.

### Tijek rada 3: Provjera cilja

**Cilj**: Provjerite jesu li sve ciljane slike ispravno otkrivene.

1. Dođite do ciljane slike.
2. Odaberite **RAW (Lens)** iz padajućeg izbornika.
3. Provjerite jesu li kalibracijski ciljevi jasno vidljivi i otkriveni.
4. Prijeđite na sljedeću ciljnu sliku.
5. Ponovite provjeru za sve mete.

### Tijek rada 4: Provjera vrijednosti piksela

**Cilj**: Provjerite vrijednosti refleksije kako biste osigurali znanstvenu točnost.

1. Otvorite obrađenu sliku.
2. Odaberite sloj **RAW (Reflectance)**.
3. Aktivirajte način rada **Pixel Percentage** (gumb na gornjoj desnoj alatnoj traci).
4. Pomaknite kursor preko područja vegetacije.
5. Provjerite jesu li vrijednosti piksela unutar očekivanih raspona (30-70% za NIR, 5-15% za crvenu).
6. Provjerite imaju li površine tla i vode odgovarajuće vrijednosti.

***

## Razumijevanje vrijednosti piksela po sloju

Različiti slojevi pokazuju različite raspone vrijednosti piksela:

### JPG sloj

* **Raspon**: 0-255 (8 bita)
* **Značenje**: Prikaz vrijednosti, s gama korekcijom
* **Uporaba**: Samo vizualni pregled, ne za znanstvena mjerenja

### RAW (Original)

* **Raspon**: 0-65535 (16 bita)
* **Značenje**: Neobrađeni digitalni brojevi senzora.
* **Upotreba**: Provjera performansi senzora, bez kalibracije.

### RAW (refleksija)

* **Raspon**: 0-65,535 (16-bitni TIFF) ili 0,0-1,0 (32-bitni postotak)
* **Značenje**: Kalibrirani postotak refleksije
* **Upotreba**: Znanstvena mjerenja i analize

**Za 16-bitni TIFF:** Podijelite sa 65,535 da biste dobili postotak refleksije **Za 32-bitni Postotak:** Vrijednosti izravno predstavljaju postotak (0,5 = 50% refleksije)

### RAW (indeksne slike)

* **Raspon**: varira ovisno o indeksu (obično -1,0 do +1,0 za normalizirane indekse)
* **Značenje**: rezultat izračuna indeksa
* **Primjeri**:
  * NDVI: -1 do +1 (vegetacija obično 0,4 do 0,9)
  * NDRE: -1 do +1 (otkrivanje stresa)
  * EVI: od 0 do 1 (poboljšana vegetacija)

***

## Savjeti i najbolja praksa

### Učinkovito prebacivanje slojeva

* **Poznavanje tipkovničkih prečaca**: Iako nema tipkovničkih prečaca za slojeve, navigacijske strelice (←/→) rade na svim slojevima
* **Dosljedni tijek rada** – Odaberite jedan sloj (npr. NDVI) i pregledajte cijeli skup podataka prije prebacivanja na drugi
* **Brze usporedbe** – Prebacite se između Izvornika i Refleksije da biste provjerili kvalitetu obrade

### Razmatranja izvedbe

* **JPG se učitava brže** – Koristite ovo za brzo pregledavanje mnogih slika.
* **RAW slojevi se učitavaju sporije**: veća razlučivost i dubina bita.
* **Indeksni slojevi**: brzina slična slojevima refleksije.
* **Prvo učitavanje je najsporije** - Sljedeći prikazi istog sloja pohranjuju se u predmemoriju i brži su.

### Provjera kvalitete

* **Uvijek provjerite RAW (Original)** - Provjerite kvalitetu izvornih podataka prije nego što povjerujete obrađenim rezultatima.
* **Usporedi slojeve** - Koristite mijenjanje slojeva da potvrdite da je obrada radila ispravno.
* **Provjerite raspone indeksa**: Koristite način postotnih piksela sa slojevima indeksa kako biste provjerili jesu li vrijednosti razumne.

***

## Rješavanje problema

### Sloj nije dostupan

**Problem**: Očekivani sloj se ne pojavljuje u padajućem izborniku.

**Mogući uzroci:**

*Slika nije obrađivana (dostupni samo JPG i RAW (original)).
*Kalibracija refleksije bila je onemogućena tijekom obrade.
* U postavkama projekta nije konfiguriran nikakav određeni indeks.
* Slika je samo ciljna slika (za ciljeve nisu generirani indeksi).

**Rješenja:**

1. Provjerite je li slika obrađena (provjerite izlaznu mapu za obrađene datoteke).
2. Provjerite konfiguraciju projekta kako biste potvrdili da su indeksi konfigurirani.
3. Ponovno obradite s omogućenim željenim indeksima.

### Prikazan neispravan sloj

**Problem**: slika se otvara na neočekivanom sloju.

**Uzrok**: Postavke sloja prenesene su s prethodne slike, ali taj sloj ne postoji na trenutnoj slici.

**Popravak**: Chloros se automatski vraća na JPG kada preferirani sloj nije dostupan; To je normalno ponašanje.

### Ciljevi kalibracije nisu vidljivi.

**Problem**: RAW (ciljani) sloj ne prikazuje otkrivanje cilja.

**Mogući uzroci:**

*Mete nisu otkrivene tijekom obrade.
*Slika zapravo ne sadrži ciljeve.
* Postavke detekcije cilja su prestroge.

**Rješenja:**

1. Provjerite zapisnik za otklanjanje pogrešaka za poruke "Cilj je pronađen".
2. Provjerite sadrži li slika stvarno vidljive ciljeve kalibracije.
3. Podesite postavke otkrivanja cilja u postavkama projekta.
4. Pogledajte [Odabir ciljanih slika](../processing-images-gui/choosing-target-images.md).

***

## Povezane funkcije

### Alati za preglednik slika

Kada gledate bilo koji sloj, možete koristiti:

* **Kontrole zumiranja**: Povećajte da biste pregledali detalje.
* **Panorama**: kliknite i povucite za pomicanje zumirane slike.
* **Provjera vrijednosti piksela**: Pregledajte vrijednosti na lokaciji kursora.
* **Navigacijske strelice**: Krećite se između slika zadržavajući sloj.
* **Pixel Percentage Mode**: Prebacivanje između DN i postotnog prikaza.

Pogledajte [Otvori sliku preko cijelog zaslona](opening-an-image-full-screen.md) za potpunu dokumentaciju preglednika slika.

### Indeks/LUT sandbox

Za interaktivno testiranje i vizualizaciju indeksa:

* **Izračun indeksa u stvarnom vremenu** - Isprobajte različite formule indeksa.
* **LUT Mapiranje boja**: Primijenite gradijente boja na indekse u sivim tonovima.
* **Izvoz vizualizacija**: Spremite indeksne slike u boji.

Pogledajte [Index Sandbox/LUT](index-lut-sandbox.md) za detalje.

***

## Sljedeći koraci

Sada kada znate slojeve slike:

* [**Otvori sliku preko cijelog zaslona**](opening-an-image-full-screen.md): Potpuni vodič za preglednik slika.
* [**Index Sandbox/LUT**](index-lut-sandbox.md): Interaktivni prikaz indeksa
* [**Formule multispektralnog indeksa**](../project-settings/multispectral-index-formulas.md): referenca dostupnih indeksa
* [**Završetak obrade**](../processing-images-gui/finishing-the-processing.md): Razumijevanje obrađenih rezultata