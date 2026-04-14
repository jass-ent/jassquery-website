# JASS Query — Website Project Notes

Context for future Claude sessions working on this repo. Read once, then trust the code for specifics.

## Session start — remind the user

**At the start of every new Claude session in this project**, remind the user to check both search engine webmaster dashboards for new data, warnings, or indexing issues:

- **Google Search Console** — https://search.google.com/search-console
- **Bing Webmaster Tools** — https://www.bing.com/webmasters

Both have jassquery.com verified and the sitemap submitted. A quick weekly check surfaces impressions/click trends, structured data warnings, and crawl errors before they become problems. Mention this once at the start of a session, not repeatedly.

## What this is

Marketing site for **JASS Query**, the umbrella brand for two Amazon-reseller tools:
- **QueryVault** — free browser extension (Chrome + Edge), already live on both stores
- **QueryForge** — SaaS automation service, waitlist-only, launching soon

The master landing page (`index.html`) introduces the brand and routes users to either product. Each product has its own detail page. A unified privacy policy covers all three scopes (website, QueryVault, QueryForge waitlist). Content, tone, and structure were built up iteratively — the existing HTML is the authoritative spec; don't second-guess copy without asking.

## Repo layout

```
repo-root/
├── CLAUDE.md              ← this file
├── .gitignore
├── wrangler.toml          ← Cloudflare Workers config, points at site/
├── docs/                  ← internal brand sheets, color masters — NOT deployed
│   └── og-templates/      ← SVG + Affinity (.af) source for OG social images
└── site/                  ← everything that ships to production
    ├── index.html         ← master JASS Query landing page
    ├── queryvault.html    ← product detail page
    ├── queryforge.html    ← product detail page (waitlist-only for now)
    ├── privacy-policy.html ← unified policy with three-logo trio header
    ├── 404.html           ← branded 404 (served by Cloudflare on unmatched routes)
    ├── robots.txt         ← allow all, disallow /404.html, points at sitemap
    ├── sitemap.xml        ← lists the 4 public pages (update lastmod on content passes)
    ├── .well-known/
    │   └── security.txt   ← RFC 9116, contact help@jassquery.com
    └── assets/
        ├── img/           ← all logos, PNGs, SVGs
        │   └── og/        ← Open Graph social share images (1200×630 PNG, one per product)
        ├── css/           ← empty (all styles are inline in the HTML files)
        └── js/            ← empty (all scripts are inline in the HTML files)
```

**Important structural rule:** `site/` is the deploy root. The `docs/` folder (brand sheets, color master) lives at the repo root and is never shipped. Don't move files between them without understanding the split.

**Note on styles/scripts:** Every page is self-contained — CSS in a `<style>` block in `<head>`, JS in a `<script>` block before `</body>`. There are no shared stylesheets or JS files. This is intentional (static simplicity, Cloudflare edge caching friendly). If you need to change a shared visual pattern, you'll touch multiple files.

## Deploy flow (CI/CD)

**Push to `main` on GitHub → Cloudflare Workers Builds auto-deploys in ~2 minutes.**

- Hosted on Cloudflare as a **Worker with static assets** (NOT a classic Pages project). The project is named `jassqueryhome` in Cloudflare. This matters because the deploy path is `npx wrangler deploy` reading `wrangler.toml`, not the classic Pages build flow.
- `wrangler.toml` at the repo root contains `name = "jassqueryhome"` and `[assets] directory = "./site"`. It also sets `not_found_handling = "404-page"` so Cloudflare serves `site/404.html` for any unmatched route. Don't change the name unless you're intentionally migrating to a new Worker.
- Custom domains: **both `jassquery.com` and `www.jassquery.com`** are bound to the Worker. If either stops resolving, check Cloudflare → Worker Settings → Domains & Routes first.
- No build command runs — it's pure static HTML. `wrangler deploy` just uploads the assets in `./site`.

**Workflow rule:** small low-risk changes (CSS tweaks, copy fixes, logo swaps) commit straight to `main`. Anything bigger — structural changes, new features, anything touching multiple pages simultaneously, anything you'd be uncomfortable shipping while stepping away — branch first, get a preview URL, merge when clean.

## Brand system

### Colors (defined as CSS variables in each file)

| Purpose | Token | Hex | Usage |
|---|---|---|---|
| Master brand | `--steel` | `#5b8fa8` | JASS Query primary, nav accents, Step 1 |
| | `--steel-mid` | `#7aaec4` | hover brighten |
| | `--steel-lt` | `#a4ccd9` | highlights, text on dark |
| QueryVault | `--teal` | `#4a9e87` | Vault accents, Step 2 |
| | `--teal-mid` | `#6db8a2` | |
| | `--teal-lt` | `#9fd4c4` | |
| QueryForge | `--mauve` | `#8e6e8a` | Forge accents, Step 3 |
| | `--mauve-mid` | `#a888a4` | |
| | `--mauve-lt` | `#c4aac0` | |
| Neutrals | `--ink` | `#2a2a2a` | dark backgrounds |
| | `--stone` | `#d4d1cc` | light backgrounds |
| | `--warm` | `#eceae6` | light text on ink |

