# Dodavanje datoteka u projekt

Nakon što ste stvorili ili otvorili projekt u Chloros, sljedeći je korak dodati vaše multispektralne slike kako biste započeli s obradom. kartica &quot;File Browser&quot; (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) olakšava uvoz slika i upravljanje vašim skupom podataka.

## Pristupanje pregledniku datoteka

1. Otvorite ili stvorite projekt u Chloros
2. Kliknite ikonu **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> u lijevoj bočnoj traci
3. Panel **File Browser** prikazat će popis datoteka vašeg projekta

{% hint style="info" %}
**Podržani formati datoteka**:

* **Survey3W / Survey3N**: parovi RAW+JPG i JPG slike (preporučuju se RAW+JPG)
* **LATTICE**: `.tif` / `.tiff` snimke — zapisane pomoću kontrole kamere Chloros ili LATTICE čvorišta
* **Podaci sa senzora svjetla**: snimke `.daq` (DAQ-U/M/E) i DAQ-M `.csv` zapisi o dolaznom svjetlu — uvoze se zajedno sa slikama za kalibraciju reflektancije
{% endhint %}

***

## Dodavanje slika u projekt

Postoje dva glavna načina za dodavanje slika u projekt:

### Metoda 1: Dodaj datoteke

Koristite ovu opciju za uvoz pojedinačnih datoteka sa slikama ili malog izbora datoteka.

1. Kliknite gumb **&quot;Dodaj datoteke&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na vrhu panela Preglednika datoteka
2. Idite do mape koja sadrži vaše slike
3. Odaberite jednu ili više datoteka sa slikama (držite **Ctrl** za odabir više datoteka)
4. Kliknite **&quot;Otvori&quot;** za uvoz odabranih datoteka

### Metoda 2: Dodaj mapu

Koristite ovu opciju za uvoz svih slika iz mape odjednom. U jednom dijaloškom okviru možete odabrati **više mapa**.

1. Kliknite gumb **&quot;Dodaj mapu&quot;** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> na vrhu panela Preglednika datoteka
2. Pronađite i odaberite mapu (ili mape) koja(e) sadrže slike s vaše sesije snimanja
3. Kliknite **&quot;Odaberi mapu&quot;** kako biste importirali sve podržane slike

{% hint style="info" %}
**Prijavljuju se datoteke koje se ne mogu učitati.** Ako mapa sadrži datoteke koje Chloros prepoznaje, ali ih ne može učitati, pojavit će se upozorenje — slike se ne gube tiho iz mreže.
{% endhint %}

***

## Uvoz mapa snimki LATTICE

