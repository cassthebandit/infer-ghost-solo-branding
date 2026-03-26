# infer-ghost-solo-branding

Brand identity and Code Injection CSS/JS for [infer.blog](https://infer.blog), built on Ghost's [Solo](https://github.com/TryGhost/Solo) theme.

No theme fork. No custom templates. Everything here works through Ghost Admin settings and Code Injection — the site survives Ghost updates and theme updates without rework.

**Blog post:** [Branding Ghost Solo Without Touching a Theme File](https://infer.blog/branding-ghost-solo-code-injection/)

## What's in here

```
├── code-injection-header.html   ← Paste into Ghost Admin → Code Injection → Site Header
├── code-injection-footer.html   ← Paste into Ghost Admin → Code Injection → Site Footer
├── implementation-guide.md      ← Step-by-step setup: Admin settings, members, email, verification
├── infer-solo-mockup.html       ← Design mockup (open in browser) — homepage, article, mobile views
├── logo-exporter.html           ← Renders logos in DM Sans via Google Fonts (open in browser, screenshot)
└── logos/
    ├── infer-logo-light.png     ← Upload as Ghost publication logo
    ├── infer-logo-light-2x.png  ← Retina version
    ├── infer-logo-dark.png      ← Dark background variant
    ├── infer-logo-light.svg     ← SVG source (needs DM Sans installed to render text)
    ├── infer-logo-dark.svg      ← SVG dark variant
    ├── infer-icon-light.svg     ← Missing Piece icon only, light background
    ├── infer-icon-dark.svg      ← Icon only, dark background
    ├── infer-favicon.svg        ← Favicon source
    ├── infer-favicon-32.png     ← Browser tab favicon
    ├── infer-favicon-180.png    ← Apple touch icon
    ├── infer-favicon-192.png    ← Upload as Ghost publication icon
    ├── infer-favicon-512.png    ← PWA icon
    └── infer-icon-600.png       ← High-res icon
```

## The approach

Ghost's Solo theme ships with oversized defaults — hero titles up to 93px, card titles at 83px, 192px footer padding. The Code Injection CSS overrides all of that with the Craftsman Modern palette:

| Token | Hex | Where set |
|-------|-----|-----------|
| Parchment | `#f6f3ef` | Solo Admin → background_color |
| Copper | `#b8510d` | Ghost Admin accent color + CSS |
| Warm Black | `#2c2418` | CSS (code block backgrounds) |

Fonts: [DM Sans](https://fonts.google.com/specimen/DM+Sans) for body/headings, [DM Mono](https://fonts.google.com/specimen/DM+Mono) for code and dates.

The CSS also converts Solo's Classic feed from a side-by-side flex layout to a card grid (image on top, title below, 3 columns → 2 → 1 responsive), fixes a mobile dead space bug in Solo's header, and overrides article page sizing.

### All Tags Display

Solo's `post.hbs` uses `{{#if primary_tag}}` which only renders the first tag on post pages. The Site Footer script fixes this without touching theme files — it reads Ghost's existing `article:tag` meta tags and `tag-{slug}` body classes, wraps all tags in a flex container, and injects the missing ones as pill links alongside the primary tag. Works on desktop (300px sticky sidebar) and mobile (inline below author info).

## Quick start

1. Install Solo in Ghost Admin → Settings → Design & branding → Change theme
2. Follow `implementation-guide.md` for Admin settings (background color, accent, navigation, header layout)
3. Paste `code-injection-header.html` into Ghost Admin → Settings → Code Injection → Site Header
4. Paste `code-injection-footer.html` into Ghost Admin → Settings → Code Injection → Site Footer
5. Upload `infer-logo-light.png` as your publication logo
6. Upload `infer-favicon-192.png` as your publication icon

The implementation guide covers members/email setup (Coolify + Mailgun), the all-tags fix, and a verification checklist.

## Built with

This project was built with [Claude](https://claude.ai) Opus. Opus wrote the CSS, generated the logo assets, built the mockups, and produced the implementation guide. The all-tags display fix was built with Claude Opus 4.6. Brand decisions, research direction, and source verification were done by [Daniel Soteldo](https://infer.blog/about/).

## License

MIT
