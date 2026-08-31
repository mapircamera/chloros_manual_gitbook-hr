# Nizovi više kamera

LATTICE **array**je dvije ili više LATTICE kamera povezanih u jednu sinkroniziranu jedinicu. Jedna kamera je**master**: ona generira hardverski GPIO trigger impuls na zajedničkoj sinkronizacijskoj liniji (po zadanom**Line2**), pa se svaki član eksponira u istom trenutku. Chloros dodaje PTP vremensku sinkronizaciju, pregled uživo (po-kamera pločice ili jedan poravnati višebandni kompozit), te sinkronizirano snimanje — svaki prolaz snimanja proizvodi jednu**grupu kadrova** u kojoj sve kamere dijele isti vremenski žig i ID kadra (prikazano kao `fid:N` na izlazu snimanja).

Nizovi su način na koji mono (M3M) kamere proizvode indekse vegetacije — jedna kamera doprinosi jednom pojasu, a niz ih poravnava u višopojasni sloj. Pogledajte [Mono kamere i indeksi vegetacije](mono-indices.md).

Postoje tri ekvivalentna načina za povezivanje niza, a svi oni pokreću isti &quot;smart-prep&quot; tijek:

| Sučelje | Ulazna točka |
| --- | --- |
| GUI | kartica Kamere → **Poveži niz** (plavo gumb) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (prvi serijski = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (prvi serijski = master) |

Smart-prep izvodi, redom: provjeru mrežnih mogućnosti (ICMP DF ping + GVSP probe), odabir sloja sinkronizacije, automatsko smanjenje veličine okvira kako bi odgovarao vezi, omogućavanje PTP-a, automatski odabir formata piksela za svaku kameru, automatsko sjetvo izloženosti iz spremljenog stanja svake kamere i konfiguraciju GPIO okidača na liniji 2.

{% hint style="info" %}
Kamere moraju biti dostupne na vezi prije nego što bilo što od ovoga proradi — pogledajte [Povezivanje kamera](connecting.md) za otkrivanje, adresiranje i preuzimanje kalibracije za prvo povezivanje. Za sustave s više kamera postavke receive-ringa mrežne kartice domaćina jednako su važne kao i brzina veze; potpuna tablica simptoma i rješenja nalazi se u referenci [CLI § Postavljanje i podešavanje mrežne kartice domaćina](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dijalog za povezivanje niza

 kartica Kamere → **Poveži niz**otvara čarobnjaka u tri koraka:**Odabir → Način prikaza → Postavke**.

### Korak 1 — Odabir mastera i slavea



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->Dijaloški okvir skenira mrežu čim se otvori (&quot;Skeniranje mreže...&quot;), a zatim provjerava ožičenje GPIO okidača (&quot;Provjera ožičenja GPIO-a...&quot;). Za izradu niza potrebno vam je najmanje **2 kamere**.

Provjera ožičenja automatski popunjava odabir uloge kad god je to moguće i prikazuje jedan od tri banera:

| Baner | Značenje |
| --- | --- |
| &quot;Detektiran GPIO master — odabiri su unaprijed popunjeni&quot; (zelena) | Provjera je pronašla topologiju okidača; potvrdni okviri za master radio i slave su već popunjeni. |
| &quot;Nije detektiran master - provjerite GPIO kabel&quot; (narančasta) | Nijedna kamera nije primila impuls okidača; provjerite sinkronizacijski kabel. I dalje možete ručno odabrati uloge. |
| &quot;Nema kabela za sinkronizaciju: {serials}&quot; (narančasta) | Navedene kamere nemaju priključen kabel za sinkronizaciju. |

Tablica kamera ima stupce **Kamera / Serijski / IP / Glavni (radio) / Rob (kvačica)**:

* Odaberite točno **jednog mastera**i**jednog ili više slavea**. Ponovnim klikom na radio trenutnog mastera on se briše.
* Kamera označena kao **&quot;Nema sinkronizacijskog kabela&quot;** nikada se ne može odabrati kao slave — slave bez kabela za okidač vječno bi čekao na sinkronizacijskoj liniji i isporučivao bi mrtvi signal. Umjesto toga povežite tu kameru kao samostalnu kameru.
* Kamere koje su već povezane kao samostalne *nisu* onemogućene: povezivanje u niz otkazuje samostalnu sesiju i ponovno otvara kameru unutar niza.

**Sljedeće: Prikazni način →**omogućuje se nakon što se odabere jedan master i najmanje jedan slave.**Ponovno skeniranje** ponovno pokreće otkrivanje i provjeru kabelaže.

{% hint style="warning" %}
**Otkaži** je onemogućeno dok je skeniranje ili provjera u tijeku — otkazivanje usred provjere može uzrokovati pad kamere SDKa na firmveru kamere LATTICE. Pričekajte da se kotačić zavrti.
{% endhint %}

### Korak 2 — Način prikaza



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->| Način | Što dobivate |
| --- | --- |
| **Odvojene kamere** | Jedna živa pločica po kameri, sve se aktiviraju istovremeno kako bi se okviri održali sinkroniziranima. Svaka kamera zadržava vlastitu boju i postavke. |
| **Kombinirane kamere** *(zadano)* | Jedna pločica koja prikazuje poravnati višebandni NDVI/indeksni kompozit. Kamere dijele boju niza. |

Način prikaza mijenja samo prezentaciju uživo — ponašanje snimanja je isto u oba.

### Korak 3 — Postavke niza i predviđeni ishod

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Prilikom ulaska u ovu scenu, Chloros traži **preporuku**od pozadinskog sustava i automatski-primjenjuje kombinaciju ROI-ja i binninga koja odgovara prijemnom prstenu vašeg NIC-a (preferira binning u odnosu na ROI obrezivanje jer binning čuva cijelo vidno polje). Svaka promjena koju napravite ponovno pokreće analizu uživo i ažurira desni panel**Predviđeni ishod**.

Lijeva stupac — postavke:

| Kontrola | Izbori | Zadano | Bilješke |
| --- | --- | --- | --- |
| **ROI (polje pogleda)** | Cijelo (2048×1536) / Polovično (1024×768) / Četvrtasto (512×384) | Cijelo | Obrezivanje senzora: Obrezivanje na polovičnu/četvrtastu veličinu na manju regiju pri nativnoj gustoći piksela. |
| **Binning** | 1× / 2× (zbroj 2×2) / 4× (zbroj 4×4) | 1× | Hardversko binanje: 2×2 = cijelo vidno polje (FoV) uz četvrtinu troškova prijenosa; 4×4 = cijelo vidno polje (FoV) uz 1/16. Skriveno ako kamere ne podržavaju binanje. |
| **Slika na strani kabela** (izlaz) | — | — | Širina×visina nakon binninga koja se zapravo šalje preko veze, zaokružena na višekratnike od 16 (minimum 64). |
| **Rješenost pin-a**| potvrdni okvir | isključeno | Chloros obično automatski pokreće binning pri povezivanju kada predviđena stopa padne ispod**1,5 fps**. Pinning zadržava odabranu veličinu okvira i prihvaća nižu stopu — i pretvara prenatrpan konfiguracijski zahtjev u čvrsto odbijanje povezivanja umjesto automatskog smanjenja. |
| **Stopa okidača** | 0,5–60 fps, korak 0,1 | prazno = automatski | Stopa ispaljivanja okidača kod mastera. Ostavite prazno da bi Chloros to izračunao. |
| **Propusnost**| 20–2000 MB/s, korak 10 | prazno = automatski | Koliko propusnosti host računalo zapravo može apsorbirati, u MB/s —**jedina brojka od koje ovisi cijelo dodjeljivanje niza.** Automatski otkriveno s mrežnog adaptera. Smanjite ga ako niz prijavljuje oštećene okvire: otkrivena vrijednost precjenjuje USB adaptere i zajedničke switcheve. Promjena vrijednosti ponovno pokreće projekciju uživo. |

Desna stupac — **Predviđeni ishod**:

* **Razina sinkronizacije** — &quot;Istovremeno snimanje&quot; (zelena), &quot;Istovremeno snimanje (FTD-razmaknuto emitiranje)&quot; (zelena), &quot;Razmaknuto snimanje (100 ms odmak)&quot; (narančasta), ili &quot;Konfiguracija je prevelika&quot; (crvena).
* **projekcija fps-a** — prikazana kao raspon (&quot;tamno → svijetlo&quot;), jer je brzina sinkroniziranog niza ograničena ekspozicijom najsporije kamere.
* **redak NIC-a** — brzina veze i održivi protok (&quot;NIC {mbps} Mbps · održivi {N} MB/s&quot;).
* **Provjera Sim-emit burst** — može li NIC hosta primiti jedan istovremeni burst sa svih kamera (&quot;Sim-emit burst: X MB · upotrebljivi prsten NIC-a: Y MB ✓/✗&quot;).
* **Provjera budžeta kabela** — agregatna potražnja u stabilnom stanju u usporedbi s gornjom granicom kabela otpornom na kolizije (&quot;Budžet kabela: {potražnja} MB/s potražnja {n} kamera · gornja granica {gornja granica} MB/s ✓/✗ prekoračeno&quot;).
* **&quot;Maksimalno broj kamera na ovoj vezi: {n} — određeno minimalnom propusnošću po kameri, tako da binning ne povećava taj broj.&quot;** — prikazuje se kada ste blizu (ili iznad) gornje granice broja kamera.
* **&quot;OKVIRI ĆE SE GUBITI pri ovim postavkama.&quot;**— crveno upozorenje s razlogom pozadinskog sustava, plus popis blokatora i plave**prijedloge za rješavanje** (&quot;Kako bi se ovaj niz uklopio u mrežu&quot; / &quot;Kako bi se omogućilo istovremeno snimanje&quot;).**Primijeni i poveži** je onemogućeno dok ne postoji projekcija, a njegova oznaka vam govori zašto je odbijeno:

| Označivač gumba | Značenje | Što zapravo pomaže |
| --- | --- | --- |
| &quot;Analizira se...&quot; | Analiza je još uvijek u tijeku. | Pričekajte. |
| **&quot;Previše kamera za ovu mrežu&quot;** | Niz preopterećuje mrežu (provjera agregata nije uspjela). | Manje kamera, jumbo paketi od početka do kraja, ili brži mrežni adapter (NIC). | Manje ROI-ja NEĆE pomoći — pogledajte dolje. |
| **&quot;Smanjite ROI za omogućavanje&quot;** | Slike bi se gubile pri ovim postavkama (provjera burst/ring je neuspjela). | Smanjite ROI, povećajte binning ili popravite NIC receive ring. |



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->Tijekom povezivanja može se pojaviti zeleni **panel za preuzimanje kalibracije** s indikatorom napretka po serijskom broju: prvi put kada se kamera poveže na računalo, Chloros preuzima svoj tvornički paket kalibracije od ~3,8 MB s kamere putem GigE veze (otprilike 70 sekundi po kameri). Pohranjenim kamerama ovaj se panel nikada ne prikazuje. Pogledajte [Povezivanje kamera](connecting.md).

## Propusnost: koliko kamera stane

Propusnost koju može podnijeti niz ovisi o mrežnom kabelu, a ne o Chlorosu, stoga se planerske brojke nalaze u priručniku za hardver: **[Planiranje propusnosti niza](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Što Chloros radi s njima: dijalog za povezivanje pokreće provjeru mreže, predviđa postignjivu brzinu prijenosa okvira i odabire odgovarajući sloj. Ako niz prekorači kapacitet kabela, povezivanje se odbija umjesto da se paketi tiho odbacuju — pogledajte gornje objašnjenje o panelu s predviđenim ishodom.

## Kada nedostaju okviri

Kamera može nedostajati u objavljenoj grupi iz dva potpuno različita razloga,
i za njih su potrebna suprotna rješenja. Chloros ih broji odvojeno umjesto da izvještava o jednom
&quot;nepotpunom&quot; broju koji ne navodi nijedan od razloga:

| Što se dogodilo | Što to znači | Gdje potražiti |
| --- | --- | --- |
| **Oštećen**— kadar je stigao i bio je strukturno oštećen | GVSP gubitak paketa na mrežnoj putanji |**Proračun žice**, NIC prsten za primanje, jumbo paketi, switch |
| **Nikada nije stigao**— nije stigao nijedan kadar | Kamera se nije aktivirala ili ništa nije izašlo iz nje |**M8 sinkronizacijski kabel**, sinkronizacijska linija, je li svaki član aktiviran |

Podjela se ponovno procjenjuje svakih 10 sekundi dok niz prenosi. Iznad 5 % zabilježava se s navedenim oba broja, a svaki oštećeni spremnik izvještava se prvi put kada se to dogodi po kameri, a zatim se zbirno bilježi jednom u minuti kako bi duga sesija ostala čitljiva.

**Okvareni okviri sa statusom &quot;nikada nije stigao&quot; znače da su pokretanje i sinkronizacija kabela savršeni**, a svaki izgubljeni okvir je na mrežnom putu. Rješenje je smanjiti**Wire Budget** i ponovno se povezati.

{% hint style="warning" %}
**Smanjenje stope okidača ne pomaže kod oštećenih sličica.** Uređenje tempa paketa kamere zapisuje se jednom, pri uspostavljanju veze. Smanjenje stope okidača mijenja koliko često se događa burst, a ne koliko brzo sam burst odlazi na mrežu. Na mjerodavnom postrojenju s 4 kamere a
Smanjenje stope okidanja za 5× nije ništa promijenilo, dok je smanjenje mrežnog budžeta s 240 na 200 MB/s istu konfiguraciju smanjilo s 10,4 % oštećenih okvira na nulu.
{% endhint %}

Aktivna matrica se ne može sama ponovno isplanirati — odspojite i ponovno spojite kako bi
program za određivanje vremena spajanja mogao raditi s novim budžetom.

### USB mrežni adapteri ograničeni su na 200 MB/s

USB Ethernet adapter oglašava svoju *Ethernet* brzinu veze, ali ono što on zapravo može održati ograničeno je USB sabirnicom i njezinim upravljačkim programom. USB 10GbE donglu pripisivala se propusnost od otprilike 1000 MB/s — broj koji nitko nikada nije izmjerio — i tempiranje
četiri kamere protiv te fantomske rezerve iskrivile su 6–18 % sličica, dok je niz i dalje izvještavao o zdravoj ciljanoj brzini sličica. Adapteri priključeni na USB sada su ograničeni na
**200 MB/s**. Ograničenje je apsolutno, a ne postotno, jer je limit
USB-bus: USB 1 GbE adapter postiže oko 80 MB/s i nije pogođen.

Ako je vaš host doista brži od ograničenja, povećajte **Wire Budget** kako biste to naznačili.

## PTP vremenska sinkronizacija

*Sinkronizacija* kadrova dolazi od hardverskog okidača; **PTP** (IEEE 1588 PTPv2) osigurava usporedive *vremenske oznake* na svim uređajima. Automatski je omogućen pri povezivanju nizom:

* Host pozadinski program **Chloros** pokreće PTP grandmajstora. Kamere LATTICE i DAQ-E senzori svjetla rade kao robovi (slave) njemu u domeni 0, tako da vremenski žigovi slika i DAQ spektri dolaze na istom satu (~1 ms).
* `--no-ptp` (CLI) onemogućuje ga za rad na klupi — vremenski pečati između kamera tada **nisu** usporedivi.
* Provjerite zdravlje sinkronizacije pomoću CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Sam kartica Kamera nema PTP indikator; na sučeljima za sinkronizaciju pojedine kamere prikazani su samo za čitanje **Uloga**(Master/Slave),**Linija sinkronizacije** i razina mogućnosti niza. Stanje PTP veze DAQ-E prikazano je u detaljima senzora na kartici Svjetlosni senzori.

## Prikaz

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->niza uživo

Glavno područje za prikaz nudi dva izgleda (prebacite u gornjoj traci): **mrežni prikaz**(svaka pločica je ćelija; povucite za promjenu redoslijeda kada je lokot na mreži otključan) i**prikaz popisa**(nizovi pune širine na vrhu, jedna aktivna kamera ispod). Klizač**Feed Zoom** određuje veličinu pločica; pri širini ćelije manjoj od 200 px, preklapajući naziv/fps automatski se skrivaju.**Odvojeni način rada** prikazuje jednu pločicu po kameri. Svaka pločica prikazuje:

* naziv kamere (gornji lijevi kut),
* **prikaz fps-a** (gornji desni) — to je *stvarna brzina snimanja* kamere koju izvještava pozadinski sustav, a ne brzina ispitivanja za pretpregled (pretpregled uživo ograničen je na 30 sličica u sekundi bez obzira na brzinu snimanja),
* točka statusa — zelena (streamanje) / narančasta (učitavanje) / crvena (pogreška),
* **vrtložak zastarjelog kadra** kada nije stigao novi kadar 2 s — normalno za ~5 s nakon bilo kakvog povezivanja/ispitivanja dok pozadinski sustav (backend) rebalansira mrežni proračun (wire budget) među kamerama.**Kombinirani način rada**prikazuje jednu kompozitnu pločicu: pozadinski sustav uklanja šum po pikselima (debayer), skalira, poravnava, uklanja šum, pretvara u radijansu po pojasu (plus DLS reflektancija kada je priključen senzor svjetla), procjenjuje izraz indeksa niza, primjenjuje LUT i prenosi rezultat kao MJPEG. Dok se ne renderira prvi poravnati kadar, pločica prikazuje svoju stanju: &quot;Priprema niza…&quot;, &quot;Kalibracija poravnanja…&quot;, &quot;Čeka se prvi kadar…&quot;, ili — ako je proračun za automatsko ponovno pokretanje poravnanja (~30 s) potrošen — &quot;Potrebno je poravnanje&quot; s gumbom**Kalibriraj poravnanje**.

Korisne činjenice o kombiniranom načinu rada:

* Kompozit je registriran na kadar **glavne**kamere. AE-ROI ciljanje i mjerenje svjetline na kompozitu točni su za glavnu kameru, a približni za pomoćne; upotrijebite**Podijeljeni prikaz** (postavke niza → &quot;Prikaži članice kamere&quot;) za pikselno točne pločice za svaku kameru bez otvaranja dodatnih veza s kamerama.
* **Prikaži slojeve**(postavke niza; zadano isključeno) omogućuje vam odabir prednjeg i pozadinskog sloja — bilo koje članice kamere ili**indeksa**. Kada je prednji sloj = indeks, piksele izvan LUT minimalne/maksimalne vrijednosti prikazuju pozadinski sloj.
* **Rendirana rezolucija** (zadano 720p) postavlja visinu prijenosa uživo *i* veličinu spremljenog izvoza kompozita. Slike pojedinačnih kamera uvijek se izvoze u punoj rezoluciji.
* Poravnanje se izračunava po sesiji i nikada se ne pohranjuje — pogledajte odjeljak za poravnanje u prozoru postavki niza za RMS reziduale i gumb Ponovno kalibriraj.

## Snimanje: nadzor naspram analize

Površine snimanja niza jasno su podijeljene na **razinu nadzora**(zapisujte ono što vidite) i**razinu analize** (zapisujte sirove podatke, kalibrirajte kasnije):

| Radni tijek | Razina | Što se spremava | GUI |CLI |
| --- | --- | --- | --- | --- |
| **Capture**(fotografije) | Analiza | Jedna sinkronizirana grupa sličica po prolazu; datoteke za svaku kameru na svakoj odabranim razinama izvoza (sirovi/dekonvoluirani/radijansko/reflektansno/prikaz/indeks) + pomoćna datoteka `.daq` | gumb**Zabilježi sve** + Postavke snimanja | `lattice array-capture` |
| **Snimanje indeksnog videa** | Promatranje | Prikazani kombinirani indeksni kompozit uživo — 8-bitni, rezolucija pretpregleda, ugrađena LUT; potreban je otvoreni prijenos uživo | ● Snimanje indeksnog videa (kombinirani nizovi) | `lattice array-record` |
| **Raw burst → izrada videozapisa**| Analiza | Sirovi senzorski kadrovi punom brzinom snimanja + manifest + `.daq`, zatim izvanlinijska rekonstrukcija u kalibrirani videozapis radijance / reflektance / indeksa, usklađen vremenski s očitanjima DAQ-a | ⦿ Snimanje raw burst-a →**Izrada videozapisa** | `lattice array-burst` → `lattice array-build-video` |

Pravilo je sljedeće: ako će se pikseli koristiti za *mjerenja*, upotrijebite snimanje ili burst (za potrebe analize); ako samo trebate *pratiti ili demonstrirati* što je niz vidio, snimite indeksni video (za potrebe nadzora).

### Postavke snimanja (GUI)



<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->Zupčanik pokraj **Snimaj sve** otvara prozor Postavke snimanja (zahtijeva otvoren projekt — snimke se spremaju u njega):

* **Način snimanja**:**Jednokratno**(jedan prolaz) /**Neprekidno**(uzastopno; ograničeno brojem snimanja, zadano 1, ili trajanjem, zadano 10 s) /**Interval** (time-lapse: N snimanja svakih X intervala za ukupno Y, zadano 1 svakih 5 s tijekom 1 minute).
* **Tipovi izvoza po kameri**: Raw, Debayered, Radiance, Reflectance, Preview, Index — sve primjenjivo je prema zadanim postavkama uključeno. Radiance/Reflectance su skriveni za kamere s filtrom za uklanjanje zamućenja (RGB-filter);**Reflectance se pojavljuje samo kada kamera ima DAQ senzor svjetla** (vlastiti ili naslijeđeni od niza); Indeks zahtijeva konfigurirani izraz indeksa.
* **Poravnano**(po nizu, zadano**uključeno**): iskrivljuje izvoze članova prema profilu poravnanja niza kako bi izvozi bili poravnati na razini piksela. Raw uvijek ostaje neiskrivljen, ali nosi transformaciju u metapodacima.
* **Najbrži snimak** (prekidač): samo sirovi podaci + dodana DAQ očitanja + besplatni kompozit kombiniranog indeksa, uz preskakanje matematike kalibracije pri snimanju za maksimalnu brzinu — ponovno izgradite radijancu/reflektancu/indeks kasnije iz spremljenog `.daq`.
* Odabiri se zadržavaju s projektom. Skrivene ili pauzirane kamere se preskaču.

Ekvivalentni CLI (isti krajnji točkić pozadine, ista semantika):

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

TIFF kompresija za snimke je `deflate` (bez gubitka, zadano) ili `none` — potpune tablice zastavica, raspored mape snimanja i pravila za ponovnu obradu nalaze se u [referenci CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Spajanje DAQ senzora svjetla

Pregledi ispravljeni refleksijom i osvjetljenjem zahtijevaju podatke o svjetlu odozgo iz DAQ senzora (povezanog na kartici **Senzori svjetla**):

* Bočna traka **red niza**prikazuje**&quot;DAQ · uključi/isključi&quot; gumb** — *uključeno* kada je na razini niza postavljen senzor svjetla **ILI** bilo koja od članica kamere ima vlastiti; njegov alatni savjet navodi točno koji senzor opskrbljuje koju kameru.
* Dodijelite za cijeli niz u postavkama niza → **Senzor ambijentalnog svjetla**→ padajući izbornik**Senzor svjetla**. Odabir se zadržava s projektom, prenosi se na svaku pripadajuću kameru i pojedinačne kamere ga i dalje mogu nadjačati vlastitim senzorom.
* Statusna linija ispod njega prikazuje trenutačno stanje: **Isključeno**→ &quot;Čeka se prvi spektar…&quot; →**&quot;Aktivno — sve kamere u nizu su korekcijski prilagođene osvjetljenju&quot;** → ili, ako nije stigao novi spektar u posljednje 3 s, obavijest o zastarjelosti — posljednje očitanje se i dalje koristi (očitanja nikada ne istječu na putu snimanja).

Nakon dodjele senzora: dostupna postaje vrsta izvoza reflektancije, pregledi uživo se ispravljaju prema osvjetljenju, a prediktivna automatska ekspozicija može koristiti spektar, i svako snimanje reflektancije zapisuje DAQ očitanje koje je zapravo korišteno kao **sporedna datoteka (sidecar) tipa `.daq`** uz sliku, tako da se snimanje može naknadno ponovno obraditi.

## `array-connect` Opcije za CLI

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatsko otkrivanje svih LATTICE kamera (potrebno ≥2) | **Prvi serijski je MAJSTOR.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sinkronizacijska linija. |
| `--target-fps F` | auto | Stopa paljenja okidača majstora. |
| `--binning {1,2,4}` | auto | Hardversko grupiranje. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Stručno preusmjeravanje odabirač­a sloja sinkronizacije. |
| `--wire-ceiling-mbps MB_PER_S` | automatski otkriveno | Budžet mrežnog kabela u MB/s — tekstualni prikaz (CLI) polja **Wire Budget**. Smanjite ga ako niz prijavljuje oštećene okvire. Pohranjuje se s projektom, pa ga naknadno ponovno povezivanje vraća. |
| `--no-recommend` | isključeno | Preskočite korak analize mreže. |
| `--no-ptp` | isključeno | Onemogućite PTP (vremenski pečati među kamerama tada nisu usporedivi). |

`lattice array-list`, `array-status` i `array-disconnect` upravljaju trajnom sesijom. Potpuni referentni vodič za podzapovijedi, uključujući poravnanje (`align-calibrate` / `align-apply`) i mrežne alate, nalazi se u referenci [CLI Reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenti za SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) nalaze se u [SDK referenci](../reference/sdk-reference.md). Prema Python, wire budget je `connect_array(..., wire_ceiling_mbps=120)`, i split uživo/pokvareno/nikad nije stiglo je na [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
