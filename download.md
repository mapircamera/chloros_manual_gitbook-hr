---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Preuzmi

Preuzmite najnoviju verziju Chloros da biste započeli s multispektralnom obradom slike.

### Zahtjevi sustava

| Zahtjev | Minimalno | Preporučeno |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativni sustav** | Windows 10 (64-bitni) | Windows 11 (64-bitni) |
| **Procesor** | Intel Core i5 ili ekvivalent | Intel Core i7 ili bolji |
| **Memorija (RAM)** | 8 GB | 16 GB ili više |
| **Grafička kartica** | DirectX 11 kompatibilan | NVIDIA GPU s 4 GB+ VRAM |
| **Skladištenje** | 6 GB slobodnog prostora | SSD s 10 GB+ slobodnog prostora |
| **Zaslon** | 1920x1080 | 2560x1440 ili više |
| **Internet** | Potrebno za \[opcionalno] Chloros+ aktivaciju licence | Potrebno za \[opcionalno] Chloros+ aktivaciju licence |

{% hint style="info" %}
**GPU ubrzanje**: Chloros+ korisnici s NVIDIA GPU-ima mogu koristiti CUDA ubrzanje za znatno bržu obradu. Chloros+ korisnici također dobivaju višenitnu obradu za maksimalnu brzinu.
{% endhint %}

***

## Preuzmite Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Preuzmite Chloros ovdje</a>

### Najnovije stabilno izdanje

**Instalacijski program Chloros za Windows*** **Verzija**: 1.0.5
* **Datum izlaska**: 10. veljače 2026
* **Veličina datoteke (preuzimanje)**: 1,6 GB
* **Veličina datoteke (instalirana)**: 5,7 GB
* **Vrsta datoteke**: .exe (Windows Installer)

#### **Koraci instalacije:**

1. Preuzmite datoteku `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dvaput kliknite instalacijski program za početak instalacije
3. Slijedite upute čarobnjaka za instalaciju
4. Odaberite instalacijski direktorij (zadano: `C:\Program Files\[USER]\Chloros\`)
5. Dovršite instalaciju i pokrenite Chloros ili Chloros CLI
6. Prijavite se sa svojim [MAPIR Cloud Chloros+ računom](https://cloud.mapir.camera/pricing) (ili nastavite s besplatnom verzijom)

{% hint style="success" %}
Instalacijski program automatski dodaje `chloros-cli` vašem sustavu PATH za pristup s naredbenog retka.
{% endhint %}

***

## Dodatni resursi

### Python SDK

Za programere i tijekove rada automatizacije, instalirajte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentacija**: [API: Python SDK](api-python-sdk.md)**Zahtjevi**: Chloros Desktop mora biti instaliran, potrebna je prijava s licencom Chloros+***

## Što je uključeno

Instalacija Chloros uključuje:

* ✅ **Chloros** - Potpuno opremljeno grafičko sučelje (GUI)
* ✅ **Chloros CLI** - sučelje naredbenog retka (zahtijeva licencu Chloros+)
* ✅ **Chloros SDK** - Python API (potrebna licenca Chloros+)
* ✅ **Profili kamere** - unaprijed konfigurirani predlošci kamere MAPIR***

## Nadogradite na Chloros+

Otključajte napredne značajke s Chloros+ pretplatom:

* 🚀 **Obrada u više niti** - Paralelno obrađujte slike
* ⚡ **GPU (CUDA) ubrzanje** - Iskoristite NVIDIA GPU snagu
* 💻 **CLI Access** - Automatizirajte pomoću alata naredbenog retka
* 🐍 **Python SDK** - programski pristup API
* 📱 **Više uređaja** - Koristite na 2-10+ uređaja (ovisno o planu)
* **🐻 Advanced Texture Aware Debayer Method** - visokokvalitetni debayer s obzirom na rubove u kombinaciji s AI/ML modelom uklanjanja šuma koji uklanja gotovo sav šum debayeringa.&#x20;
* 🧮 **Prilagođene formule** - Stvorite prilagođene multispektralne indekse

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Pogledajte planove Chloros+ &#x26; Cijene</a></p>

***

## Pomoć za instalaciju

### Rješavanje problema

**Instalacija nije uspjela s porukom o pogrešci:**

* Provjerite imate li administratorska prava
* Privremeno onemogućite antivirusni softver
* Provjerite ispunjavate li minimalne sistemske zahtjeve

**Aplikacija se ne pokreće:**

* Provjerite je li instaliran Windows 10/11 (64-bitni)
* Ažurirajte upravljačke programe za grafiku
* Provjerite Windows Event Viewer za detalje pogreške
* Kontaktirajte podršku s zapisnicima pogrešaka

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

<summary>Verzija 1.0.5</summary>

#### **Datum izlaska**: 10. veljače 2026**Nove značajke*** **Texture Aware Debayer Method \[samo Chloros+] -** Texture Aware koristi visokokvalitetni edge-aware debayer u kombinaciji s AI/ML modelom uklanjanja šuma koji uklanja gotovo sav šum debayeringa.
* **Podrška za T4P kalibracijske ciljeve*** **Brža Chloros+ GPU obrada, bolje upravljanje memorijom**

**Ispravke grešaka*** Potpuno novo sučelje (GUI), sada bi trebalo raditi na svim Windows računalima.

</details>

<detalji>

<summary>Verzija 1.0.4</summary>

#### **Datum izlaska**: 5. siječnja 2026**Nove značajke*** **Prebacivanje slike/metapodataka**: Dodano uključivanje u preglednik datoteka za pregled metapodataka odabrane slike u tablici umjesto u rešetki slike
* **Klizač za zumiranje rešetke slike**: Novi klizač korisničkog sučelja za prilagodbu veličine minijature (također podržava CTRL + kotačić miša)
* **Gumbi za izvoz rešetke slike**: Gumbi u gornjem redu za prebacivanje minijatura iz JPG u obrađene izvoze (ciljevi, refleksija, indeks, LUT)
* **Kartica Karta**: Nova interaktivna 2D karta koja prikazuje slikovne GPS oznake lokacije
  * Podržava Google Maps i ESRI pločice karte (automatski odabire najbolju uslugu pločica na temelju dostupnosti razine zumiranja)
  * Pregled minijatura pokazivača miša na oznakama karte

**Ispravke grešaka*** Poboljšana podrška za instaliranje Chloros na računala koja ne govore engleski

</details>

<detalji>

<summary>Verzija 1.0.3</summary>

#### **Datum izlaska**: 20. prosinca 2025**Nove značajke*** Početno pokretanje

**Poboljšanja*** Početno pokretanje

**Ispravke grešaka*** Početno pokretanje

**Poznati problemi*** Početno pokretanje

</details>

***

## Ugovor o licenci**Vlasnički softver** - Autorska prava (c) 2026 MAPIR Inc.

Zabranjeno je neovlašteno korištenje, distribucija ili izmjena.

**Besplatna verzija**: Dostupno za osobnu i komercijalnu upotrebu s ograničenjima značajki**Chloros+**: Licenca temeljena na pretplati za napredne značajke i komercijalne implementacije