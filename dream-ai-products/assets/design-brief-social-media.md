# Design Brief — Social Media Templates

> **Designer:** PIXEL  
> **Project:** Dream AI Social Media System  
> **Platforms:** LinkedIn, Instagram (Feed + Stories)  
> **Variants:** Quote cards, Testimonials, Stat callouts, Feature highlights  
> **Color-coded by niche:** Real Estate 🏠, MedSpa ✨, HVAC ❄️🔥, Barbershop ✂️

---

## 1. Platform Specifications

### LinkedIn

| Format | Dimensions | File Type | Max Size |
|--------|-----------|-----------|----------|
| Single image post | 1200 × 627px | PNG/JPG | 5MB |
| Carousel slide | 1080 × 1080px | PNG | 5MB per slide |
| Article cover | 1200 × 627px | PNG | 5MB |
| Profile banner | 1584 × 396px | PNG | 8MB |

### Instagram

| Format | Dimensions | File Type | Max Size |
|--------|-----------|-----------|----------|
| Feed post (square) | 1080 × 1080px | PNG/JPG | 8MB |
| Feed post (portrait) | 1080 × 1350px | PNG/JPG | 8MB |
| Story | 1080 × 1920px | PNG/JPG | 8MB |
| Reel cover | 1080 × 1920px | PNG/JPG | 8MB |
| Profile photo | 320 × 320px | PNG/JPG | — |

### Safe Zones

```
LinkedIn 1200×627:         Instagram 1080×1080:
┌────────────────┐         ┌────────────────┐
│ ←30px→         │         │ ←40px→         │
│  ┌──────────┐  │ ↑30px   │  ┌──────────┐  │ ↑40px
│  │ Safe     │  │         │  │ Safe     │  │
│  │ Content  │  │         │  │ Content  │  │
│  │ Zone     │  │ ↓30px   │  │ Zone     │  │ ↓40px
│  └──────────┘  │         │  └──────────┘  │
│         ←30px→ │         │         ←40px→ │
└────────────────┘         └────────────────┘
  No text in last           Instagram crops
  150px right (card         corners slightly
  overlay area)
```

---

## 2. Color System — Niche-Coded

### Background Palettes by Niche

Each niche gets a distinct background treatment so followers instantly recognize the content vertical.

| Niche | Primary BG | Secondary BG | Accent Overlay | Text Primary | Text Secondary |
|-------|-----------|-------------|----------------|-------------|----------------|
| **Real Estate** | `#0A0F1C` (navy) | `#131B2E` | Gold `#D4A843` @ 15% | `#FFFFFF` | `#A0AEC0` |
| **MedSpa** | `#1A0F1A` (plum) | `#241528` | Rose `#D15B8F` @ 15% | `#FFFFFF` | `#D4B8D4` |
| **HVAC** | `#1A1208` (charcoal) | `#241A0C` | Amber `#E07B39` @ 15% | `#FFFFFF` | `#D4B890` |
| **Barbershop** | `#0A1A18` (dark teal) | `#0E2420` | Teal `#2CB5A0` @ 15% | `#FFFFFF` | `#A0D4CB` |

### Universal Tokens (all niches)

```css
/* Social shared tokens — swap niche vars per template */
--social-text-white: #FFFFFF;
--social-text-muted: rgba(255, 255, 255, 0.65);
--social-border: rgba(255, 255, 255, 0.1);
--social-badge-bg: rgba(255, 255, 255, 0.12);
--social-badge-text: #FFFFFF;
--social-cta-bg: var(--niche-accent);
--social-cta-text: #FFFFFF;
--social-stat-number: var(--niche-accent);
```

---

## 3. Typography — Social Scale

```css
/* === Social Typography (px sizes for screen) === */
--social-display: 42px;     /* Hero stat, cover title */
--social-headline: 28px;    /* Post headline */
--social-subhead: 20px;     /* Section label */
--social-body: 16px;        /* Quote text, description */
--social-caption: 13px;     /* Attribution, CTA */
--social-micro: 11px;       /* Watermark, hashtags */

/* === Weights === */
--weight-display: 800;      /* ExtraBold */
--weight-headline: 700;     /* Bold */
--weight-body: 400;         /* Regular */
--weight-caption: 500;      /* Medium */

/* === Line Heights === */
--leading-tight: 1.15;
--leading-snug: 1.25;
--leading-normal: 1.5;

/* === Letter Spacing === */
--tracking-tight: -0.02em;
--tracking-normal: 0;
--tracking-wide: 0.04em;
--tracking-uppercase: 0.08em;
```

---

## 4. Template Library

### Template A: Quote Card (1080 × 1080)

