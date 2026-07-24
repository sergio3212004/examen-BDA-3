# BDA Study Lab — Design system

## Product and learning model

BDA Study Lab is a Spanish-language responsive study workspace for an Ingeniería Informática student preparing for a deliberately difficult, technical exam. It is not a passive summary page. The interface must support:

- progressive study by topic;
- conceptual synthesis plus expansions, examples, applications and edge cases;
- active recall with revealable answers;
- high-difficulty, technically worded analysis questions;
- visual comparison of trade-offs;
- a separate, clearly labeled section containing complete answers to every learning activity in the source PDF;
- local progress and completion states;
- focused reading on desktop and comfortable single-column study on mobile.

Core subject matter: horizontal scalability, large-scale distributed data processing, mash-ups, RDBMS vs NoSQL, cost-based control optimization (CBoC), distributed files/tables/locks, fault tolerance, active/standby failover, redundancy and network topology, large-scale system testing, environment automation, high-volume data ingestion, observability and validation.

## Information architecture

One continuous application shell with four primary views/tabs:

1. `Resumen`: learning dashboard, progress, learning objectives, concept map and recommended next step.
2. `Temas`: six expandable learning modules with synthesis, deep explanation, examples, “why it matters,” exam traps and mini-checks.
3. `Practicar`: flashcards, scenario questions and difficult analysis questions with hidden answers and self-assessment.
4. `Actividades resueltas`: the 15 source questions grouped into Autoaprendizaje, Debate crítico and Reflexión, each with structured model answers.

A persistent desktop sidebar becomes a compact top bar plus horizontal tab row on small screens. Include a small visible progress indicator and “Continuar estudiando” action.

## Visual direction

Academic editorial workspace with a restrained technical feel: “annotated engineering notebook meets modern data observability dashboard.” Avoid generic SaaS gradients, playful gamification, neon, glassmorphism and cartoon imagery.

Use only these tokens:

- `--ink: #17231D` — primary text and dark controls
- `--muted: #66726B` — secondary text
- `--paper: #F4F1E8` — page canvas
- `--surface: #FFFEFA` — cards/readers
- `--line: #D8D3C6` — borders and rules
- `--forest: #1F5C45` — main brand/accent
- `--forest-soft: #DCE9E1` — selected and success backgrounds
- `--amber: #C97A32` — warnings, exam traps, difficulty
- `--amber-soft: #F5E4D2`
- `--blue: #315D72` — distributed systems diagrams and concept links
- `--blue-soft: #DDE9ED`

Typography:

- UI and body: `Inter`, fallback `system-ui, sans-serif`.
- Editorial headings: `Lora`, fallback `Georgia, serif`.
- Technical labels and small metadata: `IBM Plex Mono`, fallback `ui-monospace, monospace`.
- Body 16px/1.65; compact metadata 12px/1.4; h1 44–52px desktop and 34px mobile; h2 28–34px.

Layout and shape:

- Desktop max content width 1440px; sidebar 248px; reading column ideally 760–860px.
- 8px spacing base; frequent spacing 8, 12, 16, 24, 32, 48.
- Radius 6px for controls, 10px for cards; cards use visible 1px borders rather than floating shadows.
- Shadow only for overlays: `0 16px 40px rgba(23,35,29,.12)`.
- Use thin grid lines, section indices such as `01 / 06`, small uppercase mono labels and restrained highlighted annotations.

## Components and states

- Sidebar logo mark: simple database-cylinder line icon paired with `BDA / STUDY LAB`.
- Navigation items have icon, label and active left rule/fill.
- Topic cards show index, title, short learning goal, difficulty chip and completion.
- Reader sections use clear anchors: `Idea central`, `Cómo funciona`, `Ejemplo aplicado`, `Caso límite`, `Trampa de examen`.
- Comparison tables must remain horizontally scrollable on small screens.
- Flashcards flip or reveal via an explicit button and remain keyboard accessible.
- Question answers start hidden; after reveal, show `Respuesta modelo`, `Razonamiento` and `Criterio de evaluación`.
- Controls need hover, focus-visible, active and disabled states. Minimum touch target 44px.

## Motion

Use 160–220ms ease-out transitions for accordions, tabs, progress and answer reveals. Respect `prefers-reduced-motion`. No decorative looping animation.

## Responsive behavior

- >=1100px: fixed left sidebar, dense dashboard grid and two-column supporting content.
- 720–1099px: compact top navigation, two-column cards where space permits.
- <720px: single column, horizontally scrollable tabs/tables, sticky bottom “Continuar” action only when helpful.

## Accessibility

WCAG AA contrast, semantic headings, labels on icon buttons, visible focus rings, no color-only state communication and properly expanded/collapsed ARIA state.
