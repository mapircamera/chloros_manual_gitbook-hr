# DAQ-E umrežavanje i sinkronizacija vremena

> Fizičko postavljanje mreže za senzor — kabliranje, PoE, dodjela IP adrese i vlastite mrežne postavke uređaja — nalazi se u **[DAQ korisničkom priručniku](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Ova stranica pokriva stranu Chloros a: povezivanje, sinkronizaciju vremena i što učiniti kada otkrivanje ne da rezultate.

DAQ-E je Ethernet član obitelji DAQ: napajanje putem PoE-a, otkrivanje putem mDNS-a (usluga `_daq-e._tcp`) i adresabilan pomoću naziva hosta izvedenog iz ID-a njegovog senzora — `daq-e-<6 hex>.local`, npr. `daq-e-def330.local`. Ova stranica objašnjava kako prenosi podatke mrežom i kako sudjeluje u PTP sinkronizaciji vremena.

## Modovi prijenosa

| Mod | Odredište | Potrošači | Napomene |
| --- | --- | --- | --- |
| **Multicast** (podrazumijevano) | UDP `239.10.10.10:5002` | Bilo koji broj primatelja na istoj LAN mreži prima isti stream | Svaki datagram provjerava se pomoću CRC-16/CCITT |
| **Raw** | TCP port `5000` | Točno jedan klijent (ekskluzivno) | Kompatibilan po bajtovima s DAQ-U |

Chloros prema zadanim postavkama koristi multicast, što omogućuje GUI-ju, CLI-u i SDK-u da istovremeno nadziru jedan senzor.

## Mrežni zahtjevi

* **Ista broadcast domena.** Uređaj na kojem se pokreće Chloros mora biti na istom L2 mrežnom segmentu kao i senzor — otkrivanje putem mDNS-a ne prolazi kroz usmjerivače.
* **Vatrozid.** Na sustavu Windows instalator dodaje pravilo vatrozida **Chloros Backend (UDP-In)**za pozadinski sustav, na svaki mrežni profil, koje pokriva podatke DAQ-E (UDP 5002), mDNS (UDP 5353) i PTP (UDP 319/320), te pravilo**Chloros Backend (TCP-In)**, ograničeno na lokalni podmrežu, za vezu koju DAQ-E uspostavlja s računalom tijekom ažuriranja firmvera. Na sustavu Linux, `.deb` otvara UDP priključke u `ufw` ili `firewalld` kada je taj vatrozid aktivan tijekom instalacije. Vatrozid treće strane, ili onaj omogućen naknadno, mora imati omogućene iste priključke.
* **PoE napajanje, nema LED diode za status.** DAQ-E nema vlastitu LED diodu — provjerite napajanje putem indikatora veze/PoE-a na portu switcha ili injectora i pričekajte nekoliko sekundi nakon uključivanja da se pokrene i priključi mreži.

## Povezivanje

**GUI:** kartica Light Sensors → Connect Sensor → Device Type "DAQ-E (Ethernet)". Otkrivanje se pokreće samo dok je dijalog za povezivanje vidljiv na zaslonu (mDNS pregled plus ARP skeniranje na Windows u), ponavljajući se svakih 15 sekundi; gumb Refresh odmah ponovno skenira. Otkriveni senzori pojavljuju se u padajućem izborniku; prvi otkriveni senzor automatski je odabran.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend u tijeku):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

DAQ-E koji je već u skupini prepoznaje se i po svom hostnameu i po IP adresi: njegovo ponovno povezivanje pod drugom adresom vraća postojeći unos u skupini umjesto da jedinicu otvara drugi put.

### Hostovi s više mrežnih sučelja i prvo povezivanje nakon pokretanja

