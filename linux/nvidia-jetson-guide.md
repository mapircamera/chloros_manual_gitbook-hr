# NVIDIA Jetson vodič

Chloros

za NVIDIA Jetson omogućuje obradu višespektralnih slika na rubu mreže — na terenu, na bespilotnim letjelicama i u udaljenim instalacijama.Chloros

1.2.0 otkriva vaš model Jetson pri pokretanju i optimizira njegovu strategiju obrade za hardver koji pronađe. **Nije potrebno ručno podešavanje.**

***

## Podržani Jetson modeli

| Model                | RAM            | Strategija obrade                                     | Preporučena upotreba                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB zajedničke | `GPU_PARALLEL` (2 radnika)                              | Maksimalne performanse, veliki skupovi podataka                      |
| **Jetson Orin NX**   | 8-16 GB zajedničke    | `GPU_PARALLEL` (2 radnika, 16 GB) / `GPU_SINGLE` (8 GB)   | Primarna preporuka za zračnu i terensku primjenu |
| **Jetson Orin Nano** | 8 GB zajedničke     | `GPU_SINGLE` (1 radnik, sekvencijalno)                     | Početno razina rubnog računarstva                                 |

{% hint style="info" %}
PaketLinux

arm64 zahtijeva **JetPack 6**, koji je dostupan za obitelj Jetson Orin. Stariji modeli (Nano, TX2, Xavier NX) ne mogu pokrenuti JetPack 6 i nisu podržani u ovom paketu.
{% endhint %}

***

## Zahtjevi

* **JetPack 6.x** (preporučuje se najnovija verzija)
* **NVIDIA CUDA** (uključeno u JetPack)
* **Plaćeni planChloros

+** — razina Copper ili viša (potrebno za pristup svemuCLI

/SDK

; provodi se na strani poslužitelja)

## Instalacija

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Za opće detalje o instalacijiLinux

, lokacijama datoteka i rješavanju problema pogledajte [Linux

Installation](linux-installation.md).

{% hint style="info" %}
**Postavite direktorij za ekstrakciju na brzo pohranjivanje.** Kompilirane binarne datoteke raspakiraju se u privremeni direktorij pri svakom pokretanju — što je izuzetno sporo s SD kartice.Chloros

automatski koristi `/mnt/ssd/tmp` ako on postoji; inače postavite `TMPDIR` na putanju na vašem NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dinamička prilagodba izračuna na Jetsonu

### Kako radi

Prilikom pokretanja,Chloros

profilirava vaš sustav:

1. **Detektira model Jetson** putem `/proc/device-tree/model`
2. **Čita dostupnu zajedničku memoriju GPU-a/CPU-a** (Jetson koristi unificiranu memoriju)
3. **Odabire strategiju obrade** (`GPU_PARALLEL`, `GPU_SINGLE` ili `CPU_PARALLEL`)
4. **Automatski postavlja broj radnika, vrstu cijevi i alokaciju memorije**Odluka se temelji na**ukupnoj zajedničkoj RAM memoriji**, a ne na nazivu modela:

* **Manje od 12 GB ukupne RAM memorije**(svi Jetson uređaji s 8 GB): `GPU_SINGLE` s**1 radnikom — namjerna sekvencijalna obrada**. Memorija je preograničena za istovremene radnike, stoga se slike obrađuju jedna po jedna. Na Jetsons uređajima s**8 GB ili manje**, Thread 3 u potpunosti zaobilazi bazen radnika i izvršava svoj posao po slici unutar procesa.
* **12 GB ili više**(Orin NX 16 GB, AGX Orin): jedinstvena memorija ispunjava uvjete za `GPU_PARALLEL`, ali broj radnika na Jetsonu je**ograničen na 2** — GPU, RAM radnih procesa i njihovi CUDA konteksti po radniku svi koriste isti zajednički fond, pa više radnika povećava rizik od grešaka zbog nedostatka memorije.

