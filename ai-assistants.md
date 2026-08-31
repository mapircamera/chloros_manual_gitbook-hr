# Korištenje Chloros s AI asistentima

Ovaj priručnik napisan je za dvije publike: ljude i AI asistente putem kojih ljudi sve više rade. Svaka stranica objavljuje točne vrijednosti, zadane postavke i naredbe koje se mogu kopirati i zalijepiti kako bi asistent (Claude, ChatGPT, Copilot, agent za kodiranje, …) mogao napisati funkcionalnu automatizaciju Chloros već pri prvom pokušaju.

Chloros verzija: **

1.2.0**. CLI / SDK platforme: Windows 10/11 x64 i Linux (x86_64 / Jetson aarch64).

## Što dati svom asistentu

| Resurs | URL | Za što se koristi |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Mašinski čitljiv indeks svake stranice u ovom priručniku. |
| **CLI Referenca** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Cjelovita referenca za sučelje naredbi `chloros-cli`: svaka naredba, zastavica, zadana vrijednost, izlazni kod i pravilo za izlaznu mapu. Napisano za upotrebu s LLM-om. |
| **Referenca za SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Potpuna referenca za `chloros_sdk` Python API: klase, potpisi, iznimke i obrađeni primjeri. Napisano za LLM. |
| **Bilo koja stranica kao sirovi Markdown** | dodajte `.md` na stranicu URL | npr. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vraća stranicu kao sirovi Markdown — idealno za lijepljenje u kontekst prozor ili dohvaćanje od agenta. |

Unutarnje poveznice u priručniku: [CLI Referenca](reference/cli-reference.md) · [SDK Referenca](reference/sdk-reference.md).

{% hint style="info" %}
Dvije referentne stranice su samostalne: asistent koji je pročitao jednu od njih ne treba ostatak priručnika za pisanje ispravnog skripta.
{% endhint %}

## Recepti za upis

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
Skriptanje DAQ-a s komandne linije uvijek prolazi kroz obitelj `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Ostale podzapovijedi `daq` koje vaš asistent može smisliti nisu dostupne u isporučenim izdanjima i dovode do izlaska s pogreškom.
{% endhint %}

## Zašto AI-napisane skripte dobro funkcioniraju s Chloros

Svaka od navedenih je stvarno, provjereno ponašanje sustava Chloros 1.2.0 — one uklanjaju klasične načine kvara automatizacije koju pišu strojevi:

* **Bez dodatnih postavljanja.**Pomoćnici za pametno povezivanje SDKa (`connect_camera`, `connect_array`, `connect_daq_sensor`) i ulazne točke obrade (`ChlorosLocal`, `process_folder`)**automatski pokreću lokalni backend**. Generirani skript ne treba otvorenu GUI ili ručno pokrenuti poslužitelj — potrebno mu je samo instalirano pakiranje desktop/CLI.
* **Cijeli je pipeline jedan poziv.** `chloros_sdk.process_folder("path", indices=["NDVI"])` pokreće uvoz → kalibraciju → reflektanciju → izvoz indeksa od početka do kraja. Manja površina, manje mjesta gdje generirani skripta može pogriješiti.
* **Pokretanja bez izlaza se samodijagnosticiraju.** Nakon `process()`, sažetak pokretanja priložen je rezultatu, i svaki napomeni za obradu (npr. *zašto* je izvršavanje proizvelo nulti rezultat) također se ponovno emitira kao Python `UserWarning` — tako da čak i skripta koja nikada ne pregledava rezultat dict površava dijagnostiku.
* **CLI bučno propada.**Pokretanje `chloros-cli process` koje je zatražilo proizvode, ali nije napisalo nijedan, ispisuje `Processing finished but wrote no image products.` i**izlazi s ne-nulovim izlaznim kodom**, pa ga shell skripte i CI otkrivaju jednostavnom provjerom izlaznog koda. Uspješni pokreti izvještavaju o `Image products written: N`.

Jedna asimetrija koju bi asistent trebao znati: `process()` u SDKu namjerno **ne** javlja pogrešku pri pokretanju bez rezultata — umjesto toga to prikazuje u sažetku/naznakama. Ako pipeline u Pythonu mora stati pri praznom pokretanju, provjerite sažetak (recept 2 to radi).

## Ograničenja

* **Potreban je Chloros+ prijavak.**Alati CLI i SDK zahtijevaju**plaćeni** Chloros+ paket, što se provjerava na strani poslužitelja: zahtjevi s `401 AUTH_REQUIRED` neće uspjeti ako niste prijavljeni, a s `403 PLAN_UPGRADE_REQUIRED` na besplatnom paketu. Pokrenite `chloros-cli login` jednom po računalu prije pokretanja generiranih skripti. Pogledajte [Chloros+ prijava](chloros+-login.md).
* **Naredbe za snimanje upravljaju stvarnim hardverom.** Naredbe `lattice` / `daq` / `project` i objekti sesije SDK povezuju se, prenose i aktiviraju fizičke kamere i senzore. Pregledajte generiranu skriptu prije njezina prvog pokretanja i pokrenite je uz prisustvo hardvera.
* **Povremeno provjerite izlazne podatke.** Provjerite mape proizvoda i nekoliko vrijednosti piksela prije objavljivanja rezultata. Posebno, TIFF-ovi reflektancije skalirani su prema izvoru — pročitajte XMP oznaku `Chloros:PixelScale` (LATTICE: 32768 = 1,0 reflektancija; Survey3: 65535) umjesto pretpostavljanja dijelitelja. Obje referentne stranice to dokumentiraju pod &quot;Reading reflectance pixels&quot;.
* **Male zamke koje mogu dovesti do pogrešaka u generiranom kodu:**`pool-record` piše na datotečni sustav**nazivnog računala** (nazivnog računala za pozadinske zadatke) (zadano `~/Documents/DAQ Live View/`); na računalima s više mrežnih sučelja, radije koristite `daq pool-connect --eth-host <ip-or-hostname>` nego automatsko otkrivanje; i koristite `http://127.0.0.1:5000` (nikada `localhost`) gdje god se pojavi backend URL.
