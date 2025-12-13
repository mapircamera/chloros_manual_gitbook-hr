# Index/LUT sandbox

Index/LUT Sandbox je interaktivni radni prostor unutar Chloros Image Viewera koji vam omogućuje eksperimentiranje s multispektralnim izračunima indeksa i vizualizacijama boja u stvarnom vremenu. Ovaj moćni alat pomaže vam u testiranju različitih indeksa, pročišćavanju raspona vrijednosti i stvaranju vizualizacija spremnih za objavljivanje bez potrebe za ponovnom obradom čitavog skupa podataka.

## Što je Index/LUT Sandbox?

### Svrha

Sandbox nudi:

* **Izračun indeksa u stvarnom vremenu**: Trenutačno primijenite bilo koji indeks vegetacije.
* **Interaktivna LUT prilagodba**: fino podešavanje gradijenata i raspona boja.
* **Optimizacija tijeka rada**: Odredite najbolje postavke prije skupne obrade.

### Sandbox naspram Project Processing

**Indeks/LUT sandbox (interaktivan):**

* Jedna po jedna slika
* Instant odgovor
* Eksperimentalno i iterativno
* Nema trajnih promjena datoteka
* Savršeno za istraživanje i testiranje

**Obrada projekta (serija):**

* Cijeli skup podataka odjednom.
* Prethodno konfigurirane postavke.
* Trajne izlazne datoteke.
* Zahtijeva puno vremena.
* Idealno kada su postavke dovršene.

{% hint style=&quot;uspjeh&quot; %}
**Bolji tijek rada**: Koristite sandbox za eksperimentiranje i pronalaženje optimalnih postavki indeksa i LUT-a, a zatim primijenite te postavke tijekom obrade projekta za cijeli skup podataka.
{% endhint %}

***

## Rad s indeksom/LUT sandboxom

### Razumijevanje unaprijed izračunatih indeksa

U Chlorosu se indeksi mogu primijeniti tijekom obrade projekta. Da biste odredili koje postavke indeksa i LUT želite primijeniti na svoje izvoze, najlakše je koristiti sandbox preglednika slika.

Sandbox vam omogućuje sljedeće:

* **Primijenite nove indekse i gradijente boja (LUT)** za vizualizaciju podataka.
* **Interaktivno prilagodite postavke zaslona**.
* **Pogledajte** slike već izračunatih indeksa.
* **Provjerite** vrijednosti piksela na svim razinama zumiranja.

### Otvorite testnu okolinu

Sandboxu indeksa/LUT pristupa se na kartici **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">:

1. Kliknite na sliku u rešetki slika u pregledniku datoteka i ona će se otvoriti u kartici **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">.
2. Pritisnite karticu **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> da biste otvorili skočnu bočnu traku s lijeve strane ako već nije otvorena

### Odaberite sliku na koju želite primijeniti indeks/LUT

Za rad s indeksom u Sandboxu preglednika slika <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">:

1. **Otvorite sliku** iz mreže glavne slike klikom na nju
2. Otvorit će se kartica **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">.
3. Pritisnite **Padajući izbornik slojeva** (gore desno od preglednika).
4. Odaberite sloj iz padajućeg izbornika:
   * RAW (refleksija)

### Primjena indeksa na sliku

Nakon što je slika na cijelom zaslonu i bočna traka kartice **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> otvorena je:

1. Označite okvir Index na vrhu bočne trake.
2. Odaberite filtar kamere iz padajućeg izbornika s lijeve strane.
3. Odaberite željenu formulu indeksa iz padajućeg izbornika s desne strane.
4. Povucite krugove boja kanala filtra na mjesta u donjoj formuli indeksa.
5. Nakon što je formula važeća, slika će se ažurirati i prikazati vrijednosti indeksa.
6. Pomaknite kursor miša da vidite vrijednosti na mjestu kursora.
7. Povećajte sliku kako biste vidjeli pojedinačne piksele i njihove pridružene vrijednosti.

Svaki indeks ima određeni raspon vrijednosti i značenja:

#### NDVI primjer

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

Za potpunu dokumentaciju formule indeksa, pogledajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md).

***

## Rad s LUT-ovima (tablice pretraživanja)

### Što je LUT?

**Tablica pretraživanja (LUT)** preslikava vrijednosti numeričkog indeksa u boje za prikaz:

