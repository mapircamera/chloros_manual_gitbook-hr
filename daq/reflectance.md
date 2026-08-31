# Radni postupci reflektancije

DAQ senzor svjetla pretvara radiometrijske snimke u reflektanciju. Postoje dva različita radna postupka:

1. **Jednosenzorski** — jedan DAQ mjeri dolaznu zračenost dok kamera snima, a EChloros

-a dijeli radijanciju kamere s tom referencom.
2. **S dva senzora** — dva DAQ senzora, jedan usmjeren prema nebu i jedan prema objektu, proizvode kontinuiranu spektralnu krivulju odbijanja bez upotrebe kamere.

## Jedan senzor + kamera (referenca prema dolje)

DAQ djeluje kao senzor svjetlosti prema dolje (DLS): kamera mjeri zračenje od tla **L**(W/m²/sr/nm), DAQ mjeri zračenje prema tlima**E** (W/m²/nm), aChloros

izračunava reflektancu po pojasu kao:

> ρ = π · L / E

Očitanje DAQ-a uvijek je usklađeno s vremenom ekspozicije — zbog toga DAQ i kamere dijele PTP-disciplinirani sat (vidi [DAQ-E Mreža i sinkronizacija vremena](ethernet-ptp.md)). Postavite kapu za sunčevo svjetlo (Sunshine cosine cap) za rad na otvorenom i ispravno je deklarirajte; deklaracija kape izravno skalira E (vidi [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Za kvantitativni rad, imajte na umu karakteristiku instrumenta: kvantitativna iradijanca dobiva se iz prosjeka mjerenja od najmanje 15 s.

### Snimanje uživo

Povežite DAQ s kamerom na kartici Kamere: svaki panel postavki kamere ima padajući izbornik **Senzor svjetla** koji navodi sve priključene DAQ-ove (DAQ-U/M/E) s kartice Senzori svjetla; za sinkroniziranu niz, odabir senzora svjetla za cijeli niz prenosi se na svaki član (pojedinačni kamere i dalje mogu nadjačati). Jednom povezani, spektri senzora napajaju DLS utor kamere, a izvozi reflektancije dijele se po usklađenom očitanju.



<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dva ponašanja koja vrijedi znati:

* **Nema povezanog DAQ-a → reflektancija se odbacuje, ne izmišlja.**Chloros

odbacuje proizvod reflektancije i bilježi razlog preskakanja umjesto da tiho vrati slabiji proizvod.
* **Korišteno očitanje se čuva.** Za svaki kadar reflektancije, DAQ očitanje koje se zapravo primijenilo zapisuje se kao `.daq` sporedna datoteka (sidecar) uz snimku, tako da se snimanje može ponovno obraditi kasnije ([Snimanje i .daq format](recording.md)).

### Obrada snimljenih slika

Za postletnu obradu, tijekom sesije zabilježite `.daq` i spremite ga uz slike — pipeline automatski rješava slike s odgovarajućim vremenskim oznakama, preuzimajući sve nedostajuće tvorničko kalibriranje s oblaka platformeMAPIR

pri prvom korištenju. Snimke GUI-ja automatski se dodaju otvorenom projektu kada prestanu.

Referentna vrijednost reflektancije može se odabrati tijekom obrade — `--reflectance-source` na `chloros-cli process` ili postavka izvora reflektancije u postavkama projekta GUI-ja:

| Vrijednost | Ponašanje |
| --- | --- |
| `auto` (zadano) | Kalibracijska meta u kadru koja prolazi QA je apsolutna referenca; DAQ downwelling (ρ = π·L/E) je rezervna opcija |
| `daq` | Autoritativno DAQ |
| `target` | Strogi cilj unutar kadra; bez DAQ zamjene |

Pogledajte [Calibration Targets](../calibration-targets.md) za tijekove rada s ciljevima i [poglavlje LATTICE](../lattice/README.md) te [CLI

Reference](../reference/cli-reference.md) za cijeli proces obrade. Prilikom čitanja izvezanih piksela reflektancije koristite utisnuti mjerilo (LATTICE: 32768 = ρ 1.0, XMP `Chloros:PixelScale`;Survey3

: 65535) — vidi [Format izlaznih slika](../output-image-formats.md).

### Pojasevi izvan kalibriranog raspona DAQ-a

Radiometrijski kalibrirani raspon DAQ-a je ~374–974 nm.Chloros

odbacuje reflektanciju temeljenu na DAQ-u za bilo koji spektar kamere s manje od polovice svoje spektralne težine unutar tog raspona, izvještavajući o razlogu preskakanja `dls-uncalibrated-band-<nm>`. Među isporučivim SKU-ovima ovo utječe samo na F988: reflektancija F988 kalibrira se pomoću reflektancijskog panela unutar scene: pojas se nalazi izvan kalibriranog raspona DAQ senzora svjetlosti, stogaChloros

primjenjuje vašu najnoviju snimku panela i pohranjuje je između promatranja panela. Ako se kamera F988 koristi samo s DAQ-om,Chloros

odbacuje reflektancu temeljenu na DAQ-u za tu traku s razlogom preskakanja `dls-uncalibrated-band-988` — podržani postupak je rad s panelom.

## Dvostruki senzor (ambijentalno svjetlo + objekt)

Dva DAQ senzora — bilo koji par, preko bilo kojih transporta — daju spektralni odraz u stvarnom vremenu bez kamere: jedan senzor usmjeren je prema nebu (**Izvor ambijentalnog svjetla**), a drugi prema objektu (**Skeniratelj objekta**), aChloros

izračunava po valnoj duljini:

> R(λ) = object(λ) / ambient(λ)

(nula tamo gdje je ambijentalno ≤ 0).

### U GUI-ju

Kada su oba senzora povezana na kartici Senzori svjetla, otvorite preklopnik za dodavanje senzora (gumb &quot;+&quot; na pločici grafikona u mrežnom prikazu) i odaberite **Kombiniraj ambijentalno + objekt**. Odaberite dva senzora u padajućim izbornicima Izvor svjetla okoline i Skeniratelj objekta i kliknite na Stvori. Skupina se pojavljuje kao vlastiti grafikon i kao redak u bočnoj traci s zelenom oznakom**REF**.



<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Ispod grafikona reflektancije (pregled u obliku popisa), **tablica indeksa vegetacije** u stvarnom vremenu izračunava indekse iz krivulje koristeći centre pojaseva na plavoj 450 / zelenoj 550 / crvenoj 670 / infracrvenojNIR

800 nm. Indeksi temeljeni na omjeru koji poništavaju apsolutnu ljestvicu (NDVI

,GNDVI

, ENDVI,WDRVI

,GRVI

,CVI

,GCI

, MSR) uvijek se prikazuju; indeksi kojima je potrebna apsolutna reflektancija (EVI

,SAVI

,OSAVI

, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI,LAI

, NLI, MNLI, FCI,GEMI

) pojavljuju se samo kada su oba senzora modeli kalibrirani po snazi.

### Apsolutna naspram relativne — pravilo označavanja

Chloros

označava izlaz dvaju senzora prema onome što par senzora zapravo može tvrditi:

| Par senzora | Označavanje |
| --- | --- |
| Oba senzora su kalibrirana — tvornički paket je učitan | **Apsolutna reflektancija** |
| Jedan od senzora nije kalibriran | **Relativna reflektancija** |

Sva tri modela su radiometrijska: kada se učita tvornička kalibracijska skupina senzora, njegova spektra su apsolutna (W/m²/nm), stoga se omjer para kalibriranih senzora odnosi na apsolutnu očitu reflektanciju — transport to ne određuje. Ako senzor još uvijek prenosi sirove brojeve (bundle nije dostupan), rezultat se svodi na relativnu krivulju (spektralni oblik i dalje je valjan). Oba senzora trebala bi imati ispravno deklarirane kapice ([Cap Profiles &amp; Calibrated Range](caps-and-range.md)).

### IzPython



Na zajedničkoj površiniSDK

nema namjenskog poziva za dvostruki senzor: otvorite dvije sesije s `chloros_sdk.connect_daq_sensor()` i sami izračunajte omjer njihovih spektra `latest()`, primjenjujući istu konvenciju označavanja. (Alat za snimanje s dvostrukim senzorom također postoji na internom hardverskom sučeljuMAPIR

, naveden radi potpunosti u [CLI

referenci](../reference/cli-reference.md) — nije dio isporučenogCLI

; GUI tijek rada naveden gore podržani je put za uživo.)