Snimke LATTICE-a spremaju se s **jednom podmapom po razini izvoza** — na primjer `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — s odgovarajućom datotekom downwellinga `.daq` u korijenskom direktoriju:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Postavite mapu na korijensku mapu snimki** (Chloros gore). Kada odabrana mapa sama po sebi ne sadrži slike, već ima podmapu,  automatski ulazi u njih — svi nivoi podmapa i korijenska mapa `.daq` preuzimaju se odjednom.**Kako se uvoze snimke:*** Svaka se snimka uvozi kao **jedna slika**, grupirana po snimci (a ne kao jedna stavka po razini). Ostale razine iste snimke pojavljuju se kao načini prikaza te jedne slike.
* **Obrada uvijek započinje s sirovim okvirom.** Ostale razine su vidljive, ali samo `raw` prolazi kroz pipeline — ponovna obrada već obrađenog proizvoda dovela bi do dvostrukog primjenjivanja korekcija, stoga Chloros to odbacuje. Ponovno uvezeni izvoz nikada ne može zauzeti raw slot snimke.
* Mapu snimki spremljenu **bez** raw uvoza moguće je normalno pregledati, ali obrada je preskače i o tome obavještava u zapisniku. (Zastavica CLI, `--input-level`, može prisiliti ulaznu točku za ovaj slučaj — pogledajte [referencu za CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**Sesije LATTICE huba** uvoze se na isti način: upotrijebite &#x27;Dodaj mapu&#x27; na mapu sesije kopiranu iz huba (sadrži `raw/` i `previews/`), zajedno sa svim DAQ-M `.csv` zapisnikom o protoku. Ako kalibracija kamere ili DAQ-a još nije pohranjena u predmemoriju na vašem računalu, Chloros je automatski preuzima prema serijskom broju prilikom uvoza (jednom je potreban internet).***

## Razumijevanje tablice preglednika datoteka

Nakon uvoza slika, one se pojavljuju u tablici sa sljedećim stupcima:

### Naziv datoteke

* Izvorni naziv datoteke s kamere
* Održava konvenciju imenovanja kamere (npr. IMG\_0001.RAW ili capture\_20260816\_101500\_SN213800234\_raw.tif)

### Vrijeme snimanja

* Datum i vrijeme kada je slika snimljena
* Izvučeno iz EXIF metapodataka slike
* Koristi se za usklađivanje senzora svjetla, PPK sinkronizaciju i zakazivanje kalibracijskih ciljeva

### Model kamere

* Automatski otkrivena konfiguracija kamere i filtera
* Survey3 primjeri: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE primjeri: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Koristi se za primjenu ispravnih profila obrade

### Ciljna stupac (kvačica)

* Označite ovaj okvir za slike koje sadrže ciljne predmete
* Kada je označena barem jedna slika, **samo označene slike se skeniraju** za ciljne predmete
* Pogledajte [Odabir ciljnih slika](choosing-target-images.md) za detalje

### Prikaz metapodataka slike

Klikom na gumb za prebacivanje u gornjem desnom kutu iznad tablice prikazuju se metapodaci odabrane slike u području mreže slika.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Datoteke senzora svjetla u vašem projektu

* Datoteke `.daq` i `.csv` pojavljuju se na popisu preglednika datoteka, ali nisu slike na koje se može kliknuti — one isporučuju zračenje odozgo za kalibraciju reflektancije.
* Svaki uvoženi `.daq`/`.csv` naveden je u **Postavke projekta → DAQ senzor svjetla**, gdje možete pregledati korekciju difuznog poklopca koja je na snazi za svaku datoteku. Pogledajte [Podešavanje postavki projekta](adjusting-project-settings.md).
* Snimke koje napravite na kartici **Senzori svjetla** automatski se dodaju u otvoreni projekt — nije potreban ručni uvoz.***

## Upravljanje datotekama u vašem projektu

### Uklanjanje datoteka

Da biste uklonili neželjene slike iz projekta:

1. Odaberite jednu ili više slika u tablici preglednika datoteka
2. Kliknite gumb **&quot;Ukloni odabrano&quot;** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Potvrdite uklanjanje (datoteke se ne brišu s diska, već se samo uklanjaju iz projekta)

### Sortiranje i filtriranje

* **Sortiranje po stupcu**: Kliknite na naslov bilo kojeg stupca za sortiranje slika
* **Sortiranje po vremenu snimanja**: Korisno za organiziranje kronoloških sekvenci snimanja
* **Filtriranje po modelu fotoaparata**: Grupirajte slike prema vrsti fotoaparata ako koristite više fotoaparata***

## Pregled slike

### Prikaz cijele slike

Kliknite na bilo koju minijaturu slike u pregledniku datoteka kako biste je prikazali u glavnom području za pregled:

1. Slika se pojavljuje u središnjem panelu za pregled
2. Koristite kontrole zumiranja za pregled detalja slike
3. Kretanje između slika pomoću tipki sa strelicama

### Brza navigacija

* **Prethodna slika**: Kliknite strelicu ulijevo ili pritisnite tipku ←
* **Sljedeća slika**: Kliknite strelicu udesno ili pritisnite tipku →
* **Zumiranje**: Koristite kotačić miša ili gumbe za zumiranje
* **Pomicanje**: Kliknite i povucite po slici kada je zumirana***

## Rukovanje dupliciranim datotekama

Chloros automatski otkriva i zanemaruje duplicirane datoteke:

* Datoteke s identičnim nazivima se preskaču
* Sprječava slučajnu dvostruku obradu
* Prikazuje se upozorenje kada se otkriju duplicirane datoteke

{% hint style="warning" %}
**Važno**: Nemojte preimenjivati niti mijenjati svoje izvorne datoteke sa slikama prije uvoza. Chloros se oslanja na izvorne nazive datoteka i metapodatke za pravilnu obradu.
{% endhint %}

***

## Mješoviti skupovi podataka s kamerama

Ako vaš projekt sadrži slike s više MAPIR kamera:

1. Chloros automatski otkriva svaki model kamere — Survey3, LATTICE ili mješavinu
2. Svaki se tip kamere obrađuje sa svojim odgovarajućim profilom kalibracije
3. Preglednik datoteka prikazuje model kamere u stupcu Model kamere
4. Svaka kamera dobiva vlastito stablo izlaznih mapa prilikom obrade

**Primjeri scenarija**: postavka s dvostrukom kamerom Survey3W RGN + Survey3N OCN, ili LATTICE niz s RGB glavnom kamerom i nekoliko uskopojasnih modula***

## Najbolje prakse

### Organizirajte prije uvoza

* Pohranite slike kalibracijskog cilja u istu mapu kao i slike snimanja
* Sačuvajte datoteke senzora svjetla svake sesije snimanja (`.daq` / `.csv`) zajedno sa slikama te sesije
* Održavajte izvornu strukturu mapa s vaše kamere/SD kartice/huba
* Nemojte miješati skupove podataka iz različitih sesija u jednom projektu

### Imenovanje datoteka

* Očuvajte izvorne nazive datoteka kamere (IMG\_0001.RAW, capture\_..., itd.)
* Ne preimenujte datoteke prije uvoza
* Izvorni nazivi sadrže važne metapodatke

### Slike kalibracijskih ciljeva

* Uvijek uključite 1-2 slike kalibracijskih ciljeva po sesiji (Survey3; za LATTICE može poslužiti DAQ snimka — pogledajte [Odabir ciljnih slika](choosing-target-images.md))
* Snimite ciljeve prije i nakon sesije snimanja
* Postavite ciljeve u istim uvjetima osvjetljenja kao i područje snimanja
* Označite slike ciljeva pomoću potvrdnog okvira za cilj (Target)

***

## Uobičajeni problemi i rješenja

### Slike se ne prikazuju nakon uvoza

**Mogući uzroci:**

* Format datoteke nije podržan (pogledajte popis podržanih tipova na vrhu ove stranice)
* Slike su s kamera koje nisu MAPIR (pogledajte [Podržane kamere](../supported-cameras.md))
* Oštećenje datoteke ili nepotpuni prijenos sa SD kartice

**Rješenje**: Provjerite kompatibilnost formata datoteke i modela kamere te provjerite upozorenje o učitavanju datoteka za točne datoteke koje su neuspjele

### Model kamere nije prepoznat

**Mogući uzroci:**

* Izmijenjeni EXIF metapodaci
* Slike uređene u vanjskom softveru
* Nezavršeno prebacivanje datoteka

**Rješenje**: Ponovno uvezite izvorne, neizmijenjene datoteke s kamere/SD kartice

### Nedostaju vremenski pečati

**Mogući uzroci:**

* Sat kamere nije ispravno postavljen
* Podaci EXIF su uklonjeni pomoću vanjskog softvera

**Rješenje**: Provjerite jesu li postavke vremena na fotoaparatu bile ispravne tijekom snimanja

### Izvješće o projektu pri ponovnom otvaranju prijavljuje nedostajuće datoteke

Ako su izvorne datoteke premještene ili izbrisane otkako je projekt zadnji put otvoren, Chloros će vam reći **koje** datoteke nedostaju umjesto da se otvori na praznoj mreži. Obnovite datoteke na njihove izvorne putanje ili uklonite nedostajuće unose i ponovno uvezite.***

## Sljedeći koraci

Nakon što su vaše datoteke uvezene:

1. **Pregledajte popis datoteka** - Provjerite jesu li sve slike ispravno učitane
2. **Provjerite modele fotoaparata** - Potvrdite ispravno prepoznavanje fotoaparata
3. **Označite odredišne slike** - Pogledajte [Odabir odredišnih slika](choosing-target-images.md)
4. **Podesite postavke** - Konfigurirajte opcije obrade u [Postavke projekta](adjusting-project-settings.md)
5. **Pokrenite obradu** - Pogledajte [Pokretanje obrade](starting-the-processing.md)

Za detaljne informacije o konfiguraciji projekta pogledajte [Podešavanje postavki projekta](adjusting-project-settings.md).
