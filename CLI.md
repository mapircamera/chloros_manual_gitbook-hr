# CLI : Naredbeni redak

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

**Chloros CLI** pruža moćan pristup s naredbenog retka mehanizmu za obradu slike Chloros, omogućujući automatizaciju, skriptiranje i rad bez glave za vaše tijekove rada sa slikama.

### Ključne značajke

* 🚀 **Automatizacija** - Skupna obrada skripte više skupova podataka
* 🔗 **Integracija** - Ugradite u postojeće tijekove rada i cjevovode
* 💻 **Operacije bez glave** - Pokrenite bez GUI-ja
* 🌍 **Multi-Language** - Podrška za 38 jezika
* ⚡ **Paralelna obrada** - [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) automatski optimizira za vaš hardver

### Zahtjevi

| Zahtjev | Detalji |
| -------------------- | ------------------------------------------------------------------ |
| **Operativni sustav** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licenca** | Chloros+ ([potreban plan uz plaćanje](https://cloud.mapir.camera/pricing)) |
| **Sjećanje** | Minimalno 8 GB RAM-a (preporučuje se 16 GB) |
| **Internet** | Potrebno za aktivaciju licence |
| **Prostor na disku** | Razlikuje se prema veličini projekta |

{% hint style="warning" %}
**Zahtjevi licence**: CLI zahtijeva plaćenu pretplatu Chloros+. Standardni (besplatni) planovi nemaju CLI pristup. Posjetite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) za nadogradnju.
{% endhint %}

## Brzi početak

### Instalacija

#### Windows

CLI je automatski uključen u instalacijski program Chloros:

1. Preuzmite i pokrenite **Chloros Installer.exe**

2. Dovršite čarobnjaka za instalaciju
3. CLI instaliran na: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Instalacijski program automatski dodaje `chloros-cli` vašem sistemu PATH. Ponovno pokrenite terminal nakon instalacije.
{% endhint %}

#### Linux

Instalirajte `.deb` paket za svoju arhitekturu:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Za detaljnu postavku Linux pogledajte [Instalacija Linux](linux/linux-installation.md).

### Prvo postavljanje

Prije korištenja CLI, aktivirajte svoju Chloros+ licencu:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Osnovna upotreba

Obradite mapu sa zadanim postavkama:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referenca naredbe

### Opća sintaksa

```
chloros-cli [global-options] <command> [command-options]
```

***

## Naredbe

### `process` - Obrada slika

Obradite slike u mapi s kalibracijom.

**Sintaksa:**

```bash
chloros-cli process <input-folder> [options]
```

**Primjeri:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Mogućnosti naredbe procesa

| Opcija | Upišite | Zadano | Opis |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------- |
| `<input-folder>` | Put | _Obavezno_ | Mapa koja sadrži RAW/JPG multispektralne slike |
| `-o, --output` | Put | Isto kao unos | Izlazna mapa za obrađene slike |
| `-n, --project-name` | Niz | Automatski generirano | Prilagođeni naziv projekta |
| `--vignette` | Zastava | Omogućeno | Omogući ispravak vinjete |
| `--no-vignette` | Zastava | - | Onemogući korekciju vinjete |
| `--reflectance` | Zastava | Omogućeno | Omogući kalibraciju refleksije |
| `--no-reflectance` | Zastava | - | Onemogući kalibraciju refleksije |
| `--ppk` | Zastava | Onemogućeno | Primijeni PPK ispravke iz .daq podataka svjetlosnog senzora |
| `--format` | Izbor | TIFF (16-bitni) | Izlazni format: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size` | Cijeli broj | Auto | Minimalna veličina cilja u pikselima za otkrivanje kalibracijske ploče |
| `--target-clustering` | Cijeli broj | Auto | Ciljni prag grupiranja (0-100) |
| `--debayer` | Izbor | `standard` | Debayer metoda: `standard` ili `texture-aware` (samo Chloros+) |
| `--target`, `--targets` | Zastava | Onemogućeno | Tražite samo ciljeve kalibracije u podmapi "target" ili "targets" (ubrzava obradu) |
| `--indices` | Popis | Ništa | Vegetacijski indeksi za izračunavanje (npr. `--indices NDVI NDRE GNDVI`) |
| `--exposure-pin-1` | Niz | Ništa | Zaključaj ekspoziciju za model kamere (Pin 1) |
| `--exposure-pin-2` | Niz | Ništa | Zaključaj ekspoziciju za model kamere (Pin 2) |
| `--recal-interval` | Cijeli broj | Auto | Interval rekalibracije u sekundama |
| `--timezone-offset` | Cijeli broj | 0 | Pomak vremenske zone u satima |

***

### `login` - Autentifikacija računa

Prijavite se svojim vjerodajnicama Chloros+ kako biste omogućili obradu CLI.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

**Primjer:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Posebni znakovi**: Koristite jednostruke navodnike oko zaporki koje sadrže znakove kao što su `$`, `!` ili razmake.
{% endhint %}

**Izlaz:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Obriši vjerodajnice

Izbrišite pohranjene vjerodajnice i odjavite se sa svog računa.

**Sintaksa:**

```bash
chloros-cli logout
```

**Primjer:**

```bash
chloros-cli logout
```

**Izlaz:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK korisnici**: Python SDK također pruža programsku `logout()` metodu za brisanje vjerodajnica unutar Python skripti. Pogledajte [dokumentaciju Python SDK] (api-python-sdk.md#logout) za detalje.
{% endhint %}

***

### `status` - Provjerite status licence

Prikaz trenutne licence i statusa provjere autentičnosti.

**Sintaksa:**

```bash
chloros-cli status
```

**Primjer:**

```bash
chloros-cli status
```

**Izlaz:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Provjerite napredak izvoza

Pratite napredak izvoza niti 4 tijekom ili nakon obrade.

**Sintaksa:**

```bash
chloros-cli export-status
```

**Primjer:**

```bash
chloros-cli export-status
```

**Upotreba:** Pozovite ovu naredbu dok je obrada u tijeku da provjerite napredak izvoza.***

### `language` - Upravljanje jezikom sučelja

Pogledajte ili promijenite jezik sučelja CLI.

**Sintaksa:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Primjeri:**

```bash
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
| `pt` | portugalski | português |
| `fr` | francuski | Français |
| `de` | njemački | njemački |
| `it` | talijanski | talijanski |
| `ja` | japanski | 日本語 |
| `ko` | korejski | 한국어 |
| `zh` | kineski (pojednostavljeni) | 简体中文 |
| `zh-TW` | kineski (tradicionalni) | 繁體中文 |
| `ru` | ruski | Russkij |
| `nl` | nizozemski | Nizozemski |
| `ar` | arapski | العربية |
| `pl` | poljski | Poljski |
| `tr` | turski | turski |
| `hi` | hindski | हिंदी |
| `id` | indonezijski | Bahasa Indonezija |
| `vi` | vijetnamski | Tiếng Việt |
| `th` | tajlandski | ไทย |
| `sv` | švedski | Svenska |
| `da` | danski | Dansk |
| `no` | norveški | Norsk |
| `fi` | finski | Suomi |
| `el` | grčki | Ελληνικά |
| `cs` | češki | Čeština |
| `hu` | mađarski | mađarski |
| `ro` | rumunjski | Română |
| `uk` | ukrajinski | Ukrajinska |
| `pt-BR` | brazilski portugalski | português brasileiro |
| `zh-HK` | kantonski | 粵語 |
| `ms` | malajski | Bahasa Melayu |
| `sk` | slovački | Slovenčina |
| `bg` | bugarski | Bʺlgarski |
| `hr` | hrvatski | Hrvatski |
| `lt` | litvanski | Lietuvių |
| `lv` | latvijski | Latviešu |
| `et` | estonski | Eesti |
| `sl` | slovenski | Slovenščina |

{% hint style="success" %}
**Automatska postojanost**: Vaše postavke jezika spremaju se u `~/.chloros/cli_language.json` i traju tijekom svih sesija.
{% endhint %}

***

### `set-project-folder` - Postavite zadanu mapu projekta

Promijenite zadanu lokaciju mape projekta (dijeli se s GUI na Windows).

**Sintaksa:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Primjeri:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Prikaži mapu projekta

Prikaži trenutnu zadanu lokaciju mape projekta.

**Sintaksa:**

```bash
chloros-cli get-project-folder
```

**Primjer:**

```bash
chloros-cli get-project-folder
```

**Izlaz:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Vrati na zadano

Ponovno postavite mapu projekta na zadanu lokaciju.

**Sintaksa:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Pokrenite dijagnostiku sustava

Pokrenite 7 dijagnostičkih provjera za provjeru konfiguracije vašeg sustava.

**Sintaksa:**

```bash
chloros-cli selftest
```

**Izvršena dijagnostika:**

1. Provjera verzije
2. Dostupnost luka (5000)
3. Pozadinsko pokretanje
4. API test povezivosti
5. Podaci o sustavu i otkrivanje GPU-a
6. Verifikacija modela denoisera
7. Provjera dostupnosti CUDA

{% hint style="info" %}
**Korisno za rješavanje problema**: Pokrenite `selftest` nakon instalacije kako biste provjerili je li vaš sustav ispravno konfiguriran, posebno na Linux/Jetson gdje GPU i CUDA postavke mogu trebati provjeru.
{% endhint %}

***

### `update` - Provjerite ima li ažuriranja (samo Linux)

Provjerite i instalirajte CLI ažuriranja na Linux sustave.

**Sintaksa:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opcija | Opis |
| --------- | ---------------------------------- |
| `--check` | Provjerite samo ažuriranja, nemojte instalirati |

{% hint style="info" %}
Ova naredba je dostupna samo na Linux. Na Windows ažuriranja se isporučuju kroz instalacijski program.
{% endhint %}

***

## Globalne opcije

Ove se opcije odnose na sve naredbe:

| Opcija | Upišite | Zadano | Opis |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Put | Automatski otkriveno | Put do pozadinske izvršne |
| `--port` | Cijeli broj | 5000 | Pozadinski API broj porta |
| `--restart` | Zastava | - | Prisilno ponovno pokretanje pozadine (ubija postojeće procese) |
| `--version` | Zastava | - | Prikaži informacije o verziji i izađi |
| `--help` | Zastava | - | Prikaži informacije za pomoć i izađi |

{% hint style="info" %}
**Pozadinsko automatsko otkrivanje**: `--backend-exe` put se automatski otkriva po platformi:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ručno)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Primjer s globalnim opcijama:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Vodič za postavke obrade

