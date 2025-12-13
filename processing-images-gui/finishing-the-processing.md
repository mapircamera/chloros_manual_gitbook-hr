# Završetak obrade

Nakon što Chloros dovrši obradu, vrijeme je za pregled rezultata, provjeru kvalitete ispisa i pripremu obrađenih slika za korištenje u tijeku rada. Ova vas stranica vodi kroz završne korake i sljedeće radnje.

##Obrada završene indikacije

Kada se obrada uspješno završi, vidjet ćete nekoliko indikatora:

* ✅ **Traka napretka**: Dostignite 100% dovršenost
* ✅ **Debug Log** – Prikazuje poruku "Obrada dovršena".
* ✅ **Gumb Početna** - Ponovno omogućen (spreman za sljedeće izvođenje)
* ✅ **Izlazne datoteke**: Sve obrađene slike spremaju se u podmapu modela fotoaparata

***

## Lokacija obrađenih slika

### Otvaranje izlazne mape

1. Kliknite ikonu **Glavni izbornik** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (gore lijevo).
2. Odaberite **«Otvori mapu projekta»**.
3. File explorer će se otvoriti u direktoriju projekta.
4. Pronađite svoj projekt po imenu.

***

## Pregled obrađenih slika

### Brzi pregled u pregledniku datoteka

**Integrirani pregled u sustavu Windows:**

1. Idite do podmape modela fotoaparata.
2. Odaberite slikovnu datoteku.
3. Pregled će se pojaviti u oknu pregleda Windows Explorera.
4. Koristite tipke sa strelicama za navigaciju kroz slike.

### Pregled u vanjskim preglednicima slika

**Preporučeni gledatelji:**

* **QGIS**: besplatni GIS softver (idealan za georeferenciranu multispektralnu analizu).
* **IrfanView** - Brz i lagan preglednik slika (kompatibilan s TIFF-om).
* **Adobe Photoshop**: profesionalno uređivanje (kompatibilno s TIFF-om).
* **GIMP**: besplatna alternativa Photoshopu.
* **Windows fotografije**: Osnovni prikaz (možda neće podržavati 16-bitni TIFF).

### Pregled u Chloros pregledniku slika

Koristite ugrađeni preglednik slika u Chlorosu za naprednu vizualizaciju:

1. Pritisnite minijaturu slike u pregledniku datoteka.
2. Slika se otvara u glavnom području pregleda.
3. Pritisnite karticu **Preglednik slika** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na lijevoj bočnoj traci.
4. Koristite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) za izvođenje interaktivne analize.

Pogledajte [Preglednik slika](../image-viewer-gui/opening-an-image-full-screen.md) za detaljne upute.

***

## Pregled dnevnika otklanjanja pogrešaka

### Provjerite postoje li upozorenja ili pogreške

1. Otvorite karticu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">.
2. Pomičite se kroz poruke.
3. Potražite žuta upozorenja ili crvene pogreške.
4. Pregledajte sve otkrivene probleme.
5. Kontaktirajte MAPIR Helpdesk za pomoć.

### Spremite zapis

Da biste vodili evidenciju obrade ili je poslali MAPIR službi za pomoć:

1. Pritisnite gumb **«Kopiraj»** ili **«Preuzmi»**.
2. Spremite datoteku kao tekstualnu datoteku u mapu projekta.
3. Uključiti projektnu dokumentaciju.
4. Pošaljite ga MAPIR podršci ako pronađete bilo kakve probleme.

***

## Uobičajeni problemi ispisa i rješenja

### Problem: Nedostaju izlazne datoteke

**Mogući uzroci:**

* Datoteke nisu zadovoljile kriterije obrade.
* Ciljajte samo slike (isključene iz izvoza).
* Nestalo je prostora na disku tijekom izvoza.
* Oštećenje datoteke tijekom obrade.

**Rješenja:**

1. Provjerite dnevnik otklanjanja pogrešaka za poruke o propustima/pogreškama.
2. Provjerite ima li dovoljno prostora na disku.
3. Prebrojite datoteke: moraju odgovarati (izvorni broj - odredišni broj) × (indeksi + 1).
4. Ponovno uvezite i obradite datoteke koje nedostaju.

### Problem: tamni ili svijetli rubovi (vinjetiranje je i dalje vidljivo)

