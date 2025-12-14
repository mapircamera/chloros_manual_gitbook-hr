# Postavke projekta

Bočna traka s postavkama projekta Chloros <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> omogućuje konfiguriranje svih aspekata obrade slike vašeg projekta, otkrivanje cilja kalibracije, izračune multispektralnog indeksa i opcije izvoza. Ove se postavke spremaju s vašim projektom i mogu se spremiti kao predlošci za ponovnu upotrebu u više projekata.

## Pristup postavkama projekta

Za pristup postavkama projekta:

1. Otvorite projekt u Chlorosu
2. Kliknite karticu **Postavke projekta**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na lijevoj bočnoj traci.
3. Ploča postavki prikazat će sve dostupne opcije postavki organizirane po kategorijama.***

## Otkrivanje cilja

Ove postavke kontroliraju način na koji Chloros otkriva i obrađuje ciljeve kalibracije na vašim slikama.

### Minimalno područje uzorka kalibracije (px)

* **Tip**: Broj
* **Raspon**: 0 do 10 000 piksela
* **Zadano**: 25 piksela
* **Opis**: postavlja minimalnu površinu (u pikselima) potrebnu da bi se detektirana regija smatrala važećim uzorkom cilja kalibracije. Manje vrijednosti otkrit će manje mete, ali mogu povećati broj lažno pozitivnih rezultata. Veće vrijednosti zahtijevaju veće i jasnije ciljane regije za detekciju.
* **Kada prilagoditi**:
  * Povećajte ako dobijete lažna otkrivanja na malim artefaktima slike.
  * Smanjite ako se vaši kalibracijski ciljevi čine malima na vašim slikama i ne otkrivaju se.

### Minimalno ciljano grupiranje (0-100)

* **Tip**: Broj.
* **Raspon**: Od 0 do 100.
* **Zadano**: 60.
* **Opis**: Kontrolira prag grupiranja za grupiranje regija sličnih boja prilikom otkrivanja ciljeva kalibracije. Više vrijednosti zahtijevaju grupiranje više sličnih boja, što rezultira konzervativnijom detekcijom cilja. Niže vrijednosti omogućuju veću varijaciju boja unutar ciljane skupine.
* **Kada prilagoditi**:
  * Povećajte ako su ciljevi kalibracije podijeljeni u više detekcija.
  * Smanjite ako ciljevi kalibracije s varijacijom boje nisu u potpunosti otkriveni.

***

## Obrada

Ove postavke kontroliraju način na koji Chloros obrađuje i kalibrira vaše slike.

### Ispravak vinjetiranja

* **Tip**: Potvrdni okvir
* **Zadano**: Omogućeno (označeno)
* **Opis**: Primjenjuje korekciju vinjetiranja za kompenzaciju zatamnjenja leće na rubovima slika. Vinjetiranje je uobičajeni optički fenomen u kojem kutovi i rubovi slike izgledaju tamniji od središta zbog karakteristika leće.
* **Kada onemogućiti**: Onemogućite ovu opciju samo ako je kombinacija fotoaparata i objektiva već primijenila korekciju vinjetiranja ili ako želite ručno ispraviti vinjetiranje u naknadnoj obradi.

### Kalibracija refleksije/balansa bijele boje

* **Tip**: Potvrdni okvir
* **Zadano**: Omogućeno (označeno)
* **Opis**: Omogućuje automatsku kalibraciju refleksije pomoću kalibracijskih ciljeva otkrivenih na slikama. Ovo normalizira vrijednosti refleksije u cijelom skupu podataka i osigurava dosljedna mjerenja bez obzira na svjetlosne uvjete.
* **Kada onemogućiti**: Onemogućite samo ako želite obraditi nekalibrirane slike ili ako koristite drugačiji tijek rada kalibracije.

### Demosaic metoda

* **Tip**: padajući odabir
* **Opcije**:
  * Visoka kvaliteta (brže) – trenutno jedina dostupna opcija
* **Zadano**: Visoka kvaliteta (brže)
* **Opis**: Odaberite demosaic algoritam koji se koristi za pretvaranje sirovih Bayerovih podataka senzora s uzorkom u slike u boji. Metoda "Visoke kvalitete (brže)" pruža optimalnu ravnotežu između brzine obrade i kvalitete slike.
* **Napomena**: Dodatne Debayer metode mogu se dodati u budućim verzijama Chlorosa.

