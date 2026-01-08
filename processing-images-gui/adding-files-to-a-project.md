# Dodavanje datoteka u projekt

Nakon što ste izradili ili otvorili projekt u Chloros, sljedeći korak je dodavanje vaših multispektralnih slika za početak obrade. Kartica File Browser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> olakšava uvoz slika i upravljanje vašim skupom podataka.

## Pristup pregledniku datoteka

1. Otvorite ili kreirajte projekt u Chloros
2. Kliknite ikonu **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
3. Ploča File Browser prikazat će popis datoteka vašeg projekta

{% hint style="info" %}
**Podržane vrste datoteka**: Chloros podržava RAW+JPG i JPG slikovne datoteke s MAPIR Survey3W i Survey3N kamera. Preporučuju se samo RAW+JPG.
{% endhint %}

***

## Dodavanje slika vašem projektu

Postoje dva primarna načina za dodavanje slika vašem projektu:

### Metoda 1: Dodajte datoteke

Koristite ovu opciju za uvoz pojedinačnih slikovnih datoteka ili malog odabira datoteka.

1. Kliknite gumb **"Dodaj datoteke"** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> na vrhu ploče preglednika datoteka
2. Dođite do mape koja sadrži vaše slike
3. Odaberite jednu ili više slikovnih datoteka (držite **Ctrl** za odabir više datoteka)
4. Kliknite **"Otvori"** za uvoz odabranih datoteka

### Metoda 2: Dodajte mapu

Koristite ovu opciju za uvoz svih slika iz mape odjednom.

1. Pritisnite gumb **"Dodaj mapu"** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> na vrhu ploče preglednika datoteka
2. Dođite do i odaberite mapu koja sadrži slike vaše sesije snimanja
3. Kliknite **"Odaberi mapu"** za uvoz svih podržanih slika iz te mape

***

## Razumijevanje tablice preglednika datoteka

Kada se slike uvezu, pojavljuju se u tablici sa sljedećim stupcima:

### Naziv datoteke

* Izvorni naziv datoteke s fotoaparata
* Održava konvenciju imenovanja fotoaparata (npr. IMG\_0001.RAW)

### Vremenska oznaka

* Datum i vrijeme snimanja slike
* Izdvojeno iz EXIF metapodataka slike
* Koristi se za PPK sinkronizaciju i detekciju cilja kalibracije

### Model kamere

* Automatski detektirana konfiguracija kamere i filtra
* Primjeri: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Koristi se za primjenu ispravnih profila obrade

### Ciljni stupac (potvrdni okvir)

* Označite ovaj okvir za slike koje sadrže kalibracijske ciljeve
* Uvelike ubrzava otkrivanje cilja tijekom obrade
* Pogledajte [Odabir ciljanih slika] (choosing-target-images.md) za detalje

### Pregled metapodataka slike

Klikom na gumb za prebacivanje u gornjem desnom kutu iznad tablice prikazuju se metapodaci odabrane slike u području rešetke slike.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Upravljanje datotekama u vašem projektu

### Uklanjanje datoteka

Za uklanjanje neželjenih slika iz vašeg projekta:

1. Odaberite jednu ili više slika u tablici File Browser
2. Kliknite gumb **"Ukloni odabrano"** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
3. Potvrdite uklanjanje (datoteke se ne brišu s diska, samo uklanjaju iz projekta)

### Razvrstavanje i filtriranje

* **Razvrstaj po stupcu**: Kliknite zaglavlje bilo kojeg stupca za sortiranje slika
* **Razvrstavanje vremenskim žigom**: Korisno za organiziranje kronoloških nizova snimanja
* **Filtar modela fotoaparata**: Grupirajte slike prema vrsti fotoaparata ako koristite više fotoaparata

***

## Pregled slike

### Pregled cijele slike

Kliknite bilo koju sličicu slike u pregledniku datoteka kako biste je prikazali u glavnom području pregleda:

1. Slika se pojavljuje na središnjoj ploči za pregled
2. Koristite kontrole zumiranja za pregled detalja slike
3. Krećite se između slika pomoću tipki sa strelicama

### Brza navigacija

