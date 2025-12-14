# Dodajte datoteke u projekt

Nakon što ste izradili ili otvorili projekt u Chlorosu, sljedeći korak je dodavanje vaših multispektralnih slika za početak obrade. Preglednik datoteka<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> olakšava uvoz slika i upravljanje vašim skupom podataka.

## Pristup pregledniku datoteka

1. Otvorite ili kreirajte projekt u Chlorosu
2. Kliknite ikonu **Preglednik datoteka**<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
3. Ploča File Explorer prikazat će popis datoteka u vašem projektu

{% hint style=&quot;info&quot; %}**Podržane vrste datoteka**: Chloros podržava RAW+JPG i JPG slikovne datoteke sa Survey3W i Survey3N MAPIR kamera. Preporučuju se samo RAW+JPG datoteke.
{% endhint %}***## Dodajte slike svom projektu

Postoje dva glavna načina za dodavanje slika vašem projektu:

### Metoda 1: Dodajte datoteke

Koristite ovu opciju za uvoz pojedinačnih slikovnih datoteka ili malog odabira datoteka.

1. Kliknite gumb **"Dodaj datoteke"**na vrhu ploče preglednika datoteka.
2. Dođite do mape koja sadrži vaše slike.
3. Odaberite jednu ili više slikovnih datoteka (držite**Ctrl**za odabir više datoteka).
4. Kliknite**«Otvori»**za uvoz odabranih datoteka.

### Metoda 2: Dodajte mapu

Koristite ovu opciju za uvoz svih slika u mapi odjednom.

1. Pritisnite gumb**"Dodaj mapu"**na vrhu ploče preglednika datoteka.
2. Dođite do mape koja sadrži slike iz vaše sesije snimanja i odaberite je.
3. Kliknite**«Odaberi mapu»**za uvoz svih podržanih slika iz te mape.***

## File Explorer Tablica Opis

Kada se slike uvezu, pojavljuju se u tablici sa sljedećim stupcima:

### Sličica

* Mali pregled svake slike.
* Kliknite na sličicu kako biste vidjeli cijelu sliku u glavnom području pregleda.

### Naziv datoteke

* Izvorni naziv datoteke kamere.
* Održava konvenciju imenovanja fotoaparata (npr. IMG\_0001.RAW).

### Vremenska oznaka

* Datum i vrijeme snimanja slike.
* Izdvojeno iz EXIF ​​​​metapodataka slike.
* Koristi se za PPK sinkronizaciju i otkrivanje cilja kalibracije

### Model kamere

* Postavke kamere i filtra automatski se otkrivaju
* Primjeri: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Koristi se za primjenu ispravnih profila obrade

### Stupac cilja (potvrdni okvir)

* Označite ovaj okvir za slike koje sadrže kalibracijske ciljeve
* Uvelike ubrzava otkrivanje cilja tijekom obrade
* Pogledajte [Odabir ciljanih slika](choosing-target-images.md) za detalje

***## Upravljanje datotekama u vašem projektu

### Brisanje datoteka

Za uklanjanje neželjenih slika iz vašeg projekta:

1. Odaberite jednu ili više slika u tablici preglednika datoteka
2. Pritisnite gumb**«Izbriši odabrano»**
3. Potvrdite brisanje (datoteke se ne brišu s diska, samo se brišu iz projekta)

### Sortiraj i filtriraj

* **Razvrstaj po stupcu** – Kliknite zaglavlje bilo kojeg stupca za sortiranje slika
* **Poredaj po vremenskoj oznaci**: Korisno za organiziranje kronoloških sekvenci snimanja.
* **Filtar modela kamere**: Grupirajte slike prema vrsti kamere ako koristite više kamera.***

## Pregled slike

### Pogledajte cijelu sliku

Kliknite bilo koju sličicu slike u pregledniku datoteka kako biste je prikazali u glavnom području pregleda:

1. Slika se pojavljuje u središnjoj ploči za pregled.
2. Koristite kontrole zumiranja za pregled detalja slike.
3. Krećite se između slika pomoću tipki sa strelicama.

### Brza navigacija

