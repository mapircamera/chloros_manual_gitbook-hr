# kartica DAQ u Chloros

Kartica DAQ — označena kao **Svjetlosni senzori** u bočnoj traci Chloros — je nadzorna ploča za uživo za [svjetlosne senzore DAQ-U, DAQ-M i DAQ-E](README.md): povežite senzore putem bilo kojeg prijenosa, pratite kalibrirane spektre u stvarnom vremenu, izračunajte refleksiju uživo iz para senzora i spremite datoteke `.daq` izravno u svoj projekt.

Jezgra postaje dostupna nakon što se pozadinski servis Chloros završi s pokretanjem. Grafikoni na jezičcu napajaju se putem DAQ servisa aplikacije Chloros preko veze uživo koja se automatski ponovno uspostavlja (s odmakom od 2–10 s) ako dođe do prekida; dok je servis nedostupan, u retku Status senzora piše **No Server**.

Postavka se sastoji od **bočne trake za senzore**(po jedan redak za svaki priključeni senzor) i**područja grafikona** (po jedan pločić grafikon po senzoru ili grupi).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Povezivanje senzora

Kliknite **Poveži senzor**na vrhu bočne trake. Dijalog za povezivanje otvara se u glavnom području (ili kao preklopni prozor prilikom dodavanja drugog senzora — u tom slučaju pojavljuje se gumb**Otkaži**).

| Kontrola | Ponašanje |
| --- | --- |
| **Vrsta uređaja** | `DAQ-U (USB)` (zadano), `DAQ-M (Bluetooth)` ili `DAQ-E (Ethernet)`. Prebacivanje ponovno pokreće skeniranje za novo odabrani transport. |
| **Priključak / BLE uređaj / Naziv hosta / IP** | Popisuje otkrivene uređaje kao `device - description`; automatski se odabire prvi unos prepoznat kao senzor. Tijekom skeniranja prikazuje `Scanning...` (USB), `Scanning (N)...` s odbrojavanjem od 8 sekundi (BLE) ili `Discovering ethernet sensors (N)...` s odbrojavanjem od 5 sekundi (Ethernet). Prazni rezultati čitaju `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Osvježi** | Ponovno skenira odabrani transport odmah (onemogućeno tijekom BLE/Ethernet skeniranja). |
| **Poveži** | Omogućeno kada je uređaj odabran; mijenja oznaku u `Connecting...` dok se veza uspostavlja. |

Otkrivanje se pokreće samo **dok je dijalog za povezivanje vidljiv na zaslonu** i ponavlja se svakih 15 sekundi samo za odabrani transport — samim otvaranjem kartice ne provodi se skeniranje. U slučaju neuspjeha dijalog prikazuje: *&quot;Povezivanje nije uspjelo. Pokušajte isključiti i ponovno priključiti senzor, a zatim ponovno kliknite Poveži.&quot;*

Bočna traka se automatski otvara kada se vaš prvi senzor poveže.

{% hint style="info" %}
**DAQ-E se ne prikazuje?** DAQ-E nema LED indikator statusa — provjerite PoE/link indikator na portu switcha ili injectora u koji je priključen i pričekajte nekoliko sekundi nakon uključivanja da se pokrene. Računalo s Chloros mora biti na istoj broadcast domeni (mDNS ne prolazi kroz ruter). Na Windows prihvatite upit vatrozida Defender prvi put kada se Chloros veže na svoje multicast sokete (mDNS UDP 5353, DAQ-E podaci UDP 5002, PTP UDP 319/320). Dvije DAQ-E jedinice na jednom LAN-u otkrivaju se zasebno, svaka pod vlastitim imenom domaćina `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Tip uređaja nudi DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet)</figcaption></figure>

***

## Bočna traka senzora

Svaki priključeni senzor dobiva jedan redak (plus jedan redak po grupi Ambient+Object). Redak se može premještati povlačenjem, a njihov redoslijed također mijenja redoslijed pločica grafikona. Kliknite na redak kako biste taj senzor/grupu postavili kao aktivni grafikon u prikazu popisa.

| Element | Značenje |
| --- | --- |
| Obojena lijeva granica | Boja grafikona senzora. |
| Znak prijenosa | `DAQ-U` / `DAQ-M` / `DAQ-E`, ili zelena značka `REF` za grupu reflektancije okoliš+predmet. |
| Naziv uređaja | Zadano je serijski broj senzora (njegov stabilni identitet za kalibraciju, nazive datoteka `.daq` i usklađivanje uvoza); prilagođeni nazivi se spremaju za svaki projekt. |
| **Kalibrirana** tableta (zelena) | Prikazuje se kada je učitana tvornička kalibracijska zbirka senzora, tj. spektri su u stvarnim W/m²/nm. |
| **Dostupno ažuriranje** tableta (jantarna, samo za DAQ-E) | Trenutni firmware je stariji od slike priložene uz ovu izgradnjuChloros

