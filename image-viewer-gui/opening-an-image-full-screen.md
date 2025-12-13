# Otvorite sliku na cijelom zaslonu

Chloros Image Viewer pruža namjensko sučelje preko cijelog zaslona za pregled, analizu i manipuliranje vašim multispektralnim slikama. Bilo da se radi o izvornim slikama ili obrađenim rezultatima, preglednik slika nudi moćne alate za pregled i analizu.

## Pristup pregledniku slika

### Iz preglednika datoteka

Najčešći način otvaranja slike u pregledniku slika:

1. Provjerite jeste li na kartici **File Explorer**. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite bilo koju **minijaturu slike** u rešetki slike
3. Slika se otvara u **glavnom području pregleda** (središte zaslona)
4. Slika je sada učitana i spremna za prikaz preko cijelog zaslona

### Otvori karticu preglednika slika

Nakon što se slika učita u područje pregleda:

1. Kliknite ikonu **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na lijevoj bočnoj traci.
2. Otvorit će se kartica Image Viewer, prikazujući odabranu sliku na cijelom zaslonu.
3. Napredni alati za vizualizaciju i analizu bit će dostupni na lijevoj bočnoj traci.

***

## Pregled sučelja preglednika slika

### Glavno područje prikaza

Veći dio ekrana prikazuje sliku:

* **Puna razlučivost**: Slike se prikazuju u izvornoj razlučivosti.
* **Zumiranje**: Koristite kontrole ili kotačić miša za zumiranje
* **Mogućnost pomicanja** – kliknite i povucite za pomicanje kada je zumirano
* **Održavani omjer slike**: Slike se proporcionalno mijenjaju

***

## Opcije prikaza

### Osnovna navigacija slikama

#### Pregledajte slike

Krećite se skupom slika pomoću tipkovničkih prečaca ili gumba:

* **Slika ispod**: Kliknite gumb → ili pritisnite tipku **→** (desna strelica)
* **Slika iznad**: Kliknite gumb ← ili pritisnite tipku **←** (strelica lijevo)
* **Idite na određenu sliku** - Vratite se u preglednik datoteka i kliknite željenu sličicu

#### Kontrole zumiranja

Podesite povećanje za pregled detalja slike:

**Uvećaj:**

* Pritisnite gumb **+** (plus).
* Pritisnite tipku **+** ili **=**.
* Pomičite se kotačićem miša **gore**.

**Ukloniti:**

* Pritisnite gumb **−** (minus).
* Pritisnite tipku **−** (minus).
* Pomičite se kotačićem miša **dolje**.

**Prilagodi ekranu:**

* Pritisnite gumb **↔** (Prilagodi).
* Pritisnite tipku **0** (nula).
* Dupli klik na sliku.

#### Pomicanje kada je zumirano

Kada se proširi izvan veličine zaslona:

1. Pomaknite kursor miša preko slike.
2. Kliknite i **držite lijevu tipku miša**.
3. **Povucite** za pomicanje slike.
4. Otpustite za zaustavljanje pomicanja.

**Alternativa**: Koristite tipke sa strelicama za pomicanje u malim koracima.

***

## Provjera vrijednosti piksela

### Prikaz vrijednosti piksela na pokazivaču

Dok pomičete kursor miša preko slike, vrijednosti piksela se prikazuju u stvarnom vremenu:

**Mjesto prikaza vrijednosti:**

* **Plutajući broj i crvena linija u legendi gradijenta LUT s desne strane**
* **Kada se dodatno poveća, pokretna vrijednost blizu pokazivača i istaknutog piksela**
* Prikazuje vrijednosti piksela **ispod kursora ili označeno**
* Ažuriranja kada pomaknete miš

***

## Vrste slika koje se mogu vidjeti

### Izvorne slike (pretprocesiranje)

**RAW + JPG slike s fotoaparata:**

* Prikazuje RAW podatke u pretpregledu
* Prikazuje izvorne vrijednosti bez ispravka
* Korisno za provjeru kvalitete slike prije obrade

### Kalibrirane slike refleksije

**Nakon obrade:**

* Vinjeta ispravljena.
* Kalibrirana refleksija.
* Višepojasni TIFF (crveni, zeleni, NIR, itd.).
* Znanstveni podaci spremni za analizu.

### Indeksne slike

**NDVI, NDRE, GNDVI, itd. (\_NDVI.tif datoteke):**

