# Postavke projekta

Bočna traka Postavke projekta <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

u programu Chloros

omogućuje vam konfiguriranje svih aspekata obrade slika, detekcije kalibracijskih ciljeva, izračuna višespektralnih indeksa i opcija izvoza za vaš projekt. Ove se postavke spremaju s vašim projektom i mogu se spremiti kao predlošci za ponovnu upotrebu u više projekata.

## Pristup postavkama projekta

Za pristup postavkama projekta:

1. Otvorite projekt u Chloros


2. Kliknite na karticu **Postavke projekta** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

u lijevoj bočnoj traci
3. Panel s postavkama prikazat će sve dostupne opcije konfiguracije organizirane po kategorijama

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Postavke koje ovise o drugim postavkama sive su boje.** Kada glavni prekidač onemogući neku postavku (na primjer, odabirom opcije *Kalibracija refleksije / bijela boja* onemogućuje se *Izvoz refleksije*), ovisna kontrola se onemogućuje, a u skočnom opisu se navodi naziv prekidača koji se mora promijeniti.
{% endhint %}

***

## Prikaz

### Rezolucija sličice

* **Vrsta**: padajući izbornik
* **Opcije**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Zadano**: Zadano (512 px)
* **Opis**: Rezolucija (najduža strana, u pikselsima) pri kojoj se prikazuju minijature mreže slike. Više vrijednosti izgledaju oštrije pri uvećanju, ali se sporije učitavaju i koriste više memorije. Puna rezolucija služi za veličinu izvorne slike.
* **Napomena**: Samo za prikaz — ovo nikada ne utječe na obradu ili izvezene datoteke.***

## Detekcija ciljeva

Ove postavke kontroliraju kako Chloros

detektira i obrađuje ciljeve kalibracije na vašim slikama. Oba su aktivna samo dok je omogućen **Kalibracija reflektancije / balans bijele boje** (u suprotnom su zasivljena, jer se detekcija ciljeva u potpunosti preskače).

### Minimalna površina uzorka kalibracije (px)

* **Vrsta**: Broj
* **Raspon**: 0 do 10.000 piksela
* **Zadano**: 25 piksela
* **Opis**: Postavlja minimalnu površinu (u pikselsima) potrebnu da bi se detektirana regija smatrala valjanim uzorkom kalibracijskog cilja. Manje vrijednosti detektirat će manje ciljeve, ali mogu povećati lažno pozitivne rezultate. Veće vrijednosti zahtijevaju veće, jasnije regije ciljeva za detekciju.
* **Kada prilagoditi**:
  * Povećajte ako dobivate lažne detekcije na malim artefaktima na slici
  * Smanjite ako se vaši kalibracijski ciljevi na slikama čine malima i ne detektiraju se

### Minimalno grupiranje ciljeva (0-100)

* **Tip**: Broj
* **Raspon**: 0 do 100
* **Zadano**: 60
* **Opis**: Kontrolira prag grupiranja za grupiranje sličnih obojenih područja prilikom otkrivanja kalibracijskih ciljeva. Više vrijednosti zahtijevaju da se više sličnih boja grupira zajedno, što rezultira konzervativnijim otkrivanjem ciljeva. Niže vrijednosti dopuštaju veću varijaciju boja unutar grupe ciljeva.
* **Kada prilagoditi**:
  * Povećajte ako se kalibracijske oznake dijele na više detekcija
  * Smanjite ako se kalibracijske oznake s varijacijom boje ne detektiraju u potpunosti

***

## Obrada

Ove postavke kontroliraju kako Chloros

obrađuje i kalibrira vaše slike.

### Korekcija vinjetiranja

* **Vrsta**: Polje za potvrdu
* **Zadano**: Omogućeno (označeno)
* **Opis**: Primjenjuje korekciju vinjetiranja kako bi se nadoknadilo zatamnjivanje objektiva na rubovima slike. Vinjetiranje je uobičajeni optički fenomen pri kojem kutovi i rubovi slike izgledaju tamniji od središta zbog karakteristika objektiva.
* **Sporedni učinak**: Ova postavka također odabire koji *rezervni kalibracijski proizvod* program bilježi (vidi dolje).

### Kalibracija reflektancije / balans bijele

* **Vrsta**: Polje za potvrdu
* **Zadano**: Omogućeno (označeno)
* **Opis**: Omogućuje kalibraciju reflektancije — na temelju otkrivenih ciljeva za kalibraciju unutar kadra i/ili podataka o dolaznom svjetlu s DAQ senzora svjetlosti, ovisno o kameri i dostupnim podacima. Time se normaliziraju vrijednosti reflektancije u vašem skupu podataka i osiguravaju dosljedna mjerenja bez obzira na uvjete osvjetljenja.
* **Kada je onemogućeno**: Detekcija ciljeva se u potpunosti preskače i**nijedna kamera ne može proizvesti nikakav proizvod reflektancije** — ni onaj vođen ciljevima kamere *Survey3

