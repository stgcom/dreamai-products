# Design Brief — Landing Page Visual Design

> **Designer:** PIXEL  
> **Project:** Dream AI Niche Landing Pages  
> **Variants:** Real Estate, MedSpa, HVAC, Barbershop  
> **Pages:** `dreamai.com/[niche]`  
> **Tech:** HTML/CSS (responsive), mobile-first

---

## 1. Global Layout & Grid

### Viewport Breakpoints

| Name | Width | Columns | Gutter | Margin |
|------|-------|---------|--------|--------|
| `xs` (mobile) | 320–479px | 4 | 16px | 16px |
| `sm` (mobile-lg) | 480–767px | 4 | 16px | 24px |
| `md` (tablet) | 768–1023px | 8 | 24px | 32px |
| `lg` (desktop) | 1024–1279px | 12 | 24px | auto (centered) |
| `xl` (desktop-lg) | 1280px+ | 12 | 24px | auto (max-width) |

### Max Width

```css
--page-max-width: 1200px;
--content-max-width: 720px;  /* For text-heavy sections (FAQ, problem) */
--section-padding-y: 96px;   /* Vertical padding per section */
--section-padding-y-mobile: 64px;

/* Responsive adjustment */
@media (max-width: 767px) {
  --section-padding-y: 64px;
}
```

### Content Width Control

| Section | Max Width | Alignment |
|---------|-----------|-----------|
| Hero | 1200px (full container) | Left-aligned (text left, visual right) |
| Problem/Stats | 720px | Center-aligned |
| Features (cards) | 1200px | Grid, center-aligned |
| How It Works | 800px | Center-aligned |
| Results/Testimonials | 1200px | Grid, center-aligned |
| Pricing | 1200px | Grid, center-aligned |
| FAQ | 720px | Left-aligned |
| Footer CTA | 720px | Center-aligned |

---

## 2. Color System — CSS Variables

```css
:root {
  /* === Brand Core === */
  --lp-primary: #0A0F1C;
  --lp-secondary: #1B2744;
  --lp-accent: #4F8CFF;
  --lp-accent-hover: #3A7BEE;
  --lp-surface: #F5F7FA;
  --lp-white: #FFFFFF;
  --lp-text: #2D3748;
  --lp-text-dark: #0A0F1C;
  --lp-muted: #718096;
  --lp-border: #E2E8F0;
  --lp-success: #48BB78;
  --lp-warning: #F6AD55;
  --lp-danger: #FC8181;

  /* === Niche Accent (swap per page) === */
  --niche-accent: #D4A843;          /* Real Estate */
  --niche-accent-hover: #C29A3A;
  --niche-accent-light: rgba(212, 168, 67, 0.08);
  --niche-accent-rgb: 212, 168, 67;

  /* === Gradients === */
  --gradient-hero: linear-gradient(135deg, var(--lp-primary) 0%, var(--lp-secondary) 100%);
  --gradient-hero-accent: radial-gradient(ellipse at 80% 50%, rgba(var(--niche-accent-rgb), 0.15) 0%, transparent 50%);
  --gradient-cta: linear-gradient(135deg, var(--niche-accent) 0%, var(--niche-accent-hover) 100%);
  --gradient-accent-line: linear-gradient(90deg, var(--lp-accent) 0%, var(--niche-accent) 100%);

  /* === Shadows === */
  --shadow-card: 0 1px 3px rgba(0, 0, 0, 0.06), 0 1px 2px rgba(0, 0, 0, 0.04);
  --shadow-card-hover: 0 10px 25px rgba(0, 0, 0, 0.1);
  --shadow-cta: 0 4px 14px rgba(var(--niche-accent-rgb), 0.35);
  --shadow-modal: 0 20px 60px rgba(0, 0, 0, 0.2);

  /* === Border Radius === */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-2xl: 24px;
  --radius-full: 9999px;

  /* === Transitions === */
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
  --transition-slow: 350ms ease;

  /* === Z-index Scale === */
  --z-base: 1;
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-overlay: 300;
  --z-modal: 400;
  --z-toast: 500;
}
```

