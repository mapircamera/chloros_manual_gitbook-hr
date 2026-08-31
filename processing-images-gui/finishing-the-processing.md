# Završetak obrade

Nakon štoChloros
dovrši obradu, vrijeme je da pregledate svoje rezultate, provjerite kvalitetu izlaza i pripremite svoje obrađene slike za upotrebu u svom radnom procesu. Ova stranica vodi vas kroz završne korake i sljedeće radnje.

## Indikator završetka obrade

Kada se obrada uspješno završi, vidjet ćete nekoliko indikatora:

* ✅ **Traka napretka**: Dostiže 100 % dovršenosti
* ✅ **Zapisnik o pogreškama (Debug Log)**: Prikazuje konačne retke `[RUN-SUMMARY]` s brojevima (slike, grupe kamera, ciljevi, kalibrirane slike, zapisane datoteke)
* ✅ **Gumb za pokretanje**: Ponovno postaje aktivan (spreman za sljedeću obradu)
* ✅ **Izlazne datoteke**: Sve obrađene slike spremljene su u izlazno stablo projekta (dolje)

{% hint style="warning" %}
**Pokretanje koje ne piše nijednu sliku je neuspjeh.** Ako ste zatražili slikovne proizvode, a pokretanje nije napisalo nijedan,Chloros
to prijavljuje kao neuspjeh — `[RUN-SUMMARY]` naznake u nazivu dnevnika navode vjerojatni uzrok (ništa uvezano, nije detektiran cilj ili je svaki zatraženi proizvod preskočen kao neprimjenjiv). EkvivalentCLI
izlazi s ne-nulovitim rezultatom. Namjerno pokretanje samo za metapodatke (svi izvozni proizvodi isključeni, bez indeksa) i dalje se smatra uspješnim. Pogledajte [referencu zaCLI
](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Pronalaženje obrađenih slika

### Otvaranje izlaznog mapa

1. Kliknite ikonu **Glavni izbornik**<img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(gornji lijevi kut)
2. Odaberite **&quot;Open Project Folder&quot;**

3. Otvorit će se vaš preglednik datoteka u direktoriju projekta
4. Pronađite svoj projekt po nazivu

### Izlazno stablo

Proizvodi se spremaju **ispod direktorija projekta, grupirani po kameri, a zatim po formatu datoteke**:

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

* **Mapa kamere**: `LATT-<sensor>-<lens>-F<filter>` za LATTICE (odgovara EXIF-u snimke `Model`), `<model>_<filter>` zaSurvey3
(npr. `Survey3N_RGN`). Dvije kamere koje dijele senzor i filtar, ali se razlikuju po objektivu, drže odvojena stabla — vinjeta, vidno polje i distorzija se razlikuju.
* **Mapa formata**: slijedi vašu postavku formata izvoza — `tiff16`, `tiff8`, `png8`, `jpg8`, ili `tiff32` zaTIFF
(32-bitno, postotak). Radiance je uvijek float32 i uvijek se pohranjuje u `tiff32`.
* **Mape proizvoda**:
  * `Reflectance_Calibrated_Images/` — kalibrirana reflektancija
  * `Debayered_Images/` — linearno debayered (LATTICE)
  * `Preview_Images/` — pregled na zaslonu (LATTICE)
  * `Radiance_Images/` — spektralna zračnost float32, W/m²/sr/nm (LATTICE multispektralni)
  * `Vignette_Corrected_Images/` **ili** `Sensor_Response_Images/` — nekadibrirana rezervna opcija za sličice bez referentne odbijivosti; točno jedna od te dvije postoji po pokretanju, a odabire se postavkom korekcije Vignette
  * `<INDEX>_Index_Images/` — jedna mapa po odabranom indeksu (npr. `NDVI_Index_Images`)

{% hint style="info" %}
**Svaki izvezeni proizvod zadržava naziv izvornog datoteka.**Izvoz radijance `capture_..._raw.tif` i dalje se naziva `capture_..._raw.tif` — samo se nalazi u `tiff32/Radiance_Images/`.**Sadržaj direktorija identificira proizvod, a ne naziv datoteke**, stoga pretraživanje `*radiance*.tif` neće ništa pronaći; umjesto toga, uskladite sadržaj direktorija.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->
### Koliko datoteka bi trebalo biti?

Nemojte brojati prema formuli — broj izlaznih datoteka ovisi o tome koji su proizvodi omogućeni i koji se primjenjuju na svaku kameru (npr. kamereRGB
ne dobivaju radijanciju/reflektanciju). Konačan broj nalazi se u zapisniku: zadnja linija `[RUN-SUMMARY]` izvještava o točnom broju zapisa, a pomoćne linije objašnjavaju sve što je preskočeno.

***

## Pregled obrađenih slika

### Brzi pregled u Upravitelju datoteka

**Ugrađeni pregled uWindows
u:**

1. Idite u mapu proizvoda (npr. `tiff16/Reflectance_Calibrated_Images/`)
2. Odaberite datoteku slike
3. Pregled se pojavljuje u oknu za pretpregled Upravitelja datoteka
4. Koristite tipke sa strelicama za pregledavanje slika

### Pregled u vanjskim preglednicima slika

**Preporučeni preglednici:*** **QGIS** - Besplatan GIS softver (najbolji za georeferenciranu multispektralnu analizu)
* **IrfanView** - Brz, lagan preglednik slika (podržavaTIFF
)
* **Adobe Photoshop** - Profesionalna obrada (podrška zaTIFF
)
* **GIMP** - Besplatna alternativa za Photoshop
* **Windows
Photos** - Osnovno pregledavanje (može ne podržavati 16-bitneTIFF
)

### Pregled u pregledniku slikaChloros


Koristite ugrađeni preglednik slikaChloros
za naprednu vizualizaciju:

1. Kliknite na minijaturu slike u pregledniku datoteka
2. Slika se otvara u glavnom području za pretpregled
3. Kliknite na karticu **Image Viewer**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
u lijevoj bočnoj traci
4. Koristite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) za interaktivnu analizu

