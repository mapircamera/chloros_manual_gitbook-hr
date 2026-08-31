---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Izlazni formati slika

Chlorosizvozi obrađene proizvode u četiri formata datoteka. Odaberite format u Postavkama projekta (GUI), pomoću `--format` (CLI) ili pomoću `export_format` (SDK). CLI i SDK prihvaćaju točne nizove navedene u nastavku.

| Formatni niz | Proširenje | Tip piksela | Raspon piksela | Napomene |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(podrazumijevano)* | `.tif` | uint16 digitalni broj | 0 – 65535 | Preporučeno za fotogrametriju / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odbijanja. Neke aplikacije ne mogu čitati TIFF datoteke s pomičnim zarezom; datoteke su veće. |
| `PNG (8-bit)` | `.png` | uint8 digitalni broj | 0 – 255 | Kompresija bez gubitka, pogodna za pregledavanje na webu i vizualizaciju. |
| `JPG (8-bit)` | `.jpg` | uint8 digital number | 0 – 255 | Kompresija s gubicima, najmanje datoteke. |

## Gdje se spremaju izlazne datoteke

Proizvodi se spremaju u mapu projekta, grupirani po kameri, a zatim po formatu datoteke:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Složak kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE i `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3. **Svaki izvezeni proizvod zadržava naziv izvornog datoteka — proizvod se identificira putem mape, a ne nastavka naziva datoteke.** Pogledajte [Where the outputs land](reference/cli-reference.md) u referenci CLI za potpuna pravila.

## LATTICE proizvodi (razine snimanja i izvoza)

Jedan sirovi LATTICE kadar razgranava se u svaki zatraženi proizvod u jednom prolazu. Svaka vrsta proizvoda ima vlastiti prekidač (kvačice u GUI-ju ili CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, sve je prema zadanim postavkama UKLJUČENO):

| Razina | Sadržaj | Tip podataka |
| --- | --- | --- |
| `raw` | Podaci Bayer iz senzora (monokromatske kamere: pojedinačna banda). Obrada uvijek započinje s neobrađenim podacima. | Kao snimljeno |
| `debayered` | Linearna demosaika — 3-kanalna za M3C, 1-kanalna u sivim tonovima za M3M. | Linearni DN |
| `radiance` | Apsolutna spektralna zračnost iz punog radiometrijskog lanca, u **W/m²/sr/nm**. Uvijek se zapisuje kao 32-bitni TIFF (`tiff32/Radiance_Images/`), bez obzira na odabrani format izvoza. | float32 |
| `reflectance` | Reflektancija ρ, gdje je **DN 32768 = ρ 1.0 (100%)** s rezervom do ρ 2.0. Spremno za Pix4D. | uint16 |
| `preview` | Render spremno za prikaz: RGB = balans bijele + gamma; multispektralno = rastezanje lažnih boja. | 8-bitni prikaz |

## Čitanje vrijednosti reflektancijskih piksela

Reflektancija se pohranjuje kao cjelobrojni digitalni broj, a **DN koji označava ρ = 1,0 (100 % reflektancije) ovisi o izvornoj kameri**:

| Izvorna kamera | ρ = 1,0 je DN | Kako odrediti |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva do ρ 2.0) | XMP oznaka `Chloros:PixelScale=32768` je upisana u datoteku. |
| Survey3 | `65535` (odrezano pri ρ 1.0) | Nema XMP oznaka `Chloros:*` — ta odsutnost je signal. |

**Pročitajte XMP oznaku `Chloros:PixelScale` i podijelite po njoj** umjesto da pretpostavljate konstantu. Oznaka je definirana u domeni uint16, stoga ostaje `32768` u formatima izlaza koji skaliraju — prvo normalizirajte pohranjenu vrstu podataka natrag na uint16 (×257 iz 8-bita, ×65535 iz float32).

{% hint style="warning" %}
**Jedan slučaj nema razmjer, po dizajnu.** Kada se snimanje iz 8-bitnog izvora (BayerRG8) zapisuje kao 8-bitni TIFF, pipeline ograničava raspon na 0–255 umjesto da vrši ponovno skaliranje, pa nijedna skala ne opisuje datoteku — Chloros tamo namjerno izostavlja `Chloros:PixelScale`. Ako oznaka nedostaje u datoteci reflektancije LATTICE, ne pretpostavljajte mjerilo; umjesto toga ponovno izvezite u 16-bitnom ili 32-bitnom formatu.
{% endhint %}

Za potpuna pravila (uključujući oznake kompatibilne s MicaSenseom), pogledajte **&quot;Čitanje pikselja reflektancije&quot;** u [Referenci za CLI](reference/cli-reference.md).
