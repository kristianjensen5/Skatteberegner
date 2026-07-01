# Status: Skatteberegner (repo)

Opdateret: 1. juli 2026

## Formål

Repoet indeholder to selvstændige beregnere til artikler:

1. **Skattelettelsesberegner** (`index.html`) — færdig, deployet.
2. **PFA-pensionsberegner** (`pfa-pensionsberegner.html`) — "Hvad koster det at gå tidligere på pension?". Under udvikling, endnu ikke linket separat i en artikel.

Begge er self-contained HTML-filer (CSS + JS inline) og deployes via GitHub Pages fra samme repo.

Live-version (repo-rod):
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

Kilder brugt til folkepensionsalder-tabellen:
- https://www.borger.dk/Handlingsside?selfserviceId=8557b9eb-947a-48cb-bef2-2f37aa5c9d32
- https://www.borger.dk/pension-og-efterloen/Folkepension-oversigt/foer-du-gaar-paa-folkepension
- https://star.dk/ydelser/pension-og-efterloen/folkepension-tidlig-pension-foertidspension-og-seniorpension/folkepension/folkepensionsalderen-nu-og-fremover/

Verificeret med Playwright (headless browser, ikke kun kodelæsning): alle testede fødselsår gav korrekt pensionsalder, beregningen matcher stadig Excel-facit, og fold-ud/bug-fix er bekræftet i praksis.

### Kendt opmærksomhedspunkt

Pensionsalderen er kun lovfastsat til og med fødselsår 1971 (70 år). For yngre årgange er 70 år et vejledende loft — Folketinget beslutter først eventuelle forhøjelser senest i 2030. Tabellen bør genbesøges hvis loven ændres.

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
