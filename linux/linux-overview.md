Pregled #Linuxa

Chloros 1.3.0 pruža nativnu podršku za **CLI**i**Python SDK** na Linux radnim postajama, poslužiteljima i NVIDIA Jetson rubnim uređajima — za bezglavnu obradu višespektralnih slika, uz upravljanje LATTICE kamerom uživo i DAQ senzorom svjetla.

{% hint style="info" %}
**Nema desktop GUI-ja na Linuxu.**Desktop GUI aplikacije Chloros dostupan je samo za Windows. Korisnici Linuxa komuniciraju s aplikacijom Chloros putem [CLIa](../CLI.md) i [Python SDK-a](../api-python-sdk.md). `.deb` dodaje stavku**Chloros** u izbornik vaše aplikacije — jednostavno otvara emulator terminala koji pokreće `chloros-cli`.
{% endhint %}

***

## Matrica podrške platformama

| Značajka | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop GUI** | Da | N/A | Ne | Ne |
| **CLI** (`chloros-cli`) | Da | Da | Da | Da |
| **Python SDK** (`chloros-sdk`) | Da | Da | Da | Da |
| **Proces obrade slika** | Da | Da | Da | Da |
| **Upravljanje kamerom LATTICE (uživo)** | Da (kartica Kamere) | Da (`chloros-cli lattice`, SDK) | Da | Da |
| **DAQ senzori svjetla (uživo)** | Da (zavičaj Senzori svjetla) | Da (`chloros-cli daq pool-*`, SDK) | Da | Da |
| **PTP vremenska sinkronizacija (host je grandmaster)** | Da | Da (`chloros-cli time-sync`) | Da | Da |
| **GPU akceleracija (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Debayer s podrškom za teksture** (potreban je CUDA GPU) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Dinamička prilagodba izračuna** | Da | Da | Da | Da |
| **Backend kao sustavna usluga** (`chloros-backend@<user>.service`) | Ne | Ne | Da (po želji) | Da (po želji) |
| **Ažuriranje na licu mjesta** (`chloros-cli update`) | Ne (pokrenite instalacijski program) | Ne (pokrenite instalacijski program) | Da | Da |

***

## Podržane arhitekture

| Arhitektura | Opis | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardni desktop/server procesori (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM procesori — obitelj NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Podržane distribucije sustava Linux

* **Ubuntu 22.04 LTS ili noviji** (amd64)
* **Debian 12 ili noviji** (amd64)
* **NVIDIA JetPack 6** (arm64 — platforme Jetson Orin)

***

## Što korisnici Linuxa dobivaju

* **Chloros CLI** — potpuno sučelje naredbenog retka za serijsku obradu, automatizaciju i skriptiranje
* **Chloros Python SDK** — programski Python sučelje za istraživačke cjevovode i prilagođene alate, instalira ga `.deb` za sustav `python3` (verzija kotačića usklađena s verzijom za virtualenvs nalazi se u `/usr/lib/chloros/sdk/`; PyPI je za računala bez paketa Chloros)
* **Upravljanje kamerom LATTICE** — otkrijte, povežite, konfigurirajte i snimajte s kamera LATTICE i sinkroniziranih nizova više kamera putem `chloros-cli lattice` i SDK-a; `.deb` uključuje izvršno okruženje Arena SDK-a koje kamere zahtijevaju
* **Upravljanje DAQ senzorima svjetla** — povežite DAQ-U/M/E senzore, streamajte kalibrirana spektra i snimajte `.daq` datoteke putem `chloros-cli daq pool-*` i SDK
* **PTP sinkronizacija vremena** — pozadinski program Chloros pokreće PTP grandmasteru kojemu se kamere LATTICE i DAQ-E senzori priključuju kao sluge; inspektirati ga pomoću `chloros-cli time-sync` i održavati ga u pozadini bez grafičkog sučelja pomoću sustavne jedinice systemd `chloros-backend@<user>.service` (vidi [Linux instalacija](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automacija projekta** — pokrenite spremljene projekte bez zaslona pomoću `chloros-cli project` i `open_project` iz SDKa
* **GPU akceleracija** — obrada ubrzana CUDA-om na NVIDIA GPU-ima (desktop GPU-i na amd64 i GPU Jetsona s JetPackom 6)
* **Dinamička prilagodba izračuna** — automatska detekcija hardvera i odabir strategije obrade, s mogućnošću preuzimanja kontrole putem `CHLOROS_STRATEGY` za napredne korisnike
* **Sve značajke obrade** — isti procesni put kao i na Windows: kalibracija, korekcija vinjete, indeksi vegetacije i svaki format izvoza
* **Značajke Chloros+** — višenitna (pipeline) obrada, debayer s podrškom za teksture (na CUDA GPU-u) i prilagođeni indeksi, uz plaćeni Chloros+ plan

## Što korisnici Linuxa ne dobivaju

* **Desktop GUI** — nema grafičkog sučelja; sva interakcija je putem CLIa ili Python SDKa
* **Preglednik slika** — nema interaktivnog preglednika slika, prikaza mreže ili oznaka na karti
* **Vizualno upravljanje projektima**— projekti se stvaraju i pokreću putem naredbi**CLI**i poziva**SDK** (sam hardver — kamere, senzori, snimanje — ostaje u potpunosti kontroliran iz terminala)

***

## Zahtjev za licencu

Pristup putem CLIa i SDKa zahtijeva **plaćeni Chloros+ paket — Copper ili viši**(Copper, Bronze, Silver, Gold). Besplatni**Iron** paket nema pristup putem CLIa/SDKa. Ovo ograničenje provodi pozadinski sustav (backend), a ne samo CLI:

| Situacija | Odgovor pozadinskog sustava (backend) |
| --- | --- |
| Nalog nije prijavljen | `401` s `error_code: AUTH_REQUIRED` |
| Prijavljeno na besplatnom Iron nivou | `403` s `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` radi na bilo kojoj razini — to je jedina ruta izuzeta od vrata — stoga je razlog odbijanja uvijek vidljiv.

***

## Početak rada na Linux

1. **Instalirajte Chloros** — pogledajte [Instalacija za Linux](linux-installation.md) za instalaciju `.deb`
2. **Provjera** — `chloros-cli --version` ispisuje `Chloros CLI 1.3.0`; `chloros-cli selftest` pokreće dijagnostiku u 7 koraka
3. **Provjerite Python SDK** — `.deb` ga instalira: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (u virtualnom okruženju, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Prijavite se** — `chloros-cli login your@email.com 'your-password'` (jednom po računalu; nadogradnje paketa zadržavaju prijavu, a uklanjanje paketa je uklanja)
5. **Obradite svoj prvi skup podataka** — `chloros-cli process ~/datasets/flight001`

Za NVIDIA Jetson pogledajte namjenski [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) za postavljanje specifično za platformu, toplinsko ponašanje i terensku implementaciju.

***

## Sljedeći koraci

* [Linux instalacija](linux-installation.md) — detaljna instalacija, lokacije datoteka i rješavanje problema za amd64 i arm64
* [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) — postavljanje specifično za Jetson, ponašanje memorije i toplinsko ponašanje, postavljanje na terenu
* [CLI : naredbeni redak](../CLI.md) — vodič za CLI
* [API : Python SDK](../api-python-sdk.md) — vodič za SDK
* [Referenca za CLI](../reference/cli-reference.md) i [Referenca za SDK](../reference/sdk-reference.md) — iscrpan popis naredbi/APIa za verziju 1.3.0
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — kako se Chloros prilagođava vašem hardveru

{% hint style="info" %}
**Programsko čitanje ovog priručnika.** Svaka stranica također se poslužuje kao sirovi Markdown na svojoj vlastitoj URL adresi plus `.md` (na primjer `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), a indeks cijelog priručnika objavljen je na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
