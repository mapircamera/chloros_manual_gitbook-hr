---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Podržane kamere  

Chloros

obrađuje slike s dviju obitelji kameraMAPIR

na **svim platformama** (Windows

,Linux

amd64 iLinux

arm64/Jetson):

* **Survey3** — kamereSurvey3W

(široke) iSurvey3N

(uske). Ulaznica: `RAW+JPG`.
* **LATTICE**— M3C i M3M multispektralni moduli kamera. Ulaznica: snimke s `.tif`/`.tiff`. Kamere LATTICE mogu se također**kontrolirati uživo** sChloros

— putem kartice Cameras u GUI-ju (Windows

) ili s `chloros-cli lattice` /Python

SDK

(Windows

iLinux

) — uključujući sinkronizirane nizove više kamera. Pogledajte [LATTICE vodič](lattice/).

Procesna cijev također prihvaća ulazne datoteke `.dng`.

##Survey3

<table data-header-hidden><thead><tr><th width="156">Proizvođač</th><th width="250">Model kamere</th><th width="138">Model filtra</th><th width="187">Tip slike</th></tr></thead><tbody><tr><td><strong>Proizvođač</strong></td><td><strong>Model kamere</strong></td><td><strong>Model filtera</strong></td><td><strong>Tip slike</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Linija LATTICE modularni je multispektralni sustav kamera izgrađen na Sony IMX265 senzoru s globalnim zatvaračem (3,1 MP, pikseli od 3,45 µm). Svaka kamera pohranjuje svoj identitet kao niz modela:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros

prikazuje ga s prefiksom `LATT-` (na primjer `LATT-M3M-L41-F550`), a niz modela pokreće sve dalje — profil senzora, raspored pojaseva i kalibracija rješavaju se automatski; nema ništa što se mora konfigurirati za svaku kameru. Broj objektiva je **horizontalno vidno polje u stupnjevima**: `L41` = usko 41°, `L87` = široko 87°.

Postoje dvije konfiguracije senzora:

| Konfiguracija | Senzor      | Tip filtera                           | Pojasevi po kameri                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer boja | Trostruki propusni pojas                       | 3 spektralna pojasa iz jedne ekspozicije                                 |
| **M3M**       | Monohromatski | Jedan uskopojasni interferencijski filtar | 1 kalibrirana pojas — kombinirajte više M3M kamera za indekse vegetacije |

### Mogućnosti filtara M3C (Bayer)

| Filtar | Pojasevi (naziv @ središnja valna duljina nm / širina pojasa nm)       |
| ------ | ---------------------------------------- |
| `FRGB` |Blue

475/30 ·Green

550/30 ·Red

625/30  |
| `FRGN` |Red

660/21 ·Green

550/30 ·NIR

850/30 |
| `FOCN` |Orange

615/21 ·Cyan

490/38 ·NIR

808/14 |
| `FNGB` |Blue

475/30 ·Green

550/30 ·NIR

850/30  |

### Katalog M3M (mono) filtara — 23 SKU

F-broj je oznaka SKU-a; izmjerena traka (utisnuta u svaki kalibrirani izvoz) je skeniranje filtra po seriji:

| SKU    | Središnja valna duljina (nm, izmjereno) | Rubovi prozirnosti (nm) | Širina (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379.4                 | 367–392         | 25         |
| F405   | 403.9                 | 390–417         | 27         |
| F450   | 443.7                 | 430–458         | 28         |
| F485   | 489.7                 | 478–502         | 24         |
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
| F880   | — (nominalno)          | 867–893         | 26         |
| F905   | — (nominalno)           | 892–920         | 28         |
| F940   | 940.6                 | 923–958         | 35         |
| F950   | 945.1                 | 929–961         | 32         |
| F988 † | 985.3                 | 968–1003        | 35         |

&quot;Rubovi pojaseva mjere se kao vrijednosti pune širine na pola maksimuma iz skeniranja filtera po seriji tvrtkeMAPIR

— iste vrijednosti kojeChloros

utiskuje u svaki kalibrirani izvoz.&quot; &quot;— (nominalno)&quot; = još nema skeniranja serije; za te SKU-ove navedeni centar je broj SKU-a, a širina je brojka proizvođača.

† Reflektancija F988 kalibrira se pomoću panela reflektancije unutar scene: pojas se nalazi izvan kalibriranog raspona DAQ senzora svjetla, stogaChloros

primjenjuje vašu najnoviju snimku panela i zadržava je između promatranja panela. Pogledajte [Kalibracijske mete](calibration-targets.md).

Za upravljanje kamerom uživo, nizove, postavljanje mreže i radiometrijski obradni lanac, pogledajte [LATTICE vodič](lattice/).
