# Kirjoitus-sivu — pystytysohjeet

Tämä on kysymys-sivun backend Cloudflare Workersille (Hono + D1). Frontend
(lomake, tag-valinta, share-napit) puuttuu vielä tästä pohjasta — tämä kattaa
API:n ja tietovaraston.

## 1. Esivalmistelut

```bash
npm install -g wrangler
wrangler login
```

Kloonaa/kopioi tämä projekti ja aja:

```bash
cd kirjoitus-sivu
npm install
```

## 2. Luo D1-tietokanta

```bash
npm run db:create
```

Komento tulostaa `database_id`:n — kopioi se `wrangler.toml`:iin kohtaan
`database_id = "REPLACE_WITH_ID_FROM_wrangler_d1_create"`.

Aja sitten skeema paikallisesti (testausta varten) ja tuotantoon:

```bash
npm run db:migrate:local
npm run db:migrate:remote
```

## 3. Aseta salaisuudet (secrets)

Näitä EI kirjoiteta `wrangler.toml`:iin, vaan asetetaan erikseen:

```bash
wrangler secret put GHOST_ADMIN_API_KEY
# Löytyy Ghost Admin -paneelista: Settings → Integrations → luo custom integration
# Muoto: "id:secret" (Ghost näyttää tämän suoraan integraation luonnin yhteydessä)

wrangler secret put ANTHROPIC_API_KEY
# Luo Anthropicin konsolissa: console.anthropic.com

wrangler secret put SSO_JWT_PUBLIC_CERT
# miniOrange/Auth0-dashboardista ladattu julkinen varmenne (PEM-muodossa)

wrangler secret put RESEND_API_KEY
# Jos käytät Resendia share-sähköpostien lähetykseen (resend.com)
```

## 4. Muokkaa wrangler.toml

- Vaihda `GHOST_API_URL` oikeaan domainiin
- Tarkista tag-etuliitteet (`GHOST_LANGUAGE_TAG_PREFIX` jne.) vastaamaan
  Ghostissa jo käytössä olevaa tag-nimeämiskäytäntöä
- Tarkista cron-aikataulu (`*/2 * * * *` = joka 2. minuutti) — säädä
  tarpeen mukaan, esim. `*/5 * * * *` viideksi minuutiksi

## 5. Aja paikallisesti

```bash
npm run dev
```

Testaa terveystarkistus: `curl http://localhost:8787/api/health`

## 6. Julkaise

```bash
npm run deploy
```

## Mitä tästä puuttuu ja pitää vielä rakentaa

1. **Frontend** — lomake, tag-valinta, kirjoitusohjeiden modal,
   share-napit. Voi olla erillinen Vite+React-projekti, joka kutsuu
   tätä API:a, tai lisätä samaan Workeriin staattisena assettina
   (`wrangler.toml`:iin `[assets]`-lohko).
2. **`lib/email.ts`** — Resend-integraatio kolmeen tarkoitukseen:
   (a) ilmoitus kirjoittajalle julkaisusta/hylkäyksestä,
   (b) share-napin sähköpostilähetys vastaanottajalle.
3. **SSO-kirjautumisen frontend-puoli** — Google-kirjautumispainike ja
   uudelleenohjaus miniOrangelle/Auth0:lle, joka palauttaa JWT:n
   frontendille tallennettavaksi (esim. muistiin, ei localStorageen
   ilman harkintaa XSS-riskin vuoksi).
4. **Tag-listan haku Ghostista** — `GET /api/tags`-reitti, joka hakee
   Ghost Content API:sta olemassa olevat kieli/maa/aihe-tagit, jotta
   lomakkeen pudotusvalikot pysyvät synkassa Ghostin kanssa.
5. **Duplikaattien esto ja muokkaus/poisto-oikeudet** — päätettävä
   erikseen (ks. aiempi keskustelu).

## Testaus

`test/moderation.test.ts` (ei vielä luotu tässä pohjassa) kannattaa
sisältää muutama esimerkkikysymys eri kielillä, jotta näet toimiiko
`prescreen()`/`deepReview()` odotetusti myös harvinaisemmilla kielillä
ennen kuin lisäät uuden kielivaihtoehdon sivustolle.
