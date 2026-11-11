# Povezivanje kamera

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p> kartica Kamera prije nego što je išta priključeno</p></figcaption></figure>

Chloros automatski otkriva LATTICE kamere na vezi — s kartice Kamere u GUI-ju, iz `chloros-cli lattice` ili iz Python SDK SDK-a. Niz modela kamere pokreće sve dalje: Chloros rješava profil senzora, raspored pojaseva i tvorničku kalibraciju iz `DeviceUserID` + `DeviceSerialNumber` kamere, stoga **nema ničega što treba konfigurirati za svaku kameru**.

Prije povezivanja provjerite je li mreža domaćina konfigurirana — adresiranje link-local, jumbo okviri i, za nizove, postavke prijemnog spremnika mrežne kartice (NIC). To je postavljanje na strani hardvera i nalazi se u priručniku za LATTICE: [**Postavljanje mreže**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Povezivanje s GUI-ja

Otvorite karticu **Kamere** u bočnoj traci sustava Chloros (kartice za hardver pojavljuju se nakon što se pozadinski sustav potpuno pokrene) ili upotrijebite glavni izbornik → **Poveži se na kameru**. Oba otvaraju dijalog**Povezivanje kamere(a)**.

### Dijalog **Povezivanje kamere(a)**Dijalog skenira mrežu u trenutku otvaranja ("Skeniranje mreže...") i prikazuje sve kamere koje pronađe. Svaki red prikazuje**model**kamere (npr. `LATT-M3M-L41-F550`),**serijski broj**i**IP adresu**.

* **Kliknite na red da biste ga odabrali**(zeleno isticanje). Možete odabrati**više kamera** i povezati ih odjednom — Chloros ih povezuje redom.
* Redovi s oznakom **"Povezano"** već su povezani i ne mogu se ponovno odabrati.
* Redovi s oznakom **"U nizu"** pripadaju trenutno povezanom nizu kamera. Prije nego što upotrijebite tu kameru samostalno, prvo odspojite niz.
* **Poveži** — povezuje odabranu(e) kameru(e); gumb prikazuje broj, npr. "Poveži (3)", kada je odabrano više od jedne.
* **Ponovno skeniranje** — ponovno pokreće otkrivanje.
* **Zatvori** — zatvara dijalog.
* Ako skeniranje završi bez rezultata, dijalog prikazuje **"Nije pronađena nijedna kamera na mreži"** — pogledajte [Rješavanje problema](connecting.md#troubleshooting) u nastavku.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dijalog za povezivanje kamere(a) — ovdje prikazan bez kamera na mreži</p></figcaption></figure>

### Prvo povezivanje: preuzimanje paketa za kalibraciju

**Prvi put** kada se određena kamera poveže na računalo, Chloros preuzima tvornički paket za kalibraciju kamere (oko 3,8 MB) s same kamere putem GigE veze. Dok se to izvršava, u dijaloškom okviru se prikazuje zeleni panel **"Preuzimanje kalibracijskih podataka s kamere"**s naprednom trakom po serijskom broju — očekujte otprilike**70 sekundi** po kameri. S više od šest kamera, redovi napretka se pomiču unutar panela. Paket se kešira na hostu, tako da kasnija povezivanja iste kamere u potpunosti preskaču preuzimanje (i nikada ne prikazuju ploču).

### Analiziraj sustav

Gumb **Analyze System** u dijaloškom okviru ispituje host i mrežu (oznaka prikazuje "Analyzing..." dok je u tijeku) i prikazuje izvješće o dijagnostici:

* **Host** — jezgre CPU-a i RAM; naziv i memorija GPU-a, ili "GPU: None detected".
* **Mrežni sučelja** — naziv svakog NIC-a, brzina veze, MTU (s oznakom "jumbo" gdje je aktivno), stanje (up/down) i je li priključeno na USB bus.
* **Kamere**— serijski broj, model, IP adresa i**na kojem se NIC-u svaka kamera nalazi**.
* **Performanse** — trenutačni naspram idealnog fps-a po kameri za format piksela, s zelenom linijom "Potencijal: moguće poboljšanje za N× puta" kada idealni fps prelazi trenutačni.
* **Upozorenja i numerirane preporuke** — ili "Sustav izgleda dobro za trenutni broj kamera." kada nema ništa za popraviti.

Pokrenite ga kad god se otkrivanje ili streaming ponašaju neočekivano — identificira većinu problema na strani mrežne kartice (pogrešan MTU, kamera na pogrešnom sučelju, ograničenja USB adaptera) bez napuštanja dijaloga.

### Spajanje niza

Za spajanje dviju ili više kamera kao **sinkroniziranog niza**, upotrijebite čarobnjak za spajanje niza (**Spajanje niza kamera**): on vodi kroz odabir glavne/pomoćne kamere (prethodno popunjeno sondom za sinkronizacijski kabel — bilo koja kabelski povezana kamera može biti glavna, i kamera RGB `-FRGB` prethodno je odabrana kada je priključena kabelom), odabir načina prikaza (odvojene naspram kombiniranih pločica) i scenu postavki niza s prikazom uživo postizivih fps i propusnosti kabela prije nego što potvrdite. Radnje čarobnjaka i niza obuhvaćene su u odjeljku o nizovima više kamera ovog priručnika; ekvivalent za CLI je "LATTICE Camera First-Connect Workflow" u [CLI Reference](../reference/cli-reference.md).

## Povezivanje putem

CLI a i SDK a i SDKa i SDK a zahtijeva plaćeni Chloros+ paket i prijavu; to se provodi na strani poslužitelja (`401 AUTH_REQUIRED` ako niste prijavljeni, `403 PLAN_UPGRADE_REQUIRED` na besplatnom paketu).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Potpuni potpisi, opcije i tijekovi rada za snimanje: [CLI referenca](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK reference](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Kako se kalibracija rješava pri povezivanju

Svaka LATTICE kamera nosi svoj tvornički paket kalibracije **na kameri**, a Chloros također provjerava MAPIR oblak kada se kamera poveže:

| Situacija   | Što Chloros koristi                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**najnovija kalibracija objavljena za taj serijski broj** — kopija u oblaku nadjačava kopiju na kameri. Kamera koja je ponovno kalibrirana ili ažurirana pomoću usluge MAPIR stoga se automatski ažurira; nije potrebna nikakva radnja korisnika. |
| **Izvan mreže**|**Paket na kameri**, u izvornom obliku. Potpuno izvanmrežni tijekovi rada i dalje rade; jednostavno ne preuzimaju novije kalibracije dok kamera ne bude jednom spojena na mrežu (ili dok se ne obavi tvorničko ponovno flashanje). |

U trenutku snimanja, koeficijenti koji se zapravo primijene **zapravo se zaključavaju u XMP metapodatke svake slike**. Naknadna nadogradnja kalibracije nikada ne mijenja tiho slike koje ste već snimili — ponovna obrada stare snimke koristi koeficijente zabilježene u njezinom XMP-u, a ne one koji su najnoviji danas.

## Rješavanje problema

* **"Nije pronađena nijedna kamera na mreži"**— provjerite postavke za lokalno povezivanje u [Postavke mreže](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statička IP adresa mrežne kartice hosta `169.254.x.x/16`, kamere na istoj vezi, ne očekuje se DHCP/proxy. Zatim upotrijebite**Analyze System**u dijalogu za povezivanje kako biste provjerili na kojem je mrežnom sučelju (NIC) svaka kamera vidljiva (ili nije).**Ponovno skenirajte** nakon bilo kakve promjene kabela ili mrežnog sučelja.
* **Sustav koji je prethodno radio odbija se povezati** (gateways na panelu nizova s `FRAMES WILL DROP` / `Reduce ROI to enable`) — ažuriranje upravljačkog programa mrežne kartice (NIC) tiho je resetiralo postavke prstena za primanje. Ponovno ih primijenite ili pokrenite naredbu `chloros-cli lattice network --fix` u terminalu s podignutim ovlastima; pogledajte [Postavljanje mreže](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera prikazuje "In Array"** — pripada povezanoj sesiji niza. Odspojite niz kako biste koristili kameru samostalno.
