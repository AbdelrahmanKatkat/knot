# Design system — "The Edge"

Source: the painting of a lone figure in an orange vest on a luminous cliff edge, facing a sky of clouds. Metaphor: *the moment before clarity* — a quiet figure, a big turbulent canvas, one warm point of focus. The app should feel calm in the middle of a loud meeting.

## Palette (tokens in `ui/src/theme/tokens.css`)
| Token | Hex | Painting source | Use |
|---|---|---|---|
| `--ink` | `#0D0A1E` | cliff shadow | dark background, text on light |
| `--dusk` | `#2A1B5C` | violet storm clouds | dark surfaces, sidebar |
| `--violet` | `#6A2C9B` | cliff purple | secondary accent, tags |
| `--cobalt` | `#1E57B8` | deep blue sky | links, info, focus rings |
| `--teal` | `#1FA3A6` | green-teal clouds | success, "processed" |
| `--mist` | `#CFE4F3` | pale sky | light background, borders |
| `--cloud` | `#F5F1E8` | cloud white | light surfaces, text on dark |
| `--gold` | `#F4B41A` | sunlit rock | highlights, key moments |
| `--ember` | `#F47B20` | orange rock | warnings, progress |
| `--vest` | `#E8402A` | the figure's vest | **the one primary action** (Record), live indicator |
| `--magenta` | `#C2358E` | rock gradient | decisions/emphasis chips |

Rule: `--vest` appears at most once per screen (the figure is the single focal point). Verify text contrast ≥ 4.5:1 (cloud on dusk, ink on mist).

## Themes
Dark ("Dusk", default): `--ink` → `--dusk` gradient background, cloud text. Light ("Day"): `--mist` → `--cloud` background, ink text.

## Look and feel
- Background: soft painted-cloud gradient with subtle canvas grain (CSS noise, ≤3% opacity); respect `prefers-reduced-motion`.
- Cards: slightly rough "impasto" edge (1px gradient border gold→magenta on hover), radius 14px, no heavy shadows.
- Type: headings in a humanist serif (Fraunces), UI in Inter, transcript in a readable mono-ish (IBM Plex Sans). RTL-safe (logical CSS properties).
- Layout: left rail (meetings), center (note/transcript), right drawer (follow-up chat). Laptop-first: min width 1024, comfortable at 1280–1536.
- Hero/empty state: simplified SVG of the cliff + figure; the figure looks toward the cloud where a summary will appear.
- Recording: the Record button is the vest-orange circle; while live, a gold waveform brushes across the top edge.
- Motion: clouds drift slowly on idle; processing stages animate as a figure walking a path along the cliff (4 stages: Transcribe → Summarize → Publish → Index).
- Action items: gold checkboxes; decisions: magenta chips; open questions: cobalt.

## Accessibility
Keyboard-first (global record hotkey), visible focus ring in `--cobalt`, color never the only signal, WCAG AA.
