# Postavke i načini snimanja

Snimanje na kartici Kamere pokreće jedno crveno gumbom **Snimaj sve**i jedan panel**Postavke snimanja** koji određuje što taj gumb proizvodi: koje kamere sudjeluju, koje vrste izvoza svaka kamera sprema i hoće li se okidač aktivirati jednom, kontinuirano ili u intervalima. Ova stranica opisuje cijeli tijek rada — konfiguraciju, sam proces snimanja, lokaciju na disku gdje se datoteke spremaju i kako ih kasnije ponovno obraditi u kalibrirane proizvode. Kontrole za kameru i niz nalaze se na stranici [Postavke kamere](camera-settings.md).

{% hint style="info" %}
**Za snimanje je potreban otvoren projekt.** Naredbe Snimi sve i zupčanik za postavke snimanja onemogućeni su dok projekt nije otvoren (&quot;Stvorite ili otvorite projekt za spremanje snimki&quot;). Svaka se snimka sprema u mapu projekta u `captures/`.
{% endhint %}

## Ploča za postavke snimanja

Otvorite je pomoću **zupčanika pored Snimi sve**u bočnoj traci s popisom kamera ili pomoću gumba**&quot;Otvori postavke snimanja…&quot;** na dnu bilo koje ploče s postavkama za pojedinačnu kameru. Naslov glasi &quot;Postavke snimanja&quot; s gumbom za povratak ←.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Vaši odabiri ovdje — uključene kamere, potvrdni okviri za svaku vrstu i način snimanja — spremaju se **po projektu** i vraćaju se kada ga ponovno otvorite.

### Načini snimanja

Tri gumba za načine snimanja na vrhu prozora:

| Način | Što radi | Podpodešavanja (zadano) |
| --- | --- | --- |
| **Jednokratno** *(zadano)* | Jedno snimanje za svaku odabranu kameru. | — |
| **Kontinuirano**| Uzastopni snimci dok se ne zadovolji uvjet zaustavljanja. | Zaustavljanje po**broju snimaka** (zadano 1) *ili* **duljini snimanja** (zadano 10 s; jedinice: sekunde / minute / sati / dani). |
| **Interval**(time-lapse) | Serije snimaka na tajmeru. |**Snimaka po intervalu**(zadano 1) ·**Svake**N jedinice (zadano 5 s) ·**Tijekom** N jedinica (zadano 1 m). |

U načinu neprekidnog ili intervalnog snimanja gumb &quot;Snimaj sve&quot; tijekom snimanja postaje gumb **Zaustavi (N)**, pri čemu se snimke broje kako dolaze.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Odabir kamera i vrsta izvoza

Tekst pomoći u oknu sažima: odaberite koje kamere i vrste izvoza &quot;Capture All&quot; proizvodi — sve je prema zadanim postavkama uključeno, a odabiri se spremaju s ovim projektom.

* Gumbi **Odaberi sve / Odaberi nijedno** odjednom prebacuju potvrdni okvir za uključivanje/isključivanje za svaku kameru.
* **Gumbi za masovni odabir tipova izvoza**(dva reda gumba):**Svi Raw / Svi Debayered / Svi pregled / Svi Radiance / Svi Reflectance / Svi Index**. Svaki je obojen u tri stanja: zelena ✓ = uključeno za svaku kameru koja to podržava, jantar – = uključeno za neke, sivo = nijedna. Prekidač je onemogućen kada nijedna povezana kamera ne podržava tu vrstu. Svi postaju sivi dok je aktivno najbrže snimanje (Fastest Capture).
* **Redovi po kameri**: potvrdni okvir za uključivanje, plus proširiva (▸/▾) lista primjenjivih tipova izvoza te kamere s pojedinačnim potvrdnim okvirima. Red prikazuje broj uključenih od ukupno dostupnih, npr. &quot;4/6&quot;.

### Vrste izvoza i koje ih kamere podržavaju

