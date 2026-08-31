Pregled aplikacije Linux

Chloros 1.2.0 pruža nativnu podršku za Linux na **CLI**i**Python SDK** — za glavu bez zaslona (headless) višespektralnu obradu slika, uz upravljanje LATTICE kamerom uživo i DAQ senzorom svjetla — na radnim stanicama Linux, poslužiteljima i NVIDIA Jetson rubnim uređajima.

{% hint style="info" %}
**Nema desktop GUI-ja na Linux.**Desktop GUI za Chloros dostupan je samo na Windows. Korisnici Linux komuniciraju s Chloros putem [CLI](../CLI.md) i [Python SDK](../api-python-sdk.md). `.deb` dodaje unos**Chloros CLI** u izbornik vaše aplikacije — on jednostavno otvara emulator terminala koji pokreće `chloros-cli`.
{% endhint %}

***

## Matrica podrške platformama

| Značajka | Windows (GUI) | Windows (CLI / SDK) | Linux amd64 (CLI / SDK) | Linux arm64 / Jetson (CLI / SDK) |
| --- | --- | --- | --- | --- |
| **Desktop GUI** | Da | N/A | Ne | Ne |
| **CLI** (`chloros-cli`) | Da | Da | Da | Da |
| **Python SDK** (`chloros-sdk`) | Da | Da | Da | Da |
| **Proces obrade slika** | Da | Da | Da | Da |
| **Upravljanje kamerom LATTICE (uživo)** | Da (kartica Kamere) | Da (`chloros-cli lattice`, SDK) | Da | Da |
| **DAQ senzori svjetla (uživo)** | Da (kartica Senzori svjetla) | Da (`chloros-cli daq pool-*`, SDK) | Da | Da |
| **PTP vremenska sinkronizacija (host je grandmaster)** | Da | Da (`chloros-cli time-sync`) | Da | Da |
| **GPU akceleracija (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer s podrškom za teksture** | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Dinamička prilagodba izračuna** | Da | Da | Da | Da |
| **Backend kao sustavna usluga** (`chloros-backend.service`) | Ne | Ne | Da (po želji) | Da (po želji) |
| **Ažuriranje na licu mjesta** (`chloros-cli update`) | Ne (pokrenite instalacijski program) | Ne (pokrenite instalacijski program) | Da | Da |***

## Podržane arhitekture

| Arhitektura | Opis | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardni desktop/server procesori (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM procesori — obitelj NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (izdanje JetPack 6) |

## Podržane distribucije Linux

* **Ubuntu 22.04 LTS ili noviji** (amd64)
* **Debian 12 ili noviji** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson Orin)***

## Što korisnici  dobivaju

* **Chloros CLI** — potpuno sučelje naredbenog retka za serijsku obradu, automatizaciju i skriptanje
* **Chloros Python SDK** — programsko sučelje za istraživačke cjevovode i prilagođene alate (može se instalirati s PyPI-ja, a također je uključeno u `.deb` kao paket s usklađenom verzijom)
* **Upravljanje LATTICE kamerama** — otkrijte, povežite, konfigurirajte i snimajte s LATTICE kamera i sinkroniziranih nizova više kamera putem `chloros-cli lattice` i SDK; `.deb` uključuje Arena SDK runtime koji kamere zahtijevaju
* **Kontrola DAQ senzora svjetla** — povežite DAQ-U/M/E senzore, prenosite kalibrirane spektre i snimajte `.daq` datoteke putem `chloros-cli daq pool-*` i SDK
* **PTP sinkronizacija vremena** — pozadinski program Chloros pokreće PTP grandmasteru kojemu kamere LATTICE i DAQ-E senzori rade kao sluge; inspektirajte ga pomoću `chloros-cli time-sync` i održavajte ga u pozadini aktivnim pomoću sustavne jedinice `chloros-backend.service` (vidi [Linux Instalacija](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizacija projekta** — pokretanje spremljenih projekata bez zaslona pomoću `chloros-cli project` i `open_project` sustava SDK
* **Acceleracija GPU-om** — obrada ubrzana CUDA-om na NVIDIA GPU-ima (desktop i Jetson)
* **Dinamička prilagodba izračuna** — automatska detekcija hardvera i odabir strategije obrade, s mogućnošću premošćivanja (override) pomoću `CHLOROS_STRATEGY` kao rješenjem za napredne korisnike
* **Sve značajke obrade** — isti procesni put kao i kod Windows: kalibracija, korekcija vinjete, indeksi vegetacije i svaki format izvoza
* **Značajke Chloros+** — višedretvena (pipeline) obrada, debayer s podrškom za teksture i prilagođeni indeksi, uz plaćeni Chloros+ plan

## Što korisnici Linux ne dobivaju

* **Desktop GUI** — nema grafičkog sučelja; sva interakcija je putem CLI ili Python SDK
* **Preglednik slika** — nema interaktivnog preglednika slika, prikaza u mreži ili oznaka na karti
* **Vizualno upravljanje projektima** — projekti se stvaraju i upravljaju putem naredbi CLI i poziva SDK (sam hardver — kamere, senzori, snimanje — ostaje u potpunosti kontroliran iz terminala)***

## Zahtjev za licencu

Pristup CLI i SDK zahtijeva **plaćeni Chloros+ paket — Copper ili viši**(Copper, Bronze, Silver, Gold). Besplatni**Iron** paket nema pristup CLI / SDK. Ograničenje provodi pozadinski sustav (backend), a ne samo CLI:

| Situacija | Odgovor pozadinskog sustava (backend) |
| --- | --- |
| Nije prijavljeno | `401` sa `error_code: AUTH_REQUIRED` |
| Prijavljeno na besplatnom Iron nivou | `403` s `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` radi na bilo kojoj razini — to je jedina ruta izuzeta od kapije — stoga je razlog odbijanja uvijek vidljiv.

***

## Početak rada na Linux

1. **Instalirajte Chloros** — pogledajte [Linux Installation](linux-installation.md) za instalaciju `.deb`
2. **Provjera** — `chloros-cli --version` ispisuje `Chloros CLI 1.2.0`; `chloros-cli selftest` pokreće dijagnostiku u 7 koraka
3. **Instalirajte PythonSDK** (po želji) — `pip install chloros-sdk`
4. **Prijava** — `chloros-cli login your@email.com 'your-password'` (jednom po računalu i ponovno nakon svake nadogradnje paketa)
5. **Obradite svoj prvi skup podataka** — `chloros-cli process ~/datasets/flight001`

Za NVIDIA Jetson pogledajte namjenski [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) za postavljanje specifično za platformu, toplinsko ponašanje i terensku implementaciju.

***

## Sljedeći koraci

* [Instalacija Linux](linux-installation.md) — detaljna instalacija, lokacije datoteka i rješavanje problema za amd64 i arm64
* [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) — postavljanje specifično za Jetson, ponašanje memorije i toplinsko ponašanje, postavljanje na terenu
* [CLI : Command Line](../CLI.md) — vodič za CLI
* [API : Python SDK](../api-python-sdk.md) — vodič za SDK
* [Referenca za CLI](../reference/cli-reference.md) i [Referenca za SDK](../reference/sdk-reference.md) — iscrpne liste naredbi / API za verziju 1.2.0
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagođava vašem hardveru

{% hint style="info" %}
**Programsko čitanje ovog priručnika.** Svaka stranica također se poslužuje kao sirovi Markdown na svojoj vlastitoj adresi URL plus `.md` (na primjer `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), a indeks cijelog priručnika objavljen je na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
