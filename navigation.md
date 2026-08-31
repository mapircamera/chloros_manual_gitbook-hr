# GUI : Navigacija

Kada prvi put pokrenete Chloros, pokreće se njegov pozadinski procesor. Kada je pozadinski procesor spreman, otkriva se gornja lijeva ikona glavnog izbornika <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> i otključavaju se kartice Kamere i senzori svjetla u lijevoj bočnoj traci (do tada su zamračene).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Od lijeva prema desno, gornji zaglavlje sadrži:

### Glavni izbornik aplikacije <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Iz glavnog izbornika možete:

* **Novi projekt**— stvoriti novi projekt. Ako imate spremljene predloške projekata, pojavit će se padajući izbornik**Odabir predloška** kako bi novi projekt započeo s postavkama predloška.
* **Otvori projekt**— otvoriti postojeći projekt. Popis uključuje gumb**Otvori mapu projekta** koji otvara mapu projekta u vašem pregledniku datoteka.
* **Dupiliraj projekt** — kopirajte trenutno otvoreni projekt pod novim imenom (predlaže se slobodno ime poput &quot;MyProject (2)&quot;) i otvorite kopiju. _(vidljivo nakon otvaranja projekta)_
* **Dodaj datoteke** — dodaj pojedinačne datoteke slika u trenutni projekt _(vidljivo nakon otvaranja projekta)_
* **Dodaj mapu** — dodaj jednu ili više mapa sa slikama u trenutni projekt _(vidljivo nakon otvaranja projekta)_
* **Pokreni obradu / Zaustavi obradu** — pokrenite ili zaustavite proces obrade slike _(omogućeno nakon dodavanja datoteka)_
* **Poveži se s kamerom** — idite na [karticu Kamere](lattice/) kako biste povezali LATTICE kameru ili niz. Radi bez otvorenog projekta.
* **Poveži se s senzorom svjetla** — prelazi na [karticu Senzori svjetla](daq/) za povezivanje DAQ senzora svjetla. Radi bez otvorenog projekta.

{% hint style="info" %}
**Samo zaWindows**: Desktop GUI zaChloros

dostupan je naWindows

. KorisniciLinux

trebali bi pogledati dokumentaciju za [CLI

](CLI.md) i [Python

SDK

](api-python-sdk.md) za obradu bez zaslona.
{% endhint %}

### Gumb za reprodukciju/pokretanje<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Kada je omogućen, gumb za pokretanje obrade pokreće pipeline za obradu slike.

### Traka napretka za<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

<img src=".gitbook/assets/image (5).png" alt="" data-size="line">



U besplatnom načinu radaChloros

, koji obrađuje sve datoteke sekvencijalno, traka napretka prikazat će 2 faze: Detekcija cilja i Obrada.

U plaćenom načinu rada s licencomChloros

+, koji obrađuje sve datoteke istovremeno, traka napretka prikazuje 4 faze: Otkrivanje, Analiziranje, Kalibriranje, Izvoz. Ako pređete pokazivačem miša preko trake napretkaChloros

+, pojavit će se prošireni panel s 4 trake napretka kako biste mogli pratiti napredak. Klikom na gornju traku napretka fiksirat ćete padajući panel, a ponovnim klikom ga otključat ćete.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Bočni izbornik

Izbornik bočne trake s lijeve strane sadrži razne ikone za interakciju, ovim redoslijedom odozgo prema dolje:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Postavke projekta](project-settings/project-settings.md)

Jezgra &quot;Postavke projekta&quot; omogućuje vam prilagodbu globalnih postavki projekta i postavki obrade projekta. Podesite ih prije nego što počnete obrađivati svoje datoteke.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Preglednik datoteka

Dodajte datoteke/mape i uklonite datoteke iz projekta. Duplicirane datoteke se zanemaruju. Označite okvir u stupcu odredišta za bilo koju odredišnu sliku, a obrada će tražiti odredišta samo na označenim slikama, čime se znatno ubrzava vrijeme obrade. Koristite prekidač Slika/Metapodaci za prebacivanje između prikaza mreže minijatura odabrane slike i detaljne tablice metapodataka.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Prikaz slike](image-viewer-gui/opening-an-image-full-screen.md)

Kada se slika klikne u glavnom pregledniku slika, otvara se u punom zaslonu na kartici Preglednik slika.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Preglednik karata](image-viewer-gui/map-markers.md)

Pregledajte svoje slike na interaktivnoj 2D karti na temelju njihovih GPS koordinata. Podržava Google Maps i ESRI pružatelje karata, automatski odabirući najbolju uslugu za vašu lokaciju. Pređite mišem preko oznaka kako biste vidjeli pretpregled minijatura slika.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamere](lattice/)

Povežite se i upravljajte LATTICE kamerama uživo — jednu po jednu ili kao sinkronizirane skupine više kamera. Ovaj se kartični prikaz sastoji od pločica s pregledom uživo s preklopnim slojevima i histogramima, postavki za svaku kameru i za svaki niz te postavki snimanja kojima se odabire koje kamere i koje vrste izvoza proizvodi naredba &quot;Capture All&quot;. Dostupno je nakon što je pozadinski sustav spreman; pogledajte [odjeljak o LATTICE-u](lattice/) za potpuni vodič.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Senzori svjetla](daq/)

Povežite DAQ senzore svjetla — DAQ-U (USB), DAQ-M (Bluetooth) i DAQ-E (Ethernet) — i pogledajte njihove kalibrirane grafikone spektra uživo u W/m²/nm. Odavde možete snimati datoteke `.daq` u otvoreni projekt, preimenovati senzore, odabrati profile korekcije kapa i ažurirati firmware za DAQ-E. Dostupno je nakon što je pozadinski sustav spreman; pogledajte [odjeljak DAQ](daq/) za potpuni vodič.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> zapisnik za otklanjanje poteškoća

Pregledajte zapisnik za ispisne poruke za otklanjanje poteškoća kada dođe do problema. Kopirajte/preuzmite zapisnik i pošaljite ga [MAPIR podršci](https://www.mapir.camera/community/contact) za pomoć.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prijava korisnika](chloros+-login.md)

Lijeva traka za prijavu korisnika omogućuje vam prijavu na vaš račun Chloros+ za otključavanje naprednih značajki. Također možete vidjeti trenutačnu verziju aplikacije, kao i prilagoditi jezik prikazanog teksta u korisničkom sučelju Chloros i CLI.
