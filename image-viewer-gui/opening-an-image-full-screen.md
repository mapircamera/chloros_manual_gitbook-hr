# Otvaranje slike preko cijelog zaslona

Chloros Image Viewer pruža namjensko sučelje preko cijelog zaslona za pregled, analizu i manipuliranje vašim multispektralnim slikama. Bilo da gledate izvorne slike ili obrađene izlaze, Image Viewer nudi moćne alate za pregled i analizu.

## Pristup pregledniku slika

### Iz preglednika datoteka

Najčešći način otvaranja slike u pregledniku slika:

1. Provjerite jeste li na kartici **Preglednik datoteka** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite bilo koju **minijaturu slike** u rešetki slike
3. Slika se otvara u **glavnom području pregleda** (središte zaslona)
4. Slika je sada učitana i spremna za prikaz preko cijelog zaslona

### Otvaranje kartice Preglednik slika

Nakon što se slika učita u područje pregleda:

1. Kliknite ikonu **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
2. Otvara se kartica Image Viewer, prikazujući odabranu sliku preko cijelog zaslona
3. Napredni alati za pregled i analizu postaju dostupni na lijevoj bočnoj traci

***

## Pregled sučelja preglednika slika

### Glavno područje prikaza

Najveći dio zaslona prikazuje vašu sliku:

* **Puna razlučivost**: Slike prikazane u izvornoj razlučivosti
* **Zumiranje**: Koristite kontrole ili kotačić miša za zumiranje
* **Pannable**: kliknite i povucite da biste se kretali kada ste zumirani
* **Zadržan omjer slike**: Slike se proporcionalno mijenjaju***

## Opcije gledanja

### Osnovna navigacija slikama

#### Pregledajte slike

Krećite se kroz skup slika pomoću tipkovničkih prečaca ili gumba:

* **Sljedeća slika**: Kliknite gumb → ili pritisnite tipku**→** (strelica desno).
* **Prethodna slika**: Kliknite gumb ← ili pritisnite tipku**←** (strelica lijevo).
* **Skoči na određenu sliku**: Vratite se na preglednik datoteka i kliknite željenu sličicu

#### Kontrole zumiranja

Podesite povećanje za pregled detalja slike:

**Uvećaj:*** Pritisnite gumb **+** (plus).
* Pritisnite tipku **+**ili**=*** Pomaknite kotačić miša **gore**

**Smanji:*** Pritisnite gumb **−** (minus).
* Pritisnite tipku **−** (minus).
* Pomaknite kotačić miša **dolje**

**Prilagodi ekranu:*** Pritisnite gumb **↔** (Prilagodi).
* Pritisnite tipku **0** (nula).
* Dvaput kliknite na sliku

#### Pomicanje kada je zumirano

Kada je uvećano iznad veličine zaslona:

1. Pomaknite pokazivač miša preko slike
2. Kliknite i **držite lijevu tipku miša**

3.**Povucite** za pomicanje slike
4. Otpustite za zaustavljanje pomicanja

**Alternativa**: Koristite tipke sa strelicama za pomicanje u malim koracima***

## Provjera vrijednosti piksela

### Pregled vrijednosti piksela na kursoru

Dok pomičete kursor miša preko slike, vrijednosti piksela prikazuju se u stvarnom vremenu:**Mjesto prikaza vrijednosti:*** **Plutajući broj i crvena linija u legendi gradijenta LUT s desne strane*** **Kada se dodatno poveća, plutajuća vrijednost u blizini kursora i istaknutog piksela*** Prikazuje vrijednosti za piksel **ispod pokazivača ili označeno*** Ažuriranja dok pomičete miš

***

## Vrste slika koje možete vidjeti

### Izvorne slike (prethodna obrada)

**RAW + JPG slike s fotoaparata:**

* Prikaz RAW podataka u pretpreglednom stanju
* Prikaži originalne, nekorigirane vrijednosti
* Korisno za provjeru kvalitete slike prije obrade

### Kalibrirane slike refleksije

**Nakon obrade:**

* Vinjeta ispravljena
* Refleksija kalibrirana
* Višepojasni TIFF (Red, Green, NIR, itd.)
* Znanstveni podaci spremni za analizu

### Indeksne slike

**NDVI, NDRE, GNDVI, itd. (\_NDVI.tif datoteke):**

