# Chloros Python SDK Referenca

**Verzija:**

1.2.0**Generirano:**2026-07-29 19:19 ·**Revidirano:** 2026-08-30**Paket:** `chloros-sdk` (PyPI)**Namijenjeno:** Optimizirano za upotrebu s LLM-ovima; čitljivo za ljude.**Opseg:** Sve javne klase, funkcije i pomoćne metode iz `import chloros_sdk`, s primjerima za kopiranje i lijepljenje koji pokrivaju obradu slika, upravljanje jednom kamerom, sinkronizirane nizove, DAQ senzore i automatizaciju projekta.

Ako trebate samo najvažnije informacije, idite na:
- [Instalacija i brzi početak](#installation)
- [Smart-Connect za LATTICE nizove](#smart-connect-for-lattice-cameras)
- [Sjednice o DAQ senzorima](#daq-sensor-sessions)
- [Automacija projekta](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arhitektura u 60 sekundi

SDK je tanki sloj Pythona iznad pozadinskog sustava Chloros (isti Flask poslužitelj koji koriste desktop GUI i CLI). Za automatizaciju uvozite `chloros_sdk` i pozivate metode visoke razine; u pozadini svaki poziv postaje zahtjev HTTP prema lokalnom backendu na portu 5000 — `http://127.0.0.1:5000/api/...` (namjerno ne `localhost`, koji se prvo rješava na `::1` na Windows i košta ~2 s po zahtjevu prema backendu koji podržava samo IPv4). Backend posjeduje bazen hardvera — kamere, DAQ senzori, profili poravnanja, okvir buffere — tako da skripte SDK mogu postojati zajedno s GUI-jem bez borbe za serijske priključke ili propusnost mrežne kartice.

Postoje tri sučelja koja ćete koristiti:

1. **`ChlorosLocal` + slobodne funkcije** (`process_folder`, `process_lattice_capture`) — Cijev za obradu slika. Pokrenite cijelu mapu kroz kalibraciju / debayer / izvoz indeksa iz jednog poziva Python.
2. ****Smart-connect upravljači** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otvara postojanu pozadinsku sesiju za hardver uživo. Isti &quot;smart-prep&quot; protok kao u GUI-ju: mrežni probir, automatski odabir sloja, PTP, AE seeding, konfiguracija GPIO okidača.
3. **`ChlorosProject` / `open_project`** — Učitajte spremljeni projekt (mapu s `cameras.json` + `sensors.json` + `project.json`), povežite sve odjednom i upravljajte snimanjem pomoću imenovanih rukohvata.

Surface 1 i 2 **automatski pokreću lokalni backend** ako on već ne sluša (isti priloženi binarni program koji pokreće GUI/CLI) — tako da prazna skripta radi iz novog shell-a bez da prvo pokrenete backend. Proslijedite `auto_start_backend=False` da to isključite (npr. pri upućivanju na udaljeni backend, koji se nikada ne pokreće). Pogledajte [Automatsko pokretanje backenda](#backend-auto-start). Površina 3 se ponaša drugačije: `open_project()` ne prihvaća `auto_start_backend` parametar, a `connect_all()` nikada ne pokreće backend — on jednom provjerava `http://127.0.0.1:5000` i, ako ništa ne odgovori, tiho prelazi na izravnu (bez pozadinskog sustava) kontrolu uređaja `lattice_sdk`. Samo `proj.process()` i `stream(..., overlays=True)` lijeno konstruiraju `ChlorosLocal()`X (koji se automatski pokreće).

Sva tri zahtijevaju autentifikaciju: pokrenite `chloros-cli login` jednom na računalu ili se prijavite putem desktop GUI-ja. Pozivi na SDK bez važeće sesije izazivaju `ChlorosAuthenticationError`.

Zahtjevi:
- Python 3.7+ (kako je navedeno u paketu; razvijeno/testirano na verziji 3.10)
- Chloros Desktop instaliran lokalno (binarna datoteka pozadinskog sustava isporučuje se unutar instalacijskog programa)
- Aktivno prijavljivanje putem Chloros+. Pristup razini SDK / CLI zahtijeva razinu **Copper**ili višu (Copper / Bronze / Silver / Gold); besplatni**Iron**nivo nema pristup SDK / CLI. Ovo se provodi**na strani poslužitelja**: svaki zahtjev s oznakom SDK / CLI mora imati i aktivnu sesiju i plaćeni plan, inače pozadinski sustav vraća `403` s `error_code: PLAN_UPGRADE_REQUIRED` (prikazano kao `ChlorosLicenseError` od strane `ChlorosLocal`, i kao `ChlorosConnectError` pomoću pomoćnika `connect_*`). Pozivatelj koji je odjavljен dobiva `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) umjesto toga — ta su dva pojma različita jer ponovno pokretanje `chloros-cli login` ispravlja prvi, a ne može ispraviti drugi.
- Podržana je upotreba izvan mreže unutar probnog razdoblja plana: razina se čita iz predmemorije za provjeru na poslužitelju (5 min) ili keš potpisanih licenci vezanih za računalo (30 dana za mjesečne planove, do isteka pretplate za godišnje). Kada taj rok isteče, plan se prebacuje na besplatnu verziju i pristup SDK / CLI prestaje dok računalo ne uspostavi vezu sa serverom. `chloros-cli status` (`GET /api/license-status`) ostaje dostupan na besplatnom nivou kako bi razlog bio vidljiv — to je jedina ruta SDK / CLI izuzeta od ograničenja po nivou.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS ili noviji**, ili Jetson (JetPack 6). Ubuntu 20.04**nije** podržan: ovisnosti `.deb`-a izvedene su iz onoga s čime se backend povezuje, uključujući `libc6 (>= 2.34)`, a focal isporučuje glibc 2.31.

---

## Instalacija

Python SDK je tanki sloj Pythona iznad backenda Chloros. Za sve osim nekoliko radnih tokova koji koriste samo DAQ, potrebno je lokalno instalirati **desktop paket Chloros** (Windows instalator ili Linux `.deb`) — on osigurava binarnu datoteku pozadinskog sustava, runtime okruženje Arena SDK za LATTICE kamere i pakete za kalibraciju.

Najnovije preuzimanja: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Korak 1 — Instalirajte paket platforme Chloros

#### Windows (.exe)

1. Preuzmite `Chloros-Setup-x.y.z.exe` s stranice za preuzimanje.
2. Pokrenite instalacijski program i slijedite čarobnjak. Zadana putanja instalacije je `C:\Program Files\MAPIR\Chloros\`.
3. Pokrenite Chloros barem jednom i prijavite se na svoj račun Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Korak 2 — Instalirajte Python SDK

**Instalacijski program za Chloros dolazi s odgovarajućim kotačem za SDK.** Svaki instalacijski program za Windows i .deb paket za Linux postavlja `chloros_sdk-X.Y.Z-py3-none-any.whl` na disk koji točno odgovara verziji GUI-ja / CLI / pozadinskog sustava. Ne morate pretraživati PyPI kako biste ostali u sinkronizaciji.

#### Windows

Instalator automatski pokreće `pip install` protiv priloženog kotačića koristeći vaš sustav Python (poželjniji je pokretač `py.exe`, po potrebi se vraća na `python -m pip`). Nije potrebna nikakva radnja — `import chloros_sdk` radi u vašem okruženju Python nakon uspješne instalacije. Ako na računalu nije instaliran Python, instalacijski program tiho preskače ovaj korak i GUI + CLI nastavljaju raditi.

#### Linux (.deb)

.deb paket postavlja kotač na `/usr/lib/chloros/sdk/`. `postinst` ispisuje točnu naredbu — PEP 668, distribucije prema zadanim postavkama odbijaju globalno pisanje za pip, stoga ne instaliramo automatski:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za Jetson uređaje koji su fizički odvojeni od mreže, ovo je u potpunosti offline — wheel je već na disku.

#### Javni PyPI

Za hostove koji koriste samo pip (Chloros desktop paket nije instaliran; radne procese s udaljenim backendom ili samo DAQ):

```bash
pip install chloros-sdk
```

PyPI se ažurira u buildovima instalatera za verziju izdanja, tako da objavljeni wheel odgovara najnovijem stabilnom izdanju. Dev buildovi (npr. `1.1.4.dev1`) isporučuju se samo putem priloženog wheel-a instalatera.

#### Verificirajte

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Potrebna je pretplata na Chloros+.** Svi pozivi u SDK zahtijevaju aktivno prijavljivanje na Chloros+. Pokrenite `chloros-cli login user@example.com 'YourPassword'` jednom po računalu; vjerodajnice se pohranjuju u `~/.chloros/`.

### Trebam li paket za stolna računala?

Samo pip paket **nije** dovoljan za većinu tijekova rada. Evo što je potrebno svakoj SDK površini:

| SDK površina | Trebaju li paket za stolna računala? | Zašto |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Da** | Automatski pokreće pozadinski binarni program na `/usr/lib/chloros/chloros-backend` (Linux) ili `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Da**(lokalno)**/ Ne**(daljinski) | Čisti klijenti HTTPa putem pozadinskog sustava. Lokalni pozadinski sustav → potreban paket za stolna računala. Daljinski pozadinski sustav → `backend_url=`**putem tunela** (pogledajte način rada daljinskog pozadinskog sustava — isporučeni pozadinski sustavi vežu se samo na petlju povratne veze). |
| `ChlorosProject` / `open_project` | **Da** | Pokreće spremljene projekte putem pozadinskog sustava. |
| Izravne LATTICE klase (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Da** | Potrebno je nativno izvršno okruženje Arena SDK koje se isporučuje u paketu za stolna računala. `CAMERA_AVAILABLE` je inače `False` pri uvozu. |
| Direktne DAQ klase (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Ne** | Čisti Python preko pyserial/bleak/zeroconf. Okruženje s pip-only hostom može upravljati DAQ-ovima od početka do kraja. |

### Mod udaljenog backenda (pip-only host, putem tunela)

> **Priloženi backend nije dostupan preko LAN-a.** Produkcijske
> izrade vežu se samo na loopback (obje obitelji loopbacka) i strogo odbijaju
> jedini non-loopback način rada (`CHLOROS_CLOUD_MODE`), pa
> `backend_url="http://<lan-ip>:5000"` **ne može raditi s instaliranim
> Chloros** — taj je obrazac ikada radio samo s source/dev
> backendom. Kako biste pokretali backend na drugom računalu, sami proslijedite njegov loopback
> port i usmjerite SDK na tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Hostovi bez zaslona / za CI / robotiku mogu na jednoj mašini s potpunom desktop instalacijom imati &quot;Chloros poslužitelj&quot;, a na svim ostalim `pip install chloros-sdk` — no prijenos između njih je tunel koji je korisnik postavio gore, a ne izravna LAN URL veza.

> **Poznato ograničenje — `ChlorosLocal` nema mogućnost isključivo pip-priključenja.** `ChlorosLocal(backend_url=BACKEND)` trenutno rješava lokalnu binarnu datoteku pozadinskog sustava u svom konstruktoru *prije* nego što provjeri URL i podiže `ChlorosBackendError` (&quot;Chloros backend not found…&quot;) kada nije instaliran desktop paket — čak i uz dosegljiv udaljeni backend. Samo sučelje pametnog povezivanja gore (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus `analyze_array_network` i pomoćnici `list_*` / `discover_*`) radi s hostom namijenjenim samo za pip.

### Radni tijek samo za DAQ (host samo s pip-om)

Ako vam trebaju samo DAQ senzori i ne koristite LATTICE kamere ili obradu slika, pip paket je samostalan:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Nema backenda, nema .deb paketa, nije potrebno Chloros+ prijavljivanje za rad s DAQ-om izravno na hardveru.

---

## Brzi početak

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indeks glavne razine API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Obrada slika — `ChlorosLocal`

Glavna klasa za obradu slika. Pokreće pozadinski proces pri prvom korištenju, stvara / konfigurira projekte, nadzire napredak i vraća sažetke nakon izvođenja.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metode

| Metoda | Opis |
| --- | --- |
| `create_project(project_name, camera=None)` | Stvaranje novog projekta (po želji s predloškom kamere poput `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Uvezi RAW/TIF/JPG/DNG slike **i `.daq` snimke senzora svjetla**. Vraća `count` (slike) i `scan_count` (zapise). Upozorava samo ako mapa ne sadrži ni jedno ni drugo. |
| `export_light_sensor(daq=True, csv=True)` | Zapisati kalibrirane `.daq` + `.csv` za svako svjetlo-senzorsko snimanje na projektu, u `<project>/Light Sensor/`. Vidi [Snimanja senzora svjetla](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Postavite gumbe za obradu. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Pokrenite pipeline. Vraća `{"status": "complete", "async": False}`, plus `summary`ključ  kada ga pozadinski sustav osigura — pogledajte [Sažetak nakon izvođenja i savjeti](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Provjeri stanje pozadinskog sustava. |
| `logout()` | Očisti pohranjene vjerodajnice. |
| `shutdown_backend()` | Zaustavi pozadinsku uslugu (ako je SDK -started). |
| `discover_cameras()` | Otkrij LATTICE kamere **putem ovog primjerkas backend** (`/api/camera/discover`). Vraća popis dictova (`serial`, `model`, `ip`, …) — istog oblika kao GUI/CLI. Prazna lista ako se nijedna ne pronađe ili je pozadinski sustav nedostupan. |
| `camera_capture(output_dir, format="tiff", **settings)` | Snimanje jednog kadra**putem pozadinskog sustava**(automatski pokrenuto ovim upravljačem) kako bi dobio istu pripremu kao GUI/ CLI (podrazumijevano 12-bitno, ponovna upotreba pool-a, ugrađene kalibracijske metapodatke). Riješite cilj pomoću `serial=` ili `device_index=`; proslijedite `exposure`/`gain`/`pixel_format`/`preset` kao `**settings`. Vraća naslijeđeni dict metapodataka (`filepath`).00193/`preset` kao `**settings`. Vraća naslijeđeni metapodaci dict (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Izrađuje preklapane kompozitne pretpregledne okvire iz grupe kamera — lagani MJPEG klijent preko `/api/camera/<serial>/stream-annotated` rute pozadinskog sustava (zebra / mreža / nišan / histogram / peaking / točka iscrtana na strani poslužitelja). `decode=True` vraća BGR nizove; `False` vraća sirove JPEG bajtove. Također dostupan po projektu kao `ChlorosProject.stream(overlays=True)`. |

Koristite kao upravitelj konteksta za zajamčeno čišćenje:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Snimanja senzora svjetla — kalibrirani `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E mogu se snimati **bez** njihovog kalibracijskog paketa. To je ono što javni [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
snimatelji (`record_daq.py`) to čini prema zadanim postavkama: zapisuju sirove brojeve senzora i označavaju datoteku kako bi Chloros preuzeo tvorničku kalibraciju tog senzora **po serijskom broju** — prvo iz lokalne predmemorije, a zatim iz oblaka MAPIR — i primijenio je pri uvozu.

Chloros piše rezultat natrag kao dva proizvoda po snimci, pod
`<project>/Light Sensor/`:

| Proizvod | Što je to |
| --- | --- |
| `<name>_calibrated.daq` | Obnovljiva arhiva — ista shema kao i kod snimanja uživo, sada se deklarira paket koji ju je proizveo. Ponovni uvoz **ne** kalibrira drugu kalibraciju. |
| `<name>_calibrated.csv` | Spektralna iradijanca u W/m²/nm na vlastitoj mreži valnih duljina senzora, jedan redak po očitanju, plus fotometrijske stupce (ukupna snaga, fotopski/skotopski luks, PPFD i njegova podjela na plavo/zeleno/crveno, vršna valna duljina). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Samo senzori bez paketa (DAQ-A).** Sirove spektralne vrijednosti senzora — *ne* zračenje. Pogledajte u nastavku. |

`process()` obavlja ovaj izvoz kao jednu od svojih faza. **Ne** zahtijeva snimke:
svjetlosni senzor koji se leti samostalno predstavlja potpuno valjan tijek rada, a takav projekt po svojoj je prirodi bez ikakvih slika.

**Zapisima DAQ-A izvozi se sirovi broj.** Obitelj DAQ-A prethodi sustavu paketa po serijskom broju i nema paket za preuzimanje — umjesto toga kalibrira se na terenu protiv cilja reflektancije, zbog čega mu on nikada nije bio potreban. Ta snimanja se izvoze pod `_raw` korijenskim imenom, a ne pod `_calibrated`: drugačije ime datoteke umjesto zastavice unutar datoteke, jer tvrdnja mora ostati očuvana i ako se pošalje e-poštom kao samo ime.
Zaglavlje `.csv` navodi `raw spectral sensor counts (NOT irradiance)` i upozorava da su vrijednosti usporedive **unutar** datoteke — upravo ono što koristi kalibracija temeljena na ciljanom objektu.
za koje su — a ne među senzorima. Fotometrijske stupce ovisne o snazi (ukupna snaga, fotopski/skotopski luks, PPFD) vraćaju **NULL** umjesto da budu integrirani iz brojeva.

DAQ-U / DAQ-M / DAQ-E čiji se paket jednostavno nije mogao dohvatiti i dalje se **preskače**, a ne bilježi se u sirovom obliku: u tom slučaju paket postoji i &quot;ponovno povezivanje i ponovna obrada&quot; je stvarni savjet.

Zabilježbe u naslijeđenom formatu **v1.01 / v1.02** (koje zapisuje DAQ-A-SD) ne sadrže epochu po očitanju, već samo vrijeme zapisa datoteke. Uređaj za podudaranje slike i doljnjeg toka i dalje ih odbacuje — podudaranje kadra s vremenom zapisa bilo bi pogrešno i neprimjetno. — ali ih izvoznik čita, a CSV ispisuje `clock=daq_created_on` kako bi proizvod naznačio na kojem se satu temelji.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Snimka čiji se paket kalibracije ne može dohvatiti (izvan mreže ili senzor bez spremljenog paketa kalibracije) prijavljuje se pod `skipped` **s razlogom**. Nikada se ne sprema kao &quot;kalibrirana&quot; datoteka koja sadrži sirove brojeve — povežite se s internetom i ponovno pokrenite, te će se izvoz dovršiti.

### Povratni pozivi napretka

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Sažetak i savjeti nakon izvođenja

Po završetku, `process()` dohvaća `GET /api/processing-summary` i prilaže tijelo kao `result["summary"]`. Dohvat je nastojanje najboljeg napora i nikada ne blokira uspješan povratak — ako sažetak nije dostupan, `process()` se vraća na obični oblik `{"status": "complete", "async": False}`. Svaki unos u `summary["hints"]` — potpune rečenice s predloženim ispravcima, npr. zašto je izvršavanje proizvelo nulti rezultat — također se ponovno emitira kao poruka o grešci (Python) `UserWarning`, tako da se izvršavanja s nultim rezultatom samodijagnosticiraju čak i ako nikada ne pregledate rječnik:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je mašinski čitljiv dio:

| Ključ | Što broji |
| --- | --- |
| `models` | Grupe kamera u izvršavanju. |
| `images_in_groups` | Izvorne slike unutar tih grupa. |
| `targets_found` | Otkriveni ciljevi reflektancije. |
| `images_calibrated` | Slike kalibrirane tijekom pokretanja. |
| `exported_files` | **Datoteke s izlaznim slikama koje je pokretanje zapisalo.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Snimke senzora svjetla, namjerno se računaju odvojeno — potječu iz druge faze i postoje za pokrete bez ikakvih slika, pa bi ih uključivanje učinilo DAQ- samo za trčanje koje izgleda kao da je izvezlo slike. |

Uz njih: `summary["output_dirs"]` (svaka direktorija u koju je pisano),
`summary["light_sensor_export"]`, `summary["stopped"]` (true kada je korisnik prekinuo
trčanje, tako da djelomični brojevi ne izgledaju kao dovršeno trčanje koje je proizvelo premalo), i
`summary["groups"]` (rastavljanje po grupama).

`exported_files` bilježi pipeline **dok ih piše**, a ne skeniranjem s
objekata slike projekta naknadno. Paralelne i GPU strategije grade vlastite objekte
slika (u radnim podprocesima za GPU staze), pa je staro skeniranje izvještavalo
`0 file(s) written` za svaku takvu izvedbu i zatim emitiralo naznaku o nultom izvozu — na pokretanjima gdje je sve radilo. Ako skriptirate prema ovom broju, zdravo paralelno pokretanje sada izvještava o ne-nulovoj vrijednosti.

Izvještaji o preskakanju senzora svjetla navode razlog koji je čitač zapravo utvrdio za svaku datoteku — an
nesчитljiva shema, nedostajući paket, pogreška pri pisanju — **duplikirano**, tako da dvadeset datoteka
preskočenih iz jednog razloga se čita kao jedan uzrok, a ne kao dvadeset ponavljanja istog.

> **`process()` se ne aktivira kada pokretanje ne proizvede nijednu sliku.** Ovo je jedino mjesto u kojem se SDK i CLI namjerno razlikuju: `chloros-cli process` tretira &quot;zahtijevani su proizvodi, nijedan nije napisan&quot; kao neuspjeh i izlazi s ne-nuljevnim rezultatom, dok SDK normalno završava i izvještava o
> stanje putem `summary` / naznaka. Ako bi vaš pipeline trebao stati na praznom pokretanju, provjerite to sami — pregledajte `summary` (ili prebrojite datoteke u mapi projekta) umjesto da se oslanjate na odsutnost iznimke. uobičajeni uzroci su ulazna mapa koja nije prepoznata kao
> snimanje i proizvodi koji su preskočeni kao neprimjenjivi za prisutne kamere (npr. radijansa s kamera koje podržavaju samo
> RGB).

### Pomoćne funkcije

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Podržane vrijednosti

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrijski izlaz (LATTICE višespektralni pipeline)

Razina izvoza LATTICE multispektralne (M3C/M3M) cijevi `process` — `reflectance` (zadano), `radiance`, `sensor-response` ili `all` (svaki primjenjivi način rada po slici) — mapira se na projektov **&quot;Radiometrijski izlaz&quot;** postavka obrade. `configure()` ima namjensku ključnu riječ za to:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Napredni izlazni put — pisanje ključne riječi `"Radiometric output"` projekta putem `custom_settings` — i dalje radi, ali imajte na umu da zamjenjuje cijeli blok postavki (vidi upozorenje u nastavku):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (zadano) dijeli sjaj kamere po **DAQ downwellingu usklađenom s vremenskim žigom**, automatski riješeno iz snimljenog `.daq` (DAQ-U/M/E)**ili nativnog `.csv` DAQ-M** koji se nalazi uz snimku; svaki paket kalibracije po kameri ili DAQ-u koji nedostaje lokalno automatski se preuzima s AWS-a pri prvom korištenju. CLI to izlaže kao prekidače za proizvode po vrsti na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **zamjenjuje** cijeli izračunati blok postavki (po dizajnu zaobilazi ostale ključne riječi i validaciju `configure()`-a). Kada ga koristite, uključite svaku `Project Settings` ključnu riječ koja vam je važna, kao u gornjem primjeru.

---

## Smart-Connect za LATTICE kamere

Trajne pozadinske sesije za hardver uživo. Koriste se ista odredišta kao i u GUI-ju, stoga je ponašanje identično na svim platformama: SDK / CLI / GUI.

### Pojedinačna kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Potpis `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### Metode `CameraSession`

| Metoda | Opis |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Očitavanje GenICam čvorova; vraća `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Pisanje čvorova po prijateljskom nazivu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Snimite **jedan** kadar. Vraća listu od jednog elementa koja sadrži dict-ove metapodataka kadra. (Snimanje serije/višestrukih sličica je uklonjeno — pozovite `capture()` u petlji ako vam je potrebna serija.) |
| `disconnect()` | Oslobađanje iz rezerve. Bez učinka ako smo priključeni na već otvorenu sesiju. |

`capture()` kontrole izvoza (isti model kao i niz + GUI):

- `processing` / `levels` — `processing="all"` sprema svaki primjenjivi tip izvoza; `levels=["raw","radiance"]` sprema samo one (nadjašava `processing`). Ostavite oba polja prazna za zadano ponašanje backenda.
- `force_daq=True` — sprema dodijeljeno DAQ/DLS očitanje kao `.daq` sidecar datoteku čak i pri snimanju samo sirovih podataka, tako da se kadar kasnije može ponovno obraditi u reflektancu/indeks. Neaktivno ako nije povezan DAQ.

### Sinkronizirani niz — `ArraySession` (Smart-Prep)

`connect_array` je **preporučena polazna točka** za postavke s više kamera. U pozadini pokreće cjelokupni GUI tijek pametne pripreme (smart-prep):

1. **Analiza mreže** (`/api/camera/array/recommend`) — pronalazi najveću veličinu okvira koja odgovara sloju sim-emit bez gubitka okvira.
3. **Automatski odabir sloja** — `sim-capture-sim-emit` ako mreža može to podnijeti; inače `sim-capture-ftd-stagger` ili `slip-emit-and-capture`.
3. **Automatsko smanjenje**— tiho smanjuje veličinu okvira / povećava binning kada kabel ne može održati zatraženu rezoluciju.**Ova sigurnosna mreža ne pokriva agregiranu prekomjernu pretplatu**: problem prevelikog broja kamera za kabel ne može se riješiti smanjenjem okvira — pogledajte [Prekomjerna pretplata](#over-subscription-the-per-cam-floor).
4. **PTP omogućen**prema zadanim postavkama — vremenski žigovi između kamera usklađuju se na jednom zajedničkom satu s točnošću od**~1 ms**. Istovremeno izlaganje dolazi od hardverskog okidača M8 (**&lt; 100 µs** između modula), a ne putem PTP-a: PTP usklađuje *vremenske oznake*, a ne ekspozicije.
5. **Automatski odabir formata piksela po kameri** — RGB kamere → `BayerRG8`, multispec → `BayerRG12`.
6. **AE seeding** — snimaju trenutno AE stanje svake kamere kako veza ne bi resetirala ekspoziciju usred snimanja.
7. **Konfiguracija GPIO okidača** — `connect_array` aktivira svaku kameru (`TriggerMode=On`, `TriggerSource=Line2`) kako bi masterpulsa pokreće robove putem M8 kabela. Ovo je korak samo za niz: pojedinačna kamera otvorena pomoću `LatticeCamera` umjesto toga radi neovisno.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Potpis `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

vrijednosti `force_tier`:
- `"sim-capture-sim-emit"` — istovremeno (sve kamere se aktiviraju na istom rubu takta).
- `"sim-capture-ftd-stagger"` — fleksibilno vrijeme- domeni stager (kamere emitiraju u blago pomaknutim vremenima kako bi se paketi serijalizirali na vodi).
- `"slip-emit-and-capture"` — sekvencijalni hvatanje po kameri (bez vremenske sinkronizacije; jedina opcija kada veličina okvira ne odgovara simu).

`wire_ceiling_mbps` nadjačava **trajni proračun mrežne veze hosta** u MB/s — jedini
broj na koji se veže cijelo dodjeljivanje niza. Ostavite ga na `None` kako biste koristili automatski otkrivenu vrijednost. Smanjite ga kada niz prijavi GVSP-pokvarene okvire: automatska vrijednost je izvedena
iz reklamirane brzine veze mrežne kartice, koja precjenjuje USB adaptere, uske PCIe trake i
zauzete zajedničke mrežne komponente — a ta precjena se očituje kao oštećeni okviri, a ne kao
očito spora veza. Vrijednost se zadržava u projektubloku snimanja niza, pa ga ponovno otvaranje ili kasniji `connect_array` vraća kao i svaku drugu postavku niza.
Vidi [Zdravstveno stanje niza](#array-health--which-subsystem-is-losing-frames).

#### Prekomjerno pretplaćivanje (minimalna brzina po kameri)

Sim-emit pacing dodjeljuje svakoj kameri udio u budžetu za sigurnu komunikaciju bez kolizija, s donjom granicom od **8 MB/s po kameri**(`per_cam_floor_bps`). Kad `N × floor` prekorači gornju granicu sigurne komunikacije bez kolizija, niz**nad-pretplaćuje vezu**— način kvara je gubitak paketa u GVSP-u, a ne niža brzina sličica — i ne postoji ispravak za veličinu sličice:**binning i ROI smanjuju bajte po sličici, a ne tempirane bajte po sekundi** koje agregatna provjera uspoređuje. Praktični maksimalni pragovi pune rezolucije na 1 GbE hostu: 6 kamera pri 1500 MTU, 9 s jumbo okvirima (`max_cams_collision_safe` u odgovoru analize izvještava o gornjoj granici za vašu vezu). Rješenja: manje kamera, jumbo okviri od kraja do kraja ili brži mrežni adapter.

- Odgovori `analyze_array_network()` i `/api/camera/array/connect` nose `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` i `per_cam_floor_bps`. Kada `oversubscribed` je istinit, projekcija **postavlja fps polja na nulu** (`achievable_fps_max` / `fps_bright` / `fps_dark`) umjesto da izvještava o zavaravajućoj, sporoj, ali funkcionalnoj stopi.
- `POST /api/camera/array/connect` prihvaća parametar tijela `pin_resolution` (samo za HTTP — nije SDK ključna riječ; `connect_array` ga ne izlaže). Pinning uklanja sigurnosnu mrežu za spuštanje pri binningu, pa je preopterećeni connect s postavljenim `pin_resolution` **strogo odbijen** s greškom koja navodi svako rješenje. Bez piniranja, povezivanje se nastavlja s postupkom smanjenja, ali upozorava da smanjenje ne može očistiti agregat.
- Izlazna opcija za testne uvjete: postavite `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` u okruženje pozadinskog sustava kako biste odbijanje smanjili na glasno upozorenje — svejedno se povežete i prihvaćate gubitak paketa.

#### Stanje niza — koji podsustav gubi okvire

`GET /api/camera/array/<array_id>/capability` prenosi aktivni blok `health` na
povezanom nizu, ponovno procijenjen na pokretnom **10-sekundnom** prozoru. Raspad gubitka okvira
na dva uzroka kojima su potrebna suprotna rješenja, umjesto jedne &quot;nepotpune&quot; stope koja
ne navodi nijedan od njih:

| Polje | Što to znači | Koji podsustav |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (po serijskoj vezi) | Okvir **je stigao i bio je strukturalno neispravan**— gubitak paketa GVSP-a. |**Mreža**: ograničenje propusnosti kabela, tempiranje, RX prsten mrežne kartice (NIC), MTU |
| `never_arrived_rate_pct` (po serijskoj vezi) | Okvir **nikada nije stigao**— kamera nije aktivirana ili ga ništa nije napustilo. |**Okidač / sinkronizacija**: M8 kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Stopa najgore kamere za svaki. | — |
| `per_cam_rate_pct` | Kombinirana stopa nepotpunosti po kameri (oba uzroka zajedno). | — |
| `stable_for_seconds` | Koliko dugo je svaka kamera ostala ispod 0,01 %. | — |

Uz `health`, isto izvješće prikazuje broj na kojem ovisi cijela alokacija:

| Polje | Što znači |
| --- | --- |
| `wire_ceiling_mbps` | Trajna mrežna propusnost hosta, MB/s. |
| `wire_ceiling_source` | Odakle potječe ta brojka, riječima — npr. `USB-capped 200 MB/s (was theoretical 1062; …)` ili `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` kada ga postavi `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` za USB Ethernet adapter. |

Ne postoji SDK omotač za ovu odredišnu točku — pročitajte je izravno:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Čitanje:** ne-nula `gvsp_corrupt_rate_pct` s `never_arrived_rate_pct` na 0 znači
okidanje i sinkronizacija kabela su savršeni i 100 % gubitka je na mrežnom putu — smanjite
`wire_ceiling_mbps` i ponovno se povežite. Suprotan obrazac umjesto toga ukazuje na sinkronizacijski kabel ili
liniju za okidanje.

> **`target_fps` nije poluga za oštećene okvire.** GevSCPD tempiranje se piše jednom pri
> povezivanju, pa smanjenje stope okidača mijenja radni ciklus, a ne
> stopu istovremene emisije paketa. Mjereno smanjenje potražnje za 5× nije donijelo nikakvo poboljšanje, dok
> smanjenje maksimalne brzine s 240 na 200 MB/s dovelo je istu konfiguraciju od 10,4 % oštećenih okvira na
> 0,00 %.

> **Automatsko smanjenje protoka usred prijenosa nije dostupno na firmveru TRI032S.**Aktivni niz ne može sam popraviti ovo; isključite i ponovno spojite kako bi se program za odabir brzine pri povezivanju ponovno planirao prema novom ograničenju.**

**USB Ethernet adapter je ograničen na 200 MB/s** od strane probe bez obzira na njegove specifikacije: tablica učinkovitosti koja pretvara brzinu veze u održivu vrijednost je
PCIe-izvedeni i USB mrežni adapteri oglašavaju svoju Ethernet brzinu veze dok su ograničeni USB sabirnicom i njezinim upravljačkim programom. Ograničenje je apsolutno, a ne frakcijsko — USB 1 GbE adapter postiže ~80 MB/s i na njega to nema utjecaja.

#### `ArraySession` Metode

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Jedna sinkronizirana grupa snimanja. Vraća `CaptureResult` (popis frame dictova + `.skipped`). Kontrole izvoza u nastavku. |
| `capture(..., smart=True)` | **Pametno snimanje** — čeka da se AE stabilizira na svim kamerama, a zatim aktivira. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Najbrži snimak: samo sirovi + dodijeljeno DAQ očitanje (+ slobodni kombinirani indeks). Odražava gumb &quot;Fastest Capture&quot; u GUI-ju. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Pojedinačno / Neprekidno / Intervalno unutar jedne ograničene petlje. Vraća `list[CaptureResult]`.**Zahtijeva `count` i/ili `duration_s`** tako da se prekida (SDK nema Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Pokrenite snimanje prikaza kombiniranog indeksa uživo na video/GIF → `RecorderHandle`. Jedan kompozitni snimač po nizu. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Pokrenite burst sirovog Bayer formata visokog fps-a → `RecorderHandle`. Ponovno obradite izvan mreže s `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Ofline reprocesira spremljeni raw burst u kalibrirani video(e). Blokira dok se ne završi (`wait=True`) i vraća `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Provjeri status posla izrade izvan mreže: `{running, result, error, burst_dir}`. |
| `disconnect()` | Oslobađanje cijelog niza. |

`capture()` kontrole izvoza (isti krajnji cilj koji koriste GUI/CLI):

- `processing` / `levels` — `processing="all"` (ili `levels=["raw","radiance",…]`) sprema svaki primjenjivi tip izvoza po kameri; jedna vrijednost `processing` sprema samo tu razinu.
- `aligned=True` — deformira (warp) ne-sirovi izvoz svakog člana prema [profilu poravnanja] niza(#array-alignment) (ko-registrirano); neobrađeni ostaju neizobličeni, ali nose transformaciju u metapodacima. Vraća se na neporavnato stanje (s upozorenjem prikazanim u `alignment` rezultata) ako niz nema profil.
- `render_index=False` — preskoči preklapanje indeksa vegetacije po kameri; zadano ga prikazuje tamo gdje je konfigurirano.
- `force_daq=True` — spremiti dodijeljeno DAQ/DLS očitanje kao `.daq` sidecar datoteku čak i kada ga nijedna odabrana razina ne zahtijeva.

kompresija TIFF (tipka samo za HTTP): `ArraySession.capture()` ne šalje `compression` ključ, pa se primjenjuje zadana postavka pozadinskog sustava — `POST /api/camera/array/capture` čita parametar tijela `compression`, `"deflate"` prema zadanim postavkama (bezgubitni zlib L1 + horizontalni prediktor, ~4,1 MB po kadru pune rezolucije). `"none"` zapisuje nekomprimirano (~6,3 MB/kadru) s **~5× bržim pisanjem** — oba su bezgubna i pri uvozu se čitaju identično. SDK ne izlaže nikakav kwarg za to; izlaz je `chloros-cli lattice array-capture --compression none` ili sirovi HTTP. DEFLATE također drži GIL Python-a, stoga komprimirano pisanje nije paralelizirano među nitima pisanja po kameri — za održivo snimanje 8 kamera u punoj rezoluciji pri brzini senzora potreban je `compression: "none"`. Detalji: [CLI Referenca → array-capture](cli-reference.md).**Preko-članovsko izvozno prebrisavanje (HTTP-samo):**isto odredište također prihvaća `exclude_serials` (popis — uklanjanje članova iz spremljenog skupa; niz se i dalje aktivira kao jedna sinkronizirana skupina, a isključeni članovi vraćaju se u `excluded`), `serial_levels` (`{serial: [level tokens]}` preusmjeravanja na razini kamere), i `serial_index` (`{serial: bool}` preusmjeravanja indeksnog preklopa na razini kamere). To su GUI-parity parametri tijela i**još nisu **SDK**kwargovi**; članovi koji nedostaju na mapama vraćaju se na `levels` / `render_index` za cijeli niz.

##### Pregledavanje preskočenih kamera — `CaptureResult.skipped`

`ArraySession.capture()` vraća `CaptureResult`, koja je podklasa `list`: iterirajte je, indeksirajte je, `len()` je — svaki postojeći obrazac i dalje radi. Novi kod može provjeriti atribut `.skipped` kako bi vidio koje su kamere isključene i zašto. Najčešći slučaj su kamere sRGB-filtrima u nizu miješanih filtara kada zatražite `processing="radiance"` ili `"reflectance"` — radijansa po Bayerovoj matrici nema smisla za senzor širokog spektra, pa pozadinski sustav preskače te kamere umjesto da proizvodi besmislice.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeni razloga slijede obrazac `<level>-not-applicable-to-rgb-cam` (po jedna stavka po preskočenoj razini, svaka s `level`). Preskakanja specifična za refleksiju su `reflectance-skipped-no-fresh-dls` (nije dostupno novo očitanje silaznog snopa), `reflectance-skipped-bound-daq-unavailable (…)` (vezani DAQ nije dostupan) i `dls-uncalibrated-band-<nm>` — pojas se većinom nalazi izvan radiometrijski kalibriranog raspona senzora svjetla DAQ-a (~374–974 nm), pa se odbacuje apsolutna podjela reflektancije temeljena na DAQ-u i okvir se spušta na razinu senzorskog odziva. Među dostupnim SKU-ovima samo ga F988 aktivira; podržani tijek rada za tu kameru je rad sa panelom za reflektanciju.

`processing` razine:

| Razina | Izlaz |
| --- | --- |
| `"raw"` | Jednokanalski Bayer (monokamere: pojedinačna traka) izravno sa senzora. |
| `"debayered"` *(zadanoSDK)* | 3-kanalni BGR putem bilinearne demosaike (monokamera: 1-kanalni sivotonski). |
| `"radiance"` | float32 W/m²/sr/nm putem cijelog radiometrijskog lanca. Samo za multispektralne kamere — kamere RGB se preskaču. |
| `"reflectance"` | uint16 0..32768 (spremno za Pix4D); zahtijeva uparivanje DAQ-a uživo za apsolutnu referencu. Samo višespektralno. |
| `"display"` | Potpuno usklađivanje lanca s pregledom u GUI-ju (CCM + WB + gamma prema kameriprofila). |
| `"all"` | **Jedna datoteka po primjenjivoj razini** za svaku kameru (odgovara zadanoj postavci GUI-ja &quot;Capture All&quot; / CLI). Vraćeni `CaptureResult` zatim sadrži jednu dict-datoteku po `(cam, level)`, s razinom u svakom dict-u; neprimjenjive razine pojavljuju se u `.skipped`. DAQ očitanje koje se koristi za bilo koji kadar reflektancije spremljeno je kao pomoćna datoteka (sidecar) `.daq`. |

> **Napomena — zadana vrijednost razlikuje se od one u CLI.** `ArraySession.capture()` zadano je `processing="debayered"`; naredba `chloros-cli lattice array-capture` zadano je `processing="all"`. Eksplicitno proslijedite `processing="all"` iz SDK kako biste replicirali višerazinsko spremanje CLI/GUI.

### Načini snimanja i snimači

Stranica niza odražava GUI panel za snimanje: Pojedinačno / Kontinuirano / Intervalno / Najbrži načini rada zatvarača, plus dva snimača (uživo-kompozitni video i sirovi burst → offline reprocesiranje).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je kontinuirana/intervalna petlja SDKa. Budući da ne postoji `Ctrl+C` kojim biste je prekinuli iz skripte,**morate** proslijediti `count` i/ili `duration_s` (zaustavlja se kada se jedna od njih dosegne). `interval_s` mjeri se od početka svakog prolaza (usklađeno s GUI-jem). Preostali kwargs prolaze ravno do `capture()`.
- **`record`** je *za nadzor*: bilježi kompozit kombiniranog indeksa uživo onako kako se prikazuje, stoga kombinirani tok mora biti otvoren kako bi se primali okviri. Jedan snimač kompozita po nizu (izbaci pogrešku ako je jedan već u tijeku).
- **`burst` → `build_video`** je *za analizu*: `burst` zapisuje sirove okvire + manifest za svaki okvir + jedan `.daq` po svakom različitom DLS očitavanju pod `<output>/bursts/<base>/` punom brzinom petlje hvatanja (bez lanca, bez exiftool-a, bez pregleda uživo). `build_video` vrijeme-usporedava svaki kadar s najbližim `.daq` i ponovno pokreće lanac radijance/reflektancije/indeksa uvoznog pipelinea. `products` je popis `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (zadano: kombinirani indeks). `burst().stop()` također automatski pokreće izgradnju kombiniranog indeksa po najboljem naporu, koja se vraća kao `build_job` u rezultatu zaustavljanja.

#### `RecorderHandle`

Vraća ga `ArraySession.record()` i `ArraySession.burst()`. Koristite ga kao upravljača konteksta za automatsko zaustavljanje pri izlasku iz opsega ili ga pokrenite ručno.

| Član | Opis |
| --- | --- |
| `job_id` | ID pozadinskog posla (str). |
| `kind` | `"composite"` (iz `record`) ili `"raw"` (iz `burst`). |
| `start_stats` | Rječnik koji vraća XPROTpoziv X000467. |
| `result` | `None` tijekom izvođenja; dict konačnog rezultata nakon zaustavljanja. |
| `stats(timeout=10.0)` | Statistike posla uživo (napisani kadrovi, ostvareni fps, proteklo vrijeme). |
| `stop(timeout=60.0)` | Zaustavi snimač; vraća i pohranjuje konačni rezultat. Idempotentno (drugi poziv vraća spremljeni rezultat). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Priključivanje na već povezani niz — `attach_array`

Ako je niz već pokrenut (GUI ga je otvorio ili je prethodna sesija SDK pozvala `connect_array`), upotrijebite `attach_array` kako biste dobili rukohvat za njega umjesto ponovnog povezivanja. `connect_array` uvijek vraća pogrešku &quot;Kamera <sn>je već u nizu<id>&quot; u toj situaciji, jer slanje POST-a `/array/connect` za člana-u-poolu nije idempotentno; `attach_array` čita `/api/camera/array/list` i podudara se prema array_id-u ili serijskim brojevima.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Šablona: skripte zaSDKe koje rade u istom okruženju s desktop GUI-jem trebale bi prvo pokušati `attach_array`, a zatim prijeći na `connect_array` ako još nema nijednog niza u skupu.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Važno — izlazak iz context-managera ZAISTA prekida vezu.**`ArraySession.disconnect()` uvijek POST-a `/array/disconnect`; nema attached-not-owned guard kao što postoji za `CameraSession` / `DAQSensorSession`. Ako dijelite instancu s GUI-jem i ne želite očistiti niz pri izlasku iz opsega,**ne koristite blok `with`** — pohranite handle u običnu varijablu i preskočite eksplicitni `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Pomoćnik za analizu mreže

Korisno prije otvaranja niza — provjerava hoće li vaše predložene postavke biti dovoljne:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` je jedan od `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (u suprotnom `error`). `auto_capped_fps` znači da zatražena rezolucija odgovara RX prstenu samo pri ograničenoj brzini okidača — zadržite rezoluciju i proslijedite `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (vidi [Primjer 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Kako čitati projekciju** (isti model kao i na GUI panelu Postavke niza):

- **Burst (`frame_bytes_total`) zbraja se po kameri prema stvarnom formatu piksela svake kamere.**Mono**M3M**kamere streamaju Mono12 (2 B/px) bez obzira na `pixel_format` koji navedete, dakle, okvir pune rezolucije sa četiri kamere iznosi**~25 MB** s tri mono kamere, a ne ~12,6 MB koliko pretpostavka o potpunih 8-bitima daje. Backend određuje format svake kamere prema njezinom modelu.
- **Admittance (`burst_fits_nic_ring`) je svjestan pražnjenja**, a ne cijelog bursta naspram ringa: sim-emit se uklapa kada host prazni RX ring brže nego što ga kamere pune. 10G host + 1 GbE kamere**prihvaća** punu rezoluciju čak i kada burst premaši prsten; 1 GbE host blokira (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` je konzervativni gornji prag serijskog preuzimanja** — `max(readout+emit, N×emit)` s emisijom po kameri ograničenom na 1 GbE kamerijski link, neovisno o ekspoziciji. Npr. ~2,8 fps za niz od 4 kamere pune rezolucije i 12-bitne (odgovara izmjerenim ~2,7–3,0 fps tijekom rada). Cjeloviti model: [CLI Referenca → Model fps i burst niza](cli-reference.md#array-fps--burst-model).
- **Prekomjerno pretplaćivanje (`oversubscribed: true`) znači N × po- broj kamera premašuje siguran gornji prag pri sudaru** — polja fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) čitaju 0, i automatsko smanjenje/binning to ne mogu popraviti (oni smanjuju bajtove po okviru, a ne tempirane bajtove po sekundi). Rješenja su manje kamera, jumbo okviri ili brži NIC; `max_cams_collision_safe` izvještava o gornjoj granici (6 kamera pune rezolucije na 1 GbE pri 1500 MTU, 9 s jumbo okvirima). Odgovor također sadrži `aggregate_demand_bps`, `collision_safe_ceiling_bps` i `per_cam_floor_bps` (8 MB/s). Vidi [Prekomjerno pretplaćivanje](#over-subscription-the-per-cam-floor).

### Otkrivanje i popisivanje

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE nizovi pokreću kontinuirano AE u pozadini čim se povežu, ali svježe usmjerena scena treba trenutak da se stabilizira. **Smart-capture** je praktično rješenje u paketu: provjerava ekspoziciju svake kamere, čeka dok niz ne postane stabilan unutar prozora, a zatim pokreće snimanje. To je ekvivalent GUI-ja: gumb za &quot;pametno&quot; snimanje u desktop aplikaciji poziva istu pozadinsku krajnju točku.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Kada se upravlja putem `ChlorosProject` (sljedeći odjeljak) dobivate više kontrola:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Pravilo za pametnu ekspoziciju (AE) prema zadanim postavkama je konzervativno. Pooštrite `exposure_tolerance_pct` za zahtjevne radiometrijske radove; ublažite za brzo-prijelazima između scena gdje vam je dovoljno &quot;otprilike&quot;.

---

## DAQ sesije senzora

Persistentni pozadinski bazen za spektralne senzore (DAQ-U putem USB-a, DAQ-M putem BLE-a, DAQ-E putem Ethernet-a). Oglasi površinu kamere: pametno otkrivanje, ponovna upotreba bazena, idempotentno povezivanje.

### Pametno otkrivanje (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Redoslijed prioriteta: Ethernet → BLE → USB. Navedite bilo koji jedan eksplicitni parametar za fiksiranje prijenosa.

### Fiksirani prijenos

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession` Metode

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Sažetak unosa iz bazena (stanje streaminga/snimanja, raspon valnih duljina, kalibracijska sha, vrijeme integracije, frame_avg, stanje AE). |
| `latest(n=1, timeout=10.0)` | Vraćanje do N najnovijih spektralnih okvira. |
| `stream_start()` / `stream_stop()` | Nastavak / pauza streaminga (ručka ostaje otvorena). |
| `record_start(output_dir=None, device_name=None)` | Pokrenite snimanje .daq datoteke. Vraća putanju datoteke. Odbija se za DAQ-U/M bez AWS kalibracijskog paketa (DAQ-E je izuzet). |
| `record_stop()` | Zaustavljanje snimanja. Vraća `{path, rows}`. |
| `disconnect()` | Oslobađanje iz bazena. Bez djelovanja za pridružene, ali ne i vlasničke rukohvate. |

> **Cap-profili korekcije kape (`cap_id`) nisu kotačić za podešavanje kape.** `connect_daq_sensor()` / `DAQSensorSession` ne izlažu parametar `cap_id` ili `set_cap` metodu. Odaberite profil korekcije snage flote putem CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) ili `/api/daq` HTTP rute pozadinske usluge (`/api/daq/connect` i `/api/daq/<id>/cap-id` prihvaćaju `cap_id`).

### Otkrivanje — pronalaženje adrese za povezivanje

`discover_daq_sensors()` skenira USB / BLE / ETH u potrazi za senzorima koje *možete* otvoriti. To je DAQ pandan `discover_lattice_cameras()` i jedini je način za dobivanje **DAQ-M&#x27;s BLE MAC** — DAQ-E ima naziv hosta, a DAQ-U COM priključak, ali MAC nije isprintan na uređaju niti je naveden u OS-u.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Polje | Opis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM priključak / BLE MAC / naziv hosta — proslijedite na `connect_daq_sensor` kao `port=` / `mac=` / `eth_host=`. |
| `display` | Čitljiv naziv. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, ili `None` za priključak koji skeniranje ne može identificirati (USB serijski adapteri su neprepoznatljivi bez sonde, stoga se nepoznate vrijednosti prikazuju umjesto da se skrivaju). |
| `extra` | Detalji po transportu (oglašeno ime BLE-a, proizvođač USB-a, DAQ-E ip/fw/…). Prazne vrijednosti se izostavljaju. |

| Parametar | Zadano | Opis |
| --- | --- | --- |
| `transports` | sva tri | Sekvenca (ili CSV niz) koja ograničava skeniranje. Vrijedi navesti kada znate što želite — BLE je sporiji dio. |
| `scan_timeout` | 5 | Prozir prozor po prijenosu u sekundama; pozadinski sustav ograničava na 1–20. |
| `timeout` | 60.0 | Gornja granica za HTTP za cijeli poziv (kao i drugdje u SDK). |
| `auto_start_backend` | `True` | Pokreni lokalni pozadinski program ako nijedan ne radi. Nikada se ne pokreće za udaljeni `backend_url`. |

> **Senzori koji su već otvoreni u poolu se ne prikazuju.** Spojeni BLE periferijski uređaj prestaje s oglašavanjem, a otvoreni COM priključak se ne može ispitati, stoga otkrivanje prikazuje ono što je *dostupno za povezivanje*. Očekuje se prazan rezultat odmah nakon što nešto povežete — koristite `list_daq_sensors()` za ono što već držite. Transporti čije skeniranje ne može pokrenuti (nije instaliran bleak / zeroconf) preskaču se umjesto da se javlja pogreška, tako da računalo bez Bluetootha i dalje dobiva svoje USB i ETH odgovore.

### Popis

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Dijeljenje s GUI-jem / CLI

Ako GUI već ima otvoren senzor, pozivanje `connect_daq_sensor(port="COM3")` iz Python vraća handle označen kao `already_connected=True`. `disconnect()` sesije je tada no-opcija, tako da vaš skript SDK ne istrgne senzor ispod GUI-ja pri izlasku iz opsega.

### Klasa izravnog hardvera (bez pozadinskog sustava)

`daq_sdk` je ponovno-izvozi `chloros_sdk` pa možete upravljati senzorima od početka do kraja unutar procesa bez pozadinskog sustava:

> **Dostupnost:**`daq_sdk` dolazi s desktop instalacijom Chloros,**ne** s PyPI paketom — `pip install chloros-sdk` vam daje `lattice_sdk`, ali ostavlja `chloros_sdk.DAQ_AVAILABLE == False`. Provjerite tu zastavicu prije korištenja ovih klasa; na hostu koji koristi samo pip, upravljajte senzorom putem [`connect_daq_sensor()`](#daq-sensor-sessions), koji ne zahtijeva lokalne transportne biblioteke.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Radije odaberite put pametnog povezivanja (`connect_daq_sensor`) kada želite zajedničko vlasništvo s GUI-jem; koristite izravne klase za skripte bez zaslona koje isključivo posjeduju senzor.

---

## Automatizacija projekta — `ChlorosProject`

Spremljen projekt Chloros je mapa koja sadrži `cameras.json` + `sensors.json` + `project.json`. `open_project` učitava manifest, a `connect_all` povezuje svaki spremljeni uređaj s mrežom sa svojim spremljenim postavkama — u istom hardverskom stanju koje bi GUI proizveo.

### Minimalni primjer

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Ili kao upravitelj konteksta:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metode `ChlorosProject`

| Metoda | Opis |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Pronađi i poveži svaki spremljeni uređaj. Vraća izvješće o povezivanju po klasi. Koristi pokrenuti pozadinski proces kada on sluša na `127.0.0.1:5000`; u suprotnom se tiho vraća na izravno (pozadinski-free) `lattice_sdk` upravljanje uređajima — nikada ne pokreće pozadinski proces. |
| `disconnect_all()` | Raskidanje svega. |
| `capture_all(output_dir=".")` | Jedan kadar sa svake kamere + niz + spektar sa svakog senzora. |
| `stream(camera, overlays=False, fps=10.0)` | Generator koji isporučuje BGR `numpy` kadrove iz nazvane kamere (ili niz). `overlays=False` je izravna `lattice_sdk` petlja za hvatanje (nizovi vraćaju `{serial: frame}` dictove). `overlays=True` prosljeđuje kroz `ChlorosLocal.camera_stream()` → backends `/api/camera/<serial>/stream-annotated` MJPEG feed, s spremljenim `ui.overlay` blokom kamere proslijeđenim kao parametri upita. Zahtijeva način rada pozadinske instance i **samostalnu kameru**: kamera u izravnom načinu rada pokreće `RuntimeError` (pozadinska instanca ne može preuzeti kameru koju ovaj proces posjeduje) i niz pokreće `NotImplementedError` (nadograđuje kompozit po kameri — prenosi član po imenu). Ekvivalent za jednokratnu upotrebu: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Pokrenite poravnanje na svakom trenutno priključenom nizu. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Pokrenite pipeline kalibracije / indeksiranja na slikama projekta (obuhvaća `ChlorosLocal.process`; ova četiri su **jedini** prihvaćeni kwargs — `indices=` itd. pokreće `TypeError`; postavlja indekse putem `ChlorosLocal.configure()`). Lijenim načinom konstruira `ChlorosLocal()`, koji se automatski pokreće backend. |

Atributi:
- `proj.cameras` — `Dict[str, CameraHandle]` ključiran po nazivu I serijskom broju.
- `proj.arrays` — `Dict[str, ArrayHandle]` ključiran po nazivu I array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` ključiran po nazivu I slot_id.
- `proj.config` — `project.json["config"]` dict.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Razine obrade.** `capture()`, `grab()` i `frame_stream()` svi uzimaju isti `processing`
token, i lanac je kumulativan — svaka razina izvršava sve što je iznad nje:

| Razina | Izlaz | Napomene |
| --- | --- | --- |
| `raw` | 1-kanalni Bayer, nativni za senzor | Bez demosaika. Preklapanja nisu dostupna na ovoj razini. |
| `debayered` | 3-kanalni BGR (**podrazumijevano**) | Bilinearna dimozaika. Jedina razina koja radi bez načina rada backend. |
| `radiance` | float32, W/m²/sr/nm | Potpuni radiometrijski lanac: demosaicizacija + 3×3 odvajanje mješavine (multispectralno) + DSNU + kalibracija po ravnini (flat-field) + NIST ljestvica, s izračunatim omjerom ekspozicije × pojačanja kako bi vrijednosti bile apsolutne. |
| `reflectance` | uint16, 32768 = 1.0 | Radijansa podijeljena s dolaznom zračenjem (ρ = π·L/E). Potrebno je očitanje DLS/DAQ — pogledajte napomenu u nastavku. |
| `display` | 8-bitni sRGB-olik | Renderiranje ekvivalentno GUI-ju: CCM + balans bijele + gama putem aktivnog profila boja kamere. |

Sve osim `debayered` zahtijeva način pozadinskog procesa; kamera u izravnom načinu rada podiže
`NotImplementedError`. `reflectance` treba upotrebljivu mjernu vrijednost dolaznog zračenja — krajnja točka okvira povlači
DAQ se automatski uključuje u DLS utor kamere, ali ako DAQ nije vezan, lanac odbacuje
izlaz reflektancije i iskreno zapečati degradaciju u vraćenim metapodacima umjesto da šutke vrati slabiji proizvod.

> **Skala DN reflektancije — ne kodirajte je fiksno.** LATTICE reflektancija koristi `32768` = ρ 1.0 i zapečati
> XMP `Chloros:PixelScale=32768`; Survey3 reflectance koristi `65535` = ρ 1.0 i ne sadrži oznake `Chloros:*`. Pročitajte oznaku i podijelite s njom. Definirana je u domeni uint16, stoga ostaje `32768` za svaki format koji se ponovno skalira (16-bitni TIFF, 8-bitni PNG/JPG, 32-bitni postotak) — prvo normalizirajte
> pohranjenu dtype natrag na uint16 (×257 iz 8-bit, ×65535 iz float). Jedini izuzetak: snimanje iz 8-bitnog izvora zapisano kao 8-bitni TIFF *se obrezuje*, a ne skalira, pa ga nijedna skala ne opisuje — Chloros u tom slučaju u potpunosti izostavlja `PixelScale` i MicaSense tupel. Nedostajući tag na LATTICE datoteci s reflektancijom tretirajte kao &quot;nema valjane skale&quot;, a ne kao zadanu vrijednost.

> **EXIF se prenosi do izvoza.** `process()` kopira GPS blok izvornog snimanja**i njegov ExifIFD** na svaki proizvod, tako da izvozi sadrže `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` i `CameraSerialNumber`, kao i georeferenciranje. `FocalLength` je ono pomoću čega Pix4D određuje razmak uzorkovanja tla — bez njega rekonstrukcija pada na potpuno pogrešnu mjerilu (u jednom izmjerenom slučaju lokacija od 411 m pretvorena je u onu od 47,8 km). Kopija namjerno nije `-all:all`: IFD0-ovi strukturni tagovi prekidaju LATTICE izlaz, i `ExifImageWidth`/`Height` su isključeni jer opisuju snimanje izvora, a ne izvezeni raster.

Pod-oznake faze snimanja (primjenjuju se na radiometrijske razine — `radiance`, `reflectance`, `display`):

| Zastavica | Zadano | Značenje |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST radiometrijska skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-svestan kada je DAQ povezan s kamerom. |
| `apply_index` | `False` | Procjena indeksa vegetacije. |
| `index_expression` | `None` | Preusmjeravajuća formula. Neprazno → automatski- omogućuje indeks. |
| `annotated` | `False` | Preklapa GUI dekoracije (zebra/mreža/peaking). Nije dostupno za `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Tip povrata je `CapturePathMap`, a ne `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: jednorazinski
> `processing` daje svakom serijskom jedan put, dok višerazinski (`"all"`, ili eksplicitna lista `levels`) daje mu **uređeni popis** svih proizvoda spremljenih za tu
> kameru. Živi kombinirani kompozit, ako se vršilo streaming, dolazi pod dodatnim
> ključ `"combined"`, a ne pod serijskim brojem. Kod koji pretpostavlja `str` prekida se na obliku popisa bez da se ikakav provjeritelj tipova usprotivi — anotacija je navodila `Dict[str, str]` neko vrijeme nakon što je oblik popisa pušten, zbog čega alias i postoji. Normalizirajte
> kada želite ravni oblik:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Poravnanje nizova

`ArrayHandle` izlaže cijelu površinu poravnanja. Profili su prema zadanim postavkama samo za sesiju — pozovite `export_alignment()` izričito za trajno pohranjivanje.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Poravnanje pri uspostavljanju veze

`connect_all(align=...)` može automatski poravnati svaki niz pri uspostavljanju veze:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Prelazi na `project.json["config"]["auto_align_on_connect"]` kada nije navedeno.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Izravni hardver (bez backenda)

Kada želite potpunu neovisnost o backendu (CI, headless roboti, ugrađeni sustavi), uvezite `lattice_sdk` i `daq_sdk` izravno — oba izvozi `chloros_sdk`. Guard na `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` je u PyPI paketu (ali je potreban Arena SDK runtime), dok `daq_sdk` dolazi samo s desktop instalacijom.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Presetne postavke i okidač

Tri od četiri presetne postavke **rade u slobodnom načinu rada**: kamera neprekidno ekspozira i
`capture()` vraća sljedeći kadar. `triggered` je iznimka — priprema kameru za hardverski rub na Liniji 2, stoga ne snima ništa dok on ne nastupi.

| Preset | Okidač | Koristi se kada |
| --- | --- | --- |
| `default` | kontinuirani rad | za opću upotrebu |
| `high_speed` | kontinuirano snimanje | 8-bitno, ograničenje na 60 fps, kratka ekspozicija |
| `high_quality` | kontinuirano snimanje | 12-bitno, bez ograničenja fps — uobičajen izbor za fotografije |
| `triggered` | **naoružano, Linija 2** | kamera je priključena na M8 sinkronizacijski kabel i nešto drugo je aktivira |

Ako odaberete `triggered` (ili postavite `trigger_mode="On"` sami) bez ikakvog pokretača Linije 2, svaki `capture()` će isteći — ispravno, budući da ste zatražili od kamere da čeka. SDK to objašnjava kada se dogodi; vidi
[SC_ERR_TIMEOUT tijekom snimanja](#direct-hardware-backend-free).

> **Napomena —**poruke &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` pri uspostavljanju veze nisu pogreške.**&gt; Prilikom uspostavljanja veze, SDK pokušava pregovarati o**jumbo paketima** (GVSP paketi od 9000 bajtova) radi veće propusnosti. Na izravnoj točka-točka mrežnoj vezi (npr. link-lokalna adresa `169.254.x.x`) mreža obično ne može prenositi jumbo pakete, pa ova proba istekne i zabilježe se retci poput sljedećih:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Ovo je **namjenski povratni postupak**: SDK se automatski vraća na standardne pakete od 1500 bajtova i kamera nastavlja s povezivanjem normalno (redak `[chunk-enable …]` koji slijedi dio je normalnog postupka povezivanja). Snimanje i dalje radi.
>
> Možete preskočiti ovu provjeru, ali **ona nije samo suzdrživač zapisa — ona isključuje jumbo pakete.** Kamera odgovara na pingove &quot;Don&#x27;t-Fragment&quot; samo do 1500 bajtova, bez obzira na to koliko je vaša mreža dobra, pa sama provjera pingom nikada ne može otkriti jumbo pakete; ova proba je jedino što to može. Onemogućite je i kamera će zauvijek koristiti standardne pakete od 1500 bajtova, na bilo kojoj mreži:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Isplati se samo na mreži za koju *znate* da ne može prenositi jumbo pakete, gdje štedi otprilike jednu sekundu vremena povezivanja po kameri. Budući da je to stvarna, a ne kozmetička promjena, SDK to sada i navodi kada ga koristite:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Ostavite ga na miru osim ako nemate razloga.** Ako je ostavljeno omogućeno, svako povezivanje ponovno mjeri mrežu koju zapravo imate: priključite se na switch koji podržava jumbo i sljedeće povezivanje automatski će omogućiti jumbo, bez potrebe za konfiguracijom ili ponovnim pokretanjem.
>
> Ako *želite* postići maksimalnu propusnost za jumbo, omogućite jumbo od kraja do kraja (NIC MTU 9000 + switch koji ih propušta), ili ga fiksirajte pomoću XPR-aOTX000687 kada znate da veza to podržava — iako je poželjnije koristiti `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` po nalogu umjesto trajnog postavljanja, jer fiksirana veličina preskače probu i zaustavlja prilagodbu mreži ispred sebe. **Svaki uređaj na putu mora podržavati jumbo pakete — uključujući sve PoE razdjelnike ili injektore, što je uobičajen razlog zašto inače konfiguracija sposobna za jumbo pakete ne može prenositi te pakete.

> **`SC_ERR_TIMEOUT -1011` tijekom `capture()` / `grab*()` je drugačiji problem — to je stvarna pogreška.
> Gornja napomena odnosi se samo na `-1011` zabilježen od strane **probe pri uspostavljanju veze**. Ista pogreška zabilježena u**capture**-u znači da se kamera uspješno povezala, ali ne šalje slike:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Prepoznatljiv znak je kamera čiji je *kontrolni* kanal ispravan — otkrivanje radi, postavke i zapisi `[chunk-enable …]` uspijevaju — dok *svaki* kadar dospijeva na vrijeme isteka.
>
> **Uobičajeni uzrok je da je kamera spremna za hardverski okidač.** Uz `trigger_mode="On"` i `trigger_source="Line2"`, kamera ne šalje apsolutno ništa dok na M8 sinkronizacijskom kabelu ne stigne električni impuls. Ako nemate kabel koji vodi tu liniju, svako snimanje čeka zauvijek. Kamera nije pokvarena i mreža radi ispravno — ona radi točno ono što joj je rečeno.
>
> `CameraSettings()` i `default` / `high_speed` / `high_quality` postavlja slobodni rad, a snimanje (grab) koje istekne vrijeme dok je naoružano objašnjava se umjesto da ispisuje samo `-1011`. `PRESETS["triggered"]` po zadanom naoružava Liniju2.
>
> Da biste prisilili bilo koju kameru na neprekidno snimanje:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ako i dalje dođe do isteka vremena s `trigger_mode="Off"`, kamera zaista nt delivering data — send us the log and `ip link show`.

#### Profili boja (RGB prikaz uživo) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` odabire profil boja zaslona za **uživo prikaz** na kamerama RGB (multispec kamere zanemaruju postavku):

| Profil | Značenje |
| --- | --- |
| `raw` | Potpuno zaobiđite radiometrijski lanac. |
| `linear` | DSNU + ravna + WB, bez CCM-a, bez gammatona. |
| `natural` | Linearna + izmjerena CCM + sRGB gamma, samo s jeftinim završnim obradom (izravnavanje kroma + desaturacija visokih tonova) — realistična zadana postavka. |
| `enhanced` | `natural` plus potpuni hub-parity finiš (uklanjanje šarenih rubova, vibrancija, lokalni kontrast CLAHE). Bogatiji izgled uz otprilike **dvostruko veći trošak finiša po kadru**, pa niža LIVE brzina sličica u sekundi. |
| `custom_temp` | `natural` ali WB zakvačen na `custom_cct_k` Kelvin (DLS se zanemaruje; zaključano na 2000–10000 K na strani pozadine). |

Profil je **samo s pregledom uživo** kontrola brzine/izgleda: spremljeni snimci uvijek dobivaju potpuno bogat završni izgled bez obzira na odabrani profil, stoga odabir `natural` za povrat vremena sličica ne smanjuje kvalitetu onoga što se sprema na disk. Nezaposleni profil podiže `ValueError`; kada klor backend je dostupan, promjena se također POST-a na njega tako da sljedeći pretpregledni kadar to odražava (korisnici izravnog SDK-a bez backenda i dalje dobivaju mutaciju postavki).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kamere i `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je jednopojasna: jedna ravan sive ljestvice, bez Bayerovog mozaika, bez 3×3 spektralne matrice za unakrsnu interferenciju. `Calibration` to prepoznaje i izlaže `is_mono` zastavica. Reflektancija se i dalje primjenjuje kao radiometrijska mapa po pojasu (nepomiješavanje je identična matrica), ali matematika s više pojaseva na jednoj kameri stvara smisao umjesto da vraća besmislice:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Za izgradnju indeksa vegetacije iz mono hardvera, kombinirajte nekoliko M3M kamera na različitim valnim duljinama u usklađeni multi-pojasni stack (vidi [Poravnanje nizova](#array-alignment)) i izračunajte indeks preko tog stacka umjesto na jednoj kameri.

DAQ izravni način rada:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Prihvaćeni ključevi za `apply_sensor_settings`**— točno `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; zastarjelo u korist `cap_id`), `filter_model` (DAQ-M) i `cap_id` (sve vrste DAQ-a; `None`/`""`/`"none"` = gol senzor, bez korekcije kapa). Nezavisane tipke se**tiho zanemaruju** — npr. `{"integration_time": 64}` ne radi ništa (mora biti `integration_time_ms`). Vraća `{"applied": [...], "errors": {...}}` i nikada ne podiže iznimku.

`chloros_sdk` ponovno-izvozi samo osnovnu površinu korištenu iznad. Potpuna javna API (22 imena) klase `daq_sdk` dodaje sljedeće — uvezite ih izravno iz `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Izuzeci

Uhvatite baznu klasu za rukovanje &quot;svime što je pošlo po zlu&quot;:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` i `ChlorosConfigurationError` izvoze se na vršnoj razini uz ostalih; također se mogu uvesti iz `chloros_sdk.exceptions` kao što je prikazano.

Hierarhija:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Primjeri od početka do kraja

### 1. Obrada mape s prilagođenom trakom napretka

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. LATTICE niz uživo → Reflektancija + DAQ referenca

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Kampanja snimanja vođena projektom

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Multi-kamera Frame-Stream → NumPy cijev

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skripta za snimanje bez zaslona izravno na hardveru (bez pozadinskog sustava)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Provjera mogućnosti prije povezivanja niza od 4 kamere

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent recepta za snimanje (čisti Python)

DSL za recepte (CLI) ima izravan ekvivalent u Pythonu:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatski početak pozadine

Pametne ulazne točke za povezivanje — `connect_camera`, `connect_array`, `connect_daq_sensor` i `discover_lattice_cameras` — su tanki klijenti HTTP koji pretpostavljaju backend sluša na `127.0.0.1:5000` (zadana URL pametnog povezivanja). Kada je GUI ili CLI već pokrenut, jedna je. Iz prazne skripte možda ne postoji — pa ove funkcije **automatski pokreću priloženi binarni backend** (bez prozora, na isti način kao `ChlorosLocal` radi) prije njihovog prvog poziva, a zatim čeka do `backend_startup_timeout` da se pojavi.

Pravila:

- **Uvijek se pokreće samo lokalni URL.** `backend_url` koji upućuje na `localhost` / `127.0.0.1` / `[::1]` je prihvatljiv; bilo koji drugi host smatra se tuđim računalom i nikada se ne pokreće.
- **Backend ostaje u pokretanju za ponovnu upotrebu** (isto kao kod CLI) — nema implicitnog zatvaranja kada se vaša skripta završi. Ponovno pokretanje skripte ponovno koristi isti aktivni backend.
- **Onemogućite ponovnu upotrebu pomoću `auto_start_backend=False`** na bilo kojem od tih poziva (npr. kada ste uputili na udaljeni backend ili kada sami upravljate životnim ciklusom backenda).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ako se priložena binarna datoteka ne može pronaći ili pokrenuti, naknadni poziv HTTP podiže radnju koja se može poduzeti, **svjesnu platforme** `ChlorosConnectError` umjesto sirovog traga odbijenog povezivanja — na Windows usmjerava vas na desktop aplikaciju ili naredbu `chloros-cli`; na Linux (bez GUI-ja) upućuje vas na `chloros-cli` naredbu ili `.deb`.

---

## Okruženje i zaglavlja

SDK označava svaki poziv backenda HTTP s `X-Chloros-Client: sdk`. Backend primjenjuje pravila licenciranja SDK / CLI (potrebni su prijava **i** plaćeni plan Chloros+), umjesto GUI putanje s besplatnom razinom. Ovo se postavlja automatski tijekom uvoza — ne morate ništa raditi.

`http://localhost` i `http://127.0.0.1` detektiraju se kao lokalni backend. Pozivi na druge hostove (npr. vlastitu uslugu analitike) ostaju nepromijenjeni.

Prepišite pozadinsku uslugu URL prosljeđivanjem `backend_url=` (ili `api_url=` na `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` koji nije loopback doseže samo izvorni/dev backend — isporučeni backendi vežu se samo na loopback; pogledajte način rada udaljenog backenda (Remote-Backend Mode) za obrazac tunela.)

---

## Verziranje i kompatibilnost

- Verzija SDK izložena je kao `chloros_sdk.__version__`.
- SDK veže se za ponašanje verzije priloženog backenda. Miješanje starije verzije SDK s novijom verzijom backenda obično radi (napredno kompatibilni krajevi), ali kombiniranje novije verzije SDK s starijim pozadinskim sustavom može uzrokovati pogreške `404` na novim krajevima — ažurirajte desktop aplikaciju da bi odgovarala.
- Smart-connect sučelje (`connect_camera` / `connect_array` / `connect_daq_sensor`) i endpoint za analizu mreže vraćaju stabilne sheme JSON; nova polja su dodatna.

---

## Savjeti za rješavanje problema

- **`ChlorosAuthenticationError: Login required`** → Pokrenite `chloros-cli login EMAIL PASSWORD` jednom na ovom računalu ili se prijavite putem desktop aplikacije Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect poziva automatski lokalni backend, stoga se ova poruka prikazuje samo kada se priložena binarna datoteka ne može pronaći/pokrenuti (npr. na hostu samo s pip-om bez desktop paketa). Poruka je prilagođena platformi: na Windows otvorite desktop aplikaciju ili pokrenite bilo koju `chloros-cli` naredbu; na Linux pokrenite `chloros-cli` naredbu (ne postoji GUI) ili instalirajte `.deb`. Za udaljeni backend, unesite `backend_url=` (i `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** pri uvozu → `lattice_sdk` nije uspio učitati (obično nisu instalirane DLL-ove za vrijeme izvođenja programa Arena SDK). Površina bez kamere i dalje radi.
- **Povezivanje niza vraća nativnu rezoluciju**→ Backends smart-prep automatski smanjuje veličinu okvira kako bi se uklopio u vezu. Koristite `analyze_array_network()` da vidite zašto, a zatim ili nadogradite vezu, prihvatite smanjenje ili proslijedite `force_tier="slip-emit-and-capture"` za sekvencijalno snimanje. Sigurnosna mreža za smanjenje**ne** pokriva agregatnu prekomjernu dodjelu (`oversubscribed: true`, fps polja 0): Previše kamera za vezu ne može se riješiti binningom/ROI-jem — smanjite broj kamera, omogućite jumbo pakete ili prijeđite na brži mrežni adapter (NIC) (vidi [Prekomjerno pretplaćivanje](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` izvještava o NIC RX prstenu kao malom (~0,26 MB) / connect gates s porukom &quot;FRAMES WILL DROP&quot;** → Receive ring host NIC-a je na zadanoj vrijednosti (često se resetira na 32 nakon ažuriranja upravljačkog programa mrežne kartice). Na Realtek USB 10GbE adapteru postavite `ReceiveBufferLen=256` i `PendingReceives=64` (s podignutim privilegijama), a zatim ponovno pokrenite backend-a kako bi ponovno pročitao ring. Cijeli postupak: [CLI Referenca → Postavljanje mrežne kartice hosta i Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host se zaglavi pri ponovnom pokretanju/isključivanju, kasnije WMI `Invalid class` pogreške / NIC se neće omogućiti** → zastarjeli upravljački program USB 10GbE uzrokuje `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Ažurirajte upravljački program adaptera na najnoviju verziju (≥ 2026) i ponovno primijenite postavke receive-ringa. Pogledajte [CLI Referenca → Postavljanje i podešavanje mrežne kartice domaćina](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Odbijanje reflektancije odbijeno** → U realnom vremenu DAQ mora biti povezan s kamerom (ili nizom) za apsolutnu mjeru reflektancije. Ili povežite putem GUI-ja ili upotrijebite `processing="radiance"` (W/m²/sr/nm) koji ne zahtijeva upareni senzor.
- **Prikazivanje `smart=True` traje dulje od očekivanog** → Konvergencija AE ovisi o dinamici scene; pooštrite `exposure_tolerance_pct` ili skratite `stability_window_s` ako želite brži (manje stabilan) okidač.

---

## Vidi također

- [CLI Referenca](cli-reference.md) — svaka podkomanda CLI ogleda se u pozivu SDK.
- [Vodič za DAQ senzore](../daq/README.md) — pravila ožičenja, kalibracije i snimanja specifična za senzor.
- Online dokumentacija: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