Na hostovima s više aktivnih mrežnih sučelja, **prvo** `pool-connect --eth` nakon pokretanja može ostati prazno čak i kada je senzor ispravan — pregledavanje otkrivanja može previdjeti sučelje na kojem se senzor nalazi dok je ARP predmemorija još hladna. Pouzdano rješenje je preskočiti otkrivanje i izričito navesti adresu:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` prihvaća mDNS naziv hosta ili IP adresu, uvijek usmjerava na ispravan senzor i preporučena je forma za skripte i instalacije bez zaslona. U GUI-ju upotrijebite gumb Osvježi u dijaloškom okviru za povezivanje i dopustite ciklus ponovnog skeniranja.

## Postavke uređaja i firmvera

Sensor sam po sebi ima mrežne postavke: DHCP (tvornica postavka, koja se vraća na lokalnu veznu adresu 169.254.x.x kada DHCP poslužitelj ne odgovori) ili statičku adresu. Također se postavlja i da li počinje s prijenosom u trenutku uključivanja. Promijenite ih u GUI-ju (postavke zupčanika senzora, **Mreža**:**Promijeni…**,**Prenosi pri uključivanju**,**Ponovno pokreni senzor**; vidi [The DAQ Tab](gui.md)), ili iz CLI a i SDK a pomoću `pool-set-network`, `pool-set-auto-stream` i `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; pogledajte [Referencu za CLI](../reference/cli-reference.md#daq-e-device-settings)). Senzor se ponovno pokreće kako bi se primijenila promjena mreže, a Chloros ga ponovno povezuje. Na mreži računala na kojem se pokreće Chloros mora postojati statička adresa: Chloros odbacuje onu do koje nije mogao doći nakon ponovnog pokretanja. ID senzora i njegovo korisničko ime `daq-e-<id>.local` nikada se ne mijenjaju. Novost u verziji 1.3.0.

**Ažuriranja firmvera integrirana su u GUI.** Kada priključeni DAQ-E ima firmver stariji od slike priložene uz vašu verziju programa Chloros, u njegovom redu senzora prikazuje se narančasta pilula **Dostupno ažuriranje**, a modalni prozor postavki zupčanika nudi gumb "Ažuriraj na \<version\>". Ažuriranje se prebacuje preko mreže za otprilike 30 sekundi; senzor se ponovno pokreće i automatski se ponovno povezuje, a prekidani prijenos ostavlja trenutni firmware nepromijenjenim. S CLI-a ili SDK-a, te na Linux-u koji nema GUI, pokrenite `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP sinkronizacija vremena

Firmware DAQ-E v1.2.0+ sudjeluje u IEEE 1588 PTPv2 kao običan (samo-slave) sat. Backend hosta Chloros je PTP grandmaster — svaki DAQ-E i svaka LATTICE kamera na LAN-u mu je podređen u domeni 0, održavajući sva vremenska žiga uređaja unutar tolerancije od ~1 ms. Taj zajednički sat omogućuje usklađivanje vremenskih oznaka DAQ očitanja s ekspozicijama kamere (vidi [Snimanje i format .daq](recording.md)).

Provjerite sinkronizaciju iz CLI-a:

| Naredba | Prikazuje |
| --- | --- |
| `chloros-cli time-sync status` | Stanje grandmastera hosta, prioriteti BMCA, identitet sata i `health`: `OK`, ili `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` s `health_detail` koji navodi zašto njegovi paketi ne stižu do mreže |
| `chloros-cli time-sync peers` | Svaki viđeni rob (DAQ-E senzori + LATTICE kamere) |
| `chloros-cli time-sync cameras` | Stanje PTP-a po kameri (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Ponovno pokretanje grandmaster procesa |

U GUI-ju, modalni prozor s postavkama DAQ-E-a prikazuje redak **PTP Sync** u stvarnom vremenu sa trenutnim PTP statusom senzora.**DAQ-E s isključenim PTP-om ostaje isključen.** Jedinica pohranjuje svoju postavku PTP-a (uključeno/isključeno) u vlastitu memoriju, stoga jedinica čiji je PTP isključen ostaje isključena nakon ponovnog povezivanja ili isključenja napajanja, a njezini vremenski žigovi nisu sinkronizirani s PTP-om. `chloros-cli daq pool-list` označava takvu jedinicu (`!! PTP is switched OFF on this unit …`) i ispisuje naredbu koja je ispravlja:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros nikada samostalno ne uključuje jedinicu, jer isključenje može biti namjerno.

Detalji za potrošače sa strogo poravnanjem:

* Svaki streamani datagram nosi polje zastavica; **bit 2 je postavljen na okvirima čiji je vremenski žig PTP-sinkroniziran**. Cjevovodi koji zahtijevaju strogo poravnanje kamere/DAQ-a trebali bi se temeljiti na tom bitu.
* Prije sinkroniziranog snimanja, provjerite da se senzor pojavljuje u `chloros-cli time-sync peers` i da `chloros-cli daq pool-list` ne označava njegov PTP kao isključen. (MAPIRova interna alata za izravno upravljanje hardverom također može uvjetovati snimanje PTP zaključavanjem pomoću zastavice `--wait-ptp` koja čeka do 15 s da senzor prijeđe u stanje SLAVE; ta alata nije dio isporučenog CLIa.)
* Dok je PTP aktivno u načinu sljedbenika (slaving), senzor odbija ručno nametanje taktova ("PTP osigurava takt"). To je namjerno — vjerujte PTP-u.

## Napomene o Linuxu

* **PTP priključci bez root prava.** PTP veže UDP 319/320. `.deb` piše `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), pa ih pozadinska usluga veže pokrećući se kao vaš korisnik. Od verzije 1.3.0 paket ne postavlja sposobnost datoteke (`setcap`) na pozadinskom procesu i nije mu potrebna `libcap2-bin`; nemojte sami dodavati sposobnost jer se u suprotnom raspakani pozadinski proces neće pokrenuti. Ako PTP ne može vezati priključak 319, pogledajte [Instalacija za Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux host:** paket instalira pozadinske usluge, ali ne aktivira nijednu. Za neprekidni PTP (i dostupnost DAQ-a) bez pokretanja naredbe, aktivirajte uslugu kao svoj korisnik:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Bez njega, PTP radi samo dok je neka komponenta pokrenula pozadinsku uslugu — naredba `chloros-cli` ili skripta SDKa. Pogledajte [Linux Installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) za root `chloros-backend.service` i kako se prebaciti s njega.

## Rješavanje problema: "Nije pronađena nijedna DAQ-E jedinica"

| Provjera | Detalj |
| --- | --- |
| Napajanje | Nema LED-a na senzoru — provjerite PoE i indikatore veze na prekidaču/injektoru; pričekajte nekoliko sekundi nakon uključivanja |
| Broadcast domena | Host i senzor na istom L2 segmentu; mDNS se ne usmjerava |
| Vatrozid | Windows: pravilo **Chloros Backend (UDP-In)** instalatera dopušta DAQ-E stream (UDP 5002), mDNS (5353) i PTP (319/320); vatrozid treće strane treba isto. Linux: `.deb` otvara ove priključke u `ufw` / `firewalld` samo ako je vatrozid bio aktivan tijekom instalacije — omogućite ih sami u jednom omogućenom kasnije |
| Host s više mrežnih kartica | Otkrivanje pregledava mDNS s povratnim ARP-om; `--eth-host <ip-or-hostname>` preskače otkrivanje i povezuje se izravno |
| Ponovno skeniranje GUI-ja | Otkrivanje se pokreće samo dok je dijalog za povezivanje otvoren; upotrijebite Osvježi |
