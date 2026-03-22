# Cjevovod za obradu

Chloros 1.1.0 koristi cjevovod za obradu s 4 niti koji radi kao postupna montažna linija. Svaka nit upravlja posebnom fazom tijeka obrade, dopuštajući da se više slika istovremeno obrađuje u različitim fazama.

***

## Arhitektura cjevovoda

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Svaka slika redom teče kroz sve četiri niti. S Chloros+ obradom više niti, više slika može biti u različitim nitima istovremeno — dok Thread 3 obrađuje jednu sliku, Thread 1 može detektirati sljedeću, Thread 2 može kalibrirati drugu, a Thread 4 može pisati prethodno obrađenu sliku na disk.

***

## Pojedinosti niti

### Nit 1: Otkrivanje

**Svrha**: učitavanje slika i otkrivanje ciljeva kalibracije.

* Čita slikovne datoteke s diska (RAW, JPG)
* Izdvaja EXIF metapodatke (GPS, model kamere, vremenske oznake, ekspozicija)
* Otkriva ArUco kalibracijske ciljeve u označenim slikama ciljeva
* Izlazi: slikovni podaci + metapodaci + rezultati detekcije cilja

Ovo je prvenstveno I/O i CPU vezana nit.

### Nit 2: Kalibracija

**Svrha**: Izračunajte parametre kalibracije iz otkrivenih ciljeva.

* Izračunava koeficijente kalibracije refleksije iz ciljnih slika
* Izračunava parametre korekcije vinjete
* Određuje krivulje kalibracije po pojasu
* Izlazi: parametri kalibracije za svaku sliku

Ovo je računska nit vezana za CPU.

### Nit 3: Obrada (GPU)

**Svrha**: Primijeniti korekcije i izračunati vegetacijske indekse.**Ovo je računski najzahtjevnija nit.*** **Debayering**: Pretvara RAW podatke Bayerovog uzorka u višekanalne slike
  * Standardno (brzo, srednje kvalitete) — zadano
  * Svjestan teksture (Sporo, najviša kvaliteta) — samo Chloros+, koristi AI/ML uklanjanje šuma
* **Ispravak vinjete**: Primjenjuje korekciju vinjete objektiva preko slike
* **Kalibracija refleksije**: Primjenjuje kalibracijske koeficijente za pretvorbu u vrijednosti refleksije
* **Izračun indeksa**: Izračunava indekse vegetacije (NDVI, NDRE, GNDVI, itd.)
* Izlazi: obrađeni slikovni podaci spremni za izvoz

Ova nit najviše koristi od GPU ubrzanja. Sustav [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) prvenstveno optimizira ponašanje ove niti.

### Nit 4: Izvoz

**Svrha**: Zapisivanje obrađenih slika na disk.

* Zapisuje izlazne datoteke u odabranom formatu (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Ugrađuje EXIF metapodatke u izlazne datoteke (GPS, vremenske oznake, parametri obrade)
* Organizira izlaz u podmape modela fotoaparata
* Izlazi: konačne datoteke na disku

Ovo je prvenstveno I/O-vezana nit. SSD pohrana značajno poboljšava performanse Thread 4.

***

## Sekvencijalna naspram cijevne obrade

### Slobodan način (sekvencijalni)

U besplatnoj verziji Chloros, slike se obrađuju **jedna po jedna**, uzastopno kroz sve četiri faze:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI traka napretka prikazuje 2 faze: otkrivanje cilja i obrada.

### Chloros+ način rada (protočno)

Uz Chloros+ licencu, sve četiri niti rade **istovremeno** na različitim slikama:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI traka napretka prikazuje 4 faze: otkrivanje, analiza, kalibracija, izvoz. Zadržite pokazivač iznad trake napretka da biste vidjeli napredak po nitima.

{% hint style="success" %}
**Protočna obrada s Chloros+** može biti 3-5x brža od sekvencijalne obrade, ovisno o vašem hardveru i veličini skupa podataka. Ubrzanje je najveće na sustavima s brzim GPU-ovima i SSD-ovima.
{% endhint %}

***

## Napredak izvoza niti 4

U Chloros 1.1.0, izvozna nit (nit 4) ima vlastito namjensko praćenje napretka. Napredak izvoza možete pratiti zasebno:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Obrada je dovršena kada nit 4 dosegne 100%.

***

## Odnos prema dinamičkoj prilagodbi računala

Sustav [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) primarno utječe na **Nit 3 (obrada)**:

* Strategija **`GPU_PARALLEL`**: Thread 3 pokreće više slika kroz GPU istovremeno koristeći cjevovod `fused_gpu`
* Strategija **`GPU_SINGLE`**: Nit 3 obrađuje jednu po jednu sliku pomoću memorijski učinkovitog cjevovoda `tiled_gpu`
* Strategija **`CPU_PARALLEL`**: Thread 3 koristi obradu temeljenu na CPU-u s višenitnim paralelizmom

Dodjela GPU memorije niti 3 također se dinamički mijenja kako se niti 1 i 2 dovrše — pogledajte [Dinamička dodjela GPU memorije](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Sljedeći koraci

* [Dynamic Compute Adaptation] (dynamic-compute-adaptation.md) — Kako Chloros odabire optimalnu strategiju za vaš hardver
* [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — ponašanje cjevovoda specifično za platformu na Jetsonu
* [Praćenje obrade](../processing-images-gui/monitoring-the-processing.md) — GUI praćenje napretka