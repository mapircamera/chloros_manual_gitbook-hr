# Podešavanje postavki projekta

Prije obrade vaših slika, važno je konfigurirati postavke projekta kako bi odgovarale zahtjevima vašeg tijeka rada. Postavke projekta<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">ploča pruža sveobuhvatnu kontrolu nad kalibracijom, opcijama obrade, multispektralnim indeksima i izvoznim formatima.

## Pristup postavkama projekta

1. Otvorite svoj projekt u Chlorosu
2. Kliknite **Postavke projekta**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">ikonu na lijevoj bočnoj traci
3. Ploča Postavke projekta prikazuje sve opcije konfiguracije

{% hint style="info" %}**Postavke se automatski spremaju**s vašim projektom. Kada ponovno otvorite projekt, vraćaju se sve postavke.
{% endhint %}***

## Brzo postavljanje za uobičajene tijekove rada

### Zadane postavke (preporučeno za većinu korisnika)

Za tipične tijekove rada kamere MAPIR Survey3, zadane postavke dobro funkcioniraju:

* ✅ **Ispravak vinjete**: Omogućeno
* ✅ **Kalibracija refleksije**: Omogućeno (zahtijeva slike MAPIR ciljeva)
* ✅ **Debayer metoda**: Visoka kvaliteta (brže)
* ✅ **Format izvoza**: TIFF (16-bitni)

Jednostavno uvezite svoje slike i počnite s obradom s ovim zadanim postavkama.***## Pregled postavki projekta

Ploča Postavke projekta organizirana je u nekoliko kategorija. U nastavku je sažetak svakog odjeljka. Za potpunu dokumentaciju pogledajte [Postavke projekta](../project-settings/project-settings.md).

### Otkrivanje cilja

Kontrolira način na koji Chloros identificira ciljeve kalibracije na vašim slikama.

**Ključne postavke:**

* **Minimalno područje uzorka kalibracije**: prag veličine za otkrivanje cilja (zadano: 25 piksela)
* **Minimalno ciljano grupiranje**: Prag sličnosti za grupiranje ciljnih regija (zadano: 60)**Kada prilagoditi:**

* Povećajte područje uzorka ako dođe do lažnih detekcija
* Smanjite ako se ciljevi ne otkrivaju
* Prilagodite grupiranje ako se ciljevi dijele na više detekcija

### Obrada

Glavne mogućnosti obrade slike i kalibracije.

**Ključne postavke:**

* **Korekcija vinjete**: kompenzira zatamnjenje leće na rubovima ✅ Preporučeno
* **Kalibracija refleksije**: normalizira vrijednosti pomoću ciljeva kalibracije ✅ Preporučeno
* **Debayerova metoda**: Algoritam za pretvaranje RAW-a u 3-kanalni višespektralni
* **Minimalni interval ponovne kalibracije**: Vrijeme između korištenja ciljeva kalibracije (0 = koristiti sve)**Napredne postavke:**

* **Pomak vremenske zone senzora svjetla**: Za PPK sinkronizaciju vremena (zadano: 0)
* **Primijeni PPK ispravke**: koristi GPS/podatke o ekspoziciji iz .daq datoteka
* **Pribadača za ekspoziciju 1/2**: Dodjeljuje kamere iglama za ekspoziciju za postavke s dvije kamere

### Indeks (Multispektralni indeksi)

Konfigurirajte koje indekse vegetacije izračunati i izvesti.**Kako dodati indekse:**1. Kliknite gumb**"Dodaj indeks"**2. Odaberite indeks s padajućeg izbornika (NDVI, NDRE, GNDVI itd.)
3. Konfigurirajte postavke vizualizacije (LUT boje, rasponi vrijednosti)
4. Po potrebi dodajte više indeksa**Popularni indeksi:**

* **NDVI**: Opće zdravlje vegetacije (najčešće)
* **NDRE**: Rano otkrivanje stresa uz RedEdge
* **GNDVI**: Osjetljivo na koncentraciju klorofila
* **OSAVI**: Dobro radi s vidljivim tlom
* **EVI**: regije s visokim indeksom lisne površine (LAI).**Prilagođene formule (samo Chloros+):**

* Stvorite prilagođene formule multispektralnog indeksa
* Koristite matematiku opsega sa svim kanalima slike
* Spremite prilagođene formule za ponovnu upotrebu

Za sve dostupne indekse i formule pogledajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md).

### Izvoz

Kontrolira format i kvalitetu izlazne datoteke.

**Dostupni formati:**

* **TIFF (16-bitni)**: Preporučeno za GIS i znanstvene analize (0-65,535 raspon)
* **TIFF (32-bit, postotak)**: vrijednosti refleksije s pomičnim zarezom (raspon 0,0-1,0)
* **PNG (8-bitni)**: kompresija bez gubitaka za vizualizaciju (raspon 0-255)
* **JPG (8-bitni)**: Najmanje datoteke, kompresija s gubitkom (0-255 raspon)***## Spremanje i učitavanje postavki

### Spremi predložak projekta

Stvorite predloške za višekratnu upotrebu za dosljedne tijekove rada:

1. Konfigurirajte sve željene postavke na ploči Postavke projekta
2. Pomaknite se do odjeljka **"Spremi predložak projekta"**na dnu
3. Unesite opisni naziv predloška (npr. "Survey3N\_RGN\_Agriculture")
4. Pritisnite ikonu za spremanje**Prednosti:**

* Primijenite identične postavke na više projekata
* Dijelite konfiguracije s članovima tima
* Održavajte dosljednost za ponovljene ankete

### Učitaj predložak na novom projektu

Prilikom izrade novog projekta:

1. Odaberite **"Novi projekt"**iz glavnog izbornika
2. Odaberite opciju**"Učitaj iz predloška"**3. Odaberite svoj spremljeni predložak
4. Sve postavke se automatski primjenjuju

### Radni imenik

Postavka**"Spremi mapu projekta"** određuje gdje se prema zadanim postavkama stvaraju novi projekti:

* **Zadana lokacija**:`C:\Users\[Username]\Chloros Projects`
* **Promijeni lokaciju**: Kliknite ikonu za uređivanje i odaberite novu mapu
* **Kada promijeniti**:
* Mrežni pogon za timsku suradnju
* Drugačiji pogon s više prostora za pohranu
* Organizirana struktura mapa po godini/klijentu

***

## PPK (Post-Processed Kinematic) postavke

Ako koristite MAPIR DAQ snimače s GPS-om za preciznu geolokaciju:

### Preduvjeti

* MAPIR DAQ s GPS (GNSS) modulom
* .daq log datoteka s unosima pinova za izlaganje
* Kamera spojena na DAQ igle za ekspoziciju tijekom sesije snimanja

### Konfiguracijski koraci

1. Postavite .daq log datoteku u mapu vašeg projekta
2. U postavkama projekta uključite potvrdni okvir **"Primijeni PPK ispravke"**3. Postavite**"Pomak vremenske zone senzora svjetla"** ako je potrebno (zadano: 0 za UTC)
4. Dodijelite kamere iglama ekspozicije:
* **Jedna kamera**: Automatski se dodjeljuje Pin 1
* **Dvostruke kamere**: Ručno dodijelite svakoj kameri ispravan pin**Dodjela igle za ekspoziciju:**

* **Oznaka ekspozicije 1**: Odaberite model kamere s padajućeg izbornika
* **Oznaka ekspozicije 2**: Odaberite drugu kameru ili "Ne koristi"
* Ista kamera ne može se dodijeliti objema iglama

{% hint style="warning" %}
**Važno**: igle za ekspoziciju moraju biti ispravno dodijeljene odgovarajućim kamerama. Netočna dodjela rezultirat će pogrešnim geolokacijskim podacima.
{% endhint %}***## Napredni scenariji

### Projekti s više kamera

Prilikom obrade slika s više MAPIR kamera u jednom projektu:

1. Chloros automatski detektira svaki model kamere
2. Svaka kamera dobiva odgovarajući profil obrade
3. PPK: Ručno dodijelite svakoj kameri iglu za ispravnu ekspoziciju
4. Sve kamere koriste isti izvozni format i indekse

**Primjer**: Survey3W RGN + Survey3N OCN oprema s dvije kamere

### Ankete s vremenskim odmakom ili više datuma

Za ponovljena istraživanja istog područja tijekom vremena:

1. Izradite predložak sa svojim standardnim postavkama
2. U svakoj sesiji koristite dosljedno postavljanje cilja kalibracije
3. Obradite svaki datum kao zaseban projekt
4. Koristite identične postavke za usporedive rezultate
5. Izvoz u istom formatu za vremensku analizu

### Veliki skupovi podataka

Za projekte s mnogo slika (500+):

* Razmislite o podjeli na manje projekte prema datumu ili području
* Koristite Chloros+ paralelnu obradu za brže rezultate
* Razmotrite CLI ili API za grupnu automatizaciju
* Podesite minimalni interval ponovne kalibracije kako biste smanjili vrijeme otkrivanja cilja

***

## Provjera vaših postavki

Prije početka obrade pregledajte ove ključne postavke:

* [ ] Model kamere ispravno otkriven u pregledniku datoteka
* [ ] Ispravak vinjete omogućen
* [ ] Omogućena je kalibracija refleksije
* [ ] Uvezena je najmanje jedna slika cilja kalibracije
* [ ] Dodani su željeni multispektralni indeksi
* [ ] Format izvoza prikladan za vaš tijek rada
* [ ] PPK postavke konfigurirane (ako koristite .daq s događajima izloženosti)

***## Sljedeći koraci

Nakon što su vaše postavke konfigurirane:

1.**Označite ciljane slike kalibracije**- Pogledajte [Odabir ciljnih slika](choosing-target-images.md)
2.**Pokreni obradu**- Vidi [Pokretanje obrade](starting-the-processing.md)
3.**Praćenje napretka** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)

Za sve pojedinosti o svim dostupnim postavkama pogledajte referentnu dokumentaciju [Postavke projekta](../project-settings/project-settings.md).
