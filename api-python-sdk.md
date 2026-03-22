# API : Python SDK

**Chloros Python SDK** pruža programski pristup mehanizmu za obradu slike Chloros, omogućujući automatizaciju, prilagođene tijekove rada i besprijekornu integraciju s vašim Python aplikacijama i istraživačkim kanalima.

### Ključne značajke

* 🐍 **Nativni Python** - čist, Pythonic API za obradu slike
* 🔧 **Puni pristup API** - Potpuna kontrola nad obradom Chloros
* 🚀 **Automatizacija** - Izgradite prilagođene tijekove rada skupne obrade
* 🔗 **Integracija** - Ugradite Chloros u postojeće Python aplikacije
* 📊 **Spremno za istraživanje** - savršeno za znanstvene analize
* ⚡ **Paralelna obrada** - skalira na vaše CPU jezgre (Chloros+)

### Zahtjevi

| Zahtjev | Detalji |
| -------------------- | ------------------------------------------------------------------ |
| **Chloros instaliran** | Windows: Desktop instalacija; Linux: paket `.deb` |
| **Licenca** | Chloros+ ([potreban plan uz plaćanje](https://cloud.mapir.camera/pricing)) |
| **Operativni sustav** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python** | Python 3.7 ili noviji |
| **Sjećanje** | Minimalno 8 GB RAM-a (preporučuje se 16 GB) |
| **Internet** | Potrebno za aktivaciju licence |

{% hint style="warning" %}
**Zahtjevi za licencu**: Python SDK zahtijeva plaćenu pretplatu Chloros+ za pristup API. Standardni (besplatni) planovi nemaju pristup API/SDK. Posjetite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) za nadogradnju.
{% endhint %}

## Brzi početak

### Instalacija

Instalirajte putem pipa:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Prvo postavljanje**: Prije korištenja SDK, aktivirajte svoju licencu Chloros+ otvaranjem Chloros, Chloros (preglednik) ili Chloros CLI i prijavom sa svojim vjerodajnicama. Ovo treba učiniti samo jednom. Na Linux (bez GUI-ja), koristite: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Osnovna upotreba

Obradite mapu sa samo nekoliko redaka:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Putovi na više platformi**: Primjeri kodova na ovoj stranici koriste putove u stilu Windows (npr. `C:\\DroneImages\\Flight001`). Na Linux umjesto toga koristite staze Linux (npr. `/home/user/drone_images/flight001` ili `~/drone_images/flight001`). SDK radi identično na obje platforme.
{% endhint %}

### Potpuna kontrola

Za napredne tijekove rada:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Vodič za instalaciju

### Preduvjeti

Prije instaliranja SDK, provjerite imate li:

1. **Chloros instaliran** — Windows: Desktop instalacijski program ([preuzimanje](download.md)); Linux: `.deb` paket ([Linux instalacija](linux/linux-installation.md))
2. **Python 3.7+** instaliran ([python.org](https://www.python.org))
3. **Aktivna licenca Chloros+** ([nadogradnja](https://cloud.mapir.camera/pricing))

### Instalirajte putem pip-a

**Standardna instalacija:**

```bash
pip install chloros-sdk
```

**S podrškom za praćenje napretka:**

```bash
pip install chloros-sdk[progress]
```

**Razvojna instalacija:**

```bash
pip install chloros-sdk[dev]
```

### Provjerite instalaciju

Provjerite je li SDK ispravno instaliran:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Prvo postavljanje

### Aktivacija licence

SDK koristi istu licencu kao Chloros, Chloros (preglednik) i Chloros CLI. Aktivirajte jednom putem GUI ili CLI:**Windows:**Otvorite**Chloros ili Chloros (preglednik)** i prijavite se na kartici Korisnik <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> ili koristite karticu CLI.**Linux:** Koristite CLI (GUI nije dostupan):

```bash
chloros-cli login user@example.com 'your_password'
```

Licenca se pohranjuje lokalno u predmemoriju i traje nakon ponovnog pokretanja.

{% hint style="success" %}
**Jednokratno postavljanje**: Nakon prijave putem GUI-ja ili CLI, SDK automatski koristi predmemoriranu licencu. Nije potrebna dodatna provjera autentičnosti!
{% endhint %}

{% hint style="info" %}
**Odjava**: korisnici SDK mogu programski očistiti predmemorirane vjerodajnice pomoću metode `logout()`. Pogledajte metodu [logout()](#logout) u API Referenci.
{% endhint %}

### Testiraj vezu

Provjerite da se SDK može povezati s Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Referenca

### ChlorosLocal Class

Glavna klasa za lokalnu obradu slike Chloros.

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

**Parametri:**

| Parametar | Upišite | Zadano | Opis |
| ------------------------ | ---- | ------------------------ | ------------------------------------- |
| `api_url` | str | `"http://localhost:5000"` | URL lokalne pozadine Chloros |
| `auto_start_backend` | bool | `True` | Automatski pokreni pozadinu ako je potrebno |
| `backend_exe` | str | `None` (automatsko otkrivanje) | Put do pozadinske izvršne |
| `timeout` | int | `30` | Istek zahtjeva u sekundama |
| `backend_startup_timeout` | int | `60` | Istek vremena za pokretanje pozadine (sekunde) |

**Primjeri:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Automatsko otkrivanje više platformi**: SDK automatski pokušava ispravnu pozadinsku putanju za vašu platformu:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ručno)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metode

#### `create_project(project_name, camera=None)`

Napravite novi projekt Chloros.

**Parametri:**

| Parametar | Upišite | Obavezno | Opis |
| -------------- | ---- | -------- | ---------------------------------------------------------- |
| `project_name` | str | Da | Naziv za projekt |
| `camera` | str | Ne | Predložak kamere (npr. "Survey3N\_RGN", "Survey3W\_OCN") |

**Vraća:** `dict` - Odgovor na stvaranje projekta**Primjer:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Uvoz slika iz mape.

**Parametri:**

| Parametar | Upišite | Obavezno | Opis |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/put | Da | Put do mape sa slikama |
| `recursive` | bool | Ne | Pretraživanje podmapa (zadano: False) |

**Vraća:** `dict` - Uvoz rezultata s brojem datoteka**Primjer:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurirajte postavke obrade.

**Parametri:**

| Parametar | Upišite | Zadano | Opis |
| ------------------------ | ---- | ----------------------- | -------------------------------- |
| `debayer` | str | "Standardno (brzo, srednje kvalitete)" | Debayerova metoda |
| `vignette_correction` | bool | `True` | Omogući ispravak vinjete |
| `reflectance_calibration` | bool | `True` | Omogući kalibraciju refleksije |
| `indices` | popis | `None` | Vegetacijski indeksi za izračunavanje |
| `export_format` | str | "TIFF (16-bitni)" | Izlazni format |
| `ppk` | bool | `False` | Omogući PPK ispravke |
| `custom_settings` | izreka | `None` | Napredne prilagođene postavke |

**Formati izvoza:**

* `"TIFF (16-bit)"` - Preporučeno za GIS/fotogrametriju
* `"TIFF (32-bit, Percent)"` - Znanstvena analiza
* `"PNG (8-bit)"` - Vizualni pregled
* `"JPG (8-bit)"` - Komprimirani izlaz

**Dostupni indeksi:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2, i više.**Primjer:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Obradite slike projekta.

**Parametri:**

| Parametar | Upišite | Zadano | Opis |
| ------------------ | -------- | ------------ | ------------------------------------------ |
| `mode` | str | `"parallel"` | Način obrade: "paralelni" ili "serijski" |
| `wait` | bool | `True` | Pričekajte završetak |
| `progress_callback` | pozivati ​​| `None` | Funkcija povratnog poziva napretka (progress, msg) |
| `poll_interval` | lebdjeti | `2.0` | Interval prozivanja za napredak (sekunde) |

**Povrati:** `dict` - Rezultati obrade

{% hint style="warning" %}
**Paralelni način rada**: Zahtijeva Chloros+ licencu. Automatski se skalira na vaše CPU jezgre (do 16 radnika).
{% endhint %}

**Primjer:**

```python
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
```

***

#### `get_config()`

Dobijte trenutnu konfiguraciju projekta.

**Povrati:** `dict` - trenutna konfiguracija projekta**Primjer:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Dobijte informacije o statusu pozadine uključujući napredak obrade po niti.

**Povratak:** `dict` - status pozadine sa sljedećom strukturom:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Primjer:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Isključite pozadinu (ako je pokrenuo SDK).

**Primjer:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Očistite predmemorirane vjerodajnice iz lokalnog sustava.

**Opis:**

Programski se odjavljuje uklanjanjem predmemoriranih vjerodajnica za provjeru autentičnosti. Ovo je korisno za:
* Prebacivanje između različitih Chloros+ računa
* Brisanje vjerodajnica u automatiziranim okruženjima
* Sigurnosne svrhe (npr. uklanjanje vjerodajnica prije deinstalacije)

**Povratak:** `dict` - Rezultat operacije odjave**Primjer:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Potrebna je ponovna autentifikacija**: Nakon poziva `logout()`, morate se ponovno prijaviti putem Chloros, Chloros (preglednik) ili Chloros CLI prije korištenja SDK.
{% endhint %}

***

### Pogodne funkcije

#### `process_folder(folder_path, **options)`

Pogodna funkcija jednog retka za obradu mape.

**Parametri:**

| Parametar | Upišite | Zadano | Opis |
| ------------------------ | -------- | --------------- | ------------------------------ |
| `folder_path` | str/put | Obavezno | Put do mape sa slikama |
| `project_name` | str | Automatski generirano | Naziv projekta |
| `camera` | str | `None` | Predložak kamere |
| `indices` | popis | `["NDVI"]` | Indeksi za izračun |
| `vignette_correction` | bool | `True` | Omogući ispravak vinjete |
| `reflectance_calibration` | bool | `True` | Omogući kalibraciju refleksije |
| `export_format` | str | "TIFF (16-bitni)" | Izlazni format |
| `mode` | str | `"parallel"` | Način obrade |
| `progress_callback` | pozivati ​​| `None` | Povratni poziv napredovanja |

**Povrati:** `dict` - Obrada rezultata**Primjer:**

```python
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
```

***

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

***

## Potpuni primjeri

{% hint style="info" %}
**Korisnici Linux**: Svi primjeri u nastavku koriste staze Windows. Zamijenite staze `C:\\...` svojim stazama Linux (npr. `/home/user/...` ili `~/...`). Sve funkcije SDK identične su na svim platformama.
{% endhint %}

### Primjer 1: Osnovna obrada

Obradite mapu sa zadanim postavkama:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

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
    debayer="Standard (Fast, Medium Quality)",
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

***

### Primjer 4: Integracija istraživačkog cjevovoda

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
```

***

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

***

### Primjer 6: Rješavanje grešaka

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
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
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
```

***

### Primjer 7: Upravljanje računom i odjava

Upravljajte vjerodajnicama programski:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Primjer 8: Alat za naredbeni redak

Izgradite prilagođeni alat CLI s SDK:

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
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
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

**Korištenje:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Rukovanje iznimkama

SDK pruža posebne klase izuzetaka za različite vrste grešaka:

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
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

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

***

## Rješavanje problema

### Pozadina se ne pokreće

**Problem:** SDK ne uspijeva pokrenuti pozadinu**Rješenja:**

1. Provjerite je li instaliran Chloros:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Provjerite vatrozid (Windows) ili dostupnost priključka (Linux: `lsof -i :5000`)
3. Isprobajte ručni pozadinski put:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licenca nije otkrivena**Problem:** SDK upozorava na nedostatak licence**Rješenja:**

1. Otvorite Chloros, Chloros (preglednik) ili Chloros CLI i prijavite se.
2. Provjerite je li licenca spremljena u predmemoriju:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Ako imate problema s vjerodajnicama, izbrišite predmemorirane vjerodajnice i ponovno se prijavite:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Kontaktirajte podršku: info@mapir.camera

***

### Pogreške pri uvozu**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Rješenja:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Istek vremena obrade**Problem:** Obrada je istekla**Rješenja:**

1. Povećajte vrijeme čekanja:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Obraditi manje serije
3. Provjerite raspoloživi prostor na disku
4. Pratite resurse sustava

***

### Port se već koristi**Problem:** Pozadinski priključak 5000 zauzet**Rješenja:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ili pronađite i zatvorite sukobljeni proces:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Savjeti za izvedbu

### Optimizirajte brzinu obrade

1. **Koristite paralelni način rada** (zahtijeva Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Smanjite izlaznu rezoluciju** (ako je prihvatljivo)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Onemogući nepotrebne indekse**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Proces na SSD-u** (ne na HDD-u)***

### Optimizacija memorije

Za velike skupove podataka:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Obrada u pozadini

Oslobodite Python za druge zadatke:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

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

### Bočica API

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

***

## FAQ

### P: Da li SDK zahtijeva internetsku vezu?

**O:** Samo za početnu aktivaciju licence. Nakon prijave putem Chloros, Chloros (preglednik) ili Chloros CLI licenca se lokalno predmemorira i radi izvan mreže 30 dana.***

### P: Mogu li koristiti SDK na poslužitelju bez GUI-ja?**O:** Da! SDK radi bez glave na Windows i Linux poslužiteljima.**Linux (preporučeno za bez glave):**
* Instalirajte putem `.deb` paketa
* Aktiviraj licencu: `chloros-cli login user@example.com 'password'`

**Windows poslužitelj:**
* Windows poslužitelj 2016 ili noviji
* Chloros instaliran (jednokratno)
* Licenca se aktivira putem CLI ili na bilo kojem računalu

***

### P: Koja je razlika između računala, CLI i SDK?

| Značajka | GUI radne površine | CLI naredbeni redak | Python SDK |
| --------------- | ----------- | ---------------- | ----------- |
| **Sučelje** | Pokažite i kliknite | Naredba | Python API |
| **Najbolje za** | Vizualni rad | Skriptiranje | Integracija |
| **Automatizacija** | Ograničeno | dobro | Izvrsno |
| **Fleksibilnost** | Osnovno | dobro | Maksimalno |
| **Licenca** | Chloros+ | Chloros+ | Chloros+ |***

### P: Mogu li distribuirati aplikacije izgrađene s SDK?**A:** SDK kod može se integrirati u vaše aplikacije, ali:

* Krajnji korisnici moraju imati instaliran Chloros
* Krajnji korisnici trebaju aktivne Chloros+ licence
* Komercijalna distribucija zahtijeva OEM licencu

Kontaktirajte info@mapir.camera za OEM upite.

***

### P: Kako mogu ažurirati SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### P: Gdje se spremaju obrađene slike?

Prema zadanim postavkama, u Putanju projekta:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### P: Mogu li obraditi slike iz Python skripti koje se izvode prema rasporedu?**O:** Da! Koristite svoj planer OS-a s Python skriptama:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Zakažite raspored putem Planera zadataka za svakodnevno pokretanje.**Linux:** Raspored putem cron-a:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### P: Podržava li SDK async/await?**O:** Trenutna verzija je sinkrona. Za asinkrono ponašanje koristite `wait=False` ili pokrenite u zasebnoj niti:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### P: Kako se prebacivati ​​između različitih Chloros+ računa?**A:** Upotrijebite metodu `logout()` za brisanje predmemoriranih vjerodajnica, a zatim se ponovno prijavite s novim računom:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Nakon odjave, provjerite autentičnost pomoću novog računa putem GUI-ja, preglednika ili CLI prije ponovne upotrebe SDK.

***

## Dobivanje pomoći

### Dokumentacija

* **API Referenca**: Ova stranica

### Kanali podrške

* **E-mail**: info@mapir.camera
* **Web stranica**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Cijena**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Uzorak koda

Svi ovdje navedeni primjeri testirani su i spremni za proizvodnju. Kopirajte ih i prilagodite za svoj slučaj upotrebe.

***

## Licenca**Vlasnički softver** - Autorska prava (c) 2025 MAPIR Inc.

SDK zahtijeva aktivnu Chloros+ pretplatu. Zabranjeno je neovlašteno korištenje, distribucija ili izmjena.