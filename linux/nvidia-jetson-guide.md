# NVIDIA Jetson vodič

Chloros
za NVIDIA Jetson omogućuje višespektralnu obradu slika na rubu mreže — na terenu, na bespilotnim letjelicama i u udaljenim instalacijama. Chloros
1.3.0 pri pokretanju otkriva vaš model Jetson i optimizira njegovu strategiju obrade za hardver koji pronađe. **Nije potrebno ručno podešavanje.**
{% hint style="info" %}
**Paket za Jetson 1.3.0 obrađuje se na GPU-u Jetsona.** `chloros_1.3.0_arm64_jp6.deb` je izgrađen za JetPack 6 i koristi CUDA izdanje PyTorcha iz JetPacka 6, pa se obrada, uključujući i Texture Aware debayer, odvija na GPU-u Orina. Paket ne sadrži CUDA biblioteku: koristi biblioteke CUDA i cuDNN koje instalira JetPack 6 (`sudo apt install nvidia-jetpack` na slici na kojoj one nedostaju). Gdje se ne može koristiti CUDA GPU, obrada se odvija na CPU-u, a Texture Aware koristi standardni debayer.
{% endhint %}

***

## Podržani Jetson modeli

| Model                | RAM            | Strategija obrade                                     | Preporučena upotreba                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB dijeljena | `GPU_PARALLEL` (2 radnika)                              | Maksimalne performanse, veliki skupovi podataka                      |
| **Jetson Orin NX**   | 8-16 GB zajedničke  | `GPU_PARALLEL` (2 radnika, 16 GB) / `GPU_SINGLE` (8 GB)   | Primarna preporuka za terensku implementaciju              |
| **Jetson Orin Nano** | 8 GB zajedničke     | `GPU_SINGLE` (1 radnik, sekvencijalno)                     | Uvodna razina rubnog računarstva                                |

{% hint style="info" %}
Linux
arm64 paket zahtijeva **JetPack 6**, koji je dostupan na obitelji Jetson Orin. Stariji modeli (Nano, TX2, Xavier NX) ne mogu pokretati JetPack 6 i nisu podržani u trenutnom paketu.
{% endhint %}

***

## Zahtjevi

* **JetPack 6.x** (preporučuje se najnovija verzija)
* **NVIDIA CUDA i cuDNN** iz JetPacka 6 (`sudo apt install nvidia-jetpack` ako ih vaša slika nema)
* **Plaćeni plan Chloros
+** — razina Copper ili viša (potreban za pristup svim CLI
/SDK
; provodi se na strani poslužitelja)

