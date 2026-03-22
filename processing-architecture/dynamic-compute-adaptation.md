# Dinamička prilagodba računala

Chloros 1.1.0 uvodi inteligentno otkrivanje hardvera i odabir strategije automatske obrade. Mehanizam za obradu prilagođava se vašem hardveru — od Jetson Nano do radne stanice s više GPU-a — bez ikakve ručne konfiguracije.

***

## Kako to radi

Kada se Chloros pokrene, automatski profilira vaš sustav:

1. **Otkriva operativni sustav** — Windows ili Linux
2. **Identificira CPU jezgre i ukupni RAM**

3.**Detektuje prisutnost GPU-a** — NVIDIA CUDA sposobnost, VRAM, model
4. **Identificira model Jetson** (ako je primjenjivo) — putem `/proc/device-tree/model`
5. **Provjerava toplinske senzore** (Jetson) — za obradu s obzirom na temperaturu
6. **Odabire optimalnu računsku strategiju** — na temelju svog otkrivenog hardvera
7. **Automatski konfigurira broj radnika, vrstu cjevovoda i dodjelu memorije**Rezultat se pohranjuje u predmemoriju tako da naredna izvođenja pokreću brže. Ako se hardver promijeni (npr. doda se GPU), Chloros ponovno profilira pri sljedećem pokretanju.***

## Računalne strategije

Chloros odabire jednu od tri računalne strategije na temelju vašeg hardvera:

| Strategija | Potreban GPU | Radnici | Cjevovod | Najbolje za |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Da (12GB+ VRAM ili 16GB+ dijeljeno) | 3-4 | `fused_gpu` | Grafički procesori za stolna računala s 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Da (< 12 GB VRAM) | 1-3 | `tiled_gpu` | Početni GPU-ovi, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Ne | jezgre - 1 | `cpu_fallback` | Sustavi bez NVIDIA GPU |

### Vrste cjevovoda

* **`fused_gpu`** — Put obrade punog GPU-a. Sve operacije debayera, ispravljanja i indeksiranja izvode se na GPU-u u jednom spojenom prolazu. Najveća propusnost, ali zahtijeva više VRAM-a.
* **`tiled_gpu`** — Staza GPU-a koja štedi memoriju. Obrađuje slike u pločicama kako bi stale u ograničenu GPU memoriju. Niža propusnost, ali radi na uređajima s ograničenom memorijom.
* **`cpu_fallback`** — obrada samo pomoću CPU-a korištenjem višenitnog paralelizma. Koristi se kada NVIDIA GPU nije dostupan.***

## Ponašanje specifično za platformu

| Platforma | Strategija | Radnici | Cjevovod | Bilješke |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serijski) | Način rada s učinkovitom memorijom, obrađuje jednu po jednu sliku |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (istodobno) | Preporučeni rubni uređaj — stvarna paralelna GPU obrada |
| **Jetson AGX Orin 64GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (istodobno) | Maksimalne rubne performanse |
| **Stolno računalo s 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobre performanse radne površine s pločicama koje štede memoriju |
| **Stolno računalo s 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimalne performanse radne površine |
| **Sustav samo za CPU** | `CPU_PARALLEL` | jezgre - 1 | `cpu_fallback` | GPU nije potreban, koristi ThreadPool |

{% hint style="info" %}
**Jetson objedinjena memorija**: Jetson uređaji dijele GPU i CPU memoriju. Jetson Orin NX 16 GB prijavljuje ~15,3 GB VRAM-a, ali to je isti fizički RAM koji koriste OS i CPU procesi. Chloros to uzima u obzir prilikom postavljanja pragova dodjele memorije.
{% endhint %}

***

## Dinamička dodjela GPU memorije

Chloros koristi [4-nitni cjevovod za obradu](processing-pipeline.md):

