# Korištenje Chlorosa s AI asistentima

Ovaj priručnik napisan je za dvije publike: ljude i AI asistente putem kojih ljudi sve više rade. Svaka stranica objavljuje točne vrijednosti, zadane postavke i naredbe koje se mogu kopirati i zalijepiti kako bi asistent (Claude, ChatGPT, Copilot, agent za kodiranje, …) mogao napisati funkcionalnu automatizaciju pomoću Chlorosa iz prve.

Chloros verzija: **

1.3.0**. CLI/SDK platforme: Windows 10/11 x64 i Linux (x86_64 / Jetson aarch64).

## Što dati svom asistentu

| Resurs | URL | Za što se koristi |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Mašinski čitljiv indeks svake stranice u ovom priručniku. |
| **Referenca za**CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Potpuna komandska površina `chloros-cli`: svaka naredba, zastavica, zadana vrijednost, kod izlaska i pravilo za izlaznu mapu. Napisano za konzumaciju s LLM-om. |
| **SDK referenca** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Cjeloviti `chloros_sdk` Python API: klase, potpisi, iznimke i obrađeni primjeri. Napisano za LLM. |
| **Bilo koja stranica kao sirovi Markdown** | dodajte `.md` na URL stranice | npr. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vraća stranicu kao sirovi Markdown — idealno za lijepljenje u kontekst prozor ili dohvaćanje od agenta. |

Unutarnje poveznice u priručniku: [CLI referenca](reference/cli-reference.md) · [SDK referenca](reference/sdk-reference.md).

{% hint style="info" %}
Te dvije referentne stranice su samostalne: asistent koji je pročitao jednu od njih ne treba ostatak priručnika kako bi napisao ispravan skript.
{% endhint %}

## Neka vaš asistent izravno pokrene Chloros: `chloros-mcp`

Sve gore navedeno omogućuje vašem asistentu da *napis*e skriptu ili naredbu. Uz `chloros-mcp`, asistent koji govori [Model Context Protocol](https://modelcontextprotocol.io) (MCP), kao što su Claude Desktop, Claude Code ili Cursor, upravlja samim Chlorosom. Postavljate pitanja na jednostavan način, a on poziva alate poslužitelja:

* **Obrada slika:** stvaranje projekta, uvoz mape snimki, odabir postavki, obrada i izvještavanje o tome što je izvršeno izvoženje.
* **Pokretanje hardvera:** povezivanje LATTICE kamera, nizova i DAQ senzora svjetla, pregled, postavljanje ekspozicije i boje, snimanje, zabilježavanje i ažuriranje firmvera.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Alati poslužitelja pokrivaju ono što radi [Pythonov SDK](api-python-sdk.md). Instalira se zajedno s SDKom, pa obično nema ništa drugo za instalirati: na Windowsu instalacijski program Chlorosa postavlja `chloros-mcp.exe` u mapu `Scripts` vašeg Pythona, a instalacija za Linux stavlja `chloros-mcp` u `/usr/bin`.

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**dodajte ovo u `claude_desktop_config.json` (otvara ga**Postavke &gt; Programer &gt; Uredi konfiguraciju**), zatim ponovno pokrenite Claude Desktop.**Cursor** koristi isti unos u `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Ako klijent kaže da se poslužitelj nije uspio pokrenuti, `chloros-mcp` vjerojatno nije na `PATH` koji vidi: umjesto toga navedite punu putanju (`where chloros-mcp` na Windows, `which chloros-mcp` na Linux).

Morate imati instaliran Chloros na istom računalu i prijavljeni račun, u aplikaciji ili putem `chloros-cli login`, na bilo kojoj plaćenoj razini Chloros+ (Copper ili višoj). Poslužitelj se ne može prijaviti umjesto vas.

{% hint style="info" %}
Postavke klijenta za svaku aplikaciju, opcije, svaki alat i kako se izvršavanje ponaša: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) u referenci SDKa.
{% endhint %}

## Recepti za upite

Kopirajte, ispunite `<placeholders>` i zalijepite u svoj asistent.

### 1. Obradite mapu leta u NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Serijska obrada direktorija snimki

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Povezivanje LATTICE niza i snimanje

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Snimanje spektra DAQ senzora svjetla

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skriptanje DAQ-a s komandne linije uvijek prolazi kroz obitelj `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Ostale podzapovijedi `daq` koje vaš asistent može smisliti nisu dostupne u isporučenim izdanjima i završavaju s pogreškom.
{% endhint %}

## Zašto skripte napisane umjetnom inteligencijom dobro funkcioniraju s Chlorosom

Svaka od navedenih stavki predstavlja stvarno, provjereno ponašanje sustava Chloros 1.3.0 — one uklanjaju klasične načine neuspjeha automatizacije napisane strojem:

* **Bez postavljanja.**Pomoćnici za pametno povezivanje SDKa (`connect_camera`, `connect_array`, `connect_daq_sensor`) i ulazne točke obrade (`ChlorosLocal`, `process_folder`)**automatski pokreću lokalni backend**. Generirani skript ne treba otvorenu GUI ili ručno pokrenuti poslužitelj — potrebno mu je samo instalirano desktop/CLI pakiranje.
* **Cijeli je pipeline jedan poziv.** `chloros_sdk.process_folder("path", indices=["NDVI"])` pokreće uvoz → kalibraciju → reflektanciju → izvoz indeksa od početka do kraja. Manja površina, manje mjesta za pogrešku generirane skripte.
* **Pokretanja bez izlaza se samodijagnosticiraju.** Nakon `process()`, sažetak pokretanja priložen je rezultatu, i svaki napomeni za obradu (npr. *zašto* je izvršavanje proizvelo nula izlaza) također se ponovno emitira kao Python `UserWarning` — tako da čak i skripta koja nikada ne pregledava result dict dobiva uvid u dijagnostiku.
* **CLI se glasno ne uspijeva.**Pokretanje `chloros-cli process` koje je zatražilo proizvode, ali nije napisalo nijedan, ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nuljevim rezultatom**, pa ga shell skripte i CI otkrivaju jednostavnom provjerom koda izlaska. Uspješne izvedbe izvještavaju o `Image products written: N`, plus `(plus M clamp masks)` kada su maske napisane. Naredbe za snimanje također navode svaku datoteku koju nisu mogle napisati i izlaze s kodom izlaska `1`.
* **Relativne putanje znače ono što skripta očekuje.** Kod lokalnog backenda, relativna putanja za izlaz (`-o out/` u CLI-u, `output_dir="out/"` u SDK-u) rješava se u odnosu na mapu iz koje se naredba ili skripta pokreće.

Jedna asimetrija koju bi asistent trebao znati: `process()` u SDKu namjerno **ne** podiže iznimku pri pokretanju bez rezultata — umjesto toga to prijavljuje putem sažetka/naznaka. Ako pipelajn u Pythonu mora stati na praznom pokretanju, provjerite sažetak (recept 2 to čini).

## Ograničenja

* **Potreban je prijavak na Chloros+.**CLI i SDK zahtijevaju**plaćeni** Chloros+ paket, što se provjerava na strani poslužitelja: zahtjevi ne uspijevaju s porukom `401 AUTH_REQUIRED` ako niste prijavljeni i s porukom `403 PLAN_UPGRADE_REQUIRED` na besplatnom paketu. Pokrenite `chloros-cli login` jednom po računalu prije pokretanja generiranih skripti. Pogledajte [Chloros+ prijavu](chloros+-login.md).
* **Naredbe za snimanje upravljaju stvarnim hardverom.** `lattice` / `daq` / zapovijedi `project`, objekti sesije SDKa i alati `chloros-mcp` povezuju se, prenose i aktiviraju fizičke kamere i senzore. Pregledajte generiranu skriptu prije njezina prvog pokretanja i pokrenite je uz prisutnost hardvera.
* **Provjerite izlazne podatke.** Provjerite mape proizvoda i nekoliko vrijednosti piksela prije objavljivanja rezultata. Konkretno, TIFF datoteke s reflektancijom skaliraju se po izvoru — pročitajte XMP oznaku `Chloros:PixelScale` (datoteke LATTICE je nose: 32768 = 1,0 reflektancija; datoteke Survey3 to ne čine i koriste 65535 = 1,0) umjesto pretpostavljanja dijelitelja. Obje referentne stranice to dokumentiraju pod &quot;Reading reflectance pixels&quot;.
* **Male zamke koje mogu dovesti do problema s generiranim kodom:**`pool-record` piše na datotečni sustav**nazivnog računala** (nazivno `~/Documents/DAQ Live View/`); na računalima s više mrežnih sučelja, radije koristite `daq pool-connect --eth-host <ip-or-hostname>` nego automatsko otkrivanje; koristite `http://127.0.0.1:5000` (nikada `localhost`) gdje god se pojavi URL pozadinskog sustava; skripta koja prolazi kroz izlazno stablo projekta trebala bi preskočiti mape `clamp_masks/` (njihove `*_clampmask.tif` datoteke su maske, a ne slike — vidi [Format izlaznih slika](output-image-formats.md#clamp-masks)); i na Windowsu s isključenim dugim putanjama, držite foldere projekta kratkima — folder projekta dublji od otprilike 130 znakova odbijen je prije početka obrade.
