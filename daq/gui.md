# kartica DAQ u programu Chloros

Kartica DAQ — označena kao **Light Sensors** u bočnoj traci programa Chloros — je sučelje za upravljanje uživo za [senzore svjetla DAQ-U, DAQ-M i DAQ-E](README.md): povežite senzore putem bilo kojeg prijenosa, pratite kalibrirane spektre u stvarnom vremenu, izračunajte refleksiju uživo iz para senzora i snimite datoteke `.daq` izravno u svoj projekt.

Jezgra postaje dostupna nakon što se pozadinski sustav Chloros završi s pokretanjem. Grafikoni na jezivcu napajaju se podacima s DAQ usluge sustava Chloros putem aktivne veze koja se automatski ponovno uspostavlja (nakon 2–10 s kašnjenja) ako dođe do prekida; dok je usluga nedostupna, u retku Status senzora piše **No Server**.

Izgled se sastoji od **bočne trake sa senzorima**(po jedan redak za svaki priključeni senzor) i**područja grafikona** (po jedan pločić grafikon za svaki senzor ili grupu).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Povezivanje senzora

Kliknite **Poveži senzor**na vrhu bočne trake. Dijalog za povezivanje otvara se u glavnom području (ili kao preklopni prozor prilikom dodavanja drugog senzora — u tom slučaju pojavljuje se gumb**Otkaži**).

| Kontrola | Ponašanje |
| --- | --- |
| **Vrsta uređaja** | `DAQ-U (USB)` (zadano), `DAQ-M (Bluetooth)` ili `DAQ-E (Ethernet)`. Prebacivanje ponovno pokreće skeniranje za novo odabrani transport. |
| **Priključak / BLE uređaj / Naziv hosta / IP** | Popisuje otkrivene uređaje kao `device - description`; automatski se odabire prvi unos prepoznat kao senzor. Tijekom skeniranja prikazuje `Scanning...` (USB), `Scanning (N)...` s odbrojavanjem od 8 sekundi (BLE) ili `Discovering ethernet sensors (N)...` s odbrojavanjem od 5 sekundi (Ethernet). Prazni rezultati čitaju `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Osvježi** | Ponovno skenira odabrani transport odmah (onemogućeno tijekom BLE/Ethernet skeniranja). |
| **Poveži senzor** | Omogućeno kada je uređaj odabran; mijenja se u `Connecting...` dok se veza uspostavlja. |

Otkrivanje se pokreće samo **dok je dijalog za povezivanje vidljiv na zaslonu** i ponavlja se svakih 15 sekundi samo za odabrani transport — samo otvaranje kartice ne pokreće skeniranje. U slučaju neuspjeha dijalog prikazuje: *"Povezivanje je neuspjelo. Pokušajte isključiti i ponovno priključiti senzor, a zatim ponovno kliknite Poveži."*

Bočna traka se automatski otvara kada se poveže vaš prvi senzor.

{% hint style="info" %}
**DAQ-E se ne prikazuje?**DAQ-E nema LED indikator statusa — provjerite PoE/link indikator na portu switcha ili injectora u koji je priključen i pričekajte nekoliko sekundi nakon uključivanja da se pokrene. Uređaj Chloros mora biti na istoj broadcast domeni (mDNS ne prolazi kroz ruter). Pravilo zaštitnog zida**Chloros Backend (UDP-In)** instalatera dopušta mDNS (UDP 5353), podatke DAQ-E (UDP 5002) i PTP (UDP 319/320); Vatrozid treće strane mora imati otvorene iste priključke. Dvije DAQ-E jedinice na jednoj LAN mreži otkrivaju se odvojeno, svaka pod vlastitim imenom domaćina `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Tip uređaja nudi DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet)</figcaption></figure>

***

## Bočna traka senzora

Svaki priključeni senzor dobiva jedan redak (plus jedan redak po grupi Ambient+Object). Redak se može premještati povlačenjem, a njihov redoslijed također mijenja redoslijed pločica grafikona. Kliknite na redak kako biste taj senzor/grupu postavili kao aktivni grafikon u prikazu popisa.

