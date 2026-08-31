# Procesna cijev

Chloros1.2.0 koristi procesnu cijev s 4 niti koja radi poput proizvodne trake s fazama. Svaka nit obrađuje zasebnu fazu tijeka rada, tako da nekoliko slika može istovremeno biti u različitim fazama obrade.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arhitektura procesne linije

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Svaka slika prolazi kroz sva četiri niti redoslijedom. Kod višenitne obrade u Chloros+, više slika istovremeno zauzima različite niti — dok nit 3 obrađuje jednu sliku, nit 1 može otkrivati sljedeću, nit 2 kalibrirati još jednu, a nit 4 spremati gotovu na disk.

Napredak se izvještava za svaki thread i prenosi putem Server-Sent Events (backend ih objavljuje na `/api/events`). U prikazu napretka uživo aplikacije CLI četiri su faze označene kao **Detekcija, Analiza, Obrada, Izvoz**.***

## Detalji niti

### Nit 1: Detekcija

**Svrha**: Učitavanje slika i detekcija kalibracijskih ciljeva.

* Čita datoteke sa slikama s diska — parove Survey3 `.raw`+`.jpg`, LATTICE `.tif`/`.tiff` snimke i `.dng`
* Izvlači EXIF metapodatke (GPS, model kamere, vremenski pečati, ekspozicija)
* Detektira kalibracijske ciljeve: ArUco-označene geometrijske ciljeve za LATTICE snimke i klasični detektor panela za fotografije s kalibracijskim ciljevima Survey3
* Izlazni podaci: podaci o slici + metapodaci + rezultati detekcije ciljeva

Prvenstveno je to niti vezana uz I/O i CPU.

### Niti 2: Kalibracija

**Svrha**: Izračunavanje parametara kalibracije iz otkrivenih ciljeva.

* Izračunava koeficijente kalibracije reflektancije iz slika ciljeva
* Izračunava parametre korekcije vinjetiranja
* Određuje kalibracijske krivulje za svaku bandu
* Izlazni podaci: parametri kalibracije za svaku sliku

Računalni niz vezan uz CPU. Niz 3 čeka na njega kada je omogućena kalibracija reflektancije, tako da su njegovi koeficijenti spremni prije obrade bilo koje slike.

### Niti 3: Obrada (GPU)

**Svrha**: Primjena korekcija i izračunavanje indeksa vegetacije.**Ovo je niti s najzahtjevnijom računalnom obradom.*** **Debayering**: pretvara RAW Bayer podatke u višekanalne slike
  * Standardno (Brzo, Srednja kvaliteta) — zadano, `--debayer standard`
  * S obzirom na teksturu (Sporo, Najviša kvaliteta) — samo za Chloros+, `--debayer texture-aware`, koristi model za uklanjanje šuma temeljen na umjetnoj inteligenciji/strojnom učenju
  * Snimke LATTICE mono (M3M) su jednopojasne: koraci demosaicinga i balansa bijele boje za njih se preskaču (uz poruku u zapisniku), dok sve M3C/Bayer slike u istoj izvedbi i dalje prolaze kroz te korake
* **Korekcija vinjetiranja**: primjenjuje korekciju vinjetiranja objektiva na cijelu sliku
* **Kalibracija reflektancije**: primjenjuje kalibracijske koeficijente za pretvorbu u vrijednosti reflektancije
* **Izračun indeksa**: izračunava indekse vegetacije (NDVI, NDRE, GNDVI, …)
* Izlazni podaci: obrađeni podaci slike spremni za izvoz

