# infer.blog — Implementation Guide

## Overview

Applies Craftsman Modern brand to Ghost Solo theme via Ghost Admin only.  
All settings verified against Solo source code and Ghost documentation.

**Sources referenced throughout:**
- Solo default.hbs: github.com/TryGhost/Solo/blob/main/default.hbs
- Solo index.hbs: github.com/TryGhost/Solo/blob/main/index.hbs
- Solo theme docs: solo.ghost.io/about/
- Ghost Code Injection tutorial: ghost.org/tutorials/use-code-injection-in-ghost/
- Ghost Custom Settings docs: ghost.org/docs/themes/custom-settings/
- Ghost v6 breaking changes: docs.ghost.org/changes#ghost-6-0
- Font performance: requestmetrics.com/web-performance/5-tips-to-make-google-fonts-faster/

---

## Step 1: Install Solo

Ghost Admin → Settings → Design & branding → Customize → Change theme → Official themes → **Solo** → Install → Activate

Solo is maintained in the TryGhost/Themes monorepo and receives updates alongside Ghost core releases.

---

## Step 2: Upload Brand Assets

### Publication Logo

Ghost Admin → Settings → Design & branding → Brand → **Publication logo**

Upload `infer-logo-light.png` (combined icon + "infer.blog" wordmark).

**How Ghost handles logos** (from Solo default.hbs lines 48-54):
Solo checks `{{#if @site.logo}}` — if a logo image exists, it renders `<img>` and hides the site title text. If no logo, it renders `{{@site.title}}` as text. Since our logo PNG includes the wordmark, the text won't display (which is what we want).

**Font accuracy note:** The included PNGs use Liberation Sans Bold as a proxy for DM Sans 600 (Google Fonts wasn't reachable during build). For pixel-perfect DM Sans, open `logo-exporter.html` in your browser and screenshot the rendered logos (instructions in the HTML file).

### Publication Icon (favicon)

Same Brand section → **Publication icon** → Upload `infer-favicon-192.png`

Ghost uses this for the browser favicon and Open Graph images. 192px is the standard size for PWA icons and works across all contexts.

### Cover Image

**Remove the cover image** if using "Typographic profile" header layout.  
**Keep it** if using "Side by side" (shows image beside welcome text) or "Large background" (uses it as hero background).

From Solo index.hbs lines 3-12: Solo conditionally renders the cover image based on header_section_layout. The "Typographic profile" layout uses the publication icon instead of the cover image.

---

## Step 3: Site Settings

### General

Ghost Admin → Settings → General:
- **Site title:** `infer.blog`
- **Site description:** `Learning Without Labels`

### Accent Color

Ghost Admin → Settings → Design & branding → Brand → **Accent color:** `#b8510d`

This is the single source of truth for Ghost's built-in accent system. It controls:
- Ghost Portal popup button colors (rendered in iframe, unreachable by CSS)
- .gh-primary-btn background color (Solo's subscribe button)
- Any theme CSS referencing --ghost-accent-color

Ref: ghost.org/docs/themes/members/ — Portal uses accent color for its UI.

### Navigation

Ghost Admin → Settings → Design & branding → Navigation

**Primary:** About → `/about/` · Tags → `/tags/`  
**Secondary (footer):** Your links (GitHub, etc.)

---

## Step 4: Solo Theme Settings

Ghost Admin → Settings → Design & branding → Customize

These settings are verified from Solo's default.hbs and index.hbs source code.

### Site-wide settings

**Background color → `#f6f3ef`**

This is the most important setting. Solo's inline JavaScript (default.hbs lines 19-28) reads this value, calculates YIQ luminance, and dynamically adds `has-dark-text` or `has-light-text` to the `<html>` element. The entire theme's text contrast cascades from this.

With #f6f3ef, YIQ ≈ 240 (well above the 128 dark/light threshold), so Solo will correctly apply `has-dark-text`. Do NOT override the background in Code Injection CSS — the JS runs before Code Injection loads, so it wouldn't see the override.

**Navigation layout → `Logo on the left`**

From default.hbs line 33: adds `is-head-left-logo` to body class. Other options: "Logo in the middle" (`is-head-middle-logo`), "Stacked" (`is-head-stacked`).

**Typography (Heading font / Body font) → leave both unset / default**

Ghost 5.101.1 replaced Solo's original typography dropdown ("Modern sans-serif" / "Elegant
serif" / "Consistent mono") with a built-in font picker. If you select a font here, Ghost
sets `--gh-font-heading` and `--gh-font-body` CSS variables that take precedence over our
Code Injection CSS. Leaving both unset lets our `--font-sans: 'DM Sans'` override work.

Ref: github.com/TryGhost/Ghost/issues/21644 — confirms Solo was impacted by this change.

### Homepage settings

**Header section layout → `Side by side`** (or `Typographic profile` if you don't want a cover image)

From index.hbs lines 2-55: "Side by side" adds `has-side-about` to body; "Large background" adds `is-head-transparent has-background-about`; "Typographic profile" adds `has-typographic-about` and uses publication icon instead of cover image.

**Primary header → `Learning Without Labels`**

Rendered as `.gh-about-primary` (h1 element). From index.hbs line 33: `{{{@custom.primary_header}}}` — triple braces allow HTML.

**Secondary header → `Technology, curiosity, and the things between.`**

Rendered as `.gh-about-secondary` (p element). Only shows to non-members (wrapped in `{{#unless @member}}`). From index.hbs lines 38-39.

### Post settings

**Post feed layout → `Classic`**

From default.hbs line 33: adds `has-classic-feed` to body. "Typographic" is text-only (no images), "Parallax" adds scroll effect. Classic is the standard card layout.

---

## Step 5: Code Injection

Ghost Admin → Settings → Code injection → **Site Header**

Paste the entire contents of `code-injection-header.html` → Save.

### Performance decisions documented

**Font loading:** The Google Fonts URL was trimmed from 7 DM Sans weights to 4 (400, 600, 700 regular + 400 italic) and DM Mono from 2 weights to 1 (400 only). Each unnecessary weight adds ~30ms and ~11KB. Ref: requestmetrics.com/web-performance/5-tips-to-make-google-fonts-faster/

The URL includes `display=swap` which ensures text renders immediately in a fallback font, then swaps when the custom font loads (prevents Flash of Invisible Text). Ref: web.dev/font-display/

Preconnect hints to fonts.googleapis.com and fonts.gstatic.com are included, following Ghost's own Code Injection tutorial pattern. Ref: ghost.org/tutorials/use-code-injection-in-ghost/

**CSS strategy:** Where possible, the CSS overrides Solo's custom properties (`--font-sans`, `--font-mono`) rather than overriding individual selectors. This is more efficient because it propagates through the theme's own `var()` references without needing `!important` on every property. Direct `!important` overrides are used only for properties the theme hardcodes (letter-spacing, border colors, code block styling).

**What the CSS does NOT override:**
- Background color (set via Solo admin, not CSS — see Step 4)
- Text color (Solo's JS contrast system handles this)
- Font family on most elements (handled by --font-sans variable override)
- Ghost Portal popup (iframe, unreachable by CSS)
- Button background color (handled by accent color in Admin)

---

## Step 6: Create About Page

Ghost Admin → Pages → New page → Title: `About` → Slug: `about` → Publish  
Confirm it's in primary navigation (Step 3).

---

## Step 7: Clean Up

1. Delete the "Coming soon" placeholder post
2. Add tags and excerpts to published posts via post settings sidebar

---

## Step 8: Author Profile

Ghost Admin → Settings → Staff → your profile → Photo, bio, social links.

---

## Step 9: Verify

- [ ] Homepage: parchment bg, DM Sans, copper border under .gh-about, monospace dates
- [ ] Homepage: secondary header text and subscribe form visible (requires members enabled)
- [ ] Post: warm code blocks, copper blockquote borders, DM Mono inline code
- [ ] About page: renders with warm styling
- [ ] Mobile: responsive (Solo handles this natively)
- [ ] Subscribe: enter email → receive magic link email → confirm → appears in Members list
- [ ] Subscribe: Portal popup shows copper accent
- [ ] Browser tab: favicon shows Missing Piece icon

---

## v6 Compatibility

Ghost v6 breaking changes (docs.ghost.org/changes#ghost-6-0):
1. API limit=all deprecated (100 max) → no impact on CSS
2. AMP removed → no impact
3. Custom fonts feature enhanced → our --font-sans override is compatible; Ghost's --gh-font-heading / --gh-font-body variables take precedence when custom fonts are selected in Admin, but don't conflict when they're not set

The Solo theme is maintained in TryGhost/Themes monorepo and will be updated for v6. Code Injection CSS survives Ghost upgrades unchanged.

Ref: spectralwebservices.com/blog/is-it-safe-to-upgrade-to-ghost-6-0/

---

## Files

```
infer-brand/
├── logos/
│   ├── infer-logo-light.png      ← Publication logo (Liberation Sans proxy)
│   ├── infer-logo-light-2x.png   ← Retina version
│   ├── infer-logo-dark.png       ← Dark bg variant (lighter icon fills)
│   ├── infer-logo-light.svg      ← SVG (needs DM Sans installed to render text)
│   ├── infer-logo-dark.svg       ← SVG dark variant
│   ├── infer-icon-light.svg      ← Icon only, light bg
│   ├── infer-icon-dark.svg       ← Icon only, dark bg
│   ├── infer-favicon.svg         ← Favicon source
│   ├── infer-favicon-{32,180,192,512}.png
│   └── infer-icon-600.png        ← High-res icon
├── code-injection-header.html    ← Paste into Site Header (citations inline)
├── logo-exporter.html            ← Open in browser for pixel-perfect DM Sans logos
└── implementation-guide.md       ← This file
```

---

## Step 10: Enable Members & Email Configuration

Members is what powers subscribe forms, the Portal popup, and eventually paid tiers.
Solo's homepage template conditionally hides the secondary header text AND the subscribe
email form when members is disabled (index.hbs: `{{#if @site.members_enabled}}`). So this
step is required for the homepage to look like the approved mockup.

### 10a: Enable Members

Ghost Admin → Settings → Membership → Access → **"Anyone can sign up"** → Save

This flips `@site.members_enabled` to true, which:
- Shows the Subscribe button in the nav (default.hbs lines 72-82)
- Shows the secondary header text on the homepage (index.hbs line 36)
- Shows the subscribe email form on the homepage (index.hbs lines 40-44)
- Enables Ghost Portal (the popup that handles signup/signin/account)

Ref: ghost.org/help/setup-members/

### 10b: Configure Transactional Email (SMTP)

Ghost sends two types of email that require separate configuration:
1. **Transactional** — magic login links, signup confirmations, password resets. Configured via SMTP.
2. **Bulk/newsletter** — mass emails to subscribers. Configured via Mailgun API in Ghost Admin.

Without transactional email working, the subscribe form accepts emails but never sends
the confirmation magic link. The subscriber gets stuck.

**Coolify-specific approach:** Coolify's default Ghost template uses Docker with a persistent
volume for `/var/lib/ghost/content` but NOT for the Ghost root directory where
`config.production.json` lives. Editing that file directly will be overwritten on container
restart. The solution is environment variables.

Ref: preslav.me/2024/08/21/how-to-properly-configure-email-sending-in-ghost/
(This author uses Coolify specifically and documented this exact problem.)

Ghost uses nconf, which allows nested JSON config to be set via environment variables
using double-underscore (`__`) as the nesting separator.

Ref: docs.ghost.org/config — "A custom configuration file must be a valid JSON file...
changes can be implemented using ghost restart."

**In Coolify**, add these environment variables to your Ghost service:

```
mail__transport=SMTP
mail__from=infer.blog <noreply@infer.blog>
mail__options__service=Mailgun
mail__options__host=smtp.mailgun.org
mail__options__port=587
mail__options__secure=false
mail__options__auth__user=postmaster@your-mailgun-domain
mail__options__auth__pass=your-mailgun-smtp-password
```

Notes:
- Use `smtp.mailgun.org` for US region, `smtp.eu.mailgun.org` for EU region
- The `mail__from` address should match your Mailgun sending domain
- After adding env vars in Coolify, restart the Ghost container
- Ref for env var syntax: blog.ryanhalliday.com/2018/05/ghost-docker-mail-environment-variables

### 10c: Set Up Mailgun Account

If you don't have a Mailgun account yet:

1. Sign up at mailgun.com (requires credit card; has a flex trial with 1,000 free emails)
2. Go to Domains → Add New Domain
3. Add a subdomain like `mg.infer.blog` (using a subdomain is best practice so your
   root domain reputation stays clean)
4. Add the DNS records Mailgun provides to your Cloudflare DNS:
   - TXT records for SPF and DKIM verification
   - CNAME for tracking (optional)
   - MX records (only needed if you want to receive email on that subdomain)
5. Wait for DNS propagation and verify in Mailgun
6. Go to Mailgun → Domain Settings → SMTP credentials
7. Note the SMTP username (usually `postmaster@mg.infer.blog`) and password

Ref: brightthemes.com/blog/ghost-mailgun-config

### 10d: Configure Bulk Email (Newsletter Sending)

This is separate from SMTP and is configured in Ghost Admin, not env vars.

Ghost Admin → Settings → Mailgun:
- **Mailgun region:** US or EU (match your Mailgun account region)
- **Mailgun domain:** `mg.infer.blog` (your Mailgun sending domain)
- **Mailgun private API key:** Found in Mailgun → API Security → Private API key

This enables Ghost to send newsletters to your subscriber list. Without this,
you can collect subscribers but can't email them in bulk.

Ref: ghost.org/docs/config/ — "Mailgun is a service for sending emails and provides
more than adequate resources to send bulk emails at a reasonable price."

### 10e: Test the Full Flow

1. Open infer.blog in an incognito window
2. Enter your own email in the subscribe form
3. Check your inbox for the magic link confirmation email
4. Click the magic link — you should be signed in as a free member
5. Go to Ghost Admin → Members — verify you appear in the list
6. Go to Ghost Admin → Posts → create a test newsletter post → send to yourself

If step 3 fails (no email arrives): check Coolify logs for SMTP errors, verify your
Mailgun domain is verified, and confirm the env vars are set correctly.

### 10f: Portal Customization (Optional)

Ghost Admin → Settings → Membership → Portal

Portal is the popup UI for subscribe/signin/account. It automatically uses your
accent color (#b8510d) for buttons. You can configure:
- Which page Portal shows by default (signup, signin, or account)
- Whether to show the Portal button on your site
- The Portal button icon and style
- Which tiers are visible (for now, just free)

Portal renders in an iframe — our Code Injection CSS doesn't reach it. The copper
accent is applied by Ghost from your admin accent color setting.

### Adding Paid Tiers Later

When you're ready to add paid subscriptions:
1. Connect Stripe in Ghost Admin → Settings → Membership → Stripe
2. Create membership tiers with pricing
3. No CSS changes needed — Ghost and Portal handle the payment UI
4. Solo's template already includes membership-aware conditionals
   (e.g., content gating, upgrade CTAs)

Ref: ghost.org/help/setup-members/

---

## Color Reference

| Token | Hex | Where Set |
|-------|-----|-----------|
| Parchment | #f6f3ef | Solo Admin → background_color |
| Copper | #b8510d | Admin accent color + CSS --infer-accent |
| Hover | #faf8f5 | CSS --infer-bg-hover |
| Sand | #e2ddd5 | CSS --infer-border |
| Bone | #cdc5ba | CSS --infer-border-strong |
| Warm Black | #2c2418 | CSS --infer-code-bg |
| Code Border | #3d3428 | CSS (hardcoded, code block border) |
| Warm Tint | #ede8e0 | CSS --infer-code-inline-bg |
| Inline Code | #7a4a1a | CSS --infer-code-inline-text |
| Code Text | #d4cfc7 | CSS (hardcoded, code block text) |
| Amber | #dba463 | CSS (Prism keyword token) |
| Sage | #a3be8c | CSS (Prism string token) |
| Lilac | #c4a8e0 | CSS (Prism function token) |