## Instalacija

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` instalira Python
ov SDK
za sustav `python3`, koji odgovara ovoj izradi. Samo virtualenv-u je potrebno instalirati priloženi wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (potpuno offline). Za opće detalje o instalaciji na Linux
, lokacije datoteka i rješavanje problema, pogledajte [Linux
Installation](linux-installation.md).

{% hint style="info" %}
**Više nema raspakiravanja pri pokretanju.** Od verzije 1.3.0, backend i CLI
instalirani su neotpakirani u `/usr/lib/chloros/`, pa se više ne raspakiravaju u privremeni direktorij svaki put kad se pokrenu. Privremeni se datoteke i dalje automatski spremaju u `/mnt/ssd/tmp` ako taj direktorij postoji; u suprotnom, možete postaviti `TMPDIR` na putanju na vašem NVMe-u (`export TMPDIR=/mnt/nvme/tmp`).
Chloros

***

## Dinamička prilagodba izračuna na Jetsonu

### Kako radi

Prilikom pokretanja, Chloros
profilira vaš sustav:

1. **Detektira model Jetson** putem `/proc/device-tree/model`
2. **Čita dostupnu zajedničku GPU/CPU memoriju** (Jetson koristi unificiranu memoriju)
3. **Pita PyTorch može li koristiti CUDA** — s instaliranim CUDA i cuDNN-om za JetPack 6, može
4. **Automatski odabire strategiju obrade** (`GPU_PARALLEL`, `GPU_SINGLE` ili `CPU_PARALLEL`) i broj radnika

Odluka ovisi o **ukupnoj zajedničkoj RAM memoriji**, a ne o nazivu modela:

* **Manje od 12 GB ukupne RAM memorije**(svi Jetsons uređaji s 8 GB):**1 radnik — namjerno sekvencijalno obrađivanje**. Memorije je premalo za istovremene radnike, stoga se slike obrađuju jedna po jedna. Na Jetsons uređajima s**8 GB ili manje**, Thread 3 u potpunosti zaobilazi bazen radnika i izvršava svoj posao po slici unutar procesa. Backend log zapise ovu strategiju kao `GPU_SINGLE`.
* **12 GB ili više**(Orin NX 16 GB, AGX Orin): jedinstvena memorija ispunjava uvjete za `GPU_PARALLEL`, ali broj radnika na Jetsonu je**ograničen na 2** — GPU, RAM radnih procesa i njihovi CUDA konteksti po radniku svi koriste isti zajednički fond, pa više radnika povećava rizik od grešaka zbog nedostatka memorije.
* **Nema dostupne CUDA-e** (nedostaju CUDA biblioteke za JetPack): `CPU_PARALLEL` pri 12 GB ili više, po jedna slika odjednom za manje.

Možete nadjačati automatski odabir pomoću varijable okruženja `CHLOROS_STRATEGY` — pogledajte [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Ponašanje po modelu

| Model Jetson                | Strategija       | Radnici | Izvršavanje                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | Sekvencijalna petlja unutar procesa (`tiled_gpu` pri opterećenju memorije) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Sekvencijalna petlja u procesu                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | Istovremeni radni procesi, `fused_gpu` putanja  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | Istovremeni radni procesi, `fused_gpu` putanja  |

Ključna razlika između platformi je **memorija**. Jetson od 8 GB mora obrađivati slike jednu po jednu koristeći memorijski učinkovit pristup u pločicama kada je opterećenje visoko, dok Orin od 16 GB+ može istovremeno obrađivati 2 slike putem GPU-a koristeći spojeni vod sa većom propusnošću.

### GPU proračun po modelu

Svaki model Jetsona također ima hardverski profil koji ograničava koliko procesne snage zajedničkog fonda može zauzeti i skalira veličinu serija:

| Model | Gornja granica GPU proračuna | Multiplikator veličine serije | Rezervirano za sustav/prikaz |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Detektirana RAM memorija prilagođava profil: Jetson koji prijavljuje **16 GB ili više** dobiva povećan multiplikator serije za ×1,2. Osnovna veličina serije prije multiplikatora je 8 slika.

Za potpuni referentni primjer prilagodbe izračuna pogledajte [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Ograničenje radnog takta GPU-a za Texture Aware na Nano i Orin Nano

Texture Aware debayer pokreće inferenciju neuronske mreže na GPU-u, što može izazvati **upozorenja na prenaponsko opterećenje**na niskopotrošnim Jetson modelima (klase 10-15 W) pri punoj taktu GPU-a. Kada navedete `--debayer texture-aware` na**Jetson Nano ili Orin Nano**, CLI
provjerava maksimalnu frekvenciju GPU-a i ograničava je na **510 MHz** (510000000) ako je trenutno viša:

* Ako CLI
može zapisati sysfs čvor za frekvenciju GPU-a, ograničenje se **automatski primjenjuje** i ispisuje se potvrda.
* Ako ne (potreban je root), CLI
ispisuje točnu naredbu `sudo` za ručno postavljanje ograničenja, pričeka trenutak da je možete pročitati, a zatim nastavlja — obrada se i dalje izvršava, ali može prikazati upozorenja o prekomjernoj struji.

Da biste sami primijenili ograničenje prije obrade:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeli veće snage (Orin NX 25 W, AGX Orin 60 W) rade na punoj brzini GPU-a; ograničenje nije primijenjeno. Standardni debayer nikada ne aktivira ograničenje na bilo kojem modelu.

{% endhint %}

{% hint style="info" %}
**Texture Aware na Jetsonu uvijek obrađuje jednu sliku odjednom.** Svaki radnik trebao bi imati vlastiti CUDA kontekst (~1 GB) i vlastitu kopiju modela za uklanjanje šuma, što unificirana memorija ne može priuštiti — stoga je na Jetsonu putanja Texture Aware vezana za jednog radnika, a pristup GPU-u je serijaliziran. Očekujte da će Texture Aware biti znatno sporiji od Standard načina rada na bilo kojem Jetsonu. Texture Aware se primjenjuje na slike iz Survey3
: LATTICE snimanja uvijek koriste Standard debayer. Bez upotrebljivog CUDA GPU-a, izvršavanje koristi Standard, a backend bilježi jednu liniju `[DEBAYER] … needs a CUDA GPU …` koja navodi zašto (prikazano s `chloros-cli --verbose`).
{% endhint %}

***

## Upravljanje toplinom

Jetson uređaji imaju ograničen toplinski kapacitet, posebno u zatvorenim ili zračnim postavkama. Chloros
nadzire temperaturu SoC-a i automatski prilagođava veličinu serija:

| Temperatura         | Radnja                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normalno funkcioniranje — puna brzina obrade          |
| **70 °C** (Upozorenje) | Veličina serije se postupno smanjuje (100 % → 50 % između 70 °C i 80 °C) |
| **80 °C** (Kritično) | Agresivno usporavanje (50 % → 0 % između 80 °C i 90 °C) |
| **90 °C** (Isključivanje) | Potpuno zaustavljanje obrade GPU-a — potrebno hlađenje |

{% hint style="warning" %}
**Osigurajte adekvatnu ventilaciju i odvođenje topline** za kontinuiranu obradu, posebno u zatvorenim terenskim kućištima ili zračnim sustavima. Toplinsko usporavanje smanjit će propusnost obrade radi zaštite hardvera.
{% endhint %}

***

## Upravljanje memorijom

Jetson uređaji koriste **jedinstvenu memoriju** — GPU i CPU dijele istu fizičku RAM memoriju. Prikazana VRAM memorija (npr. ~15,3 GB na Orin NX 16 GB) nije namjenska GPU memorija; to je ista RAM memorija koju koristi operativni sustav i svaki drugi proces.

### Upozorenje o zamjenskom prostoru i preporuke

Prije obrade na Jetsonu, CLI
broji RAW slike u vašoj ulaznoj mapi (`.tif`, `.tiff`, `.raw`, `.dng` — pregledne JPG slike se ne računaju), procjenjuje vršnu memoriju koja je potrebna za izvršavanje i **upozorava prije početka** ako je RAM + swap vjerojatno nedovoljno. Upozorenje je naslovljeno `LOW MEMORY WARNING - Jetson Detected`, ispisuje broj vaših slika, RAM-a, trenutne swap memorije i procijenjenog vršnog opterećenja, a zatim daje točne `fallocate` / `chmod` / `mkswap` / `swapon` naredbe dimenzionirane za vašeg projekta (nikada manji od 8 GB). Pauzira nekoliko sekundi kako poruka ne bi nestala u povijesti, a zatim se obrada nastavlja.**Procjene memorije koje koristi upozorenje:**

| Debayer mod | Baza | Po slici |
| --- | --- | --- |
| Standardno | ~1,5 GB | ~10 MB |
| Svestrano za teksture | ~2,5 GB (model + Python runtime) | ~15 MB |

Upozorenje se aktivira kada procijenjeni vrhunac premaši RAM + swap umanjen za sigurnosnu maržu od 1 GB, i uzima u obzir samo swap podržan datotekama — postavka koja koristi samo zram i dalje će biti označena.

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

Tijekom obrade, Chloros
nadzire memoriju GPU-a i uredno se degradira umjesto da se sruši:

1. Kada korištenje GPU memorije premaši **85%**, veličine serija se preventivno smanjuju
2. Ako se unatoč tome dogodi događaj nedostatka memorije, veličina serije se **smanjuje na polovicu**, a ponovno se smanjuje na polovicu pri svakom sljedećem OOM-u; svaka sljedeća uspješna serija vraća tu kaznu za jedan korak unatrag
3. Pod dugotrajnim opterećenjem, pipeline se vraća s `fused_gpu` na memorijski učinkovit `tiled_gpu` put, te na obradu CPU-om kao posljednje sredstvo

Budući da GPU dijeli RAM Jetsona, RAM i swap i dalje ograničavaju izvršavanje — upozorenje o swapu gore je provjera na koju treba obratiti pozornost.

***

## Terensko raspoređivanje

### Razmatranja o napajanju

| Model Jetsona     | Tipična potrošnja energije | Napomene                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7-15 W              | DC priključak u obliku cijevi          |
| Jetson Orin NX   | 10-25 W             | DC priključak u obliku cijevi          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD ili DC priključak u obliku cijevi |

Planirajte svoj energetski proračun za kontinuiranu obradu — vršna potrošnja energije događa se tijekom GPU-intenzivnog Thread 3 (Processing).

### Preporuke za pohranu

* **NVMe SSD** se snažno preporučuje za implementacije na arm64 arhitekturi
* SD kartice su prespori za obradu — koristite ih samo kao medij za pokretanje
* Planirajte 2-3 puta veću veličinu za obrađeni izlaz u odnosu na sirovu veličinu podataka slike

### Rad bez zaslona putem SSH
-a i

Chloros
-a je idealan za postavke Jetsona bez zaslona:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Uvijek aktivan backend za LATTICE / DAQ-E sinkronizaciju vremena

Ako vaš Jetson bežično upravlja LATTICE kamerama ili DAQ-E senzorima svjetla, omogućite pozadinski sustavni servis systemd **kao svoj korisnički račun** kako bi PTP grandmaster radio neprekidno i kako bi skripte SDK
pronašle pokrenuti pozadinski servis (jedinica je instalirana, ali nije omogućena prema zadanim postavkama):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend se tada pokreće kao vi: snimke koje piše su vaše i koristi vašu `chloros-cli login`. Pogledajte [Linux
Installation](linux-installation.md#always-on-ptp-for-headless-hosts) za detalje, uključujući root `chloros-backend.service` i kako paket čini PTP priključke 319/320 dostupnima za vezivanje bez root prava.

### Automatska obrada sa systemd-om

Nastavite sistemd uslugu za automatiziranu obradu:

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
    --indices NDVI GNDVI OSAVI
```