### Niche Variable Swaps

```css
/* Real Estate */
[data-niche="real-estate"] {
  --niche-accent: #D4A843;
  --niche-accent-hover: #C29A3A;
  --niche-accent-light: rgba(212, 168, 67, 0.08);
  --niche-accent-rgb: 212, 168, 67;
}

/* MedSpa */
[data-niche="medspa"] {
  --niche-accent: #D15B8F;
  --niche-accent-hover: #BF4A7E;
  --niche-accent-light: rgba(209, 91, 143, 0.08);
  --niche-accent-rgb: 209, 91, 143;
}

/* HVAC */
[data-niche="hvac"] {
  --niche-accent: #E07B39;
  --niche-accent-hover: #D06A2A;
  --niche-accent-light: rgba(224, 123, 57, 0.08);
  --niche-accent-rgb: 224, 123, 57;
}

/* Barbershop */
[data-niche="barbershop"] {
  --niche-accent: #2CB5A0;
  --niche-accent-hover: #24A08D;
  --niche-accent-light: rgba(44, 181, 160, 0.08);
  --niche-accent-rgb: 44, 181, 160;
}
```

---

## 3. Typography — Web Scale

```css
/* === Font Stack === */
--font-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

/* === Type Scale (rem, based on 16px root) === */
--text-xs:   0.75rem;   /* 12px — captions, badges */
--text-sm:   0.875rem;  /* 14px — small body, meta */
--text-base: 1rem;      /* 16px — body text */
--text-md:   1.125rem;  /* 18px — large body */
--text-lg:   1.25rem;   /* 20px — section intro */
--text-xl:   1.5rem;    /* 24px — card titles */
--text-2xl:  1.875rem;  /* 30px — section headings */
--text-3xl:  2.25rem;   /* 36px — large headings */
--text-4xl:  3rem;      /* 48px — hero headline */
--text-5xl:  3.75rem;   /* 60px — hero headline (desktop) */

/* === Font Weights === */
--weight-regular: 400;
--weight-medium: 500;
--weight-semibold: 600;
--weight-bold: 700;
--weight-extrabold: 800;

/* === Line Heights === */
--leading-tight: 1.15;
--leading-snug: 1.25;
--leading-normal: 1.5;
--leading-relaxed: 1.65;

/* === Letter Spacing === */
--tracking-tight: -0.025em;
--tracking-normal: 0;
--tracking-wide: 0.025em;
--tracking-uppercase: 0.08em;

/* === Mobile Type (clamp) === */
--hero-title: clamp(2rem, 5vw, 3.75rem);
--section-title: clamp(1.5rem, 3vw, 2.25rem);
--card-title: clamp(1.125rem, 2vw, 1.5rem);
```

---

## 4. Section Specifications

### 4A. Navigation Bar

```
┌────────────────────────────────────────────────────────────┐
│  [LOGO]    Features  How It Works  Pricing  FAQ   [CTA]   │
└────────────────────────────────────────────────────────────┘
```

**Desktop (≥1024px):**
- Height: 72px
- Background: `rgba(10, 15, 28, 0.95)` with `backdrop-filter: blur(12px)`
- Logo: left, 120px wide, white
- Nav links: center-right, 16px, `--lp-white` @ 80%, weight 500, 32px gap
- Hover: full opacity + `--niche-accent` underline (2px, 24px wide)
- CTA button: right, `--niche-accent` bg, white text, 40px height, 12px 24px padding, 8px radius, weight 600
- Position: `position: sticky; top: 0; z-index: var(--z-sticky)`
- Border-bottom: 1px solid `rgba(255,255,255,0.06)`

**Mobile (<768px):**
- Height: 64px
- Logo: left
- Hamburger menu: right, 24×24px, white
- Full-screen overlay menu:
  - Background: `var(--lp-primary)` at 98% opacity
  - Links: 20px, weight 600, centered, 32px vertical gap
  - CTA: full-width button at bottom

---