Postoji šest vrsta izvoza: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Samo primjenjive vrste prikazuju se u retku svake kamere:

| Vrsta izvoza | Sadržaj |RGB

(FRGB) | Bayer multispektralni (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerov mozaik (mono: pojedinačna traka) izravno iz senzora | ✓ | ✓ | ✓ |
| **Demonalizirano** | Linearna demosaikizacija (mono: 1-kanalna sivina) | ✓ | ✓ | ✓ |
| **Pregled** | Cijeli lanac prikaza (bijela boja + gama prema profilu kamere; multispektralno: rastezanje lažnih boja) | ✓ | ✓ | ✓ |
| **Zračenje** | float32 W/m²/sr/nm putem cijelog radiometrijskog lanca | — (nije dostupno) | ✓ | ✓ |
| **Reflekancija** | uint16 ρ (32768 = 1,0) | — (nije dostupno) | ✓ — prikazuje se samo kada kamera ima DAQ senzor svjetla (vlastiti ili naslijeđeni od niza) | isto kao i kod multispektralnog |
| **Indeks** | Prikaz indeksa vegetacije (LUT) | — | ✓ — zahtijeva omogućen, ne-prazan izraz indeksa na kameri i nije dostupan članovima kombiniranog niza (niz posjeduje jedan zajednički indeks) | — (indeksu su potrebne ≥2 trake; vidi [Mono kamere i indeksi vegetacije](mono-indices.md)) |

Radijansa i reflektansa nikada se ne nude za kamere sRGB

— radijansa po Bayerovom uzorku nema smisla za širokopojasni fotometrijski senzor.

### Najbrže snimanje

Prekidač **⚡ Najbrže snimanje — samo raw**(narančaste boje kada je uključen) poništava sve odabire za izvoz na**samo raw** — uz besplatni složeni indeks kombiniranih indeksa za nizove — kako bi se kadar snimio što je brže moguće: matematika radijance/reflektance/prikaza u potpunosti se preskače pri snimanju.

{% hint style="info" %}
**A `.daq` se i dalje sprema.** Kada je dodijeljen senzor svjetlosti, najbrži način snimanja (Fastest Capture) i dalje zapisuje DAQ očitanje svjetline pored sirovih kadrova — tako da se radijansa, reflektansa i indeksni proizvodi mogu izraditi kasnije ponovnom obradom (pogledajte [Ponovna obrada snimki](#re-processing-captures-into-calibrated-products)). Fastest Capture također ne uništava vaše odabrane potvrdne okvire: isključite ga i oni se vraćaju.
{% endhint %}

### Kontrole po nizu

Svaki priključeni niz dobiva vlastitu karticu grupe u oknu:

* **Kutija za potvrdu Uključi** (tri stanja za članove) i naziv niza s načinom prikaza: &quot;(kombinirano | odvojeno)&quot;.
* Kutija za potvrdu **Poravnano**(zadano**uključeno**): warpiranje članova izvozi se prema profilu poravnanja niza kako bi izvozni podaci bili pikselno usklađeni među kamerama. Raw ostaje neizobličen, ali nosi transformaciju u svojim metapodacima. (Sam profil izračunava se u [podu okvira za postavke nizova](camera-settings.md#alignment-co-registration-combined-only).)
* Redovi kamera članica ugniježđeni su unutar kartice.

Kartica niza također sadrži dva snimača. Smatrajte ih **nadzorom naspram analize**:

| Snimač | Razina | Što snima |
| --- | --- | --- |
| **● Snimanje indeksnog videa / ■ Zaustavi snimanje** *(samo za kombinirane nizove)* | **Praćenje** | Snima video uživo od kombiniranog indeksa pri 10 fps — 8-bitno, rezolucija pretpregleda, ugrađena LUT. Zahtijeva otvoreni projekt i prikaz uživo sa streamanja. Prikazuje kadrove + proteklo vrijeme snimanja. |
| **⦿ Snimanje sirovog naleta / ■ Zaustavi sirovi burst** *(bilo koji niz)* | **Analiza**| Sirovi Bayer kadrovi brzinom snimanja uživo (bez obrade) plus manifest po kadru i očitanja `.daq`, u `captures/bursts/`. Nakon serije snimki pojavljuje se gumb**Izradi video**: on ponovno obrađuje seriju izvan mreže u kalibrirani video — kombinirani indeks i/ili radijansa/reflektancije/indeksa po kameri — plus opcionalne TIFF datoteke. Izrada kombiniranog indeksa automatski se pokreće kada zaustavite seriju snimki. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Protok

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

&quot;Capture All&quot; Pritisnite **Capture All** u popisu kamera u bočnoj traci:

1. Svaka uključena, vidljiva, nezaustavljena kamera snima sa svojim odabranim vrstama izvoza. **Nizovi se aktiviraju kao jedan sinkronizirani okidač** (jedna sinkronizirana grupa za sve članove — vidi [Nizovi s više kamera](arrays.md)); samostalne kamere snimaju pojedinačno.
2. Skrivene (ikona oka) ili pauzirane kamere se preskaču. Niz se u potpunosti blokira samo kada su *svi* njegovi članovi skriveni ili pauzirani.
3. Kad god je dodijeljen senzor svjetla, odgovarajući DAQ downwelling očitanje spremaju se kao datoteka `.daq` uz snimke — čak i za snimanja samo u sirovom formatu — tako da se radiometrijski proizvodi uvijek mogu kasnije izvesti.
4. Gumb prikazuje trenutačni napredak — &quot;Snima se… gotovo/ukupno&quot; — a u načinu neprekidnog/intervalnog snimanja postaje **Zaustavi (N)**. Svaki element snimanja ima vremensko ograničenje od 300 s.
5. Kada se prolaz završi, rezultat bljeska prikazuje **&quot;Sačuvano N datoteka&quot;**ili**&quot;Sačuvano N, F neuspjelo&quot;**, plus &quot;(S skriveno/pauzirano preskočeno)&quot; kada su kamere preskočene.

## Gdje se spremaju snimke

Snimke se spremaju u otvoreni projekt u `<project>/captures/`. Svaka vrsta izvoza završava u vlastitoj podmapici, tako da se kod višerazinskog snimanja vrste nikada ne miješaju:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` je vremenski žig snimanja, a `<serial>` serijski broj kamere. Samostalne snimke nazivaju se `capture_<ts>_SN<serial>_<level>`; niz snimaka s jednog sinkroniziranog okidača nazivaju se `sync_<ts>_SN<serial>_<level>` i **dijele jedan vremenski žig među svim kamerama u grupi** ( sufiks razine se izostavlja kada kamera sprema samo jednu razinu).
* **Jedna asimetrija koju treba znati:** razina prikaza pohranjuje se u mapu naziva `preview/`, dok datoteke u nazivu zadržavaju `_display` — mapa i sufiks razlikuju se samo za tu razinu.
* Nezabilježene razine po zadanom se spremaju u mapu vlastitog naziva; ako se ne može stvoriti podmapu, datoteka se sprema u korijensku mapu snimki umjesto da se izgubi.
* TIFF-ovi snimki prema zadanim postavkama komprimiraju se bez gubitka (DEFLATE) i nose sve svoje metapodatke o kalibraciji i obradi **unutar XMP-a datoteke** — snimke su samopisujuće, bez pomoćnih datoteka osim `.daq` za čitanje.

Ovo je isti raspored `chloros-cli lattice capture` / `array-capture` zapisuju u svoj direktorij `-o` — dokumentirano u referenci [CLI

§ Kako izgleda mapa snimki](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ponovna obrada snimki u kalibrirane proizvode

Snimljeni sirovi kadrovi i spremljeni `.daq` su sve što je potrebno obradnom procesu — zato je Fastest Capture siguran za stvarni rad.

* **GUI**: dodajte mapu sa snimkama u projekt ([Dodavanje datoteka u projekt](../processing-images-gui/adding-files-to-a-project.md)) i obradite kao i obično.
* **CLI**: usmjerite `process` na **korijenski direktorij snimaka**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` obično uvozi samo mapu koju nazovete, ali kada ta mapa ne sadrži slike, a ima podmapu, automatski prelazi na nju — tako da se podmapu razine i korijenske datoteke `.daq` odabiru odjednom. Svako snimanje uvozi se kao **jedna slika** s pripojenim ostalim razinama kao načinima prikaza, a ne kao jedna slika po razini.

Izravno imenovanje podmappe razine (npr. `…/captures/raw/`) također radi, ali ostavlja iza sebe korijenske datoteke `.daq` — kopirajte ih zajedno kada ponovno izvodite radiometrijski proizvod iz `raw/`, inače usklađivanje vremenskih oznaka neće imati ništa na što bi se odnosilo.

{% hint style="warning" %}
**Obrada uvijek započinje s datoteke `raw`.**Unutar svakog snimanja sirovi kadar je izvor u procesu obrade; `debayered`, `radiance`, `reflectance`, i `preview` dolaze kao prikazivi načini rada, ali se nikada ne vraćaju kroz pipeline — ponovnom obradom izvedenog proizvoda ponovno bi se primijenili vinjetiranje, boja i radijantna matematika već ugrađeni u njegove piksele, stoga `index/` i `composite/` odbijaju ponovnu obradu. Renderi `index/` i `composite/` nikada se uopće ne obrađuju (to su izlazi, a ne snimke). Mapu &#x27;captures&#x27; spremljenu**bez** sirovih uvoza prikazuje normalno, ali `process` je preskače i o tome obavještava; `--input-level {raw,debayered,processed}` je namjenski izlaz koji prisilno stvara ulaznu točku. Pogledajte [CLI

referencu](../reference/cli-reference.md#what-a-captures-folder-looks-like) za točne poruke o preskakanju.
{% endhint %}

Još dva ponašanja koja vrijedi znati pri skripiranju ponovne obrade:

* Pokretanje `chloros-cli process` koje je zatražilo proizvode, ali nije zapisalo **ni jedan slikovni proizvod, završava s greškom i izlazi s ne-nultom vrijednošću** — nikada nećete dobiti tiho pokretanje s praznim rezultatom. Uspješna pokretanja izvještavaju o broju svojih proizvoda. (Namjerno pokretanje samo za metapodatke i dalje se smatra uspjehom.)
* Ponovno uvezene obrađene izvoze nikada ne zauzimaju sirovi slot snimanja — originalni sirovi podatak uvijek ostaje izvor u cijevi.

## Ekvivalenti zaCLI



Sve na ovoj stranici može se pokretati bez zaslona. GUI načini snimanja izravno se mapiraju na `chloros-cli lattice array-capture`:

| GUI |CLI

|
| --- | --- |
| Pojedinačno | `chloros-cli lattice array-capture` |
| Neprekidno | `array-capture --continuous [--count N] [--duration S]` |
| Intervalno | `array-capture --interval S [--duration S]` |
| Najbrže snimanje | `array-capture --fastest` |
| Poravnati potvrdni okvir | `--aligned / --no-aligned` |
| Kvadratice za izvoz | `--processing LEVEL` ili `--levels L1,L2,…` (zadano `all`) |
| Snimanje indeksa videozapisa | `chloros-cli lattice array-record` |
| Snimanje sirovog rafala / Izrada videozapisa | `chloros-cli lattice array-burst` / `array-build-video` |

Potpune tablice zastavica, opcija pametnog AE-a za zaključeno snimanje (`--smart`) i model održane brzine nalaze se u [CLI

Referenca § Načini snimanja, snimači i offline reprocesiranje](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
