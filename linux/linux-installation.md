# Instalacija Linuxa

Chloros distribuira se za Linux kao `.deb` paketi koji instaliraju CLI, pozadinski poslužitelj i Python SDK (za sustav `python3`, plus wheel paket usklađene verzije za virtualenvs).

Nazivi datoteka paketa nose verziju i arhitekturu: `chloros_1.3.0_amd64.deb` za x86_64, i `chloros_1.3.0_arm64_jp6.deb` za arm64 (NVIDIA Jetson, JetPack 6). U naredbe u nastavku umetnite datoteku koju ste zapravo preuzeli.

{% hint style="info" %}
**Novo u verziji 1.3.0: programi se isporučuju raspakirani.** Backend i CLI instaliraju se kao mape, `/usr/lib/chloros/backend/` i `/usr/lib/chloros/cli/`, umjesto pojedinačnih samoraspakirajućih datoteka. Više se ne raspakuju u privremeni direktorij pri svakom pokretanju, pa se pokreću mnogo brže (na x86_64 pozadini je ranije trebalo oko 100 sekundi za raspakiravanje pri svakom pokretanju) i više ne zahtijevaju nekoliko GB slobodnog prostora u `/tmp`. amd64 `.deb` je također otprilike pola svoje prijašnje veličine.
{% endhint %}

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

> **Ubuntu 20.04 i Debian 11 nisu podržani.** Popis ovisnosti `.deb`-a izveden je iz onoga s čime se Chloros backend zapravo povezuje, a to uključuje
> `libc6` noviji od glibc 2.31 koji dolazi s focal i bullseye, stoga `apt` instalaciju
> potpuno odbacuje umjesto da dopusti da kasnije tijekom izvođenja dođe do pogreške.

### Instalacija

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Koristite `apt install ./…`, a ne `dpkg -i`.** `apt` rješava ovisnosti paketa i također instalira ono što preporučuje: zahtjeve za PythonSDK, `python3-numpy` i `python3-requests`. `dpkg -i` ne instalira nijedno od njih. CLI i backend su samostalni i rade bez njih, ali `import chloros_sdk` ih treba; izlaz instalacije navodi sve koji nedostaju. Sačuvajte `./`, tako da `apt` instalira datoteku umjesto da traži paket s tim imenom. Paket također deklarira `libgl1`, koji ugrađeni OpenCV učitava, a minimalnoj poslužiteljskoj slici može nedostajati.
{% endhint %}

Provjerite instalaciju:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Sustavni zahtjevi

| Zahtjev | Minimalno | Preporučeno |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPackom 6 | Jetson Orin NX 16GB ili AGX Orin |
| **JetPack** | JetPack 6.x | Najnoviji JetPack 6 |
| **Memorija (RAM)** | 8 GB (zajednička GPU/CPU) | 16 GB+ zajedničke (12 GB+ je prag za paralelnu obradu) |
| **Pohrana** | 2 GB slobodnog prostora | NVMe SSD s 10 GB+ slobodnog prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Instalacija

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ista konfiguracija kao i amd64 `.deb` (uključeni Python SDK, primajuće spremnike i pozadinske usluge), za Jetson Orin / Orin NX / Orin Nano na **JetPacku 6** (L4T R36). Njegov PyTorch je CUDA izrada za Orin iz JetPacka 6, tako da se obrada na GPU-u, uključujući i Texture Aware debayer, odvija na GPU-u Jetsona. Paket ne sadrži CUDA biblioteku: on koristi CUDA i cuDNN biblioteke koje instalira JetPack 6 (`sudo apt install nvidia-jetpack` na slici na kojoj one nedostaju). Gdje se ne može koristiti CUDA GPU, obrada se odvija na CPU-u, a `--debayer texture-aware` koristi standardni debayer (backend bilježi jednu `[DEBAYER] … needs a CUDA GPU …` liniju koja navodi razlog).. Za informacije o memoriji, toplinskim karakteristikama i ponašanju pri terenskoj implementaciji Jetsona pogledajte [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md).***

## Instalacija Python SDKa (Svi Linuxi)**`.deb` instalira SDK za vas.** Oba paketa (amd64 i Jetson) instaliraju ga za sustav `python3`, tako da `import chloros_sdk` radi odmah nakon instalacije, bez pip-a i bez mreže. Njegovi paketi (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` i projektni učitavači) idu u `/usr/lib/python3/dist-packages`, u vlasništvu paketa: nadogradnja ih zamjenjuje, a uklanjanje paketa ih uklanja. Verzija uvijek odgovara CLIu i backendu koje ste instalirali.

