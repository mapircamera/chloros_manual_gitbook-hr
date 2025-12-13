# Podešavanje postavki projekta

Prije obrade slika, važno je konfigurirati postavke projekta tako da odgovaraju vašim zahtjevima tijeka rada. Ploča Postavke projekta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> nudi potpunu kontrolu nad kalibracijom, opcijama obrade, multispektralnim indeksima i izvoznim formatima.

## Pristup postavkama projekta

1. Otvorite svoj projekt u Chlorosu
2. Kliknite ikonu **Postavke projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
3. Ploča Postavke projekta prikazuje sve opcije konfiguracije

{% hint style=&quot;info&quot; %}
**Postavke se automatski spremaju** s projektom. Kada ponovno otvorite projekt, vraćaju se sve postavke.
{% endhint %}

***

## Brzo postavljanje za uobičajene tijekove rada

### Zadane postavke (preporučeno većini korisnika)

Za tipične tijekove rada kamere MAPIR Survey3, zadane postavke dobro funkcioniraju:

* ✅ **Ispravak vinjete**: Omogućeno
* ✅ **Kalibracija refleksije**: Omogućeno (zahtijeva MAPIR ciljno snimanje)
* ✅ **Debayer metoda**: Visoka kvaliteta (brže)
* ✅ **Format izvoza**: TIFF (16 bita)

Jednostavno uvezite svoje slike i počnite ih obrađivati ​​s ovim zadanim vrijednostima.

***

##Pregled konfiguracije projekta

Ploča Postavke projekta organizirana je u nekoliko kategorija. U nastavku je sažetak svakog odjeljka. Za potpunu dokumentaciju pogledajte [Postavke projekta](../project-settings/project-settings.md).

### Otkrivanje cilja

Kontrolira način na koji Chloros identificira ciljeve kalibracije na vašim slikama.

**Ključne postavke:**

* **Minimalno područje uzorka kalibracije**: Prag veličine za otkrivanje cilja (zadano: 25 piksela).
* **Minimalno ciljano grupiranje**: Prag sličnosti za ciljane regije klasteriranja (zadano: 60).

**Kada prilagoditi:**

* Povećajte područje uzorka ako dobijete lažne detekcije.
* Smanjite ga ako ciljevi nisu otkriveni.
* Prilagodite grupiranje ako su ciljevi podijeljeni u više detekcija.

### Obrada

Glavne mogućnosti obrade slike i kalibracije.

**Ključne postavke:**

* **Korekcija vinjete** – kompenzira zatamnjenje leće na rubovima ✅ Preporučeno
* **Kalibracija refleksije** – normalizira vrijednosti pomoću ciljeva kalibracije ✅ Preporučeno
* **Debayerova metoda**: algoritam za pretvaranje RAW-a u 3-kanalni multispektralni
* **Minimalni interval rekalibracije**: Vrijeme između korištenja ciljeva kalibracije (0 = koristiti sve)

**Napredne postavke:**

* **Pomak vremenske zone senzora svjetla**: Za PPK sinkronizaciju vremena (zadano: 0)
* **Primijeni PPK popravke**: koristi pin/GPS podatke o ekspoziciji iz .daq datoteka
* **Igla za ekspoziciju 1/2**: Dodijelite kamere iglama za ekspoziciju za postavke dvostruke kamere

### Indeks (multispektralni indeksi)

Konfigurirajte koje indekse vegetacije izračunati i izvesti.

**Kako dodati indekse:**

1. Pritisnite gumb **«Dodaj indeks»**
2. Odaberite indeks s padajućeg izbornika (NDVI, NDRE, GNDVI, itd.).
3. Konfigurirajte postavke prikaza (LUT boje, rasponi vrijednosti).
4. Dodajte više indeksa prema potrebi.

**Popularni indeksi:**

* **NDVI**: Opće zdravlje vegetacije (najčešće).
* **NDRE**: Rano otkrivanje stresa uz RedEdge.
* **GNDVI**: Osjetljivo na koncentraciju klorofila.
* **OSAVI**: Dobro radi s vidljivim tlom
* **EVI**: Regije s visokim indeksom lisne površine (LAI)

**Prilagođene formule (samo Chloros+):**

* Stvorite prilagođene formule multispektralnog indeksa
* Koristite matematiku opsega sa svim kanalima slike
* Spremite prilagođene formule za ponovnu upotrebu

Da biste vidjeli sve dostupne indekse i formule, pogledajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md).

### Izvoz

Kontrolira format i kvalitetu izlazne datoteke.

**Dostupni formati:**

* **TIFF (16-bitni)**: Preporučeno za GIS i znanstvene analize (raspon od 0 do 65,535).
* **TIFF (32-bit, postotak)**: Vrijednosti refleksije s pomičnim zarezom (raspon od 0,0 do 1,0).
* **PNG (8-bitni)**: Kompresija bez gubitaka za prikaz (raspon od 0 do 255).
* **JPG (8-bitni)**: manje datoteke, kompresija s gubitkom (raspon od 0 do 255).

***

## Spremite i učitajte postavke

### Spremite predložak projekta

Stvorite predloške za višekratnu upotrebu za dosljedne tijekove rada:

