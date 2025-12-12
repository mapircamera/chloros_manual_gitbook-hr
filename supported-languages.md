# Podržani jezici

Chloros pruža punu podršku za sučelje na **38 jezika diljem svijeta**, što ga čini dostupnim korisnicima diljem svijeta. Možete trenutno mijenjati jezike na svim sučeljima: Desktop, Browser, CLI i Python SDK.

Chloros podržava sljedeće jezike:

| # | Jezik | Domaće ime | CLI kod |
|---|----------|-------------|----------|
| 1 | 🇺🇸 engleski | engleski | `en` |
| 2 | 🇪🇸 španjolski | španjolski | `es` |
| 3 | 🇵🇹 portugalski | português | `pt` |
| 4 | 🇫🇷 francuski | Français | `fr` |
| 5 | 🇩🇪 njemački | njemački | `de` |
| 6 | 🇮🇹 talijanski | talijanski | `it` |
| 7 | 🇯🇵 japanski | 日本語 | `ja` |
| 8 | 🇰🇷 Korejski | 한국어 | `ko` |
| 9 | 🇨🇳 kineski (pojednostavljeni) | 简体中文 | `zh` |
| 10 | 🇹🇼 kineski (tradicionalni) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 ruski | Russkij | `ru` |
| 12 | 🇳🇱 nizozemski | Nizozemski | `nl` |
| 13 | 🇸🇦 arapski | العربية | `ar` |
| 14 | 🇵🇱 poljski | Poljski | `pl` |
| 15 | 🇹🇷 turski | turski | `tr` |
| 16 | 🇮🇳 hindski | हिंदी | `hi` |
| 17 | 🇮🇩 indonezijski | Bahasa Indonezija | `id` |
| 18 | 🇻🇳 Vijetnamski | Tiếng Việt | `vi` |
| 19 | 🇹🇭 tajlandski | ไทย | `th` |
| 20 | 🇸🇪 švedski | Svenska | `sv` |
| 21 | 🇩🇰 Danski | Dansk | `da` |
| 22 | 🇳🇴 Norveški | Norsk | `no` |
| 23 | 🇫🇮 finski | Suomi | `fi` |
| 24 | 🇬🇷 grčki | Ελληνικά | `el` |
| 25 | 🇨🇿 češki | Čeština | `cs` |
| 26 | 🇭🇺 Mađarski | mađarski | `hu` |
| 27 | 🇷🇴 Rumunjski | Română | `ro` |
| 28 | 🇺🇦 ukrajinski | Ukrajinska | `uk` |
| 29 | 🇧🇷 Brazilski portugalski | português brasileiro | `pt-BR` |
| 30 | 🇭🇰 kantonski | 粵語 | `zh-HK` |
| 31 | 🇲🇾 malajski | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 slovački | Slovenčina | `sk` |
| 33 | 🇧🇬 bugarski | Bʺlgarski | `bg` |
| 34 | 🇭🇷 Hrvatski | Hrvatski | `hr` |
| 35 | 🇱🇹 litvanski | Lietuvių | `lt` |
| 36 | 🇱🇻 Latvijski | Latviešu | `lv` |
| 37 | 🇪🇪 estonski | Eesti | `et` |
| 38 | 🇸🇮 slovenski | Slovenščina | `sl` |

## Kako promijeniti jezik

### U Chloros radnoj površini/pregledniku

1. Otvorite postavke aplikacije
2. Dođite do izbornika za odabir jezika
3. Odaberite željeni jezik s popisa
4. Sučelje će se odmah ažurirati

### U Chloros CLI

Koristite naredbu `language` za prikaz ili promjenu jezika sučelja CLI:

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

### U Chloros Python SDK

Postavite parametar jezika prilikom inicijalizacije SDK da biste dobili poruke i izlaze na željenom jeziku.

## Pokrivenost

Svih 38 jezika u potpunosti je podržano u:

* **Chloros Desktop** - Kompletan GUI prijevod
* **Chloros Browser** - Web sučelje na svim jezicima
* **Chloros CLI** - sučelje naredbenog retka i izlazne poruke
* **Chloros Python SDK** - API poruke i dokumentacija

Jezična podrška osigurava da korisnici diljem svijeta mogu učinkovito raditi na svom materinjem jeziku bez prepreka.