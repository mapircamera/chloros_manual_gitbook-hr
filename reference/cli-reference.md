# Chloros CLI Referenca

**Verzija:**

1.3.0**Generirano:**2026-07-29 19:19 ·**Revidirano:** 2026-11-06**Namijenjeno:** Optimizirano za LLM; čitljivo za ljude.**Opseg:** Svaka podkomanda `chloros-cli` namijenjena korisnicima, s opcijama i primjerima za kopiranje i lijepljenje.

Ovaj dokument je potpuna referenca za naredbeni alat `chloros-cli` koji se isporučuje s MAPIR Chloros. Namjerno je iscrpan kako bi LLM (ili čovjek) mogao sastaviti bilo koji podržani tijek rada iz navoda u nastavku bez pregledavanja izvornog koda.

Ako vam trebaju samo najvažnije informacije, idite na:
- [Brzi početak u pet minuta](#five-minute-quickstart)
- [LATTICE Camera First-Connect Workflow](#lattice-camera-first-connect-workflow)
- [DAQ Sensor First-Connect Workflow](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Načini snimanja, snimači i offline reprocesiranje](#capture-modes-recorders--offline-reprocess)

> **Pokrećete li Chloros s AI asistenta?** Claude Desktop, Claude Code, Cursor i drugi MCP klijenti mogu pokretati obradu, LATTICE kamere i nizove te DAQ senzore svjetla putem `chloros-mcp`, poslužitelj MCP-a koji se instalira uz Python SDK (i uz Linux .deb, u `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Postavke, opcije i alati nalaze se u referenci za SDK u odjeljku [MCP Server (AI Assistants)](sdk-reference.md#mcp-server-ai-assistants). Pomoćnik sa shellom također može izravno pokrenuti svaku naredbu na ovoj stranici.

> **Promjene u verziji 1.3.0** (koje postojeći skript ili tijek rada verzije 1.2.0 primjećuje).
>
> **Promjene koje skript verzije 1.2.0 možda treba obraditi:**
>
> - **Primjenjuje se `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` i `align-stream` su ga uklonili u verziji 1.2.0 i pokrenuo se na postavkama same kamere. Sada postavlja format preseta, ekspoziciju, pojačanje i brzinu snimanja: `--preset high_speed` snima 8-bit pri 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` također zadržava format preseta (u 1.2.0 kamera je postavljena na 12-bit nakon primjene preseta). Vidi [Uobičajene opcije](#common-options-shared-by-most-subcommands).
> - **Bayerov pikselni format na M3M (mono) kameri je njezina dubina.** Preseti nazivaju Bayerove formate koje mono senzor ne može podržati: u verziji 1.2.0 kamera je odbila format i zadržala onaj koji je imala, tako da `--preset high_speed` tamo nije bio 8-bitan. Sada je to Mono format iste dubine: `high_speed` i `--pixel-format BayerRG8` daju `Mono8`, `high_quality`, `triggered` i `BayerRG12` daju `Mono12`. U mješovitoj M3C + M3M nizu svaka kamera uzima zajednički format unutar svoje obitelji, a `network-analysis` / `analyze-array --pixel-format` projektira svaku kameru na toj dubini. Isto vrijedi i obrnuto: M3C koji zatraži Mono format preuzima Bayer format te dubine, pa niz čiji je master M3M više ne otvara svoje M3C kamere u Mono formatu. Pogledajte [Mono (M3M) jednobandne kamere](#mono-m3m-single-band-cameras).
> - **`lattice array-capture` izvozi su prema zadanim postavkama poravnati, kao i u aplikaciji.** Svakog člana `debayered`, `display`, `radiance`, `reflectance` i indeksne datoteke iskrivljene su u kadar glavne kamere i obrezane prema preklapanju članova (manji od senzora za offset kamere, `XMP-mapir:AlignmentJSON` `"applied": true`), pa je snimanje putem CLI-a ista datoteka kao i snimanje iste scene aplikacijom Chloros; `raw` ostaje u izvornom obliku. Prije je snimanje bez `--aligned` spremalo svaku datoteku u izvornim dimenzijama, neizobličenu. `--no-aligned` to zadržava. Isto vrijedi i za `--smart`, `--continuous` i `--interval`, kao i za SDK i `chloros-mcp`. Vidi [`array-capture` načini](#array-capture-modes).
> - **`lattice array-capture --compression` prema zadanim postavkama je `none`.** Datoteke se spremaju nekomprimirano (otprilike 6,3 MB po kadru pune rezolucije) radi najbrže kontinuirane brzine; verzija 1.2.0 spremala je `deflate` (otprilike 4,1 MB). Koristite `--compression deflate` za manje datoteke. Oba su formata bez gubitaka.
> - **PNG i JPEG podržavaju samo `display` / `preview` i indeksne proizvode.** Odabir `--format png` ili `jpg` s razinom koju 8-bitna datoteka ne može pohraniti odbacuje se prije nego što se išta snimi (HTTP 400, izlaz `1`), kao i zadani `--processing all` s `--format png`/`jpg`. Vidi [Capture Export Levels](#capture-export-levels-the-all-default).
> - **`lattice dls` treba `--mac`.**Više nema zadane MAC adrese (bila je adresa jedne jedinice); `daq pool-discover --only ble` pronalazi adresu senzora.**`lattice dls --camera SERIAL`** je nov.
> - **Više pogrešaka izlazi iz `1`.** `process` kada se izvozi pokretanje nije moglo provjeriti ili je izgubilo sliku, iako su ostali proizvodi zapisani; naredba snimanja s bilo kojom datotekom koju nije moglo zapisati (ostale datoteke se i dalje spremaju); `lattice continuous` i `multi-stream` pri punom disku; `array-burst` / `array-record` kada snimač prijavi pogrešku; `array-build-video` kada je traženi proizvod preskočen ili nije izgrađen prema zahtjevu; `project burst` / `stream` kada snimanje zakaže usput; `project sensor log` kada snimanje ne sadrži nikakvo očitavanje; `lattice exposure --set`, `gain --set` i `configure --set` s vrijednošću koju kamera odbija. `lattice network-analysis --json` izlazi kao što i sažetak (`1` za `needs_force_slip`). Obrnuto: `daq pool-disconnect` senzora koji više nije u skupu izlazi iz `0`. Vidi [Kodovi izlaska](#exit-codes).
> - **Spori `lattice array-connect` nikada ne završava pogreškom kada je niz spojen.** Čeka do 600 s (180 s u verziji 1.2.0; `project connect` 90 s), ispisujući tijekom toga korak pozadinskog sustava, a veza koja do tada nije uspostavljena poništava se od strane pozadinskog sustava — svaka kamera vraćena u stanje u kojem je pronađena — stoga `Error:` (izlaz `1`) znači da nijedna matrica nije povezana. U verziji 1.2.0, povezivanje odmah nakon uključivanja kamera moglo je trajati dulje od vremena čekanja i povezati niz nakon što je naredba propala. Ctrl+C sada otkazuje povezivanje. Vidi [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` zadržava cijeli senzor.** Bez `--width` / `--height` / `--binning`, svaka je kamera postavljena na cijeli senzor i brzina snimanja pada na razinu koju ovaj računalo i mreža mogu pružiti; izlaz za povezivanje navodi brže opcije (biniranje ili manji ROI) s njihovim projektiranim brzinama. Verzija 1.2.0 smanjivala je ili binirala okvire kako bi održala brzinu. Pogledajte [`array-connect` opcije](#array-connect-options).
> - **Spora mreža više ne odbija niz.** Svaka kamera se tempira prema svom dijelu mreže i brzina sličica pada s njom. Gdje kamere ne mogu raditi tako sporo, niz radi u serijskom emitiranju (`slip-emit-and-capture`): svaki se kadar prima, ali kamere nisu istovremeno eksponirane. `--force-tier sim-capture-sim-emit` inzistira na istovremenoj ekspoziciji. Vidi [FPS i burst model niza](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` projektira što se povezuje**: bez `--pixel-format`, formati koje postavlja connect (Mono12 / BayerRG12, prije je bio BayerRG8), i odluka o [spakiranom 12-bitnom](#packed-12-bit-transport---packed) prijenosu. Ista naredba stoga može navesti drugačiju brzinu sličica nego u verziji 1.2.0.
> - **Relativne putanje odnose se na lokaciju na kojoj pokrećete naredbu.** `-o` i ostale putanje koje pozadinski sustav otvara (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), `output` koraka recepta, i zadani `output/` od `lattice index --live` rješavaju se u odnosu na mapu iz koje pokrećete `chloros-cli`. U verziji 1.2.0 pozadinski je servis rješavao te putanje u vlastitoj instalacijskoj mapi, što je u instaliranom buildu dovodilo do pogreške "Pristup je odbijen". Pogledajte [Konvencije](#conventions).
> - **`lattice index` radi na pozadinskom procesu u svakom obliku**, kao i ostale `lattice` naredbe, stoga prvo pokrenite pozadinski proces. Offline indeks (`--input`) izračunava backend, koji čita TIFF i zapisuje rezultate pored njega ili na `-o`, i `--list-presets` / `--list-gradients` ispisuju tablice backenda. Na Windowsu sada rade: prije je Windows CLI odgovarao na svaki oblik `lattice index`, uključujući `--live`, s `Error: lattice_sdk not available`. Na Linuxu, offline indeks je ranije radio unutar CLI bez backenda i prijave; sada su mu potrebni i jedno i drugo, kao i svakoj drugoj CLI naredbi. Pogledajte [Index / Vegetation Maths](#index--vegetation-maths).
> - **Nove datoteke pored proizvoda.** Svaki proizvod radijance, reflektance i indeksa u kojem je dekonvolucija pojasa zaključala piksel dobiva masku u mapi `clamp_masks/` pored mape proizvoda, a snimke bilježe liniju `clamp mask:`. Vidi [Unmix clamp maska](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orijentacija kamere.** Snimanje s jednom kamerom sprema `raw` i `debayered` onako kako ih senzor pročita i bilježi okretanje / rotaciju kamere u XMP-u; `process` primjenjuje je na proizvode koje stvara. Datoteke članova niza ostaju u izvornom obliku osim ako ne proslijedite novi `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Pogledajte [Orijentacija kamere](#what-a-captures-folder-looks-like).
> - **Odbijeno je ime razine izvoza koje nije jedno.** `lattice capture --processing` i svaka lista `--levels` uzimaju `raw`, `debayered`, `radiance`, `reflectance` ili `display` (ili alias kao što je `preview`; `--processing` također `all`). Bilo koje drugo ime je odbijeno prije nego što se dotakne kamera (HTTP 400 `UNKNOWN_LEVEL`, izlaz `1`); 1.2.0 je spremio `raw` za to i izašao `0`.
> - **`daq pool-set-cap` treba `--cap-id`.** Bez toga se naredba izlazi `2`; 1.2.0 uklonio je korekciju kapa, ostavljajući goli senzor. `--cap-id none` traži golu sondu.
> - **`lattice resolution --binning-mode` je uklonjen.** Binning je uvijek prosjek; zastavica se i dalje prihvaća, ispisuje upozorenje i ne radi ništa. Vidi [Podešavanje senzora](#sensor-tuning).
> - **`process --debayer texture-aware` je odbijen za LATTICE snimke.** Za mapu koja sadrži LATTICE snimke, `process` izlazi iz `1` nakon uvoza, pri čemu naznačuje koliko ih ima, prije nego što išta obradi. LATTICE snimke uvijek prolaze standardnu demosaicizaciju, pa je verzija 1.2.0 prihvatila opciju i zanemarila je; aplikacijaChloros je ni za njih ne nudi. Pogledajte [`chloros-cli process`](#chloros-cli-process).
> - **Kamere se vraćaju onakve kakve su pronađene — osim `--keep-settings`.** `lattice array-disconnect`, svaka privremena naredba (`multi-capture` bez niza, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) i `lattice continuous` vraćaju svaku kameru u stanje u kojem su je zatekli: njezino ožičenje okidača i sinkronizacijske linije, njezin PTP i svaku postavku kamere koju je sesija promijenila (ekspozicija i pojačanje, cilj AE, ograničenja i prigušivanje, balans bijele, razina crne, format piksela, biniranje i ROI, preokreti, brzina sličica). `--keep-settings` umjesto toga sprema svaku kameru postavke sprema u vlastitu memoriju kamere, tako da se ona pokreće s njima; `--clear-settings` resetira tu memoriju na tvorničke postavke. U oba slučaja kamere ostaju u slobodnom radu. Projekt postavlja svoje kamere prema vlastitim spremljenim postavkama kada ih poveže. Vidi [Memorija kamere](#camera-memory----keep-settings----clear-settings).
> - **Svaki način rada sprema projekt na isti način.** Svaka promjena na kameri, nizu ili senzoru svjetla povezanom u sklopu projekta — iz aplikacije, CLI-a, SDK-a ili MCP poslužitelja — sprema se u taj projekt. Prije je samo aplikacija spremala projekt, a svako sljedeće spremanje prebrisalo je sve ostalo. Pogledajte [Jedan projekt](#one-project). `sensors.json`, a otvaranje projekta u bilo kojem načinu rada postavlja uređaje iz njega. Prije je projekt spremala samo aplikacija, a svako sljedeće spremanje prebrisalo je sve ostalo. Vidi [Jedan zapis projekta, svaki način rada](#one-project-record-every-mode).
> - **Datoteke izrađene od više kamera imenuju svaku od njih.** Višekanalni sloj (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks uživo napravljen od pojaseva više kamera i slika nizovog snimanja `composite/` nose serijski broj, model i paket kalibracije svake izvorišne kamere kao popise (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), a EXIF `SerialNumber` popise serijskih brojeva, spojene zarezom. Ranije nisu sadržavali nikakvu kameru, ili samo serijski broj prve kamere. Pogledajte [Datoteke izrađene od više kamera](#files-made-from-several-cameras).
> - **Samo datoteka `raw` sadrži oznake kalibracije kamere.** Komplet kompatibilan s MicaSenseom (`BlackLevel`, `RadiometricCalibration`, per-kanalni `Band{R,G,B}RadiometricCalibration`, skupova `MicaSenseVignetting*`) i tereta `CalibrationJSON` opisuju sirove vrijednosti senzora, stoga se nalaze samo u datoteci snimanja `raw`. Svaka druga datoteka — `debayered`, `display`, `radiance`, `reflectance` i indeksne datoteke, `lattice continuous` i `multi-stream` kadri, i svaki `process` proizvod — nosi identifikaciju kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) i, pri snimanju, vlastiti zapis okvira (`XMP-mapir:FrameJSON`: ekspozicija, pojačanje, temperatura, format piksela, vrijeme snimanja). U verziji 1.2.0 prenijeli su i sirovi skup, koji alat primjenjuje (Pix4D, Metashape) primijenjen na piksele koji nisu sirovi podaci. `process` pronalazi kalibraciju takve datoteke putem njezina serijskog broja i `CalBundleSha256`.
>
> **Ostale promjene:**
>
> - **Automatska brzina snimanja niza stabilizira se na onome što niz isporučuje.** Brzina koja izgubi više od 1 % svojih okidača se smanjuje, pri čemu se prosuđuje na temelju nekoliko duljih snimanja umjesto jednog, a brzina s koje je smanjena povremeno se ponovno isprobava. Brzina koju sami postavite ostaje nepromijenjena.
> - **Automatsko izlaganje je stabilnije.** Drži konstantnu svjetlinu, stabilizira se unutar nekoliko sekundi nakon promjene, zanemaruje male odbljeske i vraća rad kamere koja prestane reagirati na jakom suncu. Pogledajte [Kako se ponaša automatska ekspozicija](#how-auto-exposure-behaves).
> - **`process` broji samo vaše fotografije i snimke.** Ponovno pokretanje na mapi koja sadrži projektov `camera_cal/` više ne broji te datoteke za kalibraciju ("Pronađeno 96 slika" na setu od 90 slika), a projekt koji ih je već uvrstio, izbacuje ih.
> - **`lattice multi-test` prelazi iz stanja uključivanja kamera.** Uključuje PTP kamera za izvođenje, čeka da se njihovi satovi zaključaju i stabiliziraju prije prvog okidača — svaka kamera zaključana je 120 s, otprilike 2 minute od uključivanja (`--ptp-settle SECONDS` ga postavlja), budući da sat koji se upravo zaključao odstupa za do ~200 us do otprilike 2 minute — i vraća PTP svake kamere u stanje u kojem ga je zatekao. Kamera koja je još uvijek zaključana iz prethodnog testa više se ne čeka. Prije je isključivao PTP na mreži s jednim prekidačem na kojoj nije odgovorio nijedan DAQ-E i čitao slobodno tekući sat svake kamere kao PTP sat koji se nije mogao zaključati (`PTP is not locked`, svaki pokret). Vidi [Više kamera (privremeni spojevi)](#multi-camera-transient-sessions).
> - **Zapis `.daq` navodi je li njegov kapak odabran ili pretpostavljen.** DAQ-U, DAQ-M i DAQ-E isporučuju se bez kapa, a Chloros ne može vidjeti je li kapa postavljena, stoga senzor spojen bez kapa dobiva `sunshine_cosine` prema zadanim postavkama, što je *pretpostavka*. Svako snimanje pomoću Chloros (šema 1.30) sada pohranjuje `cap_id_source` uz `cap_id`: `operator` kada ste nazvali kapu (uključujući `none`), `auto_default` kada je bila zadana. Prerađivanje snimke za koju je kapu pretpostavljeno, s projektnom DAQ kapom ostavljenom na Auto, ispisuje `[daq-dls] WARNING` s nazivom datoteke: goli DAQ-U pod korekcijom sunčeve svjetlosti čita otprilike 30x više. **Nazovite kap koji ste prilagodili** kada se povežete, ili postavite DAQ kap projekta, i upozorenje će nestati.
> - **Nema više pip konzolnih prozora na Windowsu.** Na računalu s instaliranim Pythonom, pozadinski program i svaki od njegovih radnika za slike jednom su pokrenuli `pip` pri pokretanju, i svaki je otvorio konzolni prozor. To više ne čine.
> - **Snimke uživo `.daq` preživljavaju nestanak struje.**Snimanje se zapisuje s dnevnikom i potpunom sinkronizacijom diska pri svakom commitu, iza senzora, tako da spori disk odgađa datoteku, a nikada očitanja. Snimanje zaustavljeno usred commit-a (prekid napajanja, kill) može ostaviti `<name>.daq-journal` pored njega:**kopirajte ih oboje zajedno.**Chloros vraća zapisnik unatrag prvi put kad pročita datoteku, tako da se snimanje otvara sa svim što je potvrđeno prije zaustavljanja (gube se zadnje otprilike 3 s); u verziji 1.2.0 prekid napajanja u pogrešnom trenutku mogao je ostaviti cijelu datoteku nečitljivom ("database disk image is malformed").
> - **Zapisivanje na zauzetom disku više ne blokira senzor.** `daq pool-record` stavlja očitavanja u red i zapisuje ih na vlastitoj niti; očitavanje koje nije moguće zapisati se broji (`daq pool-list` prikazuje `N write error(s)` s posljednjom greškom) umjesto da zamrzne senzor (jedno pisanje trajalo je 8,4 s pored četiri kamere koje su spremale TIFF-ove).
> - **`lattice multi-capture` bez niza više ne gubi učitane okvire kada istekne vrijeme okidača.** Okidač kamere koji istekne (`SC_ERR_TIMEOUT -1011`, viđeno s četiri kamere na jednom 1 GbE prekidaču) prekida izvršavanje na toj grupi kadrova: grupe koje su već spremljene navode se s `Saved:`, zatim slijedi pogreška, izlaz `1`. U verziji 1.2.0 to je bila greška HTTP 500 bez popisa, iako su datoteke bile na disku.
> - **Kamere s softverskim okidačem više ne propuštaju svaki drugi okidač.** Kamera u načinu rada s softverskim okidačem sada stavlja u red okidač koji stigne tijekom čitanja (ili Chloros čeka da bude spreman), kao što su to već radile kamere sinkronizirane kabelom.
> - **`daq pool-discover` imenuje DAQ-E-S `DAQ-E-S`** u svojoj koloni `MODEL` (i `--json`-ov `model`). Prikazivalo je `DAQ-E` za svaki Ethernet senzor; onaj koji ne odgovori na provjeru identiteta unutar otprilike 2 s i dalje je naveden kao `DAQ-E`.
> - **Poboljšani LATTICE indeksni proizvodi** (NDVI i drugi indeksi) iz `process`, u svakom načinu obrade.
> - **OCN kamere: poboljšana radijanca, reflektancija i indeksni proizvodi na sunčevoj svjetlosti.** Korištenje korekcije flote primjenjuje se nakon odvajanja po trakama; kamere RGN ostaju nepromijenjene, a vrijednosti OCN razlikuju se od onih u verziji 1.2.0. Primjenjuje se i na snimku OCN napravljenu ranijom verzijom, kada se obrađuje na računalu koje ima kalibraciju s kojom je snimka napravljena (Chloros zadržava kalibraciju kamere nakon što je kamera povezana).
> - **`lattice continuous` bilježi sirovi kadar.** Pokretanje `tiff` zapisuje demosaicirane 3-slojne snimke, s kojih `process` ne može započeti. Njegove snimke sada su datoteka `raw` koju zapisuje niz-snimka: jedna mozaik-ploča u vlastitim vrijednostima senzora, nekomprimirana, s paketom kalibracije. Skripta koja čita tri ravnine iz tih datoteka čita samo jednu.
>
> - **Svako snimanje sprema DAQ očitanje koje je upotrijebilo, kao što to čini GUI.** `lattice capture`, `lattice continuous` (po jedan `.daq` po kadru), `array-capture` i `multi-capture` s povezanim nizom (i SDK) zapisuju `.daq` i njegov `.match.json` kad god je DAQ vezan ili objedinjen, uključujući snimanje samo u sirovom formatu; u verziji 1.2.0 sprema sirovu ili debayered snimku samo s `--force-daq`. `--no-daq` ga isključuje, `--daq-tolerance-ms MS` postavlja toleranciju podudaranja za tu snimku, a novi `daq pool-bind` povezuje kameru ili niz s DAQ-om iz CLI. Snimka koju `.daq` sprema podudara se u `process` samo s kamerama koje navede. Sada snimanje čeka sljedeće očitavanje DAQ-a kako bi moglo spojiti očitanja s obje strane ekspozicije. Pogledajte [The DAQ reading a capture saves](#the-daq-reading-a-capture-saves) i [A DAQ Reading Far From the Exposure](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** dodaje indeks projektu za pojaseve svake kamere, uključujući i novi projekt. U verziji 1.2.0 ime koje projekt još nije popisao ignoriralo se pomoću `Unknown index name(s) ignored`.
> - **`process` uz live niz:**više ne ponovno pokreće backend koji sadrži kamere ili DAQ-ove i ne čeka do 15 s na završetak izvršavanja.**Ctrl+C** zaustavlja izvršavanje u backendu, a ne samo u CLI-u. Vidi [Bilješke o `process`](#notes).
> - **Projektna mapa koja je preduboka za ograničenje putanje od 260 znakova sustava Windows** odbijena je odmah s najdužom mapom koja stane, umjesto pokretanja koje ne piše ništa. Instalacijski program za Windows sada nudi opciju "Omogući Windows dugih putanja" i izuzeće za Microsoft Defender za pozadinski program, a obje su opcije označene prema zadanim postavkama. Vidi [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** pokreće vlastito kontinuirano snimanje pozadinskog procesa: liniju napretka svakih 2 s, sažetak umjesto popisa po datotekama i ne-nulti izlazni kod za svaku nepopunjenu datoteku ili puni disk. Vidi [`array-capture` načine rada](#array-capture-modes).
> - **Prvi `array-capture` nakon `array-connect`** čeka do 20 s na prvu grupu sličica niza umjesto da zakaže s porukom HTTP 503 "Niz se još zagrijava".
> - **Ekspozicija niza unutar ROI-ja:** mali ROI više ne ograničava ekspoziciju člana niza na slobodni period senzora (4,9 ms pri 768 × 304). Pogledajte [Ekspozicija na članu niza](#exposure-on-an-array-member).
> - **Automatska ekspozicija više ne smanjuje automatsku brzinu snimanja nizom.** Niz sinkroniziran kabelom radi brzinom koju dopuštaju njegova mreža i očitavanje kamera; automatska ekspozicija popunjava razdoblje koje ta brzina daje, a zatim povećava pojačanje. Ručna ekspozicija dulja od razdoblja (`lattice exposure --set`, `configure --set`) usporava okidač kako bi se uklopila umjesto da bude odbijena, a brzina se vraća kada član pređe na automatski način rada. Vidi [Ekspozicija na članu niza](#exposure-on-an-array-member).
> - **Nizi isporučuju manje nepotpunih grupa sličica**, i manje 503 grešaka pri snimanju ("Nema nove sinkronizirane grupe sličica"): ulaz za okidanje svakog robnog kamere sada je filtriran.
> - **Komprimirani 12-bitni prijenos** (`--packed`, zadano `auto`) povećava brzinu snimanja kadrova za niz cijelog senzora za otprilike trećinu. Sada se primjenjuje `array-connect --packet-size` (u verziji 1.2.0 prihvaćena je, ali zanemarena). Vidi [Packed 12-bit transport](#packed-12-bit-transport---packed).
> - **Niz čije se kamere nakon isključenja/uključenja samostalno oporavljaju.** Nakon ciklusa isključenja/uključenja napajanja, niz `array-connect` se re-otvara kamere kada odgovore i ponovno streamuje s PTP-om uključenim; u verziji 1.2.0 to se zauvijek zaustavilo i bio je potreban novi `array-connect`. `array-status` izvještava `ptp_enabled: False` (i `ptp_off:`) dok kamera čita PTP off, gdje je prije izvještavala o postavci s kojom je niz bio povezan. Pogledajte [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` bilježi bez otvorenog GUI-ja.** Kada je kombinirani prikaz GUI-ja zatvoren, pozadinski sustav prikazuje sam prikaz: nizNDVI ili indeks koji proslijedite s novim `--expr`. Snimanje koje ne piše nijedan kadar sada ne uspijeva (`no-frames: …`, izlaz `1`). Prije je snimanje trajalo samo dok je kombinirani prikaz GUI-ja bio otvoren, a snimanje koje nije pisalo ništa i dalje je završavalo `0`. Vidi [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kamera koju već drži Chloros ne otvara se drugi put.** Zapovijed koja bi je sama otvorila umjesto toga odgovara HTTP 409 `CAMERA_HELD`, navodeći tko je drži i zapovijed koja je oslobađa. To uključuje i kameru koju trenutno koristi drugi nalog (nalog `multi-test` ili `multi-capture` koji otvara svoje kamere, nalog `lattice capture`): drugi nalog se odbija dok prvi ne završi. Ponovno otvaranje moglo bi zaustaviti stream nositelja ili srušiti pozadinski sustav. Vidi [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Privremeni nalozi za više kamera isporučuju svaku kameru ili ne uspijevaju.** `lattice multi-capture` bez priključene matrice, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` i `index --live` izlaze iz `1` navodeći kameru koja nije isporučila nijedan kadar. Streams preskače grupu kadrova ako kamera nedostaje i zaustavlja se nakon tri uzastopna; `multi-test` ispisuje redak po okidaču, `(no frame)` gdje je kamera to propustila; `align-stream` također izlazi `1` na pogrešci posla. Svaki izvršni postupak zatim vraća svaku kameru u stanje u kojem ju je zatekao: njezino ožičenje okidača i sinkronizacijske linije, njezin PTP (`PtpEnable`) i svaku postavku kamere koju je pokretanje promijenilo — ekspoziciju i pojačanje, AE cilj, granice i prigušenje, balans bijele, razinu crne, format piksela, biniranje i ROI, preokrete, brzinu sličica — osim ako `--keep-settings` / `--clear-settings` (vidi [Memorija kamere](#camera-memory----keep-settings----clear-settings)). Prije su ove naredbe zadržavale sve primljene sličice i izlazile iz `0` (`align-stream` je ispisivao pogrešku posla kao `Note:`), i ostavljale svaku kameru u `TriggerMode` `On`. Također su postavile ekspoziciju svake kamere prema sceni prije zadržavanja kadra (najviše nekoliko sekundi), na ciljnu površinu na kojoj povezan niz pokreće svaku kameru: 40 multispektralnih, 60 RGB (postavlja ga `--brightness N`), i zadržava ga tijekom rada; tako i jedan `lattice capture` kamere Chloros ne drži. Prije je prvi kadar svake kamere sniman na vlastitom ograničenju ekspozicije kamere, oko 100 ms, bez obzira na svjetlo, i često je bio odsječen. Vidi [Više kamera (privremeni sesije)](#multi-camera-transient-sessions).
> - **Privremeni naredbe za više kamera dijele mrežu na način na koji to radi `array-connect` i otvaraju samo `--serials`.** Svaka kamera dobiva svoj udio u host vezi, pa četiri kamere rade na 1 Gbps vezi; prije su četiri kamere tražile 125 MB/s među sobom, sve na 1 GbE, i svaki je kadar gubio pakete. Niz koji mreža ne može prenijeti odbijen je prije nego što se ikakvi streamovi kamera pokrenu. `--serials` sada odabire kamere, a okidač aktivira prva kamera; prije je svaka naredba otvarala sve kamere na mreži bez obzira na to što je `--serials` naredio. Greška koja je prije bila označena kao gubitak paketa GVSP-a sada to i navodi umjesto da upućuje na M8 sinkronizacijski kabel. Pogledajte [Više kamera (privremeni sesije)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` ostavlja kamere da rade neovisno, kako su i zatečene.** `TriggerMode` `Off`, i sve ostalo kako je connect to pronašao — postavke okidača i sinkronizacijske linije (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP i postavke kamere koje je sesija promijenila — pa jedan-camera naredba na prethodnom članu dobiva kadrove odmah, a master više nije ostavljen na `TriggerSource` `Software`. `--keep-settings` sprema postavke sesije u memoriju svake kamere, `--clear-settings` resetira tu memoriju (vidi [Memorija kamere](#camera-memory----keep-settings----clear-settings)). `lattice continuous` također postavlja kameru koju pronađe naoružanu da bude u načinu slobodnog rada za izvođenje (i vraća je nakon toga), a izlaz iz izvođenja bez okvira je `1`.
> - **`lattice configure --get` / `lattice exposure`** ispisuju `note:` s kameromnje stvarnu ekspoziciju ili pojačanje dok je u automatskom načinu rada. Odbijeni `lattice exposure --set` / `configure --set` navode raspon koji kamera prihvaća i vraćaju automatski način rada umjesto da isključe automatsku ekspoziciju. Vidi [Podešavanje senzora](#sensor-tuning).
> - **Nove kontrole kamere:**`lattice highlights` (podiže svijetle bijele površine u prikazu RGB) i `lattice exposure --brightness-auto`.**`lattice multi-detect`** pronalazi kamere s kabelom u jednom krugu sonde i zadano postavlja glavnu kameru na kameru u boji.
> - **Nove DAQ naredbe:** `daq pool-bind`; `pool-enable-ptp`; a za vlastite postavke DAQ-E uređaja `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` i `pool-update-firmware`. Pogledajte [Postavke uređaja DAQ-E](#daq-e-device-settings).
> - **DAQ-E koji prestane slati podatke ponovno povezuje backend.** Kada stream pooliranog DAQ-E-a utihne (zbog isključenja i ponovnog uključivanja prekidača koji ga napaja, prekinuta veza kamere), pozadinski sustav ga ponovno povezuje s prethodnim postavkama, pokušavajući ponovno u sve dužim intervalima dok je nedostupan. Snimanje na njemu se nastavlja u istom `.daq`, a u datoteci se bilježi praznina, a `pool-record --stop` navodi svaki prekid. U verziji 1.2.0 senzor je ostao tih do `pool-disconnect` + `pool-connect`, što je također prekinulo snimanje. Vidi [A DAQ-E That Stops Delivering](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** izvještava o tome dosežu li paketi PTP grandmajstora mrežu (`health`), pri čemu se `WARNING:` ispisuje na stderr kada ne. Vidi [`time-sync`](#chloros-cli-time-sync).
> - **Linux .deb:** povećava kernelove soketne prijemne buffer-e koji su potrebni za video streamove s kamere (pri zadanoj vrijednosti kernela gube se nizovi okvira i brzina snimanja), dodaje pozadinsku uslugu koja se pokreće kao vaš korisnik, a `lattice network` provjerava buffer-e. CLI nikada ne zaustavlja pozadinsku uslugu (backend) u kojoj se usluga pokreće, a nadogradnja ponovno pokreće omogućene pozadinske usluge. Pogledajte [Pokretanje pozadinske usluge (backenda) kao usluge](#running-the-backend-as-a-service-headless) i [Linux hostovi — bufferi za primanje soketa](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` navodi kada se nije moglo poravnati:**svaki razlog zbog kojeg je snimanje spremljeno neporavnato (nema profila poravnanja, članovi na različitim rezolucijama, niz koji nije sinkroniziran kabelom, okvir na koji se profil ne može mapirati) ispisuje se kao `Alignment:` linija.**`chloros-cli daq help`** je `daq --help` (izašao je iz `2`), a na Linux pomoć navodi naredbe za izravno korištenje hardvera `daq` koji build uključuje.
> - **Jetson:** paket je `chloros_1.3.0_arm64_jp6.deb`, za JetPack 6, a njegova obrada na GPU-u radi na Orinu s JetPackovim CUDA PyTorchom. Vidi [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvencije

- Sve naredbe imaju prefiks `chloros-cli`. Na Windows binarna datoteka je `chloros-cli.exe`; na Linux/Na Jetsonu je `chloros-cli`.
- Izborni argumenti prikazani su kao `--flag`. Obavezni pozicijski argumenti prikazani su bez zagrada.
- Ako je naveden zadani parametar, izostavljanjem zastavice koristi se ta vrijednost.
- CLI je lagani HTTP klijent koji koristi Chloros pozadinu (poslužitelj na `127.0.0.1:5000` koji također koristi i desktop aplikacija). **Koje naredbe ga pokreću:**
  - Osnovne naredbe — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` i `selftest` — koriste pokrenuti backend ili ga pokreću kad nijedan ne radi i zaustavljaju ga kad završe.
  - **zapovijedi `lattice`, `project` i `daq` nikada ne pokreću backend.** Potrebna im je pozadinska usluga koja već radi: desktop aplikacija sustava Windows, [pozadinska usluga sustava Linux](#running-the-backend-as-a-service-headless) ili ona koju pokrenete sami (pogledajte [Pokretanje pozadinske usluge](#start-the-backend)). Bez njega ispisuju `Error: Chloros backend not reachable at http://127.0.0.1:5000`, zajedno s naredbom za njegovo pokretanje, i izlaze `1`.
- `CHLOROS_BACKEND_URL=<url>` usmjerava **`lattice`**,**`project`**i**`daq pool-*`** obitelji naredbi na drugom pozadinskom sustavu (`lattice network-analysis` umjesto toga koristi `--backend-url`). Osnovne naredbe namjerno pinaju `http://127.0.0.1:<port>` i ignoriraju ga (doslovni IPv4 izbjegava kaznu od ~2 s-po-zahtjevu sustava Windowsza `localhost`→`::1`). Pogledajte [Promjenjive okruženja](#environment-variables).
- **Relativne putanje rješavaju se u odnosu na mapu iz koje pokrećete `chloros-cli`.** Backend je proces koji otvara izlazne i ulazne mape, stoga prije slanja putanje CLI je pretvara u apsolutnu: `-o` / `--output` (uključujući zadani `output`), `--input`, `--burst-dir`, `--profile`, `--save` i `--mask`, a [recipe](#recipe-dsl) koraka `output` (zadani `output`), i zadani `output` direktorij `lattice index --live`. `-o captures/` stoga piše u `<current folder>/captures/`, a ispisane staze su apsolutne. Chloros+ se proširuje. Na udaljenom backendu (Chloros+ naziv drugog računala) putanje se šalju onako kako su napisane i rješavaju na tom računalu.
- Za pristup svim SDK/CLI poziva (pokrenite `chloros-cli login` jednom po računalu; pohranjeno u `~/.chloros/`).
- Primjeri koriste putanje za Linux; na Windows zamijenite `/home/user/...` s `C:/Users/.../...`. DAQ-Userijski priključak je u stilu `COM3` na Windows i `/dev/ttyUSB0` / `/dev/ttyACM0` na Linux.
- Sinopse zapisuju međusobno isključive zastavice kao `{--a | --b}` i opcionalne u `[...]`.

---

## Sinopsis najviše razine

```
chloros-cli [global options] COMMAND [command options]
```

### Globalne opcije

| Zastavica | Opis |
| --- | --- |
| `--backend-exe PATH` | Prepiši automatski otkrivenu izvršnu datoteku pozadinskog sustava. |
| `--port N` | Port pozadinskog sustava HTTP (zadano: `5000`). Koriste ga samo osnovne naredbe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` i `daq` koriste `CHLOROS_BACKEND_URL` (default `http://127.0.0.1:5000`). |
| `-v, --verbose` | Omogući detaljan izlaz. |
| `--restart` | Prisilno ponovno pokrenite pozadinski proces (ukida sve aktivne `backend_server.py`). Pozadinski proces kojeg pokreće systemd usluga (Linux .deb) nikada se ne zaustavlja: naredba umjesto toga ispisuje `sudo systemctl restart <unit>` i izlazi `1`. |
| `--version` | Ispiši verziju (`Chloros CLI 1.3.0`). |
| `--help` | Prikaži pomoć visoke razine. |

### Indeks naredbi

| Naredba | Svrha |
| --- | --- |
| [`process`](#chloros-cli-process) | Obradi mapu snimki Survey3 ili LATTICE od početka do kraja. |
| [`login`](#chloros-cli-login) | Autentificirajte ovaj uređaj s računom Chloros+. |
| [`logout`](#chloros-cli-logout) | Ispraznite pohranjene vjerodajnice. |
| [`status`](#chloros-cli-status) | Prikaži trenutačni status licence / autentifikacije. |
| [`export-status`](#chloros-cli-export-status) | Prikaz napretka izvoza uživo tijekom izvođenja `process`. |
| [`language`](#chloros-cli-language) | Postavite ili popišite jezik prikaza CLIa (38 podržanih). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Zadana mapa projekta (zajednička s GUI-jem). |
| [`update`](#chloros-cli-update) | Provjera i instalacija ažuriranja za Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Dijagnostika sustava + smoke testovi. |
| [`time-sync`](#chloros-cli-time-sync) | PTP grandmaster status / kontrola. |
| [`lattice`](#chloros-cli-lattice) | Kontrola i snimanje LATTICE kamere (45+ podzapovijedi). |
| [`daq`](#chloros-cli-daq) | Kontrola DAQ spektralnih senzora (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otvorite i pokrenite spremljeni projekt za Chloros (kamere + DAQ-ovi). |

---

## Instalacija

`chloros-cli` dolazi u paketu s desktop instalacijskim programom za Chloros na svakoj podržanoj platformi — nema zasebnog preuzimanja za CLI. Instalacija paketa za platformu dodaje `chloros-cli` vašem `PATH` uz desktop aplikaciju i pozadinski binarni datotek koji ona pokreće.

Najnovija preuzimanja: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Instalacijski program također uključuje praktične skripte za pokretanje (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) koji otvaraju spremnu za upotrebu CLI školjku.

### Windows (.exe)

1. Preuzmite instalacijski program za Windows s stranice za preuzimanje.
2. Pokrenite `Chloros Installer 1.3.0.exe` i slijedite čarobnjak. Zadana putanja instalacije je `C:\Program Files\Chloros\` (CLI se nalazi u `C:\Program Files\Chloros\cli\`, koju instalator dodaje u PATH).
3. Otvorite novi terminal (Windows Terminal) kako bi ažurirani `PATH` bio prepoznat.

```powershell
chloros-cli --version
```

Instalacijski program automatski dodaje `chloros-cli.exe` u vaš sustav `PATH` i uključuje upravljačke programe za kamere LATTICE. Pozadinski program je `C:\Program Files\Chloros\backend\chloros-backend.exe` (mapu `backend` u vašoj mapi za instalaciju), koja se ne nalazi na `PATH`.

**Postavke sustava.** Na stranici instalatera *Odabir dodatnih zadataka* nalaze se dvije opcije pod **Postavke sustava**, obje**zadano označene**:

| Opcija (kako je navedeno u instalateru) | Što radi | Odustajanje od tihe instalacije |
| --- | --- | --- |
| **Omogući dugačke putanje sustava Windows** (omogućuje da se mape projekta nalaze dublje od 260 znakova; preporučuje se) | Postavlja `LongPathsEnabled` na razini cijelog sustava Windows na 1. Windows dolazi s njim isključenim, i tada nijedan put ne smije prijeći 260 znakova; proizvodi tvrtke Chloros smješteni su do otprilike 130 znakova ispod mape projekta, stoga se dublja mapa projekta odbacuje (vidi [Rješavanje problema])(#troubleshooting-pointers)). Stupa na snagu za programe pokrenute nakon instalacije. Ostaje uključeno pri deinstalaciji jer se drugi programi mogu oslanjati na to. | `/MERGETASKS="!longpaths"` |
| **Isključite pozadinski program Chloros iz skeniranja programa Microsoft Defender**(znatno brže snimanje s kamerom; preporučuje se) | Dodaje iznimku za proces Defender**process** za `chloros-backend.exe` u mapi instalacije, tako da zaštita u stvarnom vremenu ne skenira svaku datoteku koju pozadinski program zapisa, gdje god je zapisao. Nijedan drugi program nije izuzet; izuzeće se uklanja prilikom deinstalacije. Ako Defender nedostaje ili je zaključan politikom, taj se korak tiho zanemaruje i instalacija se nastavlja. | `/MERGETASKS="!defenderexclusion"` |

**Zašto je opcija Defender važna:** pri visokim brzinama snimanja, skeniranje datoteka u stvarnom vremenu košta oko 30 % brzine snimanja (izmjereno na kontinuiranom snimanju 9 kamera: isključeno 77,7 snimanja/s, 52,7 skenirano). `lattice array-capture --continuous` ispisuje bilješku kada izmjeri da Defender koristi jedno ili više jezgri CPU-a za snimanje; ponovno pokrenite instalacijski program s označenom opcijom.

Tiha instalacija (`/SILENT` ili `/VERYSILENT`) primjenjuje se na obje osim ako se ne odustane, npr. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windows vatrozid.** Instalacijski program uvijek dodaje dva pravila dolaznog prometa za `chloros-backend.exe`, na svaki mrežni profil, a deinstalacijski program ih uklanja: `Chloros Backend (UDP-In)`, za video streamove kamere, PTP, mDNS i DAQ-E stream podataka, te `Chloros Backend (TCP-In)`, ograničen na lokalnu podmrežu, za ažuriranje firmvera DAQ-E, u kojem se senzor ponovno povezuje s ovim računalom kako bi preuzeo sliku (vidi [Postavke uređaja DAQ-E](#daq-e-device-settings)). Backendov HTTP-API ostaje povezan s `127.0.0.1`.

### Linux amd64 (.deb)

Za Ubuntu 22.04 LTS ili novije / x86_64 radne stanice temeljene na Debianu.

> **Ubuntu 20.04 nije podržan.** Popis ovisnosti paketa temelji se na onome s čime se pozadinski program zapravo povezuje, a to uključuje `libc6 (>= 2.35)`; focal isporučuje glibc 2.31. `apt` odbijaju instalaciju umjesto da dopuste da dođe do pogreške tijekom
> izvođenja. Popis također navodi `libgl1`, koji učitava priloženi OpenCV, a minimalnoj
> poslužiteljskoj sliki može nedostajati; `apt install` ga donosi.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

.deb instalacije:
- `chloros-cli` u `/usr/bin/chloros-cli` i `chloros-backend` (pokreće backend) u `/usr/bin/chloros-backend`
- Kompajlirani backend i CLI, raspakirani, u `/usr/lib/chloros/backend/` i `/usr/lib/chloros/cli/`
- Vozači za kamere LATTICE
- Modeli za uklanjanje šuma, paketi za kalibraciju i konfiguracija kanala ažuriranja
- Pythonov SDK za sustav `python3` (`import chloros_sdk`). `apt install` također donosi svoje preduvjete, `python3-numpy` i `python3-requests` (preporučeno, ne obavezno: gore navedeni programi su samostalni; `dpkg -i` ih preskače, a izlaz instalacije navodi što nedostaje). U virtualenvu: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, koji povećava kernelove soketne spremnike za primanje za kameru povećava ih na 32 MB i primjenjuje odmah (vrijednost koja je već postavljena na višu vrijednost ostaje nepromijenjena). Uklanjanje paketa uklanja datoteku. Pogledajte [Linux hosts — socket receive buffers](#linux-hosts--socket-receive-buffers).
- Dvije opcionalne systemd usluge za pozadinski sustav, nijedna nije omogućena (sljedeći odjeljak)

#### Pokretanje pozadinskog sustava kao usluge (bez zaslona)

Bez usluge, pozadinski sustav (i s njim PTP grandmaster) radi samo dok ga nešto nije pokrenulo: `chloros-backend` u terminalu, skripta iz SDKa ili osnovna `chloros-cli` naredba tijekom trajanja njegova izvođenja. Za stalno aktivan backend na bezmonitornoj (headless) mašini, omogućite uslugu **kao svoj korisnik**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` pokreće backend kao taj korisnik. Snimke koje on piše njihove su, njegov detaljni zapisnik nalazi se u njihovom `~/.cache/chloros/logs`, i koristi njihov `chloros-cli login` (`~/.chloros`), stoga se prvo jednom prijavite kao taj korisnik. PTP-u nije potreban root. Kao i svaki instalirani backend, on sluša samo na `127.0.0.1:5000`.

`chloros-backend.service` pokreće backend kao **root**. I dalje je instaliran tako da računalo na kojem je omogućen nastavlja raditi, ali sve što root backend zaprimi pripada rootu, pa korisnik ne može izbrisati vlastite snimke. Za prebacivanje na svoj korisnički račun:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Detaljni zapisnik usluge root nalazi se u `/var/log/chloros`, a mogu ga čitati članovi grupe `adm` (usluga postavlja `CHLOROS_LOG_DIR`; pogledajte [Promjenjive okruženja](#environment-variables)). Instalacija paketa ispisuje naredbu `enable` s popunjenim korisničkim imenom, kao i naredbu za prebacivanje kada je usluga root omogućena.

**Omogućite jednu jedinicu backenda.** U isto vrijeme radi samo jedna pozadinska jedinica. Druga omogućena jedinica (`chloros-backend.service` uz `chloros-backend@<user>.service`, ili dvije korisničke jedinice `@`) odbija se pokrenuti i ostaje `failed` sa statusom izlaska 98; systemd ga ne ponovno pokreće. Njegov `systemctl status` navodi zašto: `Another instance is already running. Exiting.`, zatim što blokira pozadinsku uslugu (port 5000, ili datoteka zaključavanja i korisnik kojem pripada), zatim `systemd will not restart <unit> (exit status 98)`. Nakon ponovnog pokretanja, bilo koja jedinica može biti ona koja se pokrene. Za prebacivanje, onemogućite onu koju ne koristite, kao što je gore opisano.**CLI nikada ne zaustavlja backend usluge.** `chloros-cli` ga koristi onakvim kakav jest, kao što to čine GUI i SDK: `process` se na njemu pokreće, učitavajući vlastiti projekt umjesto posljednjeg. Gdje bi CLI inače ponovno pokrenuo backend (`--restart`, ili `process` čije uvoženje nije pronašlo ništa), naziva jedinicu, ispisuje `sudo systemctl restart <unit>` i izlazi `1`, jer bi backend koji se tako zaustavio ostao neaktivan. `GET /api/status` imenuje jedinicu kao `service` (`null` za backend na kojem se ne pokreće nijedna usluga).**Ažuriranje i uklanjanje.** Nadogradnja paketa zaustavlja svaku pozadinsku uslugu, a zatim ponovno pokreće svaku koja je omogućena, prvo jedinice `@<user>` (jedinica koju je ručno pokrenuto, ali nikada omogućeno, ostaje zaustavljena, kao i nakon ponovnog pokretanja). Kada je omogućeno više od jednog, radi samo jedan, a ostali ostaju neuspjeli kao gore; nadogradnja ispisuje `Chloros: N backend units are enabled: …` i `Keep one enabled: sudo systemctl disable --now <unit>`. Njegovo uklanjanje zaustavlja i onemogućuje ih. Njegovo čišćenje također briše `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ista struktura kao i amd64 .deb, uključujući primarne međuspremnike i [backend usluge](#running-the-backend-as-a-service-headless), za Jetson Orin / Orin NX / Orin Nano na **JetPacku 6** (L4T R36). Njegov PyTorch je CUDA izrada za JetPack 6 za Orin, pa se obrada na GPU-u, uključujući i debayer osjetljiv na teksture, odvija na GPU-u Jetsona. Paket ne sadrži samu CUDA biblioteku: koristi biblioteke CUDA i cuDNN koje instalira JetPack 6 (`sudo apt install nvidia-jetpack` na slici na kojoj one nedostaju). Gdje se ne može koristiti CUDA GPU, obrada se odvija na CPU-u, a obrada osjetljiva na teksture koristi standardni debayer (vidi [`process`](#chloros-cli-process)).

### Autentificirajte se jednom po računalu

Svaka platforma zahtijeva jednokratno prijavljivanje na Chloros+ prije nego što pozivi SDK/CLI-a rade:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Vjerodajnice se keširaju u `~/.chloros/user_session.json`.

### Pokrenite pozadinski sustav

Naredbe `lattice`, `project` i `daq` komuniciraju s pozadinskim sustavom koji je već pokrenut; one ga nikada ne pokreću (vidi [Konvencije](#conventions)). Držite ga pokrenutim dok ga koristite:

| Platforma | Kako |
| --- | --- |
| Windows | Otvorite desktop aplikaciju Chloros (njezin backend također služi i za CLI), ili pokrenite `"C:\Program Files\Chloros\backend\chloros-backend.exe"` u zasebnom terminalu (mapi `backend` vaše instalacijske mape) i ostavite ga otvorenim. |
| Linux / Jetson | Pokrenite `chloros-backend` u vlastitom terminalu ili neku verziju uvijek držite pokrenutom pomoću [backend usluge](#running-the-backend-as-a-service-headless). |

Istovremeno radi samo jedan backend; svaki klijent (GUI, CLI, skripte SDK) dijeli ga. Osnovne naredbe (`process`, `login`, `status`, …) također ga koriste i pokreću jedan kada nijedan ne radi.

**Backend nastavlja raditi između naredbi**, pa sljedeći ne plaća ponovno svoje pokretanje. Zaustavlja se kada se odjavite (`chloros-cli logout`, osim ako je aplikacija Chloros otvorena ili je u tijeku izvršavanje ili izvanmrežna izrada), kada pokrenete `chloros-cli backend stop` ili samostalno nakon 30 minuta ako ga nitko ne koristi: nema naredbe, skripte ili aplikacije, nema pokretanja ili izvanmrežne izgradnje, nema priključene kamere ili senzora svjetla. `backend stop` odbija dok je pokretanje ili izvanmrežna izgradnja u tijeku, nazivajući je; `--force` je sada zaustavlja i to pokretanje ili izgradnja je izgubljena (serija se zadržava i `array-build-video` je ponovno gradi). Postavite `CHLOROS_BACKEND_IDLE_MIN` prije nego što pozadinska usluga počne mijenjati 30 minuta (`0`: nikada); [pozadinska usluga](#running-the-backend-as-a-service-headless) nikada se ne zaustavlja sam od sebe. `chloros-cli backend status` prikazuje je li nešto u tijeku, što ga koristi i bilo koji offline build koji se pokreće. `process` otvara svoj projekt u pokretnom backendu, osim ako aplikacija Chloros nije otvorena s projektom koji je otvoren: tada odustaje, a vi prvo završite proces iz aplikacije ili zatvorite taj projekt. Projekt koji je raniji `process` ili skripta SDK ostavila otvorenim jednostavno se zamjenjuje.

### Provjera instalacije

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` završava `0` samo kada svih sedam provjera prođe. Posljednja zahtijeva CUDA GPU (NVIDIA GPU, ili Jetson Orin) i modele za uklanjanje šuma, pa na računalu bez njega prikazuje `Failed: CUDA + Denoiser` i izlazi iz `1`; sve osim debayera osjetljivog na teksture koji radi samo na GPU-u i dalje radi na njemu.

> **Potrebna je pretplata na Chloros+.**CLI zahtijeva aktivan plan Chloros+.**Copper**je početni nivo pretplate Chloros+ — svaki plaćeni nivo pretplate Chloros+ ima pristup CLI/SDK-u; samo besplatni nivo**Iron** nema. (Mapa ID-ova planova: `0`=Iron/besplatno, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Nadogradnju zatražite na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Ovo ograničenje provodi pozadinski sustav (backend), a ne samo CLI: zahtjev s oznakom SDK/CLI bez plaćenog plana odbijen je kodom `403 PLAN_UPGRADE_REQUIRED`, bez obzira dolazi li iz `chloros-cli`, Python SDK-a ili ručno izrađenog HTTP klijenta. Umjesto toga, korisnik koji je odjavljен dobiva `401 AUTH_REQUIRED`. Pristup radi izvan mreže tijekom razdoblja milosti plana (30 dana za mjesečne planove, do isteka za godišnje). i prestaje s radom kada taj period istekne; `chloros-cli status` nastavlja raditi kako bi razlog bio vidljiv (to je jedina ruta SDK/CLI koja je izuzeta od ograničenja po razini — `GET /api/license-status`).

---

## Brzi početak u pet minuta

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (korak 2) pokreće pozadinsku uslugu za vlastito pokretanje kada nijedna ne radi i zaustavlja je kada završi, stoga pokrenite jednu za korak 4 i nadalje. Pogledajte [Pokrenite pozadinsku uslugu](#start-the-backend).

---

## `chloros-cli process`

Obradite mapu s slikama kroz cijeli Chlorosov pipeline (detekcija ciljeva → kalibracija → vinjeta → reflektancija → izvoz indeksa).

### Sinopsis

```
chloros-cli process INPUT [OPTIONS]
```

### Pozicijski argumenti

| Argument | Opis |
| --- | --- |
| `INPUT` | Putanja do ulazne mape koja sadrži datoteke `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ili `.dng`. |

### Zajedničke opcije

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `-o, --output PATH` | nova mapa nazvana pomoću `-n` u zadanoj mapi projekta | Mapa projekta. Projekt se naziva po njemu, a mapa koja već sadrži `project.json` dobiva `_1` / `_2` se stvara kao sestrinska mapa umjesto da bude prebrisana. Zadana mapa projekta je ona koju je odabrao `set-project-folder` (ili GUI), inače `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automatski (vremenski žig) | Naziv projekta, koristi se samo **bez** `-o` (s `-o` projekt preuzima naziv mape). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` koristi neuronski debayer Chloros+ za snimke kamere Survey3; sporije, ali kvalitetnije. Namijenjen je samo za Survey3: LATTICE snimke uvijek koriste standardni demosaicing, stoga za mapu koja sadrži LATTICE snimke `process` odbijaju `texture-aware` nakon uvoza, uz pogrešku koja navodi koliko ih je, i izlaze `1` prije obrade bilo čega (Chloros aplikacija za njih također ne nudi Texture Aware). Potrebna mu je CUDA GPU (NVIDIA GPU na Windows/Linux x64, Orinova GPU na Jetsonu): bez njega izvršavanje koristi `standard`, a pozadinski sustav bilježi jednu `[DEBAYER] … needs a CUDA GPU …` liniju koja objašnjava zašto (prikazano s `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Korekcija vinjete. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibracija reflektancije (koristi panelni cilj ako se pronađe, NIST kalibracija po serijskom broju za LATTICE). Za multispektralni LATTICE ovo služi i kao **proizvod** reflektancije preklop — vidi [Per-Product Export Toggles](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | isključeno | Primijeni PPK GNSS korekcije iz pomoćnih datoteka. |
| `--exposure-pin-1 CAMERA_MODEL` | nijedan | Model kamere povezan s pinom za ekspoziciju 1, za PPK vremensku sinkronizaciju. |
| `--exposure-pin-2 CAMERA_MODEL` | none | Model kamere priključen na pin za ekspoziciju 2, za PPK vremensku sinkronizaciju. |
| `--recal-interval SECONDS` | `0` | Minimalni interval ponovne kalibracije, u sekundama (*Minimalni interval ponovne kalibracije* projekta). |
| `--timezone-offset HOURS` | `0` | Pomak vremenske zone senzora svjetla od UTC-a, u satima (*Pomak vremenske zone senzora svjetla* projekta). |
| `--format FORMAT` | `TIFF (16-bit)` | Jedan od `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros nikada ne povećava dubinu bita slike: na svim 8-bitnim slikama dublji se format zapisuje kao `TIFF (8-bit)` i program to jednom navede. U projektu koji miješa 8-bitne i 12/16-bitne snimke, svaka se snimka izvozi u svojoj dubini ili nižoj. Radiance je uvijek 32-bitni float. |
| `--indices NAME [NAME ...]` | nijedan | Indeksi vegetacije po nazivu preseta (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), bez osjetljivosti na velika i mala slova. Svaki se dodaje u projekt za pojaseve svoje kamere, uključujući novi projekt. Naziv koji nije unaprijed postavljena postavka ili koji pojasevi kamere ne mogu stvoriti (`NDRE` treba RE i NIR, što nijedna pojedinačna kamera ne pruža), naznačuje se u upozorenju i ne dobiva indeksnu sliku. |
|
`--input-level {auto,raw,debayered,processed}` | `auto` | Prisilno postavlja ulaznu točku za LATTICE TIFF-ove (Survey3.raw ostaje nepromijenjen). Također, izlazna opcija koja omogućuje obradu snimanja bez **raw** formata — vidi [Kako izgleda mapa sa snimkama](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | uključeno | Ispisuje linearni debayered proizvod (`Debayered_Images`). Pogledajte [Prekidače za izvoz po proizvodu](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | uključeno | Izlaz pretpregleda prikaza (`Preview_Images`): RGB = bijela točka (DAQ-illuminant kad je dostupan, inače svijetlosivi) + gamma; multispec = istezanje lažnih boja. |
| `--radiance / --no-radiance` | uključeno | Izlazak radijance float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referenca za LATTICE proizvod reflektancije: `auto` = apsolutna referenca je cilj u kadru koji je prošao QA (cilj s QA zastavicom natječe se s DAQ-om po mjerenoj ocjeni), DAQ-downwelling (ρ = π·L/E) kao rezervna opcija; `target` = strogo (bez zamjene DAQ-a); `daq` = autoritativno za DAQ. Vidi [Prekidače za izvoz po proizvodu](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | none | Direktorij skeniranja **izmjerenih** ciljanih reflektancija po jedinici (`<serial>.csv`); pri promašaju se vraća na nominalna T3/T4P spektra. |
| `--array-alignment / --no-array-alignment` | uključeno | LATTICE nizovi: primijeniti poravnanje modula prema modulu zabilježeno u XMP-u svake snimke (`Chloros:Alignment*`) na svaki obrađeni proizvod (debayered / pretpregled / radijanca / refleksija / indeks). Bez učinka za slike bez oznaka. |
| `--array-alignment-crop / --no-array-alignment-crop` | obrezivanje | Izvoz poravnatih podataka u zajedničku preklopljenu regiju niza kako bi svi moduli dijelili jedan otisak; `--no-…` zadržava cijelo platno senzora (crno popunjavanje izvan izvora). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Ponovno uzorkovanje za deformaciju poravnanja. `nearest` čuva točne DN-ove izvora (bez miješanja radiometrijskih vrijednosti između piksela). |

### Opcije za detekciju cilja

| Zastavica | Opis |
| --- | --- |
| `--min-target-size PIXELS` | Minimalna veličina cilja kalibracije u pikselima za detektor. |
| `--target-clustering 0-100` | Postotak grupiranja ciljeva (zadano `60`). |
| `--target / --targets` | Pretraživanje kalibracijskih ciljeva samo u podmapu `target` ili `targets` unosa (brže). |

### Primjeri

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Prekidači za izvoz po proizvodu (LATTICE multispectral)

Obrada u LATTICE-u širi se na **sve primjenjive proizvode u jednom koraku**. Četiri prekidača po vrsti — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sve su**po zadanome UKLJUČENE**; upotrijebite obrazac `--no-<type>` za isključivanje jedne. RGB glavne kamere uvijek emitiraju samo debayeri­zani + pregled (bez radijance/reflektance po pojasu), stoga su `--radiance`/`--reflectance` za njih no-op-ovi. Prekidači se zanemaruju za Survey3 `.raw` (koji slijedi standardnu putanju odbijanja/cilja). *(Stari `--radiometric-output {reflectance,radiance,sensor-response}` zastavica je **uklonjena** i zamijenjena ovim prekidačima; više ne postoji razina `sensor-response`.)*

| Proizvod | Izlaz | Je li potreban DAQ downwelling? |
| --- | --- | --- |
| `--debayered` | Linearna demosaikizacija (`Debayered_Images`): ista datoteka koju snimanje sprema na svojoj razini `debayered` — kalibracijski paketov dark offset (DSNU) s sirovog kadra, zatim OpenCV-ov demosaic osjetljiv na rubove; nema linearnosti LUT-a, flat field ili unmixa. | Ne. |
| `--preview` | Prikaz pretpregleda (`Preview_Images`): RGB = WB + gamma; multispec = istezanje lažnih boja. | Ne. |
| `--radiance` | float32 W/m²/sr/nm iz cjelovitog radiometrijskog lanca (`Radiance_Images`). | Ne. |
| `--reflectance` | uint16 reflektancija ρ (`32768` = 1.0), Spremno za Pix4D. | **Da**, osim ako ga ne učvrsti cilj unutar kadra koji je prošao QA (pogledajte dolje). |

`--reflectance-source` odabire referencu reflektancije:**`auto`**(zadano) čini cilj unutar kadra koji je prošao QA apsolutnom referencom — lanci empirijskih linija vezanih za metu ocjenjuju se unakrsno na zadržanim panelima i primijenjen je izmjereni pobjednik, pojas po pojas: pojas koji DAQ ne može kalibrirati ili čije prilagođavanje nagibu DAQ-a ne uspije, uzima liniju same mete (`ReflectanceElBandMethods` određuje koji) — cilj koji prođe QA s oznakom (ispušten panel, pojas izvan redoslijeda) također se natječe s DAQ podjelom na toj izmjerenoj ocjeni, a kadar kojem izgubi označava se `ReflectanceChain=daq` — vraća se na DAQ-ovu podjelu prema silaznom toku (ρ = π·L/E) kada cilj nije prisutan ili QA ne uspije;**`target`**je stroga (bez zamjene DAQ-om);**`daq`**prelazi na DAQ-ov ovlašteni način rada. Geometrija cilja (ArUco / fiksirani ROI / traka) dolazi iz konfiguracije cilja projekta; `--target-reflectance-dir DIR` pohranjuje**izmjerenu** skeniranja (`<serial>.csv`) pretražuju se prema serijskom/QR kodu ciljne jedinice, a nominalna T3/T4P spektra služe kao rezervna opcija.

DAQ putanja reflektancije automatski rješava **nadole usmjerenu emisiju usklađenu s vremenskim žigom**iz snimljenog**`.daq`**(DAQ-U/M/E)**ili nativni `.csv` DAQ-M-a**pronađen uz snimke. Ako paket kalibracije po kameri ili DAQ-u nije lokalno pohranjen, pipeline ga**automatski preuzima s AWS-a** pri prvom korištenju (potrebna internetska veza jednom; pohranjeno pod `~/.chloros/`).**Čije je očitanje.**Očitanje koje `.daq` sprema pri snimanju je očitanje kamere(a) koju opisuje — imenovano u popratnoj datoteci `<file>.daq.cameras.json` pored njega (`lattice capture`, `multi-capture`, `array-capture` i GUI-ju jednako; sirovi burst imenuje očitanje svakog člana u svom manifestu) — i `process` ga usklađuje**samo s slikama iz tih kamera**. Samostalna kamera bez vlastitog očitavanja stoga više ne posuđuje DAQ druge kamere; povežite DAQ s njom ili snimite jedan. Vlastito snimanje DAQ-a (`daq pool-record`, ili snimanje započeto u GUI-ju), ili bilo koji `.daq` bez te pomoćne datoteke (uključujući starije snimke), i dalje služi svakoj kameri.

#### Čitanje reflektancijskih piksela (Pix4D / Metashape / vlastiti skripti)

Reflektancija se pohranjuje kao cjelobrojni DN, a **DN koji označava ρ = 1,0 ovisi o izvornoj kameri**:

| Izvor | ρ = 1,0 je | Kako provjeriti |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (dopušteno do ρ 2.0) | XMP `Chloros:PixelScale=32768` je utisnut na datoteku. |
| Survey3 | `65535` (odrezano na ρ 1.0) | Nema XMP oznaka `Chloros:*` — taj izostanak *je* signal. |

**Pročitajte `Chloros:PixelScale` i podijelite s njim** umjesto da pretpostavljate konstantu. Oznaka je definirana u domeni uint16, pa ostaje `32768` u svim formatima izlaza koji skaliraju — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` i `TIFF (32-bit, Percent)` su svi sami po sebi opisni (prvo normalizirajte pohranjenu dtype natrag na uint16: ×257 iz 8-bita, ×65535 iz float).

> **Datoteke iz starijih izdanja mogu ne sadržavati skalu.** 8-bitni TIFF odraza normalizira se i označava `Chloros:PixelScale` kao i ostali. Izdanja prije 1.3.0 *isječena* je na 0..255 umjesto toga, tako da je svaka vrijednost iznad ρ≈0.008 poravnata na 255, i izostavili su i `Chloros:PixelScale` i tuplet `MicaSense:RadiometricCalibration`. **Ako oznaka nedostaje na LATTICE datoteci reflektancije, nemojte pretpostavljati mjerilo — ponovno je obradite** umjesto da dijelite piksele koji nikada nisu bili podijeljivi.

#### Čitanje indeksnih piksela (NDVI i drugih indeksa vegetacije)

**Integer indeksni raster je kontrastno istezanje po slici, a ne mapa od −1 do 1.** `TIFF (16-bit)`, `TIFF (8-bit)`, indeksni proizvodi `PNG (8-bit)` i `JPG (8-bit)` — uključujući 8-bitni TIFFu kojem je 8-bitni snimak zapisan — pohranjuju

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

gdje su `IndexMin` / `IndexMax` **najniže i najviše konačne indeksne vrijednosti tog kadra**. DN 0 nije −1, a puni raspon nije +1.

Chloros žigira oba krajnja točka kako bi mapiranje ostalo invertibilno:

| Označitelj | Značenje |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvencija u nastavku |
| `Chloros:IndexMin` | indeksna vrijednost pohranjena kao DN 0 |
| `Chloros:IndexMax` | indeksna vrijednost pohranjena kao puni raspon |

Pročitajte ih i inverzirajte, normalizirajući pohranjenu DN vrijednost na 0…1 prvo prema maksimalnoj vrijednosti pohranjenog dtype-a, tako da jedan izraz pokriva svaki cjelobrojni format:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Za čitanje nije potreban `-config pix4d.config` — exiftool čita ove XMP značajke samostalno. Konfiguracija je potrebna samo za njihovo *pisanje*.

> **`IndexMin == IndexMax`** znači da je svaki piksel te slike imao istu indeksnu vrijednost. Raster je popunjen srednjom vrijednošću ljestvice i nema čime dijeliti — uzmite krajnju točku.

> **Nedostatak oznaka `Chloros:Index*` znači da nema navedenog mapiranja.** Izvoz LUT-a je kolorizirani prikaz čiji pikseli sadrže boje gradijenta ključane prema vašem LUT-ovom prozoru za prag, a kadar bez piksela s konačnim indeksom nema raspon koji bi se mogao navesti. Ne pretpostavljajte mapiranje ni u jednom slučaju — isto pravilo kao i za nedostajuće `PixelScale`.**Budući da je istezanje po slici, dva kadra jednog leta nisu usporediva kao DN**— svaki nosi svoje krajnje točke, pa ista vrijednost NDVI pada na različite DN u različitim slikama. Za kvantitativni rad unutar leta (zonalna statistika, fiksni pragovi, vremenske serije), izvezite**`TIFF (32-bit, Percent)`**: indeksni rasteri tamo se zapisuju kao float32**samih indeksnih vrijednosti**, bez potrebe za ponovnim skaliranjem i bez oznaka. Unatoč nazivu formata, indeksni proizvodi nisu ×100 — to skaliranje primjenjuje se samo na cjelobrojni dtype reflektancije.

#### Unmix maskirna maska (na kojim je pikselima pojas bio ograničen)

Spektralna demiksa 3×3 može dovesti do negativnog pojasa — gdje svjetlost jedva sadrži taj pojas ili gdje scena izlazi izvan onoga što opisuje matrica za demiksu kamere. To nije stvarna vrijednost, pa je izvoz fiksira na 0 — a prikaz uživo prikazuje istu 0. Pojas se može zaklamati na nekoliko piksela ili na gotovo sve njih; maska bilježi bilo koji od ta dva slučaja. `Chloros:Band{R,G,B}UnmixClampFraction` navodi koliko se toga dogodilo na slici; maska za zaključavanje navodi **koji pikseli**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— u mapi `clamp_masks/`**pored** mape proizvoda, nikada unutar nje (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; izvoza `process` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — za svaki radijans, reflektancija i indeksni proizvod u kojem je barem jedan piksel zakvačen: bilježi i izvoze `process` na isti način (i TIFF-ovi po kadru `array-build-video --save-tiffs`). Jednopojasni `uint8`, na vlastitoj pikselnoj mreži proizvoda (uključujući orijentaciju, ROI, biniranje i poravnanje niza). Svaka maska navodi kameru svog proizvoda (`Make` / `Model` / serijski broj), vrijeme snimanja i oznake objektiva te navodi pojaseve koje njeni biti predstavljaju (`XMP-Camera:BandName`, prvi bit 0); njezini `ImageDescription` i `Software` označavaju je kao masku za pričvršćivanje i ne sadrži kalibraciju.
- **Biti**: bit 0 = pojas u R kanalu proizvoda (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Postavljeni bit znači da je vrijednost piksela u tom kanalu fiksirana na 0 — tretirajte ga, kao i bilo koji indeks izgrađen na temelju njega, kao nepouzdanog.
- **`Chloros:UnmixClampMask`** je utisnut na svaki proizvod koji je pokrenuo unmix: putanja maske u odnosu na vlastitu mapu proizvoda (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, uvijek odvojen `/`-om — spojite ga s mapom slike za otvaranje maske), ili `none` kada nije zaključan nijedan piksel. Nedostanak oznake znači da proizvod nikada nije pokrenuo unmix (RGB ili mono kamere, raw/debayered razine).
- `lattice capture` / `array-capture` ispisuju putanju maske ispod spremljene datoteke (`clamp mask: …`), a popis rezultata snimanja navodi ga kao `clamp_mask` kadra.
- `lattice index --live --save-multiband` zapisuje jednu masku po izvornoj kameri za višebandni stack — vidi [`lattice index` — maske za višebandni stack](#lattice-index--multiband-stack-masks). Video nema masku: `array-build-video` umjesto toga izvještava o clamp frakcijama svakog kadra (vidi [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Budući da nijedan folder proizvoda ne sadrži masku, na bilo kojoj razini, folder proizvoda (`radiance/`, `reflectance/`, moguće je predati izvoz `process` u mapu `*_Images/` programu Pix4D, Metashape ili drugom softveru za fotogrammetriju onakvu kakva jest — uključujući podmapu — i prima samo slike. Predajte mu mapu proizvoda, a ne mapu iznad nje (ona također sadrži `clamp_masks/` i sve ostale vrste izvoza). Chlorosov vlastiti uvoz preskače maske. Čitanje maske: `(mask & 1) != 0` je "ovdje je R-kanal zaključan", `& 2` G, `& 4` B.

#### EXIF prenesen u izvoz

`process` kopira **GPS blok i njegov ExifIFD** izvornog snimka na svaki proizvod, tako da izvoz sadrži `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` i `CameraSerialNumber` zajedno s georeferenciranjem.**`FocalLength` nije neobavezan za fotogrametriju.** Pix4D rješava udaljenost uzorkovanja tla (ground sample distance) na temelju žarišne duljine i nadmorske visine; ako oznaka nedostaje, prelazi na potpuno pogrešnu mjerilu. Na jednom letu nad narančinom nasadom s 49 snimaka, nedostajuća oznaka pretvorila je lokaciju od 411 m × 160 m u rekonstruiranu
47,8 km × 13 km — ortomapa od 455 MP s većinom neispravnih podataka, što se zatim shvatilo kao problem s podjelom na pločice i problem s BigTIFF-om prije nego što je itko provjerio GSD. Ako vaša ortomapa ispadne u nevjerojatnom mjerilu, prvo pokrenite `exiftool -FocalLength` na izvozu.

Kopija je namjerno **nije** `-all:all`: IFD0'strukturalne oznake krše LATTICE izlaz pri kopiranju, a `ExifImageWidth` / `ExifImageHeight` su isključeni jer opisuju *izvorno* snimanje — izvoz koji je ikada promijenjen veličine inače bi nosio dimenzije
što je u suprotnosti s vlastitim rasterom. XMP se piše izravno, a ne kopira, jer ExifTool
odbacuje XMP oznake iste invokacije kada se XMP blok kopira (čime bi se izgubile oznake kalibracije
MAPIR).

### Gdje se nalaze izlazne datoteke

Proizvodi se zapisuju **ispod mape projekta, grupirani po kameri, a zatim po formatu datoteke**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Mapa kamere je `LATT-<sensor>-<lens>-F<filter>` za LATTICE (što odgovara EXIF podacima snimke
`Model`) i `<model>_<filter>` za Survey3 — dvije kamere koje dijele senzor i filtar, ali se razlikuju po objektivu, drže odvojena stabla jer se vignetiranje, vidno polje i distorzija razlikuju. Format
folder prati `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, ili `tiff32` za `TIFF (32-bit, Percent)`.

> **Svaki izvezeni proizvod zadržava naziv izvornog datoteka.**Izvoz sjaja za `capture_…_raw.tif` i dalje se naziva `capture_…_raw.tif` — samo se nalazi u `tiff32/Radiance_Images/`.**Mapu identificira proizvod, a ne naziv datoteke**, stoga globbing za `*radiance*.tif` ne pronalazi ništa; umjesto toga uskladite po direktoriju.

### Snimke senzora svjetla — kalibrirani `.daq` + `.csv`

`process` također obrađuje snimke `.daq` u vašoj ulaznoj mapi, i on **ne**
ne treba nikakve slike za to: DAQ-U / DAQ-M / DAQ-E koji leti samostalno predstavlja potpuno snimanje, a mapa koja sadrži samo datoteke `.daq` valjana je ulazna datoteka.

DAQ se može snimati **bez** kalibracije — to je ono što javni
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) snimači (`record_daq.py`) to rade prema zadanim postavkama: zapisuju sirove brojeve senzora i označavaju datoteku kako bi Chloros preuzeo tvorničku kalibraciju tog senzora **prema serijskom** (prvo iz lokalne predmemorije, zatim iz oblaka MAPIR) i primjenjuje ga. `process` zapisuje rezultat natrag:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` prenosi jedan redak po očitanju: UTC oznaka vremena, vrijeme integracije, ukupna snaga,
fotopski/skotopski luks, PPFD (i njegova podjela na plavu/zelenu/crvenu boju), vršna valna duljina, a zatim
cjelokupni spektar na vlastitoj mreži valnih duljina senzora. `.daq` ponovno uvozi bez potrebe za drugom kalibracijom.

`<name>` je naziv datoteke snimanja. Dva snimanja u različitim mapama mogu dijeliti jedno — očitanja po snimci (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) spremljena u istoj tisućinki sekunde od strane dviju kamera — i svako i dalje dobiva svoje rezultate: prvo zadržava
`<name>`, a sljedeći dodaje mape koje ga razlikuju **nakon** naziva
(`<name>__<capture>_daq_calibrated.daq`), tako da datum i vrijeme snimanja i dalje dolaze na prvom mjestu.

U slučaju uspjeha, izvršavanje izvještava `Light-sensor products written: N (calibrated .daq + .csv)`.
Ono u zagradama opisuje što je zapravo napisano, pa glasi
`(RAW COUNTS — this sensor has no calibration bundle)` za senzor bez paketa i
`(N calibrated, M raw counts)` za mapu koja sadrži oboje. Naslovi vlastitih `[DAQ-EXPORT]` i `[RUN-SUMMARY]` izvode svoje formulacije na isti način — nijedan od njih troje ne može nazvati kalibriranim sirovi izvoz.

Zapis DAQ-U / DAQ-M / DAQ-E čiji se paket kalibracije ne može dohvatiti — jer ste izvan mreže ili taj senzor nema kalibraciju spremljenu — je **preskočen s razlogom** na
liniji `[DAQ-EXPORT]`, nikada nije zapisan kao datoteka "kalibrirana" koja sadrži sirove brojeve.
Povežite se na internet i ponovno pokrenite. Razlog je onaj koji je čitatelj zapravo utvrdio za tu datoteku (nečitljiva shema, nema paketa, pogreška pri pisanju), a sažetak pokretanja navodi **različite** razloge — dvadeset datoteka preskočenih iz jednog razloga računa se kao jedan razlog, a ne dvadeset ponavljanja istog. CLI navodi svako preskočeno snimanje na kraju izvršavanja, s razlogom:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Mapu snimki koja ne izvozi **ni jednu** od njih: vidi [Pokretanje koje ne piše nijednu sliku ne uspijeva](#a-run-that-writes-no-images-fails).

Snimka čije ime završava na `_calibrated` ili `_raw` (`field_raw.daq`) **nije
uvoženi**: to su nazivi koje `process` daje vlastitim proizvodima, a proizvod koji se očitava kao snimka bio bi dvaput uračunat. CLI to potvrđuje i imenuje datoteku —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— umjesto da izvještava o praznoj mapi. Preimenujte takvu snimku prije obrade.

#### Izvoz snimki DAQ-A kao sirovih brojeva

**DAQ-A**obitelj prethodi sustavu paketa po serijskom broju i nema paket kalibracije za preuzimanje — umjesto toga kalibrira se na terenu u odnosu na metu reflektancije, zbog čega mu on nikada nije bio potreban. Odbijanje tih snimki ostavilo ih je bez ikakvog načina da izvuku svoje brojeve, stoga se izvoze pod**drugim imenom**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Drugačiji naziv datoteke umjesto zastavice unutar datoteke, jer tvrdnja mora ostati očuvana
i ako se datoteka proslijedi e-poštom kao sam naziv. Zaglavlje `.csv` navodi `raw spectral sensor counts (NOT irradiance)` i upozorava da su vrijednosti usporedive **unutar**datoteke — što je upravo ono za što se kalibracija na temelju cilja koristi — a ne među senzorima. Fotometrijske stupce ovisne o snazi (ukupna snaga, fotopski i skotopski luks, PPFD) upisuje se kao**NULL** umjesto da se integriraju iz brojeva, a sažetak izvođenja
prikazuje `RAW COUNTS` pa "izvezeno" u logu se ne može čitati kao iradijansa.

Naslijeđeni zapisi **v1.01 / v1.02** (koje zapisuje DAQ-A-SD) ne sadrže epochu po očitanju,
samo vrijeme zapisa datoteke. Alat za podudaranje slike↔downwellinga i dalje ih odbacuje — podudaranje kadra s vremenom zapisa bilo bi pogrešno u pozadini — no izvoznik ih čita, a CSV ispisuje `clock=daq_created_on` pa proizvod navodi na kojem je satu.

### Napomene

- `process` automatski otkriva je li vaš direktorij Survey3, LATTICE ili mješovit.
- Progres se prikazuje putem Server-Sent Events; CLI prikazuje trenutačni napredak za svaku nit (otkrivanje, analiza, obrada, izvoz).
- Za Linux/Jetson, CLI provjerava swap i može upozoriti prije obrade velikih mapa. Debayer osjetljiv na teksture također automatski primjenjuje ograničenje frekvencije GPU-a na Jetson uređajima s niskom potrošnjom energije (Nano, Orin Nano).
- U slučaju uspjeha, izvršavanje izvještava o broju izdanih slikovnih proizvoda (`Image products written: N`, s `(plus M clamp masks)` kada je unmix ograničio bilo koji piksel).
- **`process` sigurno dijeli aktivni backend.** On automatski ponovno pokreće već aktivan backend samo kada je taj backend neaktivan, s prethodnim projektom učitanim i bez priključenih kamera, nizova ili DAQ-ova. Backend koji drži bilo koji od njih (GUI koji streamuje niz, skripta iz SDKa, snimanje DAQ-a) koristi se onakav kakav jest, nikada se ne ponovno pokreće, pa `process` može raditi paralelno s aktivnim nizom. Ako backend prijavi da je izvršavanje u tijeku, `process` čeka do 15 s da se završi (trčanje koje se upravo završilo treba oko sekunde da se očisti), zatim odbija s `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Globalna zastavica `--restart` i dalje prisiljava na novi backend i prekida svaku sesiju kamere, niz ili DAQ na njemu. Backend koji pokreće sistemd usluga na Linuxu nikada se ne ponovno pokreće: `process` ga koristi, a `--restart` umjesto toga ispisuje `sudo systemctl restart <unit>` (vidi [Pokretanje backenda kao usluge](#running-the-backend-as-a-service-headless)). Niti vlastiti backend aplikacije Chloros: kada je aplikacija otvorena, `process` ga koristi, učitavajući svoj projekt umjesto projekta aplikacije (backend pokrenut na njegovu mjestu zaključao bi otvorenu aplikaciju).
- **Pokretanje u tijeku zadržava svoj projekt.** Dok je obrada u tijeku — pokrenuta od strane aplikacije, `process`, skripte SDKa ili `chloros-mcp` — pozadinski servis odbija stvoriti, otvoriti, uvesti u, dodati ili ukloniti datoteke, premjestiti, brisati ili mijenjati postavke obrade projekta, uz HTTP 409 `PROCESSING_IN_PROGRESS` i `A processing run is in progress on project '<name>', so ... now is refused`; `process` koji to dobije izlazi iz `1`. Postavke prikaza (veličina minijature) i dalje se spremaju. Na isti način odbijeni su i način obrade te postavke ciljane reflektancije projekta (pokretanje zadržava način u kojem je započelo). Snimka spremljena u projekt tijekom izvođenja (iz aplikacije, CLIa, SDKa ili `chloros-mcp`) sprema se kao i obično, ali se pridodaje popisu datoteka projekta tek kada se izvođenje završi; njegov odgovor to potvrđuje (`project_add`), kao i snimanje senzora svjetlosti zaustavljeno u projektu. Prije je drugi klijent mogao promijeniti otvoreni projekt usred izvođenja, a izvođenje je zatim izvezeno u drugi projekt.
- **Ctrl+C zaustavlja izvršavanje u pozadini, a ne samo u CLI-ju.** CLI traži od pozadinskog sustava da prekine obradu, ispisuje `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` i čeka do 120 s na `Processing stopped.`, pa `chloros-cli process … && next` ne pokreće `next` na pozadini koja je još uvijek zauzeta trčanjem. Drugi Ctrl+C prekida čekanje. `process` zatim izlazi iz `130`, kao što to čini svaka prekinuta naredba.

#### Pokretanje koje ne piše slike ne uspijeva

Ako ste zatražili proizvode, a izvršavanje je napisalo **ništa** — samo `project.json` i
`calibration_data.json` — `process` to smatra neuspjehom: ispisuje
`Processing finished but wrote no image products.` i **izlazi s ne-nuljevim rezultatom**, pa ga skripta može
otkriti. Poruka navodi mapu projekta i uobičajene uzroke:

- ulazna mapa nije prepoznata kao snimanje (provjerite raspored i `--input-level`), ili
- svaki zatraženi proizvod je preskočen kao neprimjenjiv za te kamere (npr. traženje
  radijanciju/reflektanciju s kamera koje podržavaju samo RGB (RGB)).

Ponovno pokrenite s `--verbose` i provjerite zapisnik pozadinskog sustava (backend log) za retke s `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
što objašnjava preskakanje po kamerama koje inače ne dospijeva do izlaza CLIa.

Namjerno pokretanje samo za metapodatke — sa svim isključenim prekidačima za proizvode i bez `--indices` — i dalje je
**uspjeh**, jer je prazan izlazni prikaz slike u tom slučaju ispravan rezultat.

#### Pokretanje čiji se izvoz nije mogao provjeriti završava neuspjehom

Nakon izvođenja, `process` provjerava oznake izvezanih datoteka pomoću ExifToola. Ako ExifTool nije uspio pokrenuti se, ispisuje `Verifikacija izvoza NEUSPJELA: ExifTool se nije uspio pokrenuti, stoga oznake N od M izvezanih datoteka
nisu provjerene (pogledajte retke [EXIFTOOL] u logu pozadinskog procesa)` and `Proizvodi se pišu i ostaju
na mjestu; ponovno pokrenite za njihovu provjeru.`, does not print `Obrada dovršena!`, i **izlazi s ne-nultom vrijednošću**. Proizvodi se zadržavaju.

#### Snimke koje snimanje

Snimke dovršene prije pokretanja `process` uključene su. Snimke spremljene tijekom snimanja nisu; `process`
upozorava koliko je izostavljeno i ponovnim pokretanjem ih uključuje.

#### Pokretanje koje je izgubilo sliku je neuspješno

Ako je slika tijekom izvođenja neuspjela, `process` ispisuje `N image(s) failed and were not exported` s najčešćim uzrokom, ne ispisuje `Processing complete!` i **izlazi s ne-nuljevim rezultatom**. Ostali proizvodi se pišu i ostaju na mjestu, a u pozadinskom logu nalazi se svaki neuspjeh.

Tako je i s **pokretanjem samo za senzor svjetla**: mapu snimki `.daq` po definiciji nema slika za izvoz, a tijek se ocjenjuje prema kalibriranom `.daq` / `.csv` koji je umjesto toga napisao.
Ako nije zapisalo **ništa** — svako snimanje je preskočeno — ispisuje
`Processing finished but wrote no light-sensor products.`, imenuje svako snimanje razlogom zbog kojeg je preskočeno i
**izlazi s ne-nultom vrijednošću**.

---

## `chloros-cli login`

Provjerite autentičnost ovog računala pomoću računa u oblaku tvrtke Chloros+. Podaci za provjeru autentičnosti sigurno se pohranjuju u `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Primjeri

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (uklanjajući ga s, ili duplicirajući dijelove, lozinke). Na 401, CLI automatski ponovno pokušava s `$$` re-pridodanoj, a zatim s polovično dupliciranom lozinkom; ako ponovni pokušaj uspije, prijavljuje vas i ispisuje ispravnu sintaksu s navodnicima koju trebate koristiti sljedeći put.

> **Upotreba bez zaslona/skriptirana upotreba: nema pohranjenih sesija, što znači da će se pojaviti upit za prijavu.** Komanda za osnovnu upotrebu (`process`, `status`, `export-status`, `time-sync`, …) koji se pokreću bez pohranjene sesije traže unos `Email:` / `Password:` na stdin prije nastavka. Ako je stdin zatvoren (cron, systemd jedinica), program odmah završava s `chloros-cli logout`; ako je stdin otvoreni terminal ili cijev, program čeka unos. Pokrenite `logout` jednom po računalu prije zakazivanja bezglavog rada.

---

## `chloros-cli logout`

Čisti keširanu sesiju i prisiljava na novo prijavljivanje pri sljedećem pozivu.

Odjavljivanje dok je u tijeku izvršavanje ne zaustavlja ga: izvršavanje se dovršava u načinu obrade u kojem je započelo, a sljedeće izvršavanje koristi standardnu obradu. `logout` to navodi (`The processing run in progress finishes in <mode> mode; the next run uses standard`), kao i aplikacija kada se odjavite van.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Prikaži trenutnu razinu licence (Iron/Copper/Bronze/Silver/Gold), autentificiranog korisnika i broj veza uređaja.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Prikaže trenutni napredak izvoza Thread-4 uživo. Sigurno za pozivanje **tijekom** izvođenja `process` iz drugog shell-a.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Postavite jezik prikaza CLIa (podržano 38 jezika, uključujući CJK, RTL i indijske). Na naslijeđenim konzolama koje ne mogu prikazati skriptu, automatski se prebacuje na engleski jezik.

```
chloros-cli language [LANG_CODE] [--list]
```

### Primjeri

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Naredbe za mapu projekta

Ove naredbe upravljaju zadanoj lokacijom mape projekta (koja se dijeli s GUI-jem).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` ispisuje mapu u koju se spremaju novi projekti. Ako konfigurirana mapa više ne postoji (izbrisana je, preimenovana ili se nalazi na pogonu koji nije priključen), CLI se vraća na zadanu mapu, kao i GUI: zadana mapa se ispisuje na stdout, upozorenje s nazivom nedostajuće mape ide na stderr, a izlazni kod je `0`.

---

## `chloros-cli update`

Samo za Linux/Jetson. Provjerava `version_url` iz `/etc/chloros/update.conf` i, kada postoji novija verzija, preuzima odgovarajući `.deb` i odmah ga instalira pomoću `sudo dpkg -i` (bez potvrde; `sudo` može zatražiti vašu lozinku). Na Jetsonu se ažuriranje nudi samo sustavu JetPack 6. Na Windows naredba ne radi ništa i izvještava `up to date`: ažurirajte pomoću novog instalatera.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Na Linux/Jetson CLI također pokreće **automatsku provjeru ažuriranja pri svakom pokretanju** (neblokirajuće, nikada ne odgađa naredbu): čita `/etc/chloros/update.conf`, kešira rezultat na 1 sat u `~/.chloros/update_cache.json` i ispisuje `Update available: vX.Y.Z / Run: chloros-cli update` kada postoji novija verzija. Tiho preskače svaku pogrešku i na sustavu Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` to isključuje. Zbog predmemorije od 1 sata, izdanje objavljeno unutar tog sata i dalje se može prikazati kao ažurirano.

---

## `chloros-cli selftest`

Provodi 7-korakni "smoke" test: verzija, dostupnost porta, pokretanje pozadinskog procesa, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), postojanje modela za uklanjanje šuma, spremnost CUDA+denoiser. Koristi pokrenuti backend ili ga pokreće za test i zaustavlja nakon toga. Ispisuje `N/7 checks passed` i neuspjele provjere te izlazi `0` samo kada svih sedam prođe; provjera CUDA+denoiser ne uspijeva na računalu bez CUDA GPU-a (pogledajte [Provjerite instalaciju](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP grandmaster status i kontrola. Host Chloros pokreće PTP grandmaster; kamere LATTICE i jedinice DAQ-E rade kao robovi (slave) za njega za vremenske oznake između uređaja (otprilike 1 ms između uređaja). PTP poravnava **vremenske oznake**: kamere niza istovremeno se eksponiraju jer ih aktivira hardverski okidač M8 (za manje od 100 µs između modula), a ne zbog PTP-a. `chloros-cli time-sync` bez podzapovijedi pokreće `status`.

| Podzapovijed | Opis |
| --- | --- |
| `status` | Prikaže stanje grandmastera, BMCA prioritete, identitet sata i je li njegovi paketi dopiru do mreže: `health` je `OK` (`STOPPED` kada grandmaster nije pokrenut), ili `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` s `health_detail`, i zatim se u stderr ispisuje linija `WARNING:` (stdout ostaje JSON). `sync_sent` / `announce_sent` broje samo poruke koje su poslane; `*_send_errors` i `last_send_error` broje one koje nisu. Nakon što veza s kamerom prekine i ponovno uspostavi, grandmaster samostalno ponovno izgrađuje svoje sokete (`rebinds`, `last_rebind`). |
| `peers` | Popis robova viđenih putem Delay_Req (kamere + DAQ-E senzori). |
| `cameras` | Stanje PTP-a po kameri (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Ponovno pokrenite grandmaster proces. |
| `set-priority --priority1 N --priority2 N` | Prepišite prioritete BMCA. |

Grandmaster objavljuje na mrežnim sučeljima kamere (uključujući bilo koji adapter s adresom `169.254.x.x`). Ako nijedan nije postavljen, prelazi na adapter zadane rute samo kada je ta veza žičana, nikada ne koristi Wi-Fi ili virtualni adapter (Tailscale, WSL), budući da nijedna kamera ili DAQ-E stoji iza jednog. Kad nema što objaviti, radi bez soketa (`health` `NO_SOCKETS`), a sljedeća veza kamere ili DAQ-a ponovno ga gradi na NIC-u koji je do tada aktivan; `time-sync restart` to radi odmah.

Grandmaster radi u vlastitom procesu, uz backend (`own_process` / `process_pid` u `time-sync status`), s visokim prioritetom na Windowsu (`process_priority`), tako da zauzeti backend (izvršavanje obrade, izvoz, više naredbi odjednom) ne može odgoditi vrijeme koje bilježi; prije je jedna takva odgoda mogla pomaknuti sat svakog kamere odjednom. Taj se proces završava s backendom; ako ga nije moguće pokrenuti, grandmaster radi unutar backenda kao i prije (`own_process` false, i u logu backenda to piše).

### Primjeri

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Kontrola LATTICE kamere. Svaka podkomanda prolazi kroz backend Chloros; backend upravlja bazenom kamera pa naknadni pozivi CLI ponovno koriste isti otvoreni rukohvat.

### Uobičajene opcije (zajedničke za većinu podkomandi)

| Zastavica | Opis |
| --- | --- |
| `-d, --device N` | Indeks kamere (zadano: 0). |
| `-s, --serial SN` | Specifični serijski broj; nadjačava `--device`. |
| `--serials SN1,SN2,…` | Serijski brojevi odvojeni zarezom za rad s više kamera. [Privremeni naredbe za više kamera](#multi-camera-transient-sessions) otvaraju samo ove kamere, pri čemu prva aktivira okidač. |
| `--all` | Rad na svakoj otkrivenoj kameri. |
| `--exposure US` | Vrijeme ekspozicije u mikrosekundama. |
| `--gain DB` | Pojačanje u dB. |
| `--pixel-format FMT` | npr. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` na M3M. Naziv postavlja dubinu bita unutar vlastite obitelji kamere: na M3M Bayerovo ime postavlja mono format njegove dubine (`BayerRG8` postavlja `Mono8`), na M3C mono ime Bayerovo (`Mono12` postavlja `BayerRG12`). |
| `--width N` / `--height N` | Dimenzije slike. |
| `--preset {default,high_quality,high_speed,triggered}` | Primijeni unaprijed definirane postavke: njegov pikselni format, ekspoziciju, pojačanje i brzinu sličica (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` ih navodi). Na M3M formatu je mono, iste dubine: `high_speed` snima `Mono8`, `high_quality` i `triggered` `Mono12`. Zastavica koja se uz njega daje (`--exposure`, `--gain`, `--pixel-format`) nadjačava unaprijed postavljeno, a postavka koju unaprijed postavka ne navodi zadržava vlastitu postavku kamere. Sve besplatno-run osim `triggered`, koji aktivira kameru za hardverski rub na Liniji 2 — ako ta linija nije pod naponom, kamera će čekati zauvijek umjesto da snima. |
| `-o, --output DIR` | Izlazni direktorij (zadano: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Veličina GVSP paketa. `auto` pokreće ICMP+GVSP probe; `jumbo` = 9000; `standard` = 1500. |

### Radni tijek prve veze kamere LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referenca podzapovijedi

#### Otkrivanje i informacije

| Podzapovijed | Svrha |
| --- | --- |
| `lattice info` | Popis povezanih kamera (proizvođač, model, serijski broj, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analizira sustav domaćina radi optimalne konfiguracije kamere. `--no-discover` preskače otkrivanje kamere (brža analiza samo mrežne kartice). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Provjera/popravak postavki mrežne kartice (Linux, također i spremnici za primanje soketa kernela — vidi [Linux hostovi](#linux-hosts--socket-receive-buffers)); procjena propusnosti/FPS-a. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Backend mreža stabilne sheme-kapabilnost + preporuka niza (vraća `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` zadano koriste cijeli senzor, `2048` × `1536`. Izlazi iz `0` za `ok`, `auto_capped_fps` i `auto_shrunk`, i `1` za `needs_force_slip` i `error`, s `--json` ili bez njega. Dostiže backend na `--backend-url` (po zadanom `http://127.0.0.1:5000`), a ne na `CHLOROS_BACKEND_URL`. `auto_capped_fps` zadržava zatraženu rezoluciju, ali ograničava ciljani fps — pročitajte `recommended.recommended_target_fps` i proslijedite ga kao cilj povezivanja; tretirajte to kao uspjeh, nije pogreška. `auto_shrunk` se pojavljuje samo kada se traženi kadar ne može isporučiti ni pri jednoj brzini sličica; connect nikada sam ne smanjuje sličice. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analiza što-ako bez otvaranja kamera (`--width` / `--height` zadano `2048` × `1536`). I ova opcija i `network-analysis` predviđaju što će connect streamati: bez `--pixel-format`, formate po kameri koje primjenjuje connect (Mono12 / BayerRG12), i s `--packed auto` (po zadanom) [packed 12-bit](#packed-12-bit-transport---packed) odluku koju bi connect donio na ovom računalu — stoga je fps naveden brzina kojom niz radi. Oba ispisuju odluku o pakiranju. **`--n-active` je ukupan broj kamera na kabelu, a ne samo broj za ovaj niz**— podiže ga kada samostalne kamere istovremeno prenose podatke ili kada se mrežni budžet izračunava prema potražnji koja ih ne uzima u obzir (zadano: `len(--models)`). Uvijek ispisuje agregatne linije `Wire budget:` (MB/s potražnje naspram sigurnog gornjeg ograničenja) i `Max cameras:`, te označava `** OVER-SUBSCRIBED**` kada niz prekorači kapacitet veze — vidi [Model burst i fps niza](#array-fps--burst-model). |
| `lattice gpu` | Prikaži status GPU-a. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Provjeri ili ažurir firmver kamere. Lokalni `.fwa` odabir je zakvačen: datoteka u `firmware/<MODEL_PREFIX>/` koja odgovara `MIN_FIRMWARE_VERSION` izgradnje se flashira kada je prisutna (samo najnovija verzija kao rezervna opcija), stoga novija slika dobavljača spremljena na disk je neaktivna dok se taj pin ne promijeni — namjerno novije verzije stižu na jedinice putem potpisanog AWS manifesta, što je poželjnije kada su novije. |
| `lattice presets [--apply NAME]` | Popisati ili primijeniti postavke kamere. `--apply` postavlja što nazivi preseta označavaju (njen format piksela, ekspoziciju, pojačanje, brzinu sličica, okidač) i ispisuje te postavke kako ih kamera čita nakon: na M3M, mono format dubine preseta; brzinu sličica u sekundi kojom radi, koja može biti niža od one koju preset traži (`high_speed` traži 60 fps; kamera punog kadra na 1 Gbps vezi radi oko 35). Sve ostale postavke koje je otvorio vraćaju se onakvima kakve su bile pronađene. `--apply default` resetira kameru na tvorničke postavke: tvorničke postavke s neograničenom automatskom ekspozicijom na Liniji 2, ograničenjem propusnosti veze i isključenom korekcijom nedostataka, te sa zadržanim kalibriranom bilancom bijele boje kamere u RGB formatu (jedinica na svim ostalima). Ništa se ne sprema u memoriju kamere, pa pri sljedećem uključivanju započinje iz vlastite memorije kao i prije. Kao `--preset` snimanja ili povezivanja, `default` nije unaprijed postavka. |
| `lattice status` | Stanje kamere uživo. |

#### Snimanje

| Podzapovijed | Svrha |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Pojedinačni kadar. **Po zadanome spremaju se sve vrste izvoza**(`--processing all`); pogledajte [Razine izvoza snimanja](#capture-export-levels-the-all-default). `--processing` koristi `all`, `raw`, `debayered`, `radiance`, `reflectance` ili `display` (`preview` je prihvaćen za `display`); svako drugo ime se odbija prije nego što se dodirne kamera (izlaz `1`). `--levels` sprema eksplicitni podskup (nadjačava `--processing`). Chloros ne drži kameru koja se otvara za snimanje, a njezina ekspozicija se prvo postavlja iz scene (najviše nekoliko sekundi), kao što to čine [privremeni nalozi](#multi-camera-transient-sessions); kada se snimanje završi, kako god završilo, ta se kamera vraća u stanje u kojem je zatečena (format, ekspozicija, pipeline boja, svaka postavka koju je snimanje promijenilo — popis pod [Više kamera (privremeni sesije)](#multi-camera-transient-sessions)). Prikaz DAQ-a korišten pri snimanju sprema se kao pomoćna datoteka `.daq`, koja uključuje samo sirovi snimak; `--no-daq` / `--daq-tolerance-ms` ga podešavaju ([DAQ sprema očitanje snimke](#the-daq-reading-a-capture-saves)). `--force-daq` se i dalje prihvaća. `--jpeg-quality` = kvaliteta JPEG 1–100 (zadano 95).**Datoteke spremljene u otvoreni projekt pridružuju mu se.** Kao u aplikaciji, snimka čije datoteke završe u mapi projekta otvorenog u Chloros pridodaje ih popisu datoteka tog projekta; datoteke spremljene drugdje ostaju nepromijenjene. Naredba ispisuje jedan redak koji to potvrđuje (`Project: ...`), ili, dok je izvršavanje obrade u tijeku, da se one odgađaju i pridružuju se kada izvršavanje završi. Isto vrijedi i za `multi-capture`, `array-capture` (također `--smart`) i `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Snimanje na disk dok ne pritisnete Ctrl+C. Pokretanje `tiff`okvira je `raw` datoteka niz snimanja te kamere: mozaik senzora, jedna ravnina sa svojim vrijednostima, nekomprimirana, s kalibracijskim paketom (12-bitni kadar dimenzija 2048 x 1536 iznosi 6,0 MiB plus paket, bez obzira na scenu), pa `process` započinje od njih. `png` i `jpg` sadrže 8 bita, pa su ti nizovi demosaicirani kadrovi za pregled, od kojih `process` ne započinje. S DAQ-om vezanim (`daq pool-bind`) ili objedinjeni, svaki kadar dobiva svoj `.daq` i `.match.json` u mapi `daq/` te serije, kako `lattice capture` sprema jednu; `--no-daq` to isključuje, a zaustavljanje ispisuje koliko je snimljeno. Svaki se kadar zapisuje jednom, sa svim svojim metapodacima; pri pritisku Ctrl+C, već snimljeni kadrovi se zapravo pišu prije nego što se kamera otpusti. Zaustavlja se kad se disk napuni, ispisuje `Failed:` / `Disk full:` i izlazi `1`; svaki kadar koji nije uspio zapisati također izlazi `1`. Kamera ostavljena u pripravnosti za hardverski okidač (`TriggerMode` `On`) postavljeni su na slobodno pokretanje za izvršavanje. Kada izvršavanje završi, kamera se vraća u stanje u kojem je bila — njezini su parametri okidača i svaka postavka koju je izvršavanje promijenilo (ekspozicija, pojačanje, format, …) — osim ako je aktivan `--keep-settings`, koji sprema postavke snimanja u vlastitu memoriju kamere (slobodno snimanje), ili `--clear-settings`, koji vraća tu memoriju na tvorničke postavke. Pokretanje koje ne snimi nijedan kadar smatra se neuspješnim: ako 10 s ne snimi kadar, sam se zaustavlja, ispisuje `No frames:` s razlogom i izlazi `1`. Okviri koje pisci nisu mogli pratiti računaju se kao ispušteni, a zaustavljanje ispisuje `Dropped:` s razlogom (brzina okvira kamere, pisaca i reda čekanja); niža brzina okvira ili 8-bitni format piksela zadržavaju ih sve. Komanda se javlja backendu svakih 2 s; ako umre bez zaustavljanja snimanja (`kill -9`, nedostatak memorije), backend ga zaustavlja 30 s nakon posljednjeg javljanja i vraća kameru (`stop_reason` `client_gone`, i redak u zapisima backenda). `multi-stream` i `align-stream` rade isto. Pokretanje čija kamera treba duže da se otvori nego što naredba čeka (60 s; 180 s za `multi-stream` / `align-stream`) — kamera se još pokreće — poništava se u pozadinskom sustavu: kamera se vraća i zatvara i snimanje se ne nastavlja (HTTP 504 `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Pregled uživo MJPEG-a u pregledniku: povezuje kameru, ispisuje URL struje i otvara ga u vašem pregledniku. `--ae-damping` postavlja prigušivanje automatske ekspozicije (0,4–100). |

##### Prijenosi uživo (MJPEG; bez RTSP-a)

Chloros poslužuje video uživo kao **MJPEG putem HTTP**(`multipart/x-mixed-replace`, boundary `frame`) iz pozadinskog sustava.**RTSP nije podržan.** Otvorite URL u pregledniku ili ga pročitajte bilo kojim MJPEG klijentom (OpenCV-ov `cv2.VideoCapture(url)`, ffmpeg, VLC). Kamera se prvo mora povezati u pozadinskom sustavu (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` ili putem GUI-ja).

| URL (`http://127.0.0.1:5000` + …) | Stream |
| --- | --- |
| `/api/camera/stream/<serial>` | Prikaz s preglednika, do otprilike 30 fps. Za član niz, `?max_h=` (po zadanom `720`) i `?jpeg_q=` (po zadanom `80`) postavljaju visinu i kvalitetu JPEG-a. Ovo je URL koji otvara `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | Isti pregled s prikazanim GUI-jevim preklapanjima: `zebra=1`, `crosshair=1`, `grid=3x3` (ili `9x9`), `histogram=1`, `peaking=1`, `spot=1`; bez upita, spremljena postavka preklopa projekta za tu kameru. |
| `/api/camera/<serial>/stream-reflectance` | 8-bitna reflektancija po pojasu: ρ = piksel ÷ dio's `X-Chloros-Pixel-Scale` zaglavlje (255 osim ako je `refl_max` podignuto). `fps` (zadano `10`), `jpeg_q` (zadano `95`), `max_h` (zadano `0` = nativno), `band=<name>`, `align=1`. JPEG, tako gubičan. Odbijeno s HTTP 409 kada kamera nema refleksancu (RGB kamera, nema kalibracijskog paketa, nema DAQ očitanja). |

- **Usklađivanje kadrova između kamera.**Svaki dio koji**član niza** pošalje, na sva tri toka, nosi `X-Chloros-Group-Id` (brojač sinkronizirane skupine niza), zaglavlja `X-Chloros-Frame-Id` i `X-Chloros-Timestamp-Ns`. Dijelovi iz različitih kamera s istim `X-Chloros-Group-Id` dolaze iz istog okidača. Samostalna kamera nema grupu i ne šalje ID grupe. Preglednici zanemaruju zaglavlja po dijelu; klijent koji samostalno analizira višedijelnu streamu ih čita.
- **Pauzirano dok je prikaz niza otvoren.** `/api/camera/stream/<serial>` člana niza šalje statički "pauzirani" kadar dok je kombinirani prijenos uživo niza otvoren (na primjer, prikaz niza u GUI-ju) i nastavlja se kada se zatvori.

#### Podesivost senzora

| Podzapovijed | Svrha |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Čitanje/pisanje bilo kojeg GenICam čvora. `--set ExposureTime=…` (`Gain=…`) bez njegovih `ExposureAuto` (`GainAuto`) prekidača prvo isključuje taj automatski način rada, kao što `lattice exposure --set` radi; vrijednost koju kamera odbije ispisuje `Failed to set <node>: …` na stderr i izlazi `1`. `--get` i zadani grupirani prikaz ispisuje `note:` pod `ExposureTime` / `Gain` dok taj automatski način rada radi — vidi [Očitavanje ekspozicije i pojačanja u automatskom načinu rada](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Ekspozicija i AE. Ekspozicija `--set` ili cilj `--brightness` drži se kao vaš (Chlorosova automatska kontrola ekspozicije je ostavlja na miru); dobitak ostaje kakav je bio, pa se u automatskom načinu nastavlja prilagođavati prema cilju dok ne postavite `lattice gain`. `--auto` vraća ekspoziciju, `--brightness-auto` vraća cilj automatskom cilju Chlorosa. Bez zastavica čita čvorove ekspozicije, s istim `note:` kao `configure --get`. Vrijednost `--set` koju kamera odbije ispisuje `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` na stderr — u automatskom načinu rada, ako je pisanje isključeno, vraća se u ispravan rad umjesto da ostane isključeno — ispisuje `Exposure: … us (auto OFF)` bez retka, i izlazi `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | pojačanje i automatsko pojačanje. A `--set` vrijednost koju kamera odbija ispisati `Failed to set Gain: …` na stderr, bez `Gain: … dB (auto OFF)` retka, i izlazi `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | ROI senzora i biniranje. Biniranje je uvijek **Prosjek** — vidi dolje. |
| `lattice format [--set FMT] [--list]` | Format piksela. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardverski/softverski okidač. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (bez zastavica = jednokratno WB) | WB operacije. Samo za RGB/Bayer kamere; neaktivno (preskače se) na mono M3M. Jednokratno snimanje zadržava rješeno ravnotežu bijele (`BalanceWhiteAuto` i pojačanja); svaka druga postavka vraća se u izvorno stanje, kao i sve postavke kada se jednokratno snimanje ne može izvršiti. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB pipeline boje za prikaz. `natural` (zadano) je jeftini završni postupak uživo; `enhanced` dodaje defringe + vibrance + lokalni kontrast CLAHE za cjelokupni izgled hub-parity uz ~2× veću cijenu obrade po kadru, što rezultira nižom **live** stopom sličica — snimke spremljene u cacheu ionako uvijek dobivaju potpunu obradu. Samo za RGB/Bayer kamere; izostavljeno na mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Zasićenje/kontrast uživo (RGBkamere s filtrom); `100` je neutralno, `--reset` postavlja oboje na 100. Pohranjeni snimci nisu pogođeni. Preskače se na mono M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (bez zastavica = čitaj) | **Istaknuti detalji** (RGB-kamere s filtrom): izvlači najsvjetlije, najbjelje piksele prema dolje — npr. svijetla bijela površina prikazuje se kao ravna bijela mrlja. `0` = isključeno, `-100` = najjači. Prikaz uživo i snimke na razini prikaza ga slijede; radijansa/reflektancija nikada ne, i ne može oporaviti podatke s senzora koji su podrezani (za to smanjite ekspoziciju). Preskočeno na mono M3M. |
| `lattice filter [--list]` | Popisati modele filtara koje Chloros poznaje (`RGN-IMX265`, `OCN`, `NGB`, …). Filter kamere čita se iz njezine modelne naredbe i ne može se postaviti: `--set NAME` samo provjerava model, a zatim izlazi iz `1` s `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Ispituje snagu/termalne čvorove; prebacuje u stanje mirovanja niske potrošnje. `--sleep` ostavlja kameru u stanju mirovanja (ograničenje propusnosti veze uključeno (`DeviceLinkThroughputLimitMode` `On`) na svojoj donjoj granici, način uštede energije uključen gdje ga kamera ima) sa svakom drugom postavkom koju je otvorio vraća se na prvobitno stanje; uspjeli pokušaj uspavanja vraća se u cijelosti. Buđenje vraća vezu u stanje u kojem ju je zateklo uspavanje (ograničenje, njegov način rada i kašnjenje paketa `GevSCPD`, koje kamera drži na 0 dok je ograničenje uključeno; zapis se čuva na ovom računalu, tako da ga buđenje nakon ponovnog pokretanja pozadinskog sustava također pronađe), ili, za kameru koju ovaj računalo nije stavilo u stanje mirovanja, a koja izgleda kao da spava (ograničenje uključeno na minimalnoj razini), s isključenim ograničenjem i `GevSCPD` ostavljenim na 0 pri 80 ( tvorničke postavke); ograničenje postavljeno iznad donje granice ostaje nepromijenjeno. Sonda i probuđenje vraćaju ono što je njihovo otvaranje zapisalo. |

> **Binning je uvijek prosjek.** `lattice resolution --binning N` postavlja binning prosjeka i ne nudi prebacivanje načina rada. Sum binning množi svaki DN s N², što košta otprilike dva stopa rezervnog raspona svjetline po koraku binninga; Chloros taj faktor ponovno dijeli prije radiometrijskog lanca, ali pikseli koje je Sum doveo do gornje granice senzora se odrezuju i ne mogu se povratiti. Ne kupuje ništa što Average ne nudi — indeks je neovisan o mjerilu, a radijantni skalarni vektori kalibrirani su na mjerilu Average. Ako kamera stigne već postavljena na Sum (ostavljena tako od strane drugog alata ili tvorničkim resetiranjem), Chloros to otkriva iz očitavanja čvora i
kompenzira; pokretanje `lattice resolution --binning N` vraća ga na Average. Verzija 1.2.0
> zastavica `--binning-mode` i dalje se prihvaća pa starije skripte rade, ali ona samo ispisuje upozorenje:
> binning ostaje Average.

> **Promjena binninga, formata piksela i okidača na kameri ništa ne zaustavlja.** Na kameri
> pozadinski proces drži otvorenim i streamanje (`cam-connect`, GUI), `lattice resolution --binning`
> ne stupaju na snagu (ispisuje se binning koji je još uvijek na snazi, `Binning: 1x1`), a
> `lattice format --set` i `lattice trigger --mode` kamera odbijaju
> (`SC_ERR_ERROR -1001`). prvo ga otpustite (`lattice cam-disconnect -s <SN>`), ili postavite
> ih pri povezivanju (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Čitanje ekspozicije i pojačanja u automatskom načinu rada

Dok je `ExposureAuto` (`GainAuto`) kamere aktivan, njegov `ExposureTime` (`Gain`) čvor pohranjuje **posljednju zabilježenu vrijednost**, a ne ono što kamera trenutno radi. Za kameru, backend drži otvorenim (`cam-connect`, `array-connect`, GUI), `lattice configure --get`, zadani `lattice configure` prikaz i `lattice exposure` (bez zastavica) stoga ispisuju vrijednost čvora i, ispod nje, bilješku s najnovijim okvirom kamere:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Pročitajte vrijednost okvira iz bilješke; vrijednost čvora je samo posljednji zapis. Preko HTTP isti `POST /api/camera/<serial>/read-nodes` odgovor održava `nodes` nepromijenjenim i dodaje `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, najnovijeg kadra) i `notes` (gornja rečenica). Oba su prisutna samo dok automatski način rada nije `Off` i kad se zna za kadar; kamera otvorena samo za čitanje nema kadar i ne dobiva ništa. `lattice gain` (bez zastavica) ispisuje samo čvor — koristite `lattice configure --get Gain` za bilješku.

##### Izlaganje na članu niza

Dok niz prenosi, svaki član radi s **`AcquisitionFrameRateEnable` on**, pri brzini sličica nešto iznad stope okidanja niza: njegov period je period okidanja umanjen za 15 % (najmanje 3 ms), ili vlastita maksimalna brzina kamere kada je okidač već blizu nje. Stoga `configure --get` na članu čita `AcquisitionFrameRateEnable` kao `True` i `AcquisitionFrameRate` pri toj postavci; omogućavanje se ponovno isključuje kada niz prestaje.**Ne pišite ove dvije vrijednosti na član niza** — niz ih usklađuje sa svojim okidačem i ponovno ih zapisuje.

Ta postavka je također gornja granica ekspozicije člana: oko 85 ms pri okidaču od 10 fps (razdoblje od 100 ms), pri **bilo kojem ROI-u ili biniranju**Prije verzije 1.3.0, mala ROI je umjesto toga ograničavala izloženost člana na slobodni period senzora (4,9 ms pri 768 × 304), bez obzira na to koliko sporo je niz okidao.**Ručni `ExposureTime` duži od gornje granice usporava okidač kako bi se uklopio** kada se piše pomoću `lattice exposure --set` ili `configure --set` (ili iz GUI-ja). Niz prvo otvara gornju granicu tog člana za tu vrijednost i pušta sporiji tempo da odradi nekoliko perioda okidanja, tako da pisanje može potrajati nekoliko sekundi duže; okidač se zatim aktivira najviše onoliko brzo koliko to dopušta ekspozicija. Na nizu s automatskom brzinom, prethodna brzina se vraća čim se svaki član čije je ručno upravljanje ekspozicijom usporilo vrati na automatsku ekspoziciju. Vrijednost koju sama kamera ne može snimiti i dalje se odbija, uz navođenje raspona koji prihvaća u poruci o pogrešci, a automatska ekspozicija člana vraća se ako je isključena samo za to pisanje. Ekspozicija prema receptu `apply` ne usporava okidač: on se drži na gornjoj granici. Chloros nikada ne skraćuje ručno postavljenu ekspoziciju kako bi odgovarala bržem okidaču; fiksirana brzina (`array-connect --target-fps F`) koja je brža od one koju ekspozicija dopušta ostavlja tom članu dovoljno dugu gornju granicu za to, i taj član prihvaća okidače najbrže onoliko koliko mu ekspozicija dopušta.

#### Kalibracija i senzori

| Podzapovijed | Svrha |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibracija s reflektirajućeg cilja. Pretražuje se do `--attempts` okvira (zadano `30`); nepotpuni kadar (GVSP gubitak paketa) košta jedan pokušaj, a ne cijeli tijek. Neuspijeva s `no target detected in N attempts`, ili `no complete frame in N attempts: <last error>` kada nijedan okvir nije stigao u cijelosti. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spektr i veličine pojaseva DAQ-M-a, čita se putem Bluetootha (BLE) preko pozadinskog programa. `--mac` je vlastiti BLE MAC senzora i potreban je za bilo koju od tri radnje: nema zadane vrijednosti, budući da svaka jedinica ima svoj (`daq pool-discover --only ble` ih navodi); bez njega naredba izlazi `1`. Bez akcije ispisuje svoju pomoć. `--camera SERIAL` integrira ispisane magnitude pojaseva kroz vlastiti izmjereni spektralni odziv te kamere umjesto krivulje flote modela filtera — RGB kamere, s kalibracijskim paketom kamere učitanim (povezanim) ili spremljenim na ovom računalu. Naslov navodi čija je krivulja korištena: `Band magnitudes (camera <SERIAL>):`, ili `Band magnitudes (model <filter>):` kada kamera nema nijednu (nije RGB kamera, ili nema instaliranog ili spremljenog paketa) i umjesto toga je korištena fleet krivulja. Preko HTTP, `POST /api/dls/cli-read` uzima `camera_serial` i vraća `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Primijeni korekciju vinjete na postojeće slike, svaku sa svojim flat fieldom objektiva (iz njegovog EXIF Modela). `--lens-model` primjenjuje se na slike čija imena Modela ne navode kameru; slika čiji Model to proturječi preskače se. |

#### Višekamera (Privremeni sesije)

Privremena naredba otvara kamere za jedno izvršavanje i zatvara ih kada ono završi: `multi-capture` bez priključene trajne matrice, `multi-stream` i `multi-test` ovdje, i `align-calibrate`, `align-apply`, `align-stream` i `index --live` dolje. Ova pravila vrijede za sve njih:

- **`--serials SN1,SN2,…` bira kamere.** Otvaraju se samo one, a prva aktivira okidač (za `align-calibrate`, `--reference` kada se navedu). Serijski broj koji nije na mreži dovodi do neuspjeha pokretanja prije nego što se otvori ijedna kamera. Bez `--serials`, otvara se svaka kamera na mreži. Kamera Chloros, koja već drži (GUI, `cam-connect`, `array-connect`), odbija pokretanje bez obzira na to naziva li je `--serials` ili ne: dijeli mrežu za koju je pokretanje namijenjeno. (`index --live` bez `--serials` umjesto toga otvara kamere omogućene u svom profilu: vidi [Index / Vegetation Maths](#index--vegetation-maths).)
- **Kamere dijele mrežu na način na koji je `array-connect` dijeli.** Svaka kamera tempom prilagođena je svom udjelu u onome što nosi glavna veza (odgoda između paketa, a ne ograničenje propusnosti kamere, čiji prag od 31,25 MB/s četiri kamere na 1 GbE već premašuju), tako da četiri kamere rade na 1 Gbps vezi. Kada kamere ne mogu sve istovremeno slati brzinom koju postignu, aktiviraju se jedna za drugom, svaka sa svojom vezom, i naredba ispisuje `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Kada se čak ni jedna kamera odjednom ne može uklopiti, izvršavanje se odbija prije nego što bilo koja kamera počne s prijenosom: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, izlaz `1`.
- **Svaka kamera isporučuje, ili pokretanje ne uspijeva.** Kamera koja ne isporuči nijedan kadar predstavlja pogrešku: zapovijed ispisuje `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, gdje je `<why>` pogreška preuzimanja te kamere, zatim ono na što pokazuju uzroci, i završava `1`. Okvir koji je stigao s GVSP paketnim gubicima (`IncompleteBufferError: GVSP packet loss …`) znači da mreža ne može prenijeti kombiniranu propusnost kamera (`Every missing frame arrived with GVSP packet loss: …`): provjerite brzinu veze hosta i uplinka switcha (`lattice network`), koristite jumbo pakete, ili otvorite manje kamera s `--serials`. Kamera koja nije isporučila ništa obično ne prima okidač s glavne jedinice: provjerite njezin M8 sinkronizacijski kabel, a zatim pokrenite `lattice multi-detect`. Što svaka naredba radi s grupom okvira u kojoj nedostaje kamera, navedeno je u njezinoj retku.
- **Svaka se kamera vraća u prvobitno stanje.** Kada se izvršavanje završi, svaka kamera vraća ono što je imala kada ju je izvršavanje otvorilo: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, i `LineMode` / `LineSource` na `Line2` i `Line3` (i deglitcher za ulazni okidač niza daje liniju robota, `LineFilterSelector` / `LineFilterWidth`); njegov `PtpEnable` (i skup `PtpSlaveOnly` / `PtpDomainNumber` s njim); `ExposureAuto` / `GainAuto` (i `ExposureTime` / `Gain` gdje je bila na ručnoj vrijednosti); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (i svaki `BalanceRatio` gdje je bijela ravnoteža bila ručna); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (i `Gamma` gdje je bila uključena gamma), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; veličinu paketa (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); tempiranje, `DeviceLinkThroughputLimitMode` (i `DeviceLinkThroughputLimit` gdje je način bio `On`), inter- kašnjenje paketa `GevSCPD` i početno razmještanje `GevSCFTD` niz daje svakoj kameri; podaci o komadima koje svaki stream uključuje (`ChunkModeActive`, svaki `ChunkEnable` pod njegovim `ChunkSelector`); `AcquisitionFrameRateEnable` (i `AcquisitionFrameRate` gdje je enable uključen — kad je isključen, brzina je samo za čitanje i čita najviše što ritam dopušta, pa to nije postavka). Pisano je samo ono što se razlikuje, kamera je prva stala, tempiranje i brzina sličica prije ekspozicije zahtijevaju njihov period sličice (kamera ograničava `ExposureTime` na jedan period sličice). S `--keep-settings` priključci za okidač i sinkronizacijsku liniju vraćaju se u izvorno stanje, kamera radi neovisno, a ostatak se sprema u vlastitu memoriju kamere prema postavkama koje je zadala postavka 'run'; `--clear-settings` vraća tvorničke postavke.
- **Ekspozicija se postavlja na scenu prije nego što se zadrži prvi kadar.** Tijekom izvođenja uzimaju se sinkronizirane skupine sličica i odbacuju dok se iz tih sličica podešavaju ekspozicija i pojačanje svake kamere. Cilj je srednja svjetlina na cilju kamere bez više od 1 % sličice urezano (clipped); scena s jako osvijetljenim detaljima zaustavlja se prije cilja. To obično traje 2–6 grupa sličica i najviše 5 s.
  - Svaka kamera cilja na metu koju joj postavlja povezani niz: TargetBrightness **40**na multispektralnoj kameri (RGN, OCN, NGB) i**60** na RGB kameri. To je polovica cilja ekspozicije koju 80 daje (tri četvrtine na RGB), pa je manje vjerojatno da će svijetli panel reflektancije biti odsječen. `--brightness N` (0-255) postavlja cilj svake kamere umjesto toga (`multi-test` nema `--brightness`: njegova automatska ekspozicija je isključena). Log backenda imenuje cilj svake kamere: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Izloženost dolazi prva, do granice automatske ekspozicije same kamere. Dobit pokriva samo preostali dio.
  - Pronađena ekspozicija i pojačanje zadržavaju se tijekom cijelog izvođenja, pa se svaki kadar `multi-stream` snima pri istoj ekspoziciji. Snimanje koje mora pratiti promjenjivu svjetlost pripada povezanom nizu (`array-connect`), čija se automatska ekspozicija neprestano prilagođava.
  - `--exposure` i `--gain` zadržavaju se točno onako kako su zadani. Kada su oba zadana, nijedan kadar se ne odbacuje.
  - Kada se izvršavanje završi, automatsko izlaganje svake kamere ponovno se uključuje.
  - Log datoteka pozadinskog sustava (backend) izvještava o rezultatu: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, ili `exposure NOT solved after …` s razlogom i `(not settled)` za svaku kameru na kojoj nije uspjelo. U tom slučaju se slike i dalje snimaju.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` mijenja ograničenje od 5 s; `0` ga isključuje (vidi [Promjenjive okruženja](#environment-variables)).

**Dok Chloros drži bilo koju kameru** (GUI, `cam-connect` ili povezani niz), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` i `index --live` odbijaju se pokrenuti jer otvaraju svaku kameru na mreži i drugo otvoreno otpuštanje zaključane kamere može srušiti pozadinski sustav. (`index --live` odbija kada je kamera koju bi otvorio zaključana: njegova `--serials` ili kamere njegovog profila, i bilo koja kamera za profil koji ne navodi nijednu.) Backend odgovara HTTP 409 `CAMERA_HELD`, navodeći svaku zadržanu kameru i naredbe za otpuštanje (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), a naredba izlazi `1`. Kada je niz priključen, `array-capture --continuous` ga bilježi umjesto `multi-stream`, i `array-capture` (podrazumijevano poravnato) sprema poravnati skup umjesto `align-apply`; `multi-capture` i `align-calibrate` koriste povezanu niz koji sadrži njihove kamere umjesto da ih otvaraju.

| Podzapovijed | Svrha |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Popis svih kamera s ulogama sinkronizacije; s `--serials`, samo one, prva kao `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Jedan sinkronizirani kadar sa svake kamere. Pohranjuje **sve vrste izvoza prema zadanim postavkama** kada je povezana trajna niz, i DAQ očitanje koje se koristi ([DAQ očitanje snimanja pohranjuje](#the-daq-reading-a-capture-saves)); privremeni povratni postupak bez niza vrši samo debajeriranje (prvo pokrenite `array-connect` za ostalo). `--brightness` odnosi se samo na rezervno rješenje: priključena matrica zadržava vlastitu automatsku ekspoziciju. Rezervno rješenje ne sprema `.daq` (`--no-daq` / `--daq-tolerance-ms` nemaju tamo učinka), a `multi-capture` ne uzima `--force-daq`. U fallbacku, kamera koja ne isporuči nijedan kadar ne uspijeva u snimanju (`Error: multi-capture failed: no frame from …`, izlaz `1`) i nijedna datoteka te grupe kadrova se ne sprema. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Prijenos sinkroniziranih sličica (prolazno), `--fps` prema zadanim postavkama `3`, sve dok se ne pritisne Ctrl+C ili `--count` grupe. Okvirna grupa za koju kamera nije isporučila ništa se ne zapisuje, i tri uzastopna završavaju stream. Ako je neka grupa preskočena, izvršavanje završava ispisom `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, plus `Error: the stream stopped: …` s nazivom kamere koja je završila stream, i izlazi `1`. Zaustavlja se kad se disk napuni, ispisuje `Failed:` / `Disk full:` i izlazi `1`; svaka datoteka koju nije moguće zapisati također izlazi iz `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO sinkronizacijska vremenska proba (privremena), `--count` zadani `10` okidači. Ispisuje kako je mreža podijeljena (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` i linija `Serial emit:` kada se kamere aktiviraju jedna za drugom, čiji vremenski žigovi tada uključuju pomak), redak `Sync group: master SN:<serial> -> SN:<serial>, …` po sinkronizacijskoj grupi (`(no slaves: not synced)` za kameru bez kabela za sinkronizaciju, koja se aktivira sama), zatim jedan redak po okidaču: `[  0] SN:<serial>=<timestamp> \| …`, s `SN:<serial>=(no frame)` za kameru koja za taj okidač nije isporučila ništa. Zatim `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` i `SN:<serial>: mean <±x> us, jitter <y> us` linija po kameri, svaka kamera u odnosu na master unutar jednog takta: okidač čiji pečati miješaju satove uređaja kamera i PTP (PTP isključen ili se još zaključava) izostavljen je, a `Note:` ih broji. Svaki propušteni kadar dovodi do neuspjeha testa (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, izlaz `1`); tri uzastopna nepotpuna okidača prekidaju ga ranije. Test također propada (izlaz `1`) kada manje od dva okidača imaju svaku kameru na jednom satu (`… the clocks cannot be compared`), ili kada se satovi nikada ne podudaraju: PTP satovi su više od 1 ms (plus bilo kakvo pomicanje na serijskoj razini) udaljeni od mastera (`… PTP is not locked`), ili satovi uređaja čiji se pomak između okidača pomakne za više od toga. Pokreće PTP grandmajstora ako ne radi, uključuje PTP na svakoj kameri (kao što to radi `array-connect`, bez obzira na izgled mreže) i prije prvog okidačka čeka da se PTP sat svakog kamere zaključa na grandmajstora i stabilizira: `PtpStatus` Slave na jednom grandmasteru, unutar 100 us od njega i stabilan 3 s, i svaka kamera zaključana na taj način, bez prekida, 120 s — otprilike 2 minute od uključivanja-on. Sat koji se upravo zaključao odskače za do ~200 us otprilike 2 minute nakon što pročita Slave, a vlastiti `PtpOffsetFromMaster` kamere to ne prikazuje; 120 s dovodi kamere unutar ~10 us od stabiliziranog sata. Kamera još uvijek zaključana od ranijeg testa koji je ostavio PTP uključen, kako ga je ovaj backend vidio, broji od tog zaključavanja, pa test odmah nakon drugog ne čeka ponovno; kamera koju test uključuje, ili ona s PTP-om uključenim koju ovaj backend nije vidio da je zaključana (uključena drugdje ili prije ponovnog pokretanja backenda), čeka svojih 120 s, a u nizu su i kamere koje su se upravo zaključale i koje još uvijek čekaju svoje. Kamera koja izađe iz stanja Slave, prekorači 100 µs ili prati drugog grandmastera tri uzastopna očitanja (1,5 s) tijekom čekanja, ponovno pokreće svoje odbrojavanje; jedno ili dva takva očitanja bilježe se (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) i odbrojavanje se nastavlja, a ID grandmastera koji se ne može pročitati smatra se nepoznatim, a ne promjenom. Ponovno pokretanje odbrojavanja na svim kamerama istovremeno (unutar jedne sekunde) je promjena vremena grandmastera, a ne kamera: zapisnik pozadinskog sustava navodi `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` ga navodi u `grandmaster_steps`, a čekanje koje potom istekne započinje svoju pogrešku s `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Granica za to se ne produljuje. `--ptp-settle SECONDS` postavlja vrijeme poravnanja za izvršavanje (0-600; `0` za bez čekanja poravnanja). Čekanje je ograničeno na vrijeme poravnanja plus 90 s (210 s prema zadanim postavkama, najmanje 90 s), a naredba čeka na pozadinu 240 s plus vrijeme poravnanja (360 s prema zadanim postavkama). Ispisuje `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, a u logu pozadinskog sustava pojavljuje se redak svakih 5 s s stanjem i pomakom svake kamere, koliko je dugo zaključana i koliko je vremena preostalo: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Svaka se kamera uključuje s isključenim PTP-om; odatle se satovi zaključaju za otprilike 10 s. Nakon toga se svaka kamera vraća u stanje u kojem ju je test zatekao — njezin PTP, automatska ekspozicija i pojačanje koje je test držao isključenim tijekom izvođenja, te ostalo (osim ako `--keep-settings`) — tako da kamera ostavljena s PTP-om uključenim (`lattice configure -s <serial> --set PtpEnable=True`, ili pokretanje s `--keep-settings`) ostaje zaključana, i jednom kada je ovaj pozadinski proces vidio da je zaključana tijekom vremena za stabilizaciju, kasniji test ne čeka na nju. Ako se satovi ne usklade tijekom čekanja, test propada prije aktivacije (`PTP: NOT locked …`, izlaz `1`) i govori što učiniti: kamera koja se nikada nije zaključala (`PtpStatus Listening`) znači da je grandmaster ne doseže (`time-sync status`); ona koja je stalno ispadala imenovana je vremenom koliko je bila zaključana (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); ako se satovi još uvijek usklađuju: provjerite `time-sync status`, i da budu zaključane prije testa, uključite PTP i ostavite ga uključenim (`lattice configure -s <serial> --set PtpEnable=True` za svaku kameru, ili pokrenite test s `--keep-settings`; test vraća ono što je pronašao, pa ostaje uključen), zatim ga pokrenite. |
| `lattice multi-detect [--line LINE] [--json]` | Pronađite kamere na M8 sinkronizacijskoj liniji i zadani master: kamera RGB (model koji završava na `-FRGB`) ako je ožičena, inače prva ožičena kamera prema serijskom broju. Kabeli M8 se spajaju na jednom čvorištu, pa **bilo koja ožičena kamera može biti master** — ispisani `MASTER` je zadani, a ne žični podatak. Svaka kamera redom pokreće liniju, a ostale se provjeravaju za njezin impuls; kamera koja je već pronađena u ožičenoj skupini više se ne ispituje, tako da potpuno ožičena mreža zahtijeva jedno krug. Bez `--line` ispituje `Line2`, zatim `Line3`. Ispisuje `MASTER: SN:…`, jedan `SLAVE: SN:…` po svakoj drugoj kabliranoj kameri i `(probed on <line>)`; `--json` zbraja cjelokupan rezultat (`master`, `slaves`, `no_cable` — kamere koje puls nije dosegao — `line`, i po kameri `results`). Kada završi, svaka kamera dobiva natrag postavke okidača i sinkronizacijske linije koje je imala (isti čvorovi kao gore). |

#### Poravnanje

| Podzapovijed | Svrha |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus detektor/uskladivački gumbi `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC kotačići `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinacija više okvira `[--averaging mean\|median\|inlier_weighted]`, geometrijska ograničenja `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, prostorno ograničenje `[--roi X0,Y0,X1,Y1] [--mask PATH]` i po-slave nadjačava `[--per-cam-override SN:KEY=VALUE]` (ponovljivo) | Izračunaj profil poravnanja iz živih kamera. Zadano: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (u mapi iz koje ga pokrećete), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` ima vlastite zadane postavke; vidi [njegova tablica](#project-align-calibrate-options).) `--prefilter` prema zadanim postavkama koristi `gradient` (mapa rubova; usklađuje se s GUI/poravnavačem nizova — rubovi se prenose kroz spektralne pojaseve). `--matcher flann` se isplati za više od ~5000 značajki; `--averaging median` je otporan na jednu lošu snimku, `inlier_weighted` ponderira prema broju podudaranja; `--lock-scale` projektira na najbližu rotaciju (bez skale), `--lock-axis` postavlja na nulu jednu komponentu translacije; `--mask` primjenjuje se na svaku kameru (za postavke po kameri upotrijebite `--per-cam-override`, npr. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odbija spremiti kalibraciju čija RMS reprojekcije premašuje prag. `--method manual` uzima svoje točke iz `--points PATH`: JSON-objekt `{"<serial>": [[x, y], ...], ...}` koji daje koordinate piksela svake kamere za iste značajke, uključujući i referentnu kameru (najmanje 2 po kameri); svaka od `--frames` rješava se na temelju tih točaka. Kada povezan niz sadrži kamere, kalibracija koristi žive okvire tog niza, a profil također postaje poravnanje niza za njegova snimanja. Inače otvara [privremeni niz](#multi-camera-transient-sessions): kamera koja ne isporuči nijedan kadar ni u jednom od `--frames` ne uspijeva s kalibracijom (izlaz `1`, imenovanje), i nijedan profil se ne sprema. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Snimanje jednog poravnatih višebandnih kadra. `--bit-depth` prema zadanim postavkama podudara kameru; `--no-crop` zadržava cijeli kadar (pad crnom bojom); `--interpolation` (po zadanom `linear`) i `--border-mode`/`--border-value` (po zadanom `constant`/0) kontrolira CPU warp — GPU putnja je u svakom slučaju bilinearna. Otvara [transient array](#multi-camera-transient-sessions): kamera koja ne isporuči nijedan kadar ne uspijeva s snimanjem (izlaz `1`, imenujući je), i ništa se ne zapisuje. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Prijenos poravnatih višebandnih okvira (isti warp-regulatori kao i kod `align-apply`), `--fps` zadano `3`. Otvara [privremeni niz](#multi-camera-transient-sessions): grupa kadrova za koju kamera nije isporučila ništa se ne sprema, a tri uzastopna završavaju stream. Ako je bilo koja grupa preskočena, izvršavanje završava ispisom `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` i `Error: …`, i izlazi iz `1`. Tok koji završi zbog bilo koje druge pogreške također ispisuje `Error: …` i izlazi iz `1`. |
| `lattice align-info --profile PATH [--json]` | Prikaži detalje profila. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Promijeni redoslijed slojeva. |

#### Indeks / Matematika vegetacije

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` mapira simbole postavke na pojaseve** (NDVI: `red`, `nir`), po jedan `--channel SYMBOL=BAND`.

- **`--live`:**samo za simbol na koji hrpa ne može odgovoriti.** Kao i kod `--input`, simbol preseta koji izostavite uzima onu jednu traku iz hrpe (trake omogućenih kamera profila) čije ime počinje s njim. To se provjerava prije nego što se bilo koja kamera otvori: ako se za simbol ne odgovara nijednom pojasu ili se odgovara za više pojaseva, tada se odbijaju, imenuju se i izlazi iz `1`. Prilagođeni `--formula` zahtijeva `--channel` za svaki simbol.
- **Proizvodi `--live` navode čiji su.** `index.tif` i `index_LUT.png` nose kameru hrpe (Proizvođač, Model, serijski broj), njezino vrijeme snimanja (ekspozicija referentne kamere, UTC) i objektiv, a oba imenuju indeks kao svoju jednu traku (`XMP-Camera:BandName` `NDVI`; PNG imenuje indeks, a ne njegove kanale boja), kao što to čini proizvod `--input`. Indeks iz pojaseva više kamera umjesto toga imenuje svaku od njih, a njegov `Model` je `LATTICE` — vidi [Datoteke izrađene od više kamera](#files-made-from-several-cameras).
- **`--input`: samo za simbol na koji datoteka ne može odgovoriti.** Simbol koji izostavite uzima onu traku datoteke čije ime počinje s njim: `red` uzima `Red_660` ili `Red_660_RGN`, `nir` uzima `NIR_850`. Nazivi dolaze iz same datoteke: vlastita lista slojeva stoga, ili, na snimci radijance/reflektance uređajaChloros ili na proizvodu `process`, njegove oznake `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Dakle, `lattice index --input <reflectance capture> --preset NDVI` ne treba `--channel`.
- **Ne pretpostavlja se između dviju traka koje odgovaraju jednom simbolu** (stog od 9 kamera ima dvije NIR trake): nazovite je. Simbol koji ne odgovara ničemu odbijen je s `channel_map missing entries`, izlaz `1`.**Nazivi pojaseva u nizu** nose filter kamere: pojasevi M3C kamere su `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` na OCN-u; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` na OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` na RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` na OCN-u; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` na RGB-u; `Blue_475_NGB` NGB-u — i M3M kamerajedna je banda `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` navodi bande profila.

Kompletan skup zastavica: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (ponovljivi), `--capture-level raw|debayered|radiance|reflectance|unknown` (nadjačava razinu snimanja zabilježenu u izvornoj datoteci TIFF; zadano: čita iz metapodataka TIFFa, a u nedostatku koristi `Chloros:ProcessingLevel` iz datoteke Chloros, tako da izvještaj o refleksanciji bilježi `reflectance`), `--output PATH` / `-o` (offline: put; s ekstenzijom slike datoteke se nazivaju prema njezinoj osnovi — `-o ndvi_FRGN.png` zapisuje `ndvi_FRGN.tif` i `ndvi_FRGN_LUT.png` — i bez njega je prefiks, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; zadano pored unosa, kao `<input name>_<preset>`; s `--live` i bez `-o`, mapa `output/` unutar mape iz koje pokrećete naredbu), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Kod `--live` također se primjenjuju gumbi za pomicanje poravnanja: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (cilj automatske ekspozicije kamera, vidi pravila privremenih sesija), `--serials SN1,SN2,…`. `--live` otvara [privremeni niz](#multi-camera-transient-sessions): kamera koja ne isporuči nijedan kadar ne uspijeva u izvršavanju (izlaz `1`, s navođenjem naziva), i ne zapisuje se ni stack ni indeks. Otvara `--serials` (prva aktivira okidač), inače kamere profila čiji je unos `layer_order` omogućen, prvo njezina referenca — referenca čak i s onemogućenim trakama, budući da su ostale poravnate prema i ona pokreće okidač. Kamera čije su trake onemogućene nikada se ne kalibrira, pa RGB kamera onemogućena u profilu više ne propada indeks reflektancije (`no-absolute-radiance-scale`). Profil koji ne navodi nijednu kameru otvara svaku kameru na mreži, kao i prije.

Svaki obrazac radi na pozadinskom sustavu, stoga mora biti pokrenut: pokrenite ga iz GUI-ja Chloros ili naredbom koju CLI ispisuje kada ga ne može dohvatiti. Izvan mreže, pozadinski sustav čita `--input` i piše plutajući TIFF i sam kolorizirani PNG, pa relativni `--input` ili `-o` rješava se u odnosu na mapu iz koje pokrećete naredbu (vidi [Konvencije](#conventions)). Oba offline proizvoda nose zapis kamere unosa, kao što to čini izvoz indeksa u formatu `process`: Proizvođač / Model / serijski broj, GPS, oznake objektiva, vrijeme snimanja (vlastito vrijeme unosa; kada u njegovoj EXIF oznaci nije navedeno, trenutak ekspozicije koji je zabilježen ili oznaka iz naziva datoteke, u UTC-u) i identifikaciju kalibracije MAPIR — nikada kalibraciju sirovog DN-a, koja se ne primjenjuje na indeks — plus `Chloros:ProcessingLevel=index` i izraz indeksa; float TIFF imenuje jednu traku, preset. `--list-presets` i `--list-gradients` ispisuju tablice prednastavki i gradijenata pozadinskog sustava. Formula koju pozadinski sustav odbaci ili traka `--channel` koju TIFF nema, izlazi iz `1` s razlogom pozadinskog sustava.

> **Simboli `--channel` osjetljivi su na velika i mala slova.** Stranica simbola mora točno odgovarati nazivima kanala preseta (preseti koriste mala slova, npr. NDVI = `red`,`nir` — provjerite `--list-presets`), a strana pojasa mora odgovarati nazivu pojasa u poravnatoj hrpi (ili biti indeks pojasa s bazom 0 u izvanmrežnom načinu rada). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` radi; `--channel RED=660` ne uspijeva s `channel_map missing entries` grešku.

#### `lattice index` — maske za više-pojasne hrpe

Kod `--capture-level radiance` ili `reflectance`, dekonvolucija 3×3 svake kamere može zaklamati pojas na 0 (vidi [Unmix clamp mask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Višebandni stack sadrži pojaseve više kamera, a svaka kamera zaključava svoje, pa `--live --save-multiband` zapisuje **jednu masku po izvornoj kameri** uz stack:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Svaka maska je jednobandna `uint8` na pikselnoj mreži **stoga** (putem iste korekcije poravnanja i obrezivanja), s bitovima te kamere: bit 0 = band njenog R kanala, bit 1 = G, bit 2 = B. Pojas stoga se uzima iz kanala *c* kamere i fiksira se na mjestu gdje je bit *c* te kamere (R = 0, G = 1, B = 2) postavljen.
- Ugrađeni nazivi podrijetla svakog pojasa imenuju masku njegove kamere (`unmix_clamp_mask`, relativno prema mapi hrpe, ili `none`) i bit za čitanje (`unmix_clamp_mask_bit`).
- Kamera čije unmixiranje nije ništa zaključalo ne dobiva datoteku; mono ili RGB kamera ne provodi unmixiranje i također ne dobiva nijednu. CLI ispisuje svaku masku ispod hrpe (`clamp mask SN…: …`); odgovor ih navodi kao `clamp_masks`.
- Indeks TIFF/PNG iste izvedbe izračunava se iz istog staka, na istoj mreži, pa ga maske također opisuju. `--capture-level raw` ne provodi unmix i ne zapisuje masku.
- Stack i indeks nalaze se izravno u `--output`, stoga se maske stavljaju u vlastitu mapu `clamp_masks/` unutar `--output` — nikada iznad mape koju ste nazvali. Ta mapa miješa vrste proizvoda (stack, index, colorized render): odaberite njezine datoteke, nemojte predavati cijelu mapu softveru za fotogrametriju.

#### Trajne veze (Smart-Prep, GUI-ekvivalentni tijek)

Ove naredbe drže kamere otvorenima u pozadinskom poolu između poziva CLI-ja.

| Podzapovijed | Svrha |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Dodaje jednu kameru u bazen (jedna kamera, bez niza). [zajedničke opcije](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` i `--packet-size` (`standard` = 1500, `jumbo` = 9000, ili bajtova; `auto` probe) primjenjuju se pri povezivanju (prebacivanje na `--exposure` ili `--gain` isključuje automatski način rada). Vraća se kad stream kamere započne (čekajući oko 15 s na to, zatim 10 s na prvi potpuni buffer) i ispisuje `stream: streaming`, ili `stream: no_frames -- connected, no frames yet: …` kad stream nije isporučio ništa — za kameru s jumbo paketima, uz savjet da se ponovno povežete pomoću `--packet-size standard`. Dok se jedna kamera povezuje (prvo povezivanje preuzima njezinu kalibraciju, oko 70 s), druga naredba kojoj je potrebna kamera čeka na nju najviše 5 s, zatim završava s HTTP 503 `CAMERA_POOL_BUSY` navodeći naziv kamere koja se povezuje; ponovno je pokrenite za trenutak. `lattice status` i popis kamera nikada ne čekaju: kamera usred povezivanja prikazuje se kao povezana, sa svojom fazom. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Oslobađanje. Kamera se vraća u stanje u kojem je bila prilikom povezivanja (spojevi okidača, PTP, postavke); `--keep-settings` umjesto toga sprema svoje postavke u vlastitu memoriju, `--clear-settings` resetira tu memoriju na tvorničke postavke (vidi [Memorija kamere](#camera-memory----keep-settings----clear-settings)). Kamera koja je član niza oslobađa cijeli niz. |
| `lattice cam-list` | Popis kamera u bazenu. |
| **`lattice array-connect`**|**Povežite trajni sinkronizirani niz (PREPORUČENI ulazni dio).** Pokreće cjelokupni GUI tijek pametne pripreme. Otprilike jedna minuta za 9 kamera; odmah nakon uključivanja kamera može potrajati nekoliko. Dok čeka, ispisuje svaki korak do kojeg backend dođe, kao `    <s> s: <step>` (na primjer `    41 s: open SN:214000535 (4 of 9)`), a u logu pozadinskog sustava nalazi se po jedan redak s vremenom za svaki korak (`[array-connect <id>] <step>: <s> s …`) i posljednji redak sa najsporijim koracima. Čeka do 600 s. Ako se do tada ne uspostavi veza, nalog se poništava: svaka kamera koju je otvorio vraća se u stanje u kojem je bila prilikom otkrivanja i zatvara se, zapovijed ispisuje `Error:` s korakom do kojeg je stigla i izlazi `1`, i nijedan niz nije povezan (HTTP 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C otkazuje povezivanje na isti način (`Interrupted: …`); ako je niz već bio povezan, linija to navodi i imenuje `array-disconnect` koji ga otpušta. Naredba ubijena dok čeka (`kill -9`) ostavlja povezivanje za poništavanje kada backend nije primio nikakve podatke od njega 30 s. Niz koji je aktivan, ali još nema sinkroniziranu grupu okvira, i dalje se prijavljuje kao povezan, a njegov stream je `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Oslobađanje niza. Svaka kamera ostaje u slobodnom načinu rada (`TriggerMode` `Off`), a sve ostalo je onako kako je bilo pri povezivanju — njezini su postavljeni okidač i sinkronizacijska linija (masterov `TriggerSource` vraća na prvobitno stanje, njegovu sinkronizacijsku liniju vraća na ulaz), njegov PTP i postavke kamere koje je sesija promijenila — tako da naredba za pojedinačnu kameru na članu radi odmah. `--keep-settings` sprema svaku kameru postavke svake kamere kako ih je sesija ostavila u vlastitu memoriju kamere; `--clear-settings` resetira tu memoriju na tvorničke postavke. Vidi [Memorija kamere](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Popis povezanih nizova. |
| `lattice array-status [--array-id ID]` | Živi fps, PTP, posljednja pogreška. `ptp_enabled` je `True` samo kada svaki član čita PTP uključen; `ptp_off:` navodi člana koji to ne čini. `state:` linija čita `restarting (applying settings)` tijekom ponovnog pokretanja postavki, `recovering (…)` dok se ponovno otvara član koji je izgubio napajanje (vidi napomenu u nastavku), `stalled (…)` dok se ciklus hvatanja izvršava, ali ne objavljuje grupu kadrova, i `stopped (…)` kad se ciklus hvatanja odustane (ponovno povežite niz). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno sinkronizirano snimanje iz live niza — Pojedinačno / Neprekidno / Intervalno / Najbrže. **Zadano je `all`**(jedna datoteka po primjenjivom tipu izvoza po kameri),**poravnato** kao što je snimanje aplikacije (`--no-aligned` za izvorne datoteke). Preskočene kamere (npr. RGB isključen iz radijance/reflektance) izvještava se s `Skipped: SN:<serial> (<reason>)`; DAQ očitanje korištenog snimanja spremljeno je uz njega, uključeno je samo sirovo snimanje i izvještava se s `DAQ: <path>` ([The DAQ reading a capture saves](#the-daq-reading-a-capture-saves)). Pogledajte [Načini snimanja, snimači i offline reprocesiranje](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Snimite pregled uživo kombiniranog indeksa u video/GIF (kvalitete za nadzor; bez otvorenog GUI-ja, pozadinski sustav prikazuje NDVI niza, ili `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Burst sirovog Bayer formata visokog fps-a (za analizu; ponovno obraditi izvan mreže). Poput `array-record` i `array-capture --continuous`, prijavljuje se kod pozadinskog sustava pri svakom upitu: ubijen bez zaustavljanja (`kill -9`), pozadinski sustav ga zaustavlja 30 s nakon posljednjeg prijavljivanja (`stop_reason` `client_gone`; manifest serije se finalizira i pokreće se izrada kombiniranog indeksa, kao i kod zaustavljanja) i niz ostaje povezan. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Ponovno obradi spremljeni sirovi burst u kalibrirani video(e). |

> **Kamera koja izgubi napajanje ili vezu sama se vraća.** Kada član prestane odgovarati (ponovno pokretanje prekidača napajanja ponovno pokreće svaku kameru koju napaja), niz čeka na njega, ponovno ga otvara kada ponovno odgovori, i vraća mu ono što je `array-connect` postavio: PTP, liniju okidača i sinhronizacije, mrežni tempiranje, format piksela i veličinu slike te automatsku ekspoziciju. Zatim ponovno prenosi bez novog `array-connect`. U međuvremenu `array-status` prikazuje `state: recovering (...)` i `last_error` navodi za koju kameru čeka; nakon toga ispisuje `re-opened:`. Kamera koja se ponovno pokrenula vraća se s isključenim PTP-om, pa dok se ponovno ne otvori `ptp_enabled` je `False`, a `ptp_off:` joj dodjeljuje ime.**Zadržana kamera se nikada ne otvara drugi put.** Drugo otvaranje zaustavlja stream držača i prepisuje njegove postavke te može dovesti do pada pozadinskog procesa. Dakle, za kameru koju drži pool (GUI ili `cam-connect`), član niz, ili kameru koju drži pozadinski zadatak (pokrenuti `lattice continuous`, `multi-stream` ili `align-stream`, `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, a one-shot `white-balance` i `presets --apply` odbijaju s HTTP 409 `CAMERA_HELD`, navodeći vlasnika i naredbu za otpuštanje (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, ili za zadatak njegov ID i tko ga je pokrenuo), i izlaz `1`. Isto vrijedi i za kameru koju trenutno koristi druga naredba (`multi-test` ili `multi-capture` koja otvara svoje kamere, još jedna jednokratna naredba), za svaku naredbu ovdje: poruka o pogrešci kaže da se pričeka dok ta naredba ne završi. `lattice power` bez `--sleep` čita kroz vlastiti handle nositelja gdje postoji (pool, niz, otvorena kamera posla), i odbijen je za kameru koju koristi druga naredba. Svaki odgovor `CAMERA_HELD` nosi kod u oba polja, `code` i `error_code`. Za naredbe koje otvaraju svaku kameru, pogledajte [Više kamera (privremeni sesije)](#više-kamera-privremenih-sesija).

##### Opcije `array-connect`

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatsko otkrivanje svih LATTICE kamera (potrebno ≥2) | Prvi serijski je MAJSTOR. Ako se izostavi, otkrivanje se filtrira na modele LATTICE (M3C / M3M) i povezuje ih sve. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO sinkronizacijska linija. |
| `--target-fps F` | automatski | Brzina ispaljivanja okidača Mastera. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automatski | Prepiši odabirač sloja. |
| `--wire-ceiling-mbps MB_PER_S` | automatski otkriveno | **Održivi protok mrežne veze hosta, u MB/s — broj na kojem ovisi cjelokupna alokacija niza.** Smanjite ga kada niz prijavi oštećene GVSP okvire: automatska vrijednost izvedena je iz oglašene brzine veze mrežne kartice (NIC), koja precjenjuje USB adaptere, uske PCIe trake i zauzete zajedničke mrežne resurse. Pohranjuje se u bloku za snimanje niza projekta, pa ga ponovno otvaranje / CLI / ponovno povezivanje SDK-a vraća. Vidi [Zdravlje niza](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | nijedan (cijeli senzor) | Hardversko biniranje (uvijek Prosjek). |
| `--width N` / `--height N` / `--pixel-format FMT` | cijeli senzor; format po kameri | Eksplicitna veličina okvira ili format piksela za svaku kameru, umjesto odabira formata cijelog senzora i formata po kameri (Mono12 / BayerRG12). |
| `--no-recommend` | isključeno | Preskoči korak analize mreže (svaka kamera tada zadržava svoje trenutačne dimenzije umjesto da se postavi na cijeli senzor). |
| `--no-ptp` | off | Onemogući PTP (vremenski pečati između kamera tada **nisu** usporedivi). |
| `--packed {auto,on,off}` | `auto` | Komprimirajte i dekomprimirajte 12-bitne okvire na ovom računalu — više okvira u sekundi pri punoj veličini senzora, identični podaci. `auto` komprimira samo kada to pomaže i ovaj računalo si to može priuštiti; `off` nikada ne pakira. Pogledajte [Pakirani 12-bitni transport](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP veličina paketa za svaku kameru u nizu. `auto` omogućuje svakoj kameri da ispituje mrežu; `jumbo` = 9000, `standard` = 1500, ili veličina u bajtima je fiksira. Prije verzije 1.3.0 `array-connect` je prihvaćao ovu zastavicu i zanemarivao je. |

> **Cijeli senzor prema zadanim postavkama.** Bez `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` postavlja svaku kameru na cijeli senzor — bez
> binninga, bez ROI obrezivanja — i dopušta da se brzina sličica spusti na ono što ovaj računalo i mreža mogu isporučiti, umjesto da se smanjuju sličice kako bi se zadržala brzina. Redak za analizu mreže prikazuje predviđenu brzinu, a sljedeći redak navodi brže opcije (binning ili manji ROI) s njihovim predviđenim brzinama; prođi
> `--binning` ili `--width`/`--height` da odaberete jednu. Connect nikada ne vrši binning ili cropping sam od sebe.

##### Pakirani 12-bitni prijenos (`--packed`)

Kod punog opterećenja senzora, brzinu snimanja slika niza određuje brzina kojom svaka kamera može slati svoje slike. 12-bitni formati (`BayerRG12`, `Mono12`) obično se prenose kao 16-bitne riječi; **spakirani** prijenos šalje istih 12 bita u 1,5 bajta po pikselu i
i raspakira ih na ovom računalu, pa svaki kadar zahtijeva 25 % manje vremena za slanje —
otprilike **33 % više kadrova u sekundi pri punoj upotrebi senzora** (izmjereno: 10,7 → 14,0 fps
na nizu od 9 kamera). **Pohranjeni podaci su identični**: iste vrijednosti, iste 16-bitne datoteke, isti metapodaci.

- `auto` (po zadanom) komprimira samo kada to pomaže **i** kada računalo može priuštiti
  dekompresiju. Prilikom povezivanja pozadinski sustav mjeri vlastitu brzinu raspakiranja, broj jezgri, trenutno opterećenje CPU-a i mrežni budžet, tako da stolno računalo s viškom jezgri pakira na punoj senzorskoj snazi, dok zauzeto ili niskoenergetsko računalo prenosi raspakirane podatke. Mali ROI-ji i binirani kadrovi dobivaju
  malo.
- `off` nikada ne komprimira — koristite ga da isključite kompresiju tijekom dijagnosticiranja, ili na
  računalo koje mora zadržati sav svoj CPU za druge poslove.
- `on` uvijek komprimira 12-bitne formate.

Izlaz pri povezivanju prikazuje odluku i zašto (`packed 12-bit: on (auto:
projicirano 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` izvještava
`packed_transport` i `packed_decision`. Da biste postavili zadano vrijednost za cijeli sustav
umjesto da svaki put prosljeđujete zastavicu, postavite `CHLOROS_ARRAY_PACKED12=off` (ili `on`) u okruženju pozadinskog sustava — izričiti `--packed on|off` i dalje prevladava.
8-bitni i 16-bitni formati nikada se ne pakiraju.

### Jedan zapis projekta, svaki način rada

Mapu projekta sadrži postavke svakog uređaja: `cameras.json` — svake kamere (`cameras[].settings`, za pojedinačnu kameru i za svakog člana niza) i one koji se odnose na cijeli niz (`arrays[].capture`: format, ROI, binning, brzina okidača, budžet za žice) — i `sensors.json`, svakog senzora svjetla (vrijeme integracije, prosjek kadrova, ograničenje). Aplikacija, CLI, SDK i MCP poslužitelj svi čitaju i pišu iste datoteke, na isti način:

- **Uređaj povezan u okviru projekta sprema se u njega i bilježi svaku promjenu u njega.** "Pod projektom" znači s projektom otvorenim u pozadini (backendu aplikacije), ili — iz SDKa — putem `ChlorosProject`, koji imenuje vlastiti mapu. Kamera, niz ili senzor svjetla koje projekt još ne sadrži dodaju se u njegove datoteke kada se povežu (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplikacija, SDK, `chloros-mcp`), sa svojim postavkama. Promjena iz bilo kojeg načina rada (`lattice exposure`, `gain`, `configure --set`, `resolution`, nizpostavke i brzinu okidanja, poklopac senzora svjetlosti, kontrole aplikacije) spremaju se u datoteke tog projekta nakon kratkog vremena. Postavljena ekspozicija, pojačanje ili cilj automatske ekspozicije (AE) na fotoaparatu spremaju se kao "držano", pa ih ponovno otvaranje ponovno zadržava.
- **Otvaranje projekta postavlja njegove uređaje**, u svakom načinu rada: fotoaparat, niz ili senzor svjetla koji su povezani bez vlastitih postavki preuzimaju spremljene postavke projekta (`lattice cam-connect` / `array-connect` s otvorenim projektom, `chloros-cli project connect`, `open_project().connect_all()`, ponovno otvaranje aplikacije). Niz povezan pod novim ID-om ažurira niz koji je projekt spremio s istim kamerama; niz s drugim kamerama dodaje se uz njega.
- **Vlastito spremanje aplikacije nikada ne prepišava trenutne postavke uređaja** svojom starijom kopijom, te sprema pojedinačna preusmjeravanja koja je spremio pozivatelj iz CLIa ili SDKa.
- **Pikselni format koji postavite je format koji projekt zadržava.** Format postavljen u postavkama niza (Array Settings), pomoću `--pixel-format` ili `connect_array(pixel_format=)`, uključujući 8-bitni, spremuje se i točno se ponovno povezuje. Ako nije ništa postavljeno, svaka se kamera povezuje s podrazumijevanim 12-bitnim postavkama (`BayerRG12`, `Mono12` na mono senzoru). Naziv formata označava njegovu dubinu bita; obitelj je senzora, tako da se bojna kamera nikada ne postavlja na mono format, niti mono kamera na Bayer format, a linija `[PixelFormat <serial>] ... written` to navodi kada je format mapiran. 8- Izvoz 8-bitnog snimanja je 8-bitan, uključujući i reflektanciju: ponovno skaliran i s oznakom njegovog `PixelScale`.
- Uređaj spojen bez projekta ne bilježi ništa.
- **Snimanje prati projekt stvoren nakon njega; otvaranje ga ostavlja s njegovim projektom.**Kada se stvori projekt — `create_project()` iz SDKa, `chloros-mcp` iz `create_project`-a ili**Novi projekt**aplikacije — ili dupliciran (opcija**Duplikat**u aplikaciji, koja otvara kopiju) dok snimanje još uvijek piše u projekt koji je bio otvoren prije, snimanje se nastavlja i ono što piše završava u novom projektu, na istom mjestu unutar njega (ispod). Otvaranje postojećeg projekta — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`,**Otvori projekt** aplikacije — ostavlja snimanje u tijeku s projektom u koji snima: ono se završava ondje, a njegove se datoteke pridružuju popisu datoteka tog projekta (aplikacija zaustavlja svako snimanje prije otvaranja projekta).
  - `array-record` i `array-burst`: videozapis, odnosno mapa burst s svakim okvirom, premješta se tamo u cijelosti kada se snimanje zaustavi (njezin `--duration`, Ctrl+C). `Burst dir` koji ispisuje je novi, a `--build` i `array-build-video` rade na temelju njega. Premještanje koje disk odbije ostavlja snimku tamo gdje je bila zapisana; povratlog završetka objašnjava zašto, kao i HTTP odgovor zaustavljanja (`relocation_error`). Sirovi kadrovi zaustavljenog burst-a i `.daq` pridružuju se popisu datoteka projekta u koji završi; video (`array-record`, video sastavljen iz burst-a) nije dodan na popis datoteka projekta.
  - `array-capture --continuous`: svako snimanje nakon promjene spremljeno je u novi projekt; one spremljene prije ostaju na svom mjestu.
  - `daq pool-record` i `project sensor log` (i snimanje aplikacije vlastitim senzorom svjetlosti): `.daq` se premješta u novi projekts `light_sensor` mapu kada se snimanje zaustavi i pridružuje se popisu datoteka tog projekta.

  Snimanje u mapu izvan otvorenog projekta (`-o` negdje drugdje) ostaje tamo gdje jest, a `lattice continuous` i `multi-stream` zadržavaju mapu koju su dobili.
- **U aplikaciji, snimke nakon novog projekta idu u njega.** Kreiranje ili dupliranje projekta održava svaku pokrenutu snimku i snimanje aktivnima, a svaka snimka napravljena nakon toga (Snimanje svega, Početak snimanja) spremljena je u novi projekt i pridružuje se njegovoj listi datoteka. Otvaranje postojećeg projekta u aplikaciji prvo zaustavlja sve njih.
- **Projekt se ne premješta dok se snima.**Premještanje otvorenog projekta u drugi radni direktorij (u postavkama aplikacije**Postavke → Radni direktorij**; nijedna naredba CLI-a ne premješta projekt, a HTTP klijent koji šalje zahtjev na `/api/move-project` to odbija s HTTP 409 `PROJECT_RECORDING` dok se snimanje niza ili burst, kontinuirano snimanje niza, posao `lattice continuous` / `multi-stream` ili snimanje senzorom svjetlosti piše u njega: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplikacija odbija na isti način, prije nego što zatraži mapu, dok su u tijeku vlastiti nalogi Capture All, Start Capturing ili snimanje sa senzora svjetla.

Ovo je odvojeno od toga kako su kamere ostavljene pri prekidu veze (vidi [Memorija kamere](#camera-memory----keep-settings----clear-settings)): projekt samostalno postavlja svoje kamere kada se otvori.

### Memorija kamere — `--keep-settings` / `--clear-settings`

Kada je niz povezan, njegove kamere preuzimaju postavke niza i tada ih možete mijenjati. Kada Chloros otpusti kamere — `array-disconnect`, kraj prolazne naredbe (`multi-capture` bez niza, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), zaustavljanje `lattice continuous`, kraj `lattice capture` kamere koju ne drži — vraća svaku u stanje u kojem je bila prije povezivanja: njegovo ožičenje okidača i sinkronizacijske linije, PTP i postavke kamere (potpuni popis nalazi se pod [Više kamera (privremeni sastanci)](#multi-camera-transient-sessions)).

Za korištenje kamera izvan projekta, dvije zastavice na bilo kojoj od tih naredbi zapisuju **vlastitu memoriju** kamere — stanje s kojim se pokreće, u bilo kojem softveru:

- **`--keep-settings`** sprema postavke svake kamere onakvima kakvima su ostale nakon sesije u svoju memoriju (UserSet2, postavljeno kao boot set). Kamere i dalje rade neovisno (`TriggerMode` `Off`), a žičanje sinkronizacijske linije ostaje kako je pronađeno. Kamera bez UserSet2 sprema u UserSet1; ovo računalo to pamti, pa se niz Chlorosa samopopravlja — što resetira UserSet1 koji se udaljio od osnovne postavke Chlorosa — ostavlja ga nepromijenjenim.
- **`--clear-settings`** resetira memoriju svake kamere na tvorničke postavke: osnovnu postavku Chlorosa, onu koju njegova samoliječiteljska mreža peče — tvorničke postavke s `TriggerMode` `Off`, `TriggerSource` `Line2`, automatsko ekspoziciju, kalibriranu ravnotežu bijele boje kamere s RGB senzorom sačuvana — u UserSet1, postavljena kao početni skup.
- To dvoje se ne može kombinirati. Svaka kamera dobiva redak: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` ili `SN:…: camera memory reset to the factory settings.`; odgovor `camera_memory` ima isto po kameri.

Aplikacija ne nudi te zastavice.

**Svaki izlaz vraća kamere na početno stanje.**Isti postupak vraćanja na početno stanje izvodi se pri svakom izlasku, a ne samo za gore navedene naredbe: `cam-disconnect` (i aplikacijina kamera**X**);**X** aplikacije na nizu ili na retku bilo kojeg člana niza (niz ne može nastaviti izvršavanje bez člana, pa se cijeli niz oslobađa); zatvaranje aplikacije (što pozadinskom sustavu daje do 20 s da vrati svaku kameru, niz i pokretanje `lattice continuous` prije nego što ga zaustavi; naredba koja je tada još uvijek u tijeku — prolazna naredba koja otvara svoje kamere ili čeka njihove PTP satove, jednokratni `lattice capture`, `power`, `white-balance` ili `presets --apply` — zaustavlja se na svojoj sljedećoj sigurnoj točki, prvo vraća svoje kamere, do 8 s, i odgovara HTTP 503 `SHUTTING_DOWN`; niz connect koji se još uvijek izvršava — `lattice array-connect`, connect aplikacije — zaustavlja se prije svoje sljedeće kamere i vraća kamere koje je otvorio, do 10 s, i odgovara HTTP 503 `SHUTTING_DOWN`; i onaj koji stigne tijekom zatvaranja ne otvara ništa: HTTP 503 `SHUTTING_DOWN`); i CLI naredba koja je ubijena ili čiji se terminal zatvori (SIGTERM / SIGHUP prekida njezin rad — i vraća svoje kamere — prije nego što izađe `143` / `129`, poput Ctrl+C). `lattice continuous`, `multi-stream` ili `align-stream` koji je izravno ubijen (SIGKILL), a koji ne može ništa zaustaviti, prekida se i njegove kamere se vraćaju pomoću pozadinskog sustava 30 s nakon svoje posljednje sinkronizacije. Backend koji se sruši ili se nasilno zatvori ne može ništa vratiti; isključenje i ponovno uključivanje vraća svaku kameru u njezinu memoriju, a sljedeće povezivanje s projektom Chloros ponovno povezuje kameru koju pronađe u pripravnosti.**Projekt sam postavlja svoje kamere.** Kada se niz poveže s otvorenim projektom, svaki član dobiva postavke koje je projekt spremio za njega — ekspoziciju, pojačanje ili cilj AE-a koji je na njemu zadržan (zadržan kao vaš, tako da ga automatska ekspozicija ne dira) i način balansa bijele boje (Chlorosova korekcija; vlastiti balans bijele boje kamere ostaje isključen) — što god memorija kamere sadrži.

### Pametni AE / Pametno snimanje

LATTICE nizovi provode kontinuirani AE u pozadini čim se povežu, ali svježe usmjerena scena treba trenutak da se uskladi. `array-capture --smart` je **praktično rješenje u paketu**: čeka da se AE stabilizira na svakoj kameri u nizu, a zatim pokreće snimanje. Koristite ga kada mijenjate scene usred sesije.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Postupak stabilizacije je prema zadanim postavkama konzervativan: vremensko ograničenje od 5 s, prozor stabilnosti od 1,5 s, tolerancija raspršenosti ekspozicije od ±5 %. Podesite ga putem SDKa (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) ako želite drugačije ponašanje od automatizacije.

#### Kako se ponaša automatska ekspozicija

- **Prvo ekspozicija, zatim pojačanje.** Kada je pojačanje na automatskom načinu rada, pojačanje ostaje na 0 dB dok ekspozicija ne dosegne najdužu dopuštenu vrijednost.
- **Pri ručnoj ekspoziciji, pojačanje ostaje na automatskom** dok ne promijenite pojačanje ili ga ne prebacite na ručno. Kada se vratite na automatsko, ograničenje pojačanja je isto kao i prije.
- **Mali odsjaji ne zamračuju scenu.** Odbljesak do 1 % kadra se zanemaruje, a kod panela poznatog kalibracijskog cilja do 0,5 % panela.
- **Gornja granica ekspozicije DAQ-a primjenjuje se samo dok je kalibracijski cilj u vidokrugu.** Ona sprječava presijecanje bijelog panela; kada se panel ne vidi, kamera ekspozira prema sceni.
- **Stalno svjetlo ostaje stabilno, a promijenjeno svjetlo se stabilizira u sekundi.** Pri stalnom svjetlu ekspozicija i njezina granica ostaju nepromijenjeni, a nakon velike promjene svjetla prikaz se stabilizira unutar nekoliko sekundi. Presijecanje više ne ograničava ekspoziciju zauvijek, a promjena ROI-ja ponovno pronalazi ekspoziciju.
- **Cilj svjetline ostaje unutar dosega.** Ne raste dok je kamera na svojoj granici ekspozicije i njezin pojačanje ne može porasti, pa se okvir ne preosvjetljuje kad se svjetlost vrati.
- **Kamera čije automatsko izlaganje prestane reagirati drži se na razini za svjetlo u kojem se nalazi.** Na jakom suncu kamera bi mogla pasti na najkraće izlaganje i ostati tamo s tamnim kadrom, njegove postavke sve pokazuju normalno. Chloros prati kadar, drži tu kameru na razini na kojoj je bila za to svjetlo i pušta je kada se svjetlo promijeni. Tamni ili prekriveni senzor svjetla ne ometa to.

### Razine snimanja (zadano za `all`)

Od ove verzije, `lattice capture`, `lattice multi-capture` i `lattice array-capture` **podrazumijevano su `--processing all`** — jedna spremljena datoteka po vrsti izvoza koja se primjenjuje na svaku kameru, što odgovara ponašanju GUI-ja "Capture All". Razine su:

| Razina | Izlaz | Primjenjuje se na |
| --- | --- | --- |
| `raw` | Jednokanalni Bayer (monokamere: pojedinačna traka) izravno sa senzora. | Sve kamere. |
| `debayered` | 3-kanalni boja, spremljen u redoslijedu kanala **RGB**(TIFF s fotometrijskom interpretacijom RGB; PNG/JPEG također se otvara kao RGB): dark offset (DSNU) kalibracijskog paketa oduzet je od sirovog, zatim OpenCV-ov rub-demosaicizacija s obzirom na rubove — bez LUT-a za linearnost, ravnog polja ili unmixa (mono kamere: jednokanali u sivim tonovima). Okviri bez bajera u memoriji koje vraća PythonSDK (`grab()`, `frame_stream()`) su u**BGR** redu, OpenCV-ov redu; samo je spremljena datoteka u boji RGB. | Sve kamere. |
| `radiance` | float32 W/m²/sr/nm putem cijelog radiometrijskog lanca. | Multispektralne (M3C/M3M) samo — **preskočeno za RGB-kamere s filtrom**. |
| `reflectance` | uint16 ρ (`32768` = 1.0), spremne za Pix4D. | Samo za višespektralne kamere i **samo kada je DAQ povezan + kamera kalibrirana**; u suprotnom se preskače. |
| `display` | Cijeli lanac za pregled u GUI-ju (CCM + WB + gamma prema profilu kamere). Spremljeno u mapu `preview/`. `lattice capture` također prihvaća naziv `preview` za nju; `array-capture` / `multi-capture` prihvaćaju samo `display`. | Sve kamere. |

Proslijedite jednu razinu da biste spremili samo tu jednu (`--processing debayered`). Kad zatražite `all`, razine koje se ne odnose na odrečenu kameru preskaču se (i izvještava se), a ne da se prijavi pogreška — nevezana ili nekalaribrana kamera i dalje dobiva `raw` / `debayered` / `preview`.

#### DAQ sprema očitanje snimke

Kad god je DAQ povezan s kamerom ili nizom, ili je objedinjen, očitanje snimke prema dolje koje se koristi zapisuje se u **`.daq`** datoteku u podmapu `daq/` mape za snimke, sa svojim `.match.json` (koliko je očitanje bilo udaljeno od ekspozicije) i njegov `.cameras.json` (čije je očitanje), te se izvještava na `DAQ:` liniji. To vrijedi za svaku razinu, uključujući i sirovu ili debayered snimku, pa se svaka snimka može naknadno obraditi u radijancu, reflektancu ili indeks. To je ista datoteka, s istim imenom i sidecar datotekama, bez obzira je li snimanje došlo iz GUI-ja, CLI-a ili SDK-a. Ako uopće nema DAQ očitanja, ne postoji `.daq`, a `multi-capture` bez priključene matrice (privremeno povratno rješenje) ne sprema nijedno.

| Zastavica | Učinak |
| --- | --- |
| `--no-daq` | Ne spremite očitanje za snimanje čije razine ne koriste očitanje (`raw`, `debayered`). Razina `radiance`, `reflectance` ili `display` i dalje je zapisuje, jer ponovni uvoz treba očitanje s kojim je proizvod izrađen. |
| `--daq-tolerance-ms MS` | Koliko daleko od ekspozicije očitanje može biti i još uvijek se računati unutar tolerancije (zadano 500). Samo se presuda mijenja (`.match.json`-ov `in_tolerance` i `tolerance_ms`, reflektancijini `DlsInTolerance`), nikada korištena očitanja. Vidi [DAQ očitanje daleko od ekspozicije](#a-daq-reading-far-from-the-exposure). |
|
| `--force-daq` | Stariji način zahtijevanja `.daq` na snimanju samo u sirovom formatu. Sada je to zadano, a `lattice capture` i `array-capture` i dalje prihvaća zastavicu (`multi-capture` ne). `--no-daq` ima prednost nad njom. |

Koji DAQ: onaj vezan uz kameru ili niz (odabir senzora svjetla u GUI-ju, spremljena veza projekta ili `chloros-cli daq pool-bind`), inače najsvježiji koji back-end ima. Sirovi burst (`array-burst`) zadržava vlastiti `.daq` za svako pojedinačno očitavanje u mapi burst kao i prije.

**PNG i JPEG imaju 8 bita po kanalu, stoga uzimaju samo `display` / `preview` i indeksne proizvode.** `--format png` ili `jpg` s razinom koju ne mogu podržati odbijaju se prije nego što se išta snimi, navodeći razinu i TIFF kao format koji je podržava:

- `radiance` (float32) i `reflectance` (uint16, `32768` = 1.0) — uvijek, stoga je i `--processing all` (zadano) s `--format png`/`jpg` odbijen: koristite `--format tiff` ili `--levels display`.
- `raw` i `debayered` iz kamere koja streamuje više od 8 bita (12-bitni M3C / Mono12 M3M okviri) — CLI ne može vidjeti dubinu bita kamere, pa pozadinski sustav odbija te zahtjeve (HTTP 400). Jedini-okvir `lattice capture --processing debayered` hvata 8-bitni okvir za PNG/JPEG i nije odbijen, kao ni `multi-capture` bez trajne matrice (8-bitno debayered).

Svaka ruta snimanja odgovara na isti način (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, i HTTP rute iza njih.

### Kako izgleda mapa snimki

Svaka vrsta izvoza smješta se u svoju **podmapu**unutar `-o`, tako da višerazinsko snimanje nikada ne miješa vrste. Skica prikazuje**zadano snimanje jedne kamere pomoću `lattice capture`**u**`all`** — nekoliko razina po kameri, pa svaka datoteka ima nastavak `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Kako se nazivi datoteka razlikuju:

- **Polje kamere — serijski broj, zatim filter:** svaka snimljena datoteka naziva se imenom kamere koja je snimila, kao `SN<serial>-<filter>`, gdje je `<filter>` dio filtera modela kamere: `SN261902142-F450` za `M3M-L87-F450`, `SN213800234-FRGN` za `M3C-L87-FRGN`, `SN214701292-FRGB` za RGBnu kameru. Filtar spaja serijski broj s `-`, ne `_`, pa ime ima ista polja odvojena točkom i zarezem kao i prije (prefiks, datum, vrijeme, milisekunde, kamera, razina). Kamera čiji se model ne može pročitati (izbrisan DeviceUserID) naziva se samo `SN<serial>`. Snimke iz ranijih izdanja nemaju filtar u nazivu; Chloros čita oba.
- **Prefiks:** `lattice capture` imenuje datoteke `capture_…`; `array-capture`, `multi-capture` s povezanim nizom, i `project capture` / `burst` / `stream` imenuje datoteku imenom niza `sync_<ts>_SN<serial>-<filter>…`.
- **Sufiks razine samo kod više razina.**Kamera koja sprema**više**razina u jedno snimanje (`all`, ili `--levels a,b`) dodaje `_<level>`. Kamera koja sprema**jednu** razina zadržava jednostavno ime, bez nastavka — `--processing reflectance` zapisuje `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — tako da skripte napisane za snimke s jednom razinom i dalje rade. Razina se i dalje navodi u nazivu mape.
- **Maske za zaključavanje slijede naziv proizvoda:** `<product name>_clampmask.tif`. Jednoručna snimka reflektancije zapisuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; snimka `all` zapisuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Proširenje** slijedi `--format`: zadani `tiff` daje `.tiff`, `--format tif` daje `.tif` (maske za pričvršćivanje su uvijek `.tif`).
- **Jedan izuzetak:** `lattice capture --processing debayered` (ili `--levels debayered`) uzima brzi pojedinačni- putanju okvira i piše `capture_<ts>_SN<serial>-<filter>.tiff` izravno u `-o` — bez podmapiranja.
- **Streamovi i burstovi** nose isto polje: `lattice continuous` piše `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream` `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` i sirovi burst `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` je vremenski žig snimanja, a `<serial>` serijski broj kamere, tako da jedna sinkronizirana grupa dijeli
vremenski žig među kamerama. **Obratite pozornost na jednu asimetriju:** razina `display` pohranjuje se u mapu
naziva `preview/`, dok datoteke više razina zadržavaju `_display` u nazivu — mapa i sufiks razlikuju se samo za tu razinu. Nezavisne razine koriste mapu vlastitog naziva, a ako se podmapa ne može stvoriti, datoteka se zapisuje u korijensku mapu izlaza umjesto da se izgubi.

**Ponovna obrada mape snimki:**usmjerite `chloros-cli process` na**korijen snimki** (`output/`). `process` obično uvozi samo mapu koju nazovete, ali kada ta mapa ne sadrži
slike, a ima i podmapu u koju se automatski ulazi — tako da se svi nivoi i podmapa
`daq/` učitavaju odjednom. Svaki nivo snimanja uvozi se kao jedna slika, a ostali nivoi su dostupni kao načini rada, a ne kao jedna slika po nivou.

Izravno imenovanje **podmappe razine** (npr. `output/raw/`) također radi. Time se `daq/`
zanemaruje, stoga kopirajte ili usmjerite DAQ očitanje uz njega kada ponovno- izvedite radiometrijski proizvod iz `raw/` — inače usklađivanje vremenskog žiga nema na što se primijeniti.

**Obrada uvijek započinje s `raw`.** Unutar svakog snimanja, sirovi kadar je izvor u cijevnoj liniji;
`debayered`, `radiance`, `reflectance` i `preview` dolaze kao prikazivi načini rada, ali se nikada ne vraćaju kroz pipeline. Ponovna obrada izvedenog proizvoda ponovno bi primijenila vignetu, CCM i radijacijsku matematiku koja je već ugrađena u njegove piksele, pa Chloros odustaje od dvostruke obrade. Dvije posljedice vrijedne spomena:

- Renderiranja `index/` i `composite/` **nikada** obrađene. To su izlazi, a ne snimke — renderiranje LUT-a za NDVI nema smislenu interpretaciju radijance.
- Mapu sa snimkama izvez­enu **bez** `raw` (npr. `array-capture --processing reflectance`) ima
  nije legitiman izvor cijevi. Ta snimanja se normalno uvoze i prikazuju, ali `process` ih preskače
  i o tome obavještava:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ako doista trebate provući izvedeni proizvod — sesiju s čvorišta snimljenu pomoću
  `demosaic` uključen, ili u naslijeđenoj mapi — `--input-level {raw,debayered,processed}` prisiljava ulaznu točku i poništava preskakanje. Ta zastavica je namjenski izlaz; `auto` (po zadanom) nikada ne obrađuje snimku koja nema raw podatke.

**Oznake kalibracije: samo na `raw`.** Datoteka `raw` snimke sadrži paket kalibracije kamere: korisni teret `CalibrationJSON` i oznake kompatibilne s MicaSenseom (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), koji opisuju sirove vrijednosti senzora. Svaka datoteka drugog sloja (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) i svaki `lattice continuous` / okvir `multi-stream` nosi identifikaciju kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) i vlastiti zapis okvira, `XMP-mapir:FrameJSON` (ekspozicija, pojačanje, temperatura, format piksela, vrijeme snimanja, DAQ podudaranje), budući da svaki proizvod `process` nosi svoj identitet. `process` pronalazi kalibraciju takve datoteke po serijskom broju i `CalBundleSha256` u predmemoriji kalibracija Chloros, preuzimajući paket jednom ako ovo računalo je nema.

#### Datoteke izrađene od više kamera

Datoteka izrađena od pojaseva više od jedne kamere — višopojasni sloj (`align-apply`, `align-stream`, `index --live --save-multiband`), živi indeks napravljen od više kamera (`index.tif`, `index_LUT.png`), slika `composite/` snimanja nizom — navodi svaku kameru koja je sudjelovala:

- **Popisi po kameri, poretkom bendova** (kompozit: pozadinski sloj, zatim prednji; indeksni sloj, kamere čijih su bendova): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibracijski paket kroz koji su prošle trake svake kamere, inače — stack `align-apply` / `align-stream`, čije trake ne prolaze kroz nijedno — paket kamere Chloros je keširao, onaj koji njegove datoteke snimanja nazivaju), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralelne liste, stavka *n* svake opisuje istu kameru; `-` gdje kamera nema vrijednost (nema kalibracijski paket, nepoznat objektiv). Svaka kamera je **jedan** element: kada njezin paket ima nekoliko vrijednosti za jedno polje — certifikate halogenih lampi paketa — one se spajaju pomoću `;` (`F-1479-1;F-1480-1;F-1481-1`), nikada zarezom, tako da se popis čita kao jedna stavka po kameri, bez obzira na način ispisivanja.
- **EXIF `SerialNumber`** sadrži iste serijske brojeve, spojene zarezom (`214000535,214000551`).
- Ne nosi **ni** jedan identitet pojedinačne kamere (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): one navode jednu kameru i njezin paket, a `process` bi iz njih riješio kalibraciju datoteke. Indeks izračunat iz pojaseva nekoliko kamera (`index.tif`, `index_LUT.png`) ima EXIF `Model` `LATTICE` (oprema), a ne model jedne kamere `LATT-…`.

Datoteka pojaseva **jedne** kamere — po pojasu TIFF (`--per-band`), `align-apply --per-camera` kadar, hrpa ili živi indeks jedne kamere — sadrže vlastite oznake identiteta te kamere, kao i svaka druga datoteka snimanja. datoteke stacka, po pojasu i po kameri za `align-apply`, kao i svaki stack za `align-stream` i njegove datoteke po pojasu, nose vrijeme snimanja svoje grupe (ekspozicija referentne kamere, UTC), i okvir `--per-camera` monokamere je jedna ravnina, u `--bit-depth` kao i hrpa.**Orijentacija kamere (preokretanje / rotacija).** Postavka orijentacije kamere (GUI-jev Mirror H /
Postavka ogledala V / rotacije određuje što snimanje sprema:

- **`raw` i `debayered` spremaju se točno onako kako ih senzor pročita**, bez obzira na postavku. A
  Bayerov mozaik okrenut ili preokrenut imao bi drugačiji uzorak filtera boja, a mape kalibracije (pomak tame, ravno polje) nalaze se na vlastitoj mreži senzora.
- **`preview`/`display`, `radiance`, datoteke `reflectance` i `index/` su orijentirane** — okrenute jednom, nakon sve kalibracijske matematike, tako da se otvaraju onako kako je kamera bila okvirena. Njihove
  Datoteke `clamp_masks/…_clampmask.tif` nalaze se na istoj mreži.
- **Svaka datoteka snimanja bilježi postavku** u XMP-u: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, u smjeru kazaljki na satu, nakon preokreta) i `ChlorosOrientationApplied` — `false` na sirovim i dekodiranim (nativnim pikselima),
  `true` na orijentiranim proizvodima. `ChlorosProcessing` navodi operacije primijenjene na piksele te datoteke (`none` kada su izvorni). To uključuje svaki kadar koji `lattice continuous` streamuje (vlastito podešavanje kamere, neprimijenjeno) i svaki kadar `lattice multi-stream` (identitet: njegovi članovi su nativni, kao kod snimanja nizom). `lattice align-stream` / `align-apply`
  stacks bilježe postavku matične kamere, neprimijenjenu: njihove trake su izobličene u kadar
  matične kamere, ali nisu rotirane.
- **`process` primjenjuje zabilježenu orijentaciju** kada obrađuje sirovu datoteku: prvo u koordinatama senzora obrađuje pomak u tami, ravninu, demosaic i odvajanje, zatim iskrivljenje poravnanja niza, a zatim
  orijentacija se primjenjuje na kraju — tako da proizvodi za pretpregled, radijancu, reflektancu i indeks ispadaju točno kao i kod snimanja uživo, te se bilježe `ChlorosOrientationApplied=true`. Nekalibrirana zadana opcija
  proizvodi (`Sensor_Response_Images`, `Vignette_Corrected_Images`) orijentirani su na isti način, nakon
  ravnom polju. Debayered proizvod ostaje u izvornom formatu.
- **Preglednik slika Chloros prikazuje raw ili debayered datoteku okrenutu prema njezinom zapisu** (samo za prikaz — datoteka se ne mijenja), pa izgleda kao i njezini proizvodi; minijatura, prikaz u punoj veličini, prikaz čitanja kursora i sandbox indeksa/LUT-a koriste isti smjer okretanja. Datoteka bez zapisa i proizvod (koji je već orijentiran) prikazuju se onako kako su pohranjeni. Izvoz u sandbox bilježi orijentaciju koju nose njegovi pikseli (`ChlorosOrientationApplied=true`; identitet za sloj bez bilješke).
- **Snimke bez zapisa obrađuju se kao nativne** (nepromijenjeno ponašanje). Prije verzije 1.3.0, snimanje s jedne kamere s postavljenim okretom ili rotacijom samo je obrađivalo svoje raw i debayered datoteke; Chloros ne može prepoznati takvu već-obrnute raw datoteke iz nativne i ne nagađa, pa ih obrađuje kao pohranjene. Kalibracijske mape nalaze se na vlastitoj mreži senzora, stoga za takvo snimanje upotrijebite onu snimljenu s verzijom 1.3.0 kada je njezina kalibracija važna.
- **Nizovi prema zadanim postavkama ne okreću nijednu datoteku člana** (poravnanje, deformacija i `Alignment*` XMP nalaze se u vlastitim, neokrenutim okvirima kamera) i bilježe identitet. `array-capture --orient` okreće proizvode svakog člana, nakon transformacije poravnanja: prema orijentaciji **glavne** kamere kada su transformirani u njezin kadar (tako da su članovi i dalje sukladno poravnati), a inače prema orijentaciji vlastite kamere. Svaka datoteka bilježi orijentaciju okvira u kojem se nalaze njezini pikseli — sirovu, vlastite kamere — i sirovu koja nosi transformaciju poravnanja također bilježi `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: majstorov, za proizvod deformiran
  u okvir majstora. `process` okreće svaki proizvod prema onome što pripada njegovim pikselima:
  poravnati kada deformira sirovi (`--array-alignment`, zadano — uživo poravnati
  proizvoda snimanja), vlastitih kamere kada to ne čini (`--no-array-alignment` — za snimanje uživo
  bez poravnanja). Ista opcija postoji za `array-burst`, `project capture` / `burst` /
  `stream`, recepte i SDK.

### Preskočena snimanja u nizovima s miješanim filtrima

Kada pomiješate kamere u bojiRGB i multispektralne kamere u jednom nizu, `array-capture --processing radiance` (ili `reflectance`) pohranjuje multispektralne kadrove i **preskače** RGBne kamere — radijansa po Bayerovoj matrici nema smisla za širokopojasni senzor. CLI ispisuje svaku spremljenu datoteku (s razinom izvoza), svaki zapisani `.daq` i svako preskakanje eksplicitno, tako da broj datoteka nije iznenađujući:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeni razloga preskakanja slijede obrazac `<level>-not-applicable-to-rgb-cam`. Kamera čiji paket kalibracije nije učitan preskače `radiance` / `reflectance` s `<level>-needs-calibration (connect the camera so its cal pack loads)`, i kod odsutnosti upotrebljivog DAQ očitanja reflektancija preskače s `reflectance-skipped-no-dls`. Reflektancija također može preskočiti s `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, i s `dls-uncalibrated-band-<nm>` kada se pojas nalazi uglavnom izvan radiometrijski kalibriranog raspona senzora svjetla DAQ (~374–974 nm) — među isporučivim SKU-ovima samo F988, čiji je podržani tijek rada rad sa zrcalačnom pločom.

Koristite `--processing debayered` (ili `display`) za uključivanje svake kamere bez obzira na vrstu filtera, ili zadani `all` za dobivanje svake primjenjive razine po kameri u jednom koraku.

### DAQ očitanje daleko od ekspozicije

Svako očitanje koje snimanje sprema u `.daq` je DAQ očitanje usklađeno s tom ekspozicijom, a `.match.json` pored njega bilježi koliko su bila udaljena (`gap_ms`) i je li to unutar tolerancije (`in_tolerance`, 500 ms, osim ako snimanje nije postavilo `--daq-tolerance-ms`). Očitavanje izvan tolerancije se i dalje koristi, tako da nijedan kadar nije izgubljen, i označava se: XMP datoteke reflektancije sadrži `DlsInTolerance=False` i `DlsMatchGapS`. `lattice capture`, `multi-capture` i `array-capture` također ga ispisuju ispod `.daq` na koji se odnosi, s DAQ-omkvar protoka DAQ-a kad se objasni:

**Kako se očitanje povezuje s ekspozicijom.** Snimanje čeka prvo očitanje DAQ-a nakon ekspozicije kako bi moglo spojiti očitanja s obje strane (`method` `interpolated`). DAQ-E šalje očitanje svakih 0,4 s po danu i svakih 1–2 s u zatvorenom prostoru, pa se vrijeme čekanja zbraja na otprilike jedan i pol tih intervala za naredbu, a ne za ekspoziciju. Na DAQ koji je prestao slati očitanja se ne čeka. DAQ-E očitanje je prosjek svjetlosti tijekom intervala od prethodnog očitanja, stoga se postavlja na sredinu tog intervala: `gap_ms` mjeri se od ekspozicije do te sredine, a `bracket_stamps_ns` daje vlastite vremenske oznake očitanja. `array-capture --continuous` čeka bez usporavanja snimanja: njegove sirove datoteke zapisuju se odmah, a njihove `.daq` kada stigne očitanje. S `--interval`, svaki prolaz čeka kao jedno snimanje.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Kada iz tog očitavanja nije napisana datoteka reflektancije, prva linija umjesto toga završava `saved anyway; its .match.json says in_tolerance: false`.

Prikupljanje podataka se i dalje završava `0`: svaka je datoteka zapisana i navodi što sadrži. DAQ čiji je tok zaustavljen nastavlja isporučivati svoje posljednje očitavanje, stoga ga provjerite pomoću `chloros-cli daq pool-list` prije nego što se oslonite na reflektanciju tog prikupljanja. Backend ponovno povezuje DAQ-E iz zajedničkog fonda koji je prestao isporučivati (vidi [DAQ-E koji prestaje isporučivati](#a-daq-e-that-stops-delivering)). Mjerenje unutar tolerancije ispisuje samo svoju liniju `DAQ:`, osim ako nije sačuvano mjerenje (dolje).

**Očitavanje zadržano nakon što je njegov DAQ nestao.** Kada nijedan DAQ ne može isporučiti očitavanje za ekspoziciju (vezani DAQ nije dostupan ili nijedan DAQ iz skupa nema očitavanje, npr. nakon `daq pool-disconnect`), snimanje ponovno koristi posljednje očitanje koje je imao Chloros, a njegova `.match.json` opisuje to očitanje za ovu ekspoziciju: `method` `retained`, `gap_ms` od ove ekspozicije do uzoraka na kojima je očitavanje izvršeno, `in_tolerance` od tog razmaka i `retained_from_epoch_ns`, ekspozicije kojoj je očitavanje prvi put usporedeno. Stoga snimanje izvršeno dugo nakon što je DAQ nestao ispisuje gornje i donje linije one još jednom, što se ispisuje čak i dok je očitanje još unutar tolerancije:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Njegov `.daq` i dalje naziva DAQ koji je napravio očitanje, s kalibracijom i kapom pod kojima je to očitanje napravljeno, tako da pri ponovnom uvozu znamo koji su DAQ, kalibracija i kapa to proizveli.

### Datoteke koje se ne mogu zapisati

Kamera čija se datoteka ne može zapisati — pun disk, pogreška dozvola — ne sprječava spremanje datoteka drugih kamera. Svaka naredba za snimanje (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; i `capture`, `burst` i `stream` recepta ispisuju svaku datoteku koja je neuspjela na stderr, zatim **izlazi `1`** iako je ostatak sačuvan:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Svaki fajl koji snimanje duguje je pokriven, jediniuključujući debayered -frame `lattice capture --levels debayered`. Čitanje `.daq` čija se sidecar datoteka o podrijetlu `.match.json` nije mogla zapisati navodi se kao `[match]` (sam `.daq` je sačuvan).

Na punom disku, ponavljajuće snimanje (`array-capture --continuous` / `--interval`, `project burst` / `stream`, recept `burst` / `stream` (sam `.daq` je sačuvan).  Na punom disku, ponovljeno snimanje (`array-capture --continuous` / X) **zaustavlja se na tom prolazu** umjesto da za ostatak izvođenja ne piše ništa na isti disk. Svaki drugi neuspjeh prijavljuje se pri svakom prolazu u kojem se dogodi, a izvođenje se nastavlja do kraja, a zatim izlazi `1`. `array-capture --continuous` (kontinuirano snimanje pozadinskog sustava) ne navodi nijednu datoteku, uključujući neuspjele: broji ih u svojim linijama napretka i sažetka (`N failed`, `N file(s) failed to write.`) i izlazi iz `1` — vidi [Kontinuirani način rada](#continuous-mode---continuous).

Snimatelji — `array-burst` i `array-record` — prijavljuju vlastite greške. `array-burst` zaustavlja se kada se pogon napuni ili neposredno prije toga. Oba nastavljaju s radom nakon bilo kojeg drugog neuspjelog zapisa. Kada bilo koji od njih prijavi pogrešku, ona se ispisuje nakon brojača, a naredba **izlazi `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` umjesto `Disk full:` znači da je burst stao prije popunjavanja disk. Snimljeni okviri prije zaustavljanja se zadržavaju. `array-burst --build` preskače izradu kada ponestane prostora, jer bi videozapisi otišli na isti disk. Prvo osigurajte slobodan prostor, a zatim pokrenite `array-build-video --burst-dir …`. Nakon bilo koje druge pogreške izrada se i dalje nastavlja.

`array-record` koji nije zapisao nijedan kadar također je neuspjelo snimanje (`Recorder error: no-frames: the combined view published no frame to record …`, izlaz `1`) — vidi [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Načini snimanja, snimači i reprocesiranje izvan mreže

Svi oni rade na **trajnoj nizu** (prvo pokrenite `array-connect`). Oni odražavaju panel za snimanje GUI-ja.

### `array-capture` načini rada

`array-capture` je naredba s jednim parametrom s četiri načina rada zatvarača i skupom prekidača za izvoz:

| Način | Zastavica | Ponašanje |
| --- | --- | --- |
| **Jednostruko** *(zadano)* | (ništa) | Jedna sinkronizirana grupa snimanja, zatim izlaz. |
| **Kontinuirano** | `--continuous` | Snimanja zaredom sve dok se ne pozove `Ctrl+C`, `--count N`, `--duration S`, ili do pune particije diska. Pokreće vlastiti kontinuirani snimanje pozadinskog procesa (kontinuirani način rada GUI-ja) — pogledajte [Kontinuirani način rada](#continuous-mode---continuous) u nastavku. |
| **Interval** | `--interval S` | Jedan prolaz svakih `S` sekundi (mjereno od početka svakog prolaza), isti okviri. |
| **Najbrže** | `--fastest` | Sirovo-samo + dodijeljeno DAQ očitanje + kompozit kombiniranog indeksa; preskače matematiku radijancije/reflektancije/prikaza kako bi se kadar brzo dobio. Podrazumijeva `--processing raw --force-daq`. Ponovno obradite spremljeni `.daq` u kalibrirane proizvode kasnije. |

Prekidači za izvoz (kombinirajte s bilo kojim načinom rada; svi dijele GUI/SDK krajnju točku):

| Zastavica | Učinak |
| --- | --- |
| `--array-id ID` | Niz za snimanje (izostavite kada je priključeno samo jedno). |
| `-o, --output DIR` | Izlazna mapa (zadano `output`, u mapi iz koje pokrećete naredbu). |
| `--format {tiff,tif,png,jpg,jpeg}` | Format datoteke (zadano `tiff`). PNG / JPEG koriste samo `display` i indeks za iscrtavanje; pogledajte [Capture Export Levels](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Jedna razina izvoza (`raw`, `debayered`, `radiance`, `reflectance`, `display`), ili `all` (zadano). |
| `--levels L1,L2,…` | Eksplicitni podskup tipova izvoza (npr. `raw,radiance,reflectance`); **nadjačava `--processing`**. |
| `--aligned` / `--no-aligned` | **Zadano: poravnato, kao što je snimanje iste niske u aplikaciji Chloros**(njezina postavka**Poravnato** za snimanje niza po nizu, uključena osim ako je ne isključite). Svaki ne-raw izvoz svakog člana (`debayered`, `display`, `radiance`, `reflectance`, indeksni render) iskrivljuje se u kadar glavne kamere i obrezuje na zajednički preklop članaka, tako da su datoteke pikselski registrirane i izlaze iste veličine, s obrezom i oznakama `Alignment*` koje aplikacija postavlja — manje od senzora (ili ROI) prema pomacima članova; unutar ROI-ja transformacija se ponovno primjenjuje na prozor. `XMP-mapir:AlignmentJSON` poziva `"applied": true` i vraća `crop_roi`. Raw ostaje neizobličen u izvornim dimenzijama, ali nosi transformaciju u metapodacima (`"applied": false`). `--no-aligned` sprema svaku datoteku u izvornim dimenzijama, neizobličenu. `--aligned` je zadano razrađen. Bilo da je iskrivljenje primijenjeno ili ne, svaka datoteka člana bilježi transformaciju `Alignment*` svoje kamere čim niz dobije profil poravnanja — osim kamere s kojom se profil ne poravnava (izostavljene ili prisilno postavljene na identitet zbog neuspješnog rješavanja) i referentne kamere kada nije poravnata nijedna druga kamera: one nemaju oznake `Alignment*`. Ako na nizu nema profila, snimanje se sprema neporavnato, a CLI ispisuje razlog na `Alignment:` liniju, kao što to čini u svakom drugom slučaju kada se warp nije mogao primijeniti (članovi u različitim rezolucijama, niz koji nije sinkroniziran kabelom, kadar na koji se profil ne može mapirati). Automatsko poravnanje niza stvara profil čim se njegova ekspozicija stabilizira; `align-calibrate` stvara profil na zahtjev. |
| `--orient` | Okrenite **proizvode**svakog člana — `display`, `radiance`, `reflectance`, renderiranje `index/` i `composite/` — prema orijentaciji kamere, nakon warpiranja poravnanja: proizvod iskrivljen u okvir majice (zadano) pomoću**orientacija majstora, tako da su članovi ko-registrirani; neusklađena (`--no-aligned`, ili snimka na koju se deformacija nije mogla primijeniti) prema orijentaciji vlastite kamere. **Zadano: isključeno — svaka datoteka člana ostaje u izvornom obliku**, kao što je uvijek slučaj kod snimki nizova. `raw` i `debayered` ostaju u izvornom okviru u oba slučaja. Svaka datoteka bilježi orijentaciju koju njeni proizvodi nose (vidi [Orijentacija kamere](#what-a-captures-folder-looks-like)); XMP `Alignment*` ostaje transformacija izvornog kadra, primijenjena prije orijentacije, pa `process` (deformacija, zatim orijentacija) reproducira izvorni proizvod. |
| `--index` / `--no-index` | Spremi / preskoči preklapanje indeksa vegetacije po kameri gdje je konfigurirano. Zadano: prikaži ga. |
| `--no-daq` | Preskoči pomoćnu datoteku `.daq` kada nijedna odabranih razina ne koristi DAQ (samo snimanje sirovih podataka). Po zadanom se očitanje sprema, tako da se okviri mogu naknadno obraditi u offline načinu za reflektanciju/indeks. `--force-daq` se i dalje prihvaća (zadano). |
| `--daq-tolerance-ms MS` | Tolerancija podudaranja DAQ-a za ova snimanja (zadano 500 ms). |
| `--smart` | Pričekajte da se AE stabilizira na svim kamerama prije pokretanja (pogledajte [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresija piksela formataTIFF. `none` (zadano od verzije 1.3.0; verzija 1.2.0 je koristila `deflate`) = nekomprimirano — piksele senzora točno onako kako su očitane, ~6,3 MB po kadru pune rezolucije, i najbrža stabilna brzina; `deflate` = zlib L1 bez gubitka + horizontalni prediktor, ~4,1 MB po kadru, sporije. Oba su bez gubitka i identično sečitaju pri uvozu. |

> **TIFF s jednim pisanjem + model s održavanjem brzine.**Svaka se datoteka zapisuje**jednom**: pikseli, paket XMP, IFD0 Make/Model/CameraSerialNumber i oznake EXIF `ExposureTime` / `SerialNumber` u jednom prolasku, bez ponovnog prepisivanja. `--continuous` snima uzastopne sinkronizirane grupe — svako snimanje uzima grupu nakon prethodne, uključujući i onu objavljenu dok se prethodno snimanje još spremalo — stoga radi brzinom niza. DEFLATE drži GIL u Pythonu, pa komprimirano pisanje**ne**paralelizira pisanje u pojedinačnim nitima kamere; kontinuirano snimanje pune rezolucije brzinom senzora zahtijeva nekomprimirano pisanje (po zadanom)**i** disk klase NVMe (~600 MB/s kontinuirano za niz od 9 kamera pri ~10 fps). Isti parametar dostupan je kao `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Kontinuirani način rada (`--continuous`)

`array-capture --continuous` (bez `--interval` ili `--smart`) predaje izvršavanje **vlastitom kontinuiranom snimanju pozadine** — sesiji koju pokreće kontinuirani način rada GUI-ja. Backend samostalno sprema svaku sinkroniziranu grupu, s dvije snimke u tijeku i bez zahtjeva s CLIa između grupa, pa tako prati brzu matricu u kojoj zahtjev po snimci zaostaje (izmjereno 73–77 snimaka/s na matrici od 9 kamera s ROI-jem od 512 piksela). Svaki gumb za izvoz iznad primjenjuje se na svako snimanje.

- **Ograničenje:** `--count N` zaustavlja se nakon N snimanja (jedno snimanje = jedna sinkronizirana grupa: datoteke svakog člana); `--duration S` nakon S sekundi. Bez njih, radi dok se ne dođe do `Ctrl+C`, punog diska, pogreške ili uklanjanja niza. `Ctrl+C` zaustavlja sesiju pozadinskog sustava, zatim ispisuje sažetak.
- **Napredak** svakih 2 s, s `, N failed` i `, N groups skipped` dodanim kada su ne-nulte (preskočena grupa je ona koju je niz objavio, a koju nijedno snimanje nije obuhvatilo):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Nema popisa po datotekama.** Pokretanje završava jednom sažetom linijom — snimkama, brzinom, datotekama i izlaznom mapom — nakon čega slijedi `N file(s) failed to write.` / `N group(s) skipped.` ako je vrijednost ne-nula:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Zaustavlja se** pri punom disku s `Continuous capture stopped: the drive is full.`, i pri pogrešci ili nestanku niza s `Continuous capture stopped: <reason>.` (oba na stderr).
- **Status izlaska:** `0` kada su snimke spremljene i svaka datoteka je uspješno zapisana, uključujući `Ctrl+C`; `1` za svaku datoteku čije zapisanje nije uspjelo, puni disk, pogreška, nestanak niza ili uopće nema snimanja.
- **Microsoft Defender:** kada Defender koristi jedno ili više CPU jezgri za skeniranje datoteka koje snimanje piše, izvršavanje se jednom ispisuje na stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Pogledajte instalacijski program [Postavke sustava](#windows-exe).
- Jedno neprekidno snimanje po nizu u jednom trenutku; drugo se odbija s `a continuous capture is already running for this array`. Preko HTTP sesija je `POST /api/camera/array/continuous/start` (tijelo `/api/camera/array/capture` plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` i `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, s ili bez `--continuous`) i**`--continuous --smart`** zadržati petlju po zahtjevu: jedan zahtjev za snimanje po prolazu — u svakom zakazanom trenutku za Interval, nakon što se AE stabilizira za `--smart` — s datotekama svakog prolaza navedenima kao u načinu rada Single, redak `Done — N capture pass(es).` na kraju i `--count N` broji prolaze.

### `array-record` — video/GIF s kombiniranim indeksom (monitorske kvalitete)

Bilježi **uživo kombinirani prikaz indeksa** niza u `.avi` (i opcionalno u `.gif`). Kad je otvoren kombinirani prikaz GUI-ja, bilježi taj prikaz onako kako je prikazan. Ako nijedan nije otvoren, pozadinski sustav prikazuje sam pogled: indeks koji pošaljete pomoću `--expr` ili, prema zadanim postavkama, NDVI nizom, nakon što se niz poravna. Zadani NDVI koristi crvene i NIR pojaseve najbliže 660 nm i 850 nm, kao i prikaz uživo. GUI prikaz otvoren tijekom snimanja snima se dok god ostane otvoren. On provjerava napredak svakih 2 s i zaustavlja se na `--duration`, `Ctrl+C`, ili kada snimač završi samostalno.

Snimanje koje ne zaprimi nijedan kadar završava neuspješno: snimač prijavljuje `no-frames: …` (matrica se možda još uvijek poravnava ili podešava ekspoziciju) i naredba **izlazi `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--array-id ID` | samo niz | Ciljani niz (izostavite ako je priključen samo jedan). |
| `-o, --output DIR` | `output` | Izlazni direktorij. Relativna putanja je u poddirektoriju koji pokrećete naredbu. |
| `--fps F` | `10` | Brzina snimanja sličica. |
| `--duration S` | do Ctrl+C | Automatsko zaustavljanje nakon `S` sekundi. |
| `--gif` | isključeno | Također zapisi animirani GIF. |
| `--gif-only` | isključeno | Spremi samo GIF (bez `.avi`). |
| `--expr EXPR` | NDVI niza | Izraz indeksa za snimanje kada nije otvoren GUI prikaz, u nazivima pojaseva niza (npr. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; vidi [nazivi pojaseva](#index--vegetation-maths)). Kratki naziv boje (`NIR`, `Red`) radi samo kada ga ima jedan pojas niza. GUI prikaz koji je otvoren umjesto toga se bilježi kao prikazan. |

### `array-burst` — raw-Bayer serijski snimak visokog fps-a (za analizu)

Izravno čita spremnik sinkronizirane skupine petlje snimanja — **nije potrebna kalibracijska lanac, exiftool, niti prikaz uživo** — stoga radi punom brzinom snimanja kamere. Bilježi raw sličice + manifest za svaku sličicu + jedan `.daq` po jedinstvenom DLS očitanju pod `<output>/bursts/<base>/`. Ponovno obradite izvan mreže (sljedeća naredba) ili proslijedite `--build` kako biste to učinili odmah po zaustavljanju.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--array-id ID` | samo niz | Ciljni niz. |
| `-o, --output DIR` | `output` | Izlazna direktorija (burst se smješta u `<DIR>/bursts/<base>/`). |
| `--duration S` | do Ctrl+C | Automatsko zaustavljanje nakon `S` sekundi. |
| `--max-frames N` | neograničeno | Automatsko- zaustavljanje nakon `N` sirovih okvira. |
| `--orient` | isključeno | Proizvodi koje izrada stvara iz ovog naleta okreću se prema orijentaciji kamere — proizvodi po kamerama prema orijentaciji vlastite kamere, kombinirani indeks prema masteru — kao što to radi `array-capture --orient`. Sirovi okviri ostaju u izvornom obliku; `manifest.json` bilježi orijentaciju svakog člana i opciju. |
| `--build` | isključeno | Nakon zaustavljanja, odmah ponovno obradite burst (isto kao `array-build-video`). |
| `--products …` | `combined:index` | S `--build`: koji video(i) za izradu (vidi dolje). |
| `--fps F` | `10` | S `--build`: izlazni video fps. |
| `--save-tiffs` | isključeno | S `--build`: također spremite kalibrirane TIFF-ove po kadru (svaki sa svojom [unmix clamp mask](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) kada je piksel zaključan; rezultat izrade ih navodi kao `clamp_masks`). |
| `--gif` | isključeno | S `--build`: također piše animirane GIF-ove. |

### `array-build-video` — offline reprocesira spremljenu seriju snimaka

Vremenom usklađuje svaki sirovi kadar s najbližim spremljenim očitanjem `.daq` i prosljeđuje ga kroz **isti lanac radijance / refleksije / indeksa kao i uvozna cijev**, rেন্ডerajući jedan ili više videozapisa.

`--products` je zarezana lista stavki `kind:level`, gdje je `kind` ∈ `per_cam` | `combined` i `level` ∈ `radiance` | `reflectance` | `index`. Prazan `level` (bez `kind:`) prema zadanim postavkama postavlja se na `per_cam`. Zadana vrijednost je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--burst-dir DIR` | (potrebno) | Put do mape za izbijanja (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` popis, kao gore. |
| `--fps F` | `10` | Izlazni video fps. |
| `--save-tiffs` | isključeno | Također spremite kalibrirane TIFF-ove po kadru uz video(e). |
| `--gif` | isključeno | Također zapisujte animirane GIF-ove. |

**Video ne sadrži maskiranu pločicu** (jedan tok slika, 8-bitni). Umjesto toga, rezultat izrade sadrži `clamp_fractions`: po video proizvodu, udio piksela svakog kadra koje je unmix zaklamao u pojas koji njegov R / G / B kanal sadrži — `Chloros:Band{R,G,B}UnmixClampFraction` s kojim bi bio utisnut proizvod radijance/reflektance tog kadra (za kombinirani indeks, za svaku od članica kamere). Pokretanje ispisuje najgori kadar po proizvodu na `Clamp:` liniji. `--save-tiffs` dodaje maske po pikselu za TIFF-ove, u `clamp_masks/<level>/` pokraj mape `<level>/` za TIFF-ove.**Prijeđi na početak**
**Kratki build ne ostavlja neispravan video.** Svaki se video zapisuje kao `<name>.partial.avi` i konačno ime dobiva tek kad je dovršen, pa stariji video s tim imenom ostaje dok njegov nasljednik ne bude potpun. Pokrenite izgradnju ponovno kako biste zamijenili ostavljeni `.partial.avi`.

Pokretanje ispisuje svaki video koji je snimilo, zatim bilo koji proizvod `Skipped:` (zahtijevanu kameru/razinu — ili kombinirani indeks — koja nije proizvela video, s razlogom — npr. nema kalibracijskog paketa, nije bilo DLS-ovog očitavanja za reflektanciju, ili nije bilo crvenog/NIR-ovog uparivanja pojaseva za kombinirani indeks) i bilo koji `Note:`. RGB kamera koja je zatražila radijanciju ili reflektanciju umjesto toga dobiva svoj prikazni video, s uravnoteženom bijelom bojom kada je kalibracijski paket kamere dostupan; kamera renderirana bez njega naziva se u `Note:`. On **izlazi s ne-nultom vrijednošću** kada je traženi proizvod preskočen ili nije izgrađen prema zahtjevu (konfigurirani izraz indeksa ne procjenjuje se izvan mreže); videozapisi koji su izgrađeni i dalje se spremaju i navode.

> **Odaberite odgovarajući snimač.** `array-record` je *monitoring-grade* — snima renderirani uživo kompozit: kombinirani prikaz GUI-ja ili, ako GUI nije otvoren, vlastiti kompozit pozadinskog sustava NDVI (ili `--expr`) prikaz. `array-burst` → `array-build-video` je *za razinu analize* — sprema sirovi podatkovni niz senzora punom brzinom i nakon toga rekonstruira kalibrirane videozapise radijance/reflektance/indeksa, bez potrebe za prikazom uživo.

### Mono (M3M) jednopojasne kamere

Linija **M3M**je mono sestra modela Bayer**M3C**: jedan uskopojasni interferencijski filtar po kameri (`M3M-<lens>-F<wavelength>`, npr. `M3M-L87-F685`), tako da senzor isporučuje**jednu monohromatsku traku u sivim tonovima** bez Bayerovog mozaika. Nema čega demosaikovati, nema međukanalnog križnog smetanja za razdvajanje, niti balansa bijele za postavljanje — cijeli RGB-postupak obrade boja jednostavno se ne primjenjuje.

Što to znači na CLI-u:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detektiraju mono kameru i**preskaču s porukom od jednog retka** umjesto da šalju besmislene postavke. I dalje se normalno izvršavaju na RGB/Bayer M3C kameri u istoj sesiji.
- **Bayerov pikselni format označava njegovu dubinu.** Bayerov pikselni format označava njegovu dubinu. 388 daje `Mono8`; `BayerRG12`, `high_quality`, `triggered` i `Mono12`; `BayerRG12p` i `Mono12p`; `lattice format` i `--pixel-format Mono12`; `BayerRG12` i `lattice format --set`; A0 i A1; A2 i A3; A4 i A5; A6 i A7; A8 i A9; AA i AB; AC i AD; AE i AF; AG i AH; AJ i  daje `Mono8`; `BayerRG12`, `high_quality` i `triggered` daju `Mono12`; paketirano ime ostaje paketirano (`BayerRG12p` daje `Mono12p`). Datoteke i `lattice format` nazivaju Mono format. Niz koji miješa M3C i M3M kamere uzima jedan format na isti način: Bayer na svakoj M3C, njezina Mono dubina na svakoj M3M. Naziv je njegova bitna dubina: isto vrijedi i obrnuto, tako da se M3C nikada ne postavlja na Mono format (`--pixel-format Mono12` je `BayerRG12` na M3C kameri, bez obzira koja je kamera master u nizu), a `lattice format --set` mapira na isti način i ispisuje format koji je postavio.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** i dalje rade — radijansa i reflektancije su *po-pojasu* radiometrijske mape i savršeno su definirane za jedan pojas. Mono kadrovi nose **identitetsku** matricu odziva senzora (bez 3×3 demiksa), pa ravan prolazi kroz matematičku kalibraciju nepromijenjena.
- **Jedna mono kamera ne može proizvesti indeks vegetacije.**NDVI/NDRE itd. zahtijevaju najmanje dvije trake (npr. Red + NIR). Da biste dobili indeks iz mono hardvera, usmjerite**više** M3M kamera na različite valne duljine, poravnajte ih u jedan višetrakaški sloj i izračunajte indeks *toga*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` samostalno otvara kamere (u [privremenoj sesiji](#multi-camera-transient-sessions)), stoga prvo ništa ne povezujte: `--serials`, inače kamere omogućene u profilu (vidi [Index / Vegetation Maths](#index--vegetation-maths)). Dok Chloros drži bilo koju kameru (GUI, `cam-connect`, `array-connect`) odbija se s HTTP 409 `CAMERA_HELD` i izlazi `1`; oslobodite ih s `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

simboli `--channel` moraju **točno** odgovarati nazivima kanala preseta (osjetljivi na velika i mala slova; NDVI su pisani malim slovima `red`,`nir` — vidi `--list-presets`), a naziv stranice pojasa označava pojas u poravnatoj hrpi (u načinu izvan mreže također se prihvaćaju indeksi pojaseva počevši od 0, npr. `--channel red=0 --channel nir=1`).

Diskriminator u cijelom stacku je token `M3M` u stringu modela (nikada se ne pojavljuje u stringu `M3C`), koji se u GUI/SDKu prikazuje kao `is_mono`.

---

## Postavljanje i podešavanje mrežne kartice domaćina (LATTICE nizovi)

LATTICE kamere prenose GVSP preko Ethernet adaptera domaćina, stoga za nizove s više kamera **vozač**adaptera i**veličina prstena za primanje** jednako su važni kao i brzina veze. Pogrešne postavke ispolje se kao `FRAMES WILL DROP` / `Reduce ROI to enable` greška u odjeljku Postavke niza (i u `lattice network-analysis` / `analyze_array_network()` u SDKu), čak i kada su same kamere ispravne.

### USB 10GbE adapteri — Realtek RTL8157 ("Realtek USB 10GbE Family Controller")

Postavke u nastavku su svojstva upravljačkog programa za Windows. Na Linux USB adapter također dobiva ograničenje (400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson) (vidi [Array fps & burst model](#array-fps--burst-model)), a za svaki adapter važni su i bufferi za primanje soketa u kernelu — vidi [Linux hosts](#linux-hosts--socket-receive-buffers).

| Stavka | Potrebna vrijednost | Zašto je važno |
| --- | --- | --- |
| **Verzija upravljačkog programa**|**≥ v10.67 (siječanj 2026.)**, INF `rtump64x64sta.inf` | Naslijeđeni upravljački program iz **2016.** (v10.65, `rtump64x64.inf`) nepravilno obrađuje isključenje i bugchecke s**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**pri isključenju/ponovnom pokretanju/uspavanosti. Prijelaz se zaglavi (~5 minuta timeouta), korisnik prisilno isključuje uređaj, a ponovljena nečista isključenja**pokvare WMI repozitorij**(PowerShell/alati počinju izbacivati pogreške s `Invalid class`) i**zaglave USB sloj** pri sljedećem pokretanju (NIC se neće omogućiti; USB pogoni prestaju s popisivanjem). Ažurirajte s realtek.com (ili od dobavljača dongla) prije nego što se oslonite na čisto ponovno pokretanje. |
| **Prijemni spremnici**— ključna riječ `ReceiveBufferLen` |**256**(maksimalni driver) | RX prsten mrežne kartice. Zadana vrijednost drajvera od**32**ostavlja samo ~0,26 MB upotrebljivog prstena — što je daleko premalo za burst s više kamera — pa panel niza prijavljuje `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` i blokira povezivanje. Na**256**prsten je velik (**~13,5 MB izmjereno na laboratorijskom 10GbE hostu**), što RX cijevi daje stvaran višak kapaciteta za GVSP burstove s više kamera. (Budući da li će se određena konfiguracija zapravo *spojiti* odlučuje se dvjema provjerama — **drain-aware**provjera propusnosti i provjera**ukupne prenatrpanosti** — nije sirova usporedba bursta i prstena; vidi [Model fps i burst za niz](#array-fps--burst-model).) |
| **URB-ovi za primanje**— ključna riječ `PendingReceives` |**64** (maks.) | USB zahtjevni blokovi u letu; povećava se uz primateljske buffer-e za apsorpciju naleta. |
| **Jumbo Frame** — ključna riječ `*JumboPacket` | **9014** | Potrebno za GVSP pakete od 9000 bajtova (6× manje paketa/okviru nego kod 1500). |

> ⚠️ **Ažuriranje upravljačkog programa mrežne kartice RESETIRA ove napredne postavke na zadane vrijednosti.**Nakon ažuriranja ili zamjene upravljačkog programa adaptera,**ponovno primijenite** `ReceiveBufferLen=256` i `PendingReceives=64`, inače će se panel nizova ponovno vratiti na zadano stanje iako se "ništa nije promijenilo u hardveru". Ovo je glavni uzrok zbog kojeg se ranije ispravno funkcionirajući sustav odjednom odbija povezati.

Primijenite iz **podiignutog** PowerShella (zamijenite naziv svog adaptera, npr. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` pokriva USB 10GbE adaptere.** Sada otkriva vrstu adaptera i podešava ispravnu ključnu riječ za primajuće prstenove (receive-ring): `*ReceiveBuffers`→2048 za PCIe mrežne kartice (Intel I219, itd.), ili `ReceiveBufferLen`→256 + `PendingReceives`→64 za Realtek **USB** 10GbE kontroler (koji ne izlaže `*ReceiveBuffers`). Ciljevi su ograničeni na maksimalnu vrijednost koju svaki upravljački program prijavljuje (`NumericParameterMaxValue`), pa se nikada ne zapisuje vrijednost izvan raspona. Pokrenite ga iz **podižnog** terminala; kao i svaka prilagodba u registru, promjena stupa na snagu nakon ponovnog pokretanja adaptera ili ponovnog pokretanja sustava. Ručne `Set-NetAdapterAdvancedProperty` naredbe iznad i dalje su dobra alternativa — primjenjuju se odmah (ponovno povezuju adapter) bez ponovnog pokretanja.

Hostovi s Linuxom — ulazni spremnici za sokete

Na Linux u, uključujući NVIDIA Jetson, ulazni spremnici za sokete u kernelu jednako su važni kao i prijemni prsten mrežne kartice (NIC). Pri zadanoj vrijednosti kernela (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 bajta) serija GVSP paketa s kamere preplavljuje stream soket prije nego što ga Chloros pročita. Paketi se gube u kernelu, dok sam NIC ne prijavljuje nikakav gubitak, a niz gubi grupe sličica i smanjuje svoju brzinu prikaza. Mjereno na nizu od 4 kamere iza USB 10 GbE adaptera, povećanje obje vrijednosti na 32 MB podiglo je brzinu snimanja niza s 4,96 na 7,52 fps nakon povezivanja, a kontinuirano sirovo snimanje s 4,7 na 7,6 snimki/s, bez izgubljenih paketa u kernelu.

- **.deb ih postavlja.** Instalacija paketa piše `/etc/sysctl.d/60-chloros-gige.conf` (oba na `33554432`) i odmah ga primjenjuje. Vrijednost koja je već postavljena na višu razinu ostaje nepromijenjena.
- **`lattice network` ih provjerava.** Kada je bilo koje od njih niže, prikazuje `socket receive buffers too small for GigE Vision (…)` pod *Pronađeni problemi*, a zapisnik pozadinskog sustava sadrži istu upozorenje pri prvom povezivanju kamere ili niza.
- **Da ih podignete ručno** (do sljedećeg ponovnog pokretanja; datoteka paketa primjenjuje ih pri svakom pokretanju sustava):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` ih također podiže i zadržava, ali samo kada se pozadinski program (backend) pokreće kao root. Pozadinski program koji se pokreće kao vaš korisnik (onaj pokrenut pomoću `chloros-backend`, ili [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) ne može mijenjati postavke jezgre i umjesto toga ispisuje liniju `sudo sysctl`.

**Automatsko podešavanje adaptera utječe samo na povezane adaptere.** Prilikom prvog povezivanja kamere, backend koji se pokreće kao root podešava žičane adaptere čiji je receive ring mali. Sada podešava samo adaptere s vezom, pa neisključeni integrirani priključak zadržava svoju MTU, a prijemne međuspremnike ostavlja netaknutima. `lattice network` više ne navodi neisključeni adapterMTU kao problem. `lattice network`, `--fix`, `lattice probe` i podešavanje adaptera uzimaju u obzir samo žičane Ethernet priključke, ne Jetsonov CAN bus (`can0`) niti USB veze u načinu uređaja (`usb0`, `usb1`) koji nudi host računalu; USB Ethernet adapter za kamere se i dalje računa.

### Osnove mreže (sve LATTICE veze)

- **Adresiranje:** link-lokalno `169.254.0.0/16` (GigE Vision LLA). Host računalo koristi statičku `169.254.x.x/16`; kamere + DAQ-E sami sebi dodjeljuju adrese u istom rasponu. Nije potreban DHCP/proxy.
- **Veličina paketa:**preferira se jumbo (9000), ali neka automatsko ispitivanje (auto-probe) to odredi — ono zaobilazi ograničenje kamere od 1500 bajtova ICMP-a putem GVSP probe, pa se veličina paketa postavlja na jumbo gdje god je to fizička veza dopušta.**Svaka se kamera mjeri jednom po sesiji pozadinskog sustava (backend session), i ponovno kada se promijeni njezina veza (drugi adapter, druga brzina veze ili veza koja je nestala, kao što se događa pri zamjeni switcha); pri ponovnom povezivanju koristi se isti rezultat. Nakon promjene iza switcha koji je ostao aktivan, ponovno pokrenite pozadinski sustav (backend) kako biste ponovno izvršili mjerenje.**Prvo povezivanje ponekad može potrajati nekoliko minuta na ovoj provjeri, a zapisnik to i navodi kada se to dogodi. Povežite se pomoću `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` samo kada znate bolje od sonde, i radije koristite po-naredbeno nego trajno: povezivanje zaobilazi sondu, pa ako put zapravo ne može prenositi 9000, svako**svako** snimanje istekne s `SC_ERR_TIMEOUT -1011` (vidi [Promjenjive okruženja](#environment-variables)).
- **RX ring se skalira s `ReceiveBufferLen`:**pri zadanoj vrijednosti `32` upotrebljivi ring iznosi ~0,26 MB (premalo za bilo koji burst s više kamera); pri maksimalnoj vrijednosti `256` on je velik (~13,5 MB izmjereno na laboratorijskom 10GbE hostu), što pruža stvarnu rezervu. Hoće li se konfiguracija povezati zatim se određuje provjerom propusnosti osjetljivom na odvod**i** provjerom ukupnog preopterećenja u nastavku — a ne sirovim usporedbom bursta i prstena.

### fps i burst model niza

Kako čitati panel Postavke niza (i `lattice analyze-array` / `analyze_array_network` u SDK u):

- **Burst se zbraja po kameri prema stvarnom formatu piksela svake kamere.**Mono**M3M**kamere streamaju**Mono12 (2 B/px)**, ili s `--pixel-format` mono formatom njezine dubine (`BayerRG8` je `Mono8`, 1 B/px);**M3C** Bayer kamere streamaju 8- ili 12-bit (firmware M3C tiho emitira BayerRG12 čak i kada je zatražen BayerRG8). Dakle, kadar s punom rezolucijom sa 4 kamere iznosi ~12,6 MB ako su sve 8-bitne, ali ~25 MB s tri 12-bitne mono kamere. Projekcija određuje format svake kamere prema njezinom modelu (identitetska keš memorija), tako da se burst podudara s onim što kabel zapravo prenosi — a ne s pretpostavkom o jedinstvenom formatu BayerRG8.
- **USB Ethernet adapter ograničen je na 200 MB/s bez obzira na nazivne karakteristike.** Tablica učinkovitosti koja pretvara brzinu veze u održivu vrijednost potječe iz PCIe; USB mrežna kartica oglašava svoju *Ethernet* brzinu veze, ali je ograničena USB sabirnicom i svojim upravljačkim programom. USB 10GbE dongle nekada je postizao ~1063 MB/s "održiva" — broj koji nikada nije bio ispitivan — a rezultirajuće tempiranje pokvarilo je 6–18 % okvira, iako je i dalje izvještavalo o zdravom ciljanom fps-u. USB-priključeni mrežni adapteri sada su ograničeni na apsolutnu vrijednost: **200 MB/s** na Windows u i na NVIDIA Jetsonu, **400 MB/s** na Linux x86_64 (na Linux u, adapteru na USB 2.0 priključku dodjeljuje se **30 MB/s**, a u zapisniku o mogućnostima piše da ga treba priključiti u USB 3 priključak) (ograničenje je USB-bus i upravljački program hosta, stoga se ne skalira s nazivom modela; USB 1 GbE adapter postiže ~80 MB/s i nije pogođen). Na Linux adapter se identificira iz sysfs-a (padajući na `ethtool -i`'s `bus-info`). Svaka je brojka izmjerena: na Windows u isto je polje od 4 kamere korumpiralo 6–10 % sličica pri brzinama iznad 200 MB/s; na Linux u x86_64 isti je adapter radio bez problema pri svakoj maksimalnoj brzini do 480 MB/s (isporučeno 273 MB/s, što je vlastiti limit kamera). Na NVIDIA Jetson isti adapter radio je ispravno samo pri 200 MB/s i povremeno gubio sličice iznad te brzine. `wire_ceiling_source` u zapisu mogućnosti to navodi riječima, a `nic_is_usb` to označava. Prepišite to na bilo koji način pomoću `--wire-ceiling-mbps`.
- **Propusnost je svjesna pražnjenja, a ne cijelog paketa naspram prstena.** Istovremeni burst mora odgovarati samo *privremenom zaostatku* = `max(0, Σ per-cam arrival − host drain) × emit_window`, a ne cijelom burstu. Na tkanini s brzim hostom / sporom kamerom (a **PCIe**10G host + 4× 1 GbE kamere: dolazak ≈ 320 MB/s, odvod ≈ 1063 MB/s) host se isprazni brže nego što se kamere napune, zaostatak je ≈ 0, pa sim-emit u punoj rezoluciji**prihvaća**iako burst od 25 MB prelazi ring od 13,5 MB. Postavite ista četiri kamere iza**USB** 10GbE adaptera na Windows u i protok pražnjenja je 200 MB/s, a ne 1063 — dolazni protok ga nadmašuje, a gubitak se očituje kao oštećeni okviri, a ne kao niža brzina okvira. Na hostu s 1 GbE vezom, donja granica protoka kamera od 31,25 MB/s (DLThr) uzrokuje da dolazni promet nadmaši odlazni → ispravno **blokira** (za *ovaj* tip blokiranja, smanjite ROI ili koristite binning ≥ 2). Propusnost je jedna od **dviju** kontrolnih kapija povezivanja — druga je provjera agregatne prenatrpanosti u nastavku.
- **Predviđeni fps je ciklus najsporije kamere.**Kod istovremene emisije svaka kamera šalje svoj kadar u isto vrijeme, pa niz radi u ciklusu koji je duži od vremena očitavanja senzora i emisije najsporijeg člana (okvirni bajtovi ÷ njegova brzina emisije:**pristupna veza** kamere, 1 GbE ≈ 80 MB/s, ili njezin dio maksimalne brzine mrežne veze hosta kada je ona manja). Svako izlaganje preklapa očitavanje prethodnog kadra, pa izlaganje ne ulazi u ciklus: Automatsko eksponiranje popunjava razdoblje koje brzina određuje, zatim povećava pojačanje, a slaba scena više ne smanjuje brzinu (u verziji 1.2.0 je to činila). Samo ručno eksponiranje dulje od razdoblja usporava okidač (vidi [Eksponiranje na članu niza](#exposure-on-an-array-member)). Dodavanje kamera ne usporava niz sve dok se njihovo dijeljenje kabela ne smanji. Kod serijskog emitiranja (`slip-emit-and-capture`) vremena emitiranja kamera se umjesto toga zbrajaju, a ekspozicija svakog člana uračunata je u razmak. Mjereno na cijelom senzoru, 12-bitno, na nizu od 9 kamera s PCIe 10 GbE hostom: 10,7 fps, odnosno 14,0 fps s [paketiranim transportom](#packed-12-bit-transport---packed).
- **Nakon kontinuiranog snimanja, projekcija navodi brzinu koju je održavala.** Snimanje od 30 s ili više pri automatskoj brzini bilježi brzinu na kojoj se stabilizirala za tu veličinu niza na ovom računalu; kratki pokušaji upravljača brzinom da postigne veću brzinu izostavljeni su. Od tada linija `array-connect` i okvir Postavke niza navode tu brzinu kada je ispod one modela, zajedno s onim što ju je ograničilo: `clock-bound` (niz je radio brzinom na kojoj se stabilizirao, (grab petlja je čekala na svoj trigger sat), `loop-bound` (grab petlji nije ostalo vremena), `capture-bound` (hvatanje je zaostajalo i preskakivalo grupe) ili `measured` (ništa ne navodi koje). Brojka modela je u `wire_fps_max`. Niz i dalje raste do onoga što žica može prenijeti.
- **Niz radi na bilo kojoj mreži — ograničenje je brzina sličica.**Svaka kamera dobiva jednak dio**ograničenja mreže otpornog na sudare**(`sustained × sim_emit_factor`), tempirano pomoću GevSCPD-a, i sve se i dalje eksponiraju na istom okidaču. Na sporijoj mreži udio je mali i brzina sličica pada zajedno s njim: 8 12-bitnih kamera pune rezolucije na 1 GbE mreži bez jumbo okvira dobiva 52 / 8 = 6,5 MB/s svaka, što je oko**1 fps**(otprilike 1,3 fps pri punom opterećenju), dok ih host s 10G povezivanjem pokreće pri 11–14 fps. Ako kamere ne mogu raditi toliko sporo — povežite čitanja raspona GevSCPD svake kamere kako biste saznali — niz se prebacuje na**serijski**emit (`slip-emit-and-capture`): kamere se izmjenjuju na vodi pri punoj brzini veze, aktivirane jedna za drugom, tako da svaki kadar i dalje stigne, ali članovi**nisu istovremeno izloženi**. `analyze-array` i izlaz connect nazivaju način rada; priključak `--force-tier sim-capture-sim-emit` za odbijanje povratnog načina rada. Veličina okvira ne utječe na ovo dijeljenje, stoga**binning i manji ROI povećavaju brzinu okvira, ali ne mijenjaju hoće li niz stati**. Samo je veza koja ne može prenijeti niz čak ni serijski**preopterećena**— npr. znatno više kamera nego što jedna veza može podržati, ili mnogo drugih kamera koje na njoj streamaju — i tada je simptom gubitak paketa GVSP-a, a ne niža brzina: `analyze-array` postavlja vrijednost postignutih fps na nulu i ispisuje `** OVER-SUBSCRIBED**`, veza se nastavlja uz glasno upozorenje, a HTTP veza s `pin_resolution: true` (strogi način) **odbija** umjesto toga. Rješenja su manje kamere, jumbo paketi od početka do kraja ili brži mrežni adapter (NIC). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` spušta odbijanje na glasno upozorenje za rad na klupi — vidi [Promjenjive okruženja](#environment-variables).

### Stanje niza — koji podsustav gubi pakete

`GET /api/camera/array/<array_id>/capability` povezane niza nosi aktivni
`health` blok, koji se ponovno procjenjuje u **10-sekundnom** kliznom prozoru. Raspad gubitka okvira
na dva uzroka kojima su potrebna suprotna rješenja, umjesto da izvještava o jednoj "nepotpunoj"
stopi koja ne navodi nijedan od njih:

| Polje | Što to znači | Koji podsustav |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (po serijskoj vezi) | Okvir je **stigao i bio je strukturno oštećen**— GVSP gubitak paketa. |**Mreža**: budžet kabela, tempiranje, NIC RX prsten, MTU |
| `never_arrived_rate_pct` (po serijskoj vezi) | Okvir **nikada nije stigao**— kamera nije poslala okvir, ili ga ništa nije napustilo. |**Okidač / sinhronizacija**: M8 kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Stopa najgore kamere za svaki. | — |
| `per_cam_rate_pct` | Kombinirana stopa nepotpunih prijenosa po kameri (obje uzročnosti zajedno). | — |
| `stable_for_seconds` | Koliko dugo je svaka kamera ostala ispod 0,01 %. | — |

Iznad 5 % back-end bilježi redak `[array-health <id>] WARN` koji imenuje podjelu — pri prvom kršenju, pri promjeni pojasa ozbiljnosti, jednom u minuti dok traje i jednom kada se otkloni. Pokvarena polovica piše `[gvsp-corrupt <SN>]` u zapisnik pozadinskog sustava pri prvom zabilježenom slučaju po kameri i razlogu, a zatim svake 60 sekundi prikuplja ukupne podatke; panel zapisa aplikacije prikazuje
umjesto toga se bilježi linija `[array-health <id>]`. Svaka evaluacija i dalje završava u zapisniku pozadinskog sustava; brojači se pomiču za svaki buffer bez obzira na ono što se ispisuje.

Isti zapis prijavljuje broj na koji se cijela alokacija oslanja:

| Polje | Što znači |
| --- | --- |
| `wire_ceiling_mbps` | Kontinuirani budžet mrežne veze hosta na snazi, MB/s. |
| `wire_ceiling_source` | Odakle potječe ta brojka, riječima — npr. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (na Linux u x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) ili `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` kad ga postavi `--wire-ceiling-mbps` (ili polje **Wire Budget** u GUI-u). |
| `nic_is_usb` | `true` za USB Ethernet adapter — vidi ograničenje USB-a gore. |

**Tumačenje:** ne-nulovita vrijednost `gvsp_corrupt_rate_pct` uz `never_arrived_rate_pct` na 0
znači da su pokretanje i sinkronizacija kabela savršeni te da se 100 % gubitka događa na mrežnom putu — smanjite `--wire-ceiling-mbps` i ponovno se povežite. Suprotan obrazac umjesto toga ukazuje na sinkronizacijski kabel ili liniju za pokretanje.

> **`--target-fps` nije poluga za oštećene okvire.** GevSCPD ritam se piše jednom pri povezivanju,
> stoga smanjenje brzine okidača mijenja radni ciklus, a ne brzinu
> istovremene emisije paketa. Mjereno smanjenje potražnje za 5× nije donijelo nikakvo poboljšanje;
> smanjenje ograničenja brzine prijenosa s 240 na 200 MB/s dovelo je isti sustav s 10,4 %
> oštećenih okvira na 0,00 %.

> **Aktivni niz ne može sam ponovno isplanirati na firmveru LATTICE (M3C / M3M).** Ne može to popraviti samostalno; isključite i ponovno spojite kako bi se
> odabirač vremena spajanja mogao ponovno isplanirati s novim gornjim pragom.

### Simptom → rješenje

| Simptom (Postavke niza / povezivanje / `analyze_array_network`) | Uzrok | Rješenje |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` resetira se na 32 (obično nakon ažuriranja upravljačkog programa) | Postavite `ReceiveBufferLen`→256, `PendingReceives`→64; ponovno otvorite panel (ponovno pokrenite pozadinski program ako je pohranio staru veličinu prstena) |
| Ponovno pokretanje/isključivanje se zablokira; kasnije `Invalid class` WMI pogreške, NIC se ne aktivira, USB pogoni nedostaju | Stari Realtek USB 10GbE upravljački program iz 2016. → plavi ekran `0x9F` → prisilna isključenja | Ažurirajte upravljački program adaptera na ≥ v10.67 (2026), zatim ponovno primijenite postavke receive-ringa iznad |
| Niz se povezuje na punoj rezoluciji senzora, ali s niskom brzinom sličica | Automatski način rada zadržava punu rezoluciju senzora i dopušta da se brzina spusti na ono što veza može prenijeti | Odaberite jednu od bržih opcija koje popis izlaza za povezivanje nudi (`--binning 2` / `4`, ili manji `--width`/`--height`), omogućite jumbo pakete ili nadogradite vezu |
| Niz izvještava o zdravom ciljanom fps-u, ali isporučuje samo djelić toga; `health.gvsp_corrupt_rate_pct` ne-nula, `never_arrived_rate_pct` 0 | Procijenjeni 'wire budget' hosta precjenjuje ono što on zapravo može podržati (tipično kod USB Ethernet adaptera, uskog PCIe traka ili dijeljenog tkiva) | Ponovno povežite s nižim `--wire-ceiling-mbps` i ponovno provjerite blok zdravlja. **Ne** `--target-fps` — GevSCPD tempiranje je fiksirano pri povezivanju |
| Linux: niz gubi grupe okvira i smanjuje svoju brzinu okvira dok mrežna kartica (NIC) ne prijavljuje nikakve gubitke; `lattice network` navodi `socket receive buffers too small` | Kernelovi soketni bufferi za primanje su na zadanim postavkama, pa se paketi kamere gube u kernelu | Povećajte `net.core.rmem_max` / `rmem_default` na 32 MB — vidi [Linux hosts](#linux-hosts--socket-receive-buffers). .deb to radi tijekom instalacije |
| Nedostaju kamere u objavljenim grupama; `health.never_arrived_rate_pct` > 0, `gvsp_corrupt_rate_pct` 0 | Put okidača / sinkronizacije — kamere se ne aktiviraju, nije problem mreže | Provjerite M8 sinkronizacijski kabel i `--line`; potvrdite da je svaki član aktiviran (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` premašen u `analyze-array`, ili odbijen zahtjev za povezivanje u strogoj načinu rada (`pin_resolution`; `array over-subscribes the wire`) | Žica ne može prenositi niz na bilo kojem tempu kojeg kamere dostignu, istovremeno ili naizmjence — daleko više kamera nego što žica može dijeliti, ili druge kamere koje na njoj prenose | Manje kamera, jumbo paketi od kraja do kraja, ili brži mrežni adapter. **ROI/binning neće pomoći** (dijeljenje je neovisno o veličini okvira). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nadjačava na klupi (prihvaća gubitak paketa) |
| Niz se povezuje u `slip-emit-and-capture` (serijski) na sporoj mreži | Kamera se ne mogu dovoljno sporo sinkronizirati da bi istovremeno dijelile vezu, pa se smjenjuju; slike se ne eksponiraju u istom trenutku | Jumbo okviri ili brži NIC vraćaju istovremeno snimanje; `--force-tier sim-capture-sim-emit` inzistira na tome (i umjesto toga gubi pakete) |

---

## `chloros-cli daq`

Naredbe za spektralni senzor. Zapovijedi `pool-*` su tanki klijenti protokola HTTP koji upravljaju senzorom putem trajnog bazena pozadinskog sustava, stoga im je potreban pokrenuti pozadinski sustav (vidi [Pokreni pozadinski sustav](#start-the-backend)). Backend upravlja transportom, pa skripte za CLI i SDK dijele jedan aktivni rukohvat umjesto da se bore za serijski port. `pool-discover` je jedini način da se sazna BLE MAC adresa DAQ-M uređaja.

> **`chloros-cli daq --help`** (ili `-h`, ili `daq help`) navodi podzapovijedi `pool-*` (na Linux u, zajedno sa zapovijedima za izravan pristup hardveru koje build uključuje), i `chloros-cli daq <subcommand> --help` (ili `daq help <subcommand>`) prikazuju opcije za svaku od njih. One obuhvaćaju povezivanje, streamanje, snimanje kalibriranih `.daq` datoteka, zamjena profila kapa i povezivanje DAQ-a s kamerom, a za DAQ-E njegova mrežna postavljanja, PTP, ponovno pokretanje i ažuriranja firmvera. DAQ se također može upravljati iz Python a pomoću `chloros_sdk.connect_daq_sensor()`, koji koristi istu zajedničku putanju.
>
> **Koristite `pool-*` naredbe na svim platformama.** One se jednako ponašaju na Windows u, Linux u i Jetsonu. Na Windows u, `daq discover` i `daq list` su sinonimi za `pool-discover`, kao i svaka druga `daq` naredba koja nije `pool-*` izlazi iz `1` pomoću `DAQ direct-hardware commands require the full daq package`. Na Linux u i Jetsonu, `daq discover`, `daq list` i ostale naredbe `daq` koje nisu `pool-*` su izravne-hardverske naredbe: otvaraju senzor u vlastitom procesu CLI-a, izvan pulta pozadinske komponente (dakle ne na senzoru kojeg drže pozadinska komponenta, GUI ili skripta), a `daq discover` ondje prima `--only` i `--timeout`, ali ne i `--json`. Ova referenca dokumentira samo naredbe `pool-*`.

### Radni tijek prve veze DAQ senzora

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referenca za `pool-*`

| Podzapovijed | Svrha |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otvori senzor u pozadinskom poolu. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U na određenom serijskom sučelju. |
| `daq pool-connect --ble` | DAQ-M preko BLE, automatsko skeniranje MAC-a. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M preko BLE-a na poznatom MAC-u (podrazumijeva `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E preko Etherea na poznatom nazivu hosta ili IP-u (podrazumijeva `--eth`). |
| `daq pool-connect --eth` | DAQ-E preko Etherea, automatsko otkrivanje hosta (mDNS + rezervna opcija ARP; radi s praznom ARP predmemorijom na sustavima Windows i i Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Podesite prozor integracije / stanje AE. |
| `daq pool-connect --no-stream` | Povežite se, ali još ne započnite streaming (nastavite s `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcije kapacitivnosti. Zadano u pozadini je `sunshine_cosine`; `none` je goli senzor. [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ga zanemaruje (fiksni difuzor). **Senzori se isporučuju bez kapice, stoga navedite naziv kapice koju ste ugradili:** zadano je zabilježeno kao pretpostavljeno (`cap_id_source=auto_default`) i obrada o tome upozorava, budući da goli DAQ-U pod korekcijom sunčeve svjetlosti čita ~30x više. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skenirajte svaki transport za senzore koje biste mogli povezati, bez povezivanja. **Ovo je način na koji pronađete BLE MAC DAQ-M-a.** `--timeout` je po prijenosu (zadano 5 s, 1-20)Senzori koji su već otvoreni u poolu nisu navedeni — povezan DAQ-M prestaje s oglašavanjem — stoga za njih upotrijebite `pool-list`. `usb` navodi samo USB serijske uređaje: vlastiti serijski priključci računala (`COM1`, `/dev/ttyS*` za Linux) ne mogu biti senzori i izostavljeni su. |
| `daq pool-list` | Prikaži sve senzore u pozadinskom poolu. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Poveži kameru ili niz s DAQ-om, kao što to čini odabir senzora svjetla u GUI-ju: njezini zapisi koriste, podudaraju i spremaju očitanja tog DAQ-a. `SENSOR` je ID senzora iz `pool-list` ili adresa na kojoj se DAQ nalazi. `--clear` (no `SENSOR`) otkazuje vezu, i snimke se vraćaju na najnoviji DAQ. Povezivanje traje dok se pozadinski sustav ne ponovno pokrene ili dok se ta kamera, niz ili DAQ ne odspojiti: odspojavanje (ovdje, iz SDK a ili u aplikaciji) prekida svako povezivanje hardvera koji se odspaja, bez obzira na to tko ga je postavio, kao i povezivanje članova niza s nizom. Kamera mora biti povezana, u otvorenom projektu, ili prethodno otvorena na ovom računalu, a niz povezan ili u otvorenom projektu; svaki drugi serijski broj ili ID odbijen je po nazivu i ništa se ne povezuje. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Oslobađanje — tko god je otvorio senzor, tako da također oslobađa senzor koji je skripta srušena ostavila u bazenu (svako snimanje na njemu prvo se dovršava). Senzor koji nije u bazenu nema što osloboditi: izvodi `ID: already released (not in the pool)` i izlazi `0`; oslobađanje koje je pozadinski sustav odbio ili nije potvrdio izlazi `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N najnovijih spektralnih okvira (po zadanom 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Nastavi / pauzira streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Pokrene / zaustavi .daq snimanje. `-o` prema zadanim postavkama pohranjuje se u `~/Documents/DAQ Live View` na računalu pozadinskog sustava (relativna putanja je unutar mape iz koje pokrećete naredbu); `--duration` (`-d`, zadano `0` = do `--stop`). `--confirm-after S` upozorava kada nijedno očitanje ne stigne S sekundi nakon početka (zadano 20; 0 ga isključuje); s `--duration`, `--heartbeat S` ispisuje broj očitavanja svakih S sekundi i upozorava kada prestane rasti (zadano 30; 0 ga isključuje). Snimanje na DAQ-E koja utihne nastavlja se u istu datoteku čim se back-end ponovno poveže, a `--stop` navodi praznine ([A DAQ-E That Stops Delivering](#a-daq-e-that-stops-delivering)). Ctrl+C prekida snimanje `--duration` ranije i o tome izvještava; bez `--duration` snimanje se nastavlja nakon Ctrl+C (izlaz `130`) sve dok `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zamijenite profil korekcije kapa tijekom izvođenja. `--cap-id` je potreban; `--cap-id none` uklanja korekciju kapa (goli senzor). DAQ-E-S zadržava `as_recorded` bez obzira na postavku. |
| `daq pool-enable-ptp --sensor-id ID` | Ponovno uključite PTP za DAQ-E čiji je PTP isključen. Postavka se sprema na jedinici, tako da jedinica koja je isključena ostaje isključena — njezini vremenski pečati nisu sinkronizirani s PTP-om — sve dok se ova naredba ne izvrši. `pool-list` označava takvu jedinicu (`!! PTP is switched OFF`) i ispisuje naredbu za nju. Chloros nikada sam ne uključuje jedinicu, jer isključenost može biti namjerna. |
| `daq pool-info --sensor-id ID [--json]` | Postavke samog DAQ-E-a: njegov firmware (i je li ova verzija Chloros a sadrži noviju), njegova adresa i kako ju je dobio (DHCP / link-local / statički), spremljena mrežna postavka sve dok je senzor ne pokrene (uključujući onu spremljenu pomoću `--no-reboot`), stream pri uključivanju i PTP. Na senzoru čiji stream ne dopire do ovog računala, on čita senzor na novoj vezi i o tome obavještava. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Postavi DAQ-E na DHCP (`--auto`) ili statičku adresu (`--mask` zadano `255.255.255.0`). Senzor se ponovno pokreće kako bi se primijenile postavke i ponovno se povezuje s prethodnim postavkama, čekajući do `--timeout` sekundi (zadano 45) na to; `--no-reboot` samo ga sprema. Vidi [Postavke uređaja DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Određuje hoće li DAQ-E započeti streaming pri uključivanju bez priključenog softvera (pohranjeno na senzoru). Chloros u svakom slučaju samostalno pokreće streaming. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Ponovno pokrenuti DAQ-E i ponovno ga povezati s prethodnim postavkama (čeka do `--timeout`, zadano 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Flashat firmware DAQ-E-a uključen u ovo izdanje Chloros na senzor koji radi na starijoj verziji, zatim ga ponovno povežite (otprilike jednu minutu; ne isključujte ga; `--timeout` zadano 45 s da se vrati). Sensor koji već radi se ne dira. |

### DAQ-E koji prestaje isporučivati

DAQ-E u bazenu koji 20 sekundi ne isporučuje spektar ponovno se povezuje s pozadinskim sustavom (backend). Isključivanje i ponovno uključivanje prekidača koji napaja senzor to i radi: senzor se ponovno pokreće i ne počinje ponovno slati podatke dok mu se to ne naloži (osim ako `pool-set-auto-stream --on`).

- **Prvo provjerava je li to isti senzor.** Pita na adresi koju je senzor imao, ne mijenjajući ništa, za naziv, MAC i serijski broj senzora. Također pita pod imenom `daq-e-<id>.local` senzora, ali tom odgovoru vjeruje samo kada dolazi s vlastitog mrežnog adaptera (MAC) senzora. Ništa drugo što odgovori se ne ponovno povezuje niti se bilježi.
- **Ponovno se povezuje s postavkama sesije:** vremenom integracije, prosječnom vrijednošću kadra, automatskom ekspozicijom i profilom zatvarača kakvi su bili, a ne s tvorničkim postavkama senzora pri uključivanju. Senzor koji je imao PTP uključen, a vratio se s isključenim PTP-om, ponovno se uključuje. Zatim ponovno pokreće stream i računalni prijemnik za njega.
- **Ponovno pokušava u sve dužim intervalima:** odmah, zatim nakon 5, 10, 20 i 40 sekundi, a zatim svakih 60 sekundi dok senzor nije dostupan. Svaki pokušaj samo pita senzor tko je. Senzor koji odgovori, ali i dalje ništa ne šalje nakon tri ponovna povezivanja, ostavlja se na miru i izvještava se o njemu (`auto-reconnect GAVE UP`). `pool-stream --start` pokušava ponovno.
- **`pool-list` pokazuje gdje se nalazi.** Dok je senzor tih, njegova linija `STREAM FAULT` govori što ponovna veza radi. Redak se briše s prvim spektrom, a senzor zatim čita `reconnected automatically N time(s)`.
- **Zapis se nastavlja u istom `.daq`.** Datoteka označava svaku prazninu s dva retka: `event_type` 5 na posljednjem spektru prije praznine i 6 na prvom nakon nje, svaki samo s vremenskim žigom. `pool-record --duration` ispisuje `!! STREAM LOST …` dok senzor nije dostupan i `stream resumed after …` kada se vrati. `pool-record --stop` navodi svaki razmak i upozorava, kao što to čini za prazna datoteka (zapis koji ne sadrži nikakvo očitanje izlazi iz `1`).
- **Zapis pohranjuje jedno kalibriranje.** Ako se senzor vrati s drugačijim kalibriranjem dok zapis traje, ostatak zapisa zadržava ono s kojim je započeo, a upozorenje o zaustavljanju to i navodi. `pool-disconnect` + `pool-connect` preuzima novu kalibraciju.
- **Vaši zapovijedi imaju prednost.** Senzor koji ste `pool-disconnect` ili pauzirali pomoću `pool-stream --stop` nikada se ne ponovno povezuje, čak ni kada je ponovno povezivanje već u tijeku.
- **`pool-info` na tihom senzoru** čita senzor na novoj vezi i dodaje `!! stream NOT reaching this computer`. Senzor koji ne odgovara odmah se prijavljuje. `pool-list` više ne prikazuje PTP stanje za senzor koji je prestao odgovarati.
- DAQ-U i DAQ-M nisu obuhvaćeni. DAQ-M se samostalno ponovno povezuje putem Bluetootha, a DAQ-U koji je odspojen s USB-a ponovno treba `pool-connect`.

### DAQ-E-S — fiksni difuzor i senzor nagiba

**DAQ-E-S** je DAQ-E s dvije razlike koje su važne za softver. Sve ostalo — otkrivanje, povezivanje, PTP, snimanje, postavke uređaja u nastavku — radi točno kao i kod DAQ-E.**Njegov difuzor je trajan, stoga ne zahtijeva korekciju kapom.** Difuzor tipa Sunshine ugrađen je i bio je na jedinici kada je mjerena njezina tvornička kalibracija, pa ta kalibracija već bilježi stvarnu zračenost prema dolje. Chloros postavlja svoju kap na `as_recorded` (bez profila) bez obzira na zahtjev: poklopac nazvan za njega (`sunshine_cosine`, `fov_*`, `none`) se zanemaruje uz poruku, a njegova snimanja spremaju `cap_id_source=model`, pa upozorenje o pretpostavljenom kape za njega nikada se ne aktivira. Također nemojte ručno primjenjivati kap na njegove podatke: kap na vrhu bi čitala oko 11x previsoko.

**Sadrži troosni akcelerometar.** Kozinusna karakteristika difuzora sunčeve svjetlosti važeća je samo dok je ravan, stoga nagnuti senzor izvještava o pogrešnoj zračenosti — a reflektancija nasljeđuje tu pogrešku — bez drugih simptoma. Svako očitanje DAQ-E-S-a stoga nosi podatke o svom položaju:

| Polje | Značenje |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Kutovi u stupnjevima. `None` kada firmware označi kut kao nedefiniran (slobodan pad, mrtva os). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Ubrzanje u mili-g. `mag_mg` daleko od ~1000 znači da se jedinica kretala i da kutovi za taj uzorak nemaju smisla. |
| `cal_applied` | Postavljeno: osi i kutovi su u referentnom sustavu difuzora (primijenjena je kalibracija montaže jedinice). Poništi: sirovi referentni sustav ploče, koji može pokazivati do ~2,3° nagiba koji nije stvaran. |
| `sample_age_ms` | Starost uzorka akcelerometra kada je spektrum poslan. |
| `temp_raw` | Nekalibrirani trend temperature čipa — samo razlike, nikada očitanje termometra. |

`daq pool-latest --json` ih vraća pod ključem `imu` svakog okvira (`null` na DAQ-U, DAQ-M ili obični DAQ-E). `.daq` snimljen s DAQ-E-S pohranjuje iste vrijednosti u svojim `imu_*` stupcima (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) uz svaki spektar, pa čitatelj može razlikovati oblak od srušenog stativa. Na svim ostalim modelima te su stupci prazni.

Tok nagiba izvještava se zasebno: `daq pool-list` ispisuje `!! ATTITUDE STREAM FAULT (spectra unaffected)` dok je uređaj isključen i koliko se pogrešaka nakon toga oporavio. Pogreška tamo nikada ne označava prazninu u spektralnom snimanju. `daq pool-discover` navodi jedinicu kao `DAQ-E-S` (kao `DAQ-E` ako ne odgovori na provjeru identiteta u roku od otprilike 2 s).

### Postavke uređaja DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` bez `--gateway` ne sprema gateway** (bilo koji prethodno postavljen gateway se uklanja), što je ispravno za mrežu kamera bez rutera.
- **Statička adresa mora biti na mreži ovog računala.** Chloros odbacuje podmrežu koja ne uključuje adresu ovog računala na mreži senzora: nakon ponovnog pokretanja ništa ne može dohvatiti senzor kako bi se to vratilo. Prođite `--force` samo kada se ova adresa računala promijeni ili kada se usmjeri na novu podmrežu.
- **Ponovno pokretanje se odbija dok senzor snima.** Prvo zaustavite snimanje (`pool-record --stop`). Snimanje započeto u trenutku prije ponovnog pokretanja bit će zatvoreno njime, a `pool-set-network`, `pool-reboot` i `pool-update-firmware` završavaju linijom `Warning:` koja imenuje datoteku snimanja (čak i kada se senzor potom ne vrati).
- **Nakon ponovnog pokretanja Chloros se ponovno povezuje samo sa samim senzorom.** Čeka na mjestu gdje se senzor nalazio -- a nakon `--auto` i pod svojim `daq-e-<id>.local` imenom -- na odgovor s imenom senzora, naziv hosta i model, s mrežnog adaptera (MAC) koji je imao, i ponovno ga povezuje na adresu koja je odgovorila. Odgovor pod tim imenom prihvaća se samo s tim MAC-om, stoga se s računala koje pristupa senzoru putem usmjerivača ili iz WSL2 u njegovoj zadanoj NAT mreži, senzor koji je DHCP premjestio na novu adresu neće ponovno povezati: zapovijed to potvrđuje i ispisuje `pool-connect --eth-host <its address> …` zapovijed kojom ga vraća s prethodnim postavkama izloženosti i kapnja.
- **`pool-update-firmware` odbija senzor čiju verziju firmvera ne može pročitati.** Senzor koji se nakon bljeska ne vrati brže od zadanog vremena čekanja nije doživio kvar: čim odgovori, ponovno se povezuje sa postavkama novog firmvera i naredba upozorava da je bio spor.
- **Ažuriranje firmvera zahtijeva dolaznu TCP vezu.** Senzor se ponovno povezuje s ovim računalom kako bi preuzeo sliku. Chloros sluša na adapteru koji usmjerava prema senzoru (na svakom adapteru samo kada ne može utvrditi koji) i poslužuje samo vlastitu adresu senzora; sve druge veze zatvorene su bez usluživanja. Na Windows, pravilo vatrozida instalatera `Chloros Backend (TCP-In)` mu dopušta ulazak (vidi [Windows (.exe)](#windows-exe)); vatrozid koji ga blokira onemogućuje ažuriranje s `… device never connected back on TCP <port> …`.
- **Pod WSL2 koristite ogledalo umrežavanja** (`networkingMode=mirrored` u `%UserProfile%\.wslconfig`). U zadanoj NAT načini adresa ovog računala na mreži senzora je vlastita adresa WSL-a, stoga se svaka statička adresa odbacuje (osim `--force`), prijenos uživo ne dopire do WSL-a i ažuriranje firmvera se ne može dovršiti, jer se senzor ponovno povezuje s računalom kako bi primio sliku.
- **ID senzora i naziv hosta se ne mijenjaju.** Bez obzira na njegovu adresu, DAQ-E je i dalje `daq-e-<id>` i `daq-e-<id>.local`, pa `--eth-host daq-e-<id>.local` i dalje pronalazi.
- **GUI ima iste postavke**: kartica Svjetlosni senzori, postavke senzora (zupčanik), Mreža. GUI održava vlastitu vezu sa senzorom, stoga promijenite senzor koji je GUI otvorio iz GUI-ja: ponovno pokretanje iz CLI-a prekida GUI-jevu sesiju s njim.
- Ekvivalenti u SDK-u su `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` i `update_firmware()`.

---

## `chloros-cli project`

Otvorite, povežite se s i upravljajte spremljenim projektom Chloros (mapu s `cameras.json` + `sensors.json` + `project.json`). Sve se usmjerava kroz pozadinski sustav (backend) tako da GUI i CLI stvaraju identično stanje hardvera.

### Referenca podzapovijedi

| Podzapovijed | Svrha |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Ispiši manifest uređaja projekta (kamere, nizovi, senzori), očitaj iz njegovih datoteka i provjeri njegove izvornik datoteke (vidi [Premješteni projekt](#a-moved-project---relink)). Ne otvara projekt u pozadinskom sustavu: projekt ostaje otvoren, a njegovi uređaji i snimke ostaju nepromijenjeni. Izlazi iz naredbe `1` kada se nijedna izvornik datoteka projekta ne može pronaći. |
| `project devices PATH [--reconnect]` | Popis ili ponovno pokretanje otkrivanja. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Povežite svaku spremljenu kameru / niz / senzor. Svaki se niz povezuje kao što to radi `lattice array-connect`: do 600 s, a njegovi se koraci ispisuju na stderr (`[array <id>]    <s> s: <step>`; stdout je izvješće u formatu JSON), i niz koji se u tom roku ne poveže poništava se — njegove se kamere vraćaju na stanje u kojem su pronađene — i izvještava se o tome pod `errors`, nikada ne ostaje povezan. `--packed` postavlja komprimirani 12-bitni transport za svaki niz (po zadanom `auto`; projekt ne sprema postavke za svaki niz; vidi [Pakirani 12-bitni prijenos](#packed-12-bit-transport---packed))Svaki senzor svjetla otvara se u DAQ poolu backenda na svojoj spremljenoj adresi sa svojim spremljenim vremenom integracije, prosječanjem kadrova, automatskom ekspozicijom i kapom: senzor kojem nije odabran kap dobiva `sunshine_cosine`, jedan spremljeni bez kapa (`none`) ostaje bez kapa, a DAQ-E-S nikada ne uzima kap. GUI-jev DAQ-panel nije dio pool-a: prvo zatvorite senzor tamo. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Pojedinačno snimanje s nazivane kamere ili niza (`--prefix` zadano `capture`). `--orient` (nizovi): niz pohranjuje sirovi podatak, koji ostaje u izvornom obliku — time se bilježi da proizvodi koje `process` iz njega stvara trebaju biti orijentirani, kao `array-capture --orient`. Zadano: nativno. `project burst` / `stream` i `capture` / `burst` / `stream` recepta (`orient: true`) koriste istu opciju. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-okvirni burst iz nazvane kamere ili niza (`-n/--count` zadano 5; `-i/--interval` sekundi između okvira, zadano 0). Serijski burstovi uklanjaju duplikate ponovljenih sinkroniziranih grupa (nadzor zastarjelosti) kako bi se spriječilo da djelomično ciklirajući niz vrati N kopija istog kadra; ispisuje rezultate po iteraciji. Snimanje koje zakaže usput zaustavlja burst i izlazi iz `1`, navodeći koliko je spremljeno prije toga. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Snimanje na disk u petlji: `--fps F` snima drugi (po zadanom 3), jedan kadar po snimanju s kamere, jedna sinkronizirana grupa kadrova iz niza (s istim nadzornim tajmerom zastarjelosti kao `project burst`), dok `-n N` ne snimi (po zadanom `0` = dok se ne pritisne Ctrl+C, što ispisuje koliko je spremljeno i izlazi iz `0`, ili `1` kada nijedan nije ili kada je datoteka neuspjela; Ctrl+C prije nego što `-n N` uhvati izlazi iz `130`). Hvatanje koje zakaže usred prekida tok i izlazi iz `1`, pri čemu se ispisuje koliko je okvira spremljeno prije toga. `--poll-interval` je zastario i zanemaruje se: još uvijek se prihvaća kako bi se starije skripte izvršile, uz napomenu na stderr. |
| `project sensor read PATH NAME [--json]` | Najnoviji spektralni kadar. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Snimi .daq za `--seconds` (zadano 10). Snimanje koje ne sadrži nikakvo očitavanje izvještava se na stderr i izlazi iz `1`. |
| `project run PATH RECIPE.yaml` | Izvršite YAML/JSON recept za snimanje. `--dry-run` provjerava valjanost bez pokretanja. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Izračunajte poravnanje za niz — pogledajte [tablicu zastavica u nastavku](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Ispiši trenutni profil poravnanja. |
| `project align clear PATH NAME` | Obriši spremljeni profil. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Pomakni transformaciju jednog sluge (zadano `0`, `0`, `0`, `1.0`: bez promjene). |
| `project align export PATH NAME --to FILE` | Spremi profil u JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Učitaj spremljeni profil. |

#### `project align calibrate` Opcije

| Zastavica | Zadano | Opis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metoda poravnanja. **Ovi se pravopisni oblici razlikuju od `lattice align-calibrate`**, koji prihvaća kratice `orb` / `akaze` / `phase`; ove dvije naredbe nisu zamjenjive na ovoj zastavici. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformiraj model da pristaje. |
| `--frames N` | `1` | Sinkronizirane snimke okvira za prosjek. |
| `--reference SN` | glavni | Serijski broj referentne kamere; svaki drugi član deformiran je na nju. |
| `--max-features N` | `5000` | Gornja granica broja ORB značajki. |
| `--ratio-threshold F` | `0.75` | Lowe's omjer test. |
| `--ransac-threshold-px F` | `3.0` | RANSAC prag inlier. |
| `--min-matches N` | `15` | **Vrata kvalitete** — odbacite rješenje s manje od ovog broja inlier podudaranja. |
| `--max-reproj-err-px F` | `4.0` | **Vrata kvalitete** — odbacite rješenje s RMS pogreškom reprojekcije većom od ove. |
| `--checkerboard RxC` | — | Geometrija ploče za `--method checkerboard`, npr. `9x6`. |
| `--name PROFILE` | prazno | Naziv profila ugrađen u spremljeni JSON. **Ne naziv niza** — to je pozicijski `NAME`. |

Dva kontrolna koraka razloga su zašto kalibracija može uspješno završiti, a ipak odustati od spremanja: profil koji ne prođe bilo koji od njih tiho bi pogrešno registrirao svako kasnije snimanje, stoga se odustaje od spremanja umjesto da se sačuva.

### Premješteni projekt (`--relink`)

Projekt `project.json` imenuje svaku izvornu datoteku -- svaku sliku i snimku senzora svjetla -- njezinim punim putem. Ako premjestite mapu projekta, ili cijelu mapu projekata na drugi disk, ti putovi i dalje navode staro mjesto.

Otvaranje projekta ispravlja ono što može samo: datoteka koja se nalazila u mapi projekta traži se u mapi u kojoj se projekt sada nalazi, a svaka pronađena putanja se preusmjerava i sprema (`project.json` se prvo sigurnosno kopira kao
`project.json.before-relink-<date>-<time>`). To se događa pri svakom otvaranju -- aplikacije,
`chloros-cli project open`, `chloros-cli process` u postojeći projekt, SDK-a i `chloros-mcp`-a. Putanja se preusmjerava samo na datoteku koja je prisutna.

Datoteke koje su bile izvan mape projekta -- sesija huba, uvoz mape -- ne mogu se na taj način pronaći. Nazovite mapu u kojoj se sada nalaze:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` uzima mapu u koju su datoteke premještene ili bilo koju mapu iznad nje: mapa svake nedostajuće datoteke traži se ispod nje prema njezinim nazivima mapa na kraju
(`Hub Projects/<session>/raw/<serial>`, zatim `<session>/raw/<serial>`, ...), i svaki
datoteke se preuzimaju iz onog direktorija koji ih je pronašao u najvećem broju. `project open` ispisuje što je ponovno usmjereno, koliko je izvornih datoteka pronađeno i u kojim su direktorijima bile preostale. Neki još uvijek nedostaju: upozorenje, izlaz `0` -- projekt se otvara s ostalima. Nijedan nije pronađen: pogreška pri imenovanju tih mapa i `--relink`, izlaz `1`; pozadinski sustav isto tako
odbija otvoriti takav projekt za `process` ili `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Primjeri

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL za recepte

`project run RECIPE.yaml` prihvaća YAML ili JSON datoteku koja opisuje niz radnji:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podržane radnje: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` uzima `cameras.json` `settings` ključeve — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (starija imena `exposure_time`, `exposure_auto`, `gain`, `gain_auto` prihvaćaju se kao sinonimi). Postavljena ekspozicija, pojačanje ili ciljana vrijednost na ovaj način ostaju zadržane kao vaše: automatska kontrola ekspozicije Chlorosa (uključujući ekspoziciju mjerenu DAQ-om) ne dira ih. Pri ručnoj ekspoziciji pojačanje ostaje kakvo je bilo: u automatskom načinu kamera neprestano prilagođava pojačanje (u cijelom rasponu) prema svjetlini cilja sve dok ne postavite `gain_db` ili `auto_gain: "Off"`. Kasniji `auto_exposure: "Continuous"` vraća ekspoziciju automatici, a `target_brightness_auto: true` vraća zadržani cilj automatskom cilju kChlorosu. Akcija `burst` uzima `name` (potrebno), `count` (zadano 5), `interval` (sekunde, zadano 0), `output`, `format`, i `settings` (isti oblik postavki po kameri kao `apply`); burstovi nizova koriste isti nadzor svježeg sinhroniziranog grupe kao `project burst`.

A `capture`, `burst` ili `stream` korak `output` zadano je `output`. Relativni `output` — uključujući zadano — rješava se u odnosu na mapu iz koje pokrećete `chloros-cli project run`, a ne na mapu datoteke recepta ili mapu projekta (u verziji 1.2.0 završio je u mapi za instalaciju pozadinskog sustava).

Pokrenite ga:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Svako pokretanje se prvo provjerava na projektu.** Svaka radnja mora biti točno jedan od gore navedenih glagola kao svoj ključ — `- capture: {name: FrontLeft}`, a ne `- action: capture` ili `- type: capture` — s `name` projektov `cameras.json` / `sensors.json` ima, kamerom ili nizom za `apply` / `capture` / `burst` / `stream` i senzor svjetla za `sensor`; `stream` treba `count` > 0, a `wait` treba trajanje. Recept s bilo kojom takvom greškom ne pokreće nikakvu akciju (`[recipe] error: …`, `[recipe] not run: …`, izlaz `1`), čak i s `stop_on_error: false`. Ključevi koje korak ne koristi su `[recipe] warning:`, kao i svaki niz ili senzor svjetla kojim upravlja recept, a koji trenutno nije priključen. Recept nikada ne povezuje uređaje: prvo pokrenite `chloros-cli project connect PATH`. `--dry-run` ispisuje recept, zatim iste pogreške i upozorenja i `[recipe] valid: …` / `NOT valid: …`, ne pokreće ništa i izlazi `1` kada recept ima pogrešku.

---

## Varijable okruženja

| Varijabla | Učinak |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Prepišite pozadinski URL (zadano `http://127.0.0.1:5000`) — **poštuje se samo u obiteljima naredbi `lattice`, `project` i `daq pool-*`** (`lattice network-analysis` umjesto toga koristi `--backend-url`). Osnovne naredbe (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) pin `http://127.0.0.1:<port>` i zanemaruju ovu varijablu (IPv4 literal zaobilazi kaznu od ~2 s-per-request u Windows `localhost`→`::1`), pa uvijek ciljaju lokalni računalo. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Postavi u okruženju pozadinskog sustava. `1` snižava odbijanje prekomjerne pretplate — niz povezan s `pin_resolution`, ili [privremena višekamera naredba](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — na glasno upozorenje, prihvaćajući gubitak GVSP paketa. Samo za testiranje — pogledajte [Model fps i burst za niz](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Postavite u okruženju pozadinskog sustava: `on` ili `off` postavlja zadano [spakirani 12-bitni transport](#packed-12-bit-transport---packed) za svaki niz (zadano `auto`). Eksplicitni `--packed` i dalje ima prednost. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (ili `true` / `yes`) isključuje [automatsku provjeru ažuriranja](#chloros-cli-update) Linuxovog CLIa pri pokretanju. |
| `CHLOROS_CLI_MODE` | Postavlja sama CLI; govori backendu da omogući paralelnu obradu. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` preskače GVSP fallback sondu (samo ICMP rezultati). **Ovo isključuje jumbo, ne samo da utišava zapisnik** — kamera odgovara na DF pingove samo do 1500 na svakoj ruti, pa je ova proba jedino što može otkriti jumbo. Štedi ~1 s po kameri po vezi; košta ~1,45× gornje granice mrežne propusnosti ako je mreža *mogla* prenositi jumbo pakete.SDK upozorava kada ga postavite. |
| `CHLOROS_GVSP_SESSION_REUSE` | Postavite u okruženju pozadinskog sustava: `0` mjeri veličinu paketa pri svakom povezivanju umjesto jednom po kameri po sesiji pozadinskog sustava. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Pričvrstite veličinu GVSP paketa na N bajtova; potpuno preskače probanje. Radije koristite po-naredbeno (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) nego trajno postavljanje: fiksirana veličina zaustavlja prilagodbu mreži ispred sebe, a postavljanje vrijednosti 9000 na putu koji ne može prenositi jumbo pakete uzrokuje da **svako** snimanje istekne s `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Postavite u okruženju pozadinskog sustava: najviše sekundi koje [privremeni nalog za više kamera](#multi-camera-transient-sessions), ili `lattice capture` kamere koju ništa ne drži, troši postavljanje ekspozicije svake kamere iz scene prije nego što zadrži kadar (zadano `5`). `0` ga isključuje: automatska ekspozicija svake kamere tada se pokreće od prvog kadra, počevši od svog ograničenja ekspozicije. |
| `CHLOROS_LOG_DIR` | Postavite u okruženju pozadinskog sustava: mapu u koju se spremaju njegovi detaljni zapisi. Zadani `%LOCALAPPDATA%\Chloros\logs` na Windows, `~/.cache/chloros/logs` na Linux (`$XDG_CACHE_HOME/chloros/logs` kad je to postavljeno). Korijenski servis paketa .deb postavlja `/var/log/chloros`, koji je čitljiv za grupu `adm` — pogledajte [Pokretanje backenda kao servisa](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Mjesto za privremene datoteke. CLI automatski koristi `/mnt/ssd/tmp` ako je prisutan. |

---

## Kodi izlaza

| Kod | Značenje |
| --- | --- |
| `0` | Uspjeh. |
| `1` | Opća pogreška (većina pogrešaka podzapovijedi). Zapovijed snimanja također završava `1` kada se bilo koja datoteka koju je zatraženo zapisati ne može zapisati, iako su ostale spremljene, i `array-burst` / `array-record` izlazi `1` kada snimač prijavi pogrešku — vidi [Datoteke koje se ne mogu zapisati](#files-that-fail-to-write). `process` također izlazi `1` kada se izvozi izvoze ne mogu provjeriti, ili kada je izgubio sliku, iako su ostali proizvodi napisani. |
| `2` | Pogreška u argumentu (nepoznata ili nedostajuća zastavica; također `time-sync set-priority` bez prioriteta). |
| `130` | Prekinuto tipkom Ctrl+C: bilo koja naredba koju prekine — `lattice`, `project`, `daq` i `process` jednako (`process` nakon što je pozadinski sustav zaustavio izvršavanje, vidi [`process` Bilješke](#notes)). Naredba čije normalno zaustavljanje je Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` i `project stream` s `-n 0` — izlazi na ono što je umjesto toga učinio: `0` kad je sve napisano, `1` kad god je nešto pošlo po zlu. |
| `143` / `129` | Ubijen signalom SIGTERM / SIGHUP (128 + signal), nakon što je završilo čišćenje naredbe: zaustavljen je aktivni posao i vraćene su njegove kamere, zaustavljeno je izvršavanje obrade u pozadini. `lattice continuous` ili drugo snimanje čije normalno zaustavljanje je Ctrl+C također izlazi iz `143`, a ne iz `0`. |

`lattice network-analysis` izlazi iz `1` za presude `needs_force_slip` i `error`, a `selftest` izlazi iz `1` osim ako ne prođu svih sedam provjera.

---

## Savjeti za rješavanje problema

- **"Potreban je prijavak"** → Pokrenite `chloros-cli login EMAIL PASSWORD` jednom na ovom računalu.
- **"Mapa projekta je preduboka za ovo računalo"** → Duge staze sustava Windows isključene su na ovom računalu (zadano u sustavu Windows), stoga nijedna staza ne smije prelaziti 260 znakova, a proizvodi se nalaze do otprilike 130 znakova ispod mape projekta. Ništa nije obrađeno. Poruka navodi najdužu mapu projekta koja odgovara: upotrijebite kraću (`-o` ili `chloros-cli set-project-folder`) ili uključite dugim putovima sustava Windows i ponovno pokrenite Chloros. Instalacijski program za Windows nudi ovu opciju kao "Omogući Windows dug putanje", označenu prema zadanim postavkama; inače upotrijebite Grupnu politiku: *Konfiguracija računala > Upravljački predlošci > Sustav > Datotečni sustav > Omogući Win32 dug putanje*.
- **"backend unreachable"** → Pokrenite desktop aplikaciju Chloros (Windows), ili sami pokrenite backend: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` na Windows (mapa `backend` vaše instalacijske mape; nije na `PATH`), `chloros-backend` na Linuxu. Ili provjerite `CHLOROS_BACKEND_URL` ako je udaljen.
- **`lattice` naredbe ne uspijevaju s porukom "LATTICE upravljački programi kamere nisu pronađeni"** → Backend nije uspio učitati LATTICE runtime za kameru s kojim je instaliran. Svaka naredba `lattice` izvršava se na pozadinskom sustavu (backendu); sama CLI nema ugrađene upravljačke programe za kamere. Ponovno instalirajte Chloros: instalacijski program za Windows i .deb paket instaliraju izvršno okruženje (runtime) uz pozadinski sustav (backend).
- **Povezivanje niza / Postavke niza prikazuju "FRAMES WILL DROP" ili "Smanjite ROI za omogućavanje"** → Prsten za primanje na NIC-u domaćina premalen je (često se vraća na 32 nakon ažuriranja upravljačkog programa za mrežnu karticu). Pogledajte [Postavljanje i podešavanje mrežne kartice hosta](#host-nic-setup--tuning-lattice-arrays) — postavite `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Računalo se zamrzne pri ponovnom pokretanju/isključivanju, a zatim WMI `Invalid class` / NIC se ne aktivira / USB pogoni nedostaju** → zastarjeli upravljački program USB 10GbE adaptera uzrokuje `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Ažurirajte upravljački program adaptera — pogledajte [Postavljanje i podešavanje mrežne kartice hosta](#host-nic-setup--tuning-lattice-arrays).
- **Linux: snimke pripadaju rootu, stoga ih ne možete izbrisati, a zapisnik pozadinskog sustava nije u vašem `~/.cache`** → Pozadinski sustav radi kao root putem `chloros-backend.service`. Prebacite se na `chloros-backend@<user>.service`, koji ga pokreće kao vas — pogledajte [Pokretanje backenda kao usluge](#running-the-backend-as-a-service-headless). Do tada je zapisnik usluge root u `/var/log/chloros` (čitljiv za grupu `adm`).
- **Linux: back-end jedinica je `failed` s porukom "Another instance is already running" (status izlaska 98)** → Drugi back-end već drži priključak: u isto vrijeme radi samo jedan. Ako su omogućene dvije backend jedinice, zadržite jednu: `sudo systemctl disable --now <unit>` za onu koju ne koristite — pogledajte [Pokretanje backenda kao usluge](#running-the-backend-as-a-service-headless). Ako je samo ova jedinica omogućena, backend pokrenut izvan sustava systemd (putem naredbi `chloros-backend`, `chloros-cli` ili skripte iz SDKa) ga drži: zaustavite taj backend, a zatim upotrijebite naredbu `sudo systemctl start <unit>`.
- **Upozorenje o zamjeni na Jetsonu** → Dodajte swap podržan datotekama; CLI ispisuje točne `fallocate` / `swapon` naredbe.
- **"backend nije dostupan" iz naredbe `lattice`, `project` ili `daq`** → nijedan backend nije pokrenut: te naredbe ga nikada ne pokreću. Pokrenite `chloros-cli backend status` i pogledajte [Pokreni pozadinski sustav](#start-the-backend).
- **"`daq net ...` nije dostupan u Chloros"** (Linux i Jetson) → Neke od postavki samog DAQ-E uređaja rezervirane su za uslugu MAPIR. Postavke mreže, ponovni pokreti i ažuriranja firmvera su `pool-set-network`, `pool-reboot` i `pool-update-firmware`; za ostalo kontaktirajte podršku za MAPIR.
- **"Naredbe za izravno upravljanje hardverom DAQ-a zahtijevaju cijeli paket daq"** (Windows) → Koristite `pool-*`, koji upravlja istim senzorom putem pozadinskog sustava (backend), ili `chloros_sdk.connect_daq_sensor()` iz Pythona.

---

## Pogledajte također

- [Referentni vodič za Python SDK](sdk-reference.md) — programski ekvivalent svake naredbe CLI.
- [DAQ senzori svjetla](https://mapir.gitbook.io/chloros/daq-light-sensors) u priručniku za Chloros — senzori, njihove kapice i kalibrirani raspon, snimanje, DAQ-E umrežavanje.
- [LATTICE kamere](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) u priručniku za Chloros — povezivanje, postavke kamere, načini snimanja, nizovi.
- Online dokumentacija: `https://mapir.gitbook.io/chloros/cli`
