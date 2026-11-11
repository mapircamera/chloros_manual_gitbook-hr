# Dodavanje datoteka u projekt

Nakon što ste stvorili ili otvorili projekt u programu Chloros, sljedeći je korak dodati vaše multispektralne slike kako biste započeli s obradom. kartica **File Browser**<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> olakšava uvoz slika i upravljanje vašim skupom podataka.

## Pristupanje upravitelju datoteka

1. Otvorite ili stvorite projekt u programu Chloros
2. Kliknite ikonu **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> u lijevoj bočnoj traci
3. Panel **File Browser** prikazat će popis datoteka vašeg projekta

{% hint style="info" %}
**Podržani formati datoteka**:

* **Survey3W / Survey3N**: parovi RAW+JPG i JPG slike (preporučuju se RAW+JPG)
* **LATTICE**: snimke `.tif` / `.tiff` — zapisane pomoću kontrole kamere Chloros ili LATTICE čvorišta
* **Podaci sa senzora svjetla**: snimke `.daq` (DAQ-U/M/E) i DAQ-M `.csv` dnevnici padajućeg svjetla — uvoze se zajedno sa slikama za kalibraciju reflektancije
{% endhint %}

***

## Dodavanje slika u vaš projekt

Postoje dva glavna načina za dodavanje slika u vaš projekt:

### Metoda 1: Dodaj datoteke

Koristite ovu opciju za uvoz pojedinačnih datoteka sa slikama ili malog izbora datoteka.

1. Kliknite gumb **&quot;Dodaj datoteke&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na vrhu panela Preglednika datoteka
2. Idite do mape koja sadrži vaše slike
3. Odaberite jednu ili više slika (držite **Ctrl** za odabir više datoteka)
4. Kliknite **&quot;Otvori&quot;** za uvoz odabranih datoteka

### Metoda 2: Dodaj mapu

Koristite ovu opciju za uvoz svih slika iz mape odjednom. U jednom dijaloškom okviru možete odabrati **više mapa**.

1. Kliknite gumb **&quot;Dodaj mapu&quot;** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> na vrhu panela Preglednika datoteka
2. Pronađite i odaberite mapu (ili mape) koja(e) sadrže slike s vaše sesije snimanja
3. Kliknite **&quot;Odaberi mapu&quot;** za uvoz svih podržanih slika

{% hint style="info" %}
**Prijavljuju se datoteke koje se ne mogu učitati.** Ako mapa sadrži datoteke koje Chloros prepoznaje, ali ih ne može učitati, pojavit će se upozorenje — slike se ne gube tiho s mreže.
{% endhint %}

***

## Uvoz LATTICE mapa snimanja

