# Status: Skatteberegner (repo)

Opdateret: 13. juli 2026 (midlertidig Cloudflare-mirror pga. GitHub-konto spærret)

## ⚠️ AKUT: GitHub-kontoen kristianjensen5 er spærret

**13. juli 2026:** GitHub viser "Access to your account has been suspended due to
a violation of our Terms of Service" ved login. Bekræftet virkede stadig
12. juli kl. 16:52 (sidste succesfulde push, Trumpspillet). Rammer HELE kontoen —
alle repos og alle GitHub Pages-sites er utilgængelige, ikke kun dette projekt.
Kristian har kontaktet/skal kontakte GitHub Support for at anke. Indtil det er
løst: **GitHub Pages-linket nedenfor virker ikke** — brug Cloudflare-mirroret i
stedet (se næste afsnit). Denne advarsel bør fjernes igen når GitHub-adgangen
er genoprettet, og der er pushet normalt til `main` igen.

## Midlertidig Cloudflare-mirror (mens GitHub er nede)

Deployet direkte fra lokale filer, uden om GitHub, til et **nyt** Cloudflare
Pages-projekt (se opdagelse nedenfor for hvorfor det er nyt og ikke det gamle):

https://skatteberegner.pages.dev/index.html
https://skatteberegner.pages.dev/pfa-pensionsberegner.html

Indeholder KUN `index.html`, `pfa-pensionsberegner.html` og
`calculator-texts.json` — ikke resten af repoet. Opdateres ikke automatisk;
skal redeployes manuelt (`npx wrangler pages deploy <mappe> --project-name
skatteberegner --branch main --commit-dirty=true`) hvis der laves flere
ændringer mens GitHub er nede.

