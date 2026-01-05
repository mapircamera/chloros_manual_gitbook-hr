# Postavke projekta

Postavke projekta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> bočna traka u Chloros omogućuje vam da konfigurirate sve aspekte obrade slike, detekciju cilja kalibracije, izračune multispektralnog indeksa i opcije izvoza za vaš projekt. Ove se postavke spremaju s vašim projektom i mogu se spremiti kao predlošci za ponovnu upotrebu u više projekata.

## Pristup postavkama projekta

Za pristup postavkama projekta:

1. Otvorite projekt u Chloros
2. Kliknite karticu **Postavke projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lijevoj bočnoj traci
3. Ploča postavki prikazat će sve dostupne opcije konfiguracije organizirane po kategorijama

***

## Otkrivanje cilja

Ove postavke kontroliraju način na koji Chloros otkriva i obrađuje ciljeve kalibracije na vašim slikama.

### Minimalno područje uzorka kalibracije (px)

* **Tip**: Broj
* **Raspon**: 0 do 10 000 piksela
* **Zadano**: 25 piksela
* **Opis**: Postavlja minimalnu površinu (u pikselima) potrebnu da bi se detektirana regija smatrala važećim uzorkom cilja kalibracije. Manje vrijednosti otkrit će manje mete, ali mogu povećati broj lažno pozitivnih rezultata. Veće vrijednosti zahtijevaju veće, jasnije ciljne regije za detekciju.
* **Kada prilagoditi**:
  * Povećajte ako dobivate lažne detekcije na malim artefaktima slike
  * Smanjite ako se vaši kalibracijski ciljevi čine malima na vašim slikama i ne otkrivaju se

### Minimalno ciljano grupiranje (0-100)

* **Tip**: Broj
* **Raspon**: 0 do 100
* **Zadano**: 60
* **Opis**: Kontrolira prag grupiranja za grupiranje regija sličnih boja prilikom otkrivanja ciljeva kalibracije. Više vrijednosti zahtijevaju grupiranje više sličnih boja, što rezultira konzervativnijom detekcijom cilja. Niže vrijednosti dopuštaju više varijacija boja unutar ciljane skupine.
* **Kada prilagoditi**:
  * Povećajte ako se ciljevi kalibracije dijele na više detekcija
  * Smanjite ako kalibracijski ciljevi s varijacijom boja nisu u potpunosti otkriveni

***

## Obrada

Ove postavke kontroliraju kako Chloros obrađuje i kalibrira vaše slike.

### Ispravak vinjete

* **Tip**: Potvrdni okvir
* **Zadano**: Omogućeno (označeno)
* **Opis**: Primjenjuje korekciju vinjete za kompenzaciju zatamnjenja leće na rubovima slika. Vinjetiranje je čest optički fenomen gdje kutovi i rubovi slike izgledaju tamniji od središta zbog karakteristika leće.
* **Kada onemogućiti**: Onemogućite samo ako je vaša kombinacija fotoaparata/objektiva već primijenila korekciju vinjete ili ako želite ručno ispraviti vinjetiranje u naknadnoj obradi.

### Kalibracija refleksije / balans bijele boje

* **Tip**: Potvrdni okvir
* **Zadano**: Omogućeno (označeno)
* **Opis**: Omogućuje automatsku kalibraciju refleksije pomoću otkrivenih ciljeva kalibracije na vašim slikama. Ovo normalizira vrijednosti refleksije u vašem skupu podataka i osigurava dosljedna mjerenja bez obzira na svjetlosne uvjete.
* **Kada onemogućiti**: Onemogućite samo ako želite obraditi neobrađene, nekalibrirane slike ili ako koristite drugačiji tijek rada kalibracije.

### Debayer metoda

* **Vrsta**: Odabir s padajućeg izbornika
* **Opcije**:
  * Visoka kvaliteta (brže) - Trenutno jedina dostupna opcija
* **Zadano**: Visoka kvaliteta (brže)
* **Opis**: Odabire algoritam demosaicing koji se koristi za pretvaranje neobrađenih Bayerovih podataka senzora uzorka u slike u boji. Metoda "Visoka kvaliteta (brže)" pruža optimalnu ravnotežu između brzine obrade i kvalitete slike.
* **Napomena**: Dodatne metode debayera mogu se dodati u budućim verzijama Chloros.

