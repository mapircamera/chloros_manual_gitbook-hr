# Praćenje obrade

Nakon što je obrada započela, Chloros pruža nekoliko načina za praćenje napretka, provjeru problema i razumijevanje što se događa s vašim skupom podataka. Ova stranica objašnjava kako pratiti svoju obradu i tumačiti informacije koje pruža Chloros.

## Pregled trake napretka

Traka napretka u gornjoj zaglavnici prikazuje status obrade i postotak dovršenosti u stvarnom vremenu. Napredak se prenosi uživo s backenda putem događaja Server-Sent Events (SSE), tako da traka odražava ono što cijev za obradu (pipeline) zapravo radi.

### Traka napretka u slobodnom načinu rada

Za korisnike bez licence Chloros+:

**Prikaz napretka u 2 faze:**

1.**Detekcija ciljeva** - Pronalaženje kalibracijskih ciljeva na slikama
2. **Obrada** - Primjena ispravaka i izvoz**Traka napretka prikazuje:**

* Ukupni postotak dovršenosti (0-100%)
* Naziv trenutne faze
* Jednostavna vizualizacija vodoravne trake

### Traka napretka za Chloros+

Za korisnike s licencom Chloros+:

**Prikaz napretka u 4 faze:**

1.**Detekcija** - Pronalaženje kalibracijskih ciljeva
2. **Analiza** - Pregledavanje slika i priprema cjevovoda
3. **Kalibracija** - Primjena korekcija vinjete i reflektancije
4. **Izvoz** - Pohrana obrađenih datoteka

**Interaktivne značajke:**

* **Pređite** mišem preko trake napretka kako biste vidjeli prošireni panel s 4 faze
* **Kliknite** na traku napretka kako biste zamrznuli/prikačili prošireni panel
* **Ponovno kliknite** za otključavanje i automatsko skrivanje pri napuštanju pokazivačem miša
* Svaka faza prikazuje pojedinačni napredak (0-100%)

{% hint style="info" %}
**Paritet CLI-a**: tijekom izvođenja `chloros-cli process`-a, četiri iste niti izvještavaju o fazama Otkrivanje, Analiziranje, Obrada i Izvoz, a `chloros-cli export-status` prikazuje trenutačni napredak izvoza Niti-4 iz drugog terminala. Pogledajte [CLI referencu](../reference/cli-reference.md).
{% endhint %}

***

## Razumijevanje svake faze obrade

{% hint style="info" %}
**Arhitektura cjevovoda**: Ove 4 GUI faze odgovaraju [cjevovodu obrade s 4 niti](../processing-architecture/processing-pipeline.md). Na sustavima s GPU akceleracijom, Nit 3 (Kalibracija) koristi se [dinamičkom prilagodbom izračuna](../processing-architecture/dynamic-compute-adaptation.md), što optimizira obradu za vaš specifični hardver.
{% endhint %}

### Faza 1: Detekcija (Detekcija cilja)

**Što se događa:**

* Chloros skenira slike koje ste označili pomoću potvrdnog okvira za cilj (sve slike samo kada nijedna nije označena)
* Algoritmi računalnog vida identificiraju kalibracijske panele
* Vrijednosti reflektancije se izdvajaju iz svakog panela
* Vremenski pečati cilja se bilježe za ispravno zakazivanje kalibracije

**Trajanje:**

* S označenim ciljevima: 10-60 sekundi
* Bez označenih ciljeva: 5-30+ minuta (skenira sve slike)

**Pokazatelj napretka:**

* Otkrivanje: 0 % → 100 %
* Broj skeniranih slika (broji se samo slike koje se zapravo skeniraju)
* Broj pronađenih meta

**Na što obratiti pozornost:**

* Trebalo bi se brzo dovršiti ako su mete ispravno označene
* Ako traje predugo, mete možda nisu označene
* Provjerite zapisnik za greške (Debug Log) na poruke "Meta pronađena"

### Faza 2: Analiza

**Što se događa:**

* Čitanje EXIF metapodataka slike (vremenski žigovi, postavke ekspozicije)
* Određivanje strategije kalibracije na temelju vremenskih žigova ciljeva i dostupnih DAQ downwelling podataka
* Organiziranje reda za obradu slika
* Priprema radnika za paralelnu obradu (samo za Chloros+)

**Trajanje:** 5-30 sekundi**Pokazatelj napretka:**

* Analiza: 0 % → 100 %
* Brza faza, obično se brzo završava

**Ono što trebate pratiti:**

* Napredak bi trebao biti stalan bez pauza
* U zapisnik za otklanjanje pogrešaka (Debug Log) pojavit će se upozorenja o nedostajućim metapodacima

### Faza 3: Kalibracija

**Što se događa:**

* **Debayering**: Pretvorba RAW Bayer uzorka u 3 kanala (izostavljeno za LATTICE mono module, uz napomenu)
* **Korekcija vinjete**: Uklanjanje potamnjivanja rubova objektiva
* **Kalibracija reflektancije**: Normalizacija s ciljanim vrijednostima i/ili DAQ downwelling
* **Izračun indeksa**: Izračunavanje multispektralnih indeksa
* Obrada svake slike kroz cijeli pipeline

**Trajanje:** Većina ukupnog vremena obrade (60-80%)**Pokazatelj napretka:**

* Kalibracija: 0% → 100%
* Trenutna slika se obrađuje
* Obradene slike / Ukupno slika

**Ponašanje pri obradi:**

* **Slobodni način rada**: Obradba jedne slike odjednom, sekvencijalno
* **Chloros+ način rada**: Pokreće hardverski prilagodljivi bazen radnika — 1-4 istovremena radnika na GPU sustavima (ovisno o VRAM-u), jedan radnik po fizičkom jezgri (umanjeno za jedan) na sustavima s isključivo CPU-om. Vidi [Dinamička prilagodba izračuna](../processing-architecture/dynamic-compute-adaptation.md)
* **Acceleracija pomoću GPU-a**: Značajno ubrzava ovu fazu**Na što obratiti pozornost:**

* Stalan napredak kroz broj slika
* Provjerite zapisnik za otklanjanje poteškoća (Debug Log) radi poruka o dovršetku svake slike
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

1. Kliknite ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

u lijevoj bočnoj traci
2. Otvara se panel dnevnika koji prikazuje poruke o obradi u stvarnom vremenu
3. Automatski se pomiče prema dolje kako bi prikazao najnovije poruke

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Razumijevanje poruka u zapisniku  Zapisne linije sustava

Chloros označene su oznakama u zagradama koje imenuju podsustav — na primjer `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najvažnija za poznavanje je **pregled pokretanja**, koji se ispisuje na kraju svakog pokretanja (uključujući i zaustavljena pokretanja):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Slijede dodatne pomoćne linije `[RUN-SUMMARY]` kad god je nešto potrebno objasniti — na primjer, pokretanje koje nije dalo nikakve rezultate, ili kamera čiji je zatraženi proizvod preskočen kao neprimjenjiv. Redak `[EXPORT-CHECK]` objašnjavaju preskoke po kamerama (npr. zašto kamera u boji RGB nije dobila proizvod radijance).

Opće razine važnosti poruka (primjeri u nastavku su ilustrativni, a ne doslovni):

#### Informativne poruke (bijela/siva)

Uobičajena ažuriranja obrade: obrada započela, ciljevi otkriveni (s brojem panela), napredak kalibracije po slici, datoteke izvezene, obrada dovršena.

#### Upozorenja (žuta)

Nekritični problemi koji ne zaustavljaju obradu — npr. nedostajuće GPS podatke u kadru, veliki vremenski razmak između slika ciljeva ili nizak kontrast u kalibracijskom panelu.

**Postupak:** Pregledajte upozorenja nakon obrade, ali ne prekidajte

#### Poruke o pogreškama (Red)

Kritični problemi koji mogu uzrokovati prekid obrade — npr. pun disk, oštećena datoteka slike ili nije detektiran nijedan cilj iako je zatražena kalibracija reflektivnosti.

**Radnja:** Zaustavite obradu, riješite pogrešku, ponovno pokrenite

### Uobičajene situacije u zapisniku

| Situacija                             | Značenje                                       | Potrebna radnja                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Detektiran cilj u \[naziv_datoteke]        | Uspješno pronađen kalibracijski cilj         | Nema - normalno                                         |
| Redovi napretka po slici              | Trenutno ažuriranje napretka                       | Nema - normalno                                         |
| Nisu pronađeni ciljevi                    | Nisu otkriveni ciljevi za kalibraciju        | Označite slike ciljeva ili onemogućite kalibraciju reflektancije |
| Nedovoljno prostora na disku            | Nema dovoljno prostora za pohranu rezultata  | Oslobodite prostor na disku                                    |
| Preskakanje oštećene datoteke               | Datoteka slike je oštećena                         | Ponovno kopirajte datoteku s SD kartice                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Snimanje bez sirovog kadra se ne može obraditi | Ponovno snimite sa sirovim, ili upotrijebite CLI `[RUN-SUMMARY] ... 0 file(s) written`  |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Pokretanje nije proizvelo slikovne proizvode — prijavljeno kao neuspjeh s napomenama | Pročitajte retke s napomenama; provjerite što je preskočeno i zašto |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Uključene su Windows ove duge staze, a mapa projekta je preduboka za ograničenje staze od 260 znakova; ništa nije obrađeno | Koristite kraću mapu projekta (poruka navodi najdužu putanju koja stane), ili uključite dugim putanjama sustava Windows i ponovno pokrenite Chloros — pogledajte [Pokretanje obrade](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Nije moguće zapisati clamp masku proizvoda LATTICE; sam proizvod je u redu i prikazuje `Chloros:UnmixClampMask=unavailable` | Provjerite prostor na disku i dozvole za mapu, a zatim ponovno pokrenite obradu ako vam je potrebna maska |

### Kopiranje podataka iz dnevnika

Za kopiranje dnevnika radi otklanjanja poteškoća ili podrške:

1. Otvorite panel Debug Log
2. Kliknite gumb **Copy**(kopira dnevnik u međuspremnik) ili**Download** za spremanje kao tekstualnu datoteku
3. Zalijepite u tekstualnu datoteku ili e-poštu
4. Pošaljite podršci tvrtke MAPIR ako je potrebno

### Prijavljivanje greške s kartice dnevnika

Novo u verziji 1.3.0: ikona greške (**Prijavi grešku**) na vrhu kartice Dnevnik pogrešaka šalje izvješće izravno tvrtki MAPIR.

1. Kliknite ikonu greške na vrhu kartice Dnevnik pogrešaka
2. Opišite problem: što ste radili, što ste očekivali i što se umjesto toga dogodilo (obavezno)
3. Po želji označite **Dodaj nekoliko slika (do 10)**i odaberite slike iz otvorenog projekta ili upotrijebite**Pregledaj...** za bilo koju drugu datoteku sa slikama
4. Kliknite **Pošalji**

Izvješće uključuje vaš opis, sve priložene slike, zapisnike programa Chloros i podatke o sustavu (verzija programa Chloros, OS, GPU i postavke projekta). Morate biti prijavljeni u program Chloros (na kartici Korisnik) da biste poslali izvješće. Možete zatvoriti prozor dok se izvješće šalje; kada se pošalje, u zapisnik se upisuje referenca kao što je `Bug report sent. Reference: BR-…` — navedite je kada kontaktirate MAPIR. Ako se izvješće ne može prenijeti (na primjer, nema internetske veze), Chloros ga sprema na vaše računalo i prikazuje putanju kako biste ga mogli poslati e-poštom na support@mapir.camera.

***

## Praćenje resursa sustava

### Korištenje CPU-a

**Besplatni način rada:**

* 1 CPU jezgra na ~100 %
* Ostale jezgre su neaktivne ili dostupne
* Sustav ostaje responzivan

**Chloros+ Paralelni način rada:**

* Više jezgri s visokim opterećenjem — broj ovisi o strategiji koju odabere [Dinamička prilagodba računanja](../processing-architecture/dynamic-compute-adaptation.md)
* Sustav može djelovati sporije

**Za praćenje:**

* Upravitelj zadataka sustava Windows (Ctrl+Shift+Esc)
* kartica Performanse → odjeljak CPU
* Potražite procese "Chloros" ili "chloros-backend"

### Upotreba memorije (RAM)

**Tipična upotreba:**

* Mali projekti (< 100 slika): 2-4 GB
* Srednji projekti (100-500 slika): 4-8 GB
* Veliki projekti (500+ slika): 8-16 GB
* Paralelni način rada Chloros+ koristi više RAM-a

**Ako je memorija niska:**

* Prerađujte manje skupove
* Zatvorite druge aplikacije
* Nadogradite RAM ako redovito obrađujete velike skupove podataka

### Iskorištenje GPU-a (Chloros+ s CUDA-om)

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

**Savjet za performanse:**

* Koristite SSD za mapu projekta kad god je to moguće
* Izbjegavajte mrežne pogone za velike skupove podataka
* Pobrinite se da disk nije skoro pun (utječe na brzinu pisanja)

***

## Otkrivanje problema tijekom obrade

### Znakovi upozorenja

**Napredak se zaustavlja (nema promjene 5+ minuta):**

* Provjerite zapisnik za greške (Debug Log)
* Provjerite ima li dostupnog prostora na disku
* Provjerite Upravitelj zadataka (Task Manager) kako biste se uvjerili da Chloros radi

**Poruke o pogreškama se često pojavljuju:**

* Zaustavite obradu i pregledajte pogreške
* Uobičajeni uzroci: prostor na disku, oštećene datoteke, problemi s memorijom
* Pogledajte odjeljak za rješavanje problema u nastavku

**Sustav postaje neodzivan:**

* Paralelni način rada programaChloros

potrošio je previše resursa
* Razmislite o smanjenju istovremenih zadataka ili nadogradnji hardvera
* Slobodni način rada manje je resursno intenzivan

### Kada zaustaviti obradu

Zaustavite obradu ako vidite:

* ❌ pogreške "Disk je pun" ili "Ne može se zapisati datoteka"
* ❌ Ponavljajuće pogreške o oštećenju datoteke slike
* ❌ Sustav je potpuno zaleđen (ne reagira)
* ❌ Shvatili ste da su konfigurirane pogrešne postavke
* ❌ Uvezeni su pogrešni snimci

**Kako zaustaviti:**

1. Kliknite**gumb Zaustavi** (zamjenjuje gumb Pokreni) — jednom je dovoljno
2. Traka prikazuje "Zaustavljanje..." dok se trenutna slika dovršava, a zatim se izvršavanje završava u zaustavljenom stanju
3. Već izvezene datoteke ostaju na disku; u zapisniku se ispisuje iskren `[RUN-SUMMARY]` onoga što je dovršeno
4. Ispravite probleme i ponovno pokrenite — izvršavanje se nastavlja od početka

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
3. Razmislite o korištenju **CLI a** za serijsku obradu velikih skupova podataka

### Upozorenja o "prostoru na disku"

**Rješenja:**

1. Odmah oslobodite prostor na disku
2. Premjestite projekt na pogon s više prostora
3. Smanjite broj indeksa za izvoz
4. Onemogućite LATTICE proizvode za izvoz koji vam ne trebaju (Postavke projekta → Obrada)
5. Koristite format JPG umjesto TIFF (manje datoteke)

### Česte poruke "Oštećena datoteka"

**Rješenja:**

1. Ponovno kopirajte slike s SD kartice kako biste osigurali njihovu cjelovitost
2. Provjerite SD karticu na pogreške
3. Uklonite oštećene datoteke iz projekta
4. Nastavite s obradom preostalih slika

### Pregrijavanje / Usporavanje sustava

**Rješenja:**

1. Osigurajte adekvatnu ventilaciju
2. Očistite prašinu iz ventilacijskih otvora računala
3. Smanjite opterećenje obrade (koristite način rada Free umjesto Chloros+)
4. Obrađujte tijekom hladnijih dijelova dana

***

## Obavijest o završetku obrade

Kada obrada završi:

* Traka napretka dosegne 100%
* U zapisniku za otklanjanje pogrešaka (Debug Log) pojavljuju se retci `[RUN-SUMMARY]` s konačnim brojevima
* Gumb za pokretanje ponovno postaje aktivan
* Sve izlazne datoteke nalaze se u stablu izlaznih datoteka projekta po kamerama: `<project>/<camera>/<format>/<Product>_Images/`

***

## Sljedeći koraci

Nakon što se obrada završi:

1. **Pregledajte rezultate** - Pogledajte [Završetak obrade](finishing-the-processing.md)
2. **Provjerite izlaznu mapu** - Provjerite jesu li sve datoteke ispravno izvezene
3. **Pregledajte zapisnik o otklanjanju pogrešaka (Debug Log)** - Provjerite postoje li upozorenja ili pogreške
4. **Pregledajte obrađene slike** - Koristite preglednik slika (Image Viewer) ili vanjski softver

Za informacije o pregledavanju i korištenju vaših obrađenih rezultata pogledajte [Završetak obrade](finishing-the-processing.md).
