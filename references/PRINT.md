# Print Typography for German Text

Best practices for print-specific typography when generating PDFs from web content using CSS `@media print`.

## Widows & Orphans (Hurenkind & Schusterjunge)

German typography has specific terms for these common print problems:

| German Term    | English    | Definition                                      |
| -------------- | ---------- | ----------------------------------------------- |
| Schusterjunge  | Orphan     | First line of paragraph alone at page bottom    |
| Hurenkind      | Widow      | Last line of paragraph alone at page top        |

Both are considered typographic errors in professional German print.

### CSS Control

```css
@media print {
  p, li, dd {
    orphans: 3;  /* Minimum lines at page bottom */
    widows: 3;   /* Minimum lines at page top */
  }

  h1, h2, h3, h4, h5, h6 {
    break-after: avoid;  /* Keep headings with following content */
  }
}
```

**Recommended values**: At least 2 lines, preferably 3 for body text.

## Hyphenation (Silbentrennung)

### Basic Setup

```css
html {
  lang: de;  /* Set in HTML: <html lang="de"> */
}

@media print {
  p, li, dd, td {
    hyphens: auto;
    hyphenate-limit-chars: 6 3 2;  /* word min, before break, after break */
  }
}
```

**Important**: The `lang="de"` attribute on the `<html>` element is required for correct German hyphenation.

### German Hyphenation Rules (Duden)

Key principles:

1. **Syllable-based**: Break at syllable boundaries (Sil-ben-tren-nung)
2. **Single consonants**: Go to next syllable (tre-ten, not tret-en)
3. **Multiple consonants**: Last consonant goes to next syllable (Wes-pe, Kas-ten)
4. **ck becomes k-k**: Historically Bäk-ker, modern browsers handle this
5. **st stays together**: Fens-ter, not Fen-ster
6. **Compound words**: Break at word boundaries first (Bundes-tag, not Bun-destag)

### Problem Cases

| Word Type       | Issue                          | Solution                     |
| --------------- | ------------------------------ | ---------------------------- |
| Foreign words   | May use origin language rules  | Use soft hyphen for control  |
| Compound words  | Browser may miss boundaries    | Use soft hyphen at joints    |
| Technical terms | Incorrect automatic breaks     | Manual soft hyphens          |
| Proper nouns    | Should not be hyphenated       | `hyphens: none` on element   |

### Soft Hyphen for Manual Control

Use `&shy;` (U+00AD) to suggest break points:

```html
<p>Donau&shy;dampf&shy;schiff&shy;fahrts&shy;gesellschaft</p>
```

The hyphen only appears if the browser breaks at that point.

### Fine-Tuning

```css
@media print {
  p {
    hyphenate-limit-chars: 6 3 2;  /* Min word: 6, before: 3, after: 2 */
    hyphenate-limit-lines: 2;      /* Max consecutive hyphenated lines */
    hyphenate-limit-last: always;  /* Avoid hyphen on last line */
    hyphenate-limit-zone: 8%;      /* Hyphenation zone from margin */
  }
}
```

**Note**: Not all properties are supported in all browsers. Test thoroughly.

## A4 Page Setup

### Basic @page Rule

```css
@page {
  size: A4;  /* 210mm × 297mm */
  margin: 25mm 20mm 25mm 25mm;  /* top right bottom left */
}
```

### German Standard Margins (DIN 5008)

For business documents:

| Margin   | Value | Notes                            |
| -------- | ----- | -------------------------------- |
| Top      | 27mm  | Space for letterhead             |
| Right    | 20mm  | Standard                         |
| Bottom   | 25mm  | Space for page numbers           |
| Left     | 25mm  | Binding margin                   |

For general documents, 20–25mm on all sides is acceptable.

### Safe Print Area

Account for printer limitations (typically 5mm unprintable margin):

```css
@page {
  size: A4;
  margin: 20mm;

  @top-center {
    content: element(header);
  }

  @bottom-center {
    content: counter(page) " / " counter(pages);
  }
}
```

## Page Break Control

### Preventing Breaks

```css
@media print {
  /* Keep headings with following content */
  h1, h2, h3, h4, h5, h6 {
    break-after: avoid;
  }

  /* Prevent breaks inside elements */
  figure, table, blockquote, pre {
    break-inside: avoid;
  }

  /* Keep captions with their figures */
  figcaption {
    break-before: avoid;
  }
}
```

### Forcing Breaks

```css
@media print {
  .page-break-before {
    break-before: page;
  }

  .page-break-after {
    break-after: page;
  }

  /* Start chapters on right page (recto) */
  .chapter {
    break-before: right;
  }
}
```