* **Ulaz**: Vrijednost indeksnog piksela (npr. NDVI 0,65)
* **Izlaz**: RGB boja (npr. svijetlo zelena)
* **Svrha**: olakšati vizualizaciju i interpretaciju uzoraka

**LUT u sivim tonovima u odnosu na LUT u boji:**

* Sivi tonovi: znanstveno i neutralno, prikazuje neobrađene podatke
* LUT u boji: intuitivan i dojmljiv, naglašava uzorke i razlike

{% hint style=&quot;uspjeh&quot; %}
**Vidljivost** – Primjena LUT-a u boji na indeksnu sliku u sivim tonovima olakšava prepoznavanje uzoraka, anomalija i područja interesa na prvi pogled.
{% endhint %}

### Primjena LUT-a na indeksnu sliku

Nakon što imate indeksnu sliku koja se prikazuje

1. Kliknite gumb <img src="../.gitbook/assets/image.png" alt="" data-size="line"> «+Dodaj LUT»
2. Odaberite gradijent boje
3. Postavite minimalne/maksimalne krajnje točke usjeva
4. Podesite način obrezivanja
5. Označite okvir Index na bočnoj traci **Preglednika slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> da biste primijenili LUT.

### Odabir gradijenta boje

**Odabir gradijenta:**

1. Na ploči LUT pronađite **traku s gradijentom boja**.
2. Zadržite pokazivač iznad da biste vidjeli dostupne unaprijed postavljene gradijente.
3. Odaberite željeni gradijent.
4. Slika se **trenutačno ažurira** s novim bojama kada je okvir Index označen.

{% hint style=&quot;uspjeh&quot; %}
**Najbolja praksa**: Za vegetacijske indekse kao što je NDVI, crveno-žuto-zeleni gradijent je najintuitivniji jer slijedi prirodne asocijacije boja (zeleno = zdravo, žuto = umjereno, crveno = naglašeno).
{% endhint %}

### Postavljanje klasa boja

**Kontrola klasa** određuje koliko će se diskretnih koraka boja pojaviti u gradijentu:

**Opcije brojanja razreda:**

* **2-5 razreda**: vrlo široke kategorije, različita područja.
* **6-10 klasa**: uravnoteženo, dobro za klasifikaciju.
* **11-20 klasa**: glatki prijelazi, kontinuirani izgled.
* **Više od 20 klasa**: gotovo neprekidno, maksimalna mekoća.

**Kako se prilagoditi:**

1. Na ploči LUT pronađite **kvadratiće uzoraka boja ispod trake gradijenta**.
2. Podesite broj klasa zbrajanjem pomoću gumba +.
3. Izbrišite broj klasa dvostrukim pritiskom na uzorak boje.
4. Gradijent se ažurira u **stvarnom vremenu** na slici.

**Učinak na zaslonu:**

* **Manje klasa** (3-5): stvaranje diferenciranih područja, pojednostavljena klasifikacija, lakše razlikovanje kategorija.
* **Srednje klase** (6-10): uravnotežen pristup, pogodan za većinu primjena.
* **Više klasa** (15-20): glatki prijelazi, detaljne varijacije, fotografski izgled.

**Kada ga koristiti:**

* **Malo razreda (3-5)**: slajdovi prezentacije, mape rangiranja, jednostavna izvješća.
* **Srednji razredi (6-10)**: opća analiza, uravnoteženi detalji, standardna izvješća.
* **Mnogo predavanja (15-20)**: znanstvena analiza, detaljan pregled, rezultati kvalitete publikacije.

### Postavljanje raspona vrijednosti

**Kontrole raspona vrijednosti** određuju koje se vrijednosti indeksa preslikavaju na koje boje u gradijentu:

**Kontrole raspona na LUT ploči:**

* **Minimalna vrijednost**: donja granica ljestvice boja.
* **Maksimalna vrijednost**: gornja granica ljestvice boja.
* **Međuvrijednosti**: automatski se raspoređuju između minimuma i maksimuma (na temelju broja klasa).

#### Postavljanje minimalnih/maksimalnih vrijednosti

**Za podešavanje raspona vrijednosti:**

