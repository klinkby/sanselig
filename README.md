# sanselig.dk

Statisk website for zoneterapeut Lise Breusch Klinkby, Gunløgsgade 3, Islands Brygge.
Bygget med [Hugo](https://gohugo.io/) og udgivet på GitHub Pages.

## Kom i gang

Kræver Hugo 0.152 eller nyere, **extended**-udgaven — den bruges til at kode
billeder som WebP (se [Billeder](#billeder)); der bruges stadig ikke SCSS.

```bash
hugo server -D     # udviklingsserver på http://localhost:1313
hugo --gc --minify # byg til public/
```

## Struktur

| Sti                  | Indhold                                                       |
| -------------------- | ------------------------------------------------------------- |
| `content/`           | Alle sider som Markdown                                        |
| `data/priser.yaml`   | Prislisten — redigér her, ikke i skabelonerne                   |
| `data/anbefalinger.yaml` | Kundeudtalelser vist på forsiden                           |
| `layouts/`           | Egne skabeloner. Der bruges intet eksternt tema                |
| `assets/css/main.css`| Hele designsystemet i én fil                                   |
| `assets/img/`        | Billeder, hentet fra det eksisterende site                     |

Indholdet stammer fra Lises egne sider — det nuværende www.sanselig.dk og
WordPress-udkastet — og er redigeret, men ikke opdigtet.

### Billeder

Alle billeder ligger som JPEG/PNG i `assets/img/`. `layouts/partials/picture.html`
kører dem gennem Hugos billedpipeline til et `<picture>` med WebP (Hugos
native encoder, kræver `extended`) og original-formatet som fallback — intet
manuelt trin. Skabeloner der skal vise et nyt billede, kalder blot partialen,
fx:

```gotemplate
{{ partial "picture.html" (dict "path" "img/nyt-billede.jpg" "alt" "…") }}
```

Send `widths` (og `sizes`) med for et responsivt `srcset` — se
`hero-zoneterapi` i `layouts/index.html` som eksempel. Markdown-billeder
(`![alt](/img/x.jpg)`) går automatisk gennem samme partial via
`layouts/_default/_markup/render-image.html`.

## Design

Farverne er udledt af det rigtige logo:

| Token      | Værdi     | Brug                                              |
| ---------- | --------- | ------------------------------------------------- |
| `--brand`  | `#5C8526` | Logogrøn. Store overskrifter og dekoration         |
| `--action` | `#4A6B1E` | Knapper, links, fokusring (6,15:1 mod hvid)        |
| `--ink`    | `#2B2A27` | Brødtekst (12,98:1 mod baggrunden)                 |
| `--muted`  | `#5F5A52` | Sekundær tekst (6,19:1)                            |
| `--sand`   | `#F7F3ED` | Sidebaggrund                                       |

`--brand` giver kun 3,93:1 og bruges derfor aldrig til brødtekst — kun til
store overskrifter og grafiske elementer, hvor kravet er 3:1.

Der bruges udelukkende systemskrifttyper. Ingen Google Fonts, ingen tracking,
ingen tredjepartsdomæner — siden henter intet udefra.

## Udgivelse

`.github/workflows/deploy.yml` bygger og udgiver ved push til `main`.
`.github/workflows/build.yml` bygger hver pull request og fejler ved advarsler.

Før første udrulning skal **Settings → Pages → Source** sættes til
**GitHub Actions**.

### Domæne

Sitet ligger indtil videre på **beta.sanselig.dk**, så det live sanselig.dk
kan køre videre urørt imens.

DNS hos domæneudbyderen — én post:

```
beta   CNAME   klinkby.github.io.
```

(Underdomæner bruger en CNAME-post. Kun et apex-domæne som `sanselig.dk`
kræver A/AAAA-poster mod GitHubs adresser.)

Beta-sitet er sat til **ikke at blive indekseret**: `noindex = true` i
`hugo.toml` giver både `Disallow: /` i robots.txt og et `noindex`-metatag.
Uden det ville beta konkurrere med det live site om de samme tekster.

**Ved skift til www senere:**

1. `static/CNAME` → `www.sanselig.dk`
2. `baseURL` i `hugo.toml` → `https://www.sanselig.dk/`
3. `noindex` → `false`
4. DNS: `www CNAME klinkby.github.io.`
5. Slå **Enforce HTTPS** til i Settings → Pages, når certifikatet er udstedt

GitHub Pages tillader kun ét brugerdefineret domæne pr. repository, så beta
og www kan ikke køre samtidig fra dette repo.

## Mangler afklaring

- **Booking.** Der findes endnu ikke et offentligt booking-link. Indtil
  `booking_url` i `hugo.toml` udfyldes, peger alle "Book tid"-knapper på
  `/kontakt/`, hvor telefon og mail står.
- **To modstridende priser.** Booking-systemet angiver "Øvre ryg" til 550 kr. og
  telefonkonsultation til 600 kr., mens varekataloget siger 360 kr. og 320 kr.
  Priserne i `data/priser.yaml` følger booking-systemet. Skal bekræftes.
- **Driftsnotits.** Teksten om begrænsede åbningstider i 2026 ligger i
  `hugo.toml` som `driftsnotits`. Sæt den til `""` for at fjerne banneret.
- **Portrætbillede.** `static/img/lise.jpg` er skåret ud af en gammel collage og
  er kun 276×209 px. Et nyt foto vil løfte "Om Lise" mærkbart.

## Sikkerhed

Siden er statisk og tager ikke imod input, så angrebsfladen er lille. Derudover:

- Content-Security-Policy med `default-src 'self'` sættes som `<meta>`-tag.
  Inline JSON-LD tillades via en SHA-256-hash, der beregnes ved bygning.
- CSS og JS udgives med Subresource Integrity.
- Rå HTML i Markdown er slået fra (`markup.goldmark.renderer.unsafe = false`).
- Eksterne links har `rel="noopener noreferrer"`.
- Ingen inline event-handlere.

**Kendt begrænsning:** `frame-ancestors` og `X-Frame-Options` kan ikke sættes,
fordi GitHub Pages ikke tillader egne HTTP-headere, og begge ignoreres i et
`<meta>`-tag. Kræves beskyttelse mod clickjacking, skal siden bag en CDN eller
proxy, der kan sætte headere (fx Cloudflare).
