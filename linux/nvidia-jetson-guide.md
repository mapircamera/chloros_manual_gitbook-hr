# Vodič za NVIDIA Jetson

Chloros na NVIDIA Jetson omogućuje multispektralnu obradu slike na rubu — na terenu, na UAV-ovima i u udaljenim instalacijama. Chloros automatski otkriva vaš Jetson model i optimizira njegovu strategiju obrade za vaš hardver.

***

## Podržani modeli Jetson

| Model | RAM | Strategija obrade | Preporučena upotreba |
| -------------------- | -------------- | ----------------------------------------------------- | ---------------------------------------------------------- |
| **Jetson AGX Orin** | 32-64GB dijeljeno | `GPU_PARALLEL` (4 radnika) | Maksimalna izvedba, veliki skupovi podataka |
| **Jetson Orin NX** | 8-16 GB dijeljeno | `GPU_PARALLEL` (3 radnika, 16GB) / `GPU_SINGLE` (8GB) | Primarna preporuka za razmještaj u zraku i na terenu |
| **Jetson Orin Nano** | 8 GB podijeljeno | `GPU_SINGLE` (1 radnik) | Rubno računanje početne razine |
| **Jetson Nano** | 4-8 GB dijeljeno | `GPU_SINGLE` (1 radnik) | Početna razina, ograničena memorija |

{% hint style="info" %}
**Naslijeđeni Jetson modeli** (TX2, TX1, Xavier NX) možda neće biti podržani. Performanse će se razlikovati ovisno o dostupnoj GPU memoriji i CUDA mogućnostima.
{% endhint %}

***

## Zahtjevi

* **JetPack 6.x** (preporučeno najnovije)
* **NVIDIA CUDA** (uključeno u JetPack)
* **Licenca Chloros+** (potrebna za pristup CLI/SDK)

## Instalacija

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Za općenite detalje instalacije Linux pogledajte [Instalacija Linux](linux-installation.md).

***

## Dynamic Compute Adaptation na Jetsonu

Chloros automatski otkriva vaš Jetson model i odabire optimalnu strategiju obrade. **Nije potrebno ručno podešavanje.**

### Kako to radi

Prilikom pokretanja Chloros profilira vaš sustav:

1. **Detektuje model Jetson** putem `/proc/device-tree/model`
2. **Čita dostupnu GPU/dijeljenu memoriju**

3.**Odabire strategiju obrade** (`GPU_PARALLEL`, `GPU_SINGLE` ili `CPU_PARALLEL`)
4. **Automatski postavlja broj radnika, vrstu cjevovoda i dodjelu memorije**

### Ponašanje po modelu

| Model Jetson | Strategija | Radnici | Cjevovod | Podudarnost |
| ---------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (učinkovita memorija) | Serijalizirano |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Serijalizirano |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 2 | `tiled_gpu` | Serijalizirano |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (puni GPU put) | Istodobno |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4 | `fused_gpu` | Istodobno |

{% hint style="success" %}
**Jetson Orin NX 16GB** je slatka točka za rubnu implementaciju — dobiva strategiju `GPU_PARALLEL` s 3 istovremena radnika, isporučujući pravu paralelnu GPU obradu u kompaktnom obliku.
{% endhint %}

Ključna razlika između platformi je **memorija**. Jetson Nano s 8 GB zajedničke memorije mora obrađivati ​​slike jednu po jednu korištenjem memorijsko učinkovitog popločanog pristupa, dok Orin NX sa 16 GB može pokrenuti 3 slike kroz GPU istovremeno koristeći spojeni cjevovod veće propusnosti.

