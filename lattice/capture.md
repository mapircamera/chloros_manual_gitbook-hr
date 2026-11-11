# Postavke i načini snimanja

Snimanje na kartici Kamere pokreće jedan crveni gumb **Snimaj sve**i okno**Postavke snimanja** koje određuje što taj gumb proizvodi: koje kamere sudjeluju, koje vrste izvoza svaka kamera sprema i hoće li se okidač aktivirati jednom, kontinuirano ili u intervalima. Ova stranica opisuje cijeli tijek rada — konfiguraciju, sam proces snimanja, lokaciju na disku gdje se datoteke spremaju i kako ih kasnije ponovno obraditi u kalibrirane proizvode. Kontrole za kameru i niz nalaze se na stranici [Postavke kamere](camera-settings.md).

{% hint style="info" %}
**Za snimanje je potreban otvoren projekt.** Postavke snimanja i gumb &quot;Snimaj sve&quot; onemogućeni su dok projekt nije otvoren (&quot;Stvorite ili otvorite projekt za spremanje snimki&quot;). Svaka se snimka sprema u mapu projekta u `captures/`.
{% endhint %}

## Ploča za postavke snimanja

Otvorite je pomoću **zupčanika pokraj opcije Snimi sve**u bočnoj traci s popisom kamera ili pomoću gumba**&quot;Otvori postavke snimanja…&quot;** na dnu bilo koje ploče s postavkama za pojedinačnu kameru. Naslov glasi &quot;Postavke snimanja&quot; s gumbom za povratak ←.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Vaši odabiri ovdje — uključene kamere, potvrdni okviri za svaku vrstu i način snimanja — spremaju se **po projektu** i vraćaju se kada ga ponovno otvorite.

### Načini snimanja

Tri gumba za načine rada na vrhu prozora:

| Način | Što radi | Podpodešavanja (zadano) |
| --- | --- | --- |
| **Jednokratno** *(zadano)* | Jedno snimanje na svakoj odabranim kameri. | — |
| **Neprekidno**| Uzastopna snimanja dok se ne ispuni uvjet za zaustavljanje. | Zaustavljanje prema**broju snimki** (zadano 1) *ili* **duljini snimanja** (zadano 10 s; jedinice: sekunde / minute / sati / dani). |
| **Interval**(time-lapse) | Serije snimki s tajmerom. |**Snimki / interval**(po zadanom 1) ·**Svake**N jedinica (po zadanom 5 s) ·**Tijekom** N jedinica (po zadanom 1 m). |

