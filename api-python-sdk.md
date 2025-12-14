# API: Python SDK

**Chloros Python SDK** pruža programski pristup Chloros mehanizmu za obradu slike, omogućujući automatizaciju, prilagođene tijekove rada i besprijekornu integraciju s vašim Python aplikacijama i istraživačkim kanalima.

### Ključne značajke

* 🐍 **Izvorni Python** - čist, Pythonic API za obradu slika
* 🔧 **Puni API pristup** - Potpuna kontrola nad obradom Chlorosa
* 🚀 **Automatizacija** - Izgradite prilagođene tijekove rada skupne obrade
* 🔗 **Integracija** - Ugradite Chloros u postojeće Python aplikacije
* 📊 **Spremno za istraživanje** - savršeno za znanstvene analize
* ⚡ **Paralelna obrada**- skalira na vaše CPU jezgre (Chloros+)

### Zahtjevi

| Zahtjev          | pojedinosti                                                             |
| -------------------- | ------------------------------------------------------------------ |
|**Chloros Desktop**| Mora se instalirati lokalno                                           |
|**Licenca**| Chloros+ ([paid plan required](https://cloud.mapir.camera/pricing)) |
|**Operativni sustav**| Windows 10/11 (64-bitni)                                              |
|**Piton**| Python 3.7 ili noviji                                                |
|**Memorija**| Minimalno 8 GB RAM-a (preporučuje se 16 GB)                                  |
|**Internet**| Potrebno za aktivaciju licence                                     |

{% hint style="warning" %}**Zahtjev za licencu**: Python SDK zahtijeva plaćenu Chloros+ pretplatu za API pristup. Standardni (besplatni) planovi nemaju API/SDK pristup. Posjeti [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) za nadogradnju.
{% endhint %}

## Brzi početak

### Montaža

Instalirajte putem pipa:

```bash
pip install chloros-sdk
```

{% hint style="info" %}**Prvo postavljanje**: Prije korištenja SDK-a, aktivirajte svoju Chloros+ licencu tako da otvorite Chloros, Chloros (preglednik) ili Chloros CLI i prijavite se svojim vjerodajnicama. Ovo treba učiniti samo jednom.
{% endhint %}

### Osnovna upotreba

Obradite mapu sa samo nekoliko redaka:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Potpuna kontrola

Za napredne tijekove rada:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```***## Vodič za instalaciju

### Preduvjeti

Prije instaliranja SDK-a, provjerite imate li:

1. **Chloros Desktop**instaliran ([preuzimanje](preuzimanje.md))
2.**Python 3.7+**instaliran ([python.org](https://www.python.org))
3.**Active Chloros+ licenca**([nadogradnja](https://cloud.mapir.camera/pricing))

### Instalirajte putem pipa**Standardna instalacija:**```bash
pip install chloros-sdk
```**Uz podršku za praćenje napretka:**```bash
pip install chloros-sdk[progress]
```**Razvojna instalacija:**```bash
pip install chloros-sdk[dev]
```

### Provjerite instalaciju

Provjerite je li SDK ispravno instaliran:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```***

## Prvo postavljanje

### Aktivacija licence

SDK koristi istu licencu kao Chloros, Chloros (preglednik) i Chloros CLI. Aktivirajte jednom putem GUI ili CLI:

1. Otvorite **Chloros ili Chloros (Browser)**i prijavite se na User<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">tab. Ili otvorite**CLI**.
2. Unesite svoje vjerodajnice Chloros+ i prijavite se
3. Licenca se pohranjuje lokalno u predmemoriju (ostaje nakon ponovnog pokretanja)

{% hint style="success" %}**Jednokratno postavljanje**: Nakon prijave putem GUI ili CLI, SDK automatski koristi predmemoriranu licencu. Nije potrebna dodatna provjera autentičnosti!
{% endhint %}

### Testiraj vezu

Provjerite može li se SDK povezati s Chlorosom:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```***## API Referenca

### Kloros Lokalna klasa

Glavna klasa za lokalnu Chloros obradu slike.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametri:**| Parametar                 | Tip | Zadano                   | Opis                           |
| ------------------------ | ---- | ------------------------ | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokalne Chloros pozadine          |
| `auto_start_backend`      | bool | `True`                    | Automatski pokrenite pozadinu ako je potrebno |
| `backend_exe`             | str  | `None` (auto-detect)      | Put do pozadinske izvršne datoteke            |
| `timeout`                 | int  | `30`                      | Istek zahtjeva u sekundama            |
| `backend_startup_timeout` | int  | `60`                      | Istek vremena za pokretanje pozadine (sekunde) |**Primjeri:**```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```***

### Metode

#### `create_project(project_name, camera=None)`

Napravite novi Chloros projekt.

**Parametri:**| Parametar      | Tip | Potreban | Opis                                              |
| -------------- | ---- | -------- | ---------------------------------------------------------- |
| `project_name` | str  | Da      | Naziv za projekt                                     |
| `camera`       | str  | No       | Predložak kamere (npr. "Survey3N\_RGN", "Survey3W\_OCN") |**Povratak:**`dict` - Project creation response**Primjer:**```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```***

#### `import_images(folder_path, recursive=False)`

Uvoz slika iz mape.

**Parametri:**| Parametar     | Tip     | Potreban | Opis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/put | Da      | Put do mape sa slikama         |
| `recursive`   | bool     | No       | Pretraži podmape (zadano: False) |**Povratak:**`dict` - Import results with file count**Primjer:**```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```***

#### `configure(**settings)`

Konfigurirajte postavke obrade.**Parametri:**| Parametar                 | Tip | Zadano                 | Opis                     |
| ------------------------ | ---- | ---------------------- | -------------------------------- |
| `debayer`                 | str  | "Visoka kvaliteta (brže)" | Debayerova metoda                  |
| `vignette_correction`     | bool | `True`                  | Omogući korekciju vinjete      |
| `reflectance_calibration` | bool | `True`                  | Omogući kalibraciju refleksije  |
| `indices`                 | popis | `None`                  | Vegetacijski indeksi za izračunavanje |
| `export_format`           | str  | "TIFF (16-bitni)"         | Izlazni format                   |
| `ppk`                     | bool | `False`                 | Omogući PPK ispravke          |
| `custom_settings`         | izreka | `None`                  | Napredne prilagođene postavke        |**Formati izvoza:**

* `"TIFF (16-bit)"`- Preporučeno za GIS/fotogrametriju
* `"TIFF (32-bit, Percent)"`- Znanstvena analiza
* `"PNG (8-bit)"`- Vizualni pregled
* `"JPG (8-bit)"`- Komprimirani izlaz

**Dostupni indeksi:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 i više.**Primjer:**```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Obradite slike projekta.

**Parametri:**| Parametar           | Tip     | Zadano      | Opis                               |
| ------------------- | -------- | ------------ | ------------------------------------------ |
| `mode`              | str      | `"parallel"` | Način obrade: "paralelni" ili "serijski"   |
| `wait`              | bool     | `True`       | Pričekajte završetak                       |
| `progress_callback` | pozivivi | `None`       | Funkcija povratnog poziva napretka (progress, msg) |
| `poll_interval`     | lebdjeti    | `2.0`        | Interval prozivanja za napredak (sekunde)   |**Povratak:**`dict` - Processing results

{% hint style="warning" %}**Paralelni način rada**: Zahtijeva Chloros+ licencu. Automatski se skalira na vaše CPU jezgre (do 16 radnika).
{% endhint %}**Primjer:**```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```***

#### `get_config()`

Dobijte trenutnu konfiguraciju projekta.

**Povratak:**`dict` - Current project configuration**Primjer:**```python
config = chloros.get_config()
print(config['Project Settings'])
```***

#### `get_status()`

Dobijte informacije o statusu pozadine.

**Povratak:**`dict` - Backend status**Primjer:**```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```***

#### `shutdown_backend()`

Isključite pozadinu (ako je pokrenuo SDK).

**Primjer:**```python
chloros.shutdown_backend()
```***

### Pogodne funkcije

#### `process_folder(folder_path, **options)`

Pogodna funkcija jednog retka za obradu mape.**Parametri:**| Parametar                 | Tip     | Zadano         | Opis                    |
| ------------------------ | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/put | Potreban        | Put do mape sa slikama     |
| `project_name`            | str      | Automatski generirano  | Naziv projekta                   |
| `camera`                  | str      | `None`          | Predložak kamere                |
| `indices`                 | popis     | `["NDVI"]`      | Indeksi za izračunavanje           |
| `vignette_correction`     | bool     | `True`          | Omogući korekciju vinjete     |
| `reflectance_calibration` | bool     | `True`          | Omogući kalibraciju refleksije |
| `export_format`           | str      | "TIFF (16-bitni)" | Izlazni format                  |
| `mode`                    | str      | `"parallel"`    | Način obrade                |
| `progress_callback`       | pozivivi | `None`          | Povratni poziv napredovanja              |**Povratak:**`dict` - Processing results**Primjer:**```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```***

## Podrška za upravitelja konteksta

SDK podržava upravitelje konteksta za automatsko čišćenje:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***## Potpuni primjeri

### Primjer 1: Osnovna obrada

Obradi mapu sa zadanim postavkama:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```***

### Primjer 2: Prilagođeni tijek rada

Potpuna kontrola nad procesnim cjevovodom:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Primjer 3: Skupna obrada više mapa

Obrada više skupova podataka o letu:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***### Primjer 4: Integracija istraživačkog cjevovoda

Integrirajte Chloros s analizom podataka:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```***

### Primjer 5: Prilagođeno praćenje napretka

Napredno praćenje napretka uz bilježenje:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***### Primjer 6: Rješavanje pogrešaka

Robusno rukovanje pogreškama za proizvodnu upotrebu:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```***

### Primjer 7: Alat za naredbeni redak

Izgradite prilagođeni CLI alat sa SDK-om:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Korištenje:**```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```***

## Rukovanje iznimkama

SDK pruža posebne klase iznimki za različite vrste pogrešaka:

### Hijerarhija izuzetaka

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Primjeri izuzetaka

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Napredne teme

### Prilagođena pozadinska konfiguracija

Koristite prilagođenu pozadinsku lokaciju ili konfiguraciju:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Obrada bez blokiranja

Započnite obradu i nastavite s drugim zadacima:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Upravljanje memorijom

Za velike skupove podataka obradite u serijama:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***## Rješavanje problema

### Pozadina se ne pokreće**Izdati:**SDK fails to start backend**Rješenja:**1. Provjerite je li Chloros Desktop instaliran:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Provjerite da vatrozid za Windows ne blokira
3. Pokušajte s ručnim pozadinskim putem:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```***

### Licenca nije otkrivena

**Izdati:**SDK warns about missing license**Rješenja:**1. Otvorite Chloros, Chloros (Browser) ili Chloros CLI i prijavite se.
2. Provjerite je li licenca predmemorirana:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Kontaktirajte podršku: info@mapir.camera***

### Pogreške uvoza

**Izdati:**`ModuleNotFoundError: No module named 'chloros_sdk'`**Rješenja:**```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```***

### Istek vremena obrade

**Izdati:**Processing times out**Rješenja:**1. Povećanje vremenskog ograničenja:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Obradite manje serije
3. Provjerite raspoloživi prostor na disku
4. Pratite resurse sustava***

### Port se već koristi

**Izdati:**Backend port 5000 occupied**Rješenja:**```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ili pronađite i zatvorite sukobljeni proces:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```***

## Savjeti za izvedbu

### Optimizirajte brzinu obrade

1. **Koristite paralelni način**(zahtijeva Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2.**Smanjite izlaznu rezoluciju**(ako je prihvatljivo)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3.**Onemogući nepotrebne indekse**```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4.**Proces na SSD-u**(ne na HDD-u)***

### Optimizacija memorije

Za velike skupove podataka:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***### Obrada u pozadini

Oslobodite Python za druge zadatke:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```***

## Primjeri integracije

### Django integracija

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyterova bilježnica

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***## FAQ

### P: Zahtijeva li SDK internetsku vezu?**A:**Only for initial license activation. After logging in via Chloros, Chloros (Browser) or Chloros CLI the license is cached locally and works offline for 30 days.***

### P: Mogu li koristiti SDK na poslužitelju bez GUI-ja?

**A:** Yes! Requirements:

* Windows Server 2016 ili noviji
* Kloros instaliran (jednokratno)
* Licenca aktivirana na bilo kojem računalu (spremljena licenca kopirana na poslužitelj)

***### P: Koja je razlika između desktopa, CLI-ja i SDK-a?

| Značajka         | GUI za radnu površinu | CLI naredbeni redak | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
|**Sučelje**| Točka-klik | Naredba          | Python API  |
|**Najbolje za**| Vizualni rad | Skriptiranje        | Integracija |
|**Automatizacija**| ograničeno     | Dobro             | Izvrsno   |
|**Fleksibilnost**| Osnovno       | Dobro             | Maksimalno     |
|**Licenca**| Klor+    | Klor+         | Klor+    |***

### P: Mogu li distribuirati aplikacije izrađene pomoću SDK-a?

**A:** SDK code can be integrated into your applications, but:

* Krajnji korisnici moraju imati instaliran Chloros
* Krajnji korisnici trebaju aktivne Chloros+ licence
* Komercijalna distribucija zahtijeva OEM licenciranje

Kontaktirajte info@mapir.camera za OEM upite.

***### P: Kako mogu ažurirati SDK?

```bash
pip install --upgrade chloros-sdk
```***

### P: Gdje se spremaju obrađene slike?

Prema zadanim postavkama, u Putanju projekta:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***### P: Mogu li obraditi slike iz Python skripti koje se izvode prema rasporedu?**A:**Yes! Use Windows Task Scheduler with Python scripts:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Napravite raspored putem Task Scheduler-a za svakodnevno pokretanje.***

### P: Podržava li SDK async/await?

**A:**Current version is synchronous. For async behavior, use `wait=False` or run in separate thread:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```***

## Dobivanje pomoći

### Dokumentacija

* **API Referenca**: Ova stranica

### Kanali podrške

* **Email**: info@mapir.camera
* **Web stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cijene**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Uzorak koda

Svi ovdje navedeni primjeri testirani su i spremni za proizvodnju. Kopirajte ih i prilagodite za svoj slučaj upotrebe.***## Licenca

**Vlasnički softver** - Autorska prava (c) 2025 MAPIR Inc.

SDK zahtijeva aktivnu Chloros+ pretplatu. Zabranjeno je neovlašteno korištenje, distribucija ili izmjena.
