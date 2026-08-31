CLI brzi početak (pool-*)

Isporučeni pogoni `chloros-cli` upravljaju DAQ senzorima putem obitelji naredbi **`daq pool-*`** — tankih klijenata HTTP koji upravljaju senzorom putem trajnog pool-a senzora pozadinskog sustava Chloros. Backend upravlja transportom, pa GUI, CLI i skripte SDK dijele jednu live handle umjesto da se bore za port. Sve što korisniku treba dostupno je putem `pool-*`: povezivanje, streaming, snimanje kalibriranih `.daq` datoteka i zamjena profila kapa.

`pool-*` je također **jedina** DAQ površina u objavljenim izdanjima. `chloros-cli daq --help` navodi podzapovijedi `pool-*`, i pozivanje podzapovijedi za izravno hardversko DAQ-oviranje na distribuiranoj verziji završava eksplicitnom pogreškom koja navodi nedostajući paket i upućuje vas na `pool-*` — ništa ne propada tiho. (Zapovijedi za izravno hardversko DAQ-oviranje izvršavaju se samo iz izvlačenja izvora iz MAPIR; `pip install chloros-sdk` ih također ne pruža.)

***

## Preduvjeti

* **Backend Chloros mora biti pokrenut** — naredbe `pool-*` su klijenti HTTP, a ne upravljački programi hardvera. Na Windows-u pokrenite desktop aplikaciju Chloros (ona pokreće backend). Na uređajima bez zaslona (headless) Linux/Jetson omogućite uslugu: `sudo systemctl enable --now chloros-backend.service`.
* **Prijava na Chloros+ (plaćeni paket)**: prvo pokrenite `chloros-cli login`. Provjera se provodi na strani poslužitelja — bez prijave, naredbe ne uspijevaju s `401 AUTH_REQUIRED`; na besplatnom (Iron) planu ne uspijevaju s `403 PLAN_UPGRADE_REQUIRED`.
* Naredbe prema zadanim postavkama ciljaju `http://127.0.0.1:5000`; obitelj `daq pool-*` poštuje varijablu okruženja `CHLOROS_BACKEND_URL` ako vaš backend radi na drugom mjestu.

***

## Petominutna sesija

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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
| `daq pool-connect --eth-host HOST` | DAQ-E na poznatom hostnameu ili IP-u — **pouzdani put**. |
| `daq pool-connect --eth` | DAQ-E s automatskim otkrivanjem (mDNS, s rezervnim ARP-om). Pogledajte upozorenje u nastavku. |

Zastavice za podešavanje, sve su neobavezne:

| Zastavica | Značenje |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ručno vrijeme integracije u milisekundama. |
| `--frame-avg N` / `-f N` | Prosječni broj sličica po izvještavanom spektru. |
| `--no-ae` | Onemogući automatsku ekspoziciju (AE je omogućen prema zadanim postavkama). |
| `--no-stream` | Povežite se bez pokretanja prijenosa (nastavite kasnije pomoću `pool-stream --start`). |
| `--cap-id CAP` | Profil korekcije kapa; zadano za pozadinu je `sunshine_cosine`. Vidi [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Upozorenje za automatsko otkrivanje `--eth`.** Na hostu s više mrežnih sučelja (više od jednog aktivnog mrežnog sučela), *prvo* `pool-connect --eth` nakon pokretanja može biti prazno čak i ako je senzor ispravan — pregledavanje pri otkrivanju može propustiti sučelo senzora dok je ARP keš hladan. Ako `--eth` ne pronađe ništa, pokušajte ponovno ili u potpunosti preskočite otkrivanje pomoću `--eth-host <ip-or-hostname>`, što je pouzdan put na računalima s više mrežnih sučelja. Hostname DAQ-E uređaja je `daq-e-<id>.local` (npr. `daq-e-def330.local`); radi i njegova obična IP adresa.
{% endhint %}

## `pool-list` — pogledajte što je priključeno

Prikazuje svaki senzor u skupu pozadinskih sustava (backend pool), uključujući `sensor_id` koji je potreban svakoj drugoj naredbi:

| Model | `sensor_id` oblik | Primjer |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktetski sa crticama | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — očitavanje spektralnih okvira

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vraća najnoviji kadar, ili najnovije `--recent N` kadrove; `--json` emitira mašinski čitljiv izlaz za skriptanje. Okviri su radiometrijski kalibrirani spektralni zračni intenzitet (W/m²/nm) na mreži od 135 točaka u rasponu od 340 do 1010 nm, s već primijenjenim profilom poklopca senzora. Za kvantitativne brojke zračenja prosječno uzmite snimke u trajanju od najmanje 15 sekundi — to je karakteristika instrumenta, a ne greška.

## `pool-stream` — pauziranje ili nastavak prijenosa

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Izlazni direktorij, rješava se **na računalu na kojem se pokreće pozadinski sustav**. |
| `--device-name NAME` | — | Označivač pohranjen uz snimku. |
| `--stop` | — | Zaustavite snimanje koje je u tijeku. |

{% hint style="info" %}
Snimanje se odvija na pozadinskom sustavu (backendu), stoga se datoteka `.daq` pohranjuje na datotečni sustav **pozadinskog računala** — prema zadanim postavkama u `~/Documents/DAQ Live View/`, a ne nužno na mjestu gdje ste pokrenuli CLI. Nazivi datoteka uključuju ID senzora i vremensku oznaku.
{% endhint %}

## `pool-set-cap` — deklaracija postavljenog poklopca

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID kondenzatora odabire tvornički izmjereni profil korekcije koji se primjenjuje na svaki spektar i **mora odgovarati kondenzatoru koji je fizički ugrađen na senzor** — ni senzor ni softver ne mogu sami detektirati kondenzator, a odabir se upisuje u svaku `.daq` datoteku. Zadano je `sunshine_cosine` (svaki DAQ isporučuje se s ugrađenim korektorom Sunshine cosine, s namjernim slabljenjem od ~12× — neproglašena promjena korektora pogrešno ispravlja spektre za otprilike taj faktor).

| `--cap-id` | Dostupno na |
| --- | --- |
| `sunshine_cosine` (zadano) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | samo DAQ-U |
| `none` | samo DAQ-E — pogledajte napomenu |

ID poklopca izvan postavljenog za senzor odbijen je pri povezivanju uz jasnu pogrešku. `none` (DAQ-E) znači da je kap fizički uklonjen — i dalje primjenjuje tvornički profil geometrije za udubljeni stakleni difuzor DAQ-E, stoga nije neaktivna operacija (no-op), a goli DAQ-E je konfiguracija za radnu klupu, a ne podržana terenska konfiguracija. (Goli DAQ-U je doista goli i uopće ne treba profil korekcije; DAQ-M se koristi sa svojim Sunshine poklopcem.)

## `pool-disconnect` — oslobađanje senzora

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Sažetak naredbi

| Naredba | Svrha |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otvori senzor u pozadinskom spremniku. |
| `daq pool-list` | Prikaži svaki senzor u skupu sa svojim `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N najnovijih kalibriranih spektralnih okvira. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nastavi / pauzira streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Pokrene / zaustavi snimanje `.daq` (na strani pozadinskog sustava). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamijenite profil korekcije kapacitivnosti tijekom rada. |
| `daq pool-disconnect --sensor-id ID [--all]` | Oslobodite jedan senzor ili sve njih. |

***

## Rješavanje problema pri prvom povezivanju DAQ-E uređaja

1. DAQ-E nema LED indikator statusa — provjerite je li uređaj napajen putem PoE/link indikatora na portu switcha ili injectora te pričekajte nekoliko sekundi nakon uključivanja da se pokrene i priključi mreži.
2. Backend računalo mora biti na **istom broadcast domenu** kao i senzor — mDNS ne prolazi kroz usmjerivače.
3. Na Windows-u prihvatite upit vatrozida Defender pri prvom pokretanju (mDNS UDP 5353, DAQ-E podaci UDP 5002, PTP UDP 319/320).
4. Još uvijek nema nikakvih podataka od `--eth`? Koristite `--eth-host` s hostnim imenom (`daq-e-<id>.local`) ili IP adresom jedinice — pouzdan put, osobito na hostovima s više mrežnih priključaka.

***{% hint style="info" %}**Savjet za AI asistente.** Svaka stranica ovog priručnika isporučuje se kao sirovi Markdown — dodajte `.md` na mali slug stranice u malim slovima (URL) (ova stranica: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); strojno čitljiv indeks je `https://mapir.gitbook.io/chloros/llms.txt`. Za potpunu dokumentaciju na razini zastavica za `chloros-cli daq` i svaku drugu obitelj naredbi, preuzmite [CLI Referencu](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); putanja Python je `chloros_sdk.connect_daq_sensor()` u [SDK Referenci](../reference/sdk-reference.md).
{% endhint %}