### Table Handling

```css
@media print {
  table {
    break-inside: avoid;
  }

  /* For long tables that must break */
  thead {
    display: table-header-group;  /* Repeat header on each page */
  }

  tfoot {
    display: table-footer-group;
  }

  tr {
    break-inside: avoid;
  }
}
```

## Print-Specific CSS

### Basic Print Stylesheet

```css
@media print {
  /* Hide non-print elements */
  nav, aside, .no-print, button, .interactive {
    display: none !important;
  }

  /* Reset backgrounds and colors for printing */
  body {
    background: white !important;
    color: black !important;
  }

  /* Ensure links are visible */
  a[href]::after {
    content: " (" attr(href) ")";
    font-size: 0.8em;
  }

  /* Don't expand internal links */
  a[href^="#"]::after,
  a[href^="javascript"]::after {
    content: none;
  }
}
```

### Font Considerations

#### Prefer Web Fonts

Web fonts provide consistent typography across screen and print. When properly configured, they embed into PDFs.

```css
/* Load font with print-friendly settings */
@font-face {
  font-family: "Source Serif Pro";
  src: url("/fonts/source-serif-pro.woff2") format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@media print {
  body {
    font-family: "Source Serif Pro", Georgia, serif;
    font-size: 11pt;
    line-height: 1.4;
  }

  h1 { font-size: 18pt; }
  h2 { font-size: 14pt; }
  h3 { font-size: 12pt; }
}
```

#### Self-Hosting vs CDN

| Approach     | PDF Embedding | Recommendation                              |
| ------------ | ------------- | ------------------------------------------- |
| Self-hosted  | Reliable      | Preferred for print — fonts always available |
| Google Fonts | May fail      | CDN may be blocked; font not embedded       |
| Adobe Fonts  | Often fails   | Requires JavaScript, problematic for print  |

**Recommendation**: Self-host fonts for reliable PDF generation.

#### Font Licenses for PDF Embedding

Not all font licenses permit embedding in PDFs. Check before using:

| License Type         | PDF Embedding | Examples                                    |
| -------------------- | ------------- | ------------------------------------------- |
| OFL (Open Font)      | Allowed       | Source Serif, Noto, Fira, IBM Plex          |
| Apache 2.0           | Allowed       | Roboto, Open Sans                           |
| Commercial (varies)  | Check license | Often requires "desktop" or "ePub" license  |
| Google Fonts         | Allowed       | All Google Fonts are OFL or Apache          |
| Adobe Fonts          | Restricted    | Embedding typically not permitted           |

**Safe choices for German text**:
- **Serif**: Source Serif Pro, Noto Serif, Crimson Pro, EB Garamond
- **Sans-serif**: Source Sans Pro, Noto Sans, Fira Sans, IBM Plex Sans

These fonts have good German character support (umlauts, Eszett, quotation marks).

#### Fallback Strategy

Always provide system font fallbacks:

```css
@media print {
  body {
    font-family:
      "Your Web Font",
      Georgia,           /* macOS/Windows fallback */
      "Times New Roman", /* Universal fallback */
      serif;
  }
}
```

### Forcing Background Printing

```css
@media print {
  .print-background {
    print-color-adjust: exact;           /* Standard */
    -webkit-print-color-adjust: exact;   /* WebKit */
  }
}
```

## Color for Print

### RGB vs CMYK

Browsers work in RGB. Printed output uses CMYK. Some colors shift:

| RGB Color       | Print Result              | Recommendation               |
| --------------- | ------------------------- | ---------------------------- |
| Pure blue #0000FF | Appears purple-ish      | Use darker, less saturated   |
| Bright green    | Looks different           | Test on actual printer       |
| Neon colors     | Cannot be reproduced      | Avoid entirely               |

### Safe Print Colors

```css
:root {
  /* Colors that reproduce reliably */
  --print-black: #000000;
  --print-dark-gray: #333333;
  --print-gray: #666666;
  --print-light-gray: #999999;

  /* Muted colors work better than saturated */
  --print-blue: #2c5aa0;
  --print-red: #a02c2c;
  --print-green: #2c7a2c;
}
```

### Black Text

For body text, use pure black (`#000000` or `rgb(0,0,0)`), which prints as 100% K (black ink only).

Avoid "rich black" (`#000000` with CMYK values like 60,40,40,100) for text—it causes registration issues and blurring on small type.

### Ensuring Colors Print

```css
@media print {
  .brand-color {
    color: #2c5aa0 !important;
    print-color-adjust: exact;
    -webkit-print-color-adjust: exact;
  }
}
```

