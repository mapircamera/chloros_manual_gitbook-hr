# Podržani jezici

Chloros nudi punu podršku za sučelje na **38 jezika diljem svijeta**, što ga čini dostupnim korisnicima diljem svijeta. Možete odmah promijeniti jezik na svim sučeljima: radna površina, preglednik, CLI i Python SDK.

Chloros podržava sljedeće jezike:

| # | Jezik | Domaće ime | CLI kod |
|---|----------|-------------|----------|
| 1 | 🇺🇸 engleski | engleski | `en` |
| 2 | 🇪🇸 španjolski | španjolski | `es` |
| 3 | 🇵🇹 portugalski | portugalski | `pt` |
| 4 | 🇫🇷 francuski | francuski | `fr` |
| 5 | 🇩🇪 njemački | njemački | `de` |
| 6 | 🇮🇹 talijanski | talijanski | `it` |
| 7 | 🇯🇵 japanski | 日本語 | `ja` |
| 8 | 🇰🇷 Korejski | 한국어 | `ko` |
| 9 | 🇨🇳 Kineski (pojednostavljeni) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kineski (tradicionalni) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 ruski | ruski | `ru` |
| 12 | 🇳🇱 nizozemski | Nizozemska | `nl` |
| 13 | 🇸🇦 arapski | العربية | `ar` |
| 14 | 🇵🇱 poljski | poljski | `pl` |
| 15 | 🇹🇷 turski | Turkce | `tr` |
| 16 | 🇮🇳 hindski | हिंदी | `hi` |
| 17 | 🇮🇩 indonezijski | Bahasa Indonezija | `id` |
| 18 | 🇻🇳 Vijetnamski | Tiếng Việt | `vi` |
| 19 | 🇹🇭 tajlandski | ไทย | `th` |
| 20 | 🇸🇪 švedski | Svenska | `sv` |
| 21 | 🇩🇰 Danski | Dansk | `da` |
| 22 | 🇳🇴 Norveški | Norsk | `no` |
| 23 | 🇫🇮 finski | Suomi | `fi` |
| 24 | 🇬🇷 grčki | Ελληνικά | `el` |
| 25 | 🇨🇿 češki | Ceština | `cs` |
| 26 | 🇭🇺 Mađarski | mađarski | `hu` |
| 27 | 🇷🇴 Rumunjski | Română | `ro` |
| 28 | 🇺🇦 ukrajinski | Ukrajinska | `uk` |
| 29 | 🇧🇷 Brazilski portugalski | brazilski portugalski | `pt-BR` |
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

### Na radnoj površini/pregledniku Chloros

1. Otvorite postavke aplikacije.
2. Idite na izbornik za odabir jezika.
3. Odaberite željeni jezik s popisa.
4. Sučelje će se odmah ažurirati.

### U Chloros CLI

Koristite naredbu `language` za prikaz ili promjenu jezika CLI sučelja:

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

Postavite parametar jezika prilikom pokretanja SDK-a za primanje poruka i rezultata na željenom jeziku.

## Pokrivenost

Svih 38 jezika u potpunosti je podržano u:

* **Chloros Desktop**: Potpuni prijevod GUI-ja
* **Chloros Browser**: web sučelje na svim jezicima
* **Chloros CLI**: sučelje naredbenog retka i izlazne poruke
* **Chloros Python SDK**: API poruke i dokumentacija

Višejezična podrška osigurava da korisnici diljem svijeta mogu učinkovito raditi na svom materinjem jeziku bez prepreka.