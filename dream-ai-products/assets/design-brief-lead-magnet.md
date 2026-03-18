# Design Brief — Lead Magnet PDF (Playbook)

> **Designer:** PIXEL  
> **Project:** Dream AI Niche Playbook PDFs  
> **Variants:** Real Estate, MedSpa, HVAC, Barbershop  
> **Format:** Digital PDF (screen-optimized + print-ready)

---

## 1. Document Specifications

| Spec | Value |
|------|-------|
| Page size | US Letter (8.5" × 11") / A4 (210 × 297mm) |
| Orientation | Portrait |
| Margins | 0.75" (19mm) all sides |
| Bleed | 0.125" (3mm) for print version |
| Color space | RGB for digital, CMYK conversion for print |
| Resolution | 300 DPI |
| Page count | 12–16 pages (cover + TOC + 10–14 content pages) |
| File format | PDF/A-1b (archival), sRGB color profile |

### Page Grid

```
┌─────────────────────────────────┐
│ ← 0.75" →                      │
│  ┌───────────────────────────┐  │  ↑ 0.75"
│  │                           │  │
│  │    Content Area           │  │
│  │    7.0" × 9.5"           │  │
│  │    (177.8 × 241.3mm)     │  │
│  │                           │  │
│  │    Columns: 1 or 2       │  │  ↓ 0.75"
│  │    Gutter: 0.25" (6mm)   │  │
│  └───────────────────────────┘  │
│                     ← 0.75" →   │
└─────────────────────────────────┘
```

- **Base grid:** 12-column, 0.5" (12.7mm) column width, 0.25" gutter
- **Baseline grid:** 14pt leading (matches body text)
- **Module height:** 0.5" (5 vertical modules per page body)

---

## 2. Color System

### Brand Core Colors (all niches)

| Token | Hex | Usage |
|-------|-----|-------|
| `--brand-primary` | `#0A0F1C` | Deep navy — headings, primary text |
| `--brand-secondary` | `#1B2744` | Dark blue — subheadings, card fills |
| `--brand-accent` | `#4F8CFF` | Electric blue — CTAs, links, highlights |
| `--brand-surface` | `#F5F7FA` | Light gray — page backgrounds, cards |
| `--brand-white` | `#FFFFFF` | Card backgrounds, text on dark |
| `--brand-text` | `#2D3748` | Body text |
| `--brand-muted` | `#718096` | Captions, secondary text |
| `--brand-border` | `#E2E8F0` | Dividers, card borders |
| `--brand-success` | `#48BB78` | Checkmarks, positive stats |
| `--brand-warning` | `#F6AD55` | Caution, risk indicators |
| `--brand-danger` | `#FC8181` | "Problem" stats, loss callouts |

### Niche Accent Colors

| Niche | Accent | Hex | Light tint (bg) |
|-------|--------|-----|-----------------|
| **Real Estate** | Gold | `#D4A843` | `#FDF8ED` |
| **MedSpa** | Rose | `#D15B8F` | `#FDF0F5` |
| **HVAC** | Amber | `#E07B39` | `#FEF5EC` |
| **Barbershop** | Teal | `#2CB5A0` | `#EDFAF7` |

### Gradient Definitions

```css
--gradient-cover: linear-gradient(135deg, var(--brand-primary) 0%, var(--brand-secondary) 60%, var(--niche-accent) 100%);
--gradient-accent-bar: linear-gradient(90deg, var(--brand-accent) 0%, var(--niche-accent) 100%);
--gradient-surface: linear-gradient(180deg, var(--brand-surface) 0%, #FFFFFF 100%);
```

---

## 3. Typography

### Font Stack (embed via Google Fonts in production)

| Role | Font | Weight | Size | Line Height | Tracking |
|------|------|--------|------|-------------|----------|
| **Cover Title** | Inter | 800 (ExtraBold) | 36–42pt | 1.15 | -0.02em |
| **H1** | Inter | 700 (Bold) | 28pt | 1.20 | -0.01em |
| **H2** | Inter | 600 (SemiBold) | 20pt | 1.25 | 0 |
| **H3** | Inter | 600 (SemiBold) | 14pt | 1.30 | 0 |
| **Body** | Inter | 400 (Regular) | 11pt | 1.55 | 0 |
| **Body Bold** | Inter | 600 (SemiBold) | 11pt | 1.55 | 0 |
| **Caption** | Inter | 400 (Regular) | 9pt | 1.45 | 0.01em |
| **Stat Number** | Inter | 800 (ExtraBold) | 28pt | 1.0 | -0.03em |
| **Pull Quote** | Inter | 500 (Medium) | 14pt | 1.40 | 0 |
| **Button/CTA** | Inter | 600 (SemiBold) | 11pt | 1.0 | 0.04em (uppercase) |

