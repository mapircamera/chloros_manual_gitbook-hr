# API: Python SDK

{% hint style="info" %}
**Tražite li cjeloviti API?** Ova stranica je praktični vodič. Svaka javna klasa, metoda, točan potpis i primjer za kopiranje nalaze se u [SDK referenci](reference/sdk-reference.md), koja je optimizirana za AI asistente.**Radite li s AI asistentom?** Zalijepite ovu URLu u chat kako biste imali potpuni, ažurirani Chloros 1.3.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Svaka stranica ovog priručnika dostupna je kao sirovi markdown na svojoj malom pismenom slugu + `.md`, a cijeli priručnik indeksiran je na `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros SDK**(`chloros-sdk` na PyPI-ju) upravlja svime što desktop aplikacija može učiniti iz Python-a: grupnom obradom slika, upravljanjem LATTICE kamerom i nizom u stvarnom vremenu, DAQ sesijama sa senzorima svjetla i automatizacijom spremljenih projekata. To je tanak sloj iznad istog lokalnog pozadinskog sustava (backend) koji koriste GUI i **CLI** (HTTP na `127.0.0.1:5000`), pa je ponašanje identično na sva tri sučelja.

## Instalacija

SDK dolazi s desktop paketom Chloros, koji također pruža pozadinski sustav za obradu i hardverske izvršne okruženja te uvijek ima istu verziju.

**Korak 1 — Instalirajte Chloros.** Windows: pokrenite instalacijski program za stolna računala (zadana putanja `C:\Program Files\Chloros\`) sa stranice [Preuzimanje](download.md). Linux: instalirajte paket `.deb` pomoću `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalacija za Linux](linux/linux-installation.md)).**Korak 2 — Provjerite **SDK** (Python 3.7+). Obično nema ništa za instalirati:

* **Windows:**instalator instalira priloženo**SDK** kotač u vaš sustav Python (ako Python nije instaliran, ovaj korak preskače).
* **Linux:** `.deb` instalira SDK za sustav `python3`, u `/usr/lib/python3/dist-packages`. Njegovi zahtjevi, `python3-numpy` i `python3-requests`, preporučeni su od strane paketa: `apt install` ih donosi, `dpkg -i` ne.
* **Virtualenv** ne vidi sistemske pakete; instalirajte priloženi wheel u njega: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` na Linuxu, ili `chloros_sdk-*.whl` u mapi `sdk` instalacijske mape za Windows (`C:\Program Files\Chloros\sdk\` prema zadanim postavkama).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Na računalu bez paketa Chloros, čiji SDK pokreće pozadinski sustav (backend) na drugom računalu (pogledajte [Što dobivate instalacijom samo putem pip-a](#what-a-pip-only-install-gets)), instalirajte ga s PyPI-ja: `pip install chloros-sdk`. PyPI se ažurira za verzije za objavljivanje, tako da odgovara najnovijoj stabilnoj verziji.

**Korak 3 — Prijavite se jednom po računalu:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Vjerodajnice se pohranjuju u `~/.chloros/` (na obje platforme). Na Windowsu se možete jednako tako prijaviti putem kartice Korisnik <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> u desktop aplikaciji. SDK zahtijeva plaćeni Chloros+ plan — pogledajte [Zahtjev za licencu](#license-requirement) u nastavku.

| Zahtjev | Detalji |
| --- | --- |
| **Instaliran**Chloros**| Windows: desktop instalacijski program; Linux: paket `.deb` (osigurava binarnu datoteku pozadinskog sustava i**SDK**) |
| **Python** | 3.7 ili noviji (razvijano/testirano na 3.10) |
| **Operativni sustav** | Windows 10/11 64-bitni, Ubuntu 22.04 LTS ili noviji, ili NVIDIA Jetson (JetPack 6) |
| **Licenca** | Aktivno prijavljivanje na Chloros+, bilo koja plaćena pretplata (Copper ili viša) |

## Pobjeda u 60 sekundi

Jedan poziv stvara projekt, uvozi mapu, konfigurira obradu i pokreće pipeline — automatski pokrećući backend ako već nije u pokretu:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Na Linuxu koristite Linux putanje: `/home/user/drone_images/flight001`. SDK radi identično na obje platforme.)

Prerađujete mapu snimanja LATTICE-a? Koristite omotač prilagođen za LATTICE — primjenjuje odgovarajuće zadane postavke (bez detekcije panela-cilja, standardni debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — potpuna kontrola nad cjevovodom

Za sve što je duže od jedne naredbe, upotrijebite `ChlorosLocal`. On koristi pozadinski servis koji već radi (onaj desktop aplikacije) ili pokreće instalirani servis pri prvom korištenju (`auto_start_backend=True`; na Windowsu iz mape `backend` instalacijske mape Chlorosa, `C:\Program Files\Chloros\backend\` prema zadanim postavkama), stvara i konfigurira projekte, nadzire napredak i vraća sažetak nakon izvođenja.

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

{% hint style="info" %}
Zadržite zadani `http://127.0.0.1:5000` umjesto zamjene `localhost` — na Windows, `localhost` se prvo rješava na `::1` i košta ~2 sekunde po zahtjevu prema backendu koji podržava samo IPv4.
{% endhint %}

