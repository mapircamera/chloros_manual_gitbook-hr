# Procesna cijev

Chloros 1.3.0 koristi procesnu cijev s 4 niti koja radi poput proizvodne trake s fazama. Svaka nit obrađuje zasebnu fazu tijeka rada, tako da nekoliko slika može istovremeno biti u različitim fazama obrade.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arhitektura procesne linije

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Svaka slika prolazi kroz sva četiri niti redom. Kod višenitne obrade sustava Chloros+, više slika istovremeno zauzima različite niti — dok nit 3 obrađuje jednu sliku, nit 1 može otkrivati sljedeću, nit 2 kalibrirati još jednu, a nit 4 spremati gotovu na disk.

Napredak se izvještava za svaki thread i prenosi putem Server-Sent Events (backend ih objavljuje na `/api/events`). U prikazu napretka uživo u CLIu četiri faze su označene kao **Detekcija, Analiza, Obrada, Izvoz**.***

## Detalji niti

### Niti 1: Detekcija

**Svrha**: Učitavanje slika i detekcija kalibracijskih ciljeva.

* Čita datoteke sa slikama s diska — parove Survey3 `.raw`+`.jpg`, LATTICE `.tif`/`.tiff` snimke, i `.dng`
* Izvlači EXIF metapodatke (GPS, model kamere, vremenski pečati, ekspozicija)
* Detektira kalibracijske ciljeve: ArUco-označene geometrijske ciljeve za LATTICE snimke i klasični detektor panela za fotografije kalibracijskih ciljeva Survey3
* Izlazni podaci: podaci o slici + metapodaci + rezultati detekcije ciljeva

Prvenstveno je to niti vezana za I/O i CPU.

### Niti 2: Kalibracija

**Svrha**: Izračunavanje kalibracijskih parametara iz otkrivenih ciljeva.

* Izračunava koeficijente kalibracije reflektancije iz slika ciljeva
* Izračunava parametre korekcije vinjete
* Određuje kalibracijske krivulje po pojasu
* Izlazni podaci: kalibracijski parametri za svaku sliku

Nit za izračune vezana za CPU. Prilikom izvođenja programa *Survey3* ili mješovitog izvođenja, nit 3 čeka na nju kada je omogućena kalibracija reflektancije, tako da su njezini koeficijenti spremni prije obrade bilo koje slike. Prilikom izvođenja programa samo za LATTICE, nit 3 radi tijekom detekcije: svaki kadar započinje čim je njegova ciljna sidrenica konačna. Okvir uzima najbliži cilj u vremenu s obje strane, stoga čeka dok svaki kadar njegove kamere koji je vremenski bliži od trenutnog najbližeg cilja nije skeniran. Proizvodi su isti, samo raniji.

### Niti 3: Obrada (GPU)

**Svrha**: Primjena korekcija i izračunavanje indeksa vegetacije.**Ovo je najzahtjevniji thread za obradu.*** **Debayering**: pretvara RAW Bayer podatke u višekanalne slike
  * Standardno (Brzo, Srednja kvaliteta) — zadano, `--debayer standard`
  * S obzirom na teksturu (Sporo, Najviša kvaliteta) — samo za Chloros+, `--debayer texture-aware`, koristi model za uklanjanje šuma temeljen na umjetnoj inteligenciji/strojnom učenju
  * LATTICE mono (M3M) snimke su jednopojasne: koraci demosaicinga i bilance bijele boje za njih se preskaču (uz poruku u zapisniku od jedne linije), dok sve M3C/Bayer slike u istoj izvedbi i dalje dobivaju te korake
* **Korekcija vinjetiranja**: primjenjuje korekciju vinjetiranja objektiva na cijelu sliku
* **Kalibracija reflektancije**: primjenjuje koeficijente kalibracije za pretvorbu u vrijednosti reflektancije
* **Izračun indeksa**: izračunava indekse vegetacije (NDVI, NDRE, GNDVI, …)
* Izlaz: obrađeni podaci slike spremni za izvoz

