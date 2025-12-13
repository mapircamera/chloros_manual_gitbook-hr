---
description: This page lists some multispectral indices that Chloros uses.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---
# Formule multispektralnog indeksa

Sljedeće formule indeksa koriste kombinaciju srednjih raspona prijenosa filtra Survey3:

<table><thead><tr><th align="center">Boja filtra Survey3</th><th width="196.199951171875" align="center">Naziv filtra Survey3</th><th width="159.800048828125" align="center">Raspon prijenosa (FWHM)</th><th align="center">Prijenos mediji</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Plava</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - zelena</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - narančasta</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Crveno</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Kada koristite ove formule, ime može završiti na "\_1" ili "\_2", što odgovara NIR filtru, bilo NIR1 ili NIR2.

***

## EVI - Indeks poboljšane vegetacije

Ovaj je indeks izvorno razvijen za korištenje s MODIS podacima kao poboljšanje u odnosu na NDVI, optimizirajući signal vegetacije u područjima s visokim indeksom lisne površine (LAI). Najkorisniji je u regijama s visokim LAI, gdje NDVI može doći do zasićenja. Koristi područje plave refleksije za korekciju pozadinskih signala na tlu i smanjenje atmosferskih utjecaja, uključujući raspršivanje aerosola.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vrijednosti EVI trebale bi se kretati između 0 i 1 za vegetacijske piksele. Svijetle značajke, poput oblaka i bijelih zgrada, zajedno s tamnim značajkama, poput vode, mogu rezultirati abnormalnim vrijednostima piksela na EVI slici. Prije stvaranja EVI slike, morate maskirati oblake i svijetla obilježja sa slike refleksije i opcionalno postaviti prag vrijednosti piksela od 0 do 1.

_Referenca: Huete, A., et al. “Pregled radiometrijskih i biofizičkih performansi MODIS vegetacijskih indeksa.” Daljinska istraživanja okoliša 83 (2002): 195-213.

***

## FCI1 - Indeks šumskog pokrivača 1

Ovaj indeks razlikuje šumske krošnje od drugih tipova vegetacije pomoću multispektralnih slika refleksije koje uključuju crvenu rubnu traku.

$$
FCI1 = Red * RedEdge
$$

Pošumljena područja će imati niže vrijednosti FCI1 zbog manje refleksije stabala i prisutnosti sjena unutar krošnji.

_Referenca: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. "Robusni indeksi šumskog pokrova za multispektralne slike." Fotogrametrijsko inženjerstvo &amp; Daljinsko očitavanje 84.8 (2018): 505-512._

***

## FCI2 - Indeks šumskog pokrivača 2

Ovaj indeks razlikuje krošnje drveća od drugih tipova vegetacije korištenjem slika multispektralne refleksije koje ne uključuju crveni rubni pojas.

$$
FCI2 = Red * NIR
$$

Pošumljena područja će imati niže vrijednosti FCI2 zbog manje refleksije stabala i prisutnosti sjena unutar krošnji.

_Referenca: Becker, Sarah J., Craig S.T. Daughtry i Andrew L. Russ. "Robusni indeksi šumskog pokrova za multispektralne slike." Fotogrametrijsko inženjerstvo &amp; Daljinsko očitavanje 84.8 (2018): 505-512._

***

## GEMI - Globalni indeks praćenja okoliša

Ovaj nelinearni indeks vegetacije koristi se za globalno praćenje okoliša iz satelitskih slika i pokušava ispraviti atmosferske učinke. Sličan je NDVI-ju, ali je manje osjetljiv na atmosferske utjecaje. Pogoduje joj golo tlo pa se ne preporuča uporaba u područjima s rijetkom ili srednje gustom vegetacijom.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Gdje:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referenca: Pinty, B. i M. Verstraete. GEMI: nelinearni indeks za praćenje globalne vegetacije sa satelita. Vegetacija 101 (1992): 15-20._

***

## GARI - Indeks otpornosti na zelenu atmosferu

