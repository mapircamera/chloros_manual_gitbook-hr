# Prijava u Chloros+

## GUI prijava

Izbornik bočne trake korisnika <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> omogućuje vam prijavu na vaš račun Chloros+ i otključavanje dodatnih značajki.

**Prijaviti se morate samo jednom po računalu.** GUI, CLI i Python SDK dijele istu keširanu sesiju — prijava putem desktop GUI-ja također aktivira CLI i SDK na tom računalu (i obrnuto putem `chloros-cli login`).

Kada ste prijavljeni, prikazat će se detalji vašeg računa:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Razine planova

| Plan | `plan_id` | Vrsta |
| --- | --- | --- |
| Iron | `0` | Besplatno |
| Bakar | `1` | Plaćeno (Chloros+) |
| Bronca | `2` | Plaćeno (Chloros+) |
| Srebro | `3` | Plaćeno (Chloros+) |
| Zlato | `4` | Plaćeno (Chloros+) |

Pogledajte [planove i cijene](https://cloud.mapir.camera/pricing) kako biste vidjeli što svaka plaćena razina uključuje.

### Pristup putem CLI-ja / SDK-a zahtijeva plaćenu razinu

Pristup putem CLI a i Python SDK a zahtijeva **bilo koji plaćeni Chloros+ paket (Copper ili viši)**. Ovo se provodi**na strani poslužitelja** — svaki CLI/SDK zahtjev mora sadržavati i aktivnu sesiju i plaćeni plan:

| HTTP status | `error_code` | Značenje | Popravak |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Niste prijavljeni na ovom računalu | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Prijavljeni ste, ali je razina plana preniska (besplatna razina Iron) | Nadogradite na bilo koji plaćeni plan Chloros+ |

`chloros-cli status` ostaje dostupan na besplatnom planu, tako da uvijek možete vidjeti svoj trenutni plan i zašto je pristup odbijen.

### Ograničenja povezanog hardvera po planu

Svaki plan ograničava koliko LATTICE kamera i DAQ senzora svjetla može biti povezano uživo odjednom:

| Plan | LATTICE kamere | DAQ senzori svjetla |
| --- | --- | --- |
| Iron (besplatno / bez prijave) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Prijava na CLI



Prijavite se svojim Chloros+ vjerodajnicama kako biste omogućili obradu putem CLI. Na Linux u (bez GUI-ja) ovo je jedini način za aktivaciju vaše licence.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Korisnici SDK a**: Python SDK također pruža programsku metodu `logout()` za brisanje pohranjenih vjerodajnica. Za detalje pogledajte [SDK referencu](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Pohrana vjerodajnica

Pohranjene vjerodajnice i konfiguracija pohranjuju se u mapu `.chloros` vašeg korisničkog doma na **svim platformama**:

| Platforma | Put do predmemorije vjerodajnica |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Instalacija novije verzije preko postojeće čuva vaše prijavne podatke na Windows u (instalateru) i na Linux u (`.deb`, uključujući `chloros-cli update`). Deinstalacija programa Chloros — ili uklanjanje paketa za Linux — briše pohranjeno korisničko ime, pa se ponovno prijavljujete nakon svježe instalacije.

### Istek plana i privremeni rad bez interneta

Istek plana u grafičkom sučelju (GUI) prikazuje kada će vaša licenca postati nevažeća. Za ponavljajuće mjesečne pretplate isteču na kraju mjeseca; za godišnje pretplate isteču godinu dana nakon što ste je započeli.

Chloros provjerava valjanost vaše licence online, ali rad u offline načinu podržan je unutar razdoblja milosti:

* Uspješne provjere na poslužitelju pohranjuju se u predmemoriju na **5 minuta**, tako da normalna upotreba generira vrlo malo poziva za licencu.
* Keš licenciranih, potpisanih i vezanih za računalo pokriva duža razdoblja rada bez internetske veze: **30 dana za mjesečne planove**i**do datuma isteka vaše pretplate (najviše 365 dana) za godišnje planove**.
* Kada istekne razdoblje milosti, plan se svodi na besplatni Iron razred dok računalo ne uspije jednom pristupiti poslužitelju licenci; pristup se nastavlja nakon sljedeće uspješne provjere.

### Ograničenje uređaja

Svaki plan Chloros+ nudi drugačiji broj registriranih uređaja. Svaki uređaj na koji se prijavite sa svojim Chloros+ računom računa se u vaš broj registriranih uređaja. Uređaj možete preimenovati i ukloniti na stranici svog MAPIR Cloud računa.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ plan</th><th align="center">MESING</th><th align="center">BRONCA</th><th align="center">SREBRO</th><th align="center">ZLATO</th></tr></thead><tbody><tr><td align="right">Podržani uređaji</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Točan broj uređaja koji je dopušten na vašem računu prikazan je na stranici vašeg računa u oblaku MAPIR. Odjavljivanje s uređaja pouzdano oslobađa njegovo mjesto, a uređaj koji je već registriran uvijek se može ponovno prijaviti čak i kada je račun na ograničenju broja uređaja.
