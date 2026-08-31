# DAQ-E umrežavanje i sinkronizacija vremena

> Fizičko postavljanje mreže za senzor — kabliranje, PoE, dodjela IP adrese i vlastite mrežne postavke uređaja — nalazi se u **[DAQ korisničkom priručniku](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Ova stranica pokriva DAQ-E stranu: povezivanje, sinhronizaciju vremena i što učiniti kada otkrivanje ne da rezultate.

DAQ-E je Ethernet član obitelji DAQ: napaja se putem PoE-a, otkriva se putem mDNS-a (usluga `_daq-e._tcp`) i adresira se pomoću naziva hosta izvedenog iz ID-a njegovog senzora — `daq-e-<6 hex>.local`, npr. `daq-e-def330.local`. Ova stranica objašnjava kako prenosi podatke mrežom i kako sudjeluje u PTP sinkronizaciji vremena.

## Transportni načini

| Način | Odredište | Potrošači | Napomene |
| --- | --- | --- | --- |
| **Multicast** (podrazumijevano) | UDP `239.10.10.10:5002` | Bilo koji broj na istom LAN-u prima isti stream | Svaki datagram provjerava se pomoću CRC-16/CCITT |
| **Sirovi** | TCP port `5000` | Točno jedan klijent (isključivo) | Kompatibilan po bajtovima s DAQ-U |

Chloros

prema zadanim postavkama koristi multicast, što omogućuje GUI-ju,CLI

iSDK

da istovremeno prate jedan senzor.

## Mrežni zahtjevi

* **Ista broadcast domena.** Računalo na kojem se pokrećeChloros

mora biti na istom L2 mrežnom segmentu kao i senzor — otkrivanje putem mDNS-a ne prolazi kroz usmjerivače.
* **Upit vatrozida zaWindows

: prihvatite ga.** Prvi put kadaChloros

veže multicast sokete,Windows

Defender pita jednom. Dopuštanje tome pokriva podatke DAQ-E (UDP 5002), mDNS (UDP 5353) i PTP (UDP 319/320). NaLinux

to se ne primjećuje.
* **PoE napajanje, nema LED statusne svjetiljke.** DAQ-E nema vlastitu LED svjetiljku — provjerite napajanje putem indikatora veze/PoE-a na portu switcha ili injectora i pričekajte nekoliko sekundi nakon uključivanja da se pokrene i priključi mreži.

## Povezivanje

**GUI:** kartica Light Sensors → Poveži senzor → Vrsta uređaja &quot;DAQ-E (Ethernet)&quot;. Otkrivanje se pokreće samo dok je dijalog za povezivanje vidljiv na zaslonu (mDNS pregled plus ARP skeniranje naWindows

), ponavljajući se svakih 15 sekundi; gumb Ažuriraj odmah ponovno skenira. Otkriveni senzori pojavljuju se u padajućem izborniku; prvi otkriveni senzor automatski je odabran.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend u pokretanju):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hostovi s više mrežnih sučelja i prvo povezivanje nakon pokretanja