Ovaj niz najviše koristi od GPU akceleracije, a to je niz koji prilagođava [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Niz 4: Izvoz

**Svrha**: Zapisivanje obrađenih slika na disk.

* Pisanje izlaznih datoteka u odabranom formatu — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Umetanje metapodataka u izlazne datoteke (GPS, vremenski žigovi, parametri obrade)
* Organizira izlazne datoteke u podmapu projekta kao `<camera>/<format>/<Product>_Images/` — na primjer `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Izvožene datoteke zadržavaju naziv izvornog datoteka; mapa identificira proizvod.**
* Za LATTICE snimke, jedan izvorni kadar može se razgranati u nekoliko proizvoda (Debayered, Preview, Radiance, Reflectance, Index), svaki u svojoj mapi proizvoda
* Izlazni proizvodi: konačne datoteke na disku

Prvenstveno je riječ o niti vezanoj za I/O — SSD pohrana to primjetno poboljšava.

***

## Iza kulisa: Izvršitelji

Unutar Niti 3, rad po slici paraleliziran je pomoću standardnog `concurrent.futures`-a Pythona:

* **GPU strategije**(`GPU_SINGLE`, `GPU_PARALLEL`) koriste `ProcessPoolExecutor` s**spawn** metoda start — svaki radnik je zaseban proces sa vlastitim CUDA kontekstom (`fork` bi naslijedio inicijalizirano CUDA stanje roditelja i time ga pokvario)
* **`CPU_PARALLEL`** koristi `ThreadPoolExecutor` — NumPy i OpenCV oslobađaju GIL, stoga su niti dovoljne
* Jetson uređaji s 8 GB ili manje zajedničke RAM memorije potpuno preskaču izvršitelja i obrađuju se unutar procesa, sekvencijalno
* Texture Aware na GPU-u s manje od 7 GB VRAM-a također radi sekvencijalno — model za uklanjanje šuma ne može stati više od jednom

Chlorosne koristi nijedan distribuirani okvir treće strane (kao što je Ray). Pogledajte [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) kako bi saznali kako se odabiru strategija i broj radnika.

***

## Sekvencijalna naspram pipelirane obrade

### Besplatni način rada (sekvencijalno)

U besplatnoj verziji programa Chloros slike se obrađuju **jedna po jedna**, slijedom kroz sve četiri faze:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI prikazuje pojednostavljenu traku napretka u besplatnom načinu rada; njezine serijske faze prikazuju se kao **Detekcija cilja**, a zatim**Obrada**.

### Mod Chloros+ (pipeline)

Uz licencu Chloros+, sva četiri niti rade **istovremeno** na različitim slikama:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Traka napretka GUI-ja prikazuje četiri faze; pređite mišem preko nje kako biste vidjeli napredak za svaki niti. U CLIu, ista četiri faze prikazuju se uživo kao **Otkrivanje, Analiziranje, Obrada, Izvoz**.

{% hint style="info" %}
**Jedna oznaka, dva naziva.** U prozoru za odabir slike (CLI) treća faza se naziva _Processing_. Backend-ov feed napretka u premium načinu rada — onaj koji GUI traka napretka prikazuje — istu fazu naziva _Calibrating_. Radi se o istom threadu koji obavlja isti posao (Thread 3: debayer, korekcije, indeksi).
{% endhint %}

{% hint style="success" %}
**Paralelna obrada s Chloros+** može biti 3-5 puta brža od sekvencijalne obrade, ovisno o vašem hardveru i veličini skupa podataka. Povećanje brzine je najveće na sustavima s brzim GPU-ovima i SSD-ovima.
{% endhint %}

***

## Napredak niti 4 za izvoz

Nit za izvoz ima vlastito praćenje napretka, koje možete zasebno provjeravati:**CLI:**

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
**Pokretanje koje ne zapisa nijednu sliku predstavlja neuspjeh.**U slučaju uspjeha, `chloros-cli process` izvještava o broju zapisanih proizvoda sa slikama (`Image products written: N`). Ako su proizvodi zatraženi, a**ni jedan**nije napisan — samo `project.json` i `calibration_data.json` — CLI ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nuljevim rezultatom**, imenovanje projektne mape i uobičajeni uzroci (ulazna mapa nije prepoznata kao snimanje — provjerite raspored i `--input-level` — ili je svaki zatraženi proizvod bio neprimjenjiv na te kamere). Skripte se mogu osloniti na izlazni kod.
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
* [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Ponašanje pipelinea specifično za platformu Jetson
* [Praćenje obrade](../processing-images-gui/monitoring-the-processing.md) — Praćenje napretka putem GUI-ja
* [CLI referenca](../reference/cli-reference.md) — `process`, `export-status`, kodovi izlaska i raspored izlaza