Ovaj indeks je osjetljiviji na širok raspon koncentracija klorofila i manje osjetljiv na atmosferske utjecaje od NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gama konstanta je težinska funkcija koja ovisi o uvjetima aerosola u atmosferi. ENVI koristi vrijednost od 1,7, što je vrijednost koju preporučuju Gitelson, Kaufman i Merzylak (1996., stranica 296).

_Referenca: Gitelson, A., Y. Kaufman i M. Merzylak. «Korištenje zelenog kanala u daljinskom otkrivanju globalne vegetacije iz EOS-MODIS-a». Daljinsko očitavanje okoliša 58 (1996): 289-298._

***

## GCI - Indeks zelenog klorofila

Ovaj se indeks koristi za procjenu sadržaja klorofila u lišću u širokom rasponu biljnih vrsta.

$$
GCI = {NIR \over Green} - 1
$$

Široke NIR i zelene valne duljine omogućuju bolje predviđanje sadržaja klorofila, a istovremeno omogućuju veću osjetljivost i veći omjer signala i šuma.

_Referenca: Gitelson, A., Y. Gritz i M. Merzlyak. "Odnosi između sadržaja klorofila u lišću i spektralne refleksije i algoritmi za nedestruktivnu procjenu klorofila u lišću viših biljaka." Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Indeks zelenog lišća

Ovaj je indeks izvorno dizajniran za korištenje s RGB digitalnom kamerom za mjerenje pokrivenosti pšenice, gdje su crveni, zeleni i plavi digitalni brojevi (DN) u rasponu od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI vrijednosti se kreću između -1 i +1. Negativne vrijednosti predstavljaju tlo i nežive elemente, dok pozitivne vrijednosti predstavljaju zeleno lišće i stabljike.

_Referenca: Louhaichi, M., M. Borman i D. Johnson. "Prostorno smještena platforma i snimanje iz zraka za dokumentiranje utjecaja ispaše na pšenicu." Geocarto International 16, br. 1 (2001): 65-70._

***

## GNDVI - Zeleni normalizirani vegetacijski indeks razlike

Ovaj indeks je sličan NDVI, osim što mjeri zeleni spektar od 540 do 570 nm umjesto crvenog spektra. Ovaj indeks je osjetljiviji na koncentraciju klorofila nego NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referenca: Gitelson, A. i M. Merzlyak. "Daljinsko otkrivanje koncentracije klorofila u lišću viših biljaka." Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Indeks vegetacije prilagođen zelenom tlu

Ovaj je indeks izvorno osmišljen korištenjem infracrvene fotografije u boji za predviđanje potreba kukuruza za dušikom. Slično je OSAVI-ju, ali zamjenjuje zelenu traku crvenom.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referenca: Sripada, R., et al. "Određivanje potreba kukuruza za dušikom tijekom sezone pomoću infracrvene aerofotografije u boji." Doktorska disertacija, Državno sveučilište Sjeverne Karoline, 2005._

***

## GRVI - Vegetacijski indeks zelenog omjera

Ovaj indeks je osjetljiv na stope fotosinteze u šumskim krošnjama, budući da su zelene i crvene refleksije pod velikim utjecajem promjena u pigmentima lišća.

$$
GRVI = {NIR \over Green }
$$

_Referenca: Sripada, R., et al. "Aerosnimanje u infracrvenoj boji za određivanje potreba kukuruza za dušikom na početku sezone." Agronomski vjesnik 98 (2006): 968-977._

***

## GSAVI - Vegetacijski indeks prilagođen zelenom tlu

Ovaj je indeks izvorno osmišljen korištenjem infracrvene fotografije u boji za predviđanje potreba kukuruza za dušikom. Sličan je SAVI-ju, ali zamjenjuje zelenu traku crvenom.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referenca: Sripada, R., et al. "Određivanje sezonskih potreba za dušikom za kukuruz korištenjem infracrvene fotografije u boji iz zraka." Doktorska disertacija, Državno sveučilište Sjeverne Karoline, 2005._

***

## LAI - Indeks lisne površine

