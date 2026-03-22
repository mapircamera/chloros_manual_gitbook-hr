---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Preuzmi

Preuzmite najnoviju verziju Chloros da biste započeli s multispektralnom obradom slike.

### Zahtjevi sustava

#### Windows

| Zahtjev | Minimalno | Preporučeno |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativni sustav** | Windows 10 (64-bitni) | Windows 11 (64-bitni) |
| **Procesor** | Intel Core i5 ili ekvivalent | Intel Core i7 ili bolji |
| **Memorija (RAM)** | 8 GB | 16 GB ili više |
| **Grafička kartica** | DirectX 11 kompatibilan | NVIDIA GPU s 4 GB+ VRAM |
| **Skladištenje** | 6 GB slobodnog prostora | SSD s 10 GB+ slobodnog prostora |
| **Zaslon** | 1920x1080 | 2560x1440 ili više |
| **Internet** | Potrebno za \[opcionalno] Chloros+ aktivaciju licence | Potrebno za \[opcionalno] Chloros+ aktivaciju licence |

#### Linux amd64 (x86\_64)

| Zahtjev | Minimalno | Preporučeno |
| ----------------- | -------------------------------- | ------------------------ |
| **Distribucija** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86\_64 (Intel/AMD) | Intel Core i7 ili bolji |
| **Memorija (RAM)** | 8 GB | 16 GB ili više |
| **Grafička kartica** | Ništa (CPU obrada) | NVIDIA GPU s 4 GB+ VRAM |
| **Skladištenje** | 2 GB slobodnog prostora | SSD s 10 GB+ besplatno |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

#### Linux arm64 (NVIDIA Jetson)

| Zahtjev | Minimalno | Preporučeno |
| ---------------- | ---------------------------- | -------------------------------- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16GB ili AGX Orin |
| **Memorija (RAM)** | 8 GB (dijeljeni GPU/CPU) | 16GB+ podijeljeno |
| **Skladištenje** | 2 GB slobodnog prostora | NVMe SSD s 10 GB+ besplatno |
| **Python** | Python 3.7+ (za SDK) | Python 3.10+ |

{% hint style="info" %}
**GPU ubrzanje**: Chloros+ korisnici s NVIDIA GPU-om mogu koristiti CUDA ubrzanje za znatno bržu obradu. Ovo radi i na Windows (GPU za stolna računala) i Linux (GPU za stolna računala i NVIDIA Jetson). Korisnici Chloros+ također dobivaju višenitnu obradu za maksimalnu brzinu.
{% endhint %}

***

## Preuzmite Chloros

### Najnovije stabilno izdanje (23. ožujka 2026.): verzija 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Preuzmi Chloros za Windows (.exe)</a>

### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Preuzmi Chloros za Linux amd64 (.deb)</a>

### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Preuzmi Chloros za Linux arm64 / Jetson (.deb)</a>

#### Windows Installer (GUI + CLI + Backend)

* **Vrsta datoteke**: .exe (Windows Installer)**Koraci instalacije:**