U načinu rada Kontinuirano ili Interval, gumb Snimi sve postaje gumb **Zaustavi (N)** tijekom snimanja, brojeći snimke kako stižu. Pun disk zaustavlja snimanje (vidi [Protok snimanja svega](#the-capture-all-flow)).**Neprekidno snimanje na nizovima radi u pozadini (novost u verziji 1.3.0).** Kada svaka kamera na popisu pripada nizu (samostalne kamere su skrivene ili isključene), način rada Kontinuirano više ne šalje jedan zahtjev po snimci: pozadinski sustav Chloros

samostalno pokreće petlju snimanja, zapisujući jednu sinkroniziranu grupu dok odabire sljedeću, a gumb provjerava svoj broj. Kada su na popisu samostalne kamere, snimanja sustava Chloros

prolaze korak po korak kao i prije.

{% hint style="warning" %}
**Brzo snimanje na Windows

u: zadržite backend Chloros

a izuzetim od skeniranja Microsoft Defendera.** Skeniranje svake datoteke koju brzo snimanje piše u stvarnom vremenu u testovima tvrtke MAPIR

koštalo je oko 30 % brzine snimanja (stotine datoteka u sekundi na malom ROI-u). Instalacijski program za Windows

nudi **&quot;Isključi pozadinski proces Chloros

iz skeniranja Microsoft Defendera (znatno brže snimanje kamere; preporučeno)&quot;**, što je označeno prema zadanim postavkama, te uklanja isključenje prilikom deinstalacije. Ako ste ga odznačili, ponovno pokrenite instalacijski program s označenom opcijom ili dodajte izuzeće procesa za `chloros-backend.exe` u Defenderu (u mapi `backend` unutar mape instalacije programa Chloros

). Kada Defender koristi jedno ili više jezgri procesora tijekom kontinuiranog snimanja niza, pozadinski program **Chloros** to jednom zabilježi u svom dnevniku, s istim tim dvjema ispravcima.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Odabir kamera i vrsta izvoza

Pomoćni tekst u oknu sažima: odaberite koje kamere i vrste izvoza &quot;Capture All&quot; proizvodi — sve je prema zadanim postavkama uključeno, a odabiri se spremaju s ovim projektom.

* Gumbi **Odaberi sve / Odaberi nijedno** istovremeno prebacuju potvrdne okvire za uključivanje/isključivanje za sve kamere.
* **Gumbi za masovno prebacivanje tipova izvoza**(dva reda gumba):**Sve sirove / Sve de- Bayerizirane / Sve pretpregleda / Sve Radiance / Sve refleksije / Sve indeksa**. Svako je stanje obojeno u tri boje: zelena ✓ = uključeno za svaku kameru koja to podržava, žuta – = uključeno za neke, siva = nijedna. Preklopnik je onemogućen kada nijedna priključena kamera ne podržava taj tip. Svi postaju sivkasti dok je omogućen način &quot;Najbrži snimak&quot;.
* **Redovi po kameri**: potvrdni okvir za uključivanje, plus proširiva (▸/▾) lista primjenjivih tipova izvoza za tu kameru s pojedinačnim potvrdnim okvirima. Redak prikazuje broj uključenih od 6, npr. &quot;4/6&quot;.

### Vrste izvoza i kamere koje ih podržavaju

Postoji šest vrsta izvoza: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Samo primjenjive vrste prikazuju se u retku svake kamere:

| Vrsta izvoza | Sadržaj | RGB

(FRGB) | Bayer multispektralni (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerov mozaik (mono: pojedinačna traka) izravno sa senzora | ✓ | ✓ | ✓ |
| **Debayered** | Linearna demosaikacija: dark offset (DSNU) kalibracijskog paketa uklonjen iz sirovog datoteka, zatim demosaikacija osjetljiva na rubove — bez korekcije linearnosti, flat filed ili unmixa. Spremljeno kao 3-kanalni RGB

TIFF

(mono: 1-kanalna sivina) | ✓ | ✓ | ✓ |
| **Pregled** | Cijeli lanac prikaza (bijela ravnoteža + gamma prema profilu kamere, plus postavka za istaknuta područja kamere RGB

; multispektralno: rastezanje u lažnim bojama) | ✓ | ✓ | ✓ |
| **Zračenje** | float32 W/m²/sr/nm putem cijelog radiometrijskog lanca | — (nije dostupno) | ✓ | ✓ |
| **Odbijanje** | uint16 ρ (32768 = 1.0) | — (nije dostupno) | ✓ — prikazuje se samo kada kamera ima DAQ senzor svjetla (vlastiti ili naslijeđeni od niza) | isto kao i kod multispektralnog |
| **Indeks** | prikaz vegetacijskog indeksa (LUT) | — | ✓ — zahtijeva omogućen, ne-prazan izraz indeksa na kameri i ne nudi se članovima kombiniranog niza (niz posjeduje jedan zajednički indeks) | — (indeks treba ≥2 pojaseva; pogledajte [Mono kamere i indeksi vegetacije](mono-indices.md)) |

Radijanca i reflektancija nikada se ne nude za kamere u boji (RGB

) — radijanca po Bayerovoj matrici nema smisla za širokopojasni fotometrijski senzor.

### Najbrže snimanje

Prekidač **⚡ Najbrži snimak — samo raw**(narančaste boje kada je uključen) nadjačava sve odabire za izvoz na**samo raw** — uz besplatni složeni indeks kombiniranog niza za nizove — kako bi se kadar snimio što je brže moguće: matematika radijance/reflektance/prikaza se u potpunosti preskače pri snimanju.

{% hint style="info" %}
**`.daq` se i dalje sprema.** Kada je dodijeljen senzor svjetla, Fastest Capture i dalje zapisuje DAQ downwelling očitanje (u mapu snimki `daq/`) — tako da se radijantni, reflektantni i indeksni proizvodi mogu izraditi kasnije ponovnom obradom (vidi [Ponovna obrada snimki](#re-processing-captures-into-calibrated-products)). Fastest Capture također ne mijenja vaše odabire u poljima za potvrdu: isključite ga i oni se vraćaju.
{% endhint %}

### Kontrole po nizu

Svaki priključeni niz dobiva svoju karticu grupe u oknu:

* **Kutija za potvrdu Uključi** (tristoje stanje po članovima) i naziv niza s njegovim načinom prikaza: &quot;(kombinirano | odvojeno)&quot;.
* **Poravnano**potvrdni okvir (zadano**uključeno**): warp članova izvozi se prema profilu poravnanja niza kako bi izvozni podaci bili pikselno usklađeni među kamerama. Raw ostaje neizobličen, ali nosi transformaciju u svojim metapodacima. (Sam profil izračunava se u [paneu postavki niza](camera-settings.md#alignment-co-registration).) Datoteke članova niza spremaju se u nativnoj orijentaciji svake kamere, pri čemu se bilo koja postavka ogledala/rotacije bilježi u datoteci umjesto da se primjenjuje.
* Redovi članica kamera ugniježđeni su unutar kartice.

Kartica niza također sadrži dva snimača. Smatrajte ih **nadzorom naspram analize**:

| Snimač | Razina | Što snima |
| --- | --- | --- |
| **● Snimanje indeksnog videa / ■ Zaustavljanje snimanja** *(samo za kombinirane nizove)* | **Praćenje** | Snima kompozit uživo kombiniranog indeksa u videozapis pri 10 fps — 8-bitno, rezolucija pretpregleda, ugrađena LUT. Potrebno je otvoriti projekt i prikaz uživo za streaming. Prikazuje kadrove i proteklo vrijeme tijekom snimanja. |
| **⦿ Snimanje sirovog naleta / ■ Zaustavi sirovi burst** *(bilo koja niz)* | **Analiza**| Sirovi Bayer kadrovi brzinom snimanja uživo (bez obrade) plus manifest po kadru i očitanja `.daq`, u `captures/bursts/`. Nakon serije snimki pojavljuje se gumb**Izradi video**: on ponovno obrađuje seriju snimki izvan mreže u kalibrirani video — kombinirani indeks i/ili radijansa/reflektancije/indeksa po kameri — plus opcionalne TIFF datoteke. Izrada kombiniranog indeksa automatski se pokreće kada zaustavite seriju snimki. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Protok

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

&quot;Snimanje svega&quot;

Pritisnite **Snimanje svega** na popisu kamera u bočnoj traci:

1. Svaka uključena, vidljiva, neprekinuta kamera snima sa svojim odabranim vrstama izvoza. **Nizovi se aktiviraju kao jedan sinkronizirani okidač** (jedna sinkronizirana grupa koja obuhvaća sve članove — pogledajte [Nizovi s više kamera](arrays.md)); samostalne kamere snimaju pojedinačno.
2. Skrivene (ikona oka) ili pauzirane kamere se preskaču. Niz se u potpunosti blokira samo kada su *svi* njegovi članovi skriveni ili pauzirani.
3. Kad god je dodijeljen senzor svjetla, odgovarajući DAQ downwelling očitanje sprema se kao datoteka `.daq` u mapu sa snimkama `daq/` — čak i za snimke samo u sirovom obliku — kako bi se radiometrijski proizvodi uvijek mogli izvesti kasnije.
4. Gumb prikazuje trenutačni napredak — &quot;Snima se… gotovo/ukupno&quot; — a u načinu neprekidnog/intervalnog snimanja postaje **Zaustavi (N)**. Svaki element snimanja ima vremensko ograničenje od 300 s.
5. Kada se prolaz završi, rezultat bljeska prikazuje **&quot;Sačuvano N datoteka&quot;**ili**&quot;Sačuvano N, F nije uspjelo&quot;**, plus &quot;(S skriveno/pauzirano preskočeno)&quot; kada su kamere preskočene. Ako se disk popuni, izvršavanje se zaustavlja i prikazuje**&quot;Zaustavljeno: disk je pun. Spaseno N, F neuspjelo&quot;** — svaka datoteka koju nije bilo moguće zapisati računa se kao neuspjela.

## Gdje se pohranjuju snimke

Snimke se pohranjuju u otvorenom projektu u `<project>/captures/`. Svaka vrsta izvoza smješta se u vlastitu podmapu, pa se kod višerazinskih snimaka vrste nikada ne miješaju:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` je vremenski žig snimanja, a `<serial>` serijski broj kamere. Samostalne snimke nazivaju se `capture_<ts>_SN<serial>_<level>`; niz snimaka s jednog sinkroniziranog okidača nazivaju se `sync_<ts>_SN<serial>_<level>` i **dijele jedan vremenski žig među svim kamerama u grupi** ( sufiks razine se izostavlja kada kamera sprema samo jednu razinu).
* **Jedna asimetrija koju treba znati:** razina prikaza pohranjuje se u mapu naziva `preview/`, dok datoteke u nazivu zadržavaju `_display` — mapa i sufiks razlikuju se samo za tu razinu.
* Za nepoznate razine koristi se mapa vlastitog naziva; ako se podmapu ne može stvoriti, datoteka se sprema u korijensku mapu snimki umjesto da se izgubi.
* TIFF datoteke snimki prema zadanim postavkama komprimirane su bez gubitka (DEFLATE) i sadrže sve svoje metapodatke o kalibraciji i obradi **unutar XMP-a datoteke** — snimke su samopisne, bez pomoćnih datoteka osim `.daq` za čitanje (i njezine `.daq.cameras.json`).
* **Maske za ograničavanje (novo u verziji 1.3.0).**Dok je kod 3×3 spektralne dekonvolucije trebalo ograničiti pojas na 0 na barem jednom pikselu, proizvodi radijance, reflektance i indeksa dobivaju masku po pikselu, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = pojas u R kanalu proizvoda, bit 1 = G, bit 2 = B); XMP oznaka proizvoda `Chloros:UnmixClampMask` upućuje na nju. Maske se nalaze**pored** mapa proizvoda, nikada unutar njih, pa se mapa proizvoda poput `reflectance/` može predati softveru za fotogrammetriju u izvornom obliku. Uvoznu datoteku samog programa Chloros

ne učitava maske.
* **Orijentacija (1.3.0).** Za samostalnu kameru, proizvodi za pregled, radijancu, reflektancu i indeks okreću se postavkom ogledala/rotacije; datoteke u sirovom formatu (Raw) i one s dekodiranom bojom (Debayered) ostaju onakve kakve ih je senzor pročitao. Sve datoteke članice niza ostaju u izvornom obliku. Svaka datoteka bilježi postavku u svom XMP-u, pa se pri ponovnoj obradi primjenjuje na isti način.
* **Kamera kojoj pripada `.daq` (1.3.0).** `.daq` snimanja, tijekom obrade, podudara se samo sa slikama s kamera koje njezina `.daq.cameras.json` navodi. Samostalna kamera bez vlastitog senzora svjetla stoga više ne posuđuje očitanje druge kamere — povežite senzor s njom ili ga zabilježite na kartici Senzori svjetla. Snimanje sa senzora svjetla (i bilo koji `.daq` bez te datoteke, uključujući i starije snimke) i dalje služi svim kamerama.

Ovo je isti raspored u koji `chloros-cli lattice capture` / `array-capture` pišu u svoj direktorij `-o` — dokumentirano u [CLI

reference § Kako izgleda direktorij sa snimkama](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ponovna obrada snimki u kalibrirane proizvode

Snimljeni sirovi kadrovi i spremljena datoteka `.daq` sve su što je potrebno obradnom procesu — zato je Fastest Capture siguran za stvarni rad.

* **GUI**: dodajte mapu sa snimkama u projekt ([Dodavanje datoteka u projekt](../processing-images-gui/adding-files-to-a-project.md)) i obradite kao i obično.
* **CLI**: usmjerite `process` na **korijensku mapu sa snimkama**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` obično uvozi samo mapu koju navedete, ali kada ta mapa ne sadrži slike, a ima podmapu, automatski prelazi u nju — tako da se podmapu i mapa `daq/` učitaju odjednom. Svako snimanje uvozi se kao **jedna slika** s ostalim razinama priloženima kao prikazivi načini, a ne kao jedna slika po razini.

Izravno imenovanje podmappe razine također radi, ali ostavlja mapu `daq/` netaknutom — kopirajte je zajedno kada ponovno- izvedite radiometrijski proizvod iz `raw/`, inače usklađivanje vremenskog žiga nema ništa na što bi se moglo odnositi.

{% hint style="warning" %}
**Obrada uvijek započinje s `raw`.** Unutar svake snimke sirovi kadar je izvor u procesu obrade; `debayered`, `radiance`, `reflectance`, a `preview` dolaze kao prikazivi načini rada, ali se nikada ne vraćaju kroz pipeline — ponovnom obradom izvedenog proizvoda ponovno bi se primijenili vinjetiranje, boja i radijanska matematika već ugrađeni u njegove piksele, pa Chloros

to odbacuje umjesto dvostruke obrade. Renderiranja `index/` i `composite/` nikada se uopće ne obrađuju (to su izlazi, a ne snimke). Snimljena mapa &#x27;A captures&#x27; spremljena **bez** sirovih uvoza prikazuje se normalno, ali `process` je preskače i o tome obavještava; `--input-level {raw,debayered,processed}` je namjenski izlaz koji prisilno stvara ulaznu točku. Pogledajte [CLI

referencu](../reference/cli-reference.md#what-a-captures-folder-looks-like) za točne poruke o preskakanju.
{% endhint %}

Još dva ponašanja koja vrijedi znati pri skripiranju ponovne obrade:

* Pokretanje `chloros-cli process` koje je zatražilo proizvode, ali nije napisalo **ni jedan slikovni proizvod, završava s greškom i izlazi s ne-nultom vrijednošću** — nikada nećete dobiti tiho pokretanje s praznim rezultatom. Uspješna pokretanja izvještavaju o broju proizvedenih proizvoda. (Namjerno pokretanje samo za metapodatke i dalje se smatra uspjehom.)
* Ponovno uvezeni obrađeni izvozni podaci nikada ne zauzimaju sirovi slot snimanja — originalni sirovi podaci uvijek ostaju izvor u cijevi.

## Ekvivalenti za CLI



Sve na ovoj stranici može se pokretati bez zaslona. GUI načini snimanja izravno se mapiraju na `chloros-cli lattice array-capture`:

| GUI | CLI

|
| --- | --- |
| Pojedinačno | `chloros-cli lattice array-capture` |
| Neprekidno | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Najbrže snimanje | `array-capture --fastest` |
| Poravnani potvrdni okvir | `--aligned / --no-aligned` |
| Kvadratice za vrstu izvoza | `--processing LEVEL` ili `--levels L1,L2,…` (zadano `all`) |
| Snimanje indeksnog videa | `chloros-cli lattice array-record` |
| Snimanje sirovog naleta / Izrada videa | `chloros-cli lattice array-burst` / `array-build-video` |

Potpune tablice zastavica, opcija pametnog AE-a za zaključeno snimanje (`--smart`) i model održane stope nalaze se u [CLI

Reference § Načini snimanja, snimači i offline reprocesiranje](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