### Minimalni interval rekalibracije

* **Tip**: Broj
* **Raspon**: 0 do 3600 sekundi
* **Zadano**: 0 sekundi
* **Opis**: Postavlja minimalni vremenski interval (u sekundama) između korištenja ciljeva kalibracije. Kada je postavljen na 0, Chloros će koristiti sve otkrivene kalibracijske ciljeve. Kada se postavi na višu vrijednost, Chloros će koristiti samo ciljeve kalibracije koji su odvojeni barem ovim brojem sekundi, smanjujući vrijeme obrade skupova podataka s čestim snimanjem ciljeva kalibracije.
* **Kada prilagoditi**:
  * Postavite vrijednost na 0 za maksimalnu točnost kalibracije kada se uvjeti osvjetljenja razlikuju.
  * Povećajte vrijednost (na primjer, na 60-300 sekundi) kako biste ubrzali obradu kada je osvjetljenje konstantno i dostupne su česte slike cilja kalibracije.

### Vremenski odmak svjetlosnog senzora

* **Tip**: Broj
* **Raspon**: -12 do +12 sati
* **Zadano**: 0 sati
* **Opis**: Određuje vremensku razliku (u satima u odnosu na UTC) za vremenske oznake podataka svjetlosnog senzora. Koristi se prilikom obrade PPK (naknadno obrađena kinematika) podatkovnih datoteka kako bi se osigurala ispravna sinkronizacija između snimljenih slika i GPS podataka.
* **Kada prilagoditi**: Postavite ovu vrijednost na vremensku razliku vaše lokalne vremenske zone ako vaši PPK podaci koriste lokalno vrijeme umjesto UTC. Na primjer:
  * Pacifičko vrijeme: -8 ili -7 (ovisno o ljetnom računanju vremena)
  * Istočno vrijeme: -5 ili -4 (ovisno o ljetnom računanju vremena)
  * Srednjoeuropsko vrijeme: +1 ili +2 (ovisno o ljetnom računanju vremena)

### Primijeni PPK popravke

* **Tip**: Potvrdni okvir
* **Zadano**: Onemogućeno (neoznačeno)
* **Opis**: Omogućuje korištenje naknadno obrađenih kinematičkih (PPK) korekcija DAQ MAPIR snimača koji sadrže GPS (GNSS). Kada je omogućen, Chloros će koristiti sve .daq datoteke dnevnika koje sadrže podatke o ekspoziciji u direktoriju vašeg projekta i primijeniti precizne ispravke geolokacije na vaše slike.
* **Zahtjev**: Mora postojati .daq log datoteka s unosima pribadača za izlaganje u direktoriju projekta
* **Kada omogućiti**: Preporučuje se uvijek omogućiti PPK korekciju ako postoje unosi povratnih informacija o izloženosti u .daq datoteci dnevnika.

### Ekspozicija Pin 1

* **Tip**: padajući odabir
* **Vidljivost**: Vidljivo samo kada je omogućena "Primijeni PPK korekcije" I podaci o izloženosti dostupni su za pin 1
* **Opcije**:
  * Nazivi modela kamera otkrivenih u projektu
  * "Ne koristiti": zanemarite ovu iglu za ekspoziciju
* **Zadano**: Automatski odabrano na temelju postavki projekta
* **Opis**: Dodjeljuje određenu kameru pinu 1 ekspozicije za PPK sinkronizaciju vremena. Oznaka ekspozicije bilježi točan trenutak kada se okidač kamere otpusti, što je ključno za točnu PPK geolokaciju.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedna igla: Automatski odaberite kameru
  * Jedna kamera + dvije igle: Pin 1 automatski se dodjeljuje kameri
  * Više kamera: potreban je ručni odabir

### Ekspozicija Pin 2

* **Tip**: padajući odabir
* **Vidljivost**: Vidljivo samo kada je omogućena "Primijeni PPK korekcije" I podaci o izloženosti dostupni su za pin 2
* **Opcije**:
  * Nazivi modela kamera otkrivenih u projektu
  * "Ne koristiti": zanemarite ovu iglu za ekspoziciju
