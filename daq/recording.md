# Snimanje i format .daq

Datoteka `.daq` je format snimanja senzora svjetla tvrtkeChloros

: **SQLite baza podataka** kalibriranih spektralnih sličica s jednog DAQ senzora. Zabilježite jedan tijekom sesije snimanja i cjevovod za reflektanciju može kasnije podijeliti svaku sliku s iradijancom odozgo izmjerenom u tom točnom trenutku.

## Što .daq sadrži

| Svojstvo | Vrijednost |
| --- | --- |
| Kontejner | SQLite baza podataka, jedna datoteka po senzoru po snimanju |
| Naziv datoteke | Sadrži **ID senzora**i**vremensku oznaku**, npr. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektralni profil po kadru | 135 točaka, 340–1010 nm u koracima od 5 nm, plus CIE XYZ tristimulans |
| Jedinice | Kalibrirana spektralna zračenost, **W/m²/nm** (primijenjen paket tvorničke kalibracije + profil poklopca) |
| Zapečaćeni metapodaci | ID senzora (ključ za dohvaćanje tvorničke kalibracije te jedinice) i profil kapa koji je na snazi — vidi [Profil kapa i kalibrirani raspon](caps-and-range.md) |

Format je identičan za DAQ-U, DAQ-M i DAQ-E, stoga pri naknadnoj obradi nije važno koji je uređaj snimio podatke.

Kalibrirano snimanje zahtijeva tvornički paket kalibracije senzora. Za DAQ-U i DAQ-M, pozadinski sustav preuzima paket iz oblaka tvrtkeMAPIR

pomoću ID-a senzora (snimanje se odbija ako to ne može); jedinice DAQ-E su izuzete jer pohranjuju svoju kalibraciju na samoj jedinici.

## Snimanje putem GUI-ja

Snimanje u GUI-ju zahtijeva **otvoren projekt** (u suprotnom su gumbe za snimanje onemogućeni):

* **Snimaj sve / Zaustavi sve** — na vrhu bočne trake za senzore svjetla; istovremeno započinje ili zaustavlja snimanje `.daq` na svakom priključenom senzoru.
* **Započni / Zaustavi snimanje** — za svaki senzor, u modalnom prozoru postavki zupčanika. Tijekom snimanja u redovima s informacijama u stvarnom vremenu senzora prikazuje se crveni indikator &quot;REC&quot;.

Datoteke se spremaju u `<project>/light_sensor/`, a kada se snimanje zaustavi — bilo putem naredbe Zaustavi, Zaustavi sve ili odspajanjem snimajućeg senzora — dovršeni `.daq` **automatski se dodaje u otvoreni projekt**. Pojavljuje se na popisu datoteka projekta bez ručnog dodavanja, već spremna za obradu reflektancije.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Snimanje sCLI



CLI

snima putem pozadinskog senzorskog fonda (backend mora biti pokrenut — ovi su naredbe tankiHTTP

klijenti):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Dobijte vrijednost `--sensor-id` iz `chloros-cli daq pool-list`. Dvije zadane postavke koje vrijedi znati:

| Opcija | Zadano |
| --- | --- |
| `--duration` | `0` — zapisujte dok `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` na datotečnom sustavu **backenda**, a neCLI

a |

Razlika u direktoriju za izlaz važna je kadaCLI

cilja na backend na drugom računalu: datoteka se pohranjuje tamo gdje backend radi.

## Snimanje sPython

`DAQSensorSession` (koji vraća `chloros_sdk.connect_daq_sensor()`) izlaže istu zajedničku snimku: `record_start(output_dir=None, device_name=None)` vraća putanju datoteke, `record_stop()` vraća `{path, rows}`. Pogledajte [SDK

reference](../reference/sdk-reference.md) za punu sesijuAPI

. KlasaSDK

(samo za desktop instalacije) prema zadanim postavkama zapisuje snimke u `~/Documents/DAQ/`; za objavljene verzije podržana je putanja navedena iznad.

## Korištenje .daq datoteke tijekom obrade

Za izračun refleksance iz snimki,Chloros

-u je potrebna dolazna zračenost usklađena sa svakom ekspozicijom:

* **Sačuvajte `.daq` sa snimkama.**Tijekom obrade, pipeline automatski rješava**downwelling podudaran po vremenskom žigu** iz snimljenog `.daq` (bilo kojeg DAQ modela) — ili iz nativnog `.csv` DAQ-M — pronađen uz slike. Snimke GUI-ja to automatski zadovoljavaju, budući da se dodaju u projekt u trenutku kada prestanu.
* **Kalibracija se preuzima na zahtjev.** Ako paket tvorničke kalibracije za svaku kameru ili DAQ već nije pohranjen lokalno,Chloros

ga automatski preuzima iz oblaka tvrtkeMAPIR

pri prvom korištenju (potreban je internet samo jednom; pohranjuje se pod `~/.chloros/`).
* **Prijenosi uživo spremaju vlastitu pomoćnu datoteku.** Za svaki kadar reflektancije snimljen uživo, stvarno korišteno DAQ očitanje sprema se kao pomoćna datoteka `.daq` pored slika, tako da se prijenos može ponovno obraditi kasnije bez originalne snimke.

## Povrat radijancije

Obrada projekta također izvozi svako snimanje senzora svjetlosti koje sadrži, u
mapu `Light Sensor/` pored slikovnih proizvoda. Ovo **ne** zahtijeva slike: svjetlosni senzor koji leti samostalno predstavlja potpuno snimanje, a mapa koja sadrži samo datoteke `.daq` validan je ulazni podatak. Izvješće o izvođenju navodi koliko je proizvoda sa svjetlosnog senzora zapisano.

| Proizvod | Što je to |
| --- | --- |
| `<name>_calibrated.daq` | Arhiva podložna ponovnoj obradi u istoj shemi kao i snimka uživo, koja sada navodi paket kalibracije koji ju je proizveo. Ponovna uvoza **ne** kalibrira je drugi put. |
| `<name>_calibrated.csv` | Spektralna zračenost u W/m²/nm na vlastitoj mreži valnih duljina senzora, jedan redak po očitanju, plus fotometrijske stupce: ukupna snaga, fotopski i skotopski luks, PPFD s podjelom na plavo/zeleno/crveno, i vršna valna duljina. |

DAQ-U ili DAQ-M čiji se paket kalibracije ne može dohvatiti — jer ste izvan mreže ili taj senzor nema spremljenu kalibraciju — **preskače se uz navođenje razloga**, nikada se ne sprema kao &quot;kalibrirana&quot; datoteka koja sadrži sirove brojeve. Povežite se na internet i ponovno pokrenite. DAQ-E
ima vlastitu kalibraciju, stoga je ovo potrebno samo kada je jedinica odvojena od mreže i
ništa nije pohranjeno lokalno.

### DAQ-A: sirovi brojevi i zašto je to pravi odgovor

**DAQ-A** predaje se sustavu paketa kalibracije po serijskom broju i nema paket za dohvat. To nije propust: DAQ-A se kalibrira na terenu u odnosu na refleksijski target, a kalibracija temeljena na targetu zahtijeva samo *relativni* odziv senzora — što su upravo njegovi sirovi brojevi.Chloros

se danas kalibrira pomoću njih.

Stoga se snimka DAQ-A izvozi, ali pod drugačijim nazivom:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, a ne `_calibrated` — drugačiji naziv datoteke umjesto oznake unutar datoteke, jer tvrdnja mora ostati očuvana ako se datoteka pošalje e-poštom kao običan naziv. Zaglavlje `.csv`
navodi `raw spectral sensor counts (NOT irradiance)` i upozorava da su vrijednosti
usporedive **unutar** datoteke, a ne među senzorima. Stupci koji imaju smisla samo za stvarnu zračenost — ukupnu snagu, luks, PPFD — ostavljeni su prazni umjesto da se izračunavaju iz brojeva.

Stariji DAQ-A-SD zapisi (šema v1.01 / v1.02) bilježe samo vrijeme pisanja datoteke, a ne vremensku oznaku za svako očitanje.Chloros

neće uskladiti slike s njima — uparivanje kadra s vremenom zapisa bilo bi pogrešno, a da pritom ne izgleda pogrešno — ali izvoz ih čita ispravno, aCSV

navodi na kojem je satu.

Za cjelovitu priču o refleksiji — jednostruki senzor s kamerom i dvostruki senzor za ambijentalno/objektno osvjetljenje — pogledajte [Radni tokovi refleksije](reflectance.md).