### Minimalni interval rekalibracije

* **Tip**: Broj
* **Raspon**: 0 do 3600 sekundi
* **Zadano**: 0 sekundi
* **Opis**: Postavlja minimalni vremenski interval (u sekundama) između korištenja ciljeva kalibracije. Kada je postavljen na 0, Chloros će koristiti svaki otkriveni cilj kalibracije. Kada se postavi na višu vrijednost, Chloros će koristiti samo ciljeve kalibracije koji su odvojeni barem ovoliko sekundi, smanjujući vrijeme obrade za skupove podataka s čestim snimanjem ciljeva kalibracije.
* **Kada prilagoditi**:
  * Postavite na 0 za maksimalnu točnost kalibracije kada se uvjeti osvjetljenja razlikuju
  * Povećajte (npr. na 60-300 sekundi) za bržu obradu kada je osvjetljenje dosljedno i imate česte slike cilja kalibracije

### Pomak vremenske zone senzora svjetla

* **Tip**: Broj
* **Raspon**: -12 do +12 sati
* **Zadano**: 0 sati
* **Opis**: Određuje pomak vremenske zone (u satima od UTC) za vremenske oznake podataka svjetlosnog senzora. Ovo se koristi prilikom obrade PPK (Post-Processed Kinematic) podatkovnih datoteka kako bi se osigurala ispravna vremenska sinkronizacija između snimljenih slika i GPS podataka.
* **Kada prilagoditi**: Postavite ovo na pomak lokalne vremenske zone ako vaši PPK podaci koriste lokalno vrijeme umjesto UTC-a. Na primjer:
  * Pacifičko vrijeme: -8 ili -7 (ovisno o DST-u)
  * Istočno vrijeme: -5 ili -4 (ovisno o DST-u)
  * Srednjoeuropsko vrijeme: +1 ili +2 (ovisno o DST-u)

### Primijeni PPK ispravke

* **Tip**: Potvrdni okvir
* **Zadano**: Onemogućeno (neoznačeno)
* **Opis**: Omogućuje korištenje naknadno obrađenih kinematskih (PPK) korekcija iz MAPIR DAQ snimača koji sadrže GPS (GNSS). Kada je omogućeno, Chloros će koristiti sve .daq datoteke dnevnika koje sadrže podatke o ekspoziciji u direktoriju vašeg projekta i primijeniti precizne ispravke geolokacije na vaše slike.
* **Zahtjev**: .daq datoteka dnevnika s unosima pribadača za izlaganje mora biti prisutna u direktoriju vašeg projekta
* **Kada omogućiti**: Preporuča se uvijek omogućiti PPK korekciju ako imate unose povratnih informacija o izloženosti u vašoj .daq log datoteci.

### Ekspozicija Pin 1

* **Vrsta**: Odabir s padajućeg izbornika
* **Vidljivost**: Vidljivo samo kada je omogućena "Primijeni PPK korekcije" I podaci o izloženosti dostupni su za Pin 1
* **Opcije**:
  * Nazivi modela fotoaparata otkriveni u projektu
  * "Ne koristi" - Zanemarite ovu iglu za ekspoziciju
* **Zadano**: Automatski odabrano na temelju konfiguracije projekta
* **Opis**: Dodjeljuje određenu kameru Exposure Pin 1 za PPK sinkronizaciju vremena. Igla za ekspoziciju bilježi točno vrijeme kada se okidač kamere aktivira, što je ključno za točnu PPK geolokaciju.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedna igla: Automatski odabire kameru
  * Jedna kamera + dva pina: Pin 1 automatski se dodjeljuje kameri
  * Više kamera: potreban je ručni odabir

### Ekspozicija Pin 2

* **Vrsta**: Odabir s padajućeg izbornika
* **Vidljivost**: Vidljivo samo kada je omogućena "Primijeni PPK korekcije" I dostupni su podaci o izloženosti za Pin 2
* **Opcije**:
  * Nazivi modela fotoaparata otkriveni u projektu
  * "Ne koristi" - Zanemarite ovu iglu za ekspoziciju
