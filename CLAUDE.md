# JIINEVA — Shopify Theme Architecture

**Base:** Shopify Dawn (latest)
**Stack:** Liquid, CSS custom properties, Vanilla JS
**Branch convention:** `claude/<slug>` per session

---

## Brand

Fashion brand that sells jewelry. Bershka aesthetic — NOT luxury, NOT Mejuri.
Tone: direct, bold, editorial. Typography: uppercase + wide tracking for headings/labels.

---

## Directory Map

```
/assets
  jiineva-tokens.css        ← Phase 1 ✓  CSS custom properties (global)
  jiineva-global.css        ← Phase 1 ✓  Base styles, buttons, utils
  jiineva-announcement-bar.css/js   ← Phase 2
  jiineva-header.css/js             ← Phase 2
  jiineva-hero.css                  ← Phase 3
  jiineva-category-grid.css         ← Phase 4
  jiineva-the-edit.css              ← Phase 5
  jiineva-editorial.css             ← Phase 6
  jiineva-newsletter.css            ← Phase 7
  jiineva-collection.css            ← Phase 8
  jiineva-product.css               ← Phase 9
  jiineva-cart-drawer.css/js        ← Phase 10
  jiineva-search.css/js             ← Phase 11
  jiineva-analytics.js              ← Phase 12

/sections (new)
  jiineva-hero.liquid               ← Phase 3
  jiineva-category-grid.liquid      ← Phase 4
  jiineva-the-edit.liquid           ← Phase 5
  jiineva-editorial.liquid          ← Phase 6
  jiineva-newsletter.liquid         ← Phase 7

/sections (modify Dawn)
  announcement-bar.liquid           ← Phase 2  rotating messages
  header.liquid                     ← Phase 2  transparent+sticky, mobile menu
  footer.liquid                     ← Phase 8

/snippets (new)
  jiineva-wishlist-icon.liquid      ← Phase 2  Swym wrapper
  jiineva-badge.liquid              ← Phase 9  product badge
  jiineva-size-guide.liquid         ← Phase 9  ring size modal
  jiineva-chain-guide.liquid        ← Phase 9  chain length modal
  jiineva-analytics.liquid          ← Phase 12 dataLayer/pixel helpers

/snippets (modify Dawn)
  card-product.liquid               ← Phase 5  hover img, wishlist, no ratings
  price.liquid                      ← Phase 5  remove sale label clutter

/layout
  theme.liquid                      ← Phase 2  import tokens+global, analytics head

/config
  settings_schema.json              ← Phase 2  JIINEVA color+header settings

/templates
  index.json                        ← Phase 3–7  homepage section order
  collection.json                   ← Phase 8
  product.json                      ← Phase 9
```

---

## Dawn Files — Touch List

| File | Change |
|------|--------|
| `layout/theme.liquid` | Add `{{ 'jiineva-tokens.css' \| asset_url \| stylesheet_tag }}` + `{{ 'jiineva-global.css' \| asset_url \| stylesheet_tag }}` before `</head>`; add analytics scripts before `</body>` |
| `config/settings_schema.json` | Append JIINEVA settings group (colors, header transparency) |
| `assets/base.css` | Override Dawn typography/spacing defaults to match tokens |
| `sections/announcement-bar.liquid` | Add rotation logic (JS-free CSS animation preferred) |
| `sections/header.liquid` | Add `data-transparent` attr; transparent→solid scroll behavior |
| `sections/footer.liquid` | Replace markup with JIINEVA 4-col footer |
| `snippets/card-product.liquid` | Hover second image swap; wishlist icon overlay; remove star ratings |

---

## Naming Conventions

- All new CSS classes: `jiineva-` prefix
- All new JS files: `jiineva-` prefix
- All new Liquid sections/snippets: `jiineva-` prefix
- CSS custom properties: `--jiineva-*`
- Dawn's existing variables (`--color-base-*`, `--font-*`): referenced, not overridden globally

---

## Metafields (all `custom` namespace, `product` owner)

