# CLI: Naredbeni redak

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** pruža moćan pristup naredbenog retka mehanizmu za obradu slike Chloros, omogućujući automatizaciju, skriptiranje i izravne operacije za vaše tijekove rada sa slikama.

### Glavne značajke

* 🚀 **Automatizacija**: skriptirana skupna obrada više skupova podataka
* 🔗 **Integracija**: Integrira se u postojeće tijekove rada i procese
* 💻 **Rad bez GUI-a**: radi bez GUI-ja
* 🌍 **Višejezično**: Podržava 38 jezika
* ⚡ **Paralelna obrada**: Dinamički se prilagođava vašem CPU-u (do 16 paralelnih radnika).

### Zahtjevi

| Zahtjev | Detalji |
| -------------------- | ------------------------------------------------------------------------------- |
|**Operativni sustav**| Windows 10/11 (64-bitni) |
|**Licenca**| Chloros+ ([potreban plan plaćanja](https://cloud.mapir.camera/pricing)) |
|**Sjećanje**| Minimalno 8 GB RAM-a (preporučuje se 16 GB) |
|**Internet**| Potrebno za aktivaciju licence |
|**Prostor na disku**| Razlikuje se ovisno o veličini projekta |

{% hint style=&quot;upozorenje&quot; %}**Zahtjevi licence**: CLI zahtijeva plaćenu pretplatu na Chloros+. Standardni (besplatni) planovi nemaju pristup CLI-ju. Posjetite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) za ažuriranje.
{% endhint %}

## Brzi početak

### Objekt

CLI je automatski uključen u Chloros instalacijski program:

1. Preuzmite i pokrenite**Chloros Installer.exe**2. Dovršite čarobnjaka za instalaciju
3. CLI instaliran na: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;uspjeh&quot; %}
Instalacijski program automatski dodaje `chloros-cli` u PATH vašeg sustava. Ponovno pokrenite terminal nakon instalacije.
{% endhint %}

### Početno postavljanje

Prije korištenja CLI-ja, aktivirajte svoju Chloros+ licencu:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Osnovna uporaba

Obradi mapu sa zadanim postavkama:

```powershell
chloros-cli process "C:\Images\Dataset001"
```***

## Referenca naredbi

### Opća sintaksa

```
chloros-cli [global-options] <command> [command-options]
```

***## Naredbe

### `process`: obradite slike

Obradite slike u mapi s kalibracijom.**Sintaksa:**```bash
chloros-cli process <input-folder> [options]
```**Primjer:**```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opcije naredbe za obradu

| Opcija | Upišite | Zadano | Opis |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------- |
| `<input-folder>` | Ruta | _Obavezno_ | Mapa koja sadrži RAW/JPG multispektralne slike |
| `-o, --output` | Ruta | Isto kao unos | Izlazna mapa za obrađene slike |
| `-n, --project-name` | Lanac | Automatski generirano | Prilagođeni naziv projekta |
| `--vignette` | Indikator | Omogućeno | Omogući ispravak vinjete |
| `--no-vignette` | Indikator | - | Onemogući korekciju vinjete |
| `--reflectance` | Indikator | Omogućeno | Omogući kalibraciju refleksije |
| `--no-reflectance` | Indikator | - | Onemogući kalibraciju refleksije |
| `--ppk` | Indikator | Onemogućeno | Primijeni PPK korekcije iz podataka svjetlosnog senzora .daq |
| `--format` | Opcija | TIFF (16 bita) | Izlazni format: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size` | Cijeli broj | Automatski | Minimalna veličina cilja u pikselima za otkrivanje kalibracijske ploče |
| `--target-clustering` | Cijeli broj | Automatski | Prag ciljnog grupiranja (0-100) |
| `--exposure-pin-1` | Lanac | Ništa | Zaključaj ekspoziciju za model kamere (Pin 1) |
| `--exposure-pin-2` | Lanac | Ništa | Zaključaj ekspoziciju za model kamere (Pin 2) |
| `--recal-interval` | Cijeli broj | Auto | Interval rekalibracije u sekundama |
| `--timezone-offset` | Cijeli broj | 0 | Vremensko odstupanje u satima |***

