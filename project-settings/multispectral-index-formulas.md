---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektralne formule indeksa

Sljedeće formule indeksa koriste kombinaciju prosječnih transmisijskih raspona filtera s eSurvey3:

<table><thead><tr><th align="center">Survey3 Boja filtera</th><th width="196.199951171875" align="center">Survey3 Naziv filtera</th><th width="159.800048828125" align="center">Raspon transmisije (FWHM)</th><th align="center">Prosječna transmisija</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>

Kada se koriste ove formule, naziv može završavati na &quot;\_1&quot; ili &quot;\_2&quot;, što odgovara tome koji je filtar NIR, bilo NIR1 ili NIR2, korišten.

Za kamere LATTICE M3C (Bayer trostruka propusna traka), isti indeksni motor koristi M3C filtarske trake:

| M3C filtar | Traka 1 (središnja / FWHM) | Traka 2 (središnja / FWHM) | Traka 3 (središnja / FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M kamere su jednopojasne (jedan uskopojasni filtar po kameri), stoga se višepojasni indeksi ne izračunavaju za samostalnu M3M sliku. Za izračunavanje indeksa s M3M-om, kombinirajte dvije ili više kamera u poravnati višebandni sloj i upotrijebite LATTICE indeksni mehanizam (`chloros-cli lattice index` ili GUI-jev Index Calculator u stvarnom vremenu).

***

## Gdje svako ime indeksa radi

Chloros ima **tri** površine indeksa, a njihove unaprijed postavljene liste nisu identične. Koristite ovaj odjeljak kako biste provjerili hoće li ime raditi tamo gdje ga planirate koristiti.

| Gdje se nalazite | Koja se lista primjenjuje | Broj |
| --- | --- | --- |
| Postavke projekta → Indeks → Dodaj indeks (GUI) | Površina 1 | 27 |
| Preglednik slika [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Površina 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Površina 2 | 22 |
| SDK `process_folder(indices=[...])` | Površina 2 | 22 |
| `chloros-cli lattice index --preset` | Površina 3 | 22 (drugi 22) |
| Živi kalkulator indeksa na kartici Kamere | Površina 3 | 22 (drugi 22) |

Površine 1 i 2 rade na **jednoj slici odjednom s jedne kamere**, koristeći simboličke utore `x`/`y`/`z`(/`a`) vezane uz filtarske kanale te kamere. Površina 3 radi na**poravnatoj višebandnoj hrpi** — nekoliko LATTICE kamera su-registriranih u jedan kubus — i referira se na kanale malim slovima.

### 1. GUI Postavke projekta / padajući izbornik Image Viewer sandbox — 27 formula

Padajući izbornik ih navodi ovim redoslijedom (to je redoslijed umetanja, a ne abecedni):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

U GUI-ju povlačite filtarske kanale svoje kamere na utore za pojaseve formule, tako da se bilo koja formula može koristiti s bilo kojom dodjelom pojaseva koju vaša kamera podržava. Prilagođene formule koje ste spremili dodaju se ispod ovog popisa.

**Pet formula dostupnih samo u GUI-ju** — one koje popis CLI / SDK `--indices` ne prihvaća — implementirane su kao:

| Preddefinirano podešavanje dostupno samo u GUI-ju | Formula (kako je implementirana) | Utori |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (četiri utora) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Namjensko mapiranje za svaki od njih navedeno je u vlastitom odjeljku niže na ovoj stranici (na primjer, GARI očekuje x= Green, y= NIR, z= Blue, a= Red). GARI je jedina formula u Chloros koja koristi četvrtu poziciju.

### 2. CLI / SDK `--indices` proširenje imena — 22 unaprijed postavljena imena

Opcija `chloros-cli process --indices` (i parametar SDK `indices`) prihvaća sljedeća unaprijed postavljena imena:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Nepriznati nazivi indeksa tiho se preskaču.** Naziv izvan ovog popisa (uključujući pet formula samo za GUI `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`, i bilo koja prilagođena formula koju ste spremili u GUI-ju) zanemaruje se uz samo obavijest u dnevniku — izvršavanje se nastavlja bez tog indeksa, a samo izvršavanje i dalje prijavljuje uspjeh. Obavijest se ispisuje kao:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nazivi se uspoređuju neovisno o velikim i malim slovima nakon uklanjanja praznih znakova, tako da su `ndvi`, `NDVI` i ` NDVI ` ista predinstalacija. Preset se također preskače ako mu je potreban pojas koji filter na vašem fotoaparatu ne pruža.
{% endhint %}

Točne formule kako su implementirane (simboli `x`/`y`/`z` su utori za pojaseve; zadano mapiranje prikazano je za svaki preset):

| Preset | Formula (kako je implementirana) | Zadani filtar | Utori (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Kako ime unaprijed postavljenog postaje položaji bendova

Kada navedete samo ime kao što je `NDVI`, Chloros mora odlučiti koji kanal koje datoteke čita svaki simbol. Koristi ovu tablicu koja povezuje kod filtra s položajem niza svakog kanala:

| Kod filtra | Kanal → indeks niza |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` prihvaća se kao sinonim za Orange, također 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Zadani filtar** postavke (stubak &quot;Default filter&quot; iznad) koristi se kada projekt sadrži slike s tim filtrom. Ako ne, Chloros skenira filtre koji su stvarno prisutni u projektu redoslijedom `RGN, OCN, NGB, RGB, RE, NIR` i odabire prvi koji može isporučiti svaki kanal koji preset treba. Ako nijedan ne može, preset se zanemaruje za tu izvedbu. Zbog toga `NDVI` zatražen na skupu podataka koji sadrži samo OCN i dalje daje smislen rezultat — povezuje se s položajima Orange i NIR u OCN.

Nizovi modela LATTICE M3C nose filtar s prefiksom `F` (`LATT-M3C-L41-FRGN`), ali prefiks se izbacuje kada se kod filtra pročita s slike, pa se FRGN kamera rješava kroz redak `RGN` iznad i ne zahtijeva posebno rukovanje.

### 3. LATTICE indeksni mehanizam (`lattice index --preset`, live Index Calculator) — 22 unaprijed postavljena podešavanja

LATTICE motor radi na poravnanim višebandnim nizovima (nizovima uživo ili izvozenim višebandnim TIFF datotekama) i koristi mala slova za nazive kanala (`red`, `green`, `blue`, `red_edge`, `nir`). Njegov popis unaprijed postavljenih vrijednosti razlikuje se od prethodna dva:

| Unaprijed postavljena vrijednost | Formula | Kanali |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | crvena, nir |
| GNDVI | `(nir - green) / (nir + green)` | zelena, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | plava, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | crveni rub, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | plava, zelena, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | crvena, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | crvena, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | crvena, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | plava, crvena, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | crvena, nir |
| CVI | `(nir / green) - (red / green)` | crvena, zelena, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | crvena, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | crvena, nir |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | crvena, zelena, nir |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | crvena, zelena, plava |
| NGRDI | `(green - red) / (green + red)` | crvena, zelena |
| VARI | `(green - red) / (green + red - blue)` | crvena, zelena, plava |
| TGI | `green - 0.39*red - 0.61*blue` | crvena, zelena, plava |
| EXG | `2*green - red - blue` | crvena, zelena, plava |
| CIRE | `(nir / red_edge) - 1` | crveni rub, nir |
| CIGREEN | `(nir / green) - 1` | zelena, nir |
| NDWI | `(green - nir) / (green + nir)` | zelena, nir |

Pokrenite `chloros-cli lattice index --list-presets` za ispis ove tablice iz vaše instalirane verzije i `--list-gradients` za dostupne gradijente boja. Simboli kanala osjetljivi su na velika i mala slova i moraju odgovarati malim nazivima preseta (npr. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Kao što je implementirano u GUI-ju i popisu presjeta CLI / SDK, CVI je formula omjera omjera:

$$
CVI = {(z / y) \over (x / y)}
$$

s zadanim mapiranjem kanala RGB x= Red, y= Green, z= Blue. U GUI-ju možete povući bilo koji od kanala vaše kamere na utore x/y/z. Imajte na umu da unaprijed postavljena postavka `CVI` indeksnog mehanizma LATTICE koristi drugačiju formulu, `(NIR / Green) - (Red / Green)` — provjerite tablice iznad za površinu koju koristite.

***

## ENDVI - Unapređeni normalizirani indeks vegetacijske razlike

Ovaj indeks koristi plavi kanal uz NIR i zeleni, te je popularan kod kamera s filtrom NGB, gdje plavi pojas zamjenjuje crveni.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementacija je formula simbola `((x+y)-(2*z))/((x+y)+(2*z))` — dodijelite kanale NIR i Green svoje kamere utorima x/y, a kanal Blue utoru z (za kameru NGB: x= NIR, y= Green, z= Blue).

***

## EVI - Unapređeni indeks vegetacije

Ovaj je indeks izvorno razvijen za upotrebu s podacima iz satelita MODIS kao poboljšanje u odnosu na NDVI, optimizirajući signal vegetacije u područjima s visokim indeksom površine lišća (LAI). Najkorisniji je u regijama s visokim LAI gdje NDVI može biti zasićen. Koristi plavo područje reflektancije za korekciju pozadinskih signala tla i za smanjenje atmosferskih utjecaja, uključujući raspršivanje aerosola.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI vrijednosti bi trebale biti u rasponu od 0 do 1 za piksele vegetacije. Svetli objekti kao što su oblaci i bijele zgrade, zajedno s tamnim objektima kao što je voda, mogu rezultirati anomalijskim vrijednostima piksela na slici EVI. Prije izrade slike EVI, trebali biste maskirati oblake i svijetle objekte sa slike reflektancije i, po želji, binarizirati vrijednosti piksela iz raspona 0 do 1.

_Reference: Huete, A., i dr. &quot;Pregled radiometrijskih i biofizičkih performansi indeksa vegetacije MODIS.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indeks šumskog pokrova 1

_Samo za GUI — nije dostupan kao unaprijed postavljena vrijednost (CLI / SDK) za `--indices`._

Ovaj indeks razlikuje šumske krošnje od drugih vrsta vegetacije koristeći multispektralne snimke reflektancije koji uključuju crvenu rubnu traku.

$$
FCI1 = Red * RedEdge
$$

Šumski predjeli imat će niže vrijednosti indeksa FCI1 zbog niže reflektivnosti drveća i prisutnosti sjena unutar krošnje.

_Reference: Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indeks šumskog pokrova 2

_Samo za GUI — nije dostupan kao unaprijed postavljena postavka (CLI / SDK) `--indices`._

Ovaj indeks razlikuje šumsko pokrovište od drugih vrsta vegetacije koristeći multispektralne snimke reflektancije koje ne uključuju crvenu rubnu traku.

$$
FCI2 = Red * NIR
$$

Šumski predjeli imat će niže vrijednosti indeksa FCI2 zbog niže reflektancije drveća i prisutnosti sjena unutar krošnje.

_Reference: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. &quot;Robustni indeksi šumskog pokrova za multispektralne slike.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Globalni indeks praćenja okoliša

_Samo za GUI — nije dostupan kao unaprijed postavljena vrijednost (preset) za CLI / SDK `--indices`._

Ovaj nelinearni indeks vegetacije koristi se za globalno praćenje okoliša iz satelitskih snimki i pokušava ispraviti atmosferske učinke. Sličan je indeksu NDVI, ali je manje osjetljiv na atmosferske učinke. Pod utjecajem je golog tla; stoga se ne preporučuje za upotrebu u područjima s rijetkom ili umjereno gustom vegetacijom.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Gdje:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Reference: Pinty, B., i M. Verstraete. GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites. Vegetation 101 (1992): 15-20._

***

## GARI - Indeks otporan na atmosferu

_Samo za GUI — nije dostupan kao unaprijed postavljena postavka CLI / SDK `--indices`._

Ovaj je indeks osjetljiviji na širok raspon koncentracija klorofila i manje osjetljiv na atmosferske utjecaje od indeksa NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gamma konstanta je funkcija ponderiranja koja ovisi o uvjetima aerosola u atmosferi. ENVI koristi vrijednost od 1,7, što je preporučena vrijednost iz rada Gitelsona, Kaufmana i Merzylaka (1996., stranica 296).

_Reference: Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS- MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indeks klorofila

Ovaj se indeks koristi za procjenu sadržaja klorofila u lišću kod širokog spektra biljnih vrsta.

$$
GCI = {NIR \over Green} - 1
$$

Upotreba širokih valnih duljina crvenog (NIR) i zelenog spektra pruža bolje predviđanje sadržaja klorofila, istovremeno omogućujući veću osjetljivost i viši omjer signala i šuma.

_Reference: Gitelson, A., Y. Gritz, and M. Merzlyak. &quot;Odnos između sadržaja klorofila u listu i spektralne reflektancije te algoritmi za neuništavajuću procjenu klorofila u listovima viših biljaka.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Indeks listova Green

Ovaj je indeks izvorno osmišljen za upotrebu s digitalnim fotoaparatom RGB za mjerenje pokrivenosti pšenice, gdje se crveni, zeleni i plavi digitalni brojevi (DN) kreću od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Vrijednosti GLI kreću se od -1 do +1. Negativne vrijednosti predstavljaju tlo i nežive značajke, dok pozitivne vrijednosti predstavljaju zelene listove i stabljike.

_Reference: Louhaichi, M., M. Borman i D. Johnson. &quot;Prostorno locirana platforma i zračna fotografija za dokumentiranje utjecaja ispaše na pšenicu.&quot; Geocarto International 16, br. 1 (2001): 65-70._

***

## GNDVI - Green Normalizirani indeks razlike vegetacije

Ovaj je indeks sličan indeksu NDVI, s tim što mjeri zeleni spektar od 540 do 570 nm umjesto crvenog spektra. Ovaj je indeks osjetljiviji na koncentraciju klorofila od indeksa NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Reference: Gitelson, A., i M. Merzlyak. &quot;Daljinska detekcija koncentracije klorofila u lišću viših biljaka.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Indeks optimizirane vegetacije prilagođene tlu (Green)

Ovaj je indeks izvorno osmišljen pomoću kolor-infracrvene fotografije za predviđanje potreba za dušikom kod kukuruza. Sličan je indeksu OSAVI, ali zelenu traku zamjenjuje crvenom.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Reference: Sripada, R., i dr. &quot;Određivanje potreba za dušikom tijekom sezone za kukuruz pomoću zračne fotografije u boji i infracrvenom spektru.&quot; Doktorska disertacija, Državno sveučilište Sjeverne Karoline, 2005._

***

## GRVI - Indeks vegetacije na temelju omjera Green

Ovaj je indeks izvorno osmišljen za predviđanje potreba za dušikom kod kukuruza pomoću fotografije u boji i infracrvenom spektru. Sličan je indeksu , ali zeleni pojas zamjenjuje crvenim.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

Ovaj je indeks osjetljiv na brzine fotosinteze u krošnjama šuma jer su odražavanja zelene i crvene boje snažno pod utjecajem promjena u pigmentima lišća.

Ovaj je indeks osjetljiv na brzinu fotosinteze u krošnjama šuma jer su odražavanja zelene i crvene boje snažno pod utjecajem promjena u pigmentima lišća.

$$
GRVI = {NIR \over Green }
$$

_Reference: Sripada, R., i dr. &quot;Zračna fotografija u boji i infracrvenom spektru za određivanje ranih sezonskih potreba za dušikom u kukuruzu.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green indeks vegetacije prilagođen tlu

Ovaj je indeks izvorno osmišljen pomoću kolor-infracrvene fotografije za predviđanje potreba za dušikom kod kukuruza. Sličan je indeksu SAVI, ali zelenu traku koristi umjesto crvene.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Reference: Sripada, R., i dr. &quot;Određivanje potreba za dušikom tijekom sezone za kukuruz pomoću zračne fotografije u boji i infracrvenom spektru.&quot; Doktorska disertacija, Državno sveučilište Sjeverne Karoline, 2005._

***

## LAI - Indeks površine lista

Ovaj se indeks koristi za procjenu pokrivenosti lišća i za predviđanje rasta usjeva i prinosa. ENVI izračunava zeleni LAI koristeći sljedeću empirijsku formulu iz rada Boehga i sur. (2002.):

$$
LAI = 3.618 * EVI - 0.118
$$

Gdje je EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Visoke vrijednosti LAI obično se kreću od otprilike 0 do 3,5. Međutim, kada scena sadrži oblake i druge jarke značajke koje stvaraju zasićene piksele, vrijednosti LAI mogu premašiti 3,5. Idealno biste trebali maskirati oblake i jarke značajke iz scene prije izrade slike LAI.

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, i A. Thomsen. &quot;Zračni multispektralni podaci za kvantificiranje indeksa listne površine, koncentracije dušika i fotosintetske učinkovitosti u poljoprivredi.&quot; Remote Sensing of Environment 81, br. 2-3 (2002): 179-193._

***

## LCI - Indeks klorofila lišća

_Samo za GUI — nije dostupno kao preddefinirani CLI / SDK `--indices`._

Ovaj se indeks koristi za procjenu sadržaja klorofila u višim biljkama, osjetljiv na varijacije u odrazu uzrokovane apsorpcijom klorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Reference: Datt, B. &quot;Remote Sensing of Water Content in Eucalyptus Leaves.&quot; Journal of Plant Physiology 154, br. 1 (1999): 30-36._

***

## MNLI - Modificirani nelinearni indeks

Ovaj indeks je poboljšanje nelinearnog indeksa (NLI) koje uključuje indeks vegetacije prilagođen tlu (SAVI) kako bi se uzelo u obzir tlo kao pozadina. ENVI koristi vrijednost faktora prilagodbe pozadine krošnje (_L_) od 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Reference: Yang, Z., P. Willis, and R. Mueller. &quot;Utjecaj poboljšane AWIFS slike omjerom pojaseva na točnost klasifikacije usjeva.&quot; Zbornik radova Pecora 17 simpozija o daljinskom istraživanju (2008.), Denver, CO._

***

## MSAVI2 - Modificirani indeks vegetacije prilagođen tlu 2

Ovaj indeks je jednostavnija verzija indeksa MSAVIa koju su predložili Qi i sur. (1994.), a koji poboljšava Indeks vegetacije prilagođen tlu (SAVI). Smanjuje šum u podacima o tlu i povećava dinamički raspon signala vegetacije. MSAVI2 se temelji na induktivnoj metodi koja ne koristi konstantnu vrijednost _L_ (kao i kod SAVI) kako bi se istaknula zdrava vegetacija.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Reference: Qi, J., A. Chehbouni, A. Huete, Y. Kerr i S. Sorooshian. &quot;Modificirani indeks prilagođen tlu vegetacije.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Modificirani jednostavni omjer

Ovaj indeks je modifikacija jednostavnog omjera NIR / Red, osmišljenog kako bi se linearizirao njegov odnos s biofizičkim parametrima, te je osjetljiviji od indeksa NDVI pri većoj gustoći vegetacije.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Reference: Chen, J. &quot;Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.&quot; Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE - Normalizirana razlika RedEdge

Ovaj je indeks sličan indeksu NDVI, ali uspoređuje kontrast između NIR i RedEdge umjesto Red, što često ranije otkriva stres u vegetaciji.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Normalizirani indeks razlike vegetacije

Ovaj indeks je mjera zdrave, zelene vegetacije. Kombinacija njegove formulacije normalizirane razlike i upotrebe područja najveće apsorpcije i reflektancije klorofila čini ga robusnim u širokom rasponu uvjeta. Međutim, može doći do zasitjenja u uvjetima guste vegetacije kada LAI postane visok.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Vrijednost ovog indeksa kreće se od -1 do 1. Uobičajeni raspon za zelenu vegetaciju je od 0,2 do 0,8.

_Reference: Rouse, J., R. Haas, J. Schell i D. Deering. Praćenje vegetacijskih sustava na Velikim ravnicama pomoću ERTS-a. Treći simpozij o ERTS-u, NASA (1973): 309-317._

***

## NLI - nelinearni indeks

Ovaj indeks pretpostavlja da je odnos između mnogih indeksa vegetacije i površinskih biofizičkih parametara nelinearan. On linearizira odnose s površinskim parametrima koji su obično nelinearni.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Reference: Goel, N., i W. Qin. &quot;Utjecaj arhitekture krošnje na odnose između različitih indeksa vegetacije i LAI i Fpar: računalna simulacija.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Optimizirani indeks vegetacije prilagođen tlu

Ovaj se indeks temelji na indeksu vegetacije prilagođenom tlu (SAVI). Koristi standardnu vrijednost od 0,16 za faktor prilagodbe pozadine krošnje. Rondeaux (1996.) je utvrdio da ta vrijednost pruža veću varijaciju tla od SAVI kod niske pokrivenosti vegetacijom, dok istovremeno pokazuje povećanu osjetljivost na pokrivenost vegetacijom veću od 50 %. Ovaj se indeks najbolje koristi u područjima s relativno rijetkom vegetacijom gdje je tlo vidljivo kroz krošnju.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referenca: Rondeaux, G., M. Steven i F. Baret. &quot;Optimizacija indeksa vegetacije prilagođenih tlu.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Renormalizirani indeks razlike vegetacije

Ovaj indeks koristi razliku između bliskih infracrvenih i crvenih valnih duljina, zajedno s indeksom bez odraza (NDVI), kako bi istaknuo zdravu vegetaciju. Nije osjetljiv na učinke geometrijskih odnosa tla i Sunca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Reference: Roujean, J., and F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indeks vegetacije prilagođen tlu

Ovaj je indeks sličan indeksu NDVI, ali potiskuje učinke piksela tla. Koristi faktor prilagodbe pozadine krošnje, _L_, koji je funkcija gustoće vegetacije i često zahtijeva prethodno poznavanje količine vegetacije. Huete (1988.) predlaže optimalnu vrijednost _L_=0,5 za uzimanje u obzir varijacija pozadine tla prvog reda. Ovaj se indeks najbolje koristi u područjima s relativno rijetkom vegetacijom gdje je tlo vidljivo kroz krošnju.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Reference: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indeks vegetacije transformiranom razlikom

Ovaj je indeks koristan za praćenje pokrivenosti vegetacijom u urbanim okruženjima. Ne zasićuje se kao indeksi NDVI i SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Reference: Bannari, A., H. Asalhi i P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; u Zborniku radova simpozija o geoznanosti i daljinskom sondiranju, IGARSS &#x27;02, IEEE International, svezak 5 (2002)._

***

## VARI - Indeks vidljive otpornosti na atmosferu

Ovaj se indeks temelji na indeksu ARVI i koristi se za procjenu udjela vegetacije u prizoru s niskom osjetljivošću na atmosferske učinke.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Reference: Gitelson, A., i dr. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Indeks vegetacije širokog dinamičkog raspona

Ovaj je indeks sličan indeksu NDVI, ali koristi koeficijent ponderiranja (_a_) kako bi se smanjila razlika između doprinosa bliskog infracrvenog i crvenog signala indeksu NDVI. WDRVI je osobito učinkovit u scenama s umjerenom do visokom gustoćom vegetacije kada NDVI premaši 0,6. NDVI ima tendenciju stabilizacije kako se povećavaju udio vegetacije i indeks površine lišća (LAI), dok je WDRVI osjetljiviji na širi raspon udjela vegetacije i na promjene u LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Koeficijent ponderacije (a) može varirati od 0,1 do 0,2. Vrijednost od 0,2 preporučuju Henebry, Viña i Gitelson (2004.).

_Reference_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, br. 2 (2004): 165-173._

_Henebry, G., A. Viña i A. Gitelson. &quot;Indeks vegetacije širokog dinamičkog raspona i njegova potencijalna primjena u analizi praznina.&quot; Gap Analysis Bulletin 12: 50-56._
