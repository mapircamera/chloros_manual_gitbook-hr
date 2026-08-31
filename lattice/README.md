# LATTICE kamere

LATTICE je modularni multispektralni sustav kamera tvrtke MAPIR za poljoprivredno i znanstveno snimanje. Svaka LATTICE kamera temelji se na Sony IMX265 senzoru s globalnim zatvaračem (**3,1 MP, pikseli od 3,45 µm**) i povezuje se putem Etherneta kao**GigE Vision** uređaj.

Chloros1.2.0 upravlja LATTICE kamerama uživo — otkrivanje, pregled uživo, snimanje i sinkronizirane mreže više kamera — s tri sučelja:

| Sučelje    | Gdje                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | kartica **Kamere** u bočnoj traci Chloros                        | Windows 10/11 x64                                        |
| CLI        | obitelj naredbi `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Tražite li hardver?**Moduli kamera, leće, filtri i pojasevi, okviri i montaža, kabeli, PoE i ožičenje okidačima dokumentirani su u [**LATTICE korisničkom priručniku**](https://mapir.gitbook.io/lattice-camera). Ovo poglavlje obuhvaća upravljanje kamerama iz Chloros.

LATTICE snimke su standardne `.tif`/`.tiff` datoteke, a Chloros ih uvijek obrađuje počevši od sirove snimke. Pogledajte [CLI reference](../reference/cli-reference.md) i [SDK reference](../reference/sdk-reference.md) za potpunu naredbu i površinu API.

## Dvije konfiguracije senzora

| Konfiguracija | Senzor       | Filter                                | Što jedna kamera pruža                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer boja | trostruki propusni filtar                |**Tri kalibrirane trake iz jedne ekspozicije**                 |
| **M3M**| Monohromatski | jednobandni interferencijski filtar uskog pojasa |**Jedan kalibrirani pojas**; kombinirajte više M3M kamera za indekse |

Budući da je M3M kamera monohromatska iza jednog filtra, svaki pojas dobiva vlastitu ekspoziciju. M3C kamera pokriva sva tri svoja pojasa jednim izlaganjem senzora.

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

Niz modela pokreće sve dalje: Chloros rješava profil senzora, raspored pojaseva i tvorničku kalibraciju iz `DeviceUserID` + `DeviceSerialNumber`. Nema ništa za konfigurirati po kameri — pogledajte [Povezivanje kamera](connecting.md).

## Filtri i pojasevi

Središta pojaseva, rubovi FWHM i cijeli M3M katalog od 23 SKU nalaze se u specifikacijama proizvoda, stoga se nalaze u priručniku za hardver: [**Filtri i spektralni pojasevi**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ono što je važno s softverske strane: kod filtera u nizu modela određuje koje proizvode Chloros može izraditi. Kamere s filtrom RGB (`FRGB`) emitiraju samo debayered i pretpregledne proizvode — radijansa i reflektancija po pojasu nemaju smisla za širokopojasni senzor, stoga ih Chloros preskače i o tome obavještava. Svaki drugi filtar daje potpuni lanac radijance → reflektance → indeksa.

## Radiometrijska kalibracija ukratko

Svaka LATTICE kamera pojedinačno se kalibrira u tvornici prema lancu koji se može pratiti do NIST-a i isporučuje se s certifikatom za svaku kameru. Što to obuhvaća, kako se mjeri i točnost koju možete navesti nalazi se u priručniku za hardver: [**Factory Radiometric Calibration**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

S softverske strane, važno je da rješenje za radiometrijsku kalibraciju (Chloros) odabire ispravnu kalibraciju kada se kamera poveže i zadržava primijenjene koeficijente u svakom izvozu — pogledajte [Povezivanje kamera](connecting.md).

## U ovom poglavlju

* [Povezivanje kamera](connecting.md) — automatsko otkrivanje, GUI dijalog za povezivanje, ekvivalenti naredbi CLI / SDK i kako se rješava tvornička kalibracija (paket na kameri naspram oblaka) kada se kamera poveže.

Dodatne teme o LATTICE-u — postavke kamere i upravljanje uživo, načini snimanja, nizovi više kamera i obrada mono (M3M) snimaka i indeksi — obrađene su u vlastitim odjeljcima ovog priručnika, a cjeloviti skup naredbi nalazi se u [Referenci za CLI](../reference/cli-reference.md) i [SDK Reference](../reference/sdk-reference.md).