* **Slika iznad**: Kliknite lijevu strelicu ili pritisnite tipku ←.
* **Slika ispod**: kliknite desnu strelicu ili pritisnite tipku →.
* **Uvećanje/smanjenje**: Koristite kotačić miša ili tipke za zumiranje.
* **Panorama**: Kliknite i povucite na sliku kada je uvećana.***

## Upravljanje dupliciranim datotekama

Chloros automatski otkriva i zanemaruje duplicirane datoteke:

* Datoteke s identičnim nazivima se zanemaruju.
* Izbjegavajte slučajnu dvostruku obradu.
* Prikazuje se poruka upozorenja kada se otkriju duplikati.

{% hint style=&quot;upozorenje&quot; %}
**Važno**: Nemojte preimenovati ili mijenjati izvorne slikovne datoteke prije nego ih uvezete. Chloros se oslanja na izvorne nazive datoteka i metapodatke za pravilnu obradu.
{% endhint %}***## Mješoviti skupovi podataka kamere

Ako vaš projekt sadrži slike s više MAPIR kamera:

1. Chloros automatski detektira svaki model kamere.
2. Svaki tip kamere obrađuje se s odgovarajućim profilom kalibracije.
3. Istraživač datoteka prikazuje model kamere u stupcu Model kamere.
4. Obrada primjenjuje ispravne postavke za svaku vrstu kamere.

**Primjer scenarija**: Survey3W RGN + Survey3N OCN postavljanje dvostruke kamere.***

## Najbolji primjeri iz prakse

### Uredi prije uvoza

* Čuvajte kalibracijske slike u istoj mapi kao i slike studija.
* Zadržite izvornu strukturu mapa vašeg fotoaparata/SD kartice.
* Ne miješajte skupove podataka iz različitih sesija u istom projektu.

### Imena datoteka

* Zadržite izvorne nazive datoteka fotoaparata (IMG\_0001.RAW, itd.).
* Nemojte preimenovati datoteke prije nego ih uvezete.
* Izvorni nazivi sadrže važne metapodatke.

### Slike kalibracije

*Uvijek uključite 1-2 kalibracijske slike po sesiji.
* Hvatanje ciljeva prije i poslije sesije hvatanja.
*Postavite mete u iste svjetlosne uvjete kao i područje hvatanja.
* Provjerite kalibracijske slike s okvirom Target kako biste ubrzali obradu.

***## Uobičajeni problemi i rješenja

### Slike se ne pojavljuju nakon uvoza**Mogući uzroci:**

* Format datoteke nije podržan (samo RAW+JPG i JPG s MAPIR kamera).
* Slike su s kamera koje nisu MAPIR (pogledajte [Podržane kamere](../supported-cameras.md)).
* Oštećena datoteka ili nepotpun prijenos sa SD kartice.

**Rješenje**: Provjerite kompatibilnost formata datoteke i modela fotoaparata.

### Model kamere nije otkriven**Mogući uzroci:**

* Promijenjeni EXIF ​​​​metapodaci.
* Slike uređene u vanjskom softveru.
* Nedovršen prijenos datoteke.

**Rješenje**: Ponovno uvezite izvorne neizmijenjene datoteke s fotoaparata ili SD kartice.

### Nedostaju vremenske oznake**Mogući uzroci:**

* Sat fotoaparata nije ispravno postavljen
* EXIF podatke uklonio je vanjski softver

**Rješenje**: Provjerite je li postavka vremena kamere bila ispravna tijekom snimanja.***## Sljedeći koraci

Nakon što se datoteke uvezu:

1. **Provjerite popis datoteka**– Provjerite jesu li sve slike uspješno učitane.
2.**Provjerite modele kamera**: Provjerite je li detekcija kamere ispravna.
3.**Provjerite ciljne slike**: Pogledajte [Odabir ciljnih slika](choosing-target-images.md).
4.**Prilagodite postavke**: Postavite opcije obrade u [Postavke projekta](adjusting-project-settings.md).
5.**Pokreni obradu**: Pogledajte [Pokretanje obrade](starting-the-processing.md).

Za detalje o postavkama projekta pogledajte [Prilagođavanje postavki projekta](adjusting-project-settings.md).