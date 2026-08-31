# GUI : Projekti

Chlorosvam omogućuje stvaranje projekata koje možete ponovno otvoriti u budućnosti. Projekt je obična mapa (unutar vaše mape projekta) koja sadrži:

* `project.json` — postavke projekta, popis datoteka i postavke prikaza
* `cameras.json` — kamere i nizovi povezani dok je projekt bio otvoren, s njihovim postavkama
* `sensors.json` — DAQ senzori svjetla povezani dok je projekt bio otvoren, plus veze kamera↔senzori
* vaši snimci, snimke `.daq` i obrađene izlazne mape

Ne postoji vlasnički format projektne datoteke — mapa i njezine JSON datoteke čine projekt, što također olakšava kopiranje, arhiviranje i pokretanje projekata s [CLI](CLI.md) ili [Python SDK](api-python-sdk.md).

## Novi projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Odaberite &quot;Novi projekt&quot; iz glavnog izbornika i unesite jedinstveni naziv za svoj projekt.

Ako ste spremili neke predloške projekata, ispod polja za naziv pojavit će se padajući izbornik **Odabir predloška** — odabirom jednog pokrećete novi projekt s postavkama tog predloška. Predlošci se spremaju iz [Postavke projekta](project-settings/project-settings.md): unesite naziv u polje &quot;Naziv predloška projekta&quot; i kliknite ikonu spremanja.

## Otvori projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Otvaranje projekta prikazuje sve projekte u vašoj projektnoj mapi, a na dnu se nalazi <strong>op</strong>cija <strong>Otvori projektnu mapu</strong></p></figcaption></figure>

Odaberite &quot;Otvori projekt&quot; da biste vidjeli popis postojećih projekata u mapi projekta. Ako ne postoje projekti, bočni izbornik se neće otvoriti. Na gornjoj fotografiji možete vidjeti neke projekte stvorene pomoću GUI-ja (t1, t2, t3). Projekte DATE\_TIME stvorio je CLI koristeći zadanu shemu imenovanja projekata. Klikom na bilo koje ime projekta otvarate ga.

Klikom na gumb &quot;Open Project Folder&quot; otvara se upravitelj datoteka na putanji projekta. Stazu projekta možete prilagoditi u [Postavkama projekta](project-settings/project-settings.md).

Ako su se bilo koja od izvornika slika projekta premjestila ili izbrisala od posljednjeg otvaranja, Chloros prikazuje dijaloški okvir s točnim popisom datoteka koje nedostaju umjesto da otvori praznu mrežu.

## Duplicirati projekt

Dostupno nakon otvaranja projekta. Odaberite &quot;Duplicirati projekt&quot; kako biste kopirali trenutni projekt pod novim imenom —  predlaže sljedeće slobodno ime (npr. &quot;MyProject (2)&quot;) — i duplikat se odmah otvara.

## Dodaj datoteke

Nakon što se projekt otvori, odaberite &quot;Dodaj datoteke&quot; iz glavnog izbornika kako biste dodali pojedinačne datoteke slika u trenutni projekt. Ova funkcija je slična funkciji dodavanja u pregledniku datoteka, ali je radi praktičnosti izravno dostupna iz glavnog izbornika.

## Dodaj mapu

Nakon što se projekt otvori, odaberite &quot;Dodaj mapu&quot; iz glavnog izbornika kako biste dodali mape s slikama u trenutni projekt. Možete odabrati više mapa odjednom. Duplicirane datoteke se ignoriraju.

## Pokreni / Zaustavi obradu

Nakon što se datoteke dodaju u projekt, opcija &quot;Pokreni obradu&quot; postaje dostupna u glavnom izborniku. To je ista radnja kao i klik na gumb Play/Pokreni u gornjoj zaglavnici. Tijekom obrade stavka izbornika mijenja se u &quot;Zaustavi obradu&quot; kako biste mogli prekinuti tijek obrade.

## Povezivanje na kameru / Povezivanje na senzor svjetla

Na dnu glavnog izbornika nalaze se dva hardverska prečaca, dostupna s otvorenim ili neotvorenim projektom:

* **Povezivanje na kameru** — otvara [karticu Kamere](lattice/) za povezivanje LATTICE kamere ili niza.
* **Poveži se na senzor svjetla** — otvara [karticu Senzori svjetla](daq/) za povezivanje DAQ senzora svjetla.

Povezivanje hardvera dok je projekt otvoren sprema ga u projekt (vidi dolje). Bez projekta, veze su samo za sesiju.

{% hint style="info" %}
Stavke izbornika Dodaj datoteke, Dodaj mapu i Pokreni/Zaustavi obradu vidljive su ili omogućene samo kada je projekt otvoren i datoteke su dodane. Pružaju brzi pristup radnjama koje su također dostupne putem bočne trake i gumbima u zaglavlju preglednika datoteka.
{% endhint %}

## Projekti pamte vašu hardver

Novo u verziji 1.2.0: projekt pohranjuje hardver koji ste povezali dok je otvoren. Kamere i nizovi (s postavkama po kameri, nazivima, bojama i rasporedom mreže) spremaju se u `cameras.json`, a senzori svjetla (s nazivima, bojama i vezama s kamerama) u `sensors.json` — automatski, dok radite.

Kada **ponovno otvorite** projekt, Chloros ne dira hardver odmah. Svaki se polu-povezivač ponovno povezuje prvi put kada posjetite karticu kojoj pripada:

* Otvaranje kartice **Kamere** ponovno povezuje spremljene kamere i nizove i ponovno primjenjuje njihove spremljene postavke.
* Otvaranje kartice **Light Sensors** ponovno povezuje spremljene DAQ senzore.

Na taj način, otvaranje projekta samo za pregledavanje ili izvoz slika nikada ne uključuje kamere u streaming. Ako se spremljeni uređaj ne može pronaći kada se otvori njegova kartica, dijaloški okvir će vam reći koji su uređaji nedostupni kako biste ih mogli ponovno povezati ili ukloniti.

## DAQ snimke i .daq datoteke u projektu

* `.daq` snimke napravljene dok je projekt otvoren (iz kartice Svjetlosni senzori ili tijekom snimanja) **automatski se dodaju u projekt**.
* Uvezeni `.daq` datoteke i sve snimke projekta navedeni su u odjeljku **DAQ senzor svjetla** u [Postavkama projekta](project-settings/project-settings.md), svaka sa svojim profilom korekcije kapice.
* Tijekom obrade, `.daq` datoteke projekta osiguravaju osvjetljenje odozgo za proizvode reflektancije — pogledajte [Formati izlaznih slika](output-image-formats.md).

## Pokretanje spremljenog projekta bez grafičkog sučelja

Spremljeni projekt može se pokrenuti bez grafičkog sučelja:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` radi na putanji mape projekta — pogledajte [CLI reference](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` vraća rukohvat projekta; `connect_all()` povezuje svaku spremljenu kameru i senzor s njihovim spremljenim postavkama — pogledajte [Referencu za SDK](reference/sdk-reference.md).