* Jednopojasne slike u sivim tonovima
* Vrijednosti piksela predstavljaju rezultate izračuna indeksa
* Raspon obično od -1 do +1 za normalizirane indekse
* Može primijeniti LUT-ove u boji za vizualizaciju

***

## Indeks i LUT aplikacija

Primijenite multispektralne indekse i tablice pretraživanja u boji:

1. Pronađite **Index/LUT Sandbox**u**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> bočnoj traci
2. Odaberite indeks vegetacije (NDVI, NDRE, itd.)
3. Odaberite multispektralnu formulu ili izradite vlastitu prilagođenu (samo Chloros+)
4. Primijenite LUT gradijent boje za vizualizaciju
5. Podesite raspone vrijednosti i pragove

Pogledajte [Index/LUT Sandbox](index-lut-sandbox.md) za detaljne upute.

***

## Tipkovnički prečaci

### Navigacija

* **→** (desna strelica): Sljedeća slika
* **←** (Strelica lijevo): Prethodna slika
* **Početna**: Prva slika na popisu
* **Kraj**: Zadnja slika na popisu

### Zumiraj

* **+**ili**=**: Povećaj
* **−**: Smanji
* **0** (nula): Prilagodi zaslonu
* **Kotačić miša**: Povećaj/smanji

### Pregled kontrole

* **P**: Uključivanje/isključivanje postotka piksela
* **L**: Promjena ploče slojeva
* **Esc**: Zatvaranje cijelog zaslona ili povratak na preglednik datoteka

### Ostalo

* **Ctrl+S**: Spremi trenutnu sliku
* **F**: Prikaz preko cijelog zaslona (ako je dostupan)***

### Provjera izračuna indeksa

Provjerite jesu li indeksi ispravno izračunati:

1. Otvorite NDVI ili drugu indeksnu sliku
2. Provjerite područja vegetacije:
   * **NDVI**: Trebao bi pokazati 0,4-0,9 za zdrave biljke
   * **NDRE**: Više vrijednosti za snažan rast
   * **GNDVI**: Sličan NDVI, ali osjetljiv na klorofil
3. Provjerite ne-vegetaciju:
   * **Tlo**: Blizu 0 ili blago negativno
   * **Voda**: Negativne vrijednosti (-0,5 do 0)***

## Rješavanje problema s gledanjem

### Slika se ne otvara

**Mogući uzroci:**

* Datoteka je oštećena tijekom obrade
* Nepodržani format datoteke
* Nedovoljno memorije za veliku sliku

**Rješenja:**

1. Pokušajte otvoriti u vanjskom pregledniku da provjerite integritet datoteke
2. Provjerite odgovara li format datoteke očekivanoj vrsti
3. Zatvorite ostale aplikacije kako biste oslobodili memoriju
4. Pokušajte s manjom/različitom slikom

### Prikaz crne ili bijele slike

**Mogući uzroci:**

* Raspon vrijednosti izvan mogućnosti prikaza
* 32-bitna float slika s neobičnim vrijednostima
* Pogreška izračuna indeksa

**Rješenja:**

1. Provjerite vrijednosti piksela - ako su sve vrlo niske ili vrlo visoke, prilagodite raspon prikaza
2. Pokušajte otvoriti u QGIS-u ili sličnom s automatskim podešavanjem raspona
3. Provjerite zapisnik otklanjanja pogrešaka od obrade za pogreške

### Vrijednosti piksela čine se pogrešnim

**Mogući uzroci:**

* Gledanje pogrešne slike (izvorna naspram obrađene)
* Kalibracija nije pravilno primijenjena
* Podaci svjetlosnog senzora nisu uključeni u unos
* Način postotka nije ispravno uključen

**Rješenja:**

1. Provjerite gledate li obrađeni izlaz (provjerite sufiks naziva datoteke)
2. Provjerite stanje gumba postotnog načina rada
3. Usporedite s poznatim dobrim slikama iz istog skupa podataka

***

## Sljedeći koraci

Sada kada možete gledati slike preko cijelog zaslona:

* [**Slojevi slike**](image-layers.md) - Saznajte više o višepojasnoj vizualizaciji
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Primijenite prilagođene indekse i mapiranje boja
* [**Formule multispektralnog indeksa**](../project-settings/multispectral-index-formulas.md) - Razumijevanje dostupnih indeksa

Za radni tijek obrade pogledajte:

* [**Obrada slika (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Kompletan vodič za obradu