Koristite ga kao upravitelja konteksta. Napuštanje bloka prekida korištenje backenda za ovu instancu, a ne samog backenda: on ostaje aktivan za sljedeći skript i sam se zaustavlja nakon 30 minuta neaktivnosti. `shutdown_backend()` zaustavlja ovu instancu čim se pokrene.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` prihvaća ove ključne riječi: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` i `custom_settings`. Glavne vrijednosti:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Gumbi specifični za LATTICE (`input_level`, `radiometric_output`, obitelj `array_alignment*`) dokumentirani su sa svojim potpunim tabelama vrijednosti u [Referenciji za SDK](reference/sdk-reference.md#supported-values).

### Promatranje napretka

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Čitanje sažetka nakon izvođenja — i hvatanje praznih izvođenja

Po završetku, `process()` prilaže sažetak obrade pozadine kao `result["summary"]`. Svaki unos u `summary["hints"]` je puna rečenica koja objašnjava sve što je značajno — na primjer, zašto je izvršavanje proizvelo nulti rezultat — a svaki trag također se ponovno emitira kao Pythonov `UserWarning`, tako da se prazna izvršavanja samodijagnosticiraju čak i ako nikada ne provjerite dict:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` ne izaziva iznimku kada pokretanje ne proizvede nijednu sliku.** Ovo je jedino mjesto u kojem se SDK i CLI namjerno razlikuju: `chloros-cli process` tretira poruku &quot;zatraženi su proizvodi, nijedan nije napisan&quot; kao neuspjeh i izlazi s nestandardnim izlaznim kodom, dok SDK normalno završava i izvještava o tom stanju putem `summary` / naznaka. Ako vaša cijevna linija treba stati na praznom pokretanju, provjerite to sami — pregledajte `summary` (ili prebrojite datoteke u mapi projekta) umjesto da se oslanjate na iznimku.
{% endhint %}

## Smart Connect — hardver u stvarnom vremenu

Tri pomoćnika otvaraju trajne sesije u poolu hardvera pozadinskog sustava — istom poolu koji koriste CLI, druge skripte i kartice za kamere u GUI-ju, tako da skripte SDK postoje zajedno s njima bez borbe za serijske priključke ili mrežnu propusnost. Sva tri automatski pokreću lokalni pozadinski sustav ako nijedan ne radi. GUI-jev DAQ panel nije dio pool-a: zatvorite senzor svjetla u GUI-ju prije nego ga otvorite iz skripte.

### Jedna LATTICE kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relativne staze vode do mjesta gdje se vaša skripta pokreće (promijenjeno u verziji 1.3.0).** S lokalnim pozadinskim sustavom, relativna `output_dir` (uključujući zadanu `"output"`) — i ostale putanje koje pozadinski sustav otvara (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — rješava se u radnom direktoriju vaše skripte, pa `cam.capture("output/")` piše u `<current folder>/output/`. U verziji 1.2.0 instalirani backend ih je rješavao u vlastitoj instalacijskoj mapi i davao pogrešku &quot;Pristup je odbijen&quot;. Kod udaljenog backenda, putanje se šalju onakve kakve su napisane.
{% endhint %}

### Sinkronizirani niz — `connect_array`

`connect_array` preporučena je polazna točka za više-kamere sustave. Pokreće isti &#x27;smart-prep&#x27; tijek kao i GUI: analizu mreže, automatski odabir sloja za sinkronizaciju, PTP sinkronizaciju vremena, odabir formata piksela za svaku kameru, inicijalizaciju AE-a i aktivaciju okidača putem GPIO-a. **Prvi serijski je master** (pokreće hardverski impuls okidača); ostali su slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Dodajte `smart=True` u bilo koji niz za snimanje kako biste pričekali da se automatska ekspozicija stabilizira na svim kamerama prije aktiviranja.

Novo u verziji 1.3.0, `ArraySession` vam također izravno isporučuje sinkronizirane sličice, bez spremanja u datoteke:

* `arr.grab()` vraća jedan kadar po kameri, sve s jednog okidača, kao numpy nizove (`{serial: array}`); `.group_id` je zajednički okidač, a `.errors` navodi svaku kameru koja je izostavljena.
* `arr.frame_stream(fps=5.0)` isporučuje jedan takav set po tik-u, za obradu uživo u vlastitom procesu obrade.
* `arr.continuous("output/")` pokreće vlastitu kontinuiranu snimku pozadinskog sustava (kontinuirani način rada GUI-ja) i vraća `ContinuousHandle` s `status()`, `wait()` i `stop()` — koristite ih umjesto petlje snimanja kada trebate spremiti svaki okidač velikom brzinom.

Za načine snimanja (jedno, kontinuirano, intervalno, najbrže), snimače, burst-to-video i poravnanje niza pogledajte [Referencu za SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ senzor svjetla — `connect_daq_sensor`

Bez argumenata, `connect_daq_sensor()` pametno otkriva transport (redoslijed prioriteta: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Svaki kadar nosi 135-bodovni `spectrum` (W/m²/nm pri kalibraciji), zastavicu `is_saturated` i CIE `x`, `y`, `z`. Da biste dodijelili određeni senzor ili transport, proslijedite jednu eksplicitnu naznaku:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Imajte na umu da profili za korekciju kapica (`cap_id`) **nisu** kotačić u SDKu — umjesto toga odaberite ih putem `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Sačuvani projekti — `open_project`

Sačuvani projekt Chloros zadržava povezani hardver (`cameras.json` + `sensors.json` zajedno s `project.json`), i `chloros_sdk.open_project(path)` može ponovno povezati svoje kamere, nizove i senzore svjetla odjednom te upravljati snimanjima po nazivu uređaja. Senzori svjetla ponovno se povezuju putem pozadinskog procesa sustava Chloros, sa svojim spremljenim postavkama i kapom. Pogledajte [Automacija projekta](reference/sdk-reference.md#project-automation--chlorosproject) u referenci.

## Što dobivate instalacijom samo putem pip-a

Provjerite zastavice dostupnosti na razini modula prije korištenja hardverskih površina:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hostu s **samo** `pip install chloros-sdk` i bez desktop paketa Chloros:

* `ChlorosLocal`, `process_folder` i `process_lattice_capture` **ne** rade — potrebna im je pozadinska binarna datoteka koja se isporučuje u instalacijskom paketu za stolna računala.
* Pomoćnici za pametno povezivanje (`connect_camera`, `connect_array`, `connect_daq_sensor`) su čisti klijenti za HTTP, stoga rade s pozadinskim sustavom na drugom računalu — ali isporučeni backendi vežu se samo na loopback, stoga morate sami proslijediti port (npr. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) i proslijediti `backend_url="http://127.0.0.1:5000"` s `auto_start_backend=False`. Pogledajte [Remote-Backend Mode](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Direktne hardverske LATTICE klase (`LatticeCamera`, `CameraPool`, …) uvoza, ali im je potreban Arena SDK runtime iz desktop paketa — bez njega `CAMERA_AVAILABLE` je `False`.
* DAQ senzori se upravljaju isključivo putem (tuneliranog) backenda — `connect_daq_sensor()` ili projektnog `proj.sensors`; backend otvara senzor na vlastitom računalu.

## AI asistenti — `chloros-mcp`

SDK uključuje `chloros-mcp`, MCP poslužitelj koji omogućuje klijentima Claude Desktop, Claude Code, Cursor i drugim MCP klijentima upravljanje Chloros: obradu slika i pokretanje LATTICE kamera, nizova i DAQ svjetlosnih senzora. Njegovi alati pokrivaju ono što ovaj SDK radi. Instalira se s SDK-om, pa je za postavljanje Claude Code-a potrebna jedna linija:

```bash
claude mcp add chloros -- chloros-mcp
```

Ostali klijenti: [Korištenje Chloros-a s AI asistentima](ai-assistants.md). Opcije i svaki alat: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Zahtjev za licencu

Pristup SDK-u zahtijeva aktivno prijavljivanje na Chloros na bilo kojem plaćenom paketu — **Copper ili višem**(Copper / Bronze / Silver / Gold); besplatni Iron paket nema pristup SDK/CLI. Provedba je**na strani poslužitelja**: svaki zahtjev SDK mora sadržavati i aktivnu sesiju i plaćeni plan, inače backend vraća `403` / `PLAN_UPGRADE_REQUIRED` (prikazano kao `ChlorosLicenseError` od strane `ChlorosLocal`, i kao `ChlorosConnectError` od strane pomoćnika `connect_*`). Prijavitelj bez prijave dobiva `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) umjesto toga — ponovno pokretanje `chloros-cli login` rješava prvi slučaj, ali ne i drugi.

Korištenje izvan mreže funkcionira unutar probnog razdoblja plana: razina se čita iz predmemorije provjere poslužitelja (5 minuta) ili iz predmemorije potpisanih licenci vezanih za računalo (30 dana za mjesečne planove; do isteka pretplate za godišnje). Kada istekne razdoblje milosti, plan se postavlja na besplatni i pristup SDK-u prestaje dok se uređaj jednom ne poveže sa serverom. `chloros-cli status` ostaje dostupan na besplatnom nivou, tako da je razlog uvijek vidljiv. Pogledajte [Chloros+ Login](chloros+-login.md).

## Iznimke

Zahvatite osnovnu klasu za rukovanje &quot;svime što je pošlo po zlu s Chlorosom&quot;:

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

Sve iznimke pipelinea (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) nasljeđuju od `ChlorosError`. Jedna zamka: `ChlorosConnectError` — podiže se samo od `connect_camera` / `connect_array` / `connect_daq_sensor` — nasljeđuje od običnog `Exception`, **ne** od `ChlorosError`, stoga ga `except ChlorosError` neće uhvatiti. Potpuna hijerarhija nalazi se u [SDK referenci](reference/sdk-reference.md#exceptions).

## Vidi također

* [SDK referenca](reference/sdk-reference.md) — cjelovita API sučelje, optimizirano za AI asistente.
* [Referenca za CLI](reference/cli-reference.md) — svaka podkomanda CLI odražava poziv SDK.
* [Korištenje Chlorosa s AI asistentima](ai-assistants.md) — postavljanje `chloros-mcp` i recepti za upite.
* [Preuzimanje](download.md) — instalacijski programi za Windows i Linux.
