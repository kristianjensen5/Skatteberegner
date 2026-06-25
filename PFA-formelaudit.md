# PFA-beregner: formelaudit

Kilde: `Hvad koster det.xlsx`

Udtrukket: 24. juni 2026

## Formål

Excelarket beregner, hvad det kræver i ekstra pensionsindbetaling at gå et valgt antal år tidligere på pension uden at få en lavere årlig pension før skat.

Brugeren kan ændre enkelte input, mens flere antagelser er faste satser. Outputtet viser blandt andet:

- pension ved normal pensionsalder
- pension ved tidligere pension
- fald i årlig pension
- manglende kapital
- nødvendig ekstra indbetaling
- ekstra betaling pr. måned før skat
- nettoeffekt pr. måned efter skat

## Inputfelter

Disse felter ligger i arket som input:

```text
Nuværende alder
Pensionsalder
År tidligere på pension
Årlig bruttoløn
Eksisterende depot
Indbetalingsprocent
Procent til forsikringsdækninger
Afkast efter PAL
Inflation
Rente
Udbetalingsrater
Skatteværdi af frivillig indbetaling
```

I skærmbilledets eksempel er værdierne:

```text
Nuværende alder: 24
Pensionsalder: 70
År tidligere på pension: 1
Årlig bruttoløn: 400.000 kr.
Eksisterende depot: 0 kr.
Indbetalingsprocent: 15 %
Procent til forsikringsdækninger: 18 %
Afkast efter PAL: 5 %
Inflation: 2 %
Rente: 4 %
Udbetalingsrater: 22
Skatteværdi af frivillig indbetaling: 38 %
```

## Formelkæde

Nedenfor er formlerne omsat fra Excel til almindelig notation.

### 1. Realt afkast i opsparing

```text
realtAfkast = (1 + afkastEfterPAL) / (1 + inflation) - 1
```

Excel:

```text
=(1+B13)/(1+B14)-1
```

### 2. Rente i udbetalingsfase

I arket er denne direkte lig med `4 %`.

```text
renteIUdbetalingsfase = rente
```

### 3. År til pension

```text
aarTilPension = pensionsalder - nuvaerendeAlder
```

Excel:

```text
=B7-B6
```

### 4. År til tidlig pension

```text
aarTilTidligPension = aarTilPension - aarTidligerePaaPension
```

Excel:

```text
=E8-B8
```

### 5. Årlig pensionsindbetaling

Arket bruger løn efter AM-bidrag og fratrækker den del af pensionsindbetalingen, der går til forsikringsdækninger.

```text
indbetaling = aarligBruttoloen × 0,92 × indbetalingsprocent × (1 - procentTilForsikringsdaekninger)
```

Excel:

```text
=B9*0.92*B11*(1-B12)
```

### 6. Opsparingsfaktor

Opsparingsfaktoren svarer til fremtidsværdien af en årlig indbetaling over perioden.

```text
opsparingsfaktor = ((1 + realtAfkast)^antalAar - 1) / realtAfkast
```

Excel normal:

```text
=((1+E6)^E8-1)/E6
```

Excel tidlig:

```text
=((1+E6)^E9-1)/E6
```

### 7. Slutdepot

Eksisterende depot fremskrives med realt afkast, og de årlige indbetalinger lægges oveni via opsparingsfaktoren.

```text
slutdepot = eksisterendeDepot × (1 + realtAfkast)^antalAar + indbetaling × opsparingsfaktor
```

Excel normal:

```text
=B10*(1+E6)^E8+E10*E11
```

Excel tidlig:

```text
=B10*(1+E6)^E9+E10*E12
```

### 8. Ratepassiv

Arket bruger denne formel til at omsætte depot til årlig pension:

```text
ratepassiv = (1 - 1 / (1 + rente)^udbetalingsAar) / ln(1 + rente)
```

Excel normal:

```text
=(1-(1/POWER((1+B15),B16)))/LN(1+B15)
```

Excel tidlig:

```text
=(1-(1/POWER((1+B15),B16+B8)))/LN(1+B15)
```