* **Zadano** – automatski odabrano na temelju postavki projekta
* **Opis**: Dodjeljuje određenu kameru pinu 2 ekspozicije za PPK sinkronizaciju vremena kada se koristi postava dvostruke kamere.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedan pin: Pin 2 automatski se postavlja na "Ne koristi"
  * Jedna komora + dvije igle: Pin 2 automatski se postavlja na "Ne koristi"
  * Više kamera: potreban je ručni odabir
* **Napomena**: Ista kamera ne može se dodijeliti pinu 1 i pinu 2 istovremeno.***

## Indeks

Ove postavke omogućuju vam konfiguriranje multispektralnih indeksa za analizu i prikaz.

### Dodaj indeks

* **Tip**: Konfiguracijska ploča posebnih indeksa
* **Opis**: Otvara interaktivnu ploču na kojoj možete odabrati i konfigurirati multispektralne indekse vegetacije (NDVI, NDRE, EVI itd.) za izračun tijekom obrade slike. Možete dodati više indeksa, svaki sa svojim postavkama prikaza.
* **Dostupni indeksi**: Sustav uključuje više od 30 unaprijed definiranih multispektralnih indeksa, uključujući:
  * NDVI (normalizirani vegetacijski indeks razlike)
  * NDRE (Normalized Difference RedEdge)
  * EVI (poboljšani indeks vegetacije)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * I mnogo više (pogledajte [Formule multispektralnog indeksa](multispectral-index-formulas.md) za potpuni popis)
* **Značajke**:
  * Odaberite jednu od unaprijed definiranih formula indeksa
  * Konfigurirajte gradijente boja prikaza (LUT - Lookup Tables)
  * Postavite granične vrijednosti za analizu
  * Stvorite prilagođene formule indeksa

### Prilagođene formule (značajka Chloros+)

* **Tip**: niz definicija prilagođenih formula
* **Opis**: Omogućuje stvaranje i spremanje prilagođenih formula multispektralnog indeksa pomoću matematike pojasa. Prilagođene formule spremaju se s vašim postavkama projekta i mogu se koristiti kao ugrađeni indeksi.
* **Kako stvoriti**:
  1. Na ploči postavki indeksa pronađite opciju prilagođene formule.
  2. Definirajte svoju formulu pomoću identifikatora pojasa (na primjer, NIR, Red, Green, Blue).
  3. Spremite formulu s opisnim nazivom.
* **Sintaksa formule**: Podržane su standardne matematičke operacije, uključujući:
  * Aritmetika: `+`, `-`, `*`, `/`.
  * Zagrade za redoslijed operacija
  * Reference pojasa: NIR, crvena, zelena, plava, crveni rub, cijan, narančasta, NIR1, NIR2

***

## Izvoz

Ove postavke kontroliraju format i kvalitetu izvezenih obrađenih slika.

### Kalibrirani format slike

* **Tip**: padajući odabir
* **Opcije**:
  * **TIFF (16-bitni)**: Nekomprimirani 16-bitni TIFF format
  * **TIFF (32-bitni, postotak)**: 32-bitni TIFF s pomičnim zarezom i postotnim vrijednostima refleksije
  * **PNG (8-bitni)** - 8-bitni komprimirani PNG format.
  * **JPG (8-bitni)**: 8-bitni komprimirani JPEG format.
* **Zadano**: TIFF (16-bitni).
* **Opis**: Odaberite format datoteke za spremanje obrađenih i kalibriranih slika.
* **Preporuke za format**:
  * **TIFF (16-bitni)**: Preporučeno za znanstvene analize i profesionalne tijekove rada. Čuva maksimalnu kvalitetu podataka bez artefakata kompresije. Idealno za multispektralnu analizu i naknadnu obradu u GIS softveru.
  * **TIFF (32-bitni, postotak)**: Idealno za tijekove rada koji zahtijevaju vrijednosti refleksije u postotnom obliku (0-100%). Nudi maksimalnu preciznost za radiometrijska mjerenja.
  * **PNG (8-bitni)**: Idealan za web pregled i općenito gledanje. Manje veličine datoteka s kompresijom bez gubitaka, ali smanjenim dinamičkim rasponom.
  * **JPG (8-bitni)**: Manje veličine datoteka, idealne samo za preglede i web pregled. Koristi kompresiju s gubitkom, što nije prikladno za znanstvenu analizu.***

## Spremite predložak projekta