Automatski odabir možete nadjačati pomoću varijable okruženja `CHLOROS_STRATEGY` — pogledajte [Dinamička prilagodba izračuna](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Ponašanje po modelu

| Model Jetson                | Strategija       | Radnici | Izvršenje                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | Sekvencijalna petlja unutar procesa (`tiled_gpu` pod pritiskom memorije) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Sekvencijalna petlja u procesu                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | Istovremeni radni procesi, `fused_gpu` put  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | Istovremeni radni procesi, putanja `fused_gpu` |

Ključna razlika između platformi je **memorija**. Jetson od 8 GB mora obrađivati slike jednu po jednu koristeći memorijski učinkovit pristup u pločicama kada je opterećenje visoko, dok Orin od 16 GB+ može istovremeno obrađivati 2 slike putem GPU-a koristeći spojeni vod sa većom propusnošću.

### GPU proračun po modelu

Svaki model Jetson-a također ima hardverski profil koji ograničava koliko procesne snage zajedničkog fonda može zauzeti i skalira veličinu serija:

| Model | Gornja granica GPU proračuna | Multiplikator veličine serije | Rezervirano za sustav/prikaz |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1.5 | 4,0 GB |

Detektirana RAM memorija prilagođava profil: Jetson koji prijavljuje **16 GB ili više** dobiva povećan multiplikator serije za ×1,2. Osnovna veličina serije prije multiplikatora je 8 slika.

Za potpunu referencu o prilagodbi izračuna pogledajte [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Ograničenje radnog takta GPU-a za Texture Aware na Nano i Orin Nano

Texture Aware debayer pokreće inferenciju neuronske mreže na GPU-u, što može izazvati **upozorenja na prenaponsku struju**na niskopotrošnim Jetson modelima (klase 10-15 W) pri punoj taktnoj brzini GPU-a. Prije obrade pomoću Texture Aware na**Jetson Nano ili Orin Nano**,Chloros

provjerava maksimalnu frekvenciju GPU-a i ograničava je na **510 MHz** (510000000) ako je trenutno viša:

* AkoCLI

može zapisati čvor GPU frekvencije u sysfs-u, ograničenje se **primjenjuje automatski** i ispisuje se potvrda.
* Ako ne (potreban root),  ispisuje točnu `sudo` naredbu za ručno primjenu ograničenja, pričeka trenutak da je možete pročitati, a zatim nastavlja — obrada se i dalje izvršava, ali može prikazati upozorenja o prekomjernoj struji.

Da biste sami primijenili ograničenje prije obrade:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeli veće snage (Orin NX 25 W, AGX Orin 60 W) rade punom brzinom GPU-a; ograničenje se ne primjenjuje. Standardni debayer nikada ne aktivira ograničenje na bilo kojem modelu.

{% hint style="info" %}
**Texture Aware na Jetsonu uvijek obrađuje jednu sliku odjednom.** Svaki radnik trebao bi imati vlastiti CUDA kontekst (~1 GB) i vlastitu kopiju modela za uklanjanje šuma, što unificirana memorija ne može priuštiti — stoga je na Jetsonu putanja Texture Aware vezana za jednog radnika, a pristup GPU-u je serijaliziran. Očekujte da će Texture Aware biti znatno sporiji od Standard na bilo kojem Jetsonu.
{% endhint %}

***

## Termičko upravljanje

Jetson uređaji imaju ograničen termički kapacitet, posebno u zatvorenim ili zračnim postavkama.Chloros

nadzire temperaturu SoC-a i automatski prilagođava veličinu serija:

| Temperatura         | Radnja                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**         | Normalno funkcioniranje — puna brzina obrade          |
| **70 °C** (Upozorenje) | Veličina serije se progresivno smanjuje (100 % → 50 % između 70 °C i 80 °C) |
| **80 °C** (Kritično) | Agresivno ograničavanje (50 % → 0 % između 80 °C i 90 °C) |
| **90 °C** (Isključivanje) | Potpuno zaustavljanje obrade GPU-a — potrebno hlađenje |

{% hint style="warning" %}
**Osigurajte adekvatnu ventilaciju i odvođenje topline** za kontinuiranu obradu, posebno u zatvorenim terenskim kućištima ili zračnim sustavima. Termičko usporavanje smanjit će propusnost obrade radi zaštite hardvera.
{% endhint %}

***

## Upravljanje memorijom

Jetson uređaji koriste **jedinstvenu memoriju** — GPU i CPU dijele istu fizičku RAM memoriju. Prikazana VRAM memorija (npr. ~15,3 GB na Orin NX 16GB) nije namjenska GPU memorija; to je ista RAM memorija koju koriste operativni sustav i svaki drugi proces.

### Upozorenje o zamjenskoj memoriji i preporuke

Prije obrade na Jetsonu, CLI broji RAW slike u vašoj ulaznoj mapi (`.tif`, `.tiff`, `.raw`, `.dng` — pregledi JPG-a se ne računaju), procjenjuje vršnu memoriju koja je potrebna za izvršavanje i **upozorava prije početka** ako je RAM + swap vjerojatno nedovoljno. Upozorenje je naslovljeno `LOW MEMORY WARNING - Jetson Detected`, ispisuje broj vaših slika, RAM, trenutni swap i procijenjeni vrhunac, a zatim daje točne `fallocate` / `chmod` / `mkswap` / `swapon` naredbe dimenzionirane za vaš projekt (nikada manje od 8 GB). Pauzira na nekoliko sekundi kako poruka ne bi nestala u povijesti, a zatim se obrada nastavlja.**Procjene memorije koje koristi upozorenje:**

| Debayer način | Baza | Po slici |
| --- | --- | --- |
| Standardno | ~1,5 GB | ~10 MB |
| S podrškom za teksture | ~2,5 GB (model + Python runtime) | ~15 MB |

Upozorenje se aktivira kada procijenjeni vrhunac premaši RAM + swap umanjen za sigurnosnu maržu od 1 GB, i uzima u obzir samo swap **podržan datotekama** — postavka koja koristi samo zram i dalje će biti označena.

Za ručno dodavanje swap prostora (primjer: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

### Rješavanje problema OOM (nedostatak memorije)

Tijekom obrade,Chloros

nadzire memoriju GPU-a i uredno se degradira umjesto da se sruši:

1. Kada korištenje GPU memorije premaši **85%**, veličine serija se preventivno smanjuju
2. Ako svejedno dođe do događaja nedostatka memorije, veličina serije se **smanjuje na polovicu**, a ponovno se smanjuje na polovicu pri svakom sljedećem OOM događaju; svaka sljedeća uspješna serija vraća tu kaznu za jedan korak unatrag
3. Pod kontinuiranim opterećenjem, pipeline se vraća s `fused_gpu` na memorijski učinkovit `tiled_gpu` put, a kao krajnje sredstvo na obradu u CPU-u

***

## Terensko postavljanje

### Razmatranja o napajanju

| Model Jetson     | Tipična potrošnja energije | Napomene                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | DC barrel priključak          |
| Jetson Orin NX   | 10-25 W             | DC barrel priključak          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD ili barrel priključak |

Planirajte svoj energetski proračun za kontinuiranu obradu — vršna potrošnja energije događa se tijekom GPU-intenzivnog Thread 3 (Processing).

### Preporuke za pohranu

* **NVMe SSD** se snažno preporučuje za implementacije na arm64 arhitekturi
* SD kartice su prespori za obradu — koristite ih samo kao medij za pokretanje
* Planirajte 2-3 puta veću veličinu od sirovih podataka slike za obrađeni izlaz

### Rad bez zaslona putemSSH



Chloros

CLI

je idealan za primjenu Jetsona bez zaslona:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Uvijek aktivan pozadinski sustav za LATTICE / DAQ-E sinkronizaciju vremena

Ako vaš Jetson bežično upravlja LATTICE kamerama ili DAQ-E senzorima svjetla, omogućite pozadinsku uslugu systemd kako bi PTP grandmaster radio neprekidno (jedinica je instalirana, ali nije omogućena prema zadanim postavkama):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Pogledajte [InstalacijaLinux

](linux-installation.md#always-on-ptp-for-headless-hosts) za detalje, uključujući kako paket omogućuje da se PTP priključci 319/320 mogu povezati bez root prava.

### Automatska obrada sa systemdom

Stvorite systemd uslugu za automatsku obradu:

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

`chloros-cli process` završava s nestandardnim izlaznim kodom kada pokretanje koje je zatražilo proizvode ne piše nijednu sliku, tako da je status pogreške systemda koristan za nadzor.

Upari s systemd timerom za zakazanu obradu:

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

## Primjeri tijekova rada

### Osnovna obrada na Jetsonu

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

###PythonSDK

na Jetsonu

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

### Serijska obrada više letova

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Preporučeni Jetson sustavi za terensku upotrebu

Za terensku i zračnu primjenu, razmotrite ove opcije matičnih ploča Jetson Orin NX 16GB:

* **Zračni/za dron**: Sustavi s ocjenom otpornosti na vibracije (MIL-STD), lagani (manje od 300 g), pasivno hlađenje
* **Tereni za teške uvjete**: Vodootporna kućišta s ocjenom IP67/IP69K s povezivošću za kamere putem PoE GigE
* **Minimalni/budžetski**: Developer kompleti s dodatnim kućištima

Kontaktirajte [MAPIR

podršku](https://www.mapir.camera/community/contact) za specifične preporuke hardvera za vaš scenarij implementacije.

***

## Sljedeći koraci

* [Linux

instalacija](linux-installation.md) — Opći detalji o instalacijiLinux


* [Dinamička prilagodba izračuna](../processing-architecture/dynamic-compute-adaptation.md) — Potpuni referentni vodič za strategiju izračuna
* [Procesna cijev](../processing-architecture/processing-pipeline.md) — Razumijevanje cijevi s 4 niti
* [CLI

: Command Line](../CLI.md) — Vodič zaCLI


* [API

:Python

SDK

](../api-python-sdk.md) — Vodič zaSDK


* [CLI

Referenca](../reference/cli-reference.md) i [SDK

Referenca](../reference/sdk-reference.md) — Potpuni popisi naredbi /API

za 1.2.0