```
┌────────────────────────────────┐
│ [Niche accent gradient bar     │
│  8px height, top]              │
│                                │
│                                │
│    [Large quotation mark]      │
│    120px, niche accent @ 30%   │
│                                │
│    "Every missed call is       │
│     a $15,000 commission       │
│     left on the table."        │
│                                │
│    — Stephen G                 │
│      Founder, Dream AI         │
│                                │
│                                │
│  ┌────────────────────────┐    │
│  │ dreamai.com    [LOGO]  │    │
│  └────────────────────────┘    │
└────────────────────────────────┘
```

**Specs:**
- Canvas: 1080 × 1080px
- Background: Niche primary BG
- Accent bar: 8px, `--gradient-accent-bar` at top
- Quotation mark: Inter 800, 120px, `--niche-accent` at 30% opacity
- Quote text: 28px, `--social-text-white`, weight 700, `--leading-snug`, `--tracking-tight`
- Attribution: 16px, `--niche-accent`, weight 500
- Title/role: 13px, `--social-text-muted`
- Footer: 13px, `--social-text-muted`, right-aligned logo 24px height
- Content zone: 72px inset all sides
- Bottom margin for footer: 48px

---

### Template B: Testimonial Card (1080 × 1080)

```
┌────────────────────────────────┐
│                                │
│  ★★★★★  Niche badge            │
│  "Incredible results..."       │
│                                │
│  [Avatar circle]               │
│  Name                          │
│  Title · Company               │
│                                │
│  ┌────────────────────────┐    │
│  │  🏠 Real Estate Agent  │    │
│  └────────────────────────┘    │
│                                │
│  dreamai.com    [LOGO]         │
└────────────────────────────────┘
```

**Specs:**
- Canvas: 1080 × 1080px
- Background: Niche primary BG
- Star rating: 5× `--brand-warning` (`#F6AD55`), 20px, 4px gap
- Niche badge: 11px, `--social-badge-bg` bg, `--niche-accent` text, 20px border-radius, 6px padding H, 4px V
- Quote: 22px, `--social-text-white`, weight 500, `--leading-normal`
- Avatar: 64 × 64px, circle, 3px `--niche-accent` border
- Name: 18px, `--social-text-white`, weight 600
- Title: 14px, `--social-text-muted`
- Category pill: 13px, niche accent bg, white text, 24px border-radius, 8px 16px padding
- Spacing: 48px sections between elements

---

### Template C: Stat Callout (1080 × 1080)

```
┌────────────────────────────────┐
│                                │
│                                │
│        ┌──────────────┐        │
│        │     78%      │        │
│        │   of buyers  │        │
│        │ buy from the │        │
│        │  FIRST agent │        │
│        │  who responds│        │
│        └──────────────┘        │
│                                │
│    ┌──────────────────────┐    │
│    │ ⚡ Are you fast enough?│   │
│    └──────────────────────┘    │
│                                │
│  dreamai.com    [LOGO]         │
└────────────────────────────────┘
```

**Specs:**
- Canvas: 1080 × 1080px
- Background: Niche primary BG + subtle radial gradient overlay at center
- Stat number: 72px, `--niche-accent`, weight 800, `--tracking-tight`
- Stat description: 22px, `--social-text-white`, weight 400, max 4 lines, `--leading-snug`
- Question/statement bar: 16px, `--niche-accent` bg, white text, 8px border-radius, 12px 20px padding, center-aligned
- Radial gradient: `radial-gradient(circle at center, rgba(--niche-accent-rgb, 0.08) 0%, transparent 60%)`
- Content vertically centered with 40px margin for footer

---

### Template D: Feature Highlight (1080 × 1350 — Portrait Instagram)

```
┌────────────────────────────────┐
│ [Niche accent gradient bar]    │
│                                │
│  [ICON — 48px, white]          │
│                                │
│  AI Voice                      │
│  Receptionist                  │
│  ──────                        │
│                                │
│  Answers every call in under   │
│  3 seconds. Qualifies buyers   │
│  and sellers. Books directly   │
│  into your calendar.           │
│                                │
│  ✓ 24/7 availability           │
│  ✓ Sounds human                │
│  ✓ Transfers complex calls     │
│                                │
│  ┌────────────────────────┐    │
│  │   SEE IT IN ACTION →   │    │
│  └────────────────────────┘    │
│                                │
│  dreamai.com    [LOGO]         │
└────────────────────────────────┘
```

**Specs:**
- Canvas: 1080 × 1350px
- Background: Niche primary BG
- Icon: 48×48px, white, `--social-badge-bg` circle (72×72px)
- Feature title: 36px, `--social-text-white`, weight 800, 2 lines max
- Divider: 32px wide, 3px, `--niche-accent`
- Description: 18px, `--social-text-white`, weight 400, 4 lines max, `--leading-normal`
- Checkmarks: `--brand-success` (`#48BB78`), 16px icon + 16px text, 12px gap
- CTA button: `--niche-accent` bg, white text, 16px weight 600, 8px radius, 14px 28px padding
- Vertical spacing: 32px between sections
- Footer: 13px, bottom 40px

