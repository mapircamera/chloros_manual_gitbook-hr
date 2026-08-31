# Postavke kamere

Jezgra **Kamere** je površina za upravljanje uživo aplikacijeChloros

za LATTICE kamere: sadrži glavno područje za prikaz koje prikazuje svaku povezanu kameru kao pločicu uživo i bočnu traku koja se pomiče između triju stranica — **popisa kamera**,**panela s postavkama**(po kameri, nizu ili postavke snimanja — po jedna odjednom), i**Kalkulator indeksa**. Ova stranica opisuje svaku kontrolu u popisu kamera, u odjeljku s postavkama po kameri i u odjeljku s postavkama niza. Načini snimanja, odabir vrste izvoza i tijek &quot;Snimanje svega&quot; nalaze se na povezanoj stranici [Postavke i načini snimanja](capture.md).

Jezgra Kamera pojavljuje se u bočnoj traci kada je pozadinski sustavChloros

spreman. Sve kontrole u nastavku komuniciraju s lokalnim pozadinskim sustavom putem `127.0.0.1:5000`; promjene se odmah primjenjuju na kameru uživo, osim ako nije drugačije navedeno.

## Vrste kamera koje se koriste na ovoj stranici

Kontrole se prikazuju ili skrivaju ovisno o vrsti odabrane kamere. Priručnik koristi ove pojmove:

| Pojam | Značenje | Filtri kanala |
| --- | --- | --- |
| **RGB

kamera** | LATTICE M3C s FRGB filtrom (model sadrži `-FRGB`) |Red

/Green

/Blue

|
| **Bayer multispektralna** | LATTICE M3C s FRGN, FOCN ili FNGB filtrom | FRGN:Red

/Green

/NIR

· FOCN:Orange

/Cyan

/NIR

· FNGB:NIR

/Green

/Blue

|
| **Mono (M3M)** | LATTICE M3M — jedan uskopojasni filtar, jedna kalibrirana pojas | Jedna pojas |
| **Član niza** | Kamera povezana kao dio sinkroniziranog niza (kombinirani ili zasebni prikaz) | Prema svom filtru |

KamereRGB

dobivaju fotometrijsku obradu (bilancu bijele, profili boja, gamma); multispektralne i mono kamere dobivaju radiometrijski lanac i preskaču fotometrijske kontrole. Članovi niza prosljeđuju postavke na razini toka (format piksela, rezolucija, biniranje, okidač, brzina sličica) nizu — ti se redovi u prozoru za svaku kameru postaju samo za čitanje i premještaju se u prozor s postavkama niza.

## Glavno područje za dodavanje



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Kada nijedna kamera nije priključena, područje za dodavanje prikazuje uvodni zaslon **&quot;Priključite kameru za početak&quot;**s dva gumba:**Priključi kameru**(zeleni, otvara dijalog za priključivanje jedne kamere) i**Priključi niz** (plavi, otvara dijalog za priključivanje niza). Sami dijalozi za priključivanje dokumentirani su u [Povezivanje kamera](connecting.md); koncepti niza (sinkronizacija, razine, propusnost) u [Nizovima više kamera](arrays.md). Kada otvorite spremljeni projekt koji sadrži kamere, na uvodnom zaslonu se umjesto toga prikazuje indikator napretka s porukom &quot;Ponovno otvaranje N spremljenih kamera…&quot;, dokChloros

vraća streamove iz posljednje sesije.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Gornja traka

| Kontrola | Što radi |
| --- | --- |
| **Prebacivanje načina prikaza**| Prebacuje između**mrežnog prikaza**(sve pločice kao ćelije) i**popisnog prikaza** (nizovi pune širine na vrhu, JEDNA aktivna kamera ispod). Savjetnice: &quot;Prebaci na mrežni prikaz&quot; / &quot;Prebaci na popisni prikaz&quot;. |
| **Zaključavanje mreže**(zaključani simbol) | Zadano**zaključano** — pločice su fiksirane na mjestu. Odključajte za povlačenje i preslagivanje pločica u bilo koji utor (praznine se čuvaju). Mreža se automatski ponovno zaključava svaki put kad se poveže nova kamera. Pomoćni tekstovi: &quot;Odključaj mrežu (omogući povlačenje pločica)&quot; / &quot;Zaključaj mrežu (zamrznuti pločice na mjestu)&quot;. |
| Klizač **Zoom prijenosa** | Veličina pločice, od 60 px do pune širine spremnika. Stanice zadržavaju omjer stranica 4:3. Pri širini ćelije manjoj od 200 px naziv i prikazi fps-a se skrivaju kako bi pločica ostala pregledna. |

### Pločice prijenosa

