# Oznake karte

Kartica Karta prikazuje vaše slike na interaktivnoj 2D karti na temelju njihovih GPS koordinata. Ovo pruža geografski pregled vaše sesije snimanja i pomaže vam da vizualizirate prostornu pokrivenost. Također je korisno pri prvom uvozu vaših slika za brzo uklanjanje svih slika koje ne morate obraditi.

## Pristup kartici Karta

1. Otvorite ili kreirajte projekt u Chloros
2. Uvezite slike koje sadrže GPS metapodatke
3. Kliknite karticu **Karta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na lijevoj bočnoj traci
4. Karta će prikazati markere na GPS lokaciji svake slike

{% hint style="info" %}
**Potreban GPS**: Na karti će se pojaviti samo slike s ugrađenim GPS koordinatama u svojim EXIF metapodacima. Provjerite ima li fotoaparat uključen GPS tijekom snimanja.
{% završni savjet %}

***

## Podešavanje slika s kartice Karta

Kartica **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> ima isti dodatak <img src="../.gitbook/assets/image.png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> i uklonite <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> gumbe za datoteke kao [**Preglednik datoteka**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> kartica radi. Također prikazuje isti popis tablice datoteke projekta, ali s različitim zaglavljima stupaca:

### Naziv datoteke

* Izvorni naziv datoteke s fotoaparata
* Održava konvenciju imenovanja fotoaparata (npr. IMG\_0001.RAW)

### Geografska širina

* Geografska širina slike

### Zemljopisna dužina

* Zemljopisna dužina slike

### Nadmorska visina

* Nadmorska visina slike

{% hint style="info" %}
Klikom na zaglavlja stupaca tablice također se sortiraju podaci retka
{% završni savjet %}

***

## Markeri slika

Svaka slika s GPS podacima predstavljena je oznakom na karti:

### Prikaz markera

* Oznake označavaju točne GPS koordinate na kojima je svaka slika snimljena
* Grupirani markeri mogu se grupirati zajedno kada se umanje
* Povećajte kako biste vidjeli pojedinačne lokacije slika

{% hint style="uspjeh" %}
SUPER-ZUMIRANJE: Kada dosegnete maksimalnu razinu zumiranja od dobavljača pločica karte, pločica se zatim povećava nakon daljnjeg zumiranja, što vam omogućuje da vidite oznake koje su blizu jedna drugoj.
{% završni savjet %}

### Pregled lebdeći

* **Zadržite pokazivač miša** iznad bilo kojeg markera da biste vidjeli minijaturni pregled te slike
* Ovo omogućuje brzu vizualnu identifikaciju bez napuštanja prikaza karte
* Korisno za lociranje određenih slika unutar velike sesije snimanja

***

## Dobavljači pločica karte

{% hint style="uspjeh" %}
**Automatski odabir**: Chloros automatski odabire uslugu pločica koja pruža najbolju razinu zumiranja za vašu trenutnu lokaciju na karti. Po želji se možete ručno prebacivati ​​između pružatelja usluga.
{% završni savjet %}

Kartica Karta podržava dva pružatelja pločica za pozadinske slike karte:

### Google karte

* Standardne satelitske slike i karte s Googlea
* Najbolje za opću svjetsku pokrivenost

### ESRI

* Satelitske i zračne slike iz ESRI ArcGIS
* Često daje slike veće rezolucije u određenim regijama

***

## Vrste pločica karte

Možete odabrati vrstu sloja karte (s lijeva na desno):

&#x20;<img src="../.gitbook/assets/image (23).png" alt="" data-size="original">

### Teren

Prikazuje visinske profile i pločice karte s detaljima (ceste, itd.)

### Karta

Prikazuje standardne (niža propusnost) pločice karte s detaljima (ceste, itd.)

### Satelit

Prikazuje detaljne (veća propusnost) pločice satelitske karte

### Hibrid

Prikazuje pločice satelitske karte s dodanim detaljima (ceste, itd.)

***

## Navigacija kartom

### Kontrole zumiranja

* **Uvećanje/smanjenje**: Koristite kotačić miša ili gumbe za zumiranje
* **Cijeli zaslon**: Prikaz karte preko cijelog zaslona

### Pan kontrole

* **Pomicanje**: kliknite i povucite za kretanje po karti***

## Slučajevi upotrebe

### Vizualizacija putanje leta

* Pogledajte područje pokrivenosti sesija snimanja dronom
* Identificirajte nedostatke u pokrivenosti slike
* Provjerite izvršenje putanje leta

### Pregled terena

* Pogledajte prostornu distribuciju zemaljskih snimanja
* Locirajte slike cilja kalibracije u odnosu na područje istraživanja
* Planirajte dodatne lokacije za snimanje

### Kontrola kvalitete

* Brzo identificirajte slike snimljene na neočekivanim lokacijama
* Provjerite točnost GPS-a u skupu podataka
* Lokacija slika s unakrsnim referencama s bilješkama na terenu

***

## Rješavanje problema

### Nema oznaka

**Mogući uzroci:**

* Slike ne sadrže GPS metapodatke
* GPS je bio onemogućen na fotoaparatu tijekom snimanja
* EXIF podatke uklonio je vanjski softver

**Rješenje**: Provjerite je li GPS omogućen na vašoj kameri i ponovno uvezite originalne datoteke

### Markeri na pogrešnoj lokaciji

**Mogući uzroci:**

* GPS kamere imao je lošu satelitsku vezu
* GPS drift tijekom snimanja

**Rješenje**: ovo je obično problem s vremenom snimanja; razmislite o korištenju PPK/RTK GPS-a za precizne primjene