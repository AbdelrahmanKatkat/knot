---
name: knot-design-system
description: Apply the Knot UI design system inspired by the cliff-edge painting (palette, themes, components, motion). Use for any UI work in ui/ or desktop/.
---

# Knot design system

Source of truth: `docs/DESIGN.md`. Tokens: `ui/src/theme/tokens.css` (create from the palette table if missing).

Checklist for every screen:
- Colors only via CSS variables (`--ink --dusk --violet --cobalt --teal --mist --cloud --gold --ember --vest --magenta`); no raw hex.
- `--vest` (orange-red) is the single primary action per screen (Record) — the lone figure.
- Dark "Dusk" default plus light "Day"; both must pass WCAG AA.
- Laptop-first layout: left meetings rail, center note/transcript, right follow-up chat drawer; min width 1024.
- Fraunces headings, Inter UI text; logical CSS properties so RTL works.
- Empty/hero states use the cliff-and-figure SVG; processing uses the 4-stage path animation; honor `prefers-reduced-motion`.
- Action items gold, decisions magenta, open questions cobalt, success teal.
- Keyboard accessible, visible `--cobalt` focus ring, color never the only signal.
Reference image of the painting: `docs/assets/reference-painting.jpg` (add it when available).
