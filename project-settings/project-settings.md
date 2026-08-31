# Postavke projekta

Bočna traka Postavke projekta<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

u programuChloros

omogućuje vam konfiguriranje svih aspekata obrade slika, detekcije ciljeva kalibracije, izračuna višespektralnih indeksa i opcija izvoza za vaš projekt. Ove se postavke spremaju s vašim projektom i mogu se spremiti kao predlošci za ponovnu upotrebu u više projekata.

## Pristup postavkama projekta

Za pristup postavkama projekta:

1. Otvorite projekt uChloros


2. Kliknite na karticu **Project Settings**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

u lijevoj bočnoj traci
3. Panel s postavkama prikazat će sve dostupne opcije konfiguracije organizirane po kategorijama



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Postavke koje ovise o drugim postavkama sive su.** Kada glavni prekidač onemogući neku postavku (na primjer, odabir *Kalibracija refleksije / bilanca bijele* onemogućuje *Izvoz refleksije*), ovisna kontrola se onemogućuje, a u skočnom opisu se navodi prekidač koji je potrebno promijeniti.
{% endhint %}

***

## Prikaz

### Rezolucija sličice

* **Vrsta**: Izbornik za odabir
* **Opcije**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Zadano**: Zadano (512 px)
* **Opis**: Rezolucija (najduži rub, u pikselsima) pri kojoj se prikazuju minijature mreže slika. Više vrijednosti izgledaju oštrije pri uvećanju, ali se sporije učitavaju i koriste više memorije. Puna rezolucija služi veličini izvornog slike.
* **Napomena**: Samo za prikaz — ovo nikada ne utječe na obradu ili izvezene datoteke.***

## Detekcija ciljeva

Ove postavke kontroliraju način na kojiChloros

detektira i obrađuje ciljeve kalibracije na vašim slikama. Obje su aktivne samo dok je omogućen **Kalibracija reflektancije / balans bijele boje** (u suprotnom su sivene, jer se detekcija ciljeva u potpunosti preskače).

### Minimalna površina uzorka kalibracije (px)

* **Vrsta**: Broj
* **Raspon**: od 0 do 10.000 piksela
* **Zadano**: 25 piksela
* **Opis**: Postavlja minimalnu površinu (u pikselsima) potrebnu da bi se otkriveno područje smatralo valjanim uzorkom kalibracijskog cilja. Manje vrijednosti će otkriti manje ciljeve, ali mogu povećati lažno pozitivne rezultate. Veće vrijednosti zahtijevaju veća, jasnija područja cilja za detekciju.
* **Kada prilagoditi**:
  * Povećajte ako dobivate lažne detekcije na malim artefaktima na slici
  * Smanjite ako se vaši ciljevi kalibracije na slikama čine malima i ne detekiraju se

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

Ove postavke kontroliraju kakoChloros

obrađuje i kalibrira vaše slike.

### Korekcija vinjetiranja

* **Vrsta**: Polje za označavanje
* **Zadano**: Omogućeno (označeno)
* **Opis**: Primjenjuje korekciju vinjetiranja kako bi se nadoknadio zamračivanje objektiva na rubovima slike. Vigneting je čest optički fenomen pri kojem kutovi i rubovi slike izgledaju tamniji od središta zbog karakteristika objektiva.
* **Sporedni učinak**: Ovaj prekidač također odabire koji *nekalibrirani rezervni proizvod* program zapisuje (vidi dolje).

### Kalibracija reflektancije / balans bijele

* **Vrsta**: Potisno polje
* **Zadano**: Omogućeno (označeno)
* **Opis**: Omogućuje kalibraciju reflektancije — na temelju otkrivenih ciljeva za kalibraciju unutar kadra i/ili podataka o dolaznom svjetlu s DAQ senzora svjetla, ovisno o kameri i dostupnim podacima. To normalizira vrijednosti reflektancije u vašem skupu podataka i osigurava dosljedna mjerenja bez obzira na uvjete osvjetljenja.
* **Kada je onemogućeno**: Detekcija ciljeva se u potpunosti preskače i**nijedan proizvod reflektancije ne može biti proizveden ni jednim fotoaparatom** — ni onim vođenim ciljevima (Survey3

) ni onim vođenim DAQ-om (LATTICE). Ovisne postavke (*Izvoz reflektancije*, *Minimalni interval ponovne kalibracije* i pragovi detekcije ciljeva) su zasivljene.

### Neokalibrirani alternativni proizvodi: Izvoz odgovora senzora / Izvoz s korekcijom vignete

* **Vrsta**: Dva potvrdna okvira
* **Zadano**: Oba su omogućena (označena)
* **Opis**: Kada se kadar ne može kalibrirati reflektancijom (nije pronađen cilj za kalibraciju ili je kalibracija reflektancijom isključena), umjesto toga se zapisuje kao *nekalibrirani alternativni proizvod*. **Po jednoj izvedbi postoji točno jedan od dva zamjenska proizvoda, za svaki model kamere**, odabran prekidačem *Korekcija vinjete*:
  * Korekcija vinjete **uključena**→ `Vignette_Corrected_Images/` (uređuje**Izvoz vinjete ispravljen**)
  * Korekcija vinjete **isključena**→ `Sensor_Response_Images/` (uređuje**Izvoz senzorskog odziva**)
* Rezervna opcija koja se ne koristi sive je boje. Odabirom one koja se koristi sprječava se pisanje te datoteke.

### LATTICE proizvodi za izvoz

Za projekte koji sadrže LATTICE snimke, svaki uvezeno LATTICE sličica raspoređuje se na sve omogućene **i primjenjive**proizvode u jednom prolazu obrade. Četiri potvrdne okvire kontroliraju raspoređivanje (sve su prema zadanim postavkama**uključene**):

| Postavka | Izlazna mapa | Što izvozi |
| --- | --- | --- |
| **Izvoz debayered slike** | `Debayered_Images/` | Linearna debayered slika. Primjenjivo na kamere sRGB

i multispektralne kamere. |
| **Pregled izvoza** | `Preview_Images/` | Pregled na zaslonu.RGB

= balans bijele (DAQ-illuminant kada je dostupan, inače svijet sive) + gamma; multispektralno = rastezanje lažnih boja. |
| **Izvoz radijance** | `Radiance_Images/` | Float32 spektralna radijanca u W/m²/sr/nm. Samo za multispektralne (M3C/M3M) — ne primjenjivo naRGB

masterima. Uvijek se zapisuje kao 32-bitniTIFF

bez obzira na postavku *Calibrated image format*. |
| **Izvoz reflektancije**| `Reflectance_Calibrated_Images/` | Uint16 reflektancija, skalirana tako da**32768 = reflektancija 1.0** (označeno kao XMP `Chloros:PixelScale`). Samo za multispektralne, zapisuje se kada odgovarajući zapis `.daq` za dolazno zračenje (ili cilj unutar kadra koji je prošao QA) pokriva kadar. |

* Glavne kamereRGB

-a emitiraju debayered + pregled; radijanca/reflektancija za njih se preskaču jer nisu primjenjive.
* Bitna dubina debayered/pregleda slijedi postavku *Kalibrirani format slike*; radijanca je uvijek float32.
* ObradaSurvey3

-a nije pogođena ovim četirima prekidačima.

Ista četiri prekidača postoje u načinu rada bez zaslona kao `chloros-cli process --debayered / --preview / --radiance / --reflectance` i kao odgovarajući parametri alataSDK

. Zamijenili su stari zastavicu `--radiometric-output`, koja više ne postoji.

{% hint style="warning" %}
**Isključivanje svih primjenjivih proizvoda uzrokuje neuspjeh izvršavanja.** Od verzije 1.2.0, izvršavanje obrade za koje su zatraženi proizvodi, a nije napisano nijedno izvješće o proizvodu slike, prijavljuje neuspjeh iCLI

završava s nestandardnim izlazom, umjesto da prijavi tihi uspjeh. U zapisniku se navodi naziv proizvoda koji nije mogao biti napisan i razlog. Namjerno pokretanje samo za metapodatke (bez zahtjeva) i dalje je uspješno.
{% endhint %}

### Izvor reflektancije (postavka projekta, postavlja se putemCLI

/SDK

)

Projekt također pohranjuje koji **referentni uzorak reflektancije** koji LATTICE proizvod reflektancije koristi. U panelu postavki nema posebne kontrole; vrijednost se pohranjuje u konfiguraciji projekta kao `Processing → "Target reflectance source"` i postavlja se pomoću `chloros-cli process --reflectance-source {auto,target,daq}` ili &#x27;SDK

&#x27;s `reflectance_source` parametrom:

* **`auto`** (zadano): cilj kalibracije unutar kadra koji prođe kontrolu kvalitete (QA) postaje apsolutna referenca, a u suprotnom se koristi DAQ-ova donja granica (ρ = πL/E) kada cilj nije prisutan ili QA ne uspije.
* **`target`**: stroga reflektancija vođena ciljem — bez DAQ zamjene.
* **`daq`**: reflektancija autoritativna za DAQ; ciljevi unutar okvira ne koriste se kao referenca.

Pohranjena vrijednost uparuje se bez obzira na velika i mala slova, a nekoliko se pravopisnih varijanti prihvaća kao sinonimi: `target`, `target_image`, `empirical` i `empirical_line` sve znače **target**; `daq`, `dls`, `light_sensor` i `sensor` sve znače**daq**. Sve ostalo — uključujući nepostojeći ključ — rješava se kao**auto**.

Skeniranja cilja po jedinici **mjereno** traže se prema serijskom/QR broju ciljne jedinice, kao `<serial>.csv`, na tri mjesta: u direktoriju navedenom s `--target-reflectance-dir` (pohranjen kao `Processing → "Target reflectance dir"`), vlastitu mapu projekta `target_reflectance/` i putanju u varijabli okruženja `CHLOROS_TARGET_REFLECTANCE_DIR`. Kada za tu jedinicu ne postoji izmjerena skenirana slika, umjesto toga se koristi nominalna objavljena krivulja za ciljni model.

### Debayer metoda

* **Vrsta**: Izbornik za odabir
* **Opcije**:
  * Standardno (brzo, srednja kvaliteta)
  * S obzirom na teksturu (sporo, najviša kvaliteta) \[Chloros

+]
* **Zadano**: Standardno (Brzo, prosječna kvaliteta)
* **Opis**: Odabire algoritam demosaicinga koji se koristi za pretvorbu sirovih podataka senzora s Bayerovim uzorkom u slike u punoj boji. Metoda &quot;Standard (Brzo, prosječna kvaliteta)&quot; pruža optimalan omjer između brzine obrade i kvalitete slike. Metoda &quot;Texture Aware (Sporo, najviša kvaliteta)&quot; \[Chloros

+] koristi visokokvalitetni debayer koji očitava rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji/strojnom učenju (AI/ML), koji uklanja gotovo sav šum iz procesa debayeriranja. Model Texture Aware zahtijeva GPU memoriju (VRAM) za rad. Preporučujemo njegovo korištenje kada imate dostupno &gt;4 GB VRAM-a za bržu obradu.
* **Kada je redak uopće padajući izbornik**: padajući izbornik s dvije opcije pojavljuje se samo kada**oba** uvjeta vrijede — prijavljeni ste s kvalificiranom pretplatomChloros

+, **i** projekt ne sadrži LATTICE snimke. U suprotnom se red prikazuje kao običan tekst `Standard (Fast, Medium Quality)` bez ikakve mogućnosti odabira.
* **Napomena o LATTICE-u**: Ne postoji LATTICE-om obučen model za prepoznavanje tekstura (Texture Aware), te pipeline nameće standardno demosaikiranje za LATTICE kadrove bez obzira na spremljenu vrijednost. Ako dodate LATTICE mapu u projekt u kojem je već bio odabran Texture Aware,Chloros

vraća postavku na Standard umjesto da zadržava zastarjelu vrijednost u `project.json`.

### Minimalni interval ponovne kalibracije

* **Vrsta**: Broj
* **Raspon**: od 0 do 3600 sekundi
* **Zadano**: 0 sekundi
* **Opis**: Postavlja minimalni vremenski interval (u sekundama) između korištenja ciljeva kalibracije. Kada je postavljeno na 0,Chloros

će koristiti svaki otkriveni cilj kalibracije. Kada je postavljeno na višu vrijednost,Chloros

će koristiti samo ciljeve kalibracije koji su odvojeni najmanje tim brojem sekundi, čime se smanjuje vrijeme obrade za skupove podataka s čestim snimanjima ciljeva kalibracije.
* **Kada prilagoditi**:
  * Postavite na 0 za maksimalnu točnost kalibracije kada se uvjeti osvjetljenja mijenjaju
  * Povećajte (npr. na 60-300 sekundi) za bržu obradu kada je osvjetljenje konstantno i imate česte slike ciljeva kalibracije

### Pomak vremenske zone senzora svjetla

* **Vrsta**: Broj
* **Raspon**: -12 do +12 sati
* **Zadano**: 0 sati
* **Opis**: Određuje pomak vremenske zone (u satima od UTC-a) za vremenske oznake podataka sa senzora svjetla, koji se koristi pri usklađivanju zapisa sa senzora svjetla s vremenom snimanja slika. Noviji zapisi `.daq` sadrže vlastito porijeklo vremenske zone, stoga je ovo uglavnom potrebno za starije zapise snimljene u lokalnom vremenu.

### Primjena PPK korekcija

* **Vrsta**: Polje za potvrdu
* **Zadano**: Onemogućeno (neoznačeno)
* **Opis**: Omogućuje upotrebu postprocesiranih kinematičkih (PPK) korekcija sMAPIR

DAQ snimača koji sadrže GPS (GNSS). Kada je omogućeno,Chloros

će koristiti sve .daq datoteke dnevnika koje sadrže podatke o ekspoziciji u mapi vašeg projekta i primijeniti precizne geolokacijske korekcije na vaše slike.
* **Zahtjev**: .daq log datoteka s unosima za ekspozicijske igle mora biti prisutna u direktoriju vašeg projekta
* **Kada omogućiti**: Preporučuje se uvijek omogućiti PPK korekciju ako imate unose za povratne informacije o ekspoziciji u svojoj .daq log datoteci.

### Pin za ekspoziciju 1

* **Vrsta**: Izbornik za odabir
* **Vidljivost**: Vidljivo samo kada je omogućeno &quot;Primijeni PPK korekcije&quot; I kada su dostupni podaci o ekspoziciji za Pin 1
* **Opcije**:
  * Nazivi modela kamera otkriveni u projektu
  * &quot;Ne koristi&quot; - Ignorirajte ovaj ekspozicijski pin
* **Zadano**: Automatski odabrano na temelju konfiguracije projekta
* **Opis**: Dodjeljuje određenu kameru na ekspozicijski pin 1 za PPK vremensku sinkronizaciju. Ekspozicijski pin bilježi točno vrijeme kada se okida zatvarač kamere, što je ključno za točnu PPK geolokaciju.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedan pin: Automatski odabire kameru
  * Jedna kamera + dva pina: Pin 1 se automatski dodjeljuje kameri
  * Više kamera: Potrebna je ručna selekcija

### Pin za ekspoziciju 2

* **Vrsta**: Izbornik za odabir
* **Vidljivost**: Vidljivo samo kada je omogućen &quot;Primijeni PPK korekcije&quot; I kada su dostupni podaci o ekspoziciji za Pin 2
* **Opcije**:
  * Nazivi modela kamera otkriveni u projektu
  * &quot;Ne koristi&quot; - Ignoriraj ovaj pin za ekspoziciju
* **Zadano**: Automatski odabrano na temelju konfiguracije projekta
* **Opis**: Dodjeljuje određenu kameru Pin 2 za PPK vremensku sinkronizaciju pri korištenju postavke s dvije kamere.
* **Ponašanje automatskog odabira**:
  * Jedna kamera + jedan pin: Pin 2 se automatski postavlja na &quot;Ne koristi se&quot;
  * Jedna kamera + dva pina: Pin 2 se automatski postavlja na &quot;Ne koristi se&quot;
  * Više kamera: Potrebna je ručna selekcija
* **Napomena**: Ista kamera ne može biti dodijeljena i Pin 1 i Pin 2 istovremeno.***

## DAQ senzor svjetla

Ovaj odjeljak se pojavljuje u Postavkama projekta i prikazuje svaku DAQ datoteku za dolje usmjereno svjetlo u projektu — snimke `.daq` i zapisnike DAQ-M `.csv` za dolje usmjereno svjetlo. Zapisi napravljeni na kartici Svjetlosni senzori automatski se dodaju u otvoreni projekt.



<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Svaki red prikazuje datoteku, model senzora i korekciju difuzora-kapice koja je trenutačno na snazi za tu datoteku. Iznad redova nalazi se jedna kontrola za cijeli projekt:

### Prepisivanje kapice (sve datoteke)

* **Vrsta**: Izbornik za odabir
* **Opcije**: `Auto` plus profili korekcije kapa koji su važeći za vrste senzora prisutne u projektu
* **Zadano**: Auto
* **Pohranjeno kao**: `Processing → "DAQ cap id"` (zadano `auto`)
* **Opis**: `Auto` koristi snimljeni profil korekcije svake datoteke (pretpostavlja se profil korekcije &#x27;Sunshine&#x27; ako ništa nije snimljeno — svi DAQ-ovi tvrtkeMAPIR

isporučuju se s korektorom &#x27;Sunshine&#x27;). Odabir određenog korektora nadjačava **sve** datoteke snimanja u projektu: sirova snimanja korigiraju se njime, a snimanja koja već sadrže korektor ponovno se referenciraju (otklanja se zabilježena korekcija i primjenjuje odabrana).
* **Važno**: Odabrani korektor mora odgovarati onome koji je bio fizički postavljen tijekom snimanja. Ni senzor ni softver ne mogu detektirati fizički korektor — neusklađeni ID korektora pogrešno ispravlja spektre.

Namjerno postoji **jedna** kontrola na razini cijelog projekta umjesto padajućih izbornika za svaku datoteku: postavka se primjenjuje na sve downwelling izvore u projektu.***

## Poravnanje nizova

Ovaj se odjeljak prikazuje **samo** kada barem jedna slika u projektu sadrži transformaciju poravnanja s modula na modul koju nizovi LATTICE utiskuju pri snimanju (oznake XMP `Chloros:Alignment*`). Prikazuje koliko slika sadrži oznake poravnanja, koja je kamera referentna (oznaka `REF`), te tablicu po kamerama s brojem slika.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Primijeni poravnanje niza

* **Vrsta**: Polje za potvrdu
* **Zadano**: Omogućeno (označeno)
* **Pohranjeno kao**: `Processing → "Array alignment"`
* **Opis**: Prilagođava svaki obrađeni proizvod (debayered / pretpregled / radijansa / reflektancije / indeks) zajedničkoj referentnoj geometriji niza koristeći transformaciju zabilježenu pri snimanju. Isključeno = izvoz u nativnoj geometriji po senzoru.

### Obrezivanje na zajednički preklap

* **Tip**: Potvrdni okvir (aktivno samo dok je *Primijeni poravnanje niza* uključeno)
* **Zadano**: Omogućeno (označeno)
* **Pohranjeno kao**: `Processing → "Array alignment crop"`
* **Opis**: Obrezuje poravnate izvoze na regiju koju dijele svi moduli kamera, tako da svaki pojas ima isti otisak. Isključeno zadržava cijelo platno senzora (crno popunjavanje izvan izvora).

### Ponovno uzorkovanje

* **Vrsta**: Izbornik (aktivno samo dok je *Primijeni poravnanje niza* uključeno)
* **Opcije**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Zadano**: Bilinearno
* **Pohranjeno kao**: `Processing → "Array alignment interpolation"`
* **Opis**: Interpolacija koju koristi transformacija poravnanja. Opcija &quot;Nearest&quot; (Najbliže) zadržava točne vrijednosti izvora (bez miješanja između piksela) za strogu radiometrijsku analizu; Bilinear je najbolji za mapiranje i vizualnu upotrebu.

Ista tri opcije postoje i u načinu rada bez glave kao `chloros-cli process --array-alignment`, `--array-alignment-crop` i `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Ove postavke omogućuju konfiguraciju multispektralnih indeksa za analizu i vizualizaciju.

### Dodaj indeks

* **Vrsta**: Poseban panel za konfiguraciju indeksa
* **Opis**: Otvara interaktivni panel na kojem možete odabrati i konfigurirati multispektralne indekse vegetacije (NDVI

,NDRE

,EVI

itd.) za izračun tijekom obrade slike. Možete dodati više indeksa, svaki sa svojim postavkama vizualizacije.
* **Dostupni indeksi**: Spuštajući izbornik GUI-ja uključuje**27** unaprijed definiranih formula za višespektralne indekse (pogledajte [Formule za višespektralne indekse](multispectral-index-formulas.md) za potpuni popis, uključujući imena koja također prihvaćajuCLI

/SDK

`--indices` opcija).
* **Značajke**:
  * Odaberite unaprijed definirane formule indeksa
  * Povucite filtarske kanale vaše kamere na utore za pojaseve formule
  * Konfigurirajte bojne gradijente vizualizacije (LUT - tablice dohvata)
  * Postavite pragove i načine obrezivanja
  * Izradite prilagođene indeksne formule
* **Napomena**: Indeksi se ne izračunavaju za jednobandne mono kamere LATTICE M3M — višebandni indeksi nisu definirani za jednu bandu. ModeliSurvey3

i LATTICE M3C nisu pogođeni.



<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Svaki indeks koji dodate prikazuje svoju formulu kao matematičku izraz, s obojenim krugom za svako mjesto u formuli: crvena =Red

, zelena =Green

, plava =Blue

, narančasta =Orange

, plavozelena =Cyan

, ljubičasta =NIR

, magenta = RE. Povucite krug iz retka iznad formule na mjesto u formuli kako biste ga povezali; dvaput kliknite na povezano mjesto u formuli kako biste ga očistili. Indeks se izračunava samo jednom kada svaki slot koji formula koristi ima kanal.

### Prilagođene formule (funkcija za pretplatnike naChloros

+)

* **Tip**: Niz definicija prilagođenih formula
* **Dostupnost**: Zahtijeva prijavu s kvalificiranom pretplatom naChloros

+.
* **Opis**: Omogućuje vam stvaranje i spremanje prilagođenih formula indeksa za višespektralne slike pomoću aritmetike pojaseva. Prilagođene formule spremaju se s postavkama vašeg projekta i mogu se koristiti kao i ugrađeni indeksi.
* **Kako stvoriti**:
  1. U konfiguracijskom panelu indeksa otvorite kalkulator prilagođenih formula.
2. Napišite formulu koristeći **simbole utora pojaseva**, a ne nazive pojaseva.
3. Spremite formulu s opisnim nazivom — tada se pojavljuje na dnu padajućeg izbornika formula, a vi povlačite krugove kanala svoje kamere na njezine utore točno kao i kod ugrađenih predinstalacija.
* **Syntaxa formule**:
* Utori za pojas: `x`, `y`, `z`, `a`, `b`, `c` — šest položaja koje mapirate na stvarne kanale povlačenjem
  * Operatori: `+`, `-`, `*`, `/`, `^` i `()` za grupiranje
  * Funkcije: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Zašto simboli, a ne imena bendova**: formula napisana kao `(y-x)/(y+x)` radi na bilo kojoj kameri, jer mapiranje povuci-i-ispusti odlučuje je li `y` 850 nmNIR

filteraRGN

ili 808 nmNIR

filteraOCN

. Ugrađeni presetovi pohranjuju se na isti način — vidi [Multispektralne formule indeksa](multispectral-index-formulas.md) za točan simbolički oblik svih 27.
* **Gdje rade**: prilagođene formule spremaju se s postavkama projekta i mogu se koristiti u [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) kao i u obradi.**Ne** se prihvaćaju u popisu imenaCLI

/SDK

`--indices`, koji samo proširuje 22 ugrađena naziva predložaka.

***

## Izvoz

Ove postavke kontroliraju format i kvalitetu izvezanih obrađenih slika.

### Kalibrirani format slike

* **Vrsta**: Izbor iz padajućeg popisa
* **Opcije**:
  * **TIFF

(16-bitni)** - Nekomprimirani 16-bitni formatTIFF


  * **TIFF

(32-bitno, postotak)** - 32-bitni format s pomičnim zarezom s vrijednostima reflektancije u postotcima
  * **PNG

(8-bitno)** - komprimirani 8-bitni formatPNG


  * **JPG (8-bitno)** - komprimirani 8-bitni formatJPEG


* **Zadano**:TIFF

(16-bit)
* **Opis**: Odabire format datoteke za spremanje obrađenih i kalibriranih slika. Izvozi zemljište u podmapu za svaki format unutar mape svake kamere (`tiff16`, `tiff32`, `png8`, `jpg8`), s jednom mapom `<Product>_Images/` po proizvodu. Izvezeni datoteci zadržavaju izvorni naziv datoteke — proizvod se identificira mapom, a ne nastavkom naziva datoteke.
* **Preporuke za formate**:
  * **TIFF

(16-bitni)**: Preporučuje se za znanstvenu analizu i profesionalne radne procese. Očuvava maksimalnu kvalitetu podataka bez artefakata kompresije. Najbolje za multispektralnu analizu i daljnju obradu u GIS softveru.
  * **TIFF

(32-bita, postotak)**: Najbolje za radne procese koji zahtijevaju vrijednosti reflektancije kao postotke (0-100 %). Nudi maksimalnu preciznost za radiometrijska mjerenja.
  * **PNG

(8-bit)**: Dobar za pregled na webu i opću vizualizaciju. Manje veličine datoteka s nekompresijom bez gubitaka, ali smanjen dinamički raspon.
  * **JPG (8-bit)**: Najmanje veličine datoteka, najbolje samo za pregled i prikaz na webu. Koristi kompresiju s gubicima koja nije pogodna za znanstvenu analizu.
* **Napomena**: Radijansa LATTICE uvijek se izvozi kao 32-bitna plutajuća točkaTIFF

bez obzira na ovu postavku.

***

## Spremi predložak projekta

Ova značajka omogućuje vam spremanje trenutnih postavki vašeg projekta kao ponovno upotrebljivog predloška.

* **Vrsta**: Unos teksta + gumb za spremanje
* **Opis**: Unesite opisni naziv za svoj predložak postavki i kliknite ikonu spremanja. Predložak će pohraniti sve vaše trenutne postavke projekta (detekcija ciljeva, opcije obrade, indeksi i format izvoza) za jednostavno ponovno korištenje u budućim projektima. Predlošci se pohranjuju u mapu `Project Templates/` unutar mape za spremanje vašeg projekta i mogu se također odabrati ili izvesti iz glavnog izbornika (*Odabir predloška* / *Snimanje predloška* / *Izvoz predloška*).
* **Primjeri upotrebe**:
  * Izrada predložaka za različite sustave kamera (RGB

, multispektralne,NIR

)
  * Snimanje standardnih konfiguracija za određene vrste usjeva ili tijekove rada analize
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

* **Vrsta**: Prikaz putanje + gumb za uređivanje
* **Zadano (Windows

)**: `C:\Users\[Username]\Chloros Projects`
* **Zadano (Linux

)**: `~/Chloros Projects`
* **Opis**: Prikazuje trenutni zadani direktorij u kojem se stvaraju novi projekti zaChloros

. Kliknite ikonu za uređivanje kako biste odabrali drugi direktorij. Preusmjeravanje se pohranjuje kao jedan redak teksta u `~/.chloros/working_directory.txt` — naWindows

, koji je `C:\Users\<Username>\.chloros\working_directory.txt`. Ako ta datoteka nedostaje, ili naznačena je staza koja više ne postoji,Chloros

prelazi na gore navedenu zadanu vrijednost.CLI

čita i zapisuje istu datoteku, tako da se `chloros-cli` i GUI uvijek slažu o lokaciji projekata.
* **Predlošci projekata** nalaze se u podmapu `Project Templates/` ovog direktorija.
* **Kada promijeniti**:
  * Postavite na mrežni pogon za timsku suradnju
  * Promijenite na pogon s više prostora za pohranu za velike skupove podataka
  * Organizirajte projekte po godini, klijentu ili vrsti projekta u različitim mapama
* **Napomena**: Promjena ove postavke utječe samo na NOVE projekte. Postojeći projekti ostaju na svojim izvornim lokacijama.***

## Održavanje postavki

ProjektChloros

je **mapa**. Sve postavke projekta spremaju se unutar nje u `project.json`; povezani hardver pamti se uz njega u `cameras.json` i `sensors.json`, tako da ponovno otvaranje projekta također ponovno povezuje njegove kamere i senzore svjetla. Kada ponovno otvorite projekt, sva se postavke vraćaju točno onako kako ste ih ostavili. Spremljene projekte također je moguće upravljati bez zaslona pomoću uređaja `chloros-cli project` ili `open_project` naSDK

u.

### Hijerarhija postavki

Postavke se primjenjuju sljedećim redoslijedom:

1. **Sustavne zadane postavke** - Ugrađene zadane postavke definirane uChloros

2. **Postavke predloška** - Ako učitavate predložak prilikom izrade projekta
3. **Spremljene postavke projekta** - Postavke spremljene s datotekom projekta
4. **Ručne prilagodbe** - Sve promjene koje napravite tijekom trenutne sesije

### Postavke i obrada slika

Postavke obrade čitaju se kada se pokrene obrada. Promjena postavke ne utječe retroaktivno na proizvode koji su već spremljeni na disk — ponovno pokrenite obradu kako biste primijenili nove postavke. Nekoliko postavki nikada uopće ne utječe na obradu:

* Rezolucija minijature slike (samo za prikaz)
* Spremi predložak projekta
* Spremi mapu projekta

***

## Referenca konfiguracijskih ključeva

Za automatizaciju (CLI

`--config`,SDK

`configure`, ili čitanje `project.json` izravno), ovo su točne ključne riječi pod `Project Settings`:

| Putanja ključa | Tip | Zadano |
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
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Ključevi `Array alignment` upisuju se prvi put kada se prikazuje odjeljak poravnanja niza ili kada ih postavi poziv automatizacije. Dok su oni odsutni, pipeline koristi iste vrijednosti prikazane gore (`true`, `true`, bilinear), stoga se projekt.json bez njih ponaša identično onome s njima.

### Ključevi pohranjeni u `project.json` bez kontrole u panelu postavki

Oni se nalaze pod istim `Project Settings` stablom i čitaju se tijekom obrade, ali nećete pronaći widget za njih u bočnoj traci:

| Put do ključa | Tip | Zadano | Postavlja |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`,SDK

`input_level=`. Prepisuje način na koji se interpretiraju LATTICE ulazni TIFF-ovi; `auto` zaključuje iz svake datoteke `Chloros:ProcessingLevel` XMP oznaka plus broj kanala. Zanemaruje se zaSurvey3

`.raw` snimke. Namjerno nije postavka GUI-ja — auto je ispravno u svakom normalnom slučaju. |
| `Processing → Target reflectance dir` | putanja | `""` | `chloros-cli process --target-reflectance-dir`, ili projektna metaAPI

|
| `Processing → Target reflectance config` | rječnik indeksiran serijskim brojem kamere | `{}` | Registracija cilja unutar kadra (način rada `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | putanja | `""` |SDK

`process_folder(daq_log_path=…)`. Ukazuje na snimku `.daq` ili na mapu sa snimkama |
| `Target Detection → Minimum calibration target squares` | broj | `4` | Naslijeđena zadana postavka; bez kontrole i bez zastaviceCLI

|
| `UI → Grid thumbnail size` | broj | `160` | Klizač zumiranja vlastitih minijatura mreže slika |

Dvije postavke preglednika pohranjuju se **na najvišoj razini u `project.json`**, potpuno izvan `Project Settings`, jer su to stanja prikaza, a ne postavke obrade:

| Put do ključa | Tip | Zadano | Postavlja |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | cijeli broj 1–256 | `1` | Kontrola GSD-a (px) kartice slike — vidi [Otvaranje slike na cijelom zaslonu](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Najbolje prakse

1. **Počnite s postavkama zadane vrijednosti**: Postavke zadane vrijednosti dobro funkcioniraju za većinu sustava kameraMAPIR

i uobičajenih tijekova rada.
2. **Stvorite predloške**: Nakon što ste optimizirali postavke za određeni tijek rada ili kameru, spremite ih kao predložak kako biste osigurali dosljednost u svim projektima.
3. **Testirajte prije potpune obrade**: Kada eksperimentirate s novim postavkama, testirajte ih na malom skupu slika prije obrade cijelog skupa podataka.
4. **Dokumentirajte svoje postavke**: Koristite opisne nazive predložaka koji navode sustav kamere, vrstu obrade i namjenu (npr. &quot;Survey3

\_RGB\_NDVI\_Poljoprivreda&quot;).
5. **Odabir formata izvoza**: Odaberite format izvoza na temelju konačne namjene:
   * Znanstvena analiza →TIFF

(16-bitni ili 32-bitni)
   * GIS obrada →TIFF

(16-bitni)
   * Brza vizualizacija →PNG

(8-bitni)
   * Dijeljenje na webu → JPG (8-bit)

***

Za više informacija o multispektralnim indeksima u aplikacijiChloros

pogledajte stranicu [Multispectral Index Formulas](multispectral-index-formulas.md).