### Fallback Stack
```
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif;
```

---

## 4. Cover Page Template

```
┌─────────────────────────────────────┐
│ [Niche accent bar — 8pt height]     │
│                                     │
│  ┌──────────────────────────────┐   │
│  │                              │   │
│  │     [ICON — niche-specific]  │   │
│  │     64px × 64px, accent tint │   │
│  │                              │   │
│  │   THE [NICHE]               │   │
│  │   AI PLAYBOOK               │   │
│  │   ──────────                │   │
│  │   [Catchy subtitle]         │   │
│  │                              │   │
│  │   How to automate [pain      │   │
│  │   point] and [benefit]       │   │
│  │   with AI employees          │   │
│  │                              │   │
│  └──────────────────────────────┘   │
│                                     │
│  [Geometric accent — bottom 20%]    │
│  gradient fill with niche color     │
│                                     │
│       dreamai.com                   │
└─────────────────────────────────────┘
```

### Cover Specs

| Element | Spec |
|---------|------|
| Background | `--gradient-cover` |
| Icon container | 80×80px rounded square (border-radius: 16px), rgba(255,255,255,0.12) fill |
| Title | 42pt, `--brand-white`, weight 800, max 3 lines |
| Subtitle | 16pt, `rgba(255,255,255,0.75)`, weight 400 |
| Accent bar | 8px height, `--gradient-accent-bar`, full width |
| Bottom shape | Abstract diagonal or wave, niche color at 40% opacity |
| Logo | Bottom center, white, 24px height, 20% opacity watermark |
| Spacing | Title block centered vertically, offset 10% above center |

---

## 5. Interior Page Templates

### Template A: Problem/Stat Page

```
┌─────────────────────────────────────┐
│ [Page header — small, top-right]    │
│ Chapter title (H1, top-left)        │
│                                     │
│  ┌────────┐  ┌────────┐            │
│  │  78%   │  │  15hrs │            │
│  │ STAT   │  │ STAT   │            │
│  └────────┘  └────────┘            │
│  ┌────────┐  ┌────────┐            │
│  │  43%   │  │  $100K │            │
│  │ STAT   │  │ STAT   │            │
│  └────────┘  └────────┘            │
│                                     │
│  2–3 paragraph body text            │
│  explaining the problem             │
│                                     │
│  [Pull quote in accent tint bg]     │
│  "The stat that changes everything" │
│                                     │
│  [Callout box — niche accent left   │
│   border, light tint bg]            │
│  Key insight or warning             │
└─────────────────────────────────────┘
```

**Stat Card Specs:**
- Grid: 2×2, gap 16pt
- Card: 140pt × 100pt, `--brand-white` bg, 2px `--brand-border` border, 12px border-radius
- Number: 28pt, `--brand-danger` or `--niche-accent`, weight 800
- Label: 9pt, `--brand-muted`, uppercase, 0.04em letter-spacing
- Card padding: 16pt

**Pull Quote Specs:**
- Left border: 4px solid `--niche-accent`
- Background: `var(--niche-light-tint)`
- Padding: 16pt 20pt
- Border-radius: 0 8px 8px 0
- Text: 14pt, `--brand-primary`, weight 500, italic optional

**Callout Box:**
- Left border: 4px solid `--brand-accent`
- Background: `rgba(79, 140, 255, 0.06)`
- Padding: 14pt 18pt
- Border-radius: 0 8px 8px 0
- Icon: 16px info/warning icon, left of text

---

### Template B: Solution/Feature Page