Svaka kamera iscrtava složenu živu pločicu; kamera može dodatno prikazati tri pločice u sivim nijansama s **razdvajanjem po kanalima** (per-channel split) (vidi [Razdvajanje kanala](#display-overlays-drawn-over-the-live-feed)), a nizovi iscrtavaju kombiniranu pločicu. Aktivna pločica ima prsten odabira u boji kamere (ili niza).

Prilikom prelaska mišem preko pločice prikazuje se gumb za zatvaranje **X**:

* Zatvaranje **kompozitne** pločice dok njezine razdjele po kanalima ostaju vidljive samo skriva kompozitnu pločicu.
* Zatvaranje **zadnje vidljive pločice samostalne kamere** prekida vezu s tom kamerom.
* Podijeljene pločice članova kombiniranog niza nikada ne isključuju kameru — one se samo sakrivaju.

Kada je mreža otključana, povucite bilo koju pločicu u bilo koji utor; raspored se sprema s projektom.

## Bočna traka — popis

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamera Prva stranica bočne trake prikazuje svaku povezanu kameru i niz:

* **Poveži kameru**(zelena) /**Poveži niz** (plava, prikazuje &quot;Otkrivanje...&quot; tijekom skeniranja). Oba su onemogućena dok je otvoren dijalog za povezivanje.
* **Snimanje svega** (crveno) — snima svaku navedenu kameru s vrstama izvoza odabranim u postavkama snimanja. Zahtijeva otvoren projekt. Potpuno dokumentirano u [Postavke i načini snimanja](capture.md).
* **Zupčanik Postavki snimanja** (pored Snimi sve) — otvara [panel Postavke snimanja](capture.md#the-capture-settings-pane). Onemogućeno je bez projekta ili tijekom snimanja.

### Redovi kamera

Svaki red kamera prikazuje obojenu granicu (prilagođena boja kamere), oznaku &quot;CAM&quot; — s plavim **M**(master) ili zeleno**S** (slave) slovo za ulogu članova niza — i prikazano ime. Zadano ime je `LATTICE-MODEL (serial)`; preimenujte ga u prozoru postavki za svaku kameru. Gumbi u retku:

| Gumb | Učinak |
| --- | --- |
| **Oko**| Prebacivanje vidljivosti. Skrivene kamere izlaze iz mreže i**isključene su iz snimanja svega**. |
| **Zupčanik** | Otvori okno s postavkama za svaku kameru (sljedeći odjeljak). |
| **Pauza / Reprodukcija**| Zamrzne uživo pregled**samo na strani prikaza** — pozadinski snimanje se nastavlja. Pauzirane kamere ne mogu snimati. |
| **X** | Odspojite. Sučelje se odmah ažurira (optimistično); sam postupak odspajanja u pozadini može potrajati 10–30 s. |

### Redovi niza

Redak niza prikazuje bedž &quot;ARRAY&quot; u boji niza, naziv niza (može se preimenovati u postavkama niza) i ****gumb DAQ · uključi/isključi**—**uključeno** kada je senzor svjetla na razini niza postavljen *ILI* bilo koji član ima senzor po kameri; njegov alatni savjet navodi točno koji senzor što opskrbljuje. Članice kamere su ispod navedene uvučeno sa svojim redovima. Gumbi u redu niza: **oko**(skriva/prikazuje SVE članice zajedno),**zupčanik**(panel postavki niza),**X**(odspojite cijeli niz).

Status senzora svjetla (DLS) koji se koristi u redovima niza i u panelu postavki niza ima četiri stanja:**isključeno**,**čeka**(još nema spektra),**aktivno**(spektrum je stigao u posljednje 3 s), i**staro** — nema novog spektra u 3 s, ali se posljednje očitanje *još uvijek koristi* (DAQ očitanja nikada ne istječu na putanji snimanja).

Možete povlačiti samostalne kamere i cijele grupe niza jedne pokraj drugih u bočnoj traci kako biste promijenili redoslijed popisa; članovi niza se ne mogu neovisno povlačiti.

## Panel postavki po kameri

Otvorite pomoću **zupčanika** u retku kamere. Panel se klizi preko popisa kamera.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Naslov**:**uzorak boje**kamere (kliknite za otvaranje ugrađenog odabira boje — postavlja boju okvira bočne trake i prstena za odabir pločice),**naziv**s gumbom olovke**Promijeni naziv**(pohrana praznog naziva vraća se na zadano `MODEL (serial)`), i**×** za zatvaranje.

### Živi histogram

Na vrhu prozora nalazi se živi histogram svjetline izračunat iz pretpregleda **JPEG** s frekvencijom od ~8 Hz. Prosjek je ponderiran prema Bayerovoj maski — (R+2G+B)/4 — kako bi odgovarao vlastitom AE mjerenju kamere.

* **prikazana linijaOrange**= cilj za AE. **Povucite je vodoravno za promjenu cilja** — jedan se nalog šalje pri puštanju, a povlačenje prebacuje način rada AE cilja na ručni.
* **puna linijaGreen** = stvarna srednja lumina (ono što AE trenutno isporučuje).
* **gumbRGB*** (gornji desni kut): omogućuje/onemogućuje histograme preklapanja po pojasu obojene filtrom kamere (npr. na FRGN: sivaNIR

, zelena, crvena). Na mono (M3M) kamerama gumb je označen kao &quot;MONO&quot; i onemogućen — mono uvijek prikazuje histogram lumine po jednom pojasu.
* Oznake X-osovine prate dubinu bita senzora trenutnog formata piksela: 0..255, 0..1023, 0..4095 ili 0..65535.

### Redovi

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s informacijama o kameri

| Redak | Ponašanje |
| --- | --- |
| **Model** | Samo za čitanje (npr. `LATT-M3C-L87-FRGN`). |
| **Radiometrijska kalibracija** |Green**&quot;oznaka &quot;Active&quot; s natpisom koji prikazuje razinu kalibracije, hash, datum kalibracije i popis pojaseva, učitani iz paketa za kalibraciju kamere (vidi [Factory Radiometric Calibration](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)). **Skriveno za kamereRGB** — one imaju fotometrijsku kalibraciju bijele ravnoteže, a ne radijansu po pojasu. |
| **Izvješće o kalibraciji**| Gumb**Preuzmi** — otvara PDF certifikata o NIST kalibraciji kamere po serijskom broju u pregledniku operativnog sustava. Ako certifikat još nije pohranjen u predmemoriju,Chloros

umjesto toga prikazuje obavijest. |
| **Serijski broj** | Samo za čitanje. |
| **Firmware**| Prikazuje trenutačnu verziju, a zatim dostupnu verziju za ovaj model (pohranjeno po modelu — niz N kamera provjerava poslužitelj jednom). Stanja: &quot;Provjera…&quot; → gumb**&quot;Ažuriraj na X&quot;**→ &quot;Programiranje…&quot; → &quot;Ažurirano s A na B&quot; / &quot;Neuspjelo: …&quot; / &quot;Preskočeno: …&quot; / zelena**&quot;Ažurirano&quot;**. Pop-up obavijest gumba za ažuriranje: &quot;Vraćanje na tvorničke postavke + flashanje + reprogramiranje UserSet1. ~2–3 minute; ne isključujte.&quot; |
| **IP** | Samo za čitanje. |
| **Temperatura** | Samo za čitanje, osvježava se svakih 3 s. Postaje narančasta na ≥65 °C i crvena s ⚠ na ≥75 °C. |
| **Kutija za potvrdu *Cilj kalibracije* | Omogućuje ArUco detekciju cilja reflektancije s tablicom za provjeru *NDVI

* po panelu ispod prijenosa uživo (pregled popisa). Samo za sesiju — uvijek je isključeno. |
| padajući izbornik **Senzor svjetla** | Povezuje DAQ senzor svjetla (DAQ-E/M/U, s popisa na kartici Senzori svjetla) s ovom kamerom za korekciju osvjetljenja odozgo (DLS) i prediktivnu automatsku ekspoziciju. &quot;None&quot; otkazuje povezivanje. Ako nijedan senzor nije priključen, padajući izbornik prikazuje &quot;(nema priključenih senzora — otvorite DAQ karticu)&quot;. Povezivanje se sprema s projektom. |

### Ekspozicija i pojačanje

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Svi numerički unosi ovdje koriste rotacijske kontrole s držanjem za ubrzanje: dodir = ±1, držanje &gt;1,5 s = ±10, držanje &gt;3 s = ±100. Vrijednost se šalje kameri kada otpustite prst.

| Kontrola | Raspon / mogućnosti | Zadano | Na što se primjenjuje | Što radi |
| --- | --- | --- | --- | --- |
| **Ekspozicija (µs)**| Min/max u stvarnom vremenu kamere | Automatski | Sve | Vrijeme ekspozicije u mikrosekundama, s prekidačem**Automatski/Ručno**. Automatski = kontinuirana automatska ekspozicija s kamere. |
| **Pojačanje (dB)**| Minimalna/maksimalna vrijednost kamere uživo (npr. do 48 dB) | Ručno (isključeno) | Sve | Analogno/digitalno pojačanje s vlastitim prekidačem**Automatsko/Ručno**. |
| **AE ciljana svjetlina**| 0–255 | 80, način**Auto**| Sve (može se uređivati kada je AE ili automatsko pojačanje uključeno) | Ciljana svjetlina za AE. U načinu**Auto**(zadano) pozadinski kontroler temeljen na histogramu sam odabire ciljanu vrijednost, održavajući ekspoziciju na 60–75 % maksimalne vrijednosti senzora. Upisivanje vrijednosti ili povlačenje narančaste linije histograma prebacuje ga u**ručni** način. |
| **Izravnavanje AE-a** | 0,5–40, korak 0,1 | 8,0 | Svi | Prigušivanje AE-a. Napomena alata: &quot;Niže = AE brže reagira (može pulsirati pri visokim fps-ima). Više = glađe / sporije.&quot; Vrijednosti znatno ispod zadane mogu uzrokovati pulsiranje AE-a i destabilizirati streaming pri visokim brzinama sličica; 8.0 je stabilna zadana vrijednost. |
| **AE regija od interesa**| Kvadrat za potvrdu + gumb**Aim**| Isključeno | Svi | Kada je uključeno, AE mjeri samo isprekidanom zelenom regijom umjesto cijelog kadra.****Ciljanje** omogućuje postavljanje klikom na prijenosu uživo: klikom se centrira regija na 30 % okvira; klik-i-povlačenjem iscrtava se prilagođeni pravokutnik (minimum 5 % × 5 %). Ciljanje se automatski isključuje nakon jednog postavljanja. Područje se mapira natrag na nativne koordinate kamere prema bilo kojoj rotaciji/ogledalu koje ste postavili i spremuje se s projektom. |
| **Brzina podešavanja AE-a** | 0,1–5, korak 0,1 | 1,0 | Samo za članove niza | Koliko brzo automatski AE cilj prati promjene svjetline scene; 1,0× ponovno provjerava svakih 2,5 s. |
| **Zaštita od preosvjetljenja** | Strogo (1 %) / Normalno (5 %) / Popustljivo (15 %) | Strogo | Kamere koje omogućuju podešavanje | Koliki dio kadra može biti odsječen na bijelo prije nego što AE zamrači sliku. |

{% hint style="info" %}
**Zahtjev za osvjetljenje za Bayerove multispektralne kamere (RGN

/OCN

/NGB

):** scena mora imati dovoljno svjetla u sva tri kanala ili kalibracija neće ispravno raditi — jedna ekspozicija senzora pokriva sva tri spektra. Koristite DAQ senzor svjetla za mjerenje svjetla ili pređite na potpuno mono (M3M) tako da svaki pojas dobije vlastitu ekspoziciju. Ako snimanje prekrši ovo pravilo,Chloros

to otkriva i upozorava vas (obavijest unmix-clamp).
{% endhint %}

### Format piksela i rezolucija

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Članovi niza** prikazuju retke &quot;Trenutno&quot; (format + ŠxV) i &quot;Biniranje&quot; samo za čitanje s napomenom &quot;Postavi u postavkama niza&quot; — ponovno pokretanje prijenosa na jednom članu prekinulo bi sinkronizaciju, stoga se njima upravlja u [oknu postavki niza](#array-settings-pane).**Samostalne kamere** dobivaju:

| Kontrola | Opcije | Što radi |
| --- | --- | --- |
| **Pikselni format** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format senzorskih piksela (bitna dubina). |
| **Razlučivost** | Puna / Polovna / Četvrtina | Relativno prema trenutnom binningu: Puna = 2048/N × 1536/N za N×N binning. |
| **Binning** | 1x1 (nema) / 2x2 / 4x4 | Hardverski N×N binning — veće vrijednosti smanjuju rezoluciju, ali povećavaju SNR i brzinu snímanja. Promjena pokreće stream iznova i resetira svako ROI na novo puno vidno polje. |
| **Trenutno** | samo za čitanje | Stvarna WxH i (x, y) pomak koji je na snazi. |

### Prikaz uživo

Sve u ovom odjeljku odnosi se **samo na prikaz**— mijenja ono što vidite u prijenosu uživo, dok snimke ostaju linearne i nepromijenjene — uz jednu iznimku:**Vignette** je radiometrijska i utječe i na izvoze (objašnjeno u nastavku).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Kontrola | Opsež / opcije | Zadano | Primjenjuje se na | Što radi |
| --- | --- | --- | --- | --- |
| **Rješivost iscrtavanja** | 360p (najbrže) / 480p / 720p / 1080p / Izvorna rezolucija senzora (najsporije) | 720p | Svi | Visina na kojoj pozadinski sustav pokreće lanac radiometrijskog pretpregleda. Niža postavka smanjuje brzinu sličica bez promjene vidnog polja. |
| **Indeks**| Kvadrat za potvrdu + zupčanik | Isključeno | Samo za Bayer multispektralni senzor,**ne** za članove kombiniranog niza | Prikaz indeksa žive vegetacije. Zupčanik otvara zajednički [Kalkulator indeksa](#index-calculator-pane) unaprijed natovaren filtrima prirodnih pojaseva kamere (npr. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Prilagođeni izraz plus LUT (uključi/isključi, razina zadano 3, min zadano 0,2, max zadano 1) izračunava se na svakom sličicu pretpregleda. Članovi kombiniranog niza skrivaju ovaj redak — niz posjeduje jedan zajednički indeks. |
| **Bilans bijele** | Isključeno / Jednokratno / Kontinuirano + gumb za ponovno snimanje | Kontinuirano | Samo zaRGB

| Kontinuirani balans bijele. Gumb za osvježavanje ponovno snima balans bijele iz trenutnog DLS spektra (onemogućen kada je način rada Isključeno). |
| **Gamma** | Uključi / Isključi | Uključeno | Samo zaRGB

| Prikaz gammatona (γ = 2.2 LUT) na uživo pregledu. Pohranjene snimke ostaju linearne. |
| **Uklanjanje šuma** | Potvrdni okvir + jačina 0–100 | Isključeno / 50 | Sve (po kameri, čak i unutar nizova) | Bilateralni filter na pregledu uživo. Viša vrijednost = glađe, ali mekše detalje. |
| **Oštrina** | Potvrdni okvir + snaga 0–100 | Isključeno / 30 | Sve | Maska neoštrine na pregledu uživo, primjenjuje se na kraju. Može pojačati šum. Samo za pretpregled. |
| **Vinjeta**| Potvrdni okvir + jačina 0–100 | Isključeno / 0 | Sve | Ručno uklanjanje vinjete (osvjetljavanje kutova), primijenjeno iznad procjene pametne vinjete niza.**Radiometrijski — utječe na prikaz uživo I izvoz**, za razliku od Uklanjanja šuma/Oštrine. |
| **Profil boje** | Sirov / Linearan / Prirodan / Poboljšan / Prilagođena temperatura | Prirodan | Samo zaRGB

| Vidi dolje. |
| **Boja temperature**| 2000–10000 K, korak 100 | 5500 K | Samo za profil**RGB

** | Fiksira bilancu bijele na fiksnu koreliranu temperaturu boje (DLS ulaz se zanemaruje). Zadnja odabrana Kelvin vrijednost pamti se pri prebacivanju profila. |
| **Zasićenje** | 0–200 (100 = neutralno) | 100 | Samo zaRGB

| Zasićenje HSV-a u pregledu uživo. |
| **Kontrast** | 0–200 (100 = neutralno) | 100 | Samo zaRGB

| Linearni kontrast oko sive sredine na pregledu uživo. |
| **Ogledalo H / Ogledalo V** | Kvačice | Isključeno | Svi | Okrenite pregled vodoravno / okomito. |
| **Rotacija**| 0° / 90° / 180° / 270° | 0° | Sve | Rotira pretpregled. Orijentacija se primjenjuje na kraju lanca pretpregleda u pozadini —**pohranjene snimke ostaju u izvornoj orijentaciji kamere**, a prikazi kompozita nizova je zanemaruju. |**Semantika profila boja** (kamereRGB

):

* **Raw** — potpuno zaobiđite lanac obrade.
* **Linearno** — signal tamne kompenzacije + ravna ravnina + balans bijele; bez matrice boja, bez gamma korekcije.
* **Prirodno** *(zadano)* — linearno plus izmjerena matrica korekcije boja i tonalna krivulja prilagođena sceni.
* **Poboljšano* — Natural plus vibrancija i lokalni kontrast (CLAHE). Dodatni trošak odnosi se **samo na pregled uživo** — spremljene fotografije uvijek dobivaju potpunu obradu bez obzira na profil.
* **Prilagođena temperatura** — Natural s balansom bijele pričvršćenim na odabranu vrijednost Kelvina.

{% hint style="warning" %}
Za načine rada Prirodno, Poboljšano i Prilagođena temperatura, ploča prikazuje napomenu o tonu: sličice se pojašnjavaju prema vlastitoj sceni, stoga spremljene *prikazne* slike nisu usporedive iz kadra u kadar. **Izvozite radijancu ili reflektancu za mjerenja.**
{% endhint %}

### Preklapajući slojevi zaslona (nacrtani preko prijenosa uživo)

Ovo je samo za korisnički sučelje — iscrtava se preko videa, a nikada ne dodiruje stream niti snimke.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Preklapajući sloj | Kontrole | Zadano | Što radi |
| --- | --- | --- | --- |
| **Zebra** | Kvačica + prag 200–255 | Isključeno / 250 | Magentne dijagonalne pruge na obrezanim pikselima. |
| **Križanac** | Kvačica | Isključeno | Označavanje središta kadra. |
| **Rešetka** | Isključeno / 3 × 3 / 9 × 9 | Isključeno | Rešetka za kompoziciju. |
| **Histogram** | Potvrdni okvir + širina 0,10–0,90 okvira | Isključeno / 0,25 | Traka histograma na prijenosu. |
| ****Focus Peak** | Potvrdni okvir + prag 20–200 + uzorak boje | Isključeno / 80 / `#ff5722` | Sobelovo isticanje rubova za fokusiranje. |
| **Raspodjela kanala** | &quot;Prikaži razdvajanja (Red

/Green

/NIR

)&quot; / gumb &quot;Sakrij razdvajanja&quot; | Skriveno | Dodaje tri neovisne pločice u sivim tonovima za svaki kanal uz kompozit (naziv gumba prati filtre kamere). Svaki odvojen pločić može se premještati i dijeli boju okvira kamere. Nije dostupno na mono kamerama. Sprema se s projektom. |

### Spot mjerač

* **Klik za uzorkovanje**potvrdni okvir: kliknite na uživo prikaz da biste uzorkovali jedan piksel (ciljač ga označava), ili povucite regiju za prosjek piksela.**Očisti**briše uzorak i ciljač. Međusobno isključivo s načinom rada AE-ROI**Ciljanje**.
* Izbornik **Prikaži**:**Raw (bitna dubina)**— nativni digitalni brojevi u bitnoj dubini senzora (npr. 12-bit → 0..4095) — ili**Prikaz (8-bit)** (zadano). Kada je aktivan indeks uživo, prikaz umjesto toga prikazuje izračunatu vrijednost indeksa (npr.NDVI

).
* Panel za očitavanje navodi koordinate piksela, veličinu slike, format piksela, dubinu bita i tablicu kanala (Kan / Vrijednost / %) s oznakama pojaseva i valnim duljinama; zeleni parovi po Bayerovoj matrici prosječno se računaju; uzorci iz regije prikazuju &quot;N px avg&quot;.

Stanje mjerača točke dostupno je samo tijekom sesije.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktivna automatska ekspozicija (na temelju DLS-a)

Ovaj se odjeljak prikazuje samo kada je **povezan barem jedan DAQ senzor svjetla** — rješavaču je potreban kontinuirani nizvodni spektar za rad.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Kontrola | Opseg | Zadano | Što radi |
| --- | --- | --- | --- |
| **Omogući** | Kvačica | Uključeno (na samostalnim kamerama) | Rješavač zatvorenog oblika koristi DLS spektar i kalibraciju kamere-skalare paketa kako bi najsvjetlija traka bila blizu zasićenja, dok se najtamnija traka održava iznad donje granice SNR-a — jedno pisanje ekspozicije po rješavanju, bez petlje za stabilizaciju. Namijenjeno timelapse snimanju na solarni pogon gdje svaki kadar mora biti ispravno eksponiran. Pozadinski sustav se tiho vraća na reaktivno AE (Auto-ekspoziciju) kad god je DLS očitanje zastarjelo/izostavljeno ili paket kalibracije n&#x27;t učitana. |
| **Izravnavanje (α)** | 0,05–1,0, korak 0,05 | 0,3 | Izravnavanje uzastopnih prediktivnih rješenja (manje = glađe). |
| **Odbijanje scene**|**Gumb **Ponovno kalibriraj ρ** | — | Ponovno procjenjuje faktor odbijanja scene koji rješavač koristi. |

{% hint style="info" %}
**Spajanje nizova prema zadanim postavkama isključuje prediktivni AE** — za nizove, pametni AEChloros

-a plus automatska ekspozicija na strani kamere upravljaju ekspozicijom (s zaštitom od zasitnosti) i prediktivni AE s jednom procjenom reflektancije scene nije siguran u miješanim scenama. Možete ga ponovno omogućiti po kameri ovdje ako posebno želite radiometrijsku ekspoziciju vođenu DLS-om.
{% endhint %}

**Gornja granica ekspozicije vođena DAQ-om i AE vezan uz incidentnu svjetlost.** Neovisno o gornjoj potvrdnoj okviru, kada je DAQ senzor svjetlosti dodijeljen kameriRGB

,Chloros

izračunava — na temelju izmjerene apsolutne dolazne infracrvene zračnosti — maksimalnu ekspoziciju×pojačanje pri kojoj površina s 100 % odrazom ostaje ispod praga rezanja, i primjenjuje ga kao **gornju granicu**na automatsko izlaganje. Dok je gornja granica aktivna, kamera je**fiksirana na incidentno osvjetljenje**: radi u otvorenoj petlji na izlaganju mjerenom za incidentno osvjetljenje s pojačanjem od 0 dB — izlaganje prati mjereno svjetlo, a ne sadržaj scene. Budući da gornja granica može samo skratiti ekspoziciju, sama po sebi ne može uzrokovati izrezivanje. Gornja granica se automatski isključuje — i normalna AE ekspozicija za scenu se nastavlja — kad god DAQ očitanje nedostaje, zastarjelo je (&gt;30 s), tamno je, ili ako ≥15 % kadra bude izrezano pri fiksiranoj ekspoziciji. (što znači da senzor i kamera vide različito osvjetljenje). Ne postoji GUI prekidač; ovo je standardno ponašanje kad god je DAQ priključen na kameru s ERGB

.

### Akvizicija i okidač

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Članovi niza dodatno prikazuju retke s informacijama samo za čitanje o **Ulogi**(Master plavom bojom / Slave zelenom bojom),**liniji sinkronizacije**i**kolegama**.

| Kontrola | Opcije | Zadano | Bilješke |
| --- | --- | --- | --- |
| **Način okidača** | Isključeno / Uključeno | Uključeno | Onemogućeno za članove niza (niz upravlja okidanjem). |
| **Izvor okidača** | Softver / Linija0 (M8) / Linija1 / Linija2 | Linija0 | Skriveno dok je način okidača isključen; onemogućeno za članove niza. Line0 je M8 opto-izolirani vanjski ulaz za okidač. |
| **Brzina snimanja**| Automatski / Ručno + vrijednost | Automatski |**Automatski**: ograničenje brzine snimanja kamere je isključeno — brzinu snimanja određuje ekspozicija, a uređaj prikazuje stvarnu brzinu snimanja uživo.**Ručno**: ograničavate fps klizačem (od 1 do maksimalne vrijednosti ograničene propusnošću), pri čemu se polazišna vrijednost temelji na trenutnoj stvarnoj brzini. Članovi niza vide samo &quot;N fps (uživo)&quot; s opcijom &quot;Postavi u postavkama niza&quot;. |

### Mreža / Transport

| Redak | Ponašanje |
| --- | --- |
| **Veličina paketa**| 1500 (Standardno) / 9000 (Jumbo) — zadano**Jumbo**. |
| **Propusnost** | Samo za čitanje ograničenje protoka veze u MB/s. Backend ovo uravnotežuje na svim priključenim kamerama pri svakom povezivanju/ispadanjima veze. |
| **Obrada predmemorije** | Samo za čitanje način obrade predmemorije. |

### Snimanje

Okno završava gumbom **&quot;Otvori postavke snimanja…&quot;** koji vodi do [okna Postavke snimanja](capture.md#the-capture-settings-pane) (onemogućen dok se ne otvori projekt — &quot;Kreirajte ili otvorite projekt za spremanje snimki&quot;). Ako je kamera skrivena ili pauzirana, podsjetnik će vas podsjetiti da je prije snimanja vratite iz skrivenog stanja/nastavite.

## Panel postavki niza

Otvorite pomoću **zupčanika**na retku NIZA. Zaglavlje: naziv niza s olovkom za preimenovanje i**×** za zatvaranje. Odjeljci u nastavku označeni kao *samo kombinirano* pojavljuju se samo za nizove povezane u kombiniranom načinu prikaza.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sinkronizacija

Redovi **Master**,**Slaves**i**Sync Line** su samo za čitanje.

### Senzor ambijentalnog svjetla

Prikazano za obje kombinirane i odvojene mreže:

* **Kalibracijski cilj** potvrdni okvir — &quot;DetektirajMAPIR

ArUco cilj i provjeriNDVI

u odnosu na refleksiju panela LUT&quot;; pokreće ciljni overlay i tablicu provjere kombinirane pločice.
* **padajući izbornik**Senzor svjetla** — povezuje jedan DAQ sa cijelim nizom. Odabir se odmah zadržava, prenosi na vlastiti padajući izbornik Senzor svjetla svake kamere u nizu (i dalje ga možete nadjačati za svaku kameru) i započinje prosljeđivanje spektra nizu.
* Uživo **Status** redak: Isključeno · &quot;Čeka se prvo spektrum…&quot; · &quot;Aktivno — sve kamere u nizu su korekcijom osvjetljenja&quot; · &quot;Nema novog spektra u posljednje 3 s — još uvijek se koristi posljednje očitanje (bez zastarjelog vremenskog ograničenja)…&quot;.
* Napomena u oknu: &quot;Radiometrijska korekcija za cijelu niz. Postavke po kameri imaju prednost.&quot;

### Snimanje — jednolične postavke senzora *(samo kombinirano)*

Ove se postavke jednolično primjenjuju na svaki član (promjene po članu bi prekinule sinkronizaciju). Promjene se pripremaju i primjenjuju zajedno.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Kontrola | Izbori / raspon | Što radi |
| --- | --- | --- |
| **Format piksela** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Ujednačen format senzora za sve članove. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardverski binning — zadržava cijelo vidno polje dok povećava SNR i brzinu sličica. Promjena postavke vraća ROI polja na novo cijelo vidno polje. |
| ****Postavka rezolucije** | Cijelo / Polovica / Četvrtina | Relativno u odnosu na binning; puni ROI polja centriranim izrezom. |
| **Izrez ROI-ja (px)**| Brojčana polja W / H / X / Y | Izrez senzora. Širina/visina se prilagođavaju na višekratnike od 16 (minimalno 64); pomaci se prilagođavaju na višekratnike od 4. Naznaka &quot;max WxH&quot; prikazuje maksimalnu vrijednost, a**Resetiraj** vraća na cijelo vidno polje. Tijekom uređivanja, na pločici niza iscrtava se narančasti okvir za pregled rezanja uživo (uključujući shematski prikaz cijelog senzora pri proširivanju rezanja prema van). |
| **Stopa okidanja**| Automatsko / Ručno prebacivanje + fps 0,5–10, korak 0,5 |**Automatsko**(zadano): pozadinski sustav izračunava stopu okidanja na temelju rezolucije i propusnosti — ulaz je onemogućen i prikazuje izračunatu vrijednost.****Ručno**: fiksira vašu vrijednost pri primjeni. |

Napomena u oknu: &quot;Promjene formata/rezolucije nakratko ponovno pokreću sve kamere. Stopa okidanja primjenjuje se uživo.&quot; Gumbi **Primijeni / Otkaži** nalaze se na dnu okna.

### Poravnanje (ko-registracija) *(samo kombinirano)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Preostali** stupanj: &quot;RMS x.xx px&quot; — zelena boja za manje od 1 px, žuta za manje od 3 px, crvena inače ili ako je neka kamera neuspjela; &quot;no profile&quot; prije prvog rješavanja.
* Redak sa sažetkom: &quot;✓ Sve kamere poravnate (N)&quot; / &quot;⚠ p/N kamera poravnato — <serial (filter)="">neuspjelo&quot; / &quot;Orezivanje aktivno — ponovno kalibrirati za poravnanje (koristi cijeli senzor)&quot; / &quot;Čeka se stabilizacija ekspozicije…&quot;.
* Tablica po kameri: kamera (zadnje 4 znamenke serijskog broja + filter), pogreška reprojekcije u px s brojem podudaranja (&quot;ref&quot; za glavnu), i ocjena normalizirane unakrsne korelacije preklapanja u odnosu na donju granicu od 0,35 za prolaz.
* Gumb **Ponovno kalibriraj poravnanje** (prije prvog profila piše &quot;Kalibriraj poravnanje&quot;) — ponovno pokreće su-registraciju na novim kadrovima.
* Kvadrat za potvrdu **&quot;Automatski eksponiraj kamere za poravnanje&quot;** (zadano označen) — privremeno pojačava slabu ili ravnu kameru (prvo ekspozicija, zatim pojačanje) kako bi imale teksturu za usklađivanje, a zatim vraća AE.

Kombinirani pregled se automatski usklađuje pri otvaranju; ponovno kalibrirajte ako se promijenio fokus ili dubina scene. Usklađivanje je **namjenski samo za sesiju** — nikada se ne sprema u profil jer ovisi o trenutnoj udaljenosti scene. Snimke se i dalje mogu izvesti s pikselskom registracijom (vidi [Aligned exports](capture.md#per-array-controls)).

### Pametna vinjeta

* **Kutija za potvrdu &quot;Omogući korekciju&quot;**— primjenjuje procjenu vinjete za svaku kameru na radiometrijski lanac (uživo**i** izvoze).
* **Kalibriraj iz trenutnog pogleda**— prvo usmjeri niz prema ujednačenoj meti (ravnoj ploči, zidu ili nebu); svaka se kamera pojedinačno poravnava i status prikazuje dobit poravnatosti &quot;n/N kamera · −x.x %&quot;.**Očisti** uklanja procjenu.
* Fino podešavanje po kameri pomoću klizača **Vignette** za svaku kameru u [Prikazu uživo](#live-preview).

### Prikaz uživo *(samo kombinirano)** **Indeks**: omogućite potvrdni okvir + zupčanik — otvara zajednički [Kalkulator indeksa](#index-calculator-pane) s pojasevima izvučenima iz**svih** kamera članica. Redak za pretpregled izraza ispod prikazuje trenutni izraz (&quot;Nije postavljen izraz — otvorite kalkulator za izradu&quot;), koji se osvježava svake sekunde.
* padajući izbornik **Rendering rezolucije**(isti unaprijed definirani postavke kao i po kameri, zadano 720p): visina prijenosa uživo**i** veličina spremljenog kompozita za izvoz. Napomena u oknu: &quot;Pregled + veličina spremljenog kompozita. Slike po kameri uvijek se izvoze u punoj rezoluciji.&quot;

### Prikaz slojeva *(samo za kombinirane)** **Kutija za potvrdu** **Omogući** (zadano isključeno — prikazuje se glavna kamera izravno; uključeno = slojeviti kompozit).
* Izbornici **Prednji plan**/**Pozadina**: svaka članica kamere (po nazivu) ili**Indeks**. Kada je u Prednjem planu postavljen Indeks, piksele izvan LUT Min/Max prikazuju sloj Pozadine.

### Podijeljeni prikaz *(samo za kombinirane)*

**&quot;Prikaži članice kamere&quot;**— gumb**Podijeli / Sakrij članice kamere** koji dodaje vlastiti prijenos uživo svake članice kao zasebne pločice u mreži pored kompozita. Kvadratići čitaju postojeći spremnik sličica niza (bez dodatne veze s kamerom). Samo za prikaz u mreži; spremljeno po nizu s projektom.

### Mogućnosti

Panel za čitanje samo u čitanju koji se osvježava svakih 5 s:

* **Oznaka sloja**: &quot;Istovremeno snimanje&quot; (zelena) · &quot;Istovremeno snimanje (FTD-razmaknuto emitiranje)&quot; (zelena) · &quot;Razmaknuto snimanje (100 ms odmak)&quot; (narančasta) · &quot;Konfiguracija je prevelika&quot; (crvena).
* **Zdravstveno stanje sličice**: &quot;x.xx % nepotpuno&quot; — zelena ispod 1 %, žuta ispod 5 %, crvena na 5 % ili više.
* **Linija veze**: &quot;NIC {mbps} Mbps - održivo {MB/s} MB/s&quot;.

Ovo je proračun propusnosti niza uživo. Za temeljni fps i mrežni model — i što promijeniti kada se razina pređe na narančastu ili crvenu — pogledajte [Multi-Camera Arrays](arrays.md) i [CLI Reference](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panel Izračunatelja indeksa

Treća stranica bočne trake, zajednička za polugu Indeks po kameri i polugu Indeks za kombinirani niz (po jedna u isto vrijeme — naslov glasi &quot;Index Calculator — <camera name="">&quot; ili &quot;Index Calculator — <array name="">&quot;). Prima popis pojaseva (pojasne filtre kamere ili sve pojaseve članova niza), trenutnu izraz, konfiguraciju LUT-a (uključi/isključi, razina — zadano 3, min — zadano 0,2, max — zadano 1), plus histogram indeksa uživo. **Primijeni** (Apply) potvrđuje izraz; promjene LUT-a primjenjuju se uživo na pregled.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Postavke po kameri naspram postavki upravljanih nizom

Brzi vodič za to što se gdje nalazi kada je kamera član niza:

| Upravljano s razine niza (samo za čitanje u odjeljku kamere) | I dalje po kameri unutar niza |
| --- | --- |
| Format piksela, rezolucija, biniranje | Automatska ekspozicija (ekspozicija, pojačanje, cilj, zaglađivanje, ROI) |
| Način/izvor okidača, brzina sličica | Smanjenje šuma, oštrina, vinjetiranje |
| | Orijentacija (ogledalo/rotacija), prikazni overleji, mjerač svjetline |
| | Indeks (nizovi sa zasebnim prikazom), povezivanje sa senzorom svjetlosti |

Ostalo presječivo ponašanje:

* **Kombinirani naspram odvojenog prikaza** odabire se pri povezivanju niza: kombinirano = jedna poravnata kompozitna pločica (samo sadržaj članova putem podijeljenog prikaza); odvojeno = svaki član prikazuje vlastitu sinkroniziranu pločicu. Kamera nikada ne prikazuje i samostalni prijenos i pločicu niza.
* **Automatsko ponovno povezivanje**: otvaranje spremljenog projekta vraća njegove kamere i nizove te ponovno primjenjuje svaku spremljenu postavku na pozadinski sustav prije nego što se streamovi nastave.
* **Kontrola snimanja**: skrivene ili pauzirane kamere isključene su iz opcije &quot;Snimanje svega&quot;; niz je u potpunosti blokiran samo kada su SVI članovi skriveni/pauzirani. Pogledajte [Postavke i načini snimanja](capture.md).

## Kako se postavke spremaju

Stanje kartice kamere sprema se **s projektom**, a ne u pregledniku:

* Svaka reaktivna promjena snima stanje kamera i nizova u projektov `cameras.json` (s odgađanjem od 500 ms). To uključuje nazive i boje kamera, postavke ekspozicije/pojačanja/AE, format piksela/rezoluciju/biniranje, brzinu okidača, postavke pretpregleda (rezolucija iscrtavanja, uklanjanje šuma, oštrina, vinjetiranje, profil boja, zasićenje/kontrast), orijentaciju, nadogradnje, razdvajanje kanala, konfiguraciju indeksa, postavke prediktivnog AE-a, AE ROI, nazive nizova, način prikaza, postavke snimanja niza (uključujući položaj izrezivanja ROI-ja) i blok mreže (zum izvora, način prikaza, zaključavanje mreže, ručni redoslijed pločica, skrivene kamere, zatvorene pločice, aktivna kamera).
* Poveznice senzora svjetla spremaju se u projektnu datotekChloros.
* Ponovnim otvaranjem projekta hardver se ponovno povezuje i sve se postavke ponovno primjenjuju.
* **Nema otvorenog projekta = samo sesija**: bez projekta, ništa se ne sprema zatvaranjem prozora .
* Samo za sesiju bez obzira na projekt: stanje pauze, uzorci mjerača ekspozicije, potvrdni okvir Cilj kalibracije za svaku kameru (uvijek je isključen) i profil poravnanja niza (namjenski se ponovno izračunava po sesiji).
* Jedan izuzetak: odabrani **Postavke snimanja** i načini snimanja pohranjuju se za svaki projekt zasebno u lokalnoj pohrani aplikacije, a ne u `cameras.json` — pogledajte [Postavke i načini snimanja](capture.md).</array></camera></serial>