Ovaj se indeks koristi za procjenu pokrivenosti lišćem i predviđanje rasta usjeva i prinosa. ENVI izračunava zeleni LAI pomoću sljedeće empirijske formule iz Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Gdje je EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Visoke LAI vrijednosti obično se kreću od približno 0 do 3,5. Međutim, kada scena sadrži oblake i druge svijetle elemente koji proizvode zasićene piksele, LAI vrijednosti mogu premašiti 3,5. U idealnom slučaju, trebali biste maskirati oblake i svijetle elemente u svojoj sceni prije stvaranja LAI slike.

_Referenca: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde i A. Thomsen. "Multispektralni podaci iz zraka za kvantificiranje indeksa lisne površine, koncentracije dušika i učinkovitosti fotosinteze u poljoprivredi." Daljinska istraživanja okoliša 81, br. 2-3 (2002): 179-193._

***

## LCI - Indeks klorofila u lišću

Ovaj se indeks koristi za procjenu sadržaja klorofila u višim biljkama, osjetljivim na varijacije refleksije uzrokovane apsorpcijom klorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referenca: Datt, B. "Daljinsko očitavanje sadržaja vode u lišću eukaliptusa." Journal of Plant Physiology 154, br. 1 (1999): 30-36._

***

## MNLI: modificirani nelinearni indeks

Ovaj indeks je poboljšanje nelinearnog indeksa (NLI) koji uključuje vegetacijski indeks prilagođen tlu (SAVI) kako bi se uzela u obzir pozadina tla. ENVI koristi vrijednost faktora prilagodbe dna nadstrešnice (_L_) od 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referenca: Yang, Z., P. Willis i R. Mueller. "Utjecaj AWIFS slike poboljšanog omjera pojasa na točnost klasifikacije usjeva." Zbornik radova simpozija Pecora 17 Remote Sensing (2008.), Denver, CO._

***

## MSAVI2 - Modificirani indeks vegetacije prilagođen tlu 2

Ovaj indeks je jednostavnija verzija indeksa MSAVI koji su predložili Qi et al (1994), koji poboljšava vegetacijski indeks prilagođen tlu (SAVI). Smanjuje šum tla i povećava dinamički raspon signala vegetacije. MSAVI2 se temelji na induktivnoj metodi koja ne koristi konstantnu vrijednost _L_ (kao u slučaju SAVI) za isticanje zdrave vegetacije.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referenca: Qi, J., A. Chehbouni, A. Huete, Y. Kerr i S. Sorooshian. "Modificirani indeks vegetacije prilagođen tlu." Daljinska detekcija okoliša 48 (1994): 119-126._

***

## NDRE: RedEdge normalizirana razlika

Ovaj indeks je sličan NDVI, ali uspoređuje kontrast između NIR i RedEdge umjesto Red, koji često ranije otkriva vegetacijski stres.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizirani vegetacijski indeks razlike

Ovaj indeks je mjera zelene i zdrave vegetacije. Kombinacija njegove normalizirane formulacije razlike i korištenja područja najveće apsorpcije i refleksije klorofila čine ga robusnim u širokom rasponu uvjeta. Međutim, može postati zasićen u uvjetima guste vegetacije kada LAI postane visok.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Vrijednost ovog indeksa kreće se između -1 i 1. Uobičajeni raspon za zelenu vegetaciju je 0,2 do 0,8.

_Referenca: Rouse, J., R. Haas, J. Schell i D. Deering. Praćenje vegetacijskih sustava u Velikim ravnicama pomoću ERTS-a. Treći ERTS simpozij, NASA (1973.): 309-317._

***

## NLI – Nelinearni indeks

Ovaj indeks pretpostavlja da je odnos između mnogih vegetacijskih indeksa i površinskih biofizičkih parametara nelinearan. Linearizira odnose s površinskim parametrima koji imaju tendenciju da budu nelinearni.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referenca: Goel, N. i W. Qin. "Utjecaji arhitekture krošnji na odnose između različitih vegetacijskih indeksa i LAI i Fpar: računalna simulacija." Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimizirani vegetacijski indeks prilagođen tlu