```
┌─────────────────────────────────────┐
│ H1: Solution Title                  │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ [ICON]  Feature Name         │   │
│  │         ─────────            │   │
│  │         Description text     │   │
│  │         (3–4 lines max)      │   │
│  │                              │   │
│  │  ✓ Benefit 1                 │   │
│  │  ✓ Benefit 2                 │   │
│  │  ✓ Benefit 3                 │   │
│  └──────────────────────────────┘   │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ [ICON]  Feature 2            │   │
│  │ ...                          │   │
│  └──────────────────────────────┘   │
│                                     │
│  ┌──────────────────────────────┐   │
│  │ [ICON]  Feature 3            │   │
│  │ ...                          │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

**Feature Card Specs:**
- Full width (7.0")
- Background: `--brand-white`
- Border: 1px solid `--brand-border`
- Border-radius: 12px
- Padding: 20pt
- Margin-bottom: 12pt
- Icon: 40×40px, `--brand-accent` circle bg, white icon
- Feature name: 14pt, `--brand-primary`, weight 600
- Checkmarks: `--brand-success` color, 12pt, 8pt gap from text
- Hover state (digital): box-shadow 0 4px 12px rgba(0,0,0,0.08)

---

### Template C: Step-by-Step / How It Works

```
┌─────────────────────────────────────┐
│ H1: How It Works                    │
│                                     │
│  ┌──┐                               │
│  │01│─── Step Title                 │
│  └──┘    Description paragraph      │
│                                     │
│  ┌──┐                               │
│  │02│─── Step Title                 │
│  └──┘    Description paragraph      │
│                                     │
│  ┌──┐                               │
│  │03│─── Step Title                 │
│  └──┘    Description paragraph      │
│                                     │
│  ┌──┐                               │
│  │04│─── Step Title                 │
│  └──┘    Description paragraph      │
└─────────────────────────────────────┘
```

**Step Specs:**
- Number circle: 36×36px, `--niche-accent` bg, white text, 14pt weight 700
- Connecting line: 2px, `--brand-border`, vertical between circles
- Step title: 13pt, `--brand-primary`, weight 600
- Description: 11pt, `--brand-text`, max 3 lines
- Left padding from number: 20pt
- Step gap: 24pt vertical

---

### Template D: Results / Testimonial Page

```
┌─────────────────────────────────────┐
│ H1: Results                        │
│                                     │
│  [Testimonial card — large]         │
│  ┌──────────────────────────────┐   │
│  │ "Quote text here..."         │   │
│  │                              │   │
│  │  ○ Author Name               │   │
│  │    Title, Company            │   │
│  │    ★★★★★ (optional)          │   │
│  └──────────────────────────────┘   │
│                                     │
│  ┌────────┐  ┌────────┐  ┌────────┐│
│  │ RESULT │  │ RESULT │  │ RESULT ││
│  │ 47     │  │ 68%    │  │ 12     │
│  │ leads   │  │ faster │  │ appts  │
│  └────────┘  └────────┘  └────────┘│
└─────────────────────────────────────┘
```

**Testimonial Card:**
- Background: `--brand-surface`
- Border-left: 4px solid `--niche-accent`
- Padding: 24pt
- Border-radius: 12px
- Quote text: 13pt, italic, `--brand-secondary`
- Author: 10pt, `--brand-primary`, weight 600
- Title: 9pt, `--brand-muted`

**Result Stat Cards:**
- 3-column grid, gap 12pt
- Each card: 120pt × 90pt
- Number: 24pt, `--brand-accent`, weight 800
- Label: 8pt, `--brand-muted`, uppercase
- Background: `--brand-white`, 1px border, 10px radius

---

### Template E: CTA / Transition Page

```
┌─────────────────────────────────────┐
│                                     │
│                                     │
│          [Large icon, 80px]         │
│                                     │
│        Ready to [action]?           │
│        (H1, centered)               │
│                                     │
│     [2-sentence summary of value]   │
│                                     │
│     ┌─────────────────────────┐     │
│     │   GET STARTED FREE →    │     │
│     └─────────────────────────┘     │
│                                     │
│        dreamai.com/[niche]          │
│                                     │
│                                     │
└─────────────────────────────────────┘
```

**CTA Page Specs:**
- Centered layout
- Background: `--gradient-cover` (dark)
- Icon: 80×80px, white at 90% opacity
- Headline: 28pt, `--brand-white`, weight 700
- Subtext: 12pt, `rgba(255,255,255,0.7)`, max 2 lines
- Button: `--niche-accent` bg, white text, 14pt weight 600, padding 14pt 32pt, border-radius 8px, uppercase 0.05em tracking
- URL: 10pt, `rgba(255,255,255,0.5)`

---

## 6. Iconography

### Icon Set Style
- **Style:** Outlined / Line icons, 2px stroke
- **Size:** 24px (inline), 40px (card), 64px (cover), 80px (CTA page)
- **Corner radius:** Match stroke (2px rounded caps/joins)
- **Source:** Lucide Icons or Phosphor Icons (outlined variant)

### Niche Icon Library

| Niche | Cover Icon | Feature Icons |
|-------|-----------|---------------|
| Real Estate | 🏠 House key / keyhole | Phone, calendar, CRM database, list, handshake |
| MedSpa | ✨ Sparkle / star | Calendar, clipboard, chat, star-review, syringe/needle |
| HVAC | ❄️/🔥 Snowflake-flame | Wrench, phone, truck, thermostat, shield |
| Barbershop | ✂️ Scissors / comb | Calendar, scissors, message, users, star |

### Icon Color Rules
- Inline icons: `--brand-accent`
- Feature card icons: White on `--brand-accent` circle bg
- Checkmarks: `--brand-success`
- Warning/caution: `--brand-warning`

---

## 7. Reusable CSS Variables (Design-to-Code Spec)

```css
:root {
  /* === Brand Core === */
  --brand-primary: #0A0F1C;
  --brand-secondary: #1B2744;
  --brand-accent: #4F8CFF;
  --brand-surface: #F5F7FA;
  --brand-white: #FFFFFF;
  --brand-text: #2D3748;
  --brand-muted: #718096;
  --brand-border: #E2E8F0;
  --brand-success: #48BB78;
  --brand-warning: #F6AD55;
  --brand-danger: #FC8181;

  /* === Niche Accents (swap per variant) === */
  --niche-accent: #D4A843;          /* Real Estate gold */
  --niche-light-tint: #FDF8ED;
  --niche-accent-rgb: 212, 168, 67;

  /* === Typography === */
  --font-primary: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  --text-xs: 9pt;
  --text-sm: 10pt;
  --text-base: 11pt;
  --text-md: 13pt;
  --text-lg: 14pt;
  --text-xl: 16pt;
  --text-2xl: 20pt;
  --text-3xl: 28pt;
  --text-4xl: 36pt;
  --text-cover: 42pt;

  /* === Spacing Scale (pt) === */
  --space-1: 4pt;
  --space-2: 8pt;
  --space-3: 12pt;
  --space-4: 16pt;
  --space-5: 20pt;
  --space-6: 24pt;
  --space-8: 32pt;
  --space-10: 40pt;
  --space-12: 48pt;
  --space-16: 64pt;

  /* === Border Radius === */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* === Shadows === */
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.06);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);
  --shadow-lg: 0 8px 24px rgba(0, 0, 0, 0.12);

  /* === Gradients === */
  --gradient-cover: linear-gradient(135deg, var(--brand-primary) 0%, var(--brand-secondary) 60%, var(--niche-accent) 100%);
  --gradient-accent-bar: linear-gradient(90deg, var(--brand-accent) 0%, var(--niche-accent) 100%);
  --gradient-surface: linear-gradient(180deg, var(--brand-surface) 0%, #FFFFFF 100%);
}
```

---

## 8. Production Checklist

- [ ] Create 4 niche variants (swap accent color, icon, copy)
- [ ] Design cover template (single InDesign/Figma master, swap content)
- [ ] Design 5 interior templates as reusable blocks
- [ ] Build icon library (export SVG, 24/40/64/80px)
- [ ] Export color swatches (ASE/ACO for Adobe, JSON for code)
- [ ] Set up paragraph/character styles in InDesign
- [ ] Print test: check CMYK color accuracy for niche accents
- [ ] Digital test: embed fonts, check PDF/A-1b compliance
- [ ] Final export: PDF/A-1b digital + high-res print PDF (300dpi, bleed)
- [ ] File naming: `dream-ai-[niche]-playbook-v1.pdf`

---

## 9. File Deliverables

| Deliverable | Format | Size |
|-------------|--------|------|
| Master Figma file | `.fig` | Shared link |
| Print-ready PDF (per niche) | PDF, 300dpi, CMYK, bleed | ~5–8MB |
| Digital PDF (per niche) | PDF/A-1b, RGB | ~2–3MB |
| Icon SVGs | `.svg` per icon | <10KB each |
| Color swatches | `.ase` + `.json` | <1KB |
| Font files | `.woff2` (Inter 400/500/600/700/800) | ~120KB total |
| Cover thumbnails | PNG, 1200×1550px | ~500KB |