| Element | Značenje |
| --- | --- |
| Obojena lijeva granica | Boja grafikona senzora. |
| Znak prijenosa | `DAQ-U` / `DAQ-M` / `DAQ-E`, ili zelena značka `REF` za grupu reflektancije okoliš+predmet. |
| Naziv uređaja | Zadano je serijski broj senzora (njegov stabilni identitet za kalibraciju, nazive datoteka `.daq` i usklađivanje pri uvozu); prilagođeni nazivi se spremaju za svaki projekt. |
| **Kalibrirana** pilula (zelena) | Prikazuje se kada je učitana tvornička kalibracijska datoteka senzora, tj. spektri su u stvarnim W/m²/nm. |
| **Dostupno ažuriranje** (smeđa, samo za DAQ-E) | Trenutni firmware je stariji od slike priložene uz ovu verziju Chloros. Tijekom ažuriranja prikazuje napredak uživo (`Flashing… N%`, `Restarting sensor…`, zatim `Updated X → Y` ili `Failed`). |
| Oko | Prebacuje vidljivost ovog senzora na njegovom grafikonu. |
| Zupčanik | Otvara modal za postavke pojedinačnog senzora (dolje). |
| ✕ (crveno) | Odspaja senzor ili uklanja grupu Ambient+Object. |

Iznad redaka nalaze se dva gumba:

* **Poveži senzor** — otvara dijalog za povezivanje (prikazuje se kao `Connecting...` dok je zauzet).
* **Zapisi sve / Zaustavi sve**— započinje ili zaustavlja snimanje `.daq` na**svakom**priključenom senzoru. Potrebno je najmanje jedan senzor**i otvoren projekt** (objašnjenje na alatu: "Otvorite projekt za snimanje"); postaje crven dok god se neko snimanje odvija.

Prazno stanje prikazuje "Nema priključenih senzora".

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Postavke za pojedini senzor (modal s zupčanikom)

Otvorite pomoću ikone zupčanika u retku senzora. Sadržaj slijedi redoslijedom:

* **Retci s informacijama** — Vrsta uređaja (DAQ-U/M/E), Povezanje (`Serial (USB)` / `Bluetooth` / `Ethernet`), Port (COM priključak, BLE adresa ili host) i Serijski broj.
* **Izvješće o kalibraciji: Preuzmi** — preuzima NIST-om provjereni certifikat o kalibraciji ove jedinice (PDF) i otvara ga u vašem pregledniku PDF-ova. Dostupno čim je serijski broj poznat; certifikat se pohranjuje u predmemoriju pri prvom povezivanju.
* **Naziv uređaja** — kliknite olovku za preimenovanje; zadržava se po projektu.
* **Boja linije grafikona** — uzorak boje; zadržava se po projektu.
* **Vrijeme integracije (ms)**— klizač + broj,**1–500 ms**, zadano**32 ms**. Onemogućeno dok je AE uključen.
* **Prosjek sličica**— klizač + broj,**1–50 sličica**, zadano**20**.
* **AE: UKLJUČENO/ISKLJUČENO**— prekidač za automatsku ekspoziciju;**podrazumijevano UKLJUČENO** pri povezivanju. Isključite ga za ručno postavljanje vremena integracije.
* **Zaustavi prijenos / Pokreni prijenos** — pauzirajte ili nastavite prijenos uživo.
* **Snimanje / Zaustavi snimanje** — snimanje po senzoru `.daq` (potrebno je otvoren projekt).
* **Cap** — profil korekcije kapa (sljedeći odjeljak).
* **Redovi s informacijama uživo** — Vrijeme integracije (ms), FPS, Uzorci, Snimanje (crveno `REC` ili `Off`), i Stanje (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Samo za DAQ-E: retci za mrežu, firmware i PTP

* **Hostname / IP** — trenutna adresa jedinice.
* **Firmware** — trenutačna verzija firmvera, plus ćelija za radnju:<version\>

gumb</version\>

**Ažuriraj na \<version\>** pojavljuje se kada ova verzija Chloros a sadrži noviju sliku firmvera za DAQ-E. Ažuriranje se instalira preko mreže za otprilike 30 sekundi; senzor se automatski ponovno pokreće i povezuje, a prekinuti prijenos ostavlja trenutačni firmver nepromijenjenim. Protok napretka je uživo (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), a ćelija čita `Up to date` kada je aktivna.
* **PTP Sync** — trenutačno PTP stanje (vraća se na `unknown`). Firmware DAQ-E v1.2.0+ sudjeluje u IEEE 1588 PTPv2 samo kao robovski sat; backend hosta Chloros PTP grandmaster je, a svaka DAQ-E i LATTICE kamera na LAN-u mu je sluga u domeni 0, održavajući vremenske oznake unutar otprilike 1 ms.
* **Mreža**— adresa na kojoj jedinica radi i kako ju je dobila (DHCP, link-lokalno ili statička), te spremljena postavka koja se primjenjuje nakon ponovnog pokretanja.**Promijeni…**otvara obrazac:**Automatski (DHCP)**ili**Statička IP adresa**s IP adresom, maskom podmreže i opcionalnim prolaskom.**Primijeni i ponovno pokreni** sprema postavke, ponovno pokreće senzor i ponovno ga povezuje. Statička adresa izvan mreže ovog računala odbijena je jer Chloros ne bi mogao dohvatiti senzor nakon ponovnog pokretanja; zatim se pojavljuje potvrdni okvir za njezino dopuštanje, za računalo kojem će se promijeniti adresa ili koje će biti usmjereno na tu mrežu. **Prijenos pri uključivanju**postavlja hoće li senzor započeti prijenos čim se napaja, prije nego što se ikakav softver poveže.**Ponovno pokretanje senzora** ponovno pokreće senzor i ponovno ga povezuje s trenutačnim postavkama. Ponovno pokretanje je odbijeno dok senzor snima. Novost u verziji 1.3.0.

Za grupu Ambient+Object, modalni prozor s zupčanikom prikazuje samo izvore senzora iz grupe, Naziv uređaja i Boju grafičke linije.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Odabir kapa

Spuštajući izbornik **Cap** govori Chloros u koja je fizička kapa postavljena na difuzor senzora i primjenjuje tvornički izmjereni profil korekcije te kapa na svaki spektar. Izbori ovise o modelu:

| Model | Izbori kapa |
| --- | --- |
| DAQ-U | Nema (goli senzor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunčano (kosinusni korektor) |
| DAQ-M | Nema (goli senzor), Sunshine (kosinusni korektor) |
| DAQ-E | Nema (goli senzor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinusni korektor) |

**Zadana postavka za svaki model je Sunčeva svjetlost (kosi korektor)** — MAPIR isporučuje svaki DAQ s ugrađenim poklopcem Sunčeva svjetlost, i to je standardna vanjska konfiguracija: hemisferni vidni kut od 180° s pogreškom cosinusne krivulje ≤ ±4 % do 60° i ≤ ±4,5 % do 70° (ne preporučuje se pri kutu solarnog uzdignuća ispod ~15°), s namjernim slabljenjem (~12×). Vaš odabir ostaje sačuvan u projektu.

{% hint style="warning" %}
**Odabir kapa mora odgovarati fizički postavljenoj kapi.**Ni senzor ni softver ne mogu detektirati koja je kapa postavljena. Odabir utječe i na korekciju uživo i na oznaku zapisanu u svaku datoteku `.daq` — uz prigušenje od ~12× kape Sunshine, Neprijavljena promjena kapa pogrešno ispravlja spektre za otprilike taj faktor. (Uklanjanje i ponovno postavljanje iste kapa ponavlja se s odstupanjem od oko 1,5 %.) Odaberite**None (goli senzor)** samo kada je kapa fizički uklonjena; na DAQ-E-u, opcija "None" i dalje primjenjuje tvornički geometrijski profil za njegov udubljeni stakleni difuzor — to nije bezakcijska operacija — a goli DAQ-E je laboratorijska konfiguracija, a ne podržana terenska konfiguracija.
{% endhint %}

{% hint style="info" %}
Nadogradnja s ranije verzije: prekidač "Sunshine Diffuser Installed" na strani preglednika iz verzije 1.1.0 je uklonjen. Upravljanje kapicama sada se vrši putem ovog profila kapice za svaki senzor, koji se primjenjuje na strani poslužitelja.
{% endhint %}

***

## Područje grafikona

Ljepljiva gornja traka sadrži **prekidač za prikaz u listi ⇄ mreži**i klizač za**zumiranje grafikona** (veličina pločice 200–2000 px). Prikaz se automatski prebacuje na mrežu kada postoji više od jedne grupe grafikona, a natrag na listu kada ih je jedna ili manje. Način prikaza i veličina grafikona zadržavaju se za svaki projekt.**Spektralni grafikon** za svaki senzor prikazuje:

* **X-os** — Valna duljina (nm). Mreža senzora je 340–1010 nm u koracima od 5 nm (135 točaka), interpolirana na 1 nm za prikaz.
* **Y-os**— za kalibrirani senzor (zelena**Kalibrirano**tableta),**Snaga (W/m²/nm)**— radiometrijski kalibrirana spektralna zračenost na sva tri prijenosa — s automatskim SI prefiksom (m/µ/n) odabranim s vrha. Senzor čiji paket kalibracije nije učitan prikazuje svoja sirova očitanja kao**Intenzitet (brojeva)**, nekalibrirano. Kada se na dijagramu preklapaju oba tipa, snaga je na lijevoj osi, a brojevi na desnoj osi.
* Spektralno ispunjenje u boji duge ispod jednog traga; više senzora na jednom dijagramu preklapaju se kao obojene linije s prigušenim ispunjenjem.
* **Hover**— vertikalni pokazivač s valnom duljinom i vrijednošću po senzoru (W/m²/nm za kalibrirani senzor, inače ukupan broj);**povlačenje** za zumiranje (gumb za odzumiranje pojavljuje se tijekom zumiranja).
* Gumb **+** (samo u mrežnom prikazu) za dodavanje senzora na ovaj grafikon ili stvaranje grupe (dolje).
* Naziv uređaja je centriran na vrhu i prikazuje se kotačić dok ne stigne prvi kadar.

**Zasićenje** nije označeno na samom grafikonu: zasićeni senzor prikazuje crveni tekst statusa `SATURATED` i crveni redak `Saturated: Yes` u tablici s podacima uživo. Smanjite vrijeme integracije ili ponovno omogućite AE da biste to uklonili.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tablica s podacima uživo (prikaz popisa)

Ispod grafikona u prikazu popisa, ažurira se svakih 500 ms:

* **Svi modeli**: Uzorak boje svjetla (sRGB iz CIE XYZ), Zasićen (Da/Ne), CIE 1931 X/Y/Z, Hromatičnost x/y, CIE u′/v′, CCT (K), CRI (Ra), dominantna valna duljina (nm), vršna valna duljina (nm), čistoća ekscitacije, Duv, CIE L*/a*/b* i Munsell H/V/C.
* **Kalibrirani senzori**(bilo koji od DAQ-U / DAQ-M / DAQ-E nakon učitavanja tvorničkog paketa kalibracije — zeleni bedž**Kalibrirano** u retku senzora je znak): Ukupna snaga (W/m²), Fotopski luks (lx), Skotopski luks (lx), Omjer S/P, PPFD plus PPFD za Red /Green /Blue (µmol/m²/s) i opične iradijanse — S-čun, melanopski, rodopski, M-čun, L-čun (sve W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupe reflektancije (Ambient + Objekt)

Dva povezana senzora mogu se kombinirati u prikaz refleksije u stvarnom vremenu — bez kamere:

1. U prikazu mreže kliknite **+**na pločicu grafikona i odaberite**Kombiniraj Ambient + Objekt**.
2. Odaberite senzor **Izvor ambijentalnog svjetla**i senzor**Skeniratelj objekta**(dva različita senzora), a zatim**Stvori**.

Chloros izračunava R(λ) = object(λ) / ambient(λ) za svaku valnu duljinu iz dva uživo prijenosa (0 gdje je ambijentalno ≤ 0). Oznaka grupe slijedi kalibracijsku klasu senzora:

* Oba senzora su kalibrirana (paket je učitan) → **"Prividna reflektancija"**.
* Kalibracija bilo kojeg senzora → **"Relativna reflektancija"**.

Grupa se pojavljuje kao zeleni redak `REF` u bočnoj traci i na vlastitom grafikonu (ispunjen duginim bojama, prikaz vrijednosti s 4 decimalne znamenke, povlačenje za zumiranje).

Izbornik **+**također nudi**Dodaj novi senzor** s tri postavke: *Kombiniraj novi senzor* (pridruži ovoj grafikonu), *Premjesti postojeći senzor ovdje* ili *Prikaži novi senzor* (na vlastitoj grafikonu).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tablica indeksa vegetacije

U prikazu popisa, tablica indeksa vegetacije nalazi se ispod grafikona skupine reflektancije, izračunata iz trenutne reflektancije u središtima pojaseva **plavo 450 / zeleno 550 / crveno 670 / NIR 800 nm** (vrijednosti do 4 decimalne znamenke, `---` kada se ne može izračunati; pređite mišem preko naziva indeksa za njegovo puno ime):

* **Uvijek prikazano** (neovisno o ljestvici, bilo koja kombinacija senzora): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Samo kada su oba senzora kalibrirana po snazi** (oba paketa su učitana): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Snimanje datoteka `.daq`

* Snimanje zahtijeva **otvoren projekt** — inače su onemogućeni i gumb za snimanje Sve (u bočnoj traci) i gumb za snimanje po senzoru.
* Datoteke se spremaju u **`<project folder>/light_sensor/`**; nazivi datoteka sadrže ID senzora i vremensku oznaku, a naziv uređaja pohranjuje se zajedno sa snimkom.
* Kada se snimanje zaustavi (Zaustavi, Zaustavi sve ili prekid veze usred snimanja), dovršena datoteka `.daq` **automatski se dodaje u otvoreni projekt** — pojavljuje se u popisu datoteka projekta bez ručnog dodavanja, spreman za upotrebu kao podaci o dolaznom zračenju za [obradu reflektancije](README.md).
* Crveni indikator `REC` prikazuje se u redovima uživo modala postavki tijekom snimanja.

Za kvantitativne brojke iradijance, prosječno uzmite podatke u trajanju od najmanje 15 sekundi — to je karakteristika instrumenta, a ne kvar.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Rasporedi s više senzora i trajnost projekta

* Kombinirajte nekoliko senzora na jednom grafikonu (zajedničke osi), zadržite odvojene grafikone (automatski raspored mreže), premještajte senzore između grafikona, povlačite redove/pločice kako biste ih presložili i sakrijte pojedinačne senzore pomoću prekidača za prikaz/sakrivanje.
* Po projektu, Chloros zadržava: nazive uređaja, boje grafikona, veličinu grafikona, način prikaza i postavke svakog senzora (vrijeme integracije, prosječno računanje sličica, stanje AE, odabir okidača).
* **Ponovno otvaranje projekta automatski ponovno povezuje njegove senzore** po adresi — COM priključak za DAQ-U, BLE uređaj za DAQ-M, mDNS naziv domene za DAQ-E (rješava se čak i ako se IP jedinice promijenio) — i ponovno primjenjuje spremljeni profil kapice, prosječno vrijeme sličice, stanje AE-a i ručno vrijeme integracije svakog senzora.

***

## Spajanje kamere (DLS)

Nema se što upariti. Za razliku od DLS tijekova rada s dronovima koji unaprijed vežu senzor svjetla za kameru, Chloros usklađuje DAQ podatke sa slikovnim materijalom naknadno: u trenutku uvoza/obrade, očitanja s uređaja `.daq` interpoliraju se na vremensku oznaku ekspozicije svake snimke. Snimajte s bilo kojim priključenim senzorom (snimke `.daq` automatski se spremaju u projekt), a obrada reflektancije pronalazi odgovarajuća očitanja prema vremenu — pogledajte [DAQ senzori svjetla](README.md) kako se koriste podaci o dolaznom svjetlu. Za **uživo**snimanja LATTICE-a odaberite senzor u padajućem izborniku**Senzor svjetla** na kartici Kamere; svako snimanje zatim sprema očitanje koje je koristilo, usklađeno samo s kamerama tog snimanja (vidi [Radni procesi reflektancije](reflectance.md)).</version\>
