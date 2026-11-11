---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektralne formule indeksa

Sljedeće formule indeksa koriste kombinaciju prosječnih transmisijskih raspona filtera *Survey3*:

<table><thead><tr><th align="center">Survey3 boja filtera</th><th width="196.199951171875" align="center">Naziv filtera Survey3</th><th width="159.800048828125" align="center">Opseg transmisije (FWHM)</th><th align="center">Prosječna transmisija</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td align="center">823nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table>

Kada se koriste ove formule, naziv može završavati na &quot;\_1&quot; ili &quot;\_2&quot;, što odgovara tome je li korišten NIR-filter NIR1 ili NIR2.

Za kamere LATTICE M3C (Bayer s trostrukim propusnim pojasom), isti indeksni motor koristi M3C filtarske pojaseve:

| M3C filtar | Pojas 1 (središte/FWHM) | Pojas 2 (središte/FWHM) | Pojas 3 (središte/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orangea 615 nm / 21 nm | Cyana 490 nm / 38 nm | NIRa 808 nm / 14 nm |
| FNGB | Bluea 475 nm / 30 nm | Greena 550 nm / 30 nm | NIRa 850 nm / 30 nm |

LATTICE M3M kamere su jednobandne (jedan uskopojasni filtar po kameri), stoga se višebandni indeksi ne izračunavaju za samostalnu M3M sliku. Za izračunavanje indeksa s M3M-om, kombinirajte dvije ili više kamera u poravnati višebandni sloj (stack) i koristite LATTICE indeksni mehanizam (`chloros-cli lattice index` ili GUI-jev Index Calculator u stvarnom vremenu).

***

## Gdje svako ime indeksa radi

Chloros ima **tri** površine za indekse, a njihove unaprijed postavljene liste nisu identične. Koristite ovaj odjeljak kako biste provjerili hoće li ime raditi tamo gdje ga planirate koristiti.

| Gdje se nalazite | Koja se lista primjenjuje | Broj |
| --- | --- | --- |
| Postavke projekta → Indeks → Dodaj indeks (GUI) | Površina 1 | 27 |
| Preglednik slika [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Površina 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Površina 2 | 22 |
| SDK `process_folder(indices=[...])` | Površina 2 | 22 |
| `chloros-cli lattice index --preset` | Površina 3 | 22 (drugi 22) |
| Živi kalkulator indeksa na kartici Kamere | Površina 3 | 22 (drugi 22) |

Surface 1 i 2 rade na **jednoj slici odjednom s jedne kamere**, koristeći simboličke utore `x`/`y`/`z`(/`a`) vezane za filtarske kanale te kamere. Površina 3 radi na**poravnatoj višebandnoj hrpi** — nekoliko LATTICE kamera su-registriranih u jedan kubus — i referira se na kanale malim slovima.

### 1. GUI Postavke projekta / padajući izbornik Image Viewer sandbox — 27 formula

Padajući izbornik ih navodi ovim redoslijedom (radi se o redoslijedu umetanja, a ne abecednom redoslijedu):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

U GUI-ju povlačite filtarske kanale svoje kamere na utore za pojaseve u formuli, tako da se bilo koja formula može koristiti s bilo kojom dodjelom pojaseva koju vaša kamera podržava. Prilagođene formule koje ste spremili dodaju se ispod ovog popisa.

**Pet formula koje su dostupne samo u GUI-ju** — one koje popis CLI/SDK `--indices` ne prihvaća — implementirane su kao:

| Preddefinirano postavljanje samo za GUI | Formula (kako je implementirana) | Utori |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (četiri utora) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Namjensko mapiranje za svaki od njih navedeno je u vlastitom odjeljku niže na ovoj stranici (na primjer, GARI očekuje x=Green, y=NIR, z=Blue, a=Red). GARI jedina je formula u Chloros koja koristi četvrtu utičnicu.

### 2. CLI / SDK `--indices` proširenje naziva — 22 unaprijed definirana naziva

Opcija `chloros-cli process --indices` (i parametar SDK `indices`) prihvaća sljedeće unaprijed definirane nazive:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Nepriznati nazivi indeksa tiho se preskaču.** Naziv izvan ovog popisa (uključujući pet formula koje su dostupne samo u GUI-ju `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`, i bilo koja prilagođena formula koju ste spremili u GUI-ju) zanemaruje se uz samo obavijest u dnevniku — izvršavanje se nastavlja bez tog indeksa, a samo izvršavanje i dalje prijavljuje uspjeh. Obavijest se ispisuje kao:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nazivi se uspoređuju bez obzira na velika i mala slova nakon uklanjanja praznih znakova, stoga su `ndvi`, `NDVI` i ` NDVI ` isti prednastavak. Preset se također preskače ako mu je potreban pojas koji filter na vašem fotoaparatu ne pruža.
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
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Kako preset ime postaje položaji pojaseva

Kada navedete samo ime kao što je `NDVI`, Chloros mora odlučiti koji kanal koje datoteke čita svaki simbol. Koristi ovu tablicu koja povezuje kod filtera s položajem niza za svaki kanal:

| Kod filtera | Kanal → indeks niza |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` prihvaća se kao sinonim za Orange, također 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Zadani filtar** preseta (stupac &quot;Default filter&quot; iznad) koristi se kada projekt sadrži slike s tim filtrom. Ako ih nema, Chloros skenira filtere koji su zapravo prisutni u projektu redoslijedom `RGN, OCN, NGB, RGB, RE, NIR` i odabire prvi koji može osigurati svaki kanal koji preset treba. Ako nijedan ne može, preset se zanemaruje za tu izvedbu. Zbog toga `NDVI` zahtijevan na skupu podataka koji sadrži samo OCN i dalje daje smislen rezultat — povezuje se s pozicijama Orange i NIR u OCN-u.

Nizovi modela LATTICE M3C nose filtar s prefiksom `F` (`LATT-M3C-L41-FRGN`), ali prefiks se uklanja kada se kod filtra pročita s slike, pa FRGN kamera rješava kroz redak `RGN` iznad i ne zahtijeva posebno rukovanje.

### 3. LATTICE indeksni motor (`lattice index --preset`, kalkulator indeksa uživo) — 22 preddefinirana postavke

Pokretač LATTICE radi na poravnanim višebandnim nizovima (nizovima uživo ili izvozenim višebandnim TIFF datotekama) i koristi imena kanala s malim slovima (`red`, `green`, `blue`, `red_edge`, `nir`). Njegov popis unaprijed postavljenih vrijednosti razlikuje se od prethodna dva:

| Preset | Formula | Kanali |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | crvena, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zelena, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | plava, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | crveni rub, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | plava, zelena, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | crvena, nir |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | crvena, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | crvena, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | plava, crvena, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | crvena, nir |
| CVI | `nir * red / green**2` | crvena, zelena, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | crvena, nir |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | crvena, nir |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | plava, crvena, nir |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | crvena, zelena, plava |
| NGRDI | `(green - red) / (green + red)` | crvena, zelena |
| VARI | `(green - red) / (green + red - blue)` | crvena, zelena, plava |
| TGI | `green - 0.39*red - 0.61*blue` | crvena, zelena, plava |
| EXG | `2*green - red - blue` | crvena, zelena, plava |
| CIRE | `(nir / red_edge) - 1` | red\_edge, nir |
| CIGREEN | `(nir / green) - 1` | green, nir |
| NDWI | `(green - nir) / (green + nir)` | zelena, nir |

Pokrenite `chloros-cli lattice index --list-presets` za ispis ove tablice iz vaše instalirane verzije, i `--list-gradients` za dostupne bojne gradijente. Simboli kanala osjetljivi su na velika i mala slova i moraju odgovarati malim nazivima postavki (npr. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI - Indeks klorofila vegetacije

Ovaj indeks osjetljiv je na sadržaj klorofila u lišću u širokom rasponu gustoća krošnje. Koristi se s pojasevima NIR, crvenim i zelenim.

$$
CVI = NIR * {Red \over Green^{2}}
$$

U popisu presjeta u GUI-ju i CLI/SDK, to je formula simbola `(z/y)*(x/y)` s x=Red, y=Green, z=NIR (pojasne kamere RGN); unaprijed postavljena vrijednost za LATTICE indeksni mehanizam je `nir * red / green**2`. Svaka površina izračunava isti indeks.

_Reference: Vincini, M., E. Frazzi, and P. D&#x27;Alessio. &quot;A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.&quot; Precision Agriculture 9 (2008): 303-319._

***

## ENDVI - Unapređeni normalizirani indeks vegetacije

Ovaj indeks koristi plavi kanal uz NIR i zeleni te je popularan kod kamera s NGB filtrom, gdje plavi pojas zamjenjuje crveni.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementacija je formula simbola `((x+y)-(2*z))/((x+y)+(2*z))` — dodijelite kanale NIR i Green svoje kamere utorima x/y, a Blue utoru z (za kameru s NGB filtrom: x=NIR, y=Green, z=Blue).

***

## EVI - Unaprijedjeni indeks vegetacije

Ovaj je indeks izvorno razvijen za upotrebu s podacima sita MODIS kao poboljšanje u odnosu na NDVI optimiziranjem signala vegetacije u područjima s visokim indeksom površine lišća (LAI). Najkorisniji je u regijama s visokim LAI gdje NDVI može biti zasićen. Koristi plavo područje reflektancije za korekciju pozadinskih signala tla i za smanjenje atmosferskih utjecaja, uključujući raspršivanje aerosola.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vrijednosti indeksa EVI trebale bi se kretati od 0 do 1 za piksele sa vegetacijom. Svijetle značajke poput oblaka i bijelih zgrada, zajedno sa tamnim značajkama poput vode, mogu rezultirati anomalijskim vrijednostima piksela na slici indeksa EVI. Prije izrade slike indeksa EVI, trebali biste maskirati oblake i svijetle značajke s refleksne slike i, po želji, primijeniti prag na vrijednosti piksela od 0 do 1.

_Reference: Huete, A., et al. &quot;Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indeks šumskog pokrova 1

Samo za GUI — nije dostupan kao unaprijed postavljena vrijednost za CLI/SDK `--indices`._

Ovaj indeks razlikuje šumsko raslinje od drugih vrsta vegetacije koristeći multispektralne slike odbijanja svjetlosti koje uključuju crvenu rubnu traku.

$$
FCI1 = Red * RedEdge
$$

Šumski predjeli imat će niže vrijednosti indeksa FCI1 zbog niže reflektancije drveća i prisutnosti sjena unutar krošnje.

_Reference: Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robustni indeksi šumskog pokrova za multispektralne slike.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indeks šumskog pokrova 2

_Samo za GUI — nije dostupan kao unaprijed postavljena postavka za CLI/SDK `--indices`._

Ovaj indeks razlikuje šumsko pokrivalo od drugih vrsta vegetacije koristeći multispektralne snimke odbijanja svjetlosti koje ne uključuju crvenu rubnu traku.

$$
FCI2 = Red * NIR
$$

Šumski predjeli imat će niže vrijednosti indeksa FCI2 zbog niže reflektivnosti drveća i prisutnosti sjena unutar krošnje.

_Reference: Becker, Sarah J., Craig S.T. Daughtry, and Andrew L. Russ. &quot;Robustni indeksi šumskog pokrova za multispektralne slike.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Globalni indeks za praćenje okoliša

_Samo za GUI — nije dostupan kao CLI/SDK `--indices` preset._

Ovaj nelinearni indeks vegetacije koristi se za globalno praćenje okoliša iz satelitskih snimaka i pokušava ispraviti atmosferske učinke. Sličan je indeksu NDVI, ali je manje osjetljiv na atmosferske utjecaje. Pod utjecajem je golog tla; stoga se ne preporučuje za upotrebu u područjima s rijetkom ili umjereno gustom vegetacijom.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Gdje:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Reference: Pinty, B., and M. Verstraete. GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites. Vegetation 101 (1992): 15-20._

***

## GARI - Indeks Green otporan na atmosferu

_Samo za GUI — nije dostupan kao unaprijed postavljena vrijednost `--indices` za CLI/SDK._

Ovaj je indeks osjetljiviji na širok raspon koncentracija klorofila i manje osjetljiv na atmosferske učinke od NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammaska konstanta je funkcija ponderiranja koja ovisi o uvjetima aerosola u atmosferi. ENVI koristi vrijednost 1,7, što je preporučena vrijednost prema Gitelsonu, Kaufmannu i Merzylaku (1996., stranica 296).

_Reference: Gitelson, A., Y. Kaufman, and M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green indeks klorofila

Ovaj se indeks koristi za procjenu sadržaja klorofila u lišću kod širokog spektra biljnih vrsta.

$$
GCI = {NIR \over Green} - 1
$$

Upotreba širokog raspona valnih duljina u zelenom i NIRom području pruža bolje predviđanje sadržaja klorofila, istovremeno omogućujući veću osjetljivost i viši omjer signala i šuma.

_Referenca: Gitelson, A., Y. Gritz, i M. Merzlyak. &quot;Odnos između sadržaja klorofila u listu i spektralne reflektancije te algoritmi za neuništavajuću procjenu klorofila u listovima viših biljaka.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Indeks Green lišća

Ovaj je indeks izvorno dizajniran za upotrebu s digitalnim RGB fotoaparatom za mjerenje pokrivenosti pšenice, gdje se digitalne vrijednosti (DV) crvene, zelene i plave boje kreću od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Vrijednosti indeksa zelene lisice (GLI) kreću se od -1 do +1. Negativne vrijednosti predstavljaju tlo i nežive značajke, dok pozitivne vrijednosti predstavljaju zelene listove i stabljike.

_Reference: Louhaichi, M., M. Borman i D. Johnson. &quot;Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.&quot; Geocarto International 16, br. 1 (2001): 65-70._

***

## GNDVI - Green indeks normalizirane razlike vegetacije

Ovaj je indeks sličan indeksu NDVI, s tim što mjeri zeleni spektar od 540 do 570 nm umjesto crvenog spektra. Ovaj je indeks osjetljiviji na koncentraciju klorofila od indeksa NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Reference: Gitelson, A., i M. Merzlyak. &quot;Daljinsko snimanje koncentracije klorofila u lišću viših biljaka.&quot; Napredak u svemirskim istraživanjima 22 (1998): 689-692._

***

## GOSAVI - Indeks vegetacije optimiziran za zelenu boju i prilagođen tlu

Ovaj je indeks izvorno osmišljen pomoću kolor-infracrvene fotografije za predviđanje potreba za dušikom kod kukuruza. Sličan je indeksu OSAVI, ali u crveni pojas umeće zeleni.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Reference: Sripada, R., i dr. &quot;Određivanje potreba za dušikom tijekom sezone za kukuruz pomoću zračne fotografije u boji i infracrvenom spektru.&quot; Doktorska disertacija, Državno sveučilište Sjeverne Karoline, 2005._

***

## GRVI - Green Ratio Vegetation Index

Ovaj je indeks osjetljiv na brzinu fotosinteze u krošnjama šuma, jer su zeleni i crveni odraz snažno pod utjecajem promjena u pigmentima lišća.

$$
GRVI = {NIR \over Green }
$$

_Reference: Sripada, R., i dr. &quot;Zračna fotografija u boji i infracrvenom spektru za određivanje ranih potreba za dušikom u kukuruzu.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Indeks vegetacije prilagođen zelenom tlu

Ovaj je indeks izvorno osmišljen pomoću kolor-infracrvene fotografije za predviđanje potreba za dušikom kod kukuruza. Sličan je indeksu SAVI, ali u crvenoj pojasni zamjenjuje zelenu pojasu.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Reference: Sripada, R., et al. &quot;Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.&quot; Doktorska disertacija, Sveučilište Sjeverne Karoline, 2005._

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

Visoke vrijednosti indeksa listne površine (LAI) obično se kreću od otprilike 0 do 3,5. Međutim, kada scena sadrži oblake i druge jarke značajke koje stvaraju zasićene piksele, vrijednosti indeksa listne površine (LAI) mogu premašiti 3,5. Idealno bi bilo maskirati oblake i jarke značajke iz vaše scene prije izrade slike indeksa listne površine (LAI).

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, i A. Thomsen. &quot;Zračni multispektralni podaci za kvantificiranje indeksa površine lista, koncentracije dušika i fotosintetske učinkovitosti u poljoprivredi.&quot; Remote Sensing of Environment 81, br. 2-3 (2002): 179-193._

***

## LCI - Indeks klorofila lista

_Samo za GUI — nije dostupan kao unaprijed postavljena vrijednost CLI/SDK `--indices`._

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

_Referenca: Yang, Z., P. Willis i R. Mueller. &quot;Utjecaj poboljšane AWIFS slike omjerom pojaseva na točnost klasifikacije usjeva.&quot; Zbornik radova sa simpozija Pecora 17 o daljinskom istraživanju (2008.), Denver, CO._

***

## MSAVI2 - Modificirani indeks vegetacije prilagođen tlu 2

Ovaj indeks je jednostavnija verzija indeksa MSAVI koju su predložili Qi i sur. (1994.), koji poboljšava Indeks vegetacije prilagođen tlu (SAVI). Smanjuje šum u podacima o tlu i povećava dinamički raspon signala vegetacije. MSAVI2 se temelji na induktivnoj metodi koja ne koristi konstantnu vrijednost _L_ (kao kod SAVI) za isticanje zdrave vegetacije.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referenca: Qi, J., A. Chehbouni, A. Huete, Y. Kerr i S. Sorooshian. &quot;Modificirani indeks vegetacije prilagođen tlu.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Modificirani jednostavni omjer

Ovaj indeks je modifikacija jednostavnog omjera NIR/Red, osmišljenog za linearizaciju njegove povezanosti s biofizičkim parametrima, i osjetljiviji je od NDVI pri većoj gustoći vegetacije.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referenca: Chen, J. &quot;Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.&quot; Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE - Normalizirana razlika RedEdge

Ovaj je indeks sličan indeksu NDVI, ali uspoređuje kontrast između NIR i RedEdge umjesto Red, što često ranije otkriva stres kod vegetacije.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indeks normalizirane razlike vegetacije

Ovaj indeks je mjera zdrave, zelene vegetacije. Kombinacija njegove formulacije normalizirane razlike i upotrebe područja najveće apsorpcije i reflektancije klorofila čini ga robusnim u širokom rasponu uvjeta. Međutim, zasititi se u uvjetima guste vegetacije kada LAI postane visok.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Vrijednost ovog indeksa kreće se od -1 do 1. Uobičajeni raspon za zelenu vegetaciju je od 0,2 do 0,8.

_Referenca: Rouse, J., R. Haas, J. Schell i D. Deering. Praćenje vegetacijskih sustava na Velikim ravnicama pomoću ERTS-a. Treći simpozij o ERTS-u, NASA (1973): 309-317._

***

## NLI - nelinearni indeks

Ovaj indeks pretpostavlja da je odnos između mnogih indeksa vegetacije i površinskih biofizičkih parametara nelinearan. On linearizira odnose s površinskim parametrima koji su obično nelinearni.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Reference: Goel, N., i W. Qin. &quot;Utjecaj arhitekture krošnje na odnose između različitih indeksa vegetacije i LAI-a i Fpar-a: računalna simulacija.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Optimizirani indeks vegetacije prilagođen tlu

Ovaj se indeks temelji na Indeksu vegetacije prilagođenom tlu (SAVI). Koristi standardnu vrijednost od 0,16 za faktor prilagodbe pozadine krošnje. Rondeaux (1996.) je utvrdio da ova vrijednost pruža veću varijaciju tla od SAVIa za nisku pokrivenost vegetacijom, dok istovremeno pokazuje povećanu osjetljivost na pokrivenost vegetacijom veću od 50 %. Ovaj se indeks najbolje koristi u područjima s relativno rijetkom vegetacijom gdje je tlo vidljivo kroz krošnju.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Reference: Rondeaux, G., M. Steven, and F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Renormalizirani indeks razlike vegetacije

Ovaj indeks koristi razliku između bliskih infracrvenih i crvenih valnih duljina, zajedno s indeksom NDVI, kako bi istaknuo zdravu vegetaciju. Nije osjetljiv na učinke geometrijskih uvjeta promatranja tla i sunca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Reference: Roujean, J., and F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indeks vegetacije prilagođen tlu

Ovaj je indeks sličan NDVI-u, ali suzbija učinke piksela tla. Koristi faktor prilagodbe pozadine krošnje, _L_, koji je funkcija gustoće vegetacije i često zahtijeva prethodno poznavanje količine vegetacije. Huete (1988.) predlaže optimalnu vrijednost _L_=0,5 za uzimanje u obzir varijacija pozadine tla prvog reda. Ovaj se indeks najbolje koristi u područjima s relativno rijetkom vegetacijom gdje je tlo vidljivo kroz krošnju.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referenca: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indeks vegetacije transformirane razlike

Ovaj je indeks koristan za praćenje pokrivenosti vegetacijom u urbanim okruženjima. Ne zasićuje se kao NDVI i SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Reference: Bannari, A., H. Asalhi i P. Teillet. &quot;Indeks vegetacije transformirane razlike (TDVI) za kartiranje pokrivenosti vegetacijom&quot; U Zborniku radova simpozija o geoznanosti i daljinskom istraživanju, IGARSS &#x27;02, IEEE International, svezak 5 (2002)._

***

## VARI - Indeks vidljive otpornosti na atmosferu

Ovaj se indeks temelji na indeksu ARVI i koristi se za procjenu udjela vegetacije u prizoru s niskom osjetljivošću na atmosferske učinke.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Reference: Gitelson, A., i dr. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Indeks vegetacije širokog dinamičkog raspona

Ovaj je indeks sličan indeksu NDVI, ali koristi koeficijent ponderiranja (_a_) kako bi smanjio razliku između doprinosa bliskog infracrvenog i crvenog signala indeksu NDVI. Indeks WDRVI posebno je učinkovit u scenama s umjerenom do visokom gustoćom vegetacije kada indeks NDVI prelazi 0,6. Indeks NDVI ima tendenciju stabilizirati se kako se povećavaju udio vegetacije i indeks površine lišća (LAI), dok je indeks WDRVI osjetljiviji na širi raspon udjela vegetacije i na promjene u indeksu površine lišća (LAI).

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Koeficijent ponderiranja (_a_) može varirati od 0,1 do 0,2. Vrijednost od 0,2 preporučuju Henebry, Viña i Gitelson (2004.).

_Reference_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, br. 2 (2004): 165-173._

_Henebry, G., A. Viña i A. Gitelson. &quot;Indeks vegetacije širokog dinamičkog raspona i njegova potencijalna primjena u analizi praznina.&quot; Gap Analysis Bulletin 12: 50-56._
