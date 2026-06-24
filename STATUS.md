# Status: Skattelettelsesberegner

Opdateret: 24. juni 2026

## Formål

Projektet er en enkel skattelettelsesberegner til en artikel. Brugeren indtaster sin årsløn før skat, og beregneren viser:

- skattelettelse om året
- skattelettelse pr. måned

Live-version:
https://kristianjensen5.github.io/Skatteberegner/

GitHub-repo:
https://github.com/kristianjensen5/Skatteberegner

## Nuværende beregningslogik

Beregneren tager udgangspunkt i, at brugeren indtaster bruttoløn for et helt år før AM-bidrag.

Først omregnes lønnen til indkomst efter AM-bidrag:

```text
indkomst efter AM-bidrag = årsløn × 0,92
```

Derefter beregnes skattelettelsen som summen af:

```text
fjernet mellemskat + fjernet toptopskat
```

Satser og grænser i koden:

```text
AM-bidrag: 8 %
Mellemskat: 7,5 % fra 641.200 kr. til 777.900 kr. efter AM-bidrag
Toptopskat: 5 % over 2.592.700 kr. efter AM-bidrag
Ny topskat: 15 procentpoint over 777.900 kr. efter AM-bidrag
```

Den nye topskat på 15 procentpoint giver ikke i sig selv en skattelettelse, fordi indkomst over 777.900 kr. før udspillet allerede var ramt af 7,5 procent mellemskat og 7,5 procent topskat.

## Eksempler

```text
700.000 kr. i årsløn før skat:
Ca. 210 kr. om året / 18 kr. om måneden

850.000 kr. i årsløn før skat:
Ca. 10.253 kr. om året / 854 kr. om måneden

3.000.000 kr. i årsløn før skat:
Ca. 18.618 kr. om året / 1.551 kr. om måneden
```

Mange lønninger over ca. 845.500 kr. før AM-bidrag giver samme resultat, indtil man rammer toptopskattegrænsen. Det skyldes, at den maksimale lettelse fra mellemskatten er nået.

## Filer

- `index.html`: Selve beregneren med CSS, HTML, fallback-tekster og JavaScript.
- `calculator-texts.json`: Redaktionelle tekster, så copy kan justeres hurtigere.
- `pensionsberegner desktop.html`: Tidligere pensionsberegner brugt som inspiration.

## Tekster

Den aktive tekst ligger både i:

- `calculator-texts.json`
- fallback-JSON i `index.html`

De to skal holdes ens. Det er gjort for at beregneren stadig virker, hvis embedmiljøet ikke kan hente den eksterne JSON-fil.

## Kendte opmærksomhedspunkter

- `fetch('calculator-texts.json')` virker på GitHub Pages, hvor JSON-filen ligger ved siden af `index.html`.
- Hvis beregneren embeddes direkte i CUE/Politiken, kan den relative JSON-hentning fejle afhængigt af embedmiljøet. I så fald bruges fallback-teksten i `index.html`.
- Beregningen ser bort fra kirkeskat, fradrag og individuelle kommuneskatter.
- Kommuneskat bruges kun i forklaringsteksten om marginalskat, ikke i selve lettelsesberegningen.
- CSS-klasser og CSS-variabler er namespacet med `skb-` og `-pol-vibe` for at undgå konflikt med Politikens styles.

## Seneste status

- Toptopskatten er bekræftet afskaffet og indgår i beregningen.
- Det lille "Skat"-mærke over titlen er fjernet, fordi det ikke matchede designet.
- Intro- og inputtekster er strammet op for at undgå gentagelse med artiklens underrubrik.
- Repoet er public, og GitHub Pages er aktiveret.
