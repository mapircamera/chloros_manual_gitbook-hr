# Završetak obrade

Nakon što Chloros dovrši obradu, vrijeme je da pregledate svoje rezultate, provjerite kvalitetu izlaza i pripremite svoje obrađene slike za upotrebu u svom radnom procesu. Ova stranica vodi vas kroz završne korake i sljedeće radnje.

## Indikator završetka obrade

Kada se obrada uspješno završi, vidjet ćete nekoliko indikatora:

* ✅ **Traka napretka**: Dostiže 100 % dovršenosti
* ✅ **Zapisnik za otklanjanje pogrešaka**: Prikazuje konačne retke `[RUN-SUMMARY]` s brojevima (slike, grupe kamera, ciljevi, kalibrirane slike, zapisane datoteke)
* ✅ **Gumb za pokretanje**: Ponovno postaje aktivan (spreman za sljedeću obradu)
* ✅ **Izlazne datoteke**: Sve obrađene slike spremljene su u izlazno stablo projekta (dolje)

{% hint style="warning" %}
**Pokretanje koje ne piše nijednu sliku predstavlja neuspjeh.** Ako ste zatražili izlazne slike, a izvršavanje nije napisalo nijednu, Chloros to prijavljuje kao neuspjeh — `[RUN-SUMMARY]` naznake u nazivu dnevnika navode vjerojatni uzrok (ništa nije uvezano, nije otkriven cilj ili je svaki zatraženi proizvod preskočen kao neprimjenjiv). Ekvivalent u CLI izlazi s ne-nultim rezultatom. Namjerno pokretanje samo za metapodatke (svi proizvodi za izvoz isključeni, bez indeksa) i dalje se smatra uspjehom. Pogledajte [referencu za CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Pronalaženje obrađenih slika

### Otvaranje izlazne mape

1. Kliknite ikonu **Glavni izbornik** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(gornji lijevi kut)
2. Odaberite **"Otvori mapu projekta"**

3. Vaš preglednik datoteka otvara se u direktorij projekta
4. Pronađite svoj projekt po nazivu

### Stablo izlaznih datoteka

Proizvodi se spremaju **ispod mape projekta, grupirani po kameri, a zatim po formatu datoteke**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Mapa kamere**: `LATT-<sensor>-<lens>-F<filter>` za LATTICE (podudaranje s EXIF-om snimke `Model`), `<model>_<filter>` za Survey3 (npr. `Survey3N_RGN`). Dvije kamere koje dijele senzor i filtar, ali se razlikuju po objektivu, vode odvojena stabla — vinjeta, vidno polje i izobličenje se razlikuju.
* **Format mape**: slijedi vašu postavku formata izvoza — `tiff16`, `tiff8`, `png8`, `jpg8`, ili `tiff32` za TIFF (32-bitni, Percent). Radiance je uvijek float32 i uvijek se nalazi pod `tiff32`.
* **Mape proizvoda**:
  * `Reflectance_Calibrated_Images/` — kalibrirana reflektancija
  * `Debayered_Images/` — linearno debayered (LATTICE): uklonjena je tamna kompenzacija i demosaicirano, bez kalibracije ravnog polja ili unmixa, u nativnoj orijentaciji senzora
  * `Preview_Images/` — pretpregled za prikaz (LATTICE)
  * `Radiance_Images/` — float32 spektralna zračnost, W/m²/sr/nm (LATTICE multispektralni)
  * `Vignette_Corrected_Images/` **ili** `Sensor_Response_Images/` — nekadibrirana alternativa za kadrove bez referentne odbijivosti; točno jedna od te dvije postoji po izvršenju, a odabire se postavkom korekcije Vignette
  * `<INDEX>_Index_Images/` — jedan mapu po odabranom indeksu (npr. `NDVI_Index_Images`)
* **`clamp_masks/`** (novo u verziji 1.3.0, samo za LATTICE multispektralne M3C kamere): pojavljuje se u mapi formata, pored mapa proizvoda, kada se u proizvodu radijance ili reflektance klipirane negativne vrijednosti pojasa pomiješaju i postanu nula. Sadrži jednu masku po pikselu za svaki zahvaćeni proizvod, npr. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Maske nikada nisu unutar mape proizvoda, pa se mapa proizvoda može predati softveru za fotogrametriju u izvornom obliku. Vidi [Clamp masks](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Svaki izvezeni proizvod zadržava naziv izvornog datoteka.**Radijantni izvoz `capture_..._raw.tif` i dalje se naziva `capture_..._raw.tif` — samo se nalazi u `tiff32/Radiance_Images/`.**Mapu identificira proizvod, a ne naziv datoteke**, stoga pretraživanje za `*radiance*.tif` neće ništa pronaći; umjesto toga, uskladite pretraživanje s direktorijem.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Koliko datoteka bi trebalo biti?

Ne računajte prema formuli — broj izlaznih datoteka ovisi o tome koji su proizvodi omogućeni i koji se primjenjuju na svaku kameru (npr. kamere sRGB
bojama ne dobivaju radijanciju/reflektanciju). Konačan broj nalazi se u log datoteci: zadnja linija `[RUN-SUMMARY]` izvještava o točnom broju zapisanih datoteka, a pomoćne linije objašnjavaju sve što je preskočeno.

***

## Pregled obrađenih slika

### Brzi pregled u Upravitelju datoteka

**Ugrađeni pregled sustava Windows:**

1. Idite u mapu proizvoda (npr. `tiff16/Reflectance_Calibrated_Images/`)
2. Odaberite datoteku s slikom
3. Prikaz se pojavljuje u pretpreglednom oknu istraživača Windows Explorer
4. Koristite tipke sa strelicama za pregled slika

### Prikaz u vanjskim preglednicima slika

**Preporučeni preglednici:*** **QGIS** - Besplatan GIS softver (najbolji za georeferenciranu multispektralnu analizu)
* **IrfanView** - Brzi, lagani preglednik slika (podržava TIFF)
* **Adobe Photoshop** - Profesionalno uređivanje (podrška za TIFF)
* **GIMP** - Besplatna alternativa za Photoshop
* **Windows Photos** - Osnovno pregledavanje (može ne podržavati 16-bitni TIFF)

### Pregled u pregledniku slika [Chloros]

Koristite ugrađeni preglednik slika programa [Chloros] za naprednu vizualizaciju:

1. Kliknite na minijaturu slike u pregledniku datoteka
2. Slika se otvara u glavnom području za pregled
3. Kliknite na karticu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
u lijevoj bočnoj traci
4. Koristite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) za interaktivnu analizu

Pogledajte [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) za detaljne upute.

***

## Čitanje vrijednosti reflektancije piksela (GIS / Pix4D / Skripte)

Reflekancija se pohranjuje kao cjelobrojni DN, a **DN koji označava ρ = 1,0 ovisi o izvornoj kameri**:

| Izvor          | ρ = 1,0 je | Kako odrediti                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (rezerva do ρ 2.0) | XMP oznaka `Chloros:PixelScale=32768` je zapečaćena na datoteci |
| Survey3          | **65535** (ograničeno na ρ 1.0)     | Nema oznake `Chloros:PixelScale` — ta odsutnost je signal (Survey3 proizvodi imaju druge oznake obrade `Chloros:*`) |

Pročitajte `Chloros:PixelScale` oznaku i podijelite po njoj** umjesto da pretpostavite univerzalnu vrijednost 65535 — dijeljenje LATTICE reflektancije s 65535 tiho prepolovljava svaku vrijednost. Datoteka reflektancije iz verzije prije 1.3.0, zapisana kao 8-bitna, bila je podrezana, a ne preskalirana, i nema oznaku skale: ponovno je izvezite u 16-bitnoj ili 32-bitnoj formi umjesto da je dijelite. Pogledajte [Output Image Formats](../output-image-formats.md) za cijelu priču.***

## Metapodaci preneseni u izvoze

Svaki proizvod zadržava **GPS blok**izvornog snimanja i njegov**EXIF pod-IFD**, stoga izvoz prenosi `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` i
`CameraSerialNumber`, kao i georeferenciranje.

{% hint style="warning" %}
**Ako ortomosaika ispadne u apsurdnoj mjerilu, prvo provjerite `FocalLength`.**
Pix4D izračunava udaljenost uzorka tla (ground sample distance) na temelju žarišne duljine i visine. Bez oznake prelazi na potpuno pogrešnu mjerilu — na jednom zadanom letu s 49 snimaka, veličine 411 m × 160 m
nasan sadnja naranči rekonstruirana je kao 47,8 km × 13 km, stvarajući orto-snimku od 455 megapiksela koja se uglavnom sastoji od praznog prostora. Sporo slaganje i neočekivano velika datoteka simptomi su toga, a ne zasebni problemi.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Ne kopira se *svaka* oznaka. Strukturne oznake IFD0-a namjerno se izostavljaju (njihovo kopiranje kvari LATTICE izlaz), i `ExifImageWidth` / `ExifImageHeight` su isključeni jer opisuju izvornu snimku — izvoz bi inače tvrdio dimenzije koje su u suprotnosti s njegovim vlastitim rastrom.

Chloros također dodaje vlastite XMP oznake `Chloros:*` koje opisuju što je učinjeno svakom proizvodu (na primjer `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Od verzije 1.3.0, proizvodiSurvey3
nose ih u svakoj obradi
način, indeksni proizvodi opisuju sam indeks (`Chloros:ProcessingLevel=index`, formulu i je li izračunat na temelju reflektancije ili radijancije) umjesto da ponavljaju oznake izvornog sirovog materijala, a proizvodi u mapama dubljim od 260 znakova također zadržavaju svoje metapodatke. Vidi [Izlazni formati slika](../output-image-formats.md#index-products).

***

## Pregled dnevnika za otklanjanje pogrešaka

### Provjera upozorenja ili pogrešaka

1. Otvorite karticu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Prelistajte poruke
3. Potražite žute upozorenja ili crvene pogreške
4. Pročitajte retke `[RUN-SUMMARY]` i sve naznake
5. Za pomoć kontaktirajte podršku tvrtke MAPIR


### Spremanje dnevnika

Kako biste sačuvali zapis obrade ili ga poslali podršci tvrtke MAPIR:

1. Kliknite gumb **"Kopiraj"**ili**"Preuzmi"**

2. Spremite kao tekstualnu datoteku u mapu projekta
3. Priložite uz dokumentaciju projekta
4. Pošaljite podršci tvrtke MAPIR ako naiđete na probleme.

Da biste problem izravno poslali tvrtki MAPIR, upotrijebite ikonu greške (**Prijavi grešku**) na vrhu kartice Zapisnika otklanjanja pogrešaka (Debug Log) — pogledajte [Prijavljivanje greške s kartice zapisnika](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).***

## Uobičajeni problemi s izlaznim datotekama i njihova rješenja

### Problem: Nedostajuće izlazne datoteke

**Mogući uzroci:**

* Proizvod nije primjenjiv na tu kameru (npr. radijansa/reflektancije za kamere u boji RGB — o tome log vodič obavještava)
* Fali potrebna referenca (npr. reflektancija bez cilja i bez `.daq` zračenja prema dolje)
* Kvačica za izvoz proizvoda bila je onemogućena u postavkama projekta
* Tijekom izvoza ponestalo je prostora na disku

**Rješenja:**

1. Provjerite `[RUN-SUMMARY]` naznake i `[EXPORT-CHECK]` retke u zapisniku za otklanjanje pogrešaka (Debug Log) — one objašnjavaju preskakanja po kamerama
2. Provjerite su li potvrdni okviri za izvoz proizvoda označeni u [Postavkama projekta](adjusting-project-settings.md)
3. Provjerite je li prostor na disku dovoljan
4. Ponovno obradite nakon što otklonite uzrok

### Problem: Tamni ili svijetli rubovi (vigneta i dalje vidljiva)

**Mogući uzroci:**

* Onemogućeno ispravljanje vinjete
* Kamera/objektiv nije u bazi profila tvrtke Chloros

* Ekstremna vinjeta koja se ne može ispraviti

**Rješenja:**

1. Provjerite je li ispravljanje vinjete omogućeno u postavkama projekta
2. Provjerite je li model kamere ispravno prepoznat
3. Kontaktirajte podršku tvrtke MAPIR ako se vinjeta i dalje pojavljuje

### Problem: Neispravne boje ili vrijednosti

**Mogući uzroci:**

* Nisu otkriveni ciljevi za kalibraciju
* Odabran je pogrešan model cilja za kalibraciju
* Onemogućena je kalibracija reflektancije
* Slike ciljeva loše kvalitete

**Rješenja:**

1. Provjerite je li kalibracija reflektancije omogućena
2. Provjerite poruke "Target found" (Cilj pronađen) u zapisniku za otklanjanje pogrešaka (Debug Log)
3. Pregledajte kvalitetu slika ciljeva
4. Ponovno obradite s označenim ispravnim ciljevima

### Problem: Vrijednosti NDVI-a izgledaju netočno

**Očekivani rasponi vrijednosti NDVI-a:*** **Voda, stijene, tlo**: -0,1 do 0,2
* **Rijetka/ne zdrava vegetacija**: 0,2 do 0,4
* **Umjerena vegetacija**: 0,4 do 0,6
* **Zdrava, gusta vegetacija**: 0,6 do 0,9**Ako su vrijednosti izvan ovih raspona:**

1. Provjerite je li primijenjena kalibracija reflektancije
2. Provjerite je li uključen zapisnik senzora svjetla
3. Provjerite jesu li otkrivene kalibracijske oznake
4. Provjerite je li otkriven ispravan model kamere
5. Pregledajte vrijeme i uvjete snimanja ciljnih slika
6. Ako sami izračunavate indekse iz datoteka reflektancije, potvrdite da ste podijelili s `Chloros:PixelScale` datoteke (vidi gore)

***

## Korištenje vaših obrađenih slika

### Za fotogrametriju / izradu ortomosaika

**Preporučeni tijek rada:**

1.**Uvezite kalibrirane slike reflektancije** u softver za fotogrammetriju:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Očuvajte EXIF metapodatke**: Osigurajte očuvanje GPS podataka za geotagiranje
3. **Kalibrirani tijekovi rada**: Koristite slike reflektancije za znanstvenu točnost — reflektancija LATTICE-a sadrži XMP oznake kalibracije koje čita Pix4D
4. **Obradite indeksne mozaike**: Izradite ortomozaike NDVI-a iz pojedinačnih indeksnih slika
5. **Izvoz georeferenciranih GeoTIFF datoteka**: Za upotrebu u GIS aplikacijama

### Za GIS analizu

**Preporučeni tijek rada:**

1.**Učitavanje u QGIS, ArcGIS ili slične aplikacije**

2.**Koristite 16-bitne slike reflektancije formata TIFF za višekanalnu analizu (podijelite s `Chloros:PixelScale` datoteke)**

3.**Koristite indeksne slike** (NDVI, NDRE) kao spremne slojeve vegetacije
4. **Rasterski kalkulator**: Kombinirajte pojaseve za prilagođenu analizu
5. **Izvoz**: Izradite karte klasifikacije, karte detekcije promjena, karte zdravlja vegetacije

### Za izravnu analizu / izvještavanje

**Preporučeni tijek rada:**

1.**Koristite indeksne slike s LUT bojama** za vizualna izvješća
2. **Izvadite statistiku**: Prosjek NDVI po polju/parceli
3. **Vremenske serije**: Usporedite indekse kroz više sesija
4. **Generirajte izvješća**: Uključite karte, statistiku i vizualizacije***

## Arhiviranje i sigurnosno kopiranje

### Preporučena strategija sigurnosnog kopiranja

**Što spremiti:*** ✅ **Izvorne RAW/JPG slike ili sirovi LATTICE snimci** - Arhivirati na zasebnom disku/u oblaku; sirovi podaci su izvor cijelog procesa i sve ostalo može se regenerirati iz njih
* ✅ **datoteke senzora svjetla `.daq` / `.csv`** - Potrebne za naknadno izvođenje reflektancije
* ✅ **Obradeni rezultati** - Sačuvajte kalibrirane slike i indekse
* ✅ **Mapa projekta** (`project.json` i pripadajuće datoteke) - Sadrži sve postavke za ponovnu obradu ako je potrebno
* ✅ **Zapisnik o otklanjanju pogrešaka** - Dokumentira detalje obrade
* ✅ **Slike kalibracijskih ciljeva** - Za provjeru i ponovnu obradu**Preporuke za pohranu:*** **Neposredna sigurnosna kopija**: Vanjski tvrdi disk
* **Dugoročni arhiv**: Pohrana u oblaku (Google Drive, Dropbox itd.)
* **Kritični podaci**: Čuvajte 2-3 kopije na različitim lokacijama***

## Sljedeće izvršavanje obrade

### Ponovna upotreba postavki projekta

Ako u budućnosti obrađujete slične skupove podataka:

1. **Spremite predložak projekta** (ako to već niste učinili)
2. **Stvorite novi projekt** koristeći spremljeni predložak
3. **Uvezite nove slike**

4.**Obavite obradu**s identičnim postavkama radi dosljednosti

### Serijska obrada više sesija

Za više sesija/skupova podataka:**Opcija 1: GUI - Više projekata**

*   Izradite zaseban projekt za svaku sesiju
*   Koristite dosljedne postavke predloška
*   Obradite ih jedan po jedan

**Opcija 2: Chloros CLI (Chloros+ samo)**

* Automatizirajte grupnu obradu
* Obradite više mapa pomoću skripti
* Pogledajte [CLI dokumentaciju](../CLI.md) i [CLI referencu](../reference/cli-reference.md)

**Opcija 3: Python SDK (samo za Chloros+)**

* Programatska kontrola
* Integracija s analiznim cjevovodima
* Pogledajte [API dokumentaciju](../api-python-sdk.md) i [SDK referencu](../reference/sdk-reference.md)

***

## Rješavanje problema s naknadnom obradom

### Ponovna obrada s različitim postavkama

Ako rezultati nisu zadovoljavajući:

1. Sačuvajte izvorne slike (nikada ih ne brišite)
2. Otvorite isti projekt u Chloros u 3. Podesite postavke u odjeljku Postavke projekta
4. Ponovno obradite — izlazne datoteke spremaju se u iste mape proizvoda, stoga se datoteke istog naziva iz prethodne obrade zamjenjuju

### Obrada podskupa slika

Za ponovnu obradu samo određenih slika:

1. Kreirajte novi projekt
2. Uvezite samo slike kojima je potrebna ponovna obrada
3. Koristite istu predložak postavki
4. Obradite manji skup podataka

### Potražite pomoć

Ako naiđete na probleme:

* 📧 **E-pošta**: info@mapir.camera (priložite zapisnik o otklanjanju pogrešaka)
* 🌐 **Podrška**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **ČPP**: [Često postavljana pitanja](../faq.md)
* 📖 **Dokumentacija**: [Upute za Chloros](../)

***

## Sažetak: Cjelokupni tijek rada

Sada ste dovršili cjelokupni tijek obrade programa Chloros:

1. ✅ **Stvorili projekt** - Pogledajte [Projekti](../projects.md)
2. ✅ **Dodali datoteke** - Vidi [Dodavanje datoteka](adding-files-to-a-project.md)
3. ✅ **Podesili postavke** - Vidi [Podešavanje postavki projekta](adjusting-project-settings.md)
4. ✅ **Označili ciljeve** - Vidi [Odabir ciljnih slika](choosing-target-images.md)
5. ✅ **Pokrenuta obrada** - Vidi [Pokretanje obrade](starting-the-processing.md)
6. ✅ **Pratio se napredak** - Vidi [Praćenje obrade](monitoring-the-processing.md)
7. ✅ **Pregledanih rezultata** - Ova stranica**Vaše kalibrirane, refleksancijom korigirane multispektralne slike spremne su za analizu!**

***

## Dodatni resursi

### Napredne značajke

* [**Preglednik slika**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktivna vizualizacija i analiza
* [**Sandbox za indekse/LUT-ove**](../image-viewer-gui/index-lut-sandbox.md) - Testiranje prilagođenih indeksa
* [**Formule za višespektralne indekse**](../project-settings/multispectral-index-formulas.md) - Potpuna referenca za indekse

### Automatizacija i integracija

* [**Dokumentacija za CLI**](../CLI.md) - Serijska obrada putem naredbenog retka
* [**Python SDK**](../api-python-sdk.md) - Programatska automatizacija
* [**Značajke sustava Chloros+**](../#chloros) - Napredne mogućnosti obrade

### Podrška i učenje

* [**ČPP**](../faq.md) - Odgovori na česta pitanja
* [**Ciljevi kalibracije**](../calibration-targets.md) - Razumijevanje kalibracije reflektancije
* [**Podržane kamere**](../supported-cameras.md) - Kompatibilan hardver