Bemærk: Ved tidlig pension forlænges udbetalingsperioden med antallet af år, man går tidligere på pension.

### 9. Årlig pension før skat

```text
pensionFoerSkat = slutdepot / ratepassiv
```

Excel normal:

```text
=E13/E15
```

Excel tidlig:

```text
=E14/E16
```

### 10. Fald i årlig pension før skat

```text
faldIAarligPension = pensionFoerSkatTidlig - pensionFoerSkatNormal
```

Excel:

```text
=E18-E17
```

Dette tal er negativt, hvis tidlig pension giver lavere årlig pension.

### 11. Måldepot ved tidlig pension

Målet er at have nok depot ved tidlig pension til stadig at få samme årlige pension som ved normal pension.

```text
maalDepotTidlig = pensionFoerSkatNormal × ratepassivTidlig
```

Excel:

```text
=E17*E16
```

### 12. Manglende kapital

```text
manglendeKapital = max(0, maalDepotTidlig - slutdepotTidlig)
```

Excel:

```text
=MAX(0,E20-E14)
```

### 13. Nødvendig ekstra årlig indbetaling

Den manglende kapital fordeles baglæns over opsparingsperioden frem til tidlig pension.

```text
noedvendigEkstraAarligIndbetaling = manglendeKapital / opsparingsfaktorTidlig
```

Excel:

```text
=E21/E12
```

### 14. Ekstra indbetaling i procent af løn

```text
ekstraIndbetalingPctAfLoen = noedvendigEkstraAarligIndbetaling / aarligBruttoloen
```

Excel:

```text
=E22/B9
```

### 15. Ekstra pr. måned før skat

```text
ekstraPrMaanedFoerSkat = noedvendigEkstraAarligIndbetaling / 12
```

Excel:

```text
=E22/12
```

### 16. Nettoeffekt pr. måned efter skat

Excelarket beregner nettoeffekten sådan:

```text
nettoeffektPrMaanedEfterSkat =
  aarligBruttoloen / 12 × 0,92 × ekstraIndbetalingPctAfLoen × (1 - skattevaerdiAfFrivilligIndbetaling)
```

Excel:

```text
=B9/12*0.92*E23*(1-B17)
```

Da `ekstraIndbetalingPctAfLoen = noedvendigEkstraAarligIndbetaling / aarligBruttoloen`, kan samme formel forenkles til:

```text
nettoeffektPrMaanedEfterSkat =
  noedvendigEkstraAarligIndbetaling / 12 × 0,92 × (1 - skattevaerdiAfFrivilligIndbetaling)
```

## JavaScript-version

