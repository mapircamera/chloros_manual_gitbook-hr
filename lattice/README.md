# LATTICE kamere

LATTICE je modularni multispektralni sustav kamera tvrtke MAPIR za poljoprivredno i znanstveno snimanje. Svaka LATTICE kamera temelji se na Sony IMX265 senzoru s globalnim zatvaračem (**3,1 MP, piksela od 3,45 µm**) i povezuje se putem Ethernet mreže kao**GigE Vision** uređaj.

Chloros 1.3.0 upravlja LATTICE kamerama uživo — otkrivanje, uživo pregled, snimanje i sinkronizirane mreže više kamera — s tri sučelja:

| Sučelje    | Gdje                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | kartica **Kamere** u bočnoj traci programa Chloros                         | Windows 10/11 x64                                        |
| CLI        | obitelj naredbi `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Tražite hardver?**Moduli kamera, leće, filtri i pojasevi, okviri i montaža, kabeli, PoE i ožičenje okidača dokumentirani su u [**LATTICE korisničkom priručniku**](https://mapir.gitbook.io/lattice-camera). Ovo poglavlje pokriva upravljanje kamerama iz Chlorosa.

LATTICE snimke su standardne `.tif`/`.tiff` datoteke, a Chloros ih uvijek obrađuje počevši od sirove snimke. Pogledajte [CLIReference](../reference/cli-reference.md) i [SDKReference](../reference/sdk-reference.md) za potpunu površinu naredbi i APIa.

## Dvije konfiguracije senzora

| Konfiguracija | Senzor       | Filter                                | Što jedna kamera isporučuje                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer boja | trostruki propusni filtar                |**Tri kalibrirane trake iz jedne ekspozicije**                 |
| **M3M**| Monohromatski | jednostruki uskopojasni interferencijski filtar |**Jedna kalibrirana banda**; kombinirajte više M3M kamera za indekse |

Budući da je M3M kamera monokromatska iza jednog filtra, svaka banda dobiva vlastitu ekspoziciju. M3C kamera pokriva sva tri svoja pojasa jednim izlaganjem senzora.

## Modalne naredbe i imenovanje

Svaka kamera pohranjuje svoj identitet u GenICam `DeviceUserID` kao modalnu naredbu:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros ga prikazuje s prefiksom `LATT-` (na primjer `LATT-M3M-L87-F450`). Isti `LATT-…` niz upisuje se u EXIF `Model` oznaku svakog izvoza i koristi se kao naziv izlazne mape kamere u obrađenim projektima.

| Komponenta | Vrijednosti                                                   | Značenje                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Senzor    | `M3C` / `M3M`                                            | Bayer u boji / monohromatski                                                                        |
| Objektiv      | `L41` / `L87`                                            | Broj je **horizontalno vidno polje u stupnjevima**: L41 = usko (41°), L87 = široko (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ili `F<nm>` (M3M) | Vidi [Filteri i spektralne trake](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Niz modela pokreće sve dalje: Chloros određuje profil senzora, raspored pojaseva i tvorničku kalibraciju iz `DeviceUserID` + `DeviceSerialNumber`. Nema ništa za konfigurirati po kameri — pogledajte [Povezivanje kamera](connecting.md).

## Filtri i pojasevi

Središta pojaseva, ruboviFWHM-a i cijeli M3M katalog od 23 SKU-a specifikacije su proizvoda, stoga se nalaze u priručniku za hardver: [**Filtri i spektralni pojasevi**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ono što je važno s softverske strane: kod filtera u nizu modela određuje koje proizvode Chloros može izraditi. RGB-kamere s filtrima (`FRGB`) isporučuju samo proizvode za debayering i pregled — radijansa i reflektancija po pojasu nemaju smisla za širokopojasni senzor, stoga ih Chloros preskače i o tome obavještava. Svaki drugi filtar daje potpuni lanac radijansa → reflektancije → indeksa.

## Radiometrijska kalibracija ukratko

Svaka LATTICE kamera pojedinačno se kalibrira u tvornici prema lancu koji se može pratiti do NIST-a i isporučuje se sa svjedodžbom za svaku kameru. Što to obuhvaća, kako se mjeri i točnost koju možete navesti nalazi se u priručniku za hardver: [**Fabricka radiometrijska kalibracija**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

S softverske strane, važno je da Chloros odredi ispravnu kalibraciju kada se kamera poveže i zacementira primijenjene koeficijente u svaki izvoz — pogledajte [Povezivanje kamera](connecting.md).

## U ovom poglavlju

* [Povezivanje kamera](connecting.md) — automatsko otkrivanje, GUI dijalog za povezivanje, ekvivalenti za CLI/SDK i kako se rješava tvornička kalibracija (paket na kameri naspram oblaka) kada se kamera poveže.
* [Postavke kamere](camera-settings.md) — svaka kontrola na kartici Kamere: popis kamera, okno s postavkama za pojedinačnu kameru i okno s postavkama niza (uključujući uređivač ROI-ja i poravnanje).
* [Postavke i načini snimanja](capture.md) — Snimanje svega, pojedinačno / kontinuirano / intervalno snimanje, vrste izvoza i raspored mape sa snimkama.
* [Nizovi više kamera](arrays.md) — čarobnjak za povezivanje nizova, sonda za sinkronizacijski kabel, propusni opseg, PTP i prikaz živog niza.
* [Mono kamere i indeksi vegetacije](mono-indices.md) — M3M kamere i tijek rada poravnanje → slojevanje → indeksiranje.

Kompletna površina naredbi nalazi se u [Referenci za CLI](../reference/cli-reference.md) i [Referenci za SDK](../reference/sdk-reference.md).
