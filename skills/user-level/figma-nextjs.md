# Figma → Next.js Implementation Skill

Convert a Figma design into production-ready Next.js code with pixel-perfect accuracy.

## Usage

Provide a Figma URL:
```
/figma-nextjs https://figma.com/design/<fileKey>/<name>?node-id=<id>
```

---

## Step 1: Pre-flight — Read Project Context

Before touching Figma, understand the project:

1. Read `CLAUDE.md` and `AGENTS.md` if they exist
2. Detect versions from `package.json`:
   - **Next.js version** — determines App Router vs Pages Router, available APIs
   - **Tailwind version** — v3 vs v4 have completely different setups (see below)
   - **React version** — v19 has new APIs
3. Read `node_modules/next/dist/docs/` for any Next.js version you haven't worked with — breaking changes are common
4. Check existing component directory for reusable primitives before creating new ones

---

## Step 2: Parse the Figma URL

From `https://figma.com/design/:fileKey/:name?node-id=1-2`:
- **fileKey** = segment after `/design/`
- **nodeId** = value of `node-id` param (e.g. `3-2`)

---

## Step 3: Fetch Design Data

```
get_figma_data(fileKey="<fileKey>", nodeId="<nodeId>")
```

If the response is large (>50KB), save it to a temp file and read in chunks:
```bash
# Save
node -e "require('fs').writeFileSync('/tmp/figma.txt', '<data>')"

# Read chunks (8,000–16,000 chars at a time)
node -e "const f=require('fs').readFileSync('/tmp/figma.txt','utf8'); console.log(f.slice(0, 12000))"
node -e "const f=require('fs').readFileSync('/tmp/figma.txt','utf8'); console.log(f.slice(12000, 24000))"
# ... continue until you've read the full file
```

Also fetch a screenshot for visual validation:
```
get_figma_data(fileKey="<fileKey>", nodeId="<nodeId>")  # with screenshot=true if supported
```

---

## Step 4: Extract Design Tokens

The `styles:` block at the **end** of the Figma data file contains all resolved tokens.

| Prefix | Meaning | Key fields |
|---|---|---|
| `fill_*` | Colors | `color` (hex) |
| `style_*` | Typography | `fontFamily`, `fontSize`, `fontWeight`, `lineHeight`, `textAlignHorizontal` |
| `layout_*` | Layout | `mode`, `gap`, `padding*`, `dimensions` (width/height), `locationRelativeToParent` (x/y) |

**Critical — always check `textAlignHorizontal`:**
- `CENTER` → apply `flex flex-col items-center text-center` to the container
- `LEFT` → no centering classes
- Getting this wrong is the #1 cause of alignment mismatches

---

## Step 5: Infer Layout from Figma Tokens

### Auto Layout nodes (`mode: row` or `mode: column`)
Map directly to flexbox:
```
mode: row   → flex flex-row
mode: column → flex flex-col
gap: 16     → gap-4
paddingLeft: 24, paddingRight: 24 → px-6
```

### Absolute positioning nodes (`mode: none`)
Use `locationRelativeToParent` x/y coordinates:
```
position: absolute; left: <x>px; top: <y>px
```
In Tailwind: `absolute left-[<x>px] top-[<y>px]` or inline style if values are dynamic.

### Detecting grid column count
Use card `dimensions.width` and `locationRelativeToParent.x`:
- Card 1 at x:0, width:598 → first column
- Card 2 at x:619, width:598 → second column
- `598×2 + 19gap ≈ 1215` → 2-column grid
- If card 2 x < card 1 width + small gap → same row → multi-column

### Standalone frames (the "sibling trap")
A section might be a **sibling** of the main page frame on the canvas, not a child. This commonly happens with Navbars. Signs:
- The section looks visually connected to the page but isn't in the main frame's children list
- Its node ID resolves fine but the data doesn't appear in the page tree

Fix: Search the raw Figma data for distinctive text (e.g., the nav link labels) to find the actual node ID.

---

## Step 6: Tailwind Setup (version-specific)

### Tailwind v4 (≥4.0)

```css
/* globals.css */
@import "tailwindcss";

@theme {
  --color-brand-red: #ED1C24;
  --font-family-sans: 'Source Sans 3', sans-serif;
  /* ... all tokens as CSS custom properties */
}
```

```js
// postcss.config.mjs (required!)
export default { plugins: { '@tailwindcss/postcss': {} } }
```

- NO `tailwind.config.ts` — it's not used in v4
- Arbitrary values still work: `text-[#ED1C24]`, `rounded-[12px]`
- Underscores encode spaces in shadows: `shadow-[0px_4px_24px_rgba(0,0,0,0.10)]`

### Tailwind v3

```js
// tailwind.config.ts
export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: { 'brand-red': '#ED1C24' }
    }
  }
}
```

```js
// postcss.config.js
module.exports = { plugins: { tailwindcss: {}, autoprefixer: {} } }
```

---

## Step 7: Framer Motion TypeScript

**Always use `type Variants`** — string ease values cause TS errors:

```tsx
// CORRECT
import { motion, type Variants } from 'framer-motion'

const variants: Variants = {
  hidden: { opacity: 0, y: 16 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.5, ease: [0.25, 0.1, 0.25, 1] }  // cubic-bezier array
  }
}
```

```tsx
// WRONG — TS error: Type 'string' is not assignable to type 'Easing | Easing[] | undefined'
transition: { ease: 'easeOut' }
```

---

## Step 8: Code Conventions

- **Inline styles**: Only for values Tailwind can't express — multi-stop gradients with percentage stops, `style={{ background: 'linear-gradient(...)' }}`
- **Images**: Use `next/image` with `fill` + `sizes` for responsive images in known-dimension containers
- **Links**: Use `next/link` for internal navigation
- **Client components**: Add `'use client'` only when using hooks or browser APIs
- **Static data**: Keep arrays/objects in `src/lib/data.ts`; interfaces in `src/lib/types.ts`

---

## Step 9: Validation Checklist

Before marking a section complete, verify against the Figma screenshot:

- [ ] Layout matches (flex direction, grid columns, gaps, padding)
- [ ] Section header alignment (center vs left — derived from `textAlignHorizontal`)
- [ ] Typography (size, weight, color, line height)
- [ ] Colors exact (use hex values from `fill_*` tokens)
- [ ] Spacing (padding/margin from `layout_*` tokens)
- [ ] Interactive states (hover colors, transitions)
- [ ] Mobile responsiveness (check Figma constraints)
- [ ] No TypeScript errors
- [ ] Images render with correct aspect ratios

---

## Common Mistakes

1. **Centering headers** — always check `textAlignHorizontal` in `style_*` tokens; don't assume
2. **Grid column count** — use `locationRelativeToParent.x` coordinates, don't guess
3. **Navbar/header outside main frame** — search by text content if node isn't in the page tree
4. **Tailwind v4 config** — delete `tailwind.config.ts`, use `@theme {}` in CSS
5. **Framer Motion ease strings** — use cubic-bezier arrays, not string names
6. **`mode: none` = absolute** — don't apply flexbox to nodes with no Auto Layout
