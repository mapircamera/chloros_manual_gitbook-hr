

Chlorospruža potpunu podršku sučelja na **38 jezika diljem svijeta**, čineći ga dostupnim korisnicima širom svijeta. Jezike možete trenutačno promijeniti i u desktop sučelju i u CLI.

Chlorospodržava sljedeće jezike:

| # | Jezik | Izvorni naziv | Kôd za CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Engleski | English | `en` |
| 2 | 🇪🇸 Španjolski | Español | `es` |
| 3 | 🇵🇹 portugalski | Português | `pt` |
| 4 | 🇫🇷 francuski | Français | `fr` |
| 5 | 🇩🇪 Njemački | Deutsch | `de` |
| 6 | 🇮🇹 Talijanski | Italiano | `it` |
| 7 | 🇯🇵 Japanese | 日本語 | `ja` |
| 8 | 🇰🇷 Korean | 한국어 | `ko` |
| 9 | 🇨🇳 Kineski (pojednostavljeni) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kineski (tradicionalni) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Ruski | Русский | `ru` |
| 12 | 🇳🇱 nizozemski | Nederlands | `nl` |
| 13 | 🇸🇦 arapski | العربية | `ar` |
| 14 | 🇵🇱 Poljski | Polski | `pl` |
| 15 | 🇹🇷 Turski | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonezijski | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vijetnamski | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tajlandski | ไทย | `th` |
| 20 | 🇸🇪 Švedski | Svenska | `sv` |
| 21 | 🇩🇰 Danski | Dansk | `da` |
| 22 | 🇳🇴 Norveški | Norsk | `no` |
| 23 | 🇫🇮 Finski | Suomi | `fi` |
| 24 | 🇬🇷 Grčki | Ελληνικά | `el` |
| 25 | 🇨🇿 Češki | Čeština | `cs` |
| 26 | 🇭🇺 Mađarski | Magyar | `hu` |
| 27 | 🇷🇴 Romanian | Română | `ro` |
| 28 | 🇺🇦 Ukrainian | Українська | `uk` |
| 29 | 🇧🇷 Brazilski portugalski | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantonski | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malezijski | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovački | Slovenčina | `sk` |
| 33 | 🇧🇬 Bugarski | Български | `bg` |
| 34 | 🇭🇷 Hrvatski | Hrvatski | `hr` |
| 35 | 🇱🇹 Litavski | Lietuvių | `lt` |
| 36 | 🇱🇻 Latvijski | Latviešu | `lv` |
| 37 | 🇪🇪 Estonijski | Eesti | `et` |
| 38 | 🇸🇮 Slovenski | Slovenščina | `sl` |

## Kako promijeniti jezik

### U Chloros Desktopu

1. Otvorite postavke aplikacije
2. Idite na izbornik za odabir jezika
3. Odaberite željeni jezik s popisa
4. Sučelje će se odmah ažurirati

### U Chloros CLI

Koristite naredbu `language` za pregled ili promjenu jezika sučelja CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Za više detalja pogledajte [CLI dokumentaciju](CLI.md).

## Pokrivenost

Sva 38 jezika u potpunosti su podržana u:

* **Chloros Desktop** - Potpun prijevod GUI-ja
* **Chloros CLI** - sučelje naredbenog retka i izlazne poruke

Python SDK API i njegova [referentna dokumentacija](reference/sdk-reference.md) dostupni su na engleskom jeziku.

Podrška za jezike osigurava da korisnici diljem svijeta mogu učinkovito raditi na svom materinjem jeziku bez prepreka.
