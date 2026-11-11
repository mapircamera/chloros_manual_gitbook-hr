# Dinamička prilagodba računanja


Chloros 1.3.0 koristi detekciju hardvera i automatski odabir strategije obrade. Mehanizam za obradu prilagođava se vašem hardveru — od Jetson Orin Nano do radne stanice s više GPU-a — bez ikakve ručne konfiguracije.

***

## Kako radi

Kada se Chloros pokrene, profilira vaš sustav:

1. **Detektira operativni sustav** — Windows ili Linux
2. **Identificira jezgre CPU-a i ukupnu RAM memoriju**

3.**Detektira prisutnost GPU-a** — NVIDIA CUDA sposobnost, VRAM, model
4. **Identificira model Jetson** (ako je primjenjivo) — putem `/proc/device-tree/model`
5. **Provjerava termičke senzore** (Jetson) — za obradu osjetljivu na temperaturu
6. **Odabire strategiju izračuna** — na temelju sve otkrivene hardver
7. Automatski konfigurira broj radnika, vrstu pipelinea i alokaciju memorije

Detektirani profil pohranjuje se u predmemoriju za sesiju u memoriji i na disku, tako da kasnija pokretanja počinju brže:

| Platforma | Pohranjen profil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (podržava `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Obrišite tu datoteku kako biste prisilili novo otkrivanje — korisno nakon dodavanja GPU-a ili više RAM-a. Chloros također automatski ponovno otkriva kada je predmemoriju napisala neusklađena starija verzija.

***

## Strategije izračuna

Chloros odabire jednu od tri strategije izračuna na temelju vaše hardverne opreme:

| Strategija | Odabrano kada | Radnici | Izvršitelj | Cijev |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA GPU prijavljuje**12 GB+ VRAM-a**(na Jetson jedinstvenoj memoriji, također zahtijeva 12 GB+ ukupne dijeljene RAM memorije) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimalno 2 —**ograničeno na 2 na Jetsonu** | Radni procesi (odvojeni CUDA konteksti) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU s**2-12 GB VRAM-a**| 3 (preklapanje I/O; pristup GPU-u serijaliziran semaforom).**1 (sekvencijalno) na Jetsonima s manje od 12 GB RAM-a** | Radni procesi (odvojeni CUDA konteksti); sekvencijalno unutar procesa na Jetsonu s malo RAM-a | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Bez CUDA GPU-a ili s manje od 2 GB VRAM-a | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Radni primjeri formule za radnika `GPU_PARALLEL`: 12 GB VRAM → 2 radnika, 16 GB → 3 radnika, 20 GB+ → 4 radnika. Svaki radnik dobiva 3,5 GB, što je procijenjena potreba zadatka osjetljivog na teksture (Texture Aware), unutar 90 % preostale VRAM-a nakon rezervacije od 0,5 GB.

{% hint style="info" %}
**GPU strategije zahtijevaju PyTorch koji može koristiti CUDA.** Računalo čiji PyTorch nema upotrebljiv CUDA uređaj dobiva `CPU_PARALLEL`, bez obzira na njegov GPU. Paket Jetson 1.3.0 koristi CUDA izdanje PyTorcha iz JetPacka 6, stoga Jetson zadovoljava uvjete čim su instalirane JetPackove CUDA i cuDNN biblioteke. Jetson s manje od 12 GB RAM-a uvijek obrađuje jednu sliku odjednom, tijekom obrade (zabilježeno kao `GPU_SINGLE`).
{% endhint %}

GPU strategije pokreću skup radnih procesa, od kojih je svaki zaseban OS proces sa svojim CUDA kontekstom, a Chloros ih pokreće ponovnim pokretanjem vlastitog instaliranog pozadinskog programa (ne putem `multiprocessing`-a u Python u, jer pokretanje radnika u njemu ne radi unutar pakirane aplikacije). CPU strategija koristi bazen niti (Pythonov `ThreadPoolExecutor`). Chloros ne koristi nijedan distribuirani okvir treće strane (kao što je Ray).

### Vrste cijevi

* **`fused_gpu`** — Putanja s potpunom obradom na GPU-u. Operacije uklanjanja šuma, korekcije i indeksiranja izvode se na GPU-u u jednom spojenom prolazu. Najveća propusnost, zahtijeva najviše VRAM-a.
* **`tiled_gpu`** — GPU putanja učinkovita u korištenju memorije. Prerađuje slike u pločicama kako bi se uklopile u ograničenu GPU memoriju. Manja propusnost, ali radi na uređajima s ograničenom memorijom.
* **`cpu_fallback`** — Obrada samo na CPU-u pomoću višedretvenog paralelizma. Koristi se kada nijedan NVIDIA GPU nije dostupan i kao rješenje posljednjeg izbora kada oba GPU puta zakažu.

Lanac povratne veze pri pokretanju uvijek je `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ručno preusmjeravanje strategije

Postavite varijablu okruženja `CHLOROS_STRATEGY` kako biste prisilili određenu strategiju — stručno zaobilazno rješenje za slučaj da automatsko otkrivanje odabere nešto neprimjereno vašoj situaciji (na primjer, da GPU ostane slobodan za druge poslove):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Varijabla se uspoređuje neovisno o velikim i malim slovima; sve što nije jedno od ta tri imena zanemaruje se i automatsko otkrivanje se nastavlja normalno. Prilikom premošćivanja, Chloros i dalje bira broj radnika za vas:

| Premošćivanje | Korišteni broj radnika |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Preusmjeravanje na `GPU_SINGLE` ili `GPU_PARALLEL` na računalu bez upotrebljivog CUDA uređaja odbijeno je redkom `[STRATEGY]` u zapisniku, i izvršenje koristi `CPU_PARALLEL`.

Preporučuje se postavljanje po naredbi, a ne trajno, kako bi se normalna izvršenja i dalje automatski prilagođavala.

***

## Ponašanje specifično za platformu

| Platforma | Strategija | Radnici | Cijevovod | Bilješke |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvencijalno) | Mod učinkovite upotrebe memorije, jedna slika odjednom |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvencijalno) | Zajednička RAM memorija manja od 12 GB prisiljava na sekvencijalnu obradu |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (konkurentno) | Preporučeni rubni uređaj — Jetson ograničen na 2 radnika |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (istovremeno) | Maksimalne rubne performanse (također ograničeno na 2 radnika na Jetsonu) |
| **Desktop s 8 GB GPU-a** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 radnika preklapaju I/O dok semafor serijalizira pristup GPU-u |
| **Desktop s GPU-om od 12 GB+** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (istovremeno) | Optimalne performanse radne površine: 12 GB → 2 radnika, 16 GB → 3, 20 GB+ → 4 |
| **Sustav samo s CPU-om** | `CPU_PARALLEL` | fizička jezgra − 1 (min 2) | `cpu_fallback` | GPU nije potreban, koristi skup niti |