*Jednotračne slike u sivim tonovima
*Vrijednosti piksela predstavljaju rezultate izračuna indeksa
*Tipični raspon od -1 do +1 za normalizirane indekse
* LUT-ovi u boji mogu se primijeniti za prikaz

***

## Primjena indeksa i LUT-ova

Primijenite multispektralne indekse i tablice pretraživanja boja:

1. Pronađite **Index/LUT Sandbox** na bočnoj traci <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Odaberite vegetacijski indeks (NDVI, NDRE, itd.)
3. Odaberite multispektralnu formulu ili izradite prilagođenu (samo Chloros+)
4. Primijenite gradijent tablice pretraživanja boja (LUT) za prikaz
5. Podesite raspone vrijednosti i pragove

Pogledajte [Index/LUT Sandbox](index-lut-sandbox.md) za detaljne upute.

***

## Tipkovnički prečaci

### Navigacija

* **→** (desna strelica): sljedeća slika
* **←** (strelica lijevo): prethodna slika
* **Početna**: prva slika na popisu
* **Kraj**: Zadnja slika na popisu

###Zum

* ***** ili **=**: Povećaj
* **−**: Smanji
* **0** (nula): Prilagodi zaslonu
* **Kotačić miša**: Povećaj/smanji

### Kontrole zaslona

* **P**: Uključi/isključi način rada postotka piksela
* **L**: Promjena ploče slojeva
* **Esc**: Zatvaranje cijelog zaslona ili povratak na preglednik datoteka

### Ostali

* **Ctrl+S**: Spremi trenutnu sliku
* **F**: Način rada preko cijelog zaslona (ako je dostupan)

***

### Provjera izračuna indeksa

Provjerite jesu li indeksi ispravno izračunati:

1. Otvorite NDVI ili drugu indeksnu sliku.
2. Provjerite područja s vegetacijom:
   * **NDVI**: Treba prikazati između 0,4 i 0,9 za zdrave biljke.
   * **NDRE**: više vrijednosti za snažan rast
   * **GNDVI**: sličan NDVI, ali osjetljiv na klorofil
3. Provjerite odsutnost vegetacije:
   * **Prizemlje**: Blizu 0 ili blago negativno.
   * **Voda**: Negativne vrijednosti (od -0,5 do 0).

***

## Rješavanje problema s prikazom

### Slika se ne otvara

**Mogući uzroci:**

* Datoteka je oštećena tijekom obrade.
* Format datoteke nije podržan.
* Nedovoljno memorije za velike slike.

**Rješenja:**

1. Pokušajte je otvoriti u vanjskom pregledniku da provjerite integritet datoteke.
2. Provjerite odgovara li format datoteke očekivanoj vrsti.
3. Zatvorite ostale aplikacije kako biste oslobodili memoriju.
4. Pokušajte s manjom ili drugačijom slikom.

### Crno-bijela slika

**Mogući uzroci:**

* Raspon vrijednosti izvan kapaciteta prikaza.
* 32-bitna plutajuća slika s neobičnim vrijednostima.
* Greška u izračunu indeksa.

**Rješenja:**

1. Provjerite vrijednosti piksela: ako su sve preniske ili previsoke, prilagodite raspon prikaza.
2. Pokušajte ga otvoriti u QGIS-u ili sličnom s automatskim podešavanjem raspona.
3. Provjerite ima li pogrešaka u dnevniku otklanjanja pogrešaka obrade.

### Vrijednosti piksela pojavljuju se netočno

**Mogući uzroci:**

* Prikaz netočne slike (izvorna naspram obrađene).
* Kalibracija nije pravilno primijenjena.
*Podaci svjetlosnog senzora nisu uključeni u unos.
* Način postotka nije ispravno aktiviran.

**Rješenja:**

1. Provjerite vidite li obrađeni izlaz (provjerite nastavak naziva datoteke).
2. Provjerite status gumba postotnog načina rada.
3. Usporedite s poznatim točnim slikama iz istog skupa podataka.

***

## Sljedeći koraci

Sada kada možete vidjeti slike na cijelom ekranu:

* [**Slojevi slike**](image-layers.md): Saznajte više o višepojasnom prikazu.
* [**Index Sandbox/LUT**](index-lut-sandbox.md): Primijenite prilagođene indekse i mapiranje boja.
* [**Formule multispektralnog indeksa**](../project-settings/multispectral-index-formulas.md): Razumijevanje dostupnih indeksa.

Za radni tijek obrade pogledajte:

* [**Obrada slike (GUI)**](../processing-images-gui/adding-files-to-a-project.md): Kompletan vodič za obradu.