---

### Template E: Carousel Slide (1080 × 1080, set of 5)

**Slide 1 — Cover:**
```
┌────────────────────────────────┐
│                                │
│    THE [NICHE]                 │
│    AI PLAYBOOK                 │
│    ────────                    │
│    Swipe → for 5 automations   │
│    that run your business      │
│    while you sleep             │
│                                │
│  [Number badge] 1/5            │
│  dreamai.com    [LOGO]         │
└────────────────────────────────┘
```

**Slides 2–4 — One feature per slide:**
```
┌────────────────────────────────┐
│ [Niche icon]    2/5            │
│                                │
│  FEATURE NAME                  │
│  ──────                        │
│  Description (3 lines max)     │
│                                │
│  ┌────────┐  ┌────────┐        │
│  │ KEY    │  │ KEY    │        │
│  │ STAT   │  │ STAT   │        │
│  └────────┘  └────────┘        │
│                                │
│  dreamai.com    [LOGO]         │
└────────────────────────────────┘
```

**Slide 5 — CTA:**
```
┌────────────────────────────────┐
│                                │
│      Ready to automate?        │
│      ──────                    │
│      Get the full playbook     │
│      dreamai.com/[niche]       │
│                                │
│     ┌──────────────────┐       │
│     │  GET STARTED →    │       │
│     └──────────────────┘       │
│                                │
│  5/5                    [LOGO] │
└────────────────────────────────┘
```

**Carousel Specs:**
- All slides: 1080 × 1080px
- Slide number: 13px, `--social-text-muted`, top-right
- Cover title: 42px, weight 800
- Feature titles: 28px, weight 700
- Mini stat cards: 2-column, 120×80px, `--social-badge-bg`, 8px radius
- CTA button: full-width centered, 48px height, `--niche-accent`

---