{% hint style="info" %}
**Jetson unificirana memorija**: Jetson uređaji dijele GPU i CPU memoriju. Jetson Orin NX 16GB prijavljuje ~15,3 GB VRAM-a, ali to je ista fizička RAM memorija koju koriste OS i CPU procesi. Zato se Jetson uređaji s 16 GB+ kvalificiraju za `GPU_PARALLEL` kao desktop GPU s 12 GB+, no ograničeni su na 2 radnika — GPU, procesi radnika i njihovi CUDA konteksti po radniku svi crpe iz istog zajedničkog fonda.
{% endhint %}

### GPU proračun prema VRAM-u (diskretne GPU-ove)

Na x86_64 hostovima s diskretnim NVIDIA GPU-om, otkriveni VRAM također određuje koliko obrade kartice može zauzeti i koliko velike partije mogu rasti:

| Detektirana VRAM | Gornja granica GPU proračuna | Multiplikator veličine serije |
| --- | --- | --- |
| **8 GB+** | 90% | ×2.0 |
| **6-8 GB** | 85% | ×1.75 |
| **3,5-6 GB** | 80 % | ×1,5 |
| **2-3,5 GB** | 75 % | ×1,25 |
| **Manje od 2 GB** | 70 % | ×1,0 |

Diskretne grafičke kartice rezerviraju samo 0,5 GB za sustav jer ne dijele sistemska RAM-a. Profili za Jetson rezerviraju znatno više i imaju niži gornji limit — pogledajte [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dinamička alokacija GPU memorije

Chloros koristi [pipeline za obradu s 4 niti](processing-pipeline.md):

* **Nit 1** (Detekcija) — Učitavanje slike, parsiranje EXIF-a, detekcija cilja
* **Nit 2** (Kalibracija) — Izračun kalibracije refleksije
* **Nit 3** (Obrada) — GPU debayer, korekcija vinjete, izračun indeksa
* **Nit 4** (Izvoz) — Pisanje datoteka, ugradnja metapodataka

Niti 1, 2 i 4 su blagi potrošači GPU-a; Nit 3 je ona koja zahtijeva najviše resursa. Kako se ranije niti u procesu završavaju, njihov se GPU budžet **preraspodjeljuje preostalim aktivnim nitima**, tako da nit 3 dobiva sve više memorije kako se izvršavanje napreduje.

### Faze alokacije

| Faza | Aktivne niti | Raspodjela GPU memorije |
| --- | --- | --- |
| **Rani** | 1, 2, 3, 4 | Podijeljeno među svim nitima, većina za Nit 3 |
| **Srednje-rani** | 2, 3, 4 | Udio Niti 1 se redistribuiran |
| **Srednje-kasno** | 3, 4 | Udjeli niti 1 i 2 idu 3 i 4 |
| **Kasno** | 3 ili 4 | Posljednja aktivna nit dobiva svoju maksimalnu alokaciju |

Brojke uređuju dva pravila:

* Niti koja je **jedina** aktivna dodjeljuje se maksimalnom alokacijom svog profila.
* Kada je aktivno više *teških* GPU zadataka, osnovna alokacija svakog teškog zadatka dijeli se među njima (nikada ispod njegovog konfiguriranog minimuma).

Stvarna vrijednost koja se koristi tijekom izvođenja je **manja** od dodjele iz profila platforme i preporuke u stvarnom vremenu s monitora GPU memorije, stoga zauzeta grafička kartica uvijek prevladava nad optimističnim profilom.***

## Obrada osjetljiva na teksture

Texture Aware debayer (samo za **Chloros+** — `--debayer texture-aware`) pokreće model za uklanjanje šuma temeljen na umjetnoj inteligenciji/strojnom učenju (AI/ML) kojem je po kopiji potrebno otprilike 1,75 GB VRAM-a u FP16 formatu, stoga koristi znatno više GPU memorije od standardne metode:

* Sustavi s **manje od 7 GB VRAM-a**obrađuju Texture Aware u**sinkronoj petlji, jednu sliku odjednom** — nekoliko kopija modela ne može stati, a bazen radnika bi samo povećao konkurenciju
* Sustavi s **7 GB+ VRAM-a** mogu obrađivati Texture Aware istovremeno, iako s manjim brojem radnika u usporedbi sa Standard metodom
* Na **Jetsonu**, Texture Aware je uvijek vezan za jednog radnika
* Bez **upotrebljivog CUDA GPU-a**, izvršavanje koristi standardni debayer, a pozadinski sustav bilježi jednu liniju `[DEBAYER] … needs a CUDA GPU …` koja navodi razlog
* Na niskopotrošnim Jetson uređajima (Nano, Orin Nano), CLI postavlja ograničenje frekvencije GPU-a kada zatražite Texture Aware — pogledajte [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Upravljanje toplinom (Jetson)

Jetson uređaji imaju toplinska ograničenja, posebno u zatvorenim ili zračnim postavkama. Chloros nadzire ugrađene senzore temperature Jetsona i automatski prilagođava veličinu serija:

| Temperatura | Odgovor |
| --- | --- |
| **< 70 °C** | Normalno funkcioniranje — puna brzina |
| **70 °C** (Upozorenje) | Veličina serije se postupno smanjuje (100 % → 50 % između 70 °C i 80 °C) |
| **80 °C** (Kritično) | Agresivno prigušivanje (50 % → 0 % između 80 °C i 90 °C) |
| **90 °C** (Isključivanje) | Potpuno zaustavljanje obrade na GPU-u |

Na stolnim računalima s adekvatnim hlađenjem termičko usporavanje rijetko se aktivira.

***

## Rješavanje problema s pritiskom na memoriju

Chloros kontinuirano nadzire memoriju GPU-a tijekom obrade i reagira na tri razine.

**Veličina serije.** Serija započinje s 8 slika pomnoženih s platformskim multiplikatorom iz gornjih tablica. Chloros zatim provjerava slobodnu VRAM-u, rezerve 20 % nje za vlastite potrebe PyTorch-a i pretpostavlja otprilike 100 MB GPU memorije po slici od 12 MP — serija je ona manja vrijednost, ograničenje izvedeno iz memorije ili osnovna vrijednost platforme. Nikada ne pada ispod 1.

**Preventivno smanjenje.**Iznad**85 % iskorištenosti VRAM-a**, veličine serija smanjuju se prije nego što dođe do greške.**Smanjenje alokacije po niti.** Kako se povećava trenutačna iskorištenost, GPU proračun svake niti se smanjuje: na ×0,75 pri iskorištenosti iznad 80 % i na ×0,5 pri iskorištenosti iznad 90 %. Pragovi monitora su 70 % (konzervativno), 85 % (normalni radni prag) i 95 % (rizik od OOM-a).**Prigušivanje i oporavak od OOM-a.** Ako dođe do događaja iscrpljenja memorije (out-of-memory):

* veličina serije se **smanjuje na polovicu**, i ponovno se smanjuje na polovicu pri svakom sljedećem OOM-u — svaka sljedeća uspješna serija smanjuje tu kaznu za jedan korak
* alokacije aktivnih niti smanjuju se na 70 % njihove trenutne vrijednosti i alokator se prebacuje na svoju konzervativnu strategiju, opet se opuštajući nakon niza uspješnih alokacija
* pri jakom opterećenju pipeline se vraća s `fused_gpu` na `tiled_gpu`, a kao krajnje sredstvo na `cpu_fallback`

**RAM hosta (Jetson).** Prije obrade, CLI procjenjuje maksimalnu memoriju hosta na temelju vašeg broja slika i načina de-Bayera te upozorava ako je RAM plus swap podržan datotekama vjerojatno nedovoljan, ispisujući točne naredbe za dodavanje swap prostora — pogledajte [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Praćenje prilagodbe računalne snage

### Dijagnostika sustava

`chloros-cli selftest` najbrži je način za potvrdu onoga što sloj za računalne zadatke vidi:

```bash
chloros-cli selftest
```

Njihovih 7 provjera pokriva verziju, dostupnost porta, pokretanje pozadinskog sustava (backend), `/api/test`, informacije o sustavu, postojanje modela za uklanjanje šuma i spremnost CUDA-e i uklanjanja šuma. Provjera 5 izravno ispisuje liniju s hardverom:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Provjera 7 ispisuje `CUDA: <bool>, Denoiser: <bool>` — obje moraju biti istinite da bi Texture Aware uopće bio upotrebljiv.

### Zapisnici pozadinskog sustava

Strategija i broj radnika odabiru se unutar pozadinskog sustava na početku svakog pokretanja — ne postoji baner **CLI a** koji ih objavljuje. Kada se nešto ponaša neočekivano (nazadovanje GPU puta, ponestanak memorije, denojzer koji se ne može učitati), zapisnik pozadinskog sustava za tu sesiju je mjesto gdje se to pojavljuje:

| Platforma | Lokacija dnevnika |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jedna datoteka po pokretanju) |
| **Linux, pozadinski proces pokrenut putem CLI-ja** | također `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Napredak uživo

Tijekom izvođenja, CLI prikazuje trenutačni napredak za svaki thread (otkrivanje, analiza, obrada, izvoz) koji se prenosi putem Server-Sent Events — što je praktičan način da provjerite je li thread 3 usko grlo. Pogledajte [Processing Pipeline](processing-pipeline.md).

***

## Sljedeći koraci

* [Procesni vod](processing-pipeline.md) — Razumijevanje arhitekture procesnog voda s 4 niti
* [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Specifično postavljanje i optimizacija za Jetson
* [CLI: Command Line](../CLI.md) — Vodič za CLI

* [CLI Reference](../reference/cli-reference.md) — Potpuni popis naredbi za 1.3.0
