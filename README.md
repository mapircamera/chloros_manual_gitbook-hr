---
metaLinks: {}
---

# Početak rada

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros je softverska aplikacija tvrtke [MAPIR](https://www.mapir.camera) za obradu multispektralnih snimki, daljinski nadzor hardvera tvrtke MAPIR i snimanje podataka senzora. Chloros 1.3.0 podržava cijelu obitelj proizvoda tvrtke MAPIR:

* **Survey3 kamere** — obrađuju snimke u RAW+JPG formatu u kalibrirane karte reflektancije i indeksa vegetacije. Pogledajte [Podržane kamere](supported-cameras.md).
* **Kamere LATTICE** — povezujte GigE module višespektralnih kamera uživo, pojedinačno ili kao sinkronizirane mreže više kamera: pregledajte, snimite i obradite u kalibrirane proizvode radijance i reflektance. Pogledajte [odjeljak LATTICE](lattice/README.md).
* **DAQ senzori svjetla** — DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E te DAQ-E-S (Ethernet) spektralni senzori: kalibrirane spektre uživo, zapisima u `.daq` i osvjetljenju usmjerenom prema dolje za obradu refleksance. Pogledajte [odjeljak DAQ](daq/README.md).

{% hint style="success" %}
**Novosti u verziji Chloros 1.3.0**: AI asistenti mogu upravljati programom Chloros putem sučelja `chloros-mcp`, a CLI, SDK i `chloros-mcp` nude iste značajke. Unaprijeđeni su LATTICE index proizvodi, cjelovite matrice senzora su brže i rade na bilo kojoj mreži, a pozadinski sustav nastavlja raditi između naredbi. Pogledajte popis novosti u nastavku i [preuzmite](download.md) instalacijske programe.
{% endhint %}

{% hint style="info" %}
**Koristite li Chloros s AI asistentom?** Ovaj priručnik je napravljen za to. Uputite svog asistenta na:

* `https://mapir.gitbook.io/chloros/llms.txt` — mašinski čitljiv indeks svake stranice.
* Bilo koja stranica kao sirovi Markdown — dodajte `.md` na njezin URL (npr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI referencu](reference/cli-reference.md) i [SDK referencu](reference/sdk-reference.md) — potpune referentne stranice s točnim vrijednostima napisane za upotrebu s LLM-om.

Primjer upita: *"Pročitaj https://mapir.gitbook.io/chloros/reference/cli-reference.md,, a zatim napiši skriptu koja se prijavljuje i obrađuje mapu ~/flights/flight_001 u reflectance i NDVI GeoTIFF formate."*

Ili pokrenite Chloros izravno: `chloros-mcp`, MCP poslužitelj instaliran uz SDK, radi s Claude Desktopom, Claude Codeom, Cursorom i drugim MCP klijentima.

Cjeloviti vodič: [Korištenje Chloros a s AI asistentima](ai-assistants.md).
{% endhint %}

***

## Novosti u Chloros u 1.3.0

* **AI asistenti mogu upravljati Chlorosom.** `chloros-mcp`, MCP poslužitelj instaliran uz Pythonov SDK, omogućuje aplikacijama Claude Desktop, Claude Code, Cursor i drugim MCP klijentima obradu slika i upravljanje LATTICE kamerama, nizovima i DAQ svjetlosnim senzorima. Pogledajte [Korištenje Chloros a s AI asistentima](ai-assistants.md).
* **CLI, SDK i `chloros-mcp` nude iste značajke.** Samo prijava, postavljanje DAQ-E i izravni DAQ poslužitelji ostaju s CLI radi sigurnosti. Neke skripte verzije 1.2.0 zahtijevaju izmjene: pogledajte odjeljak **Promjene u verziji 1.3.0** na vrhu [CLI referentne dokumentacije](reference/cli-reference.md) i [SDK referentne dokumentacije](reference/sdk-reference.md).
* **Poboljšani LATTICE indeksni proizvodi.** Indeksni proizvodNDVI
i ostali indeksni proizvodi izvoženi iz LATTICE snimki poboljšani su u svakom načinu obrade.
* **Opcije instalatera za Windows.**U odjeljku**Postavke sustava**instalater nudi**Omogući dugačke putanje u sustavu Windows**i**Isključi pozadinski sustav Chloros iz skeniranja Microsoft Defendera**, a obje su opcije označene prema zadanim postavkama. Prva opcija omogućuje da mape projekta budu smještene dublje od 260 znakova; druga sprječava usporavanje snimanja kamere zbog skeniranja u stvarnom vremenu (oko 30 % brzine snimanja pri velikim brzinama). Instalacijski program sada također dodaje mapu CLI u `PATH` samo jednom i uklanja duplikate unosa koje su ostavili prethodni instalacijski programi. Pogledajte [Preuzimanje](download.md).
* **Duboke mape projekta se odbijaju unaprijed.** Kada su isključene dugotrajne staze sustava Windows (što je zadano u sustavu Windows), mapa projekta koja je preduboka za ograničenje staze od 260 znakova odbijena je prije nego što se išta pokrene, uz navođenje najduže mape projekta koja stane i upute o tome kako omogućiti dugotrajne staze. Prije, takvo pokretanje nije stvaralo nikakve proizvode. Kada su omogućene duge staze, proizvodi u vrlo dubokim mapama sada zadržavaju svoje metapodatke (GPS i XMP oznake).
* **Ograničavanje maski.** Kada razdvajanje pojaseva (band unmix) proizvoda LATTICE radiance ili reflectance da negativnu vrijednost pojasa, ta se vrijednost ograničava na nulu, a maska po pikselu u mapi `clamp_masks/` pokraj mapa proizvoda bilježi koji su pikseli i pojasevi bili pogođeni. Proizvod imenuje svoju masku u oznaci `Chloros:UnmixClampMask`. Prikazi uživo sada obrezuju točno kao i izvoz. Pogledajte [Formati izlaznih slika](output-image-formats.md).
* **Proizvodi opisuju sami sebe.** Indeksni proizvodi nose vlastite oznake obrade (`Chloros:ProcessingLevel=index`, izraz indeksa i jesu li izračunati iz radijance ili reflektance) umjesto oznaka izvornog sirovog datoteka. ProizvodiSurvey3
nose oznake obrade Chloros u svakom načinu obrade. Surovi i de- Bayer-izirani LATTICE datoteke ostaju u nativnoj orijentaciji senzora i bilježe svoju orijentaciju u XMP-u; Preglednik slika okreće ih samo za prikaz.
* **Paketi za Linux isporučuju se neotpakirani.** Backend i CLI instaliraju se kao mape unutar `/usr/lib/chloros/`, pa se više ne otpakiraju pri svakom pokretanju. Na x86_64 arhitekturi pozadinska komponenta više ne troši oko 100 sekundi na pokretanje, a `.deb` je otprilike upola manja od svoje prijašnje veličine. Uključeni ExifTool sada radi na čistoj instalaciji Ubuntua. `.deb` sada također instalira Python SDK za sustav `python3` (instalirajte ga pomoću `sudo apt install ./chloros_1.3.0_<arch>.deb`), povećava kernelove soketne prijemne buffer-e koje streamovi s kamere trebaju i dodaje pozadinsku uslugu koja se pokreće kao vaš korisnik. Jetson paket, `chloros_1.3.0_arm64_jp6.deb`, izgrađen je za JetPack 6 i obrađuje se na GPU-u Jetsona pomoću CUDA izdanja PyTorcha za JetPack. Na Linux u svaka izvezena slika sada jednom dospijeva na disk (upisana i označena u memoriji, a zatim kopirana na mjesto): na Jetsonu, gdje disk određuje tempo, izvođenje LATTICE-a od 288 slika traje 14 minuta umjesto 24. Pogledajte [Instalacija na Linux u](linux/linux-installation.md).
* **LATTICE OCN kamere preciznije čitaju u sunčevoj svjetlosti.** Radijansa, reflektancija i indeksi dobivaju korekciju za plavetnilo nakon razdvajanja po trakama, tako da sivi cilj u suncu čita blizu neutralnog NDVI. RGN kamere su nepromijenjene.
* **Prijave grešaka unutar aplikacije.** Ikona greške na vrhu kartice za zapisnik o greškama (Debug Log) šalje tvrtki MAPIR opis, do 10 slika, zapise i detalje o sustavu. Pogledajte [GUI : Navigacija](navigation.md).
* **LATTICE nizovi.**Nizovi prema zadanim postavkama zadržavaju cijeli senzor, a komprimirani 12-bitni prijenos (`--packed`, prema zadanim postavkama `auto`) povećava brzinu sličica cijelog senzora za otprilike trećinu; niz radi na bilo kojoj mreži brzinom sličica koju ona dopušta (sporija veza smanjuje brzinu sličica umjesto da odbije niz ili smanjuje njegove sličice); niz čije se kamere ponovno pokreću oporavlja se sam od sebe, s ponovno omogućenim PTP-om; automatska ekspozicija više ne smanjuje tu brzinu pri slabom osvjetljenju (ispunjava period, a zatim pojačava signal); Kontinuirano snimanje prati brze nizove; kontrola**Highlights** za RGB kamere; i `ArraySession` iz SDK dobiva `grab()`, `frame_stream()` i `continuous()`. Vidi [odjeljak LATTICE](lattice/README.md).
* **Stabilnija automatska ekspozicija.** Ekspozicija se drži na konstantnoj svjetlosti i stabilizira se unutar nekoliko sekundi nakon promjene, mali odsjaji više ne zamračuju scenu, ručna ekspozicija ostavlja pojačanje na postavci Automatski, a kamera koja prestane reagirati na jakom suncu drži se na razini za svoju svjetlost, koju očitava senzor svjetlosti ili, ako nije priključen, vlastitim kadrovima kamere. Vidi [Postavke kamere](lattice/camera-settings.md).
* **Senzori svjetla.** **Podrška za DAQ-E-S:** njegov fiksni difuzor ne zahtijeva korekciju poklopca, a svako očitavanje sadrži podatke o nagibu, rotaciji i usmjerenju jedinice. Zapisnik navodi je li kapa odabrana ili pretpostavljena, a obrada upozorava na pretpostavljenu kapu: DAQ-U, DAQ-M i DAQ-E isporučuju se bez kape, stoga naznačite onu koja je ugrađena. Nastavak snimanja uživo `.daq` preživljava prekid napajanja (čuva `.daq-journal` datoteku sa svojim `.daq`), a DAQ-E koji prestane slati podatke ponovno se povezuje s prazninom označenom u njegovoj snimci. Projekt koji sadrži samo snimke sa senzora svjetla prikazuje ih i obrađuje iz aplikacije i iz CLI-a, a snimka napravljena pomoću `chloros_scripts`-a navedena je s DAQ-om koji ju je napravio, s oznakom koju možete postaviti. Svako snimanje sprema DAQ očitanje koje je upotrijebilo. Pogledajte [odjeljak DAQ](daq/README.md)..
* **Obrada.**LATTICE okviri započinju obradu tijekom detekcije cilja. Reference za cilj i DAQ određuju se za svaku pojas. Premješteni projekt ponovno povezuje svoje datoteke. Izvozi 8-bitnog snimanja su 8-bitni, u novom**TIFF (8-bit)** formatu. Pokretanje alata Survey3 bez cilja za kalibraciju evidentira se kao njegov proizvod zadane vrijednosti (`Vignette_Corrected_Images` ili `Sensor_Response_Images`), a ne kao refleksancija. DLS oznake za svaku pojas na slici refleksancije uvijek pripadaju vlastitom kadru. Texture Aware je odbijen za LATTICE snimke, koje uvijek dobivaju standardni debayer.
* **CLI i SDK.** Relativne putanje izlaza sada se nalaze u mapi iz koje pokrećete naredbu ili skriptu (instalirani backend ih je ranije rješavao u vlastitoj instalacijskoj mapi i pri tome je dolazilo do pogreške). Backend nastavlja raditi između naredbi i skripti i sam se zaustavlja nakon 30 minuta neaktivnosti; `chloros-cli backend status` i `backend stop` njime upravljaju. Naredbe za snimanje imenuju svaku datoteku koju nisu mogle zapisati i izlaze s kodom 1, a ponavljajuća snimanja zaustavljaju se kada se disk napuni. Uspješno pokretanje `process` izvještava `Image products written: N (plus M clamp masks)`. Na Windows ovom SDK ovom `ChlorosLocal` / `process_folder` pronalazi instalirani backend (i koristi onaj koji je već u pokretanju), a `read_image_audit_tags` radi iz običnog `pip install`.

***

## Što je novo u verziji Chloros 1.2.0

* **Upravljanje kamerama uživo — novi kartica Kamere.** Spojite kamere LATTICE jednu po jednu ili kao sinkronizirane mreže više kamera (PTP vremenska sinkronizacija, snimanje okidačem na hardveru), s prikazom predviđenog kadra, histogramima po pojasu, pametnom automatskom ekspozicijom, kalkulatorom indeksa uživo i ažuriranjima firmvera kamere unutar aplikacije.
* **Senzori svjetla — novi kartica Senzori svjetla.** Povežite senzore DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet); prikaz kalibriranih spektra uživo (W/m²/nm), snimanje `.daq` datoteka u vaš projekt, odabir profila za korekciju snimanja i ažuriranje firmvera DAQ-E putem mreže.
* **Načini snimanja i snimači.** Pojedinačno / kontinuirano / intervalno snimanje, plus način najbržeg snimanja samo u sirovom formatu; projektni odabir kamera i vrsta izvoza koje "Snimanje svega" (Capture All) proizvodi; snimači nizova za indeksni video razine nadzora i sirove burstove razine analize s izvanmrežnim izradama videozapisa.
* **LATTICE pipeline za obradu.** Uvozite mape snimanja LATTICE i raspakirajte svaki sirovi kadar u debayered, pretpregled, float32 radijancu (W/m²/sr/nm) i proizvode reflektancije s prekidačima po proizvodu. Odraznost može potjecati od kalibracijske pločice unutar kadra ili od DAQ-ovog zračenja prema dolje; poravnanje niza primjenjuje se na izvoze; nedostajuća tvornička kalibracija automatski se preuzima prema serijskom broju kamere.
* **Projekti pamte hardver.** Povezane kamere i senzori svjetla spremaju se s projektom (`cameras.json` / `sensors.json`) i ponovno se povezuju sa svojim spremljenim postavkama kada ponovno otvorite projekt. Vidi [GUI : Projects](projects.md).
* **Nadogradnje preglednika slika.** Čitanje piksela/indeksa pokazivača s ispravnim skaliranjem reflektancije po datoteci, histogrami slojeva, klizač za biniranje GSD-a, mrežni načini rada po okidaču / po kameri, prikazi proizvoda LATTICE i izvoz u sandboxu indeksa/LUT-a na disk.
* **CLI i SDK, znatno prošireni.** Novi `lattice`, `daq pool-*`, `project` i `time-sync` obitelji naredbi; nove opcije `process` (`--input-level`, prekidači po proizvodu, `--reflectance-source`, zastavice poravnanja niza); SDK pametne ručke za povezivanje (`connect_camera` / `connect_array` / `connect_daq_sensor`) koje automatski pokreću pozadinski servis; automatizacija `open_project()`; kotačić SDK a priložen je instalacijskim programima i objavljen na PyPI-ju kao `chloros-sdk`.
* **Poštena semantika pogrešaka.** Pokretanje `chloros-cli process`-a koje je zatražilo proizvode, ali nije ništa zapisalo, sada glasno završava s pogreškom i izlazi s ne-nuljevnim rezultatom; uspješna pokretanja izvještavaju o broju zapisanih izlaznih slika.
* **Novi raspored izlaza.** Proizvodi se spremaju u `<project>/<camera>/<format>/<Product>_Images/` mape i zadržavaju izvorni naziv datoteke — proizvod se identificira mapom, a ne nastavkom naziva datoteke. Pogledajte [Formate izlaznih slika](output-image-formats.md).
* **Više ulaza, planova i jezika.** Podrška za ulaze `.dng`; svih 38 jezika sučelja u potpunosti je prevedeno; hardverska ograničenja po planu s besplatnom (bez prijave) upotrebom do 4 kamere i 2 senzora svjetla.
* **Pouzdanost.** Zaustavljanje obrade se odvija uredno uz iskren sažetak izvršenja, projekti s više kamera izvoze svaku kameru, a nadogradnje instalatera vas više ne odjavljuju.

