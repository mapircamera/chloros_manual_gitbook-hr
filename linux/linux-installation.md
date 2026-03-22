# Linux Instalacija

Chloros se distribuira za Linux kao paketi `.deb` koji instaliraju CLI i pozadinu. Python SDK se instalira zasebno putem pip-a.

***

## Linux amd64 (x86_64)

### Zahtjevi sustava

| Zahtjev | Minimalno | Preporučeno |
| --- | --- | --- |
| **Distribucija** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 ili bolji |
| **Memorija (RAM)** | 8 GB | 16 GB ili više |
| **Grafička kartica** | Ništa (CPU obrada) | NVIDIA GPU s 4 GB+ VRAM |
| **Skladištenje** | 2 GB slobodnog prostora | SSD s 10 GB+ slobodnog prostora |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Instalacija

Preuzmite paket `.deb` i instalirajte:

```bash
sudo dpkg -i chloros-amd64.deb
```

Provjerite instalaciju:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Zahtjevi sustava

| Zahtjev | Minimalno | Preporučeno |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16GB ili AGX Orin |
| **JetPack** | JetPack 6.x | Najnoviji JetPack 6 |
| **Memorija (RAM)** | 8 GB (dijeljeni GPU/CPU) | 16GB+ podijeljeno |
| **Skladištenje** | 2 GB slobodnog prostora | NVMe SSD s 10 GB+ besplatno |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

### Instalacija

Preuzmite paket JetPack 6 `.deb` i instalirajte:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Provjerite instalaciju:

```bash
chloros-cli --version
```

Za detaljnu postavku Jetsona, uključujući upravljanje toplinom i implementaciju na terenu, pogledajte [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Instalacija (svi Linux)

Python SDK se instalira zasebno putem pip-a i radi i na amd64 i na arm64:

```bash
pip install chloros-sdk
```

Za uključivanje dodatne podrške za strujanje napretka:

```bash
pip install chloros-sdk[progress]
```

Provjerite SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Paket `.deb` instalira Chloros CLI i pozadinu. Python SDK je zasebni pip paket koji komunicira s pozadinom putem lokalnog HTTP API.
{% endhint %}

***

## Konfiguracijski direktoriji

Chloros na Linux slijedi [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Svrha | Linux Put | Windows Ekvivalent |
| --- | --- | --- |
| **Konfiguracija** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Podaci / Projekti** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Predmemorija / vjerodajnice** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Pozadinske izvršne lokacije

Paket `.deb` instalira pozadinu na standardnu ​​lokaciju. CLI i SDK automatski otkrivaju pozadinsku putanju:

| Način instalacije | Pozadinski put |
| --- | --- |
| `.deb` paket | `/usr/lib/chloros/chloros-backend` |
| Ručno / prilagođeno | `/opt/mapir/chloros/backend/chloros-backend` |

Možete nadjačati pozadinsku stazu s `--backend-exe` CLI zastavom ili `backend_exe` SDK parametrom konstruktora.

***

## Prvo postavljanje

### 1. Aktivirajte svoju licencu

Za pristup CLI i SDK potrebna je licenca Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Provjerite status svoje licence

```bash
chloros-cli status
```

### 3. Obradite svoj prvi skup podataka

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Pokrenite dijagnostiku sustava

Provjerite je li vaš sustav ispravno konfiguriran:

```bash
chloros-cli selftest
```

Ovo pokreće 7 dijagnostičkih provjera uključujući verziju, pokretanje pozadine, API povezivost i dostupnost CUDA/GPU-a.

***

## Bash skriptni primjeri

### Obrada više skupova podataka

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Proces s prilagođenim postavkama

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatizirana obrada s Cronom

Dodajte u svoj crontab (`crontab -e`) za automatsku obradu novih skupova podataka:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Primjer

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

Ako `chloros-cli` nije pronađen nakon instaliranja paketa `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Dopuštenje odbijeno

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Nije uspjelo pokretanje pozadine

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA nije otkrivena

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Nedostaju dijeljene biblioteke

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Ažuriranje Chloros na Linux

Upotrijebite ugrađenu naredbu ažuriranja za provjeru i instaliranje ažuriranja:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Sljedeći koraci

* [Vodič za NVIDIA Jetson] (nvidia-jetson-guide.md) — optimizacija i implementacija specifična za Jetson
* [CLI: naredbeni redak](../CLI.md) — Puna referenca naredbe CLI
* [API : Python SDK](../api-python-sdk.md) — Potpuna referenca SDK
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Kako se Chloros prilagođava vašem hardveru