Na hostovima s više aktivnih mrežnih sučelja, **prvo** `pool-connect --eth` nakon pokretanja može biti prazno čak i kada je senzor ispravan — pregledavanje za otkrivanje može propustiti sučelje na kojem se senzor nalazi dok je ARP keš još hladan. Pouzdan način je preskočiti otkrivanje i izričito navesti adresu:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` prihvaća naziv hosta mDNS-a ili IP adresu, uvijek cilja na odgovarajući senzor i preporučeni je oblik za skripte i instalacije bez zaslona. U GUI-ju upotrijebite gumb Osvježi u dijaloškom okviru za povezivanje i dopustite ciklusu ponovnog skeniranja da se završi.

## Postavke uređaja i firmvera

Sama sonda sadrži mrežne postavke — statičku IP adresu nasuprot DHCP-u + adresiranju vezanom za vezu, naziv uređaja, automatski streaming pri pokretanju, OTA lozinku. Ove postavke na strani uređaja nisu dostupne kao naredbe u isporučenomCLI

-u; njima se upravlja putem GUI-jaChloros

, gdje su prikazane, ili uz podrškuMAPIR

.

Ažuriranja firmvera integrirana su u GUI. Kada je priključeni DAQ-E opremljen firmverom starijim od slike priložene uz vašu verzijuChloros

, njegov red senzora prikazuje narančastu pilulu **Dostupno ažuriranje**, a modalni prozor postavki s zupčanikom nudi <version>gumb</version>

&quot;Ažuriraj na <version>&quot; Ažuriranje se instalira preko mreže za otprilike 30 sekundi; senzor se automatski ponovno pokreće i povezuje, a prekidani prijenos ostavlja trenutni firmware nepromijenjenim.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP sinkronizacija vremena

Firmware DAQ-E-a v1.2.0+ sudjeluje u IEEE 1588 PTPv2 kao obični (samo-rob) sat. Backend hosta Chloros je PTP grandmaster — svaki DAQ-E i svaka LATTICE kamera na LAN-u mu je podređen u domeni 0, održavajući sve vremenske oznake uređaja unutar tolerancije od ~1 ms. Taj zajednički sat omogućuje usklađivanje vremenskih oznaka DAQ očitanja s ekspozicijama kamere (vidi [Snimanje i .daq format](recording.md)).

Provjerite sinkronizaciju pomoću naredbe CLI:

| Naredba | Prikazuje |
| --- | --- |
| `chloros-cli time-sync status` | Stanje host grandmastera, BMCA prioriteti, identitet sata |
| `chloros-cli time-sync peers` | Svaki vidljivi slave (DAQ-E senzori + LATTICE kamere) |
| `chloros-cli time-sync cameras` | Stanje PTP-a po kameri (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Ponovno pokrenite grandmaster proces |

U GUI-ju, modalni prozor s postavkama DAQ-E prikazuje redak **PTP Sync** u stvarnom vremenu s trenutnim PTP statusom senzora.

Detalji za potrošače s strogo poravnanjem:

* Svaki prenoseni datagram sadrži polje zastavica; **bit 2 je postavljen na okvirima čiji je vremenski žig PTP-sinkroniziran**. Cjevovodi koji zahtijevaju strogo poravnanje kamere/DAQ-a trebali bi provjeravati taj bit.
* Prije sinkroniziranog snimanja, provjerite pojavljuje li se senzor u `chloros-cli time-sync peers`. (MAPIR-ova interna alata za izravno upravljanje hardverom također može uvjetovati snimanje PTP zaključavanjem pomoću zastavice `--wait-ptp` koja čeka do 15 s da senzor prijeđe u stanje SLAVE; ta alata nije dio isporučenog CLI-a.)
* Dok je PTP aktivno u načinu sljedbenika (slaving), senzor odbija ručno nametanje takta (&quot;PTP osigurava takt&quot;). To je namjerno — vjerujte PTP-u.

## Napomene o Linux

* **PTP zahtijeva `libcap2-bin` tijekom instalacije.** Postinstalacijski skript `.deb` dodjeljuje `cap_net_bind_service=+ep` na `/usr/lib/chloros/chloros-backend` kako bi se mogao povezati na PTP priključke 319/320 bez root prava. Ako `libcap2-bin` nedostaje, taj se korak preskače i PTP se neće pokrenuti. Popravak:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** pri prvoj instalaciji generira se, ali nije omogućen, systemd unit `chloros-backend.service`. Za stalno aktivan PTP (i dostupnost DAQ-a) bez GUI-ja:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Bez njega, PTP radi samo dok je GUI Chloros-a otvoren.

## Rješavanje problema: &quot;Nije pronađena nijedna DAQ-E jedinica&quot;

| Provjera | Detalj |
| --- | --- |
| Napajanje | Nema LED-a na senzoru — provjerite PoE i indikatore veze na prekidaču/injektoru; pričekajte nekoliko sekundi nakon uključivanja |
| Broadcast domena | Host i senzor na istom L2 segmentu; mDNS se ne usmjerava |
| Vatrozid Windows | Prihvatite upit Defendera pri prvom pokretanju (UDP 5002, 5353, 319/320) |
| Host s više mrežnih kartica | Prvo otkrivanje nakon pokretanja može propustiti senzor — povežite se pomoću `--eth-host <ip-or-hostname>` |
| Ponovno skeniranje GUI-ja | Otkrivanje se pokreće samo dok je dijalog za povezivanje otvoren; upotrijebite Obnovi |</version>
