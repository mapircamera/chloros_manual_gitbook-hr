# Rešetka slika

Nakon uvoza slika u projekt vidjet ćete ih raspoređene u rešetku u glavnom području. U rešetki odabirete **koju verziju svake slike gledate** — gumbe iznad nje možete koristiti za istovremenu zamjenu svake sličice između izvornog datoteka i svakog obrađenog proizvoda.

## Veličina minijatura

Koristite klizač za zumiranje u gornjem desnom kutu za prilagodbu veličine minijatura slika. Klizač se proteže od **64 px do 1200 px**.

* **Ctrl + kotačić miša** također mijenja veličinu minijatura.
* **Ctrl + `+`**/**Ctrl + `=`**i**Ctrl + `−`** mijenjaju veličinu za 4 px pri svakom pritisku. Putanja tipkovnice zaustavlja se na 64 px na manjoj strani, a na većoj strani na veličini koja točno stane dva minijatura u red u trenutnom prozoru.
* Odabrana veličina sprema se s projektom (`UI → Grid thumbnail size` u `project.json`, zadano `160`), pa se pri ponovnom otvaranju projekta vraća na prethodnu vrijednost.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Razlučivost* sličice je zasebna postavka od *veličine* sličice: pogledajte **Prikaz → Razlučivost sličice slike** u [Postavke projekta](../project-settings/project-settings.md) (zadano 512 px po dužoj strani). Veličina je koliko je velika pločica iscrtana; rezolucija je koliko se detalja dohvaća za njezino popunjavanje.

***

## Alatna traka mreže

Red gumba iznad mreže ima do tri grupe, slijeva nadesno:

1. **Po okidaču / Po kameri** — način grupiranja. Pojavljuje se samo za projekte koji sadrže snimke LATTICE.
2. **Gumbi za filtriranje po kameri** — jedan za svaku LATTICE kameru. Pojavljuje se samo u načinu rada Po kameri.
3. **Gumbi načina izvoza/prikaza** — koji proizvod prikazuje svaka sličica.

Kada je prozor preuski za sve, grupe se sklapaju s desna ulijevo u padajuće izbornike pri prelasku mišem: prvo se sklapaju gumbi za izvoz/prikaz, a zatim gumbi kamere. Sklopljena grupa ostavlja jedan gumb s oznakom trenutno aktivne opcije, a prilikom prelaska mišem preko njega prikazuje se cijeli skup. **Per Trigger / Per Camera se nikada ne sklapaju.**<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

***

## Gumbi za prikaz izvoza

Ovi gumbi prebacuju sličice u mreži između vrsta slika. **Gumb se pojavljuje čim proizvod kojem je ime postoji** — što za izvornu datoteku znači odmah pri uvozu, a ne nakon obrade.Chloros ponovno skenira proizvode projekta dok je izvršavanje u tijeku, pa se gumbi pojavljuju tijekom obrade kako svaki proizvod počinje biti spremljen na disk.

### Osnovno gumb

Najljevije gumb za izvoz označen je za **ono što ste zapravo uvezli**:

| Ono što ste uvezli | Označje gumba |
| --- | --- |
|Survey3 RAW+JPG | `JPG` |
| LATTICE snima s pregledom na zaslonu uz raw kadar | `PNG` ili `TIFF`, ovisno o pregledima |
| LATTICE snimke gdje je osnovna datoteka **sami** raw kadar | *nema gumba* — `RAW (Original)` već prikazuje tu datoteku |

U mješovitom projektu oznaka slijedi ekstenziju koju koristi većina slika.

### Gumbi proizvoda

| Gumb | Prikazuje | Kada se pojavljuje |
| --- | --- | --- |
| **Ciljevi** | Slike s otkrivenim kalibracijskim ciljem | Nakon izvođenja koje je otkrilo ciljeve |
| **Reflektancija** | Kalibrirane slike reflektancije | Samo za projekte s kamerama Survey3 — projekti LATTICE umjesto toga koriste `RAW (Reflectance)`, stoga se mreža nikada ne prikazuje s dva gumba za reflektanciju |
| **Balančirano po bijeloj** | Proizvod balansiran po bijeloj (kamereRGB) | Nakon obrade |
| **Korekcija vinjete** | Zadana opcija za nekorigiranu vinjetu | Nakon obrade kada se kalibracija reflektancije nije mogla primijeniti i *Korekcija vinjete* je bila uključena |
| **Odziv senzora** | Neokalibrirana nadomjesna vrijednost za odziv senzora | Isto, ali s isključenom *Korekcijom vinjete* |
| **`RAW (<INDEX> Index)`** | Jedno dugme po mapiranom indeksu boja | Nakon izvođenja s konfiguriranim indeksima |
| **`<INDEX> LUT`** | Jedno gumbića po indeksu s mapiranjem boja | Nakon izvođenja s konfiguriranom LUT-om |
| **`<Index> <Index\|LUT> <NNN>`** | Jedno gumbom po izvršenju izvoza za [Index/LUT Sandbox](index-lut-sandbox.md) | U trenutku kada se izvoz sandboxa završi |

### Gumbi razine LATTICE

Projekti koji sadrže snimke LATTICE dodaju ove gumbe, označene nazivom razine umjesto nazivom proizvoda:

| Gumb | Razina |
| --- | --- |
| **RAW (Original)** | Izvorni sirovi kadar, kao što je uvezen |
| **RAW (Radijanca)** | Float32 spektralna radijanca, W/m²/sr/nm |
| **RAW (Odbijanje)** | uint16 odbijanje, 32768 = ρ 1.0 |

`RAW (Original)` je dostupan od trenutka uvoza — ne zahtijeva obradu. Kada uvoz LATTICE-a nema nikakvo osnovno gumbom (osnovna datoteka svake snimke je njezin sirovi kadar), mreža se sama pomakne na prvi dostupni gumb razine kako bi istaknuta stavka na alatnoj traci odgovarala onome što vidite.

Izvozi dviju razinaChloros

-a **nemaju vlastito gumb mreže**:

* **Debayered** — prikaz `RAW (Original)`-a već prikazuje debayered sliku, stoga bi drugi gumb na vizualno identičnoj slici bio suvišan. Proizvod `RAW (Debayered)` i dalje se zapisuje na disk i i dalje je odabran iz padajućeg izbornika sloja na cijelom zaslonu.
* **Pregled** — na kameramaRGB

pregled je registriran kao sloj `White Balanced`, koji ima gumb. Na multispektralnim kamerama registrira se kao `RAW (Preview)` i dostupan je iz padajućeg izbornika slojeva punog zaslona.

{% hint style="info" %}
Ovi gumbi razine prikazuju se samo za projekte koji doista sadrže LATTICE okvire. ProjektiSurvey3

registriraju neka od istih internih imena slojeva, pa su gumbi za njih filtrirani, tako da mrežaSurvey3

zadržava svoj poznati skup `JPG / Targets / Reflectance`.
{% endhint %}

Klikom na minijaturu mreže otvara se [Prikazivač slika](opening-an-image-full-screen.md) na cijelom zaslonu u **istom proizvodu koji mreža prikazuje** — ako je mreža postavljena na `Targets`, minijatura otvara izvezanu odredišnu sliku.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->


***

## Grupiranje LATTICE projekta: Po okidaču vs. po kameri

Array captures proizvode nekoliko slika istog trenutka iz različitih modula kamera. Grupiranje određuje kako će ih mreža složiti. Oba načina rada prikazuju sklopive zaglavne trake pune širine; **svaka grupa se u početku prikazuje otvorena**, aChloros
pamti one koje zatvorite. Stanje sklopljenosti prati se odvojeno za svaki način rada, stoga zatvaranje grupe u načinu rada Po kameri ne zatvara ništa u načinu rada Po okidaču.

### Po kameri (zadano)

Jedna grupa po kameri. Naslovnica prikazuje model i serijski broj kamere (`LATT-M3M — <serial>`) te broj fotografija. Pločice unutar grupe poredane su kronološki prema događaju snimanja.

U ovom načinu rada alatna traka dobiva i jedno **dugme filtra kamere po kameri**, označeno s `MODEL (SERIAL)`. Sve kamere su isprva odabrane; klikom na gumb odabir se poništava za tu kameru i uklanja njezinu grupu iz mreže. Ovo je brz način pregleda jedne trake preko cijelog leta.

### Po okidaču

Jedna grupa po događaju snimanja — skup okvira koje su svi moduli snimili pri istom okidaču. Zaglavlje prikazuje vrijeme snimanja, broj kamera koje su doprinijele te oznaku za svaki model kamere u grupi. Kvadrati unutar grupe poredani su prema serijskom broju kamere, tako da se ista banda nalazi u istom stupcu za svaki okidač.

Slike koje nisu u<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
LATTICE mreži u mješovitom projektu nisu grupirane — prikazuju se kao obični kvadrati nakon grupa.

***

## Minijature mreže prate veličinu GSD bloka

Ako ste u bočnoj traci kartice slike postavili veličinu bloka **GSD (px)**, minijature mreže prikazuju se pri istoj razlučivosti tla — ne samo prikaz u punom zaslonu. Veličina bloka od 8 znači da je svaki prikazani piksel prosjek bloka od 8 × 8 izvornih piksela, svugdje u aplikaciji koja prikazuje sliku.

Budući da je pločica u početku široka samo nekoliko stotina piksela, grube veličine blokova prestaju stvarati vidljivu razliku na mreži mnogo prije nego što to učine u prikazu na cijelom zaslonu: kadar od 4000 px iscrtan u pločicu od 160 px već je na otprilike 25 izvornika po prikazanom pikselu. Pogledajte [Otvaranje slike u punom zaslonu](opening-an-image-full-screen.md#gsd-block-size) za samu kontrolu.

***

## Povezane stranice

* [**Otvaranje slike u punoj veličini**](opening-an-image-full-screen.md) — preglednik u punoj veličini, vrijednosti pokazivača i histogram
* [**Slojevi slike**](image-layers.md) — padajući izbornik slojeva unutar preglednika na cijelom zaslonu
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — izrada i izvoz indeksnih vizualizacija
* [**Postavke projekta**](../project-settings/project-settings.md) — prekidači za izvoz koji određuju postoje li proizvodi uopće