Pogledajte [Prikazivač slika](../image-viewer-gui/opening-an-image-full-screen.md) za detaljne upute.

***

## Čitanje vrijednosti piksela reflektancije (GIS / Pix4D / Skripte)

Odbijanje se pohranjuje kao cjelobrojni DN, a **DN koji označava ρ = 1.0 ovisi o izvornoj kameri**:

| Izvor          | ρ = 1.0 je | Kako odrediti                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (rezerva do ρ 2.0) | XMP oznaka `Chloros:PixelScale=32768` je upisana u datoteku |
|Survey3
         | **65535** (skraćeno na ρ 1.0)     | Nema XMP oznaka `Chloros:*` — ta odsutnost je signal |

Pročitajte XMP oznaku `Chloros:PixelScale` i podijelite po njoj** umjesto da pretpostavite univerzalnu vrijednost 65535 — dijeljenje LATTICE refleksivnosti s 65535 tiho prepolovljava svaku vrijednost. Jedan rubni slučaj po dizajnu ne uključuje skaliranje: snimanje iz 8-bitnog izvora zabilježeno kao 8-bitni izlaz se obrezuje, a ne skalira, i namjerno ne dobiva oznaku skale — ponovno izvezite u 16-bitnom ili 32-bitnom formatu umjesto da ga dijelite. Pogledajte [Output Image Formats](../output-image-formats.md) za cijelu priču.***

## Metapodaci preneseni u izvoze

Svaki proizvod zadržava **GPS blok**izvornog snimanja i njegov**EXIF pod-IFD**, tako da izvoz prenosi `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` i
`CameraSerialNumber` kao i georeferenciranje.