***

Chloros dostupan je u 3 sučelja aplikacije:

## Chloros: Desktop aplikacija s grafičkim sučeljem

Samostalna zasebna aplikacija sa svim značajkama, uključujući kartice za kamere uživo i senzore svjetla. _Samo za Windows._

## [Chloros CLI: Sučelje naredbenog retka](CLI.md)

Serijska obrada putem naredbenog retka uz naredbe za uživo `lattice`, `daq pool-*`, `project` i `time-sync`. Savršeno za automatizaciju, skriptanje i rad bez zaslona. Dostupno na **Windows u, Linux u amd64 i Linux u arm64 (NVIDIA Jetson)**. _CLI
zahtijeva plaćeni Chloros+ paket za pristup._

## [Chloros API: Python SDK](api-python-sdk.md)

Programski Python sučelje za automatizaciju i prilagođene tijekove rada: obrada cijelog pipelinea, sesije s kamerom/nizom u stvarnom vremenu, DAQ sesije sa senzorima i automatizacija spremljenih projekata. Instaliran je s desktop/CLI paketom, a također je objavljen kao `pip install chloros-sdk`. _API zahtijeva plaćeni Chloros+ paket za pristup._

***

## Podržane platforme

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Da | Da | Da |
| **Linux amd64 (x86_64)** | Ne | Da | Da |
| **Linux arm64 (NVIDIA Jetson)** | Ne | Da | Da |

