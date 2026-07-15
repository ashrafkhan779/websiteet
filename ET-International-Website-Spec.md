# ET International DMCC — Website Specification
### Structure · Design System · Data Model · Components
Version 1.0 — for review before build

---

## 1. Executive summary

**What this site has to do.** It is not a brochure. It is an RFQ acquisition engine for a category where the buyer (plant manager, procurement lead, turbine service company) is time-poor, technically literate, and screening for one thing: *can this supplier be trusted with a part that spins at 3,000 rpm inside a machine worth eight figures?* Every design decision below is judged against a single question — does it shorten the distance between "I have a part requirement" and "RFQ submitted with drawing attached"?

**The three decisions that matter most:**

1. **The turbine explorer is the site's differentiator, and it is also a navigation system.** Competitors publish flat PDF part lists. An explorer that lets a buyer select Frame 9 → combustion section → fuel nozzles → *Request this part* collapses discovery and conversion into one motion. It earns its build cost only if it feeds the RFQ form with pre-filled context. That connection is non-negotiable.
2. **Credibility is built by restraint, not claims.** The brief is right to ban invented stats. Go further: every unverifiable field renders as an explicit, editable placeholder ("Material: available upon technical review"). A page that admits what it doesn't publish reads *more* credible to an engineer than one padded with round numbers.
3. **Two conversion paths, not one.** The 6-step RFQ is correct for a real requirement. It is fatal for a buyer with one question. Quick Enquiry (name, email, one free-text field) must sit on every page. Expect Quick Enquiry to carry 60–70% of volume at 3–5× the conversion rate; the full RFQ carries the value.

**Risk to flag now:** the brief's ban on customer names, logos and project counts removes the strongest trust signals a supplier normally has. ISO 9001:2015 plus the engineering-workflow narrative must carry that weight alone. Recommendation — start collecting written testimonial permission and logo approvals in parallel with the build; the CMS slots are specified below and the sections stay hidden until filled.

---

## 2. Platform recommendation (you said "not decided")

| Option | Verdict |
|---|---|
| **Base44** | Fastest to a live site with working RFQ database + admin. Cost: the Three.js turbine explorer and GSAP scroll choreography will be constrained or degraded. You get 100% of the CMS and ~50% of the visual ambition. |
| **Next.js + headless CMS (Sanity/Payload) on Vercel** | Full visual ambition, full SEO control (SSG per product page, schema, sitemap), file uploads to S3, transactional email via Resend. Needs a developer for 3–4 weeks. |
| **Webflow** | Good middle ground on design and CMS. Multi-step RFQ with file uploads and line-item repeaters fights the platform. Not recommended for this form. |