* **Nit 1** (otkrivanje) — učitavanje slike, raščlanjivanje EXIF-a, otkrivanje cilja
* **Navoj 2** (kalibracija) — proračun kalibracije refleksije
* **Nit 3** (Obrada) — GPU debayer, ispravak vinjete, izračun indeksa
* **Nit 4** (Izvoz) — Pisanje datoteke, ugrađivanje metapodataka

Kako ranije niti cjevovoda dovrše svoj posao (npr. sve slike su otkrivene), njihova dodjela GPU memorije se oslobađa i **redistribuira na preostale aktivne niti**. To znači da Thread 3 (faza koja zahtijeva GPU) progresivno dobiva više memorije kako cjevovod napreduje, poboljšavajući propusnost za računalno najintenzivniji posao.

### Faze dodjele

| pozornica | Aktivne niti | Distribucija GPU memorije |
| --- | --- | --- |
| **Rano** | 1, 2, 3, 4 | Podijeliti na sve niti |
| **Srednje rano** | 2, 3, 4 | Memorija za nit 1 redistribuirana |
| **Srednje kasno** | 3, 4 | Memorija niti 1+2 ide na 3+4 |
| **Kasno** | 3 ili 4 | Maksimalna memorija za preostalu nit |***

## Obrada svjesna teksture

Texture Aware debayer metoda (samo Chloros+) koristi značajno više GPU memorije od standardne metode zbog AI/ML modela uklanjanja šuma:

* Sustavi s **< 7 GB VRAM-a** prisiljeni su na petlju sinkrone obrade za način rada s obzirom na teksturu (jedna po jedna slika)
* Sustavi s **7 GB+ VRAM-a** mogu istovremeno obrađivati Texture Aware, iako uz manji broj radnika u usporedbi sa standardnim***

## Upravljanje toplinom (Jetson)

Uređaji Jetson imaju toplinska ograničenja, posebno u zatvorenim ili zračnim razmještajima. Chloros prati temperature GPU-a i CPU-a i automatski prilagođava obradu:

| Temperatura | Odgovor |
| --- | --- |
| **< 70°C** | Normalan rad — puna brzina |
| **70°C** (Upozorenje) | Smanjite veličinu serije |
| **80°C** (kritično) | Agresivno prigušivanje — manja konkurentnost i broj radnika |
| **90°C** (isključivanje) | Zaustavi GPU obradu u potpunosti |

Praćenje temperature koristi `tegrastats` na Jetson platformama. Na stolnim sustavima s odgovarajućim hlađenjem, termalno prigušivanje se rijetko pokreće.

***

## Rukovanje pritiskom memorije

Chloros prati pritisak memorije sustava tijekom obrade:

* **Prag memorije**: 85% iskorištenja pokreće konzervativno ponašanje
* **OOM smanjenje**: Ako se dogodi događaj nedostatka memorije, dodjela se smanjuje za 25% (0,75x množitelj)
* **Zamjena cjevovoda**: Pod velikim pritiskom memorije, cjevovod se automatski vraća s `fused_gpu` na `tiled_gpu`
* **Preporuke za zamjenu**: Na Jetsonu, Chloros vas upozorava ako swap prostor nije dovoljan za vašu veličinu skupa podataka***

## Praćenje prilagodbe računala

### CLI statusni izlaz

Kada obrada započne, CLI prikazuje otkriveni hardverski profil:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Dijagnostika sustava

Pokrenite `chloros-cli selftest` da biste vidjeli puni hardverski profil i provjerili računalne mogućnosti:

```bash
chloros-cli selftest
```

Time se provjerava dostupnost CUDA-e, GPU memorija, modeli denoisera i pozadinska povezanost.

***

## Sljedeći koraci

* [Cjevovod za obradu] (processing-pipeline.md) — Razumijevanje arhitekture cjevovoda s 4 niti
* [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — implementacija i optimizacija specifična za Jetson
* [CLI : naredbeni redak](../CLI.md) — Puna referenca CLI