*, ni onaj vođen DAQ-om *LATTICE*. Ovisne postavke (*Izvoz reflektancije*, *Minimalni interval ponovne kalibracije* i pragovi detekcije ciljeva) su zasivljene.

### Rezervne kalibrirane slike: Izvoz reakcije senzora / Izvoz s korekcijom vignetiranja

* **Vrsta**: Dva potvrdna okvira
* **Zadano**: Oba su omogućena (označena)
* **Opis**: Kada se kadar ne može kalibrirati po odrazu (nije pronađen cilj za kalibraciju ili je kalibracija po odrazu isključena), umjesto toga se zapisuje kao *nekalibrirani alternativni proizvod*. Točno jedan od ta dva zamjenska proizvoda postoji po izvršenju, za svaki model kamere**, odabran prekidačem *Korekcija vignete*:
  * Korekcija vinjete **uključena**→ `Vignette_Corrected_Images/` (podliježe postavci**Izvoz s korekcijom vinjete**)
  * Korekcija vinjete **isključena**→ `Sensor_Response_Images/` (podliježe postavci**Izvoz odgovora senzora**)
* Rezervna opcija koja se ne primjenjuje sive je boje. Odabirom one koja se primjenjuje sprječava se pisanje te datoteke.

### LATTICE izvozni proizvodi

Za projekte koji sadrže LATTICE snimke, svaki uvezeni LATTICE kadar raspoređuje se na sve omogućene **i primjenjive**proizvode u jednom prolazu obrade. Četiri potvrdne okvire kontroliraju razgranavanje (sve su prema zadanim postavkama**uključene**):

| Postavka | Izlazna mapa | Što se izvozi |
| --- | --- | --- |
| **Izvoz debayered** | `Debayered_Images/` | Linearna debayered slika: uklonjen je tamni offset kalibracijskog paketa, zatim slijedi demosaiciranje osjetljivo na rubove, bez korekcije linearnosti, flat filed ili unmixa — ista datoteka koju snimka sprema na svojoj razini nakon debayeringa. Održava se u nativnoj orijentaciji senzora (zabilježeno u XMP-u). Primjenjivo na RGB

i multispektralne kamere. |
| **Izvoz pregleda** | `Preview_Images/` | Pregled za prikaz. RGB

= balans bijele (DAQ-illuminant kada je dostupan, inače svijetlo sivo) + gamma; multispektralno = rastezanje lažnih boja. |
| **Izvoz radijance** | `Radiance_Images/` | Spektralna radijanca u formatu Float32 u W/m²/sr/nm. Samo za multispektralne (M3C/M3M) – ne primjenjivo na RGB

glavne datoteke. Uvijek se zapisuje kao 32-bitni *TIFF

* bez obzira na postavku *Kalibrirani format slike*. |
| **Izvoz reflektancije**| `Reflectance_Calibrated_Images/` | Uint16 reflektancija, skalirana tako da**32768 = reflektancija 1,0** (označeno kao XMP `Chloros:PixelScale`). Samo za multispektralne snimke, bilježi se kada odgovarajući zapis `.daq` o dolaznom zračenju (ili cilj u kadru koji je prošao QA provjeru) pokriva kadar. |

* Glavne kamere RGB

emitiraju debayered + pretpregled; radijansa/reflektancije za njih se preskače jer se ne primjenjuju.
* Bitna dubina debayered/pretpregleda slijedi postavku *Kalibrirani format slike*; radijans je uvijek float32.
* Obrada Survey3