### Template F: LinkedIn Article Cover (1200 × 627)

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  [Left 55%]              │ [Right 45%]              │
│                           │                          │
│  78% of buyers           │  [Niche icon             │
│  buy from the first      │   120×120px              │
│  agent who responds      │   niche accent bg]       │
│                           │                          │
│  ──────                  │                          │
│                           │                          │
│  How AI employees         │                          │
│  capture the leads        │                          │
│  your competition         │                          │
│  is losing                │                          │
│                           │                          │
│  dreamai.com              │                          │
│                           │                          │
└──────────────────────────────────────────────────────┘
```

**Specs:**
- Canvas: 1200 × 627px
- Left content zone: 55% width, 60px left padding
- Right graphic zone: 45%, icon centered
- Stat: 36px, `--niche-accent`, weight 800
- Headline: 24px, `--social-text-white`, weight 600, `--leading-snug`
- Divider: 40px wide, 3px, `--niche-accent`
- Icon container: 120×120px, `--niche-accent` bg at 20%, border-radius 24px
- Background: Niche primary BG

---

### Template G: Instagram Story (1080 × 1920)

```
┌──────────────────┐
│ [Niche accent    │
│  gradient bar]   │
│                  │
│  [Niche badge]   │
│                  │
│  DID YOU         │
│  KNOW?           │
│                  │
│  43%             │
│  of leads are    │
│  never followed  │
│  up with         │
│                  │
│  ──────          │
│                  │
│  [Swipe up CTA]  │
│                  │
│                  │
│ dreamai.com      │
│ [LOGO]           │
└──────────────────┘
```

**Specs:**
- Canvas: 1080 × 1920px
- Content zone: top 30% to 75% (avoid Instagram UI overlays at top/bottom)
- Stat: 72px, `--niche-accent`, weight 800
- "Did you know": 36px, white, weight 800, uppercase, `--tracking-uppercase`
- Description: 24px, white, weight 400, `--leading-normal`
- Swipe indicator: 16px, white @ 60%, animated arrow optional (design as static first)
- Top bar: 6px, `--gradient-accent-bar`
- Background: Niche primary BG with subtle diagonal stripe overlay at 3% opacity

---

## 5. Niche Color Map — Quick Reference

| Template | Real Estate | MedSpa | HVAC | Barbershop |
|----------|------------|--------|------|-----------|
| Quote Card | Gold gradient | Rose gradient | Amber gradient | Teal gradient |
| Testimonial | Gold badge + border | Rose badge + border | Amber badge + border | Teal badge + border |
| Stat Callout | Gold number | Rose number | Amber number | Teal number |
| Feature | Gold CTA + icon | Rose CTA + icon | Amber CTA + icon | Teal CTA + icon |
| Carousel | Gold accent slides | Rose accent slides | Amber accent slides | Teal accent slides |
| Story | Gold stat number | Rose stat number | Amber stat number | Teal stat number |

### Visual Identity Per Niche

| Niche | Background Mood | Accent Feel | Icon Style |
|-------|----------------|-------------|-----------|
| Real Estate | Dark navy, luxury | Warm gold, premium | House keys, buildings, handshake |
| MedSpa | Dark plum, elegant | Soft rose, calming | Sparkles, stars, clipboard |
| HVAC | Dark charcoal, industrial | Warm amber, reliable | Wrenches, thermometers, trucks |
| Barbershop | Dark teal, modern | Fresh teal, sharp | Scissors, combs, razors |

---

## 6. Design Tokens (JSON — for automation)

```json
{
  "niches": {
    "real-estate": {
      "accent": "#D4A843",
      "accent_rgb": "212, 168, 67",
      "bg_primary": "#0A0F1C",
      "bg_secondary": "#131B2E",
      "text_muted": "#A0AEC0",
      "icon": "key",
      "label": "Real Estate",
      "gradient": "linear-gradient(135deg, #0A0F1C 0%, #131B2E 60%, #D4A843 100%)"
    },
    "medspa": {
      "accent": "#D15B8F",
      "accent_rgb": "209, 91, 143",
      "bg_primary": "#1A0F1A",
      "bg_secondary": "#241528",
      "text_muted": "#D4B8D4",
      "icon": "sparkles",
      "label": "MedSpa",
      "gradient": "linear-gradient(135deg, #1A0F1A 0%, #241528 60%, #D15B8F 100%)"
    },
    "hvac": {
      "accent": "#E07B39",
      "accent_rgb": "224, 123, 57",
      "bg_primary": "#1A1208",
      "bg_secondary": "#241A0C",
      "text_muted": "#D4B890",
      "icon": "thermometer",
      "label": "HVAC",
      "gradient": "linear-gradient(135deg, #1A1208 0%, #241A0C 60%, #E07B39 100%)"
    },
    "barbershop": {
      "accent": "#2CB5A0",
      "accent_rgb": "44, 181, 160",
      "bg_primary": "#0A1A18",
      "bg_secondary": "#0E2420",
      "text_muted": "#A0D4CB",
      "icon": "scissors",
      "label": "Barbershop",
      "gradient": "linear-gradient(135deg, #0A1A18 0%, #0E2420 60%, #2CB5A0 100%)"
    }
  }
}
```

---

## 7. Production Workflow

### Figma Setup

1. **Create 1 component library** with:
   - Background variants (4 niche themes)
   - Typography styles (social scale)
   - Icon components (per niche)
   - CTA button variants
   - Stat card component
   - Testimonial card component
   - Footer watermark component

2. **Auto-layout rules:**
   - Vertical stack, 32px gap for sections
   - Horizontal auto-layout for stat cards (16px gap)
   - Padding: 40px all content zones

3. **Naming convention:**
   ```
   Social/[Platform]/[Template]/[Niche]
   Example: Social/Instagram/QuoteCard/RealEstate
   ```

### Export Settings

| Asset | Format | Scale | Naming |
|-------|--------|-------|--------|
| Feed posts | PNG | 1x | `dream-ai-[niche]-[template]-[number].png` |
| Stories | PNG | 1x | `dream-ai-[niche]-story-[topic].png` |
| LinkedIn | PNG | 1x | `dream-ai-[niche]-linkedin-[type].png` |
| Carousel | PNG set | 1x | `dream-ai-[niche]-carousel-[n].png` |
| Source files | Figma link | — | Shared in team workspace |

### Content Calendar Integration

| Day | Content Type | Template | Niche Rotation |
|-----|-------------|----------|----------------|
| Mon | Stat Callout | C | RE → MedSpa → HVAC → Barbershop |
| Tue | Quote Card | A | Rotating experts |
| Wed | Feature Highlight | D | Product features |
| Thu | Testimonial | B | Client stories |
| Fri | Carousel (educational) | E | Multi-slide explainer |

---

## 8. Delivery Checklist

- [ ] Build Figma component library (7 templates × 4 niches = 28 variants)
- [ ] Define all typography styles in Figma
- [ ] Create niche background variants as color styles
- [ ] Build icon set (per niche, 8–10 icons each)
- [ ] Design 2 example posts per template per niche (56 assets)
- [ ] Design 1 carousel set per niche (4 carousels × 5 slides = 20 assets)
- [ ] Design 3 story templates per niche (12 story assets)
- [ ] Export all as PNG at 1x
- [ ] Create Canva/Figma template links for QUILL to use for content
- [ ] Document tokens.json for BOLT to use in automation pipeline
- [ ] Review all assets on mobile (check text readability)
