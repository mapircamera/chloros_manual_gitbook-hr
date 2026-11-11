# Mono kamere i indeksi vegetacije

## Jedna kamera = jedan pojas

**M3M**kamera je mono sestra Bayerove**M3C**: monokromatski IMX265 senzor iza jednog uskopojasnog interferencijskog filtera. Naziv modela određuje pojas — `M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685` (prikazano u aplikaciji Chloros kao `LATT-M3M-L87-F685`). Senzor isporučuje **jednu monohromatsku traku** bez Bayerovog mozaika: nema što demosaikovati, nema međukanalnog smetanja za razdvajanje i nema balansa bijele za postavljanje.

Posljedice koje vrijedi znati prije nego što planirate mono sustav:

* **Zračnost i reflektivnost su u potpunosti definirani po pojasu.**To su radiometrijske mape po pojasu, tako da jedna M3M kamera proizvodi kalibriranu radijancu u float32 formatu (W/m²/sr/nm) i reflektancu u uint16 formatu (`32768` = ρ 1.0) točno kao što to čini i M3C pojas. Mono kadrovi sadrže matricu odziva senzora**identity** — nije potrebna niti se primjenjuje 3×3 dekonvolucija.
* **Jedna mono kamera ne može proizvesti indeks vegetacije.** NDVI, NDRE i srodni indeksi zahtijevaju najmanje dvije trake. Kako biste izračunali indekse pomoću mono hardvera, kombinirate nekoliko M3M kamera — pogledajte dolje.
* M3M kamere streamaju **Mono12** (12-bitno: 2 bajta po pikselu na vodi, ili 1,5 bajta po pikselu kada niz koristi [spakovani 12-bitni prijenos](arrays.md#packed-12-bit-transport)), što je važno za [planiranje propusnosti niza](arrays.md#bandwidth-how-many-cameras-fit).

## Što Chloros preskače za mono — i kako vas obavještava

Faze bojnog kanala jednostavno se ne primjenjuju na senzor s jednom trakom. Chloros **preskače ih uz poruku na jednoj liniji** umjesto da prijavi pogrešku, a svejedno ih normalno pokreće za bilo koju M3C (Bayer) kameru u istoj sesiji:

| Faza | Ponašanje za mono (M3M) | Ponašanje za M3C |
| --- | --- | --- |
| Demosaickovanje / debayer | Preskočeno — razina izvoza `debayered` je jednokanalna slika u sivim tonovima. | 3-kanalno demosaickovanje. |
| Bilans bijele (`lattice white-balance`) | Preskočeno uz poruku na jednoj liniji. | Radi normalno. |
| Profil boje (`lattice color-profile`) | Preskočeno uz poruku na jednoj liniji. | Radi normalno. |
| Zasićenje/kontrast (`lattice color`) | Preskočeno uz poruku u jednom retku. | Radi normalno. |
| Razdvajanje spektralne interferencije | Identitet (bez 3×3 matrice). | Primijenjena je 3×3 matrica po kameri. |
| Zračenje / odraz | **Radi** — po pojasu, potpuno kalibrirano. | Radi po pojasu. |

GUI primjenjuje isto ograničenje: za mono kameru, odjeljak s postavkama po kameri skriva retke namijenjene samo za **RGB (bijela ravnoteža, gama, profil boja, saturacija, kontrast, razdvajanje kanala), a histogram uživo zaključan je na jednu **MONO** trag. Diskriminator kroz cijelu hrpu je token `M3M` u nizu modela, koji se u GUI/SDK u prikazuje kao `is_mono`.

## Indeksi zahtijevaju ≥ 2 pojasa: poravnanje → slaganje → indeksiranje

Radni tijek indeksiranja za mono je uvijek isti i sastoji se od tri koraka:

1. **Poravnanje** — usmjerite nekoliko M3M kamera na različite valne duljine (npr. F650 "Red" i F850 "NIR"), povežite ih kao [niz više kamera](arrays.md), i dopustite da Chloros izračuna transformaciju za su-registraciju između kamera.
2. **Stack** — poravnati okviri postaju jedna višebandna slika (svaka kamera doprinosi jednom nazvanom bandu).
3. **Indeks** — procjenjuje se formula indeksa nad slojevima hrpe, a po želji se prikazuje putem LUT-a.

U GUI-ju je cijeli ovaj lanac prikazan kao način prikaza niza **Kombinirane kamere**: kompozit uživo je već poravnat, a Izračunavač indeksa niza (dolje) definira formulu kojom se prikazuje. Zabilježeni izvozi mogu se iskriviti na isto poravnanje pomoću opcije snimanja**Poravnano**.

## Izračunavač indeksa

Izračunavač indeksa stvara izraz za indeks koji se koristi u prikazu uživo i za izvoze indeksa po kameri. To je jedna zajednička površina, otvorena s dva mjesta u bočnoj traci kartice Kamere:

* **Po kameri**— Prikaz uživo → zupčanik**Indeks** (samo za Bayer kamere s RGN/OCN/NGB; mono kamera bez parova nema kontrolu indeksa jer jedan pojas ne može stvoriti indeks).
* **Po nizu**— postavke niza → Živi prikaz → zupčanik**Indeks**. Ovo je mono putanja: popis pojaseva obuhvaća**sve kamere članice**, pa mono par ovdje doprinosi svojim dvjema pojasevima.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Njegove kontrole, odozgo prema dolje:

* **Čipovi pojaseva** ("Pojasevi — kliknite za dodavanje u izraz") — jedno dugme za svaki dostupan pojas, označeno nazivom boje + valnom duljinom u nm (duplikati naziva boja razlikuju se, npr. "Boja 850"). Klikom se umeće token pojasa na mjesto kursora. Bande s kamera koje ne mogu proizvesti radijancu po bandi (RGB/FRGB) filtrirane su.
* **Gumbi za operatore i funkcije** — `+ - * / ( ) ^ ,` i `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Područje za unos izraza** — formula slobodnog tipa; mjesto držača prikazuje klasični oblik NDVI a `(NIR - Red) / (NIR + Red)`. Iznad njega se nalazi samo za čitanje pregled s tokeniziranim prikazom koji prikazuje čipove pojaseva, brojeve i zastavice za nepoznate tokene.
* **Banner valjanosti**— sivi "Prazno — neće se primijeniti nijedan indeks"; zeleni "Valjana izraz"; crveni s određenom greškom u parsiranju (nepoznata banda, dvosmislena banda koju prikazuju više kamera, nedostajući zagrada, …); ili žuto-smeđi kada je izraz valjan, ali**konstant** (npr. `X/X`, ili nazivnik NDVI upisan s `−` umjesto `+`) — konstanta mapira cijeli kadar na jednu boju.
* Pojavljuje se zasebno narančasto upozorenje ako je primijenjena izraz ispravan, ali je **trenutni kadar jednoličan** (ravna ili zasićena scena) — kolaps histograma se automatski otkriva.
* **Primijeni LUT**(zadano uključeno; isključeno = istezanje u sivim tonovima), ulazi**Razina**2/3/5/7-stopa (zadano 7-stopa) i**Min / Max**sa strane gradijentne trake. Min zadano iznosi**0,2**— time se rampa boja zumira u raspon relevantan za vegetaciju, dok vrijednosti ispod prolaze kao siva skala; postavite Min na −1 za cijeli raspon indeksa (gumb**Reset** vraća raspon od −1 do +1). Max zadano iznosi 1.
* **Živi histogram** raspodjele indeksa — stupci skalirani korijenom kvadrata, jantarne linije 2. i 98. percentila, bijela linija medijane i očitanja repova izvan raspona ("◀ N% < lo" / "hi < N% ▶") koje postaju narančaste iznad 1 % kao znak za proširenje prozora Min/Max.
* **Primijeni** (Apply) primjenjuje izraz na uživo stream; prilagodbe LUT-a primjenjuju se uživo bez prethodnog pritiska na gumb Primijeni. Kada je projekt otvoren, izraz i postavke LUT-a spremaju se s projektom i vraćaju se prilikom ponovnog otvaranja; bez projekta vrijede samo za sesiju.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Putanja CLI-a

Isti lanac poravnanje → hrpa → indeks, koji se može skriptirati od početka do kraja:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` koristi povezani niz koji sadrži njegove kamere ili ih sam otvara za jednu izvedbu. `index --live` uvijek sam otvara kamere (u [privremenom nizu](../reference/cli-reference.md#multi-camera-transient-sessions)) i nema `--serials`: otvara svaku kameru na mreži. Stoga oslobodite svaku kameru koju drži Chloros prije njega — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): otvaranje zadržane kamere drugi put može zaustaviti stream njezina vlasnika ili srušiti pozadinski sustav. Kamera koja ne isporuči nijedan kadar ne uspijeva u izvršavanju i ne zapisuje se ni hrpa ni indeks.

`--channel` mapira simbole preseta na nazive pojaseva hrpe. Dva pravila spašavaju od neuspješnog izvršavanja:

* **Simbolima se razlikuje velika i mala slova** i moraju se točno podudarati s nazivima kanala preseta — preseti koriste mala slova (NDVI-i su `red`,`nir`; provjerite `--list-presets`). `--channel red=Red_660` radi; `--channel RED=660` ne uspijeva s `channel_map missing entries` greškom.
* Strana pojasa mora naznačiti pojas u poravnatoj hrpi (`lattice align-info --profile align.json` ih navodi). Izvanmrežni način rada također prihvaća indekse pojaseva počevši od 0, npr. `--channel red=0 --channel nir=1`.

`lattice index` također radi izvan mreže na spremljenoj poravnatoj više-pojasnoj TIFF datoteci. Svaki obrazac radi na pozadinskom sustavu, stoga mora biti pokrenut i prijavljen na plaćenom Chloros+ razinu; relativni `--input` ili `--output` rješava se u odnosu na mapu iz koje pokrećete naredbu:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksni presetovi

`lattice index --preset` (i [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na kartici Slika, koji koristi isti mehanizam) dolazi s ovih **22 preseta**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Pokrenite `chloros-cli lattice index --list-presets` za formulu i simbole kanala svake postavke i `--list-gradients` za dostupne gradijente boja. Prilagođene formule koriste `--formula EXPR` sa sintaksom istom kao kod Izračunavača indeksa. Imajte na umu da je ovaj popis presjeta specifičan za indeksni motor LATTICE — padajući izbornik obrade u Postavkama projekta za uvožene snimke je drugačiji popis (vidi [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md)).

Kompletan skup zastavica (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, ručke za poravnanje (alignment warp knobs) za `--live` i drugo) dokumentirano je u [CLI Reference § Index / Vegetation Maths](../reference/cli-reference.md#index--vegetation-maths); Ekvivalenti za SDK nalaze se u [SDK Referenci](../reference/sdk-reference.md).

## Snimanje indeksnih proizvoda iz mono niza

S priključenim nizom i primijenjenim indeksnim izrazom, `array-capture` (ili GUI **Snimanje svega**) sprema razine izvoza po kameri *i* prikaz indeksa — `--index`/`--no-index` uključuje ga u CLI, a zadano snima svaku primjenjivu razinu. Doprinos mono kamere svakoj grupi snimanja je njezina jedna traka na razinama sirovog/debayera (skala sive)/radijance/reflektancije, plus zajednički kompozit kombiniranog indeksa kada niz radi u kombiniranom načinu rada. Vidi [Multi-Camera Arrays § Capturing](arrays.md#capturing-monitoring-vs-analysis).