**Opdagelse undervejs:** Der fandtes allerede et Cloudflare-projekt med samme
navn fra en tidligere session (ikke dokumenteret her). Det var deployet som
**hele repo-mappen**, hvilket i en uge har eksponeret `Hvad koster det.xlsx`
(den fil Kristian eksplicit bad om aldrig måtte være offentlig),
`STATUS.md`, `PFA-formelaudit.md` og hele `.git`-mappen offentligt på
`skatteberegner.pages.dev`. Det gamle projekt er slettet fuldstændigt
(fjerner også alle historiske deployment-specifikke URL'er), og et helt nyt,
rent projekt er oprettet i stedet. Verificeret efter sletning: ingen af de
følsomme filer er tilgængelige længere, hverken på hoveddomænet eller de
gamle deployment-URL'er.

## Formål

Repoet indeholder to selvstændige beregnere til artikler:

1. **Skattelettelsesberegner** (`index.html`) — færdig, deployet.
2. **PFA-pensionsberegner** (`pfa-pensionsberegner.html`) — "Hvad koster det at gå tidligere på pension?". Under udvikling. **Skal publiceres på politiken.dk** i et "vibecode element" — en iframe med låst 9:16-aspect ratio — når den er klar.

Direkte link til PFA-pensionsberegneren til brug i iframe (GitHub Pages —
**virker ikke pt., se AKUT-afsnit øverst**):
https://kristianjensen5.github.io/Skatteberegner/pfa-pensionsberegner.html

Begge er self-contained HTML-filer (CSS + JS inline) og deployes normalt via GitHub Pages fra samme repo.

Live-version (repo-rod, GitHub Pages — nede pt.):
https://kristianjensen5.github.io/Skatteberegner/

GitHub-repo:
https://github.com/kristianjensen5/Skatteberegner

---

## 1. Skattelettelsesberegner (`index.html`)

Brugeren indtaster sin årsløn før skat, og beregneren viser skattelettelse om året og pr. måned ud fra udspillet, hvor mellemskat og toptopskat afskaffes.

### Beregningslogik

```text
indkomst efter AM-bidrag = årsløn × 0,92
skattelettelse = fjernet mellemskat + fjernet toptopskat
```

Satser og grænser i koden:

```text
AM-bidrag: 8 %
Mellemskat: 7,5 % fra 641.200 kr. til 777.900 kr. efter AM-bidrag
Toptopskat: 5 % over 2.592.700 kr. efter AM-bidrag
Ny topskat: 15 procentpoint over 777.900 kr. efter AM-bidrag
```

Eksempler: 700.000 kr./år → ca. 210 kr./år; 850.000 kr./år → ca. 10.253 kr./år; 3.000.000 kr./år → ca. 18.618 kr./år.

Tekster ligger både i `calculator-texts.json` og som fallback-JSON i `index.html` — de to skal holdes ens, da beregneren falder tilbage til fallback-teksten hvis den eksterne JSON ikke kan hentes i embed-miljøet.

---

## 2. PFA-pensionsberegner (`pfa-pensionsberegner.html`)

Beregner hvor meget ekstra man skal indbetale om måneden for at gå et valgt antal år tidligere på pension uden at få en lavere årlig pension før skat. Formlerne er udtrukket fra `Hvad koster det.xlsx` (PFA's eget regneark) og dokumenteret trin for trin i `PFA-formelaudit.md`, inklusive et kontrolregnestykke der matcher Excel-facit.

### Inputfelter

Fødselsår, nuværende alder, pensionsalder, år tidligere på pension, årlig bruttoløn, eksisterende depot, indbetalingsprocent. Under en fold-ud ("Forudsætninger i beregningen"): forsikringsdækninger, afkast efter PAL, inflation, rente i udbetalingsfase, udbetalingsrater, skatteværdi af frivillig indbetaling — alle med PFA's standardsatser som default, men redigerbare.

### Session 1. juli 2026 — redaktør-feedback rettet

- **Fødselsår-felt tilføjet:** udfylder automatisk den lovfastsatte danske folkepensionsalder (tabel verificeret mod borger.dk + star.dk, se kilder nedenfor), med link til borger.dk hvis man vil dobbelttjekke. Pensionsalder-feltet forbliver redigerbart bagefter.
- **Copy strammet til standalone-brug:** intro nævner ikke længere "PFA-arket" (uforklaret jargon), men "PFA's formler", og forklarer selv formålet uden at kræve en anden artikel som kontekst.
- **Resultat-sektionen sprogligt forenklet:** labels er nu fulde sætninger ("Så meget ekstra skal du indbetale om måneden - før skat" / "Så meget koster det dig reelt om måneden - efter skat"). Den tekniske metatekst ("fordelt på 12 måneder") er fjernet fra resultatkortet og flyttet til "Sådan regner vi".
- **"Skatteværdien" forklaret i klartekst:** både i resultatkortets metatekst og i den udvidede forklaring — det er skattefradraget man får ved at indbetale ekstra til pension.
- **"Sådan regner vi" er nu en fold-ud** (`<details>`), ligesom "Forudsætninger", så siden ikke virker uoverskuelig ved første blik.
- **Bug fundet og rettet under verificering:** ugyldigt eller tomt input i Fødselsår-feltet satte tidligere stille Pensionsalder til 65 i baggrunden (fordi `Number('')` parser til 0). Rettet med et sanity-tjek (år skal være mellem 1900 og indeværende år+1) før auto-udfyldning sker.
- **Fødselsår udfylder nu også "Nuværende alder"** (indeværende år minus fødselsår), ikke kun Pensionsalder — begge felter forbliver redigerbare. Rækkefølgen i formularen er ændret til Fødselsår → Nuværende alder → Pensionsalder, så det er tydeligt at fødselsåret styrer de to andre.
- **`Hvad koster det.xlsx` er lagt i `.gitignore`** efter aftale — kildearket skal kun ligge lokalt, ikke committes til det offentlige repo.
- **Tilpasset til 9:16-iframe ("vibecode element"):** typografi, mellemrum og padding er skruet markant ned og skalerer med bredden (`clamp()`-værdier bundet til `vw`). Feltgrid er 3 kolonner ved alle bredder (var før 2 kolonner under 760px, hvilket gjorde formularen for høj). Kortet er lodret centreret i boksen via `body { display:flex; align-items:center; min-height:100dvh }`.
- **Verificeret med Playwright ved den faktiske 9:16-ratio (bredde × 16/9 = højde):** ingen scroll nødvendig fra 360px bredde og opefter (dækker jeres testede 550-620px rigeligt, samt almindelige mobilbredder). Ved 320px (meget gamle/små skærme) er der stadig 48px overflow — ikke dækket, men sjælden skærmstørrelse i dag.

Kilder brugt til folkepensionsalder-tabellen:
- https://www.borger.dk/Handlingsside?selfserviceId=8557b9eb-947a-48cb-bef2-2f37aa5c9d32
- https://www.borger.dk/pension-og-efterloen/Folkepension-oversigt/foer-du-gaar-paa-folkepension
- https://star.dk/ydelser/pension-og-efterloen/folkepension-tidlig-pension-foertidspension-og-seniorpension/folkepension/folkepensionsalderen-nu-og-fremover/

Verificeret med Playwright (headless browser, ikke kun kodelæsning): alle testede fødselsår gav korrekt pensionsalder, beregningen matcher stadig Excel-facit, og fold-ud/bug-fix er bekræftet i praksis.

### Rettet 2. juli 2026 — dobbelt-embed-sikkerhed

JavaScript'en brugte `document.getElementById(...)` og `document.querySelectorAll(...)` globalt. Rettet til at slå elementer op via `document.currentScript.previousElementSibling` som instansens egen `root`, og alle opslag går nu gennem `root.querySelector(...)`. Testet ved at indlejre to komplette instanser på samme testside (samme mønster som Dronningen-referencen): forskelligt fødselsår, forskellig løn og separate fold-outs i hver instans forblev fuldstændig uafhængige, ingen JS-fejl.

**Vigtig nuance opdaget under testen:** Da beregneren leveres i en iframe (vibecode-elementet), er hver instans allerede isoleret i sin egen `document` — to iframes kan aldrig kollidere med hinanden, uanset denne fix. Rettelsen er stadig værd at have som god praksis (beskytter hvis filen nogensinde genbruges som inline-embed uden iframe), men selve "to vibecode-elementer på samme artikel"-scenariet var aldrig i fare for ID-kollision.

### Rettet 2. juli 2026 — fylder nu boksen bedre ved bredere iframe-mål

Kristian testede i den faktiske CMS-kontekst og opdagede at vibecode-elementet reelt kan blive meget bredere end de 550-620px, der blev testet ved manuel browser-resize (op mod 900px+ bred, altså op mod 1600px høj ved låst 9:16). Ved den bredde fyldte det forrige kompakte design kun ca. 40 % af boksen — resten var tomt luft over/under, fordi skrifter/mellemrum havde et lavt loft (`clamp()`-makс-værdier) der stoppede dem i at vokse med pladsen.

Rettet:
- Feltgrid ændret fra 3 til 2 kolonner (mindre "opremsnings"-følelse, matcher Kristians feedback om at 3 kolonner virkede uoverskueligt).
- Alle `clamp()`-lofter hævet markant, så skrift/mellemrum/padding fortsætter med at vokse med bredden i stedet for at stoppe tidligt. Bunden (minimumsværdierne, der sikrer ingen scroll ved smalle mobilbredder) er urørt.

Resultat, verificeret med Playwright ved faktisk 9:16-højde (bredde × 16/9):

| Bredde | Boks-højde | Kort-højde | Fyldningsgrad |
|---|---|---|---|
| 360px | 640px | 632px | 99 % |
| 480px | 853px | 716px | 84 % |
| 620px | 1102px | 904px | 82 % |
| 900px | 1600px | 1242px | 78 % |

320px (meget sjælden skærmstørrelse) overskrider stadig med ca. 66px — samme kendte, lille edge case som før.

### Rettet 2. juli 2026 — felter flugtede ikke på mobil

Kristian testede på rigtig mobil og fandt at inputboksene i række 2 (Pensionsalder / År tidligere på pension) ikke flugtede vandret. Årsag: "Fødselsår" har en 2-linjers hjælpetekst, "Nuværende alder" har ingen — CSS Grid's standard (`align-items: stretch`) strakte derfor "Nuværende alder"-feltet til samme højde som "Fødselsår", hvilket skubbede næste række skævt. Samme problem fandtes (mindre synligt) mellem "Årlig bruttoløn" og "Eksisterende depot".

Fix: `align-items: start` tilføjet til `.pfa-calc__grid` — hvert felt bruger nu sin egen naturlige højde i stedet for at blive strukket til rækkens højeste nabo. Verificeret med Playwright: alle inputbokse i alle rækker har nu identisk `top`- og `height`-position.

Redaktørens spørgsmål om hvorfor både Fødselsår og Nuværende alder findes som felter blev besvaret og accepteret — de dækker to forskellige ting (lovfastsat pensionsalder-opslag vs. regnestykkets tidshorisont), og indholdet forbliver som det er.

### Kendt opmærksomhedspunkt

Pensionsalderen er kun lovfastsat til og med fødselsår 1971 (70 år). For yngre årgange er 70 år et vejledende loft — Folketinget beslutter først eventuelle forhøjelser senest i 2030. Tabellen bør genbesøges hvis loven ændres.

### Rettet 2. juli 2026 — tekstrettelser fra redaktør

- Intro omskrevet: "PFA's formler" → "tal fra PFA", og sætningen gjort mere præcis om hvad man rent faktisk gør ("indbetale på din pensionsopsparing for at få samme årlige pension før skat") i stedet for det mere upræcise "spare op for at gå tidligere på pension".
- Feltnavnet "Eksisterende depot" ændret til "Nuværende pensionsopsparing" (mere læserforståeligt, mindre finans-jargon). Fejlbeskeden for feltet er rettet til samme term for konsistens.
- Verificeret med Playwright: ny tekst vises korrekt, beregningen er uændret (262,43 kr. / 149,69 kr. matcher stadig Excel-facit), ingen layout-overflow.

---

## Rettet 2. juli 2026 — baggrundsfarve på begge beregnere

Body-baggrunden var uspecificeret (browser-default) i begge filer. Sat til `#fbfaf7` i både `index.html` (Skattelettelsesberegner) og `pfa-pensionsberegner.html` — den ubrugte CSS-variabel `--skb-color-bg-pol-vibe` i `index.html` (var defineret, men aldrig koblet til noget) er samtidig rettet til samme værdi og faktisk koblet til `body`. `pensionsberegner desktop.html` (gammel inspirationsfil, ikke live) er ikke rørt. Verificeret med Playwright: `getComputedStyle(body).backgroundColor` = `rgb(251, 250, 247)` på begge filer.

---

## Filer i repoet

- `index.html` — Skattelettelsesberegner
- `calculator-texts.json` — redaktionelle tekster til Skattelettelsesberegner
- `pfa-pensionsberegner.html` — PFA-pensionsberegner
- `PFA-formelaudit.md` — formelaudit + kontrolregnestykke for PFA-beregneren
- `Hvad koster det.xlsx` — kildearket bag PFA-formlerne (untracked lokalt, ikke committet)
- `pensionsberegner desktop.html` — tidligere pensionsberegner, brugt som inspiration

## Generelle opmærksomhedspunkter

- Ingen af beregnerne medregner kirkeskat, personlige fradrag eller kommuneskatteforskelle.
- CSS-klasser og -variabler er namespacet (`skb-` / `pfa-calc-`) for at undgå konflikt med Politikens styles ved embed.