### 4B. Hero Section

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  [Badge] "AI Employees for [Niche]"                        │
│                                                            │
│  Every Missed Call Is a                                   │
│  $15,000 Commission                                       │
│  Left on the Table                                        │
│                                                            │
│  The 24/7 AI Receptionist that captures, qualifies,       │
│  and books your leads — even when you're showing          │
│  properties, in a meeting, or asleep.                     │
│                                                            │
│  [See It In Action →]    [Take the Free Quiz]             │
│                                                            │
│  ✓ No credit card  ✓ 14-day trial  ✓ Setup in 5 min     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Specs:**

| Element | Desktop | Mobile |
|---------|---------|--------|
| Section height | 600px (min) | auto, 80px padding top/bottom |
| Background | `--gradient-hero` + `--gradient-hero-accent` | Same |
| Left column | 55% width, padding-right 48px | 100%, margin-bottom 40px |
| Right column | 45% (placeholder for visual/animation) | Hidden or full-width below |
| Badge | `--niche-accent-light` bg, `--niche-accent` text, 13px weight 600, 8px radius, 6px 16px padding, uppercase, `--tracking-uppercase` | Same |
| Headline | `clamp(2rem, 5vw, 3.75rem)`, weight 800, `--lp-white`, `--leading-tight` | Same clamp |
| Subheadline | 18px (desktop), 16px (mobile), `--lp-white` @ 70%, weight 400, `--leading-normal` | Same |
| Primary CTA | `--niche-accent` bg, white text, 56px height, 16px 32px padding, 12px radius, weight 600, `--shadow-cta`, `--transition-base` | 48px height, 14px 24px padding |
| Secondary CTA | Transparent bg, 2px `--lp-white` @ 30% border, white text, 56px height, 16px 32px padding, 12px radius | 48px height |
| Trust indicators | 14px, `--lp-white` @ 50%, 24px gap, `✓` icons in `--lp-success` | Stacked, 12px gap |
| Trust icons | 16px check circles, `--lp-success` | Same |

**Visual placeholder (right side):**
- Container: 400 × 320px, 16px radius, `rgba(255,255,255,0.05)` bg, 1px `rgba(255,255,255,0.1)` border
- For production: phone mockup showing AI agent conversation, or Lottie animation

---

### 4C. Problem / Stats Section

```
┌────────────────────────────────────────────────────────────┐
│                        [Surface BG: --lp-surface]          │
│                                                            │
│               The [Niche] Lead Response                    │
│                     Crisis                                 │
│                                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   78%    │  │  15hrs   │  │   43%    │  │  $100K   │  │
│  │ of buyers│  │ average  │  │ of leads │  │ lost per │  │
│  │ buy from │  │ response │  │ never    │  │ year by  │  │
│  │ 1st agent│  │   time   │  │followed up│ │ slow     │  │
│  └──────────┘  └──────────┘  └──────────┘  │ response │  │
│                                             └──────────┘  │
│  You're a great [professional]. But you can't be           │
│  everywhere at once...                                     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Section Specs:**
- Background: `--lp-surface`
- Padding: 96px top/bottom (64px mobile)
- Title: `--text-3xl`, weight 700, `--lp-text-dark`, centered, `--leading-snug`
- Title max-width: 500px, `margin: 0 auto`

**Stat Card Grid:**
- 4-column grid, gap 24px (2-column on mobile, 16px gap)
- Card:
  - Background: `--lp-white`
  - Border: 1px solid `--lp-border`
  - Border-radius: `--radius-lg` (12px)
  - Padding: 32px 24px
  - Text-align: center
  - Transition: `--transition-base`, hover: `--shadow-card-hover`, `translateY(-2px)`
- Number: `clamp(2rem, 4vw, 3rem)`, `--lp-danger` (loss framing), weight 800, `--tracking-tight`
- Label: `--text-sm`, `--lp-muted`, weight 400, `--leading-normal`, margin-top 8px

**Body text below stats:**
- Max-width: 720px, centered
- 18px (desktop), 16px (mobile), `--lp-text`, weight 400, `--leading-relaxed`
- 2–3 paragraphs

---

### 4D. Solution / Features Section

```
┌────────────────────────────────────────────────────────────┐
│  [White BG]                                                │
│                                                            │
│               Meet Your AI Employee                        │
│                                                            │
│  ┌────────────────────┐  ┌────────────────────┐           │
│  │ [ICON 48px]        │  │ [ICON 48px]        │           │
│  │                    │  │                    │           │
│  │ AI Voice           │  │ Speed-to-Lead      │           │
│  │ Receptionist       │  │ Auto-Responder     │           │
│  │ ──────             │  │ ──────             │           │
│  │ Answers every call │  │ Web lead comes in  │           │
│  │ in under 3 seconds.│  │ at 11 PM? AI       │           │
│  │ Qualifies buyers & │  │ texts them in 60   │           │
│  │ sellers. Books     │  │ seconds. No lead   │           │
│  │ directly into your │  │ goes cold.         │           │
│  │ calendar.          │  │                    │           │
│  │                    │  │                    │           │
│  │ ✓ 24/7 coverage   │  │ ✓ Instant SMS     │           │
│  │ ✓ Sounds human    │  │ ✓ Auto-call if    │           │
│  │ ✓ Smart escalation│  │   no response     │           │
│  └────────────────────┘  └────────────────────┘           │
│                                                            │
│  ┌────────────────────────────────────────────┐           │
│  │ [ICON]  CRM Auto-Fill (full-width card)    │           │
│  └────────────────────────────────────────────┘           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Card Specs:**

