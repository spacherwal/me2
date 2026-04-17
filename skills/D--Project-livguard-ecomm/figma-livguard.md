# Livguard Figma → Code Skill

Implement or update UI sections from the Livguard Figma file into this project.

## Figma File

- **File key:** `7AW0hKHcyl9yZUOvRYSA8j`
- **File name:** Livgaurd-Revamp
- **Main page node:** `3-2` (the full homepage canvas)

## Stack (already configured — no setup needed)

- Next.js 16.2.2 (App Router), React 19, TypeScript
- Tailwind CSS v4.2.2 — config lives in `src/app/globals.css` inside `@theme {}`, NOT in `tailwind.config.ts`
- Framer Motion — always `import { motion, type Variants } from 'framer-motion'`; use cubic-bezier arrays for ease, never string literals (`[0.25, 0.1, 0.25, 1]` not `'easeOut'`)
- Images: `public/images/` — already downloaded from Figma

## Component Map

| Component | File | Figma node |
|---|---|---|
| TopBar | `src/components/TopBar.tsx` | inside 3:3 |
| Navbar | `src/components/Navbar.tsx` | **3:650** (sibling of main frame, NOT inside 3:3) |
| HeroSection | `src/components/HeroSection.tsx` | inside 3:3 |
| TrustBar | `src/components/TrustBar.tsx` | inside 3:3 |
| FeaturedProducts | `src/components/FeaturedProducts.tsx` | inside 3:3 |
| ProductCard | `src/components/ProductCard.tsx` | inside 3:3 |
| BrandFamily | `src/components/BrandFamily.tsx` | inside 3:3 |
| SolarSection | `src/components/SolarSection.tsx` | inside 3:3 |
| StatsBanner | `src/components/StatsBanner.tsx` | inside 3:3 |
| WhyChooseUs | `src/components/WhyChooseUs.tsx` | inside 3:3 |
| Testimonials | `src/components/Testimonials.tsx` | inside 3:3 |
| CTABanner | `src/components/CTABanner.tsx` | inside 3:3 |
| Footer | `src/components/Footer.tsx` | inside 3:3 |

Static data lives in `src/lib/data.ts`; types in `src/lib/types.ts`.

## Design Tokens (resolved)

All tokens are already applied in `src/app/globals.css`. Reference values:

**Colors:**
- Brand red: `#ED1C24` (hover: `#C8151C`)
- Brand dark: `#231F20`
- Brand black: `#111010`
- Body text muted: `#6D6E71`
- Border light: `#EBEBEB` / `#F0F0F0`
- Background off-white: `#FAFAFA`

**Typography:**
- Font: Source Sans 3 (Google Fonts, loaded via globals.css)
- Section eyebrow: `text-[11px] font-extrabold tracking-[0.27em] uppercase text-[#ED1C24]`
- Section heading: `text-[38px] font-black text-[#231F20] leading-[1.5]`

**Section alignment rules (from `textAlignHorizontal` tokens):**
- CENTER sections: FeaturedProducts, BrandFamily, Testimonials → `flex flex-col items-center text-center`
- LEFT sections: WhyChooseUs → `flex flex-col` (no centering)

## Known Quirks

1. **Navbar is a standalone sibling frame** — node `3:650` sits outside the main page frame `3:3` on the Figma canvas. It's labeled "VARIANT B — CLEAN & PREMIUM". If Figma data looks wrong for the navbar, search the raw data for "Navigation" to locate it.

2. **Category tabs belong in HeroSection** — they appear at the bottom of the hero, not in the Navbar component.

3. **BrandFamily is 2-column** — Figma cards are 598px wide. Two fit in the 1216px content area (598×2 + 20px gap). Use `grid-cols-1 lg:grid-cols-2`.

4. **WhyChooseUs service cards are a vertical list** — `layout_HHVAR9` shows `mode: column, gap: 16px, width: 576px`. Use `flex flex-col gap-4`, NOT a grid.

5. **SolarSection steps** — Step 1 icon uses red bg `#ED1C24`; steps 2 and 3 use gray `#F5F5F5`.

6. **Floating cards in HeroSection** — use `absolute` positioning with negative offsets. Shadow: `shadow-[0px_4px_24px_rgba(0,0,0,0.10)]`.

## Workflow for New Sections

1. Fetch design context: `get_figma_data(fileKey="7AW0hKHcyl9yZUOvRYSA8j", nodeId="<node>")`
2. If response is large, save to file and read in chunks with `node -e "const f=require('fs').readFileSync('<file>','utf8'); console.log(f.slice(<start>,<end>))"`
3. Extract from the `styles:` block at the end: `fill_*` = colors, `style_*` = typography (check `textAlignHorizontal`!), `layout_*` = layout (check `mode`, `gap`, `padding`, `dimensions`)
4. If `mode: none` → absolute positioning using `locationRelativeToParent` x/y coordinates
5. If `mode: row/column` → flexbox
6. Reuse existing tokens from globals.css; add inline styles only for multi-stop gradients or values Tailwind can't express
7. Take screenshot with `get_figma_data` and validate against implementation before marking done
