# CLI brzi početak (pool-*)

Ugrađeni upravljački programi `chloros-cli` upravljaju DAQ senzorima putem obitelji naredbi **`daq pool-*`** — tankih HTTP klijenata koji upravljaju senzorom putem trajnog bazena senzora pozadinskog sustava Chloros. Backend upravlja transportom, pa GUI, CLI i skripte SDK dijele jednu aktivnu vezu umjesto da se bore za priključak. Sve što je korisniku potrebno dostupno je putem `pool-*`: povezivanje, streaming, snimanje kalibriranih `.daq` datoteka i zamjena profila kapa.

`pool-*` je podržana DAQ površina u objavljenim izdanjima. `chloros-cli daq --help` popisuje podzapovijedi `pool-*`, a `daq discover` / `daq list` pokreće isti sken kao `pool-discover`. Ako naredba DAQ-a za izravan pristup hardveru nije dostupna, program se zatvara s eksplicitnom pogreškom u kojoj se navodi nedostajući paket i upućuje vas na `pool-*` — ništa ne propada tiho. (`pip install chloros-sdk` također ne pruža naredbe za izravno hardversko snimanje.)

***

## Preduvjeti

* **Backend Chloros mora biti pokrenut** — naredbe `pool-*` su HTTP klijenti, a ne upravljački programi hardvera. Na Windowsu pokrenite desktop aplikaciju Chloros (ona pokreće backend). Na bežičnim uređajima Linux/Jetson omogućite uslugu kao svoj korisnik: `sudo systemctl enable --now chloros-backend@$USER.service` (pogledajte [Linux Installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Prijava na**Chloros+ (plaćeni paket)**: prvo pokrenite `chloros-cli login`. Provjera se provodi na strani poslužitelja — bez prijave, naredbe ne uspijevaju s `401 AUTH_REQUIRED`; na besplatnom (Iron) planu ne uspijevaju s `403 PLAN_UPGRADE_REQUIRED`.
* Naredbe prema zadanim postavkama ciljaju `http://127.0.0.1:5000`; obitelj `daq pool-*` poštuje varijablu okruženja `CHLOROS_BACKEND_URL` ako vaš backend radi na drugom mjestu.

***

## Petminutna sesija

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — otvori senzor u bazenu

| Varijanta | Značenje |
| --- | --- |
| `daq pool-connect` | Pametno otkrivanje: pronađite bilo koji DAQ na ovom računalu. |
| `daq pool-connect --port PORT` | DAQ-U na određenom serijskom sučelju (npr. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M putem BLE-a, automatsko skeniranje MAC-a. |
| `daq pool-connect --mac MAC` | DAQ-M na poznatom BLE MAC-u (podrazumijeva `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E na poznatom nazivu hosta ili IP-u — **pouzdani put**. |
| `daq pool-connect --eth` | DAQ-E s automatskim otkrivanjem (mDNS, s rezervnim ARP-om). Pogledajte upozorenje u nastavku. |

Zastavice za podešavanje, sve su neobavezne:

| Zastavica | Značenje |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ručno vrijeme integracije u milisekundama. |
| `--frame-avg N` / `-f N` | Prosječni broj okvira po izvještavanom spektru. |
| `--no-ae` | Onemogući automatsku ekspoziciju (AE je omogućen prema zadanim postavkama). |
| `--no-stream` | Poveži se bez pokretanja prijenosa (nastavi kasnije s `pool-stream --start`). |
| `--cap-id CAP` | Profil korekcije kapa; zadano za pozadinu je `sunshine_cosine`. Vidi [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Povezivanje DAQ-E uređaja koji je već u skupu pod drugom svojom adresom — svojim hostnameom `daq-e-<id>.local` ili IP adresom — vraća postojeći unos u skupu umjesto da otvara jedinicu drugi put.

{% hint style="warning" %}
**Ograničenje automatskog otkrivanja za `--eth`.** Na hostu s više mrežnih sučelja (više od jednog aktivnog), *prvo* `pool-connect --eth` nakon pokretanja može ostati prazno čak i ako je senzor ispravan — pregledavanje pri otkrivanju može propustiti sučelje senzora dok je ARP predmemorija prazna. Ako `--eth` ne pronađe ništa, pokušajte ponovno ili u potpunosti preskočite otkrivanje pomoću `--eth-host <ip-or-hostname>`, što je pouzdan put na računalima s više mrežnih sučelja. Hostname DAQ-E uređaja je `daq-e-<id>.local` (npr. `daq-e-def330.local`); radi i njegova obična IP adresa.
{% endhint %}

## `pool-discover` — skeniranje bez povezivanja

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Pretražuje senzore koje biste mogli povezati i prikazuje adresu za povezivanje, bez uspostavljanja veze. `--only` prihvaća podskup `usb,ble,eth` odvojen podacima,; `--timeout SEC` postavlja vrijeme skeniranja po transportu (zadano 5 s, ograničeno na 1–20 s). Senzori koji su već otvoreni u skupu se ne navode (povezani DAQ-M prestaje s oglašavanjem) — za njih upotrijebite `pool-list`. U isporučenoj izgradnji, `chloros-cli daq discover` i `daq list` izvršavaju isto skeniranje.

## `pool-list` — pogledajte što je priključeno

Prikazuje svaki senzor u pozadinskom poolu, uključujući `sensor_id` koji je potreban svakoj drugoj naredbi:

| Model | Oblik `sensor_id` | Primjer |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-octetni crtani | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

DAQ-E čiji je PTP isključen (postavka se sprema na jedinici) označen je u `pool-list` pomoću `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` i naredbe kojom se ponovno uključuje — vidi [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — čitanje spektralnih okvira

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vraća najnoviji kadar, ili najnovije `--recent N` kadrove; `--json` emitira mašinski čitljiv izlaz za skriptanje. Okviri su radiometrijski kalibrirani spektralna zračenost (W/m²/nm) na mreži od 135 točaka u rasponu od 340 do 1010 nm, s već primijenjenim profilom poklopca senzora. Za kvantitativne brojke zračenja prosječno uzmite snimke u trajanju od najmanje 15 sekundi — to je karakteristika instrumenta, a ne greška.

## `pool-stream` — pauzirajte ili nastavite prijenos

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — snimanje datoteke `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Zastavica | Zadano | Značenje |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Duljina snimanja u sekundama; `0` znači pokrenuti do izdavanja `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Izlazni direktorij. Kod pozadinskog sustava na ovom računalu, relativna putanja rješava se u odnosu na mapu u kojoj ste pokrenuli naredbu; kod udaljenog pozadinskog sustava putanja se rješava **na računalu na kojem pozadinski sustav radi**. |
| `--device-name NAME` | — | Označitelj pohranjen uz snimku. |
| `--stop` | — | Zaustavite snimanje koje je u tijeku. |
| `--confirm-after SEC` | `20` | Nakon pokretanja, pričekajte najviše ovu duljinu da se napiše prvi redak i glasno upozorite (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) ako nijedan ne stigne. `0` onemogućuje provjeru. |
| `--heartbeat SEC` | `30` | S `--duration`, ispišite broj redaka svakih SEC sekundi i upozorite ako prestane rasti. `0` ga onemogućuje. |

{% hint style="info" %}
Zapisivanje se odvija u pozadini, stoga se datoteka `.daq` pohranjuje na datotečni sustav **backend računala** — prema zadanim postavkama u `~/Documents/DAQ Live View/`, a ne nužno na mjestu gdje ste pokrenuli CLI. Nazivi datoteka uključuju ID senzora i vremenski žig.
{% endhint %}

## `pool-set-cap` — deklarirajte postavljenu kapu

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID kapa odabire tvornički izmjereni profil korekcije koji se primjenjuje na svaki spektar i **mora odgovarati kapi koja je fizički ugrađena na senzor** — ni senzor ni softver ne mogu sami detektirati kapu, a odabir se upisuje u svaku datoteku `.daq`. Zadano je `sunshine_cosine` (svaki DAQ isporučuje se s ugrađenim Sunshine korektorom za kosinus, s namjernim slabljenjem od ~12× — neproglašena promjena korektora pogrešno ispravlja spektre za otprilike taj faktor).

| `--cap-id` | Dostupno na |
| --- | --- |
| `sunshine_cosine` (zadano) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | samo DAQ-U |
| `none` | samo DAQ-E — pogledajte napomenu |

ID poklopca izvan postavljenog za senzor odbijen je pri povezivanju uz jasnu pogrešku. `none` (DAQ-E) znači da je kapa fizički uklonjena — i dalje primjenjuje tvornički profil geometrije za udubljeni stakleni difuzor DAQ-E, stoga nije neaktivna operacija (no-op), a goli DAQ-E je konfiguracija za radnu klupu, a ne podržana konfiguracija za terensku upotrebu. (Goli DAQ-U je doista goli i uopće ne treba profil korekcije; DAQ-M se koristi sa svojim Sunshine kapom.)

## `pool-disconnect` — oslobađanje senzora

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` oslobađa senzor onoga tko ga je otvorio, pa tako oslobađa i senzor koji je skripta s greškom ostavila u bazenu (prvo se dovršava snimanje na njemu). Senzor koji nije u skupu nema što osloboditi: naredba ispisuje `<sensor-id>: already released (not in the pool)` i izlazi `0`. Oslobađanje koje je pozadinski sustav odbio ili nije potvrdio završava `1`.

## `pool-enable-ptp` — ponovno uključivanje PTP-a DAQ-E uređaja

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E jedinica pohranjuje postavku PTP-a (uključeno/isključeno) u vlastitu memoriju, stoga jedinica čiji je PTP isključen ostaje isključena — i njezini vremenski pečati nisu sinkronizirani s PTP-om — sve dok se ponovno ne uključi. `pool-list` označava takvu jedinicu i ispisuje ovu naredbu za nju. Chloros nikada samostalno ne uključuje jedinicu, jer isključenost može biti namjerna. Novost u verziji 1.3.0.

## Postavke uređaja DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` prikazuje firmware senzora (i je li ova verzija Chloros-a isporučena s novijom verzijom), njegovu adresu i način na koji je dobivena, spremnu mrežnu postavku dok se razlikuje od aktivne, streaming pri uključivanju i PTP. `pool-set-network` ponovno pokreće senzor kako bi primijenio promjenu i ponovno ga povezuje s postavkama koje je imao; `--no-reboot` samo je sprema. Statička adresa mora biti na mreži ovog računala, inače je Chloros odbacuje; `--force` to zaobilazi, ali samo za računalo kojem će se promijeniti adresa ili koje će se usmjeriti na nju. Ponovni pokreti se odbijaju dok senzor bilježi. `pool-update-firmware` je **Update to \<version\>** za naredbeni redak</version\> GUI-ja <version\>i način ažuriranja DAQ-E-a na Linuxu, koji nema GUI. Novo u verziji 1.3.0.***

## Sažetak naredbi

| Naredba | Svrha |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otvori senzor u pozadinskom fondu. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skenirajte senzore bez povezivanja (pronađe BLE MAC DAQ-M-a). |
| `daq pool-list` | Prikaži svaki senzor u skupu s njegovim `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N najnovijih kalibriranih spektralnih okvira. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nastavi / pauziraj streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Pokreni / zaustavi snimanje `.daq` (na strani pozadinskog sustava). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamjena profila korekcije kapacitivnosti tijekom rada. |
| `daq pool-disconnect --sensor-id ID [--all]` | Oslobađanje jednog senzora ili svih njih. |
| `daq pool-enable-ptp --sensor-id ID` | Ponovno uključite PTP za DAQ-E čiji je PTP isključen. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, adresa, spremna mrežna postavka, streaming pri uključivanju i PTP DAQ-E-a. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP ili statička adresa; ponovno pokreće i ponovno povezuje senzor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streaming pri uključivanju. |
| `daq pool-reboot --sensor-id ID` | Ponovno pokrenite DAQ-E i ponovno ga povežite. |
| `daq pool-update-firmware --sensor-id ID` | Instalirajte firmware DAQ-E-a ove verzije na stariji senzor. |

***

## Rješavanje problema pri prvom povezivanju DAQ-E uređaja

1. DAQ-E uređaj nema LED indikator statusa — provjerite je li napajanje uključeno pomoću indikatora PoE/povezanja na portu switcha ili injektora i pričekajte nekoliko sekundi nakon uključivanja da se pokrene i poveže s mrežom.
2. Backend računalo mora biti na **istom broadcast domenu** kao i senzor — mDNS ne prolazi kroz usmjerivače.
3. Vatrozid mora dopuštati mDNS (UDP 5353), DAQ-E podatke (UDP 5002) i PTP (UDP 319/320). Instalacijski program za Windows dodaje pravilo za to, a Linux `.deb` otvara ih u `ufw` / `firewalld` koji je bio aktivan u vrijeme instalacije; vatrozid treće strane ili onaj koji se omogući kasnije mora im omogućiti pristup.
4. Još uvijek ništa od `--eth`? Koristite `--eth-host` s imenom domaćina (`daq-e-<id>.local`) ili IP-adresom jedinice, čime se zaobilazi otkrivanje.

***
{% hint style="info" %}
**Savjet za AI asistente.** Svaka stranica ovog priručnika isporučuje se kao sirovi Markdown — dodajte `.md` na malu slovo URL-slug stranice (ova stranica: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); strojno čitljiv indeks je `https://mapir.gitbook.io/chloros/llms.txt`. Za potpunu dokumentaciju na razini zastavica za `chloros-cli daq` i svaku drugu obitelj naredbi, preuzmite [CLI referencu](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python putanja je `chloros_sdk.connect_daq_sensor()` u [SDK referenci](../reference/sdk-reference.md).
{% endhint %}
</version\>
