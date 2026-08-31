---
metaLinks: {}
---

# Početak rada

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

je softverska aplikacija tvrtke [MAPIR

](https://www.mapir.camera) za obradu multispektralnih snimaka, daljinski nadzor hardveraMAPIR

i snimanje podataka senzora.Chloros

1.2.0 podržava cijelu obitelj proizvodaMAPIR

:

* **Survey3

kamere** — obrađuju snimke u RAW+JPG formatu u kalibrirane karte reflektancije i indeksa vegetacije. Pogledajte [Podržane kamere](supported-cameras.md).
* ****kamere LATTICE** — povezujte GigE multispektralne kamere uživo, pojedinačno ili kao sinkronizirane mreže više kamera: za pregled, snimanje i obradu u kalibrirane proizvode radijance i reflektance. Pogledajte [odjeljak LATTICE](lattice/README.md).
* **DAQ senzori svjetla** — spektralni senzori DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet): kalibrirani spektri uživo, snimke `.daq` i dolje usmjerena rasvjeta za obradu reflektancije. Pogledajte [odjeljak DAQ](daq/README.md).

{% hint style="success" %}
**Novosti u verzijiChloros

1.2.0**: upravljanje LATTICE kamerom i nizom uživo, integracija DAQ senzora svjetlosti, načini snimanja i snimači, potpuni LATTICE radiometrijski procesni niz, automatizacija projekta izCLI

/SDK

i još mnogo toga. Pogledajte popis novosti u nastavku i [Preuzmite](download.md) dnevnik promjena.
{% endhint %}

{% hint style="info" %}
**Koristite liChloros

s AI asistentom?** Ovaj je priručnik napravljen za to. Uputite svog asistenta na:

* `https://mapir.gitbook.io/chloros/llms.txt` — mašinski čitljiv indeks svake stranice.
* Bilo koju stranicu kao sirovi Markdown — dodajte `.md` na njezinURL

(npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI

referencu](reference/cli-reference.md) i [SDK

referencu](reference/sdk-reference.md) — potpune referentne stranice s točnim vrijednostima napisane za upotrebu s LLM-om.

Primjer upita: *&quot;Pročitaj https://mapir.gitbook.io/chloros/reference/cli-reference.md,, a zatim napiši skriptu koja se prijavljuje i obrađuje mapu ~/flights/flight_001 u reflectance iNDVI

GeoTIFF formate.&quot;*

Cjeloviti vodič: [KorištenjeChloros

s AI asistentima](ai-assistants.md).
{% endhint %}

***

## Novosti uChloros

1.2.0

* **Upravljanje kamerom uživo — novi kartica Kamere.** Povežite LATTICE kamere jednu po jednu ili kao sinkronizirane mreže više kamera (PTP vremenska sinkronizacija, snimanje okidačem na hardveru), s prikazom predviđanja uživo s oznakama, histogramima po pojasu, pametnom automatskom ekspozicijom, kalkulatorom indeksa uživo i ažuriranjima firmvera kamere unutar aplikacije.
* **Senzori svjetla — novi kartica Senzori svjetla.** Povežite senzore DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet); prikaz kalibriranih spektra uživo (W/m²/nm), snimanje `.daq` datoteka u vaš projekt, odabir profila za korekciju snimanja i ažuriranje firmvera DAQ-E putem mreže.
* **Načini snimanja i snimači.** Pojedinačno / kontinuirano / intervalno snimanje, plus način najbržeg snimanja samo u sirovom formatu; projektni odabir kamera i vrsta izvoza koje &quot;Snimanje svega&quot; proizvodi; nizni snimači za indeksni video razine nadzora i sirove burstove razine analize s izvanmrežnim izradama videozapisa.
* **LATTICE pipeline za obradu.** Uvozite mape snimanja LATTICE i raspakirajte svaki sirovi kadar u debayered, pretpregled, float32 radijansu (W/m²/sr/nm) i proizvode od reflektancije s prekidačima po proizvodu. Odraznost može potjecati od kalibracijske pločice unutar kadra ili od DAQ-ovog zračnog sjaja; poravnanje niza primjenjuje se na izvoze; nedostajuća tvornička kalibracija automatski se preuzima prema serijskom broju kamere.
* **Projekti pamte hardver.** Povezane kamere i senzori svjetla spremaju se s projektom (`cameras.json` / `sensors.json`) i ponovno se povezuju sa svojim spremljenim postavkama kada ponovno otvorite projekt. Vidi [GUI : Projects](projects.md).
* **Nadogradnje preglednika slika.** Čitanje piksela/indeksa pokazivača s ispravnim skaliranjem reflektancije po datoteci, histogrami slojeva, klizač za biniranje GSD-a, mrežni načini rada po okidaču / po kameri, prikazi LATTICE proizvoda i izvoz u sandboxu indeksa/LUT-a na disk.
* **CLI

iSDK

znatno su prošireni.** Novi `lattice`, `daq pool-*`, `project` i `time-sync` obitelji naredbi; nove opcije `process` (`--input-level`, prekidači po proizvodu, `--reflectance-source`, zastavice poravnanja nizova);SDK

rukohvati smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) koji automatski pokreću pozadinu; automatizacija `open_project()`; kotačićSDK

