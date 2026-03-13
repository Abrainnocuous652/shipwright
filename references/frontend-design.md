# Frontend Design & UI Craft Guidelines

## Design Thinking — Before Writing Any CSS

Before coding any frontend, commit to a clear aesthetic direction:

- **Purpose:** What problem does this interface solve? Who uses it?
- **Tone:** Pick a direction and commit fully: brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian. The key is intentionality, not intensity.
- **Differentiation:** What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL:** Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

## Typography

Choose fonts that are beautiful, unique, and interesting.

**NEVER** default to Inter, Roboto, Arial, or system fonts for headings. Pair a distinctive display font with a refined body font. Vary between projects — never converge on the same safe choices (Space Grotesk, for example).

- Headings: tight line-height (1.1-1.3), consider slight negative letter-spacing (-0.01em to -0.025em) for premium feel
- Body: weight 400+, line-height 1.5-1.7, minimum 16px on mobile
- Consistent type scale (ratio-based: 1.25 or 1.333, no arbitrary sizes)
- Font weights: light mode often benefits from slightly heavier body text (400, not 300)

## Color & Theme

Commit to a cohesive palette. Use CSS variables for every color. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.

**NEVER** use purple gradients on white backgrounds, or any other cliché AI-generated color scheme.

### Light Mode Specifics
- **Page background:** Subtle off-white (e.g., `#FAFAFA`, `#FAF9F7`) — never pure `#FFFFFF` everywhere.
- **Cards/elevated surfaces:** Pure or near-pure white (`#FFFFFF`) for natural hierarchy through background contrast.
- **Primary text:** Rich near-black (`#1A1A1A`, `#111827`, `#0F172A`) — never pure `#000000` (too harsh).
- **Secondary text:** Mid-grey (`#6B7280`, `#64748B`) that meets WCAG AA (4.5:1 minimum).
- **Tertiary/placeholder:** Lighter grey, never so light it's invisible.
- **Shadows:** Soft, layered:
  - Cards: `0 1px 3px rgba(0,0,0,0.04), 0 1px 2px rgba(0,0,0,0.06)`
  - Dropdowns: `0 4px 6px -1px rgba(0,0,0,0.05), 0 2px 4px -2px rgba(0,0,0,0.05)`
  - Modals: `0 10px 25px -5px rgba(0,0,0,0.08), 0 8px 10px -6px rgba(0,0,0,0.04)`
- **Borders:** `rgba(0,0,0,0.06)` to `rgba(0,0,0,0.10)`. Prefer whitespace and background shifts over borders.
- **Accent colors:** May need to be slightly deeper/muted than dark mode variant. Limit usage to CTAs and key indicators.

### Dark Mode Specifics
- **Never** use pure `#000000` as base — creates harsh OLED void effect.
- **Surface hierarchy (4-5 levels):**
  - Level -1 (deepest): `#09090B` to `#0C0C0E`
  - Level 0 (page): `#111113` to `#141416`
  - Level 1 (cards): `#1A1A1E` to `#1C1C20`
  - Level 2 (dropdowns): `#222228` to `#27272A`
  - Level 3 (modals): `#2A2A30` to `#2C2C32`
- Add barely perceptible color tint to greys (cool blue or warm neutral) — avoids lifeless grey.
- **Primary text:** Off-white (`#ECECEF`, `#E4E4E7`) — never pure `#FFFFFF` (eye strain).
- **Secondary text:** ~`#9898A0`, `#A0A0A8`. Must meet WCAG AA.
- **Accent colors:** Reduce saturation 10-20% from light mode. Colors pop dramatically on dark — what's muted in light becomes neon.
- **Borders:** `rgba(255,255,255,0.06)` to `rgba(255,255,255,0.10)`. Semi-transparent adapts to any surface.
- **Shadows:** Much darker/spread: `0 2px 8px rgba(0,0,0,0.3), 0 1px 3px rgba(0,0,0,0.4)` for cards.
- **Glow:** Subtle accent glow behind focus rings, primary buttons on hover. 1-2 glowing elements per viewport maximum.
- **Scrollbars:** Styled to match. Track transparent, thumb `rgba(255,255,255,0.12)`, thin (6-8px).

## Motion & Micro-Interactions

Every animation must serve a purpose: feedback, orientation, or reducing perceived latency. Remove anything gratuitous.

