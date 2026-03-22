---
metaLinks: {}
---

# Prvi koraci

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros je softverska aplikacija tvrtke [MAPIR](https://www.mapir.camera) za obradu slika i drugih podataka senzora.

***{% hint style="success" %}**Što je novo u Chloros 1.1.0**: izvorna Linux podrška (amd64 i arm64), NVIDIA Jetson rubno računalstvo, dinamička računalna adaptacija, 4-nitni procesni cjevovod, nove CLI naredbe i opcije. Pogledajte [Preuzmi](download.md) za cijeli dnevnik promjena.
{% endhint %}

Chloros dostupan je u 3 načina primjene:

## Chloros: Desktop GUI aplikacija

Samostalni zasebni prozor sa svim značajkama. _Samo Windows._

## [Chloros CLI: sučelje naredbenog retka](CLI.md)

Skupna obrada naredbenog retka. Savršeno za automatizaciju, skriptiranje i rad bez glave. Dostupno na **Windows, Linux amd64 i Linux arm64 (NVIDIA Jetson)**. _CLI zahtijeva licencu Chloros+ za pristup._

## [Chloros API: Python SDK](api-python-sdk.md)

Programsko sučelje Python za automatizaciju i prilagođene tijekove rada. Savršeno za istraživačke kanale, integraciju s postojećim Python aplikacijama i izradu prilagođenih alata. Dostupno na **svim platformama** putem `pip install chloros-sdk`. _API zahtijeva licencu Chloros+ za pristup._***

## Podržane platforme

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Da | Da | Da |
| **Linux amd64 (x86_64)** | Ne | Da | Da |
| **Linux arm64 (NVIDIA Jetson)** | Ne | Da | Da |

Za upute o instalaciji Linux pogledajte odjeljak [Linux & Edge Computing](linux/linux-overview.md).

***

## Chloros+

Iako je Chloros besplatan za korištenje za većinu zadataka, možda ćete poželjeti više. Tu vam plaćena licenca za Chloros+ može koristiti. S Chloros+ licencom možete otključati nove značajke kao što su:

* **Višenitna obrada**: uvelike ubrzajte obradu slika za veće projekte istovremenom obradom slika kroz cjevovod.
* **GPU (CUDA) ubrzanje**: iskoristite prednosti današnjih opcija veće GPU memorije kako biste dodatno ubrzali proces obrade slike. Za najbolje rezultate preporučujemo 4 GB ili više VRAM-a.
* **Chloros+**[**CLI**](CLI.md)**Pristup**: pokrenite Chloros+ iz naredbenog retka za automatizaciju i integraciju u vlastiti softver.
* **Chloros+**[**API**](api-python-sdk.md)**Pristup:** pokrenite Chloros+ iz Python za programsku kontrolu, omogućujući besprijekornu integraciju s vašim istraživačkim kanalima, tijekovima rada analize podataka i prilagođenim aplikacijama.
* **Korištenje više uređaja**: svaka Chloros+ licenca omogućuje registraciju 2+ uređaja. Koristite svoj MAPIR Cloud račun za upravljanje registriranim uređajima. Dodajte podršku za više uređaja nadogradnjom licence Chloros+.
* **Advanced Texture Aware Debayer Method:** visokokvalitetni debayer s obzirom na rubove u kombinaciji s AI/ML modelom uklanjanja šuma koji uklanja gotovo sav šum debayeringa.&#x20;
* **Prilagođene multispektralne indeksne formule:** unesite prilagođene multispektralne indekse u Chloros raster kalkulatore, kako za obradu tako i za gledanje slika u sandboxu.
* **Linux & Edge Computing:** pokrenite Chloros na Linux x86\_64 i ARM64 platformama uključujući NVIDIA Jetson za terensku i rubnu obradu. Pogledajte [Linux Pregled](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Cijene &#x26; Prijava</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>