# CLI: naredbeni redak

> **Potpuna referenca:**[CLI reference](reference/cli-reference.md) dokumentira**svaku zastavicu svake podkomande** i optimizirana je za AI asistente — zalijepite njezin URL u svoj asistent i zatražite radnu naredbu: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Savjet za AI alate:** bilo koja stranica ovog priručnika dostupna je kao sirovi Markdown dodavanjem `.md` na njezin URL (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indeksira cijeli priručnik za upotrebu s LLM-ovima.

> **Želite li da vaš asistent pokrene sam Chloros?** `chloros-mcp`, MCP poslužitelj instaliran uz Python SDK, radi s Claude Desktopom, Claude Codeom, Cursorom i drugim MCP klijentima: pogledajte [Korištenje Chloros s AI asistentima](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Što je CLI


`chloros-cli` je sučelje naredbenog retka za isti procesor koji koristi stolna aplikacija Chloros. To je jednostavan klijent za HTTP preko Chloros backenda (lokalnog poslužitelja na `127.0.0.1:5000`) — većina naredbi automatski pokreće backend, pa je jedno pozivanje `chloros-cli process …` sve što skripta treba. Backend potom ostaje u pokretanju za sljedeću naredbu: zaustavlja se na `chloros-cli logout` (osim ako je aplikacija otvorena ili je u tijeku pokretanje ili izgradnja), na `chloros-cli backend stop` ili samostalno nakon 30 minuta bez klijenta, pokretanja ili povezanog uređaja. `chloros-cli backend status` to pokazuje.

Radi na **Windows 10/11 (x64)**i**Linux (x86_64 i NVIDIA Jetson arm64 na JetPack 6)**, u bilo kojem terminalu, bez potrebe za GUI-jem. Provjerite svoju instalaciju pomoću:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Obitelji naredbi, na prvi pogled:

* **Obrada i račun** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jezika — vidi [Podržani jezici](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux/Jetson samo)
* **Hardver u stvarnom vremenu** — `lattice` (kontrola LATTICE kamere, 45+ podzapovijedi), `daq pool-*` (DAQ senzori svjetla), `time-sync` (PTP)
* **Automacija** — `project` (pokretanje spremljenog projekta Chloros bez zaslona, uključujući YAML recepte za snimanje)

Globalne opcije koje vrijedi znati: `--port N` (port pozadinskog sustava, zadano `5000`), `-v/--verbose`, `--restart` (prisilno ponovno pokretanje pozadinskog sustava), `--backend-exe PATH`. Pogledajte [Referencu za CLI](reference/cli-reference.md) za potpune popise.

***

## Instalacija

CLI **nalazi se unutar instalatera programa Chloros** na svakoj platformi — ne postoji zasebno preuzimanje CLI. Preuzmite instalater s stranice [Preuzimanje](download.md).

### Windows


Instalacijski program postavlja CLI na:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

i dodaje taj direktorij u vaš sustav `PATH` jednom (nadogradnja također uklanja duplicirane unose koje su ostavili prethodni instalatori) — **otvorite novi terminal**nakon instalacije kako bi se ažurirani `PATH` učitio. Instalacijski program također postavlja skripte za pokretanje (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) u korijenski direktorij instalacije, kao i**Chloros CLI** prečac u izborniku Start, od kojih svaki otvara terminala s `chloros-cli` spremnim za upotrebu.

### Linux


Instalirajte `.deb` za svoju arhitekturu pomoću `apt` (ne `dpkg -i`, koji preskače preporučene zahtjeve Python SDK-a):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Ovo instalira `chloros-cli` u `/usr/bin/chloros-cli` (već na `PATH`), programi za pozadinsku instancu i CLI raspakirani u `/usr/lib/chloros/backend/` i `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` je pokretač koji pokreće pozadinski program), vrijeme izvođenja Arena SDK-a koje pozadinski program treba za LATTICE kamere i Python SDK za sustav `python3`. Pogledajte [Instalaciju za [Linux]](linux/linux-installation.md) za detalje.

### Provjera

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Prijava i licenciranje

Pristup putem CLI a (i Python SDK a) zahtijeva **plaćeni**Chloros+**plan** — svaki plaćeni paket ga ima; besplatni paket nema. Ograničenje se provodi**na strani poslužitelja** od strane pozadinskog sustava (backenda), a ne binarnom datotekom CLI a: poziv bez prijave odbijen je s `401 AUTH_REQUIRED`, i prijavljeni poziv na besplatnom planu s `403 PLAN_UPGRADE_REQUIRED`, bilo da dolazi iz `chloros-cli`, SDK-a ili ručno razvijenog HTTP klijenta. Nadogradite na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Prijavite se **jednom po računalu**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Lozinke sa specijalnim znakovima**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` se kvari u shellu (CLI to otkriva kod 401 greške i automatski ponovno pokušava, ali jednostruke navodnike u potpunosti izbjegavaju problem).
{% endhint %}

Sesija se kešira u `~/.chloros/user_session.json` i nastavlja raditi izvan mreže tijekom probnog razdoblja plana (30 dana za mjesečne planove, do isteka za godišnje). `chloros-cli status` radi čak i bez plaćenog plana, tako da je razlog odbijanja uvijek vidljiv.

{% hint style="danger" %}
**Zakazujete pozadinske zadatke? Prvo se prijavite.**Naredba za pokretanje pozadinskih zadataka (`process`, `status`, `export-status`, …) pokrenute**bez pohranjenih sesija**ne završavaju brzo — one prelaze u interaktivni upit `Email:` / `Password:` na stdin. Stoga će neovereni cron zadatak ili CI korak**zastati čekajući unos**. Pokrenite `chloros-cli login EMAIL 'PASSWORD'` jednom na računalu prije zakazivanja bilo čega.
{% endhint %}

***

## Vaša prva izvedba obrade

Usmjerite `process` na mapu sa snimkama — automatski otkriva Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ili kombinacija:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Progresni streamovi su aktivni po svakoj pipeline niti (otkrivanje, analiza, obrada, izvoz), a uspješno pokretanje završava izvještavanjem o broju napisanih slikovnih proizvoda: `Image products written: N`, a zatim `(plus M clamp masks)` kada je tijekom izvršavanja također napisana [clamp maske](output-image-formats.md#clamp-masks) (one se ne računaju kao proizvodi).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Ako pozadinski proces Chloros već radi (na primjer, desktop aplikacije), `process` ga koristi i otvara svoj projekt u njemu. Odbija se dok je aplikacija otvorena na projektu koji je aplikacija otvorila: dovršite proces iz aplikacije ili zatvorite taj projekt. Ako je prethodna izvedba još uvijek u završetku, čeka do 15 sekundi prije nego što odbije. `--restart` prisilno pokreće novi backend. Na sustavu Linux, pozadinski proces koji pokreće systemd servis se nikada ne ponovno pokreće: `process` ga koristi onakvim kakav jest, i tamo gdje bi ga inače ponovno pokrenuo (`--restart`, ili import koji nije pronašao ništa), ispisuje `sudo systemctl restart <unit>` i izlazi `1`.

### Kamo se ispisuju rezultati

`process` zapisuje u **mapu projekta**, a ne u vašu ulaznu mapu:

* Bez `-o`: projekt se stvara u vašoj zadanoj mapi projekta (koja se dijeli s GUI-jem; upravljajte njome pomoću `get-project-folder` / `set-project-folder`; kad nijedan nije postavljen, `Documents/MAPIR/Chloros Projects` u vašem korisničkom mapu), nazvan po `-n/--project-name` ili po vremenskom žigu (`YYYYMMDD_HHMMSS`) ako je naziv izostavljen.
* S `-o PATH`: ta mapa **je** mapa projekta. Ako već sadrži `project.json`, `_1`/`_2`… stvara se pripadajuća datoteka s nastavkom umjesto prebrisavanja. Relativna `PATH` uzima se u odnosu na mapu iz koje pokrećete `chloros-cli`.

{% hint style="info" %}
**Relativne staze (promijenjeno u verziji 1.3.0).** Svaki putanj koji pozadinski program otvara — `-o`/`--output` u bilo kojoj naredbi (uključujući zadanu `output` naredbi za snimanje), `--input`, `--burst-dir`, `--profile`, `--save` i `--mask` — rješava se u odnosu na mapu iz koje pokrećete `chloros-cli`, pa `-o captures/` piše u `<current folder>/captures/`. U verziji 1.2.0 pozadinski je sustav rješavao te putanje u vlastitom direktoriju za instalaciju, što je rezultiralo greškom "Pristup je uskraćen". Za udaljeni pozadinski sustav (`CHLOROS_BACKEND_URL`) putanje se šalju onakve kakve jesu i rješavaju na toj računali.
{% endhint %}

{% hint style="warning" %}
**Windows: držite mapu projekta kratkom osim ako su uključene dugim putanjama.** S isključenim dugim putanjama na Windows (zadano za Windows), nijedna putanja do datoteke ne smije biti duža od 260 znakova, a proizvodi se nalaze do otprilike 130 znakova ispod mape projekta. `process` provjerava ovo prije pokretanja: pre duboka mapa projekta odbijena je porukom **"Mapa projekta je pre duboka za ovo računalo"**, najduža moguća mapa projekta i upute za uključivanje dugih putova — ništa se ne obrađuje. Koristite kraći `-o` (ili `set-project-folder`), ili uključite dugim putovima sustava Windows (opcija instalatera **Omogući dugim putovima sustava Windows**, označena prema zadanim postavkama) i ponovno pokrenite Chloros.
{% endhint %}

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

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (odgovara EXIF-u snimke `Model`) i `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3. Mapu formata slijede `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ili `tiff32` za `TIFF (32-bit, Percent)`.

Za multispektralnu kameru M3C, u mapi formata može se pojaviti i mapa `clamp_masks/`, pored mapa proizvoda (na primjer `tiff16/clamp_masks/Reflectance_Calibrated_Images/` i `tiff32/clamp_masks/Radiance_Images/`). Sadrži maske po pikselu radijantnih i reflektivnih produkata u kojima je band unmix ograničio negativnu vrijednost na nulu — vidi [Clamp masks](output-image-formats.md#clamp-masks). Nijedan folder s proizvodima nikada ne sadrži masku.

{% hint style="info" %}
**Svaki izvezeni proizvod zadržava naziv izvornog datoteka.**Izvoz radijance iz `capture_..._raw.tif` i dalje se naziva `capture_..._raw.tif` — samo se nalazi u `tiff32/Radiance_Images/`.**Mapu identificira proizvod, a ne naziv datoteke**, stoga koristite glob za direktorij, a ne za nastavak `*radiance*`.
{% endhint %}

### Opcije koje ćete zapravo koristiti

| Zastavica | Zadano | Što radi |
| --- | --- | --- |
| `-o, --output PATH` | zadani projektni direktorij | Lokacija projektnog direktorija (vidi gore). |
| `-n, --project-name NAME` | vremenski žig | Naziv projekta. |
| `--format FMT` | `TIFF (16-bit)` | Jedan od `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros ne skalira: na svim 8-bitnim slikama zapisuje se dublji format kao `TIFF (8-bit)`, i to se jednom navede. |
| `--indices NAME [NAME ...]` | none | Indeksi vegetacije za izvoz (vidi [Vegetation Indices](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuronski debayer, sporije, najviša kvaliteta (Chloros+, potreban je CUDA GPU: bez njega se izvršavanje koristi `standard`; odbijeno za LATTICE snimke, koje uvijek dobivaju `standard`). |
| `--vignette / --no-vignette` | uključeno | Korekcija vinjete. |
| `--reflectance / --no-reflectance` | uključeno | Kalibracija reflektancije; za LATTICE ovo je također prekidač za proizvod reflektancije. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Nametnite ulaznu točku u pipeline za LATTICE TIFF-ove. |

Za sve ostalo — podešavanje detekcije ciljeva, PPK, ekspozicijske oznake, oznake poravnanja niza — pogledajte [odjeljak `process` referentne dokumentacije za CLI](reference/cli-reference.md).

***

## Odabir stavki za izvoz (LATTICE proizvodi)

LATTICE obrada se primjenjuje na **sve primjenjive proizvode u jednoj fazi**. Četiri prekidača po proizvodu**podrazumijevano su UKLJUČENA**; upotrijebite obrazac `--no-` da biste isključili jedan:

| Prekidač | Proizvod |
| --- | --- |
| `--debayered` | Linearna demosaika → `Debayered_Images/`: uklonjen je dark offset kalibracijskog paketa, zatim slijedi demosaicing osjetljiv na rubove, bez korekcije linearnosti, flat filed ili unmixa — ista datoteka koju snimka sprema na svojoj razini `debayered`. Održana je izvorna orijentacija senzora (zabilježeno u XMP-u). |
| `--preview` | Prikaz pretpregleda (bijela boja + gamma; istezanje lažnih boja za multispektralnu) → `Preview_Images/` |
| `--radiance` | float32 radijansa, W/m²/sr/nm → `Radiance_Images/` (uvijek `tiff32/`) |
| `--reflectance` | uint16 reflektancija, spremno za Pix4D → `Reflectance_Calibrated_Images/` |

RGB glavne kamere uvijek emitiraju samo debayered + pregled — radijansa/reflektancija po pojasu nema smisla za širokopojasni senzor, stoga su ti prekidači za njih neaktivni. Survey3 `.raw` zanemaruje prekidače i slijedi standardnu putanju reflektancije/cilja.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (zadano `auto`) odabire referencu reflektancije: `auto` postavlja [cilj kalibracije](calibration-targets.md) u kadru koji prođe QA za apsolutnu referencu i vraća se na DAQ podjelu svjetlosnog senzora prema dolje (ρ = π·L/E) kada cilj nije prisutan (cilj koji prođe QA s oznakom natječe se s DAQ-om po mjerenom rezultatu); `target` je stroga (bez DAQ zamjene); `daq` je autoritativan za DAQ. Mogu se dostaviti skeniranja cilja po jedinici mjerenja s `--target-reflectance-dir`.

{% hint style="info" %}
**Čitanje reflektancijskih piksela:**DN vrijednost koja znači ρ = 1,0 je**po izvoru** — LATTICE datoteke upisuju `Chloros:PixelScale=32768` u XMP; Survey3 datoteke koriste 65535 i ne nose oznaku `Chloros:PixelScale` (no nose druge oznake obrade `Chloros:*`). Pročitajte oznaku i podijelite po njoj umjesto da pretpostavljate konstantu. Pojedinosti i jedan namjerno ne-skalni rubni slučaj nalaze se u [CLI referenci](reference/cli-reference.md).
{% endhint %}

**Obrada uvijek započinje s `raw`.** Izvedeni proizvodi (izvozi debayera/radijance/reflektance) nikada se ne vraćaju kroz pipeline — njihovo ponovno uvoženje i obrada bi dvaput primijenila kalibracijsku matematiku, stoga ih Chloros preskače i o tome obavještava. `--input-level` je namjenski izlaz za slučaj nužde kada doista trebate prisilno odrediti ulaznu točku.

***

## Kada pokretanje ne uspije

Od verzije 1.2.0, `process` glasno propada umjesto da "uspije" bez ikakvih rezultata:

* Pokretanje koje je **zatražilo proizvode, ali nije napisalo nijedan**— samo `project.json` i `calibration_data.json` — ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nultom vrijednošću**, tako da ih skripte mogu detektirati. Uobičajeni uzroci: ulazna mapa nije prepoznata kao snimka (provjerite raspored i `--input-level`), ili je svaki zatraženi proizvod bio neprimjenjiv za te kamere (npr. traženje radijantnosti/reflektancije od kamera koje rade samo u RGB formatu).
* Namjerno pokretanje samo za metapodatke (svaki proizvod je isključen, bez `--indices`) i dalje je uspješno — prazan izlazni prikaz slike ispravan je rezultat.
* Ponovno pokrenite s `--verbose` i provjerite zapisnik pozadinskog sustava (backend log) radi linija s `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, koje objašnjavaju preskakanja po kamerama.
* Na sustavu Windows u kojem je onemogućeno korištenje dugih putanja, obrada projekta koji je preglomazan za ograničenje putanje od 260 znakova odbijena je **prije** početka obrade, pri čemu se koristi najduža moguća putanja projekta (pogledajte [Where the outputs land](#where-the-outputs-land)).
* Naredbe snimanja (`lattice capture`, `lattice array-capture` i ostale obitelji naredbi za snimanje) navode svaku datoteku koju nisu mogle zapisati i izlaze kod `1`; ponavljajuća snimanja zaustavljaju se kada je pogon pun.

Kodovi izlaska: `0` uspjeh · `1` opća pogreška · `2` pogreška argumenta · `130` prekinuto tipkom Ctrl+C.

***

## Indeksi vegetacije

Pokrenite `--indices` s jednim ili više unaprijed definiranih naziva; svaki indeks se pohranjuje u vlastitu mapu `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 unaprijed zadanih imena koje `process --indices` prihvaća:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Dodaje se ime koje projekt još ne navodi. Ime koje nije unaprijed postavljeno ili ga pojasevi kamera ne mogu izraditi, preskače se uz upozorenje u kojem se navodi. Ako CLI ne može pročitati trenutačni popis indeksa projekta, ostavlja popis nepromijenjenim i umjesto zamjene izdaje upozorenje.

{% hint style="warning" %}
**Postoje tri popisa indeksa — nemojte ih miješati.**Spušteni izbornik Postavke projekta u GUI-ju ima 27 formula (dodaje `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — tih pet je dostupno samo u GUI-ju i**nije** važeće za `--indices`). Naredba live/offline `lattice index --preset` koristi vlastitu zasebnu listu od 22 unaprijed definirane postavke. Formula i matematičke operacije na trakama dokumentirane su u [Multispectral Index Formulas](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ senzori svjetla: Kratki pregled

Obitelj `daq pool-*` upravlja spektralnim DAQ senzorima MAPIR (DAQ-U putem USB-a, DAQ-M putem BLE-a, DAQ-E putem Ethernet) putem trajnog pool-a pozadinskog sustava — GUI, CLI i SDK dijele jednu aktivnu referencu. **`pool-*` je podržana DAQ putanja u isporučenom CLI**; ostale podzapovijedi `daq` na koje možete naići su internetska sučelja samo za izvor tvrtkeMAPIR
i izlaz s eksplicitnom greškom koja vas upućuje na `pool-*`.

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

`pool-record` bez `--duration` radi do `pool-record --stop`; zadani direktorij za izlaz je `~/Documents/DAQ Live View/` **na računalu pozadinskog sustava**. Profil korekcije kapa odabire se prilikom povezivanja (`--cap-id`, zadano na backendu `sunshine_cosine`) i može se mijenjati uživo pomoću `pool-set-cap` — profili kapa i kalibrirani raspon senzora obrađeni su u poglavljima o DAQ-u ovog priručnika.

{% hint style="warning" %}
**DAQ-E na hostu s više mrežnih kartica:** prvo automatsko otkrivanje `pool-connect --eth` nakon pokretanja može ne uspjeti čak i sa ispravnim senzorom. `--eth-host <ip-or-hostname>` je pouzdani način — koristite ga kad god otkrivanje ne da rezultate.
{% endhint %}

***

## LATTICE kamere, PTP i automatizacija projekta

Obitelj `lattice` (preko 45 podzapovijedi) pokriva rad s LATTICE kamerama od početka do kraja: otkrivanje, pojedinačna snimanja, postojane sinkronizirane nizove s pametnim tijekom povezivanja GUI-ja, pregled uživo u pregledniku, poravnanje, matematičke indekse i dijagnostiku mrežne kartice hosta. Primjer:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Uz to: `chloros-cli time-sync` izvještava o PTP grandmasteru koji pokreće host Chloros (LATTICE kamere i DAQ-E senzori mu služe kao sluge za vremenske oznake među uređajima), i `chloros-cli project` otvara spremljeni projekt Chloros i upravlja njegovim kamerama, nizovima i senzorima bez zaslona — uključujući skriptirane YAML recepte za snimanje.

Ove tri obitelji (`lattice`, `project`, `daq pool-*`) također su jedine koje podržavaju `CHLOROS_BACKEND_URL` za upravljanje **udaljenim** pozadinskim sustavom; osnovne naredbe uvijek su usmjerene na lokalno računalo.

Potpuni vodiči nalaze se u poglavljima LATTICE ovog priručnika; svaka zastavica je u [CLI referenci](reference/cli-reference.md).

***

## Rješavanje problema: Top 5

| Simptom | Rješenje |
| --- | --- |
| `Login required`, ili zakazani posao visi na upitu `Email:` | Pokrenite `chloros-cli login EMAIL 'PASSWORD'` jednom na ovoj računalu — naredbe bez spremljene sesije izvršavaju se interaktivno umjesto da brzo zakažu. |
| `backend unreachable` | Pokrenite desktop aplikaciju Chloros ili pokrenite pozadinski program sami: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` na Windows (mapa `backend` u mapi s instalacijom; nije na `PATH`), `chloros-backend` na Linux u. `chloros-cli backend status` prikazuje je li jedan pokrenut; `lattice`/`project`/`daq pool-*` nikada ne pokreće nijedan. Ako ih usmjerite na udaljeni backend, provjerite `CHLOROS_BACKEND_URL`. |
| Spojavanje niza blokirano: `FRAMES WILL DROP` / `Reduce ROI to enable` | Ring za primanje na mrežnoj kartici hosta resetiran na zadane vrijednosti — glavni uzrok zbog kojeg se ranije ispravno funkcionirajući uređaj odbija povezati, obično nakon ažuriranja upravljačkog programa mrežne kartice. Pokrenite `chloros-cli lattice network --fix` iz **podižnog** terminala (ili postavite `ReceiveBufferLen=256`, `PendingReceives=64`); pogledajte *Postavljanje i podešavanje mrežne kartice hosta* u referenci. |
| Podzapovijed `daq` izlazi: "potreban je cijeli DAQ paket…" | Koristite naredbe `daq pool-*`, kojima se upravlja istim senzorom putem pozadinskog sustava: connect, stream, record, odabir kapa, a za DAQ-E i mrežne postavke, ponovno pokretanje i ažuriranja firmvera (ili `chloros_sdk.connect_daq_sensor()` iz Python a). |
| Jetson ispisuje upozorenje o zamjenskom prostoru prije velikih mapa | Dodajte zamjenski prostor podržan datotekama — CLI ispisuje točne `fallocate`/`swapon` naredbe za pokretanje. |

***

## Pomoć

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Svaki parametar, svaka podzapovijed:** [CLI referenca](reference/cli-reference.md)
* **Python ekvivalent:** [Python SDK](api-python-sdk.md) i [SDK referenca](reference/sdk-reference.md)
* **Podrška:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
