# Radni tijekovi za reflektanciju

DAQ senzor svjetla pretvara radiometrijske snimke u reflektanciju. Postoje dva različita radna tijeka:

1. **Jednosenzorski** — jedan DAQ mjeri dolaznu zračenost dok kamera snima, a Chloros
dijeli radijanciju kamere s tom referencom.
2. **S dva senzora** — dva DAQ senzora, jedan usmjeren prema nebu i jedan prema objektu, proizvode krivulju spektralne odbijivosti uživo bez upotrebe kamere.

## Jedan senzor + kamera (referenca prema dolje)

DAQ djeluje kao senzor svjetlosti prema dolje (DLS): kamera mjeri zračenje prema gore **L**(W/m²/sr/nm), DAQ mjeri zračenje prema dolje**E** (W/m²/nm), a Chloros
izračunava reflektanciju po pojasu kao:

> ρ = π · L / E

Očitanje DAQ-a uvijek je usklađeno s vremenom ekspozicije — zbog toga DAQ i kamere dijele PTP-disciplinirani sat (vidi [DAQ-E umrežavanje i sinkronizacija vremena](ethernet-ptp.md)). Postavite kapu za sunčevo svjetlo (Sunshine cosine cap) za rad na otvorenom i ispravno je deklarirajte; deklaracija kape izravno skalira E (vidi [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Za kvantitativni rad, imajte na umu karakteristiku instrumenta: kvantitativna iradijanca dobiva se iz prosjeka od najmanje 15 sekundi očitanja.

### Snimanje uživo

Povežite DAQ s kamerom na kartici Kamere: svaki panel postavki kamere ima padajući izbornik **Senzor svjetla** koji navodi sve povezane DAQ-ove (DAQ-U/M/E) s kartice Senzori svjetla; za sinkroniziranu niz, odabir senzora svjetla za cijeli niz prenosi se na svaki član (pojedinačni senzori i dalje mogu preuzeti kontrolu). Nakon povezivanja, spektri senzora napajaju DLS utor kamere, a izvozi reflektancije dijele se prema odgovarajućem očitanju. U nizu, član sa vlastitim senzorom koristi vlastito očitanje — za svoje pojaseve u kompozitu uživo, svoje izvoze i automatsko eksponiranje — dok ostali članovi koriste očitanje niza.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dva ponašanja koja vrijedi znati:

* **Nema DAQ veze → reflektancija se odbacuje, ne izmišlja.** Chloros
odustaje od proizvoda reflektancije i bilježi razlog preskakanja umjesto da tiho vrati slabiji proizvod.
* **Korišteno očitanje se čuva.** Za svaki kadar reflektancije, DAQ očitanje koje se zapravo primijenilo zapisuje se kao `.daq` datoteka u `daq/` mapu snimki, tako da se snimka može naknadno preraditi ([Snimanje i .daq format](recording.md)). `.daq.cameras.json` pored njega imenuje kamere koje opisuje, a ponovna obrada povezuje ga samo s slikama tih kamera.

### Obrada snimljenih slika

Za postprilazni obradu snimite `.daq` tijekom sesije i spremite ga uz snimljene slike — pipeline automatski rješava slike povezane vremenskim žigom, preuzimajući svu nedostajuću tvorničku kalibraciju s oblaka tvrtke MAPIR
pri prvom korištenju. Snimke GUI-ja automatski se dodaju u otvoreni projekt kada prestanu.

Referentna vrijednost odbijanja može se odabrati tijekom obrade — `--reflectance-source` na `chloros-cli process` ili postavka izvora odbijanja u postavkama projekta GUI-ja:

| Vrijednost | Ponašanje |
| --- | --- |
| `auto` (zadano) | Kalibracijska meta unutar kadra koja prolazi QA je apsolutna referenca; DAQ downwelling (ρ = π·L/E) je rezervna opcija. Cilj koji prođe QA s oznakom (spušten panel ili izvanredni pojas) ne dobiva automatski prednost: isprobavaju se lanci cilj, cilj+DAQ i samo DAQ, redom prema njihovoj mjerenoj ocjeni na sačuvanim panelima. |
| `daq` | Autoritativni DAQ |
| `target` | Strogi cilj unutar okvira; bez DAQ zamjene |

Pogledajte [Kalibracijske mete](../calibration-targets.md) za tijekove rada s metama i [poglavlje LATTICE](../lattice/README.md) i [CLI
Reference](../reference/cli-reference.md) za cjelovitu obradu. Prilikom čitanja izvezanih pikselâ reflektancije koristite utisnuti skalni faktor (LATTICE: 32768 = ρ 1.0, XMP `Chloros:PixelScale`; Survey3
: 65535) — vidi [Izlazni formati slika](../output-image-formats.md).

### Pojasevi izvan radiometrijski kalibriranog raspona DAQ-a

Radiometrijski kalibrirani raspon DAQ-a je ~374–974 nm. Chloros
odbacuje reflektanciju temeljenu na DAQ-u za bilo koji kanal kamere s manje od polovice svoje spektralne težine unutar tog raspona, pri čemu se kao razlog preskakanja navodi `dls-uncalibrated-band-<nm>`. Među isporučivim SKU-ovima to utječe samo na F988: reflektancija F988 kalibrira se pomoću panela za reflektanciju unutar scene: pojas se nalazi izvan kalibriranog raspona DAQ senzora svjetla, stoga Chloros
primjenjuje vašu najnoviju snimku panela i pohranjuje je između mjerenja panela. Ako se F988 kamera koristi samo s DAQ-om, Chloros
odbacuje reflektancu temeljenu na DAQ-u za tu traku s razlogom preskakanja `dls-uncalibrated-band-988` — rad sa panelom je podržani postupak.

## Dvostruki senzor (ambijentalno svjetlo + objekt)

Dva DAQ senzora — bilo koji par, preko bilo kojih prijenosnika — daju spektru reflektancije uživo bez kamere: jedan senzor okrenut je prema nebu (**izvor ambijentalnog svjetla**), a drugi prema objektu (**skener objekta**), a Chloros
izračunava za svaku valnu duljinu:

> R(λ) = object(λ) / ambient(λ)

(nula tamo gdje je ambient ≤ 0).

### U GUI-ju

Kada su oba senzora povezana na kartici Senzori svjetla, otvorite preklopnik za dodavanje senzora (gumb &quot;+&quot; na pločici grafikona u mrežnom prikazu) i odaberite **Kombiniraj ambijentalno + objekt**. Odaberite dva senzora u padajućim izbornicima Izvor ambijentalnog svjetla i Skeniratelj objekta te kliknite na Stvori. Skupina se pojavljuje kao vlastiti grafikon i kao redak u bočnoj traci s zelenom oznakom**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Ispod grafikona reflektancije (pregled u obliku popisa), **tablica indeksa vegetacije u stvarnom vremenu** izračunava indekse iz krivulje koristeći centre pojaseva na plavoj 450 / zelenoj 550 / crvenoj 670 / NIR
800 nm. Indeksi temeljeni na omjeru koji poništavaju apsolutnu skalu (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) uvijek se prikazuju; indeksi koji zahtijevaju apsolutnu reflektanciju (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
) pojavljuju se samo kada su oba senzora modeli s kalibracijom snage.

### Apsolutna naspram relativne — pravilo označavanja

Chloros
označava izlaz dvaju senzora prema onome što par senzora zapravo može tvrditi:

| Par senzora | Označavanje |
| --- | --- |
| Oba senzora su kalibrirana — tvornički paket je učitan | **Apsolutna reflektancija** |
| Jedan od senzora nije kalibriran | **Relativna reflektancija** |

Sva tri modela su radiometrijska: kada se učita tvornička kalibracijska skupina senzora, njegova spektra su apsolutna (W/m²/nm), stoga se omjer para kalibriranih senzora odnosi na apsolutnu očitu reflektanciju — transport to ne određuje. Senzor koji i dalje prenosi sirove brojeve (bundle nije dostupan) spušta rezultat na razinu relativne krivulje (spektralni oblik je i dalje valjan). Oba senzora trebala bi imati ispravno deklarirane kapice ([Cap Profiles &amp; Calibrated Range](caps-and-range.md)).

### Iz Python
a

Na zajedničkoj površini SDK
a ne postoji namjenski poziv za dvostruki senzor: otvorite dvije sesije s `chloros_sdk.connect_daq_sensor()` i sami izračunajte omjer njihovih spektra (`latest()`), primjenjujući istu konvenciju označavanja. (Alat za snimanje s dva senzora također postoji na internom direktnom hardverskom sučelju tvrtke MAPIR
, naveden je u [CLI
referenci](../reference/cli-reference.md) radi potpunosti — nije dio isporučenog CLI
-a; GUI tijek rada naveden gore podržani je način rada uživo.)
