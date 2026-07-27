# BDA Study Lab — design system

## Product and job to be done

BDA Study Lab is an exam-preparation SPA for advanced database coursework. It translates lecture PDFs into rigorous, interactive study modules. Topic 14 introduces Natural Language Processing (PLN/NLP) faithfully to the provided 30-slide lecture while expanding definitions, contrasts, examples, edge cases, ethical risks and high-difficulty exam reasoning.

## Information architecture for Topic 14

The topic workspace has three tabs:

1. **Contenido** — editorial introduction, the four NLP stages, application map, paradigm comparison, animated end-to-end pipeline, limitations/ethics and exam synthesis.
2. **Quiz** — difficult questions with immediate explanations and a final review.
3. **Actividades** — the PDF’s three phases: individual research, critical group debate and group synthesis.

The content screen should include:

- A clear hero that states NLP as the bridge between unstructured human language and computational representations.
- An animated sentence-analysis pipeline: raw phrase → lexical tokenization → syntactic relations → semantic/contextual interpretation → task-specific output.
- A selectable application map covering task automation, document processing, translation, text mining/sentiment, semantic search and content generation.
- A comparison matrix for rule-based, statistical and deep-learning NLP.
- An interactive “same sentence, three paradigms” laboratory.
- A second flow aligned with the lecture: processing → extraction → analysis → training, with a clarification that training is normally an offline model-development lifecycle rather than a mandatory per-request step.
- High-difficulty examination notes: ambiguity, polysemy, negation, sarcasm, cultural context, distribution shift, hallucination, bias, privacy and explainability.
- Direct access to the source PDF.

## Visual language

Use only the existing system:

- `#17231d` ink, `#66726b` muted, `#f4f1e8` paper, `#fffefa` surface, `#d8d3c6` line.
- Topic 14 accent: use the existing blue `#315d72` and blue-soft `#dde9ed`; forest remains success/action and amber remains warning/trap.
- Inter for UI and prose, Lora for headings, IBM Plex Mono for labels and technical markers.
- 248px desktop sidebar; workspace content centered up to 1280px; cards with 6–10px radius and thin borders.
- Avoid gradients, glassmorphism, excessive shadows and decorative stock imagery. Prefer diagrams made from CSS boxes, connectors, arrows and compact annotations.

## Motion and interaction

- 220–350ms opacity/translate/scale transitions.
- Animated pipelines reveal one stage at a time and always provide explicit controls.
- Hover is supplementary; all information must be reachable by click and keyboard.
- Respect `prefers-reduced-motion: reduce`.
- Animations must explain a transformation or relationship, never be decorative.

## Responsive and accessibility requirements

- Collapse the sidebar into the existing horizontal navigation below 1000px.
- Collapse comparison tables and diagrams safely below 680px.
- Keep minimum 44px interactive targets, visible focus states, semantic headings, descriptive labels and sufficient contrast.
- Do not encode correctness or state by color alone.
