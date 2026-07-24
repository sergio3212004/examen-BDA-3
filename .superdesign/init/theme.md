# Theme

## Compact token summary

- Framework: Vue 3.5 + Vite 8; vanilla global CSS; no component library.
- Colors: text `#6b6375`, heading `#08060d`, background `#fff`, border `#e5e4e7`, accent `#aa3bff`.
- Dark colors: text `#9ca3af`, heading `#f3f4f6`, background `#16171d`, border `#2e303a`, accent `#c084fc`.
- Fonts: system UI sans; `ui-monospace` for code.
- Type: root 18px/145%, h1 56px, h2 24px; at <=1024px root 16px, h1 36px, h2 20px.
- Radius: 4–6px. Shadow: standard soft 10px/15px elevation.
- Breakpoint: `1024px`.
- Current tokens belong to the generic starter and may be replaced for the new study product.

## Raw sources

### `src/style.css`

```css
:root {
  --text: #6b6375;
  --text-h: #08060d;
  --bg: #fff;
  --border: #e5e4e7;
  --code-bg: #f4f3ec;
  --accent: #aa3bff;
  --accent-bg: rgba(170, 59, 255, 0.1);
  --accent-border: rgba(170, 59, 255, 0.5);
  --social-bg: rgba(244, 243, 236, 0.5);
  --shadow: rgba(0,0,0,.1) 0 10px 15px -3px, rgba(0,0,0,.05) 0 4px 6px -2px;
  --sans: system-ui, 'Segoe UI', Roboto, sans-serif;
  --heading: system-ui, 'Segoe UI', Roboto, sans-serif;
  --mono: ui-monospace, Consolas, monospace;
  font: 18px/145% var(--sans);
  color: var(--text);
  background: var(--bg);
}
@media (prefers-color-scheme: dark) {
  :root {
    --text: #9ca3af;
    --text-h: #f3f4f6;
    --bg: #16171d;
    --border: #2e303a;
    --code-bg: #1f2028;
    --accent: #c084fc;
    --accent-bg: rgba(192,132,252,.15);
    --accent-border: rgba(192,132,252,.5);
    --social-bg: rgba(47,48,58,.5);
  }
}
body { margin: 0; }
#app { width: 1126px; max-width: 100%; margin: 0 auto; min-height: 100svh; }
@media (max-width: 1024px) { :root { font-size: 16px; } }
```
