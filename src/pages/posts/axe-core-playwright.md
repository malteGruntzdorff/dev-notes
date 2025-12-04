---
layout: ../../layouts/PostLayout.astro
title: Automatisierte Tests mit axe-core/playwright
date: 2025-12-04
tags: ["a11y", "playwright"]
---

## Was ist axe-core?

- Open-Source a11y-Engine
- reines JavaScript
- analysiert das DOM (dadurch an kein Framework gebunden)
- funktioniert mit reinem HTML, React, Angular, Vue etc.
- bewertet nur HTML + ARIA-Attribute (keine manuellen Tastatur- oder Screenreader-Tests)


## Wie setze ich axe-core/playwright ein?
### Installation 
```bash 
npm i @axe-core/playwright 
```
### Ganze Seite testen
```ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("Beliebige-Seite Page ist a11y-konform", async ({ page }) => {
  await page.goto("https://beliebige-seite.de");

  const accessibilityScanResults = await new AxeBuilder({ page }).analyze();

  expect(accessibilityScanResults.violations).toEqual([]);
});
```
### Einzelne Komponenten Testen - include()
```ts
  const results = await new AxeBuilder({ page })
    .include('[data-testid="storefinder-offcanvas"]')
    .analyze();

  expect(results.violations).toEqual([]);
});
```
### Bereiche ausschließen - exclude()
```ts
const results = await new AxeBuilder({ page })
  .exclude("footer")
  .exclude('[data-testid="legacy-banner"]')
  .analyze();
```

### Nur bestimmte Regeln prüfen - withTags()
```ts
const results = await new AxeBuilder({ page })
  .withTags(["wcag2a", "wcag2aa"])
  .analyze();

expect(results.violations).toEqual([]);
```
`axe-core` gruppiert Regeln unter anderem nach WCAG-Leveln. Mit `withTags()` kannst du einschränken, was geprüft werden soll – zum Beispiel nur WCAG 2 A und AA.
Typische Tags sind z. B.:
-   `wcag2a`
-   `wcag2aa`
-   `wcag21a`
-   `wcag21aa`
-   `best-practice`


### Bestimmte Regeln nicht berücksichtigen - disableRoules

```ts
const results = await new AxeBuilder({ page })
  .disableRules(["color-contrast", "region"])
  .analyze();

expect(results.violations).toEqual([]);
```
Sollte niemals eine dauerhafte Lösung sein.
Regeln sind unter anderem:
- `color-contrast`
- `duplicate-id`
- `region`
- `landmark-one-main`
- `page-has-heading-one`
- `html-has-lang`
- `document-title`
- `meta-viewport`


### Nur Fehler bestimmter Schweregrade prüfen - violations.impact
```ts
  const results = await new AxeBuilder({ page }).analyze();

  const blocking = results.violations.filter(v => 
    v.impact === "critical" || v.impact === "serious"
  );

  if (blocking.length > 0) {
    console.log("Blocking accessibility issues:", JSON.stringify(blocking, null, 2));
  }

  expect(blocking).toEqual([]);
```