### Paralelna obrada i dinamička prilagodba računanja

Chloros 1.1.0 uključuje [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) — mehanizam za obradu **automatski otkriva vaš hardver** i odabire optimalnu strategiju:

| Platforma | Strategija | Radnici | Cjevovod | Bilješke |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Memorijski učinkovito, serijalizirano |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Istodobna GPU obrada |
| **Stolno računalo s 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobre performanse radne površine |
| **Stolno računalo s 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimalne performanse radne površine |
| **Sustav samo za CPU** | `CPU_PARALLEL` | jezgre - 1 | `cpu_fallback` | GPU nije potreban |

{% hint style="success" %}
**Nije potrebna ručna konfiguracija!** Chloros automatski detektira vaš CPU, GPU, RAM i (na Jetson) toplinske senzore, a zatim automatski konfigurira optimalni cjevovod obrade.
{% endhint %}

### Debayerove metode

| Metoda | CLI Zastava | Kvaliteta | Brzina | Licenca |
| --- | --- | --- | --- | --- |
| **Standardno (brzo, srednje kvalitete)** | `--debayer standard` | dobro | Brzo | Besplatno / Chloros+ |
| **Svjestan teksture (Sporo, najviša kvaliteta)** | `--debayer texture-aware` | Najviša | Sporo | Samo Chloros+ |