- **Page/route transitions:** Fast fade or slide (150-300ms). Never slower.
- **Button feedback:** Subtle scale (`transform: scale(0.98)`) or color shift on press. Smooth hover (150ms ease).
- **Skeleton loaders:** For ALL async content. Shimmer styled for both light and dark.
- **Smooth scroll** behavior across the app.
- **Toast/notification:** Enter and exit with slide + fade.
- **Modal/dropdown:** Scale + fade (`scale(0.97) opacity(0)` → `scale(1) opacity(1)`, 150-200ms).
- **List stagger:** Dashboard cards, search results loading in with slight delays.
- **Focus-visible:** Accent-colored ring, keyboard-only (not on click).
- **`prefers-reduced-motion`:** Respect it. Disable/simplify all non-essential animations.

Focus on high-impact moments: one well-orchestrated page load with staggered reveals creates more delight than scattered micro-interactions everywhere.

## Spatial Composition & Visual Details

Create atmosphere and depth rather than defaulting to solid colors:
- Gradient meshes, noise textures, geometric patterns, layered transparencies
- Dramatic shadows, decorative borders, grain overlays
- Unexpected layouts: asymmetry, overlap, diagonal flow, grid-breaking elements
- Generous negative space OR controlled density — not muddy middle ground

Match implementation complexity to aesthetic vision. Maximalist designs need elaborate code. Minimalist designs need restraint, precision, and careful spacing. Elegance comes from executing the vision well.

## Component States — The Complete List

Every interactive element must have ALL of these states. Missing states = unfinished software:

- **Resting:** Default appearance
- **Hover:** Subtle feedback (background shift, shadow change, color shift). 150ms transition.
- **Focus:** Visible ring (accent color, keyboard-only via `:focus-visible`). Must work in both themes.
- **Active/pressed:** Slightly darker than hover, optional subtle scale-down.
- **Disabled:** Reduced opacity OR muted colors. Never just greyed text with no other indication.
- **Loading:** Spinner, skeleton, or pulsing indicator. Never blank.
- **Error:** Red/destructive accent + icon + message. Never color alone.
- **Empty:** Helpful illustration/icon + clear messaging about what to do next. Never blank void.

### Specific Component Guidance

**Buttons:** Primary (solid accent, confident), secondary (subtle border or light fill, non-competing), ghost (barely-there hover). Consistent border-radius everywhere.

**Inputs:** Resting (subtle border), focus (accent ring + transition), error (red border + `aria-describedby` message), disabled (muted). Placeholder text appropriately subtle.

**Cards:** Sufficient padding (16-24px), consistent radius, subtle elevation. Content hierarchy: title → body → actions.

**Tables:** Subtle row hover (`rgba(0,0,0,0.02)` light / `rgba(255,255,255,0.03)` dark). Headers distinct through weight, not heavy borders. Mobile: scroll + sticky first column or card layout.

**Navigation:** Active item clearly indicated (accent tint or indicator). Hover subtle and fast. Sidebar background slightly different from main content.

**Modals:** Semi-transparent backdrop. Refined shadow. Enter/exit animation. Consider `backdrop-filter: blur(8px)` in dark mode.

**Toasts:** Light, unobtrusive. White/tinted background with left-edge color accent for type.

## Icons, Illustrations & Visual Assets

- Use ONE icon library consistently (Lucide, Heroicons, Phosphor, etc.). Never mix libraries.
- Icon color follows text hierarchy (primary for meaningful, secondary for supporting).
- Empty states: helpful illustration + "what to do next" messaging. Not blank.
- Error pages (404, 500): custom designs matching the app personality.
- Loading: skeleton screens for known content shapes, centered spinner for unknown.
- Logos: light-mode and dark-mode variants. Both tested.
- Favicons: set for all contexts (browser tab, mobile home screen, PWA).
- If illustrations exist: consistent style, palette matches design system, responsive sizing.

## Anti-Patterns — NEVER Do These

- Inter, Roboto, Arial, system fonts for headings
- Purple gradients on white backgrounds
- Predictable card-grid layouts with no visual personality
- Cookie-cutter components from UI libraries with zero customization
- Pure `#000000` text on pure `#FFFFFF` background
- Pure `#000000` background in dark mode
- Hard grey borders (`#333`, `#444`) in dark mode
- Heavy drop shadows that look like drawn borders
- Hover-dependent interactions with no mobile fallback
- Missing component states (especially empty, loading, error, disabled)
- Animations slower than 300ms or that don't serve a purpose
- Different border-radius values on similar components
- Mixing icon libraries or inconsistent icon weights