nije pogođena ovim četirima prekidačima.
* Proizvodi radijance i refleksance multispektralnih M3C kamera također mogu pisati po pikselu **maske ograničenja** u mapu `clamp_masks/` pokraj mapa proizvoda, bilježeći gdje je dekonvolucija pojasa isječena na nulu iz negativne vrijednosti. Vidi [Formati izlaznih slika](../output-image-formats.md#clamp-masks).

Ista četiri prekidača postoje i u načinu bez grafičkog sučelja kao `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Zamijenili su stari CLI

prekidač `--radiometric-output`, koji više ne postoji.

{% hint style="warning" %}
**Isključivanje svih primjenjivih proizvoda uzrokuje neuspjeh izvršavanja.** Od verzije 1.2.0, izvršavanje obrade koje je zatražilo proizvode, ali nije napisalo nijedan izvještaj o slici, prijavljuje neuspjeh i CLI

izlazi s ne-nuljevnom vrijednošću, umjesto da prijavi tihi uspjeh. U zapisniku se navodi proizvod koji nije mogao biti napisan i razlog. Namjerno izvršavanje samo metapodataka (ništa nije zatraženo) i dalje je uspjeh.
{% endhint %}

### Izvor refleksije (postavka projekta, postavlja se putem CLI

)

Projekt također pohranjuje koju **referencu refleksije** koristi LATTICE proizvod za refleksiju. U upravljačkoj ploči nema namjenskog upravljačkog elementa; vrijednost se pohranjuje u konfiguraciji projekta kao `Processing → "Target reflectance source"` i postavlja se pomoću `chloros-cli process --reflectance-source {auto,target,daq}` (SDK

ov `configure()` nema ključnu riječ za to):

* **`auto`** (zadano): kalibracijska meta unutar kadra koja prolazi QA postaje apsolutna referenca, a kao rezerva se koristi DAQ downwelling divide (ρ = πL/E) kada meta nije prisutna ili QA ne uspije. Cilj koji prođe QA s oznakom (ispušten panel ili traka izvan redoslijeda) ne dobiva automatski prednost: lanci cilj, cilj+DAQ i samo DAQ isprobavaju se prema redoslijedu njihovih izmjerenih rezultata na sačuvanim panelima.
* **`target`**: stroga reflektancija vođena ciljem — bez DAQ zamjene.
* **`daq`**: reflektancija autoritativna za DAQ; ciljevi unutar kadra ne koriste se kao referenca.

Pohranjena vrijednost podudara se bez obzira na velika i mala slova, a nekoliko se pravopisnih varijanti prihvaća kao sinonimi: `target`, `target_image`, `empirical` i `empirical_line` sve znače **cilj**; `daq`, `dls`, `light_sensor` i `sensor` sve znače**daq**. Sve ostalo — uključujući nepostojeći ključ — rješava se kao**auto**.**Mjereni** skenovi cilja po jedinici traže se prema serijskom/QR broju ciljne jedinice, kao `<serial>.csv`, na tri mjesta: direktor naveden s `--target-reflectance-dir` (pohranjen kao `Processing → "Target reflectance dir"`), vlastiti `target_reflectance/` direktorij projekta, i putanju u varijabli okruženja `CHLOROS_TARGET_REFLECTANCE_DIR`. Kada za tu jedinicu ne postoji izmjerena skenirana slika, umjesto toga se koristi nominalna objavljena krivulja za model cilja.

### Debayer metoda

* **Vrsta**: Izborni padajući popis
* **Opcije**:
  * Standardno (Brzo, srednja kvaliteta)
  * S obzirom na teksturu (Sporo, najviša kvaliteta) \[Chloros

+]
* **Zadano**: Standardno (Brzo, srednja kvaliteta)
* **Opis**: Odabire se demosaicing algoritam koji se koristi za pretvorbu sirovih podataka senzora s Bayerovim uzorkom u slike u punoj boji. Metoda &quot;Standard (brzo, srednja kvaliteta)&quot; pruža optimalan omjer između brzine obrade i kvalitete slike. Metoda &quot;Texture Aware (sporo, najviša kvaliteta)&quot; [Chloros

+] koristi visokokvalitetni debayer koji je osjetljiv na rubove, u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji/strojnom učenju (AI/ML), koji uklanja gotovo sav šum nastali debayerizacijom. Model Texture Aware zahtijeva GPU memoriju (VRAM) za rad. Preporučujemo ga koristiti kada imate više od 4 GB VRAM-a na raspolaganju za bržu obradu.
* **Kada je redak uopće padajući izbornik**: padajući izbornik s dvije opcije pojavljuje se samo kada**oba** su istinita — prijavljeni ste s kvalificiranom pretplatom na Chloros

+, **i** projekt ne sadrži snimke LATTICE. U suprotnom, redak se prikazuje kao običan tekst s natpisom `Standard (Fast, Medium Quality)` bez ikakve opcije za odabir.
* **Napomena za LATTICE**: Ne postoji model osjetljiv na teksture (Texture Aware) obučen za LATTICE, a pipeline nameće standardno demosaiciranje za LATTICE sličice bez obzira na pohranjenu vrijednost. Ako dodate LATTICE mapu u projekt u kojem je već bio odabran Texture Aware, Chloros

vraća postavku na Standard umjesto da zadrži zastarjelu vrijednost u `project.json`.

### Minimalni interval ponovne kalibracije

* **Vrsta**: Broj
* **Raspon**: od 0 do 3.600 sekundi
* **Zadano**: 0 sekundi
* **Opis**: Postavlja minimalni vremenski interval (u sekundama) između korištenja ciljeva kalibracije. Kada je postavljeno na 0, Chloros

će koristiti svaki otkriveni cilj kalibracije. Kada je postavljeno na višu vrijednost, Chloros

će koristiti samo ciljeve kalibracije koji su odvojeni najmanje tim brojem sekundi, čime se smanjuje vrijeme obrade za skupove podataka s čestim snimkama ciljeva kalibracije.
* **Kada prilagoditi**:
  * Postavite na 0 za maksimalnu točnost kalibracije kada se uvjeti osvjetljenja mijenjaju
  * Povećajte (npr. na 60-300 sekundi) za bržu obradu kada je osvjetljenje konstantno i imate česte slike ciljeva kalibracije

### Pomak vremenske zone senzora svjetla

* **Tip**: Broj
* **Raspon**: -12 do +12 sati
* **Zadano**: 0 sati
* **Opis**: Određuje pomak vremenske zone (u satima od UTC-a) za vremenske oznake podataka senzora svjetla, koji se koristi pri usklađivanju dnevnika senzora svjetla s vremenima snimanja slika. Noviji zapisi `.daq` sadrže vlastito porijeklo vremenske zone, stoga je ovo uglavnom potrebno za starije dnevnike snimljene u lokalnom vremenu.

### Primjena PPK korekcija

* **Tip**: Polje za potvrdu
* **Zadano**: Onemogućeno (neoznačeno)
* **Opis**: Omogućuje upotrebu kinematičkih korekcija nakon obrade (PPK) s MAPIR

DAQ snimača koji sadrže GPS (GNSS). Kada je omogućeno, Chloros

će koristiti sve .daq datoteke dnevnika koje sadrže podatke o ekspoziciji u vašem projektnom direktoriju i primijeniti precizne korekcije geolokacije na vaše slike.
* **Zahtjev**: Datoteka dnevnika .daq s unosima za ekspozicijske igle mora biti prisutna u direktoriju vašeg projekta
* **Kada omogućiti**: Preporučuje se uvijek omogućiti PPK korekciju ako imate unose povratne sprege za ekspoziciju u datoteci dnevnika .daq.

### Pin za ekspoziciju 1

* **Vrsta**: Izbornik za odabir
* **Vidljivost**: Vidljivo samo kada &quot;Primijeni PPK korekcije&quot; je omogućeno I podaci o ekspoziciji su dostupni za Pin 1
* **Opcije**:
  * Nazivi modela kamere otkriveni u projektu
  * &quot;Ne koristi&quot; - Ignoriraj ovaj pin za ekspoziciju
* **Zadano**: Automatski odabrano na temelju konfiguracije projekta
* **Opis**: Dodjeljuje određenu kameru na Pin za ekspoziciju 1 za vremensku sinkronizaciju PPK-a. Pin za ekspoziciju bilježi točno vrijeme kada se okida zatvarač kamere, što je ključno za preciznu PPK geolokaciju.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedan pin: Automatski odabire kameru
  * Jedna kamera + dva pina: Pin 1 se automatski dodjeljuje kameri
  * Više kamera: Potrebna je ručna selekcija

### Izlaganje Pin 2

* **Vrsta**: Izbor iz padajućeg popisa
* **Vidljivost**: Vidljivo samo kada je omogućen &quot;Primijeni PPK korekcije&quot; I kada su dostupni podaci o izlaganju za Pin 2
* **Opcije**:
  * Nazivi modela kamera otkriveni u projektu
  * &quot;Ne koristi&quot; - Ignorirajte ovaj ekspozicijski pin
* **Zadano**: Automatski odabrano na temelju konfiguracije projekta
* **Opis**: Dodjeljuje određenu kameru ekspozicijskom pinu 2 za vremensku sinkronizaciju PPK-a pri korištenju postavke s dvije kamere.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedan pin: Pin 2 se automatski postavlja na &quot;Ne koristi se&quot;
  * Jedna kamera + dva pina: Pin 2 se automatski postavlja na &quot;Ne koristi se&quot;
  * Više kamera: Potrebno je ručno odabrati
* **Napomena**: Iste kamere ne može se istovremeno dodijeliti i Pin 1 i Pin 2.***

## DAQ senzor svjetla

Ovaj odjeljak se pojavljuje u Postavkama projekta i navodi svaku DAQ datoteku za snimanje u projektu — snimke `.daq` i DAQ-M `.csv` zapisnici za snimanje prema dolje. Snimke napravljene na kartici Svjetlosni senzori automatski se dodaju u otvoreni projekt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Svaki red prikazuje datoteku, model senzora i korekciju difuzora-kapice koja je trenutačno na snazi za tu datoteku. Iznad redova nalazi se jedna kontrola za cijeli projekt:

### Premošćivanje kapice (sve datoteke)

* **Vrsta**: Izbor padajućeg popisa
* **Opcije**: `Auto` i profili korekcije kapa koji vrijede za vrste senzora prisutne u projektu
* **Zadano**: Auto
* **Pohranjeno kao**: `Processing → "DAQ cap id"` (zadano `auto`)
* **Opis**: `Auto` koristi zabilječenu kapu svake datoteke (pretpostavlja se kapica Sunshine ako ništa nije zabilježeno, ali DAQ-U, DAQ-M i DAQ-E isporučuju se bez kape i Chloros

ne može vidjeti što je postavljeno, pa obrada upozorava na pretpostavljenu kapu: odaberite kapu koja je bila postavljena). Odabir određene kape poništava **sve** datoteke s padajućim snimkama u projektu osim onih DAQ-E-S, čiji je difuzor dio njegove kalibracije; projekt koji sadrži samo DAQ-E-S ne prikazuje padajući izbornik: sirovi snimci se korigiraju njime, i snimke koje već imaju kapu se ponovno referenciraju (zabilježena korekcija se poništava i primjenjuje odabrana).
* **Važno**: Odabrana kapa mora odgovarati kapi koja je bila fizički postavljena tijekom snimanja. Ni senzor ni softver ne mogu detektirati fizičku kapu — neusklađeni ID kape pogrešno korigira spektre.

Namjerno postoji **jedan** padajući izbornik na razini projekta umjesto po pojedinačnim datotekama: postavka se primjenjuje na sve izvore u usponu u projektu.***

## Poravnanje nizova

Ovaj se odjeljak prikazuje **samo** kada barem jedna slika u projektu sadrži transformaciju poravnanja između modula koju matrice LATTICE utiskuju prilikom snimanja (oznake XMP `Chloros:Alignment*`). Prikazuje koliko slika ima oznake poravnanja, koja je kamera referentna (oznaka `REF`) i tablicu po kamerama s brojem slika.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Primijeni poravnanje nizova

* **Vrsta**: Polje za potvrdu
* **Zadano**: Omogućeno (potvrđeno)
* **Pohranjeno kao**: `Processing → "Array alignment"`
* **Opis**: Prilagođava svaki obrađeni proizvod (debayered / pretpregled / radijanca / reflektancija / indeks) na zajedničku referentnu geometriju niza koristeći transformaciju zabilježenu pri snimanju. Isključeno = izvoz u izvornoj geometriji po senzoru.

### Obrezivanje na zajednički preklop

* **Tip**: Potisno polje (aktivno samo dok je *Primjena poravnanja niza* uključena)
* **Zadano**: Uključeno (označeno)
* **Pohranjeno kao**: `Processing → "Array alignment crop"`
* **Opis**: Obrezuje poravnane izvoze na regiju koju dijele svi moduli kamera, tako da svaki pojas ima isti otisak. Isključeno zadržava cijelo platno senzora (crno popunjavanje izvan izvora).

### Ponovno uzorkovanje

* **Vrsta**: Izbornik (aktivno samo dok je *Primijeni poravnanje niza* uključeno)
* **Opcije**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Zadano**: Bilinearno
* **Pohranjeno kao**: `Processing → "Array alignment interpolation"`
* **Opis**: Interpolacija koju koristi warp poravnanja. Nearest (Najbliže) zadržava točne vrijednosti izvora (bez miješanja između piksela) za strogu radiometrijsku analizu; Bilinear (Bilinearna) je najbolja za mapiranje i vizualnu upotrebu.

Iste tri opcije postoje i bez glave kao `chloros-cli process --array-alignment`, `--array-alignment-crop` i `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Ove postavke omogućuju konfiguraciju multispektralnih indeksa za analizu i vizualizaciju.

### Dodaj indeks

* **Vrsta**: Poseban panel za konfiguraciju indeksa
* **Opis**: Otvara interaktivni panel na kojem možete odabrati i konfigurirati multispektralne indekse vegetacije (NDVI

, NDRE

, EVI

itd.) za izračun tijekom obrade slike. Možete dodati više indeksa, svaki sa svojim postavkama vizualizacije.
* **Dostupni indeksi**: Spuštajući izbornik GUI-ja uključuje**27** unaprijed definiranih formula za višespektralne indekse (pogledajte [Formule za višespektralne indekse](multispectral-index-formulas.md) za potpuni popis, uključujući koja se imena također prihvaćaju putem CLI

/SDK

`--indices` opcija).
* **Značajke**:
  * Odaberite unaprijed definirane formule indeksa
  * Povucite filtarske kanale vaše kamere na utore za pojaseve u formuli
  * Konfigurirajte gradijente boja za vizualizaciju (LUT - tablice dohvata)
  * Postavljanje pragova i načina obrezivanja
  * Izrada prilagođenih indeksnih formula
* **Napomena**: Indeksi se ne izračunavaju za mono kamere LATTICE M3M s jednom trakom — višetranski indeksi su neodređeni za jednu traku. ModeliSurvey3

i LATTICE M3C nisu pogođeni.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Svaki indeks koji dodate prikazuje svoju formulu kao matematičku jednadžbu, s obojenim krugom za svako mjesto pojasa: crvena = Red

, zelena = Green

, plava = Blue

, narančasta = Orange

, cyan = Cyan

, ljubičasta = NIR

, magenta = RE. Povucite krug iz retka iznad formule na utor kako biste ga povezali; dvaput kliknite na povezani utor kako biste ga očistili. Indeks se izračunava samo kada svaki utor koji formula koristi ima kanal.

### Prilagođene formule (funkcionalnost Chloros

+)