1. Na ploči LUT pronađite polja za unos **Minimalna vrijednost** i **Maksimalna vrijednost**.
2. Pritisnite polje **Minimalna vrijednost**.
3. Upišite minimalnu željenu vrijednost (na primjer, `0.2`).
4. Pritisnite **Enter** ili kliknite izvan polja.
5. Ponovite postupak za polje **Maksimalna vrijednost** (na primjer, `0.9`).
6. Zaslon se **odmah ažurira**.

{% hint style=&quot;info&quot; %}
**Automatsko skaliranje** – Kada se LUT primijeni prvi put, Chloros automatski postavlja min/max na stvarni raspon podataka slike. Zatim možete suziti ovaj raspon kako biste se usredotočili na određene raspone vrijednosti od interesa.
{% endhint %}

**Primjer postavki raspona NDVI:**

* **Cijeli raspon**: `-1.0` do `1.0` (prikaži sve moguće vrijednosti)
* **Usredotočeno na vegetaciju**: `0.2` do `0.9` (isključuje golo tlo i vodu)
* **Samo zdrava vegetacija**: `0.5` do `0.9` (istaknite samo jake biljke)
* **Provjera stresa**: `0.2` do `0.5` (naglasite problematična područja)
* **Prilagođeni raspon** – Prilagodite na temelju promatranih vrijednosti piksela

**Zašto prilagođavati raspone?**

* **Pojačajte kontrast** u području interesa
* **Isključite irelevantne vrijednosti** (npr. vodene površine, golo tlo)
* **Standardizirajte prikaz** na više slika ili datuma
* **Istaknite suptilne razlike** unutar uskog raspona vrijednosti

### Izrezivanje vrijednosti izvan raspona

Kada vrijednosti piksela padnu izvan definiranog minimalnog/maksimalnog raspona, možete kontrolirati kako se one prikazuju pomoću **načina izrezivanja**.

#### **Dostupne opcije načina obrezivanja:**

#### 1. Minimum i maksimum

* Pikseli **ispod minimuma** → prikaz pomoću **prve boje** gradijenta (npr. crvena)
* Pikseli **iznad maksimuma** → prikaz pomoću **zadnje boje** gradijenta (npr. zelena)
* **Slučaj upotrebe**: Naglasite ekstreme, prikažite cijeli raspon podataka sa zasićenim bojama na granicama
* **Primjer**: NDVI vrijednosti ispod 0,2 prikazane su crvenom bojom, vrijednosti iznad 0,9 prikazane su zelenom bojom

#### 2. Prozirna pozadina

* Pikseli **izvan dometa** postaju **potpuno prozirni**
* Samo pikseli **unutar raspona** prikazuju gradijent boje.
* **Slučaj upotrebe**: GIS preklapanje, izolacija određenih raspona vrijednosti, isticanje samo područja od interesa.
* **Primjer**: Prikaži samo NDVI 0.4-0.7 u boji, sve ostalo prozirno.

{% hint style=&quot;upozorenje&quot; %}
**Ograničenje prozirnosti**: prozirni pikseli pojavit će se kao boja pozadine u pregledniku. Prilikom izvoza tijekom obrade, prozirnost se čuva u PNG formatu, ali ne i u JPG.
{% endhint %}

#### 3. Indeksni fond

* Pikseli **izvan raspona** prikazani su u **sivim tonovima** (prikazujući sirove vrijednosti indeksa).
* Pikseli **unutar raspona** prikazuju **gradijent boje**.
* **Slučaj upotrebe**: Suptilno isticanje, održava kontekst uz naglašavanje područja interesa.
* **Primjer**: ističe vegetaciju pod stresom bojom (NDVI 0,3-0,5) i prikazuje zdrava područja sivom bojom.

#### 4. Izvorna pozadina

*Pikseli **izvan raspona** prikazani su na **izvornoj multispektralnoj slici**.
*Pikseli **unutar dometa** prikazuju **gradijent boje**
* **Slučaj upotrebe**: Najintuitivniji: kombinira prirodni kontekst slike s analitičkim slojem boja
* **Primjer**: pogledajte kako polje/usjev zapravo izgleda s naglašenim područjima prekrivenim i označenim bojama

### Odaberite pravi način obrezivanja