1. Konfigurirajte sve željene postavke na ploči Postavke projekta.
2. Pomaknite se do odjeljka **“Spremi predložak projekta”** na dnu.
3. Unesite opisni naziv za predložak (na primjer, “Survey3N\_RGN\_Agriculture”).
4. Pritisnite ikonu Spremi.

**Prednosti:**

* Primijenite iste postavke na više projekata.
* Podijelite postavke s članovima tima.
* Održavajte dosljednost u ponovljenim anketama.

### Učitaj predložak u novi projekt

Prilikom izrade novog projekta:

1. Odaberite **«Novi projekt»** iz glavnog izbornika.
2. Odaberite opciju **«Učitaj iz predloška»**.
3. Odaberite spremljeni predložak.
4. Sve postavke će se primijeniti automatski.

### Radni direktorij

Postavka **"Spremi mapu projekta"** određuje gdje se prema zadanim postavkama stvaraju novi projekti:

* **Zadana lokacija**: `C:\Users\[Username]\Chloros Projects`.
* **Promijeni lokaciju**: Kliknite ikonu za uređivanje i odaberite novu mapu.
* **Kada to promijeniti**:
  * Mrežni pogon za timsku suradnju.
  * Drugačija jedinica s više prostora za pohranu.
  * Struktura mape organizirana po godini/klijentu.

***

## PPK (Post Processed Kinematics) postavke

Ako koristite DAQ MAPIR loggere s GPS-om za preciznu geolokaciju:

### Preduvjeti

* DAQ MAPIR s GPS modulom (GNSS)
* .daq log datoteka s unosima pribadače za izlaganje
* Kamera spojena na DAQ igle za ekspoziciju tijekom sesije snimanja

### Konfiguracijski koraci

1. Postavite .daq datoteku dnevnika u mapu vašeg projekta
2. U postavkama projekta označite okvir **«Primijeni PPK popravke»**
3. Postavite **«Vremenski pomak svjetlosnog senzora»** ako je potrebno (zadano: 0 za UTC)
4. Dodijelite kamere iglama ekspozicije:
   * **Jedna kamera**: Automatski se dodjeljuje pinu 1.
   * **Dvije kamere**: Ručno dodijelite svaku kameru ispravnom pinu.

**Dodjela izložbene igle:**

* **Oznaka ekspozicije 1**: Odaberite model kamere iz padajućeg izbornika.
* **Oznaka ekspozicije 2**: Odaberite drugu kameru ili "Ne koristi".
* Ista kamera ne može se dodijeliti objema iglama.

{% hint style=&quot;upozorenje&quot; %}
**Važno**: igle za ekspoziciju moraju biti ispravno dodijeljene odgovarajućim kamerama. Netočno mapiranje rezultirat će pogrešnim geolokacijskim podacima.
{% endhint %}

***

## Napredni scenariji

### Projekti s više kamera

Prilikom obrade slika s više MAPIR kamera u projektu:

1. Chloros automatski detektira svaki model kamere
2. Svaka kamera dobiva pravi profil obrade
3. PPK: Ručno dodijelite svakoj kameri odgovarajuću iglu ekspozicije.
4. Sve kamere koriste isti izvozni format i indekse.

**Primjer**: oprema s dvostrukom kamerom Survey3W RGN + Survey3N OCN

### Studije s vremenskim odmakom ili na nekoliko datuma

Za ponovljene studije istog područja tijekom vremena:

1. Napravite predložak sa svojim standardnim postavkama.
2. Koristite dosljednu postavku cilja kalibracije svake sesije.
3. Obradite svaki datum kao zaseban projekt.
4. Koristite identične postavke za usporedive rezultate.
5. Izvezite u isti format za vremensku analizu.

### Veliki skupovi podataka

Za projekte s mnogo slika (više od 500):

* Razmotrite njihovo razbijanje na manje projekte po datumu ili području.
* Koristite Chloros+ paralelnu obradu za brže rezultate.
* Razmotrite CLI ili API za grupnu automatizaciju.
* Podesite minimalni interval ponovne kalibracije kako biste smanjili vrijeme otkrivanja cilja.

***

## Provjera konfiguracije

Prije nego počnete s obradom, pregledajte ove ključne postavke:

* [ ] Model kamere ispravno otkriven u pregledniku datoteka.
* [ ] Ispravak vinjete omogućen.
* [ ] Omogućena je kalibracija refleksije.
* [ ] Barem jedna uvezena slika cilja kalibracije.
* [ ] Dodani željeni multispektralni indeksi.
* [ ] Format izvoza prikladan za vaš tijek rada.
* [ ] Konfigurirane PPK postavke (ako koristite .daq s događajima izloženosti).

***

## Sljedeći koraci

Nakon što su postavke konfigurirane:

1. **Provjerite ciljne slike kalibracije**: Pogledajte [Odabir ciljnih slika](choosing-target-images.md)
2. **Pokrenite obradu**: Pogledajte [Pokretanje obrade](starting-the-processing.md)
3. **Praćenje napretka**: Vidi [Praćenje obrade](monitoring-the-processing.md).

Za detaljne informacije o svim dostupnim postavkama pogledajte referentnu dokumentaciju [Postavke projekta](../project-settings/project-settings.md).