The color master at `docs/jassquery-color-master_final.html` is the authoritative reference. Don't introduce new accent colors without checking it.

### Typography

- **Headings:** Syne (400/600/700/800) — used for H1/H2, wordmarks, eyebrows, button labels, tags
- **Body:** DM Sans (300/400/500) — used for paragraphs and UI copy
- Both loaded from Google Fonts via `<link>` in each file's `<head>`

### Logos

Located in `site/assets/img/`:
- `jassquery.detailed.png` — master JASS Query mark (used in nav, footer, privacy policy trio)
- `jassquery-detailed.svg` — SVG version of the master mark (alternative)
- `queryvault-detailed.svg` — QueryVault cocoon (detailed)
- `queryvault-simple.svg`, `queryvault-minimal.svg` — variants
- `queryforge-detailed.svg` — QueryForge bloom (detailed)
- `queryforge-simple.svg`, `queryforge.minimal.svg` — variants
- `jassent-detailed.580.png` — JASS Enterprises corporate logo (footer only)

**Gotcha:** `jassent-detailed.580.png` has transparent padding around a small central mark. When placing it alongside `jassquery.detailed.png` at the same visual size, bump its `width`/`height` attributes by ~60–70% (e.g., if jassquery is 24px, jassent is 40px) so the rendered marks appear the same size. This is already handled in all four footers.

## Shared design vocabulary

The whole site uses the same hover language so interactions feel like one system. When adding new interactive elements, reuse these patterns:

1. **Lift + accent shadow.** Any hoverable card, pill, or button rises 2–8px on hover with a **color-matched compound shadow** — a drop shadow in the product's accent hue plus a 1px inner ring. Examples: `.product-card`, `.story-stat`, `.family-node`, `.constel-node`. Pattern: `box-shadow: 0 Npx Mpx -Xpx rgba(accent, 0.5), 0 0 0 1px rgba(accent, 0.15)`.

2. **Border brightens to accent.** Resting border is low-alpha neutral; hover raises alpha and swaps hue to the product's accent color.

3. **Label letter-spacing breathes.** Small uppercase eyebrows/tags open from `0.16em` to `0.22em` on hover. Editorial detail, not gaudy.

4. **Cubic-bezier 0.2, 0.8, 0.2, 1.** This is the spring-ish easing used everywhere for transform transitions. Don't substitute without reason.

5. **Every interactive element has its own accent color:** steel for JASS Query / step 1, teal for QueryVault / step 2, mauve for QueryForge / step 3. Never mix.

6. **Hover-only animation.** Ambient motion is avoided. Micro-interactions trigger on hover/mousemove and rest otherwise. The hero sunbursts use JS geometric hit-testing to activate *only* the specific glow under the cursor — a deliberate gamification pattern (see `index.html` mousemove handler). Don't add always-on animations to the hero or background.

7. **`@media (prefers-reduced-motion: reduce)`** disables animations for users with the OS preference set. Honor this when adding new motion.

The guiding principle: **motion as reward, not obstacle.** Every effect should make the page feel alive, never in the way. The user explicitly framed this as "hunt the animations" — tiny rewards for exploration. Don't add anything that announces itself; add things that reward curiosity.

## Tone and copy

- **"Don't take ourselves too seriously."** The reassurance panels ("Seriously, it's free. No trial timer, no upsell nag...") and step 3 label ("Let it run") are intentionally conversational. Formal marketing-speak is wrong here. Match the register.
- **Audience:** Amazon resellers using Keepa's Product Finder, many in a 65,000+ seller community. Authentic trade vocabulary lands; vague SaaS-speak doesn't.
- **Origin story matters.** Built by an 8-year Amazon/Keepa veteran, shaped by the seller community. Keep this credibility thread visible on the landing page.
- **Help email:** `help@jassquery.com` — used in the privacy policy contact section. Do not reference `jassent.com` anywhere in the public site (deliberately removed during the April 2026 pass).

## SEO and social sharing

All four live pages (`index.html`, `queryvault.html`, `queryforge.html`, `privacy-policy.html`) ship with a complete SEO/social baseline. When adding new pages or editing heads, preserve this pattern:

- **Canonical URL** — absolute `https://jassquery.com/...` per page
- **Open Graph tags** — `og:type`, `og:site_name`, `og:locale`, `og:url`, `og:title`, `og:description`, `og:image` (absolute URL), `og:image:width=1200`, `og:image:height=630`, `og:image:alt`
- **Twitter Card tags** — `twitter:card=summary_large_image`, plus `twitter:title`, `twitter:description`, `twitter:image`
- **JSON-LD structured data** — `Organization` on index, `SoftwareApplication` on both product pages (BrowserApplication for QueryVault, BusinessApplication with `availability: PreOrder` for QueryForge). Privacy policy has no JSON-LD.
- **Discovery files** — `robots.txt`, `sitemap.xml`, and `.well-known/security.txt` all live in `site/` and ship to prod. The 404 page is deliberately excluded from the sitemap and disallowed in robots.

