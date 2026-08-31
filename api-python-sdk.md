# API : Python SDK

{% hint style="info" %}
**Tražite potpuni API?** Ova stranica je praktični vodič. Svaka javna klasa, metoda, točna potpis i primjer za kopiranje i lijepljenje nalaze se u [SDK Referenci](reference/sdk-reference.md), koja je optimizirana za AI asistente.**Radite li s AI asistentom?** Zalijepite ovu URL u chat kako biste imali potpunu, trenutačnu verziju Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Svaka stranica ovog priručnika dostupna je kao sirovi markdown pod svojim malim slugom + `.md`, a cijeli priručnik indeksiran je na `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI-ju) pokreće sve što desktop aplikacija može učiniti iz Python: grupnu obradu slika, upravljanje LATTICE kamerom i nizom u stvarnom vremenu, DAQ sesije sa senzorima svjetla i automatizaciju spremljenih projekata. To je tanak sloj iznad istog lokalnog backenda koji koriste GUI i CLI (HTTP na `127.0.0.1:5000`), pa je ponašanje identično na sva tri sučelja.

## Instalacija

Instalacija se sastoji od dva koraka: prvo Chloros desktop paket (koji osigurava pozadinski sustav za obradu i hardverske izvršne okruženja), a zatim Python paket.

**Korak 1 — Instalirajte Chloros.** Windows: pokrenite desktop instalacijski program (zadana putanja `C:\Program Files\MAPIR\Chloros\`) s [Download](download.md) stranice. Linux: instalirajte paket `.deb` ([Linux Instalacija](linux/linux-installation.md)).**Korak 2 — Instalirajte SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Možda vam neće ni trebati pip: svaki instalacijski program isporučuje odgovarajuće SDK kolo. Instalator za Windows automatski ga instalira na vaš sustav Python; instalator za Linux `.deb` postavlja ga na `/usr/lib/chloros/sdk/` i ispisuje točnu naredbu `pip install --user`. PyPI se ažurira na buildovima za objavljivanje, tako da se `pip install chloros-sdk` podudara s najnovijom stabilnom verzijom.

**Korak 3 — Prijavite se jednom po računalu:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Vjerodajnice se pohranjuju u `~/.chloros/` (na obje platforme). Na Windows možete se jednako tako prijaviti putem kartice &quot;User <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">&quot; u desktop aplikaciji. SDK zahtijeva plaćeni plan Chloros+ — pogledajte [Zahtjev za licencu](#license-requirement) u nastavku.

| Zahtjev | Detalji |
| --- | --- |
| **Chloros instaliran** | Windows: instalacijski program za stolna računala; Linux: paket `.deb` (osigurava binarnu datoteku pozadinskog sustava) |
| **Python** | 3.7 ili noviji (razvijeno/testirano na 3.10) |
| **Operativni sustav** | Windows 10/11 64-bit, Ubuntu 22.04 LTS ili noviji, ili NVIDIA Jetson (JetPack 6) |
| **Licenca** | Aktivna Chloros+ prijava, bilo koja plaćena pretplata (Copper ili viša) |

## Pobjeda u 60 sekundi

Jedan poziv stvara projekt, uvozi mapu, konfigurira obradu i pokreće pipeline — automatski pokrećući backend ako već ne radi:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Na Linuxu koristite staze Linux: `/home/user/drone_images/flight001`. SDK radi identično na obje platforme.)

Prerađujete mapu snimanja LATTICE-a? Koristite omotač prilagođen za LATTICE — primjenjuje ispravne zadane postavke (bez detekcije panela-cilja, standardni debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — potpuna kontrola nad cjevovodom

Za sve što je duže od jedne naredbe, upotrijebite `ChlorosLocal`. On pokreće pozadinski proces pri prvom korištenju (`auto_start_backend=True`), stvara i konfigurira projekte, prati napredak i vraća sažetak nakon izvršavanja.

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
Koristite zadani `http://127.0.0.1:5000` umjesto zamjene `localhost` — na Windows, `localhost` se prvo rješava na `::1` i košta ~2 sekunde po zahtjevu na pozadinskom sustavu samo za IPv4.
{% endhint %}