* **Zadano**: Automatski odabrano na temelju konfiguracije projekta
* **Opis**: Dodjeljuje određenu kameru Exposure Pin 2 za PPK sinkronizaciju vremena kada se koristi postava s dvije kamere.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedan pin: Pin 2 automatski je postavljen na "Ne koristi"
  * Jedna kamera + dvije igle: Pin 2 automatski je postavljen na "Ne koristi"
  * Više kamera: potreban je ručni odabir
* **Napomena**: Ista kamera ne može se dodijeliti i Pin 1 i Pin 2 istovremeno.***

## Indeks

Ove postavke omogućuju vam konfiguriranje multispektralnih indeksa za analizu i vizualizaciju.

### Dodaj indeks

* **Vrsta**: Posebna ploča za konfiguraciju indeksa
* **Opis**: Otvara interaktivnu ploču na kojoj možete odabrati i konfigurirati multispektralne vegetacijske indekse (NDVI, NDRE, EVI itd.) za izračun tijekom obrade slike. Možete dodati više indeksa, svaki sa svojim postavkama vizualizacije.
* **Dostupni indeksi**: Sustav uključuje 30+ unaprijed definiranih multispektralnih indeksa uključujući:
  * NDVI (normalizirani vegetacijski indeks razlike)
  * NDRE (Normalizirana razlika RedEdge)
  * EVI (poboljšani indeks vegetacije)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * I mnogo više (pogledajte [Formule multispektralnog indeksa](multispectral-index-formulas.md) za potpuni popis)
* **Značajke**:
  * Odaberite jednu od unaprijed definiranih formula indeksa
  * Konfigurirajte vizualizaciju gradijenata boja (LUT - Look-Up Tables)
  * Postavite granične vrijednosti za analizu
  * Stvorite prilagođene formule indeksa

### Prilagođene formule (značajka Chloros+)

* **Tip**: niz definicija prilagođenih formula
* **Opis**: Omogućuje stvaranje i spremanje prilagođenih formula multispektralnog indeksa pomoću matematike pojasa. Prilagođene formule spremaju se s vašim postavkama projekta i mogu se koristiti kao ugrađeni indeksi.
* **Kako stvoriti**:
  1. Na konfiguracijskoj ploči indeksa potražite opciju prilagođene formule
  2. Definirajte svoju formulu pomoću identifikatora pojasa (npr. NIR, Red, Green, Blue)
  3. Spremite formulu s opisnim nazivom
* **Sintaksa formule**: Podržane su standardne matematičke operacije, uključujući:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Zagrade za redoslijed operacija
  * Reference bendova: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Izvoz

Ove postavke kontroliraju format i kvalitetu izvezenih obrađenih slika.

### Kalibrirani format slike

* **Vrsta**: Odabir s padajućeg izbornika
* **Opcije**:
  * **TIFF (16-bitni)** - Nekomprimirani 16-bitni TIFF format
  * **TIFF (32-bit, postotak)** - 32-bitni pokretni zarez TIFF s vrijednostima refleksije kao postocima
  * **PNG (8-bitni)** - komprimirani 8-bitni PNG format
  * **JPG (8-bitni)** - komprimirani 8-bitni JPEG format
* **Zadano**: TIFF (16-bitni)
* **Opis**: Odabire format datoteke za spremanje obrađenih i kalibriranih slika.
* **Preporuke za format**:
  * **TIFF (16-bitni)**: Preporučeno za znanstvene analize i profesionalne tijekove rada. Čuva maksimalnu kvalitetu podataka bez artefakata kompresije. Najbolji za multispektralnu analizu i daljnju obradu u GIS softveru.
  * **TIFF (32-bit, postotak)**: najbolje za radne procese koji zahtijevaju vrijednosti refleksije kao postotke (0-100%). Nudi maksimalnu preciznost za radiometrijska mjerenja.
  * **PNG (8-bitni)**: Dobar za web pregled i opću vizualizaciju. Manje veličine datoteka s kompresijom bez gubitaka, ali smanjenim dinamičkim rasponom.
  * **JPG (8-bitni)**: Najmanje veličine datoteka, najbolje samo za preglede i web prikaz. Koristi kompresiju s gubitkom koja nije prikladna za znanstvenu analizu.***

## Spremi predložak projekta