| Property | Value |
|----------|-------|
| Grid | 2-column, 24px gap (stacked on mobile) |
| Card bg | `--lp-white` |
| Border | 1px solid `--lp-border` |
| Radius | `--radius-xl` (16px) |
| Padding | 40px |
| Transition | `--transition-base` |
| Hover | `--shadow-card-hover`, `translateY(-4px)` |

| Element | Spec |
|---------|------|
| Icon circle | 72×72px, `--niche-accent-light` bg, border-radius 50% |
| Icon inside | 32px, `--niche-accent` color |
| Title | `--text-xl`, `--lp-text-dark`, weight 700 |
| Divider | 32px × 3px, `--niche-accent`, margin 12px 0 |
| Description | `--text-base`, `--lp-text`, `--leading-relaxed`, margin-bottom 20px |
| Checkmarks | 20px icon in `--lp-success`, 14px text, 8px gap, 12px vertical gap between |
| Checkmark list | `display: flex; flex-wrap: wrap; gap: 8px 16px` |

**Full-width feature card variant:**
- Same padding/radius
- 3-column inner grid for icon + 2 text columns (on desktop)
- Or stacked vertically on mobile

---

### 4E. How It Works Section

```
┌────────────────────────────────────────────────────────────┐
│  [Dark BG: --lp-primary]                                   │
│                                                            │
│                   How It Works                             │
│                                                            │
│  ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐│
│  │  01  │───▶│  02  │───▶│  03  │───▶│  04  │───▶│  05  ││
│  └──────┘    └──────┘    └──────┘    └──────┘    └──────┘│
│  Lead comes   AI responds  AI qualifies  Qualified  You show
│  in (call,   instantly    timeline,     leads      up to a
│  web form)   (SMS/voice)  budget,       booked     warm,
│                           location      into your  pre-
│                                         calendar   qualified
│                                                            │
│  ┌──────────────────────────────────────────────────┐     │
│  │  [Demo CTA: "See the AI in action — live demo"]  │     │
│  └──────────────────────────────────────────────────┘     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Section Specs:**
- Background: `--lp-primary`
- Title: `--text-3xl`, `--lp-white`, weight 700, centered
- Steps: Horizontal flex, centered, with connecting arrows

**Step Specs (desktop — horizontal):**
- Step: 180px wide, text-align center
- Number circle: 56×56px, `--niche-accent` bg, white text, `--text-xl` weight 700, radius 50%
- Arrow: 24px, `--lp-white` @ 30%, between steps
- Step title: `--text-base`, `--lp-white`, weight 600, margin-top 12px
- Step description: `--text-sm`, `--lp-white` @ 60%, `--leading-normal`, max 3 lines

**Step Specs (mobile — vertical):**
- Stack vertically, 24px gap
- Number circle: 48×48px
- Arrow: downward, 20px, between steps
- Add left border line connecting circles: 2px, `--niche-accent` @ 30%

**CTA bar:**
- `--niche-accent-light` bg, 16px radius, 24px padding
- Text: `--text-lg`, `--lp-white`, weight 600
- Button: `--niche-accent` bg, white text, 12px radius

---

### 4F. Results / Testimonials Section

```
┌────────────────────────────────────────────────────────────┐
│  [Surface BG: --lp-surface]                                │
│                                                            │
│                  What [Niche] Pros Are Seeing              │
│                                                            │
│  ┌──────────────────────────────────────────────┐         │
│  │  "Quote text here — real results, real        │         │
│  │   numbers, specific to their niche..."        │         │
│  │                                               │         │
│  │  ○ [Avatar] Name, Title at Company           │         │
│  │              ★★★★★                            │         │
│  └──────────────────────────────────────────────┘         │
│                                                            │
│  ┌────────┐  ┌────────┐  ┌────────┐                      │
│  │  47    │  │  68%   │  │  12    │                      │
│  │ leads  │  │ faster │  │appts/  │                      │
│  │captured│  │response│  │ month  │                      │
│  │ /month │  │  time  │  │ booked │                      │
│  └────────┘  └────────┘  └────────┘                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Testimonial Card:**
- Max-width: 720px, centered
- Background: `--lp-white`
- Border-left: 4px solid `--niche-accent`
- Border-radius: `--radius-lg`
- Padding: 40px
- Box-shadow: `--shadow-card`
- Quote text: `--text-lg`, `--lp-text-dark`, italic, weight 400, `--leading-relaxed`
- Author section: flex, gap 12px, margin-top 24px
- Avatar: 48×48px, circle, 2px `--niche-accent` border
- Name: `--text-base`, `--lp-text-dark`, weight 600
- Title: `--text-sm`, `--lp-muted`
- Stars: 18px, `--lp-warning`

