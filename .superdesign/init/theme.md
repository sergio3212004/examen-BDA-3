# Theme tokens

## Compact token summary

- Fonts: Inter for UI/body, Lora for headings/editorial emphasis, IBM Plex Mono for labels, indices, status and code-like metadata.
- Core colors: ink `#17231d`, muted `#66726b`, paper `#f4f1e8`, surface `#fffefa`, line `#d8d3c6`.
- Semantic accents: forest `#1f5c45` / soft `#dce9e1`; amber `#c97a32` / soft `#f5e4d2`; blue `#315d72` / soft `#dde9ed`.
- Layout: fixed 248px sidebar on desktop, centered content up to 1280px, page padding 48px/40px/80px.
- Radius: 4–10px, restrained and editorial. Shadows are rare and reserved for active animated states.
- Breakpoints: 1000px for sidebar → top bar and layout collapse; 680px for single-column mobile.
- Motion: 220–350ms transitions, small translate/scale/fade; all disabled under `prefers-reduced-motion`.

## Raw source

```css
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=Inter:wght@400;500;600;700&family=Lora:ital,wght@0,500;0,600;1,500&display=swap');

:root{--ink:#17231d;--muted:#66726b;--paper:#f4f1e8;--surface:#fffefa;--line:#d8d3c6;--forest:#1f5c45;--forest-soft:#dce9e1;--amber:#c97a32;--amber-soft:#f5e4d2;--blue:#315d72;--blue-soft:#dde9ed;font-family:Inter,system-ui,sans-serif;color:var(--ink);background:var(--paper);font-synthesis:none;text-rendering:optimizeLegibility}
*{box-sizing:border-box}html{scroll-behavior:smooth}body{margin:0;background:var(--paper)}
@media(prefers-reduced-motion:reduce){*{scroll-behavior:auto!important;transition:none!important;animation:none!important}}
```

Complete stylesheet: `src/style.css`.