{% hint style="warning" %}
**Ako ortomosaik ispadne u apsurdnom mjerilu, prvo provjerite `FocalLength`.**
Pix4D rješava udaljenost uzorkovanja tla (ground sample distance) na temelju žarišne duljine i visine. Bez oznake prelazi na potpuno pogrešnu mjerilu — na jednom zadanom letu od 49 snimki, voćnjak naranči dimenzija 411 m × 160 m rekonstruiran je kao 47,8 km × 13 km, čime je proizvedena ortomapa od 455 megapiksela koja se uglavnom sastoji od praznog prostora. Sporo slaganje i neočekivano velika datoteka simptomi su toga, a ne zasebni problemi.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Ne kopira se *svaka* oznaka. Strukturne oznake IFD0-a namjerno se izostavljaju (njihovo kopiranje kvari LATTICE izlaz), a `ExifImageWidth` / `ExifImageHeight` su izostavljeni
jer opisuju originalni snimak — inače bi izvoz promijenjene veličine tvrdio dimenzije koje su u suprotnosti s njegovim vlastitim rastrom.

***

## Pregledavanje dnevnika za otklanjanje pogrešaka

### Provjera upozorenja ili pogrešaka

1. Otvorite karticu **Debug Log**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Prelistajte poruke
3. Potražite žuta upozorenja ili crvene pogreške
4. Pročitajte retke `[RUN-SUMMARY]` i sve naznake
5. Za pomoć kontaktirajte podršku naMAPIR


### Spremanje dnevnika

Kako biste sačuvali zapis o obradi ili ga poslali podršci naMAPIR
:

1. Kliknite gumb **&quot;Kopiraj&quot;**ili**&quot;Preuzmi&quot;**

2. Spremite kao tekstualnu datoteku u mapu projekta
3. Priložite uz dokumentaciju projekta
4. Pošaljite podršci naMAPIR
ako naiđete na probleme

***

## Uobičajeni problemi s izlaznim datotekama i njihova rješenja

### Problem: Nedostaju izlazne datoteke

**Mogući uzroci:**

* Proizvod nije primjenjiv na tu kameru (npr. radijansa/reflektancije za kamereRGB
— o tome log datoteka daje obavijest)
* Nedostajala je potrebna referenca (npr. reflektancija bez cilja i bez `.daq` zračenja prema dolje)
* Kvačica za izvoz proizvoda bila je onemogućena u postavkama projekta
* Tijekom izvoza ponestalo je prostora na disku

**Rješenja:**

1. Provjerite naznake `[RUN-SUMMARY]` i retke `[EXPORT-CHECK]` u dnevniku za otklanjanje pogrešaka (Debug Log) — oni objašnjavaju preskakanja po kamerama
2. Provjerite potvrdne okvire za izvoz proizvoda u [Postavke projekta](adjusting-project-settings.md)
3. Provjerite je li prostor na disku bio dovoljan
4. Ponovno obradite nakon što ste otklonili uzrok

### Problem: Tamni ili svijetli rubovi (vigneting i dalje vidljiv)

**Mogući uzroci:**

* Onemogućeno ispravljanje vinjetiranja
* Kamera/objektiv nije u bazi profila EChloros

* Ekstremno vinjetiranje koje se ne može ispraviti

**Rješenja:**

1. Provjerite je li ispravljanje vinjetiranja omogućeno u postavkama projekta
2. Provjerite je li model kamere ispravno detektiran
3. Ako se vinjetiranje i dalje javlja, obratite se podršci naMAPIR


### Problem: Neispravne boje ili vrijednosti

**Mogući uzroci:**

* Nisu otkriveni ciljevi za kalibraciju
* Odabran je pogrešan model cilja za kalibraciju
* Kalibracija reflektivnosti je onemogućena
* Slike ciljeva loše kvalitete

**Rješenja:**