Zadana metoda debayera je **Standardno**. Metoda**Texture Aware** koristi AI/ML model uklanjanja šuma za ispis najviše kvalitete, ali zahtijeva licencu Chloros+ i NVIDIA GPU.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Ispravak vinjete

**Što radi:** ispravlja pad svjetla na rubovima slike (tamniji kutovi uobičajeni na slikama fotoaparata).

* **Omogućeno prema zadanim postavkama** - većina korisnika bi ovo trebala držati uključenim
* Koristite `--no-vignette` za onemogućavanje

{% hint style="success" %}
**Preporuka**: Uvijek omogućite korekciju vinjete kako biste osigurali jednoliku svjetlinu u okviru.
{% endhint %}

### Kalibracija refleksije

Pretvara neobrađene vrijednosti senzora u standardizirane postotke refleksije pomoću kalibracijskih ploča.

* **Omogućeno prema zadanim postavkama** - Neophodno za analizu vegetacije
* Zahtijeva kalibracijske ciljne ploče na slikama
* Koristite `--no-reflectance` za onemogućavanje

{% hint style="info" %}
**Zahtjevi**: Provjerite jesu li kalibracijske ploče pravilno eksponirane i vidljive na vašim slikama radi točne konverzije refleksije.
{% endhint %}

### PPK ispravci