**Result Stats:**
- 3-column grid, gap 24px (stacked on mobile)
- Card: `--lp-white` bg, 1px `--lp-border`, `--radius-lg`, padding 32px, text-align center
- Number: `clamp(2rem, 4vw, 3rem)`, `--niche-accent`, weight 800
- Label: `--text-sm`, `--lp-muted`, weight 400

---

### 4G. Pricing Section

```
┌────────────────────────────────────────────────────────────┐
│  [White BG]                                                │
│                                                            │
│                     Choose Your Level                      │
│                                                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │   Starter   │  │     Pro ⭐  │  │    Team     │       │
│  │             │  │ Most Popular│  │             │       │
│  │   $297/mo   │  │   $497/mo   │  │   $797/mo   │       │
│  │             │  │             │  │             │       │
│  │  ✓ Feature 1│  │  ✓ Feature 1│  │  ✓ Feature 1│       │
│  │  ✓ Feature 2│  │  ✓ Feature 2│  │  ✓ Feature 2│       │
│  │  ✓ Feature 3│  │  ✓ Feature 3│  │  ✓ Feature 3│       │
│  │             │  │  ✓ Feature 4│  │  ✓ Feature 4│       │
│  │             │  │  ✓ Feature 5│  │  ✓ Feature 5│       │
│  │             │  │             │  │  ✓ Feature 6│       │
│  │             │  │             │  │  ✓ Feature 7│       │
│  │             │  │             │  │             │       │
│  │ [Get Started]│ │ [Get Started]│ │ [Get Started]│       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
│                                                            │
│  [Start Free Trial — 14 days, no credit card]              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Pricing Card Specs:**

| Property | Starter | Pro (Featured) | Team |
|----------|---------|---------------|------|
| Width | 320px | 340px (taller) | 320px |
| Background | `--lp-white` | `--lp-primary` | `--lp-white` |
| Border | 1px `--lp-border` | 2px `--niche-accent` | 1px `--lp-border` |
| Radius | `--radius-xl` | `--radius-xl` | `--radius-xl` |
| Padding | 32px | 40px | 32px |
| Shadow | `--shadow-card` | `--shadow-lg` + `--shadow-cta` | `--shadow-card` |
| Title color | `--lp-text-dark` | `--lp-white` | `--lp-text-dark` |
| Price color | `--lp-text-dark` | `--lp-white` | `--lp-text-dark` |
| CTA | Outline (accent border) | Filled (accent bg) | Outline (accent border) |

**Card Detail Specs:**
- Badge ("Most Popular"): `--niche-accent` bg, white text, 11px weight 600, uppercase, 4px 12px padding, `--radius-full`, absolute position top -14px
- Plan name: `--text-xl`, weight 700
- Price: `clamp(2rem, 3vw, 2.5rem)`, weight 800, `/mo` suffix at `--text-base` weight 400
- Divider: 1px `--lp-border` (or `rgba(255,255,255,0.1)` on dark), margin 24px 0
- Feature list: flex column, 12px gap
- Feature item: 20px check icon in `--lp-success` (or `--niche-accent` on dark), 14px text, 8px gap
- CTA button: full-width, 48px height, 12px radius, weight 600

**Free Trial Banner:**
- Centered below cards
- Text: `--text-base`, `--lp-muted`
- Link: `--niche-accent`, weight 600, underline

---

### 4H. FAQ Section

```
┌────────────────────────────────────────────────────────────┐
│  [Surface BG]                                              │
│                                                            │
│                   Frequently Asked Questions               │
│                                                            │
│  ┌──────────────────────────────────────────────┐         │
│  │ ▸ Will my leads know they're talking to AI?  │         │
│  └──────────────────────────────────────────────┘         │
│  ┌──────────────────────────────────────────────┐         │
│  │ ▸ What happens to urgent/complex calls?      │         │
│  └──────────────────────────────────────────────┘         │
│  ┌──────────────────────────────────────────────┐         │
│  │ ▸ Does it work with my CRM?                  │         │
│  └──────────────────────────────────────────────┘         │
│  ┌──────────────────────────────────────────────┐         │
│  │ ▸ What if a lead calls at 2 AM?              │         │
│  └──────────────────────────────────────────────┘         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**FAQ Item Specs:**
- Max-width: 720px, centered
- Background: `--lp-white`
- Border: 1px solid `--lp-border`
- Radius: `--radius-lg`
- Margin-bottom: 8px (accordion style)
- Padding: 20px 24px
- Cursor: pointer