**Mogući uzroci:**

* Ispravak vinjetiranja onemogućen.
* Kamera/objektiv nije uključen u bazu podataka Chloros profila.
* Ekstremno vinjetiranje koje premašuje kapacitet korekcije.

**Rješenja:**

1. Provjerite je li korekcija vinjetiranja omogućena u postavkama projekta.
2. Provjerite je li model kamere ispravno detektiran.
3. Kontaktirajte MAPIR podršku ako se vinjetiranje nastavi.

### Problem: Netočne boje ili vrijednosti

**Mogući uzroci:**

*Nisu otkriveni kalibracijski ciljevi.
* Odabran je pogrešan ciljni model kalibracije.
* Kalibracija refleksije je onemogućena.
*Ciljne slike loše kvalitete.

**Rješenja:**

1. Provjerite je li kalibracija refleksije omogućena.
2. Provjerite poruke "Cilj je pronađen" u zapisniku otklanjanja pogrešaka.
3. Provjerite kvalitetu slike objektiva.
4. Ponovno obradite s označenim odgovarajućim ciljevima.

### Problem: NDVI vrijednosti se pojavljuju netočne.

**Očekivani rasponi NDVI:**

* **Voda, kamenje, tlo**: od -0,1 do 0,2.
* **Loša/nezdrava vegetacija**: 0,2 do 0,4.
* **Umjerena vegetacija**: od 0,4 do 0,6.
* **Zdrava i gusta vegetacija**: od 0,6 do 0,9.

**Ako su vrijednosti izvan ovih raspona:**

1. Provjerite je li primijenjena kalibracija refleksije.
2. Provjerite je li uključen zapisnik svjetlosnog senzora.
3. Provjerite jesu li kalibracijski ciljevi otkriveni.
4. Provjerite je li otkriven ispravan model kamere.
5. Pregledajte vrijeme i uvjete snimanja ciljane slike.

***

## Korištenje obrađenih slika

### Za izradu fotogrametrije/ortomozaika

**Preporučeni tijek rada:**

1. **Uvezite kalibrirane slike refleksije** u softver za fotogrametriju:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Sačuvaj EXIF metapodatke** – Osigurajte da su GPS podaci sačuvani za geooznačavanje.
3. **Kalibrirani tijekovi rada** – Za znanstvenu točnost koristite refleksijsku sliku.
4. **Obradite indeksne mozaike**: Stvorite NDVI ortomozaike iz pojedinačnih indeksnih slika.
5. **Izvezi georeferencirani GeoTIFF**: za korištenje u GIS aplikacijama.

### Za GIS analizu

**Preporučeni tijek rada:**

1. **Učitaj u QGIS, ArcGIS ili slično**.
2. **Koristite 16-bitne TIFF** slike refleksije za višepojasnu analizu.
3. **Koristite indeksne slike** (NDVI, NDRE) kao slojeve vegetacije spremne za korištenje.
4. **Rasterski kalkulator**: Kombinirajte trake za prilagođenu analizu.
5. **Izvoz**: Napravite karte klasifikacije, detekciju promjena i zdravstvene karte vegetacije.

### Za izravnu analizu/izvješćivanje

**Preporučeni tijek rada:**

1. **Koristite indeksne slike s LUT bojama** za vizualno izvješćivanje.
2. **Izvadak statistike**: prosječni NDVI po polju/parceli.
3. **Vremenska serija**: usporedite indekse između nekoliko sesija.
4. **Generirajte izvješća**: uključite karte, statistiku i vizualizacije.

***

##Arhiviranje i backup

### Preporučena strategija sigurnosne kopije

**Što uštedjeti:**

* ✅ **Originalne RAW/JPG slike**: Arhivirajte na poseban disk/oblak.
* ✅ **Obrađeni rezultati** - čuvajte kalibrirane slike i indekse
* ✅ **Projektna datoteka** - sadrži sve postavke za ponovnu obradu ako je potrebno
* ✅ **Debug Log** – detalji obrade dokumenata
* ✅ **Kalibracijske slike**: za provjeru i ponovnu obradu

**Preporuke za pohranu:**