### `login` - Autentifikacija računa

Prijavite se svojim vjerodajnicama za Chloros+ kako biste omogućili CLI obradu.

**Sintaksa:**```bash
chloros-cli login <email> <password>
```**Primjer:**```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;upozorenje&quot; %}**Posebni znakovi**: Koristite jednostruke navodnike oko zaporki koje sadrže znakove kao što su `$`, `!` ili razmake.
{% endhint %}**Proizlaziti:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout`: brisanje vjerodajnica

Izbrišite pohranjene vjerodajnice i odjavite se sa svog računa.**Sintaksa:**```bash
chloros-cli logout
```**Primjer:**```powershell
chloros-cli logout
```**Izlaz:**```
✓ Logout successful
ℹ Credentials cleared from cache
```***

### `status` - Provjerite status licence

Prikazuje trenutnu licencu i status provjere autentičnosti.

**Sintaksa:**```bash
chloros-cli status
```**Primjer:**```powershell
chloros-cli status
```**Izlaz:**```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```***

### `export-status`: Provjerite napredak izvoza

Prati napredak izvoza niti 4 tijekom ili nakon obrade.

**Sintaksa:**```bash
chloros-cli export-status
```**Primjer:**```powershell
chloros-cli export-status
```**Slučaj upotrebe:**Pozovite ovu naredbu dok je obrada u tijeku da provjerite napredak izvoza.***

### `language`: upravljajte jezikom sučelja

Pogledajte ili promijenite jezik CLI sučelja.

