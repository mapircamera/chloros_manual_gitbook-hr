# Snimanje i format .daq

Datoteka `.daq` je format snimanja senzora svjetlosti tvrtke **Chloros**: **SQLite baza podataka** kalibriranih spektralnih sličica s jednog DAQ senzora. Zabilježite jednu tijekom sesije snimanja i cjevovod za reflektanciju može kasnije podijeliti svaku sliku s iradijancom odozgo izmjerenom u tom točnom trenutku.

## Što sadrži .daq

| Svojstvo | Vrijednost |
| --- | --- |
| Kontejner | SQLite baza podataka, jedna datoteka po senzoru po snimanju |
| Naziv datoteke | Sadrži **ID senzora**i**vremensku oznaku**, npr. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektralni odziv po kadru | 135 točaka, 340–1010 nm u koracima od 5 nm, plus CIE XYZ tristimulans |
| Jedinice | Kalibrirana spektralna zračenost, **W/m²/nm** (primijenjen tvornički paket kalibracije + profil poklopca) |
| Zapečaćeni metapodaci | ID senzora (ključ za dohvaćanje tvorničke kalibracije te jedinice) i profil kapa koji je na snazi — vidi [Profili kapa i kalibrirani raspon](caps-and-range.md) |

Format je identičan za DAQ-U, DAQ-M, DAQ-E i DAQ-E-S, tako da pri naknadnoj obradi nije važno koji je uređaj snimio podatke. Snimka DAQ-E-S također pohranjuje nagib (tilt), rotaciju (roll) i uzdignuće (pitch) jedinice uz svaki spektar.

**Snimke preživljavaju nestanak struje (1.3.0).**Snimanje uživo zapisuje se u dnevnik i vrši potpunu sinkronizaciju diska pri svakom zapisu, pa nestanak struje ili prekid procesa rezultira gubitkom otprilike posljednje 3 sekunde, a ne cijele datoteke. Zapisivanje koje je prekinuto usred snimanja može ostaviti datoteku `<name>.daq-journal` pored datoteke `.daq`:**sačuvajte i kopirajte obje datoteke zajedno.** Chloros

vraća dnevnik unatrag prvi put kad pročita datoteku. Zauzet disk odgađa datoteku, nikada očitanja senzora.

**DAQ-E koji prestane isporučivati se ponovno povezuje (1.3.0).** Kada DAQ-E-ov stream utihne (zbog ponovnog uključivanja napajanja njegovog prekidača ili prekinute veze), pozadinski program ga ponovno povezuje s prethodnim postavkama, a snimanje na njemu nastavlja se u istom `.daq` datoteci s označenim prekidom.

Kalibrirano snimanje zahtijeva tvorničku kalibracijsku datoteku senzora. Za DAQ-U i DAQ-M, pozadinski program preuzima datoteku iz oblaka tvrtke MAPIR

prema ID-u senzora (snimanje se odbija ako to ne može); DAQ-E jedinice su izuzete jer pohranjuju svoju kalibraciju na samoj jedinici.

## Snimanje putem GUI-ja

Snimanje u GUI-ju zahtijeva **otvoren projekt** (inak su gumbe za snimanje onemogućeni):

* **Snimaj sve / Zaustavi sve** — na vrhu bočne trake za svjetlosne senzore; istovremeno pokreće ili zaustavlja snimanje na svim priključenim senzorima.
* **Započni snimanje / Zaustavi snimanje** — za svaki senzor, u modalnom prozoru postavki zupčanika. Tijekom snimanja u redovima s informacijama u stvarnom vremenu senzora prikazuje se crveni indikator &quot;REC&quot;.

Datoteke se spremaju u `<project>/light_sensor/`, a kada se snimanje zaustavi — bilo putem naredbe Stop, Stop All ili odspajanjem snimajućeg senzora — dovršeni `.daq` **automatski se dodaje u otvoreni projekt**. Pojavljuje se u popisu datoteka projekta bez ručnog dodavanja, već spremna za obradu reflektancije.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Snimanje putem CLI

-a

CLI

