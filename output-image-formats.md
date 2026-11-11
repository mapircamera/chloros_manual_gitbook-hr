---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Izlazni formati slika

Chloros izvozi obrađene proizvode u četiri formata datoteka. Odaberite format u postavkama projekta (GUI), pomoću `--format` (CLI) ili pomoću `export_format` (SDK). CLI i SDK prihvaćaju točne nizove navedene u nastavku.

| Stranica formata | Ekstenzija | Tip piksela | Raspon piksela | Napomene |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(zadano)* | `.tif` | uint16 digitalni broj | 0 – 65535 | Preporučuje se za fotogrametriju / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8 digital number | 0 – 255 | Za sve snimke od 8 bita, koje nude samo formate od 8 bita: Chloros ne povećava dubinu bita. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % reflektancije. Neke aplikacije ne mogu čitati TIFF-ove s pomičnim zarezom; datoteke su veće. |
| `PNG (8-bit)` | `.png` | uint8 digitalni broj | 0 – 255 | Kompresija bez gubitka, pogodna za pregledavanje na webu i vizualizaciju. |
| `JPG (8-bit)` | `.jpg` | uint8 digitalni broj | 0 – 255 | Kompresija s gubicima, najmanje datoteke. |

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

Složak kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE i `<model>_<filter>` (npr. `Survey3N_RGN`) za Survey3. **Svaki izvezeni proizvod zadržava naziv izvornog datoteka — proizvod se identificira putem mape, a ne nastavka naziva datoteke.** Pogledajte [Where the outputs land](reference/cli-reference.md) u referenci za CLI za potpuna pravila.

## LATTICE proizvodi (razine snimanja i izvoza)

Jedan sirovi LATTICE kadar razgranava se u svaki zatraženi proizvod u jednom prolazu. Svaka vrsta proizvoda ima vlastiti prekidač (GUI potvrdni okviri ili CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, sve je PO podrazumijevanim postavkama):

| Razina | Sadržaj | Tip podataka |
| `raw` | Podaci iz senzora u Bayer formatu (monokromatske kamere: jednobandni). Obrada uvijek započinje s neobrađenim (raw) podatcima. | Kao snimljeno |
| `debayered` | Linearna demosaikizacija — 3-kanalna za M3C, 1-kanalna u sivim tonovima za M3M: uklonjen je mračni offset kalibracijskog paketa, zatim slijedi demosaikizacija osjetljiva na rubove, bez korekcije linearnosti, ravnog polja ili unmixa. Izvoz `process` zapisuje istu datoteku koju snimanje sprema na ovoj razini, u svakom načinu obrade. | Linearni DN |
| `radiance` | Apsolutna spektralna zračnost iz punog radiometrijskog lanca, u **W/m²/sr/nm**. Uvijek se zapisuje kao 32-bitni TIFF (`tiff32/Radiance_Images/`), bez obzira na odabrani format izvoza. | float32 |
| `reflectance` | Reflektancija ρ, gdje je **DN 32768 = ρ 1.0 (100%)** s rezervom do ρ 2.0. Spremno za Pix4D. | uint16 |
| `preview` | Render spremno za prikaz: RGB = balans bijele + gamma; multispektralno = rastezanje lažnih boja. | 8-bitni prikaz |

### Orijentacija

Datoteke `raw` i `debayered` uvijek ostaju u **izvornoj** orijentaciji senzora; postavka preokretanja/rotacije kamere primjenjuje se samo na ostale proizvode (pregled, radijansa, reflektansa, indeks). Prilikom snimanja nizova, datoteke svakog člana ostaju u izvornom položaju, osim ako ne zatražite orijentaciju (`--orient` u CLI-u, `orient=True` u SDK-u). Svaka datoteka bilježi orijentaciju koja pripada njezinim pikselima u XMP-u — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — i `ChlorosOrientationApplied` navodi prenosi li se ta informacija u vlastitim pikselima (`false` na raw i debayered datotekama). Chloros Image Viewer obrađuje neobrađene i debayered datoteke prema tom zapisu samo za prikaz; same datoteke nikada se ne prepisuju. Datoteka bez zapisa (napisana prije verzije 1.3.0 ili drugim alatom) prikazuje se onako kako je pohranjena.

## Maske za ograničavanje

Proizvodi radijance i reflektance multispektralne kamere M3C koriste 3×3 dekonvoluciju pojaseva. Ako dekonvolucija u nekom pojasu proizvede negativnu vrijednost, ta se vrijednost ograničava na nulu (uživo prikazi ograničavaju se na isti način). Nova značajka u verziji 1.3.0: proizvod u kojem je barem jedan piksel podvrgnut klipiranju dobiva pripadajuću **masku za ograničavanje** (clamp mask) koja točno navodi koji su to pikseli i pojasevi:

* **Gdje:**u mapi `clamp_masks/`**pored** mapa s proizvodima, nikada unutar njih: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, i `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Što:** jednopojasni `uint8` TIFF na vlastitoj pikselnoj mreži proizvoda (uključujući orijentaciju, ROI, biniranje i poravnanje niza). Bit 0 (`mask & 1`) postavlja se tamo gdje je pojas u R kanalu proizvoda bio odsječen, bit 1 (`& 2`) G kanal, bit 2 (`& 4`) B kanal. Udio obrezanih piksela po pojasu također je zabilježen u proizvodu kao `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` i `Chloros:BandBUnmixClampFraction`.
* **Kako proizvod to prikazuje:** XMP oznaka `Chloros:UnmixClampMask` sadrži putanju maske u odnosu na mapu proizvoda (`../clamp_masks/<product folder>/<name>_clampmask.tif`, uvijek odvojen `/`-om), `none` kada nije bilo obrezanih piksela (maska se ne zapisuje), ili `unavailable` kada se maska nije mogla zapisati. Proizvod bez oznake nikada nije pokrenuo unmix (RGB- i mono kamere, raw i debayered razine).

Budući da nijedan direktorij proizvoda ne sadrži masku, direktorij proizvoda kao što je `Reflectance_Calibrated_Images/` možete predati Pix4D-u, Metashapeu ili drugom softveru za fotogrametriju onakav kakav jest, uključujući poddirektorije — predajte direktorij proizvoda, a ne format direktorij iznad njega. Chlorosov vlastiti uvoz preskače maske za obrezivanje. CLI ih broji zasebno: `Image products written: N (plus M clamp masks)`.

## Indeks proizvoda

Svaki indeks smješta se u vlastitu mapu `<INDEX>_Index_Images/` (npr. `NDVI_Index_Images/`).

* **Integer formati**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) pohranjuju**linearno istezanje po slici**, a ne mapu −1…1: DN 0 je najniža indeksna vrijednost te slike, a puni raspon njezina najviša. Datoteka sadrži `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` i `Chloros:IndexMax`, pa `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Budući da svaki kadar ima vlastite krajnje točke, uspoređujte kadrove kao vrijednosti, a ne kao DN.
* **`TIFF (32-bit, Percent)`** pohranjuje same indeksne vrijednosti kao float32, bez ponovnog skaliranja — najbolji izbor za kvantitativni rad unutar leta.

Novo u verziji 1.3.0, indeksni proizvod opisuje **sam sebe** umjesto da ponavlja oznake izvornog sirovog materijala. Njegove GPS i kamerne oznake kopiraju se iz proizvoda iz kojeg je izračunat (proizvod reflektancije, ako postoji), a zatim:

| Oznaka | Vrijednost |
| `Chloros:ProcessingLevel` | `index` (tako da se datoteka nikada ne uvozi ponovno kao snimka) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / korištena formula |
| `Chloros:IndexLevel` | `reflectance` ili `radiance` — na temelju čega je indeks izračunat (izostavljeno ako ni jedno ni drugo) |
| `Chloros:IlluminationInvariant` | `True` za indeks izračunat iz reflektancije, `False` iz radijance ili vrijednosti senzora |
| `Chloros:ChlorosPipelineVersion` | verzija Chlorosa koja je napisala indeks |

`Chloros:PixelScale` uklonjen je iz indeksnih proizvoda — upotrijebite `IndexMin` / `IndexMax` iznad. Cjelokupna uputa za čitanje natrag nalazi se pod naslovom &quot;Integer index raster is a per-image contrast stretch&quot; u referenci [CLI](reference/cli-reference.md).

## Čitanje vrijednosti piksela reflektancije

Reflektancija se pohranjuje kao cjelobrojni digitalni broj, a **DN koji znači ρ = 1,0 (100 % reflektancije) ovisi o izvornoj kameri**:

| Izvorna kamera | ρ = 1,0 je DN | Kako utvrditi |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva do ρ 2.0) | XMP oznaka `Chloros:PixelScale=32768` je upisana u datoteku. |
| Survey3 | `65535` (odrezano pri ρ 1.0) | Nema oznake `Chloros:PixelScale` — ta odsutnost je signal. (Survey3 proizvodi nose i druge `Chloros:*` oznake obrade, kao što je `Chloros:CalibrationApplied`, u svakom načinu obrade od verzije 1.3.0.) |

Pročitajte `Chloros:PixelScale` XMP oznaku i podijelite po njoj umjesto da pretpostavljate konstantu. Oznaka je definirana u domeni uint16, stoga ostaje `32768` u formatima izlaza koji skaliraju — prvo normalizirajte pohranjenu vrstu podataka (dtype) natrag na uint16 (×257 iz 8-bita, ×65535 iz float32).

{% hint style="warning" %}
**8-bitni snimci.** Izvozni podaci 8-bitnog snimka su 8-bitni. Njegova reflektancija se ponovno skalira i označava se oznakom `Chloros:PixelScale=32768`, tako da se skala zadržava; DN proizvodi od 8 bita i datoteke reflektancije spremljene prije verzije 1.3.0 ograničeni su na raspon 0–255 i ne nose oznaku mjerila. Ako oznaka nedostaje u datoteci reflektancije LATTICE, nemojte pretpostavljati mjerilo; umjesto toga ponovno izvezite u 16-bitnom ili 32-bitnom formatu.
{% endhint %}

Za potpuna pravila (uključujući oznake kompatibilne s MicaSenseom), pogledajte **&quot;Čitanje piksela odbijanja&quot;** u [Referenci za CLI](reference/cli-reference.md).