Ova vam značajka omogućuje spremanje trenutnih postavki projekta kao predloška za višekratnu upotrebu.

* **Tip**: unos teksta + gumb Spremi
* **Opis**: Unesite opisni naziv za predložak postavki i kliknite ikonu za spremanje. Predložak će pohraniti sve vaše trenutne postavke projekta (otkrivanje cilja, opcije obrade, indekse i format izvoza) za jednostavnu ponovnu upotrebu u budućim projektima.
* **Slučajevi upotrebe**:
  * Stvorite predloške za različite sustave kamera (RGB, multispektralni, NIR)
  * Spremite standardne konfiguracije za određene vrste usjeva ili tijek rada analize
  * Podijelite dosljedne postavke u timu
* **Kako koristiti**:
  1. Konfigurirajte sve željene postavke projekta
  2. Unesite naziv predloška (npr. "RedEdge Survey3 NDVI Standard")
  3. Pritisnite ikonu za spremanje
  4. Predložak se sada može učitati prilikom izrade novih projekata

***

## Spremi mapu projekta

Ova postavka određuje gdje se novi projekti spremaju prema zadanim postavkama.

* **Tip**: Prikaz putanje direktorija + gumb Uredi
* **Zadano**: `C:\Users\[Username]\Chloros Projects`
* **Opis**: Prikazuje trenutni zadani direktorij u kojem se stvaraju novi Chloros projekti. Pritisnite ikonu za uređivanje da odaberete drugi direktorij.
* **Kada promijeniti**:
  * Postavite na mrežni pogon za timsku suradnju
  * Prijeđite na disk s više prostora za pohranu velikih skupova podataka
  * Organizirajte projekte prema godini, klijentu ili vrsti projekta u različitim mapama
* **Napomena**: Promjena ove postavke utječe samo na NOVE projekte. Postojeći projekti ostaju na svojim izvornim lokacijama.***

## Postojanost postavki

Sve postavke projekta automatski se spremaju s vašom projektnom datotekom (format projekta `.mapir`). Kada ponovno otvorite projekt, sve postavke se vraćaju točno onako kako ste ih ostavili.

### Hijerarhija postavki

Postavke se primjenjuju sljedećim redoslijedom:

1. **Zadane postavke sustava** - Ugrađene zadane postavke definira Chloros
2. **Postavke predloška** - Ako učitavate predložak prilikom izrade projekta
3. **Spremljene postavke projekta** - Postavke spremljene s datotekom projekta
4. **Ručne prilagodbe** - Sve promjene koje napravite tijekom trenutne sesije

### Postavke i obrada slike

Većina promjena postavki (osobito u kategorijama obrade i izvoza) pokrenut će ponovnu obradu slika kako bi se odrazile nove postavke. Međutim, neke su postavke "samo za izvoz" i ne zahtijevaju trenutnu ponovnu obradu:

* Spremi predložak projekta
* Radni imenik
* Kalibrirani format slike (primjenjuje se prilikom izvoza)

***

## Najbolji primjeri iz prakse

1. **Počnite sa zadanim postavkama**: Zadane postavke dobro funkcioniraju za većinu sustava kamera MAPIR i tipične tijekove rada.
2. **Stvorite predloške**: Nakon što optimizirate postavke za određeni tijek rada ili kameru, spremite ih kao predložak kako biste osigurali dosljednost u projektima.
3. **Testirajte prije potpune obrade**: Prilikom eksperimentiranja s novim postavkama, testirajte na malom podskupu slika prije obrade cijelog skupa podataka.
4. **Dokumentirajte svoje postavke**: Koristite opisne nazive predložaka koji označavaju sustav kamere, vrstu obrade i namjeravanu upotrebu (npr. "Survey3\_RGB\_NDVI\_Poljoprivreda").
5. **Odabir formata izvoza**: Odaberite format izvoza na temelju vaše krajnje upotrebe:
   * Znanstvena analiza → TIFF (16-bitni ili 32-bitni)
   * GIS obrada → TIFF (16-bitni)
   * Brza vizualizacija → PNG (8-bitni)
   * Dijeljenje na webu → JPG (8-bitni)

***

Za više informacija o multispektralnim indeksima u Chloros pogledajte stranicu [Formule multispektralnog indeksa](multispectral-index-formulas.md).