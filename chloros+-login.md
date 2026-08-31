# Chloros+ prijava

## GUI prijava

Bočni izbornik korisnika <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> omogućuje vam prijavu na vaš račun Chloros+ i otključavanje dodatnih značajki.

**Prijaviti se morate samo jednom po računalu.** GUI, CLI i Python dijele istu keširanu sesiju — prijava putem desktop GUI-ja također aktivira CLI i SDK na tom računalu (i obrnuto putem `chloros-cli login`).

Kada ste prijavljeni, prikazat će se detalji vašeg računa:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Razine plana

| Plan | `plan_id` | Vrsta |
| --- | --- | --- |
| Čelik | `0` | Besplatno |
| Bakar | `1` | Plaćeno (Chloros
+) |
| Bronca | `2` | Plaćeno (Chloros
+) |
| Srebro | `3` | Plaćeno (Chloros
+) |
| Zlato | `4` | Plaćeno (Chloros
+) |

Pogledajte [planove i cijene](https://cloud.mapir.camera/pricing) kako biste vidjeli što svaka plaćena razina uključuje.

Pristup stranicamaCLI
/SDK
zahtijeva plaćeni paket

Pristup stranicamaCLI
iPython
SDK
zahtijeva **bilo koji plaćeni paketChloros
+ (bakreni ili viši)**. Ovo se provodi**na strani poslužitelja** — svaki zahtjev naCLI
/SDK
mora sadržavati i aktivnu sesiju i plaćeni paket:

| statusHTTP
| `error_code` | Značenje | Popravak |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Nije prijavljeno na ovom računalu | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Prijavljeni, ali je razina plana preniska (besplatna razina Iron) | Nadogradite na bilo koji plaćeni planChloros
+ |

`chloros-cli status` ostaje dostupan na besplatnom paketu, tako da uvijek možete vidjeti svoj trenutni paket i zašto je pristup odbijen.

### Ograničenja povezanog hardvera po paketu

Svaki paket ograničava koliko LATTICE kamera i DAQ senzora svjetla može biti povezano uživo odjednom:

| Plan | LATTICE kamere | DAQ senzori svjetla |
| --- | --- | --- |
| Iron (besplatno / bez prijave) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Prijava naCLI


Prijavite se svojim vjerodajnicama zaChloros
+ kako biste omogućili obradu uCLI
. NaLinux
(bez GUI-ja), to je jedini način za aktivaciju vaše licence.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**KorisniciSDK
a**:Python
SDK
također pruža programsku metodu `logout()` za brisanje pohranjenih vjerodajnica. Za detalje pogledajte [SDK
referencu](reference/sdk-reference.md).
{% endhint %}

**Primjer:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Posebni znakovi**: Koristite jednostruke navodnike za lozinke koje sadrže znakove kao što su `$`, `!` ili razmake.
{% endhint %}

**Izlaz:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Pohrana vjerodajnica

Pohranjene vjerodajnice i konfiguracija spremaju se u mapu `.chloros` vašeg korisničkog doma na **svim platformama**:

| Platforma | Put do predmemorije vjerodajnica |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Istek plana i offline razdoblje

Istek plana u GUI-ju prikazuje kada će vaša licenca postati nevažeća. Za ponavljajuće mjesečne pretplate isteču na kraju mjeseca; za godišnje pretplate isteču godinu dana nakon što ste započeli pretplatu.

Chloros
provjerava valjanost vaše licence putem interneta, ali rad izvan mreže podržan je unutar prijelaznog razdoblja:

* Uspješne provjere poslužitelja pohranjuju se u predmemoriju na **5 minuta**, pa normalna upotreba generira vrlo malo poziva za licencu.
* Keš potpisanih licenci vezanih za računalo pokriva duža razdoblja rada bez internetske veze: **30 dana za mjesečne planove**i**do datuma isteka vaše pretplate (najviše 365 dana) za godišnje planove**.
* Kada istekne razdoblje milosti, plan se svodi na besplatni razinu Iron dok računalo ne može jednom pristupiti poslužitelju licenci; pristup se nastavlja nakon sljedeće uspješne provjere.

### Ograničenje uređaja

Svaki planChloros
+ nudi drugačiji broj registriranih uređaja. Svaki uređaj na koji se prijavite sa svojim računomChloros
+ računa se u vaš broj registriranih uređaja. Uređaj možete preimenovati i ukloniti na stranici svog računa u oblakuMAPIR
.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros+</th><th align="center">MESING</th><th align="center">BRONCA</th><th align="center">SREBRO</th><th align="center">ZLATO</th></tr></thead><tbody><tr><td align="right">Podržani uređaji</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Točan broj uređaja dopuštenih na vašem računu prikazan je na stranici vašeg računa u oblaku MAPIR. Odjavljivanje s uređaja pouzdano oslobađa njegovo mjesto, a uređaj koji je već registriran uvijek se može ponovno prijaviti čak i kada je račun na ograničenju broja uređaja.