**Both Google Search Console and Bing Webmaster Tools are verified and receiving the sitemap.** If you change URLs, titles, or structured data, expect GSC to surface warnings within a few days — check it after significant content passes.

### OG image workflow

The three product OG images (`site/assets/img/og/og-jassquery.png`, `og-queryvault.png`, `og-queryforge.png`) were composited in Affinity Designer from SVG templates at `docs/og-templates/`. To update an image:

1. Open the relevant `.af` file (or `.svg` template) in Affinity
2. Edit the logo, headline, or accent — keep the 1200×630 canvas and the existing layout
3. Export as PNG-24 to `site/assets/img/og/` with the same filename
4. If the image URL changed, re-run the page through Facebook's OG debugger and LinkedIn's Post Inspector to force cache refresh (LinkedIn caches for ~7 days)

### Keyword strategy

The site targets **Amazon online arbitrage and wholesale sellers using Keepa's Product Finder** — a narrow, high-intent niche. The strategy is explicit:

- **Gold (target aggressively):** `Keepa Product Finder`, `save Keepa queries`, `automate Keepa`, `online arbitrage tools`, `wholesale sourcing Amazon`. These appear in titles, H1s, meta descriptions, and body copy across the site.
- **Silver (weave naturally):** `Chrome extension for Amazon sellers`, `Amazon FBA sourcing`, `Amazon reseller tools`. Present in body copy and structured data but not in titles.
- **Bronze (appear, don't chase):** Broad terms like `Amazon FBA sourcing software`, `product research tool Amazon`. Let them appear in body copy if natural; don't optimize titles or H1s around them — too competitive, wrong intent match for new visitors.
- **Do NOT target `Keepa alternative`** — the products *require* Keepa, not replace it. Ranking for this term would mismatch searcher intent, confuse real users, and risk the relationship with Keepa. Use `Keepa add-on`, `Keepa companion`, or `Keepa Product Finder tools` instead.

When editing body copy, the "don't take ourselves too seriously" voice comes first. Keyword density is secondary. If a keyword insertion would make a sentence sound like marketing boilerplate, skip it.

## Known quirks and gotchas

- **CRLF warnings on git add** are expected and harmless. Windows LF→CRLF conversion.
- **Empty `main = ""` in wrangler.toml breaks wrangler 4.x** — it interprets the empty string as `"."` and fails with "points to a directory." For assets-only Workers, **omit the `main` field entirely.** The current `wrangler.toml` is correct — don't add it back.
- **QueryForge H1 is narrow on both ends** — the text column is ~616px wide on desktop (1100px container − 420px card − 4rem gap) and the h1 has explicit `<br>` breaks, so the clamp max is capped at `3.4rem` to keep "Running themselves." and "Your Keepa queries." on single lines. The `word-wrap:break-word; overflow-wrap:break-word; hyphens:auto` properties only live inside `@media(max-width:600px)` — NEVER add them to the base `.hero h1` rule, or desktop will break mid-word with hyphens. **If you touch the queryforge hero h1, verify on BOTH a 360px mobile viewport AND a 1200px+ desktop viewport** — this has regressed twice.
- **`.well-known/security.txt` Expires field must be kept in the future.** Currently set to `2027-04-14T00:00:00.000Z`. When it gets within ~2 months of expiring, bump the date forward one year. Expired security.txt files are treated as invalid by some scanners.
- **Local DNS caching** — when changing DNS or adding hostnames, browsers and routers can hold stale NXDOMAIN for 15+ minutes. Cellular data on a phone is the fastest way to bypass it for testing.
- **PrivacyPolicy filename** — normalized to `privacy-policy.html` (kebab-case). Earlier versions used `PrivacyPolicy.html` or `privacypolicy.html`. All references should be the kebab-case form.

## Privacy policy scope

The current policy at `site/privacy-policy.html` covers **three scopes** explicitly: the marketing website, QueryVault (browser extension), and the QueryForge waitlist (name + email via Web3Forms). It does **not** cover QueryForge as a live service — when Forge launches as real SaaS with accounts, API keys, scheduled jobs, billing, etc., a new section will need to be added and the "Last Updated" date bumped. There's a forward-looking paragraph in Section 04 noting this.

## Future QueryForge launch — what will change

When QueryForge becomes an actual running service (not just a waitlist), this site will need:
1. **Privacy policy update** — new section describing account data, Keepa API key handling, job history, billing, logs
2. **QueryForge detail page CTAs** — swap "Join waitlist" for real sign-up / login buttons
3. **Index page CTAs** — mirror the above on index.html's product card and final CTA section
4. **Nav CTA** — `nav-cta btn-disabled "Coming Soon"` should become a live link (currently hidden on mobile)
5. **Coming-soon banner at top of queryforge.html** — remove the announcement bar
6. **QueryForge JSON-LD schema** — change `availability` from `https://schema.org/PreOrder` to `https://schema.org/InStock`, add real `offers.price`, and update `og:description` / `twitter:description` / meta description to remove "Waitlist open" language
7. **Sitemap `lastmod`** — bump `sitemap.xml` dates on any page touched during the launch pass

None of these are urgent until launch is imminent.
