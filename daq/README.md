# DAQ senzori svjetla

> **Tražite li hardver?**Sami senzori — modeli, montaža, poklopci, priključci, napajanje i aplikacija SCANNER — dokumentirani su u**[DAQ korisničkom priručniku](https://mapir.gitbook.io/daq)**. Ovo poglavlje pokriva njihovu upotrebu iz programa Chloros.**DAQ** senzori svjetla programa

MAPIR mjere ambijentalno svjetlo kao radiometrijski kalibrirane spektre. U programu Chloros oni imaju dvije uloge:

* **Samostalni spektralni instrument** — grafikoni spektra uživo, kolorimetarski podaci i snimke `.daq`, sve iz [karticu Light Sensors](gui.md), [CLI](cli-quick-start.md) ili Python SDK.
* **Izvor dolazne iradijance za reflektanciju** — tijekom obrade, Chloros interpolira vaša očitanja s `.daq` svakom vremenskom žigom ekspozicije snimanja i koristi izmjerenu svjetlost koja pada na senzor za pretvaranje radijance kamere u reflektancu (`--reflectance-source daq`), bez potrebe za panelom na sceni za kalibrirane pojaseve.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tri modela, jedan format podataka

| Model | Transport | Discovery |
| --- | --- | --- |
| **DAQ-U** | USB (serijski) | skeniranje serijskog priključka |
| **DAQ-M** | Bluetooth Low Energy | BLE skeniranje po nazivu |
| **DAQ-E** | Ethernet (IPv4, napajanje putem PoE-a) | mDNS `_daq-e._tcp` (naziv hosta `daq-e-<id>.local`) |**DAQ-E-S (novo u verziji 1.3.0).** DAQ-E s trajnim difuzorom i senzorom nagiba. Chloros ga navodi kao `DAQ-E-S` i koristi ga točno kao DAQ-E, s iznimkom što ne primjenjuje korekciju kapa i svako očitanje sadrži podatke o nagibu, rotaciji i smjeru jedinice. Vidi [DAQ-E-S: fiksni difuzor i nagib](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Sva tri koriste isti protokola prijenosa podataka i isporučuju identične podatke:

* **Spektrum od 135 točaka od 340 do 1010 nm u koracima od 5 nm**, plus CIE XYZ tristimulusi, u svakom kadru.
* **Spektralna iradijansa radiometrijski kalibrirana u W/m²/nm** — tvornička kalibracijska skupina svake jedinice (plus njezin aktivni profil korekcije kapa) primjenjuje se prije nego što vam stignu podaci.
* Isti **`.daq` format snimanja** (SQLite datoteka). Naknadna obrada je identična bez obzira na to koji je transport proizveo datoteku.

Transportni slojevi (USB serial, BLE, mDNS/zeroconf) objedinjeni su u pozadinskom sustavu Chloros — nema potrebe ništa instalirati za povezivanje s bilo kojim od tri modela putem GUI-ja ili `pool-*` naredbi iz CLI-a.

***

## Kalibrirani raspon: izvještava se 340–1010 nm, kalibrirano 374–974 nm

Senzor izvještava o punoj mreži od 340–1010 nm, ali radiometrijski pojačanje koje se može pratiti do NIST-a obuhvaća otprilike **374–974 nm**. Chloros odbijaju podjelu apsolutne reflektancije za bilo koji spektar kamere s manje od polovice svoje spektralne težine unutar tog kalibriranog raspona; preskočeni spektar izvještava se uz razlog preskakanja `dls-uncalibrated-band-<nm>`.

Među isporučivanim LATTICE filtrima, samo je **F988** pogođen:

Reflekcija za F988 kalibrira se pomoću reflektirajuće ploče unutar scene: pojas se nalazi izvan kalibriranog raspona DAQ senzora svjetla, stoga Chloros primjenjuje vašu najnoviju snimku ploče i pohranjuje je između promatranja ploče.

Ako se snimanje F988 obrađuje s dostupnim samo DAQ podacima, Chloros odbacuje reflektanciju temeljenu na DAQ-u za tu traku s razlogom preskakanja `dls-uncalibrated-band-988` — [radni tijek s reflektirajućom pločom](../calibration-targets.md) je podržani put za F988.

***

## ID-ovi senzora

Svaki DAQ izvještava o stabilnom ID-u senzora. Njegov oblik razlikuje se ovisno o modelu:

| Model | Oblik ID-a | Primjer |
| --- | --- | --- |
| DAQ-U | 5-oktetski s crticom | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-ocetni s crticama | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzora je:

* utisnut u svaku datoteku `.daq` koju snima,
* ključ koji Chloros koristi za dohvaćanje tvorničkog kalibracijskog paketa te jedinice,
* vrijednost koju prosljeđujete `--sensor-id` u CLI-ovim `pool-*` naredbama, i
* za DAQ-E, također i njegovo mDNS host ime (`daq-e-def330.local`) — vrijednost koju `--eth-host` prihvaća.

***

## Tvornicka kalibracija i oblak

Svaka DAQ jedinica pojedinačno je tvornički kalibrirana radiometrijskim lancem koji se može pratiti do NIST-a, a Chloros učitava paket kalibracije svake jedinice prema ID-u njezina senzora. Izvješće o kalibraciji za svaku jedinicu (PDF) može se preuzeti iz postavki senzora na [kartici Svjetlosni senzori](gui.md).

{% hint style="warning" %}
**DAQ-U i DAQ-M za kalibraciju zahtijevaju pristup oblaku.**Nijedan model ne pohranjuje ništa na uređaju: njihovi tvornički kalibracijski paketi nalaze se u oblaku tvrtke MAPIR i preuzimaju se prema ID-u senzora (a zatim se lokalno keširaju). Chloros-u je potrebna internetska veza za isporuku kalibriranih podataka W/m²/nm s uređaja DAQ-U ili DAQ-M.**DAQ-E je iznimka** — on pohranjuje svoju kalibraciju na samom uređaju.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Odredište snimki

| Površina | Zadani odredišni `.daq` |
| --- | --- |
| GUI — kartica Svjetlosni senzori | `<project folder>/light_sensor/` (gotove snimke se automatski dodaju u projekt) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na računalu na kojem se pokreće backend (relativna putanja `-o` rješava se u odnosu na mapu u kojoj ste pokrenuli naredbu kada se pozadinski sustav (backend) nalazi na istom računalu) |

Svaki naziv datoteke `.daq` uključuje ID senzora i vremensku oznaku.

***

## U ovom poglavlju

* [**DAQ kartica u programu Chloros**](gui.md) — potpuni vodič kroz GUI: povezivanje svakog modela, postavke za svaki senzor, spektralni grafikon, kolorimetrijski podaci u stvarnom vremenu, reflektancija dvaju senzora i snimanje.
* [**CLI Quick Start (pool-\*)**](cli-quick-start.md) — upravljanje DAQ senzorima s `chloros-cli daq pool-*`, podržana putanja naredbenog retka.
* [**Profilni filtri i kalibrirani raspon**](caps-and-range.md) — koji se profilni filtri nalaze za svaki model, kako ih deklarirati i detaljan opis kalibriranog spektralnog raspona.
* [**Zapisivanje i .daq format**](recording.md) — SQLite format `.daq` i tijekovi rada zapisa.
* [**DAQ-E umrežavanje i vremenska sinkronizacija**](ethernet-ptp.md) — DAQ-E načini prijenosa i PTP vremenska sinkronizacija.
* [**Radni tokovi reflektancije**](reflectance.md) — korištenje DAQ podataka o zračenju odozgo za izračun reflektancije.
* Za potpunu dokumentaciju na razini zastavica pogledajte [CLI reference](../reference/cli-reference.md) (odjeljak `chloros-cli daq`) i [SDK reference](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), oba napisana za izravnu upotrebu od strane AI asistenata.
