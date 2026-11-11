# Postavke kamere

Jezgra **Kamere**je**Chloros ova**upravljačka ploča uživo za kamere LATTICE: područje glavnog prikaza koje prikazuje svaku povezanu kameru kao pločicu uživo i bočnu traku koja se pomiče između triju stranica —**popisa kamera**,**paneela s postavkama**(po kameri, niz ili postavke snimanja — jedna po jedna), i**Kalkulator indeksa**. Ova stranica opisuje svaku kontrolu u popisu kamera, u odjeljku s postavkama po kameri i u odjeljku s postavkama niza. Načini snimanja, odabir vrste izvoza i način snimanja "Snimanje svega" nalaze se na povezanoj stranici [Postavke i načini snimanja](capture.md).

 kartica Kamera pojavljuje se u bočnoj traci kada je pozadinski sustav Chloros spreman. Sve kontrole u nastavku komuniciraju s lokalnim pozadinskim sustavom putem `127.0.0.1:5000`; promjene se odmah primjenjuju na kameru uživo, osim ako nije drugačije naznačeno.

## Vrste kamera korištene na ovoj stranici

Kontrole se prikazuju ili skrivaju ovisno o tome koja je vrsta kamere odabrana. Priručnik koristi ove pojmove u cijelom tekstu:

| Pojam | Značenje | Filtriraj kanale |
| --- | --- | --- |
| **RGB kamera** | LATTICE M3C s FRGB filtrom (model sadrži `-FRGB`) | Red / Green / Blue |
| **Bayer multispektralna** | LATTICE M3C s filtrom FRGN, FOCN ili FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — jedan uskopojasni filtar, jedna kalibrirana traka | Jedna traka |
| **Član niza** | Kamera povezana kao dio sinkroniziranog niza (kombinirani ili odvojeni prikaz) | Prema svom filtru |

RGB kamere dobivaju fotometrijsku obradu (bilans bijele, profili boja, gamma); multispektralne i mono kamere dobivaju radiometrijski lanac i preskaču fotometrijsku kontrolu. Članovi niza ručno prenose postavke na razini struje (format piksela, rezolucija, binning, okidač, brzina sličica) na niz — ti se retci u prozoru za svaku kameru postaju samo za čitanje i umjesto toga premještaju u prozor s postavkama niza.

## Glavni područje za prijenos



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Kada nijedna kamera nije povezana, područje za prijenos prikazuje uvodni ekran **"Povežite kameru za početak"**s dva gumba:**Poveži kameru**(zelena, otvara dijalog za povezivanje pojedinačne kamere) i**Poveži niz** (plava, otvara dijalog za povezivanje niza). Sami dijalozi za povezivanje dokumentirani su u [Povezivanje kamera](connecting.md); koncepti niza (sinkronizacija, razine, propusnost) u [Mreže s više kamera](arrays.md). Kada otvorite spremljeni projekt koji sadrži kamere, umjesto toga se prikazuje kotačić s porukom "Ponovno otvaranje N spremljenih kamera…", dok Chloros obnavlja streamove iz posljednje sesije.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Gornja traka

| Kontrola | Što radi |
| --- | --- |
| **Prebacivanje načina prikaza**| Prebacuje između**mrežnog prikaza**(sve pločice kao ćelije) i**prikaza u listi**(redovi pune širine: svaki kombinirani niz kao jedan složeni red, svaki član**zasebnog** niza kao vlastiti red sa svojim prijenosom i tablicom cilja kalibracije, a zatim JEDNA samostalna kamera — aktivna — ispod). Pomoćni savjeti: "Prebacite se na prikaz mreže" / "Prebacite se na prikaz popisa". |
| **Zaključavanje mreže**(zaključani simbol) | Zadano**zaključano** — pločice su fiksirane na mjestu. Otključajte da biste povukli i posložili pločice u bilo koji utor (praznine se čuvaju). Mreža se automatski ponovno zaključava svaki put kad se poveže nova kamera. Savjeti za alat: "Odljučajte mrežu (omogućite premještanje pločica)" / "Zaključajte mrežu (zamrznite pločice na mjestu)". |
| Klizač **Zoom izvora** | Veličina pločice, od 60 px do pune širine spremnika. Stanice zadržavaju omjer stranica 4:3. Pri širini ćelije manjoj od 200 px preklapaju se naziv i fps kako bi pločica ostala pregledna. |

### Pločice izvora

