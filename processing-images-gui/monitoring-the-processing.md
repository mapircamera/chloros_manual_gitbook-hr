# Praćenje obrade

Nakon što je obrada započela, uslugaChloros

pruža nekoliko načina za praćenje napretka, provjeru problema i razumijevanje što se događa s vašim skupom podataka. Ova stranica objašnjava kako pratiti obradu i tumačiti informacije koje pružaChloros

.

## Pregled trake napretka

Traka napretka u gornjoj zaglavnici prikazuje status obrade u stvarnom vremenu i postotak dovršenosti. Napredak se prenosi uživo s backenda putem događaja poslanih od strane poslužitelja (Server-Sent Events, SSE), stoga traka odražava ono što cijev za obradu (pipeline) zapravo radi.

### Traka napretka u slobodnom načinu rada

Za korisnike bez licenceChloros

+:

**Prikaz napretka u 2 faze:**

1.**Detekcija ciljeva** - Pronalaženje kalibracijskih ciljeva na slikama
2. **Obrada** - Primjena ispravaka i izvoz**Traka napretka prikazuje:**

* Ukupni postotak dovršenosti (0-100%)
* Naziv trenutne faze
* Jednostavna vizualizacija horizontalne trake

### Traka napretka zaChloros

+

Za korisnike s licencomChloros

+:

**Prikaz napretka u 4 faze:**

1.**Detekcija** - Pronalaženje kalibracijskih ciljeva
2. **Analiza** - Pregledavanje slika i priprema cjevovoda
3. **Kalibracija** - Primjena korekcija vinjete i reflektancije
4. **Izvoz** - Pohrana obrađenih datoteka**Interaktivne značajke:*** **Pređite** mišem preko trake napretka da biste vidjeli prošireni panel s 4 faze
* **Kliknite** na traku napretka kako biste zamrznuli/prikačili prošireni panel
* **Ponovno kliknite** kako biste otključali i automatski sakrili panel kada miš napusti područje
* Svaka faza prikazuje pojedinačni napredak (0-100%)

{% hint style="info" %}
**ParitetCLI

a**: tijekom izvođenja `chloros-cli process` ista četiri niti izvještavaju o Detektiranju, Analiziranju, Obrađivanju i Izvozu, a `chloros-cli export-status` prikazuje trenutačni napredak izvoza Niti-4 iz drugog terminala. Pogledajte [CLI

referencu](../reference/cli-reference.md).
{% endhint %}

***

## Razumijevanje svake faze obrade

{% hint style="info" %}
**Arhitektura cjevovoda**: Ove 4 GUI faze odgovaraju [cjevovodu obrade s 4 niti](../processing-architecture/processing-pipeline.md). Na sustavima s GPU akceleracijom, niti 3 (Kalibracija) koristi se [dinamičkom prilagodbom izračuna](../processing-architecture/dynamic-compute-adaptation.md), što optimizira obradu za vaš specifični hardver.
{% endhint %}

### Faza 1: Detekcija (Detekcija cilja)

**Što se događa:**

*   Chloros

skenira slike koje ste označili pomoću potvrdnog okvira za cilj (sve slike samo kada nijedna nije označena)
*   Algoritmi računalnog vida identificiraju kalibracijske panele
*   Vrijednosti refleksije se izvlače iz svakog panela
*   Vremenski pečati cilja se bilježe radi pravilnog zakazivanja kalibracije

**Trajanje:**

* S označenim ciljevima: 10-60 sekundi
* Bez označenih ciljeva: 5-30+ minuta (skenira sve slike)

**Pokazatelj napretka:**

* Otkrivanje: 0 % → 100 %
* Broj skeniranih slika (broji samo slike koje se zapravo skeniraju)
* Broj pronađenih ciljeva

**Na što obratiti pozornost:**

* Trebalo bi se brzo dovršiti ako su ciljevi ispravno označeni
* Ako traje predugo, ciljevi možda nisu označeni
* Provjerite zapisnik o greškama (Debug Log) na poruke &quot;Cilj pronađen&quot;

### Faza 2: Analiza

**Što se događa:**

* Čitanje EXIF metapodataka slike (vremenski žigovi, postavke ekspozicije)
* Određivanje strategije kalibracije na temelju vremenskih žigova ciljeva i dostupnih DAQ downwelling podataka
* Organiziranje reda za obradu slika
* Priprema radnika za paralelnu obradu (Chloros

+ samo)

**Trajanje:** 5-30 sekundi**Pokazatelj napretka:**

* Analiza: 0 % → 100 %
* Brza faza, obično se brzo završava

**Na što obratiti pozornost:**

