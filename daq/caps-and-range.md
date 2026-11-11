# Profili kapača i kalibrirani raspon

> Same kapice — koja se kapica isporučuje s kojim senzorom, kako se montiraju i njihovo optičko ponašanje — dokumentirane su u **[DAQ korisničkom priručniku](https://mapir.gitbook.io/daq)**. Ova stranica obuhvaća *prijavu* ugrađene kapice tvrtki Chloros
, što je ono što ispravlja korekciju.

 tvornička radiometrijska kalibracija svakog DAQ senzora svjetla opisuje *goli* senzor. Fizički poklopac postavljen preko difuzora mijenja svjetlost koju senzor prikuplja, stoga Chloros
primjenjuje tvornički izmjereni **profil korekcije poklopca** na paket kalibracije. Deklariranje ispravnog poklopca dio je procesa dobivanja kalibriranih podataka — ova stranica objašnjava koji poklopci postoje za svaki model, kako ih deklarirati i koji je zapravo kalibrirani spektralni raspon senzora.

## Dostupnost kapa po modelu

| Profil kapa (`cap_id`) | Fizička kapa | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Kapa za korekciju sunčeve svjetlosti (uglavnom ugrađena na svim modelima) | Da | Da | Da |
| `fov_15` / `fov_45` / `fov_90` | Koni za ograničavanje vidnog polja (FOV) (15° / 45° / 90°) | Da | — | Da |
| `fov_30` / `fov_60` | Koni za ograničavanje vidnog polja (FOV) (30° / 60°) | Da | — | — |
| `none` | Nema postavljenog poklopca | — | — | Da |

Napomene specifične za model:

* **DAQ-M ima jedan profil poklopca: `sunshine_cosine`.** Bare-plus-Sunshine-cap je definicija proizvoda, a goli DAQ-M ne treba geometrijski profil.
* **Goli DAQ-U je doista goli** — uopće ne treba geometrijski profil, zbog čega ne postoji profil `none` za njega.
* **`none` na DAQ-E NIJE neaktivna operacija.** Uvučeni difuzor DAQ-E-a prekriven staklom ima vlastitu stvarnu geometrijsku korekciju, stoga je &quot;bez kapa&quot; sama po sebi izmjereni profil na ovom modelu.
* **Goli DAQ-E ne može mjeriti izravno sunce na bilo kojoj elevaciji** — kapa Sunshine je konfiguracija za terenski rad. Ne planirajte rad na otvorenom s golim DAQ-E uređajem.

U postavkama svakog senzora u GUI-ju (ikona zupčanika na kartici Senzori svjetla), padajući izbornik **Cap** također nudi &quot;None (bare sensor)&quot; na uređajima DAQ-U i DAQ-M — na ta dva modela &quot;golo&quot; jednostavno znači da se ne primjenjuje korekcija kapice, sukladno napomenama iznad. Odaberite ga samo kada je kapa fizički uklonjena.

## Deklariranje kape — i zašto je to važno

**Deklarirani `cap_id` mora odgovarati kapi koja je fizički postavljena na senzor.** Ni senzor ni softver ne mogu detektirati postavljenu kapu. Deklaracija pokreće dvije stvari:

1. **Korekciju uživo** koja se primjenjuje na svaki spektar.
2. **pečat kape** upisan u svako snimanje s `.daq`-om, kojemu naknadna obrada reflektancije vjeruje.

Sunshine kapa po dizajnu prigušuje otprilike **12×**, stoga snimanje s neispravno deklariranom kapom pogrešno skalira spektre za otprilike taj faktor. Promjene kapa odmah deklarirajte.

### Postavljanje kape

GUI: kartica Light Sensors → ikona zupčanika u retku senzora → padajući izbornik **Cap**. Zadana vrijednost za svaki model je `sunshine_cosine`, što je pretpostavka (uređaji DAQ-U, DAQ-M i DAQ-E isporučuju se bez kapa, a snimanje pretpostavlja da je kapa ugrađena dok ne odaberete jednu), a odabir ostaje sačuvan s projektom.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend mora biti pokrenut):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
sintaktički prihvaća cijelu listu `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); svaki se profil provjerava u odnosu na model senzora pri povezivanju, pa neispravan ID kapa (na primjer, E-only ID na DAQ-U) rezultira jasnom pogreškom umjesto ispravljanja pogreškom. Zadana vrijednost backenda kada se ništa ne proslijedi je `sunshine_cosine`.

Python
SDK
: nazovite kap prilikom povezivanja pomoću `connect_daq_sensor(cap_id="sunshine_cosine")` ili je zamijenite tijekom izvođenja pomoću `DAQSensorSession.set_cap("fov_45")`; pogledajte [SDK
reference](../reference/sdk-reference.md#daq-sensor-sessions).

Napredno: profili se isporučuju unutar instalacije programa Chloros
na `daq/cap_profiles/<u|m|e>/<cap_id>.json` i mogu se nadjačati za svakog korisnika na `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Neovisno o kondenzatorima, senzori koji nikada nisu ponovno kalibrirani automatski primaju malu korekciju tamne struje izbačenu iz flote — bez potrebe za korisničkim djelovanjem.