LATTICE snimke spremaju se s **jednom podmapom po razini izvoza** — na primjer `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — s odgovarajućom datotekom downwellinga `.daq` u korijenskom direktoriju:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Dodaj mapu na korijensku mapu snimki** (`output/` gore). Kada odabrana mapa sama po sebi ne sadrži slike, već ima podmapu, Chloros automatski ulazi u njih — svi nivoi podmapa i korijenska mapa `.daq` preuzimaju se odjednom.**Kako se uvoze snimke:*** Svaka se snimka uvozi kao **jedna slika**, grupirana po snimci (a ne kao jedna stavka po razini). Ostale razine iste snimke pojavljuju se kao načini prikaza te jedne slike.
* **Obrada uvijek započinje s sirovim kadrom.** Ostale razine su vidljive, ali samo `raw` ikada prolazi kroz pipeline — ponovna obrada već obrađenog proizvoda dovela bi do dvostrukog primjenjivanja korekcija, stoga Chloros to odbacuje. Ponovno uvezeni izvoz nikada ne može zauzeti raw slot snimke.
* Mapu snimke spremljenu **bez** raw uvoza može se normalno prikazati, ali obrada je preskače i o tome obavještava u zapisniku. (CLI-označitelj `--input-level` može prisiliti ulaznu točku za ovaj slučaj — pogledajte [CLI Referencu](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* Od verzije 1.3.0, mapa snimki također može sadržavati mapu `clamp_masks/` (maske po pikselu za razine zračenja i odbijanja — pogledajte [Formate izlaznih slika](../output-image-formats.md#clamp-masks)). Maske nisu slike scene i **Chloros** ih preskače pri uvozu gdje god se nalazile.**Sesije LATTICE huba** uvoze se na isti način: kliknite &quot;Dodaj mapu&quot; na mapu sesije kopiranu iz huba (sadrži `raw/` i `previews/`), zajedno sa svim DAQ-M `.csv` zapisima o padanju. Ako kalibracija kamere ili DAQ-a još nije pohranjena u predmemoriju na vašem računalu, Chloros je automatski preuzima prema serijskom broju prilikom uvoza (jednom je potreban internet).***

## Razumijevanje tablice preglednika datoteka

Nakon uvoza slika, one se pojavljuju u tablici sa sljedećim stupcima:

### Naziv datoteke

* Izvorni naziv datoteke s kamere
* Održava konvenciju imenovanja kamere (npr. IMG\_0001.RAW ili capture\_20260816\_101500\_SN213800234\_raw.tif)

### Vremenski žig

* Datum i vrijeme snimanja slike
* Izvučeno iz EXIF metapodataka slike
* Koristi se za usklađivanje senzora svjetlosti, PPK sinkronizaciju i zakazivanje cilja za kalibraciju

### Model kamere

* Automatski otkrivena konfiguracija kamere i filtera
* Primjeri Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Primjeri LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Koristi se za primjenu ispravnih profila obrade

### Kolona s ciljem (kvačica)

* Označite ovu kvačicu za slike koje sadrže ciljeve kalibracije
* Kada je označena barem jedna slika, **samo označene slike se skeniraju** za ciljeve
* Za detalje pogledajte [Odabir ciljnih slika](choosing-target-images.md)

### Prikaz metapodataka slike

Klikom na gumb za prebacivanje u gornjem desnom kutu iznad tablice prikazuju se metapodaci odabrane slike u području mreže slika.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Datoteke senzora svjetla u vašem projektu

* Datoteke `.daq` i `.csv` pojavljuju se na popisu preglednika datoteka, ali nisu slike na koje se može kliknuti — one isporučuju zračenje odozgo za kalibraciju reflektancije.
* Svaka uvožena datoteka `.daq`/`.csv` navedena je u odjeljku **Postavke projekta → DAQ senzor svjetla**, gdje možete pregledati korekciju difuznog poklopca koja je na snazi za svaku datoteku. Pogledajte [Podešavanje postavki projekta](adjusting-project-settings.md).
* Snimke koje napravite na kartici **Senzori svjetla** automatski se dodaju u otvoreni projekt — nije potreban ručni uvoz.***

## Upravljanje datotekama u vašem projektu

### Kako ukloniti datoteke

Da biste uklonili neželjene slike iz projekta:

1. Odaberite jednu ili više slika u tablici preglednika datoteka
2. Kliknite gumb **&quot;Ukloni odabrano&quot;** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Potvrdite uklanjanje (datoteke se ne brišu s diska, već se samo uklanjaju iz projekta)

### Sortiranje i filtriranje

* **Sortiranje po stupcu**: Kliknite na naslov bilo kojeg stupca za sortiranje slika
* **Sortiranje po vremenu snimanja**: Korisno za organiziranje kronoloških sekvenci snimanja
* **Filter po modelu kamere**: Grupirajte slike prema vrsti kamere ako koristite više kamera***

## Pregled slike

### Prikaz cijele slike

Kliknite na bilo koju minijaturu slike u pregledniku datoteka kako biste je prikazali u glavnom području za pregled:

1. Slika se pojavljuje u središnjem panelu za pregled
2. Koristite kontrole zumiranja za pregled detalja slike
3. Kretanje između slika pomoću tipki sa strelicama

### Brza navigacija

* **Prethodna slika**: Kliknite strelicu ulijevo ili pritisnite tipku ←
* **Sljedeća slika**: Kliknite strelicu udesno ili pritisnite tipku →
* **Povećanje/Smanjenje**: Koristite kotačić miša ili gumbe za zumiranje
* **Pomicanje**: Kliknite i povucite po slici kada je uvećana***

## Rukovanje dupliciranim datotekama

Chloros automatski otkriva i zanemaruje duplicirane datoteke:

* Datoteke s identičnim nazivima se preskaču
* Sprječava slučajnu dvostruku obradu
* Prikazuje se upozorenje kada se otkriju duplicirane datoteke

{% hint style="warning" %}
**Važno**: Ne preimenujte niti mijenjajte svoje izvornike datoteka s slikama prije uvoza. Chloros se oslanja na izvorne nazive datoteka i metapodatke za pravilnu obradu.
{% endhint %}

***

## Mješoviti skupovi podataka s kamerama

Ako vaš projekt sadrži slike s više kamera MAPIR:

1. Chloros automatski otkriva svaki model kamere — Survey3, LATTICE ili mješavinu
2. Svaki se tip kamere obrađuje sa svojim odgovarajućim profilom kalibracije
3. Preglednik datoteka prikazuje model kamere u stupcu Model kamere
4. Svaka kamera dobiva vlastito stablo izlaznih mapa pri obradi

**Primjeri scenarija**: postavljanje dvostruke kamere Survey3W RGN + Survey3N OCN, ili LATTICE niz s RGB glavnom kamerom i nekoliko uskopojasnih modula***

## Najbolje prakse

### Organizirajte prije uvoza

* Pohranite slike kalibracijskih ciljeva u istu mapu kao i slike snimanja
* Sačuvajte datoteke senzora svjetla `.daq` / `.csv` svake sesije snimanja zajedno sa slikama te sesije
* Održavajte izvornu strukturu mapa s vaše kamere/SD kartice/huba
* Nemojte miješati skupove podataka iz različitih sesija u jednom projektu

### Imenovanje datoteka

* Sačuvajte izvorne nazive datoteka kamere (IMG\_0001.RAW, capture\_..., itd.)
* Nemojte preimenovati datoteke prije uvoza
* Izvorni nazivi sadrže važne metapodatke

### Slike kalibracijskih ciljeva

* Uvijek uključite 1-2 slike kalibracijskih ciljeva po sesiji (Survey3; za LATTICE snimka DAQ-a može poslužiti kao zamjena — pogledajte [Odabir slika ciljeva](choosing-target-images.md))
* Snimite ciljeve prije i nakon sesije snimanja
* Postavite ciljeve u istim uvjetima osvjetljenja kao i područje snimanja
* Označite slike ciljeva pomoću potvrdnog okvira Cilj

***

## Uobičajeni problemi i rješenja

### Slike se ne prikazuju nakon uvoza

**Mogući uzroci:**

* Format datoteke nije podržan (pogledajte popis podržanih tipova na vrhu ove stranice)
* Slike su s kamera koje nisu MAPIR (pogledajte [Podržane kamere](../supported-cameras.md))
* Oštećenje datoteke ili nepotpuni prijenos sa SD kartice

**Rješenje**: Provjerite kompatibilnost formata datoteke i modela kamere te provjerite upozorenje o učitavanju datoteka za točne datoteke koje su neuspjele.

### Model kamere nije detektiran

**Mogući uzroci:**

* Izmijenjeni EXIF metapodaci
* Slike uređene u vanjskom softveru
* Nedovršen prijenos datoteka

**Rješenje**: Ponovno uvezite izvorne, neizmijenjene datoteke s kamere/SD kartice

### Nedostajuće vremenske oznake

**Mogući uzroci:**

* Sat kamere nije ispravno postavljen
* Podaci EXIF su uklonjeni pomoću vanjskog softvera

**Rješenje**: Provjerite jesu li postavke vremena na fotoaparatu bile ispravne tijekom snimanja

### Izvješće o nedostajućim datotekama pri ponovnom otvaranju projekta

Ako su izvorne datoteke premještene ili izbrisane otkako je projekt zadnji put otvoren, Chloros će vam reći **koje**datoteke nedostaju umjesto da se otvori na praznoj mreži. Ako je sama mapa projekta premještena, Chloros ponovno usmjerava svoje izvorišne putanje na mapu u kojoj se `project.json` sada nalazi i čuva sigurnosnu kopiju (`project.json.before-relink-<date>-<time>`). Kada ih ne pronađe, prozor**Izvorne datoteke nisu pronađene**nudi**Pregledaj** za novu lokaciju, a zatim Novo projekt ili Izbriši projekt. Iz CLIa upotrijebite `chloros-cli project open PATH --relink FOLDER`; iz SDKa, `open_project(path, relink=...)`.***

## Sljedeći koraci

Nakon što su vaše datoteke uvezene:

1. **Provjerite popis datoteka** - Provjerite jesu li sve slike ispravno učitane
2. **Provjerite modele kamera** - Potvrdite ispravno otkrivanje kamere
3. **Označite ciljne slike** - Pogledajte [Odabir ciljnih slika](choosing-target-images.md)
4. **Podesite postavke** - Konfigurirajte opcije obrade u [Postavke projekta](adjusting-project-settings.md)
5. **Pokrenite obradu** - Pogledajte [Pokretanje obrade](starting-the-processing.md)

Za detaljne informacije o konfiguraciji projekta pogledajte [Podešavanje postavki projekta](adjusting-project-settings.md).