### Python
SDK
na Jetsonu

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Preporučeni Jetson sustavi za terensku upotrebu

Za terensku i zračnu primjenu razmotrite ove opcije matičnih ploča Jetson Orin NX 16GB:

* **Zračni/dron**: Sustavi s ocjenom otpornosti na vibracije (MIL-STD), lagani (manje od 300 g), pasivno hlađenje
* **Zahtjevno terensko okruženje**: Vodootporne kućišta IP67/IP69K s PoE GigE povezivošću za kamere
* **Minimalno/budžetsko**: Razvojni setovi s dodatnim kućištima

Kontaktirajte [MAPIR
podršku](https://www.mapir.camera/community/contact) za specifične preporuke hardvera za vaš scenarij implementacije.

***

## Sljedeći koraci

* [Instalacija Linux
a](linux-installation.md) — Opći detalji o instalaciji Linux
a
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Potpuni referentni vodič za računalnu strategiju
* [Processing Pipeline](../processing-architecture/processing-pipeline.md) — Razumijevanje 4-thread pipelinea
* [CLI
: naredbeni redak](../CLI.md) — Vodič za CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Vodič za SDK

* [CLI
referenca](../reference/cli-reference.md) i [SDK
referenca](../reference/sdk-reference.md) — Potpuni popis naredbi/API
za verziju 1.3.0