## DAQ-E-S: fiksni difuzor i nagib

**DAQ-E-S**je DAQ-E s dvije razlike koje su važne za**Chloros
**. Sve ostalo (otkrivanje, povezivanje, PTP, snimanje, postavke uređaja) radi točno kao i kod DAQ-E.

* **Njegov difuzor je trajan, stoga ne zahtijeva korekciju kapa.** Difuzor je bio na jedinici kada je mjeren njezina tvornička kalibracija, pa ta kalibracija već bilježi stvarnu zračenost prema dolje. Chloros
na njega ne primjenjuje nikakav profil zaklona bez obzira na zahtjev: zaklon nazvan za njega se ignorira uz poruku, njegovi zapisi navode da je zaklon došao iz modela i upozorenje o pretpostavljenom zaklonu za njega se nikada ne aktivira. U postavkama projekta → DAQ senzor svjetla u njegovom redu ne prikazuje se kapa, a nadjačavanje kape na razini cijelog projekta ne dotiče ga. Ne primjenjujte kap na njegove podatke ručno: korekcija Sunčevog svjetla na vrhu pokazala bi vrijednost oko 11× veću.
* **Sadrži troosni akcelerometar.** Odziv difuzora na kosinus vrijedi samo dok je ravan, pa nagnuti senzor izvještava o pogrešnoj zračenosti bez drugih simptoma. Svako DAQ-E-S očitanje stoga nosi podatke o nagibu (tilt), rotaciji (roll) i usmjerenju (pitch), a `.daq` snimljen s njega pohranjuje ih uz svaki spektar (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), tako da možete razlikovati oblak od oštećene postaje.

Za svako polje položaja i način na koji se greška u toku položaja prijavljuje, pogledajte [DAQ-E-S u referenciji CLI
](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Performanse sunčane kapice (konfiguracija na otvorenom)

Brojke na kojima možete temeljiti postupke:

| Svojstvo | Vrijednost |
| --- | --- |
| Polje vidnog polja | 180° hemisferično |
| Pogreška kosinusnog odziva | ≤ ±4 % do 60° upada; ≤ ±4,5 % do 70° |
| Gornja granica za nisko sunce | Ne preporučuje se ispod ~15° visine sunca |
| Prigušenje | ~12× (po dizajnu) |
| Ponovljivost ponovnog postavljanja poklopca | ≈ 1,5 % |
| Kvantitativna iradijacija | Prosjek očitavanja od **≥ 15 s** (karakteristika instrumenta, a ne greška) |

Za bilo koju kvantitativnu vrijednost iradijacije — uključujući i referentne odraze — koristite prosjek očitavanja od najmanje 15 sekundi, a ne pojedinačni kadar.

## Kalibrirani spektralni raspon

| Svojstvo | Vrijednost |
| --- | --- |
| Spektralno uzorkovanje | 340–1010 nm u koracima od 5 nm (135 točaka) |
| Radiometrijski kalibrirani raspon | **~374–974 nm** (prisilno u softveru) |

Senzor izvještava o punoj mreži od 340–1010 nm, ali radiometrijski pojačanje koje se može pratiti do NIST-a obuhvaća raspon od ~374–974 nm. Chloros
**odbija podjelu na apsolutnu reflektanciju** za bilo koji spektar kamere s manje od polovice svoje spektralne težine unutar tog raspona, pri čemu izvještava o razlogu preskakanja `dls-uncalibrated-band-<nm>` umjesto da proizvede nekalarirani proizvod. Među isporučivanim SKU-ovima za kamere, samo se filtar F988 nalazi izvan tog raspona; on umjesto toga koristi tijek rada s panelom za odražavanje — pogledajte [Tijekovi rada s odražavanjem](reflectance.md).

Za modele senzora, transportne jedinice i ID-ove senzora pogledajte [pregled DAQ-a](README.md). Za to kako se oznaka na poklopcu koristi tijekom obrade pogledajte [Zapisivanje i format .daq](recording.md).