Ovaj niz najviše koristi od GPU akceleracije, a to je niz koji prilagođava [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Niti 4: Izvoz

**Svrha**: Zapisivanje obrađenih slika na disk.

* Zapisuje izlazne datoteke u odabranom formatu — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Umetanje metapodataka u izlazne datoteke (GPS, vremenski žigovi, parametri obrade) — uključujući oznake obrade `Chloros:*`, koje se od verzije 1.3.0 upisuju u svaki način obrade, za Survey3 kao i za LATTICE
* Organizira izlazne datoteke u sklopu projektne mape kao `<camera>/<format>/<Product>_Images/` — na primjer `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Izvozne datoteke zadržavaju naziv izvornog datoteka; mapa identificira proizvod.**
* Za LATTICE snimke, jedan izvorni kadar može se razgranati u nekoliko proizvoda (Debayered, Preview, Radiance, Reflectance, Index), svaki u vlastitoj mapi proizvoda. Indeksni proizvodi izračunavaju se iz proizvoda reflektancije koji je izrada napisala (ako ga nema, iz drugog proizvoda koji je izrada napisala, kao što je radijanca). Proizvodi radijance i reflektance višespektralnih M3C kamera također mogu dobiti maskiranje po pikselu u `<format>/clamp_masks/` (vidi [Izlazni formati slika](../output-image-formats.md#clamp-masks))
* Izlazni podaci: konačne datoteke na disku

Prvenstveno I/O-ograničena nit — SSD pohrana značajno poboljšava performanse.

***

## U pozadini: Izvršitelji

Unutar Niti 3, rad po slici paraleliziran je na sljedeći način:

* **GPU strategije**(`GPU_SINGLE`, `GPU_PARALLEL`) koriste bazen**radnih procesa**, od kojih je svaki zaseban OS proces sa vlastitim CUDA kontekstom. Chloros pokreće svaki radni proces ponovnim pokretanjem vlastitog instaliranog pozadinskog programa — a ne putem `multiprocessing`-a u Pythonu, čije pokretanje radnih procesa ne radi unutar pakirane aplikacije
* **`CPU_PARALLEL`** koristi bazen niti (Pythonov `ThreadPoolExecutor`) — NumPy i OpenCV oslobađaju GIL, stoga su niti dovoljne
* Jetson uređaji s 8 GB ili manje zajedničke RAM-a u potpunosti preskaču izvršitelj i obrađuju se sekvencijalno unutar procesa
* Texture Aware na GPU-u s manje od 7 GB VRAM-a također radi sekvencijalno — model za uklanjanje šuma ne može stati više od jednom

Chloros ne koristi nijedan distribuirani okvir treće strane (kao što je Ray). Pogledajte [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) kako bi saznali kako se odabiru strategija i broj radnika.

***

## Sekvencijalna naspram pipeline obrade

### Besplatni način rada (sekvencijalni)

U besplatnoj verziji programa Chloros slike se obrađuju **jedna po jedna**, slijedom kroz sve četiri faze:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI prikazuje pojednostavljenu traku napretka u besplatnom načinu rada; njezine serijske faze prikazuju se kao **Otkrivanje ciljeva**, a zatim**Obrada**.

### Chloros+ način rada (pipeline)

Uz licencu Chloros+, sva četiri niti rade **istovremeno** na različitim slikama:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Traka napretka GUI-ja prikazuje četiri faze; pređite mišem preko nje kako biste vidjeli napredak za svaki niti. U CLI-ju se ista četiri faze prikazuju uživo kao **Detektiraj, Analiziraj, Obradi, Izvezi**.

{% hint style="info" %}
**Jedna oznaka, dva naziva.** CLI naziva fazu 3 _Processing_. Podaci o napretku pozadinskog sustava u premium načinu rada — one koje prikazuje traka napretka GUI-ja — istu fazu naziva _Calibrating_. To je isti thread koji obavlja isti posao (Thread 3: debayer, korekcije, indeksi).
{% endhint %}

{% hint style="success" %}
**Paralelna obrada s programom**Chloros+** može biti 3-5 puta brža od sekvencijalne obrade, ovisno o vašem hardveru i veličini skupa podataka. Povećanje brzine je najveće na sustavima s brzim GPU-ovima i SSD-ovima.
{% endhint %}

***

## Napredak izvoza niti 4

Nit za izvoz ima vlastito praćenje napretka, koje možete zasebno ispitivati:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Obrada je dovršena kada nit 4 dosegne 100 %.

{% hint style="info" %}
**Pokretanje koje ne zapisa nijednu sliku predstavlja neuspjeh.**U slučaju uspjeha, `chloros-cli process` izvještava o broju zapisanih slikovnih proizvoda (`Image products written: N`). Ako su proizvodi zatraženi, a**ni jedan**nije napisan — samo `project.json` i `calibration_data.json` — CLI ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nuljevim rezultatom**, nazivanjem mape projekta i uobičajenih uzroka (ulazna mapa nije prepoznata kao snimanje — provjerite raspored i `--input-level` — ili je svaki zatraženi proizvod bio neprimjenjiv na te kamere). Skripte se mogu osloniti na izlazni kod.
{% endhint %}

***

## Odnos prema prilagodbi dinamičkog računanja

[Prilagodba dinamičkog računanja](dynamic-compute-adaptation.md) prvenstveno utječe na **Nit 3 (Obrada)**:

* **`GPU_PARALLEL`**: Niti 3 istovremeno obrađuje više slika na GPU-u koristeći `fused_gpu` pipeline
* **`GPU_SINGLE`**: Niti 3 serijalizira pristup GPU-u pomoću semafora dok radne niti istovremeno obavljaju I/O operacije, koristeći `fused_gpu` ili memorijski učinkovit `tiled_gpu` pipeline
* **`CPU_PARALLEL`**: Niti 3 koristi obradu na CPU-u s višenitnom paralelnošću

Alokacija GPU memorije niti 3 također raste kako niti 1 i 2 završavaju — pogledajte [Dinamičku alokaciju GPU memorije](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Sljedeći koraci

* [Dinamička prilagodba izračuna](dynamic-compute-adaptation.md) — Kako Chloros odabire optimalnu strategiju za vaš hardver
* [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Ponašanje cjevovoda specifično za platformu Jetson
* [Praćenje obrade](../processing-images-gui/monitoring-the-processing.md) — praćenje napretka putem GUI-ja
* [Referenca za CLI](../reference/cli-reference.md) — `process`, `export-status`, kodovi izlaska i raspored izlaza
