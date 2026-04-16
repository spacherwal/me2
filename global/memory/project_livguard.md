---
name: Livguard React Page
description: Figma design → React component conversion for Livguard brand site revamp; user plans to continue in a separate React project
type: project
---

Built a full-page React component (`LivguardPage.jsx`) from a Figma design for the Livguard brand site revamp.

**Figma URL:** https://www.figma.com/design/7AW0hKHcyl9yZUOvRYSA8j/Untitled?node-id=3-2&t=UMCEVUjngVFz5iYu-1  
**Figma file key:** `7AW0hKHcyl9yZUOvRYSA8j`, node `3-2`

**Why:** User is converting this Figma design into a real React project; started in `D:\Project\learn-node` as a scratch workspace and plans to move the component to a proper React project.

**How to apply:** When the user resumes in another project, use the Figma file key above with `get_figma_data` to fetch remaining section details. The component is self-contained with inline styles — no CSS framework needed.

**Design tokens (exact from Figma):**
- Primary red: `#ED1C24` (gradient to `#760709`)
- Near-black: `#231F20`
- Footer dark: `#111010`
- Light section bg: `#FAFAFA`
- Font: `Source Sans 3` (weights 400, 600, 700, 800, 900)
- Horizontal padding: 95px outer / 127px inner sections

**Sections built:**
1. Navbar — logo, nav (Shop, Our Brands, Solar, Services, Support), search, wishlist/account/cart
2. Hero — headline + two CTAs + image placeholder
3. Category Bar — horizontal scrollable tabs
4. Products Grid — 4-column cards
5. Stats Bar — red bg; 🏆 30+ Years | 🏠 8M+ Homes | 🔧 5000+ Service Centers | ⚡ 99.9% Uptime
6. Why Livguard — 2-col layout + 4 feature cards
7. Testimonials — 3-col review cards
8. CTA Banner — dark strip
9. Footer — brand col + Shop/Services/Help link columns + copyright bar

**Figma nodes not yet detailed** (placeholder content used):
- Hero banner images/slider (node `3:20`)
- Product sections (nodes `3:141`, `3:246`, `3:325`, `3:420`, `3:514`)
- Blog/news section (node `3:570`)
