# Povezivanje kamera

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p> kartica Kamera prije nego što je išta povezano</p></figcaption></figure>

Chloros

automatski otkriva LATTICE kamere na vezi — s kartice Kamere u GUI-ju, iz `chloros-cli lattice` ili izPythonSDK

. Stringa modela kamere pokreće sve dalje:Chloros

rješava profil senzora, raspored pojaseva i tvorničku kalibraciju iz `DeviceUserID` + `DeviceSerialNumber` kamere, stoga **nema ničega što treba konfigurirati po kameri**.

Prije povezivanja provjerite je li mreža domaćina konfigurirana — adresiranje link-local, jumbo okviri i, za nizove, postavke prijemnog spremnika mrežne kartice (NIC). To je postavljanje na razini hardvera i nalazi se u LATTICE priručniku: [**Network Setup**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Povezivanje s GUI-ja

Otvorite karticu **Kamere** u bočnoj traci programaChloros

(kartice za hardver pojavljuju se nakon što se pozadinski sustav potpuno pokrene) ili upotrijebite glavni izbornik → **Poveži se na kameru**. Oba otvaraju dijalog**Povezivanje kamere(a)**.

### Dijalog **Povezivanje kamere(a)**Dijalog skenira mrežu u trenutku otvaranja (&quot;Skeniranje mreže...&quot;) i prikazuje sve kamere koje pronađe. Svaki red prikazuje**model**kamere (npr. `LATT-M3M-L41-F550`),**serijski broj**i**IP adresu**.

* **Kliknite na red da biste ga odabrali**(zeleno isticanje). Možete odabrati**više kamera**i povezati ih odjednom — gumb**Poveži** ih povezuje redom.
* Redovi s oznakom **&quot;Povezano&quot;** već su povezani i ne mogu se ponovno odabrati.
* Redovi s oznakom **&quot;U nizu&quot;** pripadaju trenutno povezanom nizu kamera. Prvo odspojite niz kako biste tu kameru koristili samostalno.
* **Poveži** — povezuje odabranu kameru (kamere); gumb prikazuje broj, npr. &quot;Poveži (3)&quot;, kada je odabrano više od jedne.
* **Ponovno skeniranje** — ponovno pokreće otkrivanje.
* **Zatvori** — zatvara dijalog.
* Ako skeniranje završi bez rezultata, dijalog prikazuje **&quot;Nije pronađena nijedna kamera na mreži&quot;** — pogledajte [Rješavanje problema](connecting.md#troubleshooting) u nastavku.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dijalog za povezivanje kamere(a) — ovdje prikazan bez kamera na mreži</p></figcaption></figure>

### Prvo povezivanje: preuzimanje paketa za kalibraciju

**Prvi put** kada se određena kamera poveže na računalo,Chloros

preuzima tvornički paket za kalibraciju kamere (oko 3,8 MB) s same kamere putem GigE veze. Dok se to izvršava, u dijaloškom okviru se prikazuje zeleni panel **&quot;Preuzimanje kalibracijskih podataka s kamere&quot;**s naprednom trakom za svaki serijski broj — očekujte otprilike**70 sekundi** po kameri. Paket se kešira na hostu, tako da kasnija povezivanja iste kamere u potpunosti preskaču preuzimanje (i nikada ne prikazuju taj panel).

### Analiziraj sustav

Gumb **Analiziraj sustav** u dijaloškom okviru ispituje host i mrežu (oznaka prikazuje &quot;Analizira se...&quot; dok je u tijeku) i generira izvješće o dijagnostici:

* **Host** — CPU jezgre i RAM; naziv GPU-a i memorija, ili &quot;GPU: Nije otkriven&quot;.
* **Mrežni sučelja** — naziv svakog NIC-a, brzina veze, MTU (s oznakom &quot;jumbo&quot; gdje je aktivno), stanje (aktivno/neaktivno) i je li priključen na USB sabirnicu.
* **Kamere**— serijski broj, model, IP adresa i**na kojem se NIC-u svaka kamera nalazi**.
* **Performanse** — trenutačni naspram idealnog fps-a po kameri za format piksela, s zelenom linijom &quot;Potencijal: moguće poboljšanje od N× puta&quot; kada idealni fps premašuje trenutačni.
* **Upozorenja i numerirane preporuke** — ili poruka &quot;Sustav izgleda dobro za trenutni broj kamera.&quot; kada nema ništa za popraviti.

Pokrenite ga kad god se otkrivanje ili streaming ponašaju neočekivano — identificira većinu problema na strani mrežne kartice (pogrešna MTU, kamera na pogrešnom sučelju, ograničenja USB adaptera) bez napuštanja dijaloškog okvira.

### Povezivanje niza

Za povezivanje dviju ili više kamera kao **sinkroniziranog niza**, upotrijebite čarobnjak za povezivanje niza (**Poveži niz kamera**): Vodi vas kroz odabir master/slave jedinice (prethodno popunjen GPIO sondom za ožičenje), odabir načina prikaza (odvojene naspram kombiniranih pločica) i scenu postavki niza s prikazom u stvarnom vremenu postignutih fps-a i propusnosti kabela prije konačne potvrde. Čarobnjak i rad sa skupovima obuhvaćeni su u odjeljku o skupovima više kamera ovog priručnika; ekvivalent zaCLI

je &quot;LATTICE Camera First-Connect Workflow&quot; u referentnoj [CLI

](../reference/cli-reference.md).

## Za povezivanje sCLI

iSDK



CLI

iSDK

potreban je plaćeniChloros

+ paket i prijava; to se provodi na strani poslužitelja (`401 AUTH_REQUIRED` ako niste prijavljeni, `403 PLAN_UPGRADE_REQUIRED` na besplatnom paketu).

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

Potpuni potpisi, opcije i tijekovi rada za hvatanje: [CLI

Referenca](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK

Referenca](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Kako se kalibracija rješava pri povezivanju

Svaka LATTICE kamera ima svoj tvornički paket kalibracije **na kameri**, aChloros

također provjerava oblakMAPIR

-a kada se kamera spoji:

| Situacija   | ŠtoChloros

koristi                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**najnovija kalibracija objavljena za taj serijski broj** — kopija u oblaku pobjeđuje kopiju na kameri. Kamera koja je ponovno kalibrirana ili ažurirana pomoćuMAPIR

stoga se automatski ažurira; nije potrebna nikakva radnja korisnika. |
| **Izvan mreže**|**Paket na kameri**, u izvornom obliku. Potpuno izvanmrežni tijekovi rada i dalje rade; jednostavno ne preuzimaju novije kalibracije dok kamera ne bude jednom spojena na mrežu (ili dok se ne obavi tvorničko ponovno flashanje). |

U trenutku snimanja, koeficijenti koji se zapravo primijene **zapravo se zaključavaju u XMP metapodatke svake slike**. Naknadna nadogradnja kalibracije nikada ne mijenja tiho slike koje ste već snimili — ponovna obrada stare snimke koristi koeficijente zabilježene u njezinom XMP-u, a ne one koji su trenutno najnoviji.

## Rješavanje problema

* **&quot;Nije pronađena nijedna kamera na mreži&quot;**— provjerite postavke za lokalno povezivanje u [Postavke mreže](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statička IP adresa mrežne kartice hosta `169.254.x.x/16`, kamere na istoj vezi, ne očekuje se DHCP/proxy. Zatim upotrijebite**Analyze System**u dijaloškom okviru za povezivanje kako biste provjerili na kojem je mrežnom sučelju (NIC) svaka kamera vidljiva (ili nije).**Ponovno skenirajte** nakon bilo kakve promjene kabelaže ili mrežnog sučelja.
* **Ranžman koji je prethodno radio odbija se povezati** (gateways na panelu nizova s `FRAMES WILL DROP` / `Reduce ROI to enable`) — ažuriranje upravljačkog programa za mrežnu karticu tiho je resetiralo postavke prstena za primanje. Ponovno ih primijenite ili pokrenite naredbu `chloros-cli lattice network --fix` u terminalu s podignutim ovlastima; pogledajte [Postavljanje mreže](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera prikazuje &quot;In Array&quot;** — pripada povezanoj sesiji niza. Odspojite niz kako biste koristili kameru samostalno.
