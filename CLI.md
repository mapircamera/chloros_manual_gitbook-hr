# CLI: naredbeni redak

> **Potpuna referenca:**[CLI Reference](reference/cli-reference.md) dokumentira**svaku zastavicu svake podkomande** i optimizirana je za AI asistente — zalijepite njezinu URL u svoj asistent i zatražite radnu naredbu: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Savjet za AI alate:** bilo koja stranica ovog priručnika dostupna je kao sirovi Markdown dodavanjem `.md` na njezin URL (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indeksira cijeli priručnik za upotrebu s LLM-om.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Što jeCLI


`chloros-cli` je sučelje naredbenog retka za isti procesor koji koristi desktop aplikacijaChloros
. To je tanak klijentHTTP
a koji se povezuje s pozadinskim sustavomChloros
(lokalni poslužitelj na `127.0.0.1:5000`) — većina naredbi automatski pokreće pozadinski sustav, pa je jednom pozivu `chloros-cli process …` sve što skripta treba.

Pokreće se na **Windows
10/11 (x64)**i**Linux
(x86_64, i NVIDIA Jetson arm64 na JetPack 6)**, u bilo kojem terminalu, bez potrebe za GUI-jem. Provjerite svoju instalaciju pomoću:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Obitelji naredbi, na prvi pogled:

* **Obrada i račun** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jezika — vidi [Podržani jezici](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux
/samo za Jetson)
* **Hardver uživo** — `lattice` (kontrola kamere LATTICE, 45+ podzapovijedi), `daq pool-*` (DAQ senzori svjetla), `time-sync` (PTP)
* **Automatizacija** — `project` (pokretanje spremljenog projektaChloros
bez zaslona, uključujući YAML recepte za snimanje)

Globalne opcije koje vrijedi znati: `--port N` (port pozadinskog sustava, zadano `5000`), `-v/--verbose`, `--restart` (prisilno ponovno pokrenuti pozadinski sustav), `--backend-exe PATH`. Pogledajte [CLI
referencu](reference/cli-reference.md) za potpune popise.

***

## Instalacija

CLI
**dolazi unutar instalateraChloros** na svakoj platformi — nema zasebnog preuzimanjaCLI
. Preuzmite instalater s [Download](download.md) stranice.

###Windows


Instalater postavljaCLI
na:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

i dodaje tu mapu u vaš sustav `PATH` — **otvorite novi terminal**nakon instalacije kako bi ažurirani `PATH` bio prepoznat. Instalacijski program također postavlja skripte za pokretanje (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) u korijenski direktorij instalacije, kao i**Chloros
CLI
** prečac u izborniku Start, od kojih svaki otvara terminalu s `chloros-cli` spremnim za upotrebu.

###Linux


Instalirajte `.deb` za svoju arhitekturu:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Time se instalira `chloros-cli` na `/usr/bin/chloros-cli` (već na `PATH`) i pozadinski sustav na `/usr/lib/chloros/chloros-backend`, zajedno s okruženjem za izvršavanje ArenaSDK
potrebnim za LATTICE kamere. Pogledajte [Linux
Instalacija](linux/linux-installation.md) za detalje.

### Provjera

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Prijava i licenciranje

CLI
(i pristup putemPython
SDK
) zahtijeva **plaćeni planChloros
+**— svaki plaćeni paket ga ima; besplatni paket nema. Ograničenje se provodi**na strani poslužitelja** od strane pozadinskog sustava (backend), a ne binarnom datotekomCLI
: poziv bez prijave se odbija s `401 AUTH_REQUIRED`, i prijavljeni poziv na besplatnom planu s `403 PLAN_UPGRADE_REQUIRED`, bilo da dolazi s `chloros-cli`,SDK
ili ručno razvijenog klijentaHTTP
. Nadogradnja na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Prijava **jednom po računalu**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Lozinke sa specijalnim znakovima**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` se iskrivljuje u shellu (CLI
to otkriva kod 401 greške i automatski ponavlja pokušaj, ali jednostavne navodnike u potpunosti izbjegavaju problem).
{% endhint %}

Sesija se pohranjuje u `~/.chloros/user_session.json` i omogućuje rad izvan mreže tijekom probnog razdoblja plana (30 dana za mjesečne planove, do isteka za godišnje). `chloros-cli status` radi čak i bez plaćenog plana, pa je razlog odbijanja uvijek vidljiv.

{% hint style="danger" %}
**Zakazujete bezglavni posao? Prvo se prijavite.**Naredba za pokretanje pozadinskih procesa (`process`, `status`, `export-status`, …) pokrenute**bez pohranjenih sesija**ne daju brzu grešku — one prelaze u interaktivni prompt `Email:` / `Password:` na stdin. Stoga će neovereni cron zadatak ili CI korak**zastati čekajući unos**. Pokrenite `chloros-cli login EMAIL 'PASSWORD'` jednom na računalu prije zakazivanja bilo čega.
{% endhint %}

***

## Vaša prva izvedba obrade

Usmjerite `process` na mapu sa snimkama — automatski otkrivaSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng`, ili mješavina:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Progresni streamovi se generiraju za svaku nit cijevi (otkrivanje, analiza, obrada, izvoz), a uspješno pokretanje završava izvještavanjem o tome koliko je slikovnih proizvoda zapisano (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Gdje se spremaju izlazne datoteke

`process` piše u **mapu projekta**, a ne u vašu ulaznu mapu:

* Bez `-o`: projekt se stvara u vašoj zadanoj mapi projekta (zajedničkoj s GUI-jem; upravljajte njome pomoću `get-project-folder` / `set-project-folder`, rezervna `~/Chloros Projects`), nazvan od strane `-n/--project-name` ili vremenskim žigom (`YYYYMMDD_HHMMSS`) kada je izostavljen.
* Uz `-o PATH`: ta je mapa **projektna mapa**. Ako već sadrži `project.json`, umjesto prepravljanja stvara se sestrinska mapa s nastavkom `_1`/`_2`…

Unutar projekta, proizvodi su grupirani **po kameri, a zatim po formatu datoteke**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (odgovara EXIF-u snimke `Model`) i `<model>_<filter>` (npr. `Survey3N_RGN`) zaSurvey3
. Mapu formata slijede `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ili `tiff32` za `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Svaki izvezeni proizvod zadržava naziv izvornog datoteka.**Izvoz sjaja za `capture_..._raw.tif` i dalje se naziva `capture_..._raw.tif` — samo se nalazi u `tiff32/Radiance_Images/`.**Mapu identificira proizvod, a ne naziv datoteke**, stoga koristite glob za direktorij, a ne za nastavak `*radiance*`.
{% endhint %}

### Opcije koje ćete zapravo koristiti

| Zastavica | Zadano | Što radi |
| --- | --- | --- |
| `-o, --output PATH` | zadani projektni direktorij | Lokacija projektnog direktorija (vidi gore). |
| `-n, --project-name NAME` | vremenski žig | Naziv projekta. |
| `--format FMT` | `TIFF (16-bit)` | Jedan od `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | none | Indeksi vegetacije za izvoz (vidi [Indeksi vegetacije](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronski debayer, sporije, najviša kvaliteta (Chloros
+, NVIDIA GPU). |
| `--vignette / --no-vignette` | uključeno | Korekcija vinjete. |
| `--reflectance / --no-reflectance` | uključeno | Kalibracija reflektancije; za LATTICE ovo je također prekidač za proizvod reflektancije. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Nametanje ulazne točke u pipeline za LATTICE TIFF-ove. |

Za sve ostalo — podešavanje detekcije ciljeva, PPK, ekspozicijske igle, zastavice poravnanja niza — pogledajte [odjeljak `process` referentne knjigeCLI
](reference/cli-reference.md).

***

## Odabir stavki za izvoz (LATTICE proizvodi)

LATTICE obrada se primjenjuje na **sve primjenjive proizvode u jednoj fazi obrade**. Četiri prekidača po proizvodu**podrazumijevano su UKLJUČENA**; upotrijebite obrazac `--no-` da biste isključili jedan:

| Prekidač | Proizvod |
| --- | --- |
| `--debayered` | Linearna demosaika → `Debayered_Images/` |
| `--preview` | Prikaz pretpregleda (bijela ravnoteža + gamma; istezanje lažnih boja za multispektralne) → `Preview_Images/` |
| `--radiance` | float32 radijansa, W/m²/sr/nm → `Radiance_Images/` (uvijek `tiff32/`) |
| `--reflectance` | uint16 reflektancija, spremno za Pix4D → `Reflectance_Calibrated_Images/` |

RGB
glavne kamere uvijek emitiraju samo debayered + pregled — radijansa/reflektansa po pojasu nema smisla za širokopojasni senzor, pa su ti prekidači za njih neaktivni.Survey3
`.raw` zanemaruje te prekidače i slijedi standardnu putanju reflektance/cilja.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (po zadanom `auto`) odabire referencu reflektancije: `auto` provodi QA-pregled unutar kadra [cilj kalibracije](calibration-targets.md) apsolutna referenca i vraća se na DAQ-ovu podjelu svjetlosnog senzora prema dolje (ρ = π·L/E) kada cilj nije prisutan; `target` je stroga (bez DAQ zamjene); `daq` je autoritativan za DAQ. S `--target-reflectance-dir` se mogu isporučiti skeniranja cilja po jedinici mjerenja.

{% hint style="info" %}
**Čitanje pikseli odbijanog svjetla:**DN vrijednost koja znači ρ = 1,0 je**po izvoru** — Datoteke LATTICE utiskuju `Chloros:PixelScale=32768` u XMP; datotekeSurvey3
koriste 65535 (i ne nose oznake `Chloros:*`). Pročitajte oznaku i podijelite s njom umjesto da pretpostavljate konstantu. Pojedinosti i jedan namjerno ne-skalni rubni slučaj nalaze se u [referenciCLI
](reference/cli-reference.md).
{% endhint %}

**Obrada uvijek započinje s `raw`.** Izvedeni proizvodi (izvozi debayera/radijanse/reflektancije) nikada se ne vraćaju kroz pipeline — njihovo ponovno uvoženje i obrada bi dvaput primijenila kalibracijsku matematiku, stoga ihChloros
preskače i o tome obavještava.  je namjenski izlaz za slučaj nužde kada doista trebate prisilno odrediti ulaznu točku.

***

## Kada pokretanje ne uspije

Od verzije 1.2.0, `process` glasno propada umjesto da &quot;uspije&quot; bez ikakvih rezultata za pokazati:

* Pokretanje koje je **zatražilo proizvode, ali nije zapisalo nijedan**— samo `project.json` i `calibration_data.json` — ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nuljevim rezultatom**, tako da ih skripte mogu detektirati. Uobičajeni uzroci: ulazna mapa nije prepoznata kao snimanje (provjerite raspored i `--input-level`), ili je svaki zatraženi proizvod bio neprimjenjiv za te kamere (npr. traženje radijantnosti/reflektancije od kamera koje podržavaju samoRGB
).
* Namjerno pokretanje samo za metapodatke (svi prekidači za proizvode isključeni, bez `--indices`) i dalje se smatra uspješnim — prazan izlazni prikaz slike ispravan je rezultat.
* Ponovno pokrenite s `--verbose` i provjerite zapisnik pozadinskog sustava (backend log) za retke s `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, koji objašnjavaju preskakanja po kamerama.

Kodovi izlaska: `0` uspjeh · `1` opća pogreška · `2` pogreška u argumentu · `130` prekinuto tipkom Ctrl+C.

***

## Indeksi vegetacije

Pokrenite `--indices` s jednim ili više naziva unaprijed postavki; svaki indeks se smješta u vlastitu mapu `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 unaprijed definiranih naziva koje `process --indices` prihvaća:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Postoje tri indeksne liste — nemojte ih miješati.**Spuštajući izbornik Postavke projekta u GUI-ju sadrži 27 formula (dodaje `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — tih pet je dostupno samo u GUI-ju i**nije** važeće za `--indices`). Naredba live/offline `lattice index --preset` koristi vlastitu zasebnu listu od 22 unaprijed definirane postavke. Formula i matematičke operacije po trakama dokumentirane su u [Multispectral Index Formulas](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ senzori svjetla: Kratki pregled

Obitelj `daq pool-*` upravlja spektralnim DAQ senzorimaMAPIR
(DAQ-U putem USB-a, DAQ-M putem BLE-a, DAQ-E putem Ethernet-a) putem trajnog pool-a pozadinskog sustava — GUI,CLI
iSDK
dijele jedan upravljački objekt za uživo. **`pool-*` je podržana DAQ staza u isporučenomCLI
**; ostale podzapovijedi `daq` na koje možete naići suMAPIR
– površina samo za interne izvore – i izlaz s eksplicitnom greškom koja vas upućuje na `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` bez `--duration` radi do `pool-record --stop`; zadani izlazni direktorij je `~/Documents/DAQ Live View/` **na stroju pozadinskog sustava**. Profil korekcije kapa odabire se prilikom povezivanja (`--cap-id`, zadano na backendu `sunshine_cosine`) i može se mijenjati uživo pomoću `pool-set-cap` — profili kalibracije i kalibrirani raspon senzora obrađeni su u poglavljima o DAQ-u ovog priručnika.

{% hint style="warning" %}
**DAQ-E na hostu s više mrežnih kartica:** prvo automatsko otkrivanje `pool-connect --eth` nakon pokretanja može ne uspjeti čak i sa ispravnim senzorom. `--eth-host <ip-or-hostname>` je pouzdani način — koristite ga kad god otkrivanje ne da rezultate.
{% endhint %}

***

## LATTICE kamere, PTP i automatizacija projekta

Obitelj `lattice` (preko 45 podzapovijedi) pokriva rad s LATTICE kamerama od početka do kraja: otkrivanje, pojedinačna snimanja, trajne sinkronizirane nizove s GUI-jevim pametnim tijekom povezivanja za pripremu, pregled uživo u pregledniku, poravnanje, matematičke indekse i dijagnostiku mrežne kartice domaćina. Primjer:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Uz to: `chloros-cli time-sync` izvještava o PTP grandmasteru koji pokreće hostChloros
(LATTICE kamere i DAQ-E senzori rade kao sluge za njega za vremenske oznake među uređajima), a `chloros-cli project` otvara spremljeni projektChloros
i upravlja njegovim kamerama, nizovima i senzorima bez zaslona — uključujući skriptirane YAML recepte za snimanje.

Ove tri obitelji (`lattice`, `project`, `daq pool-*`) su također jedine koje podržavaju `CHLOROS_BACKEND_URL` za upravljanje **udaljenim** pozadinskim sustavom; osnovne naredbe uvijek ciljaju lokalno računalo.

Potpuni vodiči nalaze se u poglavljima LATTICE ovog priručnika; svaka zastavica nalazi se u referenci [CLI
](reference/cli-reference.md).

***

## Rješavanje problema: Top 5

| Simptom | Rješenje |
| --- | --- |
| `Login required`, ili zakazani posao zapeče na upitu `Email:` | Pokrenite `chloros-cli login EMAIL 'PASSWORD'` jednom na ovoj računalu — naredbe bez spremljene sesije reagiraju interaktivno umjesto da brzo zakažu. |
| `backend unreachable` | Pokrenite desktop aplikacijuChloros
ili izravno pokrenite pozadinski binarni program (`chloros-backend`). Ako usmjerite `lattice`/`project`/`daq pool-*` na udaljeni backend, provjerite `CHLOROS_BACKEND_URL`. |
| Spoj niza blokiran: `FRAMES WILL DROP` / `Reduce ROI to enable` | Resetiranje prstena za primanje mrežne kartice (NIC) hosta na zadane vrijednosti — glavni je razlog zašto se ranije ispravan rig odbija povezati, obično nakon ažuriranja upravljačkog programa za mrežnu karticu. Pokrenite `chloros-cli lattice network --fix` iz **podiženog** terminala (ili postavite `ReceiveBufferLen=256`, `PendingReceives=64`); pogledajte *Postavljanje i podešavanje mrežne kartice hosta* u referenci. |
| Podzapovijed `daq` izlazi: &quot;potreban je kompletan daq paket…&quot; | Očekuje se u isporučenim izgradnjama — kompajliranaCLI
isporučuje samo obitelj `daq pool-*`, koja pokriva povezivanje, streamanje, snimanje i odabir kapova. Koristite `pool-*` (ili `chloros_sdk.connect_daq_sensor()` izPython
). |
| Jetson ispisuje upozorenje o zamjenskom prostoru prije velikih mapa | Dodajte zamjenski prostor podržan datotekama —CLI
ispisuje točne `fallocate`/`swapon` naredbe za pokretanje. |

***

## Pomoć

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Svaki zastavica, svaka podzapovijed:** [CLI
Referenca](reference/cli-reference.md)
* **EkvivalentPython
:** [Python
SDK
](api-python-sdk.md) i [SDK
Referenca](reference/sdk-reference.md)
* **Podrška:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