## Quick Reference

| Property                  | Value                   | Purpose                        |
| ------------------------- | ----------------------- | ------------------------------ |
| `orphans`                 | `3`                     | Min lines at page bottom       |
| `widows`                  | `3`                     | Min lines at page top          |
| `hyphens`                 | `auto`                  | Enable automatic hyphenation   |
| `break-after`             | `avoid`                 | Keep with following content    |
| `break-inside`            | `avoid`                 | Prevent breaks inside element  |
| `break-before`            | `page`                  | Force page break before        |
| `print-color-adjust`      | `exact`                 | Print backgrounds/colors       |
| `size`                    | `A4`                    | Page dimensions                |

## Complete Print Stylesheet Example

```css
@page {
  size: A4;
  margin: 25mm 20mm;
}

@media print {
  html {
    font-size: 11pt;
  }

  body {
    font-family: Georgia, serif;
    line-height: 1.5;
    color: black;
    background: white;
  }

  p, li, dd {
    orphans: 3;
    widows: 3;
    hyphens: auto;
  }

  h1, h2, h3, h4, h5, h6 {
    break-after: avoid;
    hyphens: none;
  }

  figure, table, blockquote {
    break-inside: avoid;
  }

  nav, aside, button, .no-print {
    display: none !important;
  }

  a[href^="http"]::after {
    content: " (" attr(href) ")";
    font-size: 0.8em;
    color: #666;
  }
}
```

## Testing Print Output with Playwright

Use Playwright to automate PDF generation and verify print styles.

### Basic PDF Generation

```typescript
import { test } from "@playwright/test";

test("generate PDF", async ({ page }) => {
  await page.goto("http://localhost:3000/document");

  // Wait for web fonts to load
  await page.waitForFunction(() => document.fonts.ready);

  await page.pdf({
    path: "output.pdf",
    format: "A4",
    printBackground: true,
    margin: {
      top: "25mm",
      right: "20mm",
      bottom: "25mm",
      left: "25mm",
    },
  });
});
```

### Testing Print Styles

```typescript
import { test, expect } from "@playwright/test";

test("print styles applied correctly", async ({ page }) => {
  await page.goto("http://localhost:3000/document");

  // Emulate print media
  await page.emulateMedia({ media: "print" });

  // Verify navigation is hidden
  const nav = page.locator("nav");
  await expect(nav).toBeHidden();

  // Verify font family in print mode
  const body = page.locator("body");
  const fontFamily = await body.evaluate(
    (el) => getComputedStyle(el).fontFamily
  );
  expect(fontFamily).toContain("Source Serif");
});
```

### Visual Regression Testing

```typescript
import { test, expect } from "@playwright/test";

test("print layout visual test", async ({ page }) => {
  await page.goto("http://localhost:3000/document");
  await page.emulateMedia({ media: "print" });
  await page.waitForFunction(() => document.fonts.ready);

  // Set viewport to A4 proportions (scaled)
  await page.setViewportSize({ width: 794, height: 1123 }); // A4 at 96 DPI

  await expect(page).toHaveScreenshot("print-layout.png", {
    fullPage: true,
  });
});
```

### PDF Comparison Test

```typescript
import { test } from "@playwright/test";
import { comparePdfs } from "./pdf-utils"; // Custom utility

test("PDF matches baseline", async ({ page }) => {
  await page.goto("http://localhost:3000/document");
  await page.waitForFunction(() => document.fonts.ready);

  const pdfBuffer = await page.pdf({
    format: "A4",
    printBackground: true,
  });

  // Compare with baseline (implement based on your needs)
  const match = await comparePdfs(pdfBuffer, "baseline.pdf");
  expect(match).toBe(true);
});
```

### Playwright Configuration for Print Tests

```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  use: {
    baseURL: "http://localhost:3000",
  },
  projects: [
    {
      name: "print-tests",
      use: {
        // Chromium has best print support
        browserName: "chromium",
        // Ensure fonts are loaded
        launchOptions: {
          args: ["--font-render-hinting=none"],
        },
      },
    },
  ],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

### Testing Checklist

Use these checks to verify print output:

- [ ] Web fonts load and embed correctly
- [ ] Navigation and interactive elements hidden
- [ ] Correct page size (A4: 210mm × 297mm)
- [ ] Margins match specification
- [ ] No orphans or widows (check multi-page documents)
- [ ] Hyphenation works for German text
- [ ] Page breaks occur at appropriate locations
- [ ] Tables don't break mid-row
- [ ] Headings stay with following content
- [ ] Colors print as expected (if using `printBackground: true`)
