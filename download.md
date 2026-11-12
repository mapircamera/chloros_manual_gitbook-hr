---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Preuzimanje

Preuzmite najnoviju verziju programa Chloros kako biste započeli s obradom multispektralnih slika.

### Sustavni zahtjevi

#### Windows


| Zahtjev          | Minimalno                                              | Preporučeno                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativni sustav** | Windows 10 (64-bitni)                                  | Windows 11 (64-bitni)                                  |
| **Procesor**        | Intel Core i5 ili ekvivalent                          | Intel Core i7 ili bolji                              |
| **Memorija (RAM)**    | 8 GB                                                  | 16 GB ili više                                         |
| **Grafička kartica**    | kompatibilna s DirectX 11                                | NVIDIA GPU s 4 GB+ VRAM-a                            |
| **Pohrana**          | 6 GB slobodnog prostora                                       | SSD s 10 GB+ slobodnog prostora                            |
| **Ekran**          | 1920x1080                                            | 2560x1440 ili viši                                  |
| **Internet**         | Potreban za aktivaciju licence za \[opcionalno] Chloros+ | Potreban za aktivaciju licence za \[opcionalno] Chloros+ |

#### Linux amd64 (x86_64)

| Zahtjev            | Minimalno                | Preporučeno            |
| ----------------- | -------------------------- | ------------------------- |
| **Distribucija**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86_64 (Intel/AMD)        | Intel Core i7 ili bolji   |
| **Memorija (RAM)**  | 8 GB                        | 16 GB ili više              |
| **Grafička kartica** | Nije potrebna (obrada u CPU-u) | NVIDIA GPU s 4 GB+ VRAM-a |
| **Pohrana**       | 2 GB slobodnog prostora             | SSD s 10 GB+ slobodnog prostora      |
| **Python**        | Python 3.7+ (za SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Zahtjev      | Minimalno                      | Preporučeno                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16GB ili AGX Orin |
| **Memorija (RAM)** | 8GB (dijeljena GPU/CPU)         | 16GB+ dijeljena                    |
| **Pohrana**      | 2 GB slobodnog prostora               | NVMe SSD s 10 GB+ slobodnog prostora        |
| **Python**       | Python 3.7+ (za SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU akceleracija**: Korisnici Chloros+ s NVIDIA GPU-ovima mogu koristiti CUDA akceleraciju za znatno bržu obradu. Ovo radi s NVIDIA desktop GPU-ovima na Windows i Linux amd64, te na NVIDIA Jetson s JetPack 6. Korisnici Chloros+ također dobivaju višenitnu obradu za maksimalnu brzinu.
{% endhint %}

***

## Preuzmite Chloros


### Najnovije stabilno izdanje: Verzija 1.3.0 (11. studenog 2026.)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Preuzmite Chloros za Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Preuzmite Chloros za Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Preuzmite Chloros za Linux arm64 / Jetson (.deb)</a>


#### Instalacijski program za Windows (GUI + CLI + pozadinski program)

* **Vrsta datoteke**: .exe (Windows instalacijski program)

**Koraci za instalaciju:**

1. Preuzmite datoteku .exe
2. Dvaput kliknite instalacijski program da biste započeli instalaciju
3. Slijedite upute čarobnjaka za instalaciju
4. Odaberite direktorij za instalaciju (zadano: `C:\Program Files\Chloros\`)
5. Na stranici **Odaberite dodatne zadatke**(Select Additional Tasks), zadržite označene obje opcije**Postavke sustava** (System settings) (preporučeno — pogledajte dolje)
6. Završite instalaciju i pokrenite Chloros ili Chloros CLI
7. Prijavite se sa svojim [MAPIR Cloud Chloros+ računom](https://cloud.mapir.camera/pricing) (ili nastavite s besplatnom verzijom)

**Opcije postavki sustava** (obje su zadano označene):

| Opcija, kako je navedena u instalacijskom programu | Što radi | Odjava za tihu instalaciju |
| --- | --- | --- |
| **Omogući dugim putanjama sustava Windows** (omogućuje da se mape projekta nalaze dublje od 260 znakova; preporučuje se) | Postavlja postavku sustava Windows a Windows na razini cijelog sustava na 1. Windows dolazi s tom postavkom isključenom, a tada nijedna putanja do datoteke ne može biti duža od 260 znakova; proizvodi Chloros koriste putanje do otprilike 130 znakova dublje od mape projekta, stoga se kod isključenih dugih putanja odbijaju dublje mape projekta. Stupa na snagu za programe pokrenute nakon instalacije. Ostaje aktivno i nakon deinstalacije jer se drugi programi mogu oslanjati na njega. | `/MERGETASKS="!longpaths"` |
| **Isključi pozadinski program Chloros iz skeniranja Microsoft Defendera** (znatno brže snimanje s kamera; preporučeno) | Dodaje isključenje procesa Microsoft Defendera za `chloros-backend.exe` u mapi instalacije, tako da zaštita u stvarnom vremenu ne skenira svaku datoteku koju pozadinski program zapisa, gdje god je zapisao. Pri visokim brzinama snimanja, skeniranje je koštalo oko 30 % brzine snimanja (77,7 naspram 52,7 snimaka u sekundi pri kontinuiranom snimanju s 9 kamera). Nijedan drugi program nije izuzet, a izuzeće se uklanja prilikom deinstalacije. Ako Defender nije instaliran ili je zaključan politikom, ovaj se korak tiho zanemaruje i instalacija se nastavlja. | `/MERGETASKS="!defenderexclusion"` |

Tiha instalacija (`/SILENT` ili `/VERYSILENT`) primjenjuje obje opcije, osim ako ne odustanete pomoću prikazane vrijednosti `/MERGETASKS`.

{% hint style="success" %}
Instalator dodaje mapu CLI (`C:\Program Files\Chloros\cli\`) u vaš sustav `PATH` samo jednom — ponovna instalacija ili nadogradnja više ne dodaju duplicirani unos, već uklanjaju duplicirane unose koje su dodali prethodni instalatori. Otvorite novi terminal nakon instalacije kako biste koristili `chloros-cli`. Instalacijski program također dodaje dva pravila vatrozida sustava Windows za pozadinski program, na svakom mrežnom profilu: **Chloros Backend (UDP-ulaz)**, za streamove s kamera, PTP, mDNS i DAQ-E stream podataka, i**Chloros Backend (TCP-ulaz)**, ograničen na lokalnu podmrežu, za ažuriranje firmvera DAQ-E, u kojem se senzor ponovno povezuje s ovim računalom kako bi preuzeo sliku. Obje se uklanjaju prilikom deinstalacije.
{% endhint %}

#### Linux amd64 (.deb paket — CLI + Backend)

* **Tip datoteke**: .deb (Debian/Ubuntu paket)
* **Arhitektura**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb paket — CLI + Backend)

* **Tip datoteke**: .deb (JetPack 6)
* **Arhitektura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Koristite naziv datoteke koju ste zapravo preuzeli i zadržite `./` kako bi `apt` instalirao tu datoteku umjesto da traži paket po nazivu. Instalirajte pomoću `apt install`, a ne `dpkg -i`: apt također instalira zahtjeve Python ovog SDK a, koje paket preporučuje, a ne zahtijeva, a `dpkg -i` ih preskače. Od verzije 1.3.0, pozadinski sustav i CLI instaliraju se neotpakirani pod `/usr/lib/chloros/`, pa se pokreću bez prethodnog otpakiranja.

Jetson paket koristi CUDA izdanje PyTorcha iz JetPacka 6, pa se obrada, uključujući i debayer osjetljiv na teksture, odvija na GPU-u Jetsona. Koristi biblioteke CUDA i cuDNN koje instalira JetPack 6 (`sudo apt install nvidia-jetpack` na slici kojoj one nedostaju).

Pogledajte [instalaciju za Linux](linux/linux-installation.md) za detaljne upute za postavljanje i [NVIDIA Jetson vodič](linux/nvidia-jetson-guide.md) za upute specifične za Jetson.

#### Python SDK (Sve platforme)

Svaki instalacijski program uključuje odgovarajuće `chloros_sdk` kolo, tako da verzija SDK-a uvijek odgovara instaliranom GUI-u/CLI-u/backendu. Na Windows u instalacijski program ga automatski instalira u vaš sustav Python. Na Linux u, `.deb` instalira SDK za sam sustav `python3` (u `/usr/lib/python3/dist-packages`), tako da `import chloros_sdk` radi odmah nakon instalacije, bez pipa i bez mreže. Samo virtualenv, koji ne vidi sistemske pakete, treba instalirani priloženi wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za hostove koji koriste samo pip (nije instaliran paket Chloros; SDK tada pokreće backend na drugom računalu), SDK je također dostupan na PyPI-ju:

```bash
pip install chloros-sdk
```

Pogledajte [API: Python SDK](api-python-sdk.md) i [SDK Reference](reference/sdk-reference.md) za dokumentaciju.

{% hint style="info" %}
**Korisnici sustava**Linux **: Paket `.deb` instalira CLI, backend i Python SDK. GUI za Linux ne postoji — sva interakcija odvija se putem CLI ili SDK.
{% endhint %}

***

## Dodatni resursi

### Python SDK


Za programere i automatizirane radne procese, Chloros Python SDK dolazi sa svakim instalaterom: Windows instalater instalira ga u vaš Python, a Linux `.deb` instalira ga za sustav `python3`. Na računalu bez paketa Chloros (onog koji pokreće backend na drugom računalu), instalirajte ga s PyPI-ja:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)

**Zahtjevi**: Chloros backend za upravljanje (Windows instalator ili Linux paket `.deb` instaliraju jedan), i Chloros+ prijava na plaćenom planu

***

## Što je uključeno

### Windows instalator

* ✅ **Chloros GUI** - Grafičko sučelje s punom funkcionalnošću
* ✅ **Chloros CLI** - Sučelje naredbenog retka (zahtijeva licencu Chloros+)
* ✅ **Chloros Backend** - Procesor
* ✅ **Python SDK** - instalira se u vaš Python ako je on prisutan (zahtijeva licencu Chloros+)
* ✅ **Profili kamera** - unaprijed konfigurirani predlošci kamera MAPIR


### Linux.deb paket

* ✅ **Chloros CLI** - sučelje naredbenog retka (zahtijeva licencu Chloros+)
* ✅ **Chloros Backend** - pogon za obradu
* ✅ **Python SDK** - instaliran za sustav `python3` (zahtijeva licencu Chloros+)
* ✅ **Profili kamera** - Unaprijed konfigurirani predlošci kamera MAPIR

* ❌ Bez GUI-ja — Linux je samo bezglavi CLI/SDK


### Python SDK (PyPI, za računala bez paketa Chloros)

* ✅ **Chloros SDK** - Python API (zahtijeva licencu Chloros+ Chloros)

***

## Nadogradnja na Chloros+

Otključajte napredne značajke s pretplatom na Chloros+:

* 🚀 **Višenitna obrada** - Paralelna obrada slika
* ⚡ **Ubrzanje GPU-om (CUDA)** - Iskoristite snagu NVIDIA GPU-a
* 💻 **Pristup putem CLI-a** - Automatizirajte pomoću alata naredbenog retka
* 🐍 **Python SDK** - Programski pristup putem API-a
* 📱 **Više uređaja** - Koristite na 2-10+ uređaja (ovisno o planu)
* **🐻 Napredna metoda debayeringa osjetljiva na teksture** - visokokvalitetna metoda debayeringa osjetljiva na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji (AI/ML) koji uklanja gotovo svu šumovitost debayeringa.
* 🧮 **Prilagođene formule** - Kreirajte prilagođene multispektralne indekse

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Pregledajte planove i cijene za Chloros+</a></p>

***

## Pomoć pri instalaciji

### Rješavanje problema

**Instalacija ne uspijeva s porukom o pogrešci:**

* Provjerite imate li administratorska prava
* Privremeno onemogućite antivirusni softver
* Provjerite ispunjavate li minimalne sistemske zahtjeve

**Aplikacija se ne pokreće (Windows):**

* Provjerite je li instaliran Windows 10/11 (64-bitna verzija)
* Ažurirajte grafičke upravljačke programe
* Provjerite Prikaz događaja sustava Windows radi detalja o pogreškama
* Kontaktirajte podršku s zapisnicima o pogreškama

**CLI se ne pokreće (Linux):**

* Provjerite je li paket `.deb` ispravno instaliran: `dpkg -l | grep chloros`
* Provjerite dozvole: `sudo chmod +x /usr/bin/chloros-cli`
* Pokrenite dijagnostiku: `chloros-cli selftest`
* Provjerite nedostajuće biblioteke: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (od verzije 1.3.0, `/usr/lib/chloros/chloros-backend` je mala skripta za pokretanje; sam program se nalazi u `/usr/lib/chloros/backend/`)

**Problemi s aktivacijom licence:**

* Provjerite je li internetska veza aktivna
* Provjerite vjerodajnice na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Provjerite ne blokira li vatrozid program Chloros

* Pogledajte [Chloros+ prijava](chloros+-login.md) za detaljne upute

### Podrška

Trebate pomoć s instalacijom ili postavom?

* 📧 **E-pošta**: info@mapir.camera
* 🌐 **Web-stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Početak rada](./)
* ❓ **Često postavljana pitanja**: [Često postavljana pitanja](faq.md)

***

## Ažuriranja softvera

Chloros provjerava postoje li ažuriranja, obavještava kada je dostupna nova verzija i povezuje na ovu stranicu za preuzimanje — ažurirate pokretanjem novog potpisanog instalatera. Vaši se postavci, projekti i prijava zadržavaju nakon ažuriranja. Na sustavima Linux i Jetson, `chloros-cli update` provjerava postoji li novija verzija i nudi preuzimanje i instalaciju odgovarajućeg `.deb` (ova naredba dostupna je samo na sustavima Linux).

***

## Zapisnik promjena**Verzija 1.3.0 (Najnovija, 11. studenog 2026.)**— za potpuni popis pogledajte**Što je novo u Chloros

u 1.3.0** na stranici [Getting Started](./).**Verzija 1.2.0**— pogledajte**Što je novo u Chloros u 1.2.0** na stranici [Getting Started](./).

<details>

<summary>Verzija 1.0.5</summary>

**Datum izdanja: 10. veljače 2026.**

**Nove značajke*** **Metoda uklanjanja šuma (Debayer) osjetljiva na teksture \[Samo za Chloros+] -** Metoda osjetljiva na teksture koristi visokokvalitetni debayer osjetljiv na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji (AI/ML), koji uklanja gotovo sav šum pri pretvorbi boja.
* **Podrška za T4P kalibracijske ciljeve*** **Brža obrada na GPU-u u aplikaciji Chloros+, bolje upravljanje memorijom**

**Ispravci grešaka*** Potpuno novi frontend (GUI), sada bi trebao raditi na svim računalima sa sustavom Windows.

</details>

<details>

<summary>Verzija 1.0.4</summary>

**Datum izdanja: 5. siječnja 2026.**

**Nove značajke*** **Prebacivanje slike/metapodataka**: Dodano prebacivanje u pregledniku datoteka za prikaz metapodataka odabrane slike u tablici umjesto u mreži slika
* **Klizač za zumiranje mreže slika**: Novi klizač u korisničkom sučelju za podešavanje veličine minijatura (također podržava CTRL + kotačić miša)
* **Gumbi za izvoz mreže slika**: Gumbi u gornjem redu za prebacivanje minijatura s JPG-a na obrađene izvoze (cilj, refleksancija, indeks, LUT)
* **Kartica s kartom**: Nova interaktivna 2D karta koja prikazuje GPS oznake lokacije slike
  * Podržava Google Maps i ESRI karte (automatski odabire najbolju uslugu karata ovisno o dostupnosti razine zumiranja)
  * Prikaz sličice pri prelasku mišem preko oznaka na karti

**Ispravci grešaka*** Poboljšana podrška za instalaciju programa Chloros na računalima s neengleskim operativnim sustavom

</details>

<details>

<summary>Verzija 1.0.3</summary>

**Datum izdanja: 20. prosinca 2025.**

**Nove značajke*** Početno izdanje

**Poboljšanja*** Početno izdanje

**Ispravci grešaka*** Početno izdanje

**Poznati problemi*** Početno izdanje

</details>

***

## Ugovor o licenci**Vlasnički softver** - Autorska prava (c) 2026 MAPIR Inc.

Neovlaštena upotreba, distribucija ili izmjena zabranjena je.

**Besplatna verzija**: Dostupna za osobnu i komercijalnu upotrebu s ograničenjima značajki**Chloros+**: Licenca temeljena na pretplati za napredne značajke i komercijalnu implementaciju
