---
type: Note
---
# Claude Design prompt

Build a multi-page document as a single Design Component (.dc.html), styled in the Dualboot Partners Design System, that exports 1:1 to PDF via Cmd/Ctrl+P. Follow this exact structure and styling:

Page model

Each page is a fixed 816px × 1056px div (US Letter at 96dpi), box-sizing: border-box, padding: 60px 72px 100px 72px, overflow: hidden, break-after: page, position: relative, background: #FFFFFF.
In <helmet>: @page { size: letter; margin: 0; }, a light #E9EDF3 screen background with soft page shadows on screen only (@media screen), and @media print resets to white with no shadow. Add <meta name="omelette-owns-print" content="true">.
One [data-page] + [data-screen-label] per page. Each numbered section starts its own page — kicker + title lead the page, content follows.
Cover (page 1, no footer)

Full-bleed #005AF0 blue, white text. White horizontal logo top-left. Mono kicker ("ARCHITECTURE SPECIFICATION · PHASE 1"), 58px extrabold title, Libre Baskerville italic tagline, then a version/author/status row above a hairline rgba(255,255,255,.35) divider.
Every content page

Mono blue kicker (JetBrains Mono, 11px, letter-spacing: .14em, #005AF0, e.g. "02 — GLOSSARY").
28px extrabold #0A1828 H2 title; 17px bold section H3s (accent-blue for component names).
Body 13–14px #2A3644, line-height: 1.6–1.65.
Footer pinned bottom (position:absolute; left/right:72px; bottom:28px), hairline #D7DEE8 top border: Dualboot logo left, [WWW.DUALBOOTPARTNERS.COM](http://WWW.DUALBOOTPARTNERS.COM) + blue page number ("NN OF NN") right — all JetBrains Mono 8.5px.
Type & color system

Inter (400–800), Libre Baskerville italic for single-word emphasis, JetBrains Mono for kickers/labels/technical values.
Blue #005AF0 accent, ink #0A1828, body #2A3644, borders #D7DEE8, muted panels #F2F5F9. No gradients, tight 2–4px radii.
Inline styles only (no CSS classes); apply font-family: 'Inter', sans-serif inline on every page div.
Components to reuse: bordered spec cards in 2/3-col grids, tech-version tables and data tables with mono headers + 2px solid #0A1828 bottom rule + 1px #D7DEE8 row rules, dark navy #0A1828 inline "flow" code strip, numbered happy-path lists.

Copy the logos (logo-horizontal.svg, logo-horizontal-white.svg) from the Dualboot design system into assets/.

The content of the document is this markdown:
