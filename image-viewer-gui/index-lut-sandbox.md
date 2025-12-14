# Index/LUT Sandbox

Index/LUT Sandbox je interaktivni radni prostor unutar Chloros Image Viewera koji vam omogućuje eksperimentiranje s multispektralnim izračunima indeksa i vizualizacijama boja u stvarnom vremenu. Ovaj moćni alat pomaže vam u testiranju različitih indeksa, pročišćavanju raspona vrijednosti i stvaranju vizualizacija spremnih za objavljivanje bez ponovne obrade cijelog skupa podataka.

## Što je Index/LUT Sandbox?

### Svrha

Sandbox pruža:

* **Izračun indeksa u stvarnom vremenu** - Trenutačno primijenite bilo koji indeks vegetacije
* **Interaktivna prilagodba LUT-a** - Fino podešavanje prijelaza i raspona boja
* **Optimizacija tijeka rada**- Odredite najbolje postavke prije skupne obrade

### Sandbox naspram Project Processing**Index/LUT Sandbox (Interaktivno):**

* Jedna po jedna slika
* Instant povratne informacije
* Eksperimentalno i iterativno
* Nema trajnih promjena datoteka
* Savršeno za istraživanje i testiranje

**Obrada projekta (serija):**

* Cijeli skup podataka odjednom
* Unaprijed konfigurirane postavke
* Trajne izlazne datoteke
* Vremenski intenzivan
* Najbolje kada su postavke finalizirane

{% hint style="success" %}
**Najbolji tijek rada**: Koristite Sandbox za eksperimentiranje i pronalaženje optimalnih postavki indeksa i LUT-a, a zatim primijenite te postavke tijekom obrade projekta za cijeli skup podataka.
{% endhint %}***

## Rad s Index/LUT Sandboxom

### Razumijevanje unaprijed izračunatih indeksa

U Chlorosu se indeksi mogu primijeniti tijekom obrade projekta. Da biste odredili koje postavke indeksa i LUT želite primijeniti na izvoze, najlakše je koristiti sandbox preglednika slika.

Sandbox vam omogućuje sljedeće:

* **Primijenite novi indeks i gradijente boja (LUT)** za vizualizaciju podataka
* **Interaktivno prilagodite postavke vizualizacije**
* **Pogledajte** već izračunate indeksne slike
* **Provjerite**vrijednosti piksela na svim razinama zumiranja

### Otvaranje Sandboxa

Index/LUT Sandboxu se pristupa u**Image Vieweru**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">kartica bočne trake:

1. Pritisnite sliku u rešetki slika preglednika datoteka, ona se otvara u**Pregledniku slika**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">tab
2. Kliknite**preglednik slika**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">karticu za otvaranje lijeve iskačuće bočne trake ako već nije otvorena

### Odabir slike na koju će se primijeniti indeks/LUT

Za rad s indeksom u pregledniku slika<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">sandbox:

1.**Otvorite sliku**iz mreže glavne slike klikom na nju
2.**Preglednik slika**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">otvorit će se kartica
3. Kliknite**Padajući izbornik slojeva** (gore desno od preglednika)
4. Odaberite sloj s padajućeg izbornika:
* RAW (refleksija)

### Primjena indeksa na sliku

Nakon što je slika na cijelom zaslonu i **Preglednik slika**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">bočna traka kartice je otvorena:

1. Označite okvir Index na vrhu bočne trake
2. Odaberite filtar svoje kamere s lijevog padajućeg izbornika
3. Odaberite željenu formulu indeksa s desnog padajućeg izbornika
4. Povucite krugove boja kanala filtra na mjesta u donjoj formuli indeksa
5. Nakon što je formula valjana, slika će se ažurirati i prikazati vrijednosti indeksa
6. Pomičite pokazivač miša kako biste vidjeli vrijednosti na mjestu pokazivača
7. Povećajte kako biste vidjeli pojedinačne piksele i njihove pridružene vrijednosti

Svaki indeks ima određeni raspon vrijednosti i značenje:

#### Primjer NDVI

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Za potpunu dokumentaciju formule indeksa, pogledajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md).***## Rad s LUT-ovima (pretraživačke tablice)

### Što je LUT?

**Look-Up Table (LUT)** preslikava numeričke vrijednosti indeksa u boje za vizualizaciju:

* **Ulaz**: vrijednost indeksnog piksela (npr. NDVI 0,65)
* **Izlaz**: RGB boja (npr. svijetlo zelena)
* **Svrha**: učiniti uzorke lakšim za vidjeti i protumačiti**Nijanse sive naspram LUT u boji:**

* Sivi tonovi: znanstveno i neutralno, prikazuje neobrađene podatke
* LUT u boji: Intuitivno i dojmljivo, naglašava uzorke i razlike

{% hint style="success" %}
**Snaga vizualizacije**: Primjena LUT-a u boji na indeksnu sliku u sivim tonovima dramatično olakšava prepoznavanje uzoraka, anomalija i područja od interesa na prvi pogled.
{% endhint %}

### Primjena LUT-a na indeksnu sliku

Nakon što se prikaže indeksna slika

1. Kliknite na<img src="../.gitbook/assets/image.png" alt="" data-size="line">

Gumb "+Dodaj LUT".
2. Odaberite gradijent boje
3. Podesite minimalne/maksimalne krajnje točke izrezivanja
4. Podesite način izrezivanja
5. Označite okvir Index u**Pregledniku slika**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">bočnu traku kartice za primjenu LUT-a

### Odabir gradijenta boje**Odabir gradijenta:**1. Na ploči LUT pronađite**traku gradijenta u boji**2. Zadržite pokazivač miša iznad njega da biste vidjeli dostupne unaprijed postavljene gradijentne postavke
3. Odaberite željeni gradijent
4. Slika se**odmah ažurira**s novim bojama kada je okvir Index označen

{% hint style="success" %}**Najbolja praksa**: za indekse vegetacije kao što je NDVI, gradijent crveno-žuto-zeleno je najintuitivniji jer se usklađuje s prirodnim asocijacijama boja (zeleno=zdravo, žuto=umjereno, crveno=pod stresom).
{% endhint %}

### Podešavanje klasa boja**Kontrola klasa**određuje koliko će se diskretnih koraka boja pojaviti u vašem gradijentu:**Mogućnosti brojanja razreda:**

* **2-5 razreda**: Vrlo široke kategorije, različite zone
* **6-10 klasa**: Uravnotežen, dobar za klasifikaciju
* **11-20 razredi**: glatki gradijenti, kontinuirani izgled
* **20+ klasa**: Gotovo kontinuirano, maksimalna glatkoća**Kako se prilagoditi:**1. Na ploči LUT pronađite**kvadrate uzorka boja ispod trake gradijenta**2. Podesite broj razreda dodavanjem pomoću gumba +
3. Uklonite broj klasa dvostrukim klikom na uzorak boje
4. Gradijent se ažurira**u stvarnom vremenu**na slici**Učinak na vizualizaciju:**

* **Manje klasa** (3-5): Stvara različite zone, pojednostavljenu klasifikaciju, lakše razlikovati kategorije
* **Srednje klase** (6-10): Uravnotežen pristup, dobar za većinu primjena
* **Više razreda**(15-20): glatki prijelazi, detaljne varijacije, fotografski izgled**Kada koristiti:**

* **Malo razreda (3-5)**: Prezentacijske slajdove, klasifikacijske karte, jednostavna izvješća
* **Srednji razredi (6-10)**: Opća analiza, uravnoteženi detalji, standardna izvješća
* **Mnogo razreda (15-20)**: znanstvena analiza, detaljna inspekcija, rezultati kvalitete publikacije

### Fino podešavanje raspona vrijednosti**Kontrole raspona vrijednosti**određuju koje se vrijednosti indeksa preslikavaju na koje boje u vašem gradijentu:**Kontrole raspona u LUT ploči:**

* **Minimalna vrijednost**: Donja granica ljestvice boja
* **Maksimalna vrijednost**: Gornja granica ljestvice boja
* **Međuvrijednosti**: Automatski se raspoređuje između min. i maks. (na temelju broja razreda)

#### Podešavanje minimalnih/maksimalnih vrijednosti**Za podešavanje raspona vrijednosti:**1. Na ploči LUT pronađite polja za unos**Minimalna vrijednost**i**Maksimalna vrijednost**2. Pritisnite polje**Minimalna vrijednost**3. Upišite željenu minimalnu vrijednost (npr.`0.2`)
4. Pritisnite**Enter**ili kliknite izvan polja
5. Ponovite za polje**Maksimalna vrijednost**(npr.`0.9`)
6. Vizualizacija se**odmah ažurira**{% hint style="info" %}**Automatsko skaliranje**: Kada prvi put primijenite LUT, Chloros automatski postavlja min/max na stvarni raspon podataka na slici. Zatim možete suziti ovaj raspon kako biste se usredotočili na određene raspone vrijednosti od interesa.
{% endhint %}**Primjer podešavanja raspona NDVI:**

* **Cijeli asortiman**:`-1.0` to `1.0`(prikaži sve moguće vrijednosti)
* **Fokusiran na vegetaciju**:`0.2` to `0.9`(isključuje golo tlo i vodu)
* **Samo zdrava vegetacija**:`0.5` to `0.9`(istaknite samo jake biljke)
* **Otkrivanje stresa**:`0.2` to `0.5`(naglasiti problematična područja)
* **Prilagođeni raspon**: prilagodite na temelju opaženih vrijednosti piksela**Zašto prilagođavati raspone?**

* **Pojačajte kontrast** u području koje vas zanima
* **Isključi irelevantne vrijednosti** (npr. vodena tijela, golo tlo)
* **Standardizirajte vizualizaciju** na više slika ili datuma
* **Naglasite suptilne razlike**unutar uskog raspona vrijednosti

### Izrezivanje vrijednosti izvan raspona

Kada vrijednosti piksela padnu izvan vašeg definiranog minimalnog/maksimalnog raspona, možete kontrolirati kako se one prikazuju pomoću**načina izrezivanja**.

####**Dostupne opcije načina isjecanja:**

#### 1. Minimum i Maksimum

* Pikseli **ispod minimuma**→ prikaz koristeći**prvu boju** u gradijentu (npr. crvena)
* Pikseli **iznad maksimuma**→ prikaz pomoću**zadnje boje** u gradijentu (npr. zelena)
* **Slučaj upotrebe**: Naglasite ekstreme, prikažite cijeli raspon podataka sa zasićenim bojama na granicama
* **Primjer**: NDVI vrijednosti ispod 0,2 sve su crvene, vrijednosti iznad 0,9 sve su zelene

#### 2. Prozirna pozadina

* Pikseli **izvan raspona**postaju**potpuno prozirni**
* Samo pikseli **unutar raspona** prikazuju gradijent boje
* **Slučaj upotrebe**: GIS preklapanje, izdvajanje određenih raspona vrijednosti, isticanje samo područja od interesa
* **Primjer**: Prikaži samo NDVI 0.4-0.7 u boji, sve ostalo prozirno

{% hint style="warning" %}**Ograničenje prozirnosti**: prozirni pikseli pojavit će se kao boja pozadine u pregledniku. Prilikom izvoza tijekom obrade, prozirnost se čuva u PNG formatu, ali ne i u JPG.
{% endhint %}

#### 3. Pozadina indeksa

* Pikseli **izvan raspona**prikazuju se u**sivim tonovima** (prikazuju sirove vrijednosti indeksa)
* Pikseli **unutar raspona**pokazuju**gradijent boje**
* **Slučaj upotrebe**: Suptilno isticanje, održavanje konteksta uz naglašavanje područja interesa
* **Primjer**: bojom označite vegetaciju pod stresom (NDVI 0,3-0,5) dok zdrava područja prikazujete sivom bojom

#### 4. Izvorna pozadina

* Pikseli **izvan raspona**prikazuju**izvornu multispektralnu sliku**
* Pikseli **unutar raspona**pokazuju**gradijent boje**
* **Slučaj upotrebe**: Najintuitivniji - kombinira prirodni kontekst slike s analitičkim slojem boja
* **Primjer**: Pogledajte stvarni izgled polja/usjeva s označenim bojama označenim područjima stresa

### Odabir pravog načina izrezivanja