* **Vrsta**: Niz definicija prilagođenih formula
* **Dostupnost**: Zahtijeva prijavu s odgovarajućom pretplatom na Chloros

+.
* **Opis**: Omogućuje vam stvaranje i spremanje prilagođenih formula za višespektralne indekse pomoću matematike pojaseva. Prilagođene formule spremaju se s postavkama vašeg projekta i mogu se koristiti kao i ugrađeni indeksi.
* **Kako stvoriti**:
  1. U konfiguracijskom panelu indeksa otvorite kalkulator za prilagođene formule
  2. Napišite formulu koristeći **simbole traka i utora**, a ne nazive traka
  3. Spremite formulu s opisnim nazivom — tada se pojavljuje na dnu padajućeg izbornika formula, a vi povlačite krugove kanala svoje kamere na njezine utore točno kao i kod ugrađenih predinstalacija
* **Syntaxa formule**:
  * Utori pojaseva: `x`, `y`, `z`, `a`, `b`, `c` — šest položaja koje mapirate na stvarne kanale povlačenjem
  * Operatori: `+`, `-`, `*`, `/`, `^` i `()` za grupiranje
  * Funkcije: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Zašto simboli, a ne nazivi bendova**: formula napisana kao `(y-x)/(y+x)` radi na bilo kojoj kameri, jer mapiranje povlačenjem i ispuštanjem odlučuje je li `y` 850 nm NIR