**Sintaksa:**```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```**Primjeri:**```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Podržani jezici (ukupno 38)

| Kod | Jezik | Domaće ime |
| ------- | --------------------- | ---------------- |
| `en` | engleski | engleski |
| `es` | španjolski | španjolski |
| `pt` | portugalski | portugalski |
| `fr` | francuski | francuski |
| `de` | njemački | njemački |
| `it` | talijanski | talijanski |
| `ja` | japanski | 日本語 |
| `ko` | korejski | 한국어 |
| `zh` | Kineski (pojednostavljeni) | 简体中文 |
| `zh-TW` | Kineski (tradicionalni) | 繁體中文 |
| `ru` | ruski | ruski |
| `nl` | nizozemski | Nizozemska |
| `ar` | arapski | العربية |
| `pl` | poljski | poljski |
| `tr` | turski | Turkce |
| `hi` | hindski | हिंदी |
| `id` | indonezijski | Bahasa Indonezija |
| `vi` | vijetnamski | Tiếng Việt |
| `th` | tajlandski | ไทย |
| `sv` | švedski | Svenska |
| `da` | danski | Dansk |
| `no` | norveški | Norsk |
| `fi` | finski | Suomi |
| `el` | grčki | Ελληνικά |
| `cs` | češki | Ceština |
| `hu` | mađarski | mađarski |
| `ro` | rumunjski | Română |
| `uk` | ukrajinski | Ukrajinska |
| `pt-BR` | brazilski portugalski | brazilski portugalski |
| `zh-HK` | kantonski | 粵語 |
| `ms` | malajski | Bahasa Melayu |
| `sk` | slovački | Slovenčina |
| `bg` | bugarski | Bʺlgarski |
| `hr` | hrvatski | Hrvatski |
| `lt` | litvanski | Lietuvių |
| `lv` | latvijski | Latviešu |
| `et` | estonski | Eesti |
| `sl` | slovenski | Slovenščina |

{% hint style=&quot;uspjeh&quot; %}**Automatska postojanost**: Vaša jezična postavka sprema se u `~/.chloros/cli_language.json` i traje kroz sesije.
{% endhint %}***### `set-project-folder`: Postavite zadanu mapu projekta

Promijenite zadanu lokaciju mape projekta (dijeli se s GUI-jem).

**Sintaksa:**```bash
chloros-cli set-project-folder <folder-path>
```**Primjer:**```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```***

### `get-project-folder`: prikaži mapu projekta

Prikazuje trenutnu zadanu lokaciju mape projekta.

**Sintaksa:**```bash
chloros-cli get-project-folder
```**Primjer:**```powershell
chloros-cli get-project-folder
```**Izlaz:**```
ℹ Current project folder: C:\Projects\2025
```***

### `reset-project-folder`: vraćanje na zadane vrijednosti

Ponovno postavlja mapu projekta na zadanu lokaciju.

**Sintaksa:**```bash
chloros-cli reset-project-folder
```***

## Globalne opcije

Ove se opcije odnose na sve naredbe:

| Opcija | Upišite | Zadano | Opis |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Ruta | Automatski otkriveno | Put do pozadinske izvršne |
| `--port` | Cijeli broj | 5000 | API pozadinski port broj |
| `--restart` | Indikator | - | Prisilno ponovno pokretanje pozadine (ubija postojeće procese) |
| `--version` | Indikator | - | Prikaži informacije o verziji i izađi |
| `--help` | Indikator | - | Prikaži informacije za pomoć i izađi |

**Primjer s globalnim opcijama:**```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```***

## Vodič za konfiguraciju obrade

### Paralelna obrada

Chloros+ CLI **automatski skalira**paralelnu obradu kako bi odgovarao mogućnostima vašeg računala:**Kako radi:**

* Otkriva CPU i RAM jezgre.
* Dodjeljuje radnike: **2× CPU jezgre** (koristi hipernitnost).
* **Maksimalno: 16 radnika paralelno**(za veću stabilnost).**Razine sustava:**| Vrsta sustava | CPU | RAM | Radnici | Izvedba |
| ------------- | ---------- | -------- | -------- | --------------- |
|**Visoki raspon**| 16+ jezgri | 32+ GB | Do 16 | Najveća brzina |
|**Srednje klase**| 8-15 jezgri | 16-31 GB | 8-16 | Izvrsna brzina |
|**Niski raspon**| 4-7 jezgri | 8-15 GB | 4-8 | Dobra brzina |

{% hint style=&quot;uspjeh&quot; %}**Automatska optimizacija**: CLI automatski otkriva specifikacije vašeg sustava i konfigurira optimalnu paralelnu obradu. Nije potrebna ručna konfiguracija!
{% endhint %}

### Debayerove metode

CLI koristi**High Quality (Faster)**kao zadani i preporučeni Debayer algoritam:

| Metoda | Kvaliteta | Brzina| Opis |
| ---------------------------- | ------- | ----- | ---------------------------------------------- |
|**Visoka kvaliteta (brže)**⭐ | ⭐⭐⭐⭐ | ⚡⚡⚡ | Algoritam osjetljiv na rubove (zadano, preporučeno) |

### Ispravak vinjete**Što radi:** ispravlja gubitak svjetla na rubovima slike (tamniji kutovi uobičajeni na slikama fotoaparata).

* **Omogućeno prema zadanim postavkama**: većina korisnika trebala bi držati ovu opciju uključenom.
* Koristite `--no-vignette` da biste ga onemogućili.

{% hint style=&quot;uspjeh&quot; %}
**Preporuka**: Uvijek uključite korekciju vinjete kako biste osigurali jednoliku svjetlinu u cijelom okviru.
{% endhint %}

### Kalibracija refleksije

Pretvara neobrađene vrijednosti senzora u standardizirane postotke refleksije pomoću kalibracijskih ploča.

* **Omogućeno prema zadanim postavkama** – Neophodno za analizu vegetacije.
*Zahtijeva kalibracijske ploče na slikama.
* Koristite `--no-reflectance` da biste ga onemogućili.

{% hint style=&quot;info&quot; %}
**Zahtjevi**: provjerite jesu li kalibracijske ploče ispravno eksponirane i vidljive na vašim slikama radi točne konverzije refleksije.
{% endhint %}

### PPK popravci**Što radi:** Primjenjuje naknadno obrađene kinematičke korekcije pomoću podataka zapisnika DAQ-A-SD radi poboljšanja točnosti GPS-a.

* **Onemogućeno prema zadanim postavkama**
* Koristite `--ppk` da ga aktivirate
* Zahtijeva .daq datoteke u mapi projekta senzora svjetla MAPIR DAQ-A-SD.

### Izlazni formati

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bit dubina</th><th width="116.5999755859375">Veličina datoteke</th><th>Idealno za</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bitni cijeli broj</td><td>Veliki</td><td>GIS analiza, fotogrametrija (preporučeno)</td></tr><tr><td><strong>TIFF (32-bitni, postotak)</strong></td><td>32-bitni pokretni zarez</td><td>Vrlo veliki</td><td>Znanstvena analiza, istraživanje</td></tr><tr><td><strong>PNG (8-bitni)</strong></td><td>8-bitni cijeli broj</td><td>Srednji</td><td>Vizualni pregled, web dijeljenje</td></tr><tr><td><strong>JPG (8-bitni)</strong></td><td>8-bitni cijeli broj</td><td>Mali</td><td>Brzo pregled, komprimirani izlaz</td></tr></tbody></table>***

## Automatizacija i skriptiranje

### Skupna obrada PowerShell-a

Automatski obradi više mapa skupova podataka:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows batch skripta

Jednostavna petlja za skupnu obradu:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Python skripta za automatizaciju

Napredna automatizacija s upravljanjem greškama:

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***## Tijek obrade

### Standardni tijek rada

1.**Unos**: Mapa koja sadrži parove RAW/JPG slika
2.**Otkrivanje**: CLI automatski traži kompatibilne slikovne datoteke
3.**Procesiranje**: Paralelni način rada prilagođava se vašim CPU jezgrama (Chloros+)
4.**Izlaz**: Stvorite podmape po modelu fotoaparata s obrađenim slikama

### Primjer izlazne strukture

```
MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Procjene vremena obrade