| Način izrezivanja              | Najbolje za                                   | Stil vizualizacije          |
| --------------------------- | ------------------------------------------------ | ---------------------------- |
|**Minimum i Maksimum**| Prikaz potpunih podataka, znanstvena analiza     | Svi pikseli u boji           |
|**Prozirna pozadina**| GIS preklapanja, izdvajanje specifičnih raspona    | Boja u rasponu, prazno iznad |
|**Pozadina indeksa**| Suptilan naglasak, održavanje konteksta podataka  | Boja u rasponu, siva izvan nje  |
|**Originalna pozadina**| Izvješća, prezentacije, intuitivne analize | Boja u rasponu, fotografija izvan nje |

### Stvaranje prilagođenih LUT boja

Za potpunu kontrolu nad svojom vizualizacijom, možete stvoriti**prilagođene gradijente boja**uređivanjem pojedinačnih točaka boja.**Za izradu prilagođenog gradijenta:**1. Na ploči LUT pronađite**traku pregleda gradijenta**2. Potražite**kvadrate uzorka boje**ispod gradijenta
3.**Kliknite graničnik u boji**da biste ga odabrali
4. Otvara se**birač boja**
5. Odaberite novu boju pomoću:
* **Kotačić u boji**: Vizualni odabir boja
* **RGB/HSV klizači**: Precizna kontrola boja
* **Unos heksadecimalnog koda**: Točna specifikacija boje (npr.`#FF0000`za crveno)
6. Kliknite na birač boja**za primjenu nove boje**7. Gradijent se**odmah ažurira**na slici**Dodavanje ili uklanjanje boja zaustavlja:**

* **Dodajte stanicu**: Kliknite ikonu + da dodate novi uzorak na kraju
* **Uklonite stop**: Dvaput kliknite na kvadratić u boji da uklonite uzorak**Strategije prilagodbe:**

* **Obrnuti gradijent**: Promijenite redoslijed boja da biste obrnuli značenje (npr. zeleno=nisko, crveno=visoko)
* **Boje robne marke**: uskladite paletu boja vaše organizacije za izvješća
* **Bez daltonista**: Koristite narančasto-plave ili ljubičasto-žute kombinacije
* **Optimizacija ispisa**: Odaberite boje koje funkcioniraju i pri ispisu u boji i u sivim tonovima
* **Više pragova**: Koristite različite boje na određenim pragovima vrijednosti za klasifikaciju

{% hint style="info" %}**Spremanje prilagođenih gradijenata**: prilagođeni gradijenti se mogu spremiti i ponovno koristiti. Pritisnite ikonu za spremanje na ploči LUT da biste sačuvali prilagođene sheme boja za buduću upotrebu.
{% endhint %}***## Interaktivni tijek rada

### Ažuriranja u stvarnom vremenu

Sve LUT prilagodbe u sandboxu ažuriraju sliku **trenutačno i interaktivno**:

* **Promijeni sloj** → Slika se odmah mijenja
* **Odaberi gradijent** → Boje se trenutno ažuriraju
* **Prilagodi raspon vrijednosti** → Kontrast se mijenja u stvarnom vremenu
* **Promijeni klase** → Glatkoća gradijenta ažurira se odmah
* **Izmijeni isječak** → Pozadinski prikaz se trenutno mijenja
* **Uredite boje**→ Prilagođeni gradijent primjenjuje se odmah**Nije potreban gumb "Primijeni"**- sve promjene su aktivne i interaktivne!

{% hint style="success" %}**Povratne informacije uživo**: trenutne vizualne povratne informacije omogućuju vam brzo eksperimentiranje s različitim postavkama dok ne pronađete optimalnu vizualizaciju za svoje potrebe analize.
{% endhint %}

### Tijek rada iterativnog usavršavanja**Tipični tijek rada optimizacije LUT-a:**1.**Odaberite indeksni sloj**(npr., RAW (Reflectance))
2.**Primijeni indeks**- Odaberite filtar kamere i formulu indeksa, povucite krugove u boji na odgovarajuće mjesto u formuli indeksa
3.**Primijeni LUT gradijent**- Započnite s unaprijed postavljenim crveno-žuto-zelenim
4.**Provjerite vrijednosti piksela**- Pomičite kursor, zabilježite raspone vrijednosti
5.**Prilagodi min/max**- suzi za fokusiranje na vegetaciju (npr. 0,2 do 0,9)
6.**Odaberite isječak**- pokušajte s "Izvornom pozadinom" za kontekst
7.**Pročistite boje**- Prilagodite gradijent ako je potrebno za određeni naglasak
8.**Dovršite postavke**- Postavke dokumenta i kopirajte u Postavke projekta za obradu izvoza