Za upute za instalaciju na Linux pogledajte odjeljak [Linux i Edge Computing](linux/linux-overview.md).

***

## Počnite u tri koraka

1. **Instalacija** — preuzmite i pokrenite instalacijski program za svoju platformu. Pogledajte [Preuzimanje](download.md).
2. **Prijavite se (opcionalno za GUI)** — GUI obrađuje slike besplatno bez računa. Prijava na [Chloros+](chloros+-login.md) otključava paralelnu obradu, GPU akceleraciju, veće granice uređaja i pristup CLI/SDK.
3. **Stvorite svoj prvi projekt** — otvorite Chloros, stvorite [Novi projekt](projects.md), [dodajte svoje slike](processing-images-gui/adding-files-to-a-project.md), i [pokrenite obradu](processing-images-gui/starting-the-processing.md). Za upravljanje live hardverom umjesto toga otvorite karticu Kamere ili Senzori svjetla — pogledajte [GUI : Navigacija](navigation.md).

***

## Chloros+

Iako je Chloros besplatan za upotrebu za većinu zadataka, možda ćete poželjeti više. Tu vam može koristiti plaćena licenca za Chloros+. S licencom za Chloros+ možete otključati nove značajke kao što su:

* **Višenitna obrada**: znatno ubrzajte obradu slika za veće projekte istovremenom obradom slika kroz pipeline.
* **Ubrzanje pomoću GPU-a (CUDA)**: iskoristite današnje opcije veće GPU memorije kako biste dodatno ubrzali pipeline za obradu slika. Za najbolje rezultate preporučujemo 4 GB ili više VRAM-a.
* **Chloros+**[**CLI**](CLI.md) **Pristup**: pokrenite Chloros+ s naredbenog retka kako biste automatizirali i integrirali u vlastiti softver. Dostupno na svim plaćenim razinama; provodi se na strani poslužitelja.
* **Chloros+**[**API**](api-python-sdk.md) **Pristup:** pokrenite Chloros+ iz Python a za programatsku kontrolu, omogućujući besprijekornu integraciju s vašim istraživačkim procesima, tijekovima rada za analizu podataka i prilagođenim aplikacijama. Dostupno na bilo kojoj plaćenoj razini; provodi se na strani poslužitelja.
* **Viša ograničenja hardvera**: povežite više kamera i senzora svjetla odjednom. Bez prijave GUI povezuje do 4 kamere i 2 DAQ senzora svjetla; plaćeni paketi povećavaju oba ograničenja:

| Plan | Kamere | DAQ senzori svjetla |
| --- | --- | --- |
| Iron (besplatno, bez prijave) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Korištenje više uređaja**: svaka licenca za Chloros+ omogućuje registraciju 2+ uređaja. Koristite svoj MAPIR Cloud račun za upravljanje registriranim uređajima. Dodajte podršku za više uređaja nadogradnjom svoje licence za Chloros+.
* **Napredna metoda debayeringa osjetljiva na teksture:** visokokvalitetna metoda debayeringa osjetljiva na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji/strojnom učenju (AI/ML), koji uklanja gotovo svu šumovitost debayeringa.
* **Prilagođene formule za višespektralne indekse:** unesite prilagođene višespektralne indekse u rasterske kalkulatore programa Chloros, kako za obradu tako i za sandbox za pregled slika.
* **Linux i računalstvo na rubu mreže:** pokrenite Chloros na platformama Linux x86_64 i ARM64, uključujući NVIDIA Jetson, za terensku i rubnu obradu. Pogledajte [Pregled Linux a](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Cijene i prijava</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
