# Chloros+ Prijava

## Chloros i Chloros (preglednik) Prijava

Korisnički <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> izbornik bočne trake omogućuje vam prijavu na vaš Chloros+ račun i otključavanje dodatnih značajki.

Kada se prijavite, bit će prikazani detalji vašeg računa:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" data-size="line"><figcaption></figcaption></figure>

## CLI Prijava

Prijavite se svojim vjerodajnicama Chloros+ kako biste omogućili obradu CLI.

**Sintaksa:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK korisnici**: Python SDK također pruža programsku `logout()` metodu za brisanje predmemoriranih vjerodajnica. Pogledajte [dokumentaciju Python SDK] (api-python-sdk.md#logout) za detalje.
{% završni savjet %}

**Primjer:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="upozorenje" %}
**Posebni znakovi**: Koristite jednostruke navodnike oko zaporki koje sadrže znakove kao što su `$`, `!` ili razmake.
{% završni savjet %}

**Izlaz:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Istek plana

Istek plana u GUI-ju pokazuje kada će vaša licenca postati nevažeća. Za ponavljajuće mjesečne pretplate istek je na kraju mjeseca. Za godišnje pretplate to je godina nakon što ste započeli pretplatu. Provjera licence zahtijeva mjesečnu internetsku vezu za provjeru, uz 30 dana odgode.

### Ograničenje uređaja

Svaki plan Chloros+ nudi različit broj registriranih uređaja. Svaki uređaj na koji se prijavite s računom Chloros+ računat će se u vaš broj registriranih uređaja. Možete preimenovati i ukloniti uređaj na stranici vašeg računa MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ Plan</th><th align="center">BAKAR</th><th align="center">BRONCA</th><th align="center">SREBRO</th><th align="center">ZLATO</th></tr></thead><tbody><tr><td align="right">Podržani uređaji</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>