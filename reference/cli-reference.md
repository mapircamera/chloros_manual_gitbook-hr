# Chloros Referenca

**Verzija:**

1.2.0**Generirano:**2026-07-29 19:19 ·**Revidirano:** 2026-08-30**Publika:** Optimizirano za LLM; ljudsko-čitljiv.**Područje primjene:** Svaka podkomanda `chloros-cli` namijenjena korisnicima, s opcijama i primjerima koje možete kopirati i zalijepiti.

Ovaj dokument je cjelovita referenca za alat za naredbeni redak `chloros-cli` koji se isporučuje s MAPIR Chloros. On je namjerno iscrpan kako bi LLM (ili čovjek) mogao sastaviti bilo koji podržani tijek rada iz navedenih popisa u nastavku bez pregledavanja izvornog koda.

Ako vam trebaju samo najvažnije informacije, idite na:
- [Petominutni brzi početak](#five-minute-quickstart)
- [Prvi postupak povezivanja kamere LATTICE](#lattice-camera-first-connect-workflow)
- [Prvi postupak povezivanja DAQ senzora](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Načini snimanja, snimači i offline reprocesiranje](#capture-modes-recorders--offline-reprocess)

---

## Konvencije

- Sve naredbe imaju prefiks `chloros-cli`. Na Windows binarna datoteka je `chloros-cli.exe`; na Linux/Jetson ona je `chloros-cli`.
- Izborni argumenti prikazani su kao `--flag`. Obavezni pozicijski argumenti prikazuju se bez zagrada.
- Ako je naveden zadani vrijednost, izostavljanjem zastavice koristi se ta vrijednost.
- CLI je tanak klijent HTTPa preko Chloros backenda (Flask poslužitelj na `127.0.0.1:5000`). Backend automatski pokreću većina naredbi. `CHLOROS_BACKEND_URL=<url>` usmjerava **`lattice`**,**`project`**i obitelji naredbi**`daq pool-*`** na udaljenom backendu — osnovne naredbe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) namjerno pinajte `http://127.0.0.1:<port>` i zanemarite ga (IPv4 literal izbjegava kaznu od ~2 s-per-request za `localhost`→`::1`). Vidi [Promjenjive okruženja](#environment-variables).
- Za sve pozive SDK / CLI potrebna je prijava na račun Chloros+ (pokrenite `chloros-cli login` jednom po računalu; pohranjuje se u predmemoriju u `~/.chloros/`).
- Primjeri koriste putanje Linux; na Windows zamijenite `/home/user/...` s `C:/Users/.../...`.

---

## Sinopsis najviše razine

```
chloros-cli [global options] COMMAND [command options]
```

### Globalne opcije

| Zastavica | Opis |
| --- | --- |
| `--backend-exe PATH` | Prepišite automatski otkrivenu izvršnu datoteku pozadinskog sustava. |
| `--port N` | Port pozadinskog sustava HTTP (zadano: `5000`). |
| `-v, --verbose` | Omogući detaljan izlaz. |
| `--restart` | Prisilno ponovno pokreni pozadinski program (zaustavlja sve aktivne instance `backend_server.py`). |
| `--version` | Ispiši verziju (`Chloros CLI 1.2.0`). |
| `--help` | Prikaži pomoć na najvišoj razini. |

### Indeks naredbi

| Naredba | Svrha |
| --- | --- |
| [`process`](#chloros-cli-process) | Obradi mapu s Survey3 ili LATTICE snimkama od početka do kraja. |
| [`login`](#chloros-cli-login) | Autentificirajte ovu mašinu s računom Chloros+. |
| [`logout`](#chloros-cli-logout) | Očistite pohranjene vjerodajnice. |
| [`status`](#chloros-cli-status) | Prikažite trenutačni status licence / provjere autentičnosti. |
| [`export-status`](#chloros-cli-export-status) | Prikaz napretka izvoza uživo za Thread-4 tijekom izvođenja `process`. |
| [`language`](#chloros-cli-language) | Postavi ili popiši jezik prikaza za CLI (38 podržanih). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Zadana mapa projekta (zajednička s GUI-jem). |
| [`update`](#chloros-cli-update) | Provjeri i instaliraj ažuriranja za CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Dijagnostika sustava + testovi dimljenja. |
| [`time-sync`](#chloros-cli-time-sync) | Status / kontrola PTP grandmastera. |
| [`lattice`](#chloros-cli-lattice) | Kontrola i snimanje LATTICE kamere (45+ podzapovijedi). |
| [`daq`](#chloros-cli-daq) | Kontrola DAQ spektralnog senzora (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otvori i upravljaj spremljenim projektom Chloros (kamere + DAQ-ovi). |

---

## Instalacija

`chloros-cli` dolazi u desktop instalacijskom programu Chloros na svakoj podržanoj platformi — nema zasebnog preuzimanja CLI. Instaliranje paketa platforme dodaje `chloros-cli` u vaš `PATH` uz desktop aplikaciju i pozadinsku binarnu datoteku kojom upravlja.

Najnovija preuzimanja: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Instalacijski program također uključuje skripte za pokretanje s radne površine (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) koji otvaraju spremnu za upotrebu CLI ljusku; oni su obuhvaćeni u [CLI korisničkom vodiču](../CLI.md) i nisu ovdje ponovljeni.

### Windows (.exe)

1. Preuzmite instalacijski program za Windows s stranice za preuzimanje.
2. Pokrenite `Chloros-Setup-x.y.z.exe` i slijedite čarobnjak. Zadana putanja za instalaciju je `C:\Program Files\Chloros\` (CLI se instalira u `C:\Program Files\Chloros\cli\`, što instalacijski program dodaje na PATH).
3. Otvorite novi terminal (`cmd.exe`, PowerShell ili Windows Terminal) kako bi ažurirani `PATH` bio prepoznat.

```powershell
chloros-cli --version
```

Instalacijski program automatski dodaje `chloros-cli.exe` u vaš sustav `PATH` i uključuje izvršni program Arena SDK potreban za LATTICE kamere.

### Linux amd64 (.deb)

Za Ubuntu 22.04 LTS ili novije / radne stanice temeljene na Debianu x86_64.

> **Ubuntu 20.04 nije podržan.** Popis ovisnosti paketa temelji se na
> onome s čime se pozadinski program zapravo povezuje, a to uključuje `libc6 (>= 2.34)`;
> focal isporučuje glibc 2.31. `apt` odbija instalaciju umjesto da dopusti da dođe do pogreške
> tijekom izvođenja.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

.deb instalira:
- `chloros-cli` u `/usr/bin/chloros-cli`
- Kompajlirani backend u `/usr/lib/chloros/chloros-backend`
- Runtime za Arena SDK (za LATTICE kamere)
- Modeli za uklanjanje šuma, paketi za kalibraciju i konfiguracija kanala ažuriranja

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ista struktura kao i kod amd64 .deb paketa, s CUDA izgradnjom optimiziranom za Jetson Orin / Orin NX / Orin Nano.

### Autentificirajte se jednom po računalu

Svaka platforma zahtijeva jednokratnu prijavu na Chloros+ prije nego što pozivi u SDK / CLI rade:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Vjerodajnice se pohranjuju u `~/.chloros/user_session.json`.

### Provjerite instalaciju

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Potrebna je pretplata na Chloros+.**CLI zahtijeva aktivan plan Chloros+.**Copper**je početno Chloros+ razina — svaka plaćena Chloros+ razina ima pristup CLI / SDK; samo besplatna**Iron** razina nema. (Mapa ID-a planova: `0`=Iron/besplatno, `1`=Bakreni, `2`=Brončani, `3`=Srebrni, `4`=Gold.) Nadogradnja na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Ovo ograničenje provodi pozadinski sustav, a ne samo CLI: zastavica SDK / CLIged zahtjev bez plaćenog plana odbijen je s `403 PLAN_UPGRADE_REQUIRED`, bilo da dolazi iz `chloros-cli`, Python SDK ili ručno razvijenog HTTP klijenta. Korisnik koji je odjavljен dobiva `401 AUTH_REQUIRED` umjesto toga. Pristup radi izvan mreže tijekom probnog razdoblja plana (30 dana mjesečno, do isteka za godišnji) i prestaje kad ono istekne; `chloros-cli status` nastavlja raditi pa je razlog vidljiv (to je jedina ruta SDK / CLI izuzeta od ulaza po razini — `GET /api/license-status`).

---

## Brzi početak u pet minuta

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Obradi mapu s slikama kroz cijeli Chloros pipeline (detekcija cilja → kalibracija → vinjeta → reflektancija → izvoz indeksa).

### Sinopsis

```
chloros-cli process INPUT [OPTIONS]
```

### Argumenti po položaju

| Argument | Opis |
| --- | --- |
| `INPUT` | Putnja do ulazne mape koja sadrži `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE), ili `.dng` datoteke. |

### Zajedničke opcije

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `-o, --output PATH` | novu mapu s vremenskim žigom u vašoj zadanoj putanji projekta (`~/Chloros Projects` osim ako nije konfigurirano) | Mapu projekta za stvaranje ili ponovnu upotrebu. Ako mapa već sadrži `project.json`, `_1`/`_2` se stvara sestrinska mapa umjesto prepravljanja. |
| `-n, --project-name NAME` | automatski (vremenski žig) | Naziv projekta. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` koristi Chloros+ neuronski debayer; sporije, ali kvalitetnije. |
| `--vignette / --no-vignette` | `--vignette` | Korekcija vinjete. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibracija reflektancije (koristi panelni target ako se pronađe, NIST kalibracija po serijskom broju za LATTICE). Za multispektralni LATTICE ovo također služi kao prekidač za **proizvod** reflektancije — vidi [Per-Product Export Toggles](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | off | Primijeni PPK GNSS korekcije iz pomoćnih datoteka. |
| `--exposure-pin-1 MODEL` | isključeno | Pričvrstite model &quot;pin-1&quot; za dvokameru s dvostrukim senzorom (Survey3). |
| `--exposure-pin-2 MODEL` | isključeno | Pričvrstite model &quot;pin-2&quot;. |
| `--recal-interval SECONDS` | 0 | Prisilno ponovno pokretanje kalibracijske matematike svakih N sekundi snimanja. |
| `--timezone-offset HOURS` | lokalno | Prepišite razliku u vremenskoj zoni ugrađenu u izlazne metapodatke. |
| `--format FORMAT` | `TIFF (16-bit)` | Jedan od `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | none | Indeksi vegetacije (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Prisilno odredi ulaznu točku u pipeline za LATTICE TIFF-ova (Survey3.raw ostaje nepromijenjen). Također, izlazna opcija koja omogućuje da se snimka s **no raw** uopće obradi — vidi [Kako izgleda mapa sa snimkama](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | on | Izlazni linearan debayered proizvod (`Debayered_Images`). Pogledajte [Per-Product Export Toggles](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | uključeno | Izlazna pretpregleda zaslona (`Preview_Images`): RGB = balans bijele (DAQ-illuminant kada je dostupan, inače svijetlo sive) + gamma; multispec = istezanje lažnih boja. |
| `--radiance / --no-radiance` | uključeno | Ispuniti radijancu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenca za LATTICE proizvod reflektancije: `auto` = QA-cilj u kadru koji prolazi je apsolutna referenca, DAQ-downwelling (ρ = π·L/E) kao rezervna opcija; `target` = strogo (bez DAQ zamjene); `daq` = DAQ-ovlašteno. Vidi [Per-Product Export Toggles](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | none | Directory of per-unit **mjerenih skeniranja reflektancije cilja (`<serial>.csv`); pri pogrešci prelazi na nominalna T3/T4P spektra. |
| `--array-alignment / --no-array-alignment` | uključeno | LATTICE nizovi: primijenite poravnanje modula-na-modul zabilježeno u XMP-u svake snimke `Chloros:Alignment*` na svaki obrađeni proizvod (debayered / pretpregled / radijansa / reflektancija / indeks). Bez učinka za slike bez oznaka. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | Izrezivanje poravnatih izvoza u zajedničku preklopnu regiju niza tako da svi moduli dijele jedan otisak; `--no-…` zadržava punu površinu senzora (crno popunjavanje izvan izvora). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ponovno uzorkovanje za deformaciju poravnanja. `nearest` čuva točne DN-ove izvora (bez inter-pikselnu mješavinu radiometrijskih vrijednosti). |

### Opcije za detekciju cilja

| Zastavica | Opis |
| --- | --- |
| `--min-target-size PIXELS` | Minimalna veličina cilja na panelu (px) za detektor. |
| `--target-clustering 0-100` | Osjetljivost klasteriranja. |
| `--target / --targets` | Obradi ulaznu mapu kao samo ciljni panel (preskoči detekciju snimanja). |

### Primjeri

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Prekidači za izvoz po proizvodu (LATTICE multispectral)

Obrada u LATTICE-u širi se na **sve primjenjive proizvode u jednom koraku**. Četiri prekidača po tipu — `--debayered`, `--preview`, `--radiance`, `--reflectance` — svi su**podrazumijevano su UKLJUČENI**; upotrijebite obrazac `--no-<type>` da isključite jedan. RGB glavne kamere uvijek emitiraju samo debayered + pretpregled (bez radijance po pojasu/reflektancije), pa su `--radiance`/`--reflectance` za njih no-op. Prekidači se zanemaruju za Survey3 `.raw` (koji slijedi standardnu putanju reflektancije/cilja). *(Stari `--radiometric-output {reflectance,radiance,sensor-response}` zastavica je **uklonjena** i zamijenjen ovim prekidačima; više ne postoji razina `sensor-response`.)*

| Proizvod | Izlaz | Potreban li DAQ downwelling? |
| --- | --- | --- |
| `--debayered` | Linearna demosaika (`Debayered_Images`). | Ne. |
| `--preview` | Prikaz pretpregleda (`Preview_Images`): RGB = WB + gamma; multispec = rastezanje lažnih boja. | Ne. |
| `--radiance` | float32 W/m²/sr/nm iz potpunog radiometrijskog lanca (`Radiance_Images`). | Ne. |
| `--reflectance` | uint16 reflektancija ρ (`32768` = 1.0), Pix4D-spremno. | **Da**, osim ako ga ne učvrsti cilj u kadru koji je prošao QA (pogledajte dolje). |

`--reflectance-source` odabire referencu reflektivnosti:**`auto`**(zadano) čini cilj unutar kadra koji je prošao QA**apsolutnom referencom**— empirijske lance linija vezane za cilj unakrsno se boduje na zadržanim panelima i primjenjuje se izmjereni pobjednik — a zatim se vraća na DAQ downwelling divide (ρ = π·L/E) kada cilj nije prisutan ili QA ne uspije;**`target`**je stroga (bez DAQ zamjene);**`daq`**odabire se DAQ-ov autoritativni postupak. Geometrija cilja (ArUco / fiksirani ROI / traka) dolazi iz konfiguracije cilja projekta; `--target-reflectance-dir DIR` po jedinici**mjerenih** skenova (`<serial>.csv`) pretraženih prema serijskom/QR kodu ciljne jedinice, s nominalnim T3/T4P spektrima kao rezervnom opcijom.

DAQ putanja reflektancije automatski rješava **timestampu podudaranu s dolaznim zračenjem**iz snimljenog**`.daq`**(DAQ-U/M/E)**ili nativni DAQ-M `.csv`**pronađen uz snimke. Ako paket kalibracije po kameri ili DAQ-u nije pohranjen lokalno, pipeline ga**automatski preuzima s AWS-a** pri prvom korištenju (zahtijeva internet samo jednom; pohranjen pod `~/.chloros/`).

#### Čitanje pikselja reflektancije (Pix4D / Metashape / vlastiti skripti)

Reflektancija se pohranjuje kao cjelobrojni DN, a **DN koji označava ρ = 1.0 ovisi o izvornoj kameri**:

| Izvor | ρ = 1,0 je | Kako odrediti |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva do ρ 2,0) | XMP `Chloros:PixelScale=32768` je zapečaćen na datoteci. |
| Survey3 | `65535` (odrezano na ρ 1.0) | Nema `Chloros:*` XMP oznaka — ta odsutnost *je* signal. |

**Pročitajte `Chloros:PixelScale` i podijelite s njim** umjesto da pretpostavljate konstantu. Oznaka je definirana u domeni uint16, pa ostaje `32768` u svim formatima izlaza koji vrše ponovno skaliranje — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` i `TIFF (32-bit, Percent)` su sami po sebi opisni (prvo normalizirajte pohranjenu vrstu podataka natrag na uint16: ×257 iz 8-bita, ×65535 iz float-a).

> **Jedan slučaj nema skalu, po dizajnu.** Kada se 8-bitni izvor (BayerRG8) zapisuje kao 8-bitni TIFF, pipeline *rezreže* na 0..255 umjesto ponovnog skaliranja, pa se svaka vrijednost iznad ρ≈0.008 izravnava na 255 i nijedna skala ne opisuje datoteku.Chloros namjerno izostavlja i `Chloros:PixelScale` i tuplet `MicaSense:RadiometricCalibration` tamo, i bilježi zašto. **Ako oznaka nedostaje u LATTICE datoteci s reflektancijom, nemojte pretpostavljati skalu — ponovno izvezite u 16-bitnom ili 32-bitnom formatu** umjesto da dijelite pigmente koji nikada nisu bili podijeljivi.

#### EXIF prenesen pri izvozu

`process` kopira **GPS blok i njegov ExifIFD** na svaki proizvod, tako da izvoz prenosi `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` i `CameraSerialNumber` uz georeferenciranje.**`FocalLength` nije neobavezan za fotogrametriju.** Pix4D rješava udaljenost uzorka tla (ground sample distance) na temelju žarišne duljine i visine; ako oznaka nedostaje, prelazi na potpuno pogrešnu mjerilu. Na jednom letu nad voćnjakom od 49 snimaka, nedostajuća oznaka pretvorila je 411 m × lokaciju od 160 m u rekonstruiranu od 47,8 km × 13 km — orto sliku od 455 MP s većinom nodata, što se zatim shvatilo kao problem s preklapanjem i problem s BigTIFF-om prije nego što je itko provjerio GSD. Ako vaša orto-mapa ima nevjerojatan mjerilo, prvo pokrenite `exiftool -FocalLength` na izvozu.

Kopija je namjerno **nije** `-all:all`: strukturne oznake IFD0-a prekidaju LATTICE izlaz kada
kopirane, a `ExifImageWidth` / `ExifImageHeight` su isključeni jer opisuju *izvorno* snimanje — izvoz koji je ikada promijenio veličinu inače bi nosio dimenzije koje su u suprotnosti s vlastitom rastrom. XMP se piše izravno, a ne kopira, jer ExifTool zanemaruje XMP oznake unutar iste poziva kada se XMP blok kopira (čime bi se uklonile oznake kalibracije MAPIR).

### Gdje se nalaze izlazne datoteke

Proizvodi se spremaju **ispod mape projekta, grupirani po kameri i zatim po formatu datoteke**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (odgovara EXIF-u snimke `Model`) i `<model>_<filter>` za Survey3 — dvije kamere koje dijele senzor i filtar, ali se razlikuju
u objektivu drže odvojena stabla, jer se vignetiranje, vidno polje i izobličenje razlikuju. Format
mapu slijedi `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ili `tiff32` za
`TIFF (32-bit, Percent)`.

> **Svaki izvezeni proizvod zadržava naziv izvornog datoteka.** Izvoz sjaja
> `capture_…_raw.tif` se i dalje naziva `capture_…_raw.tif` — samo se nalazi u
> `tiff32/Radiance_Images/`. **Mapu identificira proizvod, a ne naziv datoteke**, stoga globbing
> za `*radiance*.tif` ne pronalazi ništa; umjesto toga uskladite s direktorijem.

### Snimke senzora svjetla — kalibrirani `.daq` + `.csv`

`process` također obrađuje snimke `.daq` u vašoj ulaznoj mapi i za to **ne** treba nikakve slike: DAQ-U / DAQ-M / DAQ-E koji leti samostalno je kompletan
snimanje, a mapa koja sadrži samo `.daq` datoteke validan je ulaz.

DAQ se može snimati **bez** kalibracije — to je ono što javna
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) snimači
(`record_daq.py`) to čine prema zadanim postavkama: zapisuju sirove brojeve senzora i označavaju datoteku kako bi
Chloros preuzeo tvorničku kalibraciju tog senzora **po serijskom broju** (prvo iz lokalne predmemorije,
zatim iz oblaka MAPIR) i primijenio je. `process` zapisuje rezultat natrag:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` pohranjuje jedan redak po očitanju: UTC oznaku vremena, vrijeme integracije, ukupnu snagu, fotopičke/skotopičke luks jedinice, PPFD (i njegovu podjelu na plavu/zelenu/crvenu komponentu), vršnu valnu duljinu, a zatim
cjelokupni spektar na vlastitoj mreži valnih duljina senzora. `.daq` ponovno uvozi bez
ponovne kalibracije.

U slučaju uspjeha, izvršavanje izvještava `Light-sensor products written: N (calibrated .daq + .csv)`.
Ono u zagradama opisuje što je zapravo napisan, pa se čita
`(RAW COUNTS — this sensor has no calibration bundle)` za senzor bez paketa i
`(N calibrated, M raw counts)` za mapu koja sadrži oboje. Sama pozadinska komponenta
naslovi `[DAQ-EXPORT]` i `[RUN-SUMMARY]` svoju terminologiju izvode na isti način — nijedan od
ta tri ne može nazvati sirovi izvoz kalibriranim.

Zapis DAQ-U / DAQ-M / DAQ-E čija se kalibracijska skupina ne može dohvatitićena — niste povezani na internet ili taj senzor nema spremljenu kalibraciju — **preskače se s razlogom** na
`[DAQ-EXPORT]` linija, nikada ne bude zapisana kao &quot;kalibrirana&quot; datoteka koja sadrži sirove brojeve.
Povežite se s internetom i ponovno pokrenite. Razlog je onaj koji je čitatelj zapravo utvrdio za tu datoteku (nečitljiva shema, nema paketa, greška pri pisanju), a sažetak pokretanja navodi **različite** razloge — dvadeset datoteka preskočeno zbog jednog razloga prikazuje se kao jedan razlog, a ne dvadeset ponavljanja istog.

#### Izvoz snimki DAQ-A kao sirovih brojeva

Obitelj **DAQ-A** prethodila je sustavu paketa po serijskom broju i nema paket kalibracije za preuzimanje — umjesto toga, kalibrira se na terenu u odnosu na metu reflektancije, što je
zašto mu nikada nije ni trebao. Odbijanje tih snimki ostavilo bi ih bez ikakvog načina da dobiju svoje
brojeve, pa ih izvoze pod **drugim imenom**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Drugačiji naziv datoteke umjesto zastavice unutar datoteke, jer tvrdnja mora opstati
pri slanju e-poštom kao samostalni naziv. Zaglavlje `.csv` navodi
`raw spectral sensor counts (NOT irradiance)` i upozorava da su vrijednosti usporedive **unutar** datoteke — što je upravo ono za što ih kalibracija temeljena na cilju koristi — a ne među senzorima. Kolone fotometrije ovisne o snazi (ukupna snaga, fotopski i
scotopični luks, PPFD) zapisuju se kao **NULL** umjesto da se integriraju iz brojeva, a sažetak
pokretanja kaže `RAW COUNTS` pa se &quot;izvezeni&quot; u zapisniku ne mogu čitati kao zračenje.

Naslijeđeno **v1.01 / v1.02** zapisa (koje bilježi DAQ-A-SD) ne sadrži epohu po očitanju,
samo vrijeme pisanja datoteke. Uređaj za usklađivanje slike i doljnjeg svjetla i dalje ih odbacuje — usklađujući
snimka u odnosu na vrijeme zapisa bila bi pogrešna na skriven način — ali izvoznik ih čita, a CSV ispisuje `clock=daq_created_on` pa proizvod navodi na kojem je satu.

### Bilješke

- `process` automatski otkriva je li vaš direktorij Survey3, LATTICE ili mješovit.
- Protok napretka putem Server-Sent Events; CLI prikazuje trenutačni napredak za svaku nit (Otkrivanje, Analiziranje, Obrada, Izvoz).
- Za Linux /Jetson, CLI provjerava zamjenu (swap) i može upozoriti prije obrade velikih mapa. Debayer osjetljiv na teksture također automatski primjenjuje ograničenje frekvencije GPU-a na Jetsonima s niskom potrošnjom energije (Nano, Orin Nano)..
- U slučaju uspjeha, izvršavanje izvještava o broju napisanih slikovnih proizvoda (`Image products written: N`).

#### Izvršavanje koje ne piše slike ne uspijeva

Ako ste zatražili proizvode i izvršavanje je napisalo **none** — samo `project.json` i
`calibration_data.json` — `process` to smatra neuspjehom: ispisuje
`Processing finished but wrote no image products.` i **izlazi s ne-nuljevom vrijednošću**, pa skripta to može
otkriti. Poruka navodi mapu projekta i uobičajene uzroke:

- ulazna mapa nije prepoznata kao snimanje (provjerite raspored i `--input-level`), ili
- svaki zatraženi proizvod je preskočen kao neprimjenjiv za te kamere (npr. traženje
  radijance/reflektancije od kamera koje podržavaju samo RGB.

Ponovno pokrenite s `--verbose` i provjerite zapisnik pozadinskog sustava (backend log) za retke s `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
što objašnjava preskakanja po kamerama koja inače ne dospijevaju do izlaza CLIa.

Namjerno pokretanje samo za metapodatke — sa svim isključenim prekidačima proizvoda i bez `--indices` — i dalje je
**uspjeh**, jer je prazan izlazni prikaz slike u tom slučaju ispravan rezultat.

Tako je i sa **senzorom svjetlosti-samo za metapodatke**: mapa snimki `.daq` po definiciji nema slike za izvoz, a izvođenje se ocjenjuje prema kalibriranom `.daq` / `.csv` koje je umjesto toga zapisalo.

---

## `chloros-cli login`

Provjerite autentičnost ovog uređaja pomoću računa na Chloros+ oblaku. Podaci za provjeru autentičnosti sigurno se pohranjuju u `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Primjeri

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (uklanjanje ili dupliciranje dijelova lozinke). Kod 401 greške, CLI automatski ponovno pokušava s ponovno pridodanim `$$`, a zatim s polovicom lozinke iz koje su uklonjeni duplicirani dijelovi lozinke; ako ponovni pokušaj uspije, prijavljuje vas i ispisuje ispravnu sintaksu s navodnicima koju trebate koristiti sljedeći put.

> **Upotreba bez zaslona/skriptirana upotreba: nema pohranjene sesije znači interaktivni upit, a ne brzi neuspjeh.** Svaka naredba za pokretanje pozadinskog procesa (`process`, `status`, `export-status`, `time-sync`, …) izvođenje bez pohranjene licence/sesije dovodi do interaktivnog upita `Email:` / `Password:` na stdin-u prije nastavka. Nezavisni zadatak bez pohranjene sesije stoga će se zaglaviti čekajući unos — pokrenite `chloros-cli login EMAIL PASSWORD` jednom po računalu prije zakazivanja bezglavog rada.

---

## `chloros-cli logout`

Čisti pohranjenu sesiju i prisiljava na novo prijavljivanje pri sljedećem pozivu.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Prikaže trenutnu razinu licence (Iron/Copper/Bronze/Silver/Gold), autentificiranog korisnika i broj veza uređaja.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Provjeri trenutačni napredak izvoza Thread-4 uživo. Sigurno je pozvati **tijekom** izvođenja `process` iz drugog shell-a.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Postavite jezik prikaza za CLI (podržano 38 jezika, uključujući CJK, RTL i indijske). Automatski se prebacuje na engleski na naslijeđenim konzolama koje ne mogu prikazati pismo.

```
chloros-cli language [LANG_CODE] [--list]
```

### Primjeri

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Naredbe za mapu projekta

Ove naredbe upravljaju zadanoj lokacijom mape projekta (zajednička s GUI-jem).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update` samo za

Linux/ Jetson. Provjerava `version_url` iz `/etc/chloros/update.conf` i nudi preuzimanje + instalaciju odgovarajućeg `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Na Linux/Jetson, CLI također provodi **automatsku provjeru ažuriranja pri svakom pokretanju** (neblokirajuće, nikada ne odgađa naredbu): čita `/etc/chloros/update.conf`, kešira rezultat na 1 sat u `~/.chloros/update_cache.json` i ispisuje `Update available: vX.Y.Z / Run: chloros-cli update` kada postoji novija verzija. Silprikazuje poruku o pogrešci i izlazi na Windows.

---

## `chloros-cli selftest`

Pokreće 7-korak test pušenja: verzija, dostupnost porta, pokretanje pozadinskog sustava, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prisutnost modela za uklanjanje šuma, spremnost CUDA+uklanjanja šuma.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Status i kontrola PTP grandmastera. Host Chloros pokreće PTP grandmastera; LATTICE kamere i DAQ-E jedinice rade kao robovi (slave) za njega za vremenske oznake između uređaja.

| Podzapovijed | Opis |
| --- | --- |
| `status` | Prikaži stanje grandmastera, prioritete BMCA, identitet sata. |
| `peers` | Popis robova viđenih putem Delay_Req (kamere + DAQ-E senzori). |
| `cameras` | Stanje PTP-a po kameri (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Ponovno pokrenite grandmaster proces. |
| `set-priority --priority1 N --priority2 N` | Preusmjeravanje BMCA prioriteta. |

### Primjeri

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Kontrola kamere LATTICE. Svaka podzapovijed prolazi kroz pozadinski sustav Chloros; taj pozadinski sustav upravlja bazenom kamera pa naknadni pozivi CLI ponovno koriste isti otvoreni rukohvat.

### Zajedničke opcije (dijele ih većina podzapovijedi)

| Zastavica | Opis |
| --- | --- |
| `-d, --device N` | Indeks kamere (zadano: 0). |
| `-s, --serial SN` | Specifična serijska brojka; nadjačava `--device`. |
| `--serials SN1,SN2,…` | Serijske brojke odvojene zarezom za rad s više kamera. |
| `--all` | Rad na svakoj otkrivenoj kameri. |
| `--exposure US` | Vrijeme ekspozicije u mikrosekundama. |
| `--gain DB` | Pojačanje u dB. |
| `--pixel-format FMT` | npr. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimenzije slike. |
| `--preset {default,high_quality,high_speed,triggered}` | Primijeni unaprijed definirane postavke. Sve se izvršava neprekidno osim `triggered`, koji priprema kameru za hardverski rub na liniji 2 — ako ta linija nije aktivna, uređaj će čekati zauvijek umjesto da snima. |
| `-o, --output DIR` | Izlazni direktorij (podrazumijevano: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Veličina GVSP paketa. `auto` pokreće ICMP+GVSP probe; `jumbo` = 9000; `standard` = 1500. |

### Radni tijek prve veze kamere LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referenca podzapovijedi

#### Otkrivanje i informacije

| Podzapovijed | Svrha |
| --- | --- |
| `lattice info` | Popis povezanih kamera (proizvođač, model, serijski broj, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analizira host sustav za optimalnu konfiguraciju kamere. `--no-discover` preskače otkrivanje kamere (brža, samo NIC analiza). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Provjera/popravak postavki mrežne kartice (NIC); procjena propusnosti/FPS-a. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Stable-schema pozadinska mrežna sposobnost + preporuka niza (vraća `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` zadržava traženu rezoluciju, ali ograničava ciljani fps — pročitajte `recommended.recommended_target_fps` i proslijedite ga kao povežite cilj; tretirajte ga kao uspjeh, nije pogreška. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analiza što-ako bez otvaranja kamera. **`--n-active` je ukupan broj kamera na mreži, a ne samo broj u ovom nizu**— podignite je kada samostalne kamere istovremeno prenose video, ili mreža proračun se izračunava na temelju potražnje koja ih podcjenjuje (zadano: `len(--models)`). Uvijek ispisuje agregatne linije `Wire budget:` (MB/s potražnje naspram sigurnog gornjeg ograničenja) i `Max cameras:`, te označava `** OVER-SUBSCRIBED**` kada niz nadmaši kapacitet veze — vidi [Model fps i burst niza](#array-fps--burst-model). |
| `lattice gpu` | Prikaži status GPU-a. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Provjerite ili ažurirajte firmware kamere. Lokalna `.fwa` selekcija je zakvačena: datoteka u `firmware/<MODEL_PREFIX>/` koja odgovara buildus `MIN_FIRMWARE_VERSION` se flasha kada je prisutan (samo najnovija verzija kao rezervna opcija), stoga je novija slika dobavljača spremljena na disk neaktivna dok se taj pin ne promijeni — namjerno novije verzije stižu na jedinice putem potpisanog AWS manifesta, što je poželjnije kod novijih verzija. |
| `lattice presets [--apply NAME]` | Popisati ili primijeniti postavke kamere. |
| `lattice status` | Status kamere uživo. |

#### Snimanje

| Podzapovijed | Svrha |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Pojedinačni kadar. **Po zadanom sprema svaki tip izvoza** (`--processing all`); vidi [Razine snimanja izvoza](#capture-export-levels-the-all-default). `--levels` sprema eksplicitni podskup (nadjačava `--processing`); `--force-daq` zapisuje dodijeljeno DAQ očitanje kao `.daq` sidecar čak i pri snimanju samo u sirovom formatu. `--jpeg-quality` = JPEG kvaliteta 1–100 (zadano 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Snimanje na disk dok se ne pritisne Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Prikaz uživo MJPEG-a u pregledniku. `--ae-damping` postavlja prigušivanje automatske ekspozicije (0.4–100). |

#### Podešavanje senzora

| Podzapovijed | Svrha |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Čitaj/piši bilo koji GenICam čvor. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Ekspozicija i AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Pojačanje i automatsko pojačanje. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | ROI senzora i biniranje. |
| `lattice format [--set FMT] [--list]` | Format piksela. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardverski/softverski okidač. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (bez oznaka = jednokratno WB) | WB operacije. RGB / samo za Bayer kamere; no-op (preskočeno) na mono M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB pipeline boje za prikaz. `natural` (zadano) je jeftini završni postupak uživo; `enhanced` dodaje uklanjanje šumova + živost + lokalni kontrast CLAHE za cjelokupan izgled s hub-parity efektom uz otprilike dvostruko veći trošak završne obrade po kadru, što dovodi do niže **užive** stope kadrova — spremljene snimke u svakom slučaju uvijek dobivaju punu završnu obradu. RGB /samo za Bayerove kamere; preskočeno na mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Prikaz zasićenja/kontrasta (RGB-kamere s filtrom). Preskočeno na mono M3M. |
| `lattice filter [--set NAME] [--list]` | Postavite model filtera kamere (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Isprobajte snagu/termičke čvorove; uključite/isključite mirovanje niske snage. |

#### Kalibracija i senzori

| Podzapovijed | Svrha |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibracija pomoću reflektirajućeg cilja. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Ugrađene naredbe za senzor svjetla odozgo. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Primijeniti korekciju vinjete na postojeće slike. |

#### Više kamera (privremeni sastanci)

| Podzapovijed | Svrha |
| --- | --- |
| `lattice multi-info` | Popis svih kamera s sinkronizacijskim ulogama. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Jedan sinkronizirani kadar sa svake kamere. Po zadanome spremaju **sve vrste izvoza**kada je povezana trajna matrica; privremena inačica bez matrice obrađuje se**samo debayeringom** (prvo pokrenite `array-connect` za ostale radnje). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Prijenos sinkroniziranih sličica (privremeno). |
| `lattice multi-test [--count N]` | Test vremenskog usklađivanja GPIO-a. |
| `lattice multi-detect [--line LINE] [--json]` | Automatsko otkrivanje GPIO master/slave žičanja. |

#### Poravnanje

| Podzapovijed | Svrha |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus detektor/ručice za podudaranje `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC ručice `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinacija više okvira `[--averaging mean\|median\|inlier_weighted]`, geometrijska ograničenja `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, prostorno ograničenje `[--roi X0,Y0,X1,Y1] [--mask PATH]` i pojedinačne preusmjerivače po roboti `[--per-cam-override SN:KEY=VALUE]` (ponovljivo) | Izračun profil poravnanja s aktivnih kamera. `--prefilter` zadano je `gradient` (mapa rubova; odgovara GUI/poravnavaču nizova — rubovi se zadržavaju kroz spektralne pojaseve). `--matcher flann` se isplati kod više od ~5000 značajki; `--averaging median` je otporan na jedan loš snimak, `inlier_weighted` ponderira prema broju podudaranja; `--lock-scale` projektira na najbližu rotaciju (bez skale), `--lock-axis` postavlja na nulu jednu komponentu translacije; `--mask` se primjenjuje na svaku kameru (koristite `--per-cam-override` za postavke po kameri, npr. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odbija spremiti kalibraciju čija RMS reprojekcije premašuje prag. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Snimi jedan poravnati višebandni kadar. `--bit-depth` po zadanom podudara kameru; `--no-crop` zadržava cijeli kadar (ispuni crnom bojom); `--interpolation` (po zadanom `linear`) i `--border-mode`/`--border-value` (zadano `constant`/0) kontroliraju CPU warp — GPU put je u svakom slučaju bilinearan. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Prijenos poravnatih višebandnih okvira (isti warp kontroleri kao i kod `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Prikaži detalje profila. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Promijeni redoslijed slojeva. |

#### Indeks / Matematika vegetacije

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Komplet zastavica: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (ponovljivo), `--capture-level raw|debayered|radiance|reflectance|unknown` (nadjačava razinu snimanja zabilježenu u izvoru TIFF; zadano: čitanje iz metapodataka TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Kod `--live` također se primjenjuju gumbi za pomicanje poravnanja: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Simboli**`--channel`**osjetljivi su na velika i mala slova.** Stranica simbola mora točno odgovarati nazivima kanala preseta (presetovi koriste mala slova, npr. NDVI = `red`,`nir` — provjerite `--list-presets`), a strana pojasa mora točno odgovarati nazivu pojasa u poravnatoj hrpi (ili biti indeks pojasa s bazom 0 u izvanmrežnom načinu rada). `--channel red=Red_660 --channel nir=NIR_850` radi; `--channel RED=660` ne uspijeva s greškom `channel_map missing entries`.

#### Trajne veze (Smart-Prep, GUI-ekvivalentni protok)

Ove naredbe drže kamere otvorenima u pozadinskom poolu između poziva CLI.

| Podnaredba | Svrha |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Dodavanje jedne kamere u bazen (jedna kamera, bez niza). |
| `lattice cam-disconnect [--serial SN] [--all]` | Oslobađanje. |
| `lattice cam-list` | Popis kamera u poolu. |
| **`lattice array-connect`**|**Povežite postojani sinkronizirani niz (PREPORUČENA polazna točka).** Pokreće cjelokupni GUI postupak pametne pripreme. |
| `lattice array-disconnect [--array-id ID] [--all]` | Oslobodite niz. |
| `lattice array-list` | Popis povezanih nizova. |
| `lattice array-status [--array-id ID]` | Živi fps, PTP, posljednja pogreška. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno sinkronizirano snimanje iz niza uživo — Pojedinačno / Neprekidno / Interval / Najbrži. **Zadano je `all`** (jedna datoteka po primjenjivoj vrsti izvoza po kameri). Preskočene kamere (npr. RGB) isključene su iz radijance/reflektance) izvještava se pomoću `Skipped: SN:<serial> (<reason>)`; DAQ očitanje koje se koristi za reflektanciju spremljeno je uz njega i izvještava se pomoću `DAQ: <path>`. Vidi [Načini snimanja, snimači i offline reprocesiranje](#načini-prikaza-zapisivači--offline-obradba). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Snimanje uživo kombiniranog indeksa u video/GIF (kvalitete za nadzor; potreban je otvoreni kombinirani tok). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Burst sirovog Bayer formata s visokim fps (analiza-razine; reprocesirati izvan mreže). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocesirati spremljeni raw burst u kalibrirani video(e). |

##### `array-connect` Opcije

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatsko otkrivanje svih LATTICE kamera (potrebno ≥2) | Prvi serijski priključak je MAJSTOR. Ako se izostavi, otkrivanje se filtrira na modele LATTICE (`TRI032*`) i povezuje ih sve. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sinkronizacijska linija. |
| `--target-fps F` | automatski | Stopa ispaljivanja okidača Master. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automatski | Prepiši odabirač sloja. |
| `--wire-ceiling-mbps MB_PER_S` | automatski otkriveno | **Održivi protok hosta, u MB/s — broj na kojem ovisi cjelokupna alokacija niza.** Smanjite ga kada niz prijavi GVSP-pokvarene okvire: automatska vrijednost izvedena je iz oglašene brzine veze mrežne kartice (NIC), koja precjenjuje USB adaptere, uske PCIe trake i zauzete zajedničke mrežne resurse. Pohranjuje se u blok za snimanje niza projekta, pa ga ponovno otvaranje / CLI / ponovno povezivanje SDK vraća. Vidi [Zdravlje niza](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hardversko razdvajanje. |
| `--no-recommend` | isključeno | Preskoči korak analize mreže. |
| `--no-ptp` | isključeno | Onemogući PTP (vremenski pečati među kamerama tada **nisu** usporedivi). |

### Smart-AE / Smart-Capture

LATTICE nizovi provode kontinuirani AE u pozadini čim se povežu, ali svježe usmjerena scena treba trenutak da se konvergira. `array-capture --smart` je **praktično rješenje u paketu**: čeka da se AE stabilizira na svakoj kameri u nizu, a zatim pokreće snimanje. Koristite ga kada mijenjate prizore usred sesije.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Pravilo stabilizacije je konzervativno po zadano: vremensko ograničenje od 5 s, prozor stabilnosti od 1,5 s, tolerancija raspršenosti ekspozicije od ±5 %. Podesite putem parametra SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) ako želite drugačije ponašanje od automatizacije.

### Razine izvoza snimanja (zadano za `all`)

Od ove verzije, `lattice capture`, `lattice multi-capture` i `lattice array-capture` **po zadanom su postavljeni na `--processing all`** — jedna spremljena datoteka po vrsti izvoza koja se primjenjuje na svaku kameru, što odgovara ponašanju GUI-ja &quot;Capture All&quot;. Razine su:

| Razina | Izlaz | Primjenjivo na |
| --- | --- | --- |
| `raw` | Jednokanalni Bayer (monokamera: pojedinačna traka) izravno sa senzora. | Sve kamere. |
| `debayered` | 3-kanalni BGR demosaicing (mono kamere: 1-kanalni sivkasti). | Sve kamere. |
| `radiance` | float32 W/m²/sr/nm putem cijelog radiometrijskog lanca. | Multispektralne (M3C/M3M) samo — **preskočeno za kamere s filtrom RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1.0), spremno za Pix4D. | Samo za multispektralne kamere, i **samo kada je DAQ povezan + kamera kalibrirana**; u suprotnom se preskače. |
| `preview` / `display` | Cijeli lanac pregleda GUI-ja (CCM + WB + gamma prema profilu kamere). `lattice capture` naziva ovaj `preview`; `array-capture`/`multi-capture` koriste `display`. | Sve kamere. |

Prođite kroz jednu razinu da biste spremili samo tu jednu (`--processing debayered`). Kada zatražite `all`, razine koje se ne odnose na zadanu kameru preskaču se (i izvještava se o njima), a ne javlja se pogreška — nevezana ili nekalibrirana kamera i dalje dobiva `raw` / `debayered` / `preview`.

Za svaki kadar reflektancije, stvarno korišteno DAQ downwelling očitanje zapisuje se u pomoćnu datoteku **`.daq`** pored slike (kako bi se snimka mogla ponovno obraditi kasnije) i izvještava se na liniji `DAQ:`.

### Kako izgleda mapa snimki

Svaka vrsta izvoza smješta se u **svoju podmapu** unutar `-o`, tako da višestruka-razina snimanja nikada ne miješa vrste:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` je vremenski žig snimanja, a `<serial>` serijski broj kamere, tako da jedna sinkronizirana grupa dijeli
vremenski žamp across cameras. **Note the one asymmetry:** the `display` level is stored in a folder
named `preview/` while the files themselves keep `_display` in the name — folder and suffix se razlikuju samo za tu razinu. Nezavisne razine vraćaju se u mapu vlastitog naziva, a ako se podmapu ne može stvoriti, datoteka se zapisuje u korijensku mapu izlaza umjesto da se izgubi.

**Ponovna obrada mape snimki:**usmjerite `chloros-cli process` na**korijen snimki**
(`output/`). `process` obično uvozi samo mapu koju nazovete, ali kada ta mapa ne sadrži slike, a ima podmapu, automatski prelazi u nju — tako da se podmapama korijenskog razine i XPROT-u korijenske mapeX000522 se sve učitavaju odjednom. Svaka razina snimanja uvozi se kao jedna slika, a ostale razine dostupne su kao načini rada, a ne kao jedna slika po razini.

Izravno imenovanje **podmappe razine** (npr. `output/raw/`) također radi. Time se izvorna
`.daq` ostavlja po strani, stoga kopirajte ili usmjerite DAQ očitanje uz nju kada ponovno-izvodi radiometrijski proizvod iz `raw/` — inače usklađivanje vremenskog žiga nema na što se odnositi.

**Obrada uvijek započinje s `raw`.** Unutar svake snimke sirovi kadar je izvor cijevi;
`debayered`, `radiance`, `reflectance` i `preview` dolaze kao prikazivi načini rada, ali nikada se ne hrane
natrag kroz pipeline. Ponovna obrada izvedenog proizvoda ponovno bi primijenila vignetu, CCM i radijansku matematiku koja je već ugrađena u njegove piksele, pa Chloros odustaje umjesto da ga dvaput obrađuje. Dvije posljedice koje vrijedi znati:

- `index/` i Renderiranja 32 **nikada** se ne obrađuju. To su izlazi, a ne snimke — renderiranje LUT-a za NDVI nema smislenu interpretaciju radijance.
- Mapa sa snimkama izvezena **bez** `raw` (npr. `array-capture --processing reflectance`) nema
  legitiman izvor u pipelineu. Te se snimke normalno uvoze i prikazuju, ali `process` preskače
  ih i o tome obavještava:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ako zaista trebate provući izvedeni proizvod — hub sesiju snimljenu s
  uključenim `demosaic`, ili naslijeđenu mapu — `--input-level {raw,debayered,processed}` nameće ulaznu
  točku i poništava preskakanje. Ta je zastavica namjenska izlazna rupa; `auto` (po zadanom) nikada ne obrađuje snimku koja nema sirovi materijal.

### Preskočene snimke u nizovima s miješanim filtrima

Kada pomiješate RGB i multispektralne kamere u jednom nizu, `array-capture --processing radiance` (ili `reflectance`) sprema multispektralne kadrove i **preskače** kamere RGB — Bayerova radijanca nema smisla za senzor širokog pojasa. CLI ispisuje svaku spremljenu datoteku (s razinom izvoza), svaki `.daq` koji je napisan, i svaki preskok izričito, tako da broj datoteka nije iznenađujući:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeni razloga preskakanja slijede obrazac `<level>-not-applicable-to-rgb-cam`. Reflektancija također može preskočiti s `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, i s `dls-uncalibrated-band-<nm>` kada se pojas nalazi uglavnom izvan radiometrijski kalibriranog raspona senzora svjetla DAQ-a (~374–974 nm) — među isporučivim SKU-ovima samo F988, čiji je podržani tijek rada rad sa zrcaličnim panelom.

Koristite `--processing debayered` (ili `display`) kako biste uključili svaku kameru bez obzira na vrstu filtra, ili zadani `all` kako biste odjednom dobili svaku primjenjivu razinu po kameri.

---

## Načini snimanja, snimači i reprocesiranje izvan mreže

Svi oni rade na **trajnoj nizu** (prvo pokrenite `array-connect`). Oni odražavaju panel za snimanje GUI-ja.

### Načini rada `array-capture`

`array-capture` je jednodijelna naredba s četiri načina rada zatvarača i skupom prekidača za izvoz:

| Način | Zastavica | Ponašanje |
| --- | --- | --- |
| **Jednokratno** *(zadano)* | (ništa) | Jedna sinkronizirana grupa snimanja, zatim izlaz. |
| **Kontinuirano** | `--continuous` | Uzastopni snimanja dok se ne dostigne `Ctrl+C`, `--count N` ili `--duration S`. |
| **Interval** | `--interval S` | Jedan prolaz svakih `S` sekundi (mjereno od početka svakog prolaza), isti okviri. |
| **Najbrže** | `--fastest` | Samo sirovi podaci + dodana DAQ očitanja + kompozit kombiniranog indeksa; preskače matematiku radijancije/reflektancije/prikaza kako bi se kadar brzo dobio. Podrazumijeva `--processing raw --force-daq`. Ponovno obradite spremljeni `.daq` u kalibrirane proizvode kasnije. |

Prekidači za izvoz (kombinirajte s bilo kojim načinom rada; svi dijele GUI/SDK krajnju točku):

| Zastavica | Učinak |
| --- | --- |
| `--processing LEVEL` | Jedina razina izvoza, ili `all` (zadano). |
| `--levels L1,L2,…` | Eksplicitni podskup tipova izvoza (npr.g. `raw,radiance,reflectance`); **nadjačava `--processing`**. |
| `--aligned` / `--no-aligned` | Warpira svaki ne-sirovi izvoz svakog člana na [profil poravnanja](#alignment) niza (ko-registrirano). Sirovi podaci ostaju neizobličeni, ali nose transformaciju u metapodacima. Vraća se na neporavnato (s upozorenjem) ako niz nema profil. |
| `--index` / `--no-index` | Spremi / preskoči po-preklapanje indeksa vegetacije za -kamere gdje je konfigurirano. Zadano: renderiraj ga. |
| `--force-daq` | Spremi dodijeljeno DAQ/DLS očitanje kao `.daq` sidecar čak i kada ga nijedna odabrana razina ne zahtijeva (npr. samo sirovi snimak), kako bi se kadrovi mogli naknadno obraditi u offline načinu u refleksancu/indeksu. |
| `--smart` | Pričekajte da se AE stabilizira na svim kamerama prije pokretanja (vidi [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresija piksel-piksela (TIFF). `deflate` (podrazumijevano) = bezgubitna zlib L1 + horizontalni prediktor, ~4,1 MB po kadru pune rezolucije; `none` = nekomprimirano, ~5× brže pisanje pri ~6,3 MB po kadru — koristi se za maksimalnu kontinuiranu brzinu kada disk to dopušta. Oba su bez gubitka i čitaju se identično pri uvozu. |

> **TIFF s jednim zapisom + model kontinuirane brzine.**Snimke se zapisuju u**jednom**tifffile prolaz koji prenosi piksel- + XMP + IFD0 Make/Model (mjereno na Mono12 u punoj rezoluciji: 36 ms komprimirano / 6,5 ms nekomprimirano, u usporedbi sa ~148 ms za stari postupak pisanja, a zatim ponovnog pisanja pomoću ExifToola); jedini preostali ExifTool posao (EXIF pod-poliranje IFD-a) radi se u asinkronom pozadinskom radniku, a kadar je dovršen i spreman za uvoz čak i ako se nikada ne pokrene. Imajte na umu da DEFLATE kompresija drži GIL Python-a, pa komprimirana pisanja**ne**paraleliziraju se među pisalnim nitima po kameri — kontinuirano snimanje 8 kamera u punoj rezoluciji brzinom senzora  (~10,4 fps) zahtijeva `--compression none`**i** NVMe-klase disk (~500 MB/s kontinuiranih zapisa). Isti gumb je dostupan kao `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF s kombiniranim indeksom (monitorske kvalitete)

Zapisuje sve što **prikaz kombiniranog indeksa uživo** prikazuje na `.avi` (i opcionalno na `.gif`). Budući da se priključuje na kompozit uživo, kombinirani stream mora biti otvoren (npr. niz se pregledava u GUI-ju) kako bi se zaprimili kadri. Provjerava napredak svakih 2 s i zaustavlja se na `--duration`, `Ctrl+C` ili kada snimač završi samostalno.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--array-id ID` | samo niz | Odredišni niz (izostavite ako je priključen samo jedan). |
| `-o, --output DIR` | `output` | Izlazni direktorij (backend-lokalno). |
| `--fps F` | `10` | Brzina snimanja sličica. |
| `--duration S` | do Ctrl+C | Automatsko zaustavljanje nakon `S` sekundi. |
| `--gif` | isključeno | Također zapisi animirani GIF. |
| `--gif-only` | isključeno | Spremi samo GIF (bez `.avi`). |

### `array-burst` — raw-Bayer serija visokog fps-a (za analizu)

Izravno čita spremnik sinkronizirane skupine petlje snimanja — **nije potreban lanac kalibracije, exiftool, niti prikaz uživo** — stoga radi punom brzinom snimanja kamere. Zapisuje raw okvire + manifest po okviru + jedan `.daq` po jedinstvenom DLS očitavanju pod `<output>/bursts/<base>/`. Ponovno obrađuje izvan mreže (sljedeća naredba), ili navedite `--build` da biste to učinili odmah po zaustavljanju.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--array-id ID` | samo niz | Odredišni niz. |
| `-o, --output DIR` | `output` | Izlazni direktorij (burst se smješta u `<DIR>/bursts/<base>/`). |
| `--duration S` | do Ctrl+C | Automatski- zaustavljanje nakon `S` sekundi. |
| `--max-frames N` | neograničeno | Automatsko zaustavljanje nakon `N` sirovih okvira. |
| `--build` | isključeno | Nakon zaustavljanja, odmah ponovno obradite burst (isto kao `array-build-video`). |
| `--products …` | `combined:index` | S `--build`: koji video(i) će se izraditi (vidi dolje). |
| `--fps F` | `10` | S `--build`: izlazni video fps. |
| `--save-tiffs` | isključeno | S `--build`: također spremite kalibrirane TIFF-ove po kadru. |
| `--gif` | isključeno | S `--build`: također zapisuje animirane GIF-ove. |

### `array-build-video` — offline reprocesira spremljenu seriju snimaka

Vrijeme-usporedava svaki sirovi kadar s najbližim spremljenim očitanjem `.daq` i provodi ga kroz **isti lanac radijance / reflektancije / indeksa kao i uvozni postupak**, rেন্ডerajući jedan ili više videozapisa.

`--products` je lista zarezom odvojenih stavki `kind:level`, gdje je `kind` ∈ `per_cam` | `combined` i `level` ∈ `radiance` | `reflectance` | `index`. A prazan `level` (bez `kind:`) podrazumijevano je `per_cam`. Podrazumijevano je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Zastavica | Podrazumijevano | Opis |
| --- | --- | --- |
| `--burst-dir DIR` | (potrebno) | Put do mape burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` popis, kao gore. |
| `--fps F` | `10` | Izlazni video fps. |
| `--save-tiffs` | isključeno | Također spremite kalibrirane TIFF-ove po kadru uz video(e). |
| `--gif` | isključeno | Također zapisuje animirane GIF-ove. |

> **Odaberite odgovarajući snimač.** `array-record` je *za razinu nadzora* — snima kompozit uživo kao-prikazivan i zahtijeva otvoren stream. `array-burst` → `array-build-video` je *za razinu analize* — bilježi sirove podatke senzora punom brzinom i naknadno rekonstruira kalibrirane videozapise radijance/reflektance/indeksa, bez potrebe za prikazom uživo.

### Mono (M3M) kamere s jednim pojasom

Linija **M3M**je mono sestra modela Bayer**M3C**: jedan uskopojasni interferencijski filtar po kameri (`M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685`), pa senzor isporučuje**jednu pojasnu ljestvicu sive** bez Bayerovog mozaika. Nema što demosaikovati, nema međukanalnog unakrsnog smetanja za razdvajanje i nema balansa bijele za postavljanje — cijeli postupak obrade boja na senzoru jednostavno se ne primjenjuje.

Što to znači na CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detektiraju mono kameru i**preskaču s porukom od jedne linije** umjesto da nameću besmislene postavke. I dalje normalno rade s kamerom RGB/Bayer M3C u istoj sesiji.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** i dalje rade — radijantnost i reflektancija su *po-pojasnim* radiometrijskim kartama i savršeno su definirane za jedan pojas. Mono okviri nose matricu **identiteta** odziva senzora (bez 3×3 odvajanja), pa ravan prolazi kroz kalibracijsku matematiku nepromijenjena.
- **Jedna mono kamera ne može proizvesti indeks vegetacije.**NDVI / NDRE itd. zahtijevaju najmanje dvije trake (npr. Red + NIR). Da biste dobili indeks s mono hardvera, usmjerite**više** M3M kamere na različitim valnim duljinama, poravnajte ih u jedan višebandni sloj i izradite indeks *toga*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

simboli `--channel` moraju **točno** odgovarati nazivima kanala postavki (osjetljivi na velika i mala slova; NDVI su malim slovima `red`),`nir` — vidi `--list-presets`), a ime strane označava pojas u poravnanom nizu (u načinu izvan mreže također se prihvaćaju indeksi pojaseva s bazom 0, npr. `--channel red=0 --channel nir=1`).

Diskriminator kroz cijeli niz je token `M3M` u nizu modela (nikada se ne pojavljuje u nizu `M3C`), prikazan u GUI-ju/SDK kao `is_mono`.

---

## Postavljanje i podešavanje mrežne kartice hosta (LATTICE nizovi)

LATTICE kamere prenose GVSP preko Ethernet adaptera hosta, stoga za nizove s više kamera **vozač**adaptera i**veličina receive-ringa** jednako su važni kao i brzina veze. Pogrešne postavke iskazuju se kao `FRAMES WILL DROP` / `Reduce ROI to enable` greška u odjeljku Postavke niza (i u `lattice network-analysis` /  SDK-a000664), čak i kada su same kamere ispravne.

### USB 10GbE adapteri — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| Stavka | Potrebna vrijednost | Zašto je važno |
| --- | --- | --- |
| **Verzija upravljačkog programa**|**≥ v10.67 (siječanj 2026.)**, INF `rtump64x64sta.inf` | Naslijeđeni upravljački program iz**

2016.**(v10.65, `rtump64x64.inf`) nepravilno upravlja isključivanjem napajanja i uzrokuje bugchecke s**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**pri gašenju/ponovnom pokretanju/prijelazu u stanje mirovanja. Prijelaz se zablokira (~5 minuta čekanja), korisnik prisilno isključuje uređaj, a ponovljena neispravna gašenja**pokvare WMI repozitorij**(PowerShell/alati počinju otkazivati s `Invalid class`) i**zaglavljuju USB sloj** pri sljedećem pokretanju (NIC se neće omogućiti; USB pogoni prestaju biti popisani). Ažurirajte s realtek.com (ili od dobavljača dongla) prije nego što se oslonite na čista ponovna pokretanja. |
| **Prijemni spremnici**— ključna riječ `ReceiveBufferLen` |**256**(maksimalno za upravljački program) | RX prsten mrežne kartice. Zadana vrijednost upravljačkog programa od**32**ostavlja samo ~0,26 MB upotrebljivog prstena — daleko premalo za burst s više kamera — pa kontrolna ploča niza izvještava o `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` i blokovi se povezuju. Na**256**prsten je velik (**~13,5 MB izmjereno na laboratorijskom 10GbE hostu**), što RX pipelineu daje stvarni višak za multi-cam GVSP burstove. (Budući da li se određena konfiguracija zapravo *spaja* odlučuje se dvjema provjerama — **drain-aware**provjerom prihvaćanja i**provjera agregirane prenatrpanosti** — nije sirovi usporedni test bursta i prstena; vidi [Model burstova i fps za niz](#array-fps--burst-model).) |
| **Receive URBs**— ključna riječ `PendingReceives` |**64** (maks.) | blokovi zahtjeva USB-a u letu; povećavaju se uz Receive Buffers za upijanje naleta. |
| **Jumbo Frame** — ključna riječ `*JumboPacket` | **9014** | Potrebno za GVSP pakete od 9000 bajtova (6× manje paketa/okviru nego kod 1500). |

> ⚠️ **Ažuriranje upravljačkog programa mrežne kartice RESETIRA ove napredne postavke na zadane vrijednosti.**Nakon ažuriranja ili zamjene upravljačkog programa adaptera,**ponovno primijenite** `ReceiveBufferLen=256` i `PendingReceives=64`, inače će se panel nizova ponovno vratiti na zadano stanje iako se &quot;ništa nije promijenilo u hardveru&quot;. Ovo je glavni uzrok zbog kojeg se ranije ispravan sustav odjednom odbija povezati.

Primijenite iz **podižnog** PowerShella (zamijenite naziv svog adaptera, npr. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` pokriva USB 10GbE adaptera.** Sada otkriva vrstu adaptera i podešava ispravnu ključnu riječ za primajuće prstenove (receive-ring): `*ReceiveBuffers`→2048 za PCIe mrežne kartice (Intel I219 itd.), ili `ReceiveBufferLen`→256 + `PendingReceives`→64 za Realtek **USB** 10GbE kontroler (koji ne izlaže `*ReceiveBuffers`). Ciljevi se fiksiraju na maksimalnu vrijednost koju svaki upravljački program prijavljuje (`NumericParameterMaxValue`), pa se nikada ne upisuje vrijednost izvan raspona. Pokrenite ga u **podiženom** terminalu; kao i svaka prilagodba temeljena na registru, promjena stupa na snagu nakon ponovnog pokretanja adaptera ili ponovnog pokretanja sustava. Ručne `Set-NetAdapterAdvancedProperty` naredbe navedene gore i dalje su dobra alternativa — one se primjenjuju odmah (ponovno povezuju adapter) bez ponovnog pokretanja.

### Osnove mreže (sve LATTICE poveznice)

- **Adresiranje:** poveznica-lokalni `169.254.0.0/16` (GigE Vision LLA). Host dobiva statičku `169.254.x.x/16`; kamere + DAQ-E sami sebi dodjeljuju IP adrese u istom rasponu. DHCP/gateway nije potreban.
- **Veličina paketa:**preferirajte jumbo (9000), ali dopustite da ga automatsko ispitivanje (auto-probe) pronađe — ono ponovno mjeri pri svakom povezivanju i već provjerava prolazi li iznad ograničenja kamere od 1500 bajtova ICMP-a pomoću GVSP probe, pa će se stići do jumboa gdje god kabel to doista dopušta. Povežite se pomoću `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` samo kada znate bolje od probnog paketa i preferirate po-naredbeno umjesto trajno: pin preskače probu, pa ako put zapravo ne može prenositi 9000**svako** snimanje istekne s `SC_ERR_TIMEOUT -1011` (vidi [Promjenjive okruženja](#environment-variables)).
- **RX ring se skalira s `ReceiveBufferLen`:**pri zadanoj vrijednosti `32` upotrebljivi ring je ~0,26 MB (premalo za bilo koji burst s više kamera); pri maksimalnom `256` je veliko (~13,5 MB izmjereno na laboratorijskom 10GbE hostu), što pruža stvarnu rezervu. Hoće li se konfiguracija povezati zatim se određuje provjerom propusnosti osjetljivom na odvod**i** provjerom agregatnog preopterećenja u nastavku — a ne sirovim usporedbom bursta i prstena.

### Model fps i burst niza

Kako čitati panel Postavke niza (i `lattice analyze-array` / `analyze_array_network` kamere SDK):

- **Burst se zbraja po kameri prema stvarnom formatu piksela svake kamere.**Mono**M3M**kamere streamaju**Mono12 (2 B/px)**;**M3C**Bayer kamere streamaju 8-bitne ili 12-bitne (TRI032S tiho emitira BayerRG12 čak i kada je zatražen BayerRG8). Dakle, puni niz od 4 kamere-res frame is**~12.6 MB if all 8-bit but ~25 MB with three 12-bit mono cams**. The projection resolves each cam&#x27;s format from its model (identitetska predmemorija), tako da burst odgovara onome što žica zapravo prenosi — a ne pretpostavci jedne veličine za BayerRG8.
- **USB Ethernet adapter ograničen je na 200 MB/s bez obzira na naziv.** Tablica učinkovitosti koja pretvara brzinu veze u održivu vrijednost potječe iz PCIe-a; USB NIC oglašava svoju *Ethernet* brzinu veze, ali je ograničen USB sabirnicom i svojim upravljačkim programom. USB 10GbE dongle je ranije postizao ~1063 MB/s &quot;održavana&quot; — broj koji nikada nije bio istražen — a rezultirajuće tempiranje pokvarilo je 6–18 % sličica, iako je i dalje izvještavalo o zdravom ciljanom fps-u. USB-priključeni mrežni adapteri sada su ograničeni na **200 MB/s** kao apsolutna granica (ograničenje je na busu, pa se ne povećava s nazivnom brzinom; USB 1 GbE adapter postiže ~80 MB/ i nije pogođen). `wire_ceiling_source` u zapisu mogućnosti to navodi riječima, a `nic_is_usb` to označava. Prepišite na bilo koji način pomoću `--wire-ceiling-mbps`.
- **Propusnost je svjesna pražnjenja, a ne cijelog paketa u odnosu na prsten.** Jedan istovremeni paket mora se samo uklopiti u *privremeni zaostatak* = `max(0, Σ per-cam arrival − host drain) × emit_window`, a ne cijeli burst. Na brzoj host / sporoj cam tkanini (brzi host / spori cam) (host **PCIe**10G + 4× 1 GbE cams: dolazak ≈ 320 MB/s, odvod ≈ 1063 MB/s) host se isprazni brže nego što kamere napune, zaostatak je ≈ 0, pa puna-resolucijska simultana emisija**prihvaća**iako 25 MB burst premašuje 13,5 MB ring. Postavite iste četiri kamere iza**USB**10GbE adaptera i protok je 200 MB/s, a ne 1063 — dolazak nadmašuje odlazak, a gubitak se očituje kao oštećeni kadrovi, a ne kao niža brzina kadrova. Na hostu s 1 GbE, donja granica DLThr kamera od 31,25 MB/s uzrokuje da dolazak nadmaši odlazak → ispravno**blokova* (za *ovaj* tip bloka, smanjite ROI ili koristite binning ≥ 2). Propusnost je jedna od **dviju** kontrolnih kapija — druga je provjera agregatnog preopterećenja u nastavku.
- **Predviđeni fps je konzervativni gornji prag za serijsko preuzimanje.**Host-ova petlja za hvatanje trenutno serijski vuče predmemoriju svake kamere (~jedno po prozoru emisije po kameri), stoga je ciklus ograničen `max(readout+emit, N × emit)` s emisijom po kameri ograničenom na**poveznice za pristup**(1 GbE ≈ 80 MB/s), a ne vezu za slanje hosta. Za niz od 4 kamere u punoj rezoluciji i 12-bitnoj dubini boje to je**~2,8 fps), što odgovara izmjerenim ~2,7–3,0 fps. Namjerno je **neovisan o ekspoziciji**, pa u slabije osvijetljenim scenama stvarna vrijednost može malo pasti ispod gornje granice kako se ekspozicija produljuje. Serijski preuzimanje je stvarni ograničitelj fps-a; njegova paralelaizacija podigla bi gornju granicu prema brzini pojedinačnog emitiranja.
- **Ukupna prekomjerna pretplata je čvrsta prepreka za uspostavu veze.**Dodjela propusnosti po kameri ima donju granicu od**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), pa kad se donja granica učvrsti, ukupna potražnja (`per_cam × N`) može premašiti**gornju granicu mreže sigurnu od kolizija**(`sustained × sim_emit_factor`). Praktični gornji pragovi pune rezolucije na 1 GbE:**6 kamera pri 1500 MTU, 9 s jumbo paketima**. Ovaj gornji prag ovisi samo o mreži i donjoj granici —**nezavisan je od veličine okvira**, stoga**binning i manji ROI NE pomažu** (oni smanjuju bajtove po *okviru*, a ne bajtove po *sekundi* kojima upravlja GevSCPD); jedina rješenja su manje kamera, jumbo okviri od kraja do kraja ili brži NIC. Simptom bi bio gubitak paketa u GVSP-u, a ne postepeno smanjenje fps-a, pa `analyze-array` postavlja postignute fps vrijednosti na nulu i ispisuje `**OVER-SUBSCRIBED**`, i `array-connect` s fiksnom rezolucijom **odbija se povezati** (inak &#x27;walk-down&#x27; smanjuje broj sličica, što također ne otklanja ovu vrstu blokade). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` spušta odbijanje na glasno upozorenje za rad na klupi — vidi [Varijable okruženja](#environment-variables).

### Zdravlje niza — koji podsustav gubi sličice

`GET /api/camera/array/<array_id>/capability` povezanog niza nosi informaciju uživo
`health` blok, ponovno se procjenjuje u pokretnom **10-sekundnom** prozoru. On razdvaja gubitak okvira
na dva uzroka kojima su potrebna suprotna rješenja, umjesto da izvještava o jednoj &quot;nepotpunoj&quot;
stopi koja ne navodi nijedan od njih:

| Polje | Što to znači | Koji podsustav |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (po serijskoj vezi) | Okvir je **stigao i bio strukturalno neispravan**— gubitak GVSP paketa. |**Mreža**: ograničenje protoka, tempiranje, RX prsten mrežne kartice (NIC), MTU |
| `never_arrived_rate_pct` (po serijskoj vezi) | Okvir **nikada nije stigao**— kamera nije aktivirana ili ništa nije izašlo iz nje. |**Okidač / sinkronizacija**: M8 kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Brzina najgore kamere za svaku. | — |
| `per_cam_rate_pct` | Kombinirana stopa nepotpunosti po kameri (obje uzročnosti zajedno). | — |
| `stable_for_seconds` | Koliko dugo je svaka kamera ostala ispod 0,01 %. | — |

Iznad 5 % pozadinski sustav bilježi liniju `[array-health <id>] WARN` s nazivom podjele — pri
prvom kršenju, pri promjeni pojasa ozbiljnosti, jednom u minuti dok traje, i jednom kada se
otkloni. Pokvarena polovica ispisuje `[gvsp-corrupt <SN>]` pri prvom pojavljivanju po kameri i
razlogu, a zatim zbraja svakih 60 s. Svaka evaluacija i dalje završava u datoteci dnevnika pozadinskog sustava;
brojači se pomiču za svaki buffer bez obzira na ono što se ispisuje.

Isti zapisnik izvještava o broju na koji se cijela alokacija veže:

| Polje | Što znači |
| --- | --- |
| `wire_ceiling_mbps` | Hostkontinuirani budžet za prijenos hosta koji je na snazi, MB/s. |
| `wire_ceiling_source` | Odakle taj broj potječe, riječima — npr. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` ili `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` kada ga postavi `--wire-ceiling-mbps` (ili polje **Wire Budget** u GUI-ju). |
| `nic_is_usb` | `true` za USB Ethernet adapter — vidi ograničenje od 200 MB/s gore. |

**Čitanje:** ne-nula `gvsp_corrupt_rate_pct` s `never_arrived_rate_pct` na 0 znači da su pokretanje i sinkronizacija kabela savršeni i 100 % gubitak je na mrežnom putu — smanjite `--wire-ceiling-mbps` i ponovno se povežite. Suprotan obrazac ukazuje na kabl za sinhronizaciju ili liniju za okidanje.

> **`--target-fps` nije poluga za oštećene okvire.** GevSCPD ritam se piše
> jednom prilikom povezivanja, pa smanjenje brzine okidača mijenja radni ciklus, a ne
> brzinu istovremene emisije paketa. Mjereno smanjenje potražnje za 5× nije donijelo nikakvo poboljšanje;
> smanjenje gornje granice brzine prijenosa s 240 na 200 MB/s dovelo je istu konfiguraciju od 10,4 %
> oštećenih okvira na 0,00 %.

> **Automatsko smanjenje protoka usred prijenosa nije dostupno na firmveru TRI032S.** Sustav u pogonu ne može sam to popraviti; odspojite i ponovno spojite kako bi alat za podešavanje pri spajanju mogao ponovno isplanirati s novim gornjim pragom.

### Simptom → rješenje

| Simptom (Postavke niza / povezivanje / `analyze_array_network`) | Uzrok | Rješenje |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` reset na 32 (obično nakon ažuriranja upravljačkog programa) | Postavite `ReceiveBufferLen`→256, `PendingReceives`→64; ponovno otvorite ploču (ponovno pokrenite pozadinski sustav ako je pohranio staru veličinu prstena) |
| Ponovno pokretanje/isključivanje se zablokira; kasnije `Invalid class` WMI pogreške, mrežna kartica se ne može omogućiti, USB pogoni nedostaju | Stari Realtek USB 2016 10GbE upravljački program → BSOD `0x9F` → prisilna isključenja | Ažurirajte upravljački program adaptera na ≥ v10.67 (2026), a zatim ponovno primijenite gore navedene postavke prstena za primanje |
| Povezivanje uspijeva, ali vraća nativnu rezoluciju | Smart-prep automatski smanjuje kadar da stane na vezu | Nadogradite vezu / prihvatite smanjenje / `--force-tier slip-emit-and-capture` |
| Niz izvještava o zdravom ciljanom fps-u, ali isporučuje samo djelić toga; `health.gvsp_corrupt_rate_pct` ne-nula, `never_arrived_rate_pct` 0 | Procijenjeni budžet žice hosta preko-navodi ono što zapravo održava (tipično na USB Ethernet adapteru, uskoj PCIe traci ili dijeljenoj mreži) | Ponovno povežite s nižim `--wire-ceiling-mbps` i ponovno provjerite blok zdravlja. **Ne** `--target-fps` — GevSCPD tempiranje je fiksirano pri povezivanju |
| Kamere nedostaju iz objavljenih grupa; `health.never_arrived_rate_pct` ne-nula, `gvsp_corrupt_rate_pct` 0 | Put okidača / sinkronizacije — kamere se ne aktiviraju, nije problem mreže | Provjerite M8 sinkronizacijski kabel i `--line`; potvrdite da je svaki član naoružan (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` prekoračeno u `analyze-array`, ili odbijanje povezivanja s zakvačenom rezolucijom (`array over-subscribes the wire`) | Ukupna potražnja po kameri (minimalna brzina 8 MB/s × N kamera) prelazi gornju granicu sigurne brzine pri kolizijama — 6 kamera u punoj rezoluciji na 1 GbE pri1500 MTU, 9 s jumbo paketima | Manje kamera, jumbo paketi od početka do kraja, ili brži NIC. **ROI/binning neće pomoći** (gornja granica je veličina paketa-neovisno). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nadjačava na klupi (prihvaća gubitak paketa) |

---

## `chloros-cli daq`

Naredbe za spektralni senzor. Dvije klase:
- **`pool-*`**— tanki klijenti HTTPa koji upravljaju senzorom putem trajnog pool-a pozadinskog sustava.**Ovo je podržani put i jedini koji je prisutan u isporučenom CLIu.** Pozadinski sustav posjeduje transport, pa GUI, skripte CLI i SDK dijele jedan aktivni rukohvat umjesto da se bore za serijski priključak.
- **Sve ostalo**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — izravan pristup hardveru, dokumentiran u nastavku radi potpunosti. Za njih je potreban `daq` Python paket, koji**nije uključen ni u jedan distribuirani artefakt**: kompajlirana CLI ga isključuje (`scripts/Build-CLI.ps1` postavlja `--nofollow-import-to=daq`, a transporti `pyserial` / `bleak` / `zeroconf` s njim), a ni PyPI paket SDK ga ne sadrži. Pokreću se samo iz izvornog izvlačenja, stoga ih tretirajte kao internu razvojnu stazu MAPIR, a ne kao nešto što biste koristili.
- **`discover` / `list`** obuhvaćaju oboje: to su naredbe izravno na hardveru iz izvornog izvlačenja, ali u isporučenoj izgradnji vraćaju se na `pool-discover` i pozadinski sustav (backend) provodi skeniranje. Stoga skeniranje radi svugdje — što je važno jer je to jedini način da se sazna BLE MAC DAQ-M-a.

> **`chloros-cli daq --help`** (i `-h` / `help`) popisuje podzapovijedi `pool-*` — pomoć se namjerno usmjerava klijentu pool-a kako bi odražavala naredbe koje se doista izvršavaju. Ako pozovete podnaredbu za izravan pristup hardveru na isporučenoj verziji, ona završava s eksplicitnom greškom koja navodi nedostajući paket i upućuje vas natrag na `pool-*`; ništa ne propada tiho. (`discover` / `list` su iznimka — one preusmjeravaju na `pool-discover` i jednostavno rade.)
>
> **Sve što je korisniku potrebno dostupno je putem `pool-*`** — povezivanje, streaming, snimanje kalibriranih `.daq` datoteka i zamjena profila kapa. DAQ se također može upravljati s Python pomoću `chloros_sdk.connect_daq_sensor()`, koji koristi istu zajedničku putanju.

### Radni tijek prvog povezivanja DAQ senzora

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referenca za `pool-*`

| Podzapovijed | Svrha |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otvori senzor u pozadinskom poolu. |
| `daq pool-connect --port PORT` | DAQ-U na određenom serijskom portu. |
| `daq pool-connect --ble` | DAQ-M preko BLE, MAC se automatski skenira. |
| `daq pool-connect --mac MAC` | DAQ-M preko BLE na poznatom MAC-u (podrazumijeva `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E preko Etherea na poznatom hostu. |
| `daq pool-connect --eth` | DAQ-E preko Etherea, host automatski otkriven (mDNS + ARP kao rezervna opcija; radi s praznom ARP predmemorijom na Windows i Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Podesite prozor integracije / stanje AE. |
| `daq pool-connect --no-stream` | Povežite se, ali još ne započinjite prijenos (nastavite s `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcije kapaciteta. Zadano na pozadinskom sustavu je `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skeniraj svaki transport za senzore s kojima bi se mogao povezati, bez povezivanja. **Na taj način pronalazite BLE MAC DAQ-M-a.** `daq discover` / `daq list` automatski usmjeravaju ovdje u isporučenim izgradnjama. Senzori koji su već otvoreni u poolu nisu navedeni — povezani DAQ-M prestaje oglašavati se — stoga upotrijebite `pool-list` za one. |
| `daq pool-list` | Prikaži sve senzore u pozadinskom poolu. |
| `daq pool-disconnect --sensor-id ID [--all]` | Oslobađanje. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N najnovijih spektralnih okvira. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nastavi / pauziraj streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Pokreni / zaustavi .daq snimanje. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamijeni kondenzator-profil korekcije u vrijeme izvođenja. |

### Podzapovijedi za izravni hardver (samo za preuzimanje izvornog koda — ne u isporučenim izgradnjama)

> Navedeno radi potpunosti. Za njih je potreban paket `daq` Python plus `pyserial` / `bleak` / `zeroconf`, od kojih nijedan nije uključen u kompajlirane verzije CLI ili PyPI SDK — izvršavaju se samo iz izvornog izvlačenja iz MAPIR. **Ako koristite objavljenu izgradnju Chloros, umjesto toga upotrijebite gore navedene naredbe `pool-*`**; one pokrivaju odabir povezivanja, streamanja, snimanja i ograničenja.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Otvorite, povežite se s i upravljajte spremljenim projektom Chloros (mapom s `cameras.json` + `sensors.json` + `project.json`). Sve se usmjerava putem pozadinskog sustava (backend) tako da GUI i CLI stvaraju identično stanje hardvera.

### Referenca podzapovijedi

| Podzapovijed | Svrha |
| --- | --- |
| `project open PATH` | Ispiši manifest uređaja projekta (kamere, nizovi, senzori). |
| `project devices PATH [--reconnect]` | Popis ili ponovno pokretanje otkrivanja. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Povezivanje svake spremljene kamere / niz / senzor. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Pojedinačni snimak s nazvane kamere ili niza. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Burst od N sličica s nazvane kamere ili niza (`-n/--count` zadano 5; `-i/--interval` sekundi između sličica, zadano 0). Serijski snimci nizova uklanjaju duplikate ponovno sinkroniziranih grupa (nadzor zastarjelosti) tako da niz s djelomičnim ciklusima možetornati N kopija jednog kadra; ispisuje rezultate po iteraciji. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Prijenos struje na disk putem pozadinskog zadatka. `--poll-interval` = sekunde između `/stats` polova (zadano 2.0). |
| `project sensor read PATH NAME [--json]` | Najnoviji spektralni kadar. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Snimanje .daq. |
| `project run PATH RECIPE.yaml` | Izvršite YAML/JSON recept za snimanje. `--dry-run` provjerava valjanost bez izvršavanja. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Izračunajte poravnanje za niz — pogledajte [tablicu zastavica u nastavku](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Ispiši trenutni profil poravnanja. |
| `project align clear PATH NAME` | Obriši pohranjenu profilu. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Pomaknite transformaciju jednog sluge. |
| `project align export PATH NAME --to FILE` | Spremite profil u JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Učitajte spremljeni profil. |

#### `project align calibrate` Opcije

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda poravnanja. **Ovi se oblici razlikuju od `lattice align-calibrate`**, koji prihvaća skraćene oblike `orb` / `akaze` / `phase`; ove dvije naredbe nisu zamjenjive na ovoj zastavici. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformirajte model da pristaje. |
| `--frames N` | `1` | Sinkronizirane snimke okvira s prosjekom. |
| `--reference SN` | glavni | Serijski broj referentne kamere; svaki drugi član deformiran je na nju. |
| `--max-features N` | `5000` | Gornja granica broja ORB značajki. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;om omjera. |
| `--ransac-threshold-px F` | `3.0` | RANSAC prag inlier-a. |
| `--min-matches N` | `15` | **Vrata kvalitete** — odbacite rješenje s manje od ovog broja inlier podudaranja. |
| `--max-reproj-err-px F` | `4.0` | **Vrata kvalitete** — odbaciti rješenje ako RMS pogreška reprojekcije prelazi ovu vrijednost. |
| `--checkerboard RxC` | — | Geometrija ploče za `--method checkerboard`, npr. `9x6`. |
| `--name PROFILE` | prazno | Naziv profila ugrađen u spremljeni JSON. **Nije naziv niza** — to je pozicijski `NAME`. |

Ta dva kriterija kvalitete razlog su zbog kojeg kalibracija može uspješno riješiti zadatak, a ipak odustati od spremanja: profil koji ne zadovolji bilo koji od njih tiho bi pogrešno registrirao svako kasnije snimanje, stoga se odbijanje spremanja radije odabire nego trajno pohranjivanje.

### Primjeri

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept DSL

`project run RECIPE.yaml` prihvaća YAML ili JSON datoteku koja opisuje niz radnji:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podržane radnje: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Akcija `burst` uzima `name` (obavezno), `count` (zadano 5), `interval` (sekunde, zadano 0), `output`, `format` i `settings` (isti oblik postavki po kameri kao i kod `apply`); nizovi rafala koriste isti nadzor svježeg sinhroniziranog grupe kao i `project burst`.

Pokrenite ga:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Promjenjive okruženja

| Promjenjiva | Učinak |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Prepišite pozadinski sustav URL (zadano `http://127.0.0.1:5000`) — **priznaje samo `lattice`, `project` i `daq pool-*` obitelji naredbi.** Osnovne naredbe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) pin `http://127.0.0.1:<port>` i zanemarite ovu varijablu (IPv4 doslovni zaobilazi Windows `localhost`→`::1` ~2 s-po-zahtjevu kazna), pa uvijek ciljaju lokalni stroj. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` snižava odbijanje veze zbog prekomjerne pretplate na niz (kad agregatna potražnja po kameri premaši sigurnu granicu kabela s `pin_resolution`) na glasno upozorenje i nastavak, prihvaćajući gubitak paketa GVSP-a. Za upotrebu u laboratoriju samo — vidi [Model niza fps i burst](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Postavlja sam CLI; govori pozadinskom sustavu da omogući paralelnu obradu. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` preskače GVSP fallback sondu (samo ICMP rezultati). **Ovo isključuje jumbo, ne samo da utišava zapisnik** — kamera odgovara na DF pingove samo do 1500 na svakoj vezi, pa je ova proba jedino što može otkriti jumbo. Štedi ~1 s po kameri po vezi; košta ~1,45× gornju granicu kabela ako mreža *mogla* imati prenosi jumbo pakete. Upozorenje o prekoračenju veličine (SDK) pojavljuje se kada to postavite. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Pričvrstite veličinu paketa GVSP-a na N bajtova; u potpunosti preskače provjeru. Preporučite po naredbi (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) umjesto trajnog postavljanja: zaključana veličina sprječava prilagodbu mreži, i fiksiranje vrijednosti 9000 na putanji koja ne može prenositi jumbo pakete uzrokuje da **svako** snimanje završi prekidom s `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Prepišite Nuitka direktorij za ekstrakciju jedne datoteke. CLI automatski koristi `/mnt/ssd/tmp` ako je dostupan. |

---

## Kodi izlaza

| Kôd | Značenje |
| --- | --- |
| `0` | Uspješno. |
| `1` | Opća pogreška (većina pogrešaka podzapovijedi). |
| `2` | Pogreška argumenta. |
| `130` | Prekinuto tipkom Ctrl+C. |

---

## Savjeti za rješavanje problema

- **&quot;Potreban je prijavak&quot;** → Pokrenite `chloros-cli login EMAIL PASSWORD` jednom na ovom računalu.
- **&quot;backend nedostupan&quot;** → Pokrenite desktop aplikaciju Chloros, ili izravno pokrenite binarnu datoteku backenda (`chloros-backend`), ili provjerite `CHLOROS_BACKEND_URL` ako je udaljeno.
- **zapovijedi `lattice` ne uspijevaju s porukom &quot;LATTICE camera drivers not found&quot;** → Arena SDK runtime nije instaliran; CLI isporučuje se s `win32api` priloženim na Windows, ali C runtime je dio GUI instalatera.
- **Array connect / Array Settings prikazuje &quot;FRAMES WILL DROP&quot; ili &quot;Reduce ROI to enable&quot;** → Prsten za primanje na mrežnoj kartici hosta je premalen (često se resetira na 32 nakon ažuriranja upravljačkog programa mrežne kartice). Pogledajte [Postavljanje i podešavanje mrežne kartice hosta](#host-nic-setup--tuning-lattice-arrays) — postavite `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Računalo se zamrzne pri ponovnom pokretanju/isključivanju, a zatim WMI `Invalid class` / NIC se ne može omogućiti / USB pogoni nedostaju** → zastarjeli upravljački program USB 10GbE adaptera uzrokuje `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Ažurirajte upravljački program adaptera — pogledajte [Postavljanje i podešavanje mrežne kartice domaćina](#host-nic-setup--tuning-lattice-arrays).
- **Upozorenje o zamjeni swap prostora na Jetsonu** → Dodajte swap prostor podržan datotekama; CLI ispisuje točne `fallocate` / `swapon` naredbe.
- **Nedostaju izravne DAQ naredbe** → Očekivano: isporučeni `chloros-cli` namjerno isključuje paket `daq`, pa je prisutan samo `pool-*` (SDK na PyPI-ju ga također ne sadrži). Koristite `pool-*`, koji upravlja istim senzorom putem backenda, ili `chloros_sdk.connect_daq_sensor()` s Python.

---

## Vidi također

- [Python SDK Referencija](sdk-reference.md) — programski ekvivalent svake CLI naredbe.
- [Vodič za DAQ senzore](../daq/README.md) — ožičenje + kalibracija specifična za senzor.
- Online dokumentacija: `https://mapir.gitbook.io/chloros/cli`