```js
function calculateEarlyRetirementCost(input) {
  const currentAge = input.currentAge;
  const retirementAge = input.retirementAge;
  const yearsEarly = input.yearsEarly;
  const annualGrossSalary = input.annualGrossSalary;
  const existingDepot = input.existingDepot;
  const contributionRate = input.contributionRate;
  const insuranceCoverageRate = input.insuranceCoverageRate;
  const returnAfterPal = input.returnAfterPal;
  const inflation = input.inflation;
  const payoutInterestRate = input.payoutInterestRate;
  const payoutYears = input.payoutYears;
  const voluntaryContributionTaxValue = input.voluntaryContributionTaxValue;

  const realSavingsReturn = (1 + returnAfterPal) / (1 + inflation) - 1;
  const yearsToRetirement = retirementAge - currentAge;
  const yearsToEarlyRetirement = yearsToRetirement - yearsEarly;
  const annualContribution =
    annualGrossSalary * 0.92 * contributionRate * (1 - insuranceCoverageRate);

  const savingsFactor = (years) => {
    if (realSavingsReturn === 0) return years;
    return ((1 + realSavingsReturn) ** years - 1) / realSavingsReturn;
  };

  const normalSavingsFactor = savingsFactor(yearsToRetirement);
  const earlySavingsFactor = savingsFactor(yearsToEarlyRetirement);

  const normalFinalDepot =
    existingDepot * (1 + realSavingsReturn) ** yearsToRetirement +
    annualContribution * normalSavingsFactor;

  const earlyFinalDepot =
    existingDepot * (1 + realSavingsReturn) ** yearsToEarlyRetirement +
    annualContribution * earlySavingsFactor;

  const ratePassive = (years) => {
    if (payoutInterestRate === 0) return years;
    return (1 - 1 / (1 + payoutInterestRate) ** years) / Math.log(1 + payoutInterestRate);
  };

  const normalRatePassive = ratePassive(payoutYears);
  const earlyRatePassive = ratePassive(payoutYears + yearsEarly);

  const normalAnnualPensionBeforeTax = normalFinalDepot / normalRatePassive;
  const earlyAnnualPensionBeforeTax = earlyFinalDepot / earlyRatePassive;
  const annualPensionDropBeforeTax =
    earlyAnnualPensionBeforeTax - normalAnnualPensionBeforeTax;

  const targetEarlyDepot = normalAnnualPensionBeforeTax * earlyRatePassive;
  const missingCapital = Math.max(0, targetEarlyDepot - earlyFinalDepot);
  const requiredExtraAnnualContribution = missingCapital / earlySavingsFactor;
  const extraContributionPctOfSalary =
    requiredExtraAnnualContribution / annualGrossSalary;
  const extraPerMonthBeforeTax = requiredExtraAnnualContribution / 12;
  const netEffectPerMonthAfterTax =
    extraPerMonthBeforeTax * 0.92 * (1 - voluntaryContributionTaxValue);

  return {
    realSavingsReturn,
    yearsToRetirement,
    yearsToEarlyRetirement,
    annualContribution,
    normalSavingsFactor,
    earlySavingsFactor,
    normalFinalDepot,
    earlyFinalDepot,
    normalRatePassive,
    earlyRatePassive,
    normalAnnualPensionBeforeTax,
    earlyAnnualPensionBeforeTax,
    annualPensionDropBeforeTax,
    targetEarlyDepot,
    missingCapital,
    requiredExtraAnnualContribution,
    extraContributionPctOfSalary,
    extraPerMonthBeforeTax,
    netEffectPerMonthAfterTax
  };
}
```

## Kontrol mod Excel-eksempel

Input:

```js
{
  currentAge: 24,
  retirementAge: 70,
  yearsEarly: 1,
  annualGrossSalary: 400000,
  existingDepot: 0,
  contributionRate: 0.15,
  insuranceCoverageRate: 0.18,
  returnAfterPal: 0.05,
  inflation: 0.02,
  payoutInterestRate: 0.04,
  payoutYears: 22,
  voluntaryContributionTaxValue: 0.38
}
```

Forventet output fra Excel:

```text
Realt afkast i opsparing: 2,941176 %
År til pension: 46
År til tidlig pension: 45
Indbetaling: 45.264 kr.
Slutdepot normal: 4.299.932,95 kr.
Slutdepot tidlig: 4.133.106,98 kr.
Pension før skat normal: 291.753,32 kr.
Pension før skat tidlig: 272.775,68 kr.
Fald i årlig pension før skat: -18.977,64 kr.
Måldepot tidlig: 4.420.656,92 kr.
Manglende kapital: 287.549,94 kr.
Nødvendig ekstra indbetaling: 3.149,12 kr.
Ekstra indbetaling i pct. af løn: 0,79 %
Ekstra pr. måned før skat: 262,43 kr.
Nettoeffekt pr. måned efter skat: 149,69 kr.
```

## Mulige input til en webberegner

Til en læserrettet beregner bør vi sandsynligvis kun bede om:

```text
Nuværende alder
Ønsket pensionsalder eller nuværende pensionsalder
Hvor mange år tidligere man vil gå på pension
Årlig bruttoløn
Eksisterende pensionsdepot
Indbetalingsprocent
```

Resten kan ligge som faste antagelser, men bør vises i "Sådan regner vi":

```text
18 % af pensionsindbetaling går til forsikringsdækninger
5 % afkast efter PAL
2 % inflation
4 % rente i udbetalingsfase
22 års udbetalingsperiode
38 % skatteværdi af frivillig indbetaling
```