### Provjera vrijednosti piksela

Razumijevanje stvarnih vrijednosti piksela ključno je za postavljanje efektivnih LUT raspona:**Kako provjeriti vrijednosti:**1. Vrijednosti piksela prikazuju se kada slika ima**potvrđene**okvire Index ili Index i LUT.
2.**Pomičite kursor**preko različitih područja slike
3.**Promatrajte vrijednosti piksela**prikazane u legendi dok lebdite
4. Povećajte kako biste vidjeli pojedinačne piksele istaknute plutajućom vrijednošću
5.**Zabilježite** raspone vrijednosti za različite značajke:
* **Zdrava vegetacija**: npr. NDVI 0,55-0,85
* **Vegetacija pod stresom**: npr. NDVI 0,30-0,50
* **Golo tlo**: npr. NDVI 0,05-0,25
* **Voda**(ako postoji): npr. NDVI -0,05 do 0,10**Korištenje vrijednosti piksela za postavljanje LUT raspona:**Nakon provjere vrijednosti piksela, prilagodite svoj LUT min/max u skladu s tim:**Primjer scenarija:**

* **Opažanje**: Vrijednosti tla = 0,05-0,25, Opterećeno = 0,25-0,50, Zdravo = 0,50-0,85
* **Cilj**: Vizualizirajte samo zdravlje biljaka (isključite tlo)
* **Postavke LUT-a**: Min =`0.25`, Max =`0.85`
* **Izrezivanje**: "Izvorna pozadina" za prikaz tla u prirodnoj boji
* **Rezultat**: Gradijent boja odnosi se samo na vegetaciju, tlo se prikazuje kao izvorna slika

{% hint style="info" %}**Dinamički raspon**: različiti usjevi, godišnja doba i faze rasta imat će različite raspone vrijednosti. Uvijek provjerite vrijednosti piksela u vašem određenom skupu podataka prije postavljanja LUT raspona.
{% endhint %}***## Prilagođeni indeksi (Chloros+)

### Stvaranje prilagođenih formula indeksa

{% hint style="info" %}
**Gdje stvoriti**: Prilagođeni indeksi mogu se konfigurirati u**Postavke projekta**prije obrade, kao i na bočnoj traci sandboxa preglednika slika.
{% endhint %}**Za izradu prilagođenog indeksa:**1.**Otvorite postavke projekta**(prije obrade) ili bočnu traku sandboxa preglednika slika
2. Dođite do**Padajućeg izbornika formule indeksa**3. Potražite opciju**"Custom"**(morate se prijaviti s Chloros+ licencom)
4.**Definirajte svoju formulu** koristeći varijable pojasa:
   * Band names: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operators: `+`, `-`, `*`, `/`, `^` (exponent)
* Funkcije:`sqrt()`, `abs()`, itd. (ako je podržano)
* Zagrade:`()`za redoslijed operacija
5. **Imenujte svoj indeks**(npr. "MyIndex" ili "CustomNDVI")
6.**Spremite konfiguraciju**
**Primjer prilagođenih formula:**```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}**Provjera formule**: Provjerite koristi li vaša formula trake dostupne u vašem fotoaparatu. Na primjer, RedEdge je dostupan samo na fotoaparatima s RedEdge filterom.
{% endhint %}***

## Sljedeći koraci

Sada kada razumijete Index/LUT Sandbox:

* **Primijeni na obradu**: Koristite otkrivene postavke u [Postavke projekta](../project-settings/project-settings.md)
* **Skupni proces**: Primijenite optimizirane indekse na pune skupove podataka
* **Saznajte više**: Pročitajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md)

Povezana dokumentacija:

* [**Slojevi slika**](image-layers.md) - Upravljanje slojevima i vizualizacija
* [**Otvaranje slike preko cijelog zaslona**](opening-an-image-full-screen.md) - Osnove preglednika slika
* [**Obrada slika (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Potpuni tijek obrade