snima putem pozadinskog spremnika senzora (backend mora biti pokrenut — ove su naredbe tanki HTTP

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
| `--duration` | `0` — snimanje dok `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` na datotečnom sustavu **backenda**, a ne CLI

a |

Razlika u direktoriju za izlaz važna je kada CLI

cilja backend na drugom računalu: datoteka se smješta tamo gdje backend radi. Kada se backend nalazi na istom računalu, relativna `-o` (kao što je `./out` gore) rješava se u odnosu na mapu u kojoj ste pokrenuli naredbu.

## Snimanje iz Python

`DAQSensorSession` (koji vraća `chloros_sdk.connect_daq_sensor()`) otkriva istu zajedničku snimku: `record_start(output_dir=None, device_name=None)` vraća putanju datoteke, `record_stop()` vraća `{path, rows}`. Pogledajte [Referencu za SDK

](../reference/sdk-reference.md) za cjeloviti API

za sesije. SDK

-ove klase za izravnu hardversku kontrolu (samo za desktop instalacije) prema zadanim postavkama zapisuju snimke u `~/Documents/DAQ/`; za objavljene verzije podržana je putanja za zajednički zapis gore navedena.

## Korištenje .daq datoteke pri obradi

Za izračun refleksije iz slika, Chloros

u je potrebna zračna emisija prema dolje usklađena s svakom ekspozicijom:

* **Sačuvajte `.daq` zajedno sa slikama.**Prilikom obrade, pipeline automatski pronalazi**downwelling podudaran po vremenu** iz snimljenog `.daq` (bilo kojeg DAQ modela) — ili iz izvornog `.csv` DAQ-M uređaja — pronađen uz slike. Snimke GUI-ja to automatski zadovoljavaju, budući da se dodaju u projekt u trenutku kada prestanu.
* **Kalibracija se preuzima na zahtjev.** Ako tvornička kalibracijska datoteka za pojedinu kameru ili DAQ već nije pohranjena lokalno, Chloros

je automatski preuzima s MAPIR

ova oblaka pri prvom korištenju (internet je potreban samo jednom; pohranjuje se pod `~/.chloros/`).
* **Prilikaz uživo bilježi vlastitu pomoćnu datoteku.** Za svaki kadar reflektancije snimljen uživo, stvarno korišteno DAQ očitanje sprema se kao datoteka `.daq` u mapu snimki `daq/`, tako da se snimka može ponovno obraditi kasnije bez originalne snimke.
* **Snimke s snimača `chloros_scripts` (1.3.0).**`.daq` zapis koji su snimili javni [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) rekorder naveden je kao model DAQ-a koji ga je napravio, a njegova se kapa može postaviti u**Postavke projekta → DAQ senzor svjetla** kao i kod bilo kojeg drugog snimanja. Ti snimači zapisuju sirove brojeve senzora; Chloros

preuzima tvorničku kalibraciju senzora putem serijskog broja i primjenjuje je.
* **Čije je očitanje (1.3.0).**Pokraj svakog snimanja `.daq`, `.daq.cameras.json` imenuje kameru(e) čije okvire opisuje to očitanje, a obrada ga podudara**samo s slikama s tih kamera**. Samostalna kamera bez vlastitog senzora svjetla stoga više ne posuđuje očitanje druge kamere — pripojite joj senzor ili snimite jedan. Snimka napravljena pomoću samog DAQ-a (GUI Record, `daq pool-record`) i svaka `.daq` bez te datoteke (uključujući starije snimke) i dalje služi svim kamerama.

## Povrat zračenja

Obrada projekta također izvozi svako snimanje senzora svjetla koje sadrži, u
`Light Sensor/` mapu pored slikovnih proizvoda. Za to **nije** potrebna slika: snimanje senzora svjetla samo po sebi je potpuna snimka, a mapa koja sadrži samo `.daq` datoteke valjan je ulaz. Projekt koji sadrži samo snimke senzora svjetla prikazuje ih i obrađuje i iz aplikacije i iz CLI

a, te ostaju prikazani i nakon izvođenja. Izvođenje izvještava o tome koliko je proizvoda sa senzora svjetla zapisano.

| Proizvod | Što je to |
| --- | --- |
| `<name>_calibrated.daq` | Arhiva koja se može ponovno obraditi u istoj shemi kao i snimka uživo, a sada deklarira i paket kalibracije koji ju je proizveo. Ponovni uvoz **ne** kalibrira je drugi put. |
| `<name>_calibrated.csv` | Spektralna iradijanca u W/m²/nm na vlastitoj mreži valnih duljina senzora, jedan redak po očitanju, plus fotometrijske stupce: ukupna snaga, fotopski i skotopski luks, PPFD s podjelom na plavo/zeleno/crveno i vršna valna duljina. |

DAQ-U ili DAQ-M čiji se paket kalibracije ne može dohvatiti — jer ste izvan mreže ili taj senzor nema spremljenu kalibraciju — **preskače se uz navođenje razloga**, nikada se ne sprema kao &quot;kalibrirana&quot; datoteka koja sadrži sirove brojeve. Povežite se na internet i ponovno pokrenite. DAQ-E
ima vlastitu kalibraciju, pa je treba samo kada je jedinica isključena i
ništa nije pohranjeno lokalno.

### DAQ-A: sirovi brojevi i zašto je to pravi odgovor

**DAQ-A** je stariji od sustava paketa kalibracije po serijskom broju i nema paket za dohvat. To nije propust: DAQ-A se kalibrira na terenu u odnosu na refleksijski target, a kalibracija temeljena na targetu zahtijeva samo *relativni* odziv senzora — što su upravo njegovi sirovi brojevi. Chloros

ih danas koristi za kalibraciju.

Stoga se snimanje DAQ-A izvozi, ali pod drugačijim imenom:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, a ne `_calibrated` — drugačiji naziv datoteke umjesto zastavice unutar datoteke, jer tvrdnja mora ostati očuvana ako se datoteka pošalje e-poštom kao običan naziv. Zaglavlje `.csv`
navodi `raw spectral sensor counts (NOT irradiance)` i upozorava da su vrijednosti
usporedive **unutar** datoteke, a ne među senzorima. Stupci koji imaju smisla samo za stvarnu iradijaciju — ukupnu snagu, luks, PPFD — ostavljeni su prazni umjesto da se izračunavaju iz brojeva.

Starija DAQ-A-SD snimanja (šema v1.01 / v1.02) bilježe samo vrijeme pisanja datoteke, a ne vremensku oznaku za svako očitanje. Chloros

neće uskladiti slike s tim — uparivanje kadra s vremenom zapisa bilo bi pogrešno, a da pritom ne izgleda pogrešno — no izvoz ih čita bez problema, a CSV

navodi na kojem je satu.

Za cjelovitu priču o refleksiji — jednostruki senzor s kamerom i dvostruki senzor za ambijent/objekt — pogledajte [Radni tijekovi refleksije](reflectance.md).