Uobičajena vremena obrade za 100 slika (12 MP svaka):

| Način | Vrijeme | Hardver |
| ----------------- | --------- | ---------------------------------------------- |
|**Paralelni način rada**| 5-10 min | i7/Ryzen 7, 16GB RAM, SSD (do 16 radnika) |
|**Paralelni način rada**| 10-15 min | i5/Ryzen 5, 8 GB RAM, HDD (do 8 radnika) |

{% hint style=&quot;info&quot; %}**Savjet za performanse**: Vrijeme obrade varira ovisno o broju slika, razlučivosti i specifikacijama računala.
{% endhint %}***## Rješavanje problema

### CLI nije pronađen

**Pogreška:**```
'chloros-cli' is not recognized as an internal or external command
```**Rješenja:**

1. Provjerite mjesto instalacije:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Koristite puni put ako nije u PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ručno ga dodajte u PATH:
   *Otvorite Svojstva sustava → Varijable okoline.
   * Uredite varijablu PATH.
   * Dodaj: `C:\Program Files\Chloros\resources\cli`
   * Ponovno pokrenite terminal.

***### Pogreška pri pokretanju pozadine.**Pogreška:**```
Backend failed to start within 30 seconds
```**Rješenja:**1. Provjerite je li pozadina već pokrenuta (prvo je zatvorite).
2. Provjerite da ga Windows vatrozid ne blokira.
3. Pokušajte s drugim priključkom:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Prisilno ponovno pokrenite pozadinu:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```***

### Problemi s licencom/provjerom autentičnosti

**Pogreška:**```
Chloros+ license required for CLI access
```**Rješenja:**1. Provjerite imate li aktivnu Chloros+ pretplatu.
2. Prijavite se svojim vjerodajnicama:

```powershell
chloros-cli login user@example.com 'password'
```

3. Provjerite status licence:

```powershell
chloros-cli status
```

4. Kontaktirajte podršku: info@mapir.camera***

### Slike nisu pronađene.

