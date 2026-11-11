# Chloros Python SDK Referenca

**Verzija:**

1.3.0**Generirano:**2026-07-29 19:19 ·**Revidirano:** 2026-11-06**Paket:** `chloros-sdk` (PyPI)**Namijenjeno:** Optimizirano za LLM-ove; čitljivo za ljude.**Opseg:** Sve javne klase, funkcije i pomoćne metode iz paketa `import chloros_sdk`, s primjerima za kopiranje i lijepljenje koji pokrivaju obradu slika, upravljanje jednom kamerom, sinkronizirane nizove, DAQ senzora i automatizaciju projekta, plus `chloros-mcp` poslužitelj za AI asistente.

Ako trebate samo najvažnije, idite na:
- [Instalacija i brzi početak](#installation)
- [Smart-Connect za LATTICE nizove](#smart-connect-for-lattice-cameras)
- [Sjednice DAQ senzora](#daq-sensor-sessions)
- [Pomoćni alati za kamere](#camera-utilities)
- [Privremeni nizovi, profili poravnanja i indeksi](#transient-arrays-alignment-profiles--indices)
- [Projektna mapa i recepti](#project-folder--recipes)
- [Automacija projekta](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Slike uživo u vlastitom procesu obrade](#live-frames-in-your-own-pipeline)
- [MCP poslužitelj (AI asistenti)](#mcp-server-ai-assistants)

> **Promjene u verziji 1.3.0** (što postojeća skripta verzije 1.2.0 primjećuje).
>
> **Promjene koje skripta verzije 1.2.0 možda treba obraditi:**
>
> - **Greška koja se javlja, a koju pozadinski sustav sada ne potvrđuje.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` na nizu koji je sesija otvorila, `DAQSensorSession.disconnect()` / `force_disconnect()` na senzoru koji je otvorio, i `ChlorosProject.disconnect_all()` izazivaju `ChlorosConnectError` kada pozadinski sustav odbije oslobađanje ili ga ne potvrdi na vrijeme, jer uređaj može i dalje biti zadržan. U verziji 1.2.0 tiho su vraćeni. Napuštanje bloka `with` i dalje nikada ne pokreće događaj.
> - **Posuđeni uređaj nikada se ne ruši.** `with attach_array(...)` i eksplicitni `disconnect()` na njemu ostavljaju niz u pogonu (u verziji 1.2.0, napuštanje bloka ga je rušilo, čak i kada je GUI bio njegov vlasnik); pozovite `force_disconnect()` kada ga namjeravate završiti. Isto tako, `connect_daq_sensor()` koji je pronašao senzor već u skupu, uključujući i pametno otkrivanje (smart-detect) s zastavicom nula, ostavlja ga otvorenim na `disconnect()` i pri napuštanju njegovog `with` bloka.
> - **PNG i JPEG koriste samo `display` i indeksne proizvode.** `capture()` s `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) i razine `radiance` ili `reflectance`, ili `processing="all"`, podiže `ValueError` prije bilo kakvog zahtjeva; 12-bitni `raw` / `debayered` odbijen je od strane pozadinskog sustava. Vidi [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` otvara taj senzor ili podiže** `ChlorosConnectError` navodeći što je pronašao; nikada ne prelazi na drugi senzor. Vidi [Pinned Transport](#pinned-transport).
> - **`frame_stream()` javlja grešku umjesto da utihne:** `FrameFetchError` pri odbijanju bi se sljedeći tik ponovio (reflektancijsko očitavanje bez ikakvog DAQ-a), nakon 60 s bez frejma i nakon 60 s bez novog okidača. Vidi [Jedan okidač po očitavanju](#one-trigger-per-read--group_id).
> - **Istek vremena čitanja ili prekinuta veza pokreće `ChlorosConnectError`** iz svakog poziva smart-connecta i `analyze_array_network()`, gdje je verzija 1.2.0 pokrenula osnovni istek vremena ili pogrešku veze.
> - **`connect_array()`**po zadanome čeka 600 s (`timeout`, 180 s u verziji 1.2.0), a povezivanje na kojem odustane poništava se.** Poziv šalje svoj `timeout` na pozadinski sustav, koji vraća svaku kameru u stanje u kojem je pronađena i ne povezuje ništa kada se veza ne uspostavi na vrijeme; poziv podiže `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`). U verziji 1.2.0, povezivanje odmah nakon uključivanja kamera moglo je potrajati duže od poziva i povezati niz nakon što je poziv prekinut. `connect_all()` čeka svaki spremljeni niz na isti način (120 s u verziji 1.2.0). Vidi [potpis `connect_array()`](#connect_array-signature).
> - **`connect_array()` zadržava cijeli senzor.** Bez `width` / `height` / `binning` svaka je kamera postavljena na cijeli senzor i brzina sličica pada na ono što host i mreža mogu isporučiti; 1.2.0 sužava ili spaja (binuje) sličice kako bi održao brzinu; vidi [Synchronized Array](#synchronized-array--arraysession-smart-prep).
> - **Bayerov pikselni format na M3M (mono) kameri je njezina dubina.** M3M nudi samo mono formate, a unaprijed postavljene postavke nazivaju Bayerove: u verziji 1.2.0 kamera je odbila format i zadržala svoj, dok je `settings.pixel_format` nazvan Bayer formatom. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, a `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` i niz&#x27;s zajednički format sada postavlja Mono format iste dubine na M3M: `"BayerRG8"` i `high_speed` daju `Mono8`; `"BayerRG12"`, `high_quality` i `triggered` daju `Mono12`; `cam.settings.pixel_format` ga imenuje. Isto vrijedi i obrnuto: M3C se nikada ne postavlja na Mono format, pa `"Mono12"` -- iz bilo kojeg od ovih, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` ili spremljeni format niza -- daje `BayerRG12` na M3C, koja god kamera je master niza. `CameraSettings` koji proslijedite zadržava svoje ime, pa isti objekt i dalje traži tu dubinu od svake kamere u svojoj obitelji. `connect_camera(preset=...)` zadržava format postavke (1.2.0 postavlja kameru na 12-bit nakon primjene preseta), a kamera otvorena na presetu više ne zapisuje ono što je uzela u `PRESETS`. Bilo koji drugi format koji kamera odbija (jedan od njih nedostaje na popisu `PixelFormat`, kao što je `RGB8` na M3M-u ili `BayerRG16` na firmwareu bez njega) također nije nazvan po sebi: nakon `LatticeCamera` connect, `apply_settings()` ili `set_format()`, `cam.settings.pixel_format` i `packed_transport` nazivaju format kojim kamera prenosi, a jedna ispisana linija naziva format koji je odbio.
> - **`analyze_array_network()` prikazuje što se primjenjuje na &#x27;connect&#x27;:** `pixel_format=None` (bivši `"BayerRG8"`) znači formate koje &#x27;connect&#x27; postavlja (Mono12 / BayerRG12), a `packed="auto"` odluku o spakiranom prijenosu, pa se fps koji navodi može razlikovati od 1.2.0&#x27;. `pixel_format` koji proslijedite projektima vraća M3M na toj dubini. Pogledajte [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` vraća `CaptureResult`** (još uvijek popis frame dictova) s `.skipped`, `.failed`, `.daq_files` i `.daq_matches`. `ArraySession.capture()` rezultat također nosi `.alignment` (je li primijenjena deformacija poravnanja i zašto nije) i `.oriented`.
> - **Capture nizova su poravnati prema zadanim postavkama, kao i u aplikacijama.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` i `ArrayHandle.capture()` / `capture_smart()` warpaju `debayered`, `display` svakog člana, `radiance`, `reflectance` i indeksne datoteke u kadar glavne kamere i izrezati ih na preklapanje članova (manji od senzora za količinu pomaka kamera, `XMP-mapir:AlignmentJSON` `"applied": true`), pa su to ista datoteke kao i snimka iste scene aplikacije Chloros; `raw` ostaje u izvornim dimenzijama. Prije je snimanje bez `aligned=True` spremalo svaku datoteku u izvornim dimenzijama, neizobličenu. `aligned=False` to zadržava. `chloros-mcp`&#x27;s `capture_array` i `start_capture_run` imaju istu zadanu postavku. Pogledajte [`ArraySession` Methods](#arraysession-methods).
> - **`connect_all()` navodi backend koji odbijaju SDK.** Pokrenuti backend koji je odjavljен ili je na planu nižem od Coppera izvještava se pod `errors` sa svojim razlogom (HTTP 401 / 403), a ne kao &quot;nema aktivnog Chloros backenda&quot;, i svaki poziv `connect_all()` ponovno provjerava backend, tako da pozivanje nakon prijave radi na istom objektu projekta.
> - **Relativni `output_dir` radi na lokalnom backendu.** Relativne putanje prije slanja pretvaraju se u apsolutne u odnosu na radni direktorij vaše skripte. U verziji 1.2.0 backend ih je rješavao u vlastitom instalacijskom direktoriju, što je rezultiralo greškom &quot;Pristup je uskraćen&quot;.
> - **`numpy` sada je obavezna** komponenta SDK-a (pozivi za živi kadar vraćaju numpy nizove).
> - **Odbijeno je ime razine izvoza koje nije jedno od predviđenih.** `processing=` i `levels=` uzimaju `raw`, `debayered`, `radiance`, `reflectance` ili `display` (ili alias kao što je `preview`; `processing=` također `"all"`). Bilo koje drugo ime je odbijeno prije nego što se dotakne kamera (HTTP 400 `UNKNOWN_LEVEL`: `capture()` izaziva `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 spremio ili vratio `raw` za to.
> - **Texture Aware je odbijen za LATTICE snimke.** Na projektu koji sadrži LATTICE snimke, `configure(debayer="Texture Aware (Slow, Highest Quality)")` pokreće `ChlorosConfigurationError`, kao i `process()` nakon `configure()` koji je zatražio za to prije uvoza. Vrijednost Texture Aware koju je projekt ranije spremio postavljena je na Standard, s `UserWarning`. LATTICE snimke uvijek dobivaju standardni demosaicing, pa je verzija 1.2.0 prihvatila postavku i zanemarila je; aplikacija Chloros je ni za njih ne nudi. Vidi [Podržane vrijednosti](#supported-values).
> - **Kamere se vraćaju onakvima kakve su pronađene — osim ako `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, kraj svake prolazne funkcije (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) i zaustavljanje `start_camera_continuous()` vraćaju svaku kameru u prvobitno stanje: njezino ožičenje okidača i sinkronizacijske linije, njezin PTP i svaku postavku kamere koju je sesija promijenila (ekspoziciju i pojačanje, AE cilj, ograničenja i prigušivanje, bilans bijele, razina crne, format piksela, biniranje i ROI, preokreti, brzina snimanja). `keep_settings=True` umjesto toga sprema postavke svake kamere u vlastitu memoriju kamere, tako da se ona pokreće s njima; `clear_settings=True` resetira tu memoriju na tvorničke postavke. U oba slučaja kamere se ostavljaju da rade neovisno. Projekt postavlja kamere prema vlastitim spremljenim postavkama kada ih poveže. Vidi [Memorija kamere](#camera-memory--keep_settings--clear_settings).
> - **Svaki način rada sprema projekt na isti način.** Svaka promjena na kameri, nizu ili senzoru svjetla povezanom u okviru projekta — iz SDK-a, aplikacije, CLI-a ili MCP poslužitelja — sprema se u `cameras.json` / `sensors.json`, a otvaranje projekta u bilo kojem načinu rada postavlja uređaje iz njega; `ChlorosProject` imenuje vlastiti mapu na pozadinskom sustavu. Prije je projekt spremala samo aplikacija. Vidi [Jedan zapis projekta, svaki način rada](#one-project-record-every-mode).
> - **Datoteke iz više kamera navode svaku pojedinačno.** Stog više pojaseva, indeks uživo napravljen od pojaseva više kamera i slika iz niza snimanja `composite/` nose serijski broj, model i paket kalibracije svake izvorišne kamere kao popise (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), a EXIF `SerialNumber` popise serijskih brojeva, spojene zarezom. Prije nisu sadržavali nikakav serijski broj kamere, ili samo serijski broj prve kamere. Vidi [Datoteke iz više kamera](#files-made-from-several-cameras).
> - **Samo datoteka `raw` sadrži oznake kalibracije kamere.** Kompatibilan skup za MicaSense (`BlackLevel`, `RadiometricCalibration`, per-kanalni `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*` setovi) i teret `CalibrationJSON` opisuju sirove vrijednosti senzora, stoga se nalaze u `raw`samo  datoteku za snimanje. Svaka druga datoteka — snimanjes `debayered`, `display`, `radiance`, `reflectance` i indeksne datoteke, kontinuirano snimanje i višekamera-snimke okvira, i svaki `process()` proizvod — nosi identifikaciju kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) i, na snimci, vlastiti zapis okvira (`XMP-mapir:FrameJSON`: ekspozicija, pojačanje, temperatura, format piksela, vrijeme snimanja). U verziji 1.2.0 prenijeli su i raw skup, koji alat koji ga primjenjuje (Pix4D, Metashape) primjenjuje na piksele koji nemaju raw vrijednosti. Processing pronalazi kalibraciju takve datoteke prema njezinu serijskom broju i `CalBundleSha256`.
>
> **Ostale promjene:**
>
> - **Automatska brzina snimanja niza prilagođava se onome što niz isporučuje.** Brzina koja izgubi više od 1 % svojih okidača se smanjuje, pri čemu se prosudba temelji na nekoliko duljih pregleda umjesto na jednom, a brzina s koje je smanjena sada se s vremena na vrijeme ponovno isprobava. Brzina koju sami postavite ostaje nepromijenjena.
> - **Automatska ekspozicija je stabilnija.** Drži konstantnu svjetlinu, prilagođava se unutar nekoliko sekundi nakon promjene, zanemaruje male odbljeske i vraća rad kamere koja prestane reagirati na jakom suncu. Pogledajte [Kako se ponaša automatska ekspozicija](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` računa samo vaše slike i snimke.** Mapa koja sadrži projektov `camera_cal/` više ne računa te datoteke za kalibraciju kao slike i snimke, a projekt koji ih je već uvrstio uklanja ih.
> - **Primjenjuje se `configure(radiometric_output=...)`.** Sada postavlja proizvode projekta `Export radiance` / `Export reflectance`; prije su nadjačavala postavke proizvoda svakog pojedinog projekta, tako da su `radiance`, `sensor-response` i `all` izvezli radijancu i reflektancu.
> - **`test_array_sync()` prelazi iz stanja uključivanja napajanja kamera.** Uključuje PTP kamere za vrijeme rada, čeka da se njihovi satovi zaključaju i stabiliziraju prije prvog okidača (`ptp_lock`) — svaka kamera zaključana je 120 s, otprilike 2 minute od uključivanja (`ptp_settle_s=` ga postavlja), budući da je sat koji se upravo zaključao isključen do ~200 µs na otprilike 2 minute — i vraća PTP svake kamere u stanje u kojem ga je zatekao. Kamera koja je još zaključana iz prethodnog testa više se ne čeka. Prije je ostavljao PTP isključenim na mreži s jednim prekidačem na kojoj nije odgovorio nijedan DAQ-E i čitao slobodno tekući sat svake kamere kao PTP sat koji se nije mogao zaključati (&quot;PTP nije zaključan&quot;, svaki pokret).
> - **Zapis `.daq` navodi je li njegov kapak odabran ili pretpostavljen.** DAQ-U, DAQ-M i DAQ-E isporučuju se bez kapa, a Chloros ne može vidjeti je li kapa ugrađena, pa senzor priključen bez kapa dobiva `sunshine_cosine` prema zadanim postavkama, što je *pretpostavka*. Svako snimanje pomoću Chloros (shema 1.30) sada pohranjuje `cap_id_source` uz `cap_id`: `operator` kada ste naznačili kapu (uključujući `none`), `auto_default` kad je bio zadani. Prilikom obrade snimke čiji je kondenzator pretpostavljen, a DAQ kondenzator projekta ostavljen na Auto, ispisuje se `[daq-dls] WARNING` imenujući datoteku: goli DAQ-U pod korekcijom sunčeve svjetlosti čita otprilike 30 puta više. **Navedite kap koji ste ugradili** pri povezivanju (`connect_daq_sensor(cap_id=...)` ili `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, ili iz padajućeg izbornika kapice GUI-ja), ili postavite DAQ kapicu projekta i upozorenje nestaje.
> - **Više nema pip konzolnih prozora na Windowsu.** Na računalu s instaliranim Pythonom, pozadinski program i svaki od njegovih radnika za slike jednom su pokrenuli `pip` pri pokretanju i svaki je otvorio konzolni prozor. To više ne čine.
> - **Snimke uživo XPROT000304 preživljavaju nestanak struje.**Snimanje se zapisuje s dnevnikom i potpunom sinkronizacijom diska pri svakom commitu, iza senzora, tako da spori disk odgađa datoteku, a nikada očitanja. Snimanje zaustavljeno usred commitiranja (prijemnik struje, prekid procesa) može ostaviti `<name>.daq-journal` pored njega:**kopirajte oboje zajedno.**Chloros vraća dnevnik unatrag prvi put kad pročita datoteku, tako da se snimanje otvara sa svim podacima zabilježenima prije zaustavljanja (izgube se otprilike posljednje 3 s); u verziji 1.2.0 prekid napajanja u pogrešnom trenutku mogao je ostaviti cijelu datoteku nečitljivom (&quot;database disk image is malformed&quot;).
> - **`DAQSensorSession.record_start()` na zauzetom disku više ne blokira senzor**: Očitanja se stavljaju u red i zapisuju na vlastitoj niti, a ona koja se nisu mogla zapisati broje se u `status()`-ov `record_errors` (s `record_last_error`).
> - **Direct `LatticeCamera`:**okvir bez chunk ekspozicije/pojačanja sada izvještava o**stvarnoj** ekspoziciji i pojačanju kamere (raniji način rada čita pohranjenu vrijednost koja je mogla ostati nepromijenjena tijekom cijelog automatskog izlaganja), a `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` primjenjuju isti 5 µs deglitch ulaza koji dobivaju kabelom sinkronizirani roboti tvrtke Chloros (nefiltrirano, robot je ispalio kadre bez poslanog okidača).
> - **Kamere s softverskim okidačem više ne propuštaju svaki drugi okidač.** Kamera u načinu softverskog okidača sada stavlja u redok okidač koji stigne tijekom čitanja (ili Chloros čeka da bude spreman), kao što su to već radile kamere sinkronizirane kabelom.
> - **`discover_daq_sensors()` imenuje DAQ-E-S `DAQ-E-S`.Njegov `model` bio je `DAQ-E` za svaki Ethernet senzor; onaj koji ne odgovori na provjeru identiteta unutar otprilike 2 s i dalje je naveden kao `DAQ-E`.
> - **Novo: `chloros-mcp`, MCP poslužitelj za AI asistente.** Claude Desktop, Claude Code, Cursor i drugi MCP klijenti mogu obrađivati slike (izraditi projekt, uvesti snimke, odabrati postavke, obraditi ih i izvijestiti o tome što je izvoz izvršen) te upravljati LATTICE kamerama, nizovima i DAQ senzorima svjetla: povezivanje, pretpregled, postavljanje ekspozicije i boje, snimanje, zaprimanje. Njegovi alati pokrivaju ono što radi ovaj SDK. Instalira se zajedno s SDK-om, a .deb za Linux stavlja ga u `/usr/bin`. Vidi [MCP Server (AI Assistants)](#mcp-server-ai-assistants).
> - **Novo: pomoćni programi za kamere**, `chloros-cli lattice` naredbe kao pozivi SDK-a: profil boja, zasićenost / kontrast, naglašeni tonovi, balans bijele (jedan snimak, automatski, fiksni omjeri), unaprijed postavljeni profili, modeli filtera, status firmvera i ažuriranje, napajanje (provjera, spavanje, buđenje), kalibracija cilja, mrežni adapteri (provjera, procjena, popravak), korekcija vinjete na datotekama, čitanje DLS-a i proba hosta. Vidi [Korisnički programi za kameru](#camera-utilities).
> - **Novo: ostatak CLI-a kao SDK pozivi**, tako da CLI, SDK i `chloros-mcp` nude iste značajke (CLI zadržava samo prijavu, postavljanje DAQ-E-a i izravne DAQ poslužitelje, radi sigurnosti): kamere otvorene za jednu naredbu (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), datoteke profila poravnanja (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indeksi (`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP velemajstor (`time_sync_status()` i prijatelji), mapa projekta (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) i recepte (`run_project_recipe()`, vlastiti runner CLI-a). Za senzore svjetla: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` i `light_sensor_reflectance()` (reflektancija i indeksi okoline + objekta za dva senzora); za kameru: `CameraSession.status()` i `capture_repeated()`; za niz: `ArraySession.capability()` i `set_member_settings()` (ekspozicija, pojačanje člana, ...). Pogledajte [Privremeni nizovi, profili poravnanja i indeksi](#transient-arrays-alignment-profiles--indices), [Mapa projekta i recepti](#project-folder--recipes) i [DAQ senzorske sesije](#daq-sensor-sessions).
> - **Poboljšani LATTICE indeksni proizvodi** (NDVI i drugi indeksi) iz `process()`, u svakom načinu obrade.
> - **LATTICE OCN kamere: poboljšani proizvodi radijance, reflektance i indeksa na sunčevoj svjetlosti.** Korekcija flote primjenjuje se nakon odvajanja po pojasu — uživo, u snimkama i u `process()`; RGN kamere ostaju nepromijenjene. Vidi [Radiometrijski izlaz](#radiometric-output-lattice-multispectral-pipeline).
> - **Svako snimanje sprema DAQ očitanje koje je upotrijebilo, kao što to radi GUI.** `CameraSession.capture()` i `ArraySession.capture()` / `continuous()` zapisuju `.daq` i njegov `.match.json` kad god je DAQ povezan ili grupiran, uključena je snimka samo u sirovom formatu; u 1.2.0 sirovu ili debayered snimku sprema samo `force_daq=True`. `daq=False` ga isključuje, a `daq_tolerance_ms=` postavlja toleranciju podudaranja; oba rezultata navode što je spremljeno u `.daq_files` i `.daq_matches`. Snimanje sada čeka sljedeće očitavanje DAQ-a kako bi moglo spojiti očitanja s obje strane ekspozicije. Vidi [Očitavanje DAQ-a koje snimanje sprema](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` ima live okvire i kontinuirano snimanje na strani poslužitelja.** `grab()` i `frame_stream()` vraćaju numpy okvire iz jednog okidača (isto čitanje kao `ArrayHandle.grab()` / `frame_stream()` projektnog API-a), a `continuous()` sprema svaku sinkroniziranu grupu u pozadini i vraća [`ContinuousHandle`](#continuoushandle). Pogledajte [Žive okvire u vlastitoj cijevi](#live-frames-in-your-own-pipeline).
> - **Niz `grab()` je prikvačen za jedan okidač** (`ArrayHandle.grab()` i novi `ArraySession.grab()`). U verziji 1.2.0, 12 od 20 četverih-kamera `ArrayHandle.grab()` čita straddled dva okidača. Svaki kadar sada nosi `group_id`, a niz `frame_stream()` vraća svaki okidač jednom. Vidi [Jedan okidač po čitanju](#one-trigger-per-read--group_id).
> - **Okviri po pojasu i poravnati okviri:** `grab()` / `frame_stream()` uzimaju `band=`, `align=`, `bit_depth=` i `refl_max=` na rukovanjima kamere i niza. Vidi [Izvoz pojedinačne trake](#exporting-a-single-band--band).
> - **Nove metode i argumenti `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (s `CaptureResult.group_seq` i `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` lanči uzastopne grupe i zaustavlja se na punoj pogonici.
> - **`ArraySession.record()` bilježi bez otvorenog GUI-ja.** Kada je kombinirani prikaz GUI-ja zatvoren, pozadinski sustav prikazuje sam prikaz: nizNDVI ili indeks koji proslijedite kao `expr=`. `RecorderHandle.stop()` pokreće `ChlorosConnectError` za snimanje koje je propalo, ono koje nije uključivalo nijedan okvir, i zadržava rezultat zaustavljanja na `.result` (napuštanje bloka `with` ne pokreće događaj: provjerite `rec.result`). Vidi [`RecorderHandle`](#recorderhandle).
> - **Nove metode `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()`, i za vlastite postavke DAQ-E-a `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` i `update_firmware()`. `record_stop()` vraća `warning` i `gaps`. Ponovno pokretanje koje je zatvorilo snimanje naziva ga, u svom odgovoru ili, kada ponovno pokretanje ne uspije, na `ChlorosConnectError` (`recording_closed`; vidi [Postavke uređaja DAQ-E](#daq-e-device-settings)). Vidi [`DAQSensorSession` Methods](#daqsensorsession-methods).
> - **DAQ-E u zajedničkom fondu koji prestane isporučivati ponovno se povezuje putem pozadinskog sustava**, s postavkama koje je imao, a snimanje na njemu se nastavlja u istom `.daq` s prazninom označenom u datoteci; `record_stop()` vraća praznine kao `gaps`. U verziji 1.2.0 senzor je ostao neaktivan dok se nije odspojio i ponovno povezao, što je prekinulo snimanje. Pogledajte referencu za CLI. [DAQ-E koji prestaje isporučivati](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` radi s instaliranog SDK-a.** Kolo (instalater i PyPI-ja) sada sadrži projektne učitavače koji su mu potrebni; prije je `open_project()` pokretao `ImportError` nazivajući `project_cameras` / `project_sensors` u bilo kojem pip-instaliranom SDKu. `connect_all()` povezuje spremljene kamere, nizove i senzore svjetla putem aktivnog backenda (svjetlosni senzori putem njegovog DAQ pool-a), i spremljeni uređaj za koji nema načina da ga pronađe naveden je pod `errors` s razlogom umjesto da bude preskočen bez riječi. Spremljena matrica koja je već aktivna u pozadini (GUI-jevi, ili `chloros-cli project connect`-ovih) je priključen i ostaje u pogonu, dok je prije prijavljivan kao pogreška (&quot;Kamera je<sn> već u nizu<id>&quot;). Pogledajte [`ChlorosProject` Metode](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` i `process_lattice_capture()` pronalaze Chloros na Windows.** Provjeravaju gdje instalator smjesti backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` prema zadanim postavkama, prilagođenu mapu iz vlastite evidencije instalatera), i povezuju se s pozadinskim procesom koji je već u tijeku, a da ga uopće ne traže. U verziji 1.2.0 pojavila se poruka &quot;Chloros pozadinski proces nije pronađen&quot; na instaliranom računalu, osim ako niste proslijedili `backend_exe=`.
> - **Backend koji se zatvori pri pokretanju odmah se prijavljuje.** Kada `ChlorosLocal()` ili poziv smart-connect pokrene backend koji se zatvori prije nego što odgovori, poziv se javlja unutar nekoliko sekundi, `ChlorosBackendError` iz `ChlorosLocal` i `ChlorosConnectError` iz pametne-connect pozivi, s kodom izlaska, naredbom koju je pokrenuo i posljednjim redovima koje je pozadinski proces isprintao. Verzija 1.2.0 je pričekala 60 s za vrijeme pokretanja i nije navela razlog. Vidi [Automatsko pokretanje pozadinskog procesa](#backend-auto-start).
> - **Backend nastavlja raditi između skripti.** Napuštanje bloka `with ChlorosLocal(...)`, vraćanje `process_folder()` / `process_lattice_capture()` i izlazak `chloros-mcp` više ne zaustavljaju backend koji je pokrenuo SDK; sljedeći skripta ga ponovno koristi. Zaustavlja se na `logout()` (osim ako aplikacija Chloros nije otvorena ili je izvršavanje u tijeku), na `shutdown_backend()`, ili samostalno nakon 30 minuta ako ga nitko ne koristi (`CHLOROS_BACKEND_IDLE_MIN` u okruženju backenda to mijenja; `0`: nikada).
> - **Backend koji je pokrenuo SDK traži se da se zatvori, a ne da se ubije.** `shutdown_backend()` dopušta backendu da oslobodi svoje kamere i senzore svjetla i dovrši bilo kakvo snimanje; prekida se samo ako i dalje radi 10 s kasnije. Verzija 1.2.0 prekidala ga je odmah (na Windows, prisilno ga se prekida), što može uzrokovati da DAQ senzor svjetla povezan s tim backendom ostane neaktivan. Backend koji SDK pronađe u pokretanju ostaje netaknut.
> - **Na Linux, automatski pokretanje nikada ne pokreće `backend_server.py` iz trenutnog mapa.** Bez instaliranog backenda, skripta pokrenuta u mapi koja je sadržavala jedan pokretala ga je; sada se pokreće samo instalirani backend (ili, za `ChlorosLocal`, onaj koji `backend_exe=` imenuje).
> - **`configure(indices=...)`**dodaje svaki preset u projekt za pojaseve njegove kamere, uključuje novi projekt i upozorava (`UserWarning`) na ime koje ne može dodati.****`import_images()`**izvještava o snimkama**`.daq`**kao**`scan_count`** i više ne upozorava na mapu za snimanje LATTICE.
> - **`read_image_audit_tags()` radi putem pip instalacije.** Samostalno čita metapodatke slike; u verziji 1.2.0 vraća `{}` bez unutarnjih dijelova desktop instalacije na putu Pythona.
> - **`capture()` odmah nakon `connect_array()`** čeka do 20 s na prvu grupu sličica niza umjesto da zakaže s porukom &quot;HTTP 503 &#x27;Niz se još zagrijava&#x27;&quot;.
> - **`status()` prijavljuje zastoj.** Novo polje `stalled` je True dok se ciklus hvatanja (grab loop) izvršava, ali ne objavljuje ništa, a `actual_fps` je tada 0. U verziji 1.2.0 `actual_fps` je zadržao svoju posljednju vrijednost tijekom zastoja, tako da je blokirani niz prikazivao `actual_fps > 0` dok su snimanja propadala s porukom 503 &quot;Nema nove sinkronizirane skupine kadrova&quot;.
> - **Manje nepotpunih skupova sličica i pogrešaka 503 pri snimanju:** ulaz za okidanje svake robne kamere sada je filtriran.
> - **Pakirani 12-bitni transport** (`connect_array(packed="auto")`) povećava brzine sličica cijelog senzorskog niza za otprilike trećinu. Vidi [Pakirani 12-bitni transport](#packed-12-bit-transport-packed).
> - **Ekspozicija nizom unutar ROI-ja:** mali ROI više ne ograničava ekspoziciju člana na razdoblje slobodnog rada senzora (4,9 ms pri 768 × 304). Vidi [Ekspozicija na članovima niza](#exposure-on-array-members).
> - **Automatsko izlaganje više ne smanjuje automatsku brzinu snimanja niza.**Niz sinkroniziran putem kabela radi brzinom koju dopuštaju njegova mreža i očitavanje s kamera; automatsko izlaganje popunjava razdoblje koje ta brzina daje, a zatim povećava pojačanje. Ručno izlaganje duže od razdoblja usporava okidač kako bi se uklopio u njega, umjesto da bude odbijeno. Vidi [Izlaganje na članovima niza](#exposure-on-array-members). -**`read_nodes()`**dodaje `frame_values` i `notes` dok automatsko izlaganje radi (vidi [`read_nodes()` pod automatskim izlaganjem](#read_nodes-under-auto-exposure--frame_values-and-notes)). Kada kamera odbije ekspoziciju ili pojačanje,**`set_settings()`** navodi raspon koji prihvaća i vraća automatski način rada koji je bio isključen za tu vrijednost; `set_settings(target_brightness_auto=True)` je nov.
> - **`orient=`** na `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` i `ArrayHandle.capture()` / `capture_smart()` okreće proizvode prema orijentaciji kamere. Vidi [`ArraySession` Metode](#arraysession-methods).
> - **`DLS()` treba vlastiti `mac_address` senzora.** Više nema zadane MAC adrese (bila je adresa jedne jedinice); `DLS()` bez nje izaziva `DLSError`. Vidi [DAQ-M pojasne amplitude](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Instalacijski program za Windows** nudi opcije &quot;Omogući dugačke putanje u sustavu Windows&quot; i &quot;Isključi pozadinski sustav Chloros iz skeniranja Microsoft Defendera&quot;, a obje su označene prema zadanim postavkama. Vidi [Korak 1](#step-1--install-the-chloros-platform-package).
> - **Linux .deb** instalira Python SDK za sustav `python3` (ranije je samo smjestio kotač na disk), povećava kernelove soketne prijemne buffer-e koji su potrebni za streamove s kamere i dodaje pozadinsku uslugu koja se pokreće kao vaš korisnik. Jetson paket je `chloros_1.3.0_arm64_jp6.deb`, za JetPack 6, s JetPackovim CUDA PyTorchom. Pogledajte [Korak 1](#step-1--install-the-chloros-platform-package) i [Korak 2](#linux-deb).

---

## Arhitektura u 60 sekundi

SDK je tanak sloj iznad Chloros backenda (isti Flask poslužitelj koji koriste desktop GUI i CLI). Za automatizaciju uvozite `chloros_sdk` i pozivate metode visoke razine; u pozadini, svaki poziv postaje HTTP-zahtjev prema lokalnom backendu na portu 5000 — `http://127.0.0.1:5000/api/...` (namjerno ne `localhost`, koji se na Windowsu prvo rješava u `::1` i košta ~2 s po zahtjevu prema backendu koji podržava samo IPv4). Backend upravlja hardverskim bazenom — kamerama, DAQ senzorima, profilima poravnanja, spremnicima okvira — tako da skripte SDK-a mogu postojati zajedno s GUI-jem bez borbe za serijske priključke ili propusnost mrežne kartice (NIC).

Postoje tri sučelja koja ćete koristiti:

1. **`ChlorosLocal` + slobodne funkcije** (`process_folder`, `process_lattice_capture`) — Cijev za obradu slika. Pokrenite cijelu mapu kroz kalibraciju / debayer / izvoz indeksa iz jednog poziva u Pythonu.
2. **Smart-connect upravljačke točke** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otvori trajnu pozadinsku sesiju za hardver uživo. Isti &quot;smart-prep&quot; tijek kao u GUI-ju: mrežni prob, automatski odabir sloja, PTP, AE seeding, konfiguracija GPIO okidača.
3. **`ChlorosProject` / `open_project`** — Učitavanje spremljenog projekta (mapu s `cameras.json` + `sensors.json` + `project.json`), istovremeno povezivanje njegovih kamera, nizova i senzora svjetla te upravljanje snimanjem pomoću imenovanih rukohvata.

Površine 1 i 2 automatski pokreću lokalni backend ako on već ne sluša (isti priloženi binarni program koji pokreće GUI/CLI) — tako da gola skripta radi iz novog terminala bez da prvo pokrenete backend: `ChlorosLocal` (i `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` i `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` i `analyze_array_network()` nikada ne pokreću nijedan: potrebna im je pokrenuta pozadinska usluga. Proslijedite `auto_start_backend=False` da biste isključili (npr. kada upućujete na udaljeni backend, koji se nikada ne pokreće). Pogledajte [Automatski pokretanje backenda](#backend-auto-start)Surface 3 se ponaša drugačije: `open_project()` ne prihvaća `auto_start_backend` parametar, a `connect_all()` nikada ne pokreće backend — jednom ispituje `http://127.0.0.1:5000` i, ako nitko ne odgovori, vraća se na izravnu (bez backenda) kontrolu uređaja `lattice_sdk`. To zahtijeva Arena SDK runtime i OpenCV (Python-u, koji se ne instalira pomoću SDK kotača; bez njih, svaka spremljena kamera i niz prijavljuju se kao pogreška, s razlogom. Svjetlosni senzori projekta nemaju put bez backenda: ako ne radi nijedan backend, svaki spremljeni senzor prijavljuje se kao pogreška, s razlogom (vidi [`SensorHandle`](#sensorhandle)). Samo `proj.process()` i `stream(..., overlays=True)` lijeno konstruiraju `ChlorosLocal()` (koji se automatski pokreće).

Sva tri su auth-gated: pokrenuti `chloros-cli login` jednom na računalu ili se prijaviti putem desktop GUI-ja. Ono što poziv bez važeće sesije izaziva ovisi o površini (vidi [Izuzeci](#exceptions)): `ChlorosAuthenticationError` iz `ChlorosLocal` (i `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` iz pametnog-pozivi za povezivanje i njihove sesije, `status: "error"` s `error_code` iz `analyze_array_network()`, i unos pod `errors` u `connect_all()` izvješće.

Zahtjevi:
- Python 3.7+ (kako je navedeno u paketu; razvijeno/testirano na verziji 3.10)
- lokalno instaliran Chloros Desktop (binarna datoteka pozadinskog sustava isporučuje se unutar instalacijskog programa)
- aktivno **Chloros+**prijavljivanje na**bilo koju plaćenu **Chloros+**razinu (Copper ili višu)**; besplatna**Iron**razina nema pristup**SDK/CLI. Ovo se provodi **server-strana**: svaki zahtjev s oznakom SDK/CLImora sadržavati i aktivnu sesiju i plaćeni plan. Plan niži od Copper dobiva `403` s `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` iz `ChlorosLocal`, `ChlorosConnectError` iz poziva pametnog povezivanja). Pozivatelj koji je odjavljен dobiva `401` / `AUTH_REQUIRED` umjesto toga (`ChlorosAuthenticationError` iz `ChlorosLocal`, `ChlorosConnectError` iz poziva pametnog povezivanja) — ta su dva različita jer ponovno pokretanje `chloros-cli login` popravlja `401` i ne može popraviti `403`, za što je potreban nadogradnja plana.
- Podržana je izvanmrežna upotreba unutar razdoblja milosti plana: razina se čita iz predmemorije provjere poslužitelja (5 min) ili iz predmemorije potpisane licence vezane za računalo (30 dana za mjesečne planove, do isteka pretplate za godišnje). Kada taj period isteče, plan se svodi na besplatni i pristup SDKu/CLIu prestaje dok računalo ne uspostavi vezu sa poslužiteljem. `chloros-cli status` (`GET /api/license-status`) ostaje dostupan na besplatnom nivou kako bi razlog bio vidljiv.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS ili noviji**, ili Jetson (JetPack 6). Ubuntu 20.04**nije** podržan: ovisnosti `.deb`-a izvedene su iz onoga s čime se backend povezuje, uključujući `libc6 (>= 2.35)`, a focal isporučuje glibc 2.31.

---

## Instalacija

Pythonov SDK tanki je sloj nad Chlorosovim backendom. Osim ako ne pokrećete backend na drugom računalu (pogledajte [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel)), potrebno je lokalno instalirati **Chloros desktop paket** (Windows instalacijski program ili Linux `.deb`), a to uključuje DAQ senzore svjetla. Paket sadrži binarnu datoteku backenda, Arena SDK okruženje za izvođenje za LATTICE kamere i pakete za kalibraciju.

Najnovije preuzimanja: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Korak 1 — Instalirajte paket platforme Chloros

#### Windows (.exe)

1. Preuzmite `Chloros Installer 1.3.0.exe` s stranice za preuzimanje.
2. Pokrenite instalacijski program i slijedite čarobnjak. Zadana putanja instalacije je `C:\Program Files\Chloros\`.
3. Pokrenite Chlorosbarem jednom i prijavite se na svoj račun Chloros+.

Instalacijski program omogućuje pozadinskom sustavu (`chloros-backend.exe`, ništa drugo) putem WindowsFirewalla na svakom mrežnom profilu: UDP ulazno, za videozapise s kamera, PTP, mDNS i DAQ-E stream, i TCP ulazno samo iz lokalnog podmrežnog područja, za vezu koju DAQ-E uspostavlja s računalom tijekom ažuriranja firmvera ([`update_firmware()`](#daq-e-device-settings)). Deinstalator uklanja oba pravila. HTTP-API pozadinske aplikacije ostaje na `127.0.0.1` u svakom slučaju.

Čarobnjakova grupa **Sistemski postavke** ima dvije opcije, obje označene prema zadanim postavkama:

| Opcija | Što radi | Odustajanje od tihe instalacije |
| --- | --- | --- |
| **Omogući duge putanje sustava Windows** | Omogućuje da se mape projekta nalaze dublje od ograničenja putanje sustava Windows od 260 znakova. Kada su duge putanje onemogućene (zadano u sustavu Windows), pristup mapi projekta koja je previše duboka za njegove proizvode bit će odbijen prije nego što se išta obradi — pogledajte [Rješavanje problema](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Isključi pozadinski program Chloros iz skeniranja Microsoft Defendera** | Zaustavlja stvarno-vremensko skeniranje svake datoteke koju snimanje kamere zaprimi. Pri visokim brzinama snimanja skeniranje košta oko 30 % brzine: izmjereno na ROI 512 na nizu od 9 kamera, kontinuirano snimanje sirovih podataka radilo je 52,7 snimaka/s sa skeniranjem i 77,7 snimaka/s s isključenim pozadinskim procesom. | `/MERGETASKS="!defenderexclusion"` |

Kontinuirano snimanje izvještava o opterećenju Defendera kao `defender_cores` (vidi [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Instalirajte pomoću `apt install ./…`, a ne `dpkg -i`: apt također instalira zahtjeve za Pythonov SDK, koje paket preporučuje, a ne zahtijeva (vidi [Korak 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Za Jetson Orin / Orin NX / Orin Nano na JetPack 6 (L4T R36). Njegov PyTorch je CUDA izdanje za JetPack 6, pa se obrada na GPU-u odvija na Orinovom GPU-u; koristi biblioteke CUDA i cuDNN koje instalira JetPack 6 (`sudo apt install nvidia-jetpack` na slici kojoj one nedostaju).

#### Linux: prijemni bufferi i pozadinska usluga

Oba .deb paketa povećavaju prijemne buffer-e kernela koje streamovi kamere LATTICE trebaju na 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). U zadanoj postavci kernela, paketi kamere se gube u kernelu i niz gubi sličice i brzinu sličica; `chloros-cli lattice network` izvještava o spremnicima kada su niski. Pogledajte referencu za CLI [Linux hostovi — spremnici za primanje soketa](cli-reference.md#linux-hosts--socket-receive-buffers).

Na uređaju bez zaslona pokrenite pozadinski program kao uslugu **pod svojim korisničkim računom**, kako bi skripta pronašla pokrenuti program (`connect_all()` nikada ne pokreće pozadinski program):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend se tada pokreće kao da ste vi: snimke koje piše su vaše i koristi vaš `chloros-cli login`. Korijenski `chloros-backend.service` i dalje je instaliran za strojeve na kojima je omogućen, ali svaka datoteka koju napiše korijenski backend pripada rootu. Pogledajte referencu za CLI pod naslovom [Pokretanje backenda kao usluge](cli-reference.md#running-the-backend-as-a-service-headless).

Backend koji pokreće usluga je zajednički i klijent ga nikada ne zaustavlja: skripte SDK i `chloros-cli` koriste ga onako kako ga zateknu (`ChlorosLocal.shutdown_backend()` zaustavlja samo backend koji je `ChlorosLocal` sam pokrenuo), a `GET /api/status` naziva jedinicu `service`. Ponovno ga pokrenite pomoću `sudo systemctl restart chloros-backend@$USER.service`. Nadogradnja paketa zaustavlja pozadinske usluge tijekom instalacije, tako da aktivna skripta izgubi svoje kamere i senzore, a zatim ponovno pokrene svaku omogučenu; uklanjanje paketa zaustavlja usluge pozadinskih procesa i onemogućuje omogućene. U svakom trenutku radi samo jedan pozadinski proces, stoga omogućite jednu jedinicu: druga će odbiti pokretanje i ostati `failed` (status izlaska 98, nije ponovno pokrenut), a njegova `systemctl status` objašnjava zašto. Za promjenu jedinica onemogućite onu koju ne koristite.</id></sn>

### Korak 2 — Instalirajte Python SDK

**Instalacijski program za Chloros dolazi s odgovarajućim SDK paketom (wheel).** Svaki instalacijski program za Windows i .deb paket za Linux postavlja `chloros_sdk-X.Y.Z-py3-none-any.whl` na disk koji točno odgovara verziji GUI-ja / CLI / pozadinskog sustava. Ne morate pretraživati PyPI kako biste ostali sinkronizirani.

#### Windows

Instalator automatski pokreće `pip install` protiv priloženog wheel-a koristeći vaš sustavni Python (prvenstveno se koristi pokretač `py.exe`, a kao rezerva `python -m pip`). Nije potrebno poduzimati nikakve radnje — `import chloros_sdk` radi u vašem okruženju Python nakon uspješne instalacije. Ako na računalu nije instaliran Python, instalacijski program tiho preskače ovaj korak i GUI + CLI nastavljaju raditi.

#### Linux (.deb)

Oba .deb paketa (x86_64 i Jetson) instaliraju SDK za sustav `python3`, tako da `import chloros_sdk` radi nakon instalacije, bez pip-a i bez mreže. Paketi (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` i projektni učitavači) idu u `/usr/lib/python3/dist-packages`, i naredba `chloros-mcp` SDKa ( [MCP server](#mcp-server-ai-assistants)) u `/usr/bin`, sve u vlasništvu paketa: nadogradnja ih zamjenjuje, a uklanjanje paketa ih uklanja.

Zahtjevi samog SDK-a, `python3-numpy` i `python3-requests`, **preporučuju se, ali nisu obavezni** (GUI, CLI i pozadinski sustav su samostalni): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` na Jetsonu) ih instalira, `dpkg -i` to ne čini. Izlaz instalacije prikazuje verziju SDKa koju `python3` uvozi, sve zahtjeve koji nedostaju, i `chloros_sdk` drugdje na `python3`-ovoj putanji koji zasjenjuje kopiju paketa (kao što je onaj ranije instaliran pomoću `pip install --user`).

U virtualenvu, koji ne vidi sistemske pakete, instalirajte priloženi wheel u njega:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Za Jetson deploye odvojene od mreže ovo je potpuno offline — SDK i wheel već su na disku.

#### Javni PyPI

Za hostove koji koriste samo pip (nije instaliran Chloros desktop paket; SDK tada pokreće backend na drugom računalu, pogledajte [Način rada s udaljenim backendom](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI sadrži najnovije stabilno izdanje. Održavajte SDK na istoj verziji kao i Chloros pozadinski sustav kojim upravlja (vidi [Verziranje i kompatibilnost](#versioning--compatibility)). Pomoćni program DAQ-M `DLS` također treba `bleak`: `pip install "chloros-sdk[camera]"` (ili `pip install bleak`).

#### Provjera

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Potrebna je pretplata na Chloros+.** Svi pozivi SDK-a zahtijevaju aktivno prijavljivanje na Chloros+. Pokrenite `chloros-cli login user@example.com 'YourPassword'` jednom po računalu; vjerodajnice se pohranjuju u `~/.chloros/`.

### Trebam li desktop paket?

Samo pip paket **nije** dovoljan za većinu tijekova rada. Evo što je potrebno svakoj SDK površini:

| SDK površina | Trebaju li desktop paket? | Zašto |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Da**(za pokretanje jednog)**/ Ne** (postojeći pozadinski sustav već je dostupan) | Koristite pokrenuti pozadinski sustav (onaj desktop aplikacije ili onaj koji je pokrenula neka druga skripta); inače automatski pokrenite instalirani: `/usr/lib/chloros/chloros-backend` (Linux) ili `<install folder>\backend\chloros-backend.exe` (Windows; `C:\Program Files\Chloros\backend\` prema zadanim postavkama, prilagođeni putanja instalacije čita se iz zapisa instalatera). Proslijedite `backend_exe=` da biste mu sami dali naziv. |
| `chloros-mcp` ([MCP server](#mcp-server-ai-assistants)) | **Da**(za pokretanje jednog)**/ Ne** (backend je već dostupan) | Izgrađeno na `ChlorosLocal`, pa isto kao i redak iznad. |
| `read_image_audit_tags` | **Ne** | Čita vlastite metapodatke obrađene slike (samo standardna biblioteka Pythona). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Da**(lokalno)**/ Ne**(udaljeno) | Čisti klijenti HTTPa preko pozadinske usluge. Lokalna pozadinska usluga → potreban je paket za stolna računala. Udaljena pozadinska usluga → `backend_url=`**putem tunela** (pogledajte Udaljeni način rada pozadinske usluge — isporučene pozadinske usluge vežu se samo na petlju). |
| `ChlorosProject` / `open_project` | **Da** | Pokreće spremljene projekte putem pozadinskog sustava. |
| Izravne LATTICE klase (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Da** | Potrebno je Arena SDK runtime, koji je uključen u desktop paket, i OpenCV (`cv2`) u vašem Pythonu, što ni desktop paket ni SDK ne instalira (`pip install opencv-python`). `CAMERA_AVAILABLE` je inače `False` pri uvozu. `DLS` također treba `bleak`. |

### Mod udaljenog backenda (host samo putem pip-a, kroz tunel)

> **Instalirani backend nije dostupan putem LAN-a.** Sluša samo na
> loopbacku (`127.0.0.1` / `::1`), stoga `backend_url="http://<lan-ip>:5000"`
> **ne može dohvatiti instalirani Chloros**. Da biste pokrenuli backend na drugom računalu, proslijedite njegov loopback port i usmjerite SDK na tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Hostovi bez zaslona / CI / robotske platforme mogu na jednoj mašini s potpunom instalacijom desktop okruženja imati &quot;Chloros poslužitelj&quot;, a na svim ostalim `pip install chloros-sdk` — no prijenos između njih je tunel koji je korisnik postavio, a ne izravna LAN URL adresa.

> **`ChlorosLocal` na hostu koji ima samo pip.** `ChlorosLocal(backend_url=BACKEND)` povezuje se s pozadinskim servisom kojem može pristupiti bez instalacije desktop okruženja: lokalnu binarnu datoteku pozadinskog servisa traži samo kada ga mora pokrenuti (ako nitko ne odgovara i `auto_start_backend=True`). Do verzije 1.2.0 prvo je tražio binarnu datoteku i pokretao `ChlorosBackendError` (&quot;Chloros backend nije pronađen…&quot;) na hostu s pip-om, čak i kad je backend bio dostupan. Mape koje prosljeđujete (`process_folder`, `import_images`) provjeravaju se na ovom računalu i čita ih backend, stoga kod udaljenog backenda moraju imati istu mapu na oba sustava, primjerice na zajedničkom pogonu.

### Radni tijek samo za DAQ (pip-only host)

Samo `pip install chloros-sdk` ne može upravljati DAQ senzorom: DAQ senzorom se uvijek upravlja putem backenda Chlorosa s `connect_daq_sensor()`:

- **Senzor na ovom računalu:** instalirajte desktop paket Chloros ovdje. `connect_daq_sensor()` pokreće svoj backend ako nijedan ne radi.
- **Senzor na drugom računalu:** to računalo treba desktop paket; povežite se s njegovim backendom putem tunela u [načinu daljinskog backenda](#remote-backend-mode-pip-only-host-via-tunnel). Backend otvara senzor na vlastitom računalu, tako da je DAQ-U priključen na taj uređaj, DAQ-M je u Bluetooth dometu tog uređaja, a DAQ-E je na mreži do koje može doći.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

U svakom slučaju, backend provodi plan &quot;Chloros+&quot;: prijavite se na računalo na kojem se pokreće backend (`chloros-cli login` ili desktop aplikacija) s bilo kojom plaćenom razinom (Copper ili višom). I odjavljeni backend i plan niži od Coppera izazivaju `ChlorosConnectError` koji nosi razlog backenda (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Brzi početak

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indeks glavnih APIa

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Obrada slika — `ChlorosLocal`

Glavna pipeline klasa. Pokreće pozadinski sustav kada se konstruira (ako nijedan ne odgovara i `auto_start_backend=True`), stvara / konfigurira projekte, nadzire napredak i vraća sažetke nakon izvođenja.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metode

| Metoda | Opis |
| --- | --- |
| `create_project(project_name, camera=None)` | Stvori novi projekt (po želji s predloškom kamere kao u `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Otvori spremljeni projekt (njegovu mapu ili njegovo ime u mapi projekata) i postavi ga kao otvoreni projekt. Putovi premještenog projekta preusmjeravaju se na mapu u kojoj se projekt sada nalazi; `relink` imenuje mapu u kojoj se sada nalaze njegove ostale nedostajuće datoteke. Vraća odgovor pozadinskog sustava pomoću `source_files`; izbacuje `ChlorosProjectFilesMissingError` kada nijedan ne može biti pronađen. Vidi **Premješteni projekt** u nastavku. |
| `import_images(folder_path, recursive=False)` | Uvoz slika u RAW/TIF/JPG/DNG formatu **i snimaka senzora svjetla `.daq`**. Vraća `count` (slike) i `scan_count` (zapisima). Upozorava samo ako mapa ne sadrži ni jedno ni drugo. |
| `export_light_sensor(daq=True, csv=True)` | Pisanje kalibriranih `.daq` + `.csv` za svako snimanje senzora svjetla u projektu, u `<project>/Light Sensor/`. Vidi [Zapisnici senzora svjetla](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Postavite gumbe za obradu. `indices` su nazivi unaprijed postavljenih (`NDVI`, `GNDVI`, `OSAVI`, …), svaki dodan u projekt za pojaseve svoje kamere; ime koje nije unaprijed definirano, ili koje pojasevi ne mogu stvoriti (`NDRE` treba RE i NIR, što nijedna pojedinačna kamera ne pruža), dobiva `UserWarning` koji ga imenuje i nema indeksnu sliku. `daq_log_path` imenuje snimanje senzora svjetlosti (datoteku `.daq` ili `.csv`, ili mapu s njima) s kojom se uspoređuje svaki kadar, LATTICE kao i Survey3, umjesto snimki uvezenih sa slikama; putanja koja nedostaje ili ne sadrži snimku smatra se nedostajućom i koriste se uvezene snimke. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Pokrenite pipeline. Vraća `{"status": "complete", "async": False}`, plus `summary` ključ kada ga pozadinski sustav osigura — vidi [Sažetak nakon izvođenja i savjeti](#post-run-summary--hints) — `export_check_failure` kada se izvozni podaci pokretanja nisu mogli provjeriti, i `image_failure` kada je izgubio sliku. `wait=False` vraća `{"status": "started", "async": True}` odmah. |
| `get_config()` / `get_status()` / `status()` | Pregledajte stanje pozadinskog sustava. |
| `logout()` | Odjava: briše pohranjene vjerodajnice i oslobađa utor uređaja ovog računala (`device_deregistration` u rezultatu). U tijeku izvršavanja obrade ne prekida se: dovršava se u načinu u kojem je započela (`run_in_progress: true` i `run_mode` u rezultatu), a sljedeće izvršavanje koristi standardnu obradu. |
| `shutdown_backend()` | Zaustavite backend ako ga je ova instanca pokrenula; backend koji se pronađe u pokretanju ostaje netaknut. Ostavljanje `with` bloka ili instancu koja se čisti ne zaustavlja ga: backend nastavlja raditi za sljedeću skriptu i sam se zaustavlja nakon 30 minuta ako ga nitko ne koristi. Od backenda se traži da se isključi, pa on oslobađa svoje kamere i senzore svjetla te prvo dovršava sve snimanje; prestaje raditi samo ako i dalje radi 10 s kasnije. |
| `discover_cameras()` | Otkrijte LATTICE kamere **putem backenda ove instance** (`/api/camera/discover`). Vraća popis dictova (`serial`, `model`, `ip`, …) — istog oblika kao što ga vide GUI/CLI. Prazna lista ako nijedna nije pronađena ili ako je backend nedostupan. |
| `camera_capture(output_dir, format="tiff", **settings)` | Snimanje jednog kadra**putem backenda**(automatski pokrenuto ovim handleom) kako bi dobio istu pripremu kao GUI/CLI (podrazumijevano 12-bitno, ponovna upotreba pool-a, ugrađene cal metapodatke). Riješite cilj pomoću `serial=` ili `device_index=`; proslijedi `exposure`/`gain`/`pixel_format`/`preset` kao `**settings`. Vraća naslijeđeni metapodaci dict (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Isporučuje preklapajuće složene pretpregledne okvire iz objedinjene kamere — tanki MJPEG klijent preko `/api/camera/<serial>/stream-annotated` rute pozadinskog sustava (zebra / mreža / križ / histogram / peaking / točka iscrtana na strani poslužitelja). `decode=True` vraća BGR nizove i zahtijeva OpenCV (`cv2`), koji SDK ne instalira; `False` vraća sirove JPEG bajtove. Također dostupan po projektu kao `ChlorosProject.stream(overlays=True)`. |

**Pokretanje u tijeku zadržava svoj projekt.** Dok je obrada u tijeku — ove skripte, aplikacije, `chloros-cli process` ili `chloros-mcp` — pozadinski sustav odbija stvoriti, otvoriti, uvesti ili promijeniti postavke obrade projekta (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` i `import_images()` podižu `ChlorosProcessingError`, i `configure()` izaziva `ChlorosConfigurationError`, a `process()` s modom drugačijim od onog koji se trenutno izvršava izaziva `ChlorosProcessingError` (izvršavanje zadržava način rada u kojem je pokrenuto), svaki s porukom pozadinskog sustava. Snimka spremljena u projekt zatim mu se pridružuje kada se izvršavanje završi (njen odgovor `project_add` to potvrđuje). Pričekajte da se izvršavanje završi (`get_status()`), a zatim nastavite.**Premješteni projekt.** Projektov `project.json` imenuje svaku izvornu datoteku (svaku sliku i snimku senzora svjetlosti) prema njezinoj punoj putanji, pa ako premjestite mapu projekta -- ili cijelu mapu projekata na drugi disk -- one će i dalje nazivati staro mjesto. Svako otvaranje ispravlja ono što može samo: datoteka koja se nalazila u mapi projekta traži se u mapi u kojoj se projekt sada nalazi, a svaka pronađena putanja se preusmjerava i sprema (`project.json` se prvo sprema kao sigurnosna kopija pod imenom `project.json.before-relink-<date>-<time>`). Putnja se preusmjerava samo na datoteku koja je prisutna. Datoteke koje su bile izvan mape projekta -- hub sesija, uvoz dodavanja mape -- pronalaze se samo s `relink=<the folder they are in now>` (ili bilo kojom mapom iznad nje). `open_project()` vraća `source_files`: `total`, `found`, `missing`, `missing_roots` (mape u kojima su bile nedostajuće datoteke), `missing_outside_project` i `relink` (što je ponovno usmjereno, odakle kamo). Projekt s nekim nedostajućim datotekama otvara se s preostalim; projekt u kojem nijedna nije pronađena ne otvara se, i `open_project()` pokreće `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). `project open --relink` i `chloros-mcp` CLI-ja`open_project` (`relink_folder`) rade isto.

Koristite kao upravitelja konteksta za zajamčeno čišćenje:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Snimanja senzora svjetla — kalibrirani `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E mogu se snimati **bez** svog kalibracijskog paketa. To je ono što javni [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
uređaji za snimanje (`record_daq.py`) to rade prema zadanim postavkama: bilježe sirove brojeve senzora i označavaju datoteku kako bi Chloros preuzeo tvorničku kalibraciju tog senzora **po serijskom broju** — lokalna predmemorija
prvo, zatim u oblak MAPIR — i primjenjuje ga pri uvozu.

Chloros zapisuje rezultat natrag kao dva proizvoda po snimci, pod
`<project>/Light Sensor/`:

| Proizvod | Što je to |
| --- | --- |
| `<name>_calibrated.daq` | Arhiva za ponovnu obradu — ista shema kao i kod snimanja uživo, sada s navedenim paketom koji ju je proizveo. Ponovna uvozna **ne** kalibrira drugu kalibraciju. |
| `<name>_calibrated.csv` | Spektralna iradijanca u W/m²/nm na vlastitoj mreži valnih duljina senzora, jedan redak po očitanju, plus fotometrijske stupce (ukupna snaga, fotopski/skotopski luks, PPFD i njegova podjela na plavo/zeleno/crveno, vršna valna duljina). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Samo za senzore bez paketa (DAQ-A).** Sirove spektralne vrijednosti senzora — *ne* zračenje. Vidi dolje. |

`<name>` je naziv datoteke snimanja. Dva snimanja u različitim mapama mogu dijeliti
jedan — očitanja po snimci (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) spremljena u
istoj milisekundi od strane dviju kamera — i svaka i dalje dobiva svoje proizvode: prva zadržava
`<name>`, a sljedeći dodaje mape koje ga odvajaju **iza** naziva
(`<name>__<capture>_daq_calibrated.daq`), tako da datum i vrijeme snimanja i dalje prednjače.

`process()` obavlja ovaj izvoz kao jednu od svojih faza. Ne **zahtijeva** slike: senzor svjetlosti postavljen samostalno predstavlja potpuno valjan tijek rada, a takav projekt po svojoj je prirodi bez ikakvih slika.**Snimke DAQ-A izvoze se kao sirovi brojevi.** Obitelj DAQ-A prethodi sustavu paketa po serijskom broju i nema paket za preuzimanje — umjesto toga kalibrira se na terenu protiv cilja reflektancije, zbog čega mu on nikada nije bio potreban. Ta snimanja se izvoze pod `_raw` korijenskim imenom, a ne pod `_calibrated`: drugačije ime datoteke umjesto zastavice unutar datoteke, jer tvrdnja mora ostati očuvana i ako se datoteka pošalje e-poštom kao obično ime.
Zaglavlje `.csv` navodi `raw spectral sensor counts (NOT irradiance)` i upozorava da su vrijednosti usporedive **unutar** datoteke — upravo ono što koristi kalibracija temeljena na cilju
njih za — i ne među senzorima. Fotometrijske stupce ovisne o snazi (ukupna snaga, fotopski/skopski luks, PPFD) vraćaju **NULL** umjesto da se integriraju iz brojeva.

DAQ-U / DAQ-M / DAQ-E čiji se paket jednostavno nije mogao dohvatiti i dalje se **preskače**, a ne piše u sirovom obliku: u tom slučaju paket postoji i &quot;ponovno povezivanje i ponovna obrada&quot; je stvarni savjet.

Zapisi naslijeđa **v1.01 / v1.02** (koje zapisuje DAQ-A-SD) ne sadrže epochu po očitanju,
samo vrijeme zapisa datoteke. Uređaj za usklađivanje slike↔downwellinga i dalje ih odbija — usklađujući
usporedba kadra s vremenom zapisa bila bi pogrešna na skriven način — ali izvoznik ih čita, a
CSV ispisuje `clock=daq_created_on` pa proizvod navodi na kojem je satu.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Zabilježba čiji se paket kalibracije ne može dohvatiti (offline, ili senzor bez spremljene kalibracije) prijavljuje se pod `skipped` **s razlogom**. Nikada se ne spremuje kao &quot;kalibrirana&quot; datoteka koja sadrži sirove brojeve — povežite se s internetom i ponovno pokrenite, te će se izvoz dovršiti.

### Obavijesti o napretku

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Sažetak nakon izvršavanja i savjeti

Po završetku, `process()` dohvaća `GET /api/processing-summary` i prilaže tijelo kao `result["summary"]`. Dohvat se vrši prema najboljem naporu i nikada ne blokira uspješan povrat — ako sažetak nije dostupan, `process()` se vraća na obični oblik `{"status": "complete", "async": False}`. Svaki unos u `summary["hints"]` — potpune rečenice s predloženim otklanjanjem nedostataka, npr. zašto je izvršavanje proizvelo nulti rezultat — također se emitira kao Python `UserWarning`, tako da se izvršavanja s nultim rezultatom samodijagnosticiraju čak i ako nikada ne provjerite dict:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je mašinski čitljiv dio:

| Ključ | Što broji |
| --- | --- |
| `models` | Grupe kamera u izvršavanju. |
| `images_in_groups` | Izvorne slike unutar tih grupa. |
| `targets_found` | Otkriveni ciljevi reflektancije. |
| `images_calibrated` | Slike za koje je izvršavanje napisalo refleksancijski proizvod. |
| `images_failed` | Slike koje su neuspjele i nisu izvezene. |
| `exported_files` | **Datoteke s proizvodima slika pokret je napisao**: svaki proizvod (debayering, pretpregled, zračenje, reflektancija, indeks, renderiranje cilja), u svakom formatu. |
| `reflectance_files` | Među njima, proizvodi reflektancije (`Reflectance_Calibrated_Images`), bilo da su kalibrirane ciljanom pločom ili iradijancom senzora svjetla (DAQ). Pokretanje bez ciljne ploče koje je ipak zapisalo refleksancu to naznačuje u poruci; samo se pokretanju koje nije ništa zapisalo javlja da nedostaju njegovi izlazni podaci o refleksanci. |
| `clamp_masks_written` | [unmix clamp maske](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) napisano pored njih — pratitelji, a ne proizvodi, stoga nisu u `exported_files` (CLI ih odvojeno broji na isti način). |
| `captures_left_out` | Sirovi kadrovi u mapi projekta `captures` koje ova izvedba nije uzela. Pokretanje čeka pri pokretanju na ono što je snimljeno prije njega, pa je ovo 0 osim ako tijekom pokretanja nije spremljena snimka ili je čekanje isteklo. `summary["captures_left_out_note"]` navodi što učiniti s njima. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Snimke senzora svjetla, namjerno se računaju odvojeno — dolaze iz druge faze i postoje za pokrete bez ikakvih slika, pa bi njihovo uključivanje učinilo da pokret samo za DAQ izgleda kao da je izvozio slike. |

Uz njih: `summary["output_dirs"]` (svaka mapa u koju je nešto zapisano),
`summary["light_sensor_export"]`, `summary["stopped"]` (true kada je korisnik prekinuo
izvršavanje, tako da djelomični brojevi ne izgledaju kao dovršena izvedba koja je proizvela premalo), i `summary["groups"]` (rastavljanje po grupama).

`exported_files` bilježi pipeline **dok piše**, a ne skenira se s projektnih objekata slika naknadno. Paralelne i GPU strategije grade vlastite objekte slika (u radnim podprocesima za GPU staze), pa staro skeniranje izvještava
`0 file(s) written` za svaku takvu izvedbu i zatim emitira natuknicu o nultom broju izvoza — za izvedbe u kojima je sve uspjelo. Ako na temelju ove brojke pišete skriptu, ispravna paralelna izvedba sada izvještava o neto-nultoj vrijednosti.

Izvještaji o preskočenim senzorima svjetla navode razlog koji je čitač zapravo utvrdio za svaku datoteku — nečitljiva shema, nedostajući paket, pogreška pri pisanju — **duplikati su uklonjeni**, tako da se dvadeset datoteka preskočenih iz istog razloga evidentira kao jedna, a ne kao dvadeset ponavljanja istog razloga.

> **`process()` se ne aktivira kada izvršenje ne proizvede nijednu sliku.** Ovo je jedno od dva mjesta gdje se SDK i CLI namjerno razlikuju: `chloros-cli process` tretira &quot;zahtijevani su proizvodi, nijedan nije napisan&quot; kao neuspjeh i izlazi s ne-nuljevnim rezultatom, dok SDK normalno završava i izvještava o
> uvjet putem `summary` / naznaka. Ako bi vaš pipeline trebao stati na praznom pokretanju, provjerite to sami — pregledajte `summary` (ili prebrojite datoteke u mapi projekta) umjesto da se oslanjate na
nepostojanje iznimke. Uobičajeni uzroci su ulazna mapa koja nije prepoznata kao
snimka i proizvodi preskočeni kao neprimjenjivi za prisutne kamere (npr. radijansa s
kamera koje rade samo u RGB formatu).
>
> `process()` se ne aktivira ni kada se izvozi ne mogu provjeriti: on upozorava i postavlja
> `result["export_check_failure"]`. Proizvodi su spremljeni. Da biste zaustavili proces zbog toga, označite taj ključ. Pokretanje koje je izgubilo sliku je isto: upozorava i postavlja `result["image_failure"]`, na kojem `chloros-cli process` završava s ne-nuljevim rezultatom.

### Pomoćne funkcije

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Svaka se izvedba pokreće unutar vlastitog `with ChlorosLocal(...)`; pozadinski proces koji je morala pokrenuti nastavlja raditi kada se ona vrati, za sljedeći poziv, i sam se zaustavlja nakon 30 minuta ako ga nitko ne koristi. Nijedan ne uzima `auto_start_backend`, `radiometric_output` ili `daq_log_path`; koristite `ChlorosLocal` i `configure()` za one. `process_lattice_capture()` ne prihvaća `camera`, `debayer` ili `ppk`: preskače detekciju panela i koristi standardni debayer.

### Podržane vrijednosti

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrijski izlaz (LATTICE višespektralni pipeline)

Razina izvoza LATTICE multispectral (M3C/M3M) cjevovoda `process` — `reflectance` (zadano), `radiance`, `sensor-response` ili `all` (svaki primjenjivi način rada po slici) — postavlja projektove **&quot;Izvoz radijance&quot;**i**&quot;Izvoz reflektance&quot;** proizvoda: `radiance` izvozi samo zračenje, `reflectance` samo odbijanje, `sensor-response` ni jedno ni drugo (samo proizvodi bez de-bayeringa i za pregled), `all` oboje. Proizvodi za debayering i pregled nisu oni koje treba mijenjati na razini. `configure()` ima namjensku ključnu riječ za to:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Napredni izlaz — pisanje proizvoda putem `custom_settings` — i dalje radi, ali imajte na umu da zamjenjuje cijeli blok postavki (pogledajte upozorenje u nastavku). Upišite `"Export <product>"` ključeve: vlastiti `"Export radiance"` projekta / Postavke `"Export reflectance"` nadjačavaju stariji ključ `"Radiometric output"`, tako da sam taj ključ ništa ne mijenja.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (zadano) dijeli sjaj kamere s **DAQ-ovim silaznim zračenjem usklađenim s vremenskim žigom**, automatski odraženim iz snimljenog `.daq` (DAQ-U/M/E)**ili nativni DAQ-M `.csv`** pronađen uz sliku; svaki paket kalibracije po kameri ili DAQ-a koji nedostaje lokalno automatski se preuzima s AWS-a pri prvom korištenju. CLI to izlaže prema-tip proizvoda uključuje se na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Čije je očitanje.**Očitanje koje `.daq` sprema pri snimanju je očitanje kamere(a) koju opisuje — nazvano u sidecar datoteci `<file>.daq.cameras.json` pored njega (pojedinačni, sinkronizirani i nizovi snimaka; raw burst u svom manifestu navodi očitanje svakog člana) — i podudara se**samo s slikama s tih kamera**. Samostalna kamera bez vlastitog očitanja stoga više ne posuđuje DAQ druge kamere; povežite DAQ s njom ili snimite jedan. Vlastita snimka DAQ-a (`chloros-cli daq pool-record`, ili snimanje započeto u GUI-ju), ili bilo koji `.daq` bez te pomoćne datoteke (uključujući i starije snimke), i dalje služi svakoj kameri.**OCN kamere na sunčevoj svjetlosti.** Svaki paket za kalibraciju OCN izrađen je pod svjetlom uskog pojasa, s jednom LED diodom po pojasu, dok sunčeva svjetlost miješa pojaseve. Chloros 1.3.0 primjenjuje korekciju temeljenu na suncu za cijelu flotu nakon odvajanja pojaseva, što poboljšava OCN radijanciju, reflektanciju i indeksne proizvode na sunčevoj svjetlosti. Radi gdje god radi i odvajanje — `process()`, snimke, `grab()` / `frame_stream()` i prikaz uživo — za pakete OCNa koji to podržavaju; kamere RGNa ostaju nepromijenjene. Slike OCN snimljene ranijom verzijom također dobivaju ispravak kada se obrađuju na računalu koje ima kalibraciju s kojom su snimljene (Chloros zadržava kalibraciju kamere nakon što se kamera poveže); inače se obrađuju kao i prije. Stoga se vrijednosti radijance, reflektancije i indeksa OCN razlikuju od onih u verziji 1.2.0.

> `custom_settings` **zamjenjuje** cijeli blok izračunatih postavki (namjerno zaobilazi ostale ključne riječi i provjeru `configure()`-a). Kada ga koristite, uključite svaku `Project Settings` ključnu riječ koja vam je važna, kao u gornjem primjeru.

#### Čitanje indeksnih piksela (NDVI i ostali indeksi vegetacije)

**Integer indeksni raster je pojačanje kontrasta po slici, a ne mapa od −1 do 1.** `TIFF (16-bit)`, `TIFF (8-bit)`, indeksni proizvodi `PNG (8-bit)` i `JPG (8-bit)` — uključujući 8-bitni TIFF, u koji se zapisuje 8-bitni snimak — pohranjuju

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

gdje su `IndexMin` / `IndexMax` **najniže odnosno najviše konačne indeksne vrijednosti tog kadra**. DN 0 nije −1, a puni raspon nije +1.

Chloros žigira oba krajnja točka kako bi mapiranje ostalo inverzibilno:

| Označitelj | Značenje |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvencija u nastavku |
| `Chloros:IndexMin` | indeksna vrijednost pohranjena kao DN 0 |
| `Chloros:IndexMax` | indeksna vrijednost pohranjena kao puni iznos |

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

Za čitanje nije potreban `-config pix4d.config` — exiftool čita ove XMP značajke bez pomoći. Konfiguracija je potrebna samo za njihovo *pisanje*.

> **`IndexMin == IndexMax`** znači da je svaki piksel te slike imao istu vrijednost indeksa. Raster je popunjen na sredini skale i nema čim podijeliti — uzmite krajnju točku.

> **Nedostatak oznaka `Chloros:Index*` znači da nema navedenog mapiranja.** Izvoz LUT-a je kolorizirani prikaz čiji pikseli sadrže boje gradijenta usklađene s vašim LUT pragovnim prozorom, a kadar bez piksela konačnog indeksa nema raspon koji bi mogao navesti. Ne pretpostavljajte mapiranje ni u jednom slučaju — isto pravilo vrijedi kao i za nedostajuću oznaku `PixelScale`.**Budući da je istezanje po slici, dva kadra jednog leta nisu usporediva kao DN**— svaki nosi svoje krajnje točke, pa ista vrijednost NDVI pada na različite DN u različitim slikama. Za kvantitativni rad unutar leta (zonalna statistika, fiksni pragovi, vremenske serije), izvezite**`TIFF (32-bit, Percent)`**: indeksni rasteri se tamo zapisuju kao float32**samih indeksnih vrijednosti**, bez potrebe za ponovnim skaliranjem i bez oznaka. Unatoč nazivu formata, indeksni proizvodi nisu ×100 — to skaliranje primjenjuje se samo na refleksancu cjelobrojne vrste podataka (integer-dtype).

#### Unmix clamp maska — na kojim je pikselima pojas bio zaključan

Spektralna demiksa 3×3 može dati negativan rezultat za pojas — gdje svjetlost jedva sadrži taj pojas ili gdje scena izlazi izvan onoga što opisuje matrica za demiksu kamere. To nije stvarna vrijednost, stoga je izvoz fiksira na 0 — a prikaz uživo prikazuje istu 0. Pojas se može zaklamati na nekoliko piksela ili na gotovo sve njih; maska bilježi bilo koje od toga. `Chloros:Band{R,G,B}UnmixClampFraction` kaže na kojem je dijelu kadra to dogodilo; maska zaklamavanja kaže **koji su to pikseli**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— u mapi `clamp_masks/`**pored** mape proizvoda, nikada unutar njega (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — za svaki proizvod radijance, reflektance i indeksa u kojem je barem jedan piksel zakliznut. Jednopojasni `uint8`, na vlastitoj pikselnoj mreži proizvoda (orijentacija, ROI, binning i warp poravnanja niza uključeni). Svaka maska imenuje kameru svog proizvoda (`Make` / `Model` / serijski broj), vrijeme snimanja i oznake objektiva te navodi pojaseve koje njeni biti predstavljaju (`XMP-Camera:BandName`, prvi bit 0); njezini `ImageDescription` i `Software` označavaju je kao masku za zaključavanje i ne sadrže kalibraciju.
- **Biti**: bit 0 = pojas u R kanalu proizvoda (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Postavljeni bit znači da je vrijednost piksela u tom pojasu fiksirana na 0 — tretirajte ga, kao i bilo koji indeks izgrađen na temelju njega, kao nepouzdan.
- **`Chloros:UnmixClampMask`** stavlja se na svaki proizvod koji je pokrenuo unmix: putanja maske u odnosu na vlastitu mapu proizvoda (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, uvijek `/`- odvojen — spojite ga s mapom slike za otvaranje maske), ili `none` kada nije zaključan nijedan piksel. Nema oznake znači da proizvod nikada nije pokrenuo unmix (RGB ili mono kamere, raw/debayered razine).
- Rezultati snimanja nose ga kao **`clamp_mask`** kadra (putanja maske, ili `None` kad ništa nije zaključano) — u rezultatima `CameraSession.capture()`, `ArraySession.capture()`&#x27;s `CaptureResult` kadrova i HTTP odgovora snimanja. Offline build navodi svoje maske kao `clamp_masks`.
- Višekanalni stack uživo (`/api/camera/cli-array-index` s `save_multiband`) dobiva jednu masku po izvornoj kameri, u vlastitoj `clamp_masks/` izlazne mape — pogledajte referencu za CLIs `lattice index`. Video nema masku: rezultat offline izrade umjesto toga izvještava o clamp frakciji svakog kadra.

Budući da nijedan proizvodni mapni prikaz ne sadrži masku, na bilo kojoj razini, proizvodni mapni prikaz može se predati softveru za fotogrametriju onakvim kakav jest — uključujući podmapne prikaze — a on prihvaća samo slike. Predajte mu proizvodni mapni prikaz, a ne mapu iznad nje. Vlastiti import programa Chloros preskače maske.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect za kamere LATTICE

Trajne pozadinske sesije za hardver u stvarnom vremenu. Koriste se ista odredišta kao i u GUI-ju, stoga je ponašanje identično na svim razinama: SDK / CLI / GUI.

### Jedna kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Ekspozicija, pojačanje i ciljani set ovdje se zadržavaju kao vaši: automatska kontrola ekspozicije tvrtke Chloros (uključujući ekspoziciju mjerenu DAQ-om) ne mijenja ih. `cam.set_settings(exposure_auto="Continuous")` (ili `gain_auto=`) vraća ekspoziciju (pojačanje), a `cam.set_settings(target_brightness_auto=True)` vraća cilj automatskom cilju Chlorosa.

#### Potpis `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Vraća se s pokrenutim streamom.** Backend čeka, ograničeno (otprilike 15 s da se stream pokrene, zatim 10 s za njegov prvi potpuni buffer — ne za prvi pretpregledni kadar, koji slijedi otprilike 1,5 s kasnije), tako da `status()` odmah nakon što kaže `streaming`. `connect_stream` sesije je blok odgovora `stream`: `state` `streaming`, `no_frames` (tok je započeo i nije isporučio ništa — `note` to kaže i, za kameru s jumbo paketima, ponovno se povezati s veličinom paketa od 1500 bajtova), `starting` ili `start_failed`, s `waited_s`. `None` za kameru koja je već bila povezana. Pregled (`get_camera_preview`, `/api/camera/jpeg/<serial>`) zatražen neposredno nakon čekanja, ograničeno (4 s), za prvi pretpregledni kadar kamere umjesto odgovaranja &quot;Nema dostupnog kadra&quot;; kamera čiji stream nije isporučio ništa (`no_frames`, ili aktivirana hardverskim okidačem) i dalje odgovara odmah.
- **Veličinu paketa određujete vi.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` otvara kameru s paketima od 1500 bajtova, bez ispitivanja (`chloros-cli lattice cam-connect --packet-size standard` šalje isto).
- **Jedna veza u isto vrijeme drži bazen kamera**: otvaranje, proba veličine paketa, prvo preuzimanje kalibracije (oko 70 s za kameru novu na ovom računalu), početak prijenosa. Zahtjev koji u međuvremenu treba kameru (`read_nodes`, `set_settings`, snimanje, drugi spoj) čeka na njega najviše 5 s, a zatim ne uspijeva s HTTP 503 `CAMERA_POOL_BUSY`, navodeći kameru koja se povezuje (`ChlorosConnectError`); pitati ponovno za trenutak. Popisi nikada ne čekaju: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` i `CameraSession.status()` odgovaraju odmah. Kamera usred povezivanja navedena je pod `connecting` odgovora bazena (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — i `elapsed_s`), a njegov `status()` kaže `connecting: True` sa svojim `phase`; ostale kamere čitaju se uživo kada je bazen slobodan, inače se odgovara prema njihovom posljednjem očitanju (`live: False`).

#### Metode `CameraSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=15.0)` | Trenutno stanje kamere, kako ga čita `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` ili `stopped`), `device` (model, serijski broj, firmware, temperatura, `is_mono`) i njegove trenutačne postavke. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Čita GenICam čvorove; vraća `{nodes, errors, enums, device, was_in_pool, in_array_id}`, plus `frame_values` i `notes` dok traje automatsko izlaganje (vidi dolje). |
| `set_settings(**kwargs)` | Pisanje čvorova po prijateljskom nazivu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). `exposure_time` (`gain`) je ručno izlaganje (pojačanje): napisano bez `exposure_auto` (`gain_auto`), pri čemu se automatski način rada prvo isključuje, pa vrijednost ne odbija aktivni automatski način rada. Rezultat `applied` zadržava ono što je kamera uzela; pisanje koje fotoaparat odbije nalazi se u `errors` i nije pohranjeno. Greška navodi raspon koji fotoaparat sada prihvaća (maksimalna vrijednost `ExposureTime` prati brzinu snimanja fotoaparata), i automatski način rada koji je isključen samo za odbijenu vrijednost ponovno se uključuje; `restored` ga navodi (npr. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Snimanje **jednog** okvir. Vraća `CaptureResult`: popis dictova metapodataka okvira (po jedan po razini pohrane), s `.skipped`, `.failed`, `.daq_files` i `.daq_matches`. (Serijska snimka uklonjena je — pozovite `capture()` u petlji ako vam treba serija.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | snimanja uzastopna (`interval_s=0`) ili s intervalom s ove kamere, za snimanja `count` i/ili `duration_s` (najmanje jedno, pa završava), kao što `ArraySession.capture_repeated()` radi za niz. Vraća popis `CaptureResult`-ova; zaustavlja se nakon snimanja koje prijavi puni disk. `on_capture(i, result)` se izvršava nakon svakog; ostale ključne riječi idu na `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Oslobađanje iz bazena, vraća kameru u stanje u kojem je bila prilikom povezivanja (povezivanje okidača, PTP, postavke); `keep_settings=True` sprema svoje postavke umjesto toga u vlastitu memoriju, `clear_settings=True` resetira tu memoriju (vidi [memorija kamere](#camera-memory--keep_settings--clear_settings)); vraća `camera_memory` kada se zatraži bilo koja od njih. Kamera koja je član niza oslobađa cijeli niz. Bez učinka ako smo priključeni na već otvorenu sesiju. Vraća se nakon što pozadinski sustav potvrdi oslobađanje ili kada nema ništa za osloboditi (kamera je već uklonjena ili pozadinski sustav nije pokrenut); oslobađanje koje je isteklo ili koje je pozadinski sustav odbio izaziva `ChlorosConnectError`, jer kamera možda još uvijek nije oslobođena i sljedeće povezivanje bi se sukobilo s njom. Izlaz iz bloka `with` nikada ne izaziva događaj. |

Atributi: `serial`, `device` (zapis o otkrivanju kamere) i `backend_url`.

Kontrole izvoza `capture()` (isti model kao niz + GUI):

- `processing` / `levels` — `processing="all"` sprema svaki primjenjivi tip izvoza; `levels=["raw","radiance"]` sprema samo one (nadjačava `processing`). Ostavite oboje praznim za zadano postavljanje u pozadinskom sustavu.
- `daq` — DAQ čitanje snimanja koje se koristi po zadanome se sprema kao pomoćna datoteka (sidecar) `.daq`, uključujući samo sirovu snimku, kako bi se kadar kasnije mogao ponovno obraditi u refleksancu/indeks. `daq=False` ga preskače kada ga nijedna razina ne koristi. `force_daq=True` je stariji pravopis zadane vrijednosti. Nema DAQ-a, bez `.daq`.
- `daq_tolerance_ms` — koliko daleko od ekspozicije očitanje može biti, a da i dalje bude unutar tolerancije (zadano 500). Mijenja se samo ishod.

#### `read_nodes()` pri automatskom eksponiranju — `frame_values` i `notes`

Dok `ExposureAuto` (`GainAuto`) radi, `ExposureTime` (`Gain`) čvor čita **zadnju zabilježenu vrijednost**, a ne ono što kamera trenutno radi. `nodes` i dalje drži tu vrijednost čvora. Kada zatražite `ExposureTime` ili `Gain` na priključenoj kameri čiji automatski način rada nije `Off`, odgovor dodaje:

- `frame_values` — vrijednost koju je najnoviji kadar kamere sadržavao, ispod naziva čvora (`ExposureTime` u µs, `Gain` u dB);
- `notes` — jedna rečenica po takvom čvoru koja navodi obje vrijednosti.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Oba ključa su odsutna kada je automatski način rada `Off` (čvor je tada kamerastvarnu vrijednost) i za kameru koja nije povezana s pozadinskim sustavom. Za niz, `ArraySession.status()["member_ae"]` istovremeno daje vrijednosti kadrova svakog člana.

#### Gornja granica ekspozicije mjerenog DAQ-om (HTTP)

Kada je povezan DAQ senzor svjetla, Chloros ograničava ekspoziciju × pojačanje kamere prema izmjerenom dnevnom svjetlu kako bijeli referentni panel ne bi bio izrezan. Ograničenje se primjenjuje samo dok je kalibracijska meta u vidnom polju kamere (prikazana u njezinom kadru unutar posljednjih 10 s); ako meta nije u vidnom polju, umjesto toga radi uobičajeno automatsko ekspozicijsko podešavanje kamere, sa svojom vlastitom zaštitom od preosvjetljenja, kako ostatak scene ne bi bio tamniji nego što je potrebno. Ne postoji omotač za SDK; uključite ili isključite ga za svaku kameru pomoću naredbi `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (POST tijelo `{"enabled": false}` isključuje ga za tu kameru). Odgovor sadrži `enabled`, `cap_us` (stvarni prag, `None` dok je neaktivan, uključujući i dok se kalibracijski cilj ne nalazi u vidokrugu), `k_exp_g` (konstanta pomoću koje se izračunava strop) i:

| Polje | Značenje |
| --- | --- |
| `k_exp_g_source` | Čija se konstanta koristi: `camera` — vlastita konstanta ove kamere, izračunata iz lab točke u njezinom kalibracijskom paketu (RGB kamere); `fleet` — konstantu flote, koja se koristi kada paket nema točku iz laboratorija; `override` — polje s prioritetom koje je postavljeno na pozadinskom sustavu. |
| `k_exp_g_fleet` | Flotna konstanta, za usporedbu. |

### Sinkronizirani niz — `ArraySession` (Smart-Prep)

`connect_array` je **preporučena polazna točka** za postavke s više kamera. U pozadini pokreće cjeloviti GUI tijek pametne pripreme:

1. **Analiza mreže**(`/api/camera/array/recommend`) — postavlja**cijeli senzor** (bez binninga, bez ROI izrezivanja) i dopušta da se brzina sličica prilagodi onome što ovaj host i mreža mogu pružiti, uz predviđanje formata piksela i načina pakiranja koje će konektor koristiti. Sličice se nikada ne smanjuju kako bi se održala brzina: proslijedite manji `width` / `height` ili `binning=2` / `4` za zamjenu rezolucije za brzinu sličica (krajnja točkas `alternatives` navodi te opcije s njihovim predviđenim fps).
2. **Automatski odabir sloja** — `sim-capture-sim-emit` (svaka kamera eksponira na istom okidaču, emitiranje tempirano kako bi se dijelila veza, brzina sličica opada kako se udio smanjuje) na bilo kojoj mreži na kojoj se kamere mogu tempirati za dijeljenje; `slip-emit-and-capture` (kamere se izmjenjuju na mreži, nisu istovremeno izložene) samo kada ne mogu.
3. **PTP omogućen**prema zadanim postavkama — vremenski pečati između kamera padaju na jedan zajednički sat s**~1 ms**razlikom, [PTP grandmaster](#ptp-grandmaster--time_sync_status) pozadinskog sustava. Istovremeno eksponiranje dolazi od hardverskog okidača M8 (**&lt; 100 µs** između modula), a ne od PTP-a: PTP usklađuje *vremenske oznake*, a ne eksponiranja.
4. **Automatski odabir formata piksela po kameri** — mono → `Mono12`, boja → `BayerRG12`.
5. **Početno punjenje AE-a** — snima trenutno stanje AE-a svake kamere kako se pri povezivanju ne bi resetiralo izlaganje usred snimanja.
6. **Konfiguracija GPIO okidača** — `connect_array` aktivira svaku kameru (`TriggerMode=On`, `TriggerSource=Line2`) tako da masterimpuls pokreće robove preko M8 kabela. Ovo je korak samo za niz: pojedinačna kamera otvorena pomoću `LatticeCamera` umjesto toga radi u slobodnom načinu rada. Otpuštanje niza (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) vraća svaku kameru u slobodni rad (`TriggerMode=Off`), uz sve ostalo kako ga je pronašao connect: postavke okidača i sinkronizacijske linije (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; sinkronizacijsku liniju glavne kamere vratiti na ulaz), PTP i postavke kamere koje je sesija promijenila — osim ako `keep_settings=True` / `clear_settings=True` (vidi [Memorija kamere](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atributi: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` i `packed_reason` (posljednja četiri su `None` na handleu iz `attach_array`), `connect_stream` i `backend_url`. `connect_stream` je blok `stream` odgovora na povezivanje, oblik pojedinačnog povezivanja plus `members` (`{serial: {state, why?}}`): `streaming` čim stigne prva sinhronizirana grupa niza (backend na to čeka, ograničeno na oko 10 s), ili `no_frames` s posljednjim neuspjehom preuzimanja svakog člana kao njegovim `why` i, za jumbo pakete, rezervnu opciju od 1500 bajtova u `note`. `None` na `attach_array` rukohvatu.

#### Potpis `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` je koliko dugo poziv čeka na povezivanje i šalje se na
backend kao njegov rok. Povezivanje koje do tada nije dovršeno poništava se: svaka kamera koju je otvorio vraća se u stanje u kojem je bila prilikom pronalaženja i zatvaranja, nijedna niska nije povezana i poziv vraća `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, navodeći korak do kojeg je poziv stigao). Dakle, poziv koji je podignuo nikada ne napušta niz
povezana iza njega. Niz od 9 kamera povezuje se za otprilike minutu; odmah nakon što se kamere uključe, može potrajati i nekoliko minuta. Niz koji je aktivan, ali još nema sinkroniziranu grupu sličica, vraća se, s `connect_stream["state"]`
`no_frames`. U zapisniku pozadinskog sustava nalazi se jedan redak s vremenom za svaki korak povezivanja (`[array-connect <id>] <step>: <s> s …`) i jedan završni redak koji navodi najsporiji.

vrijednosti `force_tier`:
- `"sim-capture-sim-emit"` — istovremeno (sve kamere se aktiviraju na istom rubu takta).
- `"sim-capture-ftd-stagger"` — fleksibilno pomicanje u vremenskom domenu (kamere emitiraju u blago pomaknutim vremenima kako bi se paketi serijalizirali na vodi).
- `"slip-emit-and-capture"` — sekvencijalno snimanje po kameri (bez vremenske sinkronizacije). Automatski se odabire samo kada kamere ne mogu dovoljno sporo sinkronizirati prijenos kako bi istovremeno dijelile vezu.

`wire_ceiling_mbps` nadjačava **trajni proračun propusnosti hosta** u MB/s — jedini broj na kojem se temelji cjelokupna alokacija niza. Ostavite ga na `None` kako biste koristili automatski otkrivenu vrijednost. Smanjite ga kada niz izvještava o GVSP-pokvarenim okvirima: automatska vrijednost izvedena je iz oglašene brzine veze mrežne kartice (NIC), koja precjenjuje USB adaptere, uske PCIe trake i opterećene zajedničke mrežne resurse — a ta precjena se očituje kao oštećeni okviri, a ne kao vidljivo spora veza. Vrijednost se pohranjuje u bloku za snimanje niza projekta, stoga je ponovno otvaranje ili kasnija postavka `connect_array` vraća kao i svaku drugu postavku niza.
Vidi [Zdravstveno stanje niza](#array-health--which-subsystem-is-losing-frames).

#### Spakovani 12-bitni prijenos (`packed=`)

Kod punog opterećenja senzora, brzinu snimanja slika niza određuje brzina kojom svaka kamera može slati svoje slike. 12-bitni formati (`BayerRG12`, `Mono12`) obično se prenose kao 16-bitne riječi; komprimirani transport šalje istih 12 bita u 1,5 bajta po pikselu i dekomprimira ih na hostu, tako da svaki kadar zahtijeva 25 % manje vremena za slanje — otprilike33 % više sličica u sekundi pri punoj snazi senzora** (izmjereno: 10,7 → 14,0 fps na
nizu od 9 kamera). **Pohranjeni podaci su identični**: iste vrijednosti, iste 16-bitne
datoteke, isti metapodaci.

- `packed="auto"` (podrazumijevano) komprimira samo kada to pomaže **i** host računalo može
  priuštiti dekompresiju: pri povezivanju backend mjeri vlastitu brzinu dekompresije,
  broj jezgri CPU-a, trenutno opterećenje i mrežni budžet. Desktop računalo s viškom jezgri komprimira pri punoj opterećenosti senzora; zauzet ili nisko-moćni host (računala na jednoj ploči, ugrađeni kontroleri) prenosi nekomprimirano. Mali ROI-ji i binirani kadrovi imaju malu korist.
- `packed="off"` (ili `False`) nikada ne komprimira — kako bi se to isključilo tijekom dijagnosticiranja, ili na hostu koji mora zadržati svoj CPU za druge poslove.
- `packed="on"` (ili `True`) uvijek pakira 12-bitne formate.
- Bilo koja druga vrijednost pokreće `ValueError`. 8-bitni i 16-bitni formati nikada
  nisu spakirani.

`analyze_array_network(..., packed="auto")` projekti imaju istu odluku, pa je njihova fps brzina kojom će niz raditi (odgovor nosi `packed_transport` i `packed_reason`); njegova vrijednost `pixel_format` po zadanome odgovara onome što primjenjuje konektor (Mono12 / BayerRG12), a ne 8-bitnoj.

Odluka i njezino obrazloženje nalaze se u sesiji:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` primjenjuje isto prebrisavanje na svaki niz u projektu (zadano `None`, što znači `"auto"`: projekt ne pohranjuje nijedno spakirano podešavanje za niz).
Za zadano postavljanje na razini stroja postavite `CHLOROS_ARRAY_PACKED12=off` (ili `on`) u okruženju pozadinskog sustava; izričita postavka `packed="on"` / `"off"` i dalje ima prednost.
`/api/camera/array/status` izvještava `packed_transport` i `packed_decision`.

#### Očekivane stope okvira

Mjereni stope okvira niza (sinkronizirano hardverom, 12-bitno; puni senzor = 2048 × 1536). Koristite ih za provjeru ispravnosti `status()["actual_fps"]`; `analyze_array_network()` predviđa brzinu za vaš vlastiti host i mrežu.

| Postavka | Brzina sličica niza |
| --- | --- |
| Cijeli senzor, PCIe 10 GbE mrežna kartica hosta, komprimirani transport (`packed="auto"` na stolnom računalu) | ~14 fps |
| Cijeli senzor, PCIe 10 GbE mrežna kartica hosta, nekomprimirani prijenos (`packed="off"`) | ~10,5 fps |
| Cijeli senzor, 8 kamera na USB 10 GbE adapteru | ~3,8 fps |
| Cijeli senzor, 1 GbE host veza | 1,2–1,7 fps |
| 2 × 2 binning (`binning=2`), 10 GbE host NIC | ~30–33 fps |

- Na PCIe 10 GbE vezi, pri punoj upotrebi senzora, **kamere** su ograničenje, a ne mreža: 4 i 8 kamera rade istom brzinom.
- **`target_fps` određuje brzinu okidanja, a niz nikada ne radi brže.** `target_fps=1` pokreće niz na 1 fps; `None` (podrazumijevano) pokreće ga na najvišoj brzini koju host i mreža mogu podržati. Pin iznad te brzine spušta se na nju (vidi [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` od `4294` u `status()` nije brzina veze.** To znači da Windows nije mogao očitati brzinu adaptera (vozač je prijavio najveću 32-bitnu vrijednost), što je uobičajeno za USB Ethernet adaptere. Koristiva brzina USB adaptera na Windowsu u svakom je slučaju ograničena na 200 MB/s (vidi [Zdravstveno stanje masiva](#array-health--which-subsystem-is-losing-frames)); za podešavanje pogledajte odjeljak &quot;USB 10GbE adapteri&quot; u referenci CLI pod [Postavljanje i podešavanje mrežne kartice hosta](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Ograničenje brzine USB-a primjenjuje se i na Linuxu: 400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson.** Adapter se identificira putem sysfs-a (u suprotnom se koristi `ethtool -i`), stoga se USB 5 GbE ili 10 GbE adapteru postavlja ograničenje na 400 MB/s, umjesto da se oslanja na deklariranu brzinu od 500 / ~1060 MB/s. Podatak za x86_64 mjeren je na Linuxu: isti adapter koji na Windowsu kvari okvire iznad 200 MB/s pokrenuo je niz od 4 kamere bez problema pri svakoj razini do 480 MB/s. Na NVIDIA Jetsonu isti je adapter radio bez grešaka samo do 200 MB/s i povremeno gubio okvire iznad te brzine. Ako vaš host podnosi veće brzine bez `gvsp_corrupt` okvira, povećajte ga pomoću `wire_ceiling_mbps`. Na Linuxu adapteru na USB 2.0 priključku dodjeljuje se 30 MB/s; priključite ga u USB 3 priključak.

#### Prekomjerno pretplaćivanje (minimalna stopa po kameri)

Niz radi na bilo kojoj mreži; brzina snimanja je ono što ograničava. Sim-emit ritam dodjeljuje svakoj kameri jednak dio budžeta za sigurnu komunikaciju bez kolizija i sve se i dalje eksponiraju na istom okidaču, pa je na sporom vezu udio — i brzina sličica — malen: 8 punih-kamere rezolucije 12-bit na 1 GbE bez jumbo okvira dobivaju 6,5 MB/s svaka, što je oko **1 fps**(oko 1,3 fps pri komprimiranju). Udio je ograničen na najsporiju brzinu na koju se kamere mogu postaviti (`per_cam_floor_bps`: čitajte raspon GevSCPD svake kamere pri povezivanju, i nikada ispod 1 MB/s). Ako je mreža sporija od toga, veza se vraća na**serijski**emit (`slip-emit-and-capture`): kamere se izmjenjuju brzinom pune veze, pa svaki kadar stigne, ali članovi**nisu istovremeno izloženi**— sloj odgovora i blokatori (`SERIAL: ...`) to kažu; proslijedite `force_tier="sim-capture-sim-emit"` da odbijete povratak. Veličina okvira ne utječe na dijeljenje, pa**binning i ROI povećavaju brzinu okvira, ali ne mijenjaju hoće li niz stati**. Samo veza koja ne može prenijeti niz ni pri kojem ritmu, istovremeno ili naizmjence,**prenapreže se** — način kvara je gubitak GVSP paketa, a ne niža brzina slika. Rješenja: manje kamera, jumbo paketi od kraja do kraja, ili brži mrežni adapter.

- Odgovori `analyze_array_network()` i `/api/camera/array/connect` nose `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` i `per_cam_floor_bps`. Kada je `oversubscribed` istinito, projekcija **postavlja fps polja na nulu** (`achievable_fps_max` / `fps_bright` / `fps_dark`) umjesto da izvještava o zavaravajućoj, sporoj, ali funkcionalnoj stopi.
- `POST /api/camera/array/connect` prihvaća parametar tijela `pin_resolution` (samo HTTP — nije SDK kwarg; `connect_array` to ne izlaže). Radi se o strogoj načinu rada: preopterećeni connect s postavljenim `pin_resolution` **strogo se odbija** s greškom koja navodi svako rješenje. Bez njega, uspostava veze se nastavlja uz upozorenje da će niz gubiti pakete. U svakom slučaju, uspostava veze zadržava zatraženu veličinu okvira — nikada ne grupira niti obrezuje podatke sama od sebe.
- Samo za testiranje: Postavite `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` u okruženje pozadinskog sustava kako biste odbijanje smanjili na glasno upozorenje — svejedno uspostavljate vezu i prihvaćate gubitak paketa.

#### Stanje niza — koji podsustav gubi okvire

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) prenosi aktivni `health` blok na
povezanom nizu, ponovno procijenjen na **10-sekundnoj** kliznoj razdoblje. On razdvaja gubitak okvira
na dva uzroka kojima su potrebna suprotna rješenja, umjesto jedne &quot;nepotpune&quot; stope koja
ne navodi nijedan od njih:

| Polje | Što to znači | Koji podsustav |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (po serijskoj vezi) | Okvir je **stigao i bio strukturalno neispravan**— gubitak GVSP paketa. |**Mreža**: ograničenje propusnosti kabela, tempiranje, NIC RX prsten, MTU |
| `never_arrived_rate_pct` (po serijskoj vezi) | Okvir **nikada nije stigao**— kamera nije aktivirala snimanje ili ga ništa nije napustilo. |**Okidač / sinkronizacija**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Stopa najgore kamere za po kameri. | — |
| `per_cam_rate_pct` | Kombinirana stopa nepotpunosti po kameri (oba uzroka zajedno). | — |
| `stable_for_seconds` | Koliko je dugo svaka kamera ostala ispod 0,01 %. | — |

Uz `health`, isti zapisnik bilježi broj na kojem ovisi cjelokupna alokacija:

| Polje | Što znači |
| --- | --- |
| `wire_ceiling_mbps` | Trajna mrežna propusnost hosta, MB/s. |
| `wire_ceiling_source` | Odakle potječe taj broj, riječima — npr. `USB-capped 200 MB/s (was theoretical 1062; …)` ili `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` kada ga postavi `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` za USB Ethernet adapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Čitanje:** ne-nula `gvsp_corrupt_rate_pct` uz `never_arrived_rate_pct` na 0 znači da su aktivacija i sinkronizacija kabela savršeni te da se 100 % gubitka nalazi na mrežnom putu — niži
`wire_ceiling_mbps` i ponovno povežite. Suprotan obrazac umjesto toga ukazuje na sinkronizacijski kabel ili liniju okidača.

> **`target_fps` nije poluga za oštećene okvire.** GevSCPD tempiranje se piše jednom pri
> povezivanju, pa smanjenje stope okidača mijenja radni ciklus, a ne
> stopu istovremene emisije paketa. Mjereno smanjenje potražnje za 5× nije donijelo nikakvo poboljšanje, dok
> smanjenje gornje granice brzine prijenosa s 240 na 200 MB/s dovelo je kod iste konfiguracije do pada postotka oštećenih okvira s 10,4 % na 0,00 %.

> **Aktivni niz ne može se ponovno isplanirati na firmveru LATTICE (M3C / M3M).** Ne može
> to sam popraviti; isključite i ponovno spojite kako bi alat za odabir vremena povezivanja ponovno isplanirao
> prema novom ograničenju.

**USB Ethernet adapter je ograničen na 200 MB/s** od strane sonde bez obzira na njegovu
nazivna ploča: tablica učinkovitosti koja pretvara brzinu veze u održivu vrijednost potječe od PCIe-a, a USB mrežna kartica oglašava svoju Ethernet brzinu veze dok je ograničena USB sabirnicom i svojim upravljačkim programom. Ograničenje je apsolutno, a ne frakcijsko — USB 1 GbE adapter postiže ~80 MB/s i nije pogođen. Primjenjuje se na sustavima Windows i Linux (uključujući Jetson); pogledajte [Očekivane stope sličica](#expected-frame-rates) za mjesta gdje je mjereno.

#### `ArraySession` Metode

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Trenutno stanje aktivne naredbe: `target_fps` / `target_fps_auto` (stopa okidača i posjeduje li je kontroler stope), `target_fps_clamped_reason` (zašto se fiksirana stopa drži nižom, inače `None`), `actual_fps` (izmjereno; 0 kad se ciklus hvatanja odrekne ili zablokira), `recommended_fps`, `streaming` (False kad se grab loop odrekne — provjerite ovo, a ne samo `last_error`; ostaje True nakon ponovnog pokretanja postavki, što `restarting` označava), `stalled` (True dok se grab petlja pokreće, ali nije objavila nijednu grupu kadrova otprilike deset razdoblja kadrova, 2–30 s; `last_error` to navodi tek nakon 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` samo kada je niz bio povezan s PTP **i** svaki član je posljednji put pročitao `PtpEnable`; nije isto kao zaključano, što mjeri `ptp_epoch`), `ptp_configured` (ono što je connect zatražio), `ptp_off_members` / `ptp_unread_members` (članovi koji su zadnji put pročitali `PtpEnable` isključeno / nisu odgovorili), `member_recovery` (`None`, ili član koji je izgubio napajanje ili svoju vezu: `state` `waiting` dok ne odgovara, `reopening`, zatim `recovered`, s `members` i `why` — niz ga ponovno otvara kada ponovno odgovori i vraća mu PTP vremena povezivanja, spojke za okidač i sinkronizacijsku liniju, tempiranje i postavke kamere, zatim ponovno prenosi bez novih `connect_array`), i `ptp_epoch` (izmjereno), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` znači da Windows nije mogao očitati brzinu adaptera — vidi [Očekivane stope okvira](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (da li je lanac radijance po pojasu upotrebljiv i koji je član to prekinuo), i `band_clamp` — dok prikaz uživo iscrtava niz: udio osvijetljenih piksela koje je unmix zakvačio na 0, po članu (`members`: `{serial: [band0, band1, band2]}` u redoslijedu kamera R, G, B) i po pojasu prikazuje kompozit (`composite`: `{band name: share}`), s `warn`, udio koji GUI označava kao &quot;Podaci o pojasu niske pouzdanosti&quot; (mjereno otprilike jednom u sekundi; `None` kada se ne prikazuje živi prikaz), i `member_ae` — po članu, ekspozicija (`exposure_us`) i pojačanje (`gain_db`) zadnju objavljenu vrijednost koju je prenio, iz samog kadra (`read_nodes` od `ExposureTime` / `Gain` vraća posljednju vrijednost zabilježenu dok automatsko izlaganje radi, a ne ono što trenutno radi). |
| `set_member_settings(serial, **kwargs)` | Zapisati postavke jedne kamere iz grupe, kako ih `CameraSession.set_settings()` uzima (ekspozicija, pojačanje, ciljana svjetlina, ...); član ostaje u nizu. Poziva `ChlorosConnectError` za serijski broj koji nije član. Vidi [Ekspozicija na članovima niza](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Ono što je smart-prep odredio (razinu snimanja, veličinu okvira, `pixel_format`), blok uživo `health` i žičani proračun hosta. Vidi [Zdravlje niza](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Promijenite brzinu okidanja dok niz radi, bez ponovnog povezivanja. Broj ga pinira (kontrolor brzine ga ne dira); `None` vraća ga na automatsko. Vraća `{requested, target_fps, target_fps_auto, clamped}` — `clamped` je True kada je stopa smanjena na razinu koju host i mreža mogu podnijeti. Pogledajte [Intervalno snimanje za preglede dronovima](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Jedan kadar sa svake kamere, **svi s jednog okidača**, kao numpy nizovi (ni jedna datoteka nije zapisana). Vraća `GrabResult` (`dict`): `{serial: array}`, ili `{serial: (array, metadata)}` s `with_metadata=True`. `.group_id` je okidač koji dijele okviri; `.errors` mapira svaku izostavljenu kameru na razlog (kamera kojoj se u tom okidaču ne može poslužiti se izostavlja, nikada se ne popunjava drugom). `processing` je `raw` / `debayered` / `radiance` / `reflectance` / `display`. Isto se čita kao i kod projekta API-a [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — jedna zajednička implementacija — pa svaka ključna riječ tamo (`band`, `align`, `bit_depth`, `refl_max`) radi ovdje. Potrebno je `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator koji po svakom tik-u vraća jedan `grab()` rezultat (samo ključne riječi), tempirano prema `fps`, ograničen `duration` (sekundama) i/ili `count` (`None` = neograničeno). **Svaki dobiveni skup je novi okidač:** tick čiji je najnoviji okidač skup koji je već isporučio čeka sljedeći okidač umjesto da ga ponavlja, tako da stream teče brzinom koja je manja od `fps` i vlastite stope sličica niza. Podiže `FrameFetchError` umjesto da utihne: odmah kada svaka kamera iz nekog razloga odbije ponoviti sljedeći tick, nakon 60 s tickova bez sličice i nakon 60 s bez novog okidača (&quot;nema novog okidača 60 s&quot;). Ista implementacija kao `ArrayHandle.frame_stream()`. Vidi [Živi kadrovi u vlastitoj cijevi](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Pokrenite vlastito kontinuirano snimanje pozadinskog sustava (kontinuirani način rada GUI-ja) → [`ContinuousHandle`](#continuoushandle). Pohranjuje **svaku** sinkroniziranu grupu koju niz objavljuje, kao što bi je pohranio `capture()` (iste kontrole izvoza), sve dok se ne zaustavi ili dok `max_captures` / `duration_s` (`None` = dok se ne zaustavi). Pokreće `ChlorosConnectError` ako je jedna već u tijeku za ovaj niz. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Jedna sinkronizirana grupa za hvatanje. Vraća `CaptureResult` (popis frame dictova + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Proslijedi prethodni rezultat&#x27;kao `group_seq` kao `after_seq` kako bi dobio sljedeću sinkroniziranu grupu nakon nje, umjesto da čeka novu. Kontrole izvoza u nastavku. |
| `capture(..., smart=True)` | **Pametno snimanje** — čeka da se AE stabilizira na svim kamerama, a zatim pokreće snimanje. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Najbrže snimanje: samo sirovi podaci + dodijeljeno DAQ očitanje (+ besplatni kombinirani indeks). Odgovara gumbu &quot;Fastest Capture&quot; u GUI-ju. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Pojedinačno / Neprekidno / Intervalno u jednoj ograničenoj petlji. Vraća `list[CaptureResult]`.**Zahtijeva `count` i/ili `duration_s`** pa se prekida (SDK nema Ctrl+C). Također završava ranije, nakon prolaza, kada pas prijavi punu vožnju (unosa `.failed` s `disk_full`); taj pas je posljednji rezultat. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Pokrenite snimanje uživo kombiniranog indeksnog prikaza u video/GIF → `RecorderHandle`. Video je Motion-JPEG `.avi` (nije potreban ffmpeg). Kad nije otvoren GUI prikaz, pozadinski sustav ga renderira: `expr` (izraz indeksa u nazivima pojaseva niza, npr. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), ili prema zadanim postavkama NDVI niza. Jedan kompozitni snimač po nizu. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Pokrenite burst visokog fps-a u sirovom Bayer formatu → `RecorderHandle`. Ponovno obradite izvan mreže s `build_video()`. `orient=True`: proizvodi koje stvara `build_video()` okreću se prema orijentaciji kamere (proizvodi po kameri prema orijentaciji vlastite kamere, a kombinirani indeks prema orijentaciji glavne kamere); sirovi kadrovi ostaju u izvornom obliku, a zapisnik serije bilježi orijentaciju svakog člana. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Ofline-ponovno obrađuje spremljenu sirovu seriju u kalibrirane videozapise. Blokira dok se ne završi (`wait=True`) i vraća rezultat izrade: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — vidi [rezultat izrade](#the-build_video-result) dolje. |
| `build_video_status(job_id, timeout=15.0)` | Provjerite status offline posla izrade: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Objavite cijelu niz — **ako ju je ova sesija otvorila** (`connect_array`): tada je `force_disconnect()`, i podiže `ChlorosConnectError` kada oslobađanje nije potvrđeno (izlaz iz bloka `with` nikada se ne aktivira). Na handleu iz `attach_array` ostavlja niz da radi i ispisuje jedan redak u stderr s tim informacijama: niz može pripadati GUI-ju ili drugoj skripti. Svaka se kamera ostavlja da radi neometano i u ostalom onako kako ju je connect pronašao; `keep_settings=True` sprema svoje postavke u vlastitu memoriju kamere, `clear_settings=True` resetira tu memoriju na tvorničke postavke (vidi [Memorija kamere](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Oslobađanje cijelog niza bez obzira na to tko ga je otvorio. Koristite ga na priloženoj ručki kada želite završiti niz (npr. da biste ga ponovno povezali s različitim postavkama). Vraća vrijednost čim pozadinski sustav potvrdi oslobađanje ili kada nema ničega za osloboditi (niz je već uklonjen ili pozadinski sustav nije pokrenut); ručno oslobađanje koje je isteklog vremena ili koje je pozadinski sustav odbio izaziva `ChlorosConnectError`. S bilo kojom zastavicom vraća ono što je memorija svake kamere dobila (`camera_memory`), inače `None`. |

`capture()` kontrole izvoza (isti krajnje odredište koje koriste GUI/CLI):

- `processing` / `levels` — `processing="all"` (ili `levels=["raw","radiance",…]`) sprema svaki primjenjivi tip izvoza po kameri; jedna vrijednost `processing` sprema samo tu razinu.
- `aligned` — **podrazumijevano uključeno (`None`), budući da snimanje iste nizove aplikacije**Chloros**(njegova postavka snimanja**Poravnano** po nizu). Ne-sirovi izvoz svakog člana deformira se u kadar glavne kamere pomoću [profila poravnanja] niza(#array-alignment) i obrezano na zajednički preklop članova, tako da su datoteke pikselski poravnate i izlaze veličine, obrezane i s oznakama `Alignment*` koje aplikacija postavlja (pod ROI transformacija se ponovno temelji na prozoru); `XMP-mapir:AlignmentJSON` kaže `"applied": true` sa svojim `crop_roi`. Raw ostaje neizobličen u nativnoj veličini, ali nosi transformaciju u metapodacima. `aligned=False` sprema svaku datoteku u nativnoj veličini, neizobličenu; `aligned=True` je zadana razrađena opcija. `continuous()`, `capture(smart=True)`, `capture_repeated()` i `ArrayHandle.capture()` / `capture_smart()` na razini projekta koriste istu zadanu postavku. Neovisno o tome je li warp primijenjen, svaka datoteka člana bilježi transformaciju `Alignment*` svoje kamere čim niz dobije profil poravnanja — osim kamere koju profil ne poravnava (izostavljena, ili prisiljena na identitet neuspjelim rješavanjem), i referentna kamera kada nijedna druga kamera nije poravnata: one nemaju oznake `Alignment*`. Bez profila na nizu snimanje se sprema neporavnato, a rezultat to i pokazuje: `result.alignment` je `{"requested": True, "applied": False, "warnings": [...]}`, jedno upozorenje po razlogu zbog kojeg se warp nije mogao primijeniti (nema profila, članovi u različitim rezolucijama, niz koji nije moguće poravnati-sinkroniziran, kadar na koji se profil ne može mapirati). `applied` je `True` samo kada je svaki izvoz koji je zatražio warp dobio warp. Automatsko poravnanje niza generira profil čim se njegova ekspozicija stabilizira; `ArrayHandle.calibrate_alignment()` / `align-calibrate` u CLIu generiraju profil na zahtjev. `result.oriented` navodi je li `orient=True` pretvorio proizvode.
- `render_index=False` — preskoči preklapanje indeksa vegetacije po kameri; zadano ga iscrtava tamo gdje je konfigurirano.
- `daq` / `daq_tolerance_ms` — DAQ očitanje prema zadanim postavkama sprema se kao pomoćna datoteka (sidecar) formata `.daq`, čak i kada je to nepotrebno za odabranu razinu; `daq=False` tada to preskače, i `daq_tolerance_ms` postavlja toleranciju podudaranja (zadano 500). `force_daq=True` je stariji oblik zadane vrijednosti. Vidi [The DAQ reading a capture saves](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — pretvoriti **proizvode**svakog člana (`display`, `radiance`, `reflectance`, indeks po kameri i kompozit) prema orijentaciji kamere, nakon deformacije poravnanja: prema orijentaciji**glavne**kamere kada su deformirane u njezin sustav (podrazumijevano za `aligned` — članovi ostaju su-registrirani), inače prema orijentaciji vlastite kamere.**Zadano: svaka izvornik datoteka člana**, kao što je uvijek slučaj kod snimki nizova; `raw` i `debayered` ostaju u izvornom obliku u svakom slučaju. Svaka datoteka bilježi orijentaciju okvira u kojem se nalaze njezini pikseli (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — sirovi, vlastite kamere), i `Alignment*` XMP ostaje transformacija izvornog okvira, primijenjena prije orijentacije; raw koji nosi tu transformaciju također bilježi `ChlorosAligned*` (majčin), orijentaciju proizvoda iskrivljenog u majčin okvir. Obrada okreće svaki proizvod prema onome što pripada njegovim pikselima: poravnati kada iskrivljuje raw (Poravnanje niza uključeno), onaj vlastite kamere kada ne. `capture_repeated()` i `capture_fastest()` ga propuštaju; `ArrayHandle.capture()` / `capture_smart()` na razini projekta također ga uzimaju.

**TIFF kompresija (kontrola samo za HTTP):** `ArraySession.capture()` ne šalje `compression` ključ, pa se primjenjuje zadana postavka pozadinskog sustava — `POST /api/camera/array/capture` čita parametar tijela `compression`, `"none"` prema zadanim postavkama: nekomprimirano, pikseli senzora točno onako kako su očitani (~6,3 MB po slici pune rezolucije). `"deflate"` (bezgubitni zlib L1 + horizontalni prediktor, ~4,1 MB/okvir) zapisuje manje datoteke sporije — oba su bezgubitna i čitaju se identično pri uvozu. SDK ne izlaže nikakav kwarg za to; zaobilazno rješenje je `chloros-cli lattice array-capture --compression deflate` ili sirovi HTTP. DEFLATE drži Python GIL, pa komprimirano pisanje nije paralelno na nitima pisanja po kameri — za kontinuirano snimanje pune rezolucije brzinom senzora potreban je nekomprimirani zadani način. Detalji: [CLI Referenca → niz-zadavanje](cli-reference.md).**Uzastopne grupe (HTTP):** krajnja točka vraća `group_seq`; petlja koja je pošalje natrag kao `after_seq` u svom sljedećem zahtjevu dobiva sljedeću sinkroniziranu grupu — uključujući i onu objavljenu dok se prethodno snimanje spremalo — umjesto da se čeka na grupu objavljenu nakon zahtjeva, a `skipped_groups` izvještava o svima koje je petlja propustila. `capture_repeated`, CLIov `--continuous` i GUI-jev kontinuirani način rada (Continuous mode) sve to rade; intervalni prolazi ostaju ažurni u svakom zakazanom trenutku.**Prekoidni izvoz (per-member export overrides) (samo za HTTP):**isti krajnji punkt također prihvaća `exclude_serials` (popis — uklanjanje članova iz spremljenog skupa; niz se i dalje aktivira kao jedna sinkronizirana skupina, a isključeni članovi vraćaju se u `excluded`), `serial_levels` (`{serial: [level tokens]}` preusmjeravanja na razini kamere), i `serial_index` (`{serial: bool}` preusmjeravanja indeksnog preklopa na razini kamere). Ovo su GUI-parity parametri tijela i**još nisu **SDK**kwargs**; članovi koji nedostaju u mapama vraćaju se na razinu niza `levels` / `render_index`.

##### Pregledavanje preskočenih kamera — `CaptureResult.skipped`

`ArraySession.capture()` vraća `CaptureResult`, koji je podklasa `list`: iterirajte ga, indeksirajte ga, `len()` to — svaki postojeći uzorak i dalje radi. Novi kod može pregledati atribut `.skipped` kako bi vidio koje su kamere isključene i zašto. Najčešći slučaj su kamereRGB u nizu miješanih filtara kada zatražite `processing="radiance"` ili `"reflectance"` — Bayerova radijanca nema smisla za širokopojasni senzor, pa pozadinski sustav preskače te kamere umjesto da proizvodi besmislice.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeni razloga slijede uzorak `<level>-not-applicable-to-rgb-cam` (jedan unos po preskočenoj razini, od kojih svaki sadrži `level`). Kamera čiji paket kalibracije nije učitan preskače `radiance` / `reflectance` s `<level>-needs-calibration (connect the camera so its cal pack loads)`, i sa neupotrebljivim DAQ očitanjem reflektancija preskače s `reflectance-skipped-no-dls`. Ostali preskoci specifični za reflektanciju su `reflectance-skipped-no-fresh-dls` (nije dostupno novo očitanje dolaznog zračenja), `reflectance-skipped-bound-daq-unavailable (…)` (vezani DAQ nije dostupan), i `dls-uncalibrated-band-<nm>` — pojas se većinom nalazi izvan radiometrijski kalibriranog raspona DAQ senzora svjetla (~374–974 nm), pa se odbija apsolutna podjela reflektancije temeljena na DAQ-u i okvir se glasno spušta na razinu senzorske karakteristike. Među isporučivim SKU-ovima samo F988 to pokreće; podržani tijek rada te kamere je rad sa zrcalom za reflektanciju.

##### Datoteke koje nije bilo moguće zapisati — `CaptureResult.failed`

Snimanje koje je moglo zapisati samo neke od svojih datoteka i dalje vraća: popis sadrži svaku datoteku koja **je** napisana, a `.failed` navodi ostale, po jedan dict po datoteci s `serial` (`None` za kombinirani kompozit i `.daq` sidecar niza), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, ili `match` — porijeklo `.daq` od `.match.json`, kada je `.daq` napisan, ali njegov sidecar nije) i `error`. Na punom disku zapis također sadrži `disk_full: True`; svako kasnije snimanje na taj disk završilo bi na isti način, zbog čega se `capture_repeated()` tu zaustavlja. Ne javlja se nikakva pogreška za djelomično snimanje, a datoteka u `.failed` nije ostavljena na disku nepotpuno zapisana. `.failed` je prazna kod čistog snimanja.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Učitavanje snimke DAQ-om sprema — `.daq_files` i `.daq_matches`

Kad god je DAQ povezan s kamerom ili nizom, ili je objedinjen, svaka snimka sprema niz koji je koristila: a `.daq` u podmapu `daq/` mape snimanja, sa svojim `.match.json` i svojim `.cameras.json` (čije je očitanje). To vrijedi i za sirovo ili debayered snimanje, a to je ista datoteka bez obzira je li snimanje došlo iz GUI-ja, CLI-a ili SDK-a. `daq=False` je preskače za snimanje čiji razini ne koriste DAQ (level `radiance`, `reflectance` ili `display` ga i dalje zapisuje); `daq_tolerance_ms` pomiče liniju unutar-tolerancije za taj snimanje, a nikada korišteno očitanje. Koji DAQ: onaj povezan s kamerom ili nizom (odabir senzora svjetlosti u GUI-ju, veza projekta ili `chloros-cli daq pool-bind`), inače najsvježiji koji backend ima. CLI prikazuje kontrole `--no-daq` i `--daq-tolerance-ms` ([CLI Reference](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Zapisivanje sprema očitanje senzora svjetla koje je upotrijebilo kao `.daq` s `<file>.daq.match.json` uz njega, navodeći kako je očitanje bilo povezano s ekspozicijom: `method`, `gap_ms` (od ekspozicije do najbližeg DAQ očitanja) i `in_tolerance` (unutar 500 ms, ili `daq_tolerance_ms` snimanja)XMP datoteke reflektancije nosi istu odluku kao `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` i `Chloros:DlsInTolerance`.

Prikupak čeka prvo očitavanje DAQ-a nakon ekspozicije kako bi mogao spojiti očitanja s obje strane (`method` `interpolated`). DAQ-E šalje očitanje svakih 0,4 s po danu i svakih 1–2 s u zatvorenom prostoru, stoga `capture()` vraća očitanje otprilike nakon jednog i pol takvih intervala; sama ekspozicija nije odgođena. DAQ koji je prestao slati se ne čeka. Očitanje DAQ-E-a je prosjek svjetla tijekom intervala od prethodnog očitanja, stoga se postavlja na sredinu tog intervala: `gap_ms` mjeri se do te sredine, a `.match.json`&#x27; `bracket_stamps_ns` daje vlastite vremenske oznake za očitanja. `ArraySession.continuous()` čeka bez usporavanja snimanja. `capture_repeated()`-ov povratak-povratni prijelazi ne čekaju, stoga zadržavaju svoju brzinu i usklađuju se s najbližim očitanjem prije ekspozicije.

Očitavanje nikada ne istječe, pa nijedan kadar nije izgubljen zbog kasnog očitavanja — umjesto toga, označava se:

- **Prijenos DAQ-a je zastao** — koristi se njegovo najnovije očitanje, a `gap_ms` navodi koliko je staro (`in_tolerance: false` starije od 500 ms).
- **Nema DAQ-a za čitanje**— isprekinuta veza (`disconnect()`, `chloros-cli daq pool-disconnect`), ili se povezanog DAQ-a ne može dohvatiti — zadnje čitanje kamere se ponovno koristi i opisuje se u odnosu na**ovo** ekspozicija: `method` `retained`, `gap_ms` iz ove ekspozicije, `in_tolerance` ponovno izračunato, i `retained_from_epoch_ns` imenuje ekspoziciju s kojom je prvi put upareno. Kamera koja nikada nije imala očitanje i dalje ne dobiva refleksanciju, a rezultat objašnjava zašto (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ili odbijanje snimanja).

Datoteke koje snimka sprema unutar mape projekta otvorenog u Chloros pridružuju se popisu datoteka tog projekta, kao što to čini snimka u aplikaciji (datoteke spremljene drugdje ostaju nepromijenjene): `CameraSession.capture()`, `ArraySession.capture()` (i `smart=True` također) i `capture_cameras_transient()`. Rezultat to navodi u `.project_add` (`capture_cameras_transient()`: `project_add` rječnika) -- `held`, `files` i `message` pozadinskog sustava; `held` je `True` dok je izvršavanje u tijeku, a datoteke se spajaju kad ono završi. `None` kad nijedna datoteka nije stigla u projekt.

`CameraSession.capture()` i `ArraySession.capture()` vraćaju ih kao `.daq_files` i `.daq_matches` rezultata, kao HTTP snimljene odgovore (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) sadrže `daq_files` i `daq_matches`, po jedan po `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` i `stream_fault` — vlastita pogreška toka DAQ-a, koja se provjerava samo za očitanja izvan tolerancije. Prije nego što se oslonite na reflektanciju iz snimanja, provjerite `in_tolerance` u njegovom `.match.json` ili XMP-u, ili DAQ-ov [`status()`](#daqsensorsession-methods).

Razine `processing`:

| Razina | Izlaz |
| --- | --- |
| `"raw"` | Jednokanalni Bayer (monokamere: pojedinačna traka) izravno sa senzora. |
| `"debayered"` *(zadano u SDK-u)* | 3-kanalni boji: dark offset (DSNU) kalibracijskog paketa uzet iz sirovog podatka, zatim OpenCV-ov demosaicing osjetljiv na rubove — bez LUT-a za linearnost, flat field ili unmix (mono kamere: 1-kanalni sivi tonovi). Spremljena datoteka TIFF u **RGB**redoslijedu kanala (fotometrijski RGB, pa svaki preglednik slika prikazuje istinsku boju); nizovi u memoriji `grab()` / `frame_stream()` koji se vraćaju su u OpenCV-ovom**BGR** redoslijed — obrnuti kanali (`frame[..., ::-1]`) za usporedbu jednog sa spremljenom datotekom. `chloros-cli process` zapisuje isti proizvod kao `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm putem cijelog radiometrijskog lanca. Multispektralno samo — RGB kamere se preskaču. |
| `"reflectance"` | uint16 0..32768 (spreman za Pix4D); zahtijeva uparivanje DAQ-a uživo za apsolutnu referencu. Samo multispektralno. |
| `"display"` | Cijeli lanac odgovara pretpregledu u GUI-ju (CCM + WB + gamma prema profilu kamere). |
| `"all"` | **Jedna datoteka po primjenjivoj razini** za svaku kameru (odgovara zadanoj naredbi GUI-ja &quot;Capture All&quot; / CLI). Vraćeni `CaptureResult` zatim sadrži jedan frame dict po `(cam, level)`, s razinom u svakom dict-u; neprimjenjivi nivoi pojavljuju se u `.skipped`. DAQ očitanje korištenog snimanja spremljeno je kao `.daq` pomoćna datoteka. |

> **Napomena — zadana vrijednost se razlikuje od**CLIa.** `ArraySession.capture()` zadano je `processing="debayered"`; naredba `chloros-cli lattice array-capture` zadano je `processing="all"`. Prenesite `processing="all"` izričito iz SDK-a kako biste replicirali višerazinsko spremanje CLI/GUI-ja.

##### Nazivi datoteka za snimanje niza

Svaka se razina sprema u vlastitu podmapu `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` za `display`, `index/`, `composite/`). Svaka datoteka imenuje svoju kameru kao `SN<serial>-<filter>`: serijski broj, zatim dio modela kamere koji se odnosi na filtar — `SN214000533-FOCN` za `M3C-L87-FOCN`, `SN261902142-F450` za `M3M-L87-F450` — spojeno s `-`, tako da ime ima ista polja odvojena točkom-zarezom kao i prije. Kamera čiji se model ne može pročitati je sama `SN<serial>`, a snimke iz ranijih izdanja nemaju filtar u nazivu. Naziv datoteke ovisi o tome koliko je razina član zatražio:

| Član je zatražio | Naziv datoteke | Primjer (`reflectance`, zadano `format="tiff"`) |
| --- | --- | --- |
| **Jedna razina** (`processing="reflectance"`, ili `levels` za jednu stavku) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — bez sufiksa razine, kao u ranijim izdanjima, tako da postojeći skripti i dalje rade | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Više razina** (`processing="all"`, ili `levels` s više od jedne stavke) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[clamp mask](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) slijedi naziv proizvoda, `clamp_masks/<product folder>/<product name>_clampmask.tif`: jedan-levelno snimanje reflektancije zapisuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, dok `processing="all"` zapisuje `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Pravilo za nastavak određuje se za svaki član razina koje su zatražene, pa preskočena razina (npr. reflektancija na kameri s oRGB) ne mijenja to pravilo. Čitajte putanje iz vraćenih kadrova umjesto da ih sami gradite. Snimanje s jedne kamere (`CameraSession.capture()`) uvijek dodaje razinu: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

PNG i JPEG podržavaju samo proizvode s prikazom i indeksom

PNG ili JPEG podržavaju 8 bita po kanalu. Zahtjev za razinu koju format ne podržava bit će odbijen prije snimanja, uz poruku koja navodi tu razinu i TIFF kao format koji je podržava:

- **`radiance`**(float32) i**`reflectance`**(uint16, `32768` = 1.0) — i tako dalje do `processing="all"`, koji ih uključuje. SDK provjerava ovo na argumentima: `capture()` podiže**`ValueError`** bez zahtjeva (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` iz kamere koja streamuje više od 8 bita** (12-bitni M3C, Mono12 M3M) — backend to odbija, znajući dubinu bita svake kamere: HTTP 400 s `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, vraćen kao `ChlorosConnectError` (`CameraSession` / `ArraySession`) ili `RuntimeError` (`CameraHandle` / `ArrayHandle`). Jednopoziomno snimanje `debayered` jedne kamere hvata 8-bitni kadar za PNG/JPEG i nije odbijeno.

Spremite radiometrijske podatke kao `format="tiff"` (`ext=".tiff"`); koristite PNG/JPEG za `display` i indeksne prikaze.

#### Kako se ponaša automatsko eksponiranje

- **Prvo eksponiranje, zatim pojačanje.** Kada je pojačanje na automatskom načinu rada, ono ostaje na 0 dB sve dok eksponiranje ne dosegne najdužu dopuštenu vrijednost.
- **Pri ručnom podešavanju ekspozicije, pojačanje ostaje na automatskom načinu rada** sve dok ne promijenite pojačanje ili ga ne postavite na ručni način rada. Kada se vratite na automatski način rada, ograničenje pojačanja je isto kao i prije.
- **Mali naglasci ne zamračuju scenu.** Specularni odsjaj do 1 % kadra se zanemaruje, a na panelima poznatog kalibracijskog cilja do 0,5 % panela.
- **Gornja granica ekspozicije DAQ-a primjenjuje se samo dok je kalibracijski cilj u vidokrugu.** Time se sprječava preeksponiranje bijelog panela; kada se ne vidi nijedan panel, kamera eksponira scenu.
- **Stalno svjetlo ostaje stabilno, a promijenjeno svjetlo se stabilizira u sekundi.** Pri stalnom svjetlu ekspozicija i njezina granica ostaju nepromijenjeni, a nakon velike promjene svjetla prikaz se stabilizira unutar nekoliko sekundi. Ograničenje više ne blokira ekspoziciju trajno, a promjena ROI-ja ponovno pronalazi ekspoziciju.
- **Ciljana svjetlina ostaje unutar dosega.** Ne raste dok je kamera na svojoj granici ekspozicije i njezin pojačanje ne može porasti, tako da se svjetlost koja se vrati ne odbljeskuje na kadru.
- **Kamera čije automatsko izlaganje prestane reagirati zadržava se na razini za svjetlo u kojem se nalazi.** Na jakom suncu kamera bi mogla prijeći na najkraće izlaganje i ostati tamo s tamnim kadrom, a sva njezina podešavanja pokazivala bi normalno. Chloros prati kadar, drži tu kameru na razini koju je zadržala za to svjetlo i pušta je kada se svjetlo promijeni. Tamni ili prekriveni senzor svjetla ne ometa to.

#### Ekspozicija na članovima niza

LATTICE kamera postavlja `ExposureTime` na jednu periodu svoje brzine snimanja. Dok niz prenosi, Chloros pokreće svaki član s uključenim `AcquisitionFrameRateEnable`, brzinom nešto iznad brzine okidača, tako da gornja granica ekspozicije člana leži malo ispod razdoblja okidača, **na bilo kojem ROI-u**:

- Razdoblje sličice člana je razdoblje okida umanjeno za odmak od 15 % razdoblja (najmanje 3 ms, najviše polovica razdoblja), a njegov gornji limit ekspozicije je to razdoblje sličice umanjeno za otprilike 0,1 ms. Primjer: okidač od 10 fps (100 ms) daje stopu člana od oko 11,8 fps i gornju granicu izloženosti od oko 85 ms.
- **Manualni `ExposureTime` duži od tog ograničenja usporava okidač kako bi se uklopio.** Chloros prvo otvara ograničenje tog člana za vrijednost i dopušta sporijoj brzini da radi nekoliko okidačkih perioda (pisanje može potrajati nekoliko sekundi dulje), zatim ga zapisuje, i okidač se aktivira najviše onoliko brzo koliko ekspozicija dopušta. Na nizu automatske brzine, brzina kojom je radilo prije vraća se čim se svaki član čija je ručna ekspozicija usporila vrati na automatsku ekspoziciju. Vrijednost koju sama kamera ne može snimiti i dalje se odbacuje, pri čemu se raspon koji prihvaća sada navodi u poruci o pogrešci, a automatska ekspozicija člana vraća se na prethodno stanje ako je isključena samo za to pisanje. Prije verzije 1.3.0, mali ROI ograničavao je svaki član na vlastito razdoblje slobodnog rada senzora, bez obzira na brzinu okidača: 4,9 ms pri 768 × 304, pa nijedan član nije mogao biti osvijetljen.
- **Chloros nikada ne skraćuje ručno podešenu ekspoziciju kako bi odgovarala bržem okidaču.** Automatska brzina ostaje unutar onoga što ručne ekspozicije dopuštaju; fiksirana brzina koja je brža od dopuštene ekspozicije ostavlja tom članu gornju granicu dovoljno dugu za njega i prihvaća okidače ne brže od onoga što njegova ekspozicija dopušta. Smanjite ekspoziciju ili pin kada svaki član mora izvršiti svaki okidač.
- **Očitavanje natrag:** čitanje čvorova člana prikazuje `AcquisitionFrameRateEnable` kao Istina i `AcquisitionFrameRate` na brzini člana; omogućavanje se isključuje kada niz prestane.
- **Niz posjeduje ta dva čvora. Ne pišite `AcquisitionFrameRateEnable` ili `AcquisitionFrameRate` na član niza** — postavite nizpostavite stopu pomoću `target_fps` / `set_target_fps()` umjesto toga.

Napišite izloženost člana pomoću `arr.set_member_settings(serial, exposure_time=20000)` (ključne riječi `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` radi istu stvar). Član ostaje u nizu. Nemojte otvarati član niza pomoću `connect_camera()`: connect odbacuje kameru koja je već u nizu, a `CameraSession` koji vraća uklanja kameru iz niza kada se veza prekine. `ArraySession.status()["member_ae"]` izvještava o svakom članuekspoziciju i pojačanje iz svog najnovijeg kadra.

### Jedan zapis projekta, svaki način rada

Mapa projekta sadrži postavke svakog uređaja: `cameras.json` — svake kamere (`cameras[].settings`, za pojedinačnu kameru i za svakog člana niza) i one za cijeli niz (`arrays[].capture`: format, ROI, binning, brzina okidača, budžet žica) — i `sensors.json`, za svaki senzor svjetla (vrijeme integracije, prosjek kadra, cap). Aplikacija, CLI, SDK i MCP poslužitelj svi čitaju i zapisuju iste datoteke, na isti način:

- **Uređaj povezan pod projektom sprema se u njega i bilježi svaku promjenu u njega.** &quot;Pod projektom&quot; znači s projektom otvorenim u pozadini (backendu) aplikacije ili — putem `ChlorosProject` (`open_project()`), koji naziva vlastiti mapu u backendu (`project_path`). Kamera, niz ili senzor svjetla koje projekt još ne sadrži dodaju se u njegove datoteke kada se povežu (`connect_camera()`, `connect_array()`, povezivanje senzora svjetla, CLI, aplikacija, `chloros-mcp`), sa svojim postavkama. Promjena iz bilo kojeg načina rada (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, postavke niza, `set_cap()`, CLI, kontrole aplikacije) spremaju se u datoteke tog projekta nakon kratkog vremena. Ekspozicija, pojačanje ili cilj AE postavljen na kameru spremaju se kao &#x27;držano&#x27;, pa se pri ponovnom otvaranju ponovno drže.
- **Otvaranje projekta postavlja njegove uređaje iz njega**, u svakom načinu rada: kamera, niz ili senzor svjetla priključen bez vlastitih postavki dobiva projektovs spremljenima (`lattice cam-connect` / `array-connect` s otvorenim projektom, `chloros-cli project connect`, `open_project().connect_all()`, ponovno otvaranje aplikacije). Niz povezan pod novim ID-om ažurira niz koji je projekt spremio sa istim kamerama; niz s drugim kamerama dodaje se uz njega.
- **Vlastito spremanje aplikacije nikada ne prepiše aktivna postavljanja uređaja** svojom starijom kopijom, te zadržava po-članove preusmjeravanja spremljene od strane pozivatelja iz CLIa ili SDKa.
- **Pikselni format koji postavite je onaj koji projekt zadržava.** Format postavljen u Postavkama niza, pomoću `--pixel-format` ili `connect_array(pixel_format=)`, uključujući 8-bitni, spremljen je i točno se ponovno povezuje. Ako nije postavljeno ništa, svaka se kamera povezuje s postavljenom zadanoj 12-bitnoj postavci (`BayerRG12`, `Mono12` na mono senzoru). Naziv formata označava njegovu dubinu bita; obitelj je senzora, pa se bojna kamera nikada ne postavlja na mono format, niti mono kamera na Bayer format, a linija `[PixelFormat <serial>] ... written` to navodi kada je format mapiran. Izvozi 8-bitnog snimanja su 8-bitni, uključujući reflektanciju: ponovno skalirani i označeni svojim `PixelScale`.
- Uređaj spojen bez projekta bilježi ništa.
- **Snimanje prati projekt stvoren nakon njega; otvorena ostaje sa svojim projektom.**Kada se stvori projekt — `create_project()`, `chloros-mcp`-ov `create_project` ili aplikacijin**Novi projekt**— ili duplicirano (opcija**Dupiliraj**u aplikaciji, koja otvara kopiju) dok snimanje još uvijek piše u projekt koji je bio otvoren prije, snimanje se nastavlja i ono što piše završava u novom projektu, na istom mjestu unutar njega (dolje). Otvaranje postojećeg projekta — `ChlorosProject.process()` (koji otvara njegov projekt u pozadini), `chloros-cli process`,**Otvori projekt** aplikacije — ostavlja snimanje u tijeku s projektom u koji snima: ono se završava ondje, a njegove datoteke pridružuju se popisu datoteka tog projekta (aplikacija zaustavlja svako snimanje prije otvaranja projekta).
  - `ArraySession.record()` i `ArraySession.burst()`: videozapis ili mapa burst sadržaja sa svakim okvirom premješta se tamo u cijelosti kada se snimanje zaustavi. Rezultat zaustavljanja izvještava o novoj putanji (`out_dir`, `video_path`), i izgradnja burst-a radi s njega. Premještanje koje pogon odbije ostavlja snimku tamo gdje je napisana, a rezultat to i navodi (`relocation_error`). Sirovi kadrovi zaustavljenog burst-a i `.daq` pridružuju se popisu datoteka projekta u koji se završi; videozapis se ne dodaje u popis datoteka projekta.
  - `ArraySession.continuous()`: svako snimanje nakon promjene spremno je u novi projekt; one spremljene prije ostaju tamo gdje jesu.
  - `DAQSensorSession.record_start()` (i vlastito snimanje aplikacije senzorom svjetla): `.daq` se premješta u mapu `light_sensor` novog projekta kada snimanje prestane; `record_stop()` vraća novu putanju, a datoteka se pridružuje popisu datoteka tog projekta.

  Snimanje u mapu izvan otvorenog projekta ostaje tamo gdje jest, a `start_camera_continuous()` / zadaci s više streamova zadržavaju mapu koju su dobili.
- **U aplikaciji, snimke nakon novog projekta idu u njega.** Kada se stvori ili duplicira projekt, svaka se snimljena sesija i snimanje nastavljaju, a svaka snimka napravljena nakon toga (Snimanje svega, Početak snimanja) sprema se u novi projekt i dodaje na njegov popis datoteka. Otvaranje postojećeg projekta u aplikaciji prvo zaustavlja sve snimke.
- **Projekt se ne premješta dok se snima.**Premještanje otvorenog projekta u drugi radni direktorij (u postavkama aplikacije →**Radni direktorij**; poziv iz SDK-a ne premješta projekt, prijenos otvorenog projekta u drugi radni direktorij (Postavke aplikacije → Radni direktorij; poziv SDK-a ne premješta projekt,   Ovo je odvojeno od toga kako su kamere ostavljene pri prekidu veze (vidi [Memorija kamere](#camera-memory--keep_settings--clear_sett)  Ova radnja se odbija HTTP 409 `PROJECT_RECORDING` dok se u njega piše tijekom snimanja niza ili rafala, kontinuiranog snimanja niza, kontinuiranog/višestrukog streamanja kamere ili snimanja senzora svjetlosti: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Ovo je odvojeno od toga kako su kamere ostavljene pri odspajanju (pogledajte [Memorija kamere](#camera-memory--keep_settings--clear_settings)): projekt sam postavlja svoje kamere kada se otvori.

### Sjećanje kamere — `keep_settings` / `clear_settings`

Kada je niz povezan, njegove kamere preuzimaju postavke niza, a vi ih zatim možete mijenjati. Kada Chloros otpusti kamere — `ArraySession.disconnect()` / `force_disconnect()`, na kraju prolazne funkcije, zaustavljanju `start_camera_continuous()` — vraća svaku u stanje u kojem je bila prije povezivanja: žičanje okidača i sinkronizacijske linije, PTP i postavke kamere — `ExposureAuto` / `GainAuto` (i ručni `ExposureTime` / `Gain`), `TargetBrightness` i ograničenja AE-a i prigušivanje, bilancu bijele boje, razinu crne boje, `PixelFormat`, binning i ROI, pipeline boja (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP-jev `PtpSlaveOnly` / `PtpDomainNumber`, veličinu paketa (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), tempiranje (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), podatke o komadima (`ChunkModeActive`, svaki `ChunkEnable` pod svojim `ChunkSelector`), trigger-ulazni deglitch niz daje sinkronizacijsku liniju robota (`LineFilterSelector` / `LineFilterWidth`) i brzinu snimanja okvira (`AcquisitionFrameRate` samo kada je njegov enable bio uključen: kada je isključen, brzina čitanja je najviše što ritam dopušta, a ne postavka). Pisanje se vrši samo za promijenjene vrijednosti, a kamera se prvo zaustavlja, zatim se postavlja ritam i brzina sličica prije ekspozicije koja zahtijeva njihov period sličica.

Za korištenje kamera izvan projekta, dva argumenta pišu u **vlastitu memoriju** kamere — ono s čime se ona pokreće, u bilo kojem softveru (na `disconnect()` / `force_disconnect()`, svaka prolazna funkcija, `start_camera_continuous()` i `ChlorosProject.disconnect_all()`; MCP: isti argumenti kao kod `disconnect_device` i prolaznih alata; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** sprema postavke svake kamere onakvima kakvima su ostale nakon sesije u svoju memoriju (UserSet2, postavljeno kao početni skup). Kamere su i dalje ostavljene da rade neovisno (`TriggerMode` `Off`), a ožičenje sinkronizacijske linije je ostavljeno kako se našlo. Kamera bez UserSet2 sprema u UserSet1; ovo računalo to pamti, pa Chlorosovo samozdravljenje niza — koje resetira UserSet1 koji se udaljio od Chlorosove osnovne postavke — ostavlja ga netaknutim.
- **`clear_settings=True`** resetira memoriju svake kamere na tvorničke postavke: osnovnu postavku Chlorosa, onu koju njegovo samoliječenje niza utvrđuje — tvorničke postavke s `TriggerMode` `Off`, `TriggerSource` `Line2`, automatska ekspozicija, kalibrirana bilanca bijele boje kamere u boji sačuvano — u UserSet1, postavljeno kao boot set.
- Ne oba (`ValueError`; pozadinski sustav odgovara s 400). Odgovor sadrži `camera_memory`, po kameri `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` ili `{"cleared": bool, "errors": [...]}` — `force_disconnect()` ga vraća; za zadatak dolazi s `stop()`.

Aplikacija ne nudi ove stavke.

**Svaki izlaz vraća kamere.**Vraćanje u prvobitno stanje pokreće se na svakom izlaznom putu: `CameraSession.disconnect()` (kamera aplikacije**X**),**X** na nizu ili bilo kojem članu niza (cijeli niz se oslobađa), zatvaranjem aplikacije (backend ima do 20 s da vrati svaku kameru, niz i kontinuirano snimanje koje je u tijeku; privremena funkcija ili jednokratni poziv koji je još uvijek u tijeku zaustavlja se na svojoj sljedećoj sigurnoj točki, prvo vraća svoje kamere, do 8 s, i odgovara HTTP 503 `SHUTTING_DOWN`; `connect_array()` koji se još uvijek izvršava zaustavlja se prije svoje sljedeće kamere i vraća kamere koje je otvorio, do 10 s, i podiže `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); i onaj koji stigne tijekom zatvaranja ne otvara ništa: HTTP 503 `SHUTTING_DOWN`), i `chloros-cli` naredba koja se prekida ili čiji se terminal zatvori (njezin se posao zaustavlja i njezine se kamere vraćaju prve). Backend koji se sruši ili se nasilno prekine ne može ništa vratiti; isključenje i ponovno uključivanje vraća svaku kameru u njezinu memoriju.**Projekt postavlja svoje kamere sam.** Kada se niz poveže putem projekta — otvoreni projekt aplikacije ili `ChlorosProject`, koja se sama imenuje u pozadinskom sustavu (`project_path`) — svaki član dobiva postavke koje je projekt spremio za njega: ekspoziciju, pojačanje ili cilj AE-a zadržan na njemu (zadržan kao vaš, tako da ga automatska ekspozicija ne dira) i njegov način balansa bijele boje (Chlorosovu korekciju; vlastita bilanca bijele boje kamere ostaje isključena), što god memorija kamere sadrži.

### Načini snimanja i snimači

Površina niza odražava snimni panel GUI-ja: načini pojedinačnog / kontinuiranog / intervalnog / najbržeg zatvarača, plus dva snimača (uživo-kompozitni video i raw serijski snimak → offline prerada).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je kontinuirana/intervalna petlja SDKa. Budući da ne postoji `Ctrl+C` kojim bi se iz nje izašlo iz skripte,**morate** proslijediti `count` i/ili `duration_s` (zaustavlja se kada se dođe do bilo kojeg od njih). `interval_s` mjeri se od početka svakog prolaza (usklađeno s GUI-jem). Kontinuirani prolazi (`interval_s=0`) povezuju `group_seq` → `after_seq`, pa su to uzastopne sinkronizirane grupe u nizus rate. Preostali kwargs prolaze izravno do `capture()`.
- **`continuous`**je vlastiti kontinuirani snimak pozadinskog sustava — Kontinuirani način rada GUI-ja.**Koristite ga umjesto `capture_repeated()` kada želite da svaki okidač bude snimljen na disk pri visokoj brzini sličica:** `capture_repeated()` šalje jedan HTTP-zahtjev po grupi iz vaše skripte, dok `continuous()` drži dva snimanja u tijeku unutar pozadinskog sustava bez povratnog putovanja između grupa. Mjereno na ROI 512 na nizu od 9 kamera: 73–77 snimanja/s, u usporedbi sa nizom koji objavljuje oko 78. Zadržite `capture_repeated()` za intervalno snimanje (`interval_s`) i za pasove koje pregledavate jedan po jedan. Na sustavu Windows isključite backend iz programa Microsoft Defender za visoke stope (pogledajte [Korak 1](#step-1--install-the-chloros-platform-package)).
- **`record`** je *razine nadzora*: zabilježava renderirani kompozit kombiniranog indeksa uživo. Kada je otvoren kombinirani prikaz GUI-ja, snima taj prikaz onako kako je prikazan. Ako nijedan nije otvoren, pozadinski sustav samostalno iscrtava prikaz: `expr` ako ga proslijedite, inače NDVI-a niza (crveni i NIR pojasevi najbliži 660 nm i 850 nm, kako ih odabere prikaz uživo), nakon što je niz poravnan. Nazovite pojaseve u `expr` onako kako ih naziva niz: `<band>_<filter>` za M3C kameru (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` za M3M (`Mono_850`); kratko ime boje (`NIR`, `Red`) radi samo kada ga ima jedna traka niza. GUI prikaz otvoren tijekom snimanja snima se dok god ostane otvoren. Snimanje koje ne zapisa nijedan kadar završava pogreškom s `error` `"no-frames: …"` (niz se možda još uvijek poravnava ili podešava ekspoziciju), i `stop()` se podiže (vidi [`RecorderHandle`](#recorderhandle)). Jedan kompozitni snimač po nizu (podigne se ako je jedan već u tijeku).
- **`burst` → `build_video`** je *za analizu*: `burst` zapisuje sirove kadrove + manifest po kadru + jedan `.daq` po različitoj DLS-ovoj čitanju pod `<output>/bursts/<base>/` brzinom punog ciklusa hvatanja (bez lanca, bez exiftool-a, bez live view-a). `build_video` usklađuje svaki kadar vremenski s najbližim `.daq` i ponovno pokreće pipeline za radijancu/lanac reflektancije/indeksa. `products` je popis `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (po zadanom: kombinirani indeks). `burst().stop()` također automatski pokreće kombinirani postupak po najboljem naporu-indeksne izgradnje, vraćeno kao `build_job` u rezultatu zaustavljanja.

##### Rezultat `build_video`

| Ključ | Značenje |
| --- | --- |
| `success` | `True` samo kada je svaki zatraženi proizvod izgrađen prema zahtjevu: najmanje jedan izlaz, ništa u `skipped`, i nije `degraded`. |
| `outputs` | Putanje zapisa videozapisa (i GIF-ova). |
| `errors` | Problemi, kao tekst. |
| `skipped` | Jedan `{"serial", "level", "reason"}` po zahtijevanom proizvodu koji nije proizveo **ni** video — kamere (nema kalibracijskog paketa za tu kameru, nema DLS očitanja za reflektanciju, lanac obrade ne uspijeva) ili kombiniranog indeksa, koji sadrži `"kind": "combined"` i `serial` `None` (nema para crvenog/NIR pojasa među kamerama, nema skupine s radijansom za oba pojasa). Svaki unos generira `success` `False`; ostali videozapisi se i dalje pišu i navode. |
| `substitutions` | Kamere su renderirane na drugačijoj razini od tražene i zašto — RGB kamera koja je zatražila radijancu/reflektancu dobiva svoj video za prikaz, s uravnoteženom bijelom bojom kada je dostupan paket kalibracije kamere. Nije kvar. |
| `notes` | Informativni tekst, ne problemi — npr. kamera bez izraza indeksa po kameri nema video indeksa, ili video prikaza RGB kamere nije uravnotežen po bijeloj (nema kalibracijskog paketa ili paket nema tvorničku uravnoteženost bijele), uz navođenje naziva kamere. Nije kvar. |
| `degraded` | `True` kada je proizvod napisan, ali nije onaj koji je zatražen: offline kombinirani indeks je ugrađena normalizirana razlika crvene/NIR, tako da konfigurirani izraz indeksa nije evaluiran. Pokreće `success` `False`. |
| `combined` | Sažetak kombiniranog indeksa: upotrijebljene pojaseve, raspon LUT-a, izgrađeni kadrovi. |
| `clamp_masks` | Staze [maski za zaključavanje miješanja](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) po-TIFF-ovi kalibrirani za svaki kadar (`save_tiffs=True`), u `clamp_masks/<level>/` pokraj mape `<level>/` s TIFF-ovima, po jedan za svaki TIFF u kojem je piksel zaključan. Video ne sadrži nijedan. |
| `clamp_fractions` | Što video prikazuje umjesto maske: jedan `{"kind", "serial", "level", "frames"}` po video proizvodu (`kind` `"combined"` za kamere članice kombiniranog indeksa), `frames` popis `[frame, R, G, B]` — udio piksela tog kadra koje je unmix zaključao u pojas koji njegov R / G / B kanal sadrži (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Vraćeno od strane `ArraySession.record()` i `ArraySession.burst()`. Koristite ga kao upravljača konteksta za automatsko zaustavljanje pri izlasku iz opsega ili ga pokrenite ručno.

| Član | Opis |
| --- | --- |
| `job_id` | ID pozadinskog posla (str). |
| `kind` | `"composite"` (iz `record`) ili `"raw"` (iz `burst`). |
| `start_stats` | dict koji vraća poziv `start`. |
| `result` | `None` tijekom izvođenja; dict konačnog rezultata zaustavljanja nakon zaustavljanja. |
| `stats(timeout=10.0)` | Statistike posla uživo (napisani kadrovi, ostvareni fps, proteklo). |
| `stop(timeout=60.0)` | Zaustavi snimač; vraća i kešira konačni rezultat (`.result` ostaje `None` ako je sam zahtjev za zaustavljanje propao). Za snimanje `record()` koje je neuspjelo — ono koje nije uključivalo nijedan snimljeni kadar (`error` `"no-frames: …"`) — pohranjuje rezultat na `.result`, a zatim podiže `ChlorosConnectError`. Idempotentan (drugi poziv vraća pohranjenu vrijednost, bez podizanja). Napuštanje bloka `with` zaustavlja snimač bez podizanja, stoga provjerite `rec.result.get("error")` nakon toga. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Vraća `ArraySession.continuous()`. Koristite ga kao upravitelja konteksta — blok `with` zaustavlja snimanje ako ono još uvijek traje — ili ga upravljajte pomoću `status()` / `wait()` / `stop()`. U svakom trenutku se provodi jedno kontinuirano snimanje po nizu.

| Član | Opis |
| --- | --- |
| `start_status` | Statusni dict koji se vraća pozivom za početak. |
| `result` | `None` tijekom izvođenja; konačni status dict nakon završetka snimanja (postavlja `stop()`, ili od strane `status()` / `wait()` koji je zabilježio kraj). |
| `status(timeout=10.0)` | Dikt statusa uživo (polja u nastavku). |
| `wait(timeout=None, poll_s=0.5)` | Blokirajte dok se snimanje ne završi samo od sebe — dosegnuta `max_captures` ili `duration_s`, niz je nestao, disk je pun, dogodila se pogreška — i vratite konačno stanje. S `timeout` vratite najnoviji status nakon toliko sekundi čak i ako se još uvijek izvršava: provjerite `running`. |
| `stop(timeout=30.0)` | Zaustavite snimanje i vratite njegov konačni status. Idempotentno (drugi poziv vraća spremljeni rezultat). |

Polja statusa:

| Polje | Značenje |
| --- | --- |
| `running` | Istinito dok snimanje traje. |
| `captures` | Sinhronizirane grupe spremljene. |
| `files` | Datoteke zapisane. |
| `failed_files` | Datoteke koje nije bilo moguće zapisati (kao u [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Pokušaji snimanja koji su u cjelini propali, uključujući one koje je snimanje ponovno pokušalo (HTTP-503 dok niz nije na vrijeme imao novu grupu). |
| `skipped_groups` | Grupe koje je niz objavio, a koje nisu spremljene (snimanje je zaostajalo). |
| `captures_per_s` | Prosječna brzina spremanja od početka. |
| `elapsed_s` / `started_at` | Sekunde rada do sada; vrijeme početka (sekunde Unix epohe). |
| `last_group_seq` | Najnovija spremljena sinkronizirana grupa. |
| `last_error` | Najnovija pogreška, ili `None`. |
| `stop_reason` | Kada završi: `count` (`max_captures` dosegnuto), `duration` (`duration_s` je dosegnuta), `stopped` (pozvan je `stop()`), `array_gone` (skup je bio isključen), `error` (20 neuspjeha zaredom ili zahtjev koji je backend odbio), `disk_full` (datoteka za koju disk nije imao prostora). `None` tijekom izvođenja. |
| `max_captures` / `duration_s` | Granice s kojima je pokrenut (`0` = none). |
| `defender_cores` | jezgre CPU-a koje je Microsoft Defender koristio tijekom posljednjeg prozora od 10 s; `None` dok se jedna ne izmjeri, i uvijek `None` na hostu bez Defendera. |
| `defender_warned` | Istinito kad snimanje zabilježi da ga usporava Defender (na 1 ili više jezgri). Isključite pozadinski sustav iz skeniranja Defendera — pogledajte [Korak 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Live Frames u vlastitoj cjevovodnoj liniji

Da biste usmjerili sinkronizirane sličice u vlastiti kod — strojnog učenja, prilagođeni preglednik, vlastiti snimač — upotrijebite `ArraySession.frame_stream()`. Svaki tick daje `GrabResult`: `{serial: numpy array}` za svaku kameru, **sve s jednog okidača**, s id-om tog okidača na `.group_id`. To je ista implementacija kao `ArrayHandle.frame_stream()` projektnog API-a, stoga se sve pod [Jedan okidač po čitanju](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#poravnavanje-tokova--poravnanje) i [`bit_depth`](#bitna-dubina--trgovanje-preciznošću-za-propusnost) primjenjuju se na oba.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Svaki isporučeni skup je novi okidač.** `frames.group_id` se nikada ne ponavlja unutar toka; kada zatražite više `fps` nego što niz radi, tok čeka sljedeći okidač, pa radi na sporijoj od te dvije brzine.
- **Izostavljene kamere se imenuju, a ne popunjavaju.** Kamera kojoj se u tom okidaču nije moglo poslužiti nedostaje u dict-u i navedena je u `frames.errors`.
- **Završava glasno.** `FrameFetchError` pri odbijanju sljedećeg tiketa ponovit će se, nakon 60 s bez slike ili nakon 60 s bez novog okidača.**Što isporučuje.**`frame_stream()` prenosi cijele numpy frameove preko lokalne HTTP veze i doseže gotovo**170 MB/s**. To je dovoljno za obradu sirovih kadrova u stvarnom vremenu, ili biniranog niza ili niza ROI, ali ne i za svaki okidač pri punoj rezoluciji — za zadržavanje**svakog** okidača, spremite na disk pomoću [`continuous()`](#continuoushandle). Mjereno na nizu od četiri kamere (FOCN, FRGN, FRGB, F450) s komprimiranim transportom; svaki tick bio je kompletan set od jednog okidača:

| | Cijeli senzor (niz 14,2–14,4 fps) | biniranje 2 × 2 (niz ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 setova/s | 22,2 setova/s |
| `frame_stream(processing="debayered")` | 2,3 seta/s | 9,3 seta/s |
| `frame_stream(processing="display")` | 0,8 seta/s | 3,0 seta/s |
| MJPEG po kameri, 4 streamova u 720p | 13,5 fps svaki; svaki kadar usklađen s okidačem | ~25 fps svaki; ~45 % okidača prisutno na sva 4 |

Koristite `processing="raw"` za propusnost i demosaic na vašoj strani; razina `debayered` također uklanja dark offset kalibracijskog paketa, što demosaicing `raw` ne čini.

#### MJPEG streamovi po kameri (bez RTSP-a)

**RTSP nije podržan.** Sučelje za streaming je MJPEG preko HTTP-a (`multipart/x-mixed-replace`, jedan JPEG po dijelu, svaki dio s `Content-Length`), jedna URL po kameri:

| URL | Što poslužuje |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bitni pregled, do ~30 fps. Za član nizom, `max_h` (zadano 720, 64–2160) i `jpeg_q` (zadano 80) postavljaju veličinu i kvalitetu. |
| `GET /api/camera/<serial>/stream-annotated` | Isti pretpregled s GUI nadogradama (crno-bijela šara, mreža, križni cilj, histogram, peaking, mjerač točke). `ChlorosLocal.camera_stream()` ga čita za vas. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bitna reflektancija po pojasu — vidi [stream uživo reflektancije](#live-reflectance-stream-mjpeg). |

- **Naslovi okidača.** Za član niz, svaki dio sva tri toka sadrži `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` i `X-Chloros-Timestamp-Ns`. Dijelovi iz različitih kamera s istim `X-Chloros-Group-Id` predstavljaju isti okidač: prikupljaju dijelove i grupiraju se prema tom zaglavlju kako bi se ponovno sastavio skup. Samostalna kamera nema grupu: njezini dijelovi za pregled ne nose nijedno od tih zaglavlja (njegovi dijelovi za odražavanje nose prazan `X-Chloros-Group-Id`).
- **Pred pregledni stream ispušta sličice pod opterećenjem.** Servira najnoviju sličicu i preskače sve koje nije uspio poslati na vrijeme, stoga pri visokim brzinama niza ne svaki okidač dospijeva do stream-a svake kamere (vidi gornju tablicu)Za svaki okidač upotrijebite `frame_stream()` ili `continuous()`.
- **`/api/camera/stream/<serial>` se zaustavlja dok je zajednički stream niza otvoren** (`/api/camera/array/stream/<array_id>` — prikaz niza u GUI-ju ga otvara). Svaki član niza zatim šalje jedan rezervni kadar &quot;Pauzirano (niz niza je aktivan)&quot;, bez zaglavlja `X-Chloros-*`, i ništa više dok se zajednički stream ne zatvori; zatim se samostalno nastavlja. `stream-annotated` i `stream-reflectance` se ne zaustavljaju. Zatvorite prikaz niza GUI-ja kada skripta čita pretpregledne streamove po kameri.
- **Za brojeve, a ne slike, upotrijebite `frame_stream()`.**JPEG je kompresijski format s gubicima i 8-bitni; `frame_stream()` vraća iste rezultate bez kodeka u putanji.

### Intervalno snimanje za preglede dronovima

Za pregledni let potrebna je jedna sinkronizirana grupa slika svakih nekoliko metara putovanja, a ne svaki kadar koji niz može proizvesti. Dvije postavke to kontroliraju:**`target_fps`**postavlja koliko često kamere snimaju kadar, a**`capture_repeated(interval_s=…)`** postavlja koliko često se kadar sprema.**Kamere nastavljaju s prijenosom između spremanja.* Svaki interval sprema najnoviju sinkroniziranu grupu; sličice između se ne spremaju. Automatska ekspozicija se prilagođava na svakoj sličici, tako da je ekspozicija već stabilizirana kada dođe do spremanja. Ne postoji način rada u kojem kamere prestaju snimati između fotografija i budi se za svaku pojedinačno: svaka bi fotografija tada morala čekati da stream započne i da se ekspozicija stabilizira.

**Prikačite brzinu sličica na ono što interval zahtijeva.**Ako je ostavljeno na automatsko, niz radi na najvišoj brzini koju host i mreža mogu podnijeti. Smanjenje te brzine smanjuje zagrijavanje kamere, opterećenje mreže i CPU-a hosta, a da se ne mijenja ono što se sprema. Odaberite brzinu koja daje**2–4 sličice po intervalu**:

- Svako spremanje čeka sljedeći kadar nakon što se zatraži, pa grupa spremljenih kadrova dolazi s najviše jednim vremenskim razdobljem zakašnjenja u odnosu na svoje zakazano vrijeme. Viša brzina održava taj razmak malim.
- Automatska ekspozicija prilagođava se jednom po kadru, pa vrlo niska brzina usporava brzinu kojom ekspozicija prati prolazni oblak.

**Izračunajte interval iz plana leta:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Na primjer, otisak od 60 m pri 80 % preklapanja naprijed iznosi razmak od 12 m; pri 10 m/s to je `interval_s=1.2`, a zakvačeni `target_fps=3` daje između 3 i 4 sličice po intervalu. Snimanje je vremenski bazirano: SDK nema način aktivacije po udaljenosti ili GPS-u, stoga održavajte stalnu brzinu na tlu na linijama snimanja.

**Spremite sirovi podatak na zrakoplovu i kalibrirajte kasnije.** `interval_s` mjeri se od početka svakog spremanja, pa ako spremanje traje dulje od intervala, sljedeće se spremanje pokreće kasnije i razmak se pomiče. Snimanje sirovih podataka uz očitanje senzora svjetla (`processing="raw"`; očitanje se po zadanome spremuje) održava svako snimanje kratkim i omogućuje da se svaki kadar naknadno pretvori u reflektanciju.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fiksira brzinu od početka;**`arr.set_target_fps(fps)`**je mijenja usred leta bez ponovnog povezivanja, i**`arr.set_target_fps(None)`** vraća ga u automatski način rada. Brzina veća od one koju host i mreža mogu prenijeti smanjuje se na tu gornju granicu — provjerite `clamped` u rezultatu — a najniža brzina je 0,1 fps.
- **Ne koristite `burst()` za prorjeđivanje sličica.** On zapisuje svaku sličicu koju niz proizvede; slijedi fiksirani `target_fps`, stoga je pravi alat samo kada su potrebne sve sličice.
- **`target_fps` ne popravlja oštećene sličice.** Smanjenje te vrijednosti smanjuje broj proizvedenih sličica, a ne brzinu kojom se svaka od njih šalje; za GVSP-sličice koje su oštećene umjesto toga smanjite `wire_ceiling_mbps` (vidi [Zdravstveno stanje niza](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Priključivanje na već povezani niz — `attach_array`

Ako je niz već pokrenut (GUI ga je otvorio ili je prethodna sesija SDK pozvala `connect_array`), upotrijebite `attach_array` kako biste dobili rukohvat za njega umjesto ponovnog povezivanja. `connect_array` uvijek vraća pogrešku s &quot;Kamera  <sn>je već u nizu <id>&quot; u toj situaciji, jer slanje `/array/connect` za član-u-bazi podataka nije idempotentno; `attach_array` čita `/api/camera/array/list` i podudara se prema array_id ili serijskim brojevima.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Šablona: Skripte SDKa koje rade zajedno s desktop GUI-jem trebale bi prvo pokušati `attach_array` i prebaciti se na `connect_array` ako još nema nijednog niza u skupu.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Priloženi niz se posuđuje, a ne posjeduje.** Rukohvat iz `attach_array` nikada ne ruši niz: ni izlaz iz bloka `with` ni eksplicitni `disconnect()` ga ne oslobađa (eksplicitni poziv ispisuje jedan redak na stderr koji kaže da ostavio je niz u pogonu). Isto pravilo kao kod `CameraSession` / `DAQSensorSession`. Stoga je istovremeno korištenje s GUI-jem sigurno, s `with` ili bez njega:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Kada ga doista želite završiti — primjerice, da biste se ponovno povezali s drugim `wire_ceiling_mbps`-om — pozovite `arr.force_disconnect()`. Ponovna veza nakon običnog `disconnect()`-a na priključenoj ručki ne uspijeva s &quot;Kamera je<sn> već u nizu<id>&quot;, jer ništa nije oslobođeno.

### Pomoćnik za analizu mreže

Korisno prije otvaranja niza — predviđa hoće li vaše predložene postavke biti prikladne:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nikada ne pokreće pozadinski proces: ako pozadinski proces ne odgovori, javlja se `ChlorosConnectError`, dok se odbijeni zahtjev vraća kao rezultat. Neispravan `packed` javlja `ValueError`.

`status` je jedna od `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (u suprotnom `error` — uključujući odbijeni zahtjev: nalog nije prijavljen, `error_code: AUTH_REQUIRED`, ili plan ispod donje granice SDKa, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` znači da zatražena rezolucija odgovara RX prstenu samo pri ograničenoj brzini okidača — zadržite rezoluciju i proslijedite `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (vidi [Primjer 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` pojavljuje se samo kada zatraženi kadar ne može biti isporučen pri bilo kojoj brzini sličica; connect nikada sam ne smanjuje sličice, stoga sami primijenite `recommended`.

**Kako čitati projekciju** (isti model kao i na GUI panelu za postavke niza):

- **Burst (`frame_bytes_total`) sabira se po kameri prema stvarnom pikselnom formatu svake kamere.**Mono**M3M**kamere streamaju Mono12 (2 B/px), ili s `pixel_format` mono format svoje dubine (`"BayerRG8"` je `Mono8`, 1 B/px), pa je s četiri kamere u punoj rezoluciji**~25 MB**, a s tri mono kamere ~12,6 MB, a ne ~12,6 MB koliko pretpostavka o svim 8-bitnim kamerama daje. Backend određuje format svake kamere prema njezinu modelu.
- **Propusnost (`burst_fits_nic_ring`) je svjesna pražnjenja**, ne cijeli burst-naspram-ringa: sim-emit se uklapa kada host isprazni RX ring brže nego što ga kamere pune. 10G host + 1 GbE kamere**prihvaćaju** punu rezoluciju čak i kada burst premašuje ring; a 1 GbE host blokira (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` je konzervativni gornji prag za serijsko preuzimanje** — `max(readout+emit, N×emit)` s emisijom po kameri ograničenom na 1 GbE vezu kamere, neovisno o ekspoziciji. Npr. ~2,8 fps za niz od 4 kamere pune rezolucije od 12 bita (odgovara izmjerenim ~2,7–3,0 fps tijekom rada). Cijeli model: [CLIReference → Array fps &amp; burst model](cli-reference.md#array-fps--burst-model).
- **Nakon kontinuiranog snimanja, projekcija prikazuje postignutu brzinu.** Snimanje od 30 s ili više pri automatskoj brzini bilježi brzinu na kojoj se stabilizirala za tu veličinu niza na ovom računalu; kratki pokušaji upravljača brzinom za bržu brzinu izostavljeni su. Od tada `array-connect`&#x27;redak, analiza `connect_array()` i panel Postavke niza navode tu brzinu kada je ispod one modela, s onim što ju je ograničilo u `cycle_limiter`: `"clock"` (niz je radio brzinom na kojoj se stabilizirao, a petlja hvatanja je čekala na svoj sat okidača), `"loop"` (petlji hvatanja nije ostalo vremena), `"capture"` (prikazivanje je zaostajalo i preskakivalo grupe) ili `"measured"` (ništa ne navodi koje). Brojka modela nalazi se u `wire_fps_max`. Niz i dalje raste do onoga što kabel može prenijeti.
- **Prekomjerno pretplaćivanje (`oversubscribed: true`) znači da kabel ne može prenijeti niz pri bilo kojem tempu kojeg kamere dosegnu** — polja fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) čita 0, a binning to ne može popraviti (smanjuje bajtove po kadru, a ne tempirane bajtove po sekundi). Samo spora veza to ne uzrokuje: umjesto toga pada brzina kadrova, a ako kamere ne mogu raditi dovoljno sporo, sloj postaje `slip-emit-and-capture` (serijski). Rješenja su manje kamere, jumbo okviri ili brži NIC; `max_cams_collision_safe` izvještava o gornjoj granici vaše mreže i kamera. Odgovor također sadrži `aggregate_demand_bps`, `collision_safe_ceiling_bps` i `per_cam_floor_bps`. Pogledajte [Prekomjerno pretplaćivanje](#over-subscription-the-per-cam-floor).

### Otkrivanje i popisivanje

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Svaka prima samo ključne riječi `backend_url=` i `timeout=30.0` te vraća popis dictova. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) također poziva `auto_start_backend=True`; pozivi `list_*` nikada ne pokreću pozadinski proces.

---

## Smart-AE / Smart-Capture

LATTICE nizovi izvode kontinuirani AE u pozadini čim se povežu, ali svježe usmjerena scena treba trenutak da se uskladi. **Smart-capture** je ugrađena praktična značajka: ona provjerava ekspoziciju svake kamere, čeka dok niz ne postane stabilan unutar prozora, a zatim pokreće snimanje. To je ekvivalent GUI-ja: gumb za &quot;pametno&quot; snimanje u desktop aplikaciji poziva istu krajnju točku pozadinske usluge.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Kada koristite `ChlorosProject` (sljedeći odjeljak), dobivate više kontrola:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Pravilo za pametno izlaganje (smart-AE) prema zadanim postavkama je konzervativno. Pooštrite `exposure_tolerance_pct` za zahtjevne radiometrijske radove; ublažite za brzo mijenjajuće se scene gdje vam je dovoljno da je &quot;otprilike točno&quot;.

---

## DAQ sesije senzora

Persistentni pozadinski bazen za spektralne senzore (DAQ-U putem USB-a, DAQ-M putem BLE-a, DAQ-E putem Ethernet-a). Oglasi površinu kamere: pametno otkrivanje, ponovna upotreba bazena, idempotentno povezivanje.

### Pametno otkrivanje (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Redoslijed prioriteta: Ethernet → BLE → USB. Navedite bilo koji eksplicitni trag za pričvršćivanje prijenosa.

### Pričvršćeni prijenos

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` imenuje jedan senzor i samo se taj senzor ikada otvara. On se uspoređuje sa senzorima koji su već otvoreni, zatim s hostnameom ili IP-om DAQ-E-a (DAQ-E-u koji je mrežni skener vidio samo po IP-u traži se, samo za čitanje, ime, MAC i serijski broj, tako da ga bilo koje od tih pronalazi i ono), BLE MAC adrese DAQ-M-a i ID koji DAQ-U ili DAQ-M prijavi nakon otvaranja (sve ostale otvarane za provjeru zatvaraju se ponovno). Ako se ne pronađe, `connect_daq_sensor` pokreće `ChlorosConnectError` koji navodi što je pronađeno; uz adresu (`port=` / `mac=` / `eth_host=`) koji sadrži drugi senzor, također se podiže. Nikada se ne vraća na bilo koji vidljivi senzor — za to služi poziv s zastavicom nula.</id></sn></id></sn>

### `connect_daq_sensor()` Potpis

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Senzor koji je već u bazenu vraća se onakav kakav jest (posuđen): `integration_time`, `frame_avg` i `enable_ae` se na njega ne primjenjuju. Prvo ga odspojite kako biste ga otvorili sa svojim postavkama. `cap_id` je iznimka: na posuđenom senzoru postavlja se pomoću `set_cap()`. Ako se izostavi, vrijedi zadana postavka pozadinskog sustava (pogledajte napomenu ispod tablice metoda).

### Metode `DAQSensorSession`

| Metoda | Opis |
| --- | --- |
| `status(timeout=10.0)` | Sažetak ulaska u bazen (stanje streaminga/snimanja, raspon valnih duljina, kalibracijska sha, vrijeme integracije, frame_avg, stanje AE). Za DAQ-E čiji je stream utihnuo: `reconnect` (gdje automatsko ponovno povezivanje pozadinskog sustava stoji: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` dok emitira), `stream_lost_s`, `reconnects` i `last_reconnect`, te `record_gaps` za snimanje u tijeku; `rx_stream_fault` označava stanje ponovnog povezivanja dok ne stigne prvi spektar. |
| `latest(*, n=1, timeout=10.0)` | Vraćanje do N najnovijih spektralnih okvira (svaki `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Nastavi / pauziraj streaming (rukohrvat ostaje otvoren). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Pokrenite snimanje .daq datoteke. Vraća putanju datoteke. Odbija se za DAQ-U/M bez AWS kalibracijskog paketa (DAQ-E je izuzet). |
| `record_stop()` | Zaustavljanje snimanja. Vraća `{path, rows, warning, gaps}`; `warning` je `None` za čisto snimanje, u suprotnom dijagnostiku pozadinskog sustava (na primjer prazna datoteka ili prekid u toku), koja se također ispisuje na stderr. `gaps` navodi svaki dio u kojem stream nije isporučio ništa: `start_utc`, `end_utc` (`None`: još uvijek izgubljen pri zaustavljanju), `duration_s`, `reconnected`, `marked_in_file`. Prazan je za kontinuirano snimanje. |
| `set_cap(cap_id, timeout=15.0)` | Postavi profil korekcije kape koji se primjenjuje na očitanja od sljedećeg nadalje: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, ili `"none"` (ili `None`) za golu senzorsku jedinicu. Vraća senzorsažetak, koji se čita natrag pomoću `cap_id`. DAQ-E-S pohranjuje `as_recorded` što god se zatraži (njegov difuzor je ugrađen). Podiže `ChlorosConnectError` za profil koji radi ne opterećuje. CLI: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | samo za DAQ-E: ponovno uključi PTP za jedinicu čiji je PTP isključen (`status()["rx_ptp_enabled"]` je `False`; postavka se sprema na jedinicu, pa ostaje isključena dok se ponovno ne uključi). Vraća osvježeni status. Poziva `ChlorosConnectError` za DAQ-U / DAQ-M, ili kada jedinica ne potvrdi. Chloros nikada samostalno ne ponovno uključuje jedinicu, jer isključeno može biti namjerno. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Samo za DAQ-E: vlastita postavka jedinice. `firmware` (`running`, `bundled`, `update_available`), `ip` i `ip_mode` (`dhcp` / `linklocal` / `static`: što se trenutno pokreće), `network` (pohranjena postavka: `mode`, plus `ip` / `mask` / `gateway` kada je statički), `network_change_pending` (važi dok jedinica ne primijeni spremljenu postavku, jednu spremljenu s uključenim `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` je `None` kada jedinica verzija nije mogla biti pročitana. Dok se stream jedinice ne prima na ovom računalu, on se čita na novoj vezi i rječnik dodaje `stream_silent` (`True`), `stream_fault` i `reconnect`; jedinica koja ne odgovara na pozive. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E samo: DHCP (`"auto"`) ili statičku adresu. Ponovno pokrenite jedinicu kako biste primijenili promjenu i ponovno je povežite; ova sesija nastavlja raditi i `address` prati jedinicu. `reboot=False` samo je sprema. Vidi [Postavke uređaja DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Samo za DAQ-E: streaming pri uključivanju bez priključenog softvera (pohranjeno na jedinici). Vraća potvrđenu postavku. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Samo za DAQ-E: ponovno pokrenite jedinicu i ponovno je povežite s prethodnim postavkama. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Samo za DAQ-E: flashajte firmware priložen uz ovo izdanje programa Chloros na jedinicu koja radi na starijoj verziji, zatim je ponovno povežite (otprilike minuta; ne isključujte napajanje). Vraća `updated`, `fw_before`, `fw_after`, `bundled` i `warning` kada se jedinica sporo vratila ili se vratila na drugoj verziji; `updated` je `False` za jedinicu koja je već ažurirana. Pokreće `ChlorosConnectError` za jedinicu čija se verzija ne može pročitati. CLI: `pool-update-firmware`. |
| `disconnect()` | Oslobađanje iz bazena. Ne-op za pridružene, a ne vlasničke rukohvate. Na senzoru koji je ova sesija otvorila to je `force_disconnect()`, i pokreće `ChlorosConnectError` kada oslobađanje nije potvrđeno (izlaz iz bloka `with` nikada ne podiže). |
| `force_disconnect(timeout=30.0)` | Oslobađanje senzora bez obzira tko ga je otvorio — na primjer, senzor koji je skripta koja se srušila ostavila u skupu, a do kojeg nova skripta dolazi samo kao do pridruženog ručke čiji `disconnect()` ne podiže nikakvu iznimku. Zatvaranje senzora drugog klijenta dovršava svako snimanje koje on obavlja. Vraća se nakon što pozadinski sustav potvrdi oslobađanje ili kada nema ništa za osloboditi (senzor nije u skupu, ili pozadinski servis ne radi); oslobađanje koje je isteklom vremena ili koje je pozadinski servis odbio podiže `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atributi: `sensor_id`, `model`, `transport`, `address` (slijedi DAQ-E koji je `set_network()` ponovno adresirao), `info` i `backend_url`.

> **Nazovite kapu.** DAQ-U, DAQ-M i DAQ-E isporučuju se bez kondenzatora, a Chloros ne može vidjeti je li on ugrađen: senzor priključen bez `cap_id` dobiva `sunshine_cosine`, pretpostavka koju njegova snimanja navodno potvrđuju (`cap_id_source=auto_default`). Goli DAQ-U pod tom korekcijom čita oko 30x više, stoga prođite `cap_id="none"` za goli senzor (ili `set_cap("none")`), i profil kapa koji ste inače montirali.

### DAQ-E-S — fiksni difuzor i senzor nagiba

**DAQ-E-S** je DAQ-E s dvije razlike koje su važne za softver. Sve ostalo — otkrivanje, povezivanje, PTP, snimanje, postavke uređaja u nastavku — radi točno kao i kod DAQ-E.**Njegov difuzor je trajan, stoga nije potrebna korekcija kapom.** Difuzor u stilu Sunčane svjetiljke ugrađen je i bio je na jedinici kada je mjeren tvornički kalibracijski set, pa ta kalibracija već mjeri stvarnu zračenost prema dolje. Chloros postavlja svoj poklopac na `as_recorded` (bez profila) bez obzira na zahtjev: poklopac nazvan po njemu (`sunshine_cosine`, `fov_*`, `none`) se zanemaruje uz poruku, a njegovi zapisi pohranjuju `cap_id_source=model`, pa se za njega nikada ne aktivira upozorenje o pretpostavljenom poklopcu. Ni ručno nemojte primjenjivati kapu na njegove podatke: kapica `sunshine_cosine` na vrhu bi očitala oko 11 puta previsoku vrijednost.

**Sadrži troosni akcelerometar.** Kosin odgovor difuzora sunčeve svjetlosti valjan je samo dok je ravan, stoga nagnuti senzor prijavljuje pogrešnu radijaciju — a reflektancija nasljeđuje pogrešku — bez drugih simptoma. Svako očitanje DAQ-E-S-a stoga nosi svoj položaj:

| Polje | Značenje |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Kutovi u stupnjevima. `None` kada firmware označi kut kao nedefiniran (slobodan pad, mrtva os). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Ubrzanje u mili-g. `mag_mg` daleko od ~1000 znači da se jedinica kretala i da kutovi za tu uzorak nemaju smisla. |
| `cal_applied` | Postavljeno: osi i kutovi su u referentnom sustavu difuzora (primijenjena je kalibracija montaže jedinice). Očisti: sirovi referentni sustav ploče, koji može pokazivati do ~2,3° nagiba koji nije stvaran. |
| `sample_age_ms` | Starost uzorka akcelerometra kada je spektrum poslan. |
| `temp_raw` | Nekalibrirani trend temperature čipa — samo razlike, nikada očitanje termometra. |

`DAQSensorSession.latest()` ih vraća pod ključem `imu` svakog okvira (`None` na DAQ-U, DAQ-M ili običnom DAQ-E). `.daq` snimljen s DAQ-E-S pohranjuje iste vrijednosti u svojim stupcima `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) uz svaki spektar, tako da čitatelj može razlikovati oblak od srušenog stativa. Na svim ostalim modelima te su stupci prazni.

Tok stavova izvještava o vlastitim poljima u `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), odvojeno od spektralnih kako bi prekid rada akcelerometra nikada ne bio protumačen kao praznina u snimci. `discover_daq_sensors()` i `model` sesije izvještavaju o `DAQ-E-S`.

### Postavke uređaja DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Statička adresa mora postojati na mreži ovog računala.** `set_network` pokreće `ChlorosConnectError` za onoga čija podmreža ne uključuje ovo računalo&#x27;adresu senzora na mreži senzora: nakon ponovnog pokretanja ništa ne može dosegnuti senzor kako bi ga vratilo u prethodno stanje. Postavite `force=True` samo kada će ova računala biti ponovno adresirana ili usmjerena na novu podmrežu.
- **`set_network`, `reboot` i `update_firmware` ponovno pokreću senzor**, i podižu `ChlorosConnectError` dok on bilježi, i kada se ne vrati unutar `comeback_timeout` sekundi. ID senzora i `daq-e-<id>.local` naziv hosta nikada se ne mijenjaju.
- **Zapis koji je započeo neposredno prije ponovnog pokretanja zatvara se i imenuje.** Kada se senzor vrati, vraćeni dict sadrži `recording_closed` (putanju datoteke) i `warning` koji to potvrđuje. Kada se ne vrati ili prijenos firmvera ne uspije, `ChlorosConnectError` umjesto toga o tome obavještava: njegova poruka završava upozorenjem, i `e.recording_closed` / `e.warning` su postavljeni (oba `None` kada nije bilo zatvorenog snimanja). Ponovno pokrenite snimanje čim se senzor vrati.
- **Samo se sam senzor ponovno povezuje.** Nakon ponovnog pokretanja jedinica mora odgovoriti gdje je bila -- i nakon `set_network("auto")` također pod svojim imenom `.local` -- sa svojim imenom, nazivom hosta i modelom, s mrežnog adaptera (MAC) koji je imao; `address` postaje IP adresa koja je odgovorila. Odgovor pod imenom vrijedi samo za taj MAC, stoga se senzor koji je DHCP premjestio neće ponovno povezati s računala koje pristupa senzoru putem usmjerivača ili s WSL2-ove zadane NAT mreže. (`ChlorosConnectError`; ponovno ga povežite na novu adresu).
- **Ažuriranje firmvera zahtijeva da se senzor ponovno poveže.** `update_firmware()` poziva jedinicu, a jedinica otvara TCP vezu natrag na ovo računalo za sliku, na port koji odabere OS. Backend sluša na adapteru koji doseže jedinicu (na svakom adapteru samo kada ne može odrediti koji) i služi samo toj jedinici: sve druge veze se odbijaju (`[DAQ-FW] … refused a firmware connection from <ip>` u izlazu pozadinskog programa) dok on nastavlja čekati. Vatrozid koji blokira vezu onemogućuje ažuriranje s porukom `device never connected back on TCP <port>`. Instalacijski program sustava Windows dodaje pravilo za to (vidi [Korak 1](#windows-exe)); na drugom vatrozidu dopustite dolazni TCP prema pozadinskom procesu iz podmreže senzora.
- **U sustavu WSL2 upotrijebite ogledalo mrežnog prometa** (`networkingMode=mirrored` u `.wslconfig`): u zadanoj NAT načinu rada svaka statička adresa se odbija (vlastita adresa sustava WSL nije na mreži senzora), stream ne dopire do WSL-a i ažuriranje firmvera se ne može dovršiti.
- GUI održava vlastitu vezu sa senzorom: ponovno pokretanje iz skripte prekida GUI-jevu sesiju s njim. GUI ima iste postavke (kartica Svjetlosni senzori, postavke senzora, Mreža).

### Otkrivanje — pronalaženje adrese za povezivanje

`discover_daq_sensors()` skenira USB / BLE / ETH za senzore koje *možete* otvoriti. To je DAQ pandan za `discover_lattice_cameras()` i jedini je način za dobivanje **BLE MAC adrese DAQ-M-a** — DAQ-E ima host ime, a DAQ-U COM priključak, ali MAC adresa nije ispisana na uređaju niti je navedena u OS-u.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Polje | Opis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM priključak / BLE MAC / naziv hosta — proslijedi na `connect_daq_sensor` kao `port=` / `mac=` / `eth_host=`. |
| `display` | Čitljiv naziv. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, ili `None`. Ethernet senzor je `DAQ-E-S` ili `DAQ-E` kako se prijavljuje kada se zatraži, samo za čitanje, tijekom skeniranja (`DAQ-E` kada ne odgovori na vrijeme). USB serijski priključak čita `DAQ-U` kada je njegov mostni čip jedan od onih koje koristi DAQ-U (FTDI, CH340, CP210x, Prolific) — naznaka iz opisa priključka, nije sonda — i `None` za bilo koji drugi USB uređaj, koji je i dalje naveden kako biste ga mogli predstaviti kao `port=`. Serijski priključak bez USB vendor ID-a i bez takvog mostovnog čipa (UART matične ploče kao što je `/dev/ttyS*` u Linuxu, Bluetooth serijska veza, Intel AMT) ne može biti DAQ-U i nije naveden. |
| `extra` | Detalji po prijevozu (oglašeno ime BLE-a; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Prazne vrijednosti su izostavljene. |

| Parametar | Zadano | Opis |
| --- | --- | --- |
| `transports` | sva tri | Sekvenca (ili CSV niz) koja ograničava skeniranje. Vrijedi navesti kada znate što želite — BLE je sporiji dio. |
| `scan_timeout` | 5 | Prozirni prozor po transportu u sekundama; pozadinski sustav ograničava na 1–20. |
| `timeout` | 60.0 | Gornja granica za HTTP za cijeli poziv (kao i drugdje u SDK-u). |
| `auto_start_backend` | `True` | Pokreni lokalni pozadinski proces ako nijedan ne radi. Nikada se ne pokreće za udaljeni `backend_url`. |

> **Senzori koji su već otvoreni u poolu se ne prikazuju.** Spojeni BLE periferijski uređaj prestaje s oglašavanjem i otvoreni COM priključak se ne može ispitati, stoga otkrivanje prikazuje ono što je *dostupno za povezivanje*. Očekuje se prazan rezultat odmah nakon što nešto povežete — koristite `list_daq_sensors()` za ono što već držite. Transporti čije skeniranje ne može pokrenuti (nije instaliran bleak / zeroconf) preskaču se umjesto da se javlja pogreška, tako da računalo bez Bluetootha i dalje dobiva odgovore za USB i ETH.

### Popis

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Koji DAQ napaja koju kameru — `daq_bindings()`

Refleksija snimanja i `.daq` koje sprema koriste DAQ vezan za tu kameru ili niz, inače koriste najnoviji DAQ koji ima pozadinski sustav. GUI odabir senzora svjetlosti i `chloros-cli daq pool-bind` to postavljaju; kao i:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` vraća `{"cameras": {serial: b}, "arrays": {array_id: b}}`: svaki `b` sadrži `binding` (postavljena vrijednost), `source` (`project`, iz `sensors.json` projekta, ili `runtime`), `known` / `known_as` (bilo da ključ kamere ili niz imenuje nekoga koga Chloros zna, i kako: `connected`, `project` — `cameras.json` otvorenog projekta — ili, za kameru, `seen`: otvorena na ovom računalu prije; `known: False` za ključ koji ne naziva ništa) i `resolved` (pool senzor kojem sada mapira: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Vezanje u vrijeme izvođenja nadmašuje projekts za sesiju; projektna datoteka se ne mijenja. Odspajanje kamere, niza ili DAQ-a poništava svako povezivanje tog hardvera, bez obzira tko ga je postavio (članovi niza zajedno s nizom).

`set_daq_bindings()` povezuje samo kameru koja je povezana, u otvorenom projektu ili prije otvorena na ovom računalu, i niz koji je povezan ili u otvorenom projektu (ili ključ koji je već povezan). Svaki drugi ključ izaziva `ChlorosConnectError` imenujući ga (`400`, `code` `UNKNOWN_BINDING_TARGET`, s ključevima `unknown`), i **ništa** u tom pozivu nije vezano. `None` poništava bilo koji ključ.

### Reflektancija okoliša i objekta — `light_sensor_reflectance()`

Dva DAQ-a spojena u grupu, jedan usmjeren prema nebu (ambiant) i jedan prema cilju (objekt), daju refleksivnost cilja i indekse vegetacije iz svojih najnovijih očitanja: para Ambijent + Objekt u GUI-ju i `chloros-cli daq reflectance` bez ponovnog otvaranja senzora. Oba moraju biti povezana i u prijenosu.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bande.** Reflektancija svake bande je reflektancija objekta za tu bandu-integrirano zračenje nad zračenjem okoline. `daq_band_presets(filter_code=None)` navodi skupove pojaseva: po jedan za svaki LATTICE M3C filtar (`RGN`, `OCN`, `NGB`, vlastiti model pojaseva kamere, tako da su DAQ NDVI i NDVI kamere ista količina) i `GENERIC` širokopojasni. Njegov `suggested` je unaprijed postavljeni odgovarajući `filter_code`, inače filter prve priključene kamere (`suggested_from` navodi koji).
- **Vraća** `bands` (po pojasu `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (izgrađeni-ins), `custom` (svaki s `value`, ili `valid: False` i `error`), `invalid_bands`, `reflectance_type`, `preset` i dva očitanja `timestamps` (`ts_ns`). Pojas koji senzor okoline nije mogao izmjeriti (izvan svog raspona, na razini šuma) je nevažeći, s razlogom, i svaki indeks kojem je to potrebno izostavljen je, nikada se ne izračunava u odnosu na nulu.
- **Prividni ili relativni.** Indeksi ovisni o ljestvici (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) zahtijevaju apsolutni omjer, stoga se izračunavaju samo kada su oba senzora kalibrirana po snazi (svaki ima `calibration_bundle_sha`): `"Apparent Reflectance"`. Inače je rezultat `"Relative Reflectance"`, samo s indeksima neovisnima o skali.
- **Dvije različite mreže valnih duljina** uspoređuju se po njihovom preklapanju: točke senzora okoline unutar dometa objekta, spektar objekta interpoliran na njih.
- **Dva senzora, nikad jedan.** Imenovanje jednog senzora kao oba (isti ID, u bilo kojem pisanju koje bazen prihvaća) izaziva `ChlorosConnectError`: očitanje senzora na samom sebi je 1,0 na svakom pojasu, što ne mjeri ništa. Backend to također odbija (`POST /api/daq/indices` odgovara `400`, `code` `SAME_SENSOR`, kada su `ambient_sensor_id` i `object_sensor_id` koje šalje SDK jedan objedinjeni senzor, npr. njegov ID i njegov port).

### Su-posjedovanje s CLI-om i drugim skriptama

Backendov bazen senzora DAQ dijele svi klijenti CLIa i SDKa na računalu. Ako je senzor već u bazenu — otvoren od strane `chloros-cli daq pool-connect`-a ili neke druge skripte — pozivanje `connect_daq_sensor(port="COM3")`-a iz Pythona vraća posuđeni handle: njegov `disconnect()` je no-op, pa vaša skripta ne oduzima senzor klijentu koji ga je otvorio pri izlasku iz opsega. Isto vrijedi i za `sensor_id=` (ukucano s `:` ili `-`, oboje je prihvatljivo), za DAQ-E imenovan njegovom IP adresom kada je otvoren po nazivu hosta (ili obrnuto), i za pametno otkrivanje (smart-detect) s zastavicom nula kada pozadinski sustav odgovori senzorom koji je već držao — blok `with` oslobađa samo senzor koji je sam otvorio.

GUI aplikacije Chloros **nije** dio ovog skupa: njezin DAQ panel uspostavlja vlastitu vezu sa senzorom, stoga `connect_daq_sensor` nikada ne posuđuje senzor koji je GUI otvorio. Nemojte koristiti isti senzor iz GUI-ja i iz skripte istovremeno. U tom slučaju, ako GUI drži DAQ-U ili DAQ-M, pokušaj otvaranja obično jednostavno ne uspije, ali DAQ-E prihvaća drugu sesiju: connect skripte mijenja postavke akvizicije jedinice ispod GUI-ja, a njegov `disconnect()` zaustavlja stream jedinice, što prekida pregled uživo GUI-ja i svako snimanje koje se odvija. Prvo zatvorite senzor u GUI-ju.

### PTP Grandmaster — `time_sync_status()`

Backend pokreće PTP grandmaster od kojeg kamere LATTICE i jedinice DAQ-E preuzimaju svoje vremenske oznake. Pokreće se na zahtjev: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` i `connect_all()` ga pokreću ako nije u pogonu — uključujući i povezivanje koje zatim ne uspije, kao što je `connect_all()` koji pokušava DAQ-E jedinicu koja ne postoji — kao i `test_array_sync()` i svaki poziv koji otvara kamere za jednu naredbu (`capture_cameras_transient()`, `start_multi_camera_stream()`, pozivi poravnanja, `capture_live_index()`).

- **Gdje se objavljuje.** Na žičanim mrežnim adapterima koji su okrenuti prema kamerama (uključujući `169.254.x.x` link-lokalnu adresu), inače na adapteru zadane rute ako je taj žičan. Nikada na Wi-Fi ili virtualnom adapteru (Tailscale, WSL): iza njega nema kamere ni DAQ-E. Ako takav adapter nije pokrenut, program se pokreće bez objavljivanja na ničemu (`health` `NO_SOCKETS`) i veže se za adapter koji se pokrene pri sljedećem povezivanju kamere ili DAQ-a.
- **Nakon prekida veze.** Kada adapter kamere izgubi vezu i ponovno je uspostavi (npr. zbog izvlačenja kabela, zamjene prekidača ili ponovnog uvrštavanja USB adaptera), grandmaster samostalno ponovno izgrađuje svoje sokete, bez ponovnog pokretanja, i ponovno provjerava svaki priključeni DAQ-E, ponovno aktivirajući onaj čija je veza ostala prekinuta. Također se ponovno gradi kada sve kamere i DAQ-E kojima je služio utihnu na 15 s.
- **Dostupa li žica?** Pročitajte `time_sync_status()` (`GET /api/time-sync/status`). `state` samo kaže da se grandmaster niti izvršava; `health` kaže jesu li njegovi paketi odlazni: `OK` (nije zabilježena greška), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (neki adapteri otkazuju) ili `PEERS_SILENT` (kamere i DAQ-ovi kojima je služio prestali su odgovarati), sa rečenicom u `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` broje samo poruke koje su poslane; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` i `last_delay_req_age_s` opisuju one koji nisu, a `rebinds` / `last_rebind` ponovne izgradnje. `sync_sends_slow` broji sinkronizacije čije je slanje trajalo dulje od 200 µs, a `sync_send_max_us` je najduža; vrijeme početka svakog Follow_Up-a računa se najviše 200 µs za njegovo slanje. Grandmaster se pokreće u vlastitom procesu (`own_process`, `process_pid`), s visokim prioritetom na sustavu Windows (`process_priority`), tako da zauzeti pozadinski proces ne može odgoditi vrijeme koje bilježi i pomaknuti sat svakog kamere odjednom; ako se taj proces ne može pokrenuti, on se izvršava unutar pozadinskog procesa kao i prije (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funkcija | Što radi |
| --- | --- |
| `time_sync_status()` | Stanje, zdravlje, identitet sata, prioriteti i brojači poruka glavnog majstora (gore). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP roboti koje čuje (kamere, DAQ-ovi): `peers`, svaki `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Svaki `PtpStatus` povezane nizove kamere, `PtpOffsetFromMaster` i `PtpMeanPathDelay` (kamera čiji firmware nema PTP čvorove ima samo serijski). `time-sync cameras`. |
| `restart_time_sync()` | Zaustavite i ponovno pokrenite grandmastera; PTP svakog objedinjenog DAQ-E-a ponovno se provjerava nakon (`daq_ptp_rechecked`). Pokreće `ChlorosConnectError` kada ne uspostavi vezu. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Njegovi PTP prioriteti (0-255, niži pobjeđuje u izboru najboljeg mastera), za mrežu s drugim PTP masterom. Potrebno je da radi. `time-sync set-priority`. |

Svaki uzima `backend_url=` i `timeout=` i pokreće lokalni pozadinski proces koji nije u pokretanju.

---

## Pomoćni programi za kameru

Što `chloros-cli lattice` radi osim povezivanja i snimanja, kao obične funkcije u `chloros_sdk` (modul `chloros_sdk.camera_utils`). Svaka poziva putanju pozadinske usluge koju koristi naredba CLIa, prema pravilima CLIa, tako da se obje ponašaju slično. One uzimaju **serijski broj** kamere, a ne sesiju, i iste `backend_url=` / `timeout=` ključne riječi kao i pozivi pametnog povezivanja; pokreće se backend koji nije u pokretanju, kao što to radi `connect_camera()`. Odbijanje pokreće `ChlorosConnectError` s razlogom backenda.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funkcija | Što radi |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Profil boje zaslona: `raw`, `linear`, `natural`, `enhanced` ili `custom_temp`, i njezinu temperaturu boje (ograničeno na 2000-10000 K; prikazuje se kada je profil `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Zasićenje i kontrast zaslona, 0–200 % svaki (100 = nepromijenjeno); `reset=True` vraća oboje na 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Povlači najsvjetlije, najbjelje piksele prikaza prema dolje: -100..0 (skraćeno prema nuli, zatim zaključano; 0 = isključeno). |
| `camera_white_balance_once(serial, settle_s=None)` | Jednokratna bilanca bijele: kamera nakratko snima, konvergira i vraća `red_ratio` / `blue_ratio` (`ratios_read` pogrešno kada ih ne može izvijestiti). Zadržava balans bijele boje koji je riješio; svaka druga postavka koju je otvoreno napisao vraća se onakvom kakva je pronađena. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (kontinuirano) ili `"off"`; ili fiksni omjeri ravnoteže `red` / `blue`, koji prvo isključuju automatski način rada. Omjer koji kamera odbije nazvan je pod `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Kamera postavlja unaprijed, a primjenom jedne: kamera zadržava postavke koje nazivi unaprijed postavki (odgovor `settings`, kako ih kamera čita nakon toga, npr. brzinu obrade kad se koristi; `None` za polje koje preset ne imenuje), i svaku drugu postavku koju je otvorio vraća onako kako je pronađena. `"default"` vraća kameru na tvorničke postavke (njezine tvorničke postavke s neograničenim automatskim ekspozicijom, ograničenjem propusnosti veze i isključenom korekcijom nedostataka, zadržavajući kalibriranu bilancu bijele boje kamere u boji RGB), a ne spremaju se u njezinu memoriju: odgovor sadrži `factory_defaults: True` i svako polje `settings`. U `camera_presets()`, `default` je `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Modeli filtara i detalji jednog modela (pojasni, brojni uvjet spektralnog odziva). Model filtra kamere čita se iz niza modela; ne postavlja se ručno. |
| `camera_firmware_status(serial=None)` | Svaka kamerafirmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` i `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Instalirajte firmware ove verzije Chloros na jednu kameru: tvorničko resetiranje, instalacija, ponovno pokretanje, ponovno se upisuju osnovne postavke Chloros; kalibracija kamere se zadržava. Nekoliko minuta; kamera mora ostati uključena. `force=True` ponovno flasha trenutnu kameru. |
| `camera_power_probe(serial)` | Napajanje i termalni čvorovi kamere i njezina temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Niskopotrošna pripravnost kamere (ostaje u mirovanju dok se ne probudi: ograničenje propusnosti veze uključeno (`DeviceLinkThroughputLimitMode` `On`) na donjoj granici, način uštede energije uključen gdje ga kamera ima, s temperaturom prije / poslije; buđenje vraća vezu u stanje u kojem ju je spavanje zateklo (ograničenje, njegov način rada i kašnjenje paketa `GevSCPD`, koje kamera drži na 0 dok je ograničenje uključeno; zapis se čuva na ovom računalu, pa ga i buđenje iz kasnijeg pozadinskog sustava također pronalazi), ili, za kameru koju ovaj računalo nije stavilo u stanje mirovanja, a koja izgleda kao da je u stanju mirovanja (njezin limit je uključen na donjoj granici), s limitom isključenim i `GevSCPD` ostavljenim na 0 pri 80 (tvorničke postavke); ograničenje postavljeno iznad praga ostaje nepromijenjeno. Samo se stanje napajanja pamti; sve ostale postavke vraćaju se u prvobitno stanje. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Snimanje kadrova dok se kalibracijska meta MAPIR ne pojavi u vidokrugu i prilagođavanje koeficijenata po kanalima s njenih panela: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Ethernet adapteri ovog računala za GigE Vision (MTU, brzina veze, jumbo okviri) i bilo koji `issues`; `estimate=True` dodaje brzinu snimanja i propusnost koje bi kamere `cameras` dobile. |
| `fix_network_settings()` | Primijeni postavke GigE Vision adaptera na ovaj računalo&#x27;kamere. Mijenja postavke adaptera i radi samo kada Chloros radi kao administrator; u suprotnom prikazuje poruku uz zapisnik onoga što je pronašao. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Korekcija vinjete za slike u mapi, svaka sa svojom mapom ravnog polja objektiva: `processed`, `lens_models` (leća za koju je svaka datoteka korekcijom obrađena), `skipped` (s razlozima). Bez `lens_model`, leća svake datoteke uzima se iz njezina EXIF modela; a `lens_model` se primjenjuje na datoteke čija imena Modela ne navode kameru, a datoteka čiji Model to proturječi preskače se. Samo datoteke. |
| `read_dls(mac, filter=None, camera_serial=None)` | Jedno očitanje s DLS-a (Bluetooth senzora za svjetlo koje pada prema dolje): spektra, zračenja po pojasu i veličina pojaseva kroz vlastiti odziv kamere kada se `camera_serial` navede. `discover_daq_sensors(transports=["ble"])` pronalazi MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Hardver hosta za kamere i konfiguracija koju preporučuje (`report_text`), te sustavni CPU / GPU / PyTorch runtime. |

- **Neki trebaju kameru samo za sebe.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` i `update_camera_firmware()` otvaraju kamera. Dok je Chloros drži (povezana u aplikaciji, s `connect_camera()`, kao član niza, ili je trenutno u upotrebi drugom zapovijedi, kao što je test niza ili otvaranje snimanja na njezinim kamerama) pozadinski sustav odbija s kodom HTTP 409 `CAMERA_HELD` (i u `code` i u `error_code`), nazivajući način na koji ga je osloboditi. Postavke boja, naglasci, `camera_power_probe()` i čitanja rade na priključenoj kameri.
- **Boja, naglasci i bijela ravnoteža su za kamere RGB.** Na mono kameri s jednim pojasom (M3M) postavke ne mijenjaju ništa i vraćaju `{"skipped": True, "reason": ...}`, kao što to radi i CLI. To su postavke zaslona: za prikaz uživo i snimke s razine zaslona, nikada za radijanciju ili reflektanciju.

---

## Privremene niske, profili poravnanja i indeksi

Ono što `chloros-cli lattice` radi bez trajne veze, kao i funkcije u `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` i `index`. Svaki šalje zahtjev koji šalje naredba CLI, s podrazumijevanim postavkama CLIa.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Oni sami otvaraju kamere.** Svaka naredba otvara svoje kamere, obavlja svoj posao i zatvara ih, stoga joj trebaju kamere koje Chloros ne drži (povezane u aplikaciji, s `connect_camera()`, ili kao član niza); backend odbija s HTTP 409 `CAMERA_HELD` i kaže kako osloboditi jedan. Privremeno otvaranje traje neko vrijeme (otprilike 40 s za nekoliko kamera). Za ponovljena snimanja, `connect_array()` jednom je brži put i sprema svaki tip izvoza, dok privremena snimka sprema de- Bayeriziranu sliku. `calibrate_alignment_profile()` je iznimka: na povezanom nizu tih kamera rješava iz vlastitih kadrova niza (`used_live_array`).
- **`serials`** su kamere koje se otvaraju, a prva aktivira okidač; `None` otvara svaku kameru na mreži.
- **`settings`** je dict preusmjeravanja postavki kamere, polja su zastavice koje postavlja CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` i `preset` (preset kamere po nazivu). Postavljena ekspozicija ili pojačanje isključuje tu automatsku kontrolu, kao što to čine i oznake.
- **`gpu`**: `True` prisiljava GPU warp, `False` CPU (`--gpu` / `--no-gpu`); `None` dopušta pozadinskom sustavu da odabere.
- **`keep_settings`**/**`clear_settings`** (svaka funkcija koja otvara kamere, zadano `False`): na kraju se svaka kamera vraća u stanje u kojem je pronađena; `keep_settings=True` umjesto toga sprema postavke izvođenja u vlastitu memoriju kamere (slobodno izvođenje), `clear_settings=True` resetira tu memoriju na tvornička postavke — vidi [Memorija kamere](#camera-memory--keep_settings--clear_settings). Odgovor (za zadatak, `stop()`-ov) sadrži `camera_memory`.
- **Putanje su *backenda***: relativne staze postaju apsolutne u odnosu na vaš radni direktorij kada se pozadinski sustav nalazi na ovom računalu. Funkcije datoteke profila poravnanja (`alignment_profile_info()`, `reorder_alignment_profile()`) čitaju datoteku na ovom računalu, kao što to čini i CLI.

| Funkcija | Što radi |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Snima jednu kameru neprekidno dok se ne zaustavi. Vraća `CliJob`. Okviri snimanja `tiff` su zapisi nizova iz datoteke `raw` snimanja te kamere (jedna ravan mozaika na senzoru&#x27;s vlastite vrijednosti, nekomprimirane); `png` i `jpg` pohranjuju 8 bita, pa su ti &#x27;runovi&#x27; demosaikovane slike za pregled. Ograničeni ili objedinjeni DAQ piše jedan `.daq` po kadru u &#x27;runu&#x27;s `daq/` mapu (`daq_files` u njezinim statistikama). Svaki se kadar zapisuje jednom sa svim svojim metapodacima (Model, vrijeme snimanja, objektiv, pojasevi, kalibracija na sirovom kadru); `stop()` se vraća jednom kad je svaki snimljeni kadar napisan (do 25 s), i tek tada se kamera zatvara. Kamera ostavljena u stanju pripravnosti za hardverski okidač (`TriggerMode` `On`) postavljen je na slobodni tijek za pokretanje (`trigger` u odgovoru o početku navodi što je pronađeno). pri zaustavljanju kamera se vraća u stanje u kojem je bila — s postavkama okidača i svim postavkama koje je tijekom snimanja promijenila — osim ako je aktivna `keep_settings=True`, koja sprema postavke snimanja u vlastitu memoriju kamere (slobodno snimanje), ili `clear_settings=True`, koji resetira tu memoriju. Pokretanje koje ne snimi nijedan kadar završava se nakon 10 s pomoću `stop_reason` `no_frames` i razlogom u `last_error`, a njezina `stop()` vraća taj razlog kao `error`. Izostavljeni kadrovi zbog toga što snimači nisu mogli pratiti tempo broje se, a `drop_reason` navodi razlog. Za kamera Chloros drži, upotrijebite `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Usporedno sinkronizirane grupe kadrova na `fps`, za `count` grupe ili dok se ne zaustavi. Vraća `CliJob` (njegov `start_reply` ima `n_cams` i žicu `pacing`); njezine konačne statistike broje grupe uhvaćene, zapisane i ispuštene, datoteke koje pisac nije mogao zapisati i grupe za koje kamera nije isporučila ništa. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Jedno sinkronizirano snimanje, de-bayereni prikaz po kameri: `results` (`filepath`, `serial`, `frame_id`) i `pacing`. Grupa bez kamere podiže uzbunu, navodeći koliko je datoteka prvo spremljeno. `lattice multi-capture` bez priključene matrice. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` sinkronizirani snimci pri fiksnoj ekspoziciji: `sync_groups`, `master`, jedan `group_timestamps` redak po okidaču (kamera koja je propustila jedan nedostaje u svom retku) i `sync`: pomak svake druge kamere od glavne, po okidaču (`offsets_us`: po kameri `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), na JEDNOM satu (`clock_domain`: `ptp`, ili `device` — PTP isključen, vlastiti sat radnog vremena svake kamere, tako da je njegova pomak kada se pokrenula i samo jitter mjeri sinhronizaciju). Ostavljen je i broji se okidač čiji pečati miješaju sat uređaja i PTP (PTP isključen ili se još zaključava) (`groups_mixed_domain`), kao i onaj kod kojeg je PTP sat još uvijek više od `agree_within_us` od mastera (`groups_clocks_disagree`); `notes` to potvrđuju. `clock_offset_mean_us` / `clock_offset_jitter_us` su najveći prosjek i jitter po kameri. `success` je netočan, s `error`, kada je okidač propušten, kada je na jednom satu uključeno manje od dvije kamere, ili kada satovi nikada ne budu suglasni (`sync.problems`); retuira se redovi u svakom slučaju. Pokreće PTP grandmastera ako ne radi (`time_sync` poziva `grandmaster_running` i `grandmaster_started_by_this_test`), uključuje PTP na svakoj kameri (kao što radi `connect_array()`), i prije prvog okidačkog događaja čeka da se PTP sat svakog kamere zaključa na grandmajstora i stabilizira: `PtpStatus` Slave na jednom grandmasteru, unutar 100 us od njega i stabilno 3 s, i svaka kamera zaključana na taj način, bez prekida, tijekom 120 s — otprilike 2 minute od uključivanja. Sat koji se upravo zaključao može biti u pogrešci do ~200 us otprilike 2 minute nakon što pročita Slave, a vlastiti `PtpOffsetFromMaster` kamere to ne prikazuje; 120 s dovodi kamere unutar ~10 us od stabiliziranog sata. Kamera koja je još uvijek zaključana od ranijeg testa na kojem je PTP ostavljen uključen, kako ga je ovaj backend vidio, broji od tog zaključavanja, tako da test odmah nakon drugog ne čeka ponovno; kamera koju test pokreće, ili ona pronađena s PTP-om uključenim koju ovaj backend nije vidio zaključanom (uključena drugdje ili prije ponovnog pokretanja backenda), čeka svojih 120 s, a u nizu su i kamere koje su se upravo zaključale i koje još uvijek čekaju svoje. Kamera koja izađe iz stanja Slave, prijeđe 100 µs ili prati drugog grandmastera tri uzastopna očitanja (1,5 s) tijekom čekanja ponovno pokreće svoje odbrojavanje; jedna ili dvije takve očitanja se zabilježavaju (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) i odbrojavanje se nastavlja, a ID grandmastera koji se ne može očitati smatra se nepoznatim, a ne promjenom. Ponovno pokretanje odbrojavanja svake kamere u isto vrijeme (unutar jedne sekunde) je vrijeme pomicanja grandmastera, a ne kamera: backend zapisnik kaže `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` ga navodi u `grandmaster_steps`, a ček koji zatim istekne započinje svoju pogrešku s `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Granica za njega nije produljena. `ptp_settle_s` (0-600; `0` za bez čekanja na poravnanje) postavlja vrijeme poravnanja za izvršavanje; druga vrijednost se odbija (HTTP 400). Čekanje je ograničeno na vrijeme poravnanja plus 90 s (210 s prema zadanim postavkama, najmanje 90 s), a poziv čeka na pozadinski sustav `timeout` s: prema zadanim postavkama 240 plus vrijeme poravnanja (360 s), kao što to čini `chloros-mcp`&#x27;s `test_array_sync`. `ptp_lock` ima `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (koliko dugo je svaka kamera zaključana i stabilna zajedno), `found_settled` (kamere su već zaključane na vrijeme stabilizacije kada su pronađene) i, po kameri, `status`, `offsets_ns`, `slave_s` (koliko je dugo zaključano; None: nije zaključano) i `remaining_s` (None kada se PTP nijedne kamere ne može uključiti). Log datoteka pozadinskog sustava sadrži redak svakih 5 s s stanjem svake kamere, koliko je dugo zaključana i koliko joj je još ostalo. Svaka se kamera uključuje s isključenim PTP-om; od tog trenutka satovi se zaključavaju za otprilike 10 s. Nakon toga se svaka kamera vraća u stanje u kojem ju je test zatekao — njezin PTP, automatska ekspozicija i pojačanje koje je test držao isključenim tijekom izvođenja (`ExposureAuto` / `GainAuto`, i ručni `ExposureTime` / `Gain`), i ostalo, osim ako `keep_settings=True` — tako da kamera ostane s PTP-om uključenim (`PtpEnable=True`, ili pokretanje s `keep_settings=True`) ostaje zaključana, i jednom kad je ovaj backend vidio da je zaključana tijekom vremena potvrde, kasniji test ne čeka na nju. Ako se satovi ne usklade, `success` je netočan prije bilo kakvog okidača, a `error` kaže što učiniti: kamera koja se nikada nije zaključala (`not_slave`) znači da grandmaster ne doseže do nje (`time_sync_status()`); ona koja je stalno padala naziva se prema tome koliko je dugo bila zaključana (`remaining_s`); satovi se i dalje usklađuju (`far`): provjerite `time_sync_status()`, a da bi bili zaključani prije testa, uključite PTP i ostavite ga uključenim (`PtpEnable=True` na svakoj kameri, ili `keep_settings=True`; test vraća ono što je pronašao, pa ostaje uključen), zatim ga pokrenite. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamere na liniji sinkronizacije i zadani master: `master_serial`, `slave_serials`, `line` je ispitano. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Kakav niz ovih modela kamera (nazivi proizvoda, `"LATT-M3C-L87-FOCN"`) bi se priključio na mrežu ovog računala: `tier`, `achievable_fps_max`, proračun žice (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` i `fix_suggestions`. Po zadanome, modeli otkrivenih kamera; ni jedna kamera nije otvorena. Za kamere koje imate, s postavkama koje bi se koristile za povezivanje, pogledajte [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Riješite profil poravnanja iz živih kadrova i spremite ga. `method`: `orb`, `akaze`, `phase`, `checkerboard` (s `checkerboard="9x6"`, unutarnji kutovi) ili `manual` (s `control_points={serial: [[x, y], ...]}`, i referentne kamere). `model`: `translation`, `rigid`, `affine` ili `homography`. Finije opcije su zastavice CLIa: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` je `{serial: {option: value}}`. `rms_threshold_px` odbija spremiti lošije rješenje. Vraća `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (prema kameri `reprojection_error`, `num_matches`) i `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Jedna poravnata višebandna slika kroz profil: `files` (TIFF s više banda sa svojim `bands` i veličinom, plus jedan po pojasu s `per_band` i jedan po kameri s `per_camera`) i `vignette_corrected`. `bands` odabire i `order` sortira pojaseve po nazivu (popis ili niz zareza). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Prikazuje poravnane više-pojasne slike (iste opcije iskrivljavanja kao `capture_aligned_multiband()`), za `count` ili dok se ne zaustavi. Vraća `CliJob`; njezina konačna statistika broji `frames_written` i sve `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Profilna datoteka sažeta kao `align-info` ispisuje je: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (po kameri `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (po kameri `band_names`, `wavelengths_nm`, `enabled`); `profile` je cijela datoteka. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Promijeni profilnu datoteku na licu mjesta: `order` su nazivi pojaseva (svaki pomiče sloj svoje kamere prema gore, tim redoslijedom), `enable` / `disable` su serijski brojevi. Vraća novi `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Indeksni presetovi (`presets`: svaki `name`, `channels`, `formula`) i boja `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Indeks preko poravnate višekanalne slike TIFF: `preset` (`NDVI`, `NDRE`, ...) ili `formula` nad simbolima `channel_map` mapira se na pojaseve (`{"NIR": "NIR_850", "Red": 0}`, po nazivu ili indeksu). Preset&#x27;simbol `channel_map` izostavlja ili uzima onu traku datoteke čije ime počinje s njim (`red` -&gt; `Red_660`, `nir` -&gt; `NIR_850_RGN`), nazvan po samoj datoteci (njezinom popisu pojaseva ili `Chloros:Band{R,G,B}Name` snimanja/proizvoda Chloros), dakle, NDVI na snimci reflektancije Chloros ne treba `channel_map`; dvije trake koje odgovaraju jednom simbolu odbijaju se, a ne pogađaju. `capture_level` navodi što trake sadrže: `reflectance` ili `radiance` čine vrijednosti kvantitativnima; nekalibrirani niz (`align-apply`-ov) daje kvalitativne vrijednosti; `"unknown"` čita iz datoteke, uključujući `Chloros:ProcessingLevel` iz datoteke Chloros. Vraća `stats` (oblik, raspon od 1. do 99. percentila, raspon LUT-a, korišteni `capture_level`) i `files` napisan uz ulaz (`<input stem>_<preset>.tif`, `..._LUT.png`) ili na `output`, putanja: s ekstenzijom slike datoteke se nazivaju prema njihovom korijenu (`output="D:/out/ndvi_FRGN.png"` piše `ndvi_FRGN.tif` i `ndvi_FRGN_LUT.png`), bez nje je to prefiks (`D:/out/ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`); čitaju se putanje iz `files`. One nose zapis kamere ulaza (proizvođač / model / serijski broj, GPS, objektiv, vrijeme snimanja u UTC-u, identifikaciju kalibracije, ali nikada njezinu raw-DN kalibraciju), `Chloros:ProcessingLevel=index`, i indeks kao njihovu jednu traku (`XMP-Camera:BandName`, jednako kao i TIFF i LUT PNG). Samo datoteke; bez kamere. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Snimanje, kalibracija (`reflectance` ili `radiance`; `raw` ga preskače), poravnati i izračunati indeks u jednom koraku; `save_multiband=True` pohranjuje kalibriranu hrpu, koju `compute_index()` može kasnije preuzeti. Vraća `stats` (s `serials` čije je pojaseve koristio) i `files`. Otvara `serials` (prva aktivira okidač), inače kamere profila čiji je unos `layer_order` omogućen, prvo referencu (otvorena čak i kad su njezini pojasevi onemogućeni); kamera čiji su pojasevi onemogućeni nikada se ne kalibrira. `channel_map` je potreban samo za simbol na koji hrpa ne može odgovoriti, kao u `compute_index()`: simbol preseta uzima jednu omogučenu traku čije ime počinje s njim, provjereno prije otvaranja bilo koje kamere (odbijeno je odgovaranje nijedne ili više, imenujući ih); prilagođeni `formula` treba svaki simbol. `index.tif` i LUT nose kameru hrpe, vrijeme snimanja (ekspozicija referentne kamere, UTC) i objektiv, i oba imenuju indeks kao svoju jednu traku (`XMP-Camera:BandName`, npr. `NDVI`: boja PNG naziva indeks, a ne njegove kanale boja). `lattice index --live`. |

### `CliJob`

Rukohvat koji tri toka vraćaju. Posao se pokreće na pozadini, koja posjeduje svoje kamere sve dok se ne zaustavi.

| Član | Opis |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (okviri spremljeni / napisano, ispušteno, `fps`, `elapsed`, pogreške pri pisanju, `stop_reason`; za kontinuirani zadatak također `frames_grabbed` i `frames_queued` -- svaki uhvaćeni kadar se zapisuje, stavlja u red, odbacuje ili ne uspijeva -- i `drop_reason`, `writers` i `write_ms`, prosječno vrijeme potrebno za zapis jednog kadra) i `output_dir`. `state` je `running`, `ended` (prestala je sama i još nije zaustavljena), `stopping` (zaustavljanje piše njezin redak: statistike uživo i `stopped_by`) ili `finished` (njezine konačne statistike, `error` i `stopped_by` zaustavljanja). Zapis za zaustavljeni posao čuva se 10 minuta (najviše posljednjih 32), za svakog klijenta: druga skripta ili `chloros-mcp` čita isti status i konačne statistike. Posao `chloros-cli` pokrenut (`lattice continuous`, `multi-stream`, `align-stream`) zaustavlja ga backend kada CLI prestane provjeravati status 30 s — ubijen, računalo je u stanju mirovanja — uz `stop_reason`, `client_gone` i `stopped_by` backend; čitanje njegovih statistika odavde ga ne održava aktivnim. Posao pokrenut iz SDK-a radi dok ga se ne zaustavi. Svaki početak šalje svoj `timeout` kao rok pozadinskog procesa (`client_deadline_s`): kamera koja se otvara nakon što se ona (koja se još pokreće) vrati i zatvori i nijedan zadatak nije pokrenut, a poziv vraća `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kamera koju zadatak drži odbijena je za svako drugo otvaranje (`connect_camera()`, drugi početak, privremeno snimanje) s HTTP 409 `CAMERA_HELD`, navodeći posao i tko ga je pokrenuo. |
| `stop(timeout=30.0)` | Zaustavite ga i zatvorite njegove kamere: `final_stats`, `output_dir` i `error` kada zaustavljanje nije prošlo glatko. Drugi poziv vraća isti odgovor, kao i zaustavljanje s drugog klijenta: ono poslano dok prvi još uvijek piše iz svog reda čekanja čeka na njega (do 25 s; nakon toga se javlja pogreška, a konačne statistike slijede na `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Pričekajte da se završi samo od sebe (dostignut je njegov `count`, puni disk, pogreška), zatim ga pozovite `stop()` kako biste prikupili konačne brojeve. `timeout` podiže `TimeoutError` dok je još u tijeku; posao bez `count` završava samo kada se zaustavi. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Njegov ID i vrsta (`continuous`, `multi-stream`, `align-stream`), gdje piše, početni odgovor rute i je li `stop()` pokrenut. |

Koristite ga kao upravitelja konteksta kako biste ga zaustavili pri izlasku iz opsega.

---

## Mapu projekta i recepte

**Projektna mapa** je mapa u kojoj se stvaraju novi projekti, a koju dijele aplikacija Chloros, CLI i SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Konfigurirana mapa koja više ne postoji vraća se na zadano (`Documents/MAPIR/Chloros Projects`), kao i aplikacija. Relativna putanja dana `set_project_folder()`-u uzima se iz vašeg radnog direktorija kada se pozadinski sustav nalazi na ovom računalu.

**Recept** je popis radnji u YAML / JSON formatu koji se izvršava na spremljenom projektu. `run_project_recipe()` ga pokreće pomoću CLI-a vlastiti izvršitelj (instaliran uz SDK), pa se jedna receptura ponaša isto s bilo kojeg od sljedećih:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** je `.yaml` / `.yml` / datoteka `.json` ili sam recept: dict s `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), i opcionalno `overrides` i `stop_on_error` (po zadanom true: izvršavanje se zaustavlja na prvoj radnji koja ne uspije). Radnje su [Recipe DSL](cli-reference.md#recipe-dsl) referentnog vodiča za CLI; relativni `output` rješava se u odnosu na radni direktorij vaše skripte.
- **Vraća** `ok` (svaka je radnja uspjela), `actions` (koliko), `log` (izlaz programa; `echo=True` ga također ispisuje tijekom izvođenja) i `validation`. Datoteka s receptima koja nedostaje ili se ne može parzirati uzrokuje `ChlorosConnectError`.
- **Provjerava se prije pokretanja.**`validation` je `{"ok", "errors", "warnings", "steps"}`: svaka radnja mora biti točno jedna poznata glagolska riječ kao njezin ključ (`{"capture": {"name": ...}}`, ne `{"action": "capture"}` ili `{"type": "capture"}`) koji imenuje kameru, niz ili senzor koje projekt ima, odgovarajuće vrste. Recept s greškom pokreće**ni** jednu radnju (`ok` false, greške u `log`), bez obzira na to što `stop_on_error` kaže. Neiskorišteni ključevi, i svaki niz ili senzor svjetla kojim upravlja recept, a koji trenutno nije priključen, su `warnings`. Recept ne povezuje uređaje: priključite prvo projektove (`ChlorosProject.connect_all()`, `chloros-cli project connect`, ili alat MCP `connect_project_devices`). `dry_run=True` ne pokreće ništa i vraća `{"recipe": ..., "validation": ...}`.
- Jedan recept se izvršava u jednom trenutku u procesu. YAML receptu je potreban PyYAML.

---

## Automatizacija projekta — `ChlorosProject`

Spremljen projekt Chlorosa je mapa koja sadrži `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` učitava manifest (`relink`: mapa u kojoj se sada nalaze nedostajuće izvorne datoteke premještenog projekta, koja se koristi kada ga pozadinski sustav učitava -- `process()` -- vidi **Premješteni projekt** ispod `ChlorosLocal`), a `connect_all` dovodi svaku spremljenu kameru, niz i senzor svjetla online s njezinim spremljenim postavkama — istim hardverskim stanjem koje bi GUI proizveo. Senzori svjetla povezuju se putem backenda [DAQ pool senzora](#daq-sensor-sessions), stoga im je potreban pokrenuti backend (vidi [`SensorHandle`](#sensorhandle)).

### Minimalni primjer

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Ili kao upravitelj konteksta:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metode `ChlorosProject`

| Metoda | Opis |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Otkrije i poveže svaku spremljenu kameru, niz i senzor svjetla. Vraća izvješće o povezivanju po klasi (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, svaki s `connected`, `missing` i `errors`; `arrays` također nosi `alignment`). S pozadinskim servisom koji sluša na `127.0.0.1:5000`, spremljene kamere i nizovi se pronalaze i povezuju putem njega. Ako ga nema, ovaj proces ih sam pronalazi i upravlja njima putem `lattice_sdk`, koji zahtijeva Arena SDK runtime i OpenCV (`cv2`) — kotačić SDK-a ne instalira nijedan; kada ne uspije, svaka spremljena kamera i niz navedeni su pod `errors` s razlogom, a `missing` ostaje prazan. Backend koji radi, ali odbija SDK (odjavljeno: HTTP 401; plan ispod bakra: 403) nije &quot;nema backenda&quot;: svaka spremljena kamera, niz i senzor svjetla navedeni su pod `errors` s razlogom backenda, i ništa se ne otvara u tom procesu. Svaki poziv ponovno provjerava backend, pa se `connect_all()` nakon prijave ili pokretanja Chloros povezuje na isti objekt projekta. Spremljena niz s istim kamerama (GUI-ja ili `chloros-cli project connect`-a) već prisutna u pozadini pridružuje se takva kakva jest, bez ponovnog primjenjivanja spremljenih postavki, i `disconnect_all()` ga ostavlja aktivnim. Senzori svjetla uvijek prolaze kroz backend (vidi [`SensorHandle`](#sensorhandle)). Nikada ne pokreće backend. |
| `disconnect_all()` | Rastavljanje svega. Svaki se uređaj prvo oslobađa; zatim, ako pozadinski sustav nije potvrdio oslobađanje kamere, niza ili senzora svjetla (rastavljanje je isteklom vremena ili je odbijeno), jedan `ChlorosConnectError` imenuje svaki. Uređaj koji je već uklonjen, ili ako ne radi nijedan backend, nema nikakvih izlaznih poruka. Niz ili senzor svjetla koji je drugi klijent imao otvoren ostaje u pogonu. Projektov `with` izlaz nikada se ne aktivira. |
| `capture_all(output_dir=".")` | Jedan kadar sa svake priključene kamere + niza, i jedan spektar (`SensorHandle.read()`) sa svakog priključenog senzora svjetla, uzimani redom (kamere, zatim nizovi, zatim senzori), a ne istovremeno. |
| `stream(camera, overlays=False, fps=10.0)` | Generator koji isporučuje BGR `numpy` okvire iz nazvane kamere (ili niza). `overlays=False` čita okvire pomoću upravljačas `grab()` na razini `debayered`: putem pozadinskog sustava kada je projekt povezan preko njega, inače izravna `lattice_sdk` petlja za hvatanje (nizovi vraćaju `{serial: frame}` dict-ove). `overlays=True` usmjerava kroz `ChlorosLocal.camera_stream()` → MJPEG feed backenda `/api/camera/<serial>/stream-annotated`, s pohranjenim blokom `ui.overlay` kamere proslijeđenim kao parametri upita. Zahtijeva način pozadine, OpenCV (`cv2`) i **samostalnu kameru**: kamera u izravnom načinu rada pokreće `RuntimeError` (backend ne može zauzeti kameru koju ovaj proces posjeduje) i niz pokreće `NotImplementedError` (nadograđuje kompozit po kameri — prenosi član po imenu). Jednokratni ekvivalent: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Pokrenite poravnanje na svakoj trenutačno priključenoj matrici. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Pokrenite pipeline kalibracije/indeksiranja na slikama projekta (omota `ChlorosLocal.process`; ova četiri su **jedini** prihvaćeni kwargs — `indices=` itd. povećava `TypeError`; postavlja indekse putem `ChlorosLocal.configure()`). Lijenim načinom konstruira `ChlorosLocal()`, koji automatski pokreće pozadinski procesor. |

Atributi:
- `proj.cameras` — `Dict[str, CameraHandle]` s ključem po nazivu I serijskom broju.
- `proj.arrays` — `Dict[str, ArrayHandle]` ključiran po nazivu I array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` ključiran po nazivu I slot_id.
- `proj.config` — `project.json["config"]` dict.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Razine obrade.** `capture()`, `grab()` i `frame_stream()` svi uzimaju isti `processing`
token, a lanac je kumulativan — svaka razina izvršava sve što je iznad nje:

| Razina | Izlaz | Napomene |
| --- | --- | --- |
| `raw` | 1-kanalni Bayer, nativni za senzor | Bez demosaika. Preklapanja nisu dostupna na ovoj razini. |
| `debayered` | 3-kanalni boja (**zadano** za `CameraHandle` i svaki `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` i `stream()` zadano su `raw`): BGR u memoriji (`grab()` / `frame_stream()`), spremljeno kao RGB od strane `capture()` | U načinu rada pozadine, kao snimanje: tamni pomak kalibracijskog paketa (DSNU) od sirovog kadra, zatim OpenCV-ov demosaik osjetljiv na rubove — bez LUT-a za linearnost, ravnog polja ili unmixa. Jedini je način rada koji funkcionira bez pozadinskog moda, gdje se koristi obični bilinearni demosaik biblioteke kamere (bez tamne korekcije). Spremljeni TIFF je fotometrijski RGB, pa se kanali okvira u memoriji preokreću (`frame[..., ::-1]`) za usporedbu s datotekom. |
| `radiance` | float32, W/m²/sr/nm, **jedna ravan po pojasu** | Cijeli radiometrijski lanac: demosaicing + 3×3 unmix (multispec) + DSNU + flat-field + NIST skala, s izračunatim omjerom ekspozicije × pojačanja kako bi vrijednosti bile apsolutne. |
| `reflectance` | uint16, 32768 = 1,0 (ili uint8, 255 = 1,0 — vidi `bit_depth`), **jedna ravnina po pojasu** | Radijansa podijeljena s dolaznom zračenjem (ρ = π·L/E). Potrebno je očitanje DLS/DAQ — vidi napomenu u nastavku. |
| `display` | 8-bitni sRGB-olik | GUI-ekvivalentni render: CCM + bilanca bijele + gamma putem aktivnog profilu boja kamere. |

U izravnom (bez pozadinskog procesora) načinu rada dostupan je samo `debayered`: `grab()` na drugoj razini podiže
`NotImplementedError`, `capture()` zanemaruje `processing`, a `frame_stream()` podiže
`RuntimeError` na svakoj razini. `reflectance` treba upotrebljivu downwelling očitanje — krajnje točke okvira automatski uvlače
objedinjeni DAQ u DLS utor kamere, ali bez vezanog DAQ-a lanac odbija
izlaz reflektancije i iskreno bilježi degradaciju u vraćenim metapodacima umjesto da to tiho učini
vraćajući manji proizvod.

> **Mjera DN reflektancije — ne kodirajte je fiksno.** Reflektancija LATTICE koristi `32768` = ρ 1.0 i utiskuje
> XMP `Chloros:PixelScale=32768`; reflektancija Survey3 koristi `65535` = ρ 1.0 i nema nikakvu
> `Chloros:*` oznake. Očitajte oznaku i podijelite s njom. Definirana je u domeni uint16, pa ostaje
> `32768` za svaki format koji mijenja mjerilo (16-bitni TIFF, 8-bitni TIFF, 8-bitni PNG/JPG, 32-bitni postotak)
> — prvo normalizirajte pohranjenu vrstu podataka (dtype) natrag na uint16 (×257 iz 8-bitnog, ×65535 iz float). Jedini
> izuzetak je datoteka iz izdanja prije verzije 1.3.0: tada je 8-bitni TIFF bio *obrezan*, a ne preskaliran, pa
> ne opisuje ga nikakva skala i ne sadrži `PixelScale` ili MicaSense tupel. Nedostajuću oznaku na
> LATTICE datoteci reflektance tretirajte kao &quot;nema valjane skale&quot;, a ne kao zadanu vrijednost.

**Kalibracijske oznake: samo na `raw`.** Datoteka `raw` snimke sadrži paket kalibracije kamere: payload `CalibrationJSON` i oznake kompatibilne s MicaSenseom (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), koji opisuju sirove vrijednosti senzora. Svaka datoteka drugog sloja (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) i svaki kadar kontinuiranog snimanja / višekamera-streama nosi identifikaciju kamere (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) i vlastiti zapis okvira, `XMP-mapir:FrameJSON` (ekspozicija, pojačanje, temperatura, format piksela, vrijeme snimanja, DAQ podudarnost), budući da svaki proizvod `process()` nosi taj identitet. Prilikom obrade, kalibracija takve datoteke pronalazi se prema njezinu serijskom broju i `CalBundleSha256` u predmemoriji kalibracija programa Chloros, pri čemu se paket preuzima jednom ako ga računalo nema.

#### Datoteke izrađene od više kamera

Datoteka izrađena od pojaseva više od jedne kamere — višopojasni sloj (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), živi indeks napravljen od više kamera (`index.tif`, `index_LUT.png`), `composite/` slika nizovog snimanja — navodi svaku kameru koja je sudjelovala:

- **Po-popise kamera, po redu pojaseva** (kompozit: pozadinski sloj, zatim prednji; indeksni sloj, kamere iz kojih potječu njezine trake): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s`, `capture_aligned_multiband()`, `start_aligned_stream()`, `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers`, `-`, `;`, `F-1479-1;F-1480-1;F-1481-1`, `SerialNumber`, `214000535,214000551`, `XMP-mapir:CameraSerial`, `CalBundleSha256`, `index.tif`, `index_LUT.png`, `Model`, `LATTICE`, `LATT-…`, `capture_aligned_multiband(per_camera=True)`, `capture_aligned_multiband()`, `start_aligned_stream()`, `per_camera`, `bit_depth`, `grab()`, , `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibracijski paket kroz koji su prošle trake svake kamere, inače — hrpa `capture_aligned_multiband()` / `start_aligned_stream()`, čije trake ne prolaze kroz nijedan — paket kamere Chloros je spremio u predmemoriju, onaj koji se naziva u njezinim datotekama snimanja), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralelne liste, stavka *n* svake opisuje istu kameru; `-` gdje kamera nema vrijednost (nema kalibracijski paket, nepoznat objektiv). Svaka kamera je **jedna** stavka: gdje njezin paket ima nekoliko vrijednosti za polje — certifikati halogenih lampi paketa — povezani su pomoću `;` (`F-1479-1;F-1480-1;F-1481-1`), nikada zarezom, tako da se popis čita kao jedna stavka po kameri bez obzira na način ispisivanja.
- **EXIF `SerialNumber`** sadrži iste serijske brojeve, spojene zarezom (`214000535,214000551`).
- Ne sadrži **ni** identifikaciju pojedinačne kamere (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): one označavaju jednu kameru i njezin paket, a obrada bi iz njih odredila kalibraciju datoteke. Indeks izračunat iz pojaseva nekoliko kamera (`index.tif`, `index_LUT.png`) ima EXIF `Model` `LATTICE` (oprema), ne model jedne kamere `LATT-…`.

Datoteka pojaseva **jedne** kamere — pojasni TIFF, `capture_aligned_multiband(per_camera=True)` kadar, hrpa ili živi indeks jedne kamere — sadrži vlastite oznake identiteta te kamere, kao i svaka druga datoteka snimanja. `capture_aligned_multiband()`-ovi stack, datoteke po pojasu i po kameri te svaki `start_aligned_stream()` stack i njegove datoteke po pojasu nose vrijeme snimanja svoje grupe (ekspozicija referentne kamere, UTC), a okvir `per_camera` mono kamere sastoji se od jedne ravnine, na razini `bit_depth`, kao i hrpa.**Orijentacija kamere (okretanje / rotacija).** Kadri iz `grab()` / `frame_stream()` uvijek su u
nativnoj orijentaciji kamere, na svakoj razini. Snimke koje se spremaju slijede postavku orijentacije
kamere (Ogledalo H / Ogledalo V / rotacija u GUI-ju):

- **Datoteke `raw` i `debayered` su točno ono što je senzor pročitao**, bez obzira na postavke — okrenuta ili preokrenuta Bayerova mozaika bila bi drugačiji uzorak filtera boja, a kalibracijske mape su
  na vlastitoj mreži senzora.
- **Datoteke `display`, `radiance`, `reflectance` i indeksne datoteke su orijentirane** — jednom okrenute, nakon što su svi
  matematiku kalibracije. Njihove maske stezaljki nalaze se na istoj mreži.
- **Svaka datoteka snimanja bilježi postavku** u XMP-u: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, u smjeru kazaljki na satu, nakon
  preokreta) i `ChlorosOrientationApplied` — `false` na sirovom i dekodiranom (debayered), `true` na orijentiranim
  proizvodima. `ChlorosProcessing` navodi operacije primijenjene na piksele te datoteke (`none` kada
  je nativno). To uključuje svaki kadar neprekidnog snimanja (vlastita postavka kamere,
  neprimijenjena) i višekamera snimka (identitet: njezini članovi kadrovi su nativni, kao kod
  niznog snimanja).
- **Obrada projekta primjenjuje zabilježenu orijentaciju** na proizvode koje stvara iz sirovog
  (prvo kalibracija u koordinatama senzora, zatim iskrivljenje poravnanja niza, na kraju orijentacija), tako da odgovaraju proizvodima snimanja uživo — uključujući i rezervne proizvode s nekorektiranom karakteristikom senzora / korekcijom vinjetiranja; njegov debayered proizvod ostaje u izvornom obliku.
- **Preglednik slika Chloros prikazuje sirovu ili debayered datoteku okrenutu prema zapisu** — samo za prikaz,
  datoteka se ne mijenja; datoteka bez zapisa prikazuje se onako kako je pohranjena.
- **Snimke bez zapisa obrađuju se kao nativne.** Prije verzije 1.3.0, snimanje s jedne kamere
  samo je pretvaralo svoje raw i debayered datoteke; Chloros ne može razlikovati takvu raw datoteku od nativne i
  ne pretpostavlja.
- **Snimanja nizova prema zadanim postavkama zadržavaju svaku datoteku nativnom** i bilježe njezin identitet;
  `ArraySession.capture(orient=True)` pretvara svoje proizvode (pogledajte kontrole snimanja nizova).

#### Izlaz po pojasu i miješane mreže

Na razinama `radiance` i `reflectance` kamera vraća **jedan sloj po pojasu te kamere** — tri za M3C (njezine tri nemiješane trake, redoslijed kanala BGR), a jedna za M3M (njegov
jedini uskopojasni filtar). Replicirane ravnine mono kamere su objedinjene, pa je **miješana
Niz M3C/M3M isporučuje različite oblike nizova po serijskom broju**. Pročitajte ključ `bands` iz metapodataka kadra kako biste saznali što je svaka ravan; ne zaključujte to iz broja kanala:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Uvijek provjerite `hdr["processing_level"] == "reflectance"` prije korištenja vrijednosti. Degradacija (bez DAQ očitanja, bez kalibracijskog paketa, nepoznati raspored pojaseva) prijavljuje se u `processing_level` i `radiometric_refused` umjesto unapređenja — niz koji dobijete natrag i dalje je valjana slika, samo
ne proizvod koji ste zatražili. Spremljena snimka koja je degradirana nosi isti kod u svom XMP-u kao `Chloros:RadiometricRefused` (npr. `no-band-layout`, `no-dls-snapshot`).

#### Jedan okidač po očitavanju — `group_id`

**Članovi niza isporučuju se iz sinkronizirane grupe okvira niza, a ne iz vlastitog
grab spremnika svake kamere.** Kamere hardverski sinkroniziranog niza aktiviraju se istovremeno, ali N čitanja po kameri
ne *uzorkuju* istovremeno — čitanje najnovijeg spremljenog okvira svake kamere prelazi preko okida
granica, a jedan izgubljeni kadar desinkronizira skup bez da itko primijeti.

Posluživanje iz grupe je nužno, ali samo po sebi nije dovoljno: svaki zahtjev i dalje
snimke *najnovije* skupine u vlastitom trenutku, a ciklus hvatanja može objavljivati u međuvremenu.
Mjereno na aktivnoj mreži od 4 kamere, **12 od 20 paralelnih očitavanja presjeklo je dvije skupine** — uvijek
točno susjedne. Stoga je očitavanje **zaključano**:

1. Svaki kadar nosi `group_id`, monotonički brojač objava niza.
2. `grab()` čita master (ili, ako se master ne može poslužiti, sljedeću kameru koja može),
   zatim zaključava preostale kamere na ID grupe na koji je stigao (`?group_id=N` na HTTP
   endpointu).
3. Backend pohranjuje nedavne grupe **najmanje pola sekunde**, tako da se sestrinskoj grupi koja je već nadmašena kasnijim objavama i dalje poslužuje taj okidač — dovoljno dugo za po-kamera
   reflektancijskog izvoza u oba smjera čak i na nizu od 9 kamera sa cijelim senzorom. Pin koji pozadinski sustav više ne drži odbijen je s kodovima `409` i `group_aged_out: true` umjesto da se tiho odgovori
   s drugačijim okidačem — `grab()` ponovno očitava skup.

Ponovno izmjereno s piniranjem: **20 od 20 očitavanja vratilo je jedan okidač.** `grab()` to radi umjesto vas;
`group_id=` vam je potreban samo kada izravno pozivate krajnju točku HTTP-a.

- Kamera za koju se ne može poslužiti grupa s pinom — ako nedostaje ili odbije zahtjev
  (`band` koju nema, razina koju ne može proizvesti) — **izostavlja se umjesto da se popunjava**
  iz drugog okidača, a ostale kamere se i dalje vraćaju.
- Dikt `grab()` koji se vraća je `GrabResult` (obična `dict` podklasa; `from chloros_sdk import GrabResult`): `.errors` mapira svaki
  izostavljeni serijski broj na iznimku koju je proizašao, a `.group_id` je zajednička grupa. Prazan
  `.errors` znači da je svaka kamera stvorila ovu grupu.
- Samostalna kamera nema grupu, izvještava `group_id: None` i **odbacuje** `group_id=` pin umjesto da ga zanemari.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Isti brojač objavljuje se na **svakom MJPEG streamu po kameri** člana niza —
prethodnom prikazu (`/api/camera/stream/<serial>`), anotiranom pretpregledu (`/stream-annotated`) i
reflektancijski stream (`/stream-reflectance`) — kao `X-Chloros-Group-Id` na svakom dijelu, pored
`X-Chloros-Frame-Id` i `X-Chloros-Timestamp-Ns`. Streamovi su kontinuirani, a ne trenutačni, stoga se podudaraju prikupljanjem dijelova i grupiranjem prema tom zaglavlju — nema pin-a za postavljanje. Samostalna kamera nema grupu. Vidi
[Per-camera MJPEG streams](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nikada ne utihne.**Tik koji zakaže iz razloga koji se može sam riješiti preskače se — stream kamere za to ne daje ništa, a stream niza daje kamere koje su odgovorile (moguće nijedan). Ti razlozi su: još nema kadra; snimajući thread kamere je pauzirao na sekundu ili dvije (`404 not streaming`, na što pozadinski sustav odgovara svaki put kad se neka druga kamera poveže ili odspoji, ili kad promjena postavke rebalansira bazen); ponovno pokretanje backenda ili `5xx`; poravnanje se još uspostavlja (`409 … retry shortly`); i odbijanje za refleksanciju jer DAQ još nema očitanje (`no-dls-snapshot`, zastarjelo ili prazno očitanje). Odbijanje u sljedećem tik-u samo bi se ponovilo — nepoznati `band`, `bit_depth=8` na radijancu, kalibracijski paket bez skale radijanca, odbijanje reflektancije s**ni jednim DAQ-om** (ništa nije vezano za kameru ili njezin niz, ništa nije povezano s
backendom: XPOTX003086 odbijanja je XPOTX003087), bilo koji drugi XPOTX003088 —
pokreće XPOTX003089 (a `RuntimeError`, koji nosi `.status` i `.body`) odjednom. Isto tako i **60 sekundi** preskočenih tikova bez međuframe, bilo kakav `fps` — dovoljno dugačak da preživi poravnanje i ponovno pokretanje pozadinskog sustava, dovoljno kratak da kamera koja je
zbilja nestane, prekida stream. Ta greška odustajanja je također `FrameFetchError`, za kameru kao i za niz, s vlastitom iznimkom posljednjeg očitavanja na `.errors` (a za kameru, kao `__cause__`). Za niz, &quot;odbijeno&quot; znači da je svaka kamera odbijena; iznimka `.errors` sadrži razlog svake kamere.

**Niz `frame_stream()` isporučuje svaki okidač jednom** (isto vrijedi za `ArrayHandle` i `ArraySession`). Tik koji čeka sljedeći okidač umjesto da ponovno isporuči isti skup, tako da stream teče sporije od `fps` i vlastitog kadra niza
brzinu, a uzastopni skupovi nikada ne dijele `group_id`. Niz koji prestane objavljivati podiže
`FrameFetchError` (&quot;nema novog okidača 60 s&quot;) umjesto da ponavlja svoj posljednji skup. Bez toga, stream kojem je zatraženo više nego što je niz objavio — ili koji je započeo odmah nakon povezivanja — isporučio bi ponavljanja: u jednom mjerenom testu, 20 setova sadržavalo je 11 različitih grupnih ID-ova.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Izvoz jedne trake — `band`

`grab()` / `frame_stream()` koriste `band="<name>"` za izvoz samo jedne od traka kamere kao jednokanalnog niza `(H, W, 1)` — istog oblika kao što ga proizvodi mono kamera, pa se jednobandni izlaz obrađuje jednako bez obzira na to je li došao iz M3M-a ili iz razdvajanja M3C-a. Crveni pojas FRGN-a:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Prihvaćeni pravopisni oblici su puno ime rasporeda (`Red_660`, `NIR_850`, `Green_550`) ili njegov kratki
naslov (`red`, `nir`, `green`), bez obzira na velika i mala slova. Nezadani naziv se odbacuje i prikazuje se popis dostupnih
naziva.

> **Odabir se vrši po nazivu, a ne po broju kanala — namjerno.** &quot;Kanal 1&quot; je `Green_550` u
> redoslijedu kanala BGR niza, ali *crvena* u kodu filtraredoslijeda slova (`RGN`), a ta se dva
> ne podudaraju. Odabir pogrešnog daje potpuno vjerojatan kadar pogrešnog pojasa, koji
> ništa nizvodno ne može otkriti. Iz tog razloga ne postoji numerički indeks.

#### Poravnavanje streamova — `align`

`align=True` deformira kadar u zajednički referentni okvir svoje niske. Svaka se kamera deformira
putem **iste** rezolucije profila i ROI izrezivanja, tako da se streamovi za svaku kameru koji se otvaraju neovisno
izlaze **međusobno registrirani** — ne morate ih zatražiti zajedno:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Zahtjevi i ponašanje:

- Kamera mora biti u **hardverski sinkroniziranom** nizu (M8 sinkronizacijski kabel) koji ima profil poravnanja
  — su-registracija pretpostavlja da su članovi izloženi istovremeno, što softver ili
  slobodnog tijeka niza ne pruža. U suprotnom je odbijeno s `409`.
- **Odbijeno, nikada ne ostaje tiho neusklađeno.** Putnja snimanja može spremiti neusklađeno uz upozorenje
  jer datoteka zadržava svoje oznake podrijetla i može se ponovno uskladiti kasnije; prijenos uživo nema
  takvu drugu priliku, stoga je nedostajući profil pogreška, a ne alternativno rješenje.
- **Zahtjev za poravnanje ga pokreće.** Zahtijevanje `align` na nizu bez profila pokreće kalibraciju i vraća `409` s onim na što čeka (npr. *&quot;čeka da se ekspozicija stabilizira na 214000551 (1/3)&quot;*) — pokušaj se ponavlja i uspijeva, obično unutar nekoliko desetaka sekundi.
Ovo je važno za rad bez zaslona: automatsko poravnanje pri povezivanju odustaje nakon 30 s, i svaki pokušaj
  unutar tog prozora preskače se dok se smart-AE još uvijek stabilizira, pa je niz čija se ekspozicija
  sporije konvergirao prije završavao bez profila za cijelu sesiju i bez mogućnosti dobivanja
  jednog bez otvaranja kombiniranog prikaza na radnoj površini. `alignment_status` također izvještava
  `ae_align_holdouts`, imenujući kamere koje se još poravnavaju i njihov napredak.
- Svakom toku jednog niza dodijelite **isti `max_h`**, inače će njihovi izrezani dijelovi pasti na različite pikselne
  mreže.
- Poravnanje se primjenjuje *prije* kodiranja dubine bita, tako da interpolacija iskrivljenja interpolira vrijednosti pune preciznosti.

#### `bit_depth` — žrtvovanje preciznosti radi propusnosti

`grab()` i `frame_stream()` prihvaćaju `bit_depth=16` (zadano) ili `bit_depth=8`:

| `bit_depth` | Kodiranje reflektancije | Trošak |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1.0, rezerva do ρ 2.0 | Puna preciznost. |
| `8` | uint8 obuhvaća `0..refl_max` (zadano ρ 1.0) | Polovica bajtova; ~0,004 ρ kvantizacija pri zadanoj gornjoj granici. Vrijednosti iznad gornje granice se odrezuju — vidi dolje. |

##### 8-bitni gornji prag — `refl_max`

`refl_max` postavlja ρ raspon 8-bitnog opsega, tako da su `pixel_scale = 255 / refl_max` i ρ i dalje
`pixel / pixel_scale`. `grab()` i `frame_stream()` prihvaćaju ga kao ključnu riječ i na kameri i
ručkama niza (`?refl_max=` na HTTP krajnjem točku). Povećajte ga kako biste dobili prostora za specularne
istaknute dijelove, u proporcionalno grubljim koracima:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | korak |
| --- | --- | --- |
| `1.0` (zadano) | 255 | 0.0039 ρ |
| `1.5` | 170 | 0.0059 ρ |
| `2.0` | 127.5 | 0.0078 ρ |

> **Automatska ekspozicija ne može popraviti 8-bitni klip, a pokušaj toga pogoršava podatke.**
> ρ = π·L/E pri čemu su ekspozicija i pojačanje izuzeti, pa je ρ svojstvo scene, a ne
> postavka kamere — izmjereno na živom nizu, smanjenje ekspozicije za polovicu pomaknulo je srednju vrijednost ρ za 1,3 %. Povećavanje
> ekspozicija kako bi se &quot;prilagodila&quot; rasponu umjesto toga dovodi do zasićenja **senzora**, što
> podcjenjuje L i pristrano smanjuje ρ bez podizanja pogreške (izmjereno: max ρ 1,203 → 0,985 pri
> 2× ekspoziciji, srednja vrijednost −23 %). Odaberite raspon s `refl_max`; održavanje senzora izvan
> saturacije je već zadaća AE-a, a to štiti *valjanost* ρ, a ne njegov raspon.

**Pročitajte `reflectance_clipped_fraction`, a ne zastavicu.** `reflectance_clipped_at_1_0` aktivira se na jednom pikselu — na stvarnom prizoru izvijestio je `True` dok je frakcija bila `0.000001`, jedan piksel
od milijun. Djeljak je ono što vam govori trebate li podići gornju granicu.

**`bit_depth=8` je samo za reflektanciju.** Zahtjev za `radiance` se odbacuje umjesto da se ispuni: reflektancija ima fiksni raspon od 0 do 1 koji se zadržava nakon 8-bitnog kodiranja, dok radijancija nema
fiksni raspon, pa je njegovo uklapanje u 8 bita potrebno pomoću istezanja percentila po kadru čije se mapiranje
mijenja u svakom kadru — slika, a ne mjerenje.

#### Live stream reflektancije (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` isporučuje 8-bitnu reflektancu po pojasu kao
`multipart/x-mixed-replace` — M3C kao JPEG s 3 kanala, a M3M kao sivo-skalu. Parametri upita:
`fps` (zadano 10), `jpeg_q` (zadano 95), `max_h` (0 = nativno), `band` (jedna nazvana vrpca, kao
gore), `align` (`1` za su-registraciju — ista pravila kao `align` iz SDKa), `refl_max` (gornja granica 8-bitnog
kodiranja, kao gore). Svaki dio sadrži
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` i `X-Chloros-Reflectance-Clipped`.
**Dijelovi iz različitih kamera koji nose isti `X-Chloros-Group-Id` predstavljaju isti okidač** —
tako se ponovno sastavljaju više streamova jednog niza po kameri, a to je zajamčeno
na strani poslužitelja jer se članovi niza isporučuju iz sinkronizirane grupe okvira (vidi
[Jedan okidač po čitanju](#one-trigger-per-read--group_id)). Vraća `409` kada refleksancija nije dostupna u trenutku povezivanja i prekida stream ako ona postane nedostupna usred izvođenja, tako da zaustavljeni stream znači &quot;više nema refleksancije&quot; umjesto &quot;još uvijek je u redu, samo statično&quot;.

> **JPEG je destruktivan — ovi pikseli su mjerenje.** Pogreška kompresije dolazi izravno u
> vrijednosti reflektancije i najgora je na rubovima, gdje su granice materijala. Zbog toga je kvaliteta
> zadano postavljena na 95. Ako su vam potrebni netaknuti brojevi, upotrijebite
> `grab(processing="reflectance")` / `frame_stream(...)`, koji vraćaju isti proizvod bez kodeka u putanji (i `bit_depth=8` ako je ograničenje cijena kabela, a ne kodek).

> **EXIF se prenosi do izvoza.** `process()` kopira GPS blok izvornog snimanja
> **i njegov ExifIFD** na svaki proizvod, tako da izvozi sadrže `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` i `CameraSerialNumber` kao i
> georeferenciranje. `FocalLength` je ono na temelju čega Pix4D rješava udaljenost uzorkovanja tla — bez njega rekonstrukcija pada na potpuno pogrešnu mjerilu (u jednom izmjerenom slučaju lokaciju od 411 m pretvorio je u onu od 47,8 km). Kopija namjerno nije `-all:all`: strukturalne oznake IFD0-a se kvare
> LATTICE izlaz, te `ExifImageWidth`/`Height` su isključeni jer opisuju snimanje izvora, a ne izvezeni raster.

Pod-oznake faze snimanja (primjenjuju se na radiometrijske razine — `radiance`, `reflectance`, `display`):

| Zastavica | Zadano | Značenje |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST radiometrijska skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-aware kada je DAQ povezan s kamerom. |
| `apply_index` | `False` | Procjena indeksa vegetacije. |
| `index_expression` | `None` | Prepisati formulu. Ne-prazno → automatski omogućuje indeks. |
| `annotated` | `False` | Preklapanje GUI ukrasa (zebra/rešetka/peaking). Nije dostupno za `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Vraćena vrijednost je `chloros_sdk.CapturePaths`, tipa `CapturePathMap`, a ne `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: jednorazinski
> `processing` daje svakom serijskom broju jednu stazu, dok `processing="all"` daje
> **uređenu listu** svih proizvoda spremljenih za tu kameru. `ArrayHandle.capture()` uzima
> nema `levels=`, i zanemaruje ključne riječi koje ne poznaje (`levels`, `daq`, `render_index`, `force_daq`), pa proslijedite samo one prikazane ovdje (`aligned=False`
> sprema nativne datoteke, kao na `ArraySession.capture()`); koristite
> `ArraySession.capture()` za te kontrole. Živi kombinirani kompozit, ako je bio u streamanju, stiže pod dodatnim
> ključ `"combined"`, a ne pod serijskim. Kôd koji pretpostavlja `str` prekida se na obliku popisa bez da se ikakav provjeritelj tipova usprotivi — anotacija je glasila `Dict[str, str]`
> neko vrijeme nakon što je listasti oblik isporučen, zbog čega alias postoji. Normalizirajte
> kada želite ravni oblik:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Ono što se nije moglo napisati: `.failed`.** Mapa koju `capture()` vraća je
> `chloros_sdk.CapturePaths` — obična podklasa `dict`, tako da sve gore navedeno radi
> nepromijenjeno — koja također nosi `.failed`: datoteke koje pozadinski sustav nije mogao zapisati,
> iste stavke kao [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` i `disk_full: True` na punom disku). Djelomično snimanje i dalje vraća svaku stazu koja je zapisana; na punom disku mapa je prazna i `.failed` navodi zašto. `.skipped` navodi preskoke po kameri, a `.group_seq` sinkroniziranu grupu, a `.alignment` / `.oriented` su pozadinski, kao na `CaptureResult`. `capture_smart()` ih vraća kao
> `result["frames"].failed` i `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Poravnanje niza

`ArrayHandle` izlaže cijelu površinu za poravnanje. Profili su prema zadanim postavkama samo za sesiju — pozovite `export_alignment()` eksplicitno za trajno spremanje.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Poravnanje pri povezivanju

`connect_all(align=...)` može automatski poravnati svaku matricu pri povezivanju:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Prelazi na `project.json["config"]["auto_align_on_connect"]` ako nije navedeno.

### `SensorHandle`

Spremljen senzor svjetla (DAQ-U, DAQ-M ili DAQ-E), otvoren od strane `connect_all()` u pozadini [pool senzora DAQ](#daq-sensor-sessions) — isti pool koji koriste `connect_daq_sensor()` i `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` čeka **sljedeći** spektar senzora — onaj koji je već pohranjen kada pozivate does not count — i vraća `(spectrum, is_saturated, integration_time, x, y, z)`, ili `None` ako nijedan ne stigne unutar 5 s. Pauzirani stream se nastavlja. `spectrum` je u W/m²/nm kada je senzor kalibriran.
- `.sensor` je [`DAQSensorSession`](#daq-sensor-sessions), za sve ostalo: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` i `.settings` su spremljeni unos iz `sensors.json`.

**Ono što `connect_all` obnavlja.** Svaki se senzor otvara na svojoj spremljenoj adresi — njegovom COM-portu, Bluetooth adresi ili DAQ-E hostu — i, ako je projekt zabilježio ID senzora, mora se raditi o tom senzoru: druga jedinica na toj adresi se odbacuje, ne povezuje se. Primjenjuju se njegovo spremljeno vrijeme integracije, prosječno računanje kadrova i automatska ekspozicija te njegova korekcija snimanja:

| Pohranjeni kap | Povezano s |
| --- | --- |
| Profil kapa (npr. `fov_30`) | Tim profilom. |
| Nije odabrano | `sunshine_cosine`, zadano za povezivanje, zabilježeno kao pretpostavljeno (`cap_id_source=auto_default`), pa proces upozorava. Senzori se isporučuju bez kapice: odaberite kapicu koja je ugrađena. |
| `none` — odabrano kao **Nema (goli senzor)** u GUI-ju | Nema korekcije kapa. |

DAQ-E-S ne prihvaća profil s kapom, bez obzira na ono što je spremljeno: njegov difuzor dio je njegove kalibracije (vidi [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Projekt spremljen ranijom verzijom Chloros zabilježio je goli senzor kao bez odabira, pa se ponovno povezuje s `sunshine_cosine`; ponovno odaberite **None (bare sensor)** u GUI-ju i spremite projekt.

Senzor koji je drugi klijent već otvorio u skupu (CLI, druga skripta) priključuje se onakav kakav jest, bez ponovnog primjenjivanja postavki, i `disconnect_all()` ga ostavlja aktivnim; senzor koji je otvorio `connect_all()` se oslobađa. Kao i kod `connect_daq_sensor()`, GUI programa Chloros **nije** dio bazena: zatvorite senzor u GUI-ju prije povezivanja projekta iz skripte.**Zahtjevi i izvješće.** Senzori svjetla zahtijevaju pokrenuti pozadinski program Chloros s prijavom na plaćenom Chloros+ nivou (Copper ili višem), a `connect_all()` nikada ga ne pokreće. Ako nijedan ne radi, svaki spremljeni senzor naveden je pod `report["sensors"]["errors"]`, uz napomenu o tome. Senzor koji pozadinska usluga ne može otvoriti također je naveden ondje, s razlogom pozadinske usluge; `report["sensors"]["missing"]` ostaje prazan za senzore svjetla.

---

## Izravni hardver (bez pozadinske usluge)

Kada želite potpunu neovisnost o backendu za LATTICE kamere (CI, roboti bez zaslona, ugrađeni), izvezite `lattice_sdk` izravno — `chloros_sdk` ga ponovno izvozi. Guard na `CAMERA_AVAILABLE`: `lattice_sdk` je u kotaču SDKa, ali zahtijeva prisutnost Arena SDK runtimea i OpenCV-a (`cv2`), a kotač ne instalira nijedno od njih. Senzori svjetla prolaze kroz backend — [`connect_daq_sensor()`](#daq-sensor-sessions) ili `proj.sensors` projekta — uz jednu iznimku, DAQ-M helper za amplitudu pojasa [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), koji također zahtijeva `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Prethodno podešavanje i okidač

Tri od četiri prethodna podešavanja **rade u slobodnom načinu**: kamera neprekidno ekspozira i
`capture()` vraća sljedeći kadar. `triggered` je iznimka — priprema kameru za hardverski rub na Liniji 2, pa ne snima ništa dok se on ne pojavi.

| Prednastavka | Okidač | Koristite kada |
| --- | --- | --- |
| `default` | kontinuirani rad | za opću upotrebu |
| `high_speed` | kontinuirani rad | 8-bitno, ograničenje na 60 fps, kratka ekspozicija |
| `high_quality` | kontinuirano snimanje | 12-bitno, bez ograničenja fps — uobičajen izbor za fotografije |
| `triggered` | **naoružano, Linija 2** | kamera je priključena na M8 sinkronizacijski kabel i nešto drugo je aktivira |

Ako odaberete `triggered` (ili sami postavite `trigger_mode="On"`) bez ikakvog pokretača Linije 2, svaki `capture()` će isteći — ispravno, budući da ste zatražili od kamere da pričeka. SDK to objašnjava kada se to dogodi; pogledajte
[SC_ERR_TIMEOUT during capture](#direct-hardware-backend-free).

> **Napomena — Poruke &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` pri uspostavljanju veze nisu pogreške.**
> Pri uspostavljanju veze SDK pokušava pregovarati o**jumbo paketima** (GVSP paketi od 9000 bajtova) radi veće propusnosti. Kod izravne točka-po-točka veze mrežne kartice (npr. veze s lokalnom `169.254.x.x` adresom), mreža obično ne može prenositi jumbo pakete, pa ova proba istekne i zabilježe se retci poput sljedećeg:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Ovo je **namjenski povratni postupak**: SDK automatski se vraća na standardne pakete od 1500 bajtova, a kamera nastavlja s normalnim povezivanjem (sljedeći retci s `[chunk-enable …]` dijela su normalne sekvence povezivanja). Snimanje i dalje radi.
>
> Možete preskočiti ovu provjeru, ali **ona nije samo suzdrživač dnevnika — ona isključuje jumbo pakete.** Kamera odgovara na pingove &quot;Don&#x27;t-Fragment&quot; samo do 1500 bajtova, bez obzira na to koliko je vaša mreža dobra, pa sam test pinganja nikada ne može pronaći jumbo pakete; ova proba je jedino što to može. Onemogućite je i kamera će zauvijek slati standardne pakete od 1500 bajtova, na bilo kojoj mreži:

```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```

Isplati se samo na mreži za koju *znate* da ne može prenositi jumbo pakete, gdje štedi otprilike sekundu vremena povezivanja po kameri. Budući da je to stvarna, a ne kozmetička promjena, **SDK** sada to i naznačuje kada ga upotrijebite:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Ostavite ga na miru osim ako nemate razloga.**Ako je omogućen, svaka se kamera mjeri jednom po pozadinskoj sesiji, a ponovno i kada se promijeni njezina veza (drugi adapter, druga brzina veze ili veza koja je nestala, kao što se događa pri zamjeni switcha); pri ponovnom povezivanju koristi se isti rezultat.**Nakon promjene iza switcha koji je ostao aktivan, ponovno pokrenite pozadinski sustav kako bi se mjerenje ponovilo.** Prvo povezivanje ponekad može potrajati nekoliko minuta na ovoj provjeri, a log to i navodi kada se to dogodi.
>
> Ako *želite* jumbo propusnost, omogućite jumbo od kraja do kraja (NIC MTU 9000 + usmjerivač koji ih propušta), ili ga zaključajte s `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` kada znate da veza to podržava — iako je poželjnije koristiti `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` po naredbi umjesto trajnog postavljanja, budući da zakvačena veličina preskače probu i prekida prilagodbu mreži ispred sebe. **Svaki** uređaj na putu mora prolaziti jumbo pakete — uključujući i bilo koji PoE razdjelnik ili injektor, što je uobičajeni razlog zašto inače jumbo-spremna postavka ne može prenositi te pakete.

> **`SC_ERR_TIMEOUT -1011` tijekom `capture()` / `grab*()` je drugačiji problem — to je stvarna pogreška.**
> Gornja napomena odnosi se samo na `-1011` zabilježenu**probeom pri uspostavljanju veze**. Ista pogreška zabilježena u**zapisu** znači da se kamera uspješno povezala, ali ne šalje nikakve slike:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Prepoznatljiv znak je kamera čiji je *kontrolni* kanal ispravan — otkrivanje radi, postavke i zapisi `[chunk-enable …]` uspijevaju — dok *svaki* kadar istekne vrijeme.
>
> **Uobičajeni uzrok je da je kamera aktivirana za hardverski okidač.** S `trigger_mode="On"` i `trigger_source="Line2"`, kamera uopće ne šalje ništa dok ne stigne električni impuls na M8 sinkronizacijskom kabelu. Ako nemate kabel koji pokreće tu liniju, svako hvatanje (grab) čeka zauvijek. Kamera nije pokvarena i mreža je u redu — radi točno ono što je i rečeno.
>
> `CameraSettings()` i `default` / `high_speed` / `high_quality` postavlja slobodni rad, a grab koji istekne vrijeme dok je naoružan objašnjava se umjesto da ispisuje samo `-1011`. `PRESETS["triggered"]` po zadanom naoružava Liniju2.
>
> Da biste natjerali bilo koju kameru na slobodni rad:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ako i dalje dođe do isteka vremena s `trigger_mode="Off"`, kamera doista ne isporučuje podatke — pošaljite nam zapisnik i `ip link show`.

#### Profili boja (RGB prikaz uživo) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` odabire profil boja zaslona za **prikaz uživo** na RGB kamerama (multispec kamere zanemaruju ovu postavku):

| Profil | Značenje |
| --- | --- |
| `raw` | Potpuno zaobiđite radiometrijski lanac. |
| `linear` | DSNU + ravna krivulja + WB, bez CCM-a, bez gama korekcije. |
| `natural` | Linearan + izmjereni CCM + sRGB gama, samo s jeftinim završnim obradom (izravnavanje krome + desaturacija istaknutih dijelova) — realistična zadana postavka. |
| `enhanced` | `natural` plus potpuni završni tretman s hub-parity (uklanjanje šarenih rubova, vibrancija, lokalni kontrast CLAHE). Bogatiji izgled uz otprilike **dvostruko veći trošak završne obrade po kadru**, pa niža brzina sličica uživo. |
| `custom_temp` | `natural` ali WB zakvačen na `custom_cct_k` Kelvin (DLS se zanemaruje; zaključano na 2000–10000 K na strani pozadinskog procesora). |

Profil je **samo za pregled uživo** kotačić za brzinu/izgled: spremljene snimke uvijek dobivaju potpuno bogatu obradu bez obzira na odabrani profil, pa odabir `natural` za povrat vremena sličica ne smanjuje kvalitetu onoga što završava na disku. Nepoznati profil pokreće `ValueError`; kada je kloros backend dostupan, promjena se također POST-a na njega kako bi sljedeći pretpregledni kadar odražavao tu promjenu (korisnici izravnog SDK-a bez backenda i dalje dobivaju mutaciju postavki).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Istaknuti dijelovi (RGB prikaz) — `set_highlights`

`LatticeCamera.set_highlights(value)` izdvaja najsvjetlije, najbjelje piksele **prikaza** prikaz na RGB kamerama — ispravak za svijetlu bijelu površinu koja se prikazuje kao ravna bijela mrlja. `value` je `-100`..`0` (fiksirano); `0` je isključen, a render je tada točno onakav kakav je bez postavke. Jače postavke smanjuju bijelu boju zaslona (na oko 60 % pri `-100`) i proširuju krivulju tonova, tako da površina koju je automatska svjetlina pretpregleda gurnula u čistu bijelu boju vraća svoje detalje.

To je kontrola zaslona, ali za razliku od zasićenja i kontrasta (koji mijenjaju samo uživo pregled), ona također mijenja ono što spremate: uživo pregled i snimke **na razini zaslona** prate je, pa su i te datoteke tamnije (one nose `Chloros:DisplayHighlights`); izvozi radijancu i reflektancu nikada to ne čine, i ne može oporaviti podatke senzora koji su presječeni — za to smanjite ekspoziciju. Multispec i mono kamere ga zanemaruju. Vrijednost se pohranjuje na `settings.highlights` i POST-a se na dostupan kloros backend, baš kao i `set_color_profile`; ekvivalent u CLI-u je `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M) kamere i `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je jednopojasna: jedna ravan sive ljestvice, bez Bayerovog mozaika, bez 3×3 matrice spektralnog unakrsnog smetanja. `Calibration` to prepoznaje i izlaže `is_mono` zastavicu. Refleksija se i dalje primjenjuje kao radiometrijska mapa po pojasu (nemiks je identitetska matrica), ali višespojski izračuni na jednoj kameri daju smislen rezultat, a ne besmislicu:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Za izgradnju indeksa vegetacije iz mono hardvera, kombinirajte nekoliko M3M kamera na različitim valnim duljinama u poravnati višebandni stack (pogledajte [Poravnanje nizova](#array-alignment)) i izračunajte indeks preko tog stacka umjesto na jednoj kameri.

#### Magnitudi DAQ-M pojaseva — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` povezuje se s DAQ-M putem Bluetootha pomoću vlastitog BLE MAC-a. To je jedina klasa senzora svjetla koja komunicira izravno sa senzorom, a ne putem pozadinskog sustava, stoga joj je potreban ono što je potrebno izravnim klasama kamere (`CAMERA_AVAILABLE`: Arena SDK runtime i OpenCV) plus `bleak` za Bluetooth (`pip install "chloros-sdk[camera]"` ili `pip install bleak`; bez njega `DLS` podiže `DLSError`). DAQ-M povezan drugdje (pool pozadinskog sustava, projekt, GUI) prestaje oglašavati, stoga ga prvo otpustite ondje. **Potreban je `mac_address`**: nema zadane vrijednosti, budući da svaka jedinica ima svoju, a `DLS()` bez njega pokreće `DLSError` prije nego što se pokuša bilo kakvo povezivanje. Pronađite MAC pomoću XPR-aOTX003351 (vidi [Discovery](#discovery--finding-an-address-to-connect-with)) ili `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integrira DAQ-M spektar (najnovije očitanje kada je `spectrum` `None`; `DLSError` ako još nema) kroz tri kanalska odziva kamere i vraća `{"red": …, "green": …, "nir": …}` — tipke imenuju kanale tim redoslijedom; kod krivulja kamere s oRGB-om, treća (`nir`) je njezin plavi kanal.

- Bez `spectral_response` koristi se ravna krivulja modela filtera (`filter_model` kojom je `DLS` izrađen), koja je izmjerena na jednoj jedinici.
- `spectral_response` umjesto toga koristi **vlastite** izmjerene krivulje kamere: dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, kako je pohranjeno u paketu kalibracije te kamere (`color_correction.spectral_response_smoothed` — za RGB kamere; multispektralni paketi ne sadrže takvu krivulju). Ako dict ne može upotrijebiti, vraća se na krivulju flote.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLIov `chloros-cli lattice dls --camera SERIAL` radi istu stvar putem pozadinskog sustava, pronalazeći krivulju kamere u njezinom učitavanom ili keširanom paketu kalibracije, i navodi čija je krivulja korištena. Preko HTTP-a to je `POST /api/dls/cli-read` s `mac` (potrebno: bez toga je odgovor HTTP 400, `code` `MAC_REQUIRED`) i `camera_serial` u tijelu; `band_magnitudes_source` odgovora je `camera <serial>` kada je korištena vlastita krivulja te kamere, ili `model <filter>` za krivulju flote (kamera bez krivulje: nije RGB, ili nema učitanog paketa ili u kešu).

---

## Izuzeci

Zahvatite baznu klasu za rukovanje &quot;svime što je pošlo po zlu kod Chloros&quot;:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` i `ChlorosConfigurationError` su atributi klase `chloros_sdk` (i `chloros_sdk.exceptions`) ali ne u `__all__`, stoga ih `from chloros_sdk import *` ne uvozi: uvezite ih po imenu. `ChlorosError` obitelj poziva `ChlorosLocal` i funkcije izgrađene na njoj (`process_folder()`, `process_lattice_capture()`); površinu pametnog povezivanja podiže `ChlorosConnectError`.

Hierarhija:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Primjeri od početka do kraja

### 1. Obrada mape s prilagođenom trakom napretka

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. LATTICE niz uživo → Reflektancija + DAQ referenca

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Kampanja snimanja vođena projektom

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Multi-kamera stream okvira → NumPy cijev

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skripta za snimanje bez zaslona izravno na hardveru (bez pozadinskog sustava)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Provjera mogućnosti prije povezivanja niza od 4 kamere

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent recepta za snimanje (čisti Python)

`run_project_recipe()` pokreće CLI recept onako kako jest (pogledajte [Projektna mapa i recepti](#project-folder--recipes)). Napisani umjesto toga u Python, isti koraci su:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP poslužitelj (AI asistenti)

`chloros-mcp` omogućuje AI asistentu koji govori [Model Context Protocol](https://modelcontextprotocol.io) (MCP), kao što su Claude Desktop, Claude Code ili Cursor, da upravlja Chloros: obradi slike (izradi projekt, uvozi mapu snimki, odabire postavke, obrađuje, izvještava o tome što je pokretanje izvezlo) i pokreće LATTICE kamere, nizove i DAQ senzore svjetla (povezuje, prikazuje pretpregled, postavlja ekspoziciju i boju, snima, bilježi, ažurira firmware). Njegovi alati pokrivaju ono što ovaj SDK radi. Pitate na jednostavnom jeziku, a asistent poziva alate poslužitelja:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Poslužitelj je izgrađen na ovom SDKu: alati za obradu pozivaju [`ChlorosLocal`](#image-processing--chloroslocal) i hardverske alate [smart-connect calls](#smart-connect-for-lattice-cameras), [DAQ sesije](#daq-sensor-sessions), `ChlorosProject` i [pomoćni programi za kameru](#camera-utilities), svi komuniciraju s istim pozadinskim sustavom koji koristi aplikacija Chloros. Njezini zahtjevi nose `X-Chloros-Client: sdk`, tako da backend primjenjuje provjere prijave i plana SDKa, a s njim dolaze i pravila SDKa (na primjer, Texture Aware je odbijen za LATTICE snimke).

### Instalacija

Poslužitelj se isporučuje u paketu `chloros-sdk`; nema ništa drugo za instalirati. SDK instalira naredbu `chloros-mcp`, koja je ista kao `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** instalator Chloros pomoću pip-a instalira SDK u vaš Python, čime se `chloros-mcp.exe` stavlja u mapu `Scripts` tog Pythona.
- **Linux (.deb):** paket instalira SDK za sustav `python3` i stavlja naredbu `chloros-mcp` u `/usr/bin`, tako da se nalazi na svakom korisnikovom `PATH`. U virtualenvu, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` dodaje `chloros-mcp` u virtualenvov `bin`.

Poslužitelj koristi samo standardnu biblioteku na vrhu SDKa (Python 3.7+). MCP klijent ga pokreće i komunicira s njim putem stdin/stdout; ne pokrećete ga sami, osim pomoću `--help`.

### Postavljanje klijenta

**Claude Desktop:**dodajte poslužitelj u `claude_desktop_config.json` (na Windows `%APPDATA%\Claude\claude_desktop_config.json`;**Postavke &gt; Programer &gt; Uredi konfiguraciju** otvara ga), zatim ponovno pokrenite Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kursor:** isto polje `mcpServers` kao za Claude Desktop, u `~/.cursor/mcp.json` (svaki projekt; `%USERPROFILE%\.cursor\mcp.json` na Windows) ili `.cursor/mcp.json` u jednom projektu.

Opcije se stavljaju u `args`, ili nakon naredbe za Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Da biste koristili određeni Pythonov SDK umjesto `chloros-mcp` na `PATH`, pokrenite modul s tim Pythonom (putanjom u potpunosti kao `command` ako nije prvi `python3` na `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Klijent pokreće poslužitelj bez vašeg shell-a, pa ako javi da se poslužitelj nije uspio pokrenuti, `chloros-mcp` vjerojatno nije na `PATH` koji klijent vidi. Umjesto toga navedite punu putanju (Windows, `which chloros-mcp` na Linux, gdje je .deb datoteka `/usr/bin/chloros-mcp`), ili pokrenite modul pomoću Python koji ima SDK.

### Opcije

| Opcija | Zadano | Što radi |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend Chloros koji se koristi. |
| `--no-auto-start` | isključeno | Nikada ne pokreće backend: koristi onaj koji već radi, kao što su aplikacija Chloros, usluga backenda za Linux ili tunel do backenda na drugom računalu (pogledajte [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel); staze do mapa tada moraju naznačiti istu mapu na oba računala). |
| `--backend-exe PATH` | gdje god je Chloros instaliran | Backend za pokretanje kada nijedan ne radi, za Chloros instaliran na mjestu gdje SDK ne traži. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` ili `ERROR`. Zapisnici se šalju na stderr, što većina klijenata prikazuje u svom MCP zapisniku; stdout prenosi samo protokolske poruke. |

### Alati

Argumenti svakog alata su oni poziva SDK-a koji on omota; klijent navodi svaki s njegovom potpunom shemom. Putanje su apsolutne.

**Obrada**

| Alat | Poziv SDK-a | Što radi |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Stvara projekt i postavlja ga kao otvoreni projekt na kojem alati za obradu djeluju. Običan naziv stvara mapu u zadanoj mapi projekta; apsolutna putanja stvara je ondje. |
| `open_project` | `ChlorosLocal.open_project` | Otvara spremljeni projekt i postavlja ga kao otvoreni projekt. Putovi premještenog projekta preusmjeravaju se na mapu u kojoj se projekt trenutno nalazi; `relink_folder` (apsolutno) imenuje mapu u kojoj se sada nalaze njegove ostale nedostajuće datoteke. Vraća `source_files` i `image_count`. Nije pronađeno ništa: pogreška pri popisu lokacije i imenovanju `relink_folder`; ništa se ne otvara. |
| `import_images` | `import_images` | Uvozi paroveSurvey3 `.RAW`+`.JPG`, LATTICE `.tif` snima i DAQ `.daq` snimke; vraća `count` (slike) i `scan_count` (snimke). |
| `configure_processing` | `configure` | Mijenja samo proslijeđene postavke i ništa drugo (vrijednosti: [Podržane vrijednosti](#supported-values)); `custom_settings` šalje cijelu konfiguraciju kako je zadano. |
| `get_project_settings` | `get_config` | Čita spremljene postavke obrade, izvoza i indeksiranja otvorenog projekta. |
| `start_processing` | `process(wait=False)` | Pokreće obradu otvorenog projekta i odmah se vraća. |
| `get_processing_status` | `get_status` | Izvještava `state` (`idle`, `starting`, `processing` ili `finished`) i napredak; kad je dovršeno, `summary`, `export_check_failure` kad se izvozi nije moglo provjeriti, i `image_failure` kad su slike izgubljene: ostali proizvodi su napisani; prijavite pokretanje kao neuspjelo. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Stvara projekt za mapu, uvozi ga, konfigurira i pokreće izvršavanje u jednom pozivu (`lattice: true` za LATTICE snimke). Dok je izvršavanje u tijeku, ono odbija prije nego što išta stvori, a pozadinski sustav tada također odbija promjenu projekta iz bilo kojeg alata. |
| `export_light_sensor` | `export_light_sensor` | Pisanje kalibriranih datoteka `.daq` i `.csv` za senzor svjetla projekta zapisa bez obrade slika, preuzimajući tvorničku kalibraciju iz oblaka MAPIR kada nije u predmemoriji (vidi [Light-Sensor Recordings](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Čita auditne metapodatke slike Chloros (izvor, kalibracija, primijenjene faze). |

**Kamere i nizovi**

| Alat | Poziv SDK-a | Što radi |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Što je sada povezano, u bilo kojem klijentu, svako s `connected_here`; plus poslove koje izvršava poslužitelj. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamere i senzori svjetla koji se mogu povezati. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Trenutno stanje jedne kamere, niza (s njegovom sposobnošću i zdravstvenim stanjem) ili senzora svjetla. |
| `analyze_array_network` | `analyze_array_network` | Što ovaj host i mreža mogu prenijeti za predloženu niz. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Povežite kameru ili sinkroniziranu niz (prvo majstor). |
| `connect_project_devices` | `open_project(...).connect_all` | Povežite svaku uređajnu koju pamti spremljeni projekt, s njezinim postavkama. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Osloboditi kameru, niz ili senzor svjetlosti (vidi Ponašanje). `keep_settings: true` na nizu sprema postavke svake kamere u vlastitu memoriju, `clear_settings: true` resetira tu memoriju (njegov rezultat nosi `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Uređaji projekta, zajedno. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Čitanje ili pisanje postavki kamere, po prijateljskom nazivu ili GenICnaziv čvora. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Postavite brzinu niza ili je vratite na automatsku brzinu. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Jedno snimanje (`fastest` i `wait_for_exposure` su `capture_fastest` i snimka konačne ekspozicije). Bez `output_dir` sprema se u otvoreni projekt&#x27;s `captures` mapu, a datoteke se pridružuju projektu (zadržavaju se dok se ne završi obrada); odgovor `project_add` to potvrđuje. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Neprekidna ili intervalna snimanja, kao zadatak. |
| `start_array_recording` | `ArraySession.record` / `burst` | Video kombiniranog indeksa ili sirovi burst, kao zadatak. |
| `build_burst_video` | `ArraySession.build_video` | Ponovno obraditi burst u kalibrirani video, kao zadatak. |
| `get_job_status` / `stop_job` | | Pratite ili zaustavite posao. |
| `get_camera_preview` | | Prikaz uživo svake kamere, vraćen kao slika. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Jedan kadar (ili jedan sinkronizirani skup) na razini obrade, bez spremanja: statistika po ravnini i PNG od svakog. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Poravnanje niza. |

**Senzori svjetla**

| Alat | Poziv SDKa | Što radi |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Povežite DAQ-U, DAQ-M ili DAQ-E (`cap_id` postavlja korekciju kapa). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Najnovija očitanja: vremenski žig, zasićenost, raspon spektra i vrh, te prosječeni spektar raspoređen u binoe valnih duljina (`full_spectrum` za svaku točku). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Pokrenite ili zaustavite stream. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Snimite datoteku `.daq`, kao zadatak; `duration_s` ga automatski zaustavlja. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Korekcija poklopca (`none` za golu sondu). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Koji senzor opskrbljuje reflektanciju svake kamere i niza. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Skupove pojaseva te reflektanciju i indekse okoline i objekta za dva streaming senzora. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Postavke DAQ-E uređaja, spremljene u jedinici. |

**Pomoćni programi kamere** ([Pomoćni programi kamere](#camera-utilities))

| Alat | Poziv SDKa | Opis |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` i njihovi postavljači | Boja prikaza kamere sRGB: profil i temperatura boje, zasićenost / kontrast, naglašeni tonovi. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Jednokratno podešavanje bijele ravnoteže, ili automatsko / isključeno / fiksni omjeri. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Prethodno definirani načini rada kamere. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modeli filtara, ili jedan po nazivu. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Verzije firmvera i flash (zadana postavka). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stanje napajanja i temperature; mirovanje ili buđenje. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koeficijenti iz kalibracijskog cilja u vidokrugu. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Mrežni adapteri kamere, procjena i popravak (Samo za administratora). |
| `apply_vignette_correction` | `apply_vignette_correction` | Vignetažna korekcija mape s fotografijama. |
| `read_dls` | `read_dls` | Jedno DLS očitanje. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardver i vrijeme izvršavanja hosta. |

**Privremeni kamere, profili poravnanja i indeksi** ([Privremeni nizovi, profili poravnanja i indeksi](#transient-arrays-alignment-profiles--indices)). Svaki alat ovdje koji otvara kamere koristi `keep_settings` / `clear_settings` (po zadanom false): vidi [Memorija kamere](#camera-memory--keep_settings--clear_settings).

| Alat | Poziv SDKa | Što radi |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Snimanje jedne kamere ili istovremeno snimanje više njih, kao zadatak (`duration_s` ga automatski zaustavlja). |
| `capture_cameras_transient` | `capture_cameras_transient` | Jedno sinkronizirano snimanje bez povezivanja niza. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Provjera sinkronizacije okidača i koje su kamere na liniji sinkronizacije. |
| `analyze_array_models` | `analyze_array_models` | Što bi niz ovih modela dobio na ovoj mreži. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Rješavanje i spremanje datoteke profila poravnanja (`options` sadrži detaljnije opcije rješivača). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Poravnana višebandna slika, ili niz takvih slika kao zadatak. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Sadržaj datoteke profila; redoslijed njezinih pojaseva i omogućene kamere. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Indeks presjeta i gradijenata; indeks preko višekanalnog TIFFa; snimanje u indeks u jednom koraku. |

**Sinkronizacija vremena, mapa projekta i recepti** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Project Folder &amp; Recipes](#project-folder--recipes))

| Alat | Poziv SDKa | Što radi |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Zdravlje PTP grandmajstora, njegovih vršnjaka i PTP stanje svake nizne kamere. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Ponovno ga pokrenite; postavite njegove prioritete. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Mapu u kojoj se stvaraju novi projekti. |
| `run_project_recipe` | `run_project_recipe` | Pokrenite recept spremljenog projekta kao zadatak s dnevnikom. Prvo se provjerava: recept s greškom se odbacuje i ne izvršava se ništa; upozorava se na nevezane nizove / senzore (ne povezuje ih: prvo `connect_project_devices`). `dry_run` vraća provjeru i ne izvršava se ništa. |

Ne postoji alat za otvaranje spremljenog projekta za obradu: svaki posao obrade započinje s `create_project` (ili `process_folder`).

### Zahtjevi

- **Chloros instaliran na istom računalu.** Backend aplikacije Chloros obavlja obradu. Umjesto toga, poslužitelj može koristiti backend do kojeg se pristupa putem tunela (`--backend-url` s `--no-auto-start`, vidi [Opcije](#options)).
- **Prijavljeni na tom računalu**, u aplikaciji Chloros ili pomoću `chloros-cli login`. Server se ne može prijaviti umjesto vas: alat pozvan dok niste prijavljeni odgovara &quot;Niste prijavljeni na Chloros&quot; s razlogom backenda, a asistent vam to priopćuje.
- **Bilo koja plaćena razina usluge Chloros+ (Copper ili viša).** Server ima pristup putem SDK-a, a besplatna razina Iron nema ga: alat pozvan na besplatnoj razini odgovara &quot;Nije dostupno na ovom Chloros planu&quot; s razlogom iz pozadinskog sustava.
- Python 3.7 ili noviji s SDKom, i MCP klijent koji pokreće lokalne (stdio) poslužitelje.

### Ponašanje

- **Backend.** Pokretanje poslužitelja ili popisivanje njegovih alata ništa ne pokreće; prva poziva alata uspostavlja vezu. Ako aplikacija Chloros (ili usluga backenda za Linux) radi, poslužitelj koristi njezin backend i ostavlja ga da radi. Ako nijedan backend ne radi, poslužitelj pokreće instalirani backend bez prozora. Kada se poslužitelj zatvori (prestanete koristiti klijent ili ga on ponovno pokrene), zaustavlja pokrenute zadatke snimanja i snimke senzora svjetla te oslobađa priključene uređaje; backend nastavlja raditi, tako da se neokončani postupak obrade dovršava do kraja, a sam se zaustavlja nakon 30 minuta ako ga nitko ne koristi. S `--no-auto-start` poslužitelj nikada ne pokreće pozadinsku uslugu: alat koji se pozove kada nijedna ne radi traži od vas da otvorite aplikaciju Chloros, a sljedeći poziv nakon što to učinite uspostavlja vezu.
- **Jedan otvoreni projekt.** Backend ima jedan otvoreni projekt, koji se dijeli s aplikacijom Chloros kada je aplikacija otvorena, stoga nemojte raditi na drugom projektu u aplikaciji dok asistent radi.
- **Mijenjaju se samo proslijeđena postavka.** `configure_processing` ostavlja svaku postavku za koju nije dobio vrijednost na njezinoj spremljenoj vrijednosti (SDKov `configure()` resetira debayer, korekciju vinjete, kalibraciju refleksije, PPK i format izvoza na njihove zadane vrijednosti kada im se ne proslijede vrijednosti; server ih popunjava iz projekta). Ako ne može pročitati spremljene postavke projekta, ne mijenja ništa i o tome obavještava, osim ako nisu navedene sve pet postavki. `indices` zamjenjuje popis indeksa projekta. Rezultat navodi ono što je poslano kao `settings_sent`, i pod `warnings` imenuje bilo koji indeks koji kamerna pojaseva ne mogu napraviti.
- **Obrada je start-then-poll.** `start_processing` vraća se čim se izvršavanje pokrene, što zatim traje od nekoliko minuta do nekoliko sati, i asistent ga prati s `get_processing_status`. `state` je `idle` (ništa se ne obrađuje), `starting` (backend još nije izvijestio o izvršavanju, do 20 s), `processing`, ili `finished`, s izvršavanjem [sažetak](#post-run-summary--hints): što je izvezlo, i naznake kada je izvezlo manje od očekivanog (nema slika, nije pronađen kalibracijski cilj). Proizvodi se spremaju u mapu projekta, kao u aplikaciji. `start_processing` se odbija dok je izvršavanje u tijeku, a pozadinski sustav (backend) koji prestane odgovarati prijavljuje se kao pogreška, a ne kao završeno izvršavanje.
- **Što asistent vidi.** Upozorenja SDK-a pojavljuju se u rezultatu alata pod `warnings`, a pogreške (odjavljeno, plan, neispravna putanja) kao tekst pogreške koji on može pročitati i proslijediti vam. Popis duži od 20 stavki skraćuje se na prvih 20, a njegova puna duljina pohranjuje se kao `<key>_total`. Putanje moraju biti apsolutne.
- **Uređaji se dijele.** Kamere, nizovi i senzori svjetla nalaze se u pozadini, pa pomoćnik može upravljati jednim povezanim aplikacijom Chloros, a promjene koje napravi prikazuju se u aplikaciji. `disconnect_device` objavljuje ono što je poslužitelj povezao; uređaj koji je drugi klijent povezao (aplikacija, `chloros-cli`, skripta, drugi `chloros-mcp`) oslobađa se samo s `force: true`, što je `force_disconnect` SDKa, i odbijaju se imena onih koji su ga povezali kada backend to zna; uređaj koji backend uopće ne drži prijavljuje se kao nepriključen (`released: false`, `connected: false`, nema ništa za oslobađanje). Snimanje ili stream koji back-end pokreće vodi svoj `get_job_status` prikaz s `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (grupe kadrova za stream s više kamera) — i jednolinijski `summary` (&quot;220 okvira zapisano, 55 izgubljeno, 0 neuspješno: ...&quot;), dok radi i, po završetku, prikazuje konačne brojke; drugi klijentov prikaz toga (`other_clients_jobs`) nosi iste (`list_cameras()`, `list_arrays()` i `list_daq_sensors()` nose svaki uređajs `connected_by`: `client`, `label`, `name`, `at`). Svaki `chloros-mcp` naziva se prema pozadinskom sustavu kao `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, poslan kao `X-Chloros-Client-Label`; bilo koji SDK skripta može ga postaviti). Zaustavljanje snimanja senzora svjetla drugog klijenta također zahtijeva `force`.
- **Dug posao je posao.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, i za bilo kakvo hvatanje, rješavanje ili indeksiranje zatraženo `background`, odmah vratite `job_id`; `get_job_status` slijedi ga i `stop_job` završava se. Pokretanje se nastavlja dok ga ne zaustavite osim ako mu ne date `max_captures`, `duration_s` (ili, za burst, `max_frames`); `min_free_gb` odbija pokretanje kada pogon ima manje slobodnog prostora. Nema ograničenja osim ako ne postavite jedno. Snimanje ili stream koji back-end pokreće (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) pohranjuje ID zadatka pozadinskog procesa, pa ga drugi `chloros-mcp` — onaj pokrenut kasnije — pronađe po tom ID-u: `get_job_status` ga prati (i navodi takve zadatke kao što su `other_clients_jobs`, od `GET /api/camera/cli-jobs`, svaki s `started_by`), i `stop_job` zaustavlja ga samo s `force: true`. Bilo koji drugi posao (sesija snimanja, snimanje niza ili burst, pozadinsko snimanje) radi u `chloros-mcp` koji ju je pokrenuo, i samo taj poslužitelj zna za nju: drugi koji je zatražio to kaže. Zaustavljeni čita `stopping` dok ispisuje svoj redak, zatim `finished` sa svojim konačnim brojevima, 10 minuta i iz bilo kojeg `chloros-mcp`: onaj koji je pokrenuo vidi prisilno zaustavljanje drugog na isti način. `get_job_status` nikada ne čeka na zaustavljanje: tijekom jednog (vlastitog `stop_job`-a, drugi klijent, ili zaustavljanje koje oslobađa kamere posla koji se sam završio) odmah odgovara s `stopping` i trenutačnim brojevima, a `stop_job` posla koji se već zaustavlja to i priopćuje. Posao koji je završen navodi razlog u `stop_reason`: vlastiti razlog (`no_frames`, `disk_full`, `member_missing`, ili za kontinuiranu izvedbu niza `count`, `duration`, `array_gone`), `limit reached` samo kada njezini brojevi pokazuju njezino `count` (ili burstov `max_frames`, snimanjes `duration_s`) dostignuta, `error`, `stopped` (s `stopped_by`) kada ga klijent zaustavi, inače `finished`.
- **Gdje se sprema snimanje.** u mapu `captures` otvorenog projekta, gdje ih aplikacija sprema, osim ako nije naveden `output_dir`.
- **Slike.** `get_camera_preview` i `grab_frames` vraćaju slike koje pomoćnik može pregledati, uz njihov JSON. Slike `grab_frames` su kontrastno pojačane za pregled; njezine statistike su stvarne vrijednosti.
- **Kada poslužitelj prestane s radom**, zadaci koje je pokrenuo zaustavljaju se i snimanje sa senzorom svjetla koje je pokrenuo zatvara se. Ako je poslužitelj pokrenuo pozadinski sustav, također oslobađa uređaje koje je povezao prije zaustavljanja pozadinskog sustava; u pozadinskom sustavu aplikacije oni ostaju povezani.
- **Protokoli.** Samo stdio (bez HTTP krajnje točke), samo alati (bez resursa ili upita), tekstualni i slikovni rezultati, verzije MCP protokola od 2024-11-05 do 2025-11-25.**Što je izostavljeno.** Prijava ili odjava (račun je vaš; poslužitelj se ne može prijaviti), i zaustavljanje backenda koji poslužitelj nije pokrenuo. Sve ostalo što ovaj SDK radi s obradom i hardverom ima alat. (Provizoriranje DAQ-E -- njegovo ime, auth token, OTA lozinka, zahtjev za potpisom, tvorničko resetiranje -- i izravni DAQ poslužitelji su isključivo `chloros-cli`, također u SDKu: pogrešno ime ili token zaključava svaku instalaciju Chlorosa iz jedinice.)

---

## Automatski pokretanje pozadinskih servisa

Ulaznice za pametno povezivanje — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` i `discover_daq_sensors` — su tanki klijenti za HTTP koji pretpostavljaju da pozadinski sustav (backend) sluša na `127.0.0.1:5000` (zadani URL sučelja Smart-Connect). Kada GUI ili CLI već rade, jedan je. Iz prazne skripte, možda neće postojati — pa te funkcije **automatski pokreću priloženi binarni backend** (bez prozora, na isti način kao što to radi `ChlorosLocal`) prije svog prvog poziva, a zatim čekaju do 60 s da se pokrene. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` i `analyze_array_network()` nikada ne pokreću nijedan.

Pravila:

- **Samo se lokalni URL ikada pokreće.** `backend_url` koji upućuje na `localhost` / `127.0.0.1` / `[::1]` je prihvatljiv; svaki drugi host smatra se nečijim drugim računalom i nikada se ne pokreće.
- **Backend koji završi s radom prije nego što odgovori prekida čekanje.** Poziv se odmah završava s kodom izlaska, naredbom koja je izvršena i posljednjim redovima koje je backend isprintao, umjesto da čeka 60 s. Kôd izlaska 98 iznimka je: drugi backend već drži port ili datoteku zaključavanja (Chloros aplikacija, servis backenda ili drugi skripta koji se još pokreće), pa poziv nastavlja čekati da on odgovori. `ChlorosLocal` radi isto.
- **Backend se ostavlja u pokretu za ponovnu upotrebu** (isto kao i kod CLI) — nema implicitnog zatvaranja kada se vaša skripta završi. Ponovno pokretanje skripte ponovno koristi aktivni backend.
- **Odustanite pomoću `auto_start_backend=False`** na bilo kojem od tih poziva (npr. kada ste naveli udaljeni backend ili kada sami upravljate životnim ciklusom backenda).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ako se priložena binarna datoteka ne može pronaći ili pokrenuti, sljedeći HTTP poziv izaziva `ChlorosConnectError` koji se može obraditi i koji je **prilagođen platformi**, umjesto običnog traga odbijenog povezivanja — na Windows upućuje vas na desktop aplikaciju ili naredbu `chloros-cli`; na Linux (bez GUI-ja) upućuje vas na naredbu `chloros-cli` ili `.deb`.

---

## Okruženje i zaglavlja

SDK označava svaki poziv backenda HTTP pomoću `X-Chloros-Client: sdk`. Backend primjenjuje pravila licenciranja SDK/CLI (potreban je prijavak **i** plaćeni Chloros+ plan) umjesto GUI putanje s besplatnim slojem. Ovo se postavlja automatski tijekom uvoza — ne morate ništa raditi.

Zaglavlje se dodaje svakom `requests` / `urllib` poziv čiji URL sadrži `localhost` ili `127.0.0.1`, na bilo kojem portu — uključujući i vlastite lokalne usluge — i zamjenjuje vrijednost koju ste sami postavili. Pozivi drugim hostovima ostaju nepromijenjeni.

Prepišite URL pozadinske instance prosljeđivanjem `backend_url=` (`ChlorosLocal` prima `backend_url=` ili `api_url=`; `process_folder()` i `process_lattice_capture()` prihvaćaju samo `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Instalirani backend sluša samo na petlji (loopback), stoga mu LAN `backend_url` ne može pristupiti; pogledajte [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel) za obrazac tunela.)

---

## Verziranje i kompatibilnost

- Verzija SDKa izložena je kao `chloros_sdk.__version__`.
- Ponašanje SDKa vezano je uz verziju priloženog backenda. Miješanje starijeg SDKa s novijim backendom obično radi (napredno kompatibilni krajevi), ali miješanje novijeg SDKa sa starijim backendom može dovesti do pogrešaka `404` na novim krajevima — ažurirajte desktop aplikaciju kako bi odgovarala.
- Sučelje pametnog povezivanja (`connect_camera` / `connect_array` / `connect_daq_sensor`) i krajnja točka za analizu mreže vraćaju stabilne JSON sheme; nova polja se dodaju.

---

## Savjeti za rješavanje problema

- **`ChlorosAuthenticationError: Login required`** (iz `ChlorosLocal`; pozivi pametnog povezivanja izazivaju `ChlorosConnectError: Login required. …` iz istog razloga) → Pokrenite `chloros-cli login EMAIL PASSWORD` jednom na ovom računalu ili se prijavite putem desktop aplikacije Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (ili `Failed to create project: …` iz `create_project()`) → Windows dugih putanja je isključen na ovom računalu (zadano za Windows), tako da nijedna putanja ne smije prijeći 260 znakova, a proizvodi se nalaze do otprilike 130 znakova ispod mape projekta. Ništa nije obrađeno. Poruka navodi najdužu mapu projekta koja odgovara: upotrijebite kraću ili uključite dugim putanjama Windowsa i ponovno pokrenite Chloros. Instalacijski program za Windows nudi ovu opciju kao &quot;Omogući Windows dugih putova&quot;, što je zadano označeno; inače upotrijebite Grupnu politiku: *Konfiguracija računala &gt; Upravljački predlošci &gt; Sustav &gt; Datotečni sustav &gt; Omogući Win32 dugih putova*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect poziva automatsko pokretanje lokalnog backenda, pa se ovo prikazuje samo kada priloženi binarni datoteku nije moguće pronaći/pokrenuti (npr. pip- samo host bez desktop paketa). Poruka je prilagođena platformi: na Windowsu otvorite desktop aplikaciju ili pokrenite bilo koju `chloros-cli` naredbu; na Linuxu pokrenite naredbu `chloros-cli` (ne postoji GUI) ili instalirajte `.deb`. Za udaljeni backend, proslijedite `backend_url=` (i `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** pri uvozu → `lattice_sdk` nije uspio učitati: Arena SDK runtime ili OpenCV (`cv2`), nije uvozna u ovom Pythonu. Površina vođena pozadinskim sustavom (`connect_*`, projekti kroz pozadinski sustav, `ChlorosLocal`) i dalje radi.
- **Niz se povezuje na puni senzor, ali pri niskoj brzini sličica**→ Namjenski: automatski način rada zadržava puni senzor i dopušta da se brzina spusti na ono što kabel može prenijeti. Proslijedi `binning=2` / `4` ili manji `width` / `height` za više sličica u sekundi, ili omogućite jumbo sličice / nadogradite vezu. Binning i ROI**ne** pokrivaju agregiranu prekomjernu pretplatu (`oversubscribed: true`, fps polja 0): Previše kamera za vezu ne može se riješiti binningom/ROI-jem — smanjite broj kamera, omogućite jumbo pakete ili prijeđite na brži NIC (vidi [Prekomjerno pretplaćivanje](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` izvještava da je NIC RX ring malen (~0,26 MB) / povežite vrata s porukom &quot;FRAMES WILL DROP&quot;** → Receive ring host NIC-a je na svojoj zadanoj vrijednosti (koja se često resetira na 32 nakon ažuriranja upravljačkog programa za NIC). Na Realtek USB 10GbE adapteru postavite `ReceiveBufferLen=256` i `PendingReceives=64` (podižene), zatim ponovno pokrenite pozadinski sustav (backend) kako bi ponovno pročitao prsten. Cijeli postupak: [CLI Referenca → Postavljanje i podešavanje mrežne kartice domaćina](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Domaćin se zamrzne pri ponovnom pokretanju/isključivanju, kasnije pogreške WMI `Invalid class` / mrežna kartica se ne aktivira** → zastarjeli USB 10GbE upravljački program uzrokuje `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Ažurirajte upravljački program adaptera na trenutnu verziju (≥ 2026) i ponovno primijenite postavke receive-ringa. Pogledajte [CLI reference → Postavljanje i podešavanje mrežne kartice domaćina](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Neuspjeh hvatanja niza s HTTP 503 &quot;Nema nove sinkronizirane grupe kadrova&quot;** → Niz nije objavio novu grupu kadrova tijekom čekanja. Provjerite `arr.status()`: `stalled: True` (ili `streaming: False`) s `actual_fps` 0 znači da se ciklus hvatanja zaustavio s objavljivanjem — isključite i ponovno spojite niz. Ako je `stalled` False i `actual_fps` veći od 0, ponovite pokušaj. Neposredno nakon `connect_array()`, snimanje već čeka do 20 s na prvu grupu.
- **`status()` prikazuje `link_speed_mbps: 4294`** → Windows nije mogao pročitati brzinu adaptera; to je uobičajeno za USB Ethernet adaptere i nije greška. Pogledajte [Očekivane brzine okvira](#expected-frame-rates).
- **`update_firmware()` ne uspijeva s `device never connected back on TCP <port>`** → Vatrozid je blokirao DAQ-E-ovo povezivanje s ovim računalom. Instalacijski program sustava Windows dodaje pravilo (vidi [Postavke uređaja DAQ-E](#daq-e-device-settings)); ako koristite drugi vatrozid, dopustite dolazni TCP promet na backend iz podmreže senzora.
- **Odbijeno odražavanje** → Za apsolutnu skalu reflektancije, DAQ uživo mora biti povezan s kamerom (ili nizom). Povežite ga putem GUI-ja ili upotrijebite `processing="radiance"` (W/m²/sr/nm) koji ne zahtijeva upareni senzor.
- **Prikazivanje `smart=True` traje dulje od očekivanog** → Konvergencija AE ovisi o dinamici scene. S `ArrayHandle.capture_smart()`, proširite `exposure_tolerance_pct`, skratite `stability_window_s` ili smanjite `settle_timeout_s` za brži, manje stabilan okidač.

---

## Vidi također

- [Referenca za CLI](cli-reference.md) — svaka podkomanda CLI odražava poziv SDK.
- [DAQ senzori svjetla](https://mapir.gitbook.io/chloros/daq-light-sensors) u priručniku za Chloros — senzori, njihove kapice i kalibrirani raspon, snimanje, DAQ-E umrežavanje.
- [LATTICE kamere](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) u priručniku za Chloros — povezivanje, postavke kamere, načini snimanja, nizovi.
- Online dokumentacija: `https://mapir.gitbook.io/chloros/api-python-sdk`
