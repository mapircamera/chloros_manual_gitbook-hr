# Linux Pregled

Chloros 1.1.0 donosi nativnu Linux podršku za **CLI**i**Python SDK**, omogućujući bezglavu multispektralnu obradu slike na Linux radnim stanicama, poslužiteljima i NVIDIA Jetson rubni uređaji.

{% hint style="info" %}
**Nema GUI na Linux.** Chloros Desktop GUI dostupan je samo na Windows. Korisnici Linux komuniciraju s Chloros putem [CLI](../CLI.md) i [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matrica podrške za platformu

| Značajka | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop GUI** | Da | N/A | Ne | Ne |
| **CLI** | Da | Da | Da | Da |
| **Python SDK** | Da | Da | Da | Da |
| **GPU ubrzanje (CUDA)** | Da | Da | Da | Da (JetPack 6) |
| **Texture Aware Debayer** | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) | Da (Chloros+) |
| **Dinamička računalna prilagodba** | Da | Da | Da | Da |***

## Podržane arhitekture

| Arhitektura | Opis | Način instalacije |
| --- | --- | --- |
| **amd64 (x86_64)** | Standardni desktop/poslužiteljski procesori (Intel, AMD) | `.deb` paket |
| **arm64 (aarch64)** | ARM-bazirani procesori, prvenstveno NVIDIA Jetson | `.deb` paket (JetPack 6) |

## Podržane Linux distribucije

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson platforme)***

## Što Linux korisnici dobivaju

* **Chloros CLI** — Potpuno sučelje naredbenog retka za skupnu obradu, automatizaciju i skriptiranje
* **Chloros Python SDK** — Programsko sučelje Python (`pip install chloros-sdk`) za integraciju u istraživačke kanale i prilagođene alate
* **GPU ubrzanje** — CUDA-ubrzana obrada na NVIDIA GPU-ovima (stolno računalo i Jetson)
* **Dynamic Compute Adaptation** — Automatsko otkrivanje hardvera i optimizacija strategije obrade
* **Sve značajke obrade** — Isti višespektralni cjevovod za obradu kao Windows (kalibracija, ispravak vinjete, vegetacijski indeksi, svi izvozni formati)
* **Značajke Chloros+** — obrada u više niti, debayer s obzirom na teksture, prilagođeni indeksi (s licencom Chloros+)

## Što Linux korisnici ne dobivaju

* **Desktop GUI** — Nema grafičkog sučelja; sva interakcija je putem CLI ili Python SDK
* **Preglednik slika** — Nema interaktivnog preglednika slika, prikaza mreže ili oznaka na karti
* **Vizualno upravljanje projektima** — Projektima se upravlja preko CLI naredbi i SDK poziva***

## Početak rada s Linux

1. **Instalirajte Chloros** — Pogledajte [Instalacija Linux](linux-installation.md) za instalaciju paketa `.deb`
2. **Instalirajte Python SDK** (opcionalno) — `pip install chloros-sdk`
3. **Aktivirajte svoju licencu** — `chloros-cli login your@email.com 'password'`
4. **Obradite svoj prvi skup podataka** — `chloros-cli process ~/datasets/flight001`

Za korisnike NVIDIA Jetsona, pogledajte namjenski [Vodič za NVIDIA Jetson](nvidia-jetson-guide.md) za postavljanje i optimizaciju specifičnu za platformu.

***

## Sljedeći koraci

* [Instalacija Linux] (linux-installation.md) — Detaljne upute za instalaciju za amd64 i arm64
* [Vodič za NVIDIA Jetson] (nvidia-jetson-guide.md) — Jetson specifično postavljanje, upravljanje toplinom i implementacija na terenu
* [CLI : naredbeni redak](../CLI.md) — Puna referenca CLI
* [API : Python SDK](../api-python-sdk.md) — Potpuna referenca SDK
* [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md) — Kako se Chloros prilagođava vašem hardveru