1. Provjerite je li kalibracija reflektancije omogućena
2. Provjerite poruke &quot;Target found&quot; (Pronađen cilj) u zapisniku za otklanjanje pogrešaka (Debug Log)
3. Pregledajte kvalitetu slika ciljeva
4. Ponovno obradite s označenim ispravnim ciljevima

### Problem: VrijednostiNDVI
izgledaju pogrešno

**Očekivani rasponi vrijednosti reflektancije:*** **Voda, stijene, tlo**: -0,1 do 0,2
* **Rijetka/ne zdrava vegetacija**: 0,2 do 0,4
* **Umjerena vegetacija**: 0,4 do 0,6
* **Zdrava, gusta vegetacija**: 0,6 do 0,9**Ako su vrijednosti izvan ovih raspona:**

1. Provjerite je li primijenjena kalibracija reflektancije
2. Provjerite je li uključen zapisnik senzora svjetla
3. Provjerite jesu li kalibracijske pločice otkrivene
4. Pobrinite se da je ispravan model kamere otkriven
5. Provjerite vrijeme i uvjete snimanja ciljnih slika
6. Ako sami izračunavate indekse iz datoteka reflektancije, potvrdite da ste podijelili s `Chloros:PixelScale` datoteke (vidi gore)

***

## Korištenje vaših obrađenih slika

### Za fotogrametriju / izradu ortomosaika

**Preporučeni tijek rada:**

1.**Uveзите kalibrirane slike reflektancije** u softver za fotogrametriju:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Očuvajte EXIF metapodatke**: Osigurajte očuvanje GPS podataka za geotagiranje
3. **Kalibrirani tijekovi rada**: Koristite slike reflektancije za znanstvenu točnost — reflektancija LATTICE sadrži XMP oznake kalibracije koje Pix4D čita
4. **Obradite mozaike indeksa**: Izradite ortomozaikeNDVI
iz pojedinačnih indeksnih slika
5. **Izvezite georeferenciraneGeoTIFF**: Za upotrebu u GIS aplikacijama

### Za GIS analizu

**Preporučeni tijek rada:**

1.**Učitajte u QGIS, ArcGIS ili sličan program**

2.**Koristite 16-bitne slike reflektancijeTIFF
** za višebandsku analizu (podijelite s `Chloros:PixelScale` datoteke)
3. **Koristite indeksne slike** (NDVI
,NDRE
) kao gotove slojeve vegetacije
4. **Rasterski kalkulator**: Kombinirajte pojaseve za prilagođenu analizu
5. **Izvoz**: Izradite karte klasifikacije, karte detekcije promjena, karte zdravlja vegetacije

### Za izravnu analizu / izvještavanje

**Preporučeni tijek rada:**

1.**Koristite indeksne slike s LUT bojama** za vizualna izvješća
2. **Izvadite statistiku**: ProsjekNDVI
po polju/parceli
3. **Vremenske serije**: Usporedite indekse kroz više sesija
4. **Generirajte izvješća**: Uključite karte, statistiku i vizualizacije***

## Arhiviranje i sigurnosno kopiranje

### Preporučena strategija sigurnosnog kopiranja

**Što pohraniti:*** ✅ **Izvorne RAW/JPG slike ili LATTICE sirovi snimci** - Arhivirati na zasebnom disku/u oblaku; raw je izvor cijelog procesa i sve ostalo može se regenerirati iz njega
* ✅ **Datoteke senzora svjetla `.daq` / `.csv`** - Potrebne za naknadno izračunavanje reflektancije
* ✅ **Obradeni rezultati** - Sačuvajte kalibrirane slike i indekse
* ✅ **Mapa projekta** (`project.json` i pripadajuće mape) - Sadrži sve postavke za ponovnu obradu ako je potrebno
* ✅ **Zapisnik o greškama** - Dokumentira detalje obrade
* ✅ **Slike kalibracijskih ciljeva** - Za provjeru i ponovnu obradu**Preporuke za pohranu:*** **Trenutna sigurnosna kopija**: Vanjski tvrdi disk
* **Dugoročni arhiv**: Pohrana u oblaku (Google Drive, Dropbox itd.)
* **Kritični podaci**: Čuvajte 2-3 kopije na različitim lokacijama***

## Sljedeće izvođenje obrade

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
*   Prerađujte ih jedan po jedan

**Opcija 2:Chloros
CLI
(samo zaChloros
+)**

* Automatizirajte grupnu obradu
* Obradite više mapa pomoću skripti
* Pogledajte [Dokumentaciju zaCLI
](../CLI.md) i [Referencu zaCLI
](../reference/cli-reference.md)

**Opcija 3:Python
SDK
(samo zaChloros
+)**

* Programatska kontrola
* Integracija s analiznim cjevovodima
* Pogledajte [API
dokumentaciju](../api-python-sdk.md) i [SDK
referencu](../reference/sdk-reference.md)

***

## Rješavanje problema s post-obrađivanjem

### Ponovna obrada s različitim postavkama

Ako rezultati nisu zadovoljavajući:

1. Sačuvajte izvorne slike (nikada ih ne brišite)
2. Otvorite isti projekt uChloros
3. Podesite postavke na ploči Postavke projekta
4. Ponovno obradite — izlazne datoteke smještaju se u iste mape proizvoda, stoga se datoteke s istim nazivom iz prethodne izvedbe zamjenjuju

### Obrada podskupa slika

Za ponovnu obradu samo određenih slika:

1. Stvorite novi projekt
2. Uvezite samo slike kojima je potrebna ponovna obrada
3. Koristite istu predložak postavki
4. Obradite manji skup podataka

### Potražite pomoć

Ako naiđete na probleme:

* 📧 **E-pošta**: info@mapir.camera (priložite zapisnik za otklanjanje pogrešaka)
* 🌐 **Podrška**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **ČPP**: [Često postavljana pitanja](../faq.md)
* 📖 **Dokumentacija**: [Vodič zaChloros
](../)

***

## Sažetak: Cjelokupni tijek rada

Sada ste dovršili cjelokupni tijek obradeChloros
:

1. ✅ **Kreiran projekt** - Pogledajte [Projekti](../projects.md)
2. ✅ **Dodali datoteke** - Vidi [Dodavanje datoteka](adding-files-to-a-project.md)
3. ✅ **Podesili postavke** - Vidi [Podešavanje postavki projekta](adjusting-project-settings.md)
4. ✅ **Označene ciljne slike** - Vidi [Odabir ciljnih slika](choosing-target-images.md)
5. ✅ **Pokrenuta obrada** - Vidi [Pokretanje obrade](starting-the-processing.md)
6. ✅ **Praćen napredak** - Pogledajte [Praćenje obrade](monitoring-the-processing.md)
7. ✅ **Pregledani rezultati** - Ova stranica**Vaše kalibrirane multispektralne slike korektirane po odbijanju spremne su za analizu!**

***

## Dodatni resursi

### Napredne značajke

* [**Preglednik slika**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktivna vizualizacija i analiza
* [**Sandbox za indekse/LUT-ove**](../image-viewer-gui/index-lut-sandbox.md) - Testiranje prilagođenih indeksa
* [**Jednadžbe za višespektralne indekse**](../project-settings/multispectral-index-formulas.md) - Potpuna referenca za indekse

### Automatizacija i integracija

* [**Dokumentacija zaCLI**](../CLI.md) - Serijska obrada putem naredbenog retka
* [**Python
SDK
**](../api-python-sdk.md) - Programatska automatizacija
* [**Chloros
+ značajke**](../#chloros) - Napredne mogućnosti obrade

### Podrška i učenje

* [**ČPP**](../faq.md) - Odgovori na česta pitanja
* [**Ciljevi kalibracije**](../calibration-targets.md) - Razumijevanje kalibracije reflektancije
* [**Podržane kamere**](../supported-cameras.md) - Kompatibilni hardver