a RGN

filtera ili 808 nm NIR

a OCN

filtera. ugrađeni preseti spremaju se na isti način — pogledajte [Multispectral Index Formulas](multispectral-index-formulas.md) za točan simbolički oblik svih 27.
* **Gdje rade**: prilagođene formule spremaju se s postavkama projekta i mogu se koristiti u [Index/LUT Sandboxu](../image-viewer-gui/index-lut-sandbox.md) kao i u obradi. Ne prihvaća ih popis imena CLI

/SDK

`--indices`, koji samo proširuje 22 ugrađena naziva predložaka.

***

## Izvoz

Ove postavke kontroliraju format i kvalitetu izvezanih obrađenih slika.

### Kalibrirani format slike

* **Vrsta**: Izbor padajućeg popisa
* **Opcije**:
  * **TIFF

(16-bit)** - Nekomprimirani 16-bitni format TIFF


  * **TIFF

(8-bit)** - Nepritisnuti 8-bitni format TIFF


  * **TIFF

(32-bit, Postotak)** - 32-bitni format TIFF

s pokretnim zarezom i vrijednostima reflektancije u postotcima
  * **PNG

(8-bit)** - Pritisnuti 8-bitni format PNG


  * **JPG (8-bit)** - Komprimirani 8-bitni format JPEG


* **Zadano**: TIFF

(16-bit); projekt koji je u potpunosti 8-bitni prikazuje TIFF

(8-bit) i nudi samo 8-bitne formate
* **Opis**: Odabire format datoteke za spremanje obrađenih i kalibriranih slika. Izvozi podatke u podmapu za svaki format unutar mape svake kamere (`tiff16`, `tiff32`, `png8`, `jpg8`), s jednom mapom `<Product>_Images/` po proizvodu. Izvezene datoteke zadržavaju izvorni naziv datoteke — proizvod se identificira mapom, a ne nastavkom naziva datoteke.
* **Preporuke za formate**:
  * **TIFF

(16-bitni)**: Preporučuje se za znanstvenu analizu i profesionalne radne procese. Očuvava maksimalnu kvalitetu podataka bez artefakata kompresije. Najbolje za multispektralnu analizu i daljnju obradu u GIS softveru.
  * **TIFF

(32-bita, postotak)**: Najbolje za radne procese koji zahtijevaju vrijednosti odbijanja kao postotke (0-100%). Nudi maksimalnu preciznost za radiometrijska mjerenja.
  * **PNG

(8-bit)**: Dobar za pregledavanje na webu i opću vizualizaciju. Manje veličine datoteka s kompresijom bez gubitaka, ali smanjen dinamički raspon.
  * **JPG (8-bit)**: Najmanje veličine datoteka, najbolje samo za pregled i prikaz na webu. Koristi kompresiju s gubicima koja nije pogodna za znanstvenu analizu.