Zahtjevi samog SDKa, `python3-numpy` i `python3-requests`, **preporučeni su, a ne obavezni** od strane paketa: `sudo apt install ./chloros_<version>_<arch>.deb` ih instalira, XPROT000073 ne. Izlaz instalacije izvještava o verziji SDKa koju `python3` uvozi, o bilo kojem zahtjevu koji nedostaje, i svaki `chloros_sdk` drugdje na `python3`-ovoj putanji koji zasjenjuje kopiju paketa (kao što je onaj ranije instaliran pomoću `pip install --user`).**U virtualenvu**, koji ne vidi sistemske pakete, instalirajte priloženi wheel u njega (potpuno izvan mreže):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**S PyPI-ja** — samo za računalo na kojem nije instaliran paket Chloros, čiji SDK pokreće pozadinski sustav (backend) na drugom računalu (vidi [Način daljinskog pozadinskog sustava](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Dodatne opcije, za pip instalaciju (virtualenv ili PyPI):

| Dodatak | Naredba | Dodaje |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` za praćenje napretka uživo |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` za BLE (DAQ-M) transport |

Provjerite SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pythonov SDK komunicira s pozadinskim sustavom koji `.deb` instalira putem lokalnog HTTPovog APIa (`http://127.0.0.1:5000`). `ChlorosLocal` i pomoćni programi za povezivanje automatski ga pokreću kada je potrebno; `connect_all()` spremljenog projekta to nikada ne čini, na bezglavom računalu pokrenite pozadinsku uslugu kao [uslugu](#always-on-ptp-for-headless-hosts). Uvijek koristite doslovnu IPv4 adresu umjesto `localhost` — `localhost` može se razriješiti na `::1` i koštati otprilike dvije sekunde po zahtjevu.
{% endhint %}

***

## Prvo postavljanje

### 1. Prijava

Pristup CLIu i SDKu zahtijeva plaćeni Chloros+ paket (**Copper** ili viši), što se provjerava na strani poslužitelja: korisnik bez prijave dobiva `401 AUTH_REQUIRED`, a korisnik besplatnog paketa (Iron) dobiva `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Vjerodajnice se pohranjuju u `~/.chloros/user_session.json`.

{% hint style="info" %}
**Nadogradnje zadržavaju vašu prijavu; uklanjanje je briše.** Instaliranje novije verzije `.deb` preko postojeće instalacije (uključujući putem `chloros-cli update`) zadržava `~/.chloros/user_session.json` i keširanu licencu, tako da bezglave CLI/SDK operacije nastavljaju raditi. Deinstaliranje paketa (`sudo apt remove chloros`) briše spremljene podatke o prijavi i predmemoriju licence za svakog korisnika na računalu, stoga nova instalacija ponovno zahtijeva `chloros-cli login`.
{% endhint %}

### 2. Provjerite status svoje licence

```bash
chloros-cli status
```

`chloros-cli status` radi na bilo kojoj razini (uključujući besplatnu), tako da uvijek možete vidjeti zašto je pristup dostupan ili nedostupan.

### 3. Pokrenite dijagnostiku sustava

```bash
chloros-cli selftest
```

Pokreće se sedam provjera redom, a naredba izlazi s ne-nuljevnim rezultatom ako ijedna od njih ne uspije:

| # | Provjera | Što dokazuje |
| --- | --- | --- |
| 1 | **Verzija** | CLI izvještava o svojoj verziji (`v1.3.0`). |
| 2 | **Dostupnost porta** | Port 5000 je slobodan, *ili* na njega već odgovara ispravan pozadinski sustav Chloros (što se računa kao prolaz). |
| 3 | **Pokretanje pozadinskog sustava** | Binarna datoteka pozadinskog sustava se pokreće. |
| 4 | **Test APIa (`/api/test`)** | Backend odgovara `status: ok`. |
| 5 | **Informacije o sustavu** | Ispisuje `GPU: <name>, CUDA: <bool>, PyTorch: <version>` iz `/api/system-info`. |
| 6 | **Modeli Denoisera** | Pronalazi modele `*.pth.enc` (na Linuxu: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware je zapravo upotrebljiv — potreban je CUDA**i** najmanje jedna modelna datoteka. |

Pokretanje završava `N/7 checks passed`, navodeći sve pogreške po nazivu.

### 4. Obradite svoj prvi skup podataka

```bash
chloros-cli process ~/datasets/flight001
```

***

## Datoteke i direktoriji

### Za svakog korisnika,

Chloros pohranjuje svoje vjerodajnice i konfiguraciju CLI u jednom međuplatformskom direktoriju, **`~/.chloros/`** (na Windows, `%USERPROFILE%\.chloros\`). Dva predmemorijska prostora specifična za Linuxprate XDG konvencije — oni poštuju `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` kada su postavljeni.

| Putanja | Svrha |
| --- | --- |
| `~/.chloros/user_session.json` | Keš sesije prijave koji stvara `chloros-cli login` (zadržava se tijekom nadogradnji paketa; briše se uklanjanjem paketa) |
| `~/.chloros/working_directory.txt` | Preusmjeravanje zadane mape projekta (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferencija jezika za CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Postavka jezika zajednička s GUI-jem sustava Windows — `language` ovdje ima prednost nad `cli_language.json` |
| `~/.chloros/update_cache.json` | Jednosatna predmemorija za provjeru ažuriranja pokretanja za Linux/Jetson |
| `~/.chloros/backend.log` | Zapisnik pozadinskog sustava (backend) kada je pokrenut putem CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pohranjeni paketi kalibracije LATTICE za svaku kameru, indeksirani po serijskom broju i hašu paketa |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Neobavezne korisničke nadjačavanja profila za korekciju kapaciteta DAQ-a |
| `~/.config/chloros/system_config.json` | Pohranjeni hardverski profil iz sustava Dynamic Compute Adaptation — izbrišite ga za prisilnu novu detekciju hardvera |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Zapisnici back-end poslužitelja, jedna datoteka po pokretanju |
| `~/Documents/MAPIR/Chloros Projects/` | Zadana mapa projekta kada nije postavljena drugačija |

### Za cijeli sustav

| Putanja | Svrha |
| --- | --- |
| `/usr/bin/chloros-cli` | Omotna skripta — postavlja `LD_LIBRARY_PATH` za priložene nativne biblioteke (i `TMPDIR` na `/mnt/ssd/tmp` kada ta mapa postoji), a zatim izvršava stvarnu binarnu datoteku |
| `/usr/bin/chloros-backend` | Omotna skripta — isto za pozadinu, koju označava kao instaliranu produkcijsku izgradnju |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | KompajliraniCLIi i backend, instalirani raspakirani (novo u 1.3.0); programi su `cli/chloros-cli` i `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Skripte pokretača na dvije linije koje pokreću stvarne programe iznad — pohranjene na ovim putanjama jer ih SDK, CLI i starije skripte traže ovdje |
| `/usr/lib/chloros/arena_runtime/` | Runtime za Arena SDK potreban za LATTICE kamere |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrirani modeli za uklanjanje šuma koje koristi debayer osjetljiv na teksture |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (i `lattice_sdk`, `lattice_cam`, `arena_api`, projektni učitavači) | Python SDK, instaliran za sustav `python3` i pripadajući paketu |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel koji odgovara ovoj točnoj izradi, za virtualenv |
| `/usr/lib/chloros/exiftool` | Ugrađeni pokretač ExifToola (simbolički povezan s `/usr/local/bin/exiftool` samo ako ne postoji sustavni exiftool). Od verzije 1.3.0 skripta ExifTool isporučuje se sa svojom Perl bibliotekom u `/usr/lib/chloros/exiftool-dist/`, pa radi bez sustavnog paketa `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfiguracija ažuriranja kanala koju čita `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Postavlja `net.ipv4.ip_unprivileged_port_start = 319` kako bi se pozadinski sustav mogao povezati na PTP priključke bez root prava |
| `/etc/sysctl.d/60-chloros-gige.conf` | Povećava buffer za primanje soketa u kernelu (`net.core.rmem_max`, `net.core.rmem_default`) na 32 MB za LATTICE kamerne streamove i primjenjuje ih pri instalaciji (vrijednost koja je već postavljena na višu razinu ostaje nepromijenjena). Uz zadane postavke jezgre, paketi kamere se gube u jezgri i niz gubi sličice i brzinu sličica; `chloros-cli lattice network` izvještava o spremnicima kada su niski. Vidi [Linux hosts — soketni prijemni spremnici](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Usmjerava dinamički učitavač na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Odobrava prijavljenom korisniku pristup DAQ-U USB serijskom mostu (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Usluga uvijek aktivnog pozadinskog sustava za uključivanje koja pokreće pozadinski sustav kao korisnik kojeg odaberete (instalirana, **nije omogućena**) — onu koju treba omogućiti, pogledajte [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Ista usluga koja pokreće pozadinsku uslugu kao **root**(instalirana,**nije omogućena**), zadržana kako bi računalo na kojem je omogućena nastavilo raditi |
| `/var/log/chloros/` | Detaljni zapisnik usluge root, čitljiv za grupu `adm` (izbrisana kada se paket ukloni) |
| `/usr/share/applications/chloros-cli.desktop` | stavka u izborniku aplikacije &quot;Chloros CLI&quot; koja otvara terminala |

**Firewall (novo u verziji 1.3.0).** Ako su `ufw` ili `firewalld` aktivni tijekom instalacije, paket otvara UDP priključke `5002` (DAQ-E podatke), `5353` (otkrivanje mDNS-a), `319` i `320` (PTP) te uklanja ta pravila kada se paket ukloni. Ako je vatrozid instaliran, ali nije aktivan, ne dodaje ništa i ispisuje priključke koje treba dopustiti ako ga kasnije aktivirate. Ne otvara nijedan TCP priključak: ažuriranje firmvera DAQ-E, pri čemu se senzor ponovno povezuje s ovim računalom putem TCP-a kako bi preuzeo sliku, ne uspijeva s `… device never connected back on TCP <port> …` dok aktivan vatrozid blokira to povezivanje.

## Lokacija izvršne datoteke pozadinskog procesa

CLI i SDK automatski otkrivaju pozadinski proces:

| Komponenta | Putanja |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (pokretač za `/usr/lib/chloros/backend/chloros-backend`) |

Prepišite putanju do backenda pomoću zastavice `--backend-exe` CLI ili parametra konstruktora `backend_exe` SDK, i port s `--port` (zadano `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` usmjerava **`lattice`**,**`project`**i obitelji naredbi**`daq pool-*`** na udaljenom pozadinskom sustavu. Osnovne naredbe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) namjerno ga zanemaruju i uvijek ciljaju `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE kamere i DAQ senzori svjetla na Linuxu

Sve obitelji naredbi za hardver (live-hardware) rade na Linuxu (amd64 i Jetson):

* **`chloros-cli lattice`** — otkrijte, povežite, konfigurirajte i snimajte s LATTICE kamera i sinkroniziranih nizova. `.deb` uključuje *SDK* okruženje za izvođenje Arena koje je potrebno i registrira ga kod dinamičkog učitavača.
* **`chloros-cli daq pool-*`** — povezati DAQ-U/M/E senzore svjetla putem pozadinskog pool-a, streamati kalibrirane spektre, snimati `.daq` datoteke i mijenjati profile kapa, a za DAQ-E promijeniti njegove mrežne postavke, ponovno ga pokrenuti i ažurirati njegov firmware. `chloros-cli daq discover` skenira senzore koje možete povezati, a `chloros-cli daq --help` popisuje podzapovijedi `pool-*`.
* **`chloros-cli project`** — pokrenuti spremljeni projekt (njegove kamere, senzore i postavke obrade) bez zaslonskog sučelja.
* **`chloros-cli time-sync`** — provjerite PTP grandmaster koji pozadinski program Chloros pokreće za LATTICE kamere i DAQ-E senzore.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je potreban za `pool-latest`, `pool-stream`, `pool-record` i `pool-set-cap`; `pool-list` prikazuje ID-ove koji su trenutačno u skupu.

{% hint style="info" %}
**`--eth-host` preskače otkrivanje.** `pool-connect --eth` pronalazi DAQ-E putem mDNS-a, s rezervnim ARP-om koji radi s praznom ARP predmemorijom; prosljeđivanje IP adrese ili naziva hosta senzora pomoću `--eth-host` omogućuje izravno povezivanje s njim.
{% endhint %}

**Serijske dozvole za DAQ-U** obrađuje instalirano udev pravilo (`uaccess` + grupa `dialout`). Ako senzor koji je već priključen ostane nedostupan, ponovno učitajte pravila ili ga ponovno priključite:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Pogledajte [CLIreferencu](../CLI.md) za cijeli skup naredbi.

### Uvijek aktivan PTP za hostove bez zaslona

Paket instalira dvije systemd usluge za pozadinski sustav (backend), od kojih nijedna nije omogućena. Na Jetsonu bez zaslona ili poslužitelju na kojem bi PTP sinkronizacija vremena trebala neprekidno raditi za DAQ-E senzore i LATTICE kamere, ili gdje skripte SDK-a povezuju spremljene projekte (`connect_all()` nikada ne pokreće pozadinsku uslugu), omogućite uslugu **kao svoj korisnik**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` pokreće backend kao taj korisnik: snimke koje piše pripadaju njima, njegov detaljni zapisnik nalazi se u njihovom `~/.cache/chloros/logs`, i koristi njihov `chloros-cli login` (`~/.chloros`), stoga se prvo jednom prijavite kao taj korisnik. Instalacija paketa ispisuje ovu naredbu s vašim korisničkim imenom.

Bez usluge, pozadinski sustav (i s njim PTP grandmaster) radi samo dok ga nešto nije pokrenulo — naredba `chloros-cli` ili skripta iz SDKa.

Jedinica povezuje pozadinski sustav s `127.0.0.1:5000` (postavke okruženja `CHLOROS_HOST` / `CHLOROS_PORT` unutar jedinice; poništite pomoću `sudo systemctl edit chloros-backend@.service`) i ponovno ga pokreće nakon neuspjeha nakon 5 sekundi.

**Root servis.**`chloros-backend.service` pokreće backend kao**root**. I dalje je instaliran tako da računalo na kojem je omogućen nastavlja raditi, ali sve što root backend zapiše pripada rootu, pa korisnik ne može izbrisati vlastite snimke; njegov dnevnik nalazi se u `/var/log/chloros`, čitljiv je za grupu `adm`. Za prebacivanje na svoj korisnički račun:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend usluge je zajednički, klijent ga nikada ne zaustavlja.** `chloros-cli` i skripte SDK koriste ga onako kako ga zateknu: `process` radi na njemu, učitavajući vlastiti projekt umjesto posljednjeg. Gdje bi CLI inače ponovno pokrenuo pozadinski servis (`--restart` ili `process` čije uvoze nije pronašlo ništa), naziva jedinicu, ispisuje `sudo systemctl restart <unit>` i izlazi iz `1`, jer bi se backend zaustavio i ostao neaktivan. Ponovno ga pokrenite sami pomoću `sudo systemctl restart chloros-backend@$USER.service`. Backend usluge nikada se ne zaustavlja sam pri neaktivnosti, a `chloros-cli backend stop` to odbija, nazivajući naredbu `systemctl stop`.**Kako PTP dobiva svoje priključke.** PTP koristi UDP 319/320, oba ispod uobičajene donje granice od 1024 privilegirana priključka. Paket `postinst` piše `/etc/sysctl.d/60-chloros-ptp.conf` s `net.ipv4.ip_unprivileged_port_start = 319`, što omogućuje backendu da ih veže dok se pokreće kao vaš korisnik. Od verzije 1.3.0 paket ne postavlja sposobnosti datoteka (`setcap`) na pozadinskom procesu, a `libcap2-bin` više nije ovisnost: na raspakiranom pozadinskom procesu sposobnost datoteke spriječila bi njegovo pokretanje, stoga nemojte sami dodavati takvu mogućnost.***

## Primjeri Bash skriptiranja

{% hint style="info" %}
**Kodovi izlaska prilagođeni skriptama.**`chloros-cli process` izlazi iz `0` kod uspjeha i**ne-nula kod neuspjeha — uključujući pokretanje koje je zatražilo slikovne proizvode, ali nije napisalo nijedan** (ispisuje `Processing finished but wrote no image products.` i naziv projektne mape te uobičajene uzroke). Uspješni pokretanja izvještavaju o tome koliko je slikovnih proizvoda napisano (`Image products written: N`, plus `(plus M clamp masks)` kada su napisane maske). Kodi izlaska: `0` uspjeh, `1` neuspjeh, `2` pogreška u argumentu, `130` prekinuto.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Valjane vrijednosti `--format` su točno četiri i sadrže razmake — uvijek ih navedite u navodnicima:

| `--format` vrijednost | Izlazna mapa |
| --- | --- |
| `TIFF (16-bit)` *(zadano)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` prihvaća `standard` (po zadanom) ili `texture-aware` (Chloros+).

### Automatska obrada pomoću Cron-a

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Primjer Python SDK-a

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Rješavanje problema

### CLI se ne može pronaći nakon instalacije

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend se neće pokrenuti nakon dodavanja mogućnosti datoteka

Nemojte pokretati `setcap` na backendu. Od verzije 1.3.0 paket ne postavlja nikakve mogućnosti datoteka, a PTP-ovi portovi 319/320 dolaze iz postavke `sysctl` opisane pod [Kako PTP dobiva svoje portove](#always-on-ptp-for-headless-hosts). Datotečna sposobnost stavlja neupakirani backend u način sigurnog izvođenja u kojem ne može pronaći vlastite biblioteke, pa se ne pokreće. Ako ste je dodali, uklonite je:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP se neće pokrenuti / ne može vezati priključak 319

Potvrdite je li prag za neovlaštene priključke snžen i ponovno ga primijenite za trenutno pokretanje ako nije:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Zatim provjerite grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Naredbe `lattice` ne uspijevaju s porukom &quot;Nije pronađen upravljački program za LATTICE kameru&quot;

Backend nije uspio učitati runtime kamere LATTICE s kojim je instaliran. Svaka `lattice` naredba izvršava se na backendu; sama CLI ne sadrži upravljačke programe za kamere. Reinstalirajte Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` instalira runtime uz backend, u `/usr/lib/chloros/arena_runtime`, i registrira ga kod dinamičkog učitavača. Za provjeru te registracije:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Zapisnici backenda za neuspješan pokretanje nalaze se u `~/.cache/chloros/logs/`. Ako port 5000 pripada usluzi backenda (`systemctl status chloros-backend@$USER.service`), ponovno pokrenite uslugu umjesto da prekidate njezin proces.

### CUDA nije otkrivena

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` izvještava o istoj stvari u jednoj liniji: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Nedostajuće zajedničke biblioteke

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Sporo pokretanje na sustavima sa SD-karticom

Od verzije 1.3.0, pozadinska usluga i CLI instaliraju se neotpakirani, stoga se više ne raspakuju u privremeni direktorij pri svakom pokretanju. Privremeni se datoteci automatski spremaju u `/mnt/ssd/tmp` ako taj direktorij postoji; inače im možete odrediti lokaciju na brzom datotečnom sustavu:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Ažuriranje Chlorosa na Linux

Naredba `update` dostupna je samo za Linux/Jetson. Provjerava verziju objavljenu u kanalu ažuriranja konfiguriranom na `/etc/chloros/update.conf` i nudi preuzimanje i instalaciju odgovarajućeg `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Na Linux/Jetson CLI također provodi neblokirajuću provjeru ažuriranja pri svakom pokretanju (rezultat se pohranjuje u predmemoriju na jedan sat u `~/.chloros/update_cache.json`) i ispisuje `Update available: vX.Y.Z` kada postoji novija verzija. Vaši se postavke, projekti i prijava zadržavaju nakon ažuriranja.

Nadogradnja paketa zaustavlja svaku pozadinsku uslugu tijekom instalacije, pa pokrenuta skripta gubi pristup kamerama i senzorima, a zatim ponovno pokreće svaku uslugu koja je omogućena (ona koja je ručno pokrenuta, ali nikada omogućena, ostaje zaustavljena, kao i nakon ponovnog pokretanja).

## Deinstalacija

```bash
sudo apt remove chloros
```

Deinstalacija zaustavlja i onemogućuje svaku pozadinsku uslugu, vraća zadano ograničenje na ne privilegirane priključke (1024), uklanja datoteku s postavkama za prijemni spremnik (trenutne vrijednosti ostaju do ponovnog pokretanja), simlink (skraćeni put) exiftool-a koji je priložen, konfiguraciju Arena učitavača, pravila vatrozida koja je paket dodao i Python SDK koji je instalirao, te briše keširane vjerodajnice. Vaši projekti i datoteke s podacima `~/.chloros/` ostaju netaknuti. `sudo apt purge chloros` također briše `/var/log/chloros`.

***

## Sljedeći koraci

* [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) — Optimizacija i implementacija specifična za Jetson
* [CLI : Command Line](../CLI.md) — vodič za CLI
* [API : Python SDK](../api-python-sdk.md) — vodič za SDK
* [Referenca za CLI](../reference/cli-reference.md) i [Referenca za SDK](../reference/sdk-reference.md) — iscrpne liste naredbi/API za verziju 1.3.0
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagođava vašem hardveru