**Što radi:** Primjenjuje naknadno obrađene kinematičke ispravke pomoću podataka zapisnika DAQ-A-SD za poboljšanu GPS točnost.

* **Onemogućeno prema zadanim postavkama**
* Koristite `--ppk` za omogućavanje
* Zahtijeva .daq datoteke u mapi projekta iz MAPIR DAQ-A-SD svjetlosnog senzora.

### Izlazni formati

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bit dubina</th><th width="116.5999755859375">Veličina datoteke</th><th>Najbolja Za</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitni)</strong> ⭐</td><td>16-bitni cijeli broj</td><td>Velika</td><td>GIS analiza, fotogrametrija (preporučeno)</td></tr><tr><td><strong>TIFF (32-bit, postotak)</strong></td><td>32-bitni float</td><td>Vrlo velik</td><td>Znanstvena analiza, istraživanje</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bitni cijeli broj</td><td>Srednje</td><td>Vizualno pregled, web dijeljenje</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bitni cijeli broj</td><td>Small</td><td>Brzi pregled, komprimirani izlaz</td></tr></tbody></table>

***

## Automatizacija i skriptiranje

### Skupna obrada PowerShell (Windows)

Automatski obradi više mapa skupa podataka na Windows:

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

### Windows Skupna skripta (Windows)

Jednostavna petlja za skupnu obradu na Windows:

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

### Skupna obrada Bash (Linux)

Obradi višestruke mape skupa podataka na Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python skripta za automatizaciju (više platformi)

Napredna automatizacija s rukovanjem greškama (radi na Windows i Linux):

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
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
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

***

## Tijek obrade

### Standardni tijek rada

1. **Ulaz**: Mapa koja sadrži parove RAW/JPG slika
2. **Otkriće**: CLI automatski skenira podržane slikovne datoteke
3. **Procesiranje**: Paralelni način rada skalira na vaše CPU jezgre (Chloros+)
4. **Izlaz**: Stvara podmape modela fotoaparata s obrađenim slikama

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

### Procjena vremena obrade

Uobičajena vremena obrade za 100 slika (12 MP svaka):

| Platforma | Način | Procijenjeno vrijeme | Bilješke |
| --- | --- | --- | --- |
| **Stolno računalo 12GB+ GPU** | `GPU_PARALLEL` | 5-10 min | Najbrža opcija |
| **Desktop 8GB GPU** | `GPU_SINGLE` | 10-15 min | Dobra izvedba |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 15-25 min | Rubno računanje |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30-60 min | Ograničena memorija |
| **Samo CPU** | `CPU_PARALLEL` | 20-40 min | GPU nije potreban |

{% hint style="info" %}
**Savjet za performanse**: Vrijeme obrade ovisi o broju slika, razlučivosti, metodi debayera i hardveru. Texture Aware debayer traje znatno dulje nego Standard. Pogledajte [Dynamic Compute Adaptation] (processing-architecture/dynamic-compute-adaptation.md) za detalje.
{% endhint %}

***

## Rješavanje problema

### CLI Nije pronađen

**Windows Greška:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows rješenja:**

1. Provjerite mjesto instalacije:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Koristite puni put ako nije u PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ručno dodajte u PATH:
   * Otvorite Svojstva sustava → Varijable okruženja
   * Uredite PATH varijablu
   * Dodaj: `C:\Program Files\Chloros\resources\cli`
   * Ponovno pokrenite terminal

**Linux Greška:**

```
chloros-cli: command not found
```

**Linux rješenja:**

1. Provjerite instalaciju:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Ponovno učitajte svoju ljusku:

```bash
source ~/.bashrc
```

3. Provjerite dopuštenja:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Nije uspjelo pokretanje pozadine**Pogreška:**

```

Backend failed to start within 30 seconds
```

**Rješenja:**

1. Provjerite je li pozadina već pokrenuta (prvo je zatvorite)
2. Provjerite da vatrozid ne blokira (Windows) ili provjerite dostupnost porta (Linux: `lsof -i :5000`)
3. Pokušajte s drugim priključkom:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Prisilno ponovno pokretanje pozadine:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Na Linux, provjerite postoji li pozadinska izvršna datoteka:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problemi s licencom/provjerom autentičnosti**Pogreška:**

```

Chloros+ license required for CLI access
```

**Rješenja:**

1. Provjerite imate li aktivnu Chloros+ pretplatu
2. Prijavite se svojim vjerodajnicama:

```bash
chloros-cli login user@example.com 'password'
```

3. Provjerite status licence:

```bash
chloros-cli status
```

4. Kontaktirajte podršku: info@mapir.camera

***

### Slike nisu pronađene**Pogreška:**

```

No images found in the specified folder
```

**Rješenja:**

1. Provjerite sadrži li mapa podržane formate (.RAW, .TIF, .JPG)
2. Provjerite je li putanja mape točna (upotrijebite navodnike za staze s razmacima)
3. Provjerite imate li dopuštenje za čitanje mape
4. Provjerite jesu li ekstenzije datoteka ispravne

***

### Obrada se zaustavlja ili prekida**Rješenja:**

1. Provjerite raspoloživi prostor na disku (osigurajte dovoljno za izlaz)
2. Zatvorite ostale aplikacije kako biste oslobodili memoriju
3. Smanjite broj slika (obradite u serijama)

***

### Port se već koristi**Pogreška:**

```

Port 5000 is already in use
```

**Rješenja:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## FAQ

### P: Trebam li licencu za CLI?

**O:**Da! CLI zahtijeva plaćenu**Chloros+ licencu**.

* ❌ Standardni (besplatni) paket: CLI onemogućen
* ✅ Chloros+ (plaćeni) planovi: CLI potpuno omogućen

Pretplatite se na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: Mogu li koristiti CLI na poslužitelju bez GUI-ja?**O:** Da! CLI radi potpuno bez glave. Ovo je primarni slučaj upotrebe na Linux.**Windows poslužitelj:**
* Windows poslužitelj 2016 ili noviji
* Visual C++ Redistributable instaliran

**Linux poslužitelj:**
* Ubuntu 20.04+ / Debian 11+ (amd64) ili JetPack 6 (arm64)
* Instalirajte putem `.deb` paketa

**Obje platforme:**
* Minimalno 8 GB RAM-a (preporučuje se 16 GB)
* Jednokratna aktivacija licence: `chloros-cli login user@example.com 'password'`

***

### P: Gdje se spremaju obrađene slike?**O:**Prema zadanim postavkama, obrađene slike spremaju se u**istu mapu kao ulaz** u podmapama modela fotoaparata (npr. `Survey3N_RGN/`).

Upotrijebite opciju `-o` da odredite drugu izlaznu mapu:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### P: Mogu li obraditi više mapa odjednom?**O:** Ne izravno u jednoj naredbi, ali možete koristiti skriptiranje za uzastopnu obradu mapa. Pogledajte odjeljak [Automatizacija i skriptiranje](CLI.md#automation--scripting).***

### P: Kako mogu spremiti CLI izlaz u datoteku dnevnika?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Serija:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### P: Što se događa ako pritisnem Ctrl+C tijekom obrade?**A:** CLI će:

1. Prestanite graciozno obrađivati
2. Isključite pozadinu
3. Izlaz sa kodom 130

Djelomično obrađene slike mogu ostati u izlaznoj mapi.

***

### P: Mogu li automatizirati obradu CLI?**O:** Apsolutno! CLI dizajniran je za automatizaciju. Pogledajte [Automatizacija i skriptiranje](CLI.md#automation--scripting) za PowerShell (Windows), Batch (Windows), Bash (Linux) i Python (kroz platformu) primjere.***

### P: Kako mogu provjeriti verziju CLI?**A:**

```bash
chloros-cli --version
```

**Izlaz:**

```

Chloros CLI 1.1.0
```

***

## Dobivanje pomoći

### Pomoć za naredbeni redak

Pogledajte informacije o pomoći izravno u CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanali podrške

* **E-mail**: info@mapir.camera
* **Web stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cijena**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Potpuni primjeri

### Primjer 1: Osnovna obrada

Proces sa zadanim postavkama (vinjeta, refleksija):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Primjer 2: Visokokvalitetni znanstveni rezultati

32-bitni float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Primjer 3: Brza obrada pregleda

8-bitni PNG bez kalibracije za brzi pregled:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Primjer 4: PPK-ispravljena obrada

Primijenite PPK korekcije s refleksijom:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Primjer 5: Lokacija prilagođenog izlaza

Obradi na drugu lokaciju s određenim formatom:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Primjer 6: Tijek rada provjere autentičnosti

Potpuni tijek provjere autentičnosti (isti na svim platformama):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Primjer 7: Upotreba više jezika

Promjena jezika sučelja (isti na svim platformama):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```