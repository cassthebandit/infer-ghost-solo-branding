# infer-ghost-solo-branding

Brand identity and Code Injection CSS for [infer.blog](https://infer.blog), built on Ghost's [Solo](https://github.com/TryGhost/Solo) theme.

No theme fork. No custom templates. Everything here works through Ghost Admin settings and Code Injection — the site survives Ghost updates and theme updates without rework.

**Blog post:** [Branding Ghost Solo Without Touching a Theme File](https://infer.blog/branding-ghost-solo-code-injection/)

## What's in here

```
├── code-injection-header.html   ← Paste into Ghost Admin → Settings → Code Injection → Site Header
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

## Quick start

1. Install Solo in Ghost Admin → Settings → Design & branding → Change theme
2. Follow `implementation-guide.md` for Admin settings (background color, accent, navigation, header layout)
3. Paste the contents of `code-injection-header.html` into Ghost Admin → Settings → Code Injection → Site Header
4. Upload `infer-logo-light.png` as your publication logo
5. Upload `infer-favicon-192.png` as your publication icon

The implementation guide covers members/email setup (Coolify + Mailgun), font picker gotchas, and a verification checklist.

## Built with

This project was built with [Claude](https://claude.ai) Opus. Opus wrote the CSS, generated the logo assets, built the mockups, and produced the implementation guide. Brand decisions, research direction, and source verification were done by [Daniel Soteldo](https://infer.blog/about/).

## License

MIT