| Način izrezivanja | Idealno za | Stil prikaza |
| -------------------------------- | ------------------------------------------------ | ---------------------------- |
| **Minimalno i maksimalno** | Potpuna vizualizacija podataka, znanstvena analiza | Svi pikseli u boji |
| **Prozirna pozadina** | GIS slojevi, izolacija specifičnih raspona | Boja u rasponu, prazno izvan njega |
| **Indeksni fond** | Suptilan naglasak, održavanje konteksta podataka | Boja u rasponu, siva izvan |
| **Originalna pozadina** | Izvješća, prezentacije, intuitivna analiza | Boja u rasponu, fotografija izvan |

### Stvaranje prilagođenih LUT boja

Za potpunu kontrolu nad zaslonom, možete stvoriti **prilagođene gradijente boja** uređivanjem pojedinačnih točaka boja.

**Za izradu prilagođenog gradijenta:**

1. Na ploči LUT pronađite **traku pregleda gradijenta**.
2. Potražite **kvadrate uzorka boja** ispod gradijenta.
3. **Kliknite na graničnik u boji** da biste ga odabrali.
4. Otvorit će se **birač boja**.
5. Odaberite novu boju pomoću:
   * **Kotač boja**: vizualni odabir boja.
   * **RGB/HSV klizači**: Precizna kontrola boja.
   * **Unos heksadecimalnog koda**: Točna specifikacija boje (npr. `#FF0000` za crvenu).
6. Kliknite izvan birača boja **za primjenu nove boje**.
7. Gradijent se **odmah ažurira** na slici.

**Dodajte ili uklonite graničnike u boji:**

* **Dodajte stanicu**: kliknite na ikonu + da dodate novi uzorak na kraj.
* **Brisanje točke**: Dvaput kliknite obojeni kvadrat za brisanje uzorka.

**Strategije prilagodbe:**

* **Invertiraj gradijent**: Obrnite redoslijed boja da biste obrnuli značenje (npr. zelena = niska, crvena = visoka).
* **Boje marke** – uskladite paletu boja vaše organizacije za izvješća.
* **Prikladno za daltoniste**: Koristite kombinacije narančaste i plave ili ljubičaste i žute.
* **Optimizacija ispisa** – Odaberite boje koje rade i u boji i u sivim tonovima ispisa.
* **Višestruki pragovi**: Koristite različite boje na određenim pragovima vrijednosti za klasifikaciju.

{% hint style=&quot;info&quot; %}
**Spremi prilagođene gradijente** – prilagođene gradijente možete spremiti i ponovno upotrijebiti. Kliknite ikonu Spremi na ploči LUT da biste spremili prilagođene sheme boja za buduću upotrebu.
{% endhint %}

***

## Interaktivni tijek rada

### Ažuriranja u stvarnom vremenu

Sve LUT postavke u sandboxu ažuriraju sliku **trenutačno i interaktivno**:

* **Promijeni sloj** → Slika se odmah mijenja
* **Odaberite Gradient** → Boje se trenutno ažuriraju
* **Prilagodi raspon vrijednosti** → Kontrast se mijenja u stvarnom vremenu
* **Promijeni klase** → Glatkoća gradijenta ažurira se odmah
* **Promijeni izrezivanje** → Pozadinski prikaz se trenutno mijenja
* **Uredi boje** → Prilagođeni gradijent primjenjuje se odmah

**Nije potreban gumb “Primijeni”** – sve promjene su aktivne i interaktivne!

{% hint style=&quot;uspjeh&quot; %}
**Povratne informacije uživo** – Trenutačne vizualne povratne informacije omogućuju vam brzo eksperimentiranje s različitim postavkama dok ne pronađete optimalan prikaz za svoje potrebe analize.
{% endhint %}

### Tijek rada iterativnog usavršavanja

**Tipični tijek rada optimizacije LUT-a:**

1. **Odaberite indeksni sloj** (na primjer, RAW (refleksija)).
2. **Primijeni indeks**: Odaberite filtar kamere i formulu indeksa, povucite krugove boja na odgovarajuće mjesto u formuli indeksa.
3. **Primijeni gradijent LUT** – Započnite s prethodnom postavkom crveno-žuto-zeleno.
4. **Provjerite vrijednosti piksela**: Pomaknite kursor i promatrajte raspone vrijednosti.
5. **Prilagodi min/max**: Smanjite raspon da biste se usredotočili na vegetaciju (na primjer, s 0,2 na 0,9).
6. **Odaberite izrezivanje** – pokušajte s “Izvornom pozadinom” za kontekst.
7. **Pročistite boje** – Prilagodite gradijent ako je potrebno kako biste naglasili nešto specifično.
8. **Završi konfiguraciju**: Dokumentirajte konfiguraciju i kopirajte je u konfiguraciju projekta za obradu izvoza.