* **Napomena**: Radijansa LATTICE-a uvijek se izvozi kao 32-bitni float TIFF

, bez obzira na ovu postavku.

***

## Spremanje predloška projekta

Ova značajka omogućuje vam spremanje trenutačnih postavki vašeg projekta kao ponovno upotrebljivog predloška.

* **Vrsta**: Unos teksta + gumb za spremanje
* **Opis**: Unesite opisni naziv za svoj predložak postavki i kliknite ikonu spremanja. Predložak će spremiti sve trenutačne postavke vašeg projekta (detekcija ciljeva, opcije obrade, indeksi, i format izvoza) za jednostavno ponovno korištenje u budućim projektima. Predlošci se pohranjuju u mapu `Project Templates/` unutar mape za spremanje vašeg projekta, a mogu se odabrati ili izvesti i iz glavnog izbornika (*Odabir predloška* / *Spremanje predloška* / *Izvoz predloška*).
* **Primjeri upotrebe**:
  * Izrada predložaka za različite sustave kamera (RGB

, multispektralne, NIR

)
  * Spremanje standardnih konfiguracija za određene vrste usjeva ili tijekove analize
  * Dijeljenje dosljednih postavki unutar tima
* **Upute za upotrebu**:
  1. Konfigurirajte sve željene postavke projekta
  2. Unesite naziv predloška (npr. &quot;RedEdge

Survey3

NDVI

Standard&quot;)
  3. Kliknite ikonu spremanja
  4. Predložak se sada može učitati prilikom izrade novih projekata

***

## Spremanje mape projekta

Ova postavka određuje gdje se novi projekti po zadanom spremaju.

* **Vrsta**: Prikaz putanje direktorija + gumb Uređivanje
* **Zadano (Windows

)**: `Documents\MAPIR\Chloros Projects` u vašem mapi Dokumenti — obično `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, ili gdje god su Dokumenti premješteni na OneDriveu ili u putujućem profilu
* **Zadano (Linux

)**: `~/Documents/MAPIR/Chloros Projects`
* **Opis**: Prikazuje trenutni zadani direktorij u kojem se stvaraju novi projekti za Chloros

. Kliknite ikonu za uređivanje kako biste odabrali drugi direktorij. Preusmjeravanje se pohranjuje kao jedan redak teksta u `~/.chloros/working_directory.txt` — na Windows

to je `C:\Users\<Username>\.chloros\working_directory.txt`. Ako ta datoteka nedostaje ili navodi putanju koja više ne postoji, Chloros

se vraća na gore navedenu zadanu vrijednost. CLI

čita i zapisuje istu datoteku, pa su `chloros-cli` i GUI uvijek suglasni o tome gdje se projekti nalaze.
* **Predlošci projekata** nalaze se u podmapu `Project Templates/` unutar ovog direktorija.
* **Kada promijeniti**:
  * Postavite na mrežni pogon za suradnju tima
  * Promijenite na pogon s više prostora za pohranu za velike skupove podataka
  * Organizirajte projekte po godini, klijentu ili vrsti projekta u različitim mapama
* **Napomena**: Promjena ove postavke utječe samo na NOVE projekte. Postojeći projekti ostaju na svojim izvornih lokacijama.
* **Dužina staze Windows

a**: osim ako su omogućene duge staze sustava Windows

(opcija **Omogući duge staze sustava Windows** u instalacijskom programu), mapa projekta dublja od otprilike 130 znakova bit će odbijena prije obrade jer bi njezini proizvodi premašili ograničenje duljine staze sustava Windows

od 260 znakova. Vidi [Pokretanje obrade](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Održavanje postavki

Chloros

projekt je **mapa**. Sve postavke projekta spremaju se unutar nje u `project.json`; Povezani hardver pamti se uz njega u `cameras.json` i `sensors.json`, tako da se pri ponovnom otvaranju projekta ponovno povezuju njegove kamere i senzori svjetla. Kada ponovno otvorite projekt, sva se postavke vraćaju točno onako kako ste ih ostavili. Spremljeni projekti također se mogu upravljati bez zaslona pomoću `chloros-cli project` ili `open_project` iz SDK

a.

### Hijerarhija postavki

Postavke se primjenjuju sljedećim redoslijedom:

1. **Zadane postavke sustava** - Ugrađene zadane postavke koje definira Chloros

2. **Postavke predloška** - Ako učitavate predložak prilikom izrade projekta
3. **Spremljene postavke projekta** - Postavke spremljene s datotekom projekta
4. **Ručne prilagodbe** - Sve promjene koje napravite tijekom trenutne sesije

### Postavke i obrada slika

Postavke obrade čitaju se kada se pokrene obrada. Promjena postavke ne mijenja retroaktivno proizvode koji su već spremljeni na disk — ponovno pokrenite obradu kako biste primijenili nove postavke. Nekoliko postavki nikada uopće ne utječe na obradu:

* Rezolucija sličice slike (samo za prikaz)
* Spremi predložak projekta
* Spremi mapu projekta

***

## Referenca ključeva konfiguracije

Za automatizaciju (CLI

`--config`, SDK

`configure` ili izravno čitanje `project.json`), ovo su točne ključne riječi pod `Project Settings`:

| Putanja do ključa | Tip | Zadano |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | broj 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | broj 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | broj 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | broj -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | ID profila sposobnosti ili `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | popis konfiguracija indeksa | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Ključevi `Array alignment` upisuju se prvi put kada se prikazuje odjeljak poravnanja niza ili kada ih postavi poziv automatizacije. Dok su odsutni, pipeline koristi iste vrijednosti prikazane gore (`true`, `true`, bilinear), stoga se projekt.json bez njih ponaša identično onome s njima.