priložen je instalacijskim paketima i objavljen na PyPI-ju kao `chloros-sdk`.
* **Poštena semantika pogrešaka.** Pokretanje `chloros-cli process`-a koje je zatražilo proizvode, ali nije napisalo nijedan, sada glasno greši i izlazi s ne-nulovnim rezultatom; uspješna pokretanja izvještavaju o tome koliko je slikovnih proizvoda napisano.
* **Novi raspored izlaza.** Proizvodi se spremaju u mape `<project>/<camera>/<format>/<Product>_Images/` i zadržavaju naziv izvornog datoteka — proizvod se identificira mapom, a ne nastavkom naziva datoteke. Pogledajte [Formatima izlaznih slika](output-image-formats.md).
* **Više ulaza, planova i jezika.** Podrška za ulaze `.dng`; svih 38 jezika sučelja u potpunosti je prevedeno; hardverska ograničenja po planu s besplatnom (bez prijave) upotrebom do 4 kamere i 2 senzora svjetla.
* **Pouzdanost.** Zaustavljanje obrade se odvija uredno uz iskren sažetak izvođenja, projekti s više kamera izvoze svaku kameru, a nadogradnje instalatera vas više ne odjavljuju.***

Chloros

dostupan je u 3 sučelja aplikacije:

##Chloros

: Desktop GUI aplikacija

Samostalni prozor sa svim značajkama, uključujući kartice za kamere uživo i senzore svjetla. _Samo za Windows._

## [Chloros

CLI

: Sučelje naredbenog retka](CLI.md)

Serijska obrada putem naredbenog retka uz naredbe u stvarnom vremenu `lattice`, `daq pool-*`, `project` i `time-sync`. Savršeno za automatizaciju, skriptanje i rad bez zaslona. Dostupno na **Windows

,Linux

amd64 iLinux

arm64 (NVIDIA Jetson)**. _CLI zahtijeva plaćeniChloros

+ paket za pristup._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Programski sučelje zaPython

za automatizaciju i prilagođene tijekove rada: obrada cijelog procesa, sesije kamere/nizova uživo, sesije DAQ senzora i automatizacija spremljenih projekata. Instaliran uz desktop paketCLI

i također objavljen kao `pip install chloros-sdk`. API zahtijeva plaćeniChloros

+ paket za pristup.

***

## Podržane platforme

| Platforma | GUI |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Da | Da | Da |
| **Linux

amd64 (x86_64)** | Ne | Da | Da |
| **Linux

arm64 (NVIDIA Jetson)** | Ne | Da | Da |

Upute za instalaciju zaLinux

pogledajte u odjeljku [Linux

&amp; Edge Computing](linux/linux-overview.md).

***

## Početak u tri koraka

1. **Instalacija** — preuzmite i pokrenite instalacijski program za svoju platformu. Pogledajte [Preuzimanje](download.md).
2. **Prijavite se (opcionalno za GUI)** — GUI obrađuje slike besplatno bez računa. [Prijava naChloros

+](chloros+-login.md) otključava paralelnu obradu, ubrzanje GPU-a, veće granice za uređaje i pristupCLI

/SDK

.
3. **Stvorite svoj prvi projekt** — otvoriteChloros

, stvorite [Novi projekt](projects.md), [dodajte svoje slike](processing-images-gui/adding-files-to-a-project.md) i [pokrenite obradu](processing-images-gui/starting-the-processing.md). Za upravljanje live hardverom umjesto toga otvorite karticu Kamere ili Senzori svjetla — pogledajte [GUI : Navigacija](navigation.md).

***

##Chloros

+

Iako jeChloros

besplatan za većinu zadataka, možda ćete poželjeti više. Tu vam može koristiti plaćena licenca zaChloros

+. S licencom zaChloros

+ možete otključati nove značajke kao što su:

* **Višenitna obrada**: znatno ubrzajte obradu slika za veće projekte istovremenom obradom slika kroz cijeli proces.
* **Acceleracija pomoću GPU-a (CUDA)**: iskoristite današnje opcije veće GPU memorije kako biste dodatno ubrzali proces obrade slika. Za najbolje rezultate preporučujemo 4 GB ili više VRAM-a.
* **Chloros

+**[**CLI**](CLI.md) **Pristup**: pokreniteChloros

+ iz naredbenog retka kako biste automatizirali i integrirali u vlastiti softver. Dostupno na svim plaćenim razinama; provodi se na strani poslužitelja.
* **Chloros

+**[**API**](api-python-sdk.md) **Pristup:** pokreniteChloros

+ s naredbenog retka za programatsku kontrolu, omogućujući besprijekornu integraciju s vašim istraživačkim procesima, tijekovima rada za analizu podataka i prilagođenim aplikacijama. Dostupno na bilo kojoj plaćenoj pretplati; provodi se na strani poslužitelja.
* **Viša ograničenja hardvera**: povežite više kamera i senzora svjetla odjednom. Bez prijave GUI povezuje do 4 kamere i 2 DAQ senzora svjetla; plaćeni paketi povećavaju oba ograničenja:

| Plan | Kamere | DAQ senzori svjetla |
| --- | --- | --- |
| Iron (besplatno, bez prijave) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Korištenje više uređaja**: svaka licencaChloros

+ omogućuje registraciju 2+ uređaja. Koristite svoj račun u oblakuMAPIR

za upravljanje registriranim uređajima. Dodajte podršku za više uređaja nadogradnjom svoje licenceChloros

+.
* **Napredna metoda debayeriranja osjetljiva na teksture:** visokokvalitetna metoda debayeriranja osjetljiva na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji (AI/ML) koji uklanja gotovo sav šum nastali debayeriranjem.
* **Prilagođene formule za višespektralne indekse:** unesite prilagođene višespektralne indekse u rasterske kalkulatoreChloros

, i za obradu i za sandbox za pregled slika.
* **Linux

i računalstvo na rubu mreže:** pokreniteChloros

naLinux

x86_64 i ARM64 platformama, uključujući NVIDIA Jetson, za terensku i rubnu obradu. Pogledajte [PregledLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Cijene i prijava</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
