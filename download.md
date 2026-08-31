---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Preuzimanje

Preuzmite najnoviju verziju programaChloros

kako biste započeli s obradom multispektralnih slika.

### Sustavni zahtjevi

####Windows



| Zahtjev          | Minimalno                                              | Preporučeno                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativni sustav** |Windows

10 (64-bitni)                                  |Windows

11 (64-bitni)                                  |
| **Procesor**        | Intel Core i5 ili ekvivalent                          | Intel Core i7 ili bolji                              |
| **Memorija (RAM)**     | 8 GB                                                  | 16 GB ili više                                         |
| **Grafička kartica**    | kompatibilna s DirectX 11                                | NVIDIA GPU s 4 GB+ VRAM-a                            |
| **Pohrana**          | 6 GB slobodnog prostora                                       | SSD s 10 GB+ slobodnog prostora                            |
| **Ekran**           | 1920x1080                                            | 2560x1440 ili više                                  |
| **Internet**         | Potreban za aktivaciju licence za \[opcionalno]Chloros

+ | Potreban za aktivaciju licence za \[opcionalno]Chloros

+ |

####Linux

amd64 (x86\_64)

| Zahtjev       | Minimalno                    | Preporučeno               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribucija**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86_64 (Intel/AMD)        | Intel Core i7 ili bolji   |
| **Memorija (RAM)**  | 8 GB                        | 16 GB ili više              |
| **Grafička kartica** | Nije potrebna (obrada u CPU-u) | NVIDIA GPU s 4 GB+ VRAM-a |
| **Pohrana**       | 2 GB slobodnog prostora             | SSD s 10 GB+ slobodnog       |
| **Python**        |Python

3.7+ (zaSDK

)      |Python

3.10+              |

####Linux

arm64 (NVIDIA Jetson)

| Zahtjev      | Minimalno                      | Preporučeno                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPackom 6 | Jetson Orin NX 16GB ili AGX Orin |
| **Memorija (RAM)** | 8 GB (dijeljena GPU/CPU)         | 16 GB+ dijeljena                    |
| **Pohrana**      | 2 GB slobodnog prostora               | NVMe SSD s 10 GB+ slobodnog        |
| **Python**       |Python

3.7+ (zaSDK

)        |Python

3.10+                    |

{% hint style="info" %}
**Ubrzanje GPU-a**: Korisnici sustavaChloros

+ s NVIDIA GPU-ima mogu koristiti CUDA ubrzanje za znatno bržu obradu. Ovo radi i naWindows

(desktop GPU-ovima) i naLinux

(desktop GPU-ovima i NVIDIA Jetson). Korisnici verzijeChloros

+ također dobivaju višenitnu obradu za maksimalnu brzinu.
{% endhint %}

***

## PreuzmiteChloros

### Najnovije stabilno izdanje: Verzija 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Preuzmite Chloros za Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Preuzmite Chloros za Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Preuzmite Chloros za Linux arm64 / Jetson (.deb)</a>



#### Instalator zaWindows

(GUI +CLI

+ Backend)

* **Vrsta datoteke**: .exe (Windows

instalator)

**Koraci za instalaciju:**

