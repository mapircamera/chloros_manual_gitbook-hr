# Profili kapača i kalibrirani raspon

> Same kapice — koja se kapica isporučuje s kojim senzorom, kako se montiraju i njihovo optičko ponašanje — dokumentirane su u **[DAQ korisničkom priručniku](https://mapir.gitbook.io/daq)**. Ova stranica objašnjava *deklariranje* ugrađene kapice uChloros

, što je ono što ispravlja korekciju.

Fabrčka radiometrijska kalibracija svakog DAQ senzora svjetla opisuje *goli* senzor. Fizički poklopac postavljen na difuzor mijenja svjetlo koje senzor prikuplja, stoga  primjenjuje tvornički izmjereni **profil korekcije poklopca** na paket kalibracije. Deklariranje ispravnog poklopca dio je procesa dobivanja kalibriranih podataka — ova stranica objašnjava koji poklopci postoje za svaki model, kako ih deklarirati i koji je zapravo kalibrirani spektralni raspon senzora.

## Dostupnost kapa po modelu

| Profil kapa (`cap_id`) | Fizička kapa | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Kapa za korekciju sunčevog kosinusa (**standardno na svim modelima**) | Da | Da | Da |
| `fov_15` / `fov_45` / `fov_90` | Koni za ograničavanje vidnog polja (FOV) (15° / 45° / 90°) | Da | — | Da |
| `fov_30` / `fov_60` | Koni za ograničavanje vidnog polja (FOV) (30° / 60°) | Da | — | — |
| `none` | Nema postavljenog poklopca | — | — | Da |

Napomene specifične za model:

* **DAQ-M ima jedan profil poklopca: `sunshine_cosine`.** Bare-plus-Sunshine-cap je definicija proizvoda, a goli DAQ-M ne treba geometrijski profil.
* **Goli DAQ-U je doista goli** — uopće ne treba geometrijski profil, zbog čega ne postoji profil `none` za njega.
* **`none` na DAQ-E NIJE neaktivna operacija.** Uvučeni difuzor DAQ-E-a prekriven staklom ima vlastitu stvarnu geometrijsku korekciju, stoga je &quot;bez kapa&quot; sam po sebi izmjereni profil na ovom modelu.
* **Goli DAQ-E ne može mjeriti izravno sunce na bilo kojoj visini** — kapa Sunshine je konfiguracija za terenski rad. Ne planirajte rad na otvorenom s golim DAQ-E uređajem.

U postavkama svakog senzora u GUI-ju (ikona zupčanika na kartici Senzori svjetla), padajući izbornik **Cap** također nudi &quot;None (bare sensor)&quot; na uređajima DAQ-U i DAQ-M — na ta dva modela &quot;golo&quot; jednostavno znači da se ne primjenjuje korekcija kapice, u skladu s napomenama iznad. Odaberite ga samo kada je kapa fizički uklonjena.

## Deklariranje kape — i zašto je to važno

**Deklarirani `cap_id` mora odgovarati kapi koja je fizički postavljena na senzor.** Ni senzor ni softver ne mogu detektirati postavljenu kapicu. Deklaracija pokreće dvije stvari:

1. **Korekciju uživo** koja se primjenjuje na svaki spektar.
2. **otisak kapice**upisan u svako snimanje**`.daq`**, kojemu kasnija obrada reflektancije vjeruje.

Sunshine kapica po dizajnu prigušuje za otprilike **12×**, stoga snimanje s pogrešno deklariranom kapicom pogrešno skalira spektre za otprilike taj faktor. Promjene kapica odmah deklarirajte.

### Postavljanje kapice

GUI: kartica Light Sensors → ikona zupčanika u retku senzora → padajući izbornik **Cap**. Zadana vrijednost za svaki model je `sunshine_cosine` (svi DAQ senzori isporučuju se s ugrađenim korektorom kosinusa), a odabir se zadržava s projektom.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

(backend mora biti pokrenut):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI

sintaktički prihvaća cijelu listu `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); svaki se profil provjerava u odnosu na model senzora pri povezivanju, pa neispravan ID kapa (na primjer, E-only ID na DAQ-U) dovodi do jasne pogreške umjesto do pogrešnog ispravljanja. Zadana vrijednost backenda kada se ništa ne proslijedi je `sunshine_cosine`.

Python

SDK

napomena: `cap_id` **nije** kotačić zaSDK

— `connect_daq_sensor()` / `DAQSensorSession` ne izlažu nijedan parametar kapa. Odaberite ograničenje pomoću gornjih naredbiCLI

ili iz padajućeg izbornika GUI-ja; pogledajte [Referencu zaSDK

](../reference/sdk-reference.md).

Napredno: profili se isporučuju unutar instalacijeChloros

na `daq/cap_profiles/<u|m|e>/<cap_id>.json` i mogu se nadjačati po korisniku na `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Neovisno o kapama, senzori koji nikada nisu ponovno kalibrirani automatski primaju malu korekciju tamne kompenzacije dobivenu iz flote — bez potrebe za korisničkim djelovanjem.

## Performanse kape za sunčano vrijeme (konfiguracija na otvorenom)

Brojke na kojima možete temeljiti postupke:

| Svojstvo | Vrijednost |
| --- | --- |
| Polje vidnog polja | 180° hemisferično |
| Pogreška kosinusnog odziva | ≤ ±4 % do 60° upada; ≤ ±4,5 % do 70° |
| Gornja granica za nisko sunce | Ne preporučuje se ispod ~15° visine sunca |
| Prigušenje | ~12× (po dizajnu) |
| Ponovljivost ponovnog postavljanja kapa | ≈ 1,5 % |
| Kvantitativna iradijacija | Prosjek očitanja od **≥ 15 s** (karakteristika instrumenta, a ne greška) |

Za bilo koju kvantitativnu vrijednost iradijacije — uključujući i referentne odraze — koristite prosjek očitanja od najmanje 15 sekundi, a ne pojedinačni kadar.

## Kalibrirani spektralni raspon

| Svojstvo | Vrijednost |
| --- | --- |
| Spektralno uzorkovanje | 340–1010 nm u koracima od 5 nm (135 točaka) |
| Radiometrijski kalibrirani raspon | **~374–974 nm** (prisiljeno u softveru) |

Senzor izvještava o punoj mreži od 340–1010 nm, ali radiometrijski pojačanje koje se može pratiti do NIST-a obuhvaća raspon od ~374–974 nm.Chloros

**odbija podjelu po apsolutnoj reflektanciji** za bilo koji spektar kamere s manje od polovice svoje spektralne težine unutar tog raspona, pri čemu izvještava o razlogu preskakanja `dls-uncalibrated-band-<nm>` umjesto da proizvede nekalaribran proizvod. Među isporučivanim SKU-ovima kamera samo F988 filtar izlazi iz tog raspona; umjesto toga koristi tijek rada s panelom za reflektanciju — vidi [Tijekovi rada s reflektancijom](reflectance.md).

Za modele senzora, transportne jedinice i ID-ove senzora pogledajte [pregled DAQ-a](README.md). Za to kako se oznaka poklopca koristi tijekom obrade pogledajte [Zapisivanje i format .daq](recording.md).
