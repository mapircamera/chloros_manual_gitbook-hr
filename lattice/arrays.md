# Mreže više kamera

LATTICE **mreža**je dvije ili više LATTICE kamera povezanih kao jedna sinkronizirana jedinica. Jedna kamera je**glavna**: ona generira hardverski GPIO okidački impuls na zajedničkoj sinkronizacijskoj liniji (po zadanom**Linija2**), tako da se svaki član eksponira u istom trenutku. Chloros dodaje PTP vremensku sinkronizaciju, pregled uživo (pločice po kameri ili jedinstveni usklađeni višebandni kompozit) i sinkronizirano snimanje — svaki prolaz snimanja proizvodi jednu **grupu sličica** u kojoj sve kamere dijele isti vremenski žig i ID sličice (prikazano kao `fid:N` u izlazu snimanja).

Nizovi su način na koji mono (M3M) kamere proizvode indekse vegetacije — jedna kamera doprinosi jednom pojasu, a niz ih poravnava u višespojni niz. Pogledajte [Mono kamere i indeksi vegetacije](mono-indices.md).

Postoje tri ekvivalentna načina za povezivanje niza, a svi oni pokreću isti "smart-prep" tijek:

| Sučelje | Ulazna točka |
| --- | --- |
| GUI | kartica Kamere → **Poveži niz** (plavo gumb) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (prvi serijski broj = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (prvi serijski = master) |

Smart-prep provodi: provjeru mrežnih mogućnosti (ICMP DF ping + GVSP probe), odabir sloja sinkronizacije, projekciju brzine sličica pri zatraženoj veličini sličice, odluku o spakiranom 12-bitnom prijenosu (pogledajte [Pakirani 12-bitni prijenos](#packed-12-bit-transport)), omogućavanje PTP-a, automatski odabir formata piksela za svaku kameru, inicijalizacija automatske ekspozicije iz spremljenog stanja svake kamere i konfiguracija GPIO okidača na liniji 2.

**Zadana puna senzorska slika (novo u verziji 1.3.0).** Connect zadržava veličinu slike koju ste zatražili — puni senzor od 2048×1536, bez binninga i bez obrezivanja, osim ako ne odaberete drugačije — i dopušta da se brzina sličica spusti na razinu koju mogu osigurati host i mreža. Nikada samostalno ne obavlja binning niti obrezivanje (u verziji 1.2.0 binning se automatski pokretao pri brzini ispod 1,5 fps). Binning ili manji ROI odabire se prema vašoj želji: u dijalogu za povezivanje odaberite jednu opciju, a Projektirani ishod prikazuje brzinu sličica koju ona daje; `array-connect` ispisuje brže opcije s njihovim projektiranim brzinama.

{% hint style="info" %}
Kamere moraju biti dostupne na vezi prije nego što bilo što od ovoga počne raditi — pogledajte [Povezivanje kamera](connecting.md) za otkrivanje, adresiranje i preuzimanje kalibracije pri prvom povezivanju. Za postave s više kamera postavke receive-ringa mrežne kartice domaćina jednako su važne kao i brzina veze; potpuna tablica simptoma i rješenja nalazi se u [CLI reference § Postavljanje i podešavanje mrežne kartice domaćina](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dijalog za povezivanje niza

 картиca Kamere → **Poveži niz**otvara čarobnjaka u tri koraka:**Odabir → Prikazni način → Postavke**.

### Korak 1 — Odabir master i slave kamera

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Dijaloški okvir skenira mrežu čim se otvori ("Skeniranje mreže..."), zatim provjerava sinkronizacijske kabele ("Provjera sinkronizacijskih kabela...") kako bi pronašao koje su kamere na M8 sinkronizacijskoj liniji. Potrebno vam je najmanje **2 kamere** za izgradnju niza.

M8 sinkronizacijski kabeli spajaju se na jednom čvorištu, tako da **bilo koja ožičena kamera može biti master**. Sonda unaprijed popunjava uloge — kamera**RGB** (`-FRGB`) kao glavna kada je ožičena, inače prva ožičena kamera, a ostale ožičene kamere kao sluge — i prikazuje jedan od tri banera:

| Baner | Značenje |
| --- | --- |
| "Sinkronizacijski kabel pronađen na {n} kamera. Bilo koja od njih može biti glavna; predodabrana je kamera u boji RGB (u suprotnom prva)." (zelena) | Probator je pronašao kamere na sinkronizacijskoj liniji; potvrdni okviri za glavni radio i robne uređaje već su ispunjeni. |
| "Nije detektiran master - provjerite GPIO kabel" (narančasta) | Nijedna kamera nije primila okidački impuls; provjerite sinkronizacijski kabel. Uloge su i dalje unaprijed ispunjene (RGB kamera, inače prva kamera na popisu, kao master) i možete ih promijeniti. |
| "Nema sinkronizacijskog kabela: {serijski brojevi}" (narančasta) | Navedene kamere nemaju priključen sinkronizacijski kabel. |

Tablica kamera ima stupce **Kamera / Serijski broj / IP / Glavni (radio) / Rob (kvačica)**. Redovi su navedeni po**redoslijedu kamera** — kamere M3C (Bayer) prije M3M (mono); među M3C, FRGB, FRGN, FOCN, zatim FNGB; M3M prema valnoj duljini filtra, najniža prva (F450 prije F808). Popis kamera u bočnoj traci i zadani prikaz mreže koriste isti redoslijed.

* Odaberite točno **jednog mastera**i**jednog ili više slavea**. Ponovnim klikom na radio-gumb trenutnog mastera on se poništava. Odabir drugog mastera vrši se jednim klikom: novi master preuzima ulogu mastera, a prethodni master postaje slave (osim ako nema sinkronizacijski kabel).
* Kamera označena kao **"Nema sinkronizacijskog kabela"** nikada se ne može odabrati kao rob — rob bez kabela za okidač vječno bi čekao na sinkronizacijskoj liniji i isporučivao bi mrtvi signal. Umjesto toga, povežite tu kameru kao samostalnu kameru.
* Kamere koje su već povezane kao samostalne *nisu* onemogućene: povezivanje u niz prekida samostalnu sesiju i ponovno otvara kameru unutar niza.

**Sljedeće: Prikazni način →**omogućuje se nakon što se odabere jedan glavni i najmanje jedna robna kamera.**Ponovno skeniranje** ponovno pokreće otkrivanje i provjeru ožičenja.

{% hint style="warning" %}
**Otkaži** je onemogućeno dok je skeniranje ili provjera u tijeku — otkazivanje usred provjere može dovesti do pada kamere SDK a na firmwareu kamere LATTICE. Pričekajte da se kotačić zavrti.
{% endhint %}

### Korak 2 — Način prikaza



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Način | Što dobivate |
| --- | --- |
| **Odvojene kamere** | Jedan živi prikaz po kameri, sve se aktiviraju istovremeno kako bi se okviri održali sinkroniziranima. Svaka kamera zadržava vlastitu boju i postavke. |
| **Kombinirane kamere** *(zadano)* | Jedan prikaz koji prikazuje poravnati višebandski kompozit NDVI indeksa. Kamere dijele boju niza. |

Način prikaza mijenja samo prezentaciju uživo — ponašanje snimanja je isto u oba slučaja.

### Korak 3 — Postavke niza i predviđeni ishod

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Pri ulasku u ovu scenu, Chloros traži **preporuku**od pozadinskog sustava i primjenjuje je:**cijeli senzor — bez binninga, bez ROI obrezivanja — kad god bilo koja brzina sličica to može podržati**, pri bilo kojoj brzini koju host i mreža mogu isporučiti. Tek kada nijedna brzina ne može isporučiti cijeli senzor, preporuka smanjuje sličicu, a zatim preferira binning u odnosu na ROI obrezivanje, jer binning čuva cijelo vidno polje. Od tog trenutka mijenjate rezoluciju za brzinu sličica. Svaka promjena koju napravite ponovno pokreće analizu uživo i ažurira desni panel**Predviđeni ishod**.

Lijeva stupac — postavke:

| Kontrola | Izbori | Zadano | Bilješke |
| --- | --- | --- | --- |
| **ROI (polje vidnog polja)** | Cijelo (2048×1536) / Polovično (1024×768) / Četvrtasto (512×384) | Cijelo | Izrezivanje senzora: izrezivanje na manju regiju u nativnoj gustoći piksela. |
| **Binning** | 1× / 2× (zbroj 2×2) / 4× (zbroj 4×4) | 1× | Hardverski binning: 2×2 = cijelo vidno polje (FoV) uz četvrtinu troška prijenosa; 4×4 = cijelo vidno polje (FoV) uz 1/16 troška. Skriveno ako kamere ne podržavaju binning. |
| **Slika na strani kabela** (izlazno očitavanje) | — | — | Širina×visina nakon binninga koja se zapravo šalje preko kabela, zaokruženo na višekratnike od 16 (minimum 64). |
| **Stopa okidanja** | 0,5–60 fps, korak 0,1 | prazno = automatski | Stopa okidanja mastera. Ostavite prazno da Chloros to izračuna. |
| **Protok**| 20–2000 MB/s, korak 10 | prazno = automatski | Koliki protok host zaista može podnijeti, u MB/s —**jedan broj od kojeg ovisi cijelo dodjeljivanje niza.** Automatski otkriveno s mrežnog adaptera. Smanjite ga ako niz prijavljuje oštećene okvire: otkrivena vrijednost precjenjuje USB adaptere i zajedničke switcheve. Promjena te vrijednosti ponovno pokreće projekciju uživo. Polje se otvara s proračunom s kojim je zadnja niza bila povezana (pohranjenim s projektom), tako da se proračun koji ste jednom ispravili zadržava. |

Kutija za potvrdu **Pin rezolucije** iz verzije 1.2.0 je uklonjena: 'Connect' uvijek zadržava veličinu okvira koju odaberete i umjesto toga smanjuje brzinu okvira.

Desna kolona — **Predviđeni ishod**:

* **Sync tier** — "Istovremeno snimanje" (zelena), "Istovremeno snimanje (FTD-staggered emit)" (zelena), "Nastavno snimanje (100 ms drift)" (narančasta), ili "Konfiguracija je prevelika" (crvena). Chloros odabire razinu: simultano snimanje kad god se veza može dijeliti brzinom kojom se kamere mogu uskladiti (brzina sličica opada kako se udio svake kamere smanjuje) i kaskadno snimanje samo kada to nije moguće — tako se niz povezuje na bilo kojoj mreži, pri brzini sličica koju ta mreža dopušta.
* **projekcija fps-a** — brzina sličica u sekundi kojom se očekuje da će niz raditi. Kod istovremene snimke to je jedna brojka: prvo dolazi brzina, a automatska ekspozicija popunjava razdoblje koje ona daje, a zatim pojačava signal, tako da je slaba scena ne smanjuje (vidi [Kako automatska ekspozicija troši ekspoziciju i pojačanje](camera-settings.md)). Kod pomaknutog snimanja prikazuje se kao raspon ("slabo → jako"), jer izloženost svake kamere produljuje pomak u slabo osvijetljenoj sceni.
* **Brzina NIC-a** — brzina veze i održiva propusnost ("NIC {mbps} Mbps · održiva {N} MB/s").
* **Provjera sim-emit paketa** — može li NIC hosta primiti jedan istovremeni paket sa svih kamera ("Sim-emit paket: X MB · raspoloživi prsten NIC-a: Y MB ✓/✗").
* **Provjera budžeta kabela** — agregatna potražnja u stabilnom stanju u usporedbi s gornjom granicom kabela otpornom na kolizije ("Budžet kabela: {potražnja} MB/s potražnja {n} kamera · gornja granica {gornja granica} MB/s ✓/✗ prekoračeno").
* **"Maks. kamera na ovoj vezi: {n} — određeno minimalnom propusnošću po kameri, stoga binning ne povećava taj broj."** — prikazuje se kada ste blizu (ili iznad) gornje granice broja kamera.
* **"SLIKE ĆE SE ISPADATI pri ovim postavkama."**"**— crveno upozorenje s razlogom pozadinskog sustava, plus popis blokatora i plave**prijedloge za rješenje** ("Da bi se ovaj niz uklopio u mrežu" / "Da bi se omogućilo istovremeno snimanje").**Primijeni i poveži** je onemogućeno dok ne postoji projekcija, a njegova oznaka vam govori zašto je odbijeno:

| Naziv gumba | Značenje | Što zapravo pomaže |
| --- | --- | --- |
| "Analizira se..." | Analiza je još u tijeku. | Pričekajte. |
| **"Previše kamera za ovu mrežu"**| Mreža ne može prenositi niz čak ni s jednom kamerom odjednom (provjera agregacije nije uspjela): puno je više kamera nego što ih može podržati, ili druge kamere koje se na njoj streamaju. | Manje kamera, jumbo paketi od početka do kraja, ili brži mrežni adapter (NIC).**Manji ROI NEĆE pomoći** — pogledajte dolje. |
| **"Smanjite ROI za omogućavanje"** | Slike bi se gubile pri ovim postavkama (provjera burst/ring je neuspjela). | Smanjite ROI, povećajte binning ili popravite NIC receive ring. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Tijekom povezivanja može se pojaviti zeleni **panel za preuzimanje kalibracije** s indikatorom napretka po serijskom broju: prvi put kada se kamera poveže na računalo, Chloros preuzima njezin tvornički paket kalibracije od ~3,8 MB s kamere putem GigE-a (otprilike 70 sekundi po kameri). Pohranjenim kamerama ovaj se panel nikada ne prikazuje; kod više od šest kamera redovi se pomiču. Pogledajte [Povezivanje kamera](connecting.md).

### Packed 12-bitni prijenos

12-bitni formati (`BayerRG12`, `Mono12`) obično se prenose kao 16-bitne riječi. **Pakirani**prijenos šalje istih 12 bita u 1,5 bajta po pikselu i raspakira ih na hostu, pa svaki kadar zahtijeva 25 % manje vremena za slanje — pri punom senzoru, otprilike trećinu više kadrova u sekundi.**Pohranjeni podaci su identični.** Array connect to odlučuje automatski (`auto`): spaja samo kad to pomaže i kad host ima slobodnog CPU-a za raspakiranje. GUI nema prekidač za to; iz CLI upotrijebite `array-connect --packed {auto,on,off}` (po zadanom `auto`). 8-bitni i 16-bitni formati nikada se ne pakiraju.

## Propusnost: koliko se kamera može uklopiti

Propusnost niza svojstvo je mrežne veze, a ne programa Chloros, stoga se planerski podaci nalaze u priručniku za hardver: **[Planiranje propusnosti niza](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Chloros s njima radi sljedeće: dijalog za povezivanje pokreće mrežnu provjeru, predviđa postignutu brzinu slanja okvira i odabire odgovarajući sloj. Niz se povezuje na bilo kojoj mreži: na sporijoj vezi brzina svake kamere prilagođava se njezinom udjelu u propusnosti, pa brzina slanja okvira opada, umjesto da se zahtjev za povezivanje niza odbije ili se smanjuju njegovi okviri. Samo se niz za koji mreža ne može prenijeti ni jednu kameru odjednom odbacuje (**Previše kamera za ovu mrežu**) umjesto da tiho ispušta pakete — pogledajte gornje opisani panel s predviđenim ishodom.

## Kada nedostaju sličice

Kamera može nedostajati iz objavljene grupe iz dva potpuno različita razloga,
i zahtijevaju suprotna rješenja. Chloros ih broji odvojeno umjesto da izvještava o jednom
"nepotpunom" broju koji ne navodi nijedan od njih:

| Što se dogodilo | Što to znači | Gdje potražiti |
| --- | --- | --- |
| **Oštećen**— kadar je stigao i bio je strukturalno neispravan | GVSP gubitak paketa na mrežnoj putanji |**Mrežni budžet**, NIC prsten za primanje, jumbo paketi, switch |
| **Nikada nije stigao**— nije stigao nijedan kadar | Kamera se nije aktivirala ili ništa nije izašlo |**M8 sinkronizacijski kabel**, sinkronizacijska linija, je li svaki član aktiviran |

Podjela se ponovno procjenjuje svakih 10 sekundi dok niz prenosi. Ako je iznad 5 %, bilježi se uz navođenje oba broja, a svaki oštećeni spremnik izvještava se prvi put kada se dogodi po kameri, a zatim se podaci zbrajaju jednom u minuti kako bi duga sesija ostala čitljiva.

**Okvare s nultim brojem neprimljenih paketa znače da su aktivacija i sinkronizacija kabela savršeni**, a svaki izgubljeni paket je na mrežnom putu. Rješenje je smanjiti**Wire Budget** i ponovno se povezati.

{% hint style="warning" %}
**Smanjenje brzine okidača ne pomaže kod oštećenih sličica.** Uređivanje tempa paketa kamere vrši se jednom, pri uspostavljanju veze. Smanjenje brzine okidača mijenja učestalost pojavljivanja paketa, a ne brzinu kojom se paketi sami šalju. Na testiranoj konfiguraciji s 4 kamere smanjenje stope okidača za 5× nije ništa promijenilo, dok je smanjenje *Wire Budget*-a s 240 na
200 MB/s spustilo je stopu pogrešnih okvira na istoj konfiguraciji s 10,4 % na nulu.
{% endhint %}

Aktivna matrica ne može se ponovno planirati — odspojite i ponovno spojite kako bi odabir pri spajanju mogao raditi s novim budžetom.

### USB mrežni adapteri imaju gornju granicu

USB Ethernet adapter reklamira svoju *Ethernet* brzinu veze, ali ono što on zapravo može održati ograničeno je USB sabirnicom i njezinim upravljačkim programom. USB 10GbE donglu je nekada pripisivan protok od otprilike 1000 MB/s — brojku koju nikada nitko nije izmjerio — i tempiranje četiri kamere prema tom fiktivnom višku snage korumpiralo je 6–18 % sličica dok je niz i dalje izvještavao o zdravoj ciljanoj brzini sličica. Adapteri priključeni putem USB-a sada su ograničeni na
**200 MB/s** na sustavima Windows i NVIDIA Jetson te **400 MB/s** na sustavu Linux x86_64. Ograničenje je apsolutno, a ne postotno, jer je limitirajući faktor
USB-bus: USB 1 GbE adapter postiže oko 80 MB/s i na njega to nema utjecaja.

Ako je vaš host doista brži od ograničenja, povećajte **Wire Budget** kako biste to naznačili.

## PTP sinkronizacija vremena

*Sinkronizacija* kadrova dolazi od hardverskog okidača; **PTP** (IEEE 1588 PTPv2) pruža usporedive *vremenske oznake* na svim uređajima. Podrazumijevano je omogućeno pri povezivanju niza:

* Host back-end **Chloros** pokreće PTP grandmaster. Kamere LATTICE i DAQ-E senzori svjetla rade kao sluge (slave) na njemu u domeni 0, tako da vremenski žigovi slika i DAQ spektri dolaze na istom satu (~1 ms).
* `--no-ptp` (CLI) onemogućuje ga za rad u laboratoriju — vremenski pečati između kamera tada **nisu** usporedivi.
* Provjerite stanje sinkronizacije pomoću CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Sam kartica Kamera nema indikator PTP-a; na površinama za sinkronizaciju pojedinačnih kamera prikazuju se samo **Uloga**(Master/Slave),**Linija sinkronizacije** i sloj mogućnosti niza. Stanje PTP-a za DAQ-E prikazano je u detaljima senzora na kartici Senzori svjetla.

## Prikaz

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

niza uživo Glavni područak za prijenos nudi dva izgleda (prekidač u gornjoj traci): **mrežni prikaz**(svaka pločica je ćelija; povlačenje za promjenu redoslijeda kada je lokot na mreži otključan) i**pregled u obliku popisa**(redovi pune širine:**kombinirani**niz je jedan red — njegova sastavljena pločica — dok**odvojeni**niz daje**svakom članu vlastiti red**, s vlastitim prijenosom uživo i vlastitom tablicom cilja kalibracije; ispod nizova, jedna samostalna kamera — aktivna). Klizač**Feed Zoom** određuje veličinu pločica; pri širini ćelije manjoj od 200 px nazivi/fps automatski se skrivaju. Članice kamere pojavljuju se u bočnoj traci i u zadanoj rasporedu mreže po redoslijedu kamera (M3C prije M3M, FRGB prvo, monosi prema valnoj duljini); otključavanje mreže i povlačenje i dalje nadjačava redoslijed mreže.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Odvojeni način** prikazuje jednu pločicu po kameri. Svaka pločica preklapa:

* naziv kamere (gornji lijevi kut),
* **prikaz fps-a** (gornji desni kut) — ovo je *stvarna brzina snimanja* kamere koju izvještava pozadinski sustav, a ne brzina ispitivanja za pretpregled (pretpregled uživo ograničen je na 30 fps bez obzira na brzinu snimanja),
* točku statusa — zelena (prijenos) / narančasta (učitavanje) / crvena (greška),
* **vrtložak zastarjelog kadra** kada 2 s nije stigao novi kadar — normalno za ~5 s nakon bilo kojeg povezivanja/ispitivanja dok pozadinski sustav (backend) ponovno ne uravnoteži mrežni budžet među kamerama.**Kombinirani način rada**prikazuje jednu kompozitnu pločicu: pozadinski sustav uklanja šum po blokovima (debayer), skalira, poravnava, uklanja šum, pretvara u radijancu po pojasu (plus DLS reflektanciju kada je priključen senzor svjetlosti), ocjenjuje izraz indeksa niza, primjenjuje LUT i prenosi rezultat kao MJPEG. Dok se ne renderira prva poravnata slika, pločica objašnjava svoje stanje: "Priprema niza…", "Kalibracija poravnanja…", "Čekanje prvog kadra…", ili — ako automatski pokušaji poravnanja pločice (otprilike jednu minutu) isteknu — "Potrebno je poravnanje" s gumbom**Kalibriraj poravnanje**.

Očitavanje fps-a pločice niza također izvještava o stanju prijenosa:**"Primjena postavki…"**dok promjena postavki (format, biniranje, ROI) ponovno pokreće kamere,**"Zaustavljeno"**kada je niz aktivan, ali je prestao isporučivati grupe sličica, i**""Zaustavljeno"**kada je prestalo s prijenosom — u tom slučaju ponovno povežite niz. Kada je spektralna separacija morala ograničiti pojas prikazan u kompozitu na velikom dijelu njegovih piksela (25 % ili više), pločica prikazuje**"Podaci o pojasu niske pouzdanosti · N%"**: vrijednosti indeksa izgrađene na toj traci su upitne (obično je jedna traka primila premalo svjetla).

Korisne činjenice o kombiniranom načinu rada:

* Kompozit je registriran na kadar **glavne**kamere. AE-ROI ciljanje i mjerenje svjetline na kompozitu točni su za glavnu kameru, a približni za pomoćne; upotrijebite**Podijeljeni prikaz** (postavke niza → "Prikaži članice kamere") za točne po-pikselne pločice za svaku kameru bez otvaranja dodatnih veza s kamerama.
* **Prikaz slojeva* (postavke niza; zadano isključeno) omogućuje vam odabir prednjeg i pozadinskog sloja — bilo koje priključene kamere ili **indeksa**. Kada je prednji sloj = indeks, piksele izvan LUT minimalne/maksimalne vrijednosti prikazuju pozadinski sloj.
* **Rendirana rezolucija** (zadano 720p) postavlja visinu prijenosa uživo *i* veličinu spremljenog izvoza kompozita. Slike za svaku kameru uvijek se izvoze u punoj rezoluciji.
* Poravnanje se izračunava za svaku sesiju i nikada se ne pohranjuje. Chloros poravnava **svaki niz automatski nakon povezivanja, u oba načina prikaza**, nakon što se automatska ekspozicija članova stabilizira (nastavlja pokušavati do 5 minuta). Odjeljak**Poravnanje**(Alignment) u prozoru postavki niza prikazuje RMS ostatke i gumb**Ponovno kalibriraj poravnanje** (Recalibrate alignment) — također i za odvojene nizove koji nemaju kompozitnu pločicu.

## Snimanje: nadzor naspram analize

Sučelja za snimanje nizova jasno su podijeljena na **razinu nadzora**(zapisujte ono što vidite) i**razinu analize** (zapisujte sirove podatke, kalibrirajte kasnije):

| Radni tijek | Razina | Što sprema | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Capture**(fotografije) | Analiza | Jedna sinkronizirana grupa sličica po prolazu; datoteke za svaku kameru na svakoj odabranoj razini izvoza (sirovi/dekonvoluirani/radijantni/reflektantni/pregled/indeks) + `.daq` sidecar |**gumb **Capture All** + Postavke snimanja | `lattice array-capture` |
| **Snimanje indeksnog videa** | Praćenje | Prikazani kombinirani indeksni kompozit uživo — 8-bitni, rezolucija pretpregleda, ugrađena LUT. Iz CLI-a bez otvorenog GUI prikaza, pozadinski sustav renderira NDVI niza (ili `--expr`) nakon što se niz poravna; snimanje koje ne zaprimi nijedan kadar ne uspijeva | ● Snimanje indeksnog videa (kombinirani nizovi) | `lattice array-record` |
| **Sirovi burst → izrada videozapisa**| Analiza | Sirove senzorske slike pri punoj brzini snimanja + manifest + `.daq`, zatim izvanlinijska rekonstrukcija u kalibrirani videozapis radijance / reflektance / indeksa, usklađen vremenski s očitanjima DAQ-a | ⦿ Snimanje sirovog burst-a →**Izrada videozapisa** | `lattice array-burst` → `lattice array-build-video` |

Pravilo je sljedeće: ako će se pikseli koristiti za *mjerenja*, upotrijebite snimanje ili burst (za potrebe analize); ako samo trebate *pratiti ili demonstrirati* ono što je niz vidio, snimite indeksni video (za potrebe nadzora).

### Postavke snimanja (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Zupčanik pored **Snimaj sve** otvara prozor Postavke snimanja (zahtijeva otvoren projekt — snimke se spremaju u njega):

* **Način snimanja**:**Jednokratno**(jedan prolaz) /**Neprekidno**(uzastopno; ograničeno brojem snimaka, zadano 1, ili trajanjem, zadano 10 s) /**S intervalom** (time-lapse: N snimanja svakih X intervala za ukupno Y, zadano 1 svakih 5 s tijekom 1 minute).
* **Tipovi izvoza po kameri**: Raw, Debayered, Radiance, Reflectance, Pregled, Indeks — sve primjenjivo je zadano uključeno. Radiance/Reflectance su skriveni za kamere s RGB-filterom; **Odbijanje**se pojavljuje samo kada kamera ima DAQ svjetlosni senzor (svoj ili naslijeđen od niza);**Indeks** zahtijeva konfigurirani izraz indeksa.
* **Poravnano**(po nizu, zadano**uključeno**): prilagođava izvoze članova profilu poravnanja niza kako bi izvozi bili poravnati na razini piksela. Raw format uvijek ostaje neizobličen, ali pohranjuje transformaciju u metapodacima. Datoteke članova niza spremaju se u nativnoj orijentaciji svake kamere (bilo koje postavke ogledala/rotacije bilježe se u datoteci, a ne primjenjuju se).
* **Najbrže snimanje** (prekidač): samo sirovi podatak + dodana DAQ očitanja + besplatni kompozit kombiniranog indeksa, uz preskakanje matematičke kalibracije pri snimanju radi maksimalne brzine — ponovno izgradite radijancu/reflektancu/indeks kasnije iz spremljene datoteke `.daq`.
* Odabiri se spremaju s projektom. Skrivene ili pauzirane kamere se preskaču.
* **Neprekidno** na nizovima radi unutar pozadinskog sustava Chloros (novo u verziji 1.3.0): kada svaka kamera na popisu pripada nizu, pozadinski sustav sam pokreće petlju snimanja, zapisujući jednu grupu dok odabire sljedeću, bez zahtjeva po snimci. Pun disk zaustavlja izvršavanje ("Zaustavljeno: disk pun. Spremljeno N, F nije uspjelo"). Pogledajte [Postavke i načini snimanja](capture.md#capture-modes) za napomenu o Microsoft Defenderu u vezi s visokim brzinama snimanja.

Ekvivalentni CLI (isti krajnje točke pozadinskog sustava, ista semantika):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF kompresija za snimke je `deflate` (bez gubitka, zadano) ili `none` — potpune tablice zastavica, raspored mape sa snimkama i pravila za ponovnu obradu nalaze se u [CLI Referenca](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Spajanje DAQ senzora svjetla

Pregledi ispravljeni refleksijom i osvjetljenjem zahtijevaju podatke o svjetlu koje pada odozgo s DAQ senzora (povezanog na kartici **Senzori svjetla**):

* **Redak niza**u bočnoj traci prikazuje**gumb "DAQ · uklj./isklj."** — *uključeno* kada je senzor svjetla na razini niza postavljen **ILI** kada bilo koja od članica kamere ima vlastiti; njegov alatni savjetnik točno navodi koji senzor opskrbljuje koju kameru.
* Dodijelite za cijelu niznicu u postavkama niza → padajući izbornik **Senzor ambijentalnog svjetla**→**Senzor svjetla**. Odabir se zadržava s projektom, prenosi se na svaku pripadajuću kameru, a pojedinačne kamere ga i dalje mogu nadjačati vlastitim senzorom.
* Statusna linija ispod njega prikazuje trenutačno stanje: **Isključeno**→ "Čeka se prvi spektar…" →**"Aktivno — sve kamere u nizu su korekcijski prilagođene osvjetljenju"** → ili, ako u posljednje 3 s nije stigao novi spektar, obavijest o zastarjelosti — zadnje očitavanje se i dalje koristi (očitanja nikada ne istječu na putanji snimanja).

Kad je senzor dodijeljen: postaje dostupan tip izvoza refleksije, pregledi uživo su korektirani po osvjetljenju, prediktivna automatska ekspozicija može koristiti spektar i svako snimanje refleksije zapisuje stvarno korišteno DAQ očitanje kao **datoteku `.daq`** u `daq/` mapi kako bi se snimka mogla ponovno obraditi kasnije. Pored toga, `.daq.cameras.json` imenuje kamere čije okvire opisuje to očitavanje; pri ponovnoj obradi, Chloros uparuje očitavanje samo sa slikama s tih kamera.

## `array-connect` Opcije CLI-a

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatsko otkrivanje svih LATTICE kamera (potrebno ≥2) | **Prvi serijski je MAJSTOR.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sinkronizacijska linija. |
| `--target-fps F` | auto | Brzina ispaljivanja glavnog okidača. |
| `--binning {1,2,4}` | auto | Hardversko biniranje. 'Auto' je cijeli senzor (bez biniranja) kad god bilo koja brzina sličica to omogućuje; 'connect' nikada ne binira samo da bi održao brzinu. |
| `--packed {auto,on,off}` | `auto` | [Pakirani 12-bitni transport](#packed-12-bit-transport): `auto` pakira samo kada pomaže i kada si host može priuštiti raspakiranje. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Stručni prelazak nad odabiračem sinkronizacijske razine. |
| `--wire-ceiling-mbps MB_PER_S` | automatski otkriveno | Budžet mrežne veze hosta u MB/s — CLI oblik polja **Wire Budget**. Smanjite ga ako niz prijavljuje oštećene okvire. Pohranjuje se s projektom, pa ga kasnije ponovno povezivanje vraća. |
| `--no-recommend` | isklj. | Preskočite korak analize mreže. |
| `--no-ptp` | isklj. | Onemogućite PTP (vremenski pečati među kamerama tada nisu usporedivi). |

`lattice array-list`, `array-status` i `array-disconnect` upravljaju trajnom sesijom. Potpuni referentni vodič za podzapovijedi, uključujući poravnanje (`align-calibrate` / `align-apply`) i mrežne alate, nalazi se u [CLI Reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenti u SDK u (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) nalaze se u [SDK reference](../reference/sdk-reference.md). U Python u, wire budget je `connect_array(..., wire_ceiling_mbps=120)`, a podjela između korupcije uživo i neprimitka je na [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
