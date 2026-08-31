# Instalacija paketa Linux

Chloros se za Linux distribuira kao paketi `.deb` koji instaliraju CLI i pozadinski poslužitelj. Python SDK je zaseban pip paket (također uključen unutar paketa `.deb` kao wheel paket usklađene verzije).

Nazivi paketa nose verziju i arhitekturu: `chloros_1.2.0_amd64.deb` za x86_64 i `chloros_1.2.0_arm64_jp6.deb` za JetPack 6 Jetson izgradnje. U naredbama u nastavku zamijenite datoteku koju ste zapravo preuzeli.

***

## Linux amd64 (x86_64)

### Sustavni zahtjevi

| Zahtjev | Minimalno | Preporučeno |
| --- | --- | --- |
| **Distribucija** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 ili bolji |
| **Memorija (RAM)** | 8 GB | 16 GB ili više |
| **Grafička kartica** | Nije potrebno (obrada u CPU-u) | NVIDIA GPU s 4 GB+ VRAM-a (12 GB+ otključava `GPU_PARALLEL`, 7 GB+ omogućuje da Texture Aware ostane isključeno na putanji za pojedinačnu sliku) |
| **Pohrana** | 2 GB slobodnog prostora | SSD s 10 GB+ slobodnog prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

> **Ubuntu 20.04 i Debian 11 nisu podržani.** Popis ovisnosti `.deb`-a
> izveden je iz onoga s čime se pozadinski program Chloros zapravo povezuje, a to uključuje
> `libc6 (>= 2.34)`. Focal i bullseye oboje dolaze s glibc 2.31, stoga `apt` odbijaju instalaciju
> bezuvjetno, umjesto da dopuste da kasnije dođe do pogreške tijekom izvođenja.

### Instalacija

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` ne rješava ovisnosti. Ako prijavi nedostajuće pakete, `sudo apt-get install -f` (ili `sudo apt --fix-broken install`) dovršava instalaciju — ovo je normalan postupak, a ne pogreška.
{% endhint %}

Provjerite instalaciju:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->***

## Linux arm64 (NVIDIA Jetson)

### Sustavni zahtjevi

| Zahtjev | Minimalno | Preporučeno |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPackom 6 | Jetson Orin NX 16GB ili AGX Orin |
| **JetPack** | JetPack 6.x | Najnoviji JetPack 6 |
| **Memorija (RAM)** | 8 GB (zajednička za GPU/CPU) | 16 GB+ zajedničke (12 GB+ je prag za paralelne GPU radnike) |
| **Pohrana** | 2 GB slobodnog prostora | NVMe SSD s 10 GB+ slobodnog prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Instalacija

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ista konfiguracija kao i amd64 `.deb`, s CUDA izradom optimiziranom za Jetson Orin / Orin NX / Orin Nano. Za informacije o memoriji, toplinskom upravljanju i postavljanju na terenu za Jetson, pogledajte [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalacija za Python SDK (Sve Linux)

SDK je čisti Python HTTP klijent za pozadinu (backend), stoga isti paket radi na amd64 i arm64. Dva izvora:**S PyPI-ja** — objavljena stabilna verzija:

```bash
pip install chloros-sdk
```

**Iz priloženog wheel paketa** — zajamčeno odgovara CLI/backendu koji ste upravo instalirali (koristite ovo kada je vaš `.deb` noviji od onog na PyPI-ju):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668 distribucije** (Ubuntu 23.10+, Debian 12+) odbijaju instalacije pipa na razini sustava. Koristite `pip install --user …`, virtualno okruženje ili `sudo pip install --break-system-packages …`. Instalator paketa nikada ne instalira automatski SDK na vaš sustav Python — taj izbor prepušten je vama.
{% endhint %}

Dodatne opcije:

| Dodatna opcija | Naredba | Dodaje |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` za praćenje napretka uživo |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` za BLE (DAQ-M) transport |

Provjerite SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` instalira Chloros CLI i backend. Python SDK komunicira s tim pozadinskim sustavom putem lokalnog HTTP API (`http://127.0.0.1:5000`) i automatski ga pokreće kada je potrebno. Uvijek koristite doslovnu IPv4 adresu umjesto `localhost` — `localhost` može se razriješiti na `::1` i to traje otprilike dvije sekunde po zahtjevu.
{% endhint %}

***

## Početna postavka

### 1. Prijava

