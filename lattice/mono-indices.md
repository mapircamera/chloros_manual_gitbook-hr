# Mono kamere i indeksi vegetacije

## Jedna kamera = jedan pojas

**M3M**kamera je mono verzija Bayerove**M3C**: monokromatski IMX265 senzor iza jednog uskopojasnog interferencijskog filtera. Naziv modela određuje pojas — `M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685` (prikazano uChloros

u kao `LATT-M3M-L87-F685`). Senzor isporučuje **jednu monohromatsku vrpcu** bez Bayerovog mozaika: nema što demosaikovati, nema međukanalnog preslušavanja za razdvajanje i nema balansa bijele za postavljanje.

Posljedice koje vrijedi znati prije nego što planirate mono sustav:

* **Zračnost i reflektivnost su u potpunosti definirani po pojasu.**To su radiometrijske mape po pojasu, tako da jedna M3M kamera proizvodi kalibriranu radijancu tipa float32 (W/m²/sr/nm) i reflektancu tipa uint16 (`32768` = ρ 1.0) točno kao što to čini i M3C pojas. Mono kadrovi nose matricu**identiteta** odziva senzora — nije potrebna niti se primjenjuje 3×3 dekonvolucija.
* **Jedna mono kamera ne može proizvesti indeks vegetacije.**NDVI

,NDRE