| Key | Type | Usage |
|-----|------|-------|
| `custom.materials` | `single_line_text_field` | "Sterling Silver, 18K Gold Plated" |
| `custom.chain_length` | `single_line_text_field` | "16 in, 18 in, 20 in" — triggers chain guide |
| `custom.size_guide` | `multi_line_text_field` | Ring size chart content (plain text/markdown) |
| `custom.nickel_free` | `boolean` | Renders "Nickel Free" badge when true |
| `custom.care` | `multi_line_text_field` | Populates Care accordion on PDP |
| `custom.closure_type` | `single_line_text_field` | "Lobster Clasp", "Push Back", "Toggle" |
| `custom.badge` | `single_line_text_field` | "New In", "Best Seller", "Limited" — card badge |

**Setup:** Shopify Admin → Content → Metafields → Products → Add definition (repeat for each).

---

## App Integrations

### Swym Wishlist Plus
- Install from App Store
- Swym injects script via app embed — enable in Customizer → App embeds
- `snippets/jiineva-wishlist-icon.liquid` wraps `<div class="swym-add-to-wishlist-view-product">` with JIINEVA styling
- Required `data-` attrs: `data-product-id`, `data-variant-id`, `data-product-url`, `data-product-image`
- Cross-device persistence: native to Swym (no custom code needed)

### Judge.me
- Install from App Store; enable "Autopublish" + "Email reviews"
- Theme snippets auto-injected: `judgeme_widgets`
- JIINEVA rule: hide review section if `product.metafields.judgeme.reviews_count < 5`
- JIINEVA rule: hide star ratings in card if `product.metafields.judgeme.rating_count < 10`
- Liquid check: `{% if product.metafields.judgeme.reviews_count >= 5 %}...{% endif %}`

### Analytics (GA4 + Meta Pixel + Klaviyo)
- GA4: `gtag.js` loaded in `theme.liquid` `<head>`; `assets/jiineva-analytics.js` fires custom events
- Meta Pixel: base code in `theme.liquid` `<head>`; standard events (`ViewContent`, `AddToCart`, `Purchase`) via `jiineva-analytics.js`
- Klaviyo: app embed handles `identify`; newsletter section uses Klaviyo form embed ID; `_learnq.push()` calls in `jiineva-analytics.js`
- Shopify Customer Events API (`analytics.subscribe()`) used for cart/checkout events

---

## Breakpoints (Dawn standard)

| Name | Width |
|------|-------|
| mobile | < 750px |
| tablet | ≥ 750px |
| desktop | ≥ 990px |
| wide | ≥ 1200px |

---

## Performance Rules

- Section CSS loaded with `{{ 'file.css' | asset_url | stylesheet_tag }}` inside each section (Dawn pattern)
- JS deferred: `defer` or `type="module"` on all script tags
- Images: always use `image_url` filter + `widths` for responsive srcset
- No jQuery, no lodash, no external CSS frameworks
- Swym + Judge.me load via app embed (async, non-blocking)

---

## Development Sequence

| Phase | Scope | Status |
|-------|-------|--------|
| 1 | Foundation — tokens, global styles, architecture | ✓ Done |
| 2 | Announcement Bar + Header (transparent/sticky) | Pending |
| 3 | Hero Section | Pending |
| 4 | Category Discovery (2×2 grid) | Pending |
| 5 | THE EDIT (8 products, card component) | Pending |
| 6 | Editorial Campaign Block | Pending |
| 7 | Newsletter | Pending |
| 8 | Footer + Collection Pages | Pending |
| 9 | Product Pages (PDP) | Pending |
| 10 | Cart Drawer | Pending |
| 11 | Search Overlay | Pending |
| 12 | Analytics (GA4 / Meta / Klaviyo) | Pending |
| 13 | Wishlist (Swym) integration | Pending |
| 14 | Reviews (Judge.me) conditional | Pending |
| 15 | QA, Lighthouse, cross-device testing | Pending |
