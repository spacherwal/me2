---
name: LivguardPage and SolarPage implementation status
description: Current state of the Figma→React conversion for LivguardPage.tsx and SolarPage.tsx — sections built, render order, assets, and routing
type: project
---

Both pages are complete and wired up via react-router-dom in `src/App.tsx`.

**Routes:**
- `/` → `LivguardPage`
- `/solar` → `SolarPage`

**Figma source:**
- File key: `7AW0hKHcyl9yZUOvRYSA8j`
- Home page node: `3-2`
- Solar page node: `6-2`

**LivguardPage.tsx — final render order:**
1. `TopBar` — phone, Find a Store, language toggle (left); Track Order, Warranty, Dealer (right)
2. `Navbar` — logo + nav links; "Solar" uses `useNavigate("/solar")`
3. `HeroSection` — badge, H1 "Power Your Home. / On Your Terms.", trust badges, hero-banner.png with 3 floating cards, category bar
4. `ProductsSection` — "Featured Products" centered header, 3-col card grid with BESTSELLER badge, Add to Cart, product images
5. `BrandFamilySection` — "One Family. Four Powerhouses." 2×2 brand grid (Livguard, LIVFAST, INDPOWER, Livguard Solar)
6. `SolarSection` — "Solar Made / Simple." 3-step process, solar-installation.png, "Get Free Quote →" navigates to /solar
7. `StatsBar` — red bg, 4 stats
8. `WeAreWithYouSection` — "We're with You. Always." 4 service rows left + AMC card + 2×2 quick actions right
9. `TestimonialsSection` — 3 review cards
10. `CTABanner` — dark bg, Shop Now + Talk to an Expert
11. `Footer` — 4-column dark footer

**SolarPage.tsx — sections:**
- TopBar, Navbar (Solar active with red underline), HeroSection, CTASection, Footer
- `useNavigate` back to `/` from navbar

**Downloaded assets (in `public/images/`):**
- `hero-banner.png` — Figma node 3:52
- `solar-installation.png` — Figma node 6:373
- `product-inverter.png` — Figma node 3:152
- Product images for cards 2 & 3 (IT 1548TT, Solar Panel) not yet downloaded — currently use colored placeholder backgrounds

**Design tokens used (inline styles, no CSS framework):**
- red: #ED1C24, nearBlack: #231F20, charcoal: #111010, bgLight: #FAFAFA
- Font: Source Sans 3

**Why:** Figma-to-React conversion for Livguard brand site. No CSS framework — all inline styles.

**How to apply:** When resuming, check render order above. If adding a new section, insert it into the `LivguardPage` return in the correct position and define the component function in the same file. Pending: download remaining product card images from Figma nodes 3:176 and 3:201.
