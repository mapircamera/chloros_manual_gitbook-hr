# DAQ senzori svjetla

> **Tražite hardver?**Sami senzori — modeli, montaža, poklopci, priključci, napajanje i aplikacija SCANNER — dokumentirani su u**[DAQ korisničkom priručniku](https://mapir.gitbook.io/daq)**. Ovo poglavlje pokriva njihovu upotrebu iz Chloros.**DAQ** senzori svjetla tvrtke

MAPIRmjere ambijentalno svjetlo kao radiometrijski kalibrirane spektre. U Chloros-u imaju dvije uloge:

* **Samostalni spektralni instrument** — grafikoni spektra uživo, kolorimetrički podaci i snimke `.daq`, sve iz [Light Sensors tab](gui.md), [CLI](cli-quick-start.md) ili PythonSDK.
* **Izvor zraka odozgo za reflektanciju** — tijekom obrade, Chloros interpolira vaša očitanja `.daq` s vremenskim žigom ekspozicije svakog snimanja i koristi izmjerenu svjetlost koja pada na senzor kako bi pretvorio radijancu kamere u reflektancu (`--reflectance-source daq`), bez potrebe za panelom unutar scene za kalibrirane pojaseve.



<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->***

## Tri modela, jedan format podataka

| Model | Transport | Discovery |
| --- | --- | --- |
| **DAQ-U** | USB (serijski) | skeniranje serijskog priključka |
| **DAQ-M** | Bluetooth Low Energy | BLE skeniranje po nazivu |
| **DAQ-E** | Ethernet (IPv4, napajanje putem PoE-a) | mDNS `_daq-e._tcp` (naziv hosta `daq-e-<id>.local`) |

Sva tri koriste isti komunikacijski protokol i isporučuju identične podatke:

* **Spektrum od 135 točaka od 340 do 1010 nm u koracima od 5 nm**, plus CIE XYZ vrijednosti tristimulusa, u svakom kadru.
* **Spektralna iradijansa radiometrijski kalibrirana u W/m²/nm** — tvornička kalibracijska skupina svake jedinice (plus njezin aktivni profil korekcije kapa) primjenjuje se prije nego što vam stignu podaci.
* Isti **format snimanja `.daq`** (SQLite datoteka). Naknadna obrada je identična bez obzira na to koji je transport proizveo datoteku.

Transportni slojevi (USB serial, BLE, mDNS/zeroconf) objedinjeni su u pozadinskom sustavu Chloros — nema potrebe ništa instalirati za komunikaciju s bilo kojim od tri modela putem grafičkog sučelja ili `pool-*` naredbi CLIa.

***

## Kalibrirani raspon: izvještava o 340–1010 nm, kalibrirano od ~374–974 nm

Senzor izvještava o punoj mreži od 340–1010 nm, ali radiometrijski pojačanje koje se može pratiti do NIST-a pokriva otprilike raspon **374–974 nm**. Chloros odbija podjelu apsolutne reflektancije za bilo koji spektar kamere s manje od polovice svoje spektralne težine unutar tog kalibriranog raspona; preskočeni spektar izvještava se uz razlog preskakanja `dls-uncalibrated-band-<nm>`.

Među isporučivanim LATTICE filtrima, samo je **F988** pogođen:

Reflekcija F988 kalibrira se pomoću reflektirajuće ploče unutar scene: pojas leži izvan kalibriranog raspona DAQ senzora svjetla, stoga Chloros primjenjuje vašu najnoviju snimku ploče i zadržava je između promatranja ploče.

Ako se snimanje F988 obrađuje s dostupnim samo DAQ podacima,  odbacuje reflektanciju temeljenu na DAQ-u za taj pojas s razlogom preskakanja `dls-uncalibrated-band-988` — [radni tijek s reflektirajućim panelom](../calibration-targets.md) je podržana putanja za F988.

***

## ID-ovi senzora

Svaki DAQ izvještava o stabilnom ID-u senzora. Njegov oblik razlikuje se ovisno o modelu:

| Model | Oblik ID-a | Primjer |
| --- | --- | --- |
| DAQ-U | 5-ocetni s crticama | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-octetni s crticama | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzora je:

* utisnut u svaku datoteku `.daq` koju snima,
* ključ koji Chloros koristi za dohvaćanje tvorničkog kalibracijskog paketa te jedinice,
* vrijednost koju prosljeđujete `--sensor-id` u CLI `pool-*` naredbama, i
* za DAQ-E, također i njegovo mDNS korisničko ime (`daq-e-def330.local`) — vrijednost koju `--eth-host` prihvaća.

***

## Tvornicka kalibracija i oblak

Svaka DAQ jedinica pojedinačno je tvornički kalibrirana radiometrijskim lancem koji se može pratiti do NIST-a, a Chloros učitava paket kalibracije svake jedinice prema ID-u njezina senzora. Izvješće o kalibraciji za svaku jedinicu (PDF) može se preuzeti iz postavki senzora na [kartici Svjetlosni senzori](gui.md).

{% hint style="warning" %}
**DAQ-U i DAQ-M za kalibraciju zahtijevaju pristup oblaku.**Nijedan model ne pohranjuje ništa na uređaju: njihovi tvornički kalibracijski paketi nalaze se u oblaku tvrtke MAPIR i preuzimaju se prema ID-u senzora (a zatim se lokalno keširaju). Chloros-u je potrebna internetska veza za isporuku kalibriranih podataka W/m²/nm s uređaja DAQ-U ili DAQ-M.**DAQ-E je iznimka** — svoju kalibraciju pohranjuje na samom uređaju.
{% endhint %}


<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->***

## Gdje se snimke pohranjuju

| Površina | Zadana odredišna lokacija za `.daq` |
| --- | --- |
| GUI — kartica Svjetlosni senzori | `<project folder>/light_sensor/` (gotovi zapisi se automatski dodaju u projekt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na računalu na kojem se pokreće pozadinski sustav |

Svaki naziv datoteke `.daq` uključuje ID senzora i vremensku oznaku.

***

## U ovom poglavlju

* [**Pokretanje Chloros-a**](gui.md) — potpuni pregled GUI-ja: povezivanje svakog modela, postavke za svaki senzor, spektralni grafikon, kolorimetarski podaci uživo, reflektancija dvaju senzora i snimanje.
* [**Brzi početak rada s CLI (pool-\*)**](cli-quick-start.md) — upravljanje DAQ senzorima iz `chloros-cli daq pool-*`, podržana putanja u naredbenom retku.
* [**Profilne kapice i kalibrirani raspon**](caps-and-range.md) — koje kapice postoje za svaki model, kako ih deklarirati i detaljan opis kalibriranog spektralnog raspona.
* [**Zapisivanje i format .daq**](recording.md) — SQLite format `.daq` i tijekovi rada zapisa.
* [**DAQ-E umrežavanje i vremenska sinkronizacija**](ethernet-ptp.md) — DAQ-E načini prijenosa i PTP vremenska sinkronizacija.
* [**Radni tokovi reflektancije**](reflectance.md) — korištenje DAQ podataka o zračenju prema dolje za izračun reflektancije.
* Za potpunu dokumentaciju na razini zastavica pogledajte referencu [CLI](../reference/cli-reference.md) (odjeljak `chloros-cli daq`) i referencu [SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), oba napisana tako da ih AI asistenti mogu izravno koristiti.