### Provjera vrijednosti piksela

Razumijevanje stvarnih vrijednosti piksela ključno je za uspostavljanje učinkovitih LUT raspona:

**Kako provjeriti vrijednosti:**

1. Vrijednosti piksela prikazuju se kada slika ima **označene** okvire Index ili Index i LUT.
2. **Pomičite kursor** preko različitih područja slike.
3. **Zabilježite vrijednosti piksela** prikazane u legendi kada prijeđete iznad nje.
4. Povećajte sliku kako biste vidjeli pojedinačne piksele istaknute pokretnom vrijednošću.
5. **Obratite pažnju** na raspone vrijednosti za različite karakteristike:
   * **Zdrava vegetacija**: na primjer, NDVI 0,55-0,85.
   * **Vegetacija pod stresom**: na primjer, NDVI 0,30-0,50.
   * **Golo tlo**: npr. NDVI 0,05-0,25
   * **Voda** (ako postoji): npr. NDVI -0,05 do 0,10

**Korištenje vrijednosti piksela za postavljanje LUT raspona:**

Nakon provjere vrijednosti piksela, prema tome prilagodite minimalni i maksimalni LUT:

**Primjer scenarija:**

* **Opažanje**: Vrijednosti tla = 0,05-0,25, pod stresom = 0,25-0,50, zdravo = 0,50-0,85
* **Cilj**: Pregledajte samo zdravlje biljaka (isključite tlo)
* **Postavke LUT-a**: Min. = `0.25`, maks. = `0.85`
* **Obreži**: “Izvorna pozadina” za prikaz poda u prirodnoj boji
* **Rezultat**: Gradijent boja primjenjuje se samo na vegetaciju, tlo je prikazano kao na izvornoj slici

{% hint style=&quot;info&quot; %}
**Dinamički raspon**: Različiti usjevi, godišnja doba i faze rasta imat će različite raspone vrijednosti. Uvijek provjerite vrijednosti piksela u vašem određenom skupu podataka prije postavljanja LUT raspona.
{% endhint %}

***

## Prilagođeni indeksi (Chloros+)

### Stvaranje prilagođenih formula indeksa

{% hint style=&quot;info&quot; %}
**Gdje stvoriti**: Prilagođeni indeksi mogu se konfigurirati u **Postavke projekta** prije renderiranja, kao i na bočnoj traci preglednika slika.
{% endhint %}

**Za izradu prilagođenog indeksa:**

1. **Otvorite postavke projekta** (prije obrade) ili bočnu traku preglednika slika.
2. Idite na padajući izbornik **Indeksne formule**.
3. Pronađite opciju **«Custom»** (morate biti prijavljeni s Chloros+ licencom).
4. **Definirajte svoju formulu** koristeći varijable pojasa:
   * Nazivi bendova: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, itd.
   * Operatori: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funkcije: `sqrt()`, `abs()` itd. (ako je podržano)
   * Zagrade: `()` za redoslijed operacija
5. **Nazovite svoj indeks** (na primjer, “MyIndex” ili “CustomNDVI”).
6. **Spremite postavke**.

**Primjeri prilagođenih formula:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;upozorenje&quot; %}
**Provjera formule**: Provjerite koristi li vaša formula opsege dostupne u vašem fotoaparatu. Na primjer, RedEdge je dostupan samo na fotoaparatima s RedEdge filterom.
{% endhint %}

***

## Sljedeći koraci

Sada kada znate Index/LUT Sandbox:

* **Primijeni na obradu**: Koristite postavke otkrivene u [Postavke projekta](../project-settings/project-settings.md)
* **Skupna obrada**: Primijenite optimizirane indekse na cijele skupove podataka
* **Više informacija**: pročitajte [Formule multispektralnog indeksa](../project-settings/multispectral-index-formulas.md)

Povezana dokumentacija:

* [**Slojevi slike**](image-layers.md) - Upravljanje i prikaz slojeva
* [**Otvori sliku preko cijelog zaslona**](opening-an-image-full-screen.md) - Osnove preglednika slika
* [**Obrada slike (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Potpun tijek obrade