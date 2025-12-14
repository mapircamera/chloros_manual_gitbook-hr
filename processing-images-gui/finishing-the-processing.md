# Završetak obrade

Nakon što Chloros dovrši obradu, vrijeme je da pregledate svoje rezultate, provjerite kvalitetu ispisa i pripremite obrađene slike za upotrebu u tijeku rada. Ova vas stranica vodi kroz završne korake i sljedeće radnje.

## Indikacija potpune obrade

Kada obrada uspješno završi, vidjet ćete nekoliko indikatora:

* ✅ **Traka napretka**: Dostiže 100% dovršenost
* ✅ **Debug Log**: Prikazuje poruku "Obrada dovršena".
* ✅ **Gumb Start**: ponovno postaje omogućen (spreman za sljedeću obradu)
* ✅ **Izlazne datoteke**: Sve obrađene slike spremaju se u podmapu modela fotoaparata***## Lociranje vaših obrađenih slika

### Otvaranje izlazne mape

1. Pritisnite **Glavni izbornik**<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">ikona (gore lijevo)
2. Odaberite**"Otvori mapu projekta"**3. Vaš preglednik datoteka otvara direktorij projekta
4. Pronađite svoj projekt po imenu***

## Pregled obrađenih slika

### Brzi pregled u File Exploreru

**Pretpregled ugrađen u sustav Windows:**1. Dođite do podmape modela fotoaparata
2. Odaberite slikovnu datoteku
3. Pregled se pojavljuje u oknu pregleda Windows Explorera
4. Koristite tipke sa strelicama za pregledavanje slika

### Pregledajte u vanjskim preglednicima slika**Preporučeni gledatelji:**

* **QGIS** - besplatni GIS softver (najbolji za georeferenciranu multispektralnu analizu)
* **IrfanView** - Brz, lagan preglednik slika (podržava TIFF)
* **Adobe Photoshop** - Profesionalno uređivanje (podrška za TIFF)
* **GIMP** - besplatna alternativa Photoshopu
* **Windows fotografije**- Osnovno gledanje (možda neće podržavati 16-bitni TIFF)

### Pregledajte u pregledniku slika Chloros

Koristite Chlorosov ugrađeni preglednik slika za naprednu vizualizaciju:

1. Pritisnite minijaturu slike u pregledniku datoteka
2. Slika se otvara u glavnom području pregleda
3. Kliknite**Preglednik slika**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">karticu na lijevoj bočnoj traci
4. Koristite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) za interaktivnu analizu

Pogledajte [Preglednik slika](../image-viewer-gui/opening-an-image-full-screen.md) za detaljne upute.***## Pregledavanje dnevnika otklanjanja pogrešaka

### Provjerite postoje li upozorenja ili pogreške

1. Otvorite **Debug Log**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">tab
2. Pomičite se kroz poruke
3. Potražite žuta upozorenja ili crvene pogreške
4. Pregledajte sve uočene probleme
5. Kontaktirajte MAPIR podršku za pomoć

### Spremanje dnevnika

Za vođenje evidencije obrade ili slanje MAPIR podršci:

1. Pritisnite gumb**"Kopiraj"**ili**"Preuzmi"**2. Spremi kao tekstualnu datoteku u mapu projekta
3. Priložiti projektnu dokumentaciju
4. Pošaljite MAPIR podršci ako naiđete na probleme***

## Uobičajeni problemi s izlazom i rješenja

### Problem: nedostaju izlazne datoteke

**Mogući uzroci:**

* Datoteke nisu zadovoljile kriterije obrade
* Slike samo za ciljanje (isključene iz izvoza)
* Ponestalo je prostora na disku tijekom izvoza
* Oštećenje datoteke tijekom obrade

**Rješenja:**1. Provjerite zapisnik otklanjanja pogrešaka za poruke o preskakanju/pogrešci
2. Provjerite ima li dovoljno prostora na disku
3. Broj datoteka: treba odgovarati (izvorni broj - ciljni broj) × (indeksi + 1)
4. Ponovno uvezite i ponovno obradite sve datoteke koje nedostaju