**Recommendation: Next.js on Vercel.** The turbine explorer and the RFQ line-item repeater are the two features that justify this rebuild, and they are exactly the two features the no-code platforms handle worst. If timeline forces Base44, cut the Three.js model and ship the SVG-layer explorer (spec'd in §6 as the mandatory fallback anyway) — it loses little.

This spec is platform-agnostic. The build I produce next will be framework-free HTML/CSS/JS so it ports to any of the three.

---

## 3. Design system

### 3.1 Palette

Derived from the subject: unpainted forged steel, machine-hall shadow, blueprint cyan, and the one place heat actually appears in a turbine.

| Token | Hex | Use |
|---|---|---|
| `--void` | `#080B10` | Page base. Near-black with a navy bias, not grey. |
| `--surface` | `#0F1620` | Section and card surfaces, one step up from base. |
| `--surface-raised` | `#16202C` | Hover states, active tabs, form fields. |
| `--steel` | `#243140` | Borders, dividers, disabled states. |
| `--steel-light` | `#8695A6` | Secondary text, labels, captions. |
| `--white` | `#F2F5F8` | Primary text and headings. Never pure `#FFF`. |
| `--cyan` | `#2FC6E0` | The single active colour: links, focus rings, blueprint lines, active zones, primary CTA. |
| `--cyan-dim` | `#1A7C8E` | Cyan at rest — inactive leader lines, chart gridlines. |
| `--heat` | `#E2622A` | **Budget: ≤5% of any viewport.** Reserved exclusively for the combustion/hot-gas-path zones in the turbine explorer and RFQ urgency states. It is not a brand colour; it is a temperature reading. |

Discipline rule: cyan is the only colour that moves. If an element is both cyan and animated, it is interactive. If it isn't interactive, it isn't cyan. This single rule does most of the UX work in the explorer.

### 3.2 Typography

Three faces, three jobs — this is the pairing that separates the site from a template.

- **Display — Manrope**, 700/800, tracking `-0.03em` on sizes ≥40px. Headlines only. Its slightly geometric, closed apertures read as engineered rather than editorial.
- **Body — Inter**, 400/500, `1.65` line-height, max measure 68ch. Chosen for screen legibility at 16px on the shop-floor laptops and tablets this audience actually uses.
- **Utility — IBM Plex Mono**, 400/500, uppercase, tracking `0.12em`. This is the signature typographic move: **every machine-readable value is monospaced** — part numbers, RFQ reference numbers, frame designations, spec labels, section eyebrows, form field labels, table data. It mirrors an engineering drawing's callout convention and gives the site a technical fingerprint no Inter-only competitor has.

**Scale** (1.25 ratio, clamped and fluid):

```
display-xl  clamp(44px, 6vw, 76px)   Manrope 800   hero only
display-l   clamp(34px, 4vw, 52px)   Manrope 700   page titles
h2          clamp(26px, 2.6vw, 36px) Manrope 700   section heads
h3          20px                     Manrope 700   card titles
body-l      18px                     Inter 400     intros
body        16px                     Inter 400     default
caption     14px                     Inter 400     helper text
mono-label  12px                     Plex Mono 500 eyebrows, labels, data
```

### 3.3 Space, grid, geometry

- Base unit **8px**; sub-unit 4px. Section vertical rhythm: `120px` desktop / `72px` mobile.
- Grid: 12 columns, `1280px` max content width, `24px` gutters, `20px` page margin on mobile.
- **Radius: `2px` on inputs and buttons, `4px` on cards. Nothing higher, ever.** The brief bans rounded cards; precision geometry is the point.
- Borders: `1px solid var(--steel)`. Elevation is expressed with borders and 1px inner highlights, not soft shadows — brushed metal has edges, not blur.
- Glass effect: permitted on exactly two elements — the scrolled header and the explorer's floating info panel. Nowhere else.

### 3.4 Motion

| Token | Value |
|---|---|
| `--ease` | `cubic-bezier(0.22, 1, 0.36, 1)` |
| `--dur-fast` | `180ms` (hover, focus) |
| `--dur-base` | `320ms` (panels, tabs) |
| `--dur-slow` | `700ms` (scroll reveals) |
| Reveal | 16px rise + fade, `IntersectionObserver`, one-shot, staggered 60ms |

Hard rules: nothing loops except the turbine rotation and the supply-map arcs. `prefers-reduced-motion: reduce` disables all transforms and the Three.js scene, substituting static renders — this is a real accessibility requirement, not a checkbox. No page-load turbine spinner over 400ms; first contentful paint is worth more than theatre.

### 3.5 Signature element

**The blueprint callout.** Cyan leader lines drawn (SVG `stroke-dashoffset`) from a component to a monospace label block, exactly as on a technical drawing. It appears in the hero (labels resolving around the rotor on load), in the explorer (zone → spec panel), and on product detail galleries. One idea, executed in three places, is what makes a site feel authored rather than assembled.

---

## 4. Information architecture

```
/                               Home
/about                          About Us
/products                       Products index (filter + search)
/products/[slug]                Category detail  ×12
/turbine-platforms              Platforms overview
/turbine-platforms/[slug]       Frame 3/5/6/7/9, steam, custom
/services                       Services & Capabilities
/services/[slug]                Service detail  ×12
/quality                        Quality & Certification
/industries                     Industries We Support
/request-a-quote                RFQ (6 steps)
/request-a-quote/confirmation   Reference number screen
/contact                        Contact
/privacy                        Privacy Policy
/404                            Custom 404
/admin/*                        Authenticated CMS
```

**Homepage sequence** (per brief): hero → trust bar → introduction → turbine explorer → product grid → engineering process → services → quality → industries → supply network → logos *(hidden)* → testimonials *(hidden)* → RFQ CTA → footer.

**Conversion paths:** header CTA (persistent) · hero primary · every product card ("Add to RFQ") · explorer zone panel ("Request this part") · quick enquiry in footer and product sidebars · contact page.

---

## 5. Data model

Collections, with the fields that matter. `†` = required before public display.

**`turbine_platforms`** — `slug†`, `name†` (Frame 9), `designation` (mono display), `type` (gas|steam|custom), `summary†`, `hero_image`, `component_zones[]` → refs, `related_categories[]`, `seo` → embed, `display_order`, `published`

**`component_zones`** — `slug†`, `label†` (Combustion section), `platform` → ref, `svg_path_id†` (binds zone to explorer geometry), `description†`, `typical_application`, `engineering_support`, `categories[]` → refs, `is_hot_section` (bool → drives `--heat`)

**`product_categories`** — `slug†`, `name†`, `one_liner†`, `overview`, `key_applications[]`, `platforms[]` → refs, `zone` → ref, `service_types[]`, `images[]`, `spec_fields[]` → `{label, value, is_placeholder}`, `quality_notes`, `related[]` → refs, `spec_sheet` → doc ref, `seo`, `display_order`, `published`
> `spec_fields.is_placeholder = true` renders the value in `--steel-light` italic with a "confirm with technical team" tooltip. This is the mechanism that keeps unverified data visibly unverified.

**`services`** — `slug†`, `name†`, `icon`, `description†`, `benefits[]`, `related_categories[]`, `display_order`, `published`

**`documents`** — `title†`, `type` (certificate|policy|brochure|catalogue|spec_sheet), `file†`, `is_public`, `verified_by`, `verified_at`
> Nothing renders with `is_public = false` or `verified_at = null`. This enforces the brief's certificate rule at the data layer rather than by convention.

**`testimonials`** — `customer_name†`, `position`, `company`, `quote†`, `photo`, `permission_granted` (bool)†, `published` (bool)†
> Section renders only where `count(permission_granted && published) > 0`. Otherwise the whole block is absent from the DOM — not empty, absent.

**`customer_logos`** — `company†`, `logo_mono†`, `logo_colour`, `approval_confirmed`†, `display_order`, plus a global `settings.show_logo_carousel` kill switch.

**`rfqs`** — `reference†` (`ETI-RFQ-YYMM-####`), `status†` (enum: new | under_review | technical_clarification | supplier_enquiry | quotation_in_preparation | quotation_sent | won | lost | closed), `contact{name, company, job_title, email, phone, country}`, `turbine{type, platform, manufacturer, site_name, delivery_location}`, `line_items[]` → refs, `files[]` → refs, `requirements{certification, inspection, incoterms, comments, preferred_contact}`, `consent_given†`, `source_page`, `prefill_context` (which zone/product sent them), `internal_notes[]` (staff-only), `created_at`, `assigned_to`

**`rfq_line_items`** — `rfq` → ref, `part_name†`, `part_number`, `quantity†`, `unit†`, `required_date`, `description`

**`rfq_files`** — `rfq` → ref, `filename`, `storage_key`, `mime`, `size`, `virus_scanned` — private bucket, signed URLs only, never public.

**`enquiries`** — quick-enquiry captures: `name†`, `email†`, `message†`, `source_page`, `status`, `created_at`

**`site_content`** — singleton: hero copy, intro copy, trust-bar items, business hours, linkedin_url (hidden until valid), analytics IDs (GA4, Meta Pixel, LinkedIn Insight), trademark disclaimer text.

**`seo_meta`** — per-route: `title†`, `description†`, `og_image`, `canonical`, `noindex`.

**Status workflow** (admin only): `new → under_review → technical_clarification → supplier_enquiry → quotation_in_preparation → quotation_sent → won | lost | closed`. Every transition timestamped and attributed. This gives you a measurable RFQ→quote→won funnel from day one — see §9.

---

## 6. Component library

**Global:** `Header` (transparent → solid at 80px scroll, glass blur) · `MobileMenu` (slide-in, focus-trapped, contact block in footer of panel) · `Footer` (+ animated turbine line graphic) · `CTABand` · `Breadcrumbs` (+ schema)

**Content:** `SectionHead` (mono eyebrow + Manrope h2) · `TrustBar` (5 items, staggered reveal) · `ProcessFlow` (9-step scroll-drawn line — About + Home) · `QualityPathway` (6-step, same primitive) · `IndustryCard` · `SupplyMap` (SVG world, ~30KB, 6 animated arcs from Dubai, pauses off-screen and under reduced-motion) · `LogoCarousel` (CMS-gated) · `Testimonials` (CMS-gated) · `DocumentList`

**Product:** `ProductCard` (image zoom + blueprint outline draw on hover, "View details" + "Add to RFQ") · `ProductGrid` · `FilterBar` (category · platform · zone · service type, URL-synced so filtered views are shareable and indexable) · `SearchField` (client-side fuzzy) · `SpecTable` (renders placeholders per §5) · `Gallery`

**Turbine explorer** — the centrepiece:
- `PlatformTabs` — 7 tabs, mono labels
- `TurbineStage` — Three.js layered rotor on desktop (`≥1024px`, WebGL available); **mandatory fallback:** layered SVG cross-section with the identical zone hit-map. The SVG version is the source of truth for interaction, the 3D is enhancement. Build SVG first.
- `ZoneHotspot` — hover: cyan outline draw + leader line; hot-section zones use `--heat`
- `ZonePanel` — glass panel: category · technical description · typical application · engineering support · **View products** · **Request this part** (deep-links `/request-a-quote?platform=frame-9&zone=combustion&category=fuel-nozzles` and pre-fills step 2 and line item 1 — this is the whole point of the section)
- `MobileExplorer` — swipeable diagram + accordion, no WebGL, <100KB

**Forms:** `RFQWizard` (6 steps, progress rail, per-step validation, **draft persisted to localStorage** — a procurement lead who leaves to find a drawing must not lose 5 steps of work) · `LineItemRepeater` · `FileDropzone` (PDF/XLSX/DOCX/JPG/PNG, 10MB each, 10 files, client + server type check) · `QuickEnquiry` · `ConfirmationScreen` (reference number in mono, large, copyable) · `ConsentCheckbox` (required, links `/privacy`) · honeypot + timing check + rate limit + Turnstile.

---

## 7. Content that requires client confirmation

Nothing below gets invented. Flag each to the client before launch:

- Years of operation, company founding date, any project/delivery counts
- Any customer name, logo, or testimonial (+ written permission)
- Material grades, dimensional specs, part numbers, applicable model lists
- Repair / overhaul / field-service capability claims — **excluded by default per brief**
- Lead-time figures, stocking commitments under LTPAs
- Exact office/unit number in JLT Cluster O
- LinkedIn URL · business hours · ISO certificate PDF (to upload and verify)
- GE trademark disclaimer — final wording to be approved

**Verified and usable now:** ET International DMCC · Cluster O, JLT, Dubai, UAE · sales@etinternational.ae · +971 4 589 7122 · fax +971 4 587 7402 · ISO 9001:2015 · gas and steam turbine spare parts, OEM-equivalent.

**Language rules enforced in copy:** "OEM-equivalent spare-parts supplier" — never "Original Equipment Manufacturer" for ET itself. GE-designed frames referenced descriptively only, with the disclaimer rendered on every page carrying a frame reference. No superlatives.

---

## 8. SEO structure

- Static generation per product and platform page; unique `title` / `description` from `seo_meta`, never templated.
- Schema: `Organization` (site-wide) · `Product` (each category) · `BreadcrumbList` · `ContactPoint` · `WebSite` + `SearchAction`.
- Primary targets mapped to owners: `/products/compressor-rotor-blades` → "compressor blades supplier"; `/turbine-platforms/frame-9` → "Frame 9 turbine parts"; `/services/reverse-engineering` → "reverse-engineered turbine components"; `/` → "gas turbine spare parts supplier Dubai".
- The filtered product views are the long-tail engine: `/products?platform=frame-5&category=fuel-nozzles` maps to real buyer queries. Canonicalise to the parent, but keep them crawlable.
- Sitemap · robots.txt · OG images per page · Search Console verification slot · GA4 / Meta Pixel / LinkedIn Insight fields in `site_content`.

---

## 9. Measurement

Instrument from day one — this site should tell you which frame platforms and which components generate demand, which is commercial intelligence the current site does not produce.

| Metric | Target |
|---|---|
| Quick Enquiry conversion | 3.5–5% of sessions |
| Full RFQ start rate | 1.5–2.5% |
| RFQ step-6 completion (of starts) | ≥60% — below 45% means the wizard is too long, cut step 5 |
| Explorer engagement → RFQ | ≥15% of explorer users |
| RFQ → quotation_sent | tracked via status workflow |
| quotation_sent → won | tracked; the number that decides the site's ROI |
| Lighthouse (mobile) | ≥90 performance, 100 accessibility |
| LCP | <2.0s (buyers on plant-site connections) |

Events: `rfq_start`, `rfq_step_complete` (n), `rfq_submit`, `quick_enquiry_submit`, `explorer_zone_select` (platform, zone), `product_add_to_rfq`, `spec_sheet_download`, `email_copy`, `phone_click`.

---

## 10. Build sequence

1. Design tokens + global shell (header, footer, mobile menu, 404)
2. Homepage: hero, trust bar, intro
3. **Turbine explorer — SVG version** (interaction contract locked here)
4. Product grid, filters, search; one category detail page as the pattern
5. RFQ wizard + confirmation + reference generation
6. Remaining homepage sections; process and quality flows
7. About, Services, Quality, Industries, Contact, Privacy
8. Remaining 11 category pages, 7 platform pages
9. Three.js enhancement layer + reduced-motion fallbacks
10. SEO, schema, analytics slots, accessibility audit
11. Admin/CMS (or Base44 collections)

Steps 1–8 are what I'll build as the prototype. Steps 9–11 need the platform decision and a backend.

---

**Open questions for you before I build:**
1. Logo — do you have a vector file, or should the build use a typographic lockup?
2. Turbine imagery — licensed photography available, or do I build with rendered/abstract geometry (my recommendation: rendered, because stock turbine photography is where industrial sites look cheapest)?
3. Any of the "requires confirmation" items in §7 you can answer now?