### Ključevi pohranjeni u `project.json` bez kontrole u panelu postavki

Oni se nalaze pod istim `Project Settings` stablom i čitaju se tijekom obrade, ali nećete pronaći widget za njih u bočnoj traci:

| Put do ključa | Vrsta | Zadano | Postavlja |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK

`input_level=`. Prepisuje način na koji se interpretiraju LATTICE ulazni TIFF

-ovi; `auto` zaključuje iz XMP oznake `Chloros:ProcessingLevel` svake datoteke i broja kanala. Zanemaruje se za snimke Survey3

`.raw`. Namjerno nije postavka GUI-ja — auto je ispravno u svakom normalnom slučaju. |
| `Processing → Target reflectance dir` | putanja u nizu znakova | `""` | `chloros-cli process --target-reflectance-dir`, ili projektna meta API

|
| `Processing → Target reflectance config` | rječnik indeksiran serijskim brojem kamere | `{}` | Registracija cilja unutar kadra (način rada `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | putanja | `""` | SDK

`process_folder(daq_log_path=…)`. Ukazuje na snimku `.daq` ili mapu sa snimkama |
| `Target Detection → Minimum calibration target squares` | broj | `4` | Naslijeđena zadana postavka; bez kontrole i bez zastavice CLI

|
| `UI → Grid thumbnail size` | broj | `160` | vlastiti klizač zumiranja sličica mreže slika |

Dvije postavke preglednika pohranjuju se **na najvišoj razini u `project.json`**, izvan `Project Settings` u potpunosti, jer su to stanja prikaza, a ne postavke obrade:

| Put do ključa | Tip | Zadano | Postavlja |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | cijeli broj 1–256 | `1` | Kontrola GSD-a (px) kartice slike — pogledajte [Otvaranje slike na cijelom zaslonu](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Najbolje prakse

1. **Počnite s zadanim postavkama**: Zadane postavke dobro funkcioniraju za većinu sustava kamera MAPIR

i uobičajenih tijekova rada.
2. **Stvorite predloške**: Nakon što optimizirate postavke za određeni tijek rada ili kameru, spremite ih kao predložak kako biste osigurali dosljednost u svim projektima.
3. **Testirajte prije potpune obrade**: Kada eksperimentirate s novim postavkama, testirajte ih na malom podskupu slika prije obrade cijelog skupa podataka.
4. **Dokumentirajte svoje postavke**: Koristite opisne nazive predložaka koji navode sustav kamere, vrstu obrade i namjenu (npr. &quot;Survey3

_RGB_NDVI_Poljoprivreda&quot;).
5. **Odabir formata izvoza**: Odaberite format izvoza na temelju konačne namjene:
   * Znanstvena analiza → TIFF (16-bitni ili 32-bitni)
   * GIS obrada → TIFF

(16-bitni)
   * Brza vizualizacija → PNG

(8-bit)
   * Dijeljenje na webu → JPG (8-bit)

***

Za više informacija o multispektralnim indeksima u programu Chloros

pogledajte stranicu [Multispectral Index Formulas](multispectral-index-formulas.md).