### Problem: tamni ili svijetli rubovi (vinjetiranje je i dalje vidljivo)**Mogući uzroci:**

* Ispravak vinjete onemogućen
* Kamera/objektiv nije u bazi podataka Chloros profila
* Ekstremno vinjetiranje izvan mogućnosti korekcije

**Rješenja:**1. Provjerite je li ispravak vinjete omogućen u postavkama projekta
2. Provjerite je li model fotoaparata ispravno otkriven
3. Obratite se MAPIR podršci ako se vinjetiranje nastavi

### Problem: netočne boje ili vrijednosti**Mogući uzroci:**

* Nisu otkriveni kalibracijski ciljevi
* Odabran je pogrešan ciljni model kalibracije
* Kalibracija refleksije onemogućena
* Ciljane slike loše kvalitete

**Rješenja:**1. Provjerite je li kalibracija refleksije omogućena
2. Provjerite poruke "Cilj je pronađen" u zapisniku otklanjanja pogrešaka
3. Pregledajte ciljnu kvalitetu slike
4. Ponovno obradite s označenim odgovarajućim ciljevima

### Problem: NDVI vrijednosti se čine pogrešnim**Očekivani rasponi NDVI:**

* **Voda, kamenje, tlo**: -0,1 do 0,2
* **Oskudna/nezdrava vegetacija**: 0,2 do 0,4
* **Umjerena vegetacija**: 0,4 do 0,6
* **Zdrava, gusta vegetacija**: 0,6 do 0,9**Ako su vrijednosti izvan ovih raspona:**1. Provjerite je li primijenjena kalibracija refleksije
2. Provjerite je li uključen zapisnik svjetlosnog senzora
3. Provjerite jesu li otkriveni ciljevi kalibracije
4. Provjerite je li otkriven ispravan model kamere
5. Pregledajte vrijeme i uvjete snimanja ciljne slike***

## Korištenje vaših obrađenih slika

### Za fotogrametriju / izradu ortomoza

**Preporučeni tijek rada:**1.**Uvezite kalibrirane slike refleksije** u softver za fotogrametriju:
* Pix4Dmapper
* Agisoft Metashape
* DroneDeploy
* WebODM
2. **Zadrži EXIF ​​metapodatke**: Osigurajte očuvanje GPS podataka za geooznačavanje
3.**Kalibrirani tijek rada**: koristite slike refleksije za znanstvenu točnost
4.**Obradite indeksne mozaike**: Stvorite NDVI ortomozaike iz pojedinačnih indeksnih slika
5.**Izvezi georeferencirani GeoTIFF**: Za korištenje u GIS aplikacijama

### Za GIS analizu**Preporučeni tijek rada:**1.**Učitaj u QGIS, ArcGIS ili slično**2.**Koristite 16-bitne TIFF**slike refleksije za višepojasnu analizu
3.**Koristite indeksne slike**(NDVI, NDRE) kao slojeve vegetacije spremne za upotrebu
4.**Raster kalkulator**: Kombinirajte trake za prilagođenu analizu
5.**Izvoz**: Izradite karte klasifikacije, otkrivanje promjena, karte zdravlja vegetacije

### Za izravnu analizu / izvješćivanje**Preporučeni tijek rada:**1.**Koristite indeksne slike s LUT bojama**za vizualna izvješća
2.**Izvadak statistike**: Srednji NDVI po polju/parceli
3.**Vremenska serija**: Usporedite indekse u više sesija
4.**Generirajte izvješća**: Uključite karte, statistiku i vizualizacije***## Arhiviranje i sigurnosno kopiranje

### Preporučena strategija sigurnosne kopije

**Što spasiti:**

* ✅ **Originalne RAW/JPG slike** - Arhivirajte na zasebnom disku/oblaku
* ✅ **Obrađeni rezultati** - Čuvajte kalibrirane slike i indekse
* ✅ **Projektna datoteka** - Sadrži sve postavke za ponovnu obradu ako je potrebno
* ✅ **Debug Log** - detalji obrade dokumenata
* ✅ **Slike cilja kalibracije**- Za provjeru i ponovnu obradu**Preporuke za pohranu:**

* **Trenutna sigurnosna kopija**: vanjski tvrdi disk
* **Dugoročna arhiva**: Pohrana u oblaku (Google Drive, Dropbox itd.)
* **Kritični podaci**: Čuvajte 2-3 kopije na različitim mjestima***## Sljedeće obrade

### Ponovno korištenje postavki projekta

Ako u budućnosti obrađujete slične skupove podataka:

1. **Spremi predložak projekta**(ako već nije učinjeno)
2.**Stvorite novi projekt**pomoću spremljenog predloška
3.**Uvezi nove slike**4.**Proces**s identičnim postavkama za dosljednost

### Skupna obrada više sesija

Za više sesija/skupova podataka:**Opcija 1: GUI - više projekata**

* Napravite zaseban projekt za svaku sesiju
* Koristite dosljedne postavke predloška
* Obradite jedan po jedan

**Opcija 2: Chloros CLI (samo Chloros+)**

* Automatizirajte skupnu obradu
* Obradite više mapa sa skriptama
* Pogledajte [CLI dokumentaciju](../CLI.md)

**Opcija 3: Python SDK (samo Chloros+)**

* Programsko upravljanje
* Integracija s analitičkim cjevovodima
* Pogledajte [API dokumentaciju](../api-python-sdk.md)

***

## Rješavanje problema naknadne obrade

### Ponovna obrada s različitim postavkama

Ako rezultati nisu zadovoljavajući:

1. Čuvajte originalne slike (nikad ne brišite)
2. Otvorite isti projekt u Chlorosu
3. Podesite postavke na ploči Postavke projekta
4. Ponovno obradi - izlazi će prebrisati prethodne rezultate

### Obrada podskupa slika

Za ponovnu obradu samo određenih slika:

1. Stvorite novi projekt
2. Uvezite samo slike koje je potrebno ponovno obraditi
3. Koristite isti predložak postavki
4. Obradite manji skup podataka

### Dobivanje pomoći

Ako naiđete na probleme:

* 📧 **E-pošta**: info@mapir.camera (sadrži zapisnik otklanjanja grešaka)
* 🌐 **Podrška**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Često postavljana pitanja](../faq.md)
* 📖 **Dokumentacija**: [Chloros Manual](../)***## Sažetak: Potpun tijek rada

Sada ste dovršili potpuni tijek obrade Chlorosa:

1. ✅ **Kreirani projekt**- Pogledajte [Projekti](../projects.md)
2. ✅**Dodane datoteke**- Pogledajte [Dodavanje datoteka](adding-files-to-a-project.md)
3. ✅**Prilagođene postavke**- Pogledajte [Prilagodba postavki projekta](adjusting-project-settings.md)
4. ✅**Označene mete**- Pogledajte [Odabir ciljanih slika](choosing-target-images.md)
5. ✅**Pokrenuta obrada**- Pogledajte [Pokretanje obrade](starting-the-processing.md)
6. ✅**Praćeni napredak**- Pogledajte [Nadgledanje obrade](monitoring-the-processing.md)
7. ✅**Pregledani rezultati**- Ova stranica**Vaše kalibrirane multispektralne slike s korekcijom refleksije spremne su za analizu!**
***

## Dodatni resursi

### Napredne značajke

* [**Image Viewer**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktivna vizualizacija i analiza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Testiranje prilagođenog indeksa
* [**Formule multispektralnog indeksa**](../project-settings/multispectral-index-formulas.md) - Potpuna referenca indeksa

### Automatizacija i integracija

* [**CLI dokumentacija**](../CLI.md) - Skupna obrada naredbenog retka
* [**Python SDK**](../api-python-sdk.md) - Programska automatizacija
* [**Kloros+ značajke**](../#kloros) - Napredne mogućnosti obrade

### Podrška i učenje

* [**FAQ**](../faq.md) - Odgovori na uobičajena pitanja
* [**Calibration Targets**](../calibration-targets.md) - Razumijevanje kalibracije refleksije
* [**Podržane kamere**](../supported-cameras.md) - Kompatibilni hardver