Za potpunu referencu računalne prilagodbe pogledajte [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Upravljanje toplinom

Jetson uređaji imaju ograničen toplinski prostor, posebno u zatvorenim ili zračnim razmještajima. Chloros uključuje automatski toplinski nadzor i prigušivanje:

| Temperatura | Radnja |
| ------------------ | -------------------------------------------------- |
| **< 70°C** | Normalan rad — puna brzina obrade |
| **70°C** (Upozorenje) | Automatski smanji veličinu serije |
| **80°C** (kritično) | Agresivno prigušivanje — niža konkurentnost |
| **90°C** (isključivanje) | Potpuno zaustavi GPU obradu — potrebno je hlađenje |

{% hint style="warning" %}
**Osigurajte odgovarajuću ventilaciju i odvod topline** za kontinuiranu obradu, posebno u zatvorenim terenskim prostorima ili sustavima u zraku. Toplinsko prigušivanje će smanjiti propusnost obrade radi zaštite hardvera.
{% endhint %}

***

## Upravljanje memorijom

Jetson uređaji koriste **objedinjenu memoriju** — GPU i CPU dijele isti fizički RAM. To znači da navedeni VRAM (npr. 15,3 GB na Orin NX 16 GB) nije namjenska GPU memorija; dijeli se s operativnim sustavom i drugim procesima.

### Preporuke za razmjenu

Za velike skupove podataka ili debayer obradu s obzirom na teksturu, Chloros može preporučiti stvaranje swap prostora:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Procjene memorije po slici:**

* Standardni debayer: \~10 MB po slici
* Debayer s obzirom na teksturu: \~15 MB po slici

Chloros automatski izračunava potrebnu memoriju na temelju veličine vašeg skupa podataka i upozorava vas ako se preporučuje zamjena.

### OOM (Out of Memory) zamjena

Ako se tijekom obrade otkrije stanje nedostatka memorije:

1. Chloros automatski smanjuje broj GPU radnika
2. Vraća se s `fused_gpu` na `tiled_gpu` cjevovod (memorijski učinkovitiji)
3. Nastavlja obradu uz smanjenu propusnost umjesto pada

***

## Primjena na terenu

### Razmatranja napajanja

| Model Jetson | Tipična snaga | Bilješke |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano | 5-10W | USB-C ili bačvasti priključak |
| Jetson Orin Nano | 7-15W | DC bačvasti priključak |
| Jetson Orin NX | 10-25W | DC bačvasti priključak |
| Jetson AGX Orin | 15-60W | USB-C PD ili bačvasti priključak |

Planirajte svoj proračun energije za održivu obradu — vršna potrošnja energije događa se tijekom GPU-intenzivne niti 3 (obrada).

### Preporuke za pohranu

* **NVMe SSD** se toplo preporučuje za arm64 implementacije
* SD kartice su prespore za obradu — koristite samo kao medij za pokretanje
* Planirajte 2-3 puta veću veličinu neobrađenih slikovnih podataka za obrađeni izlaz

### Rad bez glave putem SSH

Chloros CLI idealan je za bezglave Jetsonove primjene:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatizirana obrada sa systemd

Stvorite systemd uslugu za automatiziranu obradu:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Uparite sa timerom systemd za planiranu obradu:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Primjer tijeka rada

### Osnovna Jetson obrada

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK na Jetsonu

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Skupna obrada više letova

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Preporučeni sustavi Jetson za upotrebu na terenu

Za implementacije na terenu i u zraku, razmotrite ove opcije nosive ploče Jetson Orin NX 16GB:

* **Airborne/drone**: sustavi s ocjenom vibracija (MIL-STD), lagani (ispod 300 g), pasivno hlađenje
* **Čvrsto polje**: IP67/IP69K vodootporna kućišta s PoE GigE povezivanjem kamere
* **Minimalni/proračun**: Kompleti za razvojne programere s dodatnim kućištima

Obratite se [podršci za MAPIR](https://www.mapir.camera/community/contact) za posebne hardverske preporuke za vaš scenarij implementacije.

***

## Sljedeći koraci

* [Instalacija Linux](linux-installation.md) — Opći detalji instalacije Linux
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Potpuna referenca računalne strategije
* [Cjevovod za obradu](../processing-architecture/processing-pipeline.md) — Razumijevanje cjevovoda s 4 niti
* [CLI : naredbeni redak](../CLI.md) — Puna referenca CLI
* [API : Python SDK](../api-python-sdk.md) — Potpuna referenca SDK