* Napredak bi trebao biti stalan bez pauza
* Upozorenja o nedostajućim metapodacima pojavit će se u dnevniku za otklanjanje pogrešaka (Debug Log)

### Faza 3: Kalibracija

**Što se događa:*** **Debayering**: Pretvorba RAW Bayer uzorka u 3 kanala (preskače se za LATTICE mono module, uz napomenu)
* **Korekcija vinjete**: Uklanjanje potamnjivanja rubova objektiva
* **Kalibracija reflektancije**: Normalizacija s ciljanim vrijednostima i/ili DAQ downwelling
* **Izračun indeksa**: Izračunavanje multispektralnih indeksa
* Obrada svake slike kroz cijeli pipeline

**Trajanje:** Većina ukupnog vremena obrade (60-80%)**Pokazatelj napretka:**

* Kalibracija: 0% → 100%
* Trenutna slika se obrađuje
* Obradene slike / Ukupno slika

**Ponašanje pri obradi:*** **Slobodni način rada**: Obradjuje jednu sliku odjednom, sekvencijalno
* **Chloros

+ način rada**: Pokreće hardverski prilagodljivu radnu grupu — 1-4 istovremena radnika na GPU sustavima (ovisno o VRAM-u), jednog radnika po fizičkom jezgri (umanjeno za jedan) na sustavima s isključivo CPU-om. Pogledajte [Dinamičku prilagodbu izračuna](../processing-architecture/dynamic-compute-adaptation.md)
* **Acceleracija GPU-om**: Značajno ubrzava ovu fazu**Na što obratiti pozornost:**

* Stalan napredak kroz broj slika
* Provjerite zapisnik za otklanjanje pogrešaka (Debug Log) radi poruka o dovršetku svake slike
* Upozorenja o kvaliteti slike ili problemima s kalibracijom

### Faza 4: Izvoz

**Što se događa:**

* Zapisivanje obrađenih slika na disk u odabranom formatu, kako se dovršavaju
* **LATTICE**: svaki kadar se raspoređuje u svaki omogućen proizvod (debayered / pretpregled / radijansa / reflektancije)
* Izvoz višespektralnih indeksnih slika s LUT bojama
* Stvaranje izlaznog stabla `<project>/<camera>/<format>/<Product>_Images/` — izvezene datoteke zadržavaju izvorno ime datoteke; mapa identificira proizvod

**Trajanje:** 10-20 % ukupnog vremena obrade**Pokazatelj napretka:**

* Izvoz: 0 % → 100 %
* Datoteke se pišu
* Format izvoza i odredište

**Na što obratiti pozornost:**

* Upozorenja o prostoru na disku
* Pogreške pri pisanju datoteka
* Završetak svih konfiguriranih izlaznih datoteka

***

## Kartica Zapisnika za otklanjanje pogrešaka

Zapisnik za otklanjanje pogrešaka pruža detaljne informacije o napretku obrade i svim problemima na koje ste naišli. Poruke o pokretanju pozadinskog sustava također se ponovno prikazuju u zapisničkoj konzoli, tako da zapisnik prikazuje cijelu priču čak i ako ga otvorite kasno.

### Pristup dnevniku za otklanjanje pogrešaka

1. Kliknite ikonu **Debug Log**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

u lijevoj bočnoj traci
2. Otvara se panel dnevnika koji prikazuje poruke o obradi u stvarnom vremenu
3. Automatski se pomiče prema gore kako bi prikazao najnovije poruke

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Razumijevanje poruka u dnevniku

Chloros

retcima dnevnika predstavljeni su oznakama u zagradama koje imenuju podsustav — na primjer `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najvažniji za poznavanje je **sažetak pokretanja**, koji se ispisuje na kraju svakog pokretanja (uključujući i zaustavljena pokretanja):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Dodatne pomoćne linije `[RUN-SUMMARY]` pojavljuju se kad god je nešto potrebno objasniti — na primjer, pokretanje koje nije dalo nikakav rezultat ili kamera čiji je zatraženi proizvod preskočen kao neprimjenjiv. Redak `[EXPORT-CHECK]` objašnjava preskakanje po kamerama (npr. zašto kamera s eRGB

-om nije dobila proizvod radijantnosti).

Opće razine važnosti poruka (primjeri u nastavku su ilustrativni, a ne doslovni):

#### Informativne poruke (bijele/sive)

Uobičajena ažuriranja obrade: obrada započela, ciljevi otkriveni (s brojem panela), napredak kalibracije po slici, datoteke izvezene, obrada dovršena.

#### Upozorenja (Žuta)

Nekritični problemi koji ne zaustavljaju obradu — npr. nedostajuće GPS podatke u kadru, veliki vremenski razmak između slika ciljeva ili nizak kontrast na kalibracijskom panelu.

**Akcija:** Pregledajte upozorenja nakon obrade, ali ne prekidajte

#### Poruke o pogreškama (Red

)

Kritični problemi koji mogu uzrokovati neuspjeh obrade — npr. pun disk, oštećena datoteka slike ili nije detektiran nijedan cilj dok je zatražena kalibracija reflektancije.

**Radnja:** Zaustavite obradu, otklonite pogrešku, ponovno pokrenite

### Uobičajene situacije u zapisniku

| Situacija                             | Značenje                                       | Potrebna radnja                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Detektiran cilj u \[naziv_datoteke]        | Uspješno pronađen kalibracijski cilj         | Nema - normalno                                         |
| Redovi napretka po slici              | Ažuriranje trenutnog napretka                        | Nema - normalno                                         |
| Nisu pronađeni ciljevi                    | Nisu otkriveni ciljevi za kalibraciju          | Označite slike ciljeva ili onemogućite kalibraciju reflektancije |
| Nedovoljno prostora na disku            | Nema dovoljno prostora za pohranu rezultata   | Oslobodite prostor na disku                            |
| Preskakanje oštećene datoteke           | Datoteka sa slikom je oštećena            | Ponovno kopirajte datoteku s SD kartice      |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Snimanje bez sirovog kadra se ne može obraditi | Ponovno snimanje sa sirovim kadrovima ili upotrijebiteCLI

`--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Pokretanje nije proizvelo slikovne proizvode — prijavljeno kao neuspjeh s napomenama | Pročitajte retke s napomenama; provjerite što je preskočeno i zašto |

### Kopiranje dnevničkih podataka

Za kopiranje dnevnika radi otklanjanja poteškoća ili podrške:

1. Otvorite panel za otklanjanje pogrešaka (Debug Log)
2. Kliknite gumb **&quot;Kopiraj dnevnik&quot;** (ili desnom tipkom miša → Odaberi sve)
3. Zalijepite u tekstualnu datoteku ili e-poštu
4. Pošaljite podršci naMAPIR

ako je potrebno

***

## Praćenje resursa sustava

### Korištenje CPU-a

**Slobodni način rada:**

* 1 CPU jezgra na \~100 %
* Ostale jezgre su neaktivne ili dostupne
* Sustav ostaje responzivan

**Chloros

+ paralelni način rada:**

* Više jezgri s visokim opterećenjem — broj ovisi o strategiji koju odabere [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Sustav može djelovati sporije

**Za praćenje:**

* Upravitelj zadatakaWindows

(Ctrl+Shift+Esc)
* kartica Performanse → odjeljak CPU
* Potražite procese &quot;Chloros

&quot; ili &quot;chloros-backend&quot;

### Upotreba memorije (RAM)

**Uobičajena potrošnja:**

* Mali projekti (&lt; 100 slika): 2-4 GB
* Srednji projekti (100-500 slika): 4-8 GB
* Veliki projekti (500+ slika): 8-16 GB
*Chloros

+ paralelnog načina rada koristi više RAM-a

**Ako je memorije malo:**

* Prerađujte manje skupove
* Zatvorite druge aplikacije
* Nadogradite RAM ako redovito obrađujete velike skupove podataka

### Iskorištenost GPU-a (Chloros

+ s CUDA)

Kada je omogućeno ubrzanje GPU-a:

* NVIDIA GPU pokazuje visoko iskorištenje (60-90%)
* Povećana potrošnja VRAM-a (potrebno 4 GB+ VRAM-a; 7 GB+ za istovremeno Texture Aware debayering)
* Faza kalibracije je znatno brža

**Za nadzor:**

* NVIDIA ikona u sustavnoj traci
* Upravitelj zadataka → Performanse → GPU
* GPU-Z ili sličan alat za nadzor

### Disk I/O

**Ono što možete očekivati:**

* Visoka upotreba diska za čitanje tijekom faze Analiziranja
* Visoka upotreba diska za pisanje tijekom faze Izvoza
* SSD je znatno brži od HDD-a

**Savjet za bolje performanse:**

* Koristite SSD za mapu projekta kad god je to moguće
* Izbjegavajte mrežne pogone za velike skupove podataka
* Pobrinite se da disk nije skoro pun (utječe na brzinu pisanja)

***

## Otkrivanje problema tijekom obrade

### Znakovi upozorenja

**Napredak se zaustavlja (nema promjene 5+ minuta):**

* Provjerite zapisnik o greškama (Debug Log) radi pogrešaka
* Provjerite ima li dostupnog prostora na disku
* Provjerite Upravitelj zadataka (Task Manager) kako biste bili sigurni da jeChloros

u pokretu

**Poruke o pogreškama se često pojavljuju:**

* Zaustavite obradu i pregledajte pogreške
* Uobičajeni uzroci: prostor na disku, oštećene datoteke, problemi s memorijom
* Pogledajte odjeljak Rješavanje problema u nastavku

**Sustav postaje neodzivan:**

*Chloros

+ paralelnom načinu rada koristi previše resursa
* Razmislite o smanjenju istovremenih zadataka ili nadogradnji hardvera
* Besplatni način rada manje je resursno intenzivan

### Kada zaustaviti obradu

Zaustavite obradu ako vidite:

* ❌ Pogreške &quot;Disk je pun&quot; ili &quot;Ne može se zapisati datoteka&quot;
* ❌ Ponavljajuće pogreške o oštećenju datoteka slike
* ❌ Sustav je potpuno zaleđen (ne reagira)
* ❌ Otkriveno je da su konfigurirane pogrešne postavke
* ❌ Uvezeni su pogrešni snimci

**Kako zaustaviti:**

1. Kliknite**gumb Stop** (zamjenjuje gumb Start) — jednom je dovoljno
2. Traka prikazuje &quot;Zaustavljanje...&quot; dok se dovršava obrada slike, a zatim se izvršavanje završava u zaustavljenom stanju
3. Već izvezene datoteke ostaju na disku; zapisnik ispisuje točan `[RUN-SUMMARY]` onoga što je dovršeno
4. Popravite probleme i ponovno pokrenite — izvršavanje započinje ispočetka

***

## Rješavanje problema tijekom obrade

### Obrada je vrlo spora

**Mogući uzroci:**

* Neoznačene ciljne slike (skeniranje svih slika)
* Pohrana na HDD-u umjesto na SSD-u
* Nedovoljno resursa sustava
* Konfigurirano mnogo indeksa
* Pristup mrežnom pogonu

**Rješenja:**

1. Ako je proces tek započeo i nalazi se u fazi otkrivanja (Detecting): Zaustavite, označite ciljne slike, ponovno pokrenite
2. Za ubuduće: Koristite SSD, smanjite broj indeksa, nadogradite hardver
3. Razmislite o korištenju alata za grupnu obradu velikih skupova podataka (CLI

)

### Upozorenja o &quot;prostoru na disku&quot;

**Rješenja:**

1. Odmah oslobodite prostor na disku
2. Premjestite projekt na pogon s više prostora
3. Smanjite broj indeksa za izvoz
4. Onemogućite LATTICE proizvode za izvoz koji vam ne trebaju (Postavke projekta → Obrada)
5. Koristite JPG format umjestoTIFF

(manje datoteke)

### Česte poruke &quot;Oštećena datoteka&quot;

**Rješenja:**

1. Ponovno kopirajte slike s SD kartice kako biste osigurali njihovu cjelovitost
2. Testirajte SD karticu na pogreške
3. Uklonite oštećene datoteke iz projekta
4. Nastavite s obradom preostalih slika

### Pregrijavanje / Usporavanje sustava

**Rješenja:**

1. Osigurajte adekvatnu ventilaciju
2. Očistite prašinu iz ventilacijskih otvora računala
3. Smanjite opterećenje obrade (koristite način rada Free umjestoChloros

+)
4. Obrađujte u hladnijim dijelovima dana

***

## Obavijest o završetku obrade

Kada obrada završi:

* Traka napretka dosegne 100%
* U zapisniku za otklanjanje poteškoća (Debug Log) pojavljuju se retci `[RUN-SUMMARY]` sa završnim brojevima
* Gumb za pokretanje ponovno postaje aktivan
* Sve izlazne datoteke nalaze se u stablu izlaznih datoteka projekta po kamerama: `<project>/<camera>/<format>/<Product>_Images/`

***

## Sljedeći koraci

Nakon završetka obrade:

1. **Pregledajte rezultate** - Pogledajte [Završetak obrade](finishing-the-processing.md)
2. **Provjerite izlaznu mapu** - Provjerite jesu li sve datoteke ispravno izvezene
3. **Pregledajte zapisnik o pogreškama (Debug Log)** - Provjerite postoje li upozorenja ili pogreške
4. **Pregledajte obrađene slike** - Koristite preglednik slika (Image Viewer) ili vanjski softver

Za informacije o pregledavanju i korištenju vaših obrađenih rezultata pogledajte [Završetak obrade](finishing-the-processing.md).