i srodni indeksi zahtijevaju najmanje dvije trake. Da biste izračunali indekse pomoću mono hardvera, kombinirate nekoliko M3M kamera — pogledajte dolje.
* M3M kamere prenose **Mono12** (12-bitni, 2 bajta po pikselu na kabelu), što je važno za [planiranje propusnosti niza](arrays.md#bandwidth-the-rules-of-thumb).

## ŠtoChloros

preskače za mono — i kako vam to javlja

Faze bojnog procesa jednostavno se ne primjenjuju na senzor s jednom bojom.Chloros

**ih preskače uz poruku od jednog retka** umjesto da prijavi pogrešku, a svejedno ih normalno izvršava za bilo koju M3C (Bayer) kameru u istoj sesiji:

| Faza | Ponašanje za mono (M3M) | Ponašanje za M3C |
| --- | --- | --- |
| Demosaicing / debayer | Preskočeno — izlazna razina `debayered` je jednokanalna slika u sivim tonovima. | 3-kanalna demosaika. |
| Bilans bijele (`lattice white-balance`) | Preskočeno uz poruku na jednoj liniji. | Radi normalno. |
| Profil boje (`lattice color-profile`) | Preskočeno uz poruku u jednom retku. | Radi normalno. |
| Zasićenje/kontrast (`lattice color`) | Preskočeno uz poruku u jednom retku. | Radi normalno. |
| Razdvajanje spektralnog križnog signala | Identitet (bez 3×3 matrice). | Primijenjena je 3×3 matrica po kameri. |
| Zračenje / odraz | **Radi** — po pojasu, potpuno kalibrirano. | Radi po pojasu. |

GUI primjenjuje isto ograničenje: za mono kameru, panel postavki po kameri skriva retke namijenjene samo zaRGB

(bijela boja, gama, profil boja, zasićenje, kontrast, razdvajanje kanala), a histogram uživo zaključan je na jednu **MONO** trag. Diskriminator kroz cijelu hrpu je token `M3M` u nizu modela, koji se u GUI-ju /SDK

prikazuje kao `is_mono`.

## Indeksi zahtijevaju ≥ 2 pojasa: poravnanje → hrpa → indeksiranje

Radni tijek mono indeksa uvijek se sastoji od istih triju koraka:

1. **Poravnajte** — usmjerite nekoliko M3M kamera na različite valne duljine (npr. F650 &quot;Red

&quot; i F850 &quot;NIR

&quot;), povežite ih kao [niz više kamera](arrays.md), i dopustite daChloros

izračuna transformaciju za su-registraciju između kamera.
2. **Složenje** — poravnati okviri postaju jedna višebandna slika (svaka kamera doprinosi jednom nazvanom bandu).
3. **Indeks** — ocjenjuje formulu indeksa nad slojevima hrpe, po želji je prikazujući putem LUT-a.

U GUI-ju je cijeli ovaj lanac prikazano kao prikaz niza **Kombinirane kamere**: kompozit uživo je već poravnat, a Izračunavač indeksa niza (dolje) definira formulu kojom se prikazuje. Snimljeni izvoz može se deformirati u istu poravnatu poziciju pomoću opcije snimanja**Aligned**.

## Izračunavač indeksa

Izračunavač indeksa stvara izraz indeksa koji se koristi za prikaz uživo i izvoze indeksa po kameri. To je jedna zajednička površina, otvorena s dva mjesta u bočnoj traci kartice **Camera**:

* **Po kameri**— Prikaz uživo → zupčanik**Indeks** (RGN

/OCN

/NGB

samo za Bayer kamere; pojedinačna mono kamera nema kontrolu indeksa jer jedna traka ne može stvoriti indeks).
* **Po nizu**— postavke niza → Živi prikaz →**Index**zupčanik. Ovo je mono putanja: popis pojaseva obuhvaća**sve kamere članice**, pa mono par ovdje doprinosi svojim dvjema pojasevima.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Njegove kontrole, odozgo prema dolje:

* **Čipovi pojaseva** (&quot;Pojasevi — kliknite za dodavanje u izraz&quot;) — jedno dugme za svaki dostupni pojas, označeno nazivom boje + valnom duljinom u nm (duplikati naziva boja razlikuju se, npr. &quot;Boja 850&quot;). Klikom se umeće token pojasa na mjesto kursora. Pojasevi s kamera koje ne mogu proizvesti radijancu po pojasu (RGB

/FRGB) filtriraju se.
* **Gumbi za operatore i funkcije** — `+ - * / ( ) ^ ,` plus `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Područje za tekst izraza** — formula slobodnog tipa; mjesto držača prikazuje klasični oblik &quot;NDVI

&quot; `(NIR - Red) / (NIR + Red)`. Iznad njega se nalazi samo za čitanje pregled s tokeniziranim oznakama koji prikazuje čipove pojaseva, brojeve i zastavice za nepoznate tokene.
* **Banner valjanosti**— sivi &quot;Prazno — neće se primijeniti nijedan indeks&quot;; zeleni &quot;Valjana izraz&quot;; crveni s konkretnom greškom pri parsiranju (nepoznata traka, dvosmislena traka izložena više kamera, nedostajući zagrada, …); ili jantarni kada je izraz valjan, ali**konstantan** (npr. `X/X`, ili nazivnik &quot;NDVI

&quot; upisan s `−` umjesto `+`) — konstanta mapira cijeli kadar na jednu boju.
* Pojavljuje se zasebno žućkasto upozorenje ako je primijenjeni izraz ispravan, ali je **uživo kadar jednoličan** (ravna ili zasićena scena) — kolaps histograma se automatski detektira.
* **Primijeni LUT**(zadano uključeno; isključeno = istezanje u sivim tonovima),**Razina**2/3/5/7-stopa (zadano 7-stopa) i ulazi**Min / Max**sa strane gradijentne trake. Min zadano iznosi**0,2**— time se rampa boja zumira u raspon relevantan za vegetaciju, dok vrijednosti ispod prolaze kao siva skala; postavite Min na −1 za cijeli raspon indeksa (gumb**Reset** vraća raspon od −1 do +1). Max zadano iznosi 1.
* **Živi histogram** raspodjele indeksa — stupci skalirani korijenom kvadrata, jantarne linije 2. i 98. percentila, bijela linija medijane i očitanja repova izvan raspona (&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;) koje postaju narančaste iznad 1 % kao znak za proširenje prozora Min/Max.
* **Primijeni**(Apply) primjenjuje izraz na uživo prijenos; prilagodbe LUT-a primjenjuju se uživo bez pritiska na Primijeni. Izrazi su namjerno**samo za sesiju** — ne spremaju se između sesija.



<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Putanja zaCLI



Isti lanac poravnanja → slaganja → indeksiranja, koji se može programirati od početka do kraja:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mapira simbole preseta na nazive pojaseva hrpe. Dva pravila štede neuspjelo pokretanje:

* **Simboli su osjetljivi na velika i mala slova** i moraju se točno podudarati s nazivima kanala preseta — preseti koriste mala slova (NDVI

su `red`,`nir`; provjerite `--list-presets`). `--channel red=Red_660` radi; `--channel RED=660` ne uspijeva s `channel_map missing entries` greškom.
* Strana trake mora naznačiti traku u poravnatoj hrpi (`lattice align-info --profile align.json` ih navodi). Izvanmrežni način rada također prihvaća indekse traka s bazom 0, npr. `--channel red=0 --channel nir=1`.

`lattice index` također radi potpuno izvan mreže na spremljenom poravnanom višebandnomTIFF

u:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Unaprijed postavljeni indeksi

`lattice index --preset` (i [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na kartici Slika, koji koristi isti mehanizam) dolazi s ovih **22 unaprijed postavljenih indeksa**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Pokrenite `chloros-cli lattice index --list-presets` za formulu i simbole kanala svake postavke i `--list-gradients` za dostupne gradijente boja. Prilagođene formule koriste `--formula EXPR` sa sintaksom istom kao kod Izračunavača indeksa. Imajte na umu da je ovaj popis presjeta specifičan za LATTICE indeksni motor — padajući izbornik za obradu u Postavkama projekta za uvožene slike je drugačiji popis (vidi [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md)).

Kompletan skup zastavica (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, kontrole za pomicanje poravnanja za `--live` i ostalo) dokumentirano je u [CLI

Referenciji § Index / Vegetation Maths](../reference/cli-reference.md#index--vegetation-maths);SDK

ekvivalenti se nalaze u [SDK

Referenci](../reference/sdk-reference.md).

## Snimanje indeksnih proizvoda iz mono niza

S priključenim nizom i primijenjenim indeksnim izrazom, `array-capture` (ili GUI **Snimanje svega**) sprema razine izvoza po kameri *i* prikaz indeksa — `--index`/`--no-index` uključuje ga naCLI

, a zadano snima svaku primjenjivu razinu. Doprinos mono kamere svakoj grupi snimanja je njezina jedna traka na razinama sirovog/dekonvoluiranog (skala sive)/radijance/reflektance, plus zajednički kompozit kombiniranog indeksa kada niz radi u kombiniranom načinu rada. Vidi [Multi-Camera Arrays § Capturing](arrays.md#capturing-monitoring-vs-analysis).