1. Preuzmite gornju datoteku .exe
2. Dvaput kliknite instalacijski program za početak instalacije
3. Slijedite upute čarobnjaka za instalaciju
4. Odaberite instalacijski direktorij (zadano: `C:\Program Files\[USER]\Chloros\`)
5. Dovršite instalaciju i pokrenite Chloros ili Chloros CLI
6. Prijavite se sa svojim [MAPIR Cloud Chloros+ računom](https://cloud.mapir.camera/pricing) (ili nastavite s besplatnom verzijom)

{% hint style="success" %}
Instalacijski program automatski dodaje `chloros-cli` vašem sustavu PATH za pristup s naredbenog retka.
{% endhint %}

#### Linux amd64 (.deb paket — CLI + pozadina)

* **Vrsta datoteke**: .deb (Debian/Ubuntu paket)
* **Arhitektura**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb paket — CLI + Backend)

* **Vrsta datoteke**: .deb (JetPack 6)
* **Arhitektura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Pogledajte [Instalacija Linux](linux/linux-installation.md) za detaljne upute za postavljanje i [Vodič za NVIDIA Jetson](linux/nvidia-jetson-guide.md) za upute specifične za Jetson.

#### Python SDK (sve platforme)

```bash
pip install chloros-sdk
```

Pogledajte [API : Python SDK](api-python-sdk.md) za dokumentaciju.

{% hint style="info" %}
**Korisnici Linux**: Paket `.deb` instalira CLI i pozadinu. Python SDK se instalira zasebno putem pip-a. Ne postoji GUI za Linux — sva interakcija je putem CLI ili SDK.
{% endhint %}

***

## Dodatni resursi

### Python SDK

Za programere i tijekove rada automatizacije, instalirajte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)**Zahtjevi**: mora biti instaliran Chloros (instalacijski program Windows ili paket Linux `.deb`), potrebna je prijava za licencu Chloros+***

## Što je uključeno

### Instalacijski program Windows

* ✅ **Chloros GUI** - Grafičko sučelje sa svim značajkama
* ✅ **Chloros CLI** - sučelje naredbenog retka (zahtijeva licencu Chloros+)
* ✅ **Chloros Backend** - Procesor
* ✅ **Profili kamere** - unaprijed konfigurirani predlošci kamere MAPIR

### Linux .deb paket

* ✅ **Chloros CLI** - sučelje naredbenog retka (zahtijeva licencu Chloros+)
* ✅ **Chloros Backend** - Motor za obradu
* ✅ **Profili kamere** - unaprijed konfigurirani predlošci kamere MAPIR
* ❌ Nema GUI — Linux je bez glave samo CLI/SDK

### Python SDK (pip, sve platforme)

* ✅ **Chloros SDK** - Python API (potrebna licenca Chloros+)***

## Nadogradite na Chloros+

Otključajte napredne značajke s Chloros+ pretplatom:

* 🚀 **Obrada u više niti** - Paralelno obrađujte slike
* ⚡ **GPU (CUDA) ubrzanje** - Iskoristite NVIDIA GPU snagu
* 💻 **CLI Access** - Automatizirajte pomoću alata naredbenog retka
* 🐍 **Python SDK** - programski pristup API
* 📱 **Više uređaja** - Koristite na 2-10+ uređaja (ovisno o planu)
* **🐻 Advanced Texture Aware Debayer Method** - visokokvalitetni debayer s obzirom na rubove u kombinaciji s AI/ML modelom uklanjanja šuma koji uklanja gotovo sav šum debayeringa.
* 🧮 **Prilagođene formule** - Stvorite prilagođene multispektralne indekse

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Pogledajte planove Chloros+ &#x26; Cijene</a></p>

***

## Pomoć za instalaciju

### Rješavanje problema

**Instalacija nije uspjela s porukom o pogrešci:**

* Provjerite imate li administratorska prava
* Privremeno onemogućite antivirusni softver
* Provjerite ispunjavate li minimalne sistemske zahtjeve

**Aplikacija se ne pokreće (Windows):**

* Provjerite je li instaliran Windows 10/11 (64-bitni)
* Ažurirajte upravljačke programe za grafiku
* Provjerite Windows Event Viewer za detalje pogreške
* Kontaktirajte podršku s zapisnicima pogrešaka

**CLI se ne pokreće (Linux):**

* Provjerite je li paket `.deb` pravilno instaliran: `dpkg -l | grep chloros`
* Provjerite dopuštenja: `sudo chmod +x /usr/bin/chloros-cli`
* Pokrenite dijagnostiku: `chloros-cli selftest`
* Provjerite nedostajuće biblioteke: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemi s aktivacijom licence:**

* Provjerite je li internetska veza aktivna
* Provjerite vjerodajnice na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Provjerite da vatrozid ne blokira Chloros
* Pogledajte [Chloros+ Prijava](kloros+-login.md) za detaljne upute

### Dobivanje podrške

Trebate li pomoć s instalacijom ili postavljanjem?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Web stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentacija**: [Početak rada](./)
* ❓ **FAQ**: [Često postavljana pitanja](faq.md)***

## Dnevnik promjena

<detalji>

<sažetak>Verzija 1.1.0 (najnovija)</sažetak>

**Datum izlaska: ožujak 2026.**

**Nove značajke*** **Linux podrška** — Izvorni CLI i SDK za Linux amd64 (x86\_64) i arm64 (NVIDIA Jetson JetPack 6). Instalirajte putem `.deb` paketa.
* **NVIDIA Jetson podrška** — Optimizirana obrada za Jetson Nano, Orin Nano, Orin NX i AGX Orin edge uređaje.
* **Dynamic Compute Adaptation** — Automatsko otkrivanje hardvera i optimizacija strategije obrade. Chloros prilagođava se vašem hardveru od Jetson Nano do radne stanice s više GPU-a.
* **Cjevovod za obradu s 4 niti** — istodobna detekcija, kalibracija, obrada i izvoz niti s dinamičkom dodjelom GPU memorije.
* **Nove naredbe CLI** — `selftest` (dijagnostika sustava) i `update` (upravljanje ažuriranjem Linux).
* **Nove oznake procesa CLI** — `--debayer` (standardno/svjesno teksture), `--indices` (navedite indekse), `--target` (prvo pretražite ciljnu podmapu za brže otkrivanje).
* **Nove stavke GUI izbornika** — Dodaj datoteke, Dodaj mapu i Pokreni/zaustavi obradu sada dostupnima iz glavnog padajućeg izbornika.**Poboljšanja**

* Automatsko otkrivanje pozadinske platforme (putovi Windows i Linux)
* Poboljšani SDK `get_status()` s praćenjem napretka po niti
* Nove iznimke SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Termalno upravljanje i prilagodljivo prigušivanje za NVIDIA Jetson
* Automatsko upravljanje memorijom s OOM povratkom na popločanu GPU obradu

</details>

<detalji>

<summary>Verzija 1.0.5</summary>

**Datum objave: 10. veljače 2026.**

**Nove značajke*** **Texture Aware Debayer Method \[samo Chloros+] -** Texture Aware koristi visokokvalitetni edge-aware debayer u kombinaciji s AI/ML modelom uklanjanja šuma koji uklanja gotovo sav šum debayeringa.
* **Podrška za T4P kalibracijske ciljeve*** **Brža Chloros+ GPU obrada, bolje upravljanje memorijom**

**Ispravke grešaka*** Potpuno novo sučelje (GUI), sada bi trebalo raditi na svim Windows računalima.

</details>

<detalji>

<summary>Verzija 1.0.4</summary>

**Datum objave: 5. siječnja 2026.**

**Nove značajke*** **Prebacivanje slike/metapodataka**: Dodano uključivanje u preglednik datoteka za pregled metapodataka odabrane slike u tablici umjesto u rešetki slike
* **Klizač za zumiranje rešetke slike**: Novi klizač korisničkog sučelja za prilagodbu veličine minijature (također podržava CTRL + kotačić miša)
* **Gumbi za izvoz rešetke slike**: Gumbi u gornjem redu za prebacivanje minijatura iz JPG u obrađene izvoze (ciljevi, refleksija, indeks, LUT)
* **Kartica Karta**: Nova interaktivna 2D karta koja prikazuje slikovne GPS oznake lokacije
  * Podržava Google Maps i ESRI pločice karte (automatski odabire najbolju uslugu pločica na temelju dostupnosti razine zumiranja)
  * Pregled minijatura pokazivača miša na oznakama karte

**Ispravke grešaka*** Poboljšana podrška za instaliranje Chloros na računalima koja ne govore engleski

</details>

<detalji>

<summary>Verzija 1.0.3</summary>

**Datum izlaska: 20. prosinca 2025.**

**Nove značajke*** Početno pokretanje

**Poboljšanja*** Početno pokretanje

**Ispravke grešaka*** Početno pokretanje

**Poznati problemi*** Početno pokretanje

</details>

***

## Ugovor o licenci**Vlasnički softver** - Autorska prava (c) 2026 MAPIR Inc.

Zabranjeno je neovlašteno korištenje, distribucija ili izmjena.

**Besplatna verzija**: Dostupno za osobnu i komercijalnu upotrebu s ograničenjima značajki**Chloros+**: Licenca temeljena na pretplati za napredne značajke i komercijalne implementacije