* **Prethodna slika**: Kliknite lijevu strelicu ili pritisnite tipku ←
* **Sljedeća slika**: Kliknite desnu strelicu ili pritisnite tipku →
* **Uvećanje/smanjenje**: Koristite kotačić miša ili gumbe za zumiranje
* **Pomicanje**: Kliknite i povucite sliku kada je uvećana

***

## Rukovanje dupliciranim datotekama

Chloros automatski otkriva i zanemaruje duplicirane datoteke:

* Datoteke s identičnim nazivima datoteka se preskaču
* Sprječava slučajnu dvostruku obradu
* Poruka upozorenja prikazuje se kada se otkriju duplikati

{% hint style="warning" %}
**Važno**: Nemojte preimenovati ili mijenjati izvorne slikovne datoteke prije uvoza. Chloros oslanja se na izvorne nazive datoteka i metapodatke za pravilnu obradu.
{% endhint %}

***

## Mješoviti skupovi podataka kamere

Ako vaš projekt sadrži slike s više MAPIR kamera:

1. Chloros automatski otkriva svaki model kamere
2. Svaki tip kamere obrađuje se s odgovarajućim profilom kalibracije
3. Preglednik datoteka prikazuje model kamere u stupcu Model kamere
4. Obrada primjenjuje ispravne postavke za svaku vrstu kamere

**Primjer scenarija**: Survey3W RGN + Survey3N OCN postavljanje dvostruke kamere

***

## Najbolji primjeri iz prakse

### Organizirajte prije uvoza

* Držite kalibracijske ciljne slike u istoj mapi u kojoj su i slike ankete
* Održavajte izvornu strukturu mapa s vašeg fotoaparata/SD kartice
* Ne miješajte skupove podataka iz različitih sesija u jednom projektu

### Imenovanje datoteke

* Sačuvajte izvorne nazive datoteka fotoaparata (IMG\_0001.RAW, itd.)
* Nemojte preimenovati datoteke prije uvoza
* Izvorni nazivi sadrže važne metapodatke

### Slike cilja kalibracije

* Uvijek uključite 1-2 ciljane slike kalibracije po sesiji
* Hvatanje ciljeva prije i poslije sesije hvatanja
* Postavite mete u iste uvjete osvjetljenja kao područje hvatanja
* Označite ciljne slike pomoću potvrdnog okvira Target kako biste ubrzali obradu

***

## Uobičajeni problemi i rješenja

### Slike se ne pojavljuju nakon uvoza

**Mogući uzroci:**

* Format datoteke nije podržan (samo RAW+JPG i JPG s fotoaparata MAPIR)
* Slike su s kamera koje nisu MAPIR (pogledajte [Podržane kamere](../supported-cameras.md))
* Oštećenje datoteke ili nepotpun prijenos sa SD kartice

**Rješenje**: Provjerite kompatibilnost formata datoteke i modela fotoaparata

### Model kamere nije otkriven

**Mogući uzroci:**

* Izmijenjeni EXIF metapodaci
* Slike uređene u vanjskom softveru
* Nedovršen prijenos datoteke

**Rješenje**: Ponovno uvezite izvorne, neizmijenjene datoteke s fotoaparata/SD kartice

### Nedostaju vremenske oznake

**Mogući uzroci:**

* Sat fotoaparata nije ispravno postavljen
* EXIF podatke skinuo je vanjski softver

**Rješenje**: Provjerite jesu li postavke vremena kamere točne tijekom snimanja

***

## Sljedeći koraci

Nakon što se vaše datoteke uvezu:

1. **Pregledajte popis datoteka** - Provjerite jesu li sve slike ispravno učitane
2. **Provjerite modele kamera** - Provjerite ispravno otkrivanje kamere
3. **Označite ciljane slike** - Pogledajte [Odabir ciljnih slika](choosing-target-images.md)
4. **Prilagodite postavke** - Konfigurirajte opcije obrade u [Postavke projekta](adjusting-project-settings.md)
5. **Pokrenite obradu** - Pogledajte [Pokretanje obrade](starting-the-processing.md)

Za detaljne informacije o konfiguraciji projekta pogledajte [Prilagođavanje postavki projekta](adjusting-project-settings.md).