. Tijekom ažuriranja prikazuje napredak uživo (`Flashing… N%`, `Restarting sensor…`, zatim `Updated X → Y` ili `Failed`). |
| Oko | Prebacuje vidljivost ovog senzora na njegovom grafikonu. |
| Zupčanik | Otvara modalni prozor s postavkama za pojedini senzor (dolje). |
| ✕ (crveno) | Odspaja senzor ili uklanja grupu Ambijent+Predmet. |

Iznad redaka nalaze se dva gumba:

* **Poveži senzor** — otvara dijalog za povezivanje (naziv se mijenja u `Connecting...` dok je u tijeku).
* **Zabilježi sve / Zaustavi sve**— pokreće ili zaustavlja snimanje `.daq` na**svakom**priključenom senzoru. Zahtijeva najmanje jedan senzor**i otvoreni projekt** (objašnjenje na alatu: &quot;Otvorite projekt za snimanje&quot;); postaje crven dok god se neko snimanje odvija.

Prazno stanje prikazuje &quot;Nema priključenih senzora&quot;.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Postavke po senzoru (modalni prozor s ikonom zupčanika)

Otvorite pomoću ikone zupčanika u retku senzora. Sadržaj je sljedeći:

* **Retci s informacijama** — Vrsta uređaja (DAQ-U/M/E), Povezanje (`Serial (USB)` / `Bluetooth` / `Ethernet`), Port (COM priključak, BLE adresa ili host) i Serijski broj.
* **Izvješće o kalibraciji: Preuzmi** — preuzima NIST-om provjereni certifikat o kalibraciji ove jedinice (PDF) i otvara ga u vašem pregledniku PDF-ova. Dostupno čim je serijski broj poznat; certifikat se kešira pri prvom povezivanju.
* **Naziv uređaja** — kliknite olovku za preimenovanje; vrijedi za svaki projekt.
* **Boja grafičke linije** — uzorak boje; zadržava se po projektu.
* **Vrijeme integracije (ms)**— klizač + broj,**1–500 ms**, zadano**32 ms**. Onemogućeno dok je AE UKLJUČEN.
* **Prosjek sličica**— klizač + broj,**1–50 sličica**, zadano**20**.
* **AE: UKLJUČENO/ISKLJUČENO**— prekidač za automatsku ekspoziciju;**zadano UKLJUČENO** pri povezivanju. Isključite ga za ručno postavljanje vremena integracije.
* **Zaustavi/Pokreni prijenos** — pauzira ili nastavlja prijenos uživo.
* **Snimanje / Zaustavi snimanje** — snimanje po senzoru `.daq` (potrebno je otvoriti projekt).
* **Cap** — profil korekcije kapa (sljedeći odjeljak).
* **Redak s informacijama uživo** — Vrijeme integracije (ms), FPS, Uzorci, Snimanje (crveno `REC` ili `Off`) i Stanje (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Samo za DAQ-E: retci za mrežu, firmware i PTP

* **Hostname / IP** — trenutačna adresa jedinice.
* **Firmware** — trenutačna verzija firmvera, plus ćelija za radnju:<version\>

gumb</version\>

**Ažuriraj na \<version\>** pojavljuje se kada ova verzijaChloros

a sadrži noviju sliku firmvera za DAQ-E. Ažuriranje se instalira preko mreže za otprilike 30 sekundi; senzor se automatski ponovno pokreće i povezuje, a prekinuti prijenos ostavlja trenutačni firmver nepromijenjenim. Progresivni streamovi su uživo (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), a ćelija čita `Up to date` kada je aktivna.
* **PTP Sync** — trenutačno PTP stanje (vraća se na `unknown`). Firmware DAQ-E v1.2.0+ sudjeluje u IEEE 1588 PTPv2 samo kao robni sat; backend hostaChloros

je PTP grandmaster, a svaka DAQ-E i LATTICE kamera na LAN-u mu je podređena u domeni 0, održavajući vremenske oznake unutar otprilike 1 ms.

Za grupu Ambient+Object, modalni prikaz zupčanika prikazuje samo izvore senzora grupe, Naziv uređaja i Boju grafičke linije.



<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Odabir kapa

Spuštano izbornik **Cap** govoriChloros

koja je fizička kapa postavljena na difuzor senzora i primjenjuje tvornički izmjereni profil korekcije te kapa na svaki spektar. Mogućnosti ovise o modelu:

| Model | Mogućnosti kapa |
| --- | --- |
| DAQ-U | Nema (goli senzor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosinusni korektor) |
| DAQ-M | Nema (goli senzor), Sunshine (kosinusni korektor) |
| DAQ-E | Nema (goli senzor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinusni korektor) |

**Zadana postavka za svaki model je Sunčeva svjetlost (kosi korektor)** —MAPIR

isporučuje svaki DAQ s ugrađenim poklopcem Sunčeva svjetlost, a to je standardna vanjska konfiguracija: hemisferni vidni kut od 180° s pogreškom u kosinu ≤ ±4 % do 60° i ≤ ±4,5 % do 70° (ne preporučuje se pri kutu solarnog uzdignuća manjem od ~15°), s namjernim slabljenjem (~12×). Vaš odabir ostaje sačuvan u projektu.

{% hint style="warning" %}
**Odabir kapa mora odgovarati fizičkoj kapi.**Ni senzor ni softver ne mogu detektirati koja je kapa ugrađena. Odabir utječe i na korekciju u stvarnom vremenu i na oznaku zabilježenu u svaku datoteku `.daq` — s prigušenošću kape Sunshine od ~12×, neproglašena promjena kape pogrešno korigira spektre za otprilike taj faktor. (Uklanjanje i ponovno postavljanje iste kapice ponavlja se s pogreškom od oko 1,5 %). Odaberite**None (goli senzor)** samo kada je kapica fizički uklonjena; na DAQ-E-u, opcija &quot;None&quot; i dalje primjenjuje tvornički geometrijski profil za njegov udubljeni stakleni difuzor — to nije bezakcijska operacija — a goli DAQ-E je laboratorijska konfiguracija, a ne podržana terenska konfiguracija.
{% endhint %}

{% hint style="info" %}
Nadogradnja s ranije verzije: prekidač &quot;Sunshine Diffuser Installed&quot; na strani preglednika iz verzije 1.1.0 je uklonjen. Upravljanje kapom sada se vrši putem ovog profila kape po senzoru, koji se primjenjuje na strani poslužitelja.
{% endhint %}

***

## Područje grafikona

Ljepljiva gornja traka sadrži **prekidač za prikaz popisa ⇄ mreže**i klizač za**zumiranje grafikona** (veličina pločice 200–2000 px). Prikaz se automatski prebacuje na mrežu kada postoji više od jedne grupe grafikona, a na popis kada postoji jedna ili manje. Način prikaza i veličina grafikona zadržavaju se po projektu.**Grafikon spektra** za svaki senzor prikazuje:

* **X-os** — Valna duljina (nm). Mreža senzora je 340–1010 nm u koracima od 5 nm (135 točaka), interpolirana na 1 nm za prikaz.
* **Y-os** — snaga (W/m²), s automatskim SI prefiksom (m/µ/n) odabranim prema vrhu. Spektri su radiometrijski kalibrirana spektralna zračenost (W/m²/nm) na sva tri prijenosa.
* Šarenoliko spektralno ispunjenje ispod jednog traga; više senzora na jednom grafikonu preklapaju se kao obojene linije s prigušenim ispunjenjem.
* **Hover**— vertikalni pokazivač s valnom duljinom i vrijednošću za svaki senzor;**povlačenje** za zumiranje (gumb za odzumiranje pojavljuje se tijekom zumiranja).
* Gumb **+** (samo u mrežnom prikazu) za dodavanje senzora ovoj grafikonu ili stvaranje grupe (ispod).
* Naziv uređaja je centriran na vrhu i prikazuje se kotačić dok ne stigne prvi kadar.

**Zasićenje** nije označeno na samom grafikonu: zasićeni senzor prikazuje crveni tekst statusa `SATURATED` i crveni redak `Saturated: Yes` u tablici s podacima uživo. Smanjite vrijeme integracije ili ponovno omogućite AE kako biste to otklonili.



<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tablica s podacima uživo (prikaz popisa)

Ispod grafikona u prikazu popisa, ažurira se svakih 500 ms:

* **Svi modeli**: Uzorak boje svjetla (sRGB iz CIE XYZ), Zasićenost (Da/Ne), CIE 1931 X/Y/Z, Kromatičnost x/y, CIE u′/v′, CCT (K), CRI (Ra), Dominantna valna duljina (nm), Vrhunski valni duljina (nm), Čistoća ekscitacije, Duv, CIE L*/a*/b* i Munsell H/V/C.
* **Samo kalibrirani senzori**(bilo koji od DAQ-U / DAQ-M / DAQ-E nakon što je učitana tvornička kalibracijska skupina — zeleni bedž**Kalibrirano** u retku senzora je znak): Ukupna snaga (W/m²), fotopski luks (lx), skotopski luks (lx), Omjer S/P, PPFD plus PPFDRed

/Green

/Blue

(µmol/m²/s) i opične iradijanse — S-koničar, Melanopični, Rodopični, M-koničar, L-koničar (svi W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupe reflektancije (Ambijentalno + Objekt)

Dva povezana senzora mogu se kombinirati u prikaz reflektancije u stvarnom vremenu — bez kamere:

1. U prikazu mreže kliknite **+**na pločicu grafikona i odaberite**Kombiniraj Ambijentalno + Objekt**.
2. Odaberite senzor **Izvor ambijentalnog svjetla**i senzor**Skenirano tijelo**(dva različita senzora), zatim**Stvori**.

Chloros

izračunava R(λ) = object(λ) / ambient(λ) po valnoj duljini iz dva uživo prijenosa (0 tamo gdje je ambient ≤ 0). Oznaka grupe slijedi kalibracijsku klasu senzora:

* Oba senzora kalibrirana (paket je učitan) → **&quot;Prividna reflektancija&quot;**.
* Kalibracija bilo kojeg senzora → **&quot;Relativna reflektancija&quot;**.

Grupa se pojavljuje kao zeleni redak `REF` u bočnoj traci i na vlastitom grafikonu (ispunjen duginim bojama, prikaz vrijednosti s 4 decimalne znamenke, povlačenje za zumiranje).

Izbornik **+**također nudi**Dodaj novi senzor** s tri postavke: *Kombiniraj novi senzor* (pridruži ovoj grafikonu), *Premjesti postojeći senzor ovdje* ili *Prikaži novi senzor* (na vlastitoj grafikonu).



<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tablica indeksa vegetacije

U prikazu popisa, tablica indeksa vegetacije nalazi se ispod grafikona skupine reflektancije, izračunata iz trenutne reflektancije u središtima pojaseva **plavo 450 / zeleno 550 / crveno 670 /NIR

800 nm** (vrijednosti do 4 decimalne znamenke, `---` kada se ne može izračunati; pređite mišem preko naziva indeksa za njegov puni naziv):

* **Uvijek prikazano** (neovisno o mjerilu, bilo koja kombinacija senzora):NDVI

,GNDVI

, ENDVI,WDRVI

,GRVI

,CVI

,GCI

, MSR.
* **Samo kada su oba senzora kalibrirana po snazi** (oba paketa su učitana):EVI

,SAVI

,OSAVI

, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI,LAI

, NLI, MNLI, FCI,GEMI

.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Snimanje datoteka `.daq`

* Snimanje zahtijeva **otvoren projekt** — u suprotnom su onemogućeni i gumb za snimanje svega (u bočnoj traci) i gumb za snimanje po senzoru.
* Datoteke se spremaju u **`<project folder>/light_sensor/`**; nazivi datoteka sadrže ID senzora i vremensku oznaku, a naziv uređaja pohranjuje se uz snimku.
* Kada se snimanje zaustavi (Zaustavi, Zaustavi sve ili prekid veze usred snimanja), dovršena datoteka `.daq` **automatski se dodaje u otvoreni projekt** — pojavljuje se u popisu datoteka projekta bez ručnog dodavanja, spreman za upotrebu kao podaci o dolaznom zračenju za [obradu reflektancije](README.md).
* Crveni indikator `REC` prikazuje se u redovima uživo modala postavki tijekom snimanja.

Za kvantitativne brojke zračenja, prosječno uzmite podatke u trajanju od najmanje 15 sekundi — to je karakteristika instrumenta, a ne kvar.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Rasporedi sa više senzora i očuvanje postavki projekta

* Kombinirajte nekoliko senzora na jednom grafikonu (zajedničke osi), zadržite odvojene grafikone (automatski raspored mreže), premještajte senzore između grafikona, povlačite redove/pločice kako biste ih presložili i sakrijte pojedinačne senzore pomoću prekidača za prikaz/sakrivanje.
* Po projektu,Chloros

pohranjuje: nazive uređaja, boje grafikona, veličinu grafikona, način prikaza i postavke svakog senzora (vrijeme integracije, prosječno izračunavanje sličica, stanje AE, odabir kapice).
* **Ponovno otvaranje projekta automatski ponovno povezuje njegove senzore** po adresi — COM priključak za DAQ-U, BLE uređaj za DAQ-M, mDNS naziv domene za DAQ-E (rješava se čak i ako se IP jedinice promijenio) — i ponovno primjenjuje spremljeni profil kapice, prosječno vrijeme sličica, stanje AE-a i ručno vrijeme integracije svakog senzora.***

## Spajanje kamere (DLS)

Nema se što upariti. Za razliku od DLS tijekova rada s dronovima koji unaprijed vežu senzor svjetla za kameru,Chloros

usklađuje DAQ podatke sa slikama naknadno: tijekom uvoza/obrade, očitanja s uređaja `.daq` interpoliraju se na vremenski žig ekspozicije svake snimke. Zabilježite snimku bilo kojim priključenim senzorom (čitanje `.daq` automatski se učitava u projekt), a obrada reflektancije pronalazi odgovarajuća očitanja prema vremenu — pogledajte [DAQ senzori svjetla](README.md) kako se koriste podaci o dolaznom svjetlu.</version\>