Svaka kamera iscrtava složenu pločicu uživo; kamera može dodatno prikazati tri pločice u sivim tonovima s **razdvajanjem po kanalima** (per-channel split) (vidi [Razdvajanje po kanalima](#prikaz-preklapajućih-overlaya-iznad-uživo-prijenosa)), a nizovi prikazuju kombiniranu pločicu. Aktivna pločica ima prsten odabira u boji kamere (ili niza).

Prilikom prelaska mišem preko pločice otkriva se gumb za zatvaranje **X**:

* Zatvaranje **kompozitne** pločice dok su njezini kanalski splitovi vidljivi prikriva samo kompozit.
* Zatvaranje **zadnje vidljive pločice samostalne kamere** odspaja tu kameru.
* **Split pločice članova kombiniranog niza nikada ne odspajaju** kameru — one se samo prikrivaju.

Kada je mreža otključana, povucite bilo koju pločicu u bilo koji utor; raspored se sprema s projektom.

## Bočna traka — popis

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamera Prva stranica bočne trake prikazuje sve povezane kamere i nizove:

* **Poveži kameru**(zelena) /**Poveži niz** (plava, prikazuje "Otkrivanje..." tijekom skeniranja). Oba su onemogućena dok je otvoren dijalog za povezivanje.
* **Zahvati sve** (crvena) — zahvaća svaku navedenu kameru s vrstama izvoza odabranim u postavkama snimanja. Zahtijeva otvoren projekt. Potpuno dokumentirano u [Postavke i načini snimanja](capture.md).
* **Zupčanik postavki snimanja** (pored Snimi sve) — otvara [preglednik postavki snimanja](capture.md#the-capture-settings-pane). Onemogućeno je bez projekta ili tijekom snimanja.

### Redovi kamera

Svaki red kamera prikazuje obojenu granicu (prilagođena boja kamere), oznaku "CAM" — s plavim **M**(master) ili zeleno**S** (slave) slovo za ulogu članova niza — i prikazano ime. Zadano ime je `LATTICE-MODEL (serial)`; preimenujte ga u kartici postavki za svaku kameru. Gumbi u retku:

| Gumb | Učinak |
| --- | --- |
| **Oko**| Prebacivanje vidljivosti. Skrivene kamere napuštaju mrežu i**isključene su iz snimanja svega**. |
| ****Zupčanik** | Otvara okno postavki za svaku kameru (sljedeći odjeljak). |
| **Pauza / Reprodukcija**| Zamrzava pregled uživo**samo na zaslonu** — pozadinski snimanje se nastavlja. Pauzirane kamere ne mogu snimati. |
| **X** | Odspojiti. Sučelje se odmah ažurira (optimistično); samom prekidu veze u pozadini može trebati 10–30 s da se dovrši. |

### Redovi niza

Redak niza prikazuje bedž "ARRAY" u boji niza, naziv niza (može se preimenovati u postavkama niza) i gumb **DAQ · uključi/isključi**—**uključen** kada je senzor svjetla na razini niza postavljen *ILI* bilo koji član ima senzor po kameri; Njegov alatni savjet prikazuje točno koji senzor što opskrbljuje. Kamere članova su ispod navedene uvučeno sa svojim redovima. Gumbi u retku niza: **oko**(sakriva/prikazuje SVE članove zajedno),**zupčanik**(panej s postavkama niza),**X**(odspojite cijeli niz).

Status senzora svjetla (DLS) koji se koristi u retcima nizova i u paneju s postavkama niza ima četiri stanja:**isključeno**,**čekanje**(još nema spektra),**aktivno**(spektrum je stigao u posljednje 3 s), i**neaktualno** — nema novog spektra u 3 s, ali se posljednje očitanje *još uvijek koristi* (DAQ očitanja nikada ne istječu na putu snimanja).

Dok je ne presložite, popis prikazuje prvo nizove, zatim samostalne kamere, pri čemu su kamere (i članovi svakog nizova) u **redoslijedu kamera**: M3C (Bayer) ispred M3M (mono); među M3C, FRGB, FRGN, FOCN, zatim FNGB; M3M prema valnoj duljini filtra, prvo one s najnižom vrijednošću. Zadani prikaz mreže slijedi isti redoslijed. Možete povlačiti samostalne kamere i cijele grupe nizaka jedne pokraj drugih u bočnoj traci kako biste promijenili redoslijed na popisu; članovi nizaka se ne mogu neovisno povlačiti.

## Panel postavki po kameri

Otvorite pomoću **zupčanika** u retku kamere. Panel se klizi preko popisa kamera.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Naslov**:**uzorak boje**kamere (kliknite za otvaranje ugrađenog odabira boja — postavlja boju okvira bočne trake i prstena za odabir pločice),**naziv**s gumbom olovke**Preimenuj**(pohrana praznog naziva vraća se na zadano `MODEL (serial)`), i**×** za zatvaranje.

### Živi histogram

Na vrhu prozora nalazi se živi histogram svjetline izračunat iz **JPEG** pretpregleda brzinom od ~8 Hz. Prosjek je ponderiran prema Bayerovoj maski — (R+2G+B)/4 — kako bi odgovarao vlastitom AE mjerenju kamere.

* **narančasta isprekidana linija** = cilj za AE.**Povucite je vodoravno za ponovno ciljanje** — jedna se naredba šalje pri puštanju, a povlačenje prebacuje način rada AE cilja na ručni.
* **zelena puna linija** = stvarna srednja lumina (ono što AE trenutno isporučuje).
* **Gumb**RGB ** (gornji desni kut): omogućuje/onemogućuje histograme s preklapanjem po pojasu obojene prema filtru kamere (npr. na FRGN: sivi NIR, zeleni, crveni). Na mono (M3M) kamerama gumb je označen kao "MONO" i onemogućen — mono uvijek prikazuje histogram lumine jednog pojasa.
* Oznake X-osovine prate dubinu bita senzora trenutnog formata piksela: 0..255, 0..1023, 0..4095 ili 0..65535.

### Redovi

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s informacijama o kameri

| Redak | Ponašanje |
| --- | --- |
| **Model** | Samo za čitanje (npr. `LATT-M3C-L87-FRGN`). |
| **Radiometrijska kalibracija**| Znak**Green**"Aktivno" s natpisom koji prikazuje razinu kalibracije, hash, datum kalibracije i popis pojaseva, učitano iz paketa za kalibraciju kamere (vidi [Factory Radiometric Calibration](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skriveno za RGB kamere** — one imaju fotometrijsku kalibraciju bijele ravnoteže, a ne radijansu po pojasu. |
| **Izvješće o kalibraciji**| Gumb**Preuzmi** — otvara PDF certifikata o NIST kalibraciji kamere za određeni serijski broj u pregledniku vašeg operativnog sustava. Ako certifikat još nije pohranjen u predmemoriju, Chloros umjesto toga prikazuje obavijest. |
| **Serijski broj** | Samo za čitanje. |
| **Firmware**| Prikazuje trenutnu verziju, a zatim pronalazi dostupnu verziju za ovaj model (pohranjeno po modelu — niz N kamera provjerava poslužitelj jednom). Prikazuje: "Provjera…" → gumb**"Ažuriraj na X"**→ "Programiranje…" → "Ažurirano s A na B" / "Neuspjelo: …" / "Preskočeno: …" / zelena**"Ažurirano"**. Pop-up obavijest gumba za ažuriranje: "Ponovno postavljanje na tvorničke postavke + flashanje + reprogramiranje UserSet1. ~2–3 minute; ne isključujte." |
| **IP** | Samo za čitanje. |
| **Temperatura** | Samo za čitanje, osvježava se svakih 3 s. Postaje narančasta na ≥65 °C i crvena s ⚠ na ≥75 °C. |
| **Cilj kalibracije** kvačica | Omogućuje detekciju ArUco cilja reflektancije s tablicom za validaciju NDVI po panelu ispod prijenosa uživo (pregled popisa) — pogledajte [tablicu Cilj kalibracije](#the-calibration-target-tablica). Samo za sesiju — uvijek se otvara isključeno. |
| padajući izbornik **Senzor svjetla** | Povezuje DAQ senzor svjetla (DAQ-E/M/U, s popisa kartice Senzori svjetla) s ovom kamerom za korekciju osvjetljenja odozgo (DLS) i prediktivnu automatsku ekspoziciju. ""Nijedan" briše vezu. Ako nisu priključeni senzori, padajući izbornik prikazuje "(nema priključenih senzora — otvori karticu DAQ)". Poveznja se sprema s projektom. |

#### Tablica cilja kalibracije

Tablica je **samo za čitanje za validaciju** ("samo validacija — ne mijenja kalibraciju"). Njezin zaglavlje sadrži nazive serijski broj kamere, filtar, broj cilja i veličinu markera u pikselsima, a ima jedan redak po panelu:

* **Panel**, zatim jedna stupac po pojasu s vrijednošću panela — Bayer kamera prikazuje svoja tri pojasa po nazivu i valnoj duljini (npr. FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); **mono (M3M) kamera prikazuje svoju jedinu traku** (npr. `F750`) i nema stupaca za NDVI.
* **Sat** — udio piksela panela koji su zasićeni (žuto-narančasta od 1 %, crvena od 5 %).
* **NDVI Meas**,**NDVI Theo** (referentna reflektancija panela za NDVI) i **Δ**— samo za Bayerove multispektralne kamere. Mali znak nakon "NDVI Meas" označava na temelju čega je izračunata izmjerena vrijednost NDVI:**ρ**(reflektancija — dostupno je očitanje senzora svjetla),**L**(zračenje — nema očitanja senzora svjetla) ili**DN**(sirovi vrijednosti kamere). Samo je**ρ** NDVI usporediv s NDVI Theo: za L i DN stupac Δ je zasiven i napomena to navodi ("Nema očitanja senzora svjetlosti: NDVI je iz radijance, stoga Δ nije provjera kalibracije" / "NDVI je iz sirovih vrijednosti kamere, stoga Δ nije provjera kalibracije"). Kod ρ, Δ je zelen ako je manji od 0,02, žut ako je manji od 0,05, a crven inače.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Ekspozicija i pojačanje

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Svi numerički unosi ovdje koriste kotačiće za ubrzano pomicanje: dodir = ±1, držanje >1,5 s = ±10, držanje >3 s = ±100. Vrijednost se šalje kameri kada otpustite.

| Kontrola | Raspon / opcije | Zadano | Primjenjivo na | Što radi |
| --- | --- | --- | --- | --- |
| **Ekspozicija (us)**| Min/maks kamere uživo | Auto | Sve | Vrijeme ekspozicije u mikrosekundama, s**Auto/Ručno**prekidač. Auto = kontinuirana automatska ekspozicija s kamere. Postavljanjem ručne ekspozicije,**Pojačanje** ostaje na Auto ako je bilo, slobodno koristite cijeli raspon pojačanja kamere, sve dok ne uredite pojačanje ili ga ne postavite na Ručno. |
| **Pojačanje (dB)**| Minimalna/maksimalna vrijednost kamere uživo (npr. do 48 dB) | Ručno (isključeno) | Svi | Analogno/digitalno pojačanje sa vlastitim prekidačem**Automatski/Ručno**. |
| **Ciljana svjetlina za AE**| 0–255 | 80, način**Automatski**| Svi (može se uređivati kada je AE ili automatsko pojačanje uključeno) | Svjetlina koju AE cilja. U**Auto**(zadano) pozadinski kontroler temeljen na histogramu samostalno odabire ciljanu vrijednost, održavajući ekspoziciju na 60–75 % maksimalne vrijednosti senzora. Upisivanje vrijednosti ili povlačenje narančaste linije histograma prebacuje ga na**Manualno**. |
| **Izravnavanje AE-a** | 0,5–40, korak 0,1 | 8,0 | Sve | Prigušivanje AE-a. Napomena alata: "Niže = AE brže reagira (može pulsirati pri visokim fps-ima). Više = glađe / sporije." Vrijednosti znatno ispod zadane mogu uzrokovati pulsiranje AE-a i destabilizirati streaming pri visokim brzinama sličica; 8.0 je stabilna zadana vrijednost. |
| **AE regija od interesa**| Polje za potvrdu**Omogući**+ gumb**Ciljanje**| Isključeno | Svi | Kada je uključeno, AE mjeri samo isprekidan zeleni područje umjesto cijelog kadra.**Ciljanje** aktivira postavljanje klikom na prijenosu uživo: klikom se centrira regija na 30 % okvira; klik-i-povlačenje iscrtava prilagođeni pravokutnik (minimalno 5 % × 5 %). Ciljanje se automatski deaktivira nakon jednog postavljanja. Područje se mapira natrag na nativne koordinate kamere prema bilo kojoj rotaciji/ogledalu koje ste postavili i spremuje se s projektom. |
| **Brzina prilagodbe AE-a** | 0,1–5, korak 0,1 | 1,0 | Samo za članove niza | Koliko brzo cilj automatskog AE-a prati promjene u svjetlini scene; 1,0× ponovno provjerava svakih 2,5 s. |
| **Zaštita od preosvjetljenja** | Strogo (1 %) / Normalno (5 %) / Popustljivo (15 %) | Strogo | Kamera koje izlažu postavku | Koliki dio kadra može biti odsječen na bijelo prije nego što AE zamrači sliku. |

{% hint style="info" %}
**Zahtjev za osvjetljenje za Bayerove multispektralne kamere (RGN / OCN / NGB):** scena mora imati dovoljno svjetla u sva tri kanala ili kalibracija neće ispravno raditi — jedna ekspozicija senzora pokriva sva tri spektra. Koristite DAQ senzor svjetla za mjerenje svjetla ili pređite na potpuno mono (M3M) kako bi svaki pojas dobio svoju ekspoziciju. Ako snimanje prekrši ovo pravilo, Chloros će to otkriti i upozoriti vas (obavijest o unmix-clampu). U načinu uživo, pločica s podjelom kanala prikazuje **"Low-confidence band data · N%"** kada je unmix zaključao tu traku na 25 % ili više svojih piksela.
{% endhint %}

{% hint style="info" %}
**Kako automatska ekspozicija troši ekspoziciju i pojačanje (1.3.0).** Kada je pojačanje na automatskom načinu rada (Auto), Chloros drži pojačanje na 0 dB sve dok ekspozicija ne dosegne najdužu dopuštenu vrijednost (gornju granicu AE-a, razdoblje okidanja niza ili gornju granicu ekspozicije DAQ-a); tek tada pojačanje raste. U nizu, prioritet ima brzina okidanja: automatska ekspozicija popunjava razdoblje koje ta brzina određuje, zatim povećava pojačanje, tako da se zamračenje scene više ne smanjuje Auto brzinu okidača (u 1.2.0 je to bilo). **Ručna**ekspozicija na članu niza dulja od razdoblja okidanja usporava okidanje niza kako bi se uklopila; Automatska stopa se vraća čim se svaki član koji ju je usporio vrati na automatsko eksponiranje. Kada su i**Ekspozicija**i**AE ciljana svjetlina** postavljeni na Automatski, a DAQ senzor svjetlosti daje očitanje, ekspozicija svake kamere se zadržava nakon što se automatsko eksponiranje stabilizira, sve dok izmjerena svjetlost ostane unutar otprilike ±25 %; veća, trajna promjena svjetla, ponovno pokretanje postavki (na primjer promjena ROI-ja) ili vaša vlastita promjena ekspozicije ponovno je aktiviraju.
{% endhint %}

{% hint style="info" %}
**Kako se ponaša automatska ekspozicija (1.3.0).**

* Ručna ekspozicija ostavlja pojačanje na Auto dok ne promijenite pojačanje.
* Mali odsjaji ne zamračuju scenu: spekularni odsjaj do 1 % kadra se zanemaruje, a kod panela poznatog kalibracijskog cilja do 0,5 % panela.
* Gornja granica ekspozicije DAQ-a primjenjuje se samo dok je kalibracijski cilj u vidokrugu; ako se ne vidi nijedan panel, kamera eksponira scenu.
* Ekspozicija ostaje stabilna pri konstantnom svjetlu i prilagođava se unutar nekoliko sekundi nakon velike promjene. Snimak više ne fiksira ekspoziciju, a promjena ROI-ja ponovno prilagođava ekspoziciju.
* Ciljana svjetlina ne prelazi izvan raspona kameredometa senzora, pa povratna svjetlost ne uzrokuje prebljesak na slici.
* Kamera čije automatsko izlaganje prestane reagirati na jakom suncu zadržava se na razini svjetla u kojem se nalazi, koju mjeri senzor svjetla ili, ako nije priključen, vlastitim slikama, i otpušta se kada se svjetlo promijeni. Tamni ili prekriveni senzor svjetla ne ometa to.
{% endhint %}

### Format piksela i rezolucija

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Članovi niza** prikazuju retke "Trenutno" (format + ŠxV) i "Binning" samo za čitanje s napomenom "Postavi u postavkama niza" — ponovno pokretanje prijenosa na jednom članu prekinulo bi sinkronizaciju, stoga se njima upravlja u [podučaku za postavke niza](#array-settings-pane).**Samostalne kamere** dobivaju:

| Kontrola | Opcije | Što radi |
| --- | --- | --- |
| **Pikselni format** | 8-bit / 10-bit / 12-bit / 16-bit | Dubina bita senzora; obitelj formata je senzorska. Zadano je 12-bit, a projekt sprema ono što postavite, uključujući i 8-bit, i ponovno se povezuje na tu postavku. |
| **Razlučivost** | Puna / Polovna / Četvrtina | U odnosu na trenutno biniranje: Puna = 2048/N × 1536/N za N×N binning. |
| **Binning** | 1x1 (nema) / 2x2 / 4x4 | Hardverski N×N binning — veće vrijednosti smanjuju rezoluciju, ali povećavaju SNR i brzinu sličica. Promjena pokreće stream iznova i resetira svako ROI na novo puno vidno polje. |
| **Trenutno** | samo za čitanje | Stvarna WxH i (x, y) pomak koji je na snazi. |

### Prikaz uživo

Sve u ovom odjeljku odnosi se **samo na prikaz**— mijenja ono što vidite u prijenosu uživo, dok snimke spremljene u datoteku ostaju linearne i nepromijenjene — uz jednu iznimku:**Vignette** je radiometrijska i utječe i na izvoze (objašnjeno u nastavku).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Kontrola | Opsež / mogućnosti | Zadano | Na što se primjenjuje | Što radi |
| --- | --- | --- | --- | --- |
| **Razlučivost renderiranja** | 360p (najbrže) / 480p / 720p / 1080p / Izvorna razlučivost senzora (najsporije) | 720p | Sve | Visina na kojoj pozadinski sustav pokreće lanac radiometrijskog pregleda. Niža postavka smanjuje brzinu sličica bez promjene vidnog polja. |
| **Indeks**| Kvadrat za potvrdu + zupčanik | Isključeno | Samo za Bayer multispektralni,**ne** za članove kombiniranog niza | Pregled indeksa vegetacije uživo. Zupčanik otvara zajednički [Kalkulator indeksa](#index-calculator-pane) unaprijed natovaren filtrima kamere (npr. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Prilagođeni izraz plus LUT (uključi/isključi, razina zadano 7 stopa, min zadano 0,2, max zadano 1) izračunava se na svakom sličcu pretpregleda. Članovi kombiniranog niza skrivaju ovaj redak — niz posjeduje jedan zajednički indeks. |
| **Bijela ravnoteža** | Isključeno / Jednom / Kontinuirano + gumb za ponovno određivanje | Kontinuirano | Samo zaRGB

| Kontinuirana bilanca bijele. Gumb za osvježavanje ponovno određuje bilancu bijele boje na temelju trenutnog DLS spektra (onemogućen kada je način rada isključen). |
| **Gamma** | Uključi / Isključi | Uključeno | SamoRGB

| Prikaz gammine (γ = 2,2 LUT) u uživo. Pohranjene snimke ostaju linearne. |
| **Uklanjanje šuma** | Kvačica + jačina 0–100 | Isključeno / 50 | Sve (po kameri, čak i unutar nizova) | Bilateralni filter na pregledu uživo. Viša vrijednost = glađe, ali mekše detalje. |
| **Oštrina** | Potvrdni okvir + jačina 0–100 | Isključeno / 30 | Sve | Neoštra maska na pregledu uživo, primjenjuje se na kraju. Može pojačati šum. Samo za pregled. |
| **Vinjeta**| Kvačica + jačina 0–100 | Isključeno / 0 | Sve | Ručno uklanjanje vinjete za preostale artefakte (osvjetljuje kutove), slojevito na procjeni pametne vinjete niza.**Radiometrijsko — utječe na prikaz uživo I izvoz**, za razliku od Uklanjanja šuma/Oštrine. |
| **Profil boje** | Raw / Linearan / Prirodan / Poboljšan / Prilagođena temperatura | Prirodan | Samo zaRGB

| Vidi dolje. |
| **Temperatura boje** | 2000–10000 K, korak 100 | 5500 K | RGB, samo prilagođeni profil temperature | Prikvači bilancu bijele na fiksnu koreliranu temperaturu boje (DLS ulaz se zanemaruje). Posljednja odabrana vrijednost Kelvina pamti se pri prebacivanju profila. |
| **Istaknuta područja** | −100 do 0, korak 1 | 0 (isključeno) | Samo za RGB | Novost u verziji 1.3.0. Zatamnjuje najsvjetlija, najbjelja područja slike (0 = nepromijenjeno, −100 = najjače). Primjenjuje se na pregled uživo **i na snimke na razini zaslona (Pregled)**, koji bilježe postavku (`Chloros:DisplayHighlights`); izvoz radijance i reflektance nikada nije pogođen. Ne može oporaviti podatke s senzora koji su presječeni — za to smanjite ekspoziciju. Zebrasti uzorak prati zamračenu sliku. Onemogućeno s Raw profilom boja ("Ne primjenjuje se s Raw profilom boja."). |
| **Zasićenje** | 0–200 (100 = neutralno) | 100 | Samo RGB | HSV zasićenje u uživo pretpregledu. |
| **Kontrast** | 0–200 (100 = neutralno) | 100 | Samo RGB | Linearni kontrast oko sive sredine na pregledu uživo. |
| **Ogledalo H / Ogledalo V** | Zaštićene okvire | Isključeno | Sve | Okreni pretpregled vodoravno / okomito. |
| **Rotacija**| 0° / 90° / 180° / 270° | 0° | Sve | Okreni pretpregled. Orijentacija se primjenjuje na kraju lanca pregleda pozadine.**Spremljeni snimci (1.3.0):**samostalni proizvodi kamere Preview, Radiance, Reflectance i Index okreću se pomoću ogledala/rotacije, dok se**Raw i Debayered ostaju u izvornoj orijentaciji kamere**; svaka snimljena datoteka bilježi postavku u svom XMP-u, tako da se pri kasnijoj obradi raw datoteke primjenjuje na isti način.**Datoteke članova niza ostaju u izvornoj orijentaciji** (postavka se bilježi, ali ne primjenjuje), a kompozitni prikazi niza je ignoriraju. |**Semantika profila boja** (RGB kamere):

* **Raw** — potpuno zaobilazi lanac obrade.
* **Linearan** — signal tame + ravna polja + balans bijele; bez matrice boja, bez gamma.
* **Prirodno** *(zadano)* — linearno plus izmjerena matrica za korekciju boja i tonalna krivulja prilagođena sceni.
* **Unaprijeđeno**— Prirodno plus živost boja i lokalni kontrast (CLAHE). Dodatni trošak odnosi se**samo na pregled uživo** — spremljene fotografije uvijek dobivaju konačnu obradu u punoj kvaliteti, bez obzira na profil.
* **Prilagođena temperatura** — Natural s ravnotežom bijele pričvršćenom na odabranu vrijednost Kelvina.

{% hint style="warning" %}
Za načine rada Prirodno, Poboljšano i Prilagođena temperatura, ploča prikazuje napomenu o tonu: sličice se posvjetljuju prema vlastitoj sceni, stoga spremljene *prikazne* slike nisu usporedive iz sličice u sličicu. **Izvozite radijancu ili reflektancu za mjerenja.**
{% endhint %}

### Preklapajući slojevi zaslona (nacrtani preko prijenosa uživo)

Oni su isključivo na strani klijenta — iscrtavaju se preko videa, a nikada ne dodiruju stream niti snimke.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Preklapajući sloj | Kontrole | Zadano | Što radi |
| --- | --- | --- | --- |
| **Zebra** | Kvačica + prag 200–255 | Isključeno / 250 | Magentne dijagonalne pruge na odsječenim pikselima. |
| **Križanac** | Kvačica | Isključeno | Označavanje središta kadra. |
| **Rešetka** | Isključeno / 3 × 3 / 9 × 9 | Isključeno | Rešetka za kompoziciju. |
| **Histogram** | Potpuna opcija + širina 0,10–0,90 okvira | Isključeno / 0,25 | Traka histograma na prijenosu. |
| **Focus Peak** | Potvrdni okvir + prag 20–200 + uzorak boje | Isključeno / 80 / `#ff5722` | Sobelovo isticanje rubova za fokusiranje. |
| **Razdvajanje kanala** | "Prikaži razdvajanje (Red / Green / NIR)" / gumb "Sakrij podjele" | Skriveno | Dodaje tri neovisne pločice u sivim tonovima za svaki kanal uz kompozit (oznaka gumba prati filtre kamere). Svaka podijeljena pločica može se premještati i dijeli boju okvira kamere. Nije dostupno na mono kamerama. Sprema se s projektom. |

### Spot mjerač

* **Kutija za potvrdu**Klikni za uzorkovanje**: kliknite na uživo prikaz kako biste uzorkovali jedan piksel (retikul s križem ga označava), ili povucite regiju za prosjek piksela.**Očisti**briše uzorak i retikul. Međusobno isključivo s načinom rada AE-ROI**Ciljanje**.
* **Prikaži**padajući izbornik:**Sirovi (bitna dubina)**— nativni digitalni brojevi na bitnoj dubini senzora (npr. 12-bit → 0..4095) — ili**Prikaz (8-bit)** (zadano). Kada je aktivan indeks uživo, prikaz Display umjesto toga prikazuje izračunatu vrijednost indeksa (npr. NDVI).
* Ploča za očitavanje prikazuje koordinate piksela, veličinu slike, format piksela, dubinu bita i tablicu kanala (Kan / Vrijed / %) s oznakama pojaseva i valnim duljinama; zeleni parovi po Bayerovoj matrici se prosječe; uzorci regije prikazuju "N px avg".

Stanje mjerača točke dostupno je samo tijekom sesije.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktivna automatska ekspozicija (vođena DLS-om)

Ovaj se odjeljak prikazuje samo kada je **povezan barem jedan DAQ senzor svjetla** — rješavaču je potreban kontinuirani nizvodni spektar za svoje funkcioniranje.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Kontrola | Opseg | Zadano | Što radi |
| --- | --- | --- | --- |
| **Omogući** | Kvačica | Uključeno (samostalni fotoaparati) | Rješavač zatvorenog oblika koristi DLS spektar i skalare kalibracijskog paketa kamere kako bi najsvjetliju traku doveo blizu zasićenja, a istovremeno najtamniju traku održao iznad donje granice SNR-a — jedno pisanje ekspozicije po rješavanju, bez petlje za stabilizaciju. Namijenjeno je timelapseu na solarni pogon gdje svaka snimka mora biti ispravno eksponirana. Pozadinski sustav se tiho vraća na reaktivno AE (automatsko eksponiranje) kad god je očitanje DLS-a zastarjelo/izostavljeno ili paket za kalibraciju nije učitan. |
| **Izravnavanje (α)** | 0,05–1,0, korak 0,05 | 0,3 | Izravnavanje uzastopnih prediktivnih rješenja (manje = glađe). |
| **Odbijanje scene**| gumb**Ponovno kalibriraj ρ** | — | Ponovno procjenjuje faktor odbijanja scene koji rješavač koristi. |

{% hint style="info" %}
**Povezivanje nizova prema zadanim postavkama isključuje prediktivnu AE** — za nizove, pametna AE aplikacijeChloros

i automatska ekspozicija na strani kamere upravljaju ekspozicijom (s zaštitom od zasitnosti) i prediktivnom AE za pojedinačnu scenu-procjenu reflektancije nije sigurno koristiti u mješovitim scenama. Možete je ponovno omogućiti za svaku kameru ovdje ako izričito želite radiometrijsku ekspoziciju vođenu DLS-om.
{% endhint %}

**Gornja granica ekspozicije vođena DAQ-om i AE vezan uz incidentnu svjetlost.** Neovisno o gornjoj potvrdnoj okviru, kada je DAQ senzor svjetla dodijeljen RGB kameri, Chloros izračunava — na temelju izmjerene apsolutne dolazne zračnosti — maksimalnu ekspoziciju×pojačanje pri kojoj površina s 100 % reflektancije ostaje ispod praga klipiranja i primjenjuje ga kao **gornju granicu**automatske ekspozicije. Od verzije 1.3.0 gornja granica se primjenjuje samo dok je kalibracijska ploča u vidnom polju te kamere (prikazana u njezinom kadru unutar posljednjih 10 s); kad ciljni predmet nije u vidokrugu, umjesto toga radi uobičajeno automatsko izlaganje kamere, sa svojom vlastitom zaštitom od preekspozicije najsvjetlijih tonova, tako da ostatak scene nije tamniji nego što treba biti. Dok je gornja granica aktivna, kamera je**fiksirana na incidentno mjerenje**: radi u otvorenoj petlji na ekspoziciji mjerenoj mjeračem svjetla s pojačanjem od 0 dB — ekspozicija prati izmjerenu svjetlost, a ne sadržaj scene. Budući da gornja granica može samo skratiti ekspoziciju, sama po sebi ne može uzrokovati izrezivanje. Od verzije 1.3.0 gornja granica izračunava se pomoću vlastite konstante ekspozicije kamere iz njezina kalibracijskog paketa (prosječna vrijednost flote ako paket ne sadrži vlastitu). Gornja granica se automatski isključuje — i normalna AE scene se nastavlja — kad god cilj kalibracije nije u vidokrugu, DAQ očitanje nedostaje, zastarjelo (>30 s), ili je tamno, ili ako ≥15 % kadra bude izrezano pri zadanom ekspozicijskom ograničenju (što znači da senzor i kamera vide različito osvjetljenje). Ne postoji GUI prekidač; ovo je standardno ponašanje kad god je DAQ vezan za kameru u boji RGB.

### Akvizicija i okidač

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Članovi niza dodatno prikazuju retke s informacijama samo za čitanje o **Ulogi**(Glavni u plavom / Rob u zelenom),**sinkronizacijskoj liniji**i**kolegama**.

| Kontrola | Opcije | Zadano | Bilješke |
| --- | --- | --- | --- |
| **Način okidača** | Isključeno / Uključeno | Uključeno | Onemogućeno za članove niza (niz upravlja okidanjem). |
| **Izvor okidača** | Softver / Linija0 (M8) / Linija1 / Linija2 | Linija0 | Skriveno dok je način okidanja isključen; onemogućeno za članove niza. Linija0 je M8 optički izolirani vanjski ulaz za okidanje. |
| **Brzina snimanja**| Automatski / Ručno + vrijednost | Automatski |**Automatski**: ograničenje brzine snimanja kamere je isključeno — brzinu određuje ekspozicija, a uređaj prikazuje stvarnu brzinu prijenosa uživo.**Ručno**: ograničavate fps klizačem (od 1 do maksimalne vrijednosti ograničene propusnošću), pri čemu se polazna vrijednost temelji na trenutnoj stvarnoj brzini. Članovi niza vide prikaz "N fps (uživo)" koji se ne može mijenjati i opciju "Postavi u postavkama niza". |

### Mreža / Transport

| Redak | Ponašanje |
| --- | --- |
| **Veličina paketa**| 1500 (Standardno) / 9000 (Jumbo) — zadano**Jumbo**. |
| **Propusnost** | Ograničenje propusnosti veze koje se ne može mijenjati, u MB/s. Backend ga uravnotežuje na svim priključenim kamerama prilikom svakog povezivanja i isključivanja. |
| **Obrada međuspremnika** | Načini rada obrade međuspremnika koji se ne mogu mijenjati. |

### Snimanje

Okno završava gumbom **"Otvori postavke snimanja…"** koji vodi do [okna Postavke snimanja](capture.md#the-capture-settings-pane) (onemogućen dok se ne otvori projekt — "Stvorite ili otvorite projekt za spremanje snimki"). Ako je kamera skrivena ili pauzirana, obavijest vas podsjeća da je prije snimanja otkrijete/nastavite.

## Panel postavki niza

Otvorite pomoću **zupčanika**u retku ARRAYA. Zaglavlje: naziv niza s olovkom za preimenovanje i**×** za zatvaranje. Odjeljci u nastavku označeni *samo za kombinirane* pojavljuju se samo za nizove povezane u kombiniranom načinu prikaza.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sinkronizacija

Redovi **Master**,**Slaves**i**Sync Line** su samo za čitanje.

### Senzor ambijentalnog svjetla

Prikazano za obje kombinirane i odvojene nizove:

* Spuštanje izbornika **Senzor svjetla** — povezuje jedan DAQ s cijelim nizom. Odabir se odmah zadržava, propagira se na padajući izbornik Svjetlosni senzor svake pojedine kamere (i dalje ga možete nadjačati za svaku kameru), i započinje prosljeđivanje spektra na niz.
* Linija **Status** uživo: Isključeno · "Čeka se prvi spektar…" · "Aktivno — sve kamere u nizu su korekcijom osvjetljenja" · "Nema novog spektra u posljednje 3 s — još uvijek se koristi posljednje očitanje (nema zastarjelog vremenskog ograničenja)…".
* Napomena u oknu: "Radiometrijska korekcija za cijeli niz. Postavke po kameri imaju prednost."

### Cilj kalibracije

Vlastita grupa odmah nakon senzora ambijentalnog svjetla, i za kombinirane i za odvojene nizove:

* **Kutija za potvrdu** **Omogući**— "Detektiraj cilj**MAPIR**ArUco i validiraj**NDVI**u odnosu na LUT refleksije panela"; pokreće tablice ciljnih preklopa i validacije kombiniranog pločica. Na**odvojenoj** mreži uključuje vlastitu [tablicu kalibracijskog cilja] svakog člana(#the-calibration-target-table) (po jedna po redu kamera u prikazu popisa).

### Snimanje — ujednačena postavka senzora

Prikazano za kombinirane i odvojene nizove (naslov grupe glasi "Snimanje (ujednačeno među kamerama)"). Ova se postavka primjenjuje jednako na sve članove (promjene za pojedinačne članove bi prekinule sinkronizaciju). Promjene se spremaju u čeku i primjenjuju zajedno pomoću **Primijeni**— osim prekidača**ROI uključi / ROI isključi**, koji se primjenjuje odmah.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Kontrola | Izbori / raspon | Što radi |
| --- | --- | --- |
| **Pikselni format** | 8-bit / 10-bit / 12-bit / 16-bit | Jedinstvena bitna dubina za sve članove; svaka kamera preuzima format te dubine za vlastiti senzor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardversko binanje — zadržava cijelo vidno polje dok povećava SNR i brzinu sličica. Promjenom se ROI polja resetiraju na novo cjelovito vidno polje. |
| **Preddefinirana rezolucija** | Puna / Polovična / Četvrtina | Relativno u odnosu na binning; puni ROI polja centriranim izrezom. |
| **ROI izrez (px)**| prekidač**ROI uključi / ROI isključi**, gumb**Odaberi**, polja Š / V / X / Y, mapa cijelog senzora | Izrez senzora — pogledajte [Uređivač ROI-ja](#the-roi-editor). Širina/visina se poravnavaju na višekratnike od 16 (minimalno 64); pomaci se prilagođavaju množenju 4. Napomena "max WxH" prikazuje gornju granicu, a**Resetiraj** vraća polja na cijelo vidno polje. Tijekom uređivanja, narančasti okvir za obrezivanje prati svaku promjenu na karti i na pločici niza. |
| **Stopa okidanja**| Prekidač Automatski / Ručni + fps 0,5–10, korak 0,5 |**Automatski**(zadano): backend izračunava brzinu okidanja na temelju rezolucije i propusnosti — ulaz je onemogućen i prikazuje izračunatu vrijednost.**Ručno**: zaključava vašu vrijednost pri primjeni. Brzina okidanja koja je zaključana i koju nova veličina sličice ne može podržati smanjit će se na najbržu brzinu koju ta veličina dopušta, a vratit će se kada veličina sličice to ponovno dopusti. |

Napomena u oknu: "Promjene formata/rezolucije nakratko ponovno pokreću sve kamere. Brzina okida primjenjuje se uživo." Gumbi **Primijeni / Otkaži** nalaze se na dnu okna.

#### Uređivač ROI-ja

Novo u verziji 1.3.0. Redak za obrezivanje ROI-ja i mapa ispod njega rade zajedno:

* **ROI uključi / ROI isključi**—**odmah**uključuje ili isključuje obrezivanje senzora (kamere se nakratko ponovno pokreću; pločica niza prikazuje "Primjenjuju se postavke…").****Isključeno**čita cijeli senzor i**zapamćuje vaš izrez**— polja ga i dalje prikazuju;**Uključeno** ga ponovno primjenjuje. Uključivanje ROI-ja bez postavljenog izreza primjenjuje središnji izrez polovične veličine ("Nije postavljen izrez, pa je primijenjen središnji izrez polovične veličine. Podesite ga pomoću polja.").
* **Odabir**— iscrtava izrez na karti: povucite preko njega ili kliknite na dva suprotna kuta (Esc otkazuje). Pravokutnik se pojavljuje u poljima Š/V/X/Y, prikazana na gornjim stupnjevima; ništa se ne primjenjuje dok ne pritisnete**Primijeni**, pa ga prvo možete precizno podesiti pomoću strelica na poljima. Jedan pravokutnik po pritisku na Odabir.
* **Karta**prikazuje izrez na cijelom senzoru, pa ga možete ponovno proširiti: "Cijeli senzor, uživo" dok kamere čitaju cijeli senzor, ili "Snimka cijelog kadra s {vremenskog}". Dokle god kamere čitaju samo iz obrezanog područja, mapa prikazuje "Još nema prikaza cijelog kadra: ovaj niz je uvijek streamao samo obrezane kadrove." s gumbom**Prikaži puni kadar**, koji isključuje ROI. Označivač kropanja `W×H @ X,Y` nalazi se izvan okvira (iznad njega ili ispod njega kada okvir dodiruje vrh), na karti i na pločici uživo.
* **Primijeni**potvrđuje polja: rezanje uključuje ROI; puni kadar (nakon**Resetiranja**) ga isključuje, a prethodno rezanje se i dalje pamti; ROI koji je isključen i čija polja i dalje prikazuju zapamćeno rezanje ostaje isključen kada primijenite drugu promjenu (format, biniranje, brzina).
* Na poravnatoj nizu, prozor za obrezivanje svakog člana postavlja se pomoću poravnanja, tako da svaka kamera snima isti dio scene. Niz koji je obrezan prije poravnanja prvo se poravnava na cijelom senzoru (uklanja se obrezivanje, rješava se poravnanje i vraća se obrezivanje s prozorima za svaku kameru).
* Nakon promjene izrezivanja, automatska ekspozicija se ponovno usmjerava za novo okvirenje. Unutar izrezanog područja, ekspozicija člana i dalje može trajati do razdoblja okidača.
* Stanje uključeno/isključeno ROI-ja i zapamćeni izrez spremaju se s projektom.

### Poravnanje (ko-registracija)

Prikazano za **obje kombinirane i odvojene niske** (verzija 1.2.0 prikazivala ga je samo za kombinirane niske); naslov grupe glasi "Poravnanje".

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Ostatak** (Residual): "RMS x.xx px" — zelena boja za manje od 1 px, žuta za 3 px ili manje, crvena inače ili ako je neka kamera neuspjela; "no profile" prije prvog rješavanja.
* Redak sa sažetkom: "✓ Sve kamere poravnate (N)" / "⚠ p/N kamera poravnato — <serial (filter)="">neuspjelo" / "Orezivanje aktivno — ponovno kalibrirajte za poravnanje (koristi cijeli senzor)" / "Čeka se stabilizacija ekspozicije…" / "Primjenjuju se postavke…" (kamere se ponovno pokreću) / "Niz je prestao isporučivati sličice — ponovno povežite niz".
* Tablica po kamerama: kamera (zadnje 4 znamenke serijskog broja + filtar), pogreška reprojekcije u px s brojem podudaranja ("ref" za glavnu kameru) i ocjena normalizirane unakrsne korelacije preklapanja u odnosu na donju granicu od 0,35 za prolaz.
* **Ponovno kalibriranje poravnanja** gumb (prije prvog profila piše "Kalibriraj poravnanje") — ponovno pokreće su-registraciju na novim kadrovima.
* **"Automatska ekspozicija kamera za poravnanje"** potvrdni okvir (zadano označen) — privremeno pojačava slabu ili ravnu kameru (prvo ekspozicija, zatim pojačanje) kako bi imala teksturu za usporedbu, a zatim vraća AE.

Chloros automatski poravnava svaki niz nakon povezivanja, u oba načina prikaza, čim se automatska ekspozicija članova stabilizira (nastavlja pokušavati do 5 minuta); kombinirani pregled također se poravnava pri otvaranju. Odvojeni niz nema kompozitnu pločicu, stoga ovaj odjeljak provjerava stanje poravnanja i pokreće ponovnu kalibraciju. Ponovno kalibrirajte ako se promijenio fokus ili dubina scene. Poravnanje je **namjenski samo za sesiju** — nikada se ne sprema u profil jer ovisi o trenutnoj udaljenosti scene. Snimke se i dalje mogu izvesti s pikselskom registracijom (vidi [Usklađeni izvoz](capture.md#per-array-controls)).

### Pametna vinjeta

* **Kutija za potvrdu Omogući korekciju**— primjenjuje procjenu vinjete za svaku kameru na radiometrijski lanac (uživo**i** pri izvozu).
* **Kalibriraj iz trenutnog pogleda**— prvo usmjeri niz prema jednolikoj meti (ravnoj ploči, zidu ili nebu); svaka se kamera pojedinačno poravnava i status prikazuje dobit poravnatosti "n/N kamera · −x.x %".**Očisti** uklanja procjenu.
* Status: "Kalibrirano", "Nekalibrirano" ili "Djelomično kalibrirano — ponovno pokrenite" kada se procjene nekih članova još uvijek primjenjuju na njihovo trenutno okviriranje (na primjer nakon što je ponovna kalibracija pomaknula prozor obrezivanja nekog člana) — ponovno pokrenite Kalibraciju.
* Fino podešavanje po kameri pomoću klizača **Vignette** u [Pregledu uživo](#live-preview).

### Prikaz uživo

* **Histogram**potvrdni okvir + klizač širine (0,10–0,90 okvira) — traka histograma na prijenosu na svakoj pločici svakog člana odjednom (uključene su i pločice kanala), po kameri [Histogramski sloj](#display-overlays-drawn-over-the-live-feed) postavljen za cijelu niz. Prikazuje se kao uključeno kada ga svaki član već ima. U**odvojenoj** nizu ovo je jedini redak u grupi.
* *Samo za kombinirano:* **Indeks**: potvrdni okvir + zupčanik — otvara zajednički [Kalkulator indeksa](#index-calculator-pane) s pojasevima iscrtanima s**svih** kamera članova. Redak za pretpregled izraza ispod prikazuje trenutni izraz ("Nije postavljen izraz — otvorite kalkulator za izradu novog"), koji se osvježava svake sekunde.
* *Samo za kombinirano:* padajući izbornik **Rješivost renderiranja**(isti predlošci kao i po kameri, zadano 720p): visina prijenosa uživo**i** veličina spremljenog kompozita za izvoz. Napomena u oknu: "Pregled + veličina spremljenog kompozita. Slike po kameri uvijek se izvoze u punoj rezoluciji."

### Prikaz slojeva *(samo za kombinirane)**   **Omogući** potvrdni okvir (zadano isključeno — glavna kamera se prikazuje izravno; uključeno = slojeviti kompozit).
*   **padajući izbornici**Prednji plan**/**Pozadina**: svaka članica kamere (po nazivu) ili**Indeks**. Kada je za Prednji plan odabran Indeks, pikseli izvan LUT minimalne/maksimalne vrijednosti prikazuju sloj Pozadine.

### Podijeljeni prikaz *(samo za kombinirani način)*

**"Prikaži članice kamere"**— gumb**Podijeli / Sakrij pomoćne kamere** koji dodaje vlastiti prijenos uživo svake pomoćne kamere kao zasebne pločice u mreži pored kompozita. Pločice čitaju postojeći spremnik sličica niza (bez dodatne veze s kamerom). Samo prikaz u mreži; spremljeno po nizu s projektom.

### Mogućnosti

Panel za čitanje samo u čitanju koji se osvježava svakih 5 s:

* **Oznaka sloja**: "Istovremeno snimanje" (zelena) · "Istovremeno snimanje (FTD-razmaknuto emitiranje)" (zelena) · "Razmaknuto snimanje (100 ms odmak)" (narančasta) · "Konfiguracija je prevelika" (crvena).
* **Zdravstveno stanje sličice**: "x.xx % nepotpuno" — zelena ispod 1 %, narančasta ispod 5 %, crvena na 5 % ili više.
* **Stavka veze**: "NIC {mbps} Mbps - održivo {MB/s} MB/s".

Ovo je proračun propusnosti niza u stvarnom vremenu. Za temeljni model fps-a i mreže — i što promijeniti kada se razina prebacuje na narančastu ili crvenu — pogledajte [Multi-Camera Arrays](arrays.md) i [CLI Reference](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Panel kalkulatora indeksa

Treća stranica bočne trake, zajednička za polugu indeksa po kameri i polugu indeksa kombiniranog niza (po jedna u isto vrijeme — naslov glasi "Kalkulator indeksa — <camera name="">" ili "Kalkulator indeksa — <array name="">"). Prima popis pojaseva (filtera kamere-prirodnih pojaseva ili svih pojaseva preko članova niza), trenutni izraz i konfiguraciju LUT-a (uključeno/isključeno, razina — zadano 7 stopa, min — zadano 0,2, max — zadano 1), plus histogram indeksa uživo. **Primijeni** potvrđuje izraz; Promjene LUT-a primjenjuju se uživo na pregled.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Postavke po kameri naspram postavki upravljanih nizom

Kratki pregled gdje se što nalazi kada je kamera član niza:

| Upravljano nizom (samo za čitanje u prozoru kamere) | Još uvijek po kameri unutar niza |
| --- | --- |
| Format piksela, rezolucija, biniranje | Automatsko ekspozicije (ekspozicija, pojačanje, cilj, izravnavanje, ROI) |
| Način/izvor okidača, brzina sličica | Uklanjanje šuma, oštrina, vigneta |
| | Orijentacija (ogledalo/rotacija), prikazni slojevi (postavke niza → Uvidi u stvarnom vremenu postavlja histogram na sve članove odjednom), mjerač svjetline |
| | Indeks (nizovi s odvojenim prikazom), povezivanje sa senzorom svjetlosti |

Ostalo međufunkcionalno ponašanje:

* **Kombinirani naspram odvojenog prikaza** odabire se pri povezivanju niza: kombinirani = jedna poravnata kompozitna pločica (slike članova samo putem podijeljenog prikaza); odvojeni = svaki član prikazuje vlastitu sinkroniziranu pločicu. Kamera nikada ne prikazuje istovremeno samostalni prikaz i pločicu niza.
* **Automatsko ponovno povezivanje**: otvaranje spremljenog projekta vraća njegove kamere i nizove i ponovno primjenjuje svaku spremljenu postavku na pozadinski sustav prije nego što se streamovi nastave.
* **Kontrola snimanja**: skrivene ili pauzirane kamere isključene su iz opcije Snimi sve; niz je u potpunosti blokiran samo kada su SVI članovi skriveni/pauzirani. Vidi [Postavke i načini snimanja](capture.md).

## Kako se postavke spremaju

Stanje kartice kamere sprema se **s projektom**, a ne u pregledniku:

* Svaka reaktivna promjena snima stanje kamera i nizova u `cameras.json` projekta (s odgađanjem od 500 ms). To obuhvaća nazive i boje kamera, postavke ekspozicije/pojačanja/AE, format piksela/rezoluciju/biniranje, brzinu okidača, postavke pretpregleda (rezolucija iscrtavanja, uklanjanje šuma, oštrina, vinjetiranje, profil boja, istaknuta područja, zasićenje/kontrast), orijentaciju, nadsvjetljenja, razdvajanje kanala, konfiguraciju indeksa, postavke prediktivnog AE-a, AE ROI, nazive nizova, način prikaza, postavke snimanja nizova (uključujući izrez ROI-ja, je li ROI uključen i zapamćeni izrez dok je isključen), Wire Budget niza i blok mreže (zum napajanja, način prikaza, zaključavanje mreže, ručni redoslijed pločica, skrivene kamere, zatvorene pločice, aktivna kamera).
* Poveznice senzora svjetla spremaju se u projektnu datoteku `sensors.json`.
* Ponovno otvaranje projekta ponovno povezuje hardver i ponovno primjenjuje sve postavke.
* **Nema otvorenog projekta = samo sesija**: bez projekta, ništa se ne sprema pri zatvaranju programa Chloros.
* Samo za sesiju bez obzira na projekt: stanje pauze, uzorci mjerača točke, potvrdni okvir Cilj kalibracije za svaku kameru (uvijek je isključen), i profil poravnanja niza (namjerno se ponovno izračunava po sesiji).
* Jedan izuzetak: odabrani **Postavke snimanja** i načini snimanja pohranjuju se za svaki projekt zasebno u lokalnom pohranu aplikacije, a ne u `cameras.json` — pogledajte [Postavke i načini snimanja](capture.md).</array></camera></serial>
