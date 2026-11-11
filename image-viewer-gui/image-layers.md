# Slojevi slike

**Padajući izbornik slojeva** u gornjem desnom kutu preglednika slika omogućuje prebacivanje između svih verzija slike koju gledate — od izvornog snimka, preko svakog obrađenog proizvoda, do izračunatih indeksnih slika — bez napuštanja preglednika.

## Što su slojevi slike?

&quot;Sloj&quot; u programu Chloros

je jedna **datoteka proizvoda**registrirana za jednu izvornu sliku. Uvoz vam daje izvorne datoteke; obrada dodaje sloj za svaki proizvod koji je generirala. Izvožene datoteke zadržavaju izvorni naziv datoteke —**mapa** je ta koja identificira proizvod, a naziv sloja je oznaka Chloros

a za tu mapu.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Popis slojeva

### Uvijek prisutni

| Sloj | Što je to |
| --- | --- |
| **JPG**(ili**PNG

**/**TIFF

**) | Osnovna datoteka koja je došla s snimkom. Survey3

uvozi `.JPG` pored svakog `.RAW`; snimke LATTICE-a donose PNG

ili TIFF

pregled prikaza. Označeno prema onome što je zapravo uvezeno |
| **RAW (originalni)** | Izvorni si okvir, de-bayeren za prikaz bez primijenjenih korekcija. Dostupan od trenutka uvoza — ne zahtijeva obradu |

LATTICE snimka čija je osnovna datoteka **njena** sirova slika nema zaseban osnovni unos: `RAW (Original)` ga već pokriva.

### Survey3

-ovi proizvodi obrade

| Sloj | Zapisano u | Postoji kada |
| --- | --- | --- |
| **RAW (Cilj)** | — | Okvir je identificiran kao da sadrži kalibracijski cilj |
| **RAW (Reflekancija)** | `Reflectance_Calibrated_Images/` | Kalibracija refleksancije uspješno je provedena na ovom okviru |
| **Korekcija vinjete**| `Vignette_Corrected_Images/` | Okvir nije mogao biti kalibriran po odrazu**i** *korekcija vinjete* je bila uključena |
| **Odziv senzora**| `Sensor_Response_Images/` | Okvir nije mogao biti kalibriran po odrazu**i** *korekcija vinjete* je bila isključena |
| **Balanse bijele** | `White_Balanced_Images/` | Napisan je proizvod s balansom bijele |

{% hint style="info" %}
**Korekcija vinjete i odziv senzora su alternative, nikada oboje.** Za svaku izvedbu, za svaki model kamere, postoji točno jedan nekalaribran proizvod kao rezervna opcija, a prekidač *Korekcija vinjete* odabire koji će se koristiti. Pogledajte [Postavke projekta](../project-settings/project-settings.md).
{% endhint %}

### Razine mreže (LATTICE)

Mreža (LATTICE) se u jednom prolazu obrade širi na ove. Koje postoje ovisi o prekidačima za izvoz po proizvodu u postavkama projekta i o onome što se primjenjuje na kameru.

| Sloj | Zapisano u | Primjenjivo na |
| --- | --- | --- |
| **RAW (dekonvoluiran)** | `Debayered_Images/` | RGB

i multispektralni |
| **RAW (pregled)** | `Preview_Images/` | Višespektralni (prikaz u lažnim bojama) |
| **Balans bijele** | `Preview_Images/` | RGB

glavne kamere — pretpregled RGB

registriran je pod ovim nazivom kako bi se podudarao s slojem Survey3

istog naziva |
| **RAW (Zračenje)** | `Radiance_Images/` | Samo višespektralno |
| **RAW (reflektancija)** | `Reflectance_Calibrated_Images/` | Samo za multispektralne kamere, i to samo kada odgovarajući zapis `.daq` za snimanje prema dolje ili cilj unutar kadra koji je prošao QA pokriva kadar |

RGB

glavne kamere nemaju radiometriju po pojasu, stoga se radijansa i reflektansa za njih preskaču kao **neprimjenjivo** — zapisnik to navodi umjesto da šutljivo doživjeti neuspjeh.

{% hint style="info" %}
**Slojevi u sirovom i dekodiranom formatu podudaraju se s proizvodima (novo u verziji 1.3.0).**Datoteke LATTICE u sirovom i dekodiranom formatu spremaju se u nativnoj orijentaciji senzora i u XMP-u bilježe orijentaciju koja im pripada; ostali proizvodi spremaju se s tom orijentacijom već primijenjenom. Preglednik i mreža slika na temelju te bilješke prikazuju**RAW (Original)**i**RAW (Debayered)**, pa se kadar više ne preokreće kada prebacujete između sloja proizvoda i sirovog sloja. Ovo je samo za prikaz — datoteke se nikada ne prepisuju. Datoteke bez zapisa o orijentaciji (snimljene prije verzije 1.3.0 ili napisane drugim softverom) prikazuju se onako kako su spremljene. Vidi [Formati izlaznih slika](../output-image-formats.md#orientation).
{% endhint %}

Maske za ograničavanje (`*_clampmask.tif` u mapi `clamp_masks/`) nisu slojevi: Chloros

ih preskače pri uvozu. Pogledajte [Formati izlaznih slika](../output-image-formats.md#clamp-masks).

### Indeksni, LUT i sandbox slojevi

| Uzorak sloja | Primjer | Odakle potječe |
| --- | --- | --- |
| **RAW (`<INDEX>` indeks)** | `RAW (NDVI Index)` | Jedan po indeksu konfiguriranom u postavkama projekta, izračunat tijekom obrade |
| **`<INDEX>` LUT** | `NDVI LUT` | Verzija indeksa s mapiranjem boja |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jedan po [Index/LUT Sandbox](index-lut-sandbox.md) izvoza |

Ako je isto ime indeksa konfigurirano više puta s različitim postavkama, drugom i kasnijim dodjeljuje se broj u naziv (`RAW (NDVI2 Index)`) kako bi slojevi ostali razlikovni.

***

## Korištenje odabira slojeva

1. Otvorite sliku u cijelom zaslonu klikom na minijaturu u mreži
2. Kliknite **padajući izbornik slojeva** u gornjem desnom kutu preglednika
3. Odaberite sloj — slika se odmah ažurira

Spuštajući izbornik prvo stavlja **JPG, RAW (Original), RAW (Cilj), RAW (Reflekcija)**, tim redoslijedom, a sve ostalo nakon njih u redoslijedu u kojem su proizvodi registrirani.

### Prioritet sloja pri navigaciji

Pritiskanjem tipki **←**/**→** prelazite na sljedeću sliku i pokušava vas zadržati na istom sloju:

1. **Prvo točno podudaranje** — ako sljedeća slika ima sloj s istim imenom, on se aktivira. To vas zadržava na `RAW (NDVI Index)` dok prolazite kroz cijeli set
2. **Zatim podudaranje po vrsti** — indeksni sloj traži bilo koji indeksni sloj, LUT bilo koji LUT, reflektanciju za reflektanciju, cilj za cilj, original za original, osnovu za osnovu
3. **Zatim, samo za slojeve za izvoz** — naziv se zadržava čak i ako popis slojeva još nije dostigao tu točku jer datoteka već postoji na disku. To vam omogućuje pregledavanje proizvoda dok je izvršavanje još uvijek u tijeku.
4. **U suprotnom** — prvi dostupan sloj, što je obično osnovna slika

Spremljene datoteke sidecar `.daq` i `.csv` u projektu preskaču se navigacijom tipkama sa strelicama, tako da pri pregledavanju slika nikada nećete doći do snimke senzora svjetla.

Zumiranje i pomicanje također se prenose na sljedeće slike, što usporedbu prije/poslije istog položaja na terenu čini jednostavnom.

***

## Razumijevanje vrijednosti piksela po slojevima

[Ploča s vrijednostima pokazivača](opening-an-image-full-screen.md#cursor-values) prikazuje stvarnu vrijednost po kanalu ispod vašeg pokazivača, u jedinici u kojoj je sloj pohranjen. Njezine se stupce mijenjaju ovisno o sloju:

| Sloj | Prikazana jedinica | Napomene |
| --- | --- | --- |
| Baza (JPG / PNG

/ TIFF

pregled) | DN, 0–255 | Prikaz vrijednosti, gamma-korigirano na RGB

. Samo za vizualni pregled |
| RAW (originalno) | DN | Digitalni brojevi senzora (Raw). Os histograma pokazuje dubinu: 255 (8-bitno), 4095 (12-bitno) ili 65535 (16-bitno) |
| RAW (dekonvoluirano) | DN | Linearno, bez istezanja prikaza |
| RAW (pregled) / balansirana bijela | DN | Prikaz proizvoda — istegnuto ili s korekcijom gamme. Nije za mjerenje |
| RAW (Radijansa) | **W/m²/sr/nm** | Float32 fizička radijansa. Nema stupca DN |
| RAW (Reflekancija) | DN **i %** | Postotak izračunat prema vlastitoj skali datoteke — vidi dolje |
| Indeks / LUT / izvoz u sandbox | Vrijednost indeksa ili RGB

komponente | Jednokanalna datoteka s indeksom izvještava o vrijednosti indeksa; datoteka s LUT-om (mapiranjem boja) izvještava o Red

/Green

/Blue

komponentama |

### Reflektancija: mjerilo je po datoteci

{% hint style="warning" %}
**&quot;Podijeli sa 65.535&quot; točno je samo za Survey3

.** Reflektancija LATTICE-a pohranjuje se u drugačijem omjeru, a miješanje ta dva dijelitelja najčešći je način da se dobiju vrijednosti reflektancije koje su točno upola manje od onih koje bi trebale biti.
{% endhint %}

| Izvor | DN koji je jednak reflektanciji 1.0 | Identificirano pomoću |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP oznaka `Chloros:PixelScale=32768` utisnuta u svaki izvoz reflektancije LATTICE. Razmak od 2× znači da se ρ iznad 1.0 može predstaviti, a ne biti odrezan |
| **Survey3**| **65535**| Nema XMP oznake skale**Chloros

**— kalibracija**Survey3

** bilježi ρ × dtype-max i obrezuje na 1.0 |

Za GIS i skriptanje: pročitajte `Chloros:PixelScale` iz datoteke i podijelite s njim. Ako oznaka nedostaje, datoteka je u razmjeri Survey3

(65535). Preglednik, sandbox indeksa/LUT-a i izvoz indeksa svi rješavaju razmjer na isti način, tako da je broj koji pročitate na pokazivaču broj koji je koristila matematika indeksa.

Pohrana specifična za format na vrhu te skale:

* **TIFF

(32-bit, Postotak)** pohranjuje DN / 65535 kao decimalni broj
* **PNG

(8-bit)**i**JPG (8-bit)** pohranjuju DN × 255 / 65535
* **8-bitni DN proizvod 8-bitnog snimanja**, ili datoteka reflektancije spremljena prije verzije 1.3.0, ograničena je na raspon 0–255 i ne sadrži oznaku mjerila.**Panel ispisuje DN samo za te datoteke, bez stupca u postotku.**

### Rasponi vrijednosti indeksa

| Obitelj indeksa | Tipičan raspon | Očitanje |
| --- | --- | --- |
| Normalizirana razlika (NDVI

, GNDVI

, NDRE

, ENDVI…) | od −1 do +1 | Zdrava vegetacija obično 0,4–0,9; golo tlo blizu 0; voda negativna |
| Prilagođeni tlu (SAVI

, OSAVI

, MSAVI2…) | otprilike od −1 do +1,5 | Slično očitanje kao i kod NDVI

, s potisnutom pozadinom tla |
| Omjer (GRVI

, GCI

, MSR, CIRE…) | neograničeno iznad | Omjeri rastu bez ograničenja kako denominatorski pojas teži nuli |
| EVI

/ LAI

| 0 do ~1, 0 do ~3,5 | Oblaci i drugi zasićeni pikseli gurnu oba izvan raspona — prvo ih zamaskirajte |

Pogledajte [Formule za multispektralne indekse](../project-settings/multispectral-index-formulas.md) za točnu formulu iza svakog unaprijed postavljenog parametra.

***

## Uobičajeni tijekovi rada

### Usporedba prije i poslije

1. Odaberite **RAW (Original)** i zabilježite vinjetiranje i nekompenzirane vrijednosti
2. Prebacite se na **RAW (reflektancija)**

3. Usporedite — vinjetiranje je uklonjeno, vrijednosti su kalibrirane. Zoom i pomicanje slike ostaju zadržani, tako da gledate isto područje tla

### Pregled jednog indeksa kroz cijeli set

1. Otvorite prvu obrađenu sliku i odaberite sloj indeksa
2. Pritisnite **→** više puta — sloj indeksa prati vašu sliku od slike do slike
3. Promatrajte histogram u bočnoj traci dok to radite: kadar čija distribucija skoči vrijedi detaljnijeg pregleda

### Provjerite ciljne oznake kalibracije

1. Odaberite **RAW (Cilj)** na ciljanom kadru
2. Potvrdite da je cilj jasno vidljiv i detektiran
3. Pređite na sljedeći ciljani kadar — sloj cilja prati

### Provjerite vrijednosti reflektancije radi točnosti

1. Odaberite **RAW (Reflektancija)**

2. Pročitajte stupac**%** u odjeljku Vrijednosti pokazivača (Cursor Values) — on je već ispravno skaliran za tu datoteku
3. Provjerite logičnost u odnosu na poznate materijale na slici: zdrava vegetacija ima visoku vrijednost u NIR

-u i nisku u crvenoj boji; kalibracijski cilj trebao bi pokazivati vrijednost blisku njegovoj objavljenoj reflektanciji

***

## Rješavanje problema

### Sloj koji sam očekivao nije u padajućem izborniku

**Mogući uzroci**

* Slika nikada nije obrađena — postoje samo baza i `RAW (Original)`
* Prekidač za izvoz proizvoda nije označen u postavkama projekta
* Proizvod se ne odnosi na tu kameru (zračenje i reflektancija na RGB

masteru; bilo koji indeks na M3M mono kameri s jednom trakom)
* Kalibracija reflektancije nije imala s čime raditi — nema pokrivenosti dolaznog zračenja `.daq` i nema cilja u kadru koji je prošao QA — stoga je kadar prešaltan na ispravljanje vinjete ili odgovor senzora

**Što učiniti**

1. Provjerite zapisnik izvršavanja: Chloros

navodi kada je zatraženi izvozni proizvod bio nemoguć i zašto
2. Provjerite prekidače za izvoz za svaki proizvod u [Postavkama projekta](../project-settings/project-settings.md)
3. Potvrdite da mapa proizvoda postoji u stablu izlaza projekta
4. Ponovno obradite s omogućenim proizvodom

### Popis slojeva izgleda zastarjelo

Chloros

ponovno skenira projektne mape s proizvodima dok je izvršavanje u tijeku i popravlja nedostajuće registracije slojeva na temelju onoga što se zapravo nalazi na disku, tako da se sloj čije je izvozno izvršavanje završeno normalno pojavljuje samostalno unutar ispitivanja. Prebacivanje s prikaza i natrag prisiljava na novo rješavanje.

### Vrijednosti reflektancije izgledaju kao da su upola manje od očekivanih

Gotovo sigurno dijelite LATTICE datoteku s 65535. Koristite `Chloros:PixelScale` (32768) ili pročitajte stupac **%**, u kojem je to već primijenjeno.

### Indeksni sloj postoji, ali slika je prazna

Indeksu su potrebne trake koje vaš sloj nema — na primjer, indeks koji čita treći kanal primijenjen na datoteku s jednim ili dva kanala. Prebacite se na sloj s više traka (reflektancije ili debajerirani), ili odaberite indeks koji odgovara filtru kamere.

***

## Sljedeći koraci

* [**Otvaranje slike u punom zaslonu**](opening-an-image-full-screen.md) — prikaz očitanja kursora, histograma i kontrole GSD-a
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktivna vizualizacija indeksa i izvoz
* [**Multispektralne formule indeksa**](../project-settings/multispectral-index-formulas.md) — referenca za indeks
* [**Završetak obrade**](../processing-images-gui/finishing-the-processing.md) — stablo izlazne mape u koju ove slojevi upućuju
