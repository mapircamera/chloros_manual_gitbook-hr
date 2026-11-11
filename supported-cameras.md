---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Podržane kamere

Chloros obrađuje slike s dviju obitelji kamera MAPIR na **svim platformama** (Windows, Linux amd64 i Linux arm64/Jetson):

* **Survey3** — kamere Survey3W (široka) i Survey3N (uska). Ulaz: `RAW+JPG`.
* **LATTICE**— M3C i M3M multispektralni moduli kamera. Ulaz: snimke s `.tif`/`.tiff`. Kamere LATTICE mogu se također**kontrolirati uživo** iz programa Chloros — putem kartice Cameras u GUI-ju (Windows) ili putem `chloros-cli lattice` / Python SDK (Windows i Linux) — uključujući sinkronizirane nizove više kamera. Pogledajte [LATTICE vodič](lattice/).

Procesna cijev također prihvaća `.dng` ulazne datoteke.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Proizvođač</th><th width="250">Model kamere</th><th width="138">Model filtera</th><th width="187">Tip slike</th></tr></thead><tbody><tr><td><strong>Proizvođač</strong></td><td><strong>Model kamere</strong></td><td><strong>Model filtera</strong></td><td><strong>Tip slike</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Linija LATTICE modularni je sustav višespektralnih kamera izgrađen na Sony IMX265 senzoru s globalnim zatvaračem (3,1 MP, piksela 3,45 µm). Svaka kamera pohranjuje svoj identitet kao niz modela:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros ga prikazuje s prefiksom `LATT-` (na primjer `LATT-M3M-L41-F550`), a modelni niz pokreće sve dalje — profil senzora, raspored pojaseva i kalibracija rješavaju se automatski; nema ništa što treba konfigurirati za svaku kameru. Broj objektiva označava **horizontalno vidno polje u stupnjevima**: `L41` = usko 41°, `L87` = široko 87°.

Postoje dvije konfiguracije senzora:

| Konfiguracija | Senzor      | Tip filtera                           | Pojasevi po kameri                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer boja | Tropojasni propusni filter                       | 3 spektralna pojasa iz jedne ekspozicije                                 |
| **M3M**       | Monohromatski | Jednopojasni uskopojasni interferencijski filter | 1 kalibrirano pojas — kombinirajte više M3M kamera za indekse vegetacije |

### M3C (Bayer) opcije filtera

| Filter | Pojasevi (naziv @ sredina nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 618.4/46 · Cyan 494.1/40 · NIR 821.6/54 (izmjereno — vidi dolje) |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

Za `FOCN` vrijednosti su **izmjerena** polumaksimalna širina filtera (Chloros 1.3.0 i novije; ranije verzije su koristile vrijednosti iz tehničke specifikacije 615/21 · 490/38 · 808/14). Nazivi pojaseva ostaju `Orange_615`, `Cyan_490` i `NIR_808`, a izmjereni središnji valni broj i širina su ono što Chloros utiskuje u svaki kalibrirani izvoz.

### Katalog M3M (mono) filtara — 23 SKU-a

F-broj je oznaka SKU-a; izmjerena traka (otisnuta u svaki kalibrirani izvoz) je skeniranje filtra po seriji:

| SKU    | Središnja vrijednost (nm, izmjerena) | FWHM rubovi (nm) | Širina (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519.9                 | 504–536         | 32         |
| F550   | 548.4                 | 531–566         | 35         |
| F590   | 589.0                 | 570–608         | 38         |
| F615   | 623.8                 | 614–634         | 20         |
| F632   | 633.4                 | 616–651         | 35         |
| F650   | 651.1                 | 636–666         | 30         |
| F685   | 686.2                 | 675–698         | 23         |
| F715   | — (nominalno)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826.1                 | 810–843         | 33         |
| F850   | 846.5                 | 828–865         | 37         |
| F880   | — (nominalno)           | 867–893         | 26         |
| F905   | — (nominalno)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985.3                 | 968–1003        | 35         |

"Rubovi pojasa mjere se vrijednostima pune širine na pola maksimuma iz skeniranja filtara po seriji tvrtke MAPIR — iste vrijednosti koje Chloros utiskuje u svaki kalibrirani izvoz." "— (nominalno)" = još nije izvršeno skeniranje serije; za te SKU-ove navedeni središnji dio je broj SKU-a, a širina je vrijednost proizvođača.

† Reflektancija F988 kalibrira se pomoću panela za reflektanciju unutar scene: pojas se nalazi izvan kalibriranog raspona DAQ svjetlosnog senzora, stoga Chloros primjenjuje vašu najnoviju snimku panela i zadržava je između promatranja panela. Vidi [Ciljevi kalibracije](calibration-targets.md).

Za upravljanje kamerom uživo, nizove, postavljanje mreže i radiometrijski obradni lanac, vidi [LATTICE vodič](lattice/).
