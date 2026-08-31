# Dinamička prilagodba računanja

Chloros
1.2.0 koristi detekciju hardvera i automatski odabir strategije obrade. Mehanizam za obradu prilagođava se vašem hardveru — od Jetson Orin Nano do radne stanice s više GPU-a — bez ikakve ručne konfiguracije.

***

## Kako radi

Kada se pokreneChloros
, on profilira vaš sustav:

1. **Detektira operativni sustav** —Windows
iliLinux
2. **Identificira jezgre CPU-a i ukupnu RAM memoriju**

3.**Detektira prisutnost GPU-a** — NVIDIA CUDA sposobnost, VRAM, model
4. **Identificira model Jetson** (ako je primjenjivo) — putem `/proc/device-tree/model`
5. **Provjerava termičke senzore** (Jetson) — za obradu osjetljivu na temperaturu
6. **Odabire strategiju izračuna** — na temelju sve otkrivene hardverske opreme
7. **Automatski konfigurira broj radnika, vrstu cjevovoda i alokaciju memorije**

Otkriveni profil pohranjuje se u predmemoriju za sesiju u memoriji i na disku, tako da kasniji pokreti započinju brže:

| Platforma | Pohranjen profil |
| --- | --- |
| **Linux
/ Jetson** | `~/.config/chloros/system_config.json` (podržava `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Obrišite tu datoteku kako biste prisilili novo otkrivanje — korisno nakon dodavanja GPU-a ili više RAM-a.Chloros
također automatski ponovno otkriva kada je predmemoriju napisala neusklađena starija verzija.

***

## Strategije za obradu

Chloros
odabire jednu od tri strategije za obradu na temelju vaše hardverne opreme:

| Strategija | Odabrano kada | Radnici | Izvršitelj | Cijev |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA GPU prijavljuje**12 GB+ VRAM-a**(na Jetson jedinstvenoj memoriji, također zahtijeva 12 GB+ ukupne zajedničke RAM-a) | `min(4, VRAM ÷ 4GB)`, minimalno 2 —**ograničeno na 2 na Jetsonu** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU s**2-12 GB VRAM-a**| 3 (preklapanje I/O; pristup GPU-u serijaliziran semaforom).**1 (sekvencijalno) na Jetsonima s manje od 12 GB RAM-a** | `ProcessPoolExecutor` (spawn); sekvencijalno unutar procesa na Jetsonu s malo RAM-a | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Bez CUDA GPU-a ili s manje od 2 GB VRAM-a | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Radni primjeri formule za radnike `GPU_PARALLEL`: 12 GB VRAM-a → 3 radnika, 16 GB+ → 4 radnika, bilo koji Jetson → 2 radnika.

Paralelizam je implementiran standardnim `concurrent.futures`-om izPython
-a: GPU strategije koriste `ProcessPoolExecutor` s metodom pokretanja **spawn** (svaki radnik je zaseban proces sa svojim CUDA kontekstom — `fork` bi kopirao već inicijalizirano CUDA stanje i oštetio podređene procese), a CPU strategija koristi `ThreadPoolExecutor`.Chloros
ne koristi nijedan distribuirani okvir treće strane (kao što je Ray).

### Vrste cijevi

* **`fused_gpu`** — Potpuna GPU putanja za obradu. Operacije uklanjanja šuma, korekcije i indeksiranja izvode se na GPU-u u jednom spojenom prolazu. Najveća propusnost, zahtijeva najviše VRAM-a.
* **`tiled_gpu`** — GPU putanja učinkovita u korištenju memorije. Prerađuje slike u pločicama kako bi stala u ograničenu GPU memoriju. Manja propusnost, ali radi na uređajima s ograničenom memorijom.
* **`cpu_fallback`** — Obrada isključivo na CPU-u pomoću višedretvene paralelnosti. Koristi se kada nijedan NVIDIA GPU nije dostupan i kao krajnje rješenje kada oba GPU puta zakažu.

Lanac rezervnih rješenja u vrijeme izvođenja uvijek je `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ručno preusmjeravanje strategije

Postavite varijablu okruženja `CHLOROS_STRATEGY` kako biste prisilili određenu strategiju — stručna sigurnosna opcija za slučaj da automatsko otkrivanje odabere nešto neprimjereno vašoj situaciji (na primjer, da GPU ostane slobodan za druge zadatke):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Varijabla se uspoređuje bez obzira na velika i mala slova; sve što nije jedno od ta tri naziva zanemaruje se i automatsko otkrivanje se nastavlja normalno. Prilikom ručnog premošćivanja,Chloros
i dalje bira broj radnika za vas:

| Preusmjeravanje | Koristi se broj radnika |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Preporučuje se postavljanje po naredbi, a ne trajno, kako bi se normalna izvođenja i dalje automatski prilagođavala.

***

## Ponašanje specifično za platformu

| Platforma | Strategija | Radnici | Cijevovod | Bilješke |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvencijalno) | Mod učinkovite upotrebe memorije, jedna slika odjednom |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvencijalno) | Zajednička RAM memorija manja od 12 GB prisiljava na sekvencijalnu obradu |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (konkurentno) | Preporučeni rubni uređaj — Jetson ograničen na 2 radnika |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (istovremeno) | Maksimalne rubne performanse (također ograničeno na 2 radnika) |
| **Desktop s 8GB GPU-om** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 radnika preklapaju I/O dok semafor serijalizira pristup GPU-u |
| **Desktop s GPU-om od 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` (istovremeno) | Optimalne performanse radne stanice: 12 GB → 3 radnika, 16 GB+ → 4 |
| **Sustav samo s CPU-om** | `CPU_PARALLEL` | fizička jezgra − 1 (min 2) | `cpu_fallback` | GPU nije potreban, koristi skup niti |

{% hint style="info" %}
**Jedinstvena memorija Jetsona**: Jetson uređaji dijele memoriju GPU-a i CPU-a. Jetson Orin NX 16GB prijavljuje ~15,3 GB VRAM-a, ali to je ista fizička RAM memorija koju koriste OS i CPU procesi. Zato Jetson uređaji s 16 GB+ memorije ispunjavaju uvjete za `GPU_PARALLEL` kao desktop GPU s 12 GB+, no ograničeni su na 2 radnika — GPU, procesi radnika i njihovi CUDA konteksti po radniku svi crpe iz istog zajedničkog fonda.
{% endhint %}

### GPU proračun prema VRAM-u (diskretne GPU-ove)

Na x86_64 hostovima s diskretnim NVIDIA GPU-om, otkriveni VRAM također određuje koliko obrade kartice može zauzeti i koliko velike partije mogu rasti:

| Detektirana VRAM | Gornja granica GPU proračuna | Multiplikator veličine serije |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6-8 GB** | 85 % | ×1,75 |
| **3,5-6 GB** | 80 % | ×1,5 |
| **2-3,5 GB** | 75 % | ×1,25 |
| **Manje od 2 GB** | 70 % | ×1,0 |

Diskretne grafičke kartice (GPU-ovi) rezerviraju samo 0,5 GB za sustav jer ne dijele RAM sustava. Profili za Jetson rezerviraju znatno više i imaju niže gornje granice — pogledajte [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dinamička alokacija GPU memorije

Chloros
koristi [procesnu cijev s 4 niti](processing-pipeline.md):

* **Nit 1** (Detekcija) — Učitavanje slike, parsiranje EXIF-a, detekcija cilja
* **Nit 2** (Kalibracija) — Izračun kalibracije reflektancije
* **Nit 3** (Obrada) — GPU debayer, korekcija vinjete, izračun indeksa
* **Nit 4** (Izvoz) — Pisanje datoteka, ugradnja metapodataka

Niti 1, 2 i 4 su blage potrošačice GPU-a; Nit 3 je ona koja zahtijeva najviše resursa. Kako se ranije niti u procesu završavaju, njihov se GPU budžet **preraspodjeljuje preostalim aktivnim nitima**, tako da nit 3 dobiva sve više memorije kako se izvršavanje napreduje.

### Faze alokacije

| Faza | Aktivne niti | Raspodjela GPU memorije |
| --- | --- | --- |
| **Rani** | 1, 2, 3, 4 | Podijeljeno među svim nitima, većina za Nit 3 |
| **Srednje-rani** | 2, 3, 4 | Udio Niti 1 se preraspodjeljuje |
| **Srednje-kasno** | 3, 4 | Udjeli niti 1 i 2 idu nitima 3 i 4 |
| **Kasno** | 3 ili 4 | Posljednja aktivna nit dobiva svoju maksimalnu alokaciju |

Brojke uređuju dva pravila:

* Niti koja je **jedina** aktivna dobiva maksimalnu alokaciju svog profila.
* Kada je aktivno više *teških* GPU zadataka, osnovna alokacija svakog teškog zadatka dijeli se među njima (nikada ispod njegovog konfiguriranog minimuma).

Stvarna vrijednost koja se koristi tijekom izvođenja je **manja** od dodjele iz profila platforme i preporuke u stvarnom vremenu s monitora GPU memorije, tako da zauzeta kartica uvijek pobjeđuje optimistični profil.***

## Obrada osjetljiva na teksture

Texture Aware debayer (samo za **Chloros
+** — `--debayer texture-aware`) pokreće model za uklanjanje šuma temeljen na umjetnoj inteligenciji/strojnom učenju (AI/ML) kojem je po kopiji potrebno otprilike 1,75 GB VRAM-a u FP16 formatu, stoga koristi znatno više GPU memorije od standardne metode:

* Sustavi s **manje od 7 GB VRAM-a**obrađuju Texture Aware u**sinkronoj petlji, jednu sliku odjednom** — nekoliko kopija modela ne može stati, a bazen radnika samo bi stvorio sukob
* Sustavi s **7 GB+ VRAM-a** mogu obrađivati Texture Aware istovremeno, iako s manjim brojem radnika u usporedbi sa Standard metodom
* Na **Jetson**, Texture Aware je uvijek dodijeljen jednom radniku, a na modelima s niskom potrošnjom energije (Nano, Orin Nano) automatski se primjenjuje ograničenje frekvencije GPU-a — pogledajte [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Termičko upravljanje (Jetson)

Jetson uređaji imaju termička ograničenja, posebno u zatvorenim ili zračnim okruženjima.Chloros
nadzire ugrađene senzore temperature Jetson uređaja i automatski prilagođava veličinu serija:

| Temperatura | Odgovor |
| --- | --- |
| **&lt; 70 °C** | Normalno funkcioniranje — puna brzina |
| **70 °C** (Upozorenje) | Veličina serije se postupno smanjuje (100 % → 50 % između 70 °C i 80 °C) |
| **80 °C** (Kritično) | Agresivno ograničavanje (50 % → 0 % između 80 °C i 90 °C) |
| **90 °C** (Isključivanje) | Potpuno zaustavljanje obrade GPU-a |

Na stolnim računalima s adekvatnim hlađenjem termičko usporavanje rijetko se aktivira.

***

## Rješavanje problema s pritiskom na memoriju

Chloros
kontinuirano nadzire memoriju GPU-a tijekom obrade i reagira na tri razine.

**Veličina serije.** Serija započinje s 8 slika pomnoženih s platformskim multiplikatorom iz gornjih tablica.Chloros
zatim provjerava slobodnu VRAM-u, rezervira 20 % nje za vlastite potrebe PyTorch-a i pretpostavlja otprilike 100 MB GPU memorije po slici od 12 MP — veličina serije je ona manja vrijednost, ograničenje iz memorije ili osnovna vrijednost platforme. Nikada ne pada ispod 1.

**Preventivno smanjenje.**Iznad**85% iskorištenosti VRAM-a**, veličine serija smanjuju se prije nego što nešto zakaže.**Prilagodba alokacije po niti.** Kako se trenutačna iskorištenost povećava, GPU proračun svakog niti se smanjuje: na 0,75 × iznad 80 % iskorištenosti i na 0,5 × iznad 90 %. Granice monitora su 70 % (konzervativno), 85 % (normalno radno ograničenje) i 95 % (rizik od OOM-a).**OOM odgađanje i oporavak.** Ako se unatoč tome dogodi događaj nedostatka memorije:

* veličina serije se **smanjuje na polovicu**, i ponovno se smanjuje na polovicu pri svakom sljedećem OOM-u — svaka sljedeća uspješna serija poništava taj kazneni koeficijent za jedan korak
* aktivne alokacije niti smanjuju se na 70 % svoje trenutačne vrijednosti i alokator se prebacuje na svoju konzervativnu strategiju, opet se opuštajući nakon niza uspješnih alokacija
* pri jakom opterećenju pipeline se vraća s `fused_gpu` na `tiled_gpu`, a kao krajnje sredstvo na `cpu_fallback`

**RAM hosta (Jetson).** Prije obrade,CLI
procjenjuje maksimalnu količinu host memorije na temelju broja vaših slika i načina rada debayera te upozorava ako je RAM plus swap podržan datotekama vjerojatno nedovoljan, ispisujući točne naredbe za dodavanje swap prostora — pogledajte [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Praćenje prilagodbe računanja

### Dijagnostika sustava

`chloros-cli selftest` najbrži je način za potvrdu onoga što sloj za računanje vidi:

```bash
chloros-cli selftest
```

Njegovih 7 provjera pokriva verziju, dostupnost porta, pokretanje pozadinskog sustava, `/api/test`, informacije o sustavu, prisutnost modela za uklanjanje šuma i spremnost CUDA + uklanjanja šuma. Provjera 5 izravno ispisuje liniju s hardverom:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Provjera 7 ispisuje `CUDA: <bool>, Denoiser: <bool>` — obje moraju biti istinite da bi Texture Aware uopće bio upotrebljiv.

### Zapisnici pozadinskog sustava

Strategija i broj radnika odabiru se unutar pozadinskog sustava na početku svake izvedbe — ne postoji baner zaCLI
koji ih objavljuje. Kada se nešto ponaša neočekivano (nazadovanje GPU staze, OOM, denojzer koji se nije htio učitati), to se pojavljuje u dnevniku pozadinskog sustava za tu sesiju:

| Platforma | Lokacija dnevnika |
| --- | --- |
| **Linux
/ Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jedna datoteka po pokretanju) |
| **Linux
,CLI
-pokrenut backend** | također `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Napredak uživo

Tijekom izvršavanja,CLI
prikazuje napredak uživo za svaku nit (otkrivanje, analiza, obrada, izvoz) koji se prenosi putem Server-Sent Events — praktičan uvid u to je li nit 3 usko grlo. Pogledajte [Processing Pipeline](processing-pipeline.md).

***

## Sljedeći koraci

* [Procesna cijev](processing-pipeline.md) — Razumijevanje arhitekture cijevi s 4 niti
* [Vodič za NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Postavljanje i optimizacija specifična za Jetson
* [CLI
: Command Line](../CLI.md) — Vodič zaCLI

* [CLI
Reference](../reference/cli-reference.md) — Potpuna lista naredbi za 1.2.0