* **Trenutna sigurnosna kopija**: vanjski tvrdi disk
* **Dugoročna arhiva**: pohrana u oblaku (Google Drive, Dropbox, itd.)
* **Kritični podaci** – Spremite 2-3 kopije na različitim lokacijama

***

## Predstojeća obrada

### Ponovno korištenje konfiguracije projekta

Ako ćete u budućnosti obrađivati ​​slične skupove podataka:

1. **Spremite predložak projekta** (ako već niste)
2. **Stvorite novi projekt** pomoću spremljenog predloška
3. **Uvezi nove slike**
4. **Proces** s istim postavkama za održavanje dosljednosti

### Skupna obrada s više sesija

Za više sesija/skupova podataka:

**Opcija 1: GUI - više projekata**

* Napravite zaseban projekt za svaku sesiju
* Koristite dosljednu konfiguraciju predloška
* Obradite jedan po jedan

**Opcija 2: Chloros CLI (samo Chloros+)**

* Automatizirajte skupnu obradu.
* Obradite više mapa sa skriptama.
* Pogledajte [CLI dokumentaciju](../CLI.md)

**Opcija 3: Python SDK (samo Chloros+)**

* Programska kontrola
* Integracija s procesima analize
* Pogledajte [API dokumentaciju](../api-python-sdk.md)

***

## Rješavanje problema s naknadnom obradom

### Ponovna obrada s različitim postavkama

Ako rezultati nisu zadovoljavajući:

1. Zadržite originalne slike (nikada ih ne brišite)
2. Otvorite isti projekt u Chlorosu
3. Podesite postavke na ploči Postavke projekta
4. Ponovno obradite - rezultati će prebrisati prethodne

### Obrada podskupa slika

Za ponovnu obradu samo određenih slika:

1. Napravite novi projekt
2. Uvezite samo slike koje je potrebno ponovno obraditi
3. Koristite isti konfiguracijski predložak
4. Obradite manji skup podataka

### Potražite pomoć

Ako imate problema:

* 📧 **E-pošta**: info@mapir.camera (uključuje dnevnik otklanjanja pogrešaka).
* 🌐 **Tehnička pomoć**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact).
* 📚 **Često postavljana pitanja**: [Često postavljana pitanja](../faq.md)
* 📖 **Dokumentacija**: [Chloros Manual](../)

***

## Sažetak: Potpuni tijek rada

Sada ste dovršili cijeli tijek obrade Chlorosa:

1. ✅ **Projekt stvoren**: Pogledajte [Projekti](../projects.md)
2. ✅ **Dodane datoteke** - Pogledajte [Dodaj datoteke](adding-files-to-a-project.md)
3. ✅ **Prilagođene postavke** - Pogledajte [Prilagodba postavki projekta](adjusting-project-settings.md)
4. ✅ **Provjereni ciljevi**: Pogledajte [Odabir ciljanih slika](choosing-target-images.md)
5. ✅ **Pokrenuta obrada**: Pogledajte [Pokretanje obrade](starting-the-processing.md)
6. ✅ **Praćeni napredak**: Pogledajte [Praćenje obrade](monitoring-the-processing.md)
7. ✅ **Revidirani rezultati**: ova stranica

**Vaše kalibrirane multispektralne slike s korekcijom refleksije spremne su za analizu!**

***

## Dodatni resursi

### Napredne značajke

* [**Preglednik slika**](../image-viewer-gui/opening-an-image-full-screen.md): Interaktivna vizualizacija i analiza.
* [**Index Sandbox/LUT**](../image-viewer-gui/index-lut-sandbox.md): Testiranje prilagođenih indeksa.
* [**Formule multispektralnog indeksa**](../project-settings/multispectral-index-formulas.md): Potpuna referenca indeksa

### Automatizacija i integracija

* [**CLI dokumentacija**](../CLI.md): Skupna obrada iz naredbenog retka
* [**Python SDK**](../api-python-sdk.md) - Programska automatizacija
* [**Kloros+ značajke**](../#kloros) - Napredne mogućnosti obrade

### Podrška i učenje

* [**Često postavljana pitanja**](../faq.md) - Odgovori na uobičajena pitanja
* [**Calibration Targets**](../calibration-targets.md) - Razumijevanje kalibracije refleksije
* [**Podržane kamere**](../supported-cameras.md) - Podržani hardver