Pristup stranicama CLI i SDK zahtijeva plaćeni Chloros+ paket (**Copper** ili viši), što se provodi na strani poslužitelja: korisnik bez prijave dobiva `401 AUTH_REQUIRED`, a korisnik besplatnog paketa (Iron) dobiva `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Vjerodajnice se pohranjuju u `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Morate se ponovno prijaviti nakon svake instalacije ili nadogradnje.** Skripta paketa `prerm` namjerno briše `~/.chloros/user_session.json` i keširanu licencu za svakog korisnika na računalu, tako da nova izrada uvijek ponovno provjerava licencu umjesto da vjeruje zastarjelom kešu.
{% endhint %}

### 2. Provjerite status svoje licence

```bash
chloros-cli status
```

`chloros-cli status` radi na bilo kojoj razini (uključujući besplatnu), tako da uvijek možete vidjeti zašto je pristup dostupan ili nije.

### 3. Pokrenite dijagnostiku sustava

```bash
chloros-cli selftest
```

Sedam provjera se izvršava redom, a naredba vraća ne-nuljevu vrijednost ako ijedna od njih ne uspije:

| # | Provjera | Što dokazuje |
| --- | --- | --- |
| 1 | **Verzija** | CLI izvještava o svojoj verziji (`v1.2.0`). |
| 2 | **Dostupan port** | Port 5000 je slobodan, *ili* je već odgovorio zdrav Chloros backend (što se računa kao prolaz). |
| 3 | **Pokretanje backenda** | Pokreće se binarna datoteka backenda. |
| 4 | **Test APIa (`/api/test`)** | Backend odgovara `status: ok`. |
| 5 | **Informacije o sustavu** | Ispisuje `GPU: <name>, CUDA: <bool>, PyTorch: <version>` iz `/api/system-info`. |
| 6 | **Modeli za Denoiser** | Pronalazi modele za `*.pth.enc` (na Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware je zapravo upotrebljiv — potreban je CUDA**i** najmanje jedna modelna datoteka. |

Pokretanje završava s `N/7 checks passed`, navodeći sve neuspjehe po imenu.

### 4. Obradite svoj prvi skup podataka

```bash
chloros-cli process ~/datasets/flight001
```

***

## Datoteke i direktoriji


Chlorosza svakog korisnika pohranjuje svoje vjerodajnice i konfiguraciju u jednom međuplatformskom direktoriju, **`~/.chloros/`** (na Windows, `%USERPROFILE%\.chloros\`). Dvije predmemorije specifične za Linux umjesto toga slijede XDG konvencije — one poštuju `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` kada su postavljene.

| Put | Svrha |
| --- | --- |
| `~/.chloros/user_session.json` | Predmemorija sesije prijave koju piše `chloros-cli login` (čisti se pri svakoj instalaciji/nadogradnji paketa) |
| `~/.chloros/working_directory.txt` | Preusmjeravanje zadane mape projekta (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferencija jezika za CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Postavka jezika zajednička s GUI-jem Windows — `language` ovdje ima prednost nad `cli_language.json` |
| `~/.chloros/update_cache.json` | Jednosatna predmemorija za provjeru ažuriranja pri pokretanju Linux/Jetson |
| `~/.chloros/backend.log` | Zapisnik pozadinskog sustava (backend) kada je pokrenut pomoću CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pohranjeni paketi kalibracije LATTICE za svaku kameru, indeksirani serijskim brojem i hash-om paketa |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Neobavezne korisničke preusmjeravajuće postavke za profile korekcije DAQ kapa |
| `~/.config/chloros/system_config.json` | Pohranjeni hardverski profil iz sustava Dynamic Compute Adaptation — izbrišite ga za prisilnu novu detekciju hardvera |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Zapisnici pozadinskog poslužitelja, jedna datoteka po pokretanju |
| `~/Chloros Projects/` | Zadana mapa projekta kada nije postavljena drugačija |

### Za cijeli sustav

| Putanja | Svrha |
| --- | --- |
| `/usr/bin/chloros-cli` | Omotna skripta — postavlja `LD_LIBRARY_PATH` za priložene nativne biblioteke, a zatim pokreće stvarnu binarnu datoteku |
| `/usr/bin/chloros-backend` | Omotna skripta — isto, plus `CHLOROS_PRODUCTION=1` kako se stražnja vrata za provjeru autentičnosti nikada ne bi mogla tiho onemogućiti |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Kompilirane binarne datoteke |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK runtime potreban za LATTICE kamere |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrirani modeli za uklanjanje šuma koje koristi debayer osjetljiv na teksture |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK kotač koji odgovara ovoj točnoj izradi |
| `/usr/lib/chloros/exiftool` | Uključeni exiftool (simbolički povezan s `/usr/local/bin/exiftool` samo ako ne postoji sustavni exiftool) |
| `/etc/chloros/update.conf` | Konfiguracija ažuriranja kanala koju čita `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Postavlja `net.ipv4.ip_unprivileged_port_start = 319` kako bi pozadinski sustav mogao vezati PTP priključke bez root prava |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Usmjerava dinamički učitavač na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Odobrava prijavljenom korisniku pristup DAQ-U USB serijskom mostu (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Usluga pozadinske aplikacije koja se može omogućiti (instalirana, **nije omogućena**) |
| `/usr/share/applications/chloros-cli.desktop` | Stavka u izborniku aplikacije &quot;Chloros CLI&quot; koja otvara terminala |

## Lokacija izvršne datoteke pozadinske aplikacije

CLI i SDK automatski otkrivaju pozadinsku uslugu:

| Komponenta | Putanja |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Pozadinska usluga | `/usr/lib/chloros/chloros-backend` |

Prepišite putanju backenda pomoću zastavice `--backend-exe` CLI ili parametra konstruktora `backend_exe` SDK, a port pomoću `--port` (zadano `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` usmjerava **`lattice`**,**`project`**i**`daq pool-*`** obitelji naredbi na udaljenom pozadinskom sustavu. Osnovne naredbe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) namjerno ga zanemaruju i uvijek ciljaju `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE kamere i DAQ senzori svjetla na Linux

Sve obitelji naredbi za hardver uživo rade na Linux (amd64 i Jetson):

* **`chloros-cli lattice`** — otkrijte, povežite, konfigurirajte i snimajte s LATTICE kamera i sinkroniziranih nizova. `.deb` uključuje izvršni program Arena SDK koji je potreban i registrira ga kod dinamičkog učitavača.
* **`chloros-cli daq pool-*`** — povezuje DAQ-U/M/E senzore svjetla putem pozadinskog pool-a, prenosi kalibrirana spektra i snima `.daq` datoteke. Kompilirani CLI isporučuje samo obitelj `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — pokretanje spremljenog projekta (njegove kamere, senzore i postavke obrade) bez zaslona.
* **`chloros-cli time-sync`** — pregledajte PTP grandmaster na kojem radi pozadinski program Chloros za kamere LATTICE i senzore DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je potreban za `pool-latest`, `pool-stream`, `pool-record` i `pool-set-cap`; `pool-list` prikazuje ID-ove koji su trenutno u skupu.

{% hint style="info" %}
**Za prvu DAQ-E vezu na računalu s više mrežnih sučelja upotrijebite `--eth-host`.** Automatsko otkrivanje pretražuje mDNS i može propustiti sučelje senzora zbog praznog ARP predmemorija, pa prvi `pool-connect --eth` nakon pokretanja može ne uspjeti čak i kada je senzor potpuno ispravan. Unos IP adrese ili naziva hosta senzora u potpunosti preskače otkrivanje.
{% endhint %}

**Serijske dozvole za DAQ-U** obrađuje instalirano udev pravilo (`uaccess` + grupa `dialout`). Ako senzor koji je već priključen ostane nedostupan, ponovno učitajte pravila ili ga ponovno priključite:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Pogledajte [CLI referencu](../CLI.md) za potpun skup naredbi.

### Uvijek aktivan PTP za hostove bez zaslona

Prilikom prve instalacije generira se systemd jedinica `chloros-backend.service`, ali **nije omogućena**. Na Jetsonu bez zaslona ili poslužitelju na kojem PTP sinkronizacija vremena treba neprekidno raditi za senzore DAQ-E i kamere LATTICE, omogućite je:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Bez toga, PTP radi samo dok pozadinski program `127.0.0.1:5000` radi — to jest, tijekom aktivne sesije CLI / SDK.

Jedinica povezuje pozadinski sustav s `127.0.0.1:5000` (postavke okruženja `CHLOROS_HOST` / `CHLOROS_PORT` unutar jedinice; nadjačati pomoću `sudo systemctl edit chloros-backend.service`) i ponovno ga pokreće u slučaju neuspjeha nakon 5 sekundi.

**Kako PTP dobiva svoje priključke.** PTP koristi UDP 319/320, oba ispod normalnog 1024 privilegiranog-port floor. `postinst` paketa piše `/etc/sysctl.d/60-chloros-ptp.conf` s `net.ipv4.ip_unprivileged_port_start = 319`, što omogućuje pozadinskoj aplikaciji da ih veže dok se pokreće kao vaš korisnik. Također primjenjuje `setcap cap_net_bind_service,cap_net_raw=+ep` na binarnu datoteku backenda kao dodatnu sigurnosnu mjeru — zato je `libcap2-bin` deklarirana ovisnost paketa.***

## Primjeri Bash skriptiranja

{% hint style="info" %}
**Kodovi izlaska prilagođeni skriptama.** `chloros-cli process` završava `0` kod uspjeha s kodom 0, a kod neuspjeha s ne-nulom — uključujući i pokretanje koje je zatražilo slikovne proizvode, ali nije napisalo nijedan (ispisuje `Processing finished but wrote no image products.` i naziv projektne mape te uobičajene uzroke). Uspješne izvedbe izvještavaju o tome koliko je slikovnih proizvoda napisano (`Image products written: N`). Kodovi izlaska: `0` uspjeh, `1` neuspjeh, `2` pogreška u argumentu, `130` prekinuto.
{% endhint %}

### Obrada više skupova podataka

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Obrada s prilagođenim postavkama

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Valjane vrijednosti `--format` su točno četiri i sadrže razmake — uvijek ih navedite u navodnicima:

| `--format` vrijednost | Izlazna mapa |
| --- | --- |
| `TIFF (16-bit)` *(zadano)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` prihvaća `standard` (zadano) ili `texture-aware` (Chloros+).

### Automatska obrada pomoću Cron-a

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Primjer Python-a SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Rješavanje problema

### CLI nije pronađen nakon instalacije

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Odbijeno dopuštenje

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### &quot;setcap failed&quot; tijekom instalacije

`.deb` primjenjuje `cap_net_bind_service` na `/usr/lib/chloros/chloros-backend` kako bi se mogao povezati na PTP priključke 319/320 bez root prava. Ako je `libcap2-bin` nedostajao tijekom instalacije, poziv se preskače. Instalirajte ga i ponovno instalirajte paket:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP se neće pokrenuti / Ne može povezati priključak 319

Potvrdite da je prag za neprivilegirane priključke snžen i ponovno ga primijenite za trenutno pokretanje ako nije bio:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Zatim provjerite grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### &quot;Nisu pronađeni upravljački programi za kameru LATTICE&quot;

Runtime za Arena SDK se ne rješava. Potvrdite da je konfiguracija učitavača koju paket piše prisutna i osvježena:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Backend nije uspio pokrenuti

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Zapisnici pozadinskog sustava za neuspješan pokret su u `~/.cache/chloros/logs/`.

### CUDA nije otkrivena

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` izvještava o istoj stvari u jednoj liniji: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Nedostajuće zajedničke biblioteke

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Sporo pokretanje na sustavima sa SD-karticom

Kompilirane binarne datoteke raspakuju se u privremeni direktorij pri svakom pokretanju. Ako `/mnt/ssd/tmp` postoji, Chloros ga automatski koristi; u suprotnom postavite `TMPDIR` na brzi datotečni sustav:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Ažuriranje Chloros na Linux

Naredba `update` namijenjena je samo za Linux /Jetson. Provjerava verziju objavljenu u kanalu ažuriranja konfiguriranom na `/etc/chloros/update.conf` i nudi preuzimanje i instalaciju odgovarajućeg `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Na Linux/Jetson, CLI također provodi neblokirajuću provjeru ažuriranja pri svakom pokretanju (rezultat je pohranjen u predmemoriji u `~/.chloros/update_cache.json` na jedan sat) i ispisuje `Update available: vX.Y.Z` kada postoji novija verzija. Vaše postavke i projekti ostaju sačuvani nakon ažuriranja; nakon toga ćete se morati ponovno prijaviti.

## Deinstalacija

```bash
sudo apt remove chloros
```

Uklanjanje zaustavlja `chloros-backend.service`, vraća zadani prag ne privilegiranog porta (1024), uklanja priloženi symlink exiftool i konfiguraciju Arena loadera te briše pohranjene vjerodajnice. Vaši projekti i datoteke s podacima `~/.chloros/` ostaju netaknuti.

***

## Sljedeći koraci

* [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) — optimizacija i implementacija specifična za Jetson
* [CLI : Naredbeni redak](../CLI.md) — vodič za CLI
* [API : Python SDK](../api-python-sdk.md) — vodič za SDK
* [Referenca za CLI](../reference/cli-reference.md) i [Referenca za SDK](../reference/sdk-reference.md) — iscrpne liste naredbi/API za verziju 1.2.0
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagođava vašem hardveru