Koristite ga kao upravitelja konteksta za zajamčeno čišćenje:

```python
import chloros_sdk

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

`configure()` prihvaća ove ključne riječi: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` i `custom_settings`. Glavne vrijednosti:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Kontrole specifične za LATTICE (`input_level`, `radiometric_output`, obitelj `array_alignment*`) dokumentirane su sa svojim potpunim tabelama vrijednosti u referenci [SDK](reference/sdk-reference.md#supported-values)..

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

### Čitanje sažetka nakon izvođenja — i otkrivanje praznih izvođenja

Po završetku, `process()` prilaže sažetak obrade pozadinskog sustava kao `result["summary"]`. Svaki unos u `summary["hints"]` je puna rečenica koja objašnjava sve što je značajno — na primjer, zašto je izvršavanje proizvelo nulti rezultat — a svaki trag također se ponovno šalje kao Python `UserWarning`, tako da su prazna izvršavanja samodijagnostička čak i ako nikada ne provjerite dict:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` se ne aktivira kada izvršavanje ne proizvede nijednu sliku.** Ovo je jedino područje u kojem se SDK i CLI namjerno razlikuju: `chloros-cli process` tretira poruku &quot;products were requested, none were written&quot; kao neuspjeh i izlazi s ne-nulovitim rezultatom, dok SDK normalno završava i prijavljuje stanje putem `summary` / hints. Ako bi vaš pipeline trebao stati na praznom pokretanju, provjerite to sami — pregledajte `summary` (ili prebrojite datoteke u mapi projekta) umjesto da se oslanjate na iznimku.
{% endhint %}

## Smart Connect — hardver u stvarnom vremenu

Tri pomoćnika otvaraju trajne sesije u hardverskom poolu pozadinskog sustava — istom poolu koji GUI koristi, tako da skripte SDK-a koegzistiraju s desktop aplikacijom bez borbe za serijske priključke ili mrežnu propusnost. Sva tri automatski pokreću lokalni pozadinski sustav ako nijedan ne radi.

### Jedna LATTICE kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Sinkronizirani niz — `connect_array`

`connect_array` je preporučena polazna točka za sustave s više kamera. Pokreće isti &#x27;smart-prep&#x27; postupak kao i GUI: analizu mreže, automatski odabir sloja za sinkronizaciju, PTP sinkronizaciju vremena, odabir formata piksela za svaku kameru, postavljanje AE-a i aktivaciju okidača putem GPIO-a. **Prvi serial je master** (pokreće hardverski impuls okidača); ostali su slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Dodajte `smart=True` bilo kojem snimanju niza kako biste pričekali da se automatska ekspozicija stabilizira na svim kamerama prije aktiviranja. Za načine snimanja (jedno/kontinuirano/intervalno/najbrže), snimače, burst-to-video i poravnanje niza pogledajte [Referencu za SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ senzor svjetla — `connect_daq_sensor`

Bez argumenata, `connect_daq_sensor()` pametno otkriva transport (redoslijed prioriteta: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Svaki kadar nosi 135 bodova `spectrum` (W/m²/nm pri kalibraciji), zastavicu `is_saturated` i CIE `x`, `y`, `z`. Da biste dodijelili određeni senzor ili transport — pouzdan izbor na hostovima s više mrežnih sučelja, gdje Ethernet automatsko otkrivanje može propustiti ispravan DAQ-E pri prvom pokušaju — proslijedite jedan eksplicitan nagovještaj:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Imajte na umu da profili korekcije kape (`cap_id`) **nisu** kotačić za korekciju kape — umjesto toga odaberite ih putem `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Sačuvani projekti — `open_project`

Sačuvani projekt Chloros-a zadržava povezan hardver (`cameras.json` + `sensors.json` zajedno s `project.json`), i `chloros_sdk.open_project(path)` mogu ponovno povezati sve odjednom i upravljati snimanjima po nazivu uređaja. Pogledajte [Projektnu automatizaciju](reference/sdk-reference.md#project-automation--chlorosproject) u referenci.

## Što dobivate instalacijom samo putem pip-a

Provjerite zastavice dostupnosti na razini modula prije korištenja hardverskih površina:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hostu s **samo** `pip install chloros-sdk` i bez desktop paketa Chloros:

* `ChlorosLocal`, `process_folder` i `process_lattice_capture` **ne** rade — potrebna im je pozadinska binarna datoteka koja se isporučuje u instalacijskom paketu za stolna računala.
* Pomoćnici za pametno povezivanje (`connect_camera`, `connect_array`, `connect_daq_sensor`) su čisti klijenti za daljinski pristup (HTTP), stoga rade s pozadinskim poslužiteljem na drugom računalu — ali isporučeni backendi vežu se samo na loopback, stoga port morate sami proslijediti (npr. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) i proslijediti `backend_url="http://127.0.0.1:5000"` s `auto_start_backend=False`. Pogledajte [Remote-Backend Mode](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Direktne hardverske LATTICE klase (`LatticeCamera`, `CameraPool`, …) import, ali im je potreban Arena SDK runtime iz desktop paketa — bez njega `CAMERA_AVAILABLE` je `False`.
* `daq_sdk` (direktne DAQ klase) dolazi s desktop instalacijom, a ne s PyPI paketom, dakle `DAQ_AVAILABLE` je `False` na hostu koji koristi samo pip — umjesto toga upravljajte DAQ senzorima putem `connect_daq_sensor()` prema (tuneliranom) backendu.

## Zahtjev za licencu

Pristup SDK zahtijeva aktivno prijavljivanje na Chloros+ na bilo kojem plaćenom nivou — **Copper ili višem**(Copper / Bronze / Silver / Gold); besplatni nivo Iron nema pristup SDK / CLI. Provjera se provodi**na strani poslužitelja**: svaki zahtjev za SDK mora sadržavati i aktivnu sesiju i plaćeni plan, inače pozadinski sustav vraća `403` / `PLAN_UPGRADE_REQUIRED` (izražen kao `ChlorosLicenseError` od strane `ChlorosLocal`, i kao `ChlorosConnectError` od strane pomoćnika `connect_*`). Prijavitelj bez prijave dobiva `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) umjesto toga — ponovno pokretanje `chloros-cli login` rješava prvi slučaj, ali ne i drugi.

Korištenje izvan mreže funkcionira unutar probnog razdoblja plana: razina se čita iz predmemorije provjere poslužitelja (5 minuta) ili iz predmemorije potpisane licence vezane za računalo (30 dana za mjesečne planove; do isteka pretplate za godišnje). Kada taj razdoblje istekne, plan se postavlja na besplatni i pristup u načinu rada &quot;SDK&quot; prestaje dok se računalo jednom ne poveže sa poslužiteljem. `chloros-cli status` ostaje dostupan na besplatnom nivou, tako da je razlog uvijek vidljiv. Pogledajte [Chloros+ Login](chloros+-login.md).

## Iznimke

Zahvatite osnovnu klasu za rukovanje &quot;svime što je pošlo po zlu u Chloros&quot;:

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

Sve iznimke pipelinea (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) izvedeni su iz `ChlorosError`. Jedna zamka: `ChlorosConnectError` — podiže se samo od `connect_camera` / `connect_array` / `connect_daq_sensor` — nasljeđuje od običnog `Exception`, **ne** od `ChlorosError`, stoga ga `except ChlorosError` neće uhvatiti. Cjelokupna hijerarhija nalazi se u [SDK Referenciji](reference/sdk-reference.md#exceptions).

## Vidi također

* [SDK Referencija](reference/sdk-reference.md) — potpuna površina API, optimizirana za AI asistente.
* [CLI Referenca](reference/cli-reference.md) — svaka podzapovijed CLI ogleda se u pozivu SDK.
* [Preuzimanje](download.md) — instalacijski programi za Windows i Linux.