1. Preuzmite datoteku .exe
2. Dvaput kliknite na instalacijski program da biste započeli instalaciju
3. Slijedite upute čarobnjaka za instalaciju
4. Odaberite direktorij za instalaciju (podrazumijevano: `C:\Program Files\MAPIR\Chloros\`)
5. Završite instalaciju i pokreniteChloros

iliChloros

CLI

6. Prijavite se sa svojim [MAPIR

CloudChloros

+ računom](https://cloud.mapir.camera/pricing) (ili nastavite s besplatnom verzijom)

{% hint style="success" %}
Instalacijski program automatski dodaje `chloros-cli` u sustavnu PATH putanju za pristup putem naredbenog retka.
{% endhint %}

####Linux

amd64 (.deb paket —CLI

+ pozadinski sustav)

* **Tip datoteke**: .deb (Debian/Ubuntu paket)
* **Arhitektura**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

####Linux

arm64 — NVIDIA Jetson (.deb paket —CLI

+ Backend)

* **Tip datoteke**: .deb (JetPack 6)
* **Arhitektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Pogledajte [Linux

instalaciju](linux/linux-installation.md) za detaljne upute za postavljanje i [NVIDIA Jetson vodič](linux/nvidia-jetson-guide.md) za upute specifične za Jetson.

####Python

SDK

(Sve platforme)

Svaki instalacijski program uključuje odgovarajuće `chloros_sdk` kotač, tako da verzijaSDK

uvijek odgovara instaliranom GUI-ju/CLI

/backendu. NaWindows

instalator ga automatski instalira u vaš sustavPython

; naLinux

`.deb` postavlja kotačić na `/usr/lib/chloros/sdk/` i ispisuje naredbu za instalaciju:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za hostove koji koriste samo pip (nije instaliran paketChloros

),SDK

je također na PyPI-ju:

```bash
pip install chloros-sdk
```

Pogledajte [API

:Python

SDK

](api-python-sdk.md) i [SDK

Reference](reference/sdk-reference.md) za dokumentaciju.

{% hint style="info" %}
**KorisniciLinux

a**: Paket `.deb` instaliraCLI

i pozadinski sustav (backend). Ne postoji GUI zaLinux

— sva interakcija odvija se putemCLI

iliSDK

.
{% endhint %}

***

## Dodatni resursi

###Python

SDK



Za programere i automatizirane radne procese, instalirajteChloros

Python

SDK

:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API

:Python

SDK

](api-python-sdk.md)

**Zahtjevi**: Instaliran mora bitiChloros

(Windows

instalacijski program ili paketLinux

`.deb`), potrebna je prijava s licencomChloros

+

***

## Što je uključeno

### Instalacijski program zaWindows



* ✅ **Chloros

GUI** - Grafičko sučelje s punom funkcionalnošću
* ✅ **Chloros

CLI

** - Sučelje naredbenog retka (zahtijevaChloros

+ licencu)
* ✅ **Chloros

Backend** - Procesor
* ✅ **Profili kamere** - Unaprijed konfigurirani predlošci kamereMAPIR



###Linux

.deb paket

* ✅ **Chloros

CLI

** - Sučelje naredbenog retka (zahtijeva licencuChloros

+)
* ✅ **Chloros

pozadinski sustav** - Procesor
* ✅ **Profili kamere** - Unaprijed konfigurirani predlošci kamere zaMAPIR


* ❌ Nema GUI-ja —Linux

je bez zaslona, samoCLI

/SDK



###Python

SDK

(pip, sve platforme)

* ✅ **Chloros

SDK

** -Python

API

(zahtijeva licencuChloros

+)

***

## Nadogradnja naChloros

+

Otključajte napredne značajke s pretplatom naChloros

+:

* 🚀 **Višenitna obrada** - Obradite slike paralelno
* ⚡ **Acceleracija GPU-om (CUDA)** - Iskoristite snagu NVIDIA GPU-a
* 💻 **PristupCLI

u** - Automatizirajte pomoću alata naredbenog retka
* 🐍 **Python

SDK

** - Programski pristupAPI

u
* 📱 **Više uređaja** - Koristite na 2-10+ uređaja (ovisno o planu)
* **🐻 Napredna metoda debayeringa osjetljiva na teksture** - visokokvalitetni debayer osjetljiv na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji (AI/ML) koji uklanja gotovo svu šum pri debayeringu.
* 🧮 **Prilagođene formule** - Kreirajte prilagođene multispektralne indekse

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Pregled planova i cijena za Chloros+</a></p>***

## Pomoć pri instalaciji

### Rješavanje problema

**Instalacija ne uspijeva s porukom o pogrešci:**

* Provjerite imate li administratorska prava
* Privremeno onemogućite antivirusni softver
* Provjerite ispunjavate li minimalne sistemske zahtjeve

**Aplikacija se neće pokrenuti (Windows

):**

* Provjerite je li instaliranWindows

10/11 (64-bit)
* Ažurirajte grafičke upravljačke programe
* Provjerite detalje o pogreškama u pregledniku događaja (Windows

)
* Kontaktirajte podršku s zapisnicima o pogreškama

**CLI

se ne pokreće (Linux

):**

* Provjerite je li paket `.deb` ispravno instaliran: `dpkg -l | grep chloros`
* Provjerite dozvole: `sudo chmod +x /usr/bin/chloros-cli`
* Pokrenite dijagnostiku: `chloros-cli selftest`
* Provjerite nedostajuće biblioteke: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemi s aktivacijom licence:**

* Provjerite je li internetska veza aktivna
* Provjerite vjerodajnice na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Provjerite ne blokira li vatrozidChloros

* Pogledajte [Chloros

+ Login](chloros+-login.md) za detaljne upute

### Potražite podršku

Trebate pomoć s instalacijom ili postavom?

* 📧 **E-pošta**: info@mapir.camera
* 🌐 **Web-stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Početak rada](./)
* ❓ **ČPP**: [Često postavljana pitanja](faq.md)***

## Ažuriranja softvera

Chloros

provjerava postoje li ažuriranja, obavještava kada je dostupna nova verzija i povezuje na ovu stranicu za preuzimanje — ažurirate pokretanjem novog potpisanog instalatera. Vaši se postavci i projekti zadržavaju nakon ažuriranja. NaLinux

i Jetson, `chloros-cli update` provjerava postoji li novija verzija i nudi preuzimanje i instalaciju odgovarajućeg `.deb` (ova naredba je dostupna samo naLinux

).

***

## Zapisnik promjena**Verzija 1.2.0 (najnovija)**— za potpunu listu značajki pogledajte**Što je novo uChloros

1.2.0** na stranici [Getting Started](./).

<details>

<summary>Verzija 1.0.5</summary>

**Datum izdanja: 10. veljače 2026.**

**Nove značajke*** **Metoda uklanjanja šuma (Debayer) prilagođena teksturi \[ Samo zaChloros

+] -** Metoda prilagođena teksturi koristi visokokvalitetni debayer osjetljiv na rubove u kombinaciji s modelom za uklanjanje šuma temeljenim na umjetnoj inteligenciji/strojnom učenju (AI/ML), koji uklanja gotovo sav šum nastali pri debayeriranju.
* **Podrška za T4P kalibracijske ciljeve*** **Brža obrada naChloros

+ GPU-u, bolje upravljanje memorijom**

**Ispravci grešaka*** Potpuno novi frontend (GUI), sada bi trebao raditi na svim računalima sWindows

.

</details>

<details>

<summary>Verzija 1.0.4</summary>

**Datum izlaska: 5. siječnja 2026.**

**Nove značajke*** **Prekidač za sliku/metapodatke**: Dodan prekidač u pregledniku datoteka za prikaz metapodataka odabrane slike u tablici umjesto u mreži slika
* **Klizač za zumiranje mreže slika**: Novi klizač u korisničkom sučelju za podešavanje veličine minijatura (također podržava CTRL + kotačić miša)
* **Gumbi za izvoz mreže slika**: Gumbi u gornjem redu za prebacivanje minijatura s JPG-a na obrađene izvoze (Cilj, Reflektancija, Indeks, LUT)
* **Kartica s kartom**: Nova interaktivna 2D karta koja prikazuje oznake GPS lokacije slike
  * Podržava Google Maps i ESRI karte (automatski odabire najbolju uslugu karata ovisno o dostupnosti razine zumiranja)
  * Prikaz pretpregleda minijature pri prelasku mišem preko oznaka na karti

**Ispravci grešaka*** Poboljšana podrška za instalaciju aplikacije Chloros na računalima s neengleskim operativnim sustavom

</details>

<details>

<summary>Verzija 1.0.3</summary>

**Datum izdanja: 20. prosinca 2025.**

**Nove značajke*** Prvo izdanje

**Poboljšanja*** Prvo izdanje

**Ispravci grešaka*** Prvo izdanje

**Poznati problemi*** Prvo izdanje

</details>

***

## Ugovor o licenci**Vlasnički softver** - Autorska prava (c) 2026 MAPIR Inc.

Neovlaštena upotreba, distribucija ili izmjena zabranjena je.

**Besplatna verzija**: Dostupna za osobnu i komercijalnu upotrebu s ograničenjima značajki**Chloros+**: Licenca temeljena na pretplati za napredne značajke i komercijalnu implementaciju