Ova vam značajka omogućuje spremanje trenutnih postavki projekta kao predloška za višekratnu upotrebu.

* **Tip**: unos teksta + gumb Spremi
* **Opis**: Unesite opisni naziv za svoj predložak konfiguracije i kliknite ikonu Spremi. Predložak će pohraniti sve trenutne postavke vašeg projekta (otkrivanje cilja, mogućnosti obrade, indekse i format izvoza) za jednostavnu ponovnu upotrebu u budućim projektima.
* **Slučajevi upotrebe**:
  * Stvorite predloške za različite sustave kamera (RGB, multispektralni, NIR)
  * Spremite standardne postavke za određene vrste usjeva ili tijek rada analize
  * Dijelite dosljedne postavke u cijelom timu
* **Kako ga koristiti**:
  1. Konfigurirajte sve postavke projekta koje želite
  2. Unesite naziv predloška (na primjer, "RedEdge Survey3 NDVI Standard").
  3. Pritisnite ikonu za spremanje.
  4. Sada se predložak može učitati prilikom izrade novih projekata.***

## Spremi mapu projekta

Ova postavka određuje gdje se novi projekti spremaju prema zadanim postavkama.

* **Tip**: Prikaz putanje direktorija + gumb Uredi
* **Zadano**: `C:\Users\[Username]\Chloros Projects`
* **Opis**: Prikazuje trenutni zadani direktorij u kojem se stvaraju novi Chloros projekti. Pritisnite ikonu za uređivanje da odaberete drugi direktorij.
* **Kada to promijeniti**:
  * Postavite mrežni pogon za timsku suradnju.
  * Prijeđite na pogon s više prostora za pohranu velikih skupova podataka.
  * Organizirajte projekte prema godini, klijentu ili vrsti projekta u različitim mapama.
* **Napomena**: Promjena ove postavke utječe samo na NOVE projekte. Postojeći projekti ostaju na svojim izvornim lokacijama.***## Postojanost konfiguracije

Sve postavke projekta automatski se spremaju s datotekom projekta (format projekta __INLINE0006___). Kada ponovno otvorite projekt, sve postavke će se vratiti točno onakve kakve ste ih ostavili.

### Hijerarhija postavki

Postavke se primjenjuju sljedećim redoslijedom:

1. **Zadane postavke sustava**: Ugrađene zadane postavke koje definira Chloros
2.**Postavke predloška**: Ako učitate predložak prilikom izrade projekta
3.**Postavke projekta spremljene**: Postavke spremljene s datotekom projekta
4.**Ručne postavke**: sve promjene koje napravite tijekom trenutne sesije

### Postavljanje i obrada slike

Većina promjena postavki (osobito u kategorijama Renderiranje i Izvoz) uzrokovat će ponovnu obradu slika kako bi odražavale nove postavke. Međutim, neke su postavke "samo za izvoz" i ne zahtijevaju trenutnu ponovnu obradu:

* Spremite predložak projekta
* Radni imenik
* Kalibrirani format slike (primjenjuje se prilikom izvoza)

***## Najbolji primjeri iz prakse

1.**Počnite sa zadanim postavkama**– Zadane postavke dobro funkcioniraju za većinu MAPIR sustava kamera i tipičnih radnih procesa.
2.**Stvorite predloške**– Nakon što ste optimizirali postavke za određeni tijek rada ili kameru, spremite ih kao predložak kako biste osigurali dosljednost između projekata.
3.**Testirajte prije potpune obrade**: Kada eksperimentirate s novim postavkama, testirajte ih na malom podskupu slika prije obrade cijelog skupa podataka.
4.**Dokumentirajte svoje postavke**: Koristite opisne nazive predložaka koji označavaju sustav kamere, vrstu obrade i namjeravanu upotrebu (na primjer, “Survey3\_RGB\_NDVI\_Agriculture”).
5.**Odabir formata izvoza**: Odaberite format izvoza na temelju krajnje upotrebe:
   * Znanstvena analiza → TIFF (16 bit ili 32 bit)
   * GIS obrada → TIFF (16 bita)
   * Brzi pregled → PNG (8 bita)
   * Podijelite na webu → JPG (8 bita)

***

Za više informacija o multispektralnim indeksima u Chlorosu, pogledajte stranicu [Formule multispektralnog indeksa](multispectral-index-formulas.md).