Ovaj indeks temelji se na vegetacijskom indeksu prilagođenom tlu (SAVI). Koristite standardnu ​​vrijednost od 0,16 za faktor prilagodbe dna nadstrešnice. Rondeaux (1996.) je utvrdio da ova vrijednost osigurava veću varijaciju tla od SAVI za niski vegetacijski pokrov, dok pokazuje veću osjetljivost na vegetacijski pokrov veći od 50%. Ovaj indeks je najbolje koristiti u područjima s relativno rijetkom vegetacijom, gdje je tlo vidljivo kroz krošnje.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

Referenca: Rondeaux, G., M. Steven i F. Baret. "Optimizacija vegetacijskih indeksa prilagođenih tlu." Daljinsko očitavanje okoliša 55 (1996): 95-107._

***

## RDVI: renormalizirani vegetacijski indeks razlike

Ovaj indeks koristi razliku između bliskih infracrvenih i crvenih valnih duljina, zajedno s NDVI, kako bi istaknuo zdravu vegetaciju. Neosjetljiv je na učinke geometrije promatranja tla i sunca.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referenca: Roujean, J. i F. Breon. "Procjena PAR-a apsorbiranog vegetacijom iz dvosmjernih mjerenja refleksije." Daljinsko očitavanje okoliša 51 (1995): 375-384._

***

## SAVI - Vegetacijski indeks prilagođen tlu

Ovaj je indeks sličan NDVI-ju, ali potiskuje učinke prizemnih piksela. Koristi faktor prilagodbe dna krošnje, _L_, koji je funkcija gustoće vegetacije i često zahtijeva prethodno poznavanje količine vegetacije. Huete (1988) predlaže optimalnu vrijednost od _L_=0,5 kako bi se uzele u obzir varijacije prvog reda u pozadini tla. Ovaj indeks je najbolje koristiti u područjima s relativno rijetkom vegetacijom, gdje je tlo vidljivo kroz krošnje.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referenca: Huete, A. «Indeks vegetacije prilagođen tlu (SAVI)». Daljinsko očitavanje okoliša 25 (1988): 295-309._

***

## TDVI - Indeks transformirane razlike vegetacije

Ovaj indeks je koristan za praćenje vegetacijskog pokrova u urbanim sredinama. Ne zasićuje kao NDVI i SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referenca: Bannari, A., H. Asalhi i P. Teillet. "Transformirani vegetacijski indeks (TDVI) za kartiranje vegetacijskog pokrova." U Zborniku simpozija o geoznanostima i daljinskom otkrivanju, IGARSS &#x27;02, IEEE International, svezak 5 (2002)._

***

## VARI - indeks otpornosti na vidljivu atmosferu

Ovaj se indeks temelji na ARVI i koristi se za procjenu udjela vegetacije u sceni s niskom osjetljivošću na atmosferske utjecaje.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referenca: Gitelson, A., et al. "Linije vegetacije i tla u vidljivom spektralnom prostoru: koncept i tehnika za daljinsku procjenu udjela vegetacije." International Journal of Remote Sensing 23 (2002): 2537-2562._

***

## WDRVI - Vegetacijski indeks širokog dinamičkog raspona

Ovaj je indeks sličan NDVI-ju, ali koristi težinski koeficijent (_a_) kako bi se smanjio nerazmjer između doprinosa bliskih infracrvenih i crvenih signala NDVI-ju. WDRVI je posebno učinkovit u scenama s umjerenom do visokom gustoćom vegetacije kada NDVI prelazi 0,6. NDVI ima tendenciju stabiliziranja kako se udio vegetacije i indeks lisne površine (LAI) povećavaju, dok je WDRVI osjetljiviji na širi raspon frakcija vegetacije i promjene u LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Težinski koeficijent (_a_) može se kretati između 0,1 i 0,2. Henebry, Viña i Gitelson (2004) preporučuju vrijednost od 0,2.

_Reference_

_Gitelson, A. "Vegetacijski indeks širokog dinamičkog raspona za daljinsku kvantifikaciju biofizičkih karakteristika vegetacije." Journal of Plant Physiology 161, br. 2 (2004): 165-173._

_Henebry, G., A. Viña i A. Gitelson. "Indeks vegetacije širokog dinamičkog raspona i njegova potencijalna korisnost za analizu nedostataka." Bilten analize nedostataka 12: 50-56._