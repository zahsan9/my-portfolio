# Portfolio Typography System

The portfolio uses **Alte Haas Grotesk** throughout. A single family keeps the visual language cohesive; hierarchy comes from size, weight, line height, and tracking. `--sans`, `--serif`, `--serif-display`, `--mono`, and `--card-title` are compatibility aliases for existing component rules and all resolve to this face.

## Typeface and weights

- Local regular face: `AlteHaasGroteskRegular.ttf`, weight 400.
- Local bold face: `AlteHaasGroteskBold.ttf`, weight 700.
- Use 400 for prose and quiet labels; use 700 for titles and emphasis. The font has no separate 500 or 600 face, so avoid relying on intermediate weights or synthetic bolding.
- Inter is a fallback only, not a second typographic voice.

## Size roles

The final type-scale block in `portfolio.html` owns shared sizes. Change these tokens rather than adding another one-off size for the same role.

| Token | Size | Use |
| --- | ---: | --- |
| `--fs-4xl` | 48px | About and Experience page statements |
| `--fs-3xl` | 36px | Portfolio name |
| `--fs-2xl` | 36px | Case-study hero titles |
| `--fs-xl` | 28px | Desktop card/page titles and case-study questions |
| `--fs-lg` | 20px | Case-study ledes and section heads |
| `--fs-base` | 16px | Body copy and primary compact-card headings |
| `--fs-md` | 15px | Compact supporting copy and figure titles |
| `--fs-sm` | 13px | Secondary copy and metadata values |
| `--fs-xs` | 11px | Labels, dates, captions, and eyebrows |

Mobile sizes are tuned in the existing responsive rules. Keep body copy at or above 15px where space allows; 11px is reserved for short metadata, never paragraphs.

## Line height and tracking

- Body copy: 1.5–1.65, with long case-study text near 1.5 and a readable measure.
- Display headlines: about 1–1.05; card titles: about 1.1; section headings: 1.2–1.35.
- Tighten display tracking modestly (around -0.04em), titles around -0.03em, and keep small headings close to neutral. Avoid a single tracking value across every size.
- Labels use `--tracking-label`; compact UI uses `--tracking-ui`.

## Maintenance

1. Reuse the existing role token for typography with the same purpose.
2. Use explicit component exceptions only when the content or available space requires them, and keep the exception in the responsive/component rules next to that component.
3. Preserve the local face and its licensed files; do not add a second editorial or monospaced family without intentionally revising this system.
4. Use tabular numerals for changing values and `text-wrap: balance` for headlines or `text-wrap: pretty` for longer copy where supported.

