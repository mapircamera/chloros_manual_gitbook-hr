# GUI : Navigacija

Kada prvi put pokrenete Chloros, pokreće se njegov pozadinski procesor. Kada je pozadinski procesor spreman, otkriva se gornja lijeva ikona glavnog izbornika <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, a kartice Kamere i Senzori svjetla otključavaju se u lijevoj bočnoj traci (do tada su zamračene).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Od lijeva prema desno, gornji naslov sadrži:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Glavni izbornik

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Iz glavnog izbornika možete:

* **Novi projekt**— stvoriti novi projekt. Ako imate spremljene predloške projekata, pojavit će se padajući izbornik**Odabir predloška** kako bi novi projekt započeo s postavkama predloška.
* **Otvori projekt**— otvoriti postojeći projekt. Popis uključuje gumb**Otvori mapu projekta** koji otvara mapu projekta u vašem pregledniku datoteka.
* **Dupiliraj projekt** — kopirajte trenutno otvoreni projekt pod novim imenom (predlaže se slobodno ime poput &quot;MyProject (2)&quot;) i otvorite kopiju. _(vidljivo nakon otvaranja projekta)_
* **Dodaj datoteke** — dodaj pojedinačne datoteke slika u trenutni projekt _(vidljivo nakon otvaranja projekta)_
* **Dodaj mapu** — dodaj jednu ili više mapa sa slikama u trenutni projekt _(vidljivo nakon otvaranja projekta)_
* **Pokreni obradu / Zaustavi obradu** — pokrenite ili zaustavite proces obrade slika _(omogućeno nakon dodavanja datoteka)_
* **Poveži se s kamerom** — idite na [karticu Kamere](lattice/) kako biste povezali LATTICE kameru ili niz. Radi bez otvorenog projekta.
* **Poveži se na senzor svjetla** — prelazi na [karticu Senzori svjetla](daq/) za povezivanje DAQ senzora svjetla. Radi bez otvorenog projekta.

{% hint style="info" %}
**Samo za Windows

:** Desktop GUI aplikacije Chloros

dostupan je na Windows

. Korisnici Linux

trebali bi pogledati dokumentaciju za [CLI

](CLI.md) i [Python

SDK

SDK](api-python-sdk.md) za obradu bez grafičkog sučelja.
{% endhint %}

### Gumb za reprodukciju/pokretanje <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Kada je omogućen, gumb za pokretanje obrade pokreće cjevovod za obradu slike.

### Traka napretka <img src=".gitbook/assets/image (4).png" alt="" data-size="line"><img src=".gitbook/assets/image (5).png" alt="" data-size="line">



U besplatnom načinu rada Chloros

, koji obrađuje sve datoteke sekvencijalno, traka napretka prikazat će 2 faze: Otkrivanje cilja i Obrada.

U plaćenom licenciranom načinu rada Chloros

+, koji obrađuje sve datoteke istovremeno, traka napretka prikazuje 4 faze: Otkrivanje, Analiziranje, Kalibriranje, Izvoz. Ako pređete mišem preko trake napretka programa Chloros

+, prikazat će se prošireni panel s 4 trake napretka kako biste mogli pratiti napredak. Klikom na gornju traku napretka zamrznut će se padajući panel, a ponovnim klikom se odmrzava.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Bočni izbornik

Lijevi bočni izbornik sadrži razne ikone za interakciju, ovim redoslijedom odozgo prema dolje:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Postavke projekta](project-settings/project-settings.md)

Jezgra Postavke projekta omogućuje vam prilagodbu globalnih postavki projekta i postavki obrade projekta. Prilagodite ih prije početka obrade datoteka.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> preglednik datoteka

Dodajte datoteke/mape i uklonite datoteke iz projekta. Duplicirane datoteke se ignoriraju. Označite okvir u stupcu odredišta za bilo koju odredišnu sliku, a obrada će tražiti odredišta samo na označenim slikama, čime se znatno ubrzava vrijeme obrade. Koristite prekidač Slika/Metapodaci za prebacivanje između prikaza mreže minijatura odabrane slike i detaljne tablice metapodataka.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Prikaz slike](image-viewer-gui/opening-an-image-full-screen.md)

Kada se klikne na sliku u glavnom pregledniku slika, ona se otvara u punom zaslonu na kartici Preglednik slika.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Preglednik karata](image-viewer-gui/map-markers.md)

Pregledajte svoje slike na interaktivnoj 2D karti prema njihovim GPS koordinatama. Podržava Google Maps i ESRI pružatelje podatkovnih pločica, automatski odabirući najbolju uslugu za vašu lokaciju. Pređite mišem preko oznaka kako biste vidjeli pretpregled minijatura slika.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamere](lattice/)

Povežite se i upravljajte LATTICE kamerama uživo — jednu po jednu ili kao sinkronizirane skupine više kamera. Ova kartica prikazuje pločice s pregledom uživo s nadsvjetlima i histogramima, postavke za svaku kameru i za svaki niz te postavke snimanja kojima se odabire koje kamere i koje vrste izvoza proizvodi naredba &quot;Capture All&quot;. Dostupno kada je pozadinski sustav spreman; pogledajte [odjeljak LATTICE](lattice/) za potpuni vodič.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Senzori svjetla](daq/)

Povežite DAQ senzore svjetla — DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet) — i pogledajte njihove kalibrirane grafikone spektra uživo u W/m²/nm. Odavde možete snimati  datoteke u otvoreni projekt, preimenovati senzore, odabrati profile korekcije poklopca i ažurirati firmware za DAQ-E. Dostupno nakon što je pozadinski sustav spreman; pogledajte [odjeljak DAQ](daq/) za potpuni vodič.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> zapisnik za otklanjanje pogrešaka

Provjerite zapisnik radi ispisaka za otklanjanje pogrešaka kada dođe do problema. Kopirajte/preuzmite zapisnik i pošaljite ga [podršci MAPIR](https://www.mapir.camera/community/contact) radi pomoći.

Novo u verziji 1.3.0: ikona greške na vrhu kartice dnevnika (**Prijavi grešku**) šalje tvrtki MAPIR izvješće s vašim opisom, do 10 slika, dnevnicima programa Chloros i detaljima sustava. Morate biti prijavljeni da biste ga poslali. Pogledajte [Prijavu greške s kartice dnevnika](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prijava korisnika](chloros+-login.md)

Bočna traka za prijavu korisnika omogućuje vam prijavu na vaš račun za Chloros+ kako biste otključali napredne značajke. Također možete pogledati trenutačnu verziju aplikacije, kao i prilagoditi jezik prikazanog teksta u grafičkom korisničkom sučelju (GUI) i naredbenom retku (CLI) aplikacije Chloros.
