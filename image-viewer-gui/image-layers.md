# Slojevi slike

**Padajući izbornik slojeva** u gornjem desnom kutu preglednika slika omogućuje prebacivanje između svake verzije slike koju gledate — od izvornog snimka, preko svakog obrađenog proizvoda, do izračunatih indeksnih slika — bez napuštanja preglednika.

## Što su slojevi slika?

&quot;Sloj&quot; uChloros

u je jedna **datoteka proizvoda**registrirana za jednu izvornu sliku. Uvoz vam daje izvornu datoteku; obrada dodaje sloj za svaki proizvod koji je generirala. Izvožene datoteke zadržavaju izvorni naziv datoteke —**mapu** koja identificira proizvod, a naziv sloja je oznakaChloros

a za tu mapu.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Popis slojeva

### Uvijek prisutni

| Sloj | Što je to |
| --- | --- |
| **JPG**(ili**PNG

**/**TIFF

**) | Izvorna datoteka koja je došla s fotografijom.Survey3

uvozi `.JPG` pored svakog `.RAW`; LATTICE snimke donosePNG

iliTIFF

pregled prikaza. Označeno prema onome što je zapravo uvezeno |
| **RAW (Original)** | Izvorni sirovi kadar, debayeren za prikaz bez primijenjenih korekcija. Dostupan od trenutka uvoza — ne zahtijeva obradu |

LATTICE snimka čija je osnovna datoteka **njena** sirova slika nema zaseban osnovni unos: `RAW (Original)` ga već pokriva.

### Proizvodi obradeSurvey3



| Sloj | Zapisano u | Postoji kada |
| --- | --- | --- |
| **RAW (Cilj)** | — | Okvir je identificiran kao da sadrži kalibracijski cilj |
| **RAW (Reflekcija)** | `Reflectance_Calibrated_Images/` | Kalibracija refleksije je uspješno provedena na ovom okviru |
| **Korekcija vinjete**| `Vignette_Corrected_Images/` | Okvir nije mogao biti kalibriran po odrazu**i** *korekcija vinjete* je bila uključena |
| **Odziv senzora**| `Sensor_Response_Images/` | Okvir nije mogao biti kalibriran po odrazu**i** *korekcija vinjete* je bila isključena |
| **Balanse bijele** | `White_Balanced_Images/` | Napisan je proizvod s balansom bijele |

{% hint style="info" %}
**Korekcija vinjete i odziv senzora su alternative, nikada oboje.** Za svaku izvedbu, za svaki model kamere, postoji točno jedan nekalaribirani rezervni proizvod, a prekidač *Korekcija vinjete* odabire koji. Pogledajte [Postavke projekta](../project-settings/project-settings.md).
{% endhint %}

### Razine mreže (LATTICE)

Mreža (LATTICE) se širi na njih tijekom jednog prolaza obrade. Koje postoje ovisi o prekidačima za izvoz po proizvodu u postavkama projekta i o onome što se primjenjuje na kameru.

| Sloj | Zapisuje se u | Primjenjivo na |
| --- | --- | --- |
| **RAW (dekibajerizirano)** | `Debayered_Images/` |RGB

i multispektralno |
| **RAW (pregled)** | `Preview_Images/` | Višespektralna (prikaz u lažnim bojama) |
| **Balančirano po bijeloj** | `Preview_Images/` |RGB

glavne kamere — pretpregledRGB

registriran je pod ovim imenom kako bi se podudarao sa slojemSurvey3

istog naziva |
| **RAW (Radijansa)** | `Radiance_Images/` | Samo višespektralno |
| **RAW (Reflekancija)** | `Reflectance_Calibrated_Images/` | Samo za multispektralne snimke i samo kada odgovarajući zapis dolaznog zračenja `.daq` ili cilj unutar kadra koji je prošao QA pokriva kadar |

RGB

glavne kamere nemaju radiometriju po pojasu, stoga se radijansa i reflektansa za njih preskaču kao **neprimjenjivo** — zapisnik to navodi umjesto da šutljivo doživjeti neuspjeh.

### Indeksne, LUT i sandbox slojevi

| Uzorak sloja | Primjer | Odakle potječe |
| --- | --- | --- |
| **RAW (`<INDEX>` indeks)** | `RAW (NDVI Index)` | Jedan po indeksu konfiguriran u postavkama projekta, izračunat tijekom obrade |
| **`<INDEX>` LUT** | `NDVI LUT` | Verzija indeksa s mapiranjem boja |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jedan po [Index/LUT Sandbox](index-lut-sandbox.md) izvozu |

Ako je isto ime indeksa konfigurirano više puta s različitim postavkama, drugom i kasnijim dodjeljuje se broj u nazivu (`RAW (NDVI2 Index)`) kako bi slojevi ostali razlikovni.

***

## Korištenje odabira slojeva

1. Otvorite sliku u punom zaslonu klikom na minijaturu u mreži
2. Kliknite na **padajući izbornik slojeva** u gornjem desnom kutu preglednika
3. Odaberite sloj — slika se odmah ažurira

Spuštajući izbornik prvo stavlja **JPG, RAW (Original), RAW (Ciljana), RAW (Refleksija)**, tim redoslijedom, a sve ostalo nakon njih u redoslijedu u kojem su proizvodi registrirani.

### Prioritet sloja pri navigaciji

Pritiskanjem tipki **←**/**→** prelazite na sljedeću sliku i pokušava vas zadržati na istom sloju:

1. **Prvo točno podudaranje** — ako sljedeća slika ima sloj s istim imenom, on se aktivira. Ovo vas zadržava na `RAW (NDVI Index)` dok prolazite kroz cijeli set
2. **Zatim podudaranje po vrsti** — indeksni sloj traži bilo koji indeksni sloj, LUT bilo koji LUT, refleksanciju za refleksanciju, ciljni za ciljni, izvorni za izvorni, osnovni za osnovni
3. **Zatim, samo za slojeve za izvoz** — naziv se zadržava čak i ako popis slojeva još nije dostigao tu stavku, jer datoteka već postoji na disku. To vam omogućuje pregledavanje proizvoda dok je izvršavanje još uvijek u tijeku.
4. **U suprotnom** — prvi dostupan sloj, što je obično osnovna slika

Spremljene datoteke sidecar `.daq` i `.csv` u projektu preskaču se navigacijom tipkama sa strelicama, tako da pri pregledavanju slika nikada nećete doći do snimke senzora svjetla.

Zumiranje i pomicanje također se prenose na slike, što usporedbu prije/poslije iste pozicije na terenu čini jednostavnom.

***

## Razumijevanje vrijednosti piksela po slojevima

[Ploča s vrijednostima pokazivača](opening-an-image-full-screen.md#cursor-values) prikazuje stvarnu vrijednost po kanalu ispod vašeg pokazivača, u jedinici u kojoj je sloj pohranjen. Njezine se stupci mijenjaju ovisno o sloju:

| Sloj | Prikazana jedinica | Napomene |
| --- | --- | --- |
| Osnova (JPG /PNG

/TIFF

pregled) | DN, 0–255 | Prikaz vrijednosti, korekcija gamme naRGB

. Samo za vizualni pregled |
| RAW (Originalno) | DN | Sirovi digitalni brojevi senzora. Os histograma prikazuje dubinu: 255 (8-bitno), 4095 (12-bitno) ili 65535 (16-bitno) |
| RAW (dekonvoluirano) | DN | Linearno, bez istezanja prikaza |
| RAW (Pregled) / Bijela uravnotežena | DN | Prikaz proizvoda — rastegnut ili s korekcijom gamme. Nije za mjerenje |
| RAW (Radijansa) | **W/m²/sr/nm** | Fizička radijansa Float32. Nema stupca DN |
| RAW (reflektancija) | DN **i %** | Postotak izračunat prema vlastitoj skali te datoteke — vidi dolje |
| Indeks / LUT / sandbox izvozi | Vrijednost indeksa ili komponenteRGB

| Jednokanalna datoteka s indeksom izvještava o vrijednosti indeksa; datoteka s LUT-om (mapiranjem boja) izvještava o komponentamaRed

/Green

/Blue

|

### Reflektancija: mjerilo je po datoteci

{% hint style="warning" %}
**&quot;Podjela sa 65.535&quot; točna je samo zaSurvey3

.** Reflektancija LATTICE pohranjuje se u drugačijoj skali, a miješanje ta dva podijelitelja najčešći je način dobivanja vrijednosti reflektancije koje su točno upola manje od onih koje bi trebale biti.
{% endhint %}

| Izvor | DN koji odgovara reflektanciji 1,0 | Identificirano pomoću |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP oznaka `Chloros:PixelScale=32768` utisnuta u svaki izvoz reflektancije LATTICE. Razmak od 2× znači da se ρ iznad 1.0 može predstaviti, umjesto da se odrezuje |
| **Survey3**| **65535** | Nema XMP oznake skaleChloros

— kalibracijaSurvey3

bilježi ρ × dtype-max i obrezuje na 1.0 |

Za GIS i skriptanje: pročitajte `Chloros:PixelScale` iz datoteke i podijelite s njim. Ako oznaka nedostaje, datoteka je u razmjeriSurvey3

(65535). Preglednik, indeksna/LUT sandbox i izvoz indeksa sve rješavaju razmjer na isti način, tako da je broj koji pročitate na pokazivaču broj koji je koristila indeksna matematika.

Pohrana specifična za format na vrhu te razmjere:

* **TIFF

(32-bita, postotak)** pohranjuje DN / 65535 kao pomični zarez
* **PNG

(8-bita)**i**JPG (8-bita)** pohranjuju DN × 255 / 65535
* **8-bitni izvozTIFF

a snimke iz 8-bitnog izvora** ograničen je na raspon 0–255 umjesto da se ponovno skalira i namjerno ne sadrži oznaku skale. Panel ispisuje DN samo za te datoteke, bez stupca s postotkom

### Rasponi vrijednosti indeksa

| Obitelj indeksa | Uobičajeni raspon | Očitanje |
| --- | --- | --- |
| Normalizirana razlika (NDVI

,GNDVI

,NDRE

, ENDVI…) | od −1 do +1 | Zdrava vegetacija obično 0,4–0,9; golo tlo blizu 0; voda negativna |
| Prilagođeno tlu (SAVI

,OSAVI

, MSAVI2…) | otprilike od -1 do +1,5 | Slično očitanje kao i kodNDVI

, s potisnutom pozadinom tla |
| Omjer (GRVI

,GCI

, MSR, CIRE…) | neograničeno | Omjeri rastu bez ograničenja jer se nazivnik približava nuli |
|EVI

/LAI

| 0 do ~1, 0 do ~3,5 | Oblaci i drugi zasićeni pikseli gurnu oba izvan raspona — prvo ih zamaskirajte |

Pogledajte [Multispektralne formule indeksa](../project-settings/multispectral-index-formulas.md) za točnu formulu iza svakog unaprijed postavljenog parametra.

***

## Uobičajeni tijekovi rada

### Usporedba prije i poslije

1. Odaberite **RAW (Original)** i zabilježite vinjetiranje i nekompenzirane vrijednosti
2. Prebacite se na **RAW (Reflectance)**

3. Usporedite — vinjetiranje je uklonjeno, vrijednosti su kalibrirane. Zumiranje i pomicanje ostaju zadržani, tako da gledate isti dio terena

### Pregled jednog indeksa kroz cijeli set

1. Otvorite prvu obrađenu sliku i odaberite sloj indeksa
2. Ponavljano pritisnite **→** — sloj indeksa prati vas od slike do slike
3. Pratite histogram u bočnoj traci dok to radite: kadar čija se distribucija mijenja vrijedi detaljnijeg pregleda.

### Provjerite ciljne oznake kalibracije

1. Odaberite **RAW (Cilj)** na ciljnom kadru
2. Potvrdite da je cilj jasno vidljiv i detektiran
3. Pređite na sljedeći ciljni kadar — sloj cilja prati

### Provjerite vrijednosti reflektancije radi točnosti

1. Odaberite **RAW (Reflectance)**

2. Pročitajte stupac**%** u odjeljku Vrijednosti pokazivača (Cursor Values) — on je već ispravno skaliran za tu datoteku
3. Provjerite sukladnost s poznatim materijalima na kadru: zdrava vegetacija ima visoku vrijednost u zelenoj (NIR

) i nisku u crvenoj; kalibracijska pločica trebala bi pokazati vrijednost blisku svojoj objavljenoj reflektanciji

***

## Rješavanje problema

### Sloj koji sam očekivao nije u padajućem izborniku

**Mogući uzroci**

* Slika nikada nije obrađena — postoje samo baza i `RAW (Original)`
* Prekidač za izvoz proizvoda nije označen u postavkama projekta
* Proizvod se ne primjenjuje na tu kameru (zračenje i reflektancija na masteru s više pojaseva; bilo koji indeks na mono kameri M3M s jednim pojasom)
* Kalibracija reflektancije nije imala s čime raditi — nema pokrivenosti zračenja prema dolje (`.daq`) niti cilja u kadru koji je prošao QA — stoga je kadar pao na ispravljenu vinjetu (Vignette Corrected) ili odgovor senzora (Sensor Response)

**Što učiniti**

1. Provjerite zapisnik izvršavanja:Chloros

navodi kada je zatraženi proizvod za izvoz bio nemoguć i zašto
2. Provjerite prekidače za izvoz po proizvodu u [Postavke projekta](../project-settings/project-settings.md)
3. Potvrdite da mapa proizvoda postoji u stablu izlaza projekta
4. Ponovno obradite s omogućenim proizvodom

### Popis slojeva izgleda zastarjelo

Chloros

ponovno skenira projektne mape proizvoda dok je izvršavanje u tijeku i popravlja nedostajuće registracije slojeva na temelju onoga što se zapravo nalazi na disku, tako da se sloj čije je izvozno izvršavanje završeno normalno pojavljuje samostalno unutar ispitivanja. Prebacivanje s prikaza i natrag prisiljava na novo rješavanje.

### Vrijednosti reflektancije izgledaju kao da su upola manje od očekivanih

Gotovo sigurno dijelite LATTICE datoteku s 65535. Koristite `Chloros:PixelScale` (32768) ili pročitajte stupac **%**, u kojem je to već primijenjeno.

### Indeksni sloj postoji, ali slika je prazna

Indeksu su potrebne trake koje vaš sloj nema — na primjer, indeks koji čita treći kanal primijenjen na datoteku s jednim ili dva kanala. Prebacite se na sloj s više traka (reflektancije ili debayered), ili odaberite indeks koji odgovara filtru kamere.

***

## Sljedeći koraci

* [**Otvaranje slike u punom zaslonu**](opening-an-image-full-screen.md) — očitavanje kursora, histogram i kontrola GSD-a
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktivna vizualizacija indeksa i izvoz
* [**Formule za višespektralne indekse**](../project-settings/multispectral-index-formulas.md) — referenca za indeks
* [**Završetak obrade**](../processing-images-gui/finishing-the-processing.md) — stablo izlaznih mapa na koje ove slojevi upućuju