**Pogreška:**```
No images found in the specified folder
```**Rješenja:**1. Provjerite sadrži li mapa podržane formate (.RAW, .TIF, .JPG).
2. Provjerite je li putanja mape točna (upotrijebite navodnike za staze s razmacima).
3. Provjerite imate li dopuštenja za čitanje mape.
4. Provjerite jesu li ekstenzije datoteka ispravne.***

### Obrada se zaustavlja ili prekida

**Rješenja:**1. Provjerite raspoloživi prostor na disku (provjerite ima li dovoljno za izlaz).
2. Zatvorite ostale aplikacije kako biste oslobodili memoriju.
3. Smanjite broj slika (serijski proces).***

### Port se već koristi

**Pogreška:**```
Port 5000 is already in use
```**Otopina:**Navedite drugi port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```***

## Često postavljana pitanja

### P: Trebam li licencu za CLI?

**O:**Da! CLI zahtijeva**plaćenu Chloros+ licencu**.

* ❌ Standardni plan (besplatno): CLI onemogućen
* ✅ Chloros+ planovi (plaćeni): CLI potpuno omogućen

Pretplatite se na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***### P: Mogu li koristiti CLI na poslužitelju bez GUI-ja?**O:** Da! CLI radi potpuno bez grafičkog sučelja. Zahtjevi:

* Windows Server 2016 ili noviji
* Visual C++ Redistributable instaliran
* Dovoljno RAM-a (minimalno 8 GB, preporučeno 16 GB)
* Jednokratna aktivacija GUI licence na bilo kojem računalu

***### P: Gdje se spremaju obrađene slike?**O:**Prema zadanim postavkama, obrađene slike spremaju se u**istu mapu kao i ulaz**u podmapama modela fotoaparata (npr. `Survey3N_RGN/`).

Upotrijebite opciju `-o` da odredite drugu izlaznu mapu:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```***

### P: Mogu li obraditi više mapa odjednom?

**O:**Ne izravno s jednom naredbom, ali možete koristiti skripte za uzastopnu obradu mapa. Pogledajte odjeljak [Automatizacija i skriptiranje](CLI.md#automation--scripting).***

### P: Kako mogu spremiti CLI izlaz u datoteku dnevnika?

**PowerShell:**```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```**Serija:**```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```***

### P: Što se događa ako pritisnem Ctrl+C tijekom obrade?

**A:**CLI će učiniti sljedeće:

1. Zaustavit će obradu na uredan način.
2. Isključit će pozadinu.
3. Izaći će s kodom 130.

Djelomično obrađene slike mogu ostati u izlaznoj mapi.***

### P: Mogu li automatizirati CLI obradu?

**O:**Naravno! CLI je dizajniran za automatizaciju. Pogledajte [Automatizacija i skriptiranje](CLI.md#automation--scripting) za PowerShell, Batch i Python primjere.***

### P: Kako mogu provjeriti CLI verziju?

**A:**```powershell
chloros-cli --version
```**Izlaz:**```
Chloros CLI 1.0.2
```***

## Potražite pomoć

### Pomoć za naredbeni redak

Pogledajte informacije o pomoći izravno u CLI-ju:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanali podrške

* **Email**: info@mapir.camera
* **Web stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cijene**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***## Potpuni primjeri

### Primjer 1: Osnovna obrada

Obrada sa zadanim postavkama (vinjeta, refleksija):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Primjer 2: Visokokvalitetni znanstveni rezultat

32-bitni plutajući TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```***### Primjer 3: Brza obrada pregleda

8-bitni PNG bez kalibracije za brzi pregled:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Primjer 4: Ispravljena obrada s PPK

Primijenite PPK korekcije s refleksijom:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```***### Primjer 5: Lokacija prilagođenog izlaza

Obradite na drugom pogonu s određenim formatom:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Primjer 6: Tijek rada provjere autentičnosti

Potpuni tijek provjere autentičnosti:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```***

### Primjer 7: Višejezična upotreba

Promjena jezika sučelja:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```