**Question (collapsed state):**
- Font: `--text-base`, `--lp-text-dark`, weight 600
- Icon: chevron-right, 20px, `--lp-muted`, `--transition-base` rotate 90° when open

**Answer (expanded state):**
- Font: `--text-base`, `--lp-text`, weight 400, `--leading-relaxed`
- Padding-top: 16px, margin-top 16px, border-top 1px `--lp-border`
- Transition: `max-height 300ms ease, opacity 200ms ease`

**Active/Expanded Item:**
- Border-color: `--niche-accent`
- Left border: 3px solid `--niche-accent`

---

### 4I. Footer CTA Section

```
┌────────────────────────────────────────────────────────────┐
│  [Dark BG with niche accent glow]                          │
│                                                            │
│     Stop Losing Leads to Voicemail                         │
│                                                            │
│  Your next [buyer/patient/client] is calling right now.    │
│  Make sure someone answers.                                │
│                                                            │
│     [Get Your Free AI Demo →]                              │
│     [Take the 30-Second Readiness Quiz →]                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Specs:**
- Background: `--lp-primary` + `radial-gradient(ellipse at 50% 100%, rgba(var(--niche-accent-rgb), 0.12) 0%, transparent 60%)`
- Padding: 120px top/bottom (80px mobile)
- Title: `clamp(1.75rem, 4vw, 3rem)`, `--lp-white`, weight 800, centered
- Subtext: `--text-lg`, `--lp-white` @ 70%, centered, max-width 500px, `margin: 0 auto`
- Primary CTA: `--niche-accent` bg, white text, 56px height, 16px 40px padding, 12px radius, `--shadow-cta`
- Secondary CTA: Transparent, 2px `--lp-white` @ 30% border, white text, 56px height, 16px 40px padding, 12px radius
- CTA stack: flex column, 16px gap (desktop: can be side-by-side at ≥768px)

---

## 5. Mobile Responsive Details

### Global Mobile Rules

```css
@media (max-width: 767px) {
  :root {
    --section-padding-y: 64px;
  }

  /* Stack all grids */
  .grid-2col, .grid-3col, .grid-4col {
    grid-template-columns: 1fr;
    gap: 16px;
  }

  /* Increase touch targets */
  .btn {
    min-height: 48px;
    padding: 12px 24px;
    font-size: 16px; /* Prevents iOS zoom on focus */
  }

  /* Adjust pricing cards */
  .pricing-grid {
    grid-template-columns: 1fr;
    max-width: 400px;
    margin: 0 auto;
  }

  .pricing-card--featured {
    order: -1; /* Pro card first on mobile */
  }

  /* FAQ full-width */
  .faq-item {
    padding: 16px 20px;
  }

  /* How it works: vertical */
  .steps-horizontal {
    flex-direction: column;
    align-items: center;
  }

  .step-arrow {
    transform: rotate(90deg);
  }
}
```

### Mobile-Specific Components

| Component | Desktop | Mobile Change |
|-----------|---------|---------------|
| Nav | Horizontal links | Hamburger → full-screen overlay |
| Hero | 2-column (text + visual) | Stacked, visual hidden or below |
| Stats | 4-column grid | 2-column grid |
| Features | 2-column grid | Stacked single column |
| Pricing | 3-column side-by-side | Stacked, Pro card first |
| How It Works | Horizontal steps with arrows | Vertical steps with connecting line |
| FAQ | Max 720px, centered | Full-width with padding |
| Footer CTA | Side-by-side buttons | Stacked buttons |
| Touch targets | 40px min | 48px min |

### iOS Input Fix

```css
/* Prevent zoom on input focus in iOS */
input, select, textarea {
  font-size: 16px;
}
```

---

## 6. Micro-Interactions & Animation

### Hover States

| Element | State | Transition |
|---------|-------|-----------|
| Primary CTA | `translateY(-2px)`, shadow increases | `250ms ease` |
| Secondary CTA | Border becomes `--niche-accent` | `200ms ease` |
| Feature cards | `translateY(-4px)`, `--shadow-card-hover` | `250ms ease` |
| Stat cards | `translateY(-2px)`, slight scale `1.02` | `200ms ease` |
| Pricing cards | (Pro card already elevated) | — |
| FAQ items | Border-left color change, chevron rotates | `300ms ease` |

### Scroll Animations (CSS-only or Intersection Observer)

```css
/* Fade-up on scroll */
.reveal {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 600ms ease, transform 600ms ease;
}

.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}

/* Stagger children */
.reveal-stagger > * {
  opacity: 0;
  transform: translateY(16px);
  transition: opacity 400ms ease, transform 400ms ease;
}

.reveal-stagger.visible > *:nth-child(1) { transition-delay: 0ms; }
.reveal-stagger.visible > *:nth-child(2) { transition-delay: 100ms; }
.reveal-stagger.visible > *:nth-child(3) { transition-delay: 200ms; }
.reveal-stagger.visible > *:nth-child(4) { transition-delay: 300ms; }
```

### Loading States

```css
/* Skeleton loader for social proof / testimonials */
.skeleton {
  background: linear-gradient(90deg,
    var(--lp-border) 25%,
    rgba(226, 232, 240, 0.5) 50%,
    var(--lp-border) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: var(--radius-md);
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

---

## 7. Performance Budget

| Metric | Target |
|--------|--------|
| First Contentful Paint | < 1.5s |
| Largest Contentful Paint | < 2.5s |
| Cumulative Layout Shift | < 0.1 |
| Total page weight | < 500KB (HTML + CSS + JS, excl. images) |
| Hero image (WebP) | < 80KB |
| Font files | < 100KB (Inter woff2, 3 weights) |
| CSS | < 30KB unminified |
| JS | < 20KB (FAQ accordion + scroll reveal) |

### Image Formats

| Use | Format | Sizes | Max |
|-----|--------|-------|-----|
| Hero visual | WebP (AVIF fallback) | 400w, 800w, 1200w | 80KB |
| Avatars | WebP | 48w, 96w | 15KB |
| Icons | SVG (inline) | — | 2KB each |
| Social proof logos | SVG or PNG @2x | — | 10KB |

---

## 8. Component Class Naming (BEM-inspired)

```
/* Navigation */
.nav              → .nav
.nav__link        → .nav__link
.nav__cta         → .nav__cta

/* Hero */
.hero             → .hero
.hero__badge      → .hero__badge
.hero__title      → .hero__title
.hero__subtitle   → .hero__subtitle
.hero__actions    → .hero__actions
.hero__trust      → .hero__trust

/* Stats */
.stats            → .stats
.stats__grid      → .stats__grid
.stat-card        → .stat-card
.stat-card__number → .stat-card__number
.stat-card__label → .stat-card__label

/* Features */
.features         → .features
.feature-card     → .feature-card
.feature-card__icon   → .feature-card__icon
.feature-card__title  → .feature-card__title
.feature-card__desc   → .feature-card__desc
.feature-card__checks → .feature-card__checks

/* Pricing */
.pricing          → .pricing
.pricing__grid    → .pricing__grid
.pricing-card     → .pricing-card
.pricing-card--featured → .pricing-card--featured
.pricing-card__badge    → .pricing-card__badge
.pricing-card__name     → .pricing-card__name
.pricing-card__price    → .pricing-card__price
.pricing-card__features → .pricing-card__features
.pricing-card__cta      → .pricing-card__cta

/* FAQ */
.faq              → .faq
.faq__item        → .faq__item
.faq__question    → .faq__question
.faq__answer      → .faq__answer

/* CTA */
.cta-section      → .cta-section
.cta-section__title    → .cta-section__title
.cta-section__subtitle → .cta-section__subtitle
.cta-section__actions  → .cta-section__actions
```

---

## 9. Production Checklist

- [ ] Build HTML/CSS skeleton with all CSS variables defined
- [ ] Implement responsive grid system (4/8/12 column)
- [ ] Build all 9 section templates
- [ ] Create `data-niche` attribute system for color swapping
- [ ] Implement navigation (desktop + mobile hamburger)
- [ ] Implement FAQ accordion (vanilla JS, accessible)
- [ ] Add scroll reveal animations (Intersection Observer)
- [ ] Optimize hero images (WebP, srcset, lazy loading)
- [ ] Test on Chrome, Firefox, Safari, Edge
- [ ] Test on iPhone SE, iPhone 14 Pro, iPad, Android
- [ ] Run Lighthouse audit (target: 90+ performance, 95+ accessibility)
- [ ] Validate HTML (W3C)
- [ ] Check WCAG 2.1 AA contrast ratios
- [ ] Set up analytics event tracking on all CTAs
- [ ] A/B test hero headlines (QUILL to provide variants)

---

## 10. Accessibility Requirements

| Requirement | Spec |
|-------------|------|
| Color contrast | Minimum 4.5:1 for body text, 3:1 for large text (WCAG AA) |
| Focus states | Visible focus ring: 2px solid `--lp-accent`, offset 2px |
| Alt text | All images require descriptive alt text |
| ARIA labels | FAQ accordion: `aria-expanded`, `aria-controls` |
| Keyboard nav | All interactive elements keyboard-focusable, logical tab order |
| Reduced motion | `@media (prefers-reduced-motion: reduce)` → disable animations |
| Font scaling | All `rem`-based, respects user font-size preferences |
| Touch targets | Minimum 44×44px (Apple HIG) / 48×48px (Material) |

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }

  .reveal {
    opacity: 1;
    transform: none;
  }
}
```
