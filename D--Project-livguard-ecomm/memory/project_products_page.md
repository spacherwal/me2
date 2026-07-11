---
name: Products Page Implementation
description: Status and details of the /products page built from the search API
type: project
---

We built a `/products` page accessible from the "Shop Now →" button on HeroSection (opens in a new tab).

**Why:** User wanted the Shop Now button to navigate to a product listing page fetching real data from the backend API.

**How to apply:** When continuing work on the products page, reference the files below and the API shape already handled.

## Files changed
- `src/app/products/page.tsx` — Server Component, fetches products via axios, renders grid
- `src/components/HeroSection.tsx` — Shop Now button now points to `/products` with `target="_blank"`
- `.env.local` — Contains `API_URL` and `API_TOKEN` (user-filled)

## API details
- Endpoint: `GET ${API_URL}/api/seller/count.json?id=solar&query=solar&userMobile=0987654321`
- Headers: `Accept: application/json`, `Authorization: ${API_TOKEN}`, `version: 3500000`
- Products live at: `response.data.successResponse.data.searchInstanceList`

## Key fields in each product (SearchInstance)
`id`, `title`, `brand`, `category`, `subcategory`, `thumbnail`, `imageUrl`, `offerId`, `canOrder`, `canRFQ`, `offerPrice`, `regularPrice`, `qtyLeft`, `stockAvailability`

## Products page layout
- TopBar + Navbar at top (same as home page)
- Page title bar with item count
- 4-column product grid (responsive: 1→2→3→4)
- Each card: image, discount badge, availability badge, category·brand, title, subcategory, pricing, qty left, Add to Cart / Request Quote CTA
- Footer at bottom
