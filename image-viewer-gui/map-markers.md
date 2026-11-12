# Označivači na karti

Na kartici Karta iscrtavaju se vaše slike na interaktivnoj 2D karti prema njihovim GPS koordinatama. Ona vam pruža geografski pregled sesije snimanja i najbrži je način, odmah nakon uvoza, da odložite slike koje ne želite obraditi.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Pristup karti

1. Otvorite ili stvorite projekt u programu Chloros
2. Uvezite slike koje sadrže GPS metapodatke
3. Kliknite na karticu **Mapa** (<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">) u lijevoj bočnoj traci
4. Na karti se prikazuje oznaka na GPS lokaciji svake slike

{% hint style="info" %}
**Potrebne GPS koordinate**: na karti se prikazuju samo slike s GPS koordinatama u njihovim EXIF metapodacima. Slika bez koordinata i dalje je u projektu i normalno se obrađuje — jednostavno nema oznaku.
{% endhint %}

***

## Podesiti slike s kartice Mapa

Kartica **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> ima ista gumba za datoteke za dodavanje <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> i uklanjanje <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> kao i kartica [**Preglednik datoteka**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Prikazuje isti popis projektnih datoteka, s geografskim stupcima:

| Stupac        | Sadržaj                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Naziv**      | Naziv datoteke kakav je došao s kamere                             |
| **Sjeverna širina** | Dekadni stupnjevi, šest decimalnih mjesta                        |
| **Geografska dužina** | Dekadni stupnjevi, šest decimalnih mjesta                        |
| **Visina**  | Metri, s jednim decimalnim mjestom — `-` kada slika ne sadrži visinu |

{% hint style="info" %}
Kliknite na naslov bilo kojeg stupca za sortiranje po njemu; kliknite ponovno za obrnuti redoslijed.
{% endhint %}

{% hint style="warning" %}
**Visina je visina iznad razine mora, a ne visina iznad tla.** Vrijednost potječe iz EXIF oznake slike `GPSAltitude`, koja se referencira na prosječnu razinu mora. To nije visina leta iznad terena i Chloros neće iz toga izvesti udaljenost uzorkovanja tla — nad poljem na 300 m nadmorske visine, dron na 100 m AGL ovdje bilježi otprilike 400 m. Koristite stupac za uočavanje odstupanja i potvrdu dosljedne visine leta, a ne kao mjerenje AGL.
{% endhint %}

***

## Označivači na slikama

Svaka slika s GPS podacima dobiva označivač na svojim koordinatama.

### Prikaz označivača

* Označivači se nalaze na točnim koordinatama zabilježenim za svako snimanje
* Markeri koji su blizu jedan drugome mogu se vizualno preklapati pri malom uvećanju — uvećajte prikaz da biste ih razdvojili
* Odabrani i istaknuti markeri iscrtavaju se iznad ostalih

### Pregled pri prelasku mišem

* **Pređite** mišem preko bilo kojeg markera da biste prikazali minijaturu te slike s njezinim nazivom datoteke
* **Kliknite**na marker kako biste odabrali sliku i**prikvačili** skočni prozor — ostaje dok ne kliknete drugdje. Dok je skočni prozor prikvačen, preletanje mišem preko drugih markera ne zatvara ga
* Ovo je brz način za pronalaženje jednog određenog kadra u velikoj sesiji bez napuštanja karte

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Na kartici Mapa iscrtana je svaka geotagirana slika u projektu</p></figcaption></figure>

### Super-zumiranje

{% hint style="success" %}
**SUPER-ZUMIRANJE**: kada dođete do maksimalnog zumiranja koje pružatelj karte podržava, daljnje zumiranje nastavlja povećavati pločice umjesto da stane, tako da možete razdvojiti oznake koje se nalaze gotovo jedna na drugoj.
{% endhint %}

* Super-zoom se aktivira samo kada ste **na** maksimalnom zoomu pružatelja usluga za tu lokaciju i kada su se pločice u potpunosti učitale. Ispod te razine, zumiranje se ponaša normalno
* Raspon je **od 1× do 32×** na vrhu maksimalnog zooma samog pružatelja usluga
* Indikator u kutu prikazuje trenutni super-zum kao postotak, a gumb **×** pored njega vraća vas na normalni zum jednim klikom
* Odzumiranje uvijek prolazi do same karte, tako da se nikada ne možete zaglaviti u super-zumu
* Prilikom zumiranja i pomicanja dok je aktivan super-zum, rezultat se prosljeđuje karti, tako da područje koje ste pomaknuli i koje je izvan središta i dalje traži pločice umjesto da ostane prazno
* Označivači se iscrtavaju kao vektorski elementi, a ne rasterizirani, stoga ostaju oštri na svakoj razini super-zuma

***

## Pružatelji karte

{% hint style="success" %}
**Automatski odabir**: Chloros odabire uslugu karte koja nudi najbolju razinu zumiranja za lokaciju vaših slika. Možete ručno prebaciti u bilo kojem trenutku.
{% endhint %}

| Pružatelj usluge | Napomene                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Široka svjetska pokrivenost; podržava sve četiri vrste pločica                                                                                                            |
| **Esri ArcGIS**| Često zračne snimke veće razlučivosti u određenim regijama. Vrsta pločice**Terrain** nije dostupna za Esri i njezino je gumbom onemogućeno dok je Esri odabran |

***

## Vrste karte

Odaberite vrstu sloja karte pomoću gumba (s lijeva na desno):

![](<../.gitbook/assets/image (14).png>)

| Vrsta                 | Prikazuje                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Reljef**          | Sjenčanje visine s detaljima karte (ceste, oznake). Samo Google       |
| **Karta**              | Standardne karte ulica — opcija s najmanjom propusnošću              |
| **Satelitska**        | Detaljne satelitske snimke, bez oznaka — opcija s najvećom propusnošću |
| **Hibridna** (zadano) | Satelitske snimke s cestama i oznakama iscrtanima preko njih                |

Zavižak **Karta**otvara se na načinu**Hibridni**. Vaš odabir zadržava se i pri prebacivanju pružatelja usluge ako on to podržava.

***

## Kretanje po karti

* **Zoomiranje**: kotačić miša, ili gumbe za zumiranje na karti
* **Pomicanje**: kliknite i povucite
* **Pun zaslon**: kontrola za pun zaslon proširuje kartu na cijeli prozor***

## Primjene

### Pregled putanje leta

* Na prvi pogled vidite područje pokrivenosti sesije drona
* Uočite praznine gdje je propušten prolaz
* Potvrdite je li let slijedio planirani uzorak

### Pregled terenskog snimanja

* Pogledajte kako su raspoređene snimke sa zemlje
* Lokirajte okvire s kalibracijskim ciljem u odnosu na područje snimanja
* Odlučite gdje su potrebne dodatne snimke

### Kontrola kvalitete

* Pronađite slike snimljene na neočekivanom mjestu i uklonite ih prije obrade
* Sortirajte po visini kako biste uočili kadar snimljen na pogrešnoj visini ili onaj gdje je GPS pozicija bila loša
* Usporedite lokacije slika s terenskim bilješkama

***

## Rješavanje problema

### Ne pojavljuju se oznake

**Mogući uzroci**

* Slike ne sadrže GPS metapodatke
* GPS je bio isključen na kameri tijekom snimanja
* EXIF podaci su uklonjeni drugim softverom prije uvoza

**Što učiniti**: provjerite je li GPS omogućen na fotoaparatu i ponovno uvezite izvorne datoteke. Možete provjeriti ima li određena datoteka koordinate tako da je potražite u tablici datoteka na kartici Mapa — slika bez koordinata neće imati redak.

### Označivači su na pogrešnom mjestu

**Mogući uzroci**: loša satelitska pozicija u trenutku snimanja ili odstupanje GPS-a tijekom sesije.**Što učiniti**: ovo je problem u trenutku snimanja, a ne nešto što Chloros može ispraviti naknadno. Za precizniji rad koristite PPK/RTK GPS radni postupak — pogledajte postavku**Primijeni PPK korekcije** u [Postavke projekta](../project-settings/project-settings.md).

### Karta je prazna ili se pločice prestaju učitavati

Pružatelji pločica su internetske usluge. Ako pločice prestanu pristizati, provjerite mrežnu vezu računala, a zatim pokušajte promijeniti pružatelja. Ako ste bili jako zumirani, pritisnite gumb za reset **×** kako biste se vratili na normalnu razinu zumiranja i dopustili karti da ponovno zatraži pločice.

***

## Povezane stranice

* [**Mreža slika**](image-grid.md) — isti skup slika kao i za minijature
* [**Otvaranje slike na cijelom zaslonu**](opening-an-image-full-screen.md) — detaljno pregledavanje jedne slike
* [**Dodavanje datoteka u projekt**](../processing-images-gui/adding-files-to-a-project.md) — gumbe za dodavanje/uklanjanje datoteka